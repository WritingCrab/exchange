## shell

---

这里将不光写shell脚本的笔记，shell的一些命令也会写在这里

有一个很需要注意的就是win下面的回车是CR LF，而在linux下是LF

```bash
var="hahah" # 定义变量，等号两边都不能有空格
for file in `ls /etc`; do # do要么放在新行，要么前面加分号，注意那个不是单引号
	echo $file
done
for file in $(ls /etc); do
	echo $file
done
url='ftp:x.x.x.x:xx'
readonly url # 只读变量
unset url # 删除变量
```

- **局部变量**：脚本内部定义的，仅在当前shell实例中有效。
- **环境变量**：所有程序都能调用，shell脚本也可以定义环境变量。
- **shell变量**：shell程序设置的特殊变量，一部分是环境变量，一部分是局部变量。

### 字符串

```bash
str='this is a string' # 原样输出，不能放变量名，不能放单个单引号，成对可以，作为拼接
str2="haha, ${str},\n" # 可以转义，可以放变量名
str3="str2"$str2"and"$str"ha" # 拼接还有点意义不太清晰
len=${#str} # 字符串长度，要用花括号包围
str4=${str3:20:30} # 截取子字符串
echo `expr index "$string" io` # 查找字符i或者o的位置
```

### 数组

```bash
arr=(val0 val1 val2 val3 val4) # 括号括起来，成员用空格分割
arr[0]=val5 # 单独定义某个成员，注意键值是对应的
${arr[5]}
${arr[@]} # 调用全部成员
${#arr[@]} # 数组长度
${#arr[*]} # 单个元素的长度，存疑，还有点问题
```

### 注释

```bash
# 单行注释
:<<EOF
aksjdhka
sajdhbja
sajkdh
EOF
#↑听说这个是多行注释，先不管
```

### shell传参

```bash
$0 # 执行的文件名称
$1 # 传递的第一个参数
# ...
$# # 传到脚本的参数个数
$* # 以单个字符串显示所有参数，每个元素是一个字符串
"$*" # 与上面一样，但是所有参数是一个字符串
$$ # 脚本的当前进程id
$! # 后台运行的最后一个进程id
$@ # 与$*相同，并在引号中返回每个参数，跟$*一样啊
"$@" # 甚至看不出跟不带引号的有啥区别，有机会详细了解
$- # 显示shell使用的当前选项
$? # 显示最后退出命令的状态，0为没有错误，其他值有错误
```

### shell基本运算符

原生bash不支持数学运算，但可以通过其他命令实现，`awk`，`expr`

```bash
val=`expr 2 + 2` #里面也可以是变量，运算符两边要有空格，被反引号包含
```

**算数运算符**

