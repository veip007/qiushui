Github 地址：[net-speeder](https://github.com/snooda/net-speeder)  

安装过程：  

**CentOS安装**  
```
https://raw.githubusercontent.com/iMeiji/shadowsocks_install/master/net_speeder_lazyinstall.sh
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