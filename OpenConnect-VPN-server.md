OpenConnet Server（ocserv）是通过实现Cisco的AnyConnect协议，用DTLS作为主要的加密传输协议。我认为它的主要好处在于——

- AnyConnect的VPN协议默认使用UDP DTLS作为数据传输，但如果有什么网络问题导致UDP传输出现问题，它会利用最初建立的TCP TLS通道作为备份通道，降低VPN断开的概率。
- AnyConnect作为Cisco新一代的VPN解决方案，被用于许多大型企业，这些企业依赖它提供正常的商业运作，这些正常运作对应的经济效益（读作GDP），是我们最好的伙伴。
- OpenConnet的架设足够麻烦，我的意思是，如果你不是大型企业，你会用AnyConnect的概率无限趋近于零。再者，如果它足够简单，我就不用写这篇文章了。
  至于它的自定义路由表支持，我觉得都是次要了。

介绍到此，让我们按步骤干好事情。

（下文选用 Debian8/Ubuntu14 和 ocser v0.11.8 作为标准环境）  

## 配置环境

1. 安装依赖

   ```shell
   apt-get install -y build-essential pkg-config libgnutls28-dev libreadline-dev libseccomp-dev libwrap0-dev libnl-nf-3-dev liblz4-dev libev-dev autogen
   ```


1. 编译安装

   ```shell
   # 查看最新版本 http://www.infradead.org/ocserv/download.html
   wget ftp://ftp.infradead.org/pub/ocserv/ocserv-0.11.8.tar.xz
   tar xvf ocserv-*.tar.xz && cd ocserv-*
   ./configure 
   make && make install && cd ..
   ```



## 配置 OpenConnect Server

1. 配置证书

   ```
   安装证书工具  
   apt-get install gnutls-bin
   cd ~ && mkdir certificates && cd certificates
   ```

   在此目录下创建一个名为`vi ca.tmpl` 的CA证书模板，写入如下语句：

   ```
   cn = "Your CA name" 
   organization = "Your fancy name" 
   serial = 1 
   expiration_days = 3650
   ca 
   signing_key 
   cert_signing_key 
   crl_signing_key
   ```

   生成CA密钥和证书

   ```
   certtool --generate-privkey --outfile ca-key.pem
   certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile ca-cert.pem
   ```

   然后我们生成服务器证书`vi server.tmpl`，这里注意` cn` 项必须对应你服务器的域名或IP，内容如下：

   ```
   cn = "Your hostname or IP" 
   organization = "Your fancy name" 
   expiration_days = 3650
   signing_key 
   encryption_key
   tls_www_server
   ```

   生成Server密钥和证书

   ```
   certtool --generate-privkey --outfile server-key.pem
   certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem
   ```

   把证书移动到合适的地方：

   ```
   cp ca-cert.pem /etc/ssl/private/my-ca-cert.pem
   cp server-cert.pem /etc/ssl/private/my-server-cert.pem
   cp server-key.pem /etc/ssl/private/my-server-key.pem
   ```

