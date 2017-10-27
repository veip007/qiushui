### 1. 系统层面
基于kvm架构vps的优化  
这方面SS给出了非常详尽的优化指南，主要有：优化内核参数，开启TCP Fast Open  
####1.1优化内核参数
编辑```vi /etc/sysctl.conf```   
复制进去   
```
# max open files
fs.file-max = 1024000
# max read buffer
net.core.rmem_max = 67108864
# max write buffer
net.core.wmem_max = 67108864
# default read buffer
net.core.rmem_default = 65536
# default write buffer
net.core.wmem_default = 65536
# max processor input queue
net.core.netdev_max_backlog = 4096
# max backlog
net.core.somaxconn = 4096

# resist SYN flood attacks
net.ipv4.tcp_syncookies = 1
# reuse timewait sockets when safe
net.ipv4.tcp_tw_reuse = 1
# turn off fast timewait sockets recycling
net.ipv4.tcp_tw_recycle = 0
# short FIN timeout
net.ipv4.tcp_fin_timeout = 30
# short keepalive time
net.ipv4.tcp_keepalive_time = 1200
# outbound port range
net.ipv4.ip_local_port_range = 10000 65000
# max SYN backlog
net.ipv4.tcp_max_syn_backlog = 4096
# max timewait sockets held by system simultaneously
net.ipv4.tcp_max_tw_buckets = 5000
# TCP receive buffer
net.ipv4.tcp_rmem = 4096 87380 67108864
# TCP write buffer
net.ipv4.tcp_wmem = 4096 65536 67108864
# turn on path MTU discovery
net.ipv4.tcp_mtu_probing = 1

# for high-latency network
net.ipv4.tcp_congestion_control = hybla
# forward ipv4
net.ipv4.ip_forward = 1
```
保存生效  
`sysctl -p`  
其中最后的hybla是为高延迟网络（如美国，欧洲）准备的算法，需要内核支持，测试内核是否支持，在终端输入：  
`sysctl net.ipv4.tcp_available_congestion_control`  
如果结果中有hybla，则证明你的内核已开启hybla，如果没有hybla，可以用命令`modprobe tcp_hybla`开启。
  
对于低延迟的网络（如日本，香港等），可以使用`htcp`，可以非常显著的提高速度，首先使用`modprobe tcp_htcp`开启，再将`net.ipv4.tcp_congestion_control = hybla`改为`net.ipv4.tcp_congestion_control = htcp`，建议EC2日本用户使用这个算法。


***

#### 1.2 TCP优化  
1.修改文件句柄数限制  
如果是ubuntu/centos均可修改`/etc/sysctl.conf`  
找到`fs.file-max`这一行，修改其值为`1024000`，并保存退出。然后执行`sysctl -p`使其生效  
修改`vi /etc/security/limits.conf`文件，加入  
```
*               soft    nofile           512000
*               hard    nofile          1024000
```
针对centos,还需要修改`vi /etc/pam.d/common-session`文件，加入  
`session required pam_limits.so`  

2.修改`vi /etc/profile`文件，加入  
`ulimit -SHn 1024000`  
然后重启服务器执行`ulimit -n`，查询返回1024000即可。  
  
```
sysctl.conf报错解决方法
修复modprobe的：
rm -f /sbin/modprobe 
ln -s /bin/true /sbin/modprobe
修复sysctl的：
rm -f /sbin/sysctl 
ln -s /bin/true /sbin/sysctl
```
***

