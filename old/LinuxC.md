

这里搞一个Linux下的东西

首先hello.c 

然后

```bash
gcc hello.c –o hello.out
```

此时出现一个hello.out就可以运行了

这里如果不规定名字的话会出现a.out 

另外还可以用

```bash
gcc –E hello.c hello.out
```

此时出现的是预处理之后的东西，-E就是

-E                       Preprocess only; do not compile, assemble or link

差不多就这些

---

**死锁**：如果有两个线程A和B（更多个也是一样的），线程A拿到了资源1的锁并等待资源2，线程B拿到了资源2的锁并等待资源1，这样在没有外力的情况下，两个线程都无法正常执行下去，就发生了死锁。

---

对于`extern`，用的时候，被extern的变量不能再被赋值，不然编译就过不去了。

---

关于`void *`，`void *`可以接收任意类型的指针而无需转换

---

**关于`sleep()`**

```c
#inlcude <unistd.h>
```

```c
sleep([seconds]);
usleep();
```

---

**关于`fflush`**

```c
#include <stdio.h>
```

例

```c
#include <stdio.h>
#include <unistd.h>

int main()
{
    fprintf(stdout, "11111111111111111111111");
    fflush();
    sleep(5);
    fprintf(stdout, "22222222222222222222222");
}
```

首先输出到`stdout`，是不会直接打印出来的，要等到程序运行结束才会一起输出，上述的`fflush(stdout)`，会刷新输出缓冲区，直接输出，也就是运行中先直接输出一堆1，然后停了5s，然后在运行结束前输出一堆2.

这算是C++的东西，无序表，即unordered_map，用法上跟map应该差不多，不同的是底层的实现上，map等使用的是红黑树，如果需要得到有序序列，则选择这个，如果需要更高的查询效率，则使用unordered_map等，简单写一下用法

```c++
#include <iostream>
#include <unordered_map>

using namespace std;

int main()
{
    std::unordered_map<int, double> *m = new std::unordered_map<int, double>;
    m->insert(std::make_pair(1, 1.11111));
    m->insert(std::make_pair(2, 2.22222));

    std::cout << m->at(1) << m->at(2) << endl;
}
```

搞了一个更通用的：

```c++
#include <iostream>
#include <unordered_map>

using namespace std;

int main()
{
    std::unordered_map<double *, double> *m = new std::unordered_map<double *, double>;
    double a[] = {1.23456, 9.87654};
    m->insert(std::make_pair(&a[0], a[0]));
    m->insert(std::make_pair(&a[1], a[1]));

    for(int i = 0; i < 3; i++)
    {
        if(m->find(&a[i]) != m->end())
            std::cout << m->at(&a[i]) << endl;
        else
            std::cout << "cant find key " << i << endl;
    }
}
```

再稍微详细点整理的话：

```c++
#include <unordered_map>
//声明和定义，使用类型
std::unordered_map<int, double> *m = new std::unordered_map<int, double>;
//插入一条数据
m->insert(std::make_pair<int, double>(1, 2.0));
//取对应键的值，也可以对其值进行修改
m->at(1);
//迭代器
std::unordered_map<int, double>::iterator it;
//begin应该是第一个数据，返回的应该是个迭代器
//end应该不是数据，end应该意味着到达了最后
//查找某键对应的值个数，但unordered_map应该是一个键对应一个值，所以应该只有0或者1
m->count(1);
//判断容器是否为空，返回一个bool值
m->empty();
//查找，如果找到，则返回找到的那个的迭代器，如果没找到，则返回m->end()
m->find(1);
//容器中的元素个数
m->size()
```

---

