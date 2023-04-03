
# Hello World

---

## [Linux VPS 服务器基础安全设置](https://p3terx.com/archives/improve-linux-server-security.html)
### 前言  
大多数的 VPS 购买来默认是 root 账户登录，这就像在电脑城买的电脑奸商装的盗版 Win­dows 默认账户是 Ad­min­is­tra­tor 。起初跟大多数人一样，我都是直接使用的，为了方便也曾把 GCP 弄成了 root 账户直接登录，所以对用户权限、密钥登录等安全知识没什么概念。后来通过一系列的学习和实践，开始配置密钥登录，也随着使用 Linux 桌面版 和 WSL ，开始习惯使用普通账户，对这些安全知识也逐渐有了全面的认识。这篇文章将讲解在 Linux 下如何添加普通用户、授予用户 sudo 权限、配置密钥登录、禁止 root 登录等一些列提升安全性的操作，相信这对于刚接触 VPS 的萌新应该会很有用。
### 添加普通用户
为了安全，平时我们应该以普通用户的身份操作 VPS，所以需要添加一个普通用户。添加用户有两个命令，adduser 和 useradd，在不同系统中的定义以及用法上有区别  
以添加用户名为 `p3terx` 的普通用户为例子，输入命令
```
useradd -m -s /bin/bash p3terx
```
然后对该用户设置密码，输入命令后会提示输入两次密码
```
passwd p3terx
```
### 授予普通用户 sudo 权限
有时需要使用 root 权限，比如安装软件、启动服务等操作时就需要用到 `sudo` 命令来提升权限才能进行操作。授予用户 sudo 权限最简单的方法是把用户添加到 sudo 用户组。  
如果系统中没有 `sudo`，需要先安装。
```
# Debian
apt install sudo -y
# Centos
yum install sudo -y
```
以添加 `p3terx` 这个用户到 sudo 用户组为例子，输入下面命令：
```
usermod -aG sudo p3terx
```
### 配置 SSH 密钥登录
密码的缺点是很容易被暴力破解，而且密码需要记忆，使用起来麻烦。密钥的好处是，你只需要一对密钥文件：公钥和私钥，公钥相当于门锁，装在 VPS 上，私钥相当于钥匙，放在本地计算机上，登录的过程就像是用钥匙去开锁，有钥匙的人才能打得开，不仅安全，而且方便。  
需要注意的是配置前一定要切换到之前新建的普通用户，以切换到 `p3terx` 这个用户为例子，输入以下命令：
```
su -l p3terx
```
### 禁用不安全的登录方式
前面的一系列操作都是铺垫，是为禁止 root 账户登录和密码登录以及修改 SSH 端口做准备，这才是提升 VPS 安全性的主要目的。  
打开 sshd 配置文件 (`/etc/ssh/sshd_config`) 进行修改。
```
sudo nano /etc/ssh/sshd_config
```
#### 禁止密码登录
找到 `PasswordAuthentication`，一般情况看到的应该是这样的：
```
#PasswordAuthentication yes
```
去掉前面的`#`，把 `yes` 改为 `no`，像下面这样：
```
PasswordAuthentication no
```
#### 禁止 root 登录
找到 `PermitRootLogin`，对默认使用 root 登录的 VPS ，看到的应该是这样的：
```
PermitRootLogin yes
```
要完全禁止 root 登录，把 `yes` 改为 `no`，像下面这样：
```
PermitRootLogin no
```
#### 修改 SSH 端口
找到 `Port`，默认情况下这个选项是被注释，像下面这样：
```
#Port 22
```
去掉前面的`#`，把后面的 `22` 换成其它端口，比如 `2333`，像下面这样：
```
Port 2333
```
#### 重启 sshd 服务
为了使以上修改生效，需要重启 sshd 服务
```
sudo service sshd restart
```
#### 清除 root 用户密码
当清除 root 用户密码后，就无法使用 `su` 命令切换到 root 用户。只有被授予 sudo 权限的用户执行 `sudo -i` 命令并输入当前用户的密码才能切换到 root 用户，进一步提升了安全性。
```
sudo passwd -d root
```
### OVER

---

