### 1. 搭建phpMyAdmin
### 1.1 Install a LAMP Stack
`yum install httpd mysql-server php php-mysql`  
### 1.2 Configure the LAMP Stack
```
service httpd start  #Start the Web Server
http://server_IP_addr  #Check that the server is up and running by visiting your server's IP address in a web browser:
```
Configure MySQL  
```
service mysqld start
mysql_secure_installation
mysql -u root -p  #You can test that your database password was set correctly
mysql>
mysql> exit  #Type exit to return to your shell session
```
### 1.3 Install phpMyAdmin
```
yum install epel-release
yum install phpmyadmin
```
### 1.4 Configure phpMyAdmin
```
vi /etc/httpd/conf.d/phpMyAdmin.conf  #Configure Apache Files
```
You need to restart the server with the following command:  
`service httpd restart`  
### 1.5 See the Results!
`http://VPS_IP_address/phpmyadmin`


***

### 2. 安装shadowsocks多用户后端shadowsocks-manyuser
项目地址：https://github.com/mengskysama/shadowsocks-rm/tree/manyuser  
#### 2.1 先安装需要的环境依赖
```
apt-get install python-pip python-m2crypto  #Debian / Ubuntu
yum install m2crypto python-setuptools  #CentOS:
easy_install pip
```

#### 2.2 安装cymysql
`pip install cymysql`  

#### 2.3 安装shadowsocks-manyuser
`git clone -b manyuser https://github.com/mengskysama/shadowsocks.git`  
或者`git clone -b manyuser https://github.com/mengskysama/shadowsocks-rm.git`  
如果提示没有安装git，则先安装git `yum -y install git`  

#### 2.4 创建数据库shadowsocks
登陆`http://VPS_IP_address/phpmyadmin`，第一步的时候已经创建用户  
创建数据库shadowsocks（名称随意），然后将shadowsocks-manyuser文件夹中的shadowsocks.sql导入到你创建的数据库中。  

#### 2.5 配置数据库连接
cd /用户名/shadowsocks/shadowsocks 打开Config.py所在文件夹  
编辑Config.py文件：vi Config.py， vi config.json 修改对应的端口、密码等等操作。如下格式  
```
#Config
MYSQL_HOST = '127.0.0.1' #这一行是服务器IP，127.0.0.1表示本机
MYSQL_PORT = 3306 #数据库端口号
MYSQL_USER = 'ss' #数据库用户名
MYSQL_PASS = 'ss' #数据库密码
MYSQL_DB = 'shadowsocks' #数据库名称
MANAGE_PASS = 'ss233333333'
#if you want manage in other server you should set this value to global ip
MANAGE_BIND_IP = '127.0.0.1'
#make sure this port is idle
MANAGE_PORT = 23333
```
```
{
    "server":"VPS IP",
    "server_ipv6": "[::]",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"m",
    "timeout":300,
    "method":"aes-256-cfb"
}
```

#### 2.6 开启服务
`python server.py`  
如返回显示为 db start server at port 50000  ...之类的就表示服务端已设置成功   
   
如果放在后台运行则可以  
```
nohup python server.py > /var/log/shadowsocks.log 2>&1 &
cat /var/log/shadowsocks.log
```
也可以用screen或者配置supervisor进程守护。  

