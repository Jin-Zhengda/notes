## Samba 文件共享服务
Samba 服务是 Linux 系统与 Windows 系统之间共享文件的程序。

安装完毕后打开 Samba 服务程序的主配置文件（/etc/samba/smb.conf），
其中第 17～ 22 行代表共享每位登录用户的家目录内容，建议删除。

具体参数：
- `[global]` 全局参数
- `workgroup = SAMBA` 工作组名称 
- `security = user` 安全验证的方式，共有 4 种
- `passdb backend = tdbsam` 定义用户后台的类型，共有 3 种 
- `printing = cups` 打印服务协议 
- `printcap name = cups` 打印服务名称 
- `load printers = yes` 是否加载打印机
- `cups options = raw` 打印机的选项 
- `[homes]` 共享名称 
- `comment = Home Directories` 描述信息
- `valid users = %S, %D%w%S` 可用账户 
- `browseable = No` 指定共享信息是否在“网上邻居”中可见 
- `read only = No` 是否只读 
- `inherit acls = Yes` 是否继承访问控制列表 
- `[printers]` 共享名称 
- `comment = All Printers` 描述信息 
- `path = /var/tmp` 共享路径 
- `printable = Yes` 是否可打印 
- `create mask = 0600` 文件权限 
- `browseable = No` 指定共享信息是否在“网上邻居”中可见 
- `[print$]` 共享名称 
- `comment = Printer Drivers` 描述信息 
- `path = /var/lib/samba/drivers` 共享路径 
- `write list = @printadmin root` 可写入文件的用户列表 
- `force group = @printadmin` 用户组列表 
- `create mask = 0664` 文件权限 
- `directory mask = 0775` 目录权限

在上面的代码中，`security` 参数代表用户登录 Samba 服务时采用的验证方式。
- `share`：代表主机无须验证密码。
- `user`：代表登录 Samba 服务时需要使用账号密码进行验证，通过后才能获取到文件，是默认的验证方式，最为常用。
- `domain`：代表通过域控制器进行身份验证，用来限制用户的来源域。
- `server`：代表使用独立主机验证来访用户提供的密码，相当于集中管理账号。

### 配置 Samba 服务
Samba 服务程序的主配置文件包括全局配置参数和区域配置参数。
全局配置参数用于设置整体的资源共享环境，区域配置参数则用于设置单独的共享资源，且仅对该资源有效。

1. 创建用于访问共享资源的账户信息：
Samba 服务程序的数据库要求账户必须在当前系统中已经存在，否则 日后创建文件时将导致文件的权限属性混乱不堪，由此引发错误。

`pdbedit` 命令用于管理 Samba 服务程序的账户信息数据库，格式为`pdbedit [选项] 账户`。
在第一次将账户信息写入数据库时需要使用 `-a` 参数。
参数：
- `-a` 用户名 建立 Samba 用户 
- `-x` 用户名 删除 Samba 用户
- `-L` 列出用户列表 
- `-Lv` 列出用户详细信息的列表
- `-u` 指定账户

```Shell
pdbedit -a -u linuxprobe
```
2. 创建用于共享资源的文件目录：
在创建时，要考虑到文件读写权限的问题，由于 /home 目录是系统中普通用户的家目录，若启用了 SELinux 服务，还需要考虑应用于该目录的 SELinux 安全上下文所带来的限制。
设置文件权限：
```Shell
mkdir /home/databas

chown -Rf linuxprobe:linuxprobe /home/database
```
设置 SELinux 安全上下文：
```Shell
semanage fcontext -a -t samba_share_t /home/database

restorecon -Rv /home/database
```
3. 若开启了 SELinux 服务，则需要设置 SELinux 服务与策略，使其允许通过 Samba 服务程序访问普通用户家目录。
```Shell
setsebool -P samba_enable_home_dirs on
```
4. 在 Samba 服务程序的主配置文件中，写入：
```Shell
vim /etc/samba/smb.conf

[global] //共享名称为 database
	workgroup = SAMBA 
	security = user 
	passdb backend = tdbsam 
[database] 
	comment = Do not arbitrarily modify the database file //警告用户不要随意修改数据库 
	path = /home/database //共享目录为 /home/database
	public = no //关闭“所有人可见” 
	writable = yes //允许写入操作
```
5. 重启 smb 服务并加入到启动项中，保证在重启服务器后依然能够为用户持续提供服务：
```Shell
systemctl restart smb

systemctl enable smb
```
将 Samba 服务添加到 firewalld 防火墙中：
```Shell
firewall-cmd --zone=public --permanent --add-service=samba

firewall-cmd --reload
```
6. 查看 Samba 服务启用状态：
```Shell
systemctl status smb
```
查看 Samba 服务的共享目录：
```Shell 
smbclient -U linuxprobe -L 192.168.10.10
```

### Linux 挂载共享
Linux 客户端需要安装支持文件共享服务的软件包（cifs-utils）。

在 Linux 客户端创建一个用于挂载 Samba 服务共享资源的目录。
该目录可以与服务器上的共享名称同名，以便于日后查找。
将共享目录挂载：
```Shell
mkdir /database

mount -t cifs -o username=linuxprobe,password=redhat //192.168.10.10/database /database
```
其中 `-t` 参数用于指定协议类型，`-o` 参数用于指定用户名和密码，后面是服务器 IP 地址、共享名称和本地挂载目录 。服务器 IP 地址后面的共享名称指的是配置文件中 `[database]` 的值，而不是服务器本地挂载的目录名称。

