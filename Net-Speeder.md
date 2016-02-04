Github 地址：[net-speeder](https://github.com/snooda/net-speeder)  

安装过程：  

**CentOS一键安装**  
```
wget --no-check-certificate https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/net_speeder_lazyinstall.sh
sh net_speeder_lazyinstall.sh
```

安装完毕后再敲入：  
```
nohup /usr/local/net_speeder/net_speeder venet0 "ip" >/dev/null 2>&1 &
```
注意这里引号中的IP不需要动，有的地方说需要改成自己的IP地址，其实不用改！  

关闭net_speeder：  
```
killall net_speeder
```

指定端口加速的方法:  
比如`net_speeder venet0 "src port 22"`就是针对22号端口流出的数据包加速。  
加速多个端口：`net_speeder venet0 "src port 22 || 80 || 443"﻿`  

**Debian/Ubuntu 一键安装**  
```
wget --no-check-certificate https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/debian_netspeeder_tennfy.sh
chmod a+x debian_netspeeder_tennfy.sh
bash debian_netspeeder_tennfy.sh
```

查看 net-speeder 是否运行  
```
ps aux|grep net_speeder|grep -v grep
```

停止net-speeder  
```
killall net_speeder
```
启动net-speeder（OPENVZ环境）  
```
nohup /root/net_speeder venet0 "ip" >/dev/null 2>&1 &
```

***

注：CentOS 下安装需要使用额外的 EPEL源 较麻烦，Github 上有教程，大家可以参看  
1. 安装运行及编译的依赖库   
``` 
apt-get install libnet1
apt-get install libpcap0.8
apt-get install libnet1-dev
apt-get install libpcap0.8-dev
```
2. 下载源码到 服务器  
```
cd /var
wget https://github.com/snooda/net-speeder/raw/master/net_speeder.c
wget https://github.com/snooda/net-speeder/raw/master/build.s
```
3. 编译  
```
chmod +x build.sh
./build.sh -DCOOKED
```
4. 运行并加入开机启动  
```
echo "nohup /usr/local/net_speeder/net_speeder venet0 "ip" >/dev/null 2>&1 &" >> /etc/rc.local
```

[参考](http://www.cmsky.com/vps-net-speeder/)