以前会使用clear等等去清屏幕，今天发现了一些好玩的东西，本质上跟\033[1;31m是一样的，所以进行一些记录，需要的时候就过来找一下

可以通过echo -e进行测试

 ```bash
NAME
       echo - display a line of text

SYNOPSIS
       echo [SHORT-OPTION]... [STRING]...
       echo LONG-OPTION

DESCRIPTION
       Echo the STRING(s) to standard output.

       -n     do not output the trailing newline

       -e     enable interpretation of backslash escapes

       -E     disable interpretation of backslash escapes (default)

       --help display this help and exit

       --version
              output version information and exit

       If -e is in effect, the following sequences are recognized:

       \\     backslash

       \a     alert (BEL)

       \b     backspace

       \c     produce no further output

       \e     escape

       \f     form feed

       \n     new line

       \r     carriage return
       
       \t     horizontal tab

       \v     vertical tab

       \0NNN  byte with octal value NNN (1 to 3 digits)

       \xHH   byte with hexadecimal value HH (1 to 2 digits)
 ```

几个小测试


- `-n` 发现不会换行了
```bash
ippfcox@ubuntu:~$ echo -n aa
aaippfcox@ubuntu:~$ 
```

- `\\`就不说了`\a`没法测试

- `\b` 删除前一个字符
```bash
ippfcox@ubuntu:~$ echo -e "123\b456"
12456
```

- `\c` 终止后面的所有输出
```bash
ippfcox@ubuntu:~$ echo -e "123\c456"
123ippfcox@ubuntu:~$ 
```

- `\e` 说是esc，打出来的是个奇怪的东西，5315上的效果跟`\c`一样
```bash
ippfcox@ubuntu:~$ echo -e "123\e456"
123 456
```

- `\f` 换行但水平方还在同一位置
```bash
ippfcox@ubuntu:~$ echo -e "123\f456"
123
   456
```

- `\n` 就不测了，太熟悉了


- `\r` 回到行首，可以看到先输出的1234被456覆盖掉了123
```bash
ippfcox@ubuntu:~$ echo -e "1234\r456"
4564
```

- `\t`也不测了
- `\v`纵向制表符，怎么看起来跟`\f`是一样的

```bash
ippfcox@ubuntu:~$ echo -e "1234\v456"
1234
    456
```

- `\0NNN`和`\xHH`，一个是跟八进制数，一个是跟十六进制数，输出带颜色的就是用的八进制的那个，更详细的用法还要再研究

ANSI/3.64控制码标准：均以`Esc[`作为控制码开始的标志，在ASCII中Esc是八进制033，十六进制1B，所以我们可以使用`\033[`或者`\x1B`使用ANSI控制码

- 以m结尾的是显示相关的
  - 0m          关闭所有属性
  - 1m          加粗
  - 4m          下划线
  - 5m          闪烁
  - 7m          反显
  - 8m          消隐
  - 30m ~ 37m   设置前景色
  - 40m ~ 47m   设置背景色
- ANSI颜色代码
  - 0: 黑 
  - 1: 深红
  - 2: 绿 
  - 3: 黄色
  - 4: 蓝色
  - 5: 紫色
  - 6: 深绿
  - 7: 白色  
- 其他ANSI控制代码
  - nA 光标上移n行 
  - nB 光标下移n行 
  - nC 光标右移n行 
  - nD 光标左移n行 
  - y;xH设置光标位置 
  - 2J 清屏 
  - K 清除从光标到行尾的内容 
  - s 保存光标位置 
  - u 恢复光标位置 
  - ?25l 隐藏光标 
  - ?25h 显示光标

除了这个，我研究了一个Linux的非阻塞键盘输入

直接贴代码

```c
#include <stdio.h>
#include <stdlib.h>

#define TTY_PATH "/dev/tty"
#define STTY_US "stty raw -echo -F "
#define STTY_DEF "stty -raw echo -F "

#define STDIN 0

static int kbhit()
{
    fd_set rfds;
    struct timeval tv;
    int ch = 0;
    
    FD_ZERO(&rfds);
    FD_SET(STDIN, &rfds);
    tv.tv_sec = 0;
    tv.tv_usec = 1000;
    
    if(select(STDIN + 1, &rfds, NULL, NULL, &tv) > 0)
    {
        return 1;
    }
    else
    {    
        return 0;
    }
}

int main()
{
    int ch = 0;
    system(STTY_US TTY_PATH);
    
    while(1)
    {
        if(kbhit())
        {
            ch = getchar();
            if(ch == 27) //Esc
            {
                int ch2 = getchar();
                int ch3 = getchar();
                printf("[%d %d %d]\n\r", ch, ch2, ch3);   
            }
            else
            {
                printf("[%d-%c]\n\r", ch, ch);
            }
            fflush(stdout);
            if(ch == 3)
            {
                system(STTY_DEF TTY_PATH);
                break;
            }
        }
    }
    
    return 0;
}
```

比较值得注意的是方向键的按下检测，4个方向键实际是Esc[A，Esc[B，Esc[C，Esc[D，也就是三个字符，这与前面ANSI控制的是差不多的

stty可以获取当前终端的行数列数等信息

---

关于C++类的一点写法

我觉得这样是比较好的，在.h中声明类的结构，包括所有的成员函数，在.cpp中进行各个函数的定义

---

如果有一个结构体

```c
typedef struct aaaaaa
{
    [blahblah]
}aaaaaa_t;
```

当调用`sizeof`计算其空间大小的时候

```c
sizeof(aaaaaa_t); //是可以的
sizeof(struct aaaaaa); //也是可以的
//sizeof(aaaaaa); //是不可以的
```

函数指针

```c
#include <stdio.h>

int fun1(int a, int b)
{
    printf("this is fun1\n");
    return a + b;
}

int fun2(int a, int b)
{
    printf("this is fun2\n");
    return a - b;
}

int fun3(int a, int b)
{
    printf("this is fun3\n");
    return a * b;
}

void aaa(int a, int b, int (*pf)(int, int))
{
    printf("aaa: %d\n", pf(a, b));
}

int main()
{
    int (*pfs[3])(int, int) = {fun1, fun2, fun3};
    for(int i = 0; i < 3; i++)
    {
        aaa(12, 20, pfs[i]);
    }
}
```

---

```c++
 const char Xorg_parh[] = "/usr/bin/Xorg";
    std::vector<char*> Xorg_argv;
    Xorg_argv.push_back(const_cast<char *>("Xorg"));
    Xorg_argv.push_back(const_cast<char *>("-depth"));
    Xorg_argv.push_back(const_cast<char *>("24"));
    Xorg_argv.push_back(NULL);

    system("rm /tmp/.X0-lock");

    int pid = fork();

    if(pid < 0)
    {
        RPTERR("Xorg start failed.");
        return OD_ERROR;
    }
    else if(pid == 0)
    {
        if(execv(Xorg_parh, &Xorg_argv[0]) < 0)
        {
            RPTERR("Xorg start failed.");
        }
        exit(0);
    }
    else
    {
        obj->Xorg_pid = pid;
        od_msleep(2000);
        RPTWRN("Xorg start successfully: %d", obj->Xorg_pid);
        system("ps -ef | grep Xorg | grep -v grep");
        OD_PRINT("\033[1;35mEnd Print\033[0m");
        return OD_OK;
    }


//    if(waitpid(obj->Xorg_pid, NULL, WNOHANG) > 0)
//    {
//        RPTERR("Xorg[%d] stoped, try to restart!", obj->Xorg_pid);
//        obj->malloc_free_rst = DEF_RST;
//        obj->real_time_rst = DEF_RST;
//        tube_put_buf(obj->ipcInYuv, hBufYuvShareInfo);
//        return OD_ERROR;
//    }
```

以上例子简单说明fork+exec启动子进程以及监测其活动（后续整理）

根据我查阅的资料来讲，fork是复制一个与原进程一模一样的进程出来，如果是多线程应用的话只复制当前线程，其他线程直接蒸发，所以这里会引发一些问题，主要是锁的问题，如果原进程中某个锁在其他线程中被拿到了，那么在新产生的子进程中这个锁仍旧是被锁定的，但是却没有主人了，作为其主人的那个线程直接蒸发了，所以尽量避免这种情况出现。

当然，很多情况我们fork只是为了执行exec系列的函数，这些函数最终是系统调用，其效果是如果其执行的程序成功了，那么这一函数后面的代码就都不执行了，即上文19行后面就没有了，整个进程的上下文都变成了执行的程序的上下文。

我在上面这里使用fork的目的是我希望Xorg成为我的子进程，我可以通过waitpid对其进行监控，在其死掉的时候可以方便地进行重启（后来失败了，还有一些其他的原因），但是使用system是成功的，system实际上是fork+execl，跟我在代码里用的差不多，为什么就可以呢？仔细看参数，system中excel始终都是启动的sh，即此时我们的子进程是个shell程序，通过shell去执行的另一个程序（具体是不是变成了shell的子进程没去深究），所以我也尝试将上面直接启动Xorg改为通过shell去执行，果然是可行的。但是为啥直接启动就不行呢？

---

mac地址转成8位的字节该怎么写：

```c
sscanf(mac_str, "%hhx:%hhx:%hhx:%hhx:%hhx:%hhx", &mac[0], &mac[1], &mac[2], &mac[3], &mac[4], &mac[5]);
```

这个%hhx是关键

---

更换了编译器版本，并且更换了链接的搜索路径为9350E在用的那个，然后就出现了一些问题，主要是libpthread和libc链接不上，说找不到

/lib/libpthread.so.0 /usr/lib/libpthread_nonshared.a，看了一下，发现libc.so和libpthread.so是两个rwrr的文件，与其他的文件权限都不一样，当时也没多想，就重新链到了动态库上，然后就开始出现各种问题，找到一个帖子

https://www.linuxquestions.org/questions/linux-general-1/undefined-reference-to-%60__libc_csu_fini%27-849247/，发现这里的问题是类似的，于是看了一下，这两个.so文件实际上是两个ascii文件，里面是有内容的

```
jiangdongchao@ubuntu:~/work_dir/9350Ev3/kernel/rootfs/usr/lib$ cat libc.so 
/* GNU ld script
   Use the shared library, but some functions are only in
   the static library, so try that secondarily.  */
OUTPUT_FORMAT(elf64-littleaarch64)
GROUP ( /lib/libc.so.6 /usr/lib/libc_nonshared.a  AS_NEEDED ( /lib/ld-linux-aarch64.so.1 ) )
jiangdongchao@ubuntu:~/work_dir/9350Ev3/kernel/rootfs/usr/lib$ cat libpthread.so 
/* GNU ld script
   Use the shared library, but some functions are only in
   the static library, so try that secondarily.  */
OUTPUT_FORMAT(elf64-littleaarch64)
GROUP ( /lib/libpthread.so.0 /usr/lib/libpthread_nonshared.a )
```

这样也就说明了当时为啥会报找不到/lib/libpthread.so.0 /usr/lib/libpthread_nonshared.a，是因为里面指向了这样两个文件，运行的时候是没有问题，但是链接的时候就会指向错误的文件了，所以这里尝试修改成相对路径试试

```
jiangdongchao@ubuntu:~/work_dir/9350Ev3/kernel/rootfs/usr/lib$ cat libc.so       
/* GNU ld script
   Use the shared library, but some functions are only in
   the static library, so try that secondarily.  */
OUTPUT_FORMAT(elf64-littleaarch64)
GROUP ( ../../lib/libc.so.6 ../../usr/lib/libc_nonshared.a  AS_NEEDED ( ../../lib/ld-linux-aarch64.so.1 ) )
jiangdongchao@ubuntu:~/work_dir/9350Ev3/kernel/rootfs/usr/lib$ cat libpthread.so 
/* GNU ld script
   Use the shared library, but some functions are only in
   the static library, so try that secondarily.  */
OUTPUT_FORMAT(elf64-littleaarch64)
GROUP ( ../../lib/libpthread.so.0 ../../usr/lib/libpthread_nonshared.a )
```

之前出现的问题解决了！高光！！！