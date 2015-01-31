Centos 6.5 基本设置
=============

# 一、更改Host名称

当拿到系统的时候，或者刚刚完成系统的建立。第一步是更改主机的名称：
查看hostname的命令为：
```bash
hostname
hostname -f
```
第一个命令显示短的主机名，后一个显示完整的主机名fully qualified domain name (FQDN)。

更改hostname的方式：
需要修改两个地方，对不同的版本可以google不同系统的配置方法。
首先更改：
```shell
vi /etc/sysconfig/network
NETWORKING=yes
HOSTNAME=newHostName
```
然后是：
```shell
vi /etc/hosts
127.0.0.1 newHostName.com newHostName
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```
一般可以用域名yourdomian.com做主机名或者简单的yourdomain。有效的hostname明明规则可以在[wiki](http://en.wikipedia.org/wiki/Hostname)查看。其中第二步修改的是系统的FQND，必须要在域名解析中有与之对应的'A'记录与之对应。
修改完成之后需要重新启动系统使得更改在shell中和`hostname`命令中显示正常。

更多的关于更改hostname的方法可以自行搜索。

# 二、设置正确的系统日期和时间

具体的设置方法请google。

# 三、更新系统
```shell
yum update
```