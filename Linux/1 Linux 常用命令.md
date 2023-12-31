## 系统工作命令
`echo [字符串] [变量]` 在终端输出字符串或者变量的值
`date [+格式]` 显示或设置系统时间与日期
`tiedatectl [参数]` 设置系统时间
`reboot` 重启系统
`poweroff` 关闭系统
`wget [参数] 网址` 在终端命令行中下载网络文件
`ps [参数]` 查看系统中的进程状态
`pstree` 以树状图形式展示进程之间的关系
`top` 动态监视进程获得及系统负载信息
`nice 优先级数字 服务名称` 调整进程的优先级
`pdiof [参数] 服务名称` 查询某个指定服务进程的PID值
`kill [参数] 进程的PID` 终止某个指定PID值的服务进程
`killall [参数] 服务名称` 终止某个指定名称的服务所对应的全部进程

## 系统状态检测命令
`ifconfig [参数] [网络设备]` 获取网卡配置与网络状态等信息
`uname [-a]` 查看系统内核版本与系统架构等信息
`uptime` 查看系统的负载信息
`free [-h]` 显示当前系统中内存的使用量信息
`who` 查看当前登入主机的用户终端信息
`last` 调取主机的被访问记录
`ping [参数] 主机地址` 测试主机之间的网络连通性
`tracepath [参数] 域名` 显示数据包到达目的之际时途中经过的所有路由信息
`netstat [参数]` 显示如网络连接、路由表、接口状态等网络相关信息
`history [-c]` 显示执行过的命令历史 
`sosreport` 收集系统配置及架构信息并输出诊断文档

## 查找定位文件命令
`pwd` 查看当前所处的工作目录
`cd [参数] [目录]` 切换当前的工作路径
`ls [参数] [文件名称]` 显示目录中的文件信息
`tree` 以树状图形式列出目录内容及结构
`find [查找范围] 寻找条件` 按照指定条件来查找文件所对应的位置
`locate 文件名称` 按照名称快速搜索文件对应的位置
`updateddb` 生成文件索引库文件，通常在使用`locate`前使用
`whereis 命令名称` 按照名称快速搜索二进制程序（命令）、源代码以及帮助文件所对应的位置
`which 命令名称` 按照指定名称快速搜索二进制程序程序（命令）所对应的位置

## 文本文件编辑命令
`cat [参数] 文件名称` 查看纯文本文件（内容较少的）
`more [参数] 文件名称` 查看纯文本文件（内容较多的）
`head [参数] 文件名称` 查看纯文本文件的前N行
`tail [参数] 文件名称` 查看纯文本文件的后N行或持续刷新文件的最新内容
`tr [原始字符] [目标字符]` 替换文本内容中的字符
`wc [参数] 文件名称` 统计指定文本文件的行数、字数或者字节数
`stat 文件名称` 查看文件具体存储细节和时间等信息
`grep [参数] 查找键 文件名称` 按行提取文本内容
`cut [参数] 文件名称` 按列提取文本内容 
`diff [参数] 文件名称A 文件名称B` 比较多个文件之间内容的差异
`uniq [参数] 文件名称` 去除文本中连续的重复行
`sort [参数] 文件名称` 对文本内容进行排序

## 文件目录管理命令
`touch [参数] 文件名称` 创建空白文件或设置文件的时间
`mkdir [参数] 目录名称` 创建空白目录
`cp [参数] 源文件名称 目标文件名称` 复制文件或者目录
`mv [参数] 源文件名称 目标文件名称` 剪切或重命名文件
`rm [参数] 文件名称` 删除文件或目录
`dd if=参数值 of=参数值 count=参数值 bs=参数值` 按照指定大小和个数的数据块来复制文件或转换文件
`file 文件名称` 查看文件的类型
`tar 参数 文件名称` 对文件进行打包压缩或解压
