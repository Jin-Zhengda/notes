## Vim文本编辑器

### Vim编辑器的三种模式：
- 命令模式：控制光标移动，可对文本进行复制、粘贴、删除和查找等工作；
- 输入模式：正常的文本录入；
- 末行模式：保存或退出文档，以及设置编辑环境。

### Vim模式切换
命令模式  -> 输入模式   a、i、o等键
输入模式  -> 命令模式   ESC键
命令模式  -> 末行模式   : 键
末行模式  -> 命令模式   ESC键

每次运行Vim编辑器，默认进入命令模式。

### Vim常用命令
命令模式：
- `dd` 删除（剪切）光标所在整行
- `5dd` 删除（剪切）从光标处开始的5行
- `yy` 复制光标所在整行
- `5yy` 复制从光标开始的5行
- `n` 显示搜索命令定位到的下一个字符串
- `N` 显示搜索命令定位到的上一个字符串
- `u` 撤销上一步操作
- `p` 将之前删除或复制过的数据粘贴到光标后面

末行模式：
- `:w` 保存
- `:q` 退出
- `:q!` 强制退出（放弃对文档的修改内容）
- `:wq!` 强制保存退出
- `:set nu` 显示行号
- `:set nonu` 不显示行号
- `:命令` 执行该命令
- `:整数` 跳转到该行
- `:s/a/b` 将当前光标所在行的第一个 a 替换成 b
- `:s/a/b/g` 将当前光标所在行的所有 a 替换成 b
- `:%s/a/b/g` 将全文中所有 a 替换为 b
-  `?字符串` 在文本中从上至下搜索该字符串
- `/字符串` 在文本中从上至下搜索该字符串


## Shell脚本

一个简单的Shell脚本：
```Shell
#! /bin/bash
# For Example
pwd
ls -al
```
第一行为脚本声明，指定执行该脚本的Shell解释器，
第二行为注释，
其余行数为可执行语句和命令。

### 接收用户参数
在Shell脚本中，`$0` 表示当前Shell脚本程序的名称，`$N` 对应第N个参数，`$#` 对应参数个数，
`$*` 对应所有位置的参数，`$?` 对应的是显示上一次命令执行的返回值。
例如：
```Shell
#! /bin/bash
# For Example
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*"
echo "第1个参数为$1，第5个参数为$5"
```

### 判断用户参数
使用测试语句：
```Shell
[ 条件表达式 ]
```
条件成立返回0，否则返回非0.

##### 文件测试运算符
- `-d` 测试文件是否为目录类型
- `-e` 测试文件是否存在
- `-f` 测试是否为一般文字
- `-r` 测试当前用户是否有权限读取
- `-w` 测试当前用户是否有权限写入
- `-x` 测试当前用户是否有权限执行

例如，测试文件是否为目录类型：
```Shell
[ -d /etc/fstab ]
echo $?
```

##### 逻辑测试运算符
`&&` 逻辑与，当前语句执行成功才会执行后面的语句
`||` 逻辑或，当前语句执行失败才会执行后面的语句
`!` 逻辑非，对当前逻辑测试值取反

例如，在root用户下：
```Shell
[ ! $USER = root ] && echo "user" || echo "root"
```
输出 `root`

##### 整数比较运算符
- `-eq` 是否相等
- `-ne` 是否不等于
- `-gt` 是否大于
- `-lt` 是否小于
- `-le` 是否等于或小于
- `-ge` 是否大于或等于
例如，判断可用内存量是否小于1024：
```Shell
FreeMem = `free -m | grep Mem: | awk{print $4}`
[ $FreeMem -lt 1024 ] && echo "Insufficient Memory"
```

##### 字符串比较运算符
- `=` 比较字符串是否相等
- `!=` 比较字符串内容是否相同
- `-z` 判断字符串内容是否为空

例如，判断当前语系环境变量值LANG是否为en.US:
```Shell
[ ! $LANG = 'en.US' ] $$ echo "Not en.US"
```

### 流程控制语句
#### IF分支
```Shell
if 条件测试操作1
	then 
		命令序列1
elif 条件测试操作2
	then 
		命令序列2
else 
	命令序列3
fi
```
例如，验证某台主机是否在线：
```Shell
vim chkhost.sh
#! /bin/bash
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null #指定执行3次，每次间隔0.2秒
if [ $? -eq 0 ]
then
        echo "host$1 is online"
else
        echo "host$1 is offline"
fi

bash chkhost.sh 192.168.10.10
host192.168.10.10 is online

bash chkhost.sh 192.168.10.20
host192.168.10.20 is offline

```
上面的 /dev/null 被称为Linux黑洞，将输出重定向到这个文件等同于删除数据（类似于没有回收功能的垃圾箱）

#### FOR循环
```Shell
for 变量名 in 取值列表
do 
	命令序列
done
```
例如，为每个不存在的用户创建档案：
```Shell
vim addusers.sh
#! /bin/bash
read -p "Enter the Users Password:" PASSWD
for UNAME in `cat users.txt`
do 
	id $UNAME &> /dev/null
	if [ $? -eq 0 ]
	then 
		echo "$UNME , Already exists"
	else
		useradd $UNAME &> /dev/null
		echo "$PASSWD" | passwd --stdin $UNAME &> /dev/null
		echo "$UNAME, Create success"
	fi
done

bash addusers.sh
Enter the Users Password:040210
andy, Create success
barry, Create success
carl, Create success
duke, Create success
```

#### WHILE条件循环语句
```Shell
while 条件测试操作
do 
	命令序列
done
```

#### CASE条件测试语句
```Shell
case 变量值  in
模式1)
	命令序列1
	;;
模式2)
	命令序列2
	;;
	......
*)
	默认命令序列
esac
```


### 计划任务服务程序
##### 一次性计划任务
`at [参数] 时间` 
计划于设定时间执行一次任务，
时间可以指定年月日时分秒，也可以以 `now +数字 时间单位` 的方式在当前时间基础上计算所得。
参数：
- `-f` 指定包含命令的任务文件
- `-q` 指定新任务的名称
- `-l` 显示待执行的任务列表
- `-d` 删除指定的待执行的任务
- `-m` 任务执行后向用户发送邮件

可以使用`atrm 任务序号`  将计划任务删除。

at命令是以交互形式设置执行任务的，可通过管道符重定向来直接设置。

##### 长期性计划任务
`crontab [参数] `
参数：
- `-e` 编辑计划任务
- `-u` 指定用户名称
- `-l`列出任务列表
- `-r` 删除任务计划

使用`crontab -e`设置计划任务会直接打开vim编辑器进行设置，
任务格式为：
`分, 时, 日, 月, 星期 命令`
若有字段未被设置，则需要使用`*`来占位。
可以使用`-`表示一段连续的时间周期，`/`表示执行任务的间隔时间。

`crontab`命令的计划任务参数中，所有命令都要使用绝对路径来写。




