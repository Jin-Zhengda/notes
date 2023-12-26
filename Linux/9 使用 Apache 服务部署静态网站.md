## 网站服务程序
Apache（httpd）服务程序可以运行在 Linux 系统、UNIX 系统甚至是 Windows 系统中，它支持基于 IP、域名及端口号的虚拟主机功能，支持多种认证方式，集成有代理服务器模块、安全 Socket（SSL）， 能够实时监视服务状态与定制日志消息，并支持各类丰富的模块。

安装 Apache 后，启用并将其加入开机启动项之中：
```Shell
systemctl start httpd
systemctl enable httpd
```


## 配置服务文件参数
在 httpd 服务程序的主配置文件中，存在3种类型的信息：注释行信息、全局配置、区域配置。 
全局配置参数可作用于所有的子站点，既保证了子站点的正常访问，也有效降低了频繁写入重复参数的工作量。区域配置参数则是单独针对每个独立的子站点进行设置的。
httpd 服务的配置文件：

|作用|文件名称|
|---|---|
|服务目录|/etc/httpd|
|主配置文件|/etc/httpd/conf/httpd.conf|
|网站数据目录|var/www/html|
|访问日志|/var/log/httpd/access_log|
|错误日志|/var/log/httpd/error_log|

主配置文件参数：
```Shell
ServerRoot //服务目录 
ServerAdmin //管理员邮箱 
User //运行服务的用户 
Group //运行服务的用户组 
ServerName //网站服务器的域名 
DocumentRoot //网站数据目录 
Listen //监听的IP地址与端口号 
DirectoryIndex //默认的索引页页面 
ErrorLog //错误日志文件 
CustomLog //访问日志文件 
Timeout //网页超时时间，默认为300秒
```
在默认情况下，网站数据保存在 /var/www/html 目录中。


## SELinux 安全子系统
SELinux 能从多方面监控违法行为：对服务程序的功能进行限制（SELinux 域限制可以确保服务程序做不了出格的事情）；对文件资源的访问进行限制（SELinux 安全上下文确保文件资源只能被其所属的服务程序进行访问）。

SELinux 服务有3种配置模式：
- enforcing：强制启用安全策略模式，将拦截服务的不合法请求。
- permissive：遇到服务越权访问时，只发出警告而不强制拦截。
- disabled：对于越权的行为不警告也不拦截。

SELinux 域和 SELinux 安全上下文是 Linux 系统中的双保险。

在 SELinux 服务主配置文件中可以定义其默认状态：
```Shell
vim /etc/selinux/config

//修改下列信息
SELINUX=enforcing
```
查看当前 SELinux 运行模式：
```Shell
getenforce
```
暂时修改 SELinux 运行模式（0 为禁用，1 为启用）：
```Shell
setenforce 0
```

`semanage` 命令用于管理 SELinux 的策略，格式为 `semanage [参数] [文件]`。

参数：
- `-l` 查询
- `-a` 添加
- `-m` 修改
- `-d` 删除

在文件上设置的 SELinux 安全上下文是由用户段、角色段以及类型段等多个信息项共同组成的。其中，用户段 system_u 代表系统进程的身份，角色段 object_r 代表文件目录的角色， 类型段 httpd_sys_content_t 代表网站服务的系统文件。

在 ls 命令中，-Z 参数用于查看文件的安全上下文值，-d 参数代表对象是个文件夹：
```Shell
ls -Zd /var/www/html

ls -Zd /home/wwwroot
```

安全上下文匹配时，文件操作才会被允许。
例如，向 /home/wwwroot 目录中新添加一条 SELinux 安全上下文，使这个目录能够作为网站数据目录，即该目录以及里面的所有文件能够被 httpd 服务程序访问到：
```Shell
semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/*
```
并使改设置立即生效：
```Shell
restorecon -Rv /home/wwwroot/
```
`-R` 表示对目录进行递归操作，`-v` 表示显示详细信息。


## 个人用户主页功能
httpd 服务程序提供的个人用户主页功能可以使系统内所有的用户在自己的家目录中管理个人的网站。

