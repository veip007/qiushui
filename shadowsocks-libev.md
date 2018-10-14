shadowsocks libev版本的特点是内存占用小（600k左右），低 CPU 消耗，甚至可以安装在基于 OpenWRT 的路由器上。  

# 1.安装shadowsocks libev服务端

## 1.1从源码编译安装(推荐)

### Ubuntu/Debian：
先安装必须的包：  
```
mkdir shadowsocks-libev && cd shadowsocks-libev
apt-get install build-essential autoconf libtool libssl-dev \
  gawk debhelper dh-systemd init-system-helpers pkg-config git
```  
然后通过git下载源码：    
`git clone https://github.com/shadowsocks/shadowsocks-libev.git`  
然后生成deb包并安装，一步步执行（留意是否出错 如果出错需要检查系统或者之前的步骤）：  
```
cd shadowsocks-libev
dpkg-buildpackage -us -uc
cd ..
dpkg -i shadowsocks-libev*.deb
```

### CentOS：  
```
yum install -y gcc automake autoconf libtool make build-essential autoconf libtool gcc
yum install -y curl curl-devel zlib-devel openssl-devel perl perl-devel cpio expat-devel gettext-devel
yum install -y git
```
然后通过git下载源码：  
```
git clone https://github.com/madeye/shadowsocks-libev.git
```
从源码编译安装：  
```
cd shadowsocks-libev
# Start building
./autogen.sh && ./configure && make
sudo make install
```

### Other:
其他Unix-like的系统，特别是Debian-based的Linux发行版如： Ubuntu, Debian or Linux Mint  
```
apt-get install build-essential autoconf libtool libssl-dev
git clone https://github.com/madeye/shadowsocks-libev.git
cd shadowsocks-libev
./configure && make
make install
```

## 1.2 直接从作者提供的软件源安装（Ubuntu/Debian）
由于作者更新源码后并不一定更新这些预编译的包，所以无法保证最新版本，但步骤比较简单。不推荐这种方式。  
先添加GPG Key：  
```
wget -O- http://shadowsocks.org/debian/1D27208A.gpg | sudo apt-key add -
```
在`/etc/apt/sources.list`末尾添加：
```
# Ubuntu 14.04 or above
deb http://shadowsocks.org/ubuntu trusty main

# Debian Wheezy, Ubuntu 12.04 or any distribution with libssl > 1.0.1
deb http://shadowsocks.org/debian wheezy main
```
最后安装  
```
apt-get update
apt-get install shadowsocks-libev
```

## 1.3 从官方仓库中安装（ArchLinux）
```
sudo pacman -S shadowsocks-libev
```

# 2.配置与启动

## 2.1配置文件
编辑 `/etc/shadowsocks-libev/config.json`（ArchLinux用户建议放在`/etc/shadowsocks`目录下），格式说明：  
```
{
	"server":"example.com or X.X.X.X",
	"server_port":443,
	"password":"password",
	"timeout":300,
	"method":"rc4-md5"
}
```
其中：  
> server：主机域名或者IP地址，尽量填IP  
server_port：服务器监听端口  
password：密码  
timeout：连接超时时间，单位秒。要适中  
method：加密方式 默认为table,其他有rc4,rc4-md5,aes-128-cfb, aes-192-cfb, aes-256-cfb,bf-cfb, camellia-128-cfb, camellia-192-cfb,camellia-256-cfb, cast5-cfb, des-cfb  

如果客户端有OpenWRT路由器等设备，推荐rc4-md5，性能更好；否则可以选用安全性更好的aes-256-cfb等，不过计算复杂度上升，会有性能的损失，不过对于PC机以及现在的只能手机来说没有任何问题。路由器性能较弱所以可以考虑rc4-md5。  

## 2.2 启动

Ubuntu/Debian 通过deb包安装的（deb包安装的默认会开启自启）：  
`service shadowsocks-libev restart`  
 
CentOS，拷贝启动脚本shadowsocks-libev到/etc/init.d/目录下后，启动：  
`/etc/init.d/shadowsocks-libev start`  
 
ArchLinux，假设配置文件为，`/etc/shadowsocks/foo.json` 启动：  
`sudo systemctl start shadowsocks-libev-server@foo`  
设置开机自启动：  
`sudo systemctl enable shadowsocks-libev-server@foo` 

或者直接调用ss-server命令运行，具体用法如下：
```
ss-[local|redir|server|tunnel]

       -s <server_host>           host name or ip address of your remote server

       -p <server_port>           port number of your remote server

       -l <local_port>            port number of your local server

       -k <password>              password of your remote server

       [-m <encrypt_method>]      encrypt method: table, rc4, rc4-md5,
                                  aes-128-cfb, aes-192-cfb, aes-256-cfb,
                                  bf-cfb, camellia-128-cfb, camellia-192-cfb,
                                  camellia-256-cfb, cast5-cfb, des-cfb, idea-cfb,
                                  rc2-cfb, seed-cfb, salsa20 ,chacha20 and
                                  chacha20-ietf

       [-f <pid_file>]            the file path to store pid

       [-t <timeout>]             socket timeout in seconds

       [-c <config_file>]         the path to config file

       [-i <interface>]           network interface to bind,
                                  not available in redir mode

       [-b <local_address>]       local address to bind,
                                  not available in server mode

       [-u]                       enable udprelay mode,
                                  TPROXY is required in redir mode

       [-U]                       enable UDP relay and disable TCP relay,
                                  not available in local mode

       [-A]                       enable onetime authentication

       [-L <addr>:<port>]         specify destination server address and port
                                  for local port forwarding,
                                  only available in tunnel mode

       [-d <addr>]                setup name servers for internal DNS resolver,
                                  only available in server mode

       [--fast-open]              enable TCP fast open,
                                  only available in local and server mode,
                                  with Linux kernel > 3.7.0

       [--acl <acl_file>]         config file of ACL (Access Control List)
                                  only available in local and server mode

       [--manager-address <addr>] UNIX domain socket address
                                  only available in server and manager mode

       [--executable <path>]      path to the executable of ss-server
                                  only available in manager mode

       [-v]                       verbose mode

notes:

    ss-redir provides a transparent proxy function and only works on the
    Linux platform with iptables.
```
一个例子如：  
`setsid ss-server -c /etc/shadowsocks/config.json -u`  

查看shadowsocks是否正确启动并监听相应端口，看到有ss-server进程LISTEN正确的端口就表示成功：  
`netstat -lnp`  


***
参考  
https://github.com/shadowsocks/shadowsocks-libev  
https://cokebar.info/archives/767  
https://wiki.archlinux.org/index.php/Shadowsocks_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)