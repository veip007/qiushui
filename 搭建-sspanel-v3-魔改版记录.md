# 搭建 sspanel v3 魔改版记录



前端使用赵大的魔改版(master分支版本) https://github.com/esdeathlove/ss-panel-v3-mod

后端也使用赵大的魔改版 https://github.com/esdeathlove/shadowsocks

演示环境 : VirMach 512RAM机子 Ubuntu 14 x64

附:  
[搭建 sspanel v3 魔改版new_master分支记录](https://github.com/iMeiji/ss-panel-v3-mod/wiki/%E5%AE%89%E8%A3%85%E8%AF%B4%E6%98%8E-lnmp1.4)  
[从master分支升级到new_master分支](https://github.com/iMeiji/ss-panel-v3-mod/wiki/%E4%BB%8Emaster%E5%88%86%E6%94%AF%E5%8D%87%E7%BA%A7%E5%88%B0new_master%E5%88%86%E6%94%AF)  

## 安装前端

1.安装 LNMP

```sh
screen -S lnmp
2G以上内存
wget http://soft.vpser.net/lnmp/lnmp1.3-full.tar.gz && tar xvzf lnmp1.3-full.tar.gz
cd lnmp1.3-full && ./install.sh
1G以下内存
wget http://soft.vpser.net/lnmp/lnmp1.2-full.tar.gz && tar xvzf lnmp1.2-full.tar.gz
cd lnmp1.2-full && ./install.sh
```

安装过程要求输入MySQL密码, 选择MySQL版本>=5.5, PHP版本>5.6.

安装大概需要半小时, 如果中途 ssh 断线, 输入 `screen -r lnmp` 

2.设置虚拟主机

`lnmp vhost add`

要求输入你的域名, 然后其余项都选 `no`

接着修改下 nginx

编辑 `/usr/local/nginx/conf/vhost/你的域名.conf`

然后添加下面这一段到 server

```sh
location / 
{
	try_files $uri $uri/ /index.php$is_args$args;		                
}
```

修改 root 那一行为

`root /home/wwwroot/你的域名/public;`

3.下载 sspanel 代码

```sh
cd /home/wwwroot/你的域名
apt-get install git -y
git clone -b master https://github.com/glzjin/ss-panel-v3-mod.git tmp && mv tmp/.git . && rm -rf tmp && git reset --hard
chown -R root:root *
chmod -R 755 *
chown -R www:www storage
chattr -i .user.ini
mv .user.ini public
cd public
chattr +i .user.ini
service nginx restart
```

4.安装 radius , 不使用VPN的话, 可以不进行这一步

```sh
# 先安装perl
apt-get install perl  
# 然后需要安装perl的DBI组件
perl -MCPAN -e shell
cpan>install DBI

//安装完成后退出cpan
cpan>quit
# 再安装其它组件
apt-get install freeradius freeradius-mysql freeradius-utils
```

5.配置数据库

浏览器打开 `http://你的vps ip/phpmyadmin`

用户 : root

密码 :安装 lnmp 时设置的

需要创建一个数据库和一个访问这个数据库的用户

点击 `用户` -> `新建` -> `添加用户` 

- 登录信息 :

  Username 选择 使用文本域 , 填写你的用户名 如 sspanel   

  Host 选择任意主机  %

  密码 选择使用文本域 填写密码

- 用户数据库 :

  勾选 创建与用户同名的数据库并授予所有权限

- 全局权限 :

  全选

接着按执行 选择刚刚新建的数据库 sspanel 导入程序目录下的 glzjin_all.sql

不适用 VPN 的话跳下一步



接着配置 radius , 创建个 radius 数据库和用户 (重复以上步骤)

选择 radius 数据库 导入 https://github.com/glzjin/Radius-install/raw/master/all.sql

回到 ssh 继续设置 radius

编辑 `/etc/freeradius/sql.conf`

配置 login(用户名), password(密码), radius_db(数据库名)等字段

找到 readclients 一行，设为 yes 并去掉注释符号#

然后下面是几个文件的覆盖

```sh
wget https://github.com/glzjin/Radius-install/raw/master/radiusd.conf -O /etc/freeradius/radiusd.conf
wget https://github.com/glzjin/Radius-install/raw/master/default -O /etc/freeradius/sites-enabled/default
wget https://github.com/glzjin/Radius-install/raw/master/dialup.conf -O /etc/freeradius/sql/mysql/dialup.conf
wget https://github.com/glzjin/Radius-install/raw/master/dictionary -O /etc/freeradius/dictionary
wget https://github.com/glzjin/Radius-install/raw/master/counter.conf -O /etc/freeradius/sql/mysql/counter.conf
```

Radius 配置完成,

```
service freeradius start
```

然后你也可以将该 freeradius 设为开机启动项

6.配置 sspanel

```sh
cd /home/wwwroot/你的域名
cp config/.config.php.example config/.config.php
# 编辑以下文件 建议使用 FTP 下载到本地修改
vi config/.config.php
```

由于配置太多 这里只说重点

```sh
$System_Config['key'] = '';			//修改此key为随机字符串确保网站安全
$System_Config['appName'] = '';             //站点名称
$System_Config['baseUrl'] = 'https://zhaojin97.cn';            // 站点地址
$System_Config['timeZone'] = 'PRC';        // RPC 天朝时间  UTC 格林时间
$System_Config['pwdMethod'] = 'sha256';       // 密码加密   可选 md5,sha256
$System_Config['salt'] = '';               // 密码加密用，从旧版升级请留空
$System_Config['authDriver'] = 'cookie';   // 登录验证存储方式,推荐使用Redis   可选: cookie,redis

$System_Config['mailDriver'] = 'mailgun';   // 邮件 可选 mailgun or smtp 需要支持qq邮箱的选 smtp

$System_Config['checkinMin'] = '100';       // 签到最少流量 单位MB
$System_Config['checkinMax'] = '500';       // 签到最多流量
$System_Config['defaultTraffic'] = '100';      // 用户初始流量 单位GB
$System_Config['inviteNum'] = '0';			// 注册后获得的邀请码数量

# database 数据库配置
$System_Config['db_driver'] = 'mysql';
$System_Config['db_host'] = 'localhost';		// 数据库地址
$System_Config['db_database'] = '';				// 数据库名称 sspanel
$System_Config['db_username'] = '';				// 数据库用户 sspanel
$System_Config['db_password'] = '';				// sspanel用户的密码
$System_Config['db_charset'] = 'utf8';
$System_Config['db_collation'] = 'utf8_general_ci';
$System_Config['db_prefix'] = '';

# redis
$System_Config['redis_scheme'] = 'tcp';			// 登录验证存储方式选了 redis 的话需要配置
$System_Config['redis_host'] = '127.0.0.1';
$System_Config['redis_port'] = '6379';
$System_Config['redis_database'] = '0';
$System_Config['redis_password']="";

# smtp
$System_Config['smtp_host'] = '';				// 例如 smtp.qq.com
$System_Config['smtp_username'] = '';
$System_Config['smtp_port'] = '25';
$System_Config['smtp_name'] = '';
$System_Config['smtp_sender'] = '';
$System_Config['smtp_passsword'] = '';
$System_Config['smtp_ssl'] = 'false';

#功能开关  需要用到的才开 建议先别动
$System_Config['enable_wecenter']='false';
$System_Config['enable_radius']='false';		// 配置了 radius 的话就开
$System_Config['enable_cloudxns']='false';
$System_Config['enable_duoshuo']='false';
$System_Config['enable_rss']='true';
$System_Config['enable_paymentwall']='false';

#Radius数据库设置
$System_Config['radius_db_host']='';		// 跟 上面 database 数据库配置差不多 换成radius即可
$System_Config['radius_db_database']='';
$System_Config['radius_db_user']='';
$System_Config['radius_db_password']='';

#Radius连接密钥
$System_Config['radius_secret']='';			// 这个重要 必须设

#端口池
$System_Config['min_port']='10000';			// SSR 分配端口号范围
$System_Config['max_port']='65535';

#两种方式相对于ss端口的偏移
$System_Config['pacp_offset']='-20000';		// PAC+ 和 PAC++ 用到
$System_Config['pacpp_offset']='-20000';

#测速周期/h
$System_Config['Speedtest_duration']='6';	// 对应后端 SSR 的 userapiconfig.py 里的 SPEEDTEST

#随机分组，注册时随机分配到的分组，多个分组请用英文半角逗号分隔。
$System_Config['ramdom_group']='0';			// 组别用于区分用户组 对应组只能访问对应组和0组的服务器 明白后再修改 

#充值返利百分比
$System_Config['code_payback']='20';		// 用户充值后 给邀请他注册的人返利多少%

#注册时的流量重置日以及需要重置的流量,0不重置
$System_Config['reg_auto_reset_day']='0';
$System_Config['reg_auto_reset_bandwidth']='100';	// 单位G
```

以上为 config 部分配置 完成后保存并上传到原目录下

切换到 ssh 窗口, 在你的网站目录下执行以下命令创建管理员

`php xcat createAdmin`  

按照提示, 输入管理员邮箱密码等信息, 然后执行以下命令同步用户

`php xcat syncusers`  

此时管理员创建完成

接下来需要对服务器进行计划任务的设置, 执行 crontab -e命令, 添加以下五段

```sh
30 22 * * * php /home/wwwroot/站点文件夹/xcat sendDiaryMail 
*/1 * * * * php /home/wwwroot/站点文件夹/xcat synclogin
*/1 * * * * php /home/wwwroot/站点文件夹/xcat syncvpn
0 0 * * * php -n /home/wwwroot/站点文件夹/xcat dailyjob
*/1 * * * * php /home/wwwroot/站点文件夹/xcat checkjob    
*/1 * * * * php -n /home/wwwroot/站点文件夹/xcat syncnas
```

重启Crontab
`/etc/init.d/cron restart`  

7.注意事项

检查时间是否为天朝时间

如果VPS默认是非中国时区的话, 如下命令可以用来更改为中国时区

`cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime`  

检查防火墙是否屏蔽数据库端口

```sh
# 允许本机访问
iptables -A INPUT -s 127.0.0.1/32 -p tcp -m tcp --dport 3306 -j ACCEPT
# 允许节点访问
iptables -A INPUT -s 节点IP -p tcp -m tcp --dport 3306 -j ACCEPT
# 允许所有IP访问
iptables -A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
```

查看防火墙规则

```sh
查看已添加的iptables规则
iptables -L -n --line-numbers
删除已添加的iptables规则
iptables -D INPUT line-numbers
```

规则保存

```sh
# Ubuntu
iptables-save > /etc/iptables.rules
# CentOS
service iptables save
```





## 安装后端

1.安装 libsodium

```sh
apt-get install build-essential wget -y
wget https://github.com/jedisct1/libsodium/releases/download/1.0.10/libsodium-1.0.10.tar.gz
tar xf libsodium-1.0.10.tar.gz && cd libsodium-1.0.10
./configure && make -j2 && make install
ldconfig
```

2.安装SSR

```sh
apt-get install python-pip git -y
pip install cymysql
cd ..
git clone -b manyuser https://github.com/glzjin/shadowsocks.git
cd shadowsocks
chmod +x *.sh
# 配置程序
cp apiconfig.py userapiconfig.py
cp config.json user-config.json
vi userapiconfig.py
```

然后主要编辑 userapiconfig.py, 来解释下里面各项配置的意思

```sh
# Config
#节点ID 对应前端节点列表的ID
NODE_ID = 1

#自动化测速，为0不测试，此处以小时为单位，要和 ss-panel 设置的小时数一致
SPEEDTEST = 6

#云安全，自动上报与下载封禁IP，1为开启，0为关闭
CLOUDSAFE = 1

#自动封禁SS密码和加密方式错误的 IP，1为开启，0为关闭
ANTISSATTACK = 0

#是否接受上级下发的命令，如果你要用这个命令，请参考我之前写的东西，公钥放在目录下的 ssshell.asc
AUTOEXEC = 1

#是否以多线程模式运行，关闭这个限速就会无效。请优先测试 1 ，开启试试，能运行没。
MULTI_THREAD = 0

#多端口单用户设置，看重大更新说明。
MU_SUFFIX = 'zhaoj.in'
#多端口单用户设置，看重大更新说明。
MU_REGEX = '%5m%id.%suffix'

#不明觉厉
SERVER_PUB_ADDR = '127.0.0.1' # mujson_mgr need this to generate ssr link
#此处不要修改
API_INTERFACE = 'glzjinmod' #mudbjson, sspanelv2, sspanelv3, sspanelv3ssr, muapiv2(not support)
#mudb，不要管
MUDB_FILE = 'mudb.json'

# Mysql 数据库连接信息
MYSQL_HOST = '127.0.0.1'
MYSQL_PORT = 3306
MYSQL_USER = 'ss'
MYSQL_PASS = 'ss'
MYSQL_DB = 'shadowsocks'
MYSQL_UPDATE_TIME = 60

# 是否启用SSL连接，0为关，1为开
MYSQL_SSL_ENABLE = 0

# 客户端证书目录，请看 https://github.com/glzjin/shadowsocks/wiki/Mysql-SSL%E9%85%8D%E7%BD%AE
MYSQL_SSL_CERT = '/root/shadowsocks/client-cert.pem'
MYSQL_SSL_KEY = '/root/shadowsocks/client-key.pem'
MYSQL_SSL_CA = '/root/shadowsocks/ca.pem'

# API，不用管
API_HOST = '127.0.0.1'
API_PORT = 80
API_PATH = '/mu/v2/'
API_TOKEN = 'abcdef'
API_UPDATE_TIME = 60

# Manager 不用管
MANAGE_PASS = 'ss233333333'

#if you want manage in other server you should set this value to global ip
MANAGE_BIND_IP = '127.0.0.1'

#make sure this port is idle
MANAGE_PORT = 23333
```

3.运行SSR

运行的话, 有几种方式

- python server.py 用于调错的
- ./run.sh 无日志后台运行
- ./logrun.sh 有日志后台运行
- supervisord

这里说下 使用Supervisor守护进程启动ssr

```sh
# 安装
apt-get install supervisor -y

# 写入配置
vi /etc/supervisor/conf.d/ssr.conf

# 写入以下内容
[program:ssr]
command=python /root/shadowsocks/server.py 
autorestart=true
autostart=true
user=root

# 重启Supervisor服务。
/etc/init.d/supervisor restart

# 重启 ssr
supervisorctl restart ssr

# 查看Supervisor服务运行状态。
supervisorctl status

# 如果遇到问题，可以检查日志：
supervisorctl tail -f ssr stderr

# 如果使用supervisor进程守护，需要修改文件vi /etc/default/supervisor，添加一行：
ulimit -n 1024000
```



## 强化安全

1.将网站支持 SSL 强化安全

需要提前准备好 SSL 证书文件, 没有的话可以使用[Let's Encrypt](https://certbot.eff.org/) 搞个免费SSL证书, 接着配置 nginx

编辑 ` /usr/local/nginx/conf/vhost/域名.conf`

```sh
server
{
	listen 80;
	#listen [::]:80; #有ipv6的开
	server_name 域名;
	rewrite ^(.*) https://$server_name$1 permanent;
}

server
{
	listen 443 ssl;
	#listen [::]:443 ssl; #有ipv6的开
	ssl_certificate /etc/letsencrypt/live/你的域名/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/你的域名/privkey.pem;
	ssl_trusted_certificate /etc/letsencrypt/live/你的域名/chain.pem;
}
```

2.禁止 http 的访问请求

由于 http 仍然可以访问, 所以我们需要将 http 的请求手动转移到 https, 由于 SSpanel 本身使用了重定向, 那么在不使用其他重定向的情况下, 最简单的方法就是用 html 网页的<meta> 参数

```html
<!DOCTYPE html>
<html>
<head>
  <meta http-equiv="refresh" content="0;url=https://域名">
  <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
</head>
</html>
```

3.转移 phpMyAdmin 目录

- 转移目录

  LNMP安装完毕以后默认的会在IP地址网站根目录生成一个 phpMyAdmin 的目录, 但是正是因为这个原因, 暴露了该目录, 我一般都会直接把这个文件夹转移到新的网站目录下, 比如转移到其他二级域名下的某个目录, 只需要在LNMP中新建一个二级域名而已, 然后把 phpMyAdmin 这个目录再转移到这个二级域名的网站目录下

- 限制IP

  在 phpMyAdmin 的目录下新增 `.htaccess` 写入以下

  ```html
  allow from 59.168.1.0/24
  allow from 59.168.2.50
  deny from all
  ```

  除了 59.168.1.0 至 59.168.1.24 的区间IP 与 59.168.2.50可以进入后台其于IP都封锁, 就算对方知道路径, IP不允许也是无法进入




***


59.168.1.0/24   是 1.1---1.254    并不是 59.168.1.0 至 59.168.1.24 

参考

[**ss-panel-v3-mod**安装说明](https://github.com/esdeathlove/ss-panel-v3-mod/wiki/%E5%AE%89%E8%A3%85%E8%AF%B4%E6%98%8E) 

[说明以及安装方法](https://github.com/esdeathlove/shadowsocks/wiki/%E8%AF%B4%E6%98%8E%E4%BB%A5%E5%8F%8A%E5%AE%89%E8%A3%85%E6%96%B9%E6%B3%95)

[Linux上iptables防火墙的基本应用教程](https://www.vpser.net/security/linux-iptables.html) 

