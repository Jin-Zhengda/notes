## 配置网络服务
### 配置网卡参数
编辑配置文件：
```Shell
vim /etc/sysconfig/network-scripts/ifcfg-ens160
```
向配置文件中写入：
```Shell
TYPE=Ethernet //设备类型
BOOTPROTO=static //地址分配模式
NAME=ens160 //网卡名称
ONBOOT=yes //是否启动
IPADDR=192.168.10.10 //IP地址 
NETMASK=255.255.255.0 //子网掩码
GATEWAY=192.168.10.1 //网关地址
DNS1=192.168.10.1 //DNS地址
```
最后执行重启网卡设备命令：
```Shell
nmcli connection reload ens160

nmcli connection up ens160
```
测试网络是否能连通：
```Shell
ping 192.168.10.10
```

也可以使用 `nmtui` 命令运行网络配置工具（具有 GUI）来配置网卡参数。

### 创建网络会话
NetworkManager 是一种动态管理网络配置的守护进程，能够让网络设备保持连接状态。可以使用 `nmcli` 命令来管理NetworkManager 服务程序。
网络会话功能允许用户在多个网络配置文件中快速切换。

例如，查看网络信息或网络状态：
```Shell
nmcli connection show
```

使用下列格式来设置网络会话：
```Shell
nmcli connection add con-name 会话名称 ifname 网卡名称 [autoconnect no] type 连接类型 [ip4 IP地址 gw4 网关地址]
```
例如，设置公司网络手动指定 IP 地址（`autoconnect no` 参数将网络会话设置为默认不被自动激活， `ip4` 及 `gw4` 参数手动指定网络的 IP 地址）：
```Shell
nmcli connection add con-name company ifname ens160 autoconnect no type ethernet ip4 192.168.10.10/24 gw4 192.168.10.1
```
设置家庭网络使用 DHCP 自动分配 IP 地址：
```Shell
nmcli connection add con-name house type ethernet ifname ens160
```

使用 `nmcli` 命令配置过的网络会话是永久生效的，需要使用相应的网络会话，激活即可：
```Shell
nmcli connection up company
```
不需要网络会话时，删除即可：
```Shell
nmcli connection delete company
```

### 绑定双网卡
若对两块网卡实施了绑定，在正常工作中它们会共同传输数据，使网络传输的速度变得更快。
即使有一块网卡突然出现故障，另外一块网卡也会立即自动顶替上去，保证数据传输不会中断。

绑定步骤：
1. 创建出绑定网卡：
```Shell
nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=balance-rr"
```
即创建一个类型为 bond（绑定）、名称为 bond0、网卡名为 bond0 的绑定设备，绑定模式为 balance-rr。
balance-rr 网卡绑定模式全称为平衡轮循模式，会根据设备顺序依次传输数据包，提供负载均衡的效果，让带宽的性能更好；一旦某个网卡发生故障，会马上切换到另外一台网卡设备上，保证网络传输不被中断。
active-backup 网卡绑定模式称为主备模式，平时只有一块网卡正常工作，另一个网卡随时待命，一旦工作中的网卡发生损坏，待命的网卡会自动顶替上去，具有较强的冗余能力。
2. 向绑定网卡设备中添加丛属网卡：
```Shell
nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname ens160 master bond0

nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname ens192 master bond0
```
即将 ens160 和 ens192 两块网卡添加至 bond0 绑定网卡中。
3. 配置绑定网卡设备的网络信息：
```Shell
nmcli connection modify bond0 ipv4.addresses 192.168.10.10/24

nmcli connection modify bond0 ipv4.gateway 192.168.10.1

nmcli connection modify bond0 ipv4.dns 192.168.10.1

nmcli connection modify bond0 ipv4.dns-search linuxprobe.com

nmcli connection modify bond0 ipv4.method manual
```
依次配置了网络的 IP 地址及子网掩码、网关、DNS、搜索域、手动配置参数。也可直接编辑网卡配置文件来实现相同效果。
4. 启动绑定网卡设备并查看设备状态：
```Shell
nmcli connection up bond0

nmcli device status
```


## 远程控制服务
SSH（Secure Shell）是一种能够以安全的方式提供远程登录的协议，也是目前远程管理 Linux 系统的首选方式。
要使用 SSH 协议来远程管理 Linux 系统，需要配置部署 sshd 服务程序。

