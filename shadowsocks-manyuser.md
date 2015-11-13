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
编辑Config.py文件：vi Config.py，修改对应的端口、密码等等操作。如下格式  
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
#### 2.6 开启服务
`python server.py`  
![图片来自cmsky](http://cmsky.qiniudn.com/wp-content/uploads/2015/06/shadowsocksmanyuser.png)
如上图所示就算安装成功了。  





THX  
* [How To Install and Secure phpMyAdmin on a CentOS 6.4 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-a-centos-6-4-vps)  
* [centos安装phpmyadmin](http://www.cnblogs.com/tippoint/archive/2013/11/20/3434035.html)  
* [CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程](http://www.cmsky.com/shadowsocks-manyuser-sspanel/)
* [详细说下shadowsocks服务端的部署](http://www.bigf.info/makediess-manyuser-config-diy)
* CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程  