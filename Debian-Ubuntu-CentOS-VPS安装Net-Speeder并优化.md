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