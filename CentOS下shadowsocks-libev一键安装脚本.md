**本脚本适用环境：**   
系统支持：CentOS 32或64位   
内存要求：≥128M   
日期：2015年08月01日   

**关于本脚本：**  
一键安装 libev 版的 shadowsocks 最新版本。该版本的特点是内存占用小（600k左右），低 CPU 消耗，甚至可以安装在基于 OpenWRT 的路由器上。  
友情提示：如果你有问题，请先参考这篇《Shadowsocks Troubleshooting》后再问。  


**默认配置：**  
服务器端口：自己设定（如不设定，默认为 8989）  
客户端端口：1080  
密码：自己设定（如不设定，默认为teddysun.com）  

**客户端下载：**  
http://sourceforge.net/projects/shadowsocksgui/files/dist/  

**使用方法：**  
使用root用户登录，运行以下命令：  
```
wget --no-check-certificate 
https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/shadowsocks-libev.sh
chmod +x shadowsocks-libev.sh
./shadowsocks-libev.sh 2>&1 | tee shadowsocks-libev.log
```   
**安装完成后，脚本提示如下：**   
```
Congratulations, shadowsocks-libev install completed!
Your Server IP:your_server_ip
Your Server Port:your_server_port
Your Password:your_password
Your Local IP:127.0.0.1
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:http://teddysun.com/357.html
Enjoy it!
```   
**卸载方法：**   
使用 root 用户登录，运行以下命令：
`./shadowsocks-libev.sh uninstall`   

**其他事项：**
客户端配置的参考链接：http://teddysun.com/339.html  
安装完成后即已后台启动 shadowsocks ，运行：  
`ps -ef | grep ss-server | grep -v ps | grep -v grep`  
可以查看进程是否存在。  
本脚本安装完成后，会将 shadowsocks-libev 加入开机自启动。  

**使用命令：**  
启动：`/etc/init.d/shadowsocks start`  
停止：`/etc/init.d/shadowsocks stop`  
重启：`/etc/init.d/shadowsocks restart`  
查看状态：`/etc/init.d/shadowsocks status`  

参考链接：  
https://github.com/madeye/shadowsocks-libev  