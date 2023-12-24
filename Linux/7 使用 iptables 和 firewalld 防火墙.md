## iptables
### 策略与规则链
防火墙会按照从上到下的顺序来读取配置的策略规则，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。
一般而言，防火墙策略规则的设置有两种：“通”（即放行） 和“堵”（即阻止）。
当防火墙的默认策略为拒绝时，就要设置允许规则；如果防火墙的默认策略为允许，就要设置拒绝规则。

iptables 服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类：
- 在进行路由选择前处理数据包（PREROUTING）
- 处理流入的数据包（INPUT）
- 处理流出的数据包（OUTPUT）
- 处理转发的数据包（FORWARD）
- 在进行路由选择后处理数据包（POSTROUTING）

iptables 服务中的处理动作为：
- ACCEPT（允许流量通过）
- REJECT（拒绝流量通过）
- LOG（记录日志信息）
- DROP（拒绝流量通过）

其中 REJECT 会在拒绝流量之后回复”信息收到但被拒绝”，而 DROP 会直接将流量丢弃不响应。

### 基本命令参数
`iptables` 命令参数：
- `-P` 设置默认策略
- `-F` 清空规则链
- `-L` 查看规则链
- `-A` 在规则链的尾部加入新规则
- `-I num` 在规则链的头部加入新规则
- `-D num` 删除某一条规则
- `-s` 匹配来源地址 IP / MASK，加 “!” 表示除了这个 IP 外
- `-d` 匹配目标地址
- `-i 网卡名称` 匹配从这块网卡流入的数据
- `-o 网卡名称` 匹配从这块网卡流出的数据
- `-p` 匹配协议，如 TCP、UDP、ICMP
- `--dport num` 匹配目标端口号
- `--sport num` 匹配来源端口号

具体操作示例：
1. 查看已有防火墙规则链：
```Shell
iptables -L
```
2. 清空已有防火墙规则链：
```Shell
iptables -F
```
3. 将 INPUT 规则链的默认策略设置为拒绝：
```Shell
iptables -P INPUT DROP
```
规则链的默认策略拒绝动作只能是 DROP，而不能是 REJECT。
4. 向 INPUT 链中添加允许 ICMP 流量进入的策略规则：
```Shell
iptables -I INPUT -p icmp -j ACCEPT
```
向防火墙的 INPUT 规则链中添加一条允许 ICMP 流量进入的策略规则默认允许 ping 命令检测行为。
5. 删除 INPUT 规则链中刚加入的允许 ICMP 流量的策略，并且将默认策略设置为允许：
```Shell
iptables -D INPUT 1

iptables -P INPUT ACCEPT
```
6. 将 INPUT 规则链设置为只允许指定网段的主机访问本机的 22 端口（ ssh 服务），拒绝来自其他所有主机的流量：
```Shell
iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -j REJECT
```
要对某台主机进行匹配，可直接写出它的 IP 地址；如需对网段进行匹配，则需要写为子 网掩码的形式。防火墙策略规则是按照从上到下的顺序匹配的，因此一定要把允许动作放到拒绝动作之前，否则所有的流量将被拒绝，从而导致任何主机都无法访问我们的服务。
7. 向 INPUT 规则链中添加拒绝所有人访问本机 12345 端口的策略规则：
```Shell
iptables -I INPUT -p tcp --dport 12345 -j REJECT

iptables -I INPUT -p udp --dport 12345 -j REJECT
```
8. 向 INPUT 规则链中添加拒绝 192.168.10.5 主机访问本机 80 端口（ Web 服务）的策略规则：
```Shell
iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
```
9. 向 INPUT 规则链中添加拒绝所有主机访问本机 1000 ~1024 端口的策略规则：
```Shell
iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT

iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
```
10. 使用 iptables 命令配置的防火墙规则默认会在系统下一次重启时失效，若想让配置的防火墙策略永久生效，还要执行保存命令：
```Shell
iptables -save
```


