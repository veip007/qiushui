**本脚本适用环境：**  
系统支持：CentOS，Debian，Ubuntu  
内存要求：≥128M  
日期：2015年08月01日  

**关于本脚本：**  
一键安装 go 版的 shadowsocks 最新版本 1.1.4。据说 go 版本有 buff 。与 python 版不同的是，其客户端程序能使用多个服务端配置，本脚本安装的是服务端程序。作者默认推荐 aes-128-cfb 加密，基于一致性，脚本使用了 aes-256-cfb 加密方式。  



**默认配置：**  
服务器端口：自己设定（如不设定，默认为 8989）  
客户端端口：1080  
密码：自己设定（如不设定，默认为teddysun.com）  

**客户端下载：**  
http://sourceforge.net/projects/shadowsocksgui/files/dist/  

**使用方法：**  
使用root用户登录，运行以下命令：  
```
wget --no-check-certificate https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/shadowsocks-go.sh
chmod +x shadowsocks-go.sh
./shadowsocks-go.sh 2>&1 | tee shadowsocks-go.log
```  

**安装完成后，脚本提示如下：**  
```
Congratulations, shadowsocks-go install completed!
Your Server IP:your_server_ip
Your Server Port:your_server_port
Your Password:your_password
Your Local Port:1080
Your Encryption Method:aes-256-cfb

Welcome to visit:http://teddysun.com/392.html
Enjoy it!
```  

**卸载方法：**  
使用 root 用户登录，运行以下命令：  
`./shadowsocks-go.sh uninstall`

**其他事项：**  
客户端配置的参考链接：http://teddysun.com/339.html  
安装完成后即已后台启动 shadowsocks-go ，运行：  
`/etc/init.d/shadowsocks status`  
可以查看 shadowsocks-go 进程是否已经启动。  
本脚本安装完成后，已将 shadowsocks-go 加入开机自启动。  

**使用命令：**   
启动：`/etc/init.d/shadowsocks start`  
停止：`/etc/init.d/shadowsocks stop`  
重启：`/etc/init.d/shadowsocks restart`  
状态：`/etc/init.d/shadowsocks status`  

**多用户多端口配置文件 sample（2015年01月08日）：**  
配置文件路径：`vi /etc/shadowsocks/config.json`  
```
{
    "port_password":{
         "8989":"password0",
         "9001":"password1",
         "9002":"password2",
         "9003":"password3",
         "9004":"password4"
    },
    "method":"aes-256-cfb",
    "timeout":600
}
```   
参考链接：  
https://github.com/shadowsocks/shadowsocks-go  