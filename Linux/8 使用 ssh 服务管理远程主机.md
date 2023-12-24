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