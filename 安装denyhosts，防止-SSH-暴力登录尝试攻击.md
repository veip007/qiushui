这个方法比较省时省力。denyhosts 是 Python 语言写的一个程序，它会分析 sshd 的日志文件，当发现重复的失败登录时就会记录 IP 到 /etc/hosts.deny 文件，从而达到自动屏 IP 的功能  

**安装**  
```
# Debian/Ubuntu：
$ sudo apt-get install denyhosts
 
# RedHat/CentOS
$ yum install denyhosts
 
# Archlinux
$ yaourt denyhosts
 
# Gentoo
$ emerge -av denyhosts
```

默认配置就能很好的工作，如要个性化设置可以修改 `/etc/denyhosts.conf`  
```
SECURE_LOG = /var/log/auth.log #ssh 日志文件，它是根据这个文件来判断的。
HOSTS_DENY = /etc/hosts.deny #控制用户登陆的文件
PURGE_DENY = #过多久后清除已经禁止的，空表示永远不解禁
BLOCK_SERVICE = sshd #禁止的服务名，如还要添加其他服务，只需添加逗号跟上相应的服务即可
DENY_THRESHOLD_INVALID = 5 #允许无效用户失败的次数
DENY_THRESHOLD_VALID = 10 #允许普通用户登陆失败的次数
DENY_THRESHOLD_ROOT = 1 #允许root登陆失败的次数
DENY_THRESHOLD_RESTRICTED = 1
WORK_DIR = /var/lib/denyhosts #运行目录
SUSPICIOUS_LOGIN_REPORT_ALLOWED_HOSTS=YES
HOSTNAME_LOOKUP=YES #是否进行域名反解析
LOCK_FILE = /var/run/denyhosts.pid #程序的进程ID
ADMIN_EMAIL = root@localhost #管理员邮件地址,它会给管理员发邮件
SMTP_HOST = localhost
SMTP_PORT = 25
SMTP_FROM = DenyHosts <nobody@localhost>
SMTP_SUBJECT = DenyHosts Report
AGE_RESET_VALID=5d #用户的登录失败计数会在多久以后重置为0，(h表示小时，d表示天，m表示月，w表示周，y表示年)
AGE_RESET_ROOT=25d
AGE_RESET_RESTRICTED=25d
AGE_RESET_INVALID=10d
RESET_ON_SUCCESS = yes #如果一个ip登陆成功后，失败的登陆计数是否重置为0
DAEMON_LOG = /var/log/denyhosts #自己的日志文件
DAEMON_SLEEP = 30s #当以后台方式运行时，每读一次日志文件的时间间隔。
DAEMON_PURGE = 1h #当以后台方式运行时，清除机制在 HOSTS_DENY 中终止旧条目的时间间隔,这个会影响PURGE_DENY的间隔。
```

查看已拦截的IP`cat /etc/hosts.deny`  
查看拦截记录`cat /etc/hosts.deny | wc -l`  


[参考](http://www.lovelucy.info/vps-anti-ssh-login-attempts-attack.html)