## 文件传输协议
FTP 是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用 20、21号端口，其中端口20用于进行数据传输，端口21用于接受客户端发出的相关FTP 命令与参数。
FTP 服务器是按照 FTP 协议在互联网上提供文件存储和访问服务的主机，FTP 客户端则是向服务器发送连接请求，以建立数据传输链路的主机。

FTP 协议有两种工作模式。 
- FTP 服务器主动向客户端发起连接请求。
- FTP 服务器等待客户端发起连接请求（默认工作模式）。

vsftpd 是一个运行在 Linux 操作系统上的 FTP 服务程序 ，具有很高的安全性、传输速度。

iptables 防火墙管理工具默认禁止了 FTP 协议的端口号，因此在正式配置 vsftpd 服务程序之前，需要清空 iptables 防火墙的默认策略，并将当前已经被清理的防火墙策略状态保存下来：
```Shell
iptables -F

iptables -save
```
然后再将 FTP 协议添加到 firewalld 服务的允许列表中：
```Shell
firewall-cmd --permanent --zone=public --add-service=ftp

firewall-cmd --reload
```

vsftpd 服务程序的主配置文件（/etc/vsftpd/vsftpd.conf）常用参数：

|参数|作用|
|---|---|
|listen=\[YES\|NO] |是否以独立运行的方式监听服务|
|listen_address=IP 地址|设置要监听的 IP 地址|
|listen_port=21|设置 FTP 服务的监听端口|
|download_enable＝\[YES\|NO]|是否允许下载文件|
|userlist_enable=\[YES\|NO] ,userlist_deny=\[YES\|NO]|设置用户列表为允许或禁止操作|
|max_clients=0|最大客户端连接数，0为不限制| 
|max_per_ip=0|同一 IP 地址的最大连接数，0为不限制| 
|anonymous_enable=\[YES\|NO]|是否允许匿名用户访问|
|anon_upload_enable=\[YES\|NO]|是否允许匿名用户上传文件|
|anon_umask=022|匿名用户上传文件的 umask 值| 
|anon_root=/var/ftp|匿名用户的 FTP 根目录| 
|anon_mkdir_write_enable=\[YES\|NO]|是否允许匿名用户创建目录| 
|anon_other_write_enable=\[YES\|NO]|是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限）|
|anon_max_rate=0|匿名用户的最大传输速率（字节/秒），0 为不限制| 
|local_enable=\[YES\|NO]|是否允许本地用户登录|
|FTP local_umask=022|本地用户上传文件的 umask 值| 
|local_root=/var/ftp|本地用户的 FTP 根目录| 
|chroot_local_user=\[YES\|NO]|是否将用户权限禁锢在 FTP 目录，以确保安全| 
|local_max_rate=0|本地用户最大传输速率（字节/秒），0为不限制|
|write_enable=\[YES\|NO]|是否设置可写权限|
|local_umask=022|本地用户模式创建文件的 umask 值|
|userlist_deny=\[YES\|NO]|是否启用禁止用户名单，名单文件为 ftpusers 和 user_list|
|userlist_enable=\[YES\|NO]|是否开启用户作用名单文件功能|


## vsftpd 服务程序
vsftpd 作为更加安全的文件传输协议服务程序，允许用户以 3 种认证模式登录 FTP 服务器。 
- 匿名开放模式：是最不安全的一种认证模式，任何人都可以无须密码验证而直接登录到 FTP 服务器。
- 本地用户模式：是通过 Linux 系统本地的账户密码信息进行认证的模式。
- 虚拟用户模式：需要为 FTP 服务单独建立用户数据库文件， 虚拟用来进行密码验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供 FTP 服务程序进行认证使用。

ftp 是 Linux 系统中以命令行界面的方式来管理 FTP 传输服务的客户端工具。

### 匿名开放模式
一般用来访问不重要的公开文件。
vsftpd 服务程序默认关闭了匿名开放模式。