1. 启用 httpd 服务的个人用户主页功能：
```Shell
vim /etc/httpd/conf.d/userdir.conf

//对下面信息进行注释
UserDir disabled

//取消对下面信息的注释，该信息表示网站数据在用户家目录中的保存目录名称
UserDir public_html 
```
2. 在用户家目录中建立用于保存网站数据的目录及首页面文件，并将家目录的权限修改为 755，保证其他人也有权限读取里面的内容。
```Shell
su - linuxprobe

mkdir public_html

chmod -R 755 /home/linuxprobe
```
3. 修改 SELinux 的安全域策略，使 httpd 的个人用户主页功能可以生效：
```Shell
setsebool -P httpd_enable_homedirs=on
```
`-P` 参数表示永久生效。
4. 重新启动 httpd 服务程序，在浏览器的地址栏中输入网址，其格式为“网址/~用户名”
```Shell
exit //退出用户linuxprobe

systemctl restart httpd
```


### 设置网站密码
1. 使用 htpasswd 命令生成密码数据库，`-c` 参数表示第一次生成。再分别添加密码数据库的存放文件，以及验证要用到的用户名称（该用户不必是系统中已有的本地账户）：
```Shell
htpasswd -c /etc/httpd/passwd linuxprobe
```
2. 编辑个人用户主页功能的配置文件：
```Shell
vim /etc/httpd/conf.d/userdir.conf

<Directory "/home/*/public_html">
	AllowOverride all  
	authuserfile "/etc/httpd/passwd" //密码验证文件保存路径
	authname "My privately website"  //当用户访问网站时的提示信息
	authtype basic //验证方式为密码模式
	require user linuxprobe //访问网站时需要验证的用户名称
</Directory>
```


## 虚拟主机功能
利用虚拟主机功能，可以将一台处于运行状态的物理服务器分割成多个“虚拟的服 务器”。

Apache 的虚拟主机功能是服务器基于用户请求的不同 IP 地址、主机域名或端口号，提供多个网站同时为外部提供访问服务的技术。

### 基于 IP 地址
1. 配置多个 IP 地址：
```Shell
vim /etc/sysconfig/network-scripts/ifcfg-ens160:0

TYPE=Ethernet //设备类型
BOOTPROTO=static //地址分配模式
NAME=ens160 //网卡名称
ONBOOT=yes //是否启动
IPADDR=192.168.10.10 //IP地址 
NETMASK=255.255.255.0 //子网掩码
GATEWAY=192.168.10.1 //网关地址
DNS1=192.168.10.1 //DNS地址

vim /etc/sysconfig/network-scripts/ifcfg-ens160:1

TYPE=Ethernet //设备类型
BOOTPROTO=static //地址分配模式
NAME=ens160 //网卡名称
ONBOOT=yes //是否启动
IPADDR=192.168.10.20 //IP地址 
NETMASK=255.255.255.0 //子网掩码
GATEWAY=192.168.10.1 //网关地址
DNS1=192.168.10.1 //DNS地址

vim /etc/sysconfig/network-scripts/ifcfg-ens160:2

TYPE=Ethernet //设备类型
BOOTPROTO=static //地址分配模式
NAME=ens160 //网卡名称
ONBOOT=yes //是否启动
IPADDR=192.168.10.30 //IP地址 
NETMASK=255.255.255.0 //子网掩码
GATEWAY=192.168.10.1 //网关地址
DNS1=192.168.10.1 //DNS地址
```
2. 分别在 /var/www/html 中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。
```Shell
mkdir -p  /var/www/html/10

mkdir -p  /var/www/html/20

mkdir -p  /var/www/html/30
```
3. 从 httpd 服务的配置文件中大约第 132 行处开始，分别追加写入3个基于 IP 地址的虚拟主机网站参数，然后保存并退出。
```Shell
vim /etc/httpd/conf/httpd.conf

//其余类似
<VirtualHost 192.168.10.10>
	DocumentRoot /var/www/html/10
	ServerName www.linuxprobe.com 
	<Directory /var/www/html/10>
	AllowOverride None 
	Require all granted
	</Directory>
</VirtualHost>
```
再重启 httpd 服务：
```Shell
systemctl restart httpd
```

