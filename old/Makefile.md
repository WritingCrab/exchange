# Makefile

**目标（target）**：是要干什么，make后生成什么

**依赖（dependency）**：目标所依赖的其他目标

**命令（command）**：make如何生成目标

代码放在192.165.53.15下面的~/work_dir/C_test_prjs/makefile_test_180801/

```makefile
all:
	echo "Hello World"
```

`all`是一个目标，目标放在`:`前面

第二行的那句`echo "Hello World"`就是命令，生成目标的命令可以是操作系统命令行的命令或make定义的函数

命令所在行必须以Tab开头，四个空格是不行的，在vim中，需要命令`:set noexpandtab`

执行的时候可以

```bash
$ make
$ make all
```

输出的内容是`echo`那行命令和结果

```bash
echo "Hello World"
Hello World
```



一个Makefile可以定义多个目标，`make`的时候带不同的参数就可以构建不同的目标，第一个没参数的`make`与后面的输出相同是由于存在默认目标，默认目标就是Makefile中定义的第一个目标

`@`可以压制命令本身的输出，如果前面

```makefile
	@echo "Hello World"
```

那么输出就只有

```bash
Hello World
```

依赖

如果Makefile：

```makefile
all: test
	@echo "echo all"
test:
	@echo "echo test"
```

执行`make`，输出

```bash
echo test
echo all
```

这个`test`就是`all`这一目标的依赖，或者先决条件。make会按照从左到右（统一规则中），从上到下（不同规则中）的先后顺序构建一个规则所依赖的每一个目标

依赖即指出在构建目标之前，目标所依赖的先决条件必须首先被满足

规则的语法：

```makefile
targets: prerequisites
	command
	...
```

一个规则可以定义多个目标（也就是说一个规则实际上就是一个Makefile）

采用Makefile进行编译的时候，Makefile的先决条件大多是指具体的程序文件

---

做一个很小的simple项目

**simple/foo.c**

```c
#include <stdio.h>

void foo()
    printf("This is foo().\n");
}
```

**simple/main.c**

```c
extern void foo();

int main()
{
    foo();
    return 0;
}
```

**simple/Makefile**

```makefile
all: main.o foo.o
	gcc -o simple main.o foo.o #将foo.o和main.o链接输出到simple
main.o: main.c
	gcc -o main.o -c main.c #编译不链接，输出到main.o
foo.o: foo.c
	gcc -o foo.o -c foo.c #编译不链接，输出到foo.o
clean:
	rm main.o foo.o #删除产生的中间文件
```

看如下的两次构建过程

```bash
$ make
gcc -o main.o -c main.c
gcc -o foo.o -c foo.c
gcc -o simple main.o foo.o
$ make
gcc -o simple main.o foo.o
```

make通过时间戳来判定是否需要重新编译，如果发现目标的依赖比目标更新，那么就需要重新构建这个目标，因此时间在make的使用中十分重要。

可以注意到，执行`make`、`make all`的时候，都会进行链接，如果执行`make foo.o`即

```bash
$ make foo.o
make: 'foo.o' is up to date.
```

出现上述不同的原因是，`make`或者`make all`的时候，`all`不是可生成的目标，因此会去重新构建，而foo.o本来就存在，就不需要再去重新构建了。

---

**假目标**：`.PHONY`

假如没有假目标，在调用`make clean`的时候，如果目录中有一个`clean`的文件，那么操作将不会被执行，目录中的文件名与目标名重名是难以避免的，此时就需要假目标，其实也就是说，被定义为假目标的的目标，在make的时候一定会被构建。此时的Simple项目：

**Simple/makefile**

```makefile
.PHONY: clean

all: main.o foo.o
	gcc -o simple main.o foo.o # 将foo.o和main.o链接输出到simple
main.o: main.c
	gcc -o main.o -c main.c # 编译不链接，输出到main.o
foo.o: foo.c
	gcc -o foo.o -c foo.c # 编译不链接，输出到foo.o
clean:
	rm main.o foo.o # 删除产生的中间文件
```

**变量**

变量定义

```makefile
变量名 = 右值 # 右值是可以不存在的
```

引用变量的时候可以用`$(变量名)`或者`${变量名}`，此时的SImple项目：

```makefile
.PHONY: clean

CC = gcc
RM = rm

OUT = simple
OBJS = foo.o main.o

$(OUT): $(OBJS)
	$(CC) -o $(OUT) $(OBJS)
main.o: main.c
	$(CC) -o main.o -c main.c
foo.o: foo.c
	$(CC) -o foo.o -c foo.c
clean: 
	$(RM) -fr $(OUT) $(OBJS)
```

**自动变量**

在Makefile中，目标名和依赖名可能会重复出现，如果一个发生改变，可能就要改一大片，可以使用以下自动变量：

- `$@`：用于表示一个规则中的目标，当规则中有多个目标时，`$@`是任何造成命令被运行的目标；
- `$^`：表示规则中的所有依赖；
- `$<`：表示规则中的第一个依赖。