将 vsftpd 配置文件中相关信息修改为：
```Shell
anonymous_enable=YES
anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```
再重启 vsftpd 服务：
```Shell
systemctl restart vsftpd

systemctl enable vsftpd
```
接着在客户端执行 ftp 命令连接到远程的 FTP 服务器：
```Shell
ftp 192.168.10.10
```
在 vsftpd 服务程序的匿名开放认证模式下，其账户统一为 anonymous，密码为空。在连接 FTP 服务器后，默认访问的是 /var/ftp 目录。

若开启了 SELinux 服务，则需要修改 SELinux 的域策略：
```Shell
setsebool -P ftpd_full_access=on
```

### 本地用户模式
unmask 一般被称为权限掩码或权限补码，能够直接影响到新建文件的权限值。
普通文件的默认权限是 666，目录的默认权限是 777，默认权限 − umask＝实际权限。

将 vsftpd 配置文件中相关信息修改为：
```Shell
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
```
再重启 vsftpd 服务即可。

vsftpd 服务程序所在的目录中默认存放着两个名为用户名单的文件（ftpusers 和 user_list），用户名存在于这两个文件中任意一个中时，该用户便无法登录到 FTP 服务器上。
vsftpd 服务程序为了保证服务器的安全性而默认禁止了 root 管理员和大多数系统用户的登录行为。

若将主配置文件中 userlist_deny 的参数值改成 NO，那么 user_list 列表就变成了强制白名单，功能与之前完全相反，只允许列表内的用户访问，拒绝其他人的访问。

在采用本地用户模式登录 FTP 服务器后，默认访问的是该用户的家目录，而且该目录的默认所有者、所属组都是该用户自己，因此不存在写入权限不足的情况。
但是，若开启了 SELinux 服务，则需要修改 SELinux 的域策略：
```Shell
setsebool -P ftpd_full_access=on
```

### 虚拟用户模式
虚拟用户模式专门创建出一个账号来登录 FTP 传输服务，而且该账号不能用于以 SSH 方式登录服务器。

1. 创建用于进行 FTP 认证的用户数据库文件，其中奇数行 为账户名，偶数行为密码。例如，分别创建 zhangsan 和 lisi 两个用户，密码均为 redhat：
```Shell
cd /etc/vsftpd/

vim vuser.list

zhangsan redhat lisi redhat
```
明文信息既不安全，也不符合使 vsftpd 服务程序直接加载的格式，需要使用 db_load 命令用 hash 算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限以避免其他人看到数据库文件的内容，然后再将原始的明文信息文件删除：
```Shell
db_load -T -t hash -f vuser.list vuser.db

chmod 600 vuser.db

rm -f vuser.list
```
2. 创建 vsftpd 服务程序用于存储文件的根目录以及用于虚拟用户映射的系统本地用户：
```Shell
useradd -d /var/ftproot -s /sbin/nologin virtual

ls -ld /var/ftproot/

chmod -Rf 755 /var/ftproot/
```
使虚拟用户默认登录到与之有映射关系的系统本地用户的家目录中，虚拟用户创建的文件的属性也都归属于该系统本地用户，从而避免 Linux 系统无法处理虚拟用户所创建文件的属性权限。
为了方便管理 FTP 服务器上的数据，可以将该系统本地用户的家目录设置为 /var 目录（该目录用来存放经常发生改变的数据）。并且为了安全起见，将该系统本地用户设置为不允许登录 FTP 服务器。
3. 建立用于支持虚拟用户的 PAM 文件：
```Shell
vim /etc/pam.d/vsftpd.vu

auth    required   pam_userdb.so db=/etc/vsftpd/vuser 
account required   pam_userdb.so db=/etc/vsftpd/vuser
```
PAM（可插拔认证模块）是一种认证机制，通过一些动态链接库和统一的 API 将系统提供的服务与认证方式分开，使系统管理员可以根据需求灵活调整服务程序的不同认证方式。
上述命令新建了一个用于虚拟用户认证的 PAM 文件 vsftpd.vu，其中 PAM 文件内的 `db=` 参数为使用 `db_load` 命令生成的账户密码数据库文件的路径。
4. 在 vsftpd 服务程序的主配置文件中通过 `pam_service_name` 参数将 PAM 认证文件的名称修改为 vsftpd.vu：
```Shell
vim /etc/vsftpd/vsftpd.conf

anonymous_enable=NO
local_enable=YES
write_enable=YES
guest_enable=YES
guest_username=virtual
allow_writeable_chroot=YES
pam_service_name=vsftpd.vu
```
PAM 作为应用程序层与鉴别模块层的连接纽带，可以让应用程序根据需求灵活地在自身插入所需的鉴别功能模块。当应用程序需要 PAM 认证时，则需要在应用程序中定义负责认证的 PAM 配置文件，实现所需的认证功能。
5. 为虚拟用户设置不同的权限：
```Shell
mkdir /etc/vsftpd/vusers_dir/

cd /etc/vsftpd/vusers_dir/

vim zhangsan

anon_upload_enable=YES 
anon_mkdir_write_enable=YES 
anon_other_write_enable=YES
```
然后再次修改 vsftpd 主配置文件，通过添加 `user_config_dir` 参数来定义这两个虚拟用户 不同权限的配置文件所存放的路径。需要重启 vsftpd 服务程序并将该服务添加到开机启动项中使修改后的参数立即生效：
```Shell
vim /etc/vsftpd/vsftpd.conf

user_config_dir=/etc/vsftpd/vusers_dir

systemctl restart vsftpd

systemctl enable vsftpd
```
6. 若开启了 SELinux 服务，则还需要修改 SELinux 域策略：
```Shell
setsebool -P ftpd_full_access=on
```


