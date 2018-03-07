# Project V 配置 WebSocket+TLS+Web+CDN



本文是记录搭建 Project V 过程，不是入门教程，建议先阅读[官方教程](https://www.v2ray.com)和[白话文教程](https://toutyrater.github.io/)



## 准备材料

- 一台 VPS（ip认证无所谓） 
- 一个域名，要求使用国外 DNS 解析
- 了解常用 Linux 指令
- 域名指向 VPS 的 ip



## 搭建网站

本文使用 LNMP 1.5 测试版来安装 Nginx 和 SSL 证书

1. 安装 Nginx，安装前建议使用 screen，执行：

   ```
   apt update && apt install screen wget curl -y
   screen -S lnmp
   wget -c http://soft.vpser.net/lnmp/lnmp1.5beta.tar.gz && tar zxf lnmp1.5beta.tar.gz && cd lnmp1.5 && ./install.sh nginx
   ```


1. 添加域名

   由于只安装了 Nginx，这里需要手动配置下 lnmp 命令

   ```
   cp conf/lnmp /bin/lnmp && chmod +x /bin/lnmp
   ```

   添加 vhost，填写域名，一路回车，最后开启 SSL （选2）

   ```
    lnmp vhost add
   ```

   等待证书生成，成功后用浏览器打开 `https://你的域名`  ，看看网站和证书是否正常


1. 开启 TLS1.3（可选）

   TLS 1.3 最引人之处是 0-RTT (零来回时间)。另，可与TLS 1.2同存，无兼容问题。

   第一步，在 /root/lnmp1.5 下找到 lnmp.conf，在其中的 Nginx_Modules_Options='  ' ，引号中添加

   ```
   --with-openssl=/root/openssl-tls1.3/ --with-openssl-opt=enable-tls1_3
   ```

   第二步，当前浏览器最高仅支持 tls1.3-draft-18，因此只能下载该版

   ```
   apt install git -y
   cd /root && git clone -b tls1.3-draft-18 --single-branch https://github.com/openssl/openssl.git openssl-tls1.3
   ```

   第三步，升级 Nginx

   ```
   cd /root/lnmp1.5
   ./upgrade.sh nginx
   # 输入当前最新版 1.13.8
   ```

   第四步，在网站的.conf配置文件中加入

   ```
   cd /usr/local/nginx/conf/vhost/
   # 修改网站的.conf

   ssl_protocols TLSv1.2 TLSv1.3;
   ssl_ciphers 'TLS13-CHACHA20-POLY1305-SHA256:TLS13-AES-128-GCM-SHA256:TLS13-AES-256-GCM-SHA384:TLS13-AES-128-CCM-SHA256:TLS13-AES-128-CCM-8-SHA256:EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES256:EECDH+3DES:RSA+3DES:!MD5'
   ```

   最后，TLS 1.3 还需浏览器开启支持，Firefox 中，在地址栏输入 about:config，找到security.tls.version.max，将值改为4，重启；Chrome 在地址栏输入 chrome://flags/，找到 Maximum TLS version enabled，将值改为TLS 1.3，重启；

   ​

## 搭建 Project V

要求客户端和服务端的系统 UTC 时间误差在两分钟之内，时区无关。

1. 服务端安装官方脚本

   ```
   bash <(curl -L -s https://install.direct/go.sh)
   ```

2. 修改配置

   ```
   vi /etc/v2ray/config.json

   # 把自带的配置全删掉，复制以下内容进去

   {
       "log": {
           "access": "/var/log/v2ray/access.log",
           "error": "/var/log/v2ray/error.log",
           "loglevel": "info"
       },
       "inbound": {
           "port": 10000,
           "listen": "127.0.0.1",
           "protocol": "vmess",
           "allocate": {
               "strategy": "always"
           },
           "settings": {
               "clients": [{
                   "id": "6d8a82b9-94d6-442e-a340-2b9cd5752c77",
                   "level": 1,
                   "alterId": 64,
                   "security": "chacha20-poly1305"
               }]
           },
           "streamSettings": {
               "network": "ws",
               "wsSettings": {
                   "connectionReuse": false,
                   "path": "/phpmyadmin/"
               }
           }
       },
       "outbound": {
           "protocol": "freedom",
           "settings": {}
       },
       "outboundDetour": [{
           "protocol": "blackhole",
           "settings": {},
           "tag": "blocked"
       }],
       "routing": {
           "strategy": "rules",
           "settings": {
               "rules": [{
                   "type": "field",
                   "ip": ["0.0.0.0/8", "10.0.0.0/8", "100.64.0.0/10", "127.0.0.0/8", "169.254.0.0/16", "172.16.0.0/12", "192.0.0.0/24", "192.0.2.0/24", "192.168.0.0/16", "198.18.0.0/15", "198.51.100.0/24", "203.0.113.0/24", "::1/128", "fc00::/7", "fe80::/10"],
                   "outboundTag": "blocked"
               }]
           }
       }
   }
   ```

3. 运行 service v2ray start 来启动 V2Ray 进程；
   之后可以使用 service v2ray start|stop|status|reload|restart|force-reload 控制 V2Ray 的运行

4. 客户端安装

   点[这里](https://github.com/v2ray/v2ray-core/releases)下载 V2Ray 压缩包，压缩包均为 zip 格式，找到对应平台的压缩包，下载解压，修改 config.json 配置为以下内容，**注意 address （第25行）填写网站域名**

   ```
   {
     "log": {
       "access": "",
       "error": "",
       "loglevel": ""
     },
     "inbound": {
       "port": 1085,
       "listen": "0.0.0.0",
       "protocol": "socks",
       "settings": {
         "auth": "noauth",
         "udp": true,
         "ip": "127.0.0.1",
         "clients": null
       },
       "streamSettings": null
     },
     "outbound": {
       "tag": "agentout",
       "protocol": "vmess",
       "settings": {
         "vnext": [
           {
             "address": "网站域名",
             "port": 443,
             "users": [
               {
                 "id": "6d8a82b9-94d6-442e-a340-2b9cd5752c77",
                 "alterId": 64,
                 "security": "chacha20-poly1305"
               }
             ]
           }
         ]
       },
       "streamSettings": {
         "network": "ws",
         "security": "tls",
         "tcpSettings": null,
         "kcpSettings": null,
         "wsSettings": {
           "connectionReuse": true,
           "path": "/phpmyadmin/",
           "headers": null
         }
       },
       "mux": {
         "enabled": true
       }
     },
     "inboundDetour": null,
     "outboundDetour": [
       {
         "protocol": "freedom",
         "settings": {
           "response": null
         },
         "tag": "direct"
       },
       {
         "protocol": "blackhole",
         "settings": {
           "response": {
             "type": "http"
           }
         },
         "tag": "blockout"
       }
     ],
     "dns": {
       "servers": [
         "8.8.8.8",
         "8.8.4.4",
         "localhost"
       ]
     },
     "routing": {
       "strategy": "rules",
       "settings": {
         "domainStrategy": "IPIfNonMatch",
         "rules": [
           {
             "type": "field",
             "port": null,
             "outboundTag": "direct",
             "ip": [
               "0.0.0.0/8",
               "10.0.0.0/8",
               "100.64.0.0/10",
               "127.0.0.0/8",
               "169.254.0.0/16",
               "172.16.0.0/12",
               "192.0.0.0/24",
               "192.0.2.0/24",
               "192.168.0.0/16",
               "198.18.0.0/15",
               "198.51.100.0/24",
               "203.0.113.0/24",
               "::1/128",
               "fc00::/7",
               "fe80::/10"
             ],
             "domain": null
           },
           {
             "type": "field",
             "port": null,
             "outboundTag": "direct",
             "ip": null,
             "domain": [
               "geosite:cn"
             ]
           },
           {
             "type": "field",
             "port": null,
             "outboundTag": "direct",
             "ip": [
               "geoip:cn"
             ],
             "domain": null
           }
         ]
       }
     }
   }
   ```

5. 配置 Nginx ，使用 Nginx 转发流量，接着重启 Nginx

   ```
   cd /usr/local/nginx/conf/vhost/
   # 修改网站的.conf
   # 在 server 内插入以下内容，同时开启 error_page 

   location /phpmyadmin/ {
             proxy_redirect off;
             #proxy_pass http://127.0.0.1:10000;
             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection "upgrade";
             proxy_set_header Host $http_host;
             proxy_intercept_errors on;
             if ($http_upgrade = "websocket" ){
                proxy_pass http://127.0.0.1:10000;
             }
           }
   ```

6. 客户端运行 v2ray 或 v2ray.exe，直接运行即可

   本地 socks 代理为 127.0.0.1:1085

   第三方客户端配置 address（网站域名）、port（443）、id（很长那串）、alterId（64）、security（chacha20-poly1305）、network（ws）、path（/v2ray/）、底层传输安全（tls）



## 开启 CDN

使用 CDN 最主要原因是隐藏 ip，或者你的 ip 已被认证，否则不太建议套上免费减速CDN（Cloudflare）。

需要注意 CDN 支持 WebSockets，同时网站开启了 TLS1.3 在 CDN 这里也要打开




***


参考：

https://www.17huiwei.com/enable-tls-1-3-in-lnmp/

https://toutyrater.github.io

https://www.v2ray.com