在Makefile中，想要输出`$`首先得在前面加`$`，也就是`$$`，但是`$@`对bash shell也有含义，因此还需要加反斜杠`\`，也就是`\$$@`。

采用自动变量后，Simple项目的Makefile：

```makefile
.PHONY: clean

CC = gcc
RM = rm

OUT = simple
OBJS = foo.o main.o

$(OUT): $(OBJS)
	$(CC) -o $@ $^
main.o: main.c
	$(CC) -o $@ -c $^
foo.o: foo.c
	$(CC) -o $@ -c $^
clean:
	$(RM) -fr $(OUT) $(OBJS)
```

**特殊变量**

- `$(MAKE)`：当前处理Makefile的命令，若一个Makefile调用另一个Makefile的时候可能会用到；
- `$MAKECMDGOALS`：当前构建的目标名，仅支持命令行输入的名称，仅`make`不会输出默认目标。

**变量的类别与赋值**

- `=`：递归扩展变量赋值
- `:=`：简单扩展变量赋值

一个简单的例子

```makefile
.PHONY: all

x = jkj
xx := 111
y = $(x) lll
yy := $(xx) 222
x = ooo
xx := 333
x = ppp
xx := 444

all:
        @echo "y = $(y), yy = $(yy)"
```

输出

```
y = ppp lll, yy = 111 222
```

递归赋值的等式右端会一直走到最后，递归展开，简单赋值只要进行一次可行（不清楚这个说法是否准确）的展开

另外，如果使用

```makefile
y = 111
y = $(y) 222
```

会报错，因为会导致`$(y)`无限展开，应当使用

```makefile
y = 111
y := $(y) 222
```

这样是正确的用法

- `?=`：条件赋值

当变量未被定义，那么将被赋值，当变量已经被定义，则不会改变原值

```makefile
.PHONY: all

x = 1
x ?= 2

y ?= 3

z = 
z ?= 4

all:
        @echo "x = $(x), y = $(y), z = $(z)"
```

输出为

```
x = 1, y = 3, z = 
```

也就是说，只有未定义才能被定义并赋值，如果已经被定义，即使值是空的，也一样不会再次被赋值

- `+=`：追加赋值

**变量及其值的来源**

除Makefile中对变量进行赋值外，还有其他的方式：

- 自动变量，其值是在每个规则中根据上下文自动获得的
- `make var=value`
- shell中使用`export var=value`

**高级变量引用功能**

可以在引用变量的时候进行字符串的替换

```makefile
.PHONY: all

fff = 1.a 2.a 3.b 4.a
uuu = $(fff:.a=.z)

all:
        @echo "uuu = $(uuu)"
```

输出

```
uuu = 1.z 2.z 3.b 4.z
```

**避免变量被覆盖的方法**

可以使用`override`来避免参数被覆盖，如下例子

```makefile
.PHONY: all

a = aaaaaa
override b = bbbbbb

all:
        @echo "a = $(a)"
        @echo "b = $(b)"
```

使用指令

```bash
make a=zzzzz b=yyyyy
```

得到输出

```
a = zzzzz
b = bbbbbb
```

发现`a`被覆盖掉了，而`b`由于有指令`override`的保护，没有被覆盖掉

**通配符**

`%`：类似*

看上去很厉害，但我觉得应用空间不大

**函数**

注意函数的调用方式

```makefile
$([function] ...)
```

- **`abspath`**

```makefile
$(abspath _name)
```

将`_name`中的路径转换成绝对路径

```makefile
.PHONY: all

PWD = $(abspath ./)

all:
        @echo "[abspath] PWD = $(PWD)"
```

输出

```
[abspath] PWD = /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801/func_test
```

- **`addprefix`**

```makefile
$(addprefix _prefix, _name)
```

给`_name`中的所有名字前面加`_prefix`前缀

```makefile
.PHONY: all

names = qqq www eee rrr
NAMES = $(addprefix 666- 7, $(names))

all:
        @echo "[addprefix] NAMES = $(NAMES)"
```

输出

```
[addprefix] NAMES = 666- 7qqq 666- 7www 666- 7eee 666- 7rrr
```

- **`addsuffix`**

```makefile
$(addsuffix _suffix, _name)
```

给`_name`中的所有名字增加`_suffix`后缀

```makefile
.PHONY: all

names2 = aaa sss ddd fff

NAMES2 = $(addsuffix 888- 9, $(names2))

all:
        @echo "[addsuffix] NAMES2 = $(NAMES2)"
```

输出

```
[addsuffix] NAMES2 = aaa888- 9 sss888- 9 ddd888- 9 fff888- 9
```

- **`eval`**

```makefile
$(eval _text)
```

使make再一次解析`_text`中的语句

- **`filter`**

```makefile
$(filter _pattern, _text)
```

从名字列表`_text`中得到满足`_pattern`的名字列表

```makefile
.PHONY: all

