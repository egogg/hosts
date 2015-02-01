配置安装LAMP环境
=================

在CentOS上配置基本的Apache，MySql，PHP环境。

#一、安装Apache

- 安装Apache Server的命令：
```shell
yum update
yum install httpd
```
- 设置网站目录
在/srv目录下创建`www`目录和相应的网站目录。
```shell
sudo mkdir -p /srv/www/yoursite/public
sudo mkdir -p /srv/www/yoursite/logs
```
修改目录权限：
```shell
chown -R user:user /srv/www/yoursite/public
```
让`www`目录可以被任何人访问：
```shell
chmod 755 /srv/www
```
- 配置基于命名的虚拟主机
一般在`/et/httpd/conf.d/`目录下以`.conf`结尾的文件，Apache都把它当成配置文件。现在改目录下创建一个`vhost.conf`文件
```shell
touch /etc/httpd/conf.d/vhost.conf
vi /etc/httpd/conf.d/vhost.conf
```
- 启动Apache Server
Apache的配置文件在`/etc/httpd/conf/httpd.conf`中。修改完之后可以可以使用一下命令让修改生效。
```shell
sudo service httpd restart
```
把服务添加到系统启动中：
```shell
sudo chkconfig --levels 235 httpd on
```
当修改`vhost.conf`之后，需要重新reload系统：
```shell
sudo service httpd reload
```
# 二、安装和配置MySql

- MySql安装
```shell
yum install mysql-server 
```
- 启动MySql服务
```shell
sudo service mysqld start
```
- 将服务加入启动
```shell
sudo chkconfig --levels 235 mysqld on
```
MySql Server的配置文件在`/etc/my.cnf`中。

- 配置MySql
```shell
sudo mysql_secure_installation
```
需要当前root密码的时候直接按`Enter`，然后按照提示配置新的root密码。

# 三、安装和配置PHP

- 安装php
```shell
sudo yum install php php-pear
```
php的配置文件在`/etc/php.ini`中。

- 安装php-mysql
```shell
sudo yum install php php-mysql
```
如果需要其他php模块可以使用`yum search php-`命令查找，然后可以使用`yum info php-module-name`来查看模块的具体功能。当完成修改之后需要重新启动Apache服务：
```shell
sudo service httpd restart
```