## firewalld
firewalld 支持动态更新技术并加入了区域（zone）的概念
区域指的是 firewalld 预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。

firewalld 中常见的区域名称（默认为 public）以及相应的策略规则如下：

|区域|默认规则策略|
|---|---|
|trusted|允许所有的数据包|
|home|拒绝流入的流量，除非与流出的流量相关；而如果流量与 ssh、mdns、ipp-client、 smba-client、dhcpv6-client 服务相关，则允许流量|
|internal|等同于 home 区域|
|work|拒绝流入的流量，除非与流出的流量相关；而如果流量与 ssh、ipp-client 与 dhcpv6-client 服务相关，则允许流量|
|public|拒绝流入的流量，除非与流出的流量相关；而如果流量与 ssh、dhcpv6-client 服务相关， 则允许流量|
|external|拒绝流入的流量，除非与流出的流量相关；而如果流量与 ssh 服务相关，则允许流量|
|dmz|拒绝流入的流量，除非与流出的流量相关；而如果流量与 ssh 服务相关，则允许流量|
|block|拒绝流入的流量，除非与流出的流量相关|
|drop|拒绝流入的流量，除非与流出的流量相关|


### 终端管理工具
`firewall-cmd` 命令参数：
- `--get-default-zone` 查询默认区域名称
- `--set-default-zone=<区域名称>` 设置默认区域，使其永久生效
- `--get-zones` 显示可用区域
- `--get-services` 显示预先定义的服务
- `--get-active-zones` 显示当前正在使用的区域与网卡名称
- `--add-sources=` 将源自此 IP 或子网的流量导向某个区域
- `--remove-source=` 不再将源自此 IP 或子网的流量导向某个区域
- `--add-interface=<网卡名称>` 将源自该网卡的所有流量导向某个指定区域
- `--change-interface=<网卡名称>` 将某个网卡与区域进行关联
- `--list-all` 显示当前区域的网卡配置参数、资源、端口以及服务等信息
- `--list-all-zones` 显示所有区域的网卡配置参数、资源、端口以及服务等信息
- `--add-service=<服务名>` 设置默认区域允许该服务的流量
- `-add-port=<端口号/协议>` 设置默认区域允许该端口的流量
- `--remove-service=<服务名>` 设置默认区域不再允许该服务的流量
- `--remove-port=<端口号/协议>` 设置默认区域不再允许该端口的流量
- `--reload` 使永久生效的配置规则立即生效，并覆盖当前的配置规则
- `--panic-on` 开启应急状况模式
- `--panic-off` 关闭应急状况模式

使用 firewalld 配置的防火墙策略默认为运行时（Runtime）模式，又称为当前生效模式，会随着系统的重启而失效。
若想使配置策略一直存在，需要使用永久（Permanent）模式，即在用 firewall-cmd 命令正常设置防火墙策略时添加 `--permanent` 参数。