### 基于主机域名
当服务器无法为每个网站都分配一个独立 IP 地址的时候，可以尝试让 Apache 自动识别用户请求的域名，从而根据不同的域名请求来传输不同的内容。

/etc/hosts 是 Linux 系统中用于强制将某个主机域名解析到指定 IP 地址的配置文件。
只要该文件配置正确，即使网络参数中没有 DNS 信息也依然能够将域名解析为某个 IP 地址。

1. 手动定义 IP 地址与域名之间对应关系的配置文件，保存并退出后会立即生效。
```Shell
vim /etc/hosts

192.168.10.10 www.linuxprobe.com www.linuxcool.com www.linuxdown.com
```
2. 分别在 /var/www/html 中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。
```Shell
mkdir -p /var/www/html/linuxprobe 

mkdir -p /var/www/html/linuxcool 

mkdir -p /var/www/html/linuxdown
```
3. 从 httpd 服务的配置文件中大约第 132 行处开始，分别追加写入 3 个基于主机名的虚拟主机网站参数，然后保存并退出：
```Shell

<VirtualHost 192.168.10.10>
	DocumentRoot /var/www/html/linuxprobe
	ServerName www.linuxprobe.com 
	<Directory /var/www/html/linuxprobe>
	AllowOverride None
	Require all granted 
	</Directory>
</VirtualHost>
```
再重启 httpd 服务：
```Shell
systemctl restart httpd
```

### 基于端口号
基于端口号的虚拟主机功能可以让用户通过指定的端口号来访问服务器上的网站资源。 

我们不仅要考虑 httpd 服务程序的配置因素，还需要考虑到 SELinux 服务对新开设端口的监控。
一般来说，使用80、443、8080等端口号来提供网站访问服务是比较合理的，如果使用其他端口号 则会受到 SELinux 服务的限制。

1. 分别在 /var/www/html 中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。
```Shell
mkdir -p /var/www/html/6111 

mkdir -p /var/www/html/6222 

mkdir -p /var/www/html/6333
```
2. 在 httpd 服务配置文件的第46行～48行分别添加用于监听6111、6222和6333端口的参数。
```Shell
vim /etc/httpd/conf/httpd.conf

Listen 6111 
Listen 6222 
Listen 6333
```
3. 从 httpd 服务的配置文件中大约第 132 行处开始，分别追加写入 3 个基于端口号的虚拟主机网站参数，然后保存并退出：
```Shell
vim /etc/httpd/conf/httpd.conf

//其余类似
<VirtualHost 192.168.10.10:6111>
	DocumentRoot /var/www/html/6111
	ServerName www.linuxprobe.com 
	<Directory /var/www/html/6111>
	AllowOverride None
	Require all granted 
	</Directory>
</VirtualHost>
```
再重启 httpd 服务：
```Shell
systemctl restart httpd
```
4. SELinux 允许的与 HTTP 协议相关的端口号中默认没有包含 6111、6222 和 6333， 需要手动添加这3个端口号。该操作会立即生效，在系统重启后依然有效。
```Shell
semanage port -a -t http_port_t -p tcp 6111

semanage port -a -t http_port_t -p tcp 6222

semanage port -a -t http_port_t -p tcp 6333
```


## Apache 的访问控制
Apache 可以基于源主机名、源 IP 地址或源主机上的浏览器特征等信息对网站上的资源进行访问控制，通过 Allow 指令允许某个主机访问服务器上的网站资源，通过 Deny 指令实现禁止访问。在允许或禁止访问网站资源时， Order 指令用来定义 Allow 或 Deny 指令起作用的顺序，其匹配原则是按照顺序进行匹配，若匹配成功则执行后面的默认指令。

例如，只允许 Firefox 浏览器访问：
```Shell
vim /etc/httpd/conf/httpd.conf

<Directory "/var/www/html/server">
	SetEnvIf User-Agent "Firefox" ff=1
	Order allow,deny
	Allow from env=ff
</Directory>

systemctl restart httpd
```
或者，只允许 IP 地址为 192.168.10.20 的主机访问：
```Shell
vim /etc/httpd/conf/httpd.conf

<Directory "/var/www/html/server">
	Order allow,deny
	Allow from 192.168.10.20
</Directory>

systemctl restart httpd
```