## 简单文件传输协议（TFTP）
简单文件传输协议（TFTP）是一个基于 UDP 协议在客户端和服务器之间进行简单文件传输的协议。

TFTP 的命令功能不如 FTP 服务强大，甚至不能遍历目录，在安全性方面也弱于 FTP 服务。
TFTP 在传输文件时采用的是 UDP 协议，占用的端口号为 69，文件的传输过程也不像 FTP 协议那样可靠。
TFTP 不需要客户端的权限认证，减少了无谓的系统和网络带宽消耗，在传输琐碎的文件时效率更高。

tfpt-server 是 TFPT 的服务程序，tftp 是 TFTP 服务用于连接测试客户端的工具，xinetd 是 TFPT 服务的管理服务。
在 Linux 系统中，TFTP 服务是使用 xinetd 服务程序来管理的。xinetd 服务可以用来管理多种轻量级的网络服务，而且具有强大的日志功能。


需要编辑 xinetd 服务的主配置文件来启用 TFTP 服务：
```Shell
vim /etc/xinetd.d/tftp

service tftp 
{ 
	socket_type = dgram 
	protocol = udp 
	wait = yes 
	user = root 
	server = /usr/sbin/in.tftpd 
	server_args = -s /var/lib/tftpboot 
	disable = no 
	per_source = 11 
	cps = 100 2 
	flags = IPv4 
}
```
再重启 xinetd 服务并将其添加到系统开机启动项之中：
```Shell
systemctl restart tftp

systemctl enable tftp

systemctl restart xinetd

systemctl enable xinetd
```
编辑防火墙配置，确保 TFTP 使用的 69 端口被允许：
```Shell
firewall-cmd --zone=public --permanent --add-port=69/udp

firewall-cmd --reload
```


TFTP 的根目录为 /var/lib/tftpboot，可以使用 `tftp` 命令尝试访问其中的文件。
参数：
- `?` 帮助信息
- `put` 上传文件
- `get` 下载文件
- `verbose` 显示详细的处理信息
- `status` 显示当前状态信息
- `binary` 使用二进制进行传输
- `ascii` 使用 ASCII 码进行传输
- `timeout` 设置重传的超时时间
- `quit` 退出

例如：
```Shell
tftp 192.168.10.10

tftp> get readme.txt 
tftp> quit
```





