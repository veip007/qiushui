# shadowsocks libev SSR 服务端安装教程
>版本的特点是内存占用小（600k左右）,低 CPU 消耗,支持SSR海螺特征,甚至可以安装在基于 OpenWRT 的路由器上  
项目地址https://github.com/breakwa11/shadowsocks-libev

基本库安装 
-----
以下命令均以root用户执行，或sudo方式执行

**centos:**  
```
yum install m2crypto git libsodium
yum install gcc automake autoconf libtool make build-essential autoconf libtool gcc
yum install curl curl-devel zlib-devel openssl-devel perl perl-devel cpio expat-devel gettext-devel
yum install git
```

**ubuntu/debian:**  
 
    apt-get install m2crypto git build-essential autoconf libtool libssl-dev gawk debhelper dh-systemd init-system-helpers pkg-config

如果要使用 salsa20 或 chacha20 或 chacha20-ietf 算法，请安装 [libsodium](https://github.com/jedisct1/libsodium) :

```
apt-get install build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.8/libsodium-1.0.8.tar.gz
tar xf libsodium-1.0.8.tar.gz && cd libsodium-1.0.8
./configure && make -j2 && make install
ldconfig
```
如果曾经安装过旧版本，亦可重复用以上步骤更新到最新版，仅1.0.4或以上版本支持chacha20-ietf


编译并安装
-----
```
mkdir shadowsocks-libev && cd shadowsocks-libev
git clone https://github.com/breakwa11/shadowsocks-libev.git
```

然后生成deb包并安装，一步步执行（留意是否出错 如果出错需要检查系统或者之前的步骤）：  
```
cd shadowsocks-libev
dpkg-buildpackage -us -uc
cd ..
dpkg -i shadowsocks-libev*.deb
```

服务器配置
---
建立配置文件 `/etc/shadowsocks-libev/config.json`  
```
{
    "server": "0.0.0.0",
    "server_ipv6": "::",
    "server_port": 8388,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "mypassword",
    "timeout": 120,
    "method": "aes-256-cfb",
    "protocol": "auth_sha1_compatible",
    "protocol_param": "",
    "obfs": "tls1.0_session_auth_compatible",
    "obfs_param": "",
    "redirect": "",
    "dns_ipv6": false,
    "fast_open": false,
    "workers": 1
}
```
各选项说明：

Name    |    Explanation  | 中文说明
------- | --------------- | ---------------
server |	the address your server listens | 监听地址
server_ipv6 |   the ipv6 address your server listens  | ipv6地址
server_port |	server port                     | 监听端口
local_address|	the address your local listens  | 本地地址
local_port |	local port                      | 本地端口
password |	password used for encryption    | 密码
timeout |	in seconds                      | 超时时间
method |	default: "aes-256-cfb", see Encryption | 加密方式
protocol |      default："origin"     | 协议插件，默认"origin"
protocol_param |      default：""     | 协议插件参数，默认""
obfs   |      default："tls1.0_session_auth_compatible"     | 混淆插件，默认"tls1.0_session_auth_compatible"
obfs_param |      default：""     | 混淆插件参数，默认""
redirect |      default：""     | 重定向参数，默认""
dns_ipv6|     default:false  | 是否优先使用IPv6地址，有IPv6时可开启
fast_open |	use TCP_FASTOPEN, true / false         | 快速打开(仅限linux客户端)
workers	| number of workers, available on Unix/Linux   |线程（仅限linux客户端）

其中protocol有如下取值：

protocol| 说明
------- | ----------
"origin"|原版协议
"verify_simple"|带校验的协议
"verify_deflate"|带压缩的协议
"verify_sha1"|带验证抗CCA攻击的协议，可兼容libev的OTA
"auth_simple"|抗重放攻击的协议
"auth_sha1"|带验证抗CCA攻击且抗重放攻击的协议

其中obfs有如下取值：

obfs   | 说明
-------|----------
"plain"|不混淆
"http_simple"|伪装为http协议
"tls_simple"|伪装为tls协议（不建议使用）
"random_head"|发送一个随机包再通讯的协议
"tls1.0_session_auth"|伪装为tls session握手协议，同时能抗重放攻击

各混淆插件的说明请点击这里查看：[混淆插件说明]

注：客户端的protocol和obfs配置必须与服务端的一致。

redirect参数说明：

值为空字符串或一个列表，若为列表示例如  
"redirect":["bing.com", "cloudflare.com:443"],  
作用是在连接方的数据不正确的时候，把数据重定向到列表中的其中一个地址和端口（不写端口则视为80），以伪装为目标服务器。

dns_ipv6参数说明：

为true则指定服务器优先使用IPv6地址。仅当服务器能访问IPv6地址时可以用，否则会导致有IPv6地址的网站无法打开。

一般情况下，只需要修改以下五项即可：
```
"server_port":8388,        //端口
"password":"password",     //密码
"protocol":"origin",       //协议插件
"obfs":"http_simple",      //混淆插件
"method":"aes-256-cfb",    //加密方式
```

如果客户端有OpenWRT路由器等设备，推荐rc4-md5，性能更好；否则可以选用安全性更好的aes-256-cfb等，不过计算复杂度上升，会有性能的损失，不过对于PC机以及现在的只能手机来说没有任何问题。路由器性能较弱所以可以考虑rc4-md5。  


启动
---
Ubuntu/Debian 通过deb包安装的（deb包安装的默认会开启自启）：  
`service shadowsocks-libev restart`  
CentOS，拷贝启动脚本shadowsocks-libev到/etc/init.d/目录下后，启动：  
`/etc/init.d/shadowsocks-libev start`  

或者直接调用ss-server命令运行,具体用法如下：
```
usage:

  ss-server

     -s <server_host>           Host name or ip address of your remote server.
     -p <server_port>           Port number of your remote server.
     -l <local_port>            Port number of your local server.
     -k <password>              Password of your remote server.
     -m <encrypt_method>        Encrypt method: table, rc4, rc4-md5,
                                aes-128-cfb, aes-192-cfb, aes-256-cfb,
                                bf-cfb, camellia-128-cfb, camellia-192-cfb,
                                camellia-256-cfb, cast5-cfb, des-cfb, idea-cfb
                                rc2-cfb, seed-cfb, salsa20 and chacha20.

     [-a <user>]                Run as another user.
     [-f <pid_file>]            The file path to store pid.
     [-t <timeout>]             Socket timeout in seconds.
     [-c <config_file>]         The path to config file.
     [-n <number>]              Max number of open files.
     [-i <interface>]           Network interface to bind.

     [-u]                       Enable UDP relay,
     [-U]                       Enable UDP relay and disable TCP relay.
     [-A]                       Enable onetime authentication.
     [-w]                       Enable white list mode (when ACL enabled).

     [-d <addr>]                Name servers for internal DNS resolver.
     [--fast-open]              Enable TCP fast open.
                                with Linux kernel > 3.7.0.
     [--acl <acl_file>]         Path to ACL (Access Control List).
     [--manager-address <addr>] UNIX domain socket address.

     [-v]                       Verbose mode

```
一个例子如：  
`setsid ss-server -c /etc/shadowsocks-libev/config.json -u`  

查看shadowsocks是否正确启动并监听相应端口，看到有ss-server进程LISTEN正确的端口就表示成功：  
`netstat -lnp`  

参考
---
https://github.com/breakwa11/shadowsocks-rss/wiki/Server-Setup  