具体操作示例：
1. 查看当前 firewalld 服务当前所使用的区域：
```Shell
firewall-cmd --get-default-zone
```
在配置防火墙策略前，必须查看当前生效的是哪个区域，否则配置的防火墙策略将不会立即生效。
2. 查询指定网卡在 firewalld 服务中绑定的区域：
```Shell
firewall-cmd --get-zone-of-interface=ens160
```
在生产环境中，服务器大多不止有一块网卡。一般来说，充当网关的服务器有两块网卡，一块对公网，另外一块对内网。这两块网卡在审查流量时所用的策略肯定也是不一致的，可以根据网卡针对的流量来源，为网卡绑定不同的区域，实现对防火墙策略的灵活管控。
3. 将网卡默认区域修改为 external ，并在重启系统后生效：
```Shell 
firewall-cmd --permanant --zone=external --change-interface=ens160
```
4. 将 firewalld 服务的默认区域设置为 public：
```Shell
firewall-cmd --set-default-zone=public
```
默认区域也叫全局配置，指的是对所有网卡都生效的配置，优先级较低。
5. 启动和关闭 firewalld 防火墙的应急状况模式：
```Shell
firewall-cmd --panic-on

firewall-cmd --panic-off
```
使用 `--panic-on` 参数会立即切断一切网络连接，而使用 `--panic-off` 则会恢复网络连接。
6. 查询 SSH 和 HTTPS 协议的流量是否允许放行：
```Shell
firewall-cmd --zone=public --query-service=ssh

firewall-cmd --zone=public --query-service=https
```
7. 将 HTTPS 协议的流量设置为永久允许放行，并立即生效：
```Shell
firewall-cmd --permanant --zone=public --add-service=https

firewall-cmd --reload
```
8. 将 HTTP 协议的流量设置为永久拒绝，并立即生效：
```Shell
firewall-cmd --permanant --zone=public --remove-service=http

firewall-cmd -reload
```
9. 将访问 8080 和 8081 端口的流量策略设置为允许，但仅限当前生效：
```Shell
firewall-cmd --zone=public --add-port=8080-8081/tcp
```
10. 将原本访问本机 888 端口的流量转发到 22 端口，并且当前和长期均有效：
```Shell
firewall-cmd --permanent --zone=public --add-forward-port= port=888:proto=tcp:toport=22:toaddr=192.168.10.10
```
格式为：
```Shell
firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>
```
11. 富规则设置：
```Shell
firewall-cmd --permanent --zone=public --add-rich-rule= "rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"

firewall-cmd --reload
```
富规则表示更细致、更详细的防火墙策略配置，可以针对系统服务、端口号、源地址和目标地址等诸多信息进行更有针对性的策略配置，优先级在所有的防火墙策略中也是最高的。

SNAT 是一种为了解决 IP 地址匮乏而设计的技术，可以使得多个内网中的用户通 过同一个外网 IP 接入 Internet。
## 服务的访问控制列表
TCP Wrapper 是一款流量监控程序，能够根据来访主机的地址与本机的目标服务程序做出允许或拒绝的操作。
TCP Wrapper 服务的防火墙策略由两个控制列表文件所控制，用户可以编辑允许控制列表文件来放行对服务的请求流量，也可以编辑拒绝控制列表文件来阻止对服务的请求流量。 控制列表文件修改后会立即生效，系统将会先检查允许控制列表文件（/etc/hosts.allow），若匹配到相应的允许策略则放行流量；若没有匹配，则会进一步匹配拒绝控制列表文件 （/etc/hosts.deny），若找到匹配项则拒绝该流量。如果这两个文件都没有匹配到，则默认放行流量。

在配置 TCP Wrapper 服务时需要遵循两个原则：
- 编写拒绝策略规则时，填写的是服务名称，而非协议名称；
- 先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果。

## Cockpit 驾驶舱管理工具
Cockpit 是一个基于 Web 的图形化服务管理工具，具备很好的跨平台性，被广泛应用于服务器、容器、虚拟机等多种管理场景。

在 Cockpit 服务启动后，打开系统自带的浏览器，在地址栏中输入“本机地址:9090”即可访问。由于访问 Cockpit 的流量会使用 HTTPS 进行加密，而证书又是在本地签发的，因此还需要进行添加并信任本地证书的操作。进入 Cockpit 的登录界面后，输 入 root 管理员的账号与系统密码，单击 Log In 按钮后即可进入。

Cockpit 总共分为 13 个功能模块： 
- 系统状态（System）
- 日志信息（Logs）
- 硬盘存储（Storage）
- 网卡网络（Networking）
- 账 户安全（Accounts）
- 服务程序（Services）
- 软件仓库（Applications）
- 报告分析（Diagnostic Reports）
- 内核排错（Kernel Dump）
- SElinux
- 更新软件（Software Updates）
- 订阅服务（Subscriptions）
- 终端界面（Terminal）