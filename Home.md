**本脚本适用环境：**  
系统支持：CentOS，Debian，Ubuntu  
内存要求：≥128M  
日期：2017 年 07 月 28 日

**关于本脚本：**  
一键安装 ShadowsocksR 服务端。  
[ShadowsocksR 协议插件文档](https://github.com/iMeiji/shadowsocks_install/blob/master/shadowsocksR-wiki/ShadowsocksR%20%E5%8D%8F%E8%AE%AE%E6%8F%92%E4%BB%B6%E6%96%87%E6%A1%A3.md)  

**默认配置：**  
服务器端口：自己设定（如不设定，默认为 8989）  
密码：自己设定（如不设定，默认为 teddysun.com）  
加密方式：自己设定（如不设定，默认为 aes-256-cfb）  
协议（Protocol）：自己设定（如不设定，默认为 origin）  
混淆（obfs）：自己设定（如不设定，默认为 plain）  

**客户端下载：**  
https://github.com/iMeiji/shadowsocks_install/releases/tag/0.13

**使用方法：**  
使用root用户登录，运行以下命令：
```
wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

**安装完成后，脚本提示如下：**
```
Congratulations, ShadowsocksR server install completed!
Your Server IP        :your_server_ip
Your Server Port      :your_server_port
Your Password         :your_password
Your Protocol         :your_protocol
Your obfs             :your_obfs
Your Encryption Method:your_encryption_method

Welcome to visit:https://shadowsocks.be/9.html
Enjoy it!
```

**卸载方法：**  
使用 root 用户登录，运行命令：`./shadowsocksR.sh uninstall`  

安装完成后即已后台启动 ShadowsocksR ，运行：`/etc/init.d/shadowsocks status`    
可以查看 ShadowsocksR 进程是否已经启动。  
本脚本安装完成后，已将 ShadowsocksR 自动加入开机自启动。  

**使用命令：**  
启动：`/etc/init.d/shadowsocks start`  
停止：`/etc/init.d/shadowsocks stop`  
重启：`/etc/init.d/shadowsocks restart`  
状态：`/etc/init.d/shadowsocks status`  

配置文件路径：`/etc/shadowsocks.json`  
日志文件路径：`/var/log/shadowsocks.log`  
代码安装目录：`/usr/local/shadowsocks`  

**如果你想修改配置文件，请参考：**  
https://github.com/iMeiji/shadowsocks_install/blob/master/shadowsocksR-wiki/config.json.md  

**注意事项：**  
本脚本没有对防火墙（IPv4 是 iptables，IPv6 是 ip6tables）进行任何设置。  
因此，在安装完毕，如果你发现连接不上，可以尝试更改防火墙设置或关闭防火墙。  

**参考链接：**  
https://github.com/breakwa11/shadowsocks-rss  
https://shadowsocks.be/9.html   