2. 准备配置文件

   我们把配置文件放到 ocserv 默认读取的位置：

   ```shell
   mkdir /etc/ocserv
   cd ~/ocserv*
   cp doc/sample.config /etc/ocserv/ocserv.conf
   vi /etc/ocserv/ocserv.conf
   ```

   配置文件可以[官方手册](http://www.infradead.org/ocserv/manual.html)手册来写，不过这里我们重点要确保以下条目正确：

   ```
   # 登陆方式，目前先用密码登录
   auth = "plain[/etc/ocserv/ocpasswd]"
    
   # 允许同时连接的客户端数量
   max-clients = 4
    
   # 限制同一客户端的并行登陆数量
   max-same-clients = 2
    
   # 服务监听的IP（服务器IP，可不设置）
   listen-host = 1.2.3.4
    
   # 服务监听的TCP/UDP端口（选择你喜欢的数字）
   tcp-port = 9000
   udp-port = 9001
    
   # 自动优化VPN的网络性能
   try-mtu-discovery = true
    
   # 确保服务器正确读取用户证书（后面会用到用户证书）
   cert-user-oid = 2.5.4.3
    
   # 服务器证书与密钥
   ca-cert = /etc/ssl/private/my-ca-cert.pem
   server-cert = /etc/ssl/private/my-server-cert.pem
   server-key = /etc/ssl/private/my-server-key.pem
    
   # 客户端连上vpn后使用的dns
   dns = 8.8.8.8
   dns = 8.8.4.4
    
   # 注释掉所有的route，让服务器成为gateway
   #route = 192.168.1.0/255.255.255.0
    
   # 启用cisco客户端兼容性支持
   cisco-client-compat = true

   #据说可以优化速度
   output-buffer = 23000 
   try-mtu-discovery = true 
   ```

3. 测试服务器

   创建一个登陆用的用户名与密码
   `ocpasswd -c /etc/ocserv/ocpasswd your-username`  

   修改内核设置，使得支持转发，`vi /etc/sysctl.conf`,将`net.ipv4.ip_forward=0`改为`net.ipv4.ip_forward=1` , 保存生效`sysctl -p`  

   查看网卡信息`ifconfig`用于修改防火墙  
   开启NAT转发, 开启端口(对应上面配置的端口)

   ```
   iptables -t nat -A POSTROUTING -j MASQUERADE
   iptables -A INPUT -p tcp --dport 443 -j ACCEPT
   iptables -A INPUT -p udp --dport 443 -j ACCEPT
   # 规则保存 Ubuntu
   iptables-save > /etc/iptables.rules
   # 规则保存 Debian
   apt-get install iptables-persistent -y
   service netfilter-persistent save
   ```

   使用`iptables -t nat -L`来验证转发是否开启成功  

   ```
   Chain PREROUTING (policy ACCEPT)
   target     prot opt source               destination
    
   Chain INPUT (policy ACCEPT)
   target     prot opt source               destination
    
   Chain OUTPUT (policy ACCEPT)
   target     prot opt source               destination
    
   Chain POSTROUTING (policy ACCEPT)
   target     prot opt source               destination
   MASQUERADE  all  --  192.168.1.0/24       anywhere
   ```

   使用 `iptables -L -n` 查看端口是否开启

   ```
   `Chain INPUT (policy ACCEPT)
   target     prot opt source               destination         
   ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:443
   ACCEPT     udp  --  0.0.0.0/0            0.0.0.0/0            udp dpt:443

   Chain FORWARD (policy ACCEPT)
   target     prot opt source               destination         

   Chain OUTPUT (policy ACCEPT)
   target     prot opt source               destination         
   ```

   使用`ocserv -f -d 1`命令来启动下服务啦
   打开你手机上的 OpenConnect / Cisco Anyconnect 新建一个 VPN, 添加服务器 IP 就是你的 vps的IP:端口,例如`X.X.X.X:443` 
   好了，如果你看到如下信息，那服务器应该已经能够正常运行了：  

   ```
   Parsing plain auth method subconfig using legacy format
   Setting 'plain' as primary authentication method
   listening (TCP) on 0.0.0.0:443...
   listening (UDP) on 0.0.0.0:443...
   ocserv[16104]: main: initialized ocserv 0.11.8
   ocserv[16105]: sec-mod: reading supplemental config from files
   ocserv[16105]: sec-mod: sec-mod initialized (socket: /var/run/ocserv-socket.16104)
   ocserv[16109]: GnuTLS error (at worker-vpn.c:468): A TLS fatal alert has been received.: Unknown certificate
   ocserv[16104]: main: 60.0.14.48:9890 user disconnected
   ocserv[16105]: sec-mod: using 'plain' authentication to authenticate user (session: FXS0l)
   ocserv[16104]: main: 60.0.14.48:36627 user disconnected
   ocserv[16105]: sec-mod: initiating session for user 'test' (session: FXS0l)
   ocserv[16104]: main[test]: 60.0.14.48:9663 new user session
   ocserv[16104]: main[test]: 60.0.14.48:9663 user logged in
   ocserv[16104]: main: 60.0.14.48:46429 user disconnected
   ocserv[16104]: main[test]: 60.0.14.48:9663 user disconnected
   ocserv[16105]: sec-mod: temporarily closing session for test (session: FXS0l)
   ocserv[16105]: sec-mod: initiating session for user 'test' (session: FXS0l)ocserv[16104]: main[test]: 60.0.14.48:38135 new user session
   ocserv[16104]: main[test]: 60.0.14.48:38135 user logged in
   ```

   好了，目前已经搞定了OpenConnect server，下面讲的是一些优化，创建客户端证书，智能分流  

## 优化OpenConnectServer

1. 制作启动脚本

   首先，来写个启动脚本——毕竟，不能每次都用调试模式启动不是吗？

   ```
   vi /etc/init.d/ocserv
   ```

   在配置文件中写入如下脚本：

   ```shell
   #!/bin/sh
   ### BEGIN INIT INFO
   # Provides:          ocserv
   # Required-Start:    $remote_fs $syslog
   # Required-Stop:     $remote_fs $syslog
   # Default-Start:     2 3 4 5
   # Default-Stop:      0 1 6
   ### END INIT INFO
   # Copyright Rene Mayrhofer, Gibraltar, 1999
   # This script is distibuted under the GPL

   PATH=/bin:/usr/bin:/sbin:/usr/sbin:/usr/local/sbin
   DAEMON=/usr/local/sbin/ocserv
   PIDFILE=/var/run/ocserv.pid
   DAEMON_ARGS="-c /etc/ocserv/ocserv.conf"

   case "$1" in
   start)
   if [ ! -r $PIDFILE ]; then
   echo -n "Starting OpenConnect VPN Server Daemon: "
   start-stop-daemon --start --quiet --pidfile $PIDFILE --exec $DAEMON -- \
   $DAEMON_ARGS > /dev/null
   echo "ocserv."
   else
   echo -n "OpenConnect VPN Server is already running.\n\r"
   exit 0
   fi
   ;;
   stop)
   echo -n "Stopping OpenConnect VPN Server Daemon: "
   start-stop-daemon --stop --quiet --pidfile $PIDFILE --exec $DAEMON
   echo "ocserv."
   rm -f $PIDFILE
   ;;
   force-reload|restart)
   echo "Restarting OpenConnect VPN Server: "
   $0 stop
   sleep 1
   $0 start
   ;;
   status)
   if [ ! -r $PIDFILE ]; then
   # no pid file, process doesn't seem to be running correctly
   exit 3
   fi
   PID=`cat $PIDFILE | sed 's/ //g'`
   EXE=/proc/$PID/exe
   if [ -x "$EXE" ] &&
   [ "`ls -l \"$EXE\" | cut -d'>' -f2,2 | cut -d' ' -f2,2`" = \
   "$DAEMON" ]; then
   # ok, process seems to be running
   exit 0
   elif [ -r $PIDFILE ]; then
   # process not running, but pidfile exists
   exit 1
   else
   # no lock file to check for, so simply return the stopped status
   exit 3
   fi
   ;;
   *)
   echo "Usage: /etc/init.d/ocserv {start|stop|restart|force-reload|status}"
   exit 1
   ;;
   esac

   exit 0
   ```

   赋予其可执行权限

   ```
   chmod 755 /etc/init.d/ocserv`
   update-rc.d ocserv defaults
   ```

   之后就可以使用

   ```
   /etc/init.d/ocserv start | stop | restart
   service ocserv start | stop | restart
   ```

