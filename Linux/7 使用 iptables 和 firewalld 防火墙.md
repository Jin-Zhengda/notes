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
iptables 参数：
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






