## phpMyAdmin
### Step One — Install a LAMP Stack
`yum install httpd mysql-server php php-mysql`  
### Step Two — Configure the LAMP Stack
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
### Step Three — Install phpMyAdmin
```
yum install epel-release
yum install phpmyadmin
```
### Step Four — Configure phpMyAdmin
```
vi /etc/httpd/conf.d/phpMyAdmin.conf  #Configure Apache Files
```
You need to restart the server with the following command:  
`service httpd restart`  
### Step Five — See the Results!
`http://VPS_IP_address/phpmyadmin`


***

THX  
* [How To Install and Secure phpMyAdmin on a CentOS 6.4 VPS](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-on-a-centos-6-4-vps)  
* [centos安装phpmyadmin](http://www.cnblogs.com/tippoint/archive/2013/11/20/3434035.html)  
* [CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程](http://www.cmsky.com/shadowsocks-manyuser-sspanel/)
* [详细说下shadowsocks服务端的部署](http://www.bigf.info/makediess-manyuser-config-diy)
* CENTOS下SHADOWSOCKS多用户后端MANYUSER+前端SSPANEL搭建过程  