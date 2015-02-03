安装WordPress
=============

# 一、配置和安装数据库

- 使用登陆mysql系统。
首先使用我们之前配置的root用户登陆mysql数据库：
```shell
mysql -u root -p
```
- 创建并为该数据库增加一个用户，之后我们需要该用户名和数据库来安装wordpress。在Mysql环境中`mysql >`输入以下命令，注意以`;`来结束命令。
```shell
CREATE DATABASE dbname;
```
为数据库创建一个用户名和相应的密码来使用该数据库：
```shell
GRANT ALL ON dbname.* TO 'username' IDENTIFIED BY 'password';
```
然后刷新用户权限，使其生效：
```shell
FLUSH PRIVILEGES;
```
使用quit命令退出MySql：
```shell
QUIT
```
# 二、下载并上传wordpress
最新版的wordpress程序可以在https://cn.wordpress.org/上下载到。
下载之后将文件上传到相应的网站目录`/srv/www/sitename/public`。
当文件上传完成之后，可以将解压后的所有文件放在`/srv/www/sitename/public`目录下。

# 三、WordPress目录及文件权限设置
目前的用户状况为，我们有自己的ssh服务器管理账户，Web server程序apache为另外一个用户。如果要查看apache的用户名，我们可以使用下面的命令：
```shell
sudo lsof -i | grep :http
```
在列出的结果中，第三行即为httpd服务的用户名。使用下面的命令也可以找出httpd服务的用户名和用户组：
```shell
egrep -iw --color=auto '^user|^group' /etc/httpd/conf/httpd.conf
```
现在我们保证：

> 1. 我们的user是所有wordpress目录和文件的拥有者
> 2. 我们的user和httpd服务的用户名在同一个用户组中

- 用户组设置
先找出user本身的用户组：
```shell
groups
```
再找出httpd服务的用户组：
```shell
egrep -iw --color=auto '^user|^group' /etc/httpd/conf/httpd.conf
```
或者可以简单的用php代码来测试：
```php
echo exec( 'groups' );
```
在网站根目录建立一个`test.php`输入以上内容即可看到输出。
随后需要将user添加到httpd服务的用户名组：
```shell
sudo usermod -a -G httpd_group_name user_name
```
当执行完这个命令后，使用以下命令来确认命令生效：
```shell
groups user_name
```
我们需要把wordpress目录中的所有目录和子目录都改成`httpd_group_name`：
```shell
sudo find . -exec chown user_name:httpd_group_name {} +
```
使用`ls -al`命令来确认改变生效。
 
- 设置目录及文件访问权限
文件及目录权限的基本规则为：

> 1. 所有文件都是664权限，即所有文件都是所有者和其他人可读，所有者（wordpress）可写。
> 2. 所有目录都是775权限，即所有目录所有者和其他人都可访问。
> 3. `wp-config.php`文件权限为660，即其他人无法访问。

我们可以执行以下命令来进行我们的修改：
```shell
sudo find . -type f -exec chmod 664 {} +
sudo find . -type d -exec chmod 775 {} +
```
改变完权限之后，我们可以执行wordpress的安装，等安装完成之后，再修改`wp-config.php`的权限。

# 四、安装WordPress
按照提示进行安装，需要的时候输入数据的名称和密码即可。安装完成之后，修改`wp-config.php`的权限。
```shell
sudo chmod 660 wp-config.php
```
WordPress更新的时候会跳出需要输入FTP信息的对话框，需要在wp-config.php中加入
```php
define('FS_METHOD', 'direct' );
```
来使得更新能顺利执行。

# 参考文章

- [Proper WordPress Filesystem Permissions And Ownerships](http://www.smashingmagazine.com/2014/05/08/proper-wordpress-filesystem-permissions-ownerships/)