按照 Samba 服务的用户名、密码、共享域的顺序将相关信息写入认证文件中：
```Shell
vim auth.smb

username=linuxprobe 
password=redhat 
domain=SAMBA
```
将文件权限修改为仅 root 管理员可以读写：
```Shell
chmod 600 auth.smb
```
将挂载信息写入 /etc/fstab 文件中，以确保共享挂载信息在服务器重启后依然生效：
```Shell
vim /etc/fstab

//192.168.10.10/database database cifs credentials=/root/auth.smb 0 0
```


## 网络文件系统（NFS）
NFS 服务可以将远程 Linux 系统上的文件共享资源挂载到本地主机的目录上，从而使得本地主机 （Linux 客户端）基于 TCP/IP 协议，能够像使用本地主机上的资源那样读写远程 Linux 系统上的共享文件。

1. 在 NFS 服务器上建立用于 NFS 文件共享的目录，并设置足够的权限确保其他人也有写入权限。
```Shell
mkdir /nfsfile

chmod -R 777 /nfsfile
```
2. 配置 NFS 服务的主配置文件：
NFS 服务程序的配置文件为 /etc/exports，默认情况下无任何内容。
需要按照 `共享目录的路径 允许访问的NFS客户端（共享权限参数）` 格式，定义共享目录与相应的权限。
具体参数：
- `ro` 只读 
- `rw` 读写 
- `root_squash` 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的匿名用户 
- `no_root_squash` 当 NFS 客户端以 root 管理员访问时，映射为 NFS 服务器的 root 管理员 
- `all_squash` 无论 NFS 客户端使用什么账户访问，均映射为 NFS 服务器的匿名用户 
- `sync` 同时将数据写入到内存与硬盘中，保证不丢失数据 
- `async` 优先将数据保存到内存，然后再写入硬盘；效率更高，但可能会丢失数据

例如：
```Shell
vim /etc/exports

/nfsfile 192.168.10.*(rw,sync,root_squash)
```
其中`192.168.10.*`通配格式，代表允许来自 192.168.10.0/24 网段的主机访问。
3. 启动和启用 NFS 服务程序：
```Shell
systemctl restart rpcbind 

systemctl enable rpcbind 

systemctl start nfs-server 

systemctl enable nfs-server
```
在使用 NFS 服务进行文件共享之前，需要使用 RPC（远程过程调用）服务将 NFS 服务器的 IP 地址和端口号等信息发送给客户端，故需要一并重启并启用 rpcbind 服务程序。
4. 配置好防火墙，以免默认的防火墙策略禁止正常的 NFS 共享服务：
```Shell
firewall-cmd --permanent --zone=public --add-service=nfs

firewall-cmd --permanent --zone=public --add-service=rpc-bind

firewall-cmd --permanent --zone=public --add-service=mountd

firewall-cmd --reload
```
5. 配置 NFS 服务客户端：
使用 `showmount` 命令查询 NFS 服务器的远程共享信息，该命令输出格式为 `共享的目录名称 允许使用客户端地址`。
参数：
- `-e` 显示 NFS 服务器的共享列表 
- `-a` 显示本机挂载的文件资源的情况 NFS 资源的情况 
- `-v` 显示版本号

```Shell
showmount -e 192.168.10.10
```

接着在 NFS 客户端创建挂载目录。
使用 `mount` 命令并结合 `-t` 参数，指定要挂载的文件系统的类型，并在命令后写上服务器的 IP 地址、服务器上的共享目录以及要挂载到本地系统（即客户端）的目录：
```Shell
mkdir /nfsfile

mount -t nfs 192.168.10.10:/nfsfile /nfsfile
```

若希望 NFS 文件共享服务一直有效，则需要将其写入到 fstab 文件中：
```Shell
vim /etc/fstab

192.168.10.10:/nfsfile /nfsfile nfs defaults 0 0
```

## autofs 自动挂载服务
autofs 服务程序是一 种 Linux 系统守护进程，当检测到用户试图访问一个尚未挂载的文件系统时，将自动挂载该文件系统。
将挂载信息填入/etc/fstab 文件后，系统在每次开机时都自动将其挂载， 而 autofs 服务程序则是在用户需要使用该文件系统时才去动态挂载，从而节约了网络资源和服务器的硬件资源。

在 autofs 服务程序的主配置文件中需要按照`挂载目录 子配置文件`的格式填写需要挂载的文件：
```Shell
vim /etc/auto.master

/media /etc/iso.misc
```
挂载目录是设备挂载位置的上一级目录。对应的子配置文件则是对这个挂载目录内的挂载设备信息作进一步的说明，子配置文件需要用户自行定义， 文件名字没有严格要求，但后缀建议以.misc 结束。
在子配置文件中，应按照 `挂载目录 挂载文件类型及权限 :设备名称` 的格式进行填写：
```Shell
vim /etc/iso.misc

iso -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom
```
上述代码表明要将光盘设备挂载到 /media/iso 目录中，`-fstype` 为文件系统格式参数，`iso9660` 为光盘设备格式，`ro`、`nosuid` 及 `nodev` 为光盘设备具体的权限参数，/dev/cdrom 则是定义要挂载的设备名称。
再将 autofs 服务程序启动并加入到系统启动项中：
```Shell
systemctl start autofs

systemctl enable autofs
```

例如，配置自动挂载 NFS 服务：
```Shell
vim /etc/auto.misc

nfsfile 192.168.10.10:/nfsfile
```