names3 = aaa bbb cca bbc ddc

NAMES3 = $(filter %a %c, $(names3))

all:
        @echo "[filter] NAMES3 = $(NAMES3)"
```

输出

```
[filter] NAMES3 = aaa cca bbc ddc
```

- **`filter-out`**

```makefile
$(filter-out _pattern, _text)
```

从名字列表`_text`中滤除满足`_pattern`的名字列表

```makefile
.PHONY: all

names3 = aaa bbb cca bbc ddc

NAMES4 = $(filter-out %a %c, $(names3))

all:
        @echo "[filter-out] NAMES4 = $(NAMES4)"
```

输出

```
[filter-out] NAMES4 = bbb
```

- **`notdir`**

```makefile
$(notdir _names)
```

用来从路径`_name`中抽取文件名，并将文件名返回

```makefile
.PHONY: all

dir1 = /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801/func_test/Makefile

DIR1 = $(notdir $(dir1))

all:
        @echo "[notdir] DIR1 = $(DIR1)"
```

输出

```
[notdir] DIR1 = Makefile
```

- **`patsubst`**

```makefile
$(patsubst _pattern, _replacement, _text)
```

将`_text`中符合`_pattern`的名字替换为`_replacement`

```makefile
.PHONY: all

names3 = aaa bbb cca bbc ddc

NAMES5 = $(patsubst %a, %vv, $(names3))

all:
        @echo "[patsubst] NAMES5 = $(NAMES5)"
```

输出

```
[patsubst] NAMES5 =  aavv bbb  ccvv bbc ddc
```

注意，似乎只能匹配一个pattern

- **`realpath`**

```makefile
$(realpath _names)
```

用于获取`_names`所对应的真实路径名

```makefile
.PHONY: all

dir1 = /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801/func_test/Makefile
dir2 := $(dir1) ./ ./..

NAMES6 = $(realpath $(dir2))

all:
        @echo "[realpath] NAMES6 = $(NAMES6)"
```

输出

```
[realpath] NAMES6 = /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801/func_test/Makefile /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801/func_test /home/jiangdongchao/work_dir/C_test_prjs/makefile_test_180801
jiangdongchao@ubuntu:~/work_dir/C_test_prjs/makefile_test_180801/func_test$ cat Makefile 
```

多个是可以的，文件路径会获取所在目录的路径

- **`strip`**

```makefile
$(strip _string)
```

清除名字列表`_string`中的多余空格

```makefile
.PHONY: all

names4 = aaa           bbb      ccc d

NAMES7 = $(strip $(names4))

all:
        @echo "[strip] NAMES7 = $(NAMES7)"
```

输出

```
[strip] NAMES7 = aaa bbb ccc d
```

- **`wildcard`**

```makefile
$(wildcard _pattern)
```

可以从当前工作目录获取满足`_pattern`模式的文件或目录名

**需要用*去匹配文件名，%匹配不出来！**

---

一个稍微复杂一点点的项目complicated

源码

**complicated/foo.h**

```c
#ifndef _FOO_H
#define _FOO_H

void foo();

#endif
```

**complicated/foo.c**

```c
#include <stdio.h>
#include "foo.h"

void foo()
{
    printf("This is fvcing foo\n\n");
}
```

**complicated/main.c**

```c
#include "foo.h"

int main()
{
    foo();
    return 0;
}
```

对目录结构的需求

- 将所有的目标文件放入objs子目录
- 将最终生成的可执行程序放入exe子目录

```makefile
.PHONY: all clean

MKDIR = mkdir
RM = rm
RMFLAGS = -fr

CC = gcc

DIR_OBJS = objs
DIR_EXES = exes
DIRS = $(DIR_OBJS) $(DIR_EXES)
EXE = complicated
EXE := $(addprefix $(DIR_EXES)/, $(EXE))
SRCS = $(wildcard *.c)
OBJS = $(SRCS:.c=.o)
OBJS := $(addprefix $(DIR_OBJS)/, $(OBJS))

all: $(DIRS) $(EXE)

$(DIRS):
        $(MKDIR) $(DIRS)

$(EXE): $(OBJS)
        $(CC) -o $@ $^

$(DIR_OBJS)/%.o: %.c
        $(CC) -o $@ -c $^

clean:
        $(RM) $(RMFLAGS) $(DIRS) $(EXES)
```





---

静态模式

```makefile
TARGETS ...: TARGET-PATTERN: PREREQ-PATTERNS ...
COMMANDS
...
```

静态模式的依赖文件应该是类似的而不是相同的，我的理解就是，所有的目标和其各自的依赖之间的关系应该是类似的，如果用前述complicated的例子来说明的话

```makefile
$(DIR_OBJS)/%.o: %.c
        $(CC) -o $@ -c $^
```

可以改为

```makefile
$(OBJS): $(DIR_OBJS)/%.o: %.c
```

然后make的结果是一样的，大概语法是这样，具体有什么用呢