sshd 是基于SSH 协议开发的一款远程管理服务程序，提供两种安全验证方式：
- 基于密码的验证：使用账号和密码来验证登录
- 基于密钥的验证：在本地生成密钥对，再将密钥对中的公钥上传至服务器， 并与服务器中的公钥进行比较

sshd 服务的配置信息保存在 /etc/ssh/sshd_config 文件中
```Shell
Port 22 //默认的 sshd 服务端口 
ListenAddress 0.0.0.0 //设定sshd服务器监听的IP地址 
Protocol 2 SSH //协议的版本号 
HostKey /tc/ssh/ssh_host_key SSH //协议版本为1时，DES私钥存放的位置 
HostKey /etc/ssh/ssh_host_rsa_key SSH 协议版本为2时，RSA私钥存放的位置 
HostKey /etc/ssh/ssh_host_dsa_key SSH 协议版本为2时，DSA私钥存放的位置 
PermitRootLogin yes //设定是否允许root管理员直接登录 
StrictModes yes //当远程用户的私钥改变时直接拒绝连接 
MaxAuthTries 6 //最大密码尝试次数 
MaxSessions 10 //最大终端数 
PasswordAuthentication yes //是否允许密码验证 
PermitEmptyPasswords no //是否允许空密码登录（不安全）
```

在客户端使用 ssh 命令远程连接服务器，其格式为`ssh [参数] 主机IP地址`，需要退出时执行 `exit` 命令。
### 安全密钥验证
配置 sshd 服务密钥验证方式：
假设服务器主机地址为：192.168.10.10；客户端主机地址为：192.168.10.20。
1. 在客户端主机中生成密钥对：
```Shell
ssh-keygen
```
2. 将客户端主机中生成的公钥文件传送至远程服务器：
```Shell
ssh-copy-id 192.168.10.10
```
3. 对服务器进行设置，使其仅允许密钥验证，拒绝密码验证方式：
```Shell
vim /etc/ssh/sshd_config
```
修改为下列信息：
```Shell
PasswordAuthentication no
```
修改保存后重启 sshd 服务程序：
```Shell
systemctl restart sshd
```
4. 此时客户端尝试登录服务器无需输入密码：
```Shell
ssh 192.168.10.10
```

### 远程传输命令
scp（secure copy）是基于 SSH 协议在网络之间进行安全传输的命令。
输送文件：`scp [参数] 本地文件 远程账户@远程IP地址:远程目录`
拉取文件：`scp [参数] 远程用户@远程IP地址:远程文件 本地目录`

`cp` 命令只能在本地硬盘中进行文件复制。
`scp` 不仅能够通过网络传送数据，而且所有的数据都将进行加密处理。

参数：
- `-v` 显示详细的连接速度
- `-P` 指定远程主机的 sshd 端口号
- `-r` 用于传送文件夹
- `-6` 使用 IPV6 协议

输送：
```Shell
scp /root/readme.txt 192.168.10.10:/home
```
拉取：
```Shell
scp 192.168.10.10:/etc/redhat-release /root
```


## 不间断会话服务
Terminal Multiplexer（终端复用器，Tmux）是一款能够实现多窗口远程控制的开源服务程序，是为了解决网络异常中断或为了同时控制多个远程终端窗口而设计的程序。
- 会话恢复：即便网络中断，也可让会话随时恢复，确保用户不会失去对远程会话的控制。
- 多窗口：每个会话都是独立运行的，拥有各自独立的输入输出终端窗口，终端窗口内显示过的信息也将被分开隔离保存，以便下次使用时依然能看到之前的操作记录。
- 会话共享：当多个用户同时登录到远程服务器时，便可以使用会话共享功能让用户之间的输入输出信息共享。

### 管理远程会话
使用 `tmux` 命令进入 Tmux 会话窗口：
```Shell
tmux
```
会话窗口的底部出现了一个绿色的状态栏，显示的是会话编号、名称、主机名及系统时间。使用 `exit` 可以退出 Tmux 会话窗口，回到终端界面。

会话窗口的编号是从 0 开始自动排序（即 0、1、2、3、……），创建一个指定名称为 backup 的窗口：
```Shell
tmux new -s backup
```
可以用 `detach` 参数将会话隐藏到后台，此时会话窗口中执行的进程不会被中断：
```Shell
tmux detach
```
可以查看后台的会话状态：
```Shell
tmux ls
```
恢复指定的会话窗口：
```Shell
tmux attach -t backup
```
不需要该会话时，可以直接使用 `kill` 命令关闭。

