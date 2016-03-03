### 一：修改SSH登录端口
1、用下面命令进入配置文件`vi /etc/ssh/sshd_config`  
2、找到#port 22，将前面的#去掉，然后修改端口 port 12345（自己设定）。  
3、然后重启ssh服务
```
#Debian/ubuntu  /etc/init.d/ssh restart   or    service ssh restart
#CentOS         service sshd restart
``` 

### 二：使用密钥登录SSH  
执行以下命令在VPS上生成密钥文件。  
`ssh-keygen -t rsa`  
第一个提示是路径 默认即可，第二个提示输入passphrase，这个就是给这个密钥文件设置一个密码，在这里我强烈建议大家一定要设置，如果没有设置 密钥泄漏了什么的，就呵呵呵了。  
输入完成以后 你会看见一个非常美丽又莫名其妙的图形，这就说明密钥公钥已经产生。  
  
这个时候执行命令  
```
cd ~/.ssh
ls
```
你会看见有几个文件,其中有2个文件特别重要分别是 id_rsa 和 id_rsa.pub  前者为密钥，也就是我们登陆所需要的东西。先通过SFTP或者其他方法下载下来进行保存**（非常重要，如果忘记了，服务器没法登陆了)**  
  
接下来执行这个命令,将公钥文件复制一份 并改名为 authorized_keys  
`cat id_rsa.pub >> ~/.ssh/authorized_keys`  
  
然后编辑ssh配置文件，通常来说配置文件是：vi /etc/ssh/sshd_config
修改下面这几个地方  
```
#设置公钥
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile      %h/.ssh/authorized_keys

#关闭密码认证 因为我们打算用公钥密钥来连接了
PasswordAuthentication no
```
  
配置好服务器以后 再次确认 密钥文件是否下载了，否则悲剧了别找我 重启SSH服务器 生效  
```
#Debian/ubuntu  /etc/init.d/ssh restart  or  service ssh restart
#CentOS         service sshd restart
``` 
  
至于客户端如何连接，具体方法就是 将密钥导入到密钥管理器一类的东西里面，SSH软件过多，我这里就简单的说下 我用的 Bitvise SSH Client  
![](http://xloli.net/wp-content/uploads/rsa.jpg)

### 四：Denyhosts防暴力攻击
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
重启服务`service denyhosts restart`  
写入自启`echo "service denyhosts restart" >> /etc/rc.local`  

参考  
http://www.lovelucy.info/vps-anti-ssh-login-attempts-attack.html  
http://www.freehao123.com/vps-ssh/  
http://xloli.net/html/201307/thread-3110-1-1.html  