##### 2.6.1 使用screen进程守护的方法
```
screen -S shadowsocks
cd /root/shadowsocks/shadowsocks/
python server.py
```
此时按住Ctrl+a，然后按d退出就可以了。 
查看screen运行任务可以screen -ls可以看到刚才创建的ss任务在运行。  
如果需要恢复执行：  
screen -r shadowsocks  
当然，出意外的话killall也是可以。  
Shadowsocks开机自动启动并后台运行  
`vi /etc/rc.local`  
添加以下内容  
```
cd /root/shadowsocks/shadowsocks
screen -dmS Shadowsocks python server.py
```
（`screen -dmS Shadowsocks python server.py`，也就是让程序运行后就在后台了。）  
至此多用户SS后端安装完毕。  
##### 2.6.2 使用supervisor进程守护的方法
Supervisor是一个C/S系统,它可以在类UNIX系统上控制系统进程，由python编写，它提供了大量的功能来实现对进程的管理。  
安装  
```
pip
pip install supervisor

easy_install
easy_install supervisor

apt-get (Debian/Ubuntu)
apt-get update
apt-get install supervisor

yum (Centos)
yum install supervisor
```
编辑 `/etc/supervisor/conf.d/shadowsocks.conf`
```
[program:shadowsocks]
command=cd /root/shadowsocks/shadowsocks  python server.py #自己修改对应的路径
autostart=true
autorestart=true
user=nobody  #如果端口 < 1024，把 user=nobody 改成 user=root。
```
在 `/etc/default/supervisor` 最后加一行：`ulimit -n 51200`  
  
执行  
```
service supervisor start
supervisorctl reload
```
你可以检查日志或者控制 Shadowsocks 的运行：  
```
supervisorctl tail -f shadowsocks stderr
supervisorctl restart shadowsocks
```
如果更新了 Supervisor 的配置文件（/etc/supervisor.d/*.conf），可以 supervisorctl update 来更新配置。  

常用命令  
```
获得所有程序状态 supervisorctl status
关闭目标程序 supervisorctl stop shadowsocks
启动目标程序 supervisorctl start shadowsocks
关闭所有程序 supervisorctl shutdown
```
### 3. 安装shadowsocks多用户前端MakeDieSS
项目地址：https://github.com/mengskysama/MakeDieSS  

#### 3.1 安装
```
cd /var/www/html
git clone https://github.com/mengskysama/MakeDieSS.git
```  
打开MakeDieSS目录，修改function.php  
```
<?php

$COOKIEEXPTIME = 3600 * 24;
$BASEURL = '192.168.1.215';	//用户管理面板的域名
$mysql_host = '127.0.0.1';	//sql地址
$mysql_user = 'sqlname';	//sql用户名
$mysql_pass = 'sqlpassword';	//sql密码
$mysql_db = 'shadowsocks';	//sql名
$init_transfer = 1024 * 1024 * 1024;
$arr_server = array(
    array('uri' => 'http://mdss01.mengsky.net:80', 'key' => 'nicaicai', 'type' => 1)
    //,
    //array('uri'=>'http://mdss10.mengsky.net:80', 'key'=>'nicaicai', 'type'=>1)
);

```
接着是panel.php，这里要修改服务器的地址、兑换临时流量大小的前端显示  
![panel](https://pic.honeyhaw.com/images/panelumu.jpg)

在api.php里找到如下图所示的地方，修改兑换临时流量的大小  
![api](https://pic.honeyhaw.com/images/api.jpg)

最后是reg.php里面修改邀请码,默认v2ex  
最后将sql.sql导入到你新建的数据库中，你也可以在导入前先作适当的修改，参考下图
![sql](https://pic.honeyhaw.com/images/sql.jpg)
默认用户名test@test.com，密码10010  


***

THX  
* [How To Install and Secure phpMyAdmin on a CentOS 6.4 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-a-centos-6-4-vps)  
* [centos安装phpmyadmin](http://www.cnblogs.com/tippoint/archive/2013/11/20/3434035.html)  
* [CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程](http://www.cmsky.com/shadowsocks-manyuser-sspanel/)
* [详细说下shadowsocks服务端的部署](http://www.bigf.info/makediess-manyuser-config-diy)
* [用 Supervisor 运行 Shadowsocks](https://github.com/shadowsocks/shadowsocks/wiki/%E7%94%A8-Supervisor-%E8%BF%90%E8%A1%8C-Shadowsocks/9b73d02b379470f56d55aa01da7f7cd9b29f6d50)
* [shadowsocks前端管理面板ss-panel详细配置及修改图文教程](http://www.bigf.info/ss-panel-config-diy) 