## [Debian/Ubuntu 中安装和配置 UFW（简单防火墙）](https://p3terx.com/archives/installing-and-configuring-ufw-in-debian.html)
### 前言
UFW 是一个 Arch Linux、De­bian 或 Ubuntu 中管理防火墙规则的前端，可大大简化防火墙的配置过程。
### 安装 UFW
如还没有安装，可以使用 apt 命令来安装
```
apt-get update && apt-get install ufw
```
在使用前，你应该检查下 UFW 是否已经在运行。
```
ufw status
```
如果你发现状态是 `inactive` ，意思是没有被激活或不起作用。
### 启用/禁用
```
ufw enable  #启用
ufw disable  #禁用
```
### 使用与配置
#### 列出当前UFW规则
```
ufw status verbose
```
#### 添加规则  
允许入站（allow）  
默认情况，没有允许就是拒绝（入站），使用 `ufw allow <端口>` 来添加允许访问的端口或协议。
```
ufw allow ssh  #添加22端口

ufw allow http  #添加80端口

ufw allow https  #添加443端口

ufw allow 2333/tcp   #添加2333端口，仅TCP协议

ufw allow 6666/udp   #添加6666端口，仅UDP协议

ufw allow 8888:9999  #添加8888到9999之间的端口
```
拒绝访问（deny）  
使用 `ufw deny <端口>` 来添加拒绝入站的端和协议，与添加允许的类似。
#### 删除规则  
先使用 `ufw status` 查看规则，再使用 `ufw delete [规则] <端口>` 来删除规则。
```
ufw delete allow 2333/tcp
```
如果你有很多条规则，使用 `numbered` 参数，可以在每条规则上加个序号数字。  
然后使用 `ufw delete <序号>` 来删除规则。
```
root@p3terx:~# ufw status numbered  #列出规则，并加上序号。
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 20,21,22,80,888,8888/tcp   ALLOW IN    Anywhere
[ 2] 39000:40000/tcp            ALLOW IN    Anywhere
[ 3] 8896/tcp                   ALLOW IN    Anywhere
[ 4] 8896/udp                   ALLOW IN    Anywhere
[ 5] 443/tcp                    ALLOW IN    Anywhere
[ 6] 20,21,22,80,888,8888/tcp (v6) ALLOW IN    Anywhere (v6)
[ 7] 39000:40000/tcp (v6)       ALLOW IN    Anywhere (v6)
[ 8] 8896/tcp (v6)              ALLOW IN    Anywhere (v6)
[ 9] 8896/udp (v6)              ALLOW IN    Anywhere (v6)
[10] 443/tcp (v6)               ALLOW IN    Anywhere (v6)

root@p3terx:~# ufw delete 4  #删除上面的第4条规则
Deleting:
 allow 8896/udp
Proceed with operation (y|n)? y  #最后会询问你是否进行操作
```
以上都是一些简单常用的一些命令，想要深入了解，可以输入 man ufw 查看 ufw 用户手册。
### 参考  
[Debian/Ubuntu 系统中安装和配置 UFW－简单的防火墙](https://p3terx.com/go/aHR0cHM6Ly9saW51eC5jbi9hcnRpY2xlLTI0ODktMS5odG1s)
### OVER

---

## [Linux 下使用 vnStat 统计 VPS 流量](https://p3terx.com/archives/statistics-vps-traffic-using-vnstat-under-linux.html)
### 前言  
除了服务商提供的面板，我们也可以安装 vnStat 来监控你的 VPS 或服务器的流量使用情况，vn­Stat 安装方法很简单，可分为编译安装或者直接通过源安装。由于源安装一般不是最新版本，推荐使用编译安装。
### 安装
+ 编译安装
输入以下命令下载源文件
```
git clone https://github.com/vergoh/vnstat.git
```
进入 `vnstat` 目录
```
cd vnstat
```
编译文件
```
./configure --prefix=/usr --sysconfdir=/etc && make
```
安装
```
make install
```
如果需要卸载则输入 `make uninstall`
### 安装服务脚本
`examples` 目录下包含了最常用的服务脚本文件，根据不同的系统进行以下操作。  

Debian / Ubuntu:
```
cp -v examples/init.d/debian/vnstat /etc/init.d/
update-rc.d vnstat defaults
service vnstat start
```
Red Hat / CentOS:
```
cp -v examples/init.d/redhat/vnstat /etc/init.d/
chkconfig vnstat on
service vnstat start
```
如遇到 `Failed to restart vnstat.service: Unit vnstat.service is masked.` 请删除 `/etc/systemd/system/` 下的 `vnstat.service` 文件。

### 源安装  
源安装比编译安装方法更简单，但一般不是最新版本。  
Debian / Ubuntu 下直接使用 `apt-get` 安装即可：
```
apt-get install vnstat
```
Centos 需要先安装 epel 源后才能使用 `yum` 来安装：
```
yum install epel-release -y
yum install -y vnstat
```
### 修改配置  
输入 `ifconfig` 命令查看自己的网卡名。一般来说 OVZ 的网卡是 `venet0`，而 XEN 和 KVM 的网卡是 `eth0`。  
然后修改配置文件
```
vi /etc/vnstat.conf
```
修改 `Interface` 选项
```
## KVM / XEN
Interface "eth0"

## OpenVZ
Interface "venet0"
```
`MonthRotate` 为每月流量结算日期，也就是每月流量重新计算的日期，默认为每月 1 日，根据需要修改。  
其它选项可查看[官方配置文档](https://p3terx.com/go/aHR0cDovL2h1bWRpLm5ldC92bnN0YXQvbWFuL3Zuc3RhdC5jb25mLmh0bWw)  
修改好配置后使用 `service vnstat restart` 命令来重启 vnStat。  
### 生成数据库  
同样的，OVZ 的网卡是 `venet0`，而 XEN 和 KVM 的网卡是 `eth0`，根据实际情况来输入以下命令来生成数据库。
```
## KVM / XEN
vnstat -u -i eth0

## OpenVZ
vnstat -u -i venet0
```
数据库目录：`/var/lib/vnstat/`  
删除数据库 `vnstat --delete --force -i eth0`
### 使用方法
使用 `vnstat --help` 命令来查看详细使用方法。
#### 流量统计查询
```
vnstat -h    #按小时查询
vnstat -d    #按天数查询
vnstat -m    #按月数查询
vnstat -w    #按周数查询
vnstat -t    #查询TOP10
```
#### 查询实时流量
```
## KVM / XEN
vnstat -l -i eth0 -ru

## OpenVZ
vnstat -l -i venet0 -ru
```
#### 服务命令  
启动 vnStat：`service vnstat start`

停止 vnStat：`service vnstat stop`

重启 vnStat：`service vnstat restart`

查看 vnStat 状态：`service vnstat status`
### 使用 ServerStatus-V 查看流量统计
---
## [ServerStatus-V](https://github.com/P3TERX/ServerStatus-V)  
ServerStatus-V 是一个酷炫高逼格的云探针、云监控、服务器云监控、多服务器探针。使用方便，信息直观。ServerStatus-V 是 ServerStatus 中文版 项目的优化 / 修改版。原版调用的网卡流量数据，缺点是重启后流量信息会清零。而 ServerStatus-V 直接调用 vnStat 月流量数据。
![demo-desktop](https://user-images.githubusercontent.com/128745328/229630425-4ef244a0-5771-4075-b88f-9862ee32be32.png)
### 目录介绍
+ `clients` 客户端文件
+ `server` 服务端文件
+ `web` 网站文件
+ `init.d` 服务脚本
### 安装教程  
执行下面的代码下载并运行脚本，如果脚本有更新，也可使用下面的命令来更新。
```
wget -N --no-check-certificate https://raw.githubusercontent.com/P3TERX/ServerStatus-V/master/status.sh && chmod +x status.sh && bash status.sh
```
下载脚本后，根据需要安装客户端或者服务端：
```
# 客户端菜单
bash status.sh c
 
# 服务端菜单
bash status.sh s
```
运行脚本后会出现脚本操作菜单，选择并输入`1`就会开始安装。
### 简单步骤
首先安装服务端，安装过程中会提示：
```
是否由脚本自动配置HTTP服务(服务端的在线监控网站)[Y/n]
 
# 如果你不懂，那就直接回车，如果你想用其他的HTTP服务自己配置，那么请输入 n 并回车。
# 注意，当你曾经安装过 服务端，同时没有卸载Caddy(HTTP服务)，那么重新安装服务端的时候，请输入 n 并回车。
```
然后 添加或修改 初始示例的节点配置，注意用户名每个节点配置都不能重复，其他的参数都无所谓了。  
然后安装客户端，根据提示填写 服务端的IP 和前面添加/修改 对应的 节点用户名和密码（用于和服务端验证），然后启动就好了，有问题请贴出 详细步骤+日志(如果有)联系我。
### 使用说明    
进入下载脚本的目录并运行脚本：
```
# 客户端管理菜单
./status.sh c
# 服务端管理菜单
./status.sh s
```
然后选择你要执行的选项即可。
```
ServerStatus-V 安装&管理脚本 [v1.3.1]
https://github.com/P3TERX/ServerStatus-V

 0. 切换到 服务端菜单  
——————————————  
 1. 安装 客户端  
 2. 卸载 客户端  
——————————————  
 3. 启动 客户端  
 4. 停止 客户端  
 5. 重启 客户端  
——————————————  
 6. 设置 客户端配置  
 7. 查看 客户端信息  
 8. 查看 客户端日志  
——————————————  
 9. 更新脚本  

 当前状态: 客户端 未安装  

 请输入数字 [0-9]:
 ```
### 其他操作  
#### 客户端  
启动：`service status-client start`

停止：`service status-client stop`

重启：`service status-client restart`

查看状态：`service status-client status`  

查看客户端日志：`tail -f tmp/serverstatus_client.log`  
#### 服务端  
启动：`service status-server start`

停止：`service status-server stop`

重启：`service status-server restart`

查看状态：`service status-server status`

查看服务端日志：`tail -f /tmp/serverstatus_server.log`  
#### Caddy（HTTP服务）  
启动：`service caddy start`

停止：`service caddy stop`

重启：`service caddy restart`

查看状态：`service caddy status`

Caddy配置文件：`/usr/local/caddy/caddy`  

默认脚本只能一开始安装的时候设置配置文件。Caddy的使用方法，可以参考[官方手册](https://caddyserver.com/docs)。

### 目录  
安装目录：`/usr/local/ServerStatus`

网页文件：`/usr/local/ServerStatus/web`

配置文件：`/usr/local/ServerStatus/server/config.json`

### 其他说明
`实时网速`单位为：G=GB/s，M=MB/s，K=KB/s

`流量`单位为：T=TB，G=GB，M=MB，K=KB