2. 免密码登录

   创建客户端证书，省的老输入密码

   ```
   cd ~/certificates/
   vi user.tmpl
   ```

   写入如下内容：

   ```
   cn = "some random name"
   unit = "some random unit"
   expiration_days = 365
   signing_key
   tls_www_client
   ```

   生成User密钥和证书

   ```
   certtool --generate-privkey --outfile user-key.pem
   certtool --generate-certificate --load-privkey user-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template user.tmpl --outfile user-cert.pem	
   ```

   然后要将证书和密钥转为PKCS12的格式

   ```
   certtool --to-p12 --load-privkey user-key.pem --pkcs-cipher 3des-pkcs12 --load-certificate user-cert.pem --outfile user.p12 --outder
   ```

   期间会要求你输入证书名字和密码。

   然后你需要把这个证书放到一个可以被直接访问的地方，然后通过 URL 将 user.p12 文件导入AnyConnect，具体位置在诊断标签页的证书栏目下，导入成功之后，将对应的VPN设置的高级设置部分的证书栏目，改为导入的这张证书。

   现在，为了让服务器能够认得这张证书，我们再来修改一下配置：

   ```
   vi /etc/ocserv/ocserv.conf
    
   # 改为证书登陆，注释掉原来的登陆模式
   auth = "certificate"
    
   # 证书认证不支持这个选项，注释掉这行
   #listen-clear-file = /var/run/ocserv-conn.socket
    
   # 启用证书验证
   ca-cert = /etc/ssl/private/my-ca-cert.pem
   ```

   这样，我们使用`service ocserv restart`来启动它即可！		

3. 智能分流

   编译 ocserv 前需要修改 src/vpn.h 来支持超过 96 行(ocserv默认值)但不超过200行(Cisco AnyConnect最大值)的路由表:

   ```
   vi ~/ocserv*/src/vpn.h
   #把96改为200以上
   #define DEFAULT_CONFIG_ENTRIES 96
   ```

   修改 ocserv 配置文件，添加[这些内容](https://github.com/CNMan/ocserv-cn-no-route/blob/master/cn-no-route.txt)

------

补充:   
一键脚本(新) https://moeclub.org/2017/06/22/268/    
一键脚本 https://github.com/fanyueciyuan/eazy-for-ss/tree/master/ocservauto    
注意Anyconnect和速锐一起使用的话,首先要禁止掉udp-port(详细看配置文件) ,接着重启速锐 最后重启Anyconnect    

参考:   
http://bitinn.net/11084/   
https://www.logcg.com/archives/1343.html   
http://www.fanyueciyuan.info/fq/ocserv-debian.html   
https://github.com/CNMan/ocserv-cn-no-route   