| 运算符 | 说明 | 举例           |
| ------ | ---- | -------------- |
| `+`    | -    | `\expr 2 + 2\` |
| `-`    | -    | `expr 2 - 2`   |
| `*`    | -    | `expr 2 * 2`   |
| `/`    | -    | `expr 2 / 2`   |
| `%`    | -    | `expr 2 % 2`   |
| `=`    | -    | `a=\$b`        |
| `==`   | -    | `[ $a == $b ]` |
| `!=`   | -    | `[ $a != $b ]` |

条件表达式方括号内侧也要有空格

**关系运算符**

| 运算符 | 说明             | 举例          |
| ------ | ---------------- | ------------- |
| `-eq`  | equal            | `[$a -eq $b]` |
| `-ne`  | not equal        | `[$a -ne $b]` |
| `-gt`  | greater than     | `[$a -gt $b]` |
| `-lt`  | less than        | `[$a -lt $b]` |
| `-ge`  | greater or equal | `[$a -ge $b]` |
| `-le`  | less or equal    | `[$a -le $b]` |

**布尔运算符**

| 运算符 | 说明 | 举例           |
| ------ | ---- | -------------- |
| `!`    | not  | `[ !$a ]`      |
| `-o`   | or   | `[ $a -o $b ]` |
| `-a`   | and  | `[ $a -a $b ]` |

**逻辑运算符**

| 运算符 | 说明    | 举例 |
| ------ | ------- | ---- |
| `&&`   | 逻辑AND |      |
| `||`   | 逻辑OR  |      |

**字符串运算符**

| 运算符 | 说明             | 举例        |
| ------ | ---------------- | ----------- |
| `=`    | 是否相等？       | -           |
| `!=`   | 是否不相等？     | -           |
| `-z`   | 长度是否为0？    | `[ -z $a ]` |
| `-n`   | 长度是否不为0？  | `[ -n $a]`  |
| `str`  | 字符串是否为空？ | `[ $a ]`    |

**文件测试运算符**

| 运算符 | 说明 | 举例 |
| ------ | ---- | ---- |
| `-b file`     | 是块设备文件？ | `[ -b $file ]`    |
| `-c file`     | 是字符设备文件？ | `[ -c $file ]`    |
| `-d file`     | 是目录？ | `[ -d $file ]`    |
| `-f file`     | 是普通文件？（不是目录不是设备文件） | `[ -f $file ]`    |
| `-g file`     | 设置了SGID位？（*是什么*） | `[ -g $file ]`    |
| `-k file`     | 文件设置了黏着位（Sticky bit）？ | `[ -k $file ]`    |
| `-p file`     | 是有名管道？（emmm） | `[ -p $file ]`    |
| `-u file`     | 设置了SUID位？（*是什么*） | `[ -u $file ]`    |
| `-r file`     | 可读？ | `[ -r $file ]`    |
| `-w file`     | 可写？ | `[ -w $file ]`    |
| `-x file`     | 可执行？ | `[ -x $file ]`    |
| `-s file`     | 不为空？（文件大小大于0） | `[ -s $file ]`    |
| `-e file`     | 存在？（包括目录） | `[ -e $file ]`    |

### echo

*先不看*

### printf

*先不看*

### test

*先不看*

### Shell流程控制

**`if`**

```bash
if [condition]
then
	[cmd1]
	[cmd2]
	[...]
fi
```

也可以

```bash
if [condition]; then
	[cmd1]
	[cmd2]
	[...]
fi
```



**`if-else`**

```bash
if [condition]
then
	[cmd1]
	[...]
else
	[cmd2]
	[...]
fi
```

**`if-elif-else`**

```bash
if [condition1]
then
	[cmd1]
	[...]
elif [condition2]
	[cmd2]
	[...]
else
	[cmd3]
	[...]
fi
```

**`for`**

```bash
for var in item1 item2 item3 ...
do
	[cmd1]
	[...]
done
```

或者

```bash
for var in item1 item2 item3 ...; do [cmd1]; [...]; done
```

甚至可以

```bash
for((i=1;i<5;i++)); do
	[something]
done
```



多条件

```shell
if [[ condition1 && condition2 ]]; then
	something
fi
```

有多种写法，可以自行搜索



**`while`**

```bash
while [condition]
do
	[cmd1]
	[...]
done
```

*bash let命令，值得了解一下*

```bash
while :
do
	[some]
	[thing]
	[exciting]
	[!]
done
```

**`until`**

```bash
until [condition]
do
	[c1]
	[..]
done
```

就跟`while`几乎相反，先不看

**`case`**

```bash
case [var] in
	[situation1])
		[cmd1]
		[...]
		;;
	[situation1])
		[cmd2]
		[...]
		;;
	*) #default
		[cmd3]
		[...]
		;;
esac
		
```

### 跳出循环

**`break`**

终止执行后面的所有循环

```bash
while :
do
	echo -n "input a num in [1, 5]: "
	read aNum
	case $aNum in
		1|2|3|4|5) echo "your num is $aNum"
		;;
		*) echo "follow your heart"
			break
		;;
	esac
done
```

**`continue`**

退出当前循环

*嵌套情形会怎么处理？*

### shell单例

单例可以靠`trap`实现

```bash
trap ["cmd"] [signal(s)] # 接收到信号后执行cmd
```

`signal`可以使用`trap -l`查看，常用的信号有

```
1. SIGHUP 挂起或父进程被杀死
2. SIGINT 来自键盘的中断（Ctrl+C）
3. SIGQUIT 从键盘退出（跟上面有区别么，会产生core文件，类似一个错误信号）
8. SIGFPE 发生致命运算错误
9. SIGKILL 立即结束程序的运行，不能被阻塞、处理、忽略
15. SIGTERM 程序结束，可以被阻塞和处理，shell命令的kill缺省产生这个信号
```

如果想要单例运行的话，那么

```bash
#!/bin/sh