#### 1.3 锐速
锐速是TCP底层加速软件,官方已停止推出永久免费版本,但网上有破解版可以继续使用。需要购买的话先到锐速官网注册帐号,并确认[内核版本](http://dl.serverspeeder.com/ls.do?m=availables)是否支持锐速的版本。  

一键安装速锐破解版  
```
wget -N --no-check-certificate https://github.com/91yun/serverspeeder/raw/master/serverspeeder.sh && bash serverspeeder.sh
```

一键卸载
```
chattr -i /serverspeeder/etc/apx* && /serverspeeder/bin/serverSpeeder.sh uninstall -f
```

设置  
```
Enter your accelerated interface(s) [eth0]: eth0
Enter your outbound bandwidth [1000000 kbps]: 1000000
Enter your inbound bandwidth [1000000 kbps]: 1000000
Configure shortRtt-bypass [0 ms]: 0
Auto load ServerSpeeder on linux start-up? [n]:y #是否开机自启
Run ServerSpeeder now? [y]:y #是否现在启动
```
执行`lsmod`，看到有`appex0`模块即说明锐速已正常安装并启动。   

至此，安装就结束了，但还有后续配置。  
修改`vi /serverspeeder/etc/config`文件的几个参数以使锐速更好的工作  
```
accppp="1" #加速PPTP、L2TP V-P-N；设为1表示开启，设为0表示关闭
advinacc="1" #高级入向加速开关；设为 1 表示开启，设为 0 表示关闭；开启此功能可以得到更好的流入方向流量加速效果；
maxmode="1" #最大传输模式；设为 1 表示开启；设为 0 表示关闭；开启后会进一步提高加速效果，但是可能会降低有效数据率。
rsc="1" #网卡接收端合并开关；设为 1 表示开启，设为 0 表示关闭；在有些较新的网卡驱动中，带有 RSC 算法的，需要打开该功能。
l2wQLimit="512 4096" #从 LAN 到 WAN 加速引擎在缓冲池充满和空闲时分别能够缓存的数据包队列的长度的上限；该值设置的高会获得更好的加速效果，但是会消耗更多的内存
w2lQLimit="512 4096" #从 WAN 到 LAN 加速引擎在缓冲池充满和空闲时分别能够缓存的数据包队列的长度的上限；该值设置的高会获得更好的加速效果，但是会消耗更多的内存
```

重读配置以使配置生效`/serverspeeder/bin/serverSpeeder.sh reload`   

查看锐速当前状态`/serverspeeder/bin/serverSpeeder.sh stats`  

查看所有命令`/serverspeeder/bin/serverSpeeder.sh help`  

停止`/serverspeeder/bin/serverSpeeder.sh stop`  

启动`/serverspeeder/bin/serverSpeeder.sh start`  

重启锐速`/serverspeeder/bin/serverSpeeder.sh restart`  

***


#### 1.4 开启TCP Fast Open
这个需要服务器和客户端都是Linux 3.7+的内核，一般Linux的服务器发行版只有debian jessie有3.7+的，客户端用Linux更是珍稀动物，所以这个不多说，如果你的服务器端和客户端都是Linux 3.7+的内核，那就在服务端和客户端的`vi /etc/sysctl.conf`文件中再加上一行。    
```
# turn on TCP Fast Open on both client and server side
net.ipv4.tcp_fastopen = 3
```
然后把`vi /etc/shadowsocks.json`配置文件中"fast_open": false改为"fast_open": true。这样速度也将会有非常显著的提升。


***


### 2. 加密层面
####2.1 安装M2Crypto
这个可以提高SS的加密速度，安装办法：  
Debian/Ubuntu  
`apt-get install python-m2crypto`  
安装之后重启SS，速度将会有一定的提升  


CentOS  
先安装依赖包：  
`yum install -y openssl-devel gcc swig python-devel autoconf libtool`  
安装setuptools：  
```
wget --no-check-certificate https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/ez_setup.py
python ez_setup.py install
```
再通过pip安装M2Crypto：  
`pip install M2Crypto`
或者`pip install M2Crypto --upgrade`  
***

#### 2.3 使用CHACHA20加密算法
首先，安装libsodium，让系统支持chacha20算法。  
Debian/Ubuntu   
```
apt-get install build-essential
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar xf LATEST.tar.gz && cd libsodium*
./configure && make && make install
ldconfig
```
CentOS  
```
yum groupinstall "Development Tools"
wget https://download.libsodium.org/libsodium/releases/LATEST.tar.gz
tar zxf LATEST.tar.gz
cd libsodium* 
./configure
make
make install
vi /etc/ld.so.conf
添加一行：
/usr/local/lib
保存退出后，运行命令：
ldconfig
```
然后修改ss加密方式：  
`vi /etc/shadowsocks.json`
"method":"aes-256-cfb"改成"method":"chacha20",重启SS即可`/etc/init.d/shadowsocks restart`    



***

### 3. 网络层面
此外，选择合适的端口也能优化梯子的速度，广大SS用户的实践经验表明，检查站（GFW）存在一种机制来降低自身的运算压力，即常用的协议端口（如http，smtp，ssh，https，ftp等）的检查较少，所以建议SS绑定这些常用的端口（如：21，22，25，80，443），速度也会有显著提升。  
如果你还要给小伙伴爬，那我建议开启多个端口而不是共用，这样网络会更加顺畅。  

#### 3.1 防火墙设置（如有）
自动调整MTU  
`iptables -I FORWARD -p tcp --tcp-flags SYN,RST SYN -j TCPMSS --clamp-mss-to-pmtu`  

开启 NAT （记得把 eth0 改成自己的网卡名，openvz 的基本是 venet0 ）  
`iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE`  

开启 IPv4 的转发  
`sysctl -w net.ipv4.ip_forward=1`  

打开 443 端口  
```
iptables -I INPUT -p tcp --dport 443 -j ACCEPT
iptables -I INPUT -p udp --dport 443 -j ACCEPT
```

重启防火墙iptables：  
`service iptables restart`  


***
参考  
https://www.zxc.so/shadowsocks-ladder.html  
https://teddysun.com/339.html  