在日常的生产环境中，也不是必须先创建会话，然后再开始工作。可以直接使用 `tmux` 命令执行要运行的指令，这样命令中的一切操作都会被记录下来，当命令执行结束后，后台 会话也会自动结束。

### 管理多窗格
|命令|作用|
|---|---|
|tmux split-window|创建上下切割的多窗格终端界面|
|tmux split-window -h|创建左右切割的多窗格终端界面|
|tmux select-pane -U|切换至上方的窗格|
|tmux select-pane -D|切换至下方的窗格|
|tmux select-pane -L|切换至左方的窗格|
|tmux select-pane -R|切换至右方的窗格|
|tmux swap-pane -U|将当前窗格与上方的窗格互换|
|tmux swap-pane -D|将当前窗格与下方的窗格互换|
|tmux swap-pane -L|将当前窗格与左方的窗格互换|
|tmux swap-pane -R|将当前窗格与右方的窗格互换|

快捷键：
- Ctrl + B + 方向键：调整窗格的尺寸
- Ctrl + B 组合键进入快捷键模式，再按下：
	- % ：划分为左右两个窗格
	- "： 划分为上下两个窗格
	- <方向键>： 切换到上下左右相邻的一个窗格
	- ; ：切换至上一个窗格
	- o：切换至下一个窗格
	- {：将当前窗格与上一个窗格位置互换
	- }：将当前窗格与下一个窗格位置互换
	- x：关闭窗格
	- !：将当前窗格拆分成独立窗口，而不在与其他窗格同处一个界面
	- q：显示窗格编号


### 会话共享功能
当多个用户同时控制服务器的时候，Tmux 可以把服务器屏幕内容共享出来，每个用户都能够看到相同的内容，一起同时操作。

要实现会话共享功能，首先使用 ssh 服务将客户端 A 远程连接到服务器，随后使用 Tmux 服务创建一个新的会话窗口，名称为 share：
```Shell
ssh 192.168.10.10

tmux new -s share
```
然后，使用 ssh 服务将客户端 B 也远程连接到服务器，并执行获取远程会话的命令即可：
```Shell
ssh 192.168.10.10

tmux attach-session -t share
```


## 检索日志信息
rsyslog 是默认的日志服务程序。
常见日志文件保存路径：

|文件路径|作用|
|---|---|
|/var/log/boot.log|系统开机自检事件及引导过程等信息|
|/var/log/lastlog|用户登录成功时间、终端名称及 IP 地址等信息|
|/var/log/btmp|记录登录失败的时间、终端名称及 IP 地址等信息|
|/var/log/messages|系统及各个服务的运行和报错信息|
|/var/log/secure|系统安全相关的信息|
|/var/log/wtmp|系统启动与关机等相关信息|

日志分为三类：
- 系统日志：主要记录系统运行情况和内核信息。
- 用户日志：主要记录用户的访问信息，包含用户名、终端名称、登入及退出时间、来源 IP 地址和执行过的操作等。
- 程序日志：多数服务一般都会保存一份与其同名的日志文件，记录着服务运行过程中各种事件的信息；每个服务程序都有自己独立的日志文件，且格式相差较大。

`journalctl` 命令用于检索和管理系统日志信息，格式为 `journalctl 参数`。
参数：
- `-k` 内核日志
- `-b` 启动日志
- `-u` 指定服务
- `-n` 指定条数
- `-p` 指定类型
- `-f` 实时刷新（追踪日志）
- `--since --until` 指定时间
- `--disk-usage` 占用空间

在 rsyslog 服务程序中，日志根据重要程度被分为9个等级：

|日志等级|说明|
|---|---|
|emerg|系统出现严重故障，比如内核崩溃|
|alert|应立即修复的故障，比如数据库损坏|
|crit|危险性较高的故障，比如硬盘损坏导致程序运行失败|
|err|危险性一般的故障，比如某个服务启动或运行失败|
|warning|警告信息，比如某个服务参数或功能出错|
|notice|不严重的一般故障，只是需要抽空处理的情况|
|info|通用性消息，用于提示一些有用的信息|
|debug|调试程序所产生的信息|
|none|没有优先级，不进行日志记录|


