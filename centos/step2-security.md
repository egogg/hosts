Centos基本安全设置
=================

# 一、添加非root用户

root用户一旦进行了误操作损失巨大，因此最好是先建立一个非root用户，用来做日常的系统管理。

- 添加用户命令：
```shell
adduser newuser
```
- 设置新用户的密码：
```shell
passwd newuser
```
- 增加用户的sodu权限
```shell
visudo
```
打开文件之后按照root用户的格式为新加用户添加权限。
```shell
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
newuser        ALL=(ALL)       ALL
```
之后退出root用户，使用新加用户进行系统管理。当需要系统权限的时候可以使用`sudo`，所有的`sudo`操作都会被添加到系统日志`/var/log/auth.log`里面。

# 二、禁用root用户远程ssh登陆

root用户名太容易猜测，因此需要将其禁用，另外ssh端口号最好也可以修改一下。

- 禁用远程root用户登陆
```shell
sudo vi /etc/ssh/sshd_conf
```
将远程root用户登陆禁用，修改下面行为no：
```shell
PermitRootLogin no
```
如果要修改默认端口号，可以将Port行修改为大于1024的任何值。
```shell
# Run ssh on a non-standard port:
Port 2345  #Change me
```
完成修改后，执行以下命令使得修改生效：
```shell
sudo service sshd restart
```

# 三、设置防火墙
设置防火墙可以关掉一些不常见的端口，大大降低系统网络消耗，同时也可以使得系统更加健壮。对不同版本的操作系统可以搜索相应的防火墙配置方法，对有的系统，可能默认的已经带了有很好特性的防火墙了。

- 防火墙默认设置查看：
```shell
sudo iptables -L
```
如果系统没有进行过设置，可以看到一些默认的设置：
```shell
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
- 创建防火墙规则文件
```shell
sudo vim /etc/iptables.firewall.rules
```
添加如下内容：
```shell
*filter

#  Allow all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT -d 127.0.0.0/8 -j REJECT

#  Accept all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

#  Allow all outbound traffic - you can modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

#  Allow HTTP and HTTPS connections from anywhere (the normal ports for websites and SSL).
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT

#  Allow SSH connections
#
#  The -dport number should be the same port number you set in sshd_config
#
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

#  Allow ping
-A INPUT -p icmp --icmp-type echo-request -j ACCEPT

#  Log iptables denied calls
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

#  Drop all other inbound - default deny unless explicitly allowed policy
-A INPUT -j DROP
-A FORWARD -j DROP

COMMIT
```
这些规则把出了HTTP (80)， HTTPS (443)， SSH (22) 还有 ping之外的所有端口都屏蔽掉了。
>**注意**：当以后增加了新的服务之后，要再重新检查一下这些规则。

- 让这些规则生效
```shell
sudo iptables-restore < /etc/iptables.firewall.rules
```
之后重新输入查看命令`sudo iptables -L`，我们看到规则已经生效：
```shell
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere
REJECT     all  --  anywhere             loopback/8          reject-with icmp-port-unreachable
ACCEPT     all  --  anywhere             anywhere            state RELATED,ESTABLISHED
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:http
ACCEPT     tcp  --  anywhere             anywhere            tcp dpt:https
ACCEPT     tcp  --  anywhere             anywhere            state NEW tcp dpt:ssh
ACCEPT     icmp --  anywhere             anywhere            icmp echo-request
LOG        all  --  anywhere             anywhere            limit: avg 5/min burst 5 LOG level debug prefix `iptables denied: '
DROP       all  --  anywhere             anywhere

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  anywhere             anywhere

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  anywhere             anywhere
```
- 让防火墙规则重启有效
```shell
sudo service iptables save
```
# 四、安装和配置Fail2Ban
Fail2Ban可以防止密码字典暴力破解。当SSH, HTTP 和 SMTP 服务检测到多次重复失败登陆时，Fail2Ban创建临时防火墙规则来禁用这个ip。默认Fail2Ban只对SSH登陆做防护。

- Fail2Ban的安装
```shell
sudo yum install epel-release
sudo yum install fail2ban
```
- 修改fail2ban的配置文件
我们需要创建一个jail.local文件来覆盖fail2ban的默认规则。我们可以把jail.conf拷贝一份，让后修改里面的选项。
```shell
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
我们可以编辑`/etc/fail2ban/jail.local`中的一些选项来进行配置。
例如：
`bantime`表示封禁时间
`maxretry`表示最大可尝试次数
修改之后保存文件。输入以下命令让修改生效：
```shell
sudo service fail2ban restart
```
如果要查看fail2ban的日志文件，可以在`/var/log/fail2ban.log`中找到。Fail2ban的官方网站为!http://www.fail2ban.org。

