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
    "server":"127.0.0.1",
    "server_ipv6": "[::]",
    "server_port":8388,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"m",
    "timeout":300,
    "method":"rc4-md5"
}
```
#### 2.6 开启服务
`python server.py`  
如返回显示为 db start server at port 50000  ...之类的就表示服务端已设置成功    
如果放在后台运行则可以，`nohup python server.py & nohup` 用法可以百度，也可以用screen或者配置supervisor进程守护。
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
###### 2.6.2.1 debian使用supervisor进程守护的方法  
安装supervisor `apt-get install supervisor`  
配置supervisor进程守护  
在目录/etc/supervisor/conf.d/下， 新建一个文件，名字：shadowsocks.conf  
在shadowsocks.conf文件里编辑添加：  
```
[program:shadowsocks]
command=python /shadowsocks/shadowsocks/server.py -c /shadowsocks/shadowsocks/config.json #/此处目录请自行修改
autorestart=true
user=root
```
修改以下文件  
```
vi /etc/profile  
vi /etc/default/supervisor  
在文件结尾处添加以下3行内容
ulimit -n 51200
ulimit -Sn 4096
ulimit -Hn 8192
```
启动supervisor  
```
service supervisor start #启动
supervisorctl reload #重载
supervisorctl tail -f shadowsocks stderr #debug查看连接日志等,Ctrl+C 取消查看
```
###### 2.6.2.2 centos使用supervisor进程守护的方法  
安装`easy_install supervisor`  
运行`echo_supervisord_conf`测试是否安装成功。  
创建配置文件：`echo_supervisord_conf > /etc/supervisord.conf`  
修改配置文件：
在supervisord.conf最后增加：
```
[program:shadowsocks]
command = python /shadowsocks/shadowsocks/server.py -c /shadowsocks/shadowsocks/config.json #/此处目录请自行修改
autostart=true
autorestart=true
startsecs=3
```
使用指定配置文件启动：`/usr/bin/supervisord -c /etc/supervisord.conf/`  
-c 表示配置文件的路径，读取这里个配置文件，之前也是可以根据自己的情况放在不同的文件夹下  
修改配置文件之后：`supervisorctl reload` 重载 服务重新启动  
debug查看连接日志：`supervisorctl tail -f shadowsocks stderr` #Ctrl+C 取消查看  
设置supervisord开机启动  
编辑文件：`vi /etc/rc.local`  
在末尾另起一行添加`supervisord`  
supervisorctl常用命令
控制命令基本都通过supervisorctl执行，输入help可以看到命令列表。这是一些常用命令：
```
获得所有程序状态 supervisorctl status
关闭目标程序 supervisorctl stop spider
启动目标程序 supervisorctl start spider
关闭所有程序 supervisorctl shutdown
```






THX  
* [How To Install and Secure phpMyAdmin on a CentOS 6.4 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-a-centos-6-4-vps)  
* [centos安装phpmyadmin](http://www.cnblogs.com/tippoint/archive/2013/11/20/3434035.html)  
* [CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程](http://www.cmsky.com/shadowsocks-manyuser-sspanel/)
* [详细说下shadowsocks服务端的部署](http://www.bigf.info/makediess-manyuser-config-diy)
* CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程  