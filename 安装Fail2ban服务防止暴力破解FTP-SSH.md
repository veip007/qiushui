**安装**  
在ubuntu，Debian 或 Linux Mint上安装fail2ban：  
`apt-get install fail2ban`

CentOS 6 
```
32位 rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
64位 rpm -Uvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
yum install fail2ban
```
CentOS 7_64  
```
rpm -Uvh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-2.noarch.rpm
yum install fail2ban
```


**配置**	  
`vi /etc/fail2ban/jail.conf`
一般，我们设置这几个就可以，具体的含义如下：
```
ignoreip = 127.0.0.1 #忽略的IP列表,不受设置限制（白名单）
bantime = 600 #屏蔽时间，单位：秒
findtime = 600 #这个时间段内超过规定次数会被ban掉
maxretry = 3 #最大尝试次数
backend = auto #日志修改检测机制（gamin、polling和auto这三种）
[ssh-iptables] #针对各服务的检查配置，如设置bantime、findtime、maxretry和全局冲突，服务优先级大于全局设置
enabled = true #是否激活此项（true/false）
filter = sshd #过滤规则filter的名字，对应filter.d目录下的sshd.conf
action = iptables[name=SSH, port=ssh, protocol=tcp] #动作的相关参数
sendmail-whois[name=SSH, dest=root, sender=fail2ban@example.com] #触发报警的收件人
logpath = /var/log/secure #检测的系统的登陆日志文件
maxretry = 5 #最大尝试次数
```
**监控**  
/var/log/fail2ban.log，该文件记录在fail2ban中发生的任何敏感事件。  
`tail -f /var/log/fail2ban.log`

**启动**  
在 Debian, Ubuntu 或 CentOS/RHEL 6:  
`service fail2ban restart`  
在 Debian, Fedora 或 CentOS/RHEL 7:  
`systemctl restart fail2ban`  


**开机启动**  
在 Debian, Ubuntu 或 CentOS/RHEL 6:  
`chkconfig fail2ban on`  
在 Debian, Fedora 或 CentOS/RHEL 7:  
`systemctl enable fail2ban`  

参考  
https://linux.cn/article-5067-1.html  
http://www.centoscn.com/CentosSecurity/SoftSecurity/2015/0324/4996.html  