LOCKFILE="/tmp/lock.tmp"

trap "echo 'sh rm lockfile'; rm $LOCKFILE; exit" 1 2 3 9 15

if [ -f $LOCKFILE ]; then
    echo "Instance is running"
    exit 0
else
    touch $LOCKFILE
    chmod 600 $LOCKFILE
    echo "sh start!"
    sleep 10
    echo "finish"
fi

rm $LOCKFILE
```

也就是遇到上述信号的时候或者程序正常退出的时候会删除临时文件，否则临时文件存在的时候是不允许再创建临时文件的。

### `pgrep`查询进程id

```bash
pgrep "[process_name]"
```

如果一个名字有不止一个进程，那么命令会列出所有叫这个名字的进程的pid

`kill`杀掉进程

```bash
kill [pid]
```

`kill`只能杀pid，所以首先得知道pid才能精确杀，这块可以这样

```bash
pgrep "[process_name]" | xargs sudo kill
```

这里用的是管道，也就是`|`，另外`xargs`是真的很强大，主要的作用是将标准输入（stdin）转化为命令行参数

### `ln`链接

`ln`可以创建两种链接，一种是软链接`-s`，一种是硬链接，软链接在目标处不生成文件，不会多占用存储空间，硬链接则会生成等大的文件，无论是哪一种，两者都是同步变化的

```bash
ln -s [source] [destination]
```

### `mount`和`umount`

```
mount -t nfs -o nolock [要挂载的] [到本地哪里]
```

`-t`执行设备的文件类型，`nfs`即网络文件系统

### `sleep`

```bash
sleep 1
sleep 1s # 未验证
sleep 1m # 未验证
sleep 1h # 未验证
```

### 反引号

```bash
$a=`echo "world"`
```

将后面命令的运行结果送给`$a`

### *

```
ps -ef | grep [process name] | grep -v grep | wc -l
```

数所有叫`[process name]`的进程的数目

### shell函数

```bash
[ function ] dunname [()]
{
    action;
    [return int;]
}
```

- 可以带`function func()`定义，也可以直接`func()`定义，不带参数
- 可以加`return`返回，如果不加则以最后一条命令的运行结果作为返回值，后面是数字(0~255)

参数与传递到脚本的参数的处理方式是一样的

---

### grep命令

用于查找文件里符合条件的字符串。

如果发现某文件的内容符合所指定的范本样式，grep指令会将含有范本样式的一列（？）显示出来。若不指定任何文件名称，或是给出的文件名称为`-`，grep指令会从标准输入设备读取数据。

```bash
grep [-abcEFGhHilLnqrsvVwxy][-A<显示列数>][-B<显示列数>][-C<显示列数>][-d<进行动作>][-e<范本样式>][-f<范本文件>][--help] [范本样式][文件或目录]
```

- **`-a`或`--text`**：不要忽略二进制数据
- **`-A<显示行数>`或`--after-context=<显示行数>`**：除了显示符合范本样式的那一行之外，显示该行之后的内容
- **`-b`或`--byte-offset`**：在显示符合样式的那一行之前，标示出该行第一个字符的编号
- **`-B<显示行数>`或`--before-context=<显示行数>`**：除了显示符合范本样式的那一行之外，显示该行之前的内容
- **`-c`或`--count`**：计算符合样式的行数
- **`-C<显示行数>`或`--context=<显示行数>`或`-显示行数`**：除了显示符合样式的那一行之外，显示该行前后的内容
- **`-d<动作>`或`--directories=<动作>`**：当指定要查找的是目录而非文件时，必须使用这项参数，否则将会报信息并停止动作
- **`-e<范本样式>`或`--regexp=<范本样式>`**：指定字符串作为查找文件内容的样式
- **`-E`或`--extended-regexp`**：将样式视为延伸的普通表示法来使用
- **`-f<规则文件>`或`--file=<规则文件>`**：指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式
- **`-F`或`--fixed-regexp`**：将样式视为固定字符串的列表
- **`-G`或`--basic-regexp`**：将样式视为普通的表示法来使用
- **`-h`或`--no-filename`**：在显示符合样式的那一行之前，不标示该行所属的文件名称
- **`-H`或`--with-filename`**：在显示符合样式的那一行之前，标示该行所述的文件名称
- **`-i`或`--ignore-case`**：忽略自负大小写的差别
- **`-l`或`--file-with-matches`**：列出文件内容符合指定的样式的文件名称
- **`-L`或`--files-without-match`**：列出文件内容不符合指定的样式的文件名称
- **`-n`或`--line-number`**：在显示符合样式的那一行之前，标示出该行
- **`-q`或`--quiet或--silent`**：不显示任何信息
- **`-r`或`--recursive`**：此参数的效果和指定-d recurse参数相同
- **`-s`或`--no-messages`**：不显示错误信息
- **`-v`或`--revert-match`**：显示不包含匹配文本的所有行
- **`-V`或`--version`**：现实版本信息
- **`-w`或`--word-regexp`**：只显示全词匹配的行
- **`-x`或 `--line-regexp`**：只显示全行符合的行
- **`-y`**：此参数指定的效果和指定-i参数相同

挺多的，更专业一点的看`grep --help`，grep是在文件中搜索字符串的工具。

简单应用

```bash
grep lalal *
```

在当前目录下查找带有lalal的行

---

### Linux管道命令

**命令执行顺序控制**

- 顺序执行多条命令：

```bash
command1; command2; command3;
```

- 有条件的执行多条语句：

```bash
command1 && command2 || command3
```

上述表示`command1`执行成功（返回0）再执行`command2`，`command2`执行失败（返回非0）才执行`command3`

**管道命令**

> 管道是一种通信机制，通常用于进程间的通信（也可以通过socket进行网络通信），它表现出来的形式将前面每一个进程的输出（stdout）直接作为下一个进程的输入（stdin）。

`|`

- 管道命令仅能处理standard output，对于standard error output会予以忽略。`less`，`more`，`head`，`tail`等是可以接收stdin的命令，所以它们是管道命令，`ls`，`cp`，`mv`不接受stdin的命令，所以它们不是
- 管道命令必须要能够接受来自前一个命令的数据成为stdin继续处理才行

**more和less**

都是用来查看文件，不同之处在于，more是把当前查看的内容打出来给你看，而less是类似于man查看文档的那种，不会打印出来，并且可以用pageup/pagedn上下翻页

结合管道，可以用

```bash
ls -al /usr/lib | less
```

**cut**

```bash
cut -d '[分割字符]' -f field
```

用于分割字符

```bash
cut -c [字符范围]
```

字符范围

`-d`：后面接分割字符，通常与`-f`一起使用

`-f`：根据`-d`将信息分割成数段，`-f`后接数字，表示取出第几段

`-c`：以字符为单位取出固定字符区间的信息

**剩下的再找时间看**

https://www.jianshu.com/p/9c0c2b57cb73

---

### 输出带颜色的文本

**使用[ANSI转义码](https://en.wikipedia.org/wiki/ANSI_escape_code)进行颜色的设置**

| 颜色             | 前景色 | 背景色 |
| ---------------- | ------ | ------ |
| 黑色 (Black)     | 30     | 40     |
| 红色 (Red)       | 31     | 41     |
| 绿色 (Green)     | 32     | 42     |
| 黄色 (Yellow)    | 33     | 43     |
| 蓝色 (Blue)      | 34     | 44     |
| 紫红色 (Magenta) | 35     | 45     |
| 青色 (Cyan)      | 36     | 46     |
| 白色 (White)     | 37     | 47     |

code

```shell
echo -e "\033[34m文本\033[0m"
```

需要的参数是`-e`，文字的位置是要输出的内容，左右的`\033[xxm`控制颜色，为0的时候就恢复原来的颜色。

前景色和背景色，其实前景色和背景色不分先后，因为颜色值的范围不一样

code

```shell
echo -e "\033[前景色;背景色m文本\033[0m"
```

加粗与否也可以这样控制

| ANSI 码 | 含义           |
| ------- | -------------- |
| 0       | 常规文本       |
| 1       | 粗体文本       |
| 4       | 含下划线文本   |
| 5       | 闪烁文本       |
| 7       | 反色(补色)文本 |

---

获取当前ssh登录的IP

```bash
echo $SSH_CLIENT |awk ' { print $1 }'
```

