BBR 目的是要尽量跑满带宽, 并且尽量不要有排队的情况,  效果并不比速锐差

最新 Ubuntu4.9-rc8 内核已 tcp_bbr 下面简单讲述如何开启  
本人已在 Debian8/Ubuntu14 下测试成功  

- 下载最新内核,最新内核查看[这里](http://kernel.ubuntu.com/~kernel-ppa/mainline)  
```
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.9-rc8/linux-image-4.9.0-040900rc8-lowlatency_4.9.0-040900rc8.201612041731_amd64.deb
```

- 安装内核
```
dpkg -i linux-image-4.9.0*.deb
```

-  删除其余内核
```
dpkg -l|grep linux-image 
apt-get pruge 旧内核
```

- 更新 grub 系统引导文件并重启
```
update-grub
reboot
```

- 开启bbr
```
echo "/sbin/modprobe tcp_bbr" » /etc/modules
vi /etc/sysctl.conf
# 添加下面一行
net.ipv4.tcp_congestion_control=bbr
```
保存生效`sysctl -p`  
执行`sysctl net.ipv4.tcp_available_congestion_control`  
如果结果中有`bbr`, 则证明你的内核已开启bbr  
执行`lsmod | grep bbr`, 看到有 tcp_bbr 模块即说明bbr已启动  