# ALSA音频

## ALSA

### 概述

ALSA表示高级Linux声音体系结构(Advanced Linux Sound Architecture)，ALSA相对底层并且支持多种音频处理的API，ALSA由许多声卡的驱动程序组成，提供一个libasound库，应用程序开发者应该使用的是libasound而不是内核中的ALSA接口，libasound提供高级的编程接口，提供设备逻辑命名（但是如果不是由开发者提供设备文件，那这个库是怎么知道设备文件的呢）

> 物理设备是实际存在的,逻辑设备是依靠物理设备存在的.没有物理设备不可能存在逻辑设备,但有物理设备不一定有逻辑设备.

ALSA分成几个主要的接口

- 控制接口：提供管理声卡注册和请求可用设备的通用功能 

- PCM接口：管理数字音频回放(playback)和录音(capture)的接口。它们是开发数字音频程序最常用到的接口。

- Raw MIDI接口:支持MIDI(Musical Instrument Digital Interface),标准的电子乐器。这些API提供对声卡上MIDI总线的访问。这个原始接口基于MIDI事件工作，由程序员负责管理协议以及时间处理。

- 定时器(Timer)接口：为同步音频事件提供对声卡上时间处理硬件的访问。

- 时序器(Sequencer)接口

- 混音器(Mixer)接口

API库使用的是逻辑设备名而不是设备文件，逻辑设备名可以是真实的硬件名字也可以是插件的名字，硬件名字使用hw:i,j的格式，i是卡号，j是卡上的设备号，插件使用的名字是plughw:表示一个插件，插件不提供对硬件设备的访问，提供重采样等软件特性，硬件本身不支持这些特性。

### 声音缓存和数据传输

声卡有一个硬件缓冲区，记录保存下来的样本（大概就是我们的FIFO），当缓冲区满的时候将产生一个中断，内核将调用dma将缓冲区数据传送到ddr，play的时候也是类似的。

应用程序的缓冲区（这个是指的alsa相关的吗）是可以调整的，缓冲区太大的话会出现播放延迟的情况。ALSA将缓冲区拆分成一系列的周期（period），ALSA以period为单元传送数据。

每个period包含一些帧，每一帧包含两个通道或一个通道的样本，在交错模式interleaved下，左右通道的样本交替保存在同一个帧内，非交错模式下，一个信道的所有样本数据存储在另一个信道的数据后面。

**over run**：采集时，如果应用程序读取过慢，硬件的ring buffer会出现未使用的数据被新数据覆盖的现象

**under run**：播放时，如果应用程序播放过慢，硬件播放设备会读到旧数据（或者空，取决于是不是会被清空，我觉得不会）

### 一个典型的音频程序

- 打开capture或play接口
- 设置硬件参数
- 数据的循环处理
- 结束

但是这块我看到的是，音频设备的初始化是通过一个结构体进行的，这个结构体中包含了音频设备的逻辑设备名和参数，然后调用一个接口，哦我明白了，实际这里使用的是一个odin中的模块，在里面看的话，确实是先打开设备然后进行参数的设置的。

---

# tcpdump

## tcpdump

下载：http://www.tcpdump.org/#latest-releases

参考：https://blog.csdn.net/solure/article/details/76100585

编译pcap

```bash
./configure --prefix=/home/jiangdongchao/work_dir/tcpdump/out --host=arm-linux --target=arm-linux CC=aarch64-linux-gnu-gcc --with-pcap=linux
make
make install
```

编译tcpdump

```bash
./configure --prefix=/home/jiangdongchao/work_dir/tcpdump/out --host=arm-linux --target=arm-linux CC=aarch64-linux-gnu-gcc ac_cv_linux_vers=2
make
make install
```

简单使用举例

```bash
sudo tcpdump -i eth0 -w /home/admin/sip_159.pcap -n -s0 udp port 5060
```

---

# 正则表达式

正则表达式（Regular Expression）是一种文本模式，包括普通字符（例如a-z字母）和特殊字符（称为元字符）

正则表达式使用单个字符串描述、匹配一系列某种句法的字符串们

## 为什么使用正则表达式

- 测试字符串内的模式

  测试输入字符车内是否出现如电话等模式：数据验证

- 替换文本

  其实跟上面没啥区别，多了一步替换而已

- 提取子字符串

  也没啥区别

## 语法

### 普通字符

没有显式指定为元字符的所有可打印和不可打印字符，包括所有大写字母和小写字母，所有数字，所有标点符号和一些其他符号

### 非打印字符

| 字符  | 描述                                                         |
| ----- | ------------------------------------------------------------ |
| `\c☺` | 匹配由☺指明的控制字符。`\cM`匹配一个Ctrl+M或者回车，☺必须为A-Z或a-z，否则将视为原义c |
| `\f`  | 匹配一个换页符。等价于`\x0c`和`\cL`                          |
| `\n`  | 匹配一个换行符。等价于`\x0a`和`\cJ`                          |
| `\r`  | 匹配一个回车符。等价于`\x0d`和`\cM`                          |
| `\t`  | 匹配一个制表符。等价于`\x09`和`\cl`                          |
| `\v`  | 匹配一个垂直制表符。等价于`\x0b`和`\cK`                      |
| `\s`  | 匹配任何空白字符，包括空格、制表符、换页符等，等价于`[\f\n\r\t\v]`。Unicode表达式会匹配全角空格 |
| `\S`  | 匹配任何非空白字符。等价于`[^ \f\n\r\t\v]`                   |

### 特殊字符

特殊字符需要用`\`进行转义

| 特殊字符 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| `$`      | 匹配输入字符串的结束位置。若设置了RegExp对象的Multiline属性，则`$`也匹配`\n`或`\r` |
| `()`     | 标记子表达式的开始和结束位置。子表达式可以获取供以后使用     |
| `*`      | 匹配前面的子表达式零次或多次                                 |
| `+`      | 匹配前面的子表达式一次或多次                                 |
| `.`      | 匹配除换行符`\n`之外的任何单字符                             |
| `[`      | 标记一个中括号表达式的开始                                   |
| `?`      | 匹配前面的子表达式零次或一次，或指明一个非贪婪限定符         |
| `\`      | 将下一个字符标记为或特殊字符、或原义字符。或向后引用、或八进制转义符 |
| `^`      | 匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符合集 |
| `{`      | 标记限定符表达式的开始                                       |
| `|`      | 指明两项之间的一个选择                                       |

- *什么叫获取供以后使用*
- *什么叫贪婪和非贪婪* **贪婪即尽可能多的匹配**
- *什么叫向后引用*
- *什么叫限定符表达式* **表示匹配次数**

### 限定符

限定符用来指定正则表达式的一个给定组件必须要出现多少次才能满足匹配，`*`或`+`或`?`合屏`{n}`或`{n,}`或`{n,m}`

| 字符    | 描述                                                     |
| ------- | -------------------------------------------------------- |
| `*`     | 匹配前面的子表达式零次或多次。等价于`{0,}`               |
| `+`     | 匹配前面的子表达式一次或多次。等价于`{1,}`               |
| `?`     | 匹配前面的子表达式零次或一次。等价于`{0,1}`              |
| `{n}`   | 匹配前面的子表达式n次。                                  |
| `{n,}`  | 匹配前面的子表达式至少n次。                              |
| `{n,m}` | 匹配前面的子表达式n到m次。如果有超过m个可匹配，只匹配m个 |

`*`、`+`都是贪婪的，会尽可能多的匹配文字，在后面加个`?`就可以实现非贪婪或者最小匹配

### 定位符

定位符将正则表达式固定到行首或行尾

| 字符 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| `^`  | 匹配输入字符串开始的位置，如果设置了Multiline属性，还会匹配`\n`或`\r`之后的内容 |
| `$`  | 匹配输入字符串结尾的位置，如果设置了Multiline属性，还会匹配`\n`或`\r`之后的内容 |
| `\b` | 匹配一个单词边界，即字与空格之间的位置                       |
| `\B` | 匹配非单词边界                                               |

**注意**：不能讲限定符和定位符一起使用。不允许`^*`之类的表达式。

若要匹配一行文本开始处的文本，要在正则表达式的开头使用`^`，不要和`[]`中的`^`混淆

若要匹配一行文本结束处的文本，要在正则表达式的结尾使用`$`

`\b`的位置，若在开头，在单词的开始处查找匹配项，若在结尾，则在结尾处查找

`\B`的位置不重要，又跟开头和结尾没关系

### 选择

用圆括号将所有的选项括起来，相邻选项用`|`分割。但使用圆括号会使相关的匹配被缓存，可以用`?:`放在第一个选项前来消除这种副作用。

`?:`是非捕获元之一，还有`?=`和`?!`

`?=`正向预查，在任何开始匹配圆括号内的正则表达式模式的位置来匹配搜索字符串

`?!`负向预查，在任何开始不匹配该正则表达式模式的位置来匹配搜索字符串

*emmmmm*

### 反向引用

对一个正则表达式模式或部分模式两边添加圆括号将导致相关匹配被存储到一个临时缓冲区，所捕获的每个子匹配都按照在正则表达式模式中从左到右出现的顺序储存，编号从1开始，最多存储99个。每个缓冲区都可以使用`\n`访问，其中`n`为标识特定缓冲区的一位或两位十进制数

可以使用前述非捕获元来重写捕获，忽略对相关匹配的保存。

给出栗子：

```
/\b([a-z]+) \1\b/
```

`[a-z]+`匹配一个单词，外面被括号扩起意味着匹配结果会被保存在临时缓冲区，通过`\1`可以访问到之前保存下来的内容，两边的`\b`意味着匹配的内容两边都是单词边界

综上，匹配的应该是两个相同的单词，中间用空格隔开

## 元字符

---

# 换字体

```bash
cd /usr/share/fonts
sudo mkdir YaheiConsolas
cd YaheiConsolas
sudo cp ~/下载/YaHei.Consolas.1.11.ttf
sudo chmod 644 YaHei.Consolas.1.11.ttf 
sudo mkfontscale
sudo mkfontdir
sudo fc-cache -fv 
```

# 消息队列

消息队列操作主要是三个函数

mq_open

mq_send

mq_receive

还有mq_timereceive

另外还有mq_close

mq_unlink

可以通过将mqueue挂载在dev里的方式进行查看

```bash
sudo mount -t mqueue none /dev/mqueue
```

另外open了的，即使关闭也不会删掉，需要unlink

上代码

```c
#include <fcntl.h>
#include <mqueue.h>
#include <stdio.h>
#include <string.h>

int main()
{
    int flag = O_RDWR | O_CREAT;
    int mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;
    mqd_t mq_id = mq_open("/mq1", flag, mode, NULL);
    if(mq_id < 0)
    {
        perror("snd mq create failed");
    }
    else
    {
        printf("snd mq create successfully\n");
    }
    
    char *buf = "snd alksdjflajsdf;aksdjf;aksdf;klajs;kfja;ksdf;alkf;aklj";
    mq_send(mq_id, buf, strlen(buf), 20);
    
    mq_close(mq_id);
    
    return 0;
}
```

```c
#include <fcntl.h>
#include <mqueue.h>
#include <stdio.h>

int main()
{
    int flag = O_RDWR;
    int mode = S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH;
    mqd_t mq_id = mq_open("/mq1", flag, mode, NULL);
    if(mq_id < 0)
    {
        perror("rcv mq create failed");
    }
    else
    {
        printf("rcv mq create successfully\n");
    }
    
    char buf[1024] = {0};
    int prio = 0;
    struct mq_attr attr;
    mq_getattr(mq_id, &attr);
    mq_receive(mq_id, buf, attr.mq_msgsize, &prio);
    
    printf("%s", buf);
    
    mq_close(mq_id);
    
    return 0;
}
```

编译需链接-lrt

可用代码，以后可以详细研究

---

# 几个音频算法

## G.711

[原文](https://blog.csdn.net/szfhy/article/details/52448906)

特点，采样率8kHz，典型的算法延迟为0.125ms

声音清晰，语音自然度高，但压缩率低，数据量常在32kbps以上，常用于电话语音（推荐64kbps），分为$\mu$率和$a$率

$a$率取S16（大概是16位采样）的13位（应该是最多13位，页有可能只取6位呢），第一位作为符号位，后面有8种情况，$a$率也叫g711a，运算过程：

- 取符号位并取反得到s；
- 获取强度位eee；
- 获取高位样本wxyz；
- 组合为seeewxyz，并将其逢偶取补

| Linear input code | Compressed code |
| ----------------- | --------------- |
| s0000000wxyza...  | s000wxyz        |
| s0000001wxyza...  | s001wxyz        |
| s000001wxyzab...  | s010wxyz        |
| s00001wxyzabc...  | s011wxyz        |
| s0001wxyzabcd...  | s100wxyz        |
| s001wxyzabcde...  | s101wxyz        |
| s01wxyzabcdef...  | s110wxyz        |
| s1wxyzabcdefg...  | s111wxyz        |

## G.729

1- 占用带宽小

使用普通编码的语音通讯需要占用64Kbps的带宽，而G.729仅仅需要8Kbps。

2- 占用CPU时间多

使用G.729时CPU的使用时间大约为G.711的4倍，所以使用G.729时需要注意服务器是否有足够的处理能力。

## AAC

AAC是一个有损压缩的音频编码集（新的工具也是支持无损的），其设计目标是替代原有的MP3标准，在相似的码率下质量优于MP3，这一目标已经达到。

AAC不是一个单一的解码器，而是一个音频编工具合集

**AAC-LC**，(Advanced Audio Coding Low-Complexity)是MPEG-2格式，设计用于数字电视，用于存储空间和计算能力有限的情况。AAC-LC充分利用心理声学原理，最大程度减少用于表达信号的比特数据，实现音频信号有效快速压缩，而不再追求输出信号和原始信号的相似度。重点技术有，Temporal Noise Shaping：瞬时噪声整形；Intensity Stereo：利用心理声学原理，人耳对高频信号的相位不敏感的原理；AAC-LC把6kHz作为声强立体声处理的起始频率在这个频率上的都进行处理；Perceptual Noise Substition：感知噪声替代用于频谱成分类似噪声（功率谱密度均匀）时，用人造噪声代替；Middle/Side：立体声编码，对一对声道的信号间的相关性去冗余。

---

# 视频的一些概念

8bit BT656(4:2:2) SDTV

[Here](https://www.cnblogs.com/cyyljw/p/6871756.html)

## 1. 帧的概念（Frame）

一个视频序列，N个帧，采集图像：逐行扫描（progressive scaning），一种是隔行扫描（interlaced scaning）。隔行扫描每一帧有2个场，顶场（top field）和底场（bottom field），顶场包含所有奇数行，底场包含所有偶数行。

## 2. 场的概念（field）

场由三部分组成：

场=垂直消隐顶场（First Vertical Blanking）+有效数据行（Active Video）+垂直消隐底场（Second Vertical Blanking）

顶场的有效数据行就是所有偶数行，底场的有效数据行就是所有奇数行，顶场和底场空白行的个数也不同，格式：

![](./pic_BTxxx/1.png)

对PAL制式，一帧625行，顶场有效数据行288行，底场有效数据行288行，其余行为垂直消隐信号，PAL制式的SDTV或D1的分辨率为720*576，一帧576行。顶场有效数据起始行为23行，底场有限山水起始行为335行（**为什么**），F标记奇偶场，V标记是否为垂直消隐信号。

## 3. 每一行的组成

行=结束码（EAV）+水平消隐（Horizontal Vertical Blanking）+起始码（SAV）+有效数据（Active Video）

![](./pic_BTxxx/2.png)

为什么水平消隐是280字节，作者也不清楚

为什么一行数据1440字节？PAL制式SDTV或D1的分辨率为720*576，YUV422，一行有720列Y，720列UV，也就是1440字节。

## 4. EAV和SAV

EAV和SAV都是4个字节，SAV后面跟着的就是有效视频数据，格式：

`FF 00 00 XY`

前3个字节是固定的，第4个是根据场和消隐信息决定的，8个bit含义如下：

`1 F V H  P3 P2 P1 P0`

其中

`F`：标记场信息，传输顶场为0，传输底场为1

`V`：标记消隐信息，传输消隐数据为1，传输有效数据为0

`H`：标记是EAV还是SAV，EAV为1，SAV为0

`P0~P3`：保护bit，起到校验作用，算法如下

![](./pic_BTxxx/3.jpg)

## 5. 总结

可以参看[《A Brief Introduction to Digital Video》](http://www.spacewire.co.uk/video_standard.html) 

## ITU-R BT.1120-6

CIF系统（$1920\times1080$），总行数1125，有效行1080

---

# 颜色空间等

**YUV**：RGB色彩空间的正交变换

**Y**：亮度

**U/Cb**：色差

**V/Cr**：色度

亮度最重要，于是使用4:2:2或者4:2:0减少数据量

 这块有一篇博客：<https://www.jianshu.com/p/a91502c00fb0>，写的不错，**其理解方式存疑**

 420是4个Y，U和V一共是2,422是4个Y，然后2个U和2个V，这里说的就没有440，所以这个说法不大一样，还得思考一下

 JPEG、MPEG-2、H.264有损压缩

RAR、ZIP无损压缩

 H.264 YUV4:2:0，标清D1@25fps

标清：720*576

D1是什么？帧长度？D1是制式，决定分辨率的

25fps

每秒数据量720*576*12*25=124Mbps，压缩后为1:30~1:150

 12是怎么来的呢，由于是YUV420，拿出4个像素，为4个Y，一个U和一个V，也就是6\*8bit，均分到每个像素上就是6\*8/4=12bit

 MPEG-2压缩率是264的一半，HEVC比264高20%~50%

RAR的压缩率为1:2（毕竟无损

MPEG-2（1995）、H.264（2003）、H.265（2013）

 

**PSNR**（信噪比）来衡量图像质量（客观）

用PQA（软件）计算DMOS（指标Difference Mean of Score）

 

**帧类型**：

**I**：编码只依赖自己：码流切割和错误恢复，压缩率低

**P**：参考在显示时间上位于其之前的帧

**B**：参考在显示时间上位于其之前和之后的帧，压缩率最高

 **GOP**：从一个I帧开始到下一个I帧为止的一些帧

编码帧顺序，没看懂

 编码流程

 每帧被分为宏块（MacroBlock）

---

## MIPI

一个联盟，将一些接口标准化

**CSI**(CMOS Sensor Interface)是for camera的

**DSI**(Display Serial Interface)是for display的

**D-PHY**：物理层目前的标准，采用1对源同步的差分时钟和1~4对差分数据线进行数据传输，DDR方式，是在时钟的上下边沿都有数据传输，支持HS(High Speed)和LP(Low Power)两种模式

**eDP**(embedded Display Port)

# VUE离线安装

首先下载nodejs

cmd，然后配置镜像站

```bash
npm config set registry=http://registry.npm.taobao.org
```

然后开始安装包

```bash
npm install vue -g
npm install vue-router -g
npm install webpack -g
npm install vue-cli -g
# vue-cli工具是内置了模板包括 webpack 和 webpack-simple,前者是比较复杂专业的项目，他的配置并不全放在根目录下的 webpack.config.js 中
npm install webpack-dev-server -g
# 是为了规避错误"执行 node start env提示 'webpack-dev-server 不是内部或外部命令，也不是可运行的程序
npm install -D webpack-cli -g
# 是为了规避错误Cannot find module 'webpack/bin/config-yargs
npm install html-webpack-plugin -g
# 是为了规避错误Cannot find module 'html-webpack-plugin
```

下载后的文件在"C:\Users\XXX账户文件夹\AppData\Roaming\npm\node_modules"。

将C:\Users\jiangdongchao\AppData\Roaming下的npm和npm-cache复制过来

将C:\Users\jiangdongchao\\.vue-templates复制过来



把原有目录中的node_modules给解压出来

基本上就差不多了，然后使用npm run dev，发现有个叫node_sass的东西有问题，所以又跑去公用机下载下来，然后传过来，再次执行，顺利启动了。

npm run build可以编译出来，但是需要在根目录放一个static的目录，然后就可以编译成功了，结果在dist中，但是9550的web目录下面还有一个style的目录，不知道是咋回事

---

#TS流基本概念

## 0. 

- PS（Program Stream）节目流，用于没有误差的媒体存储
- TS（Transport Streatm）传输流，用于有信道噪声的传播信道

## 1. 基本概念

- ES流（Elementary Stream）基本码流，编码后的裸数据；
- PES流，分割打包的ES流，加入了PES头，PES包可变长度，PES头中有显示时间标记PTS（Presentation Time Stamp），和解码时间标记（Decode Time Stamp），再加上节目参考时钟PCR，就可以重建视频流。

## 2.TS流格式

### 2.1 TS流格式

![](./pic_TS/1.png)

TS header是4bytB，1Byte的同步字节，1bit传输错误包差错表示，1bit净荷单元起始指示，1bit传送优先权，13bit包标识符PID，2bit传送加扰控制，2bit调整字段控制和4bit连续计数器

- 同步字节（sync byte）：1Byte，0x47；
- 传输差错指示符（transport error indicator）：1，不可纠正错误；
- 有效单元载荷起始符（payload unit start indicator）：1bit，TS包带有PES包数据时，置1，表示TS包的有效净荷以PES包的第一个字节开始；当TS包带有PSI数据时，置1，表示TS包带有PSI部分的第一个字节，即第一个字节带有指针pointer_field；空包时，该位置为0；
- 传输优先级（transport priority）：1bit，置1，标识相关包比其他具有相同PID的包具有更高优先级；
- PID：13bit，表示传送包的有效净荷中的数据类型，如下

| PID           | 描述                                                     |
| ------------- | -------------------------------------------------------- |
| 0x0000        | 节目关联表（program associate table，PAT）               |
| 0x0001        | 条件访问表（conditional access table，CAT）              |
| 0x0002        | 传输流描述表（transport stream description table，TSDT） |
| 0x0003~0x000F | 保留                                                     |
| 0x0010~0x1FFE | 分配network PID、Program map PID、elementary PID，或其他 |
| 0x1FFF        | 空包                                                     |

PID直接表征本TS包的用途，PAT和PMT是比较重要的

- 传输加扰控制（transport scrambling control）：2bit，指示传送包有效净荷的加扰方式；

- 自适应控制字段（adaption field control）：2bit，表示传送流包首部是否跟随有调整字段和/或有效净荷。如下

| 调整字段值 | 描述                               |
| ---------- | ---------------------------------- |
| 00         | 保留                               |
| 01         | 没有调整字段，仅含有184B有效净荷   |
| 10         | 没有有效净荷，仅含有183B的调整字段 |
| 11         | 0~182B调整字段后为有效净荷         |

  连续计数器（continuity counter）：4bit，随具有相同PID的包增加而增加。

### 2.2 TS流调整字段

在MPEG-2 TS中，为了传送打包后长度不足188B的不完整TS，以及在系统层插入参考时钟（Program clock reference，PCR），需要再TS包中插入可变长字节的调整字段，一般在视频帧的TS包调整字段中，每隔一定传输时间，将传送系统时钟27MHz的一个抽样值个抽样机，作为解码器解码时的PCR，每100ms至少传输一次，pCR数值的含义是，解码器在读完这个抽样值得最后一个字节时，解码器的本地时钟应该处于的状态，PCR通常不直接改变解码器本地时钟，而是作为参考基准来调整本地时钟，使之与PCR趋于一致。

### 2.3 PES包格式

![2](./pic_TS/2.png)

PES包的第五个字节标识一整个包的长度，一般来说，一个PES包包含一阵图像，获取包长度Len，当接收到Len个字节后，将接收的字节组成一个block，放入FIFO中，等待解码线程进行解码，DTS和PTS包也在PES包中传送。

## 3. 节目专用信息（Program Special Information，PSI）

PAT和PMT都是PSI之一，TS包携带两类信息，一类是PES包，另一类是PSI，由传送的PID进行识别。

### 3.1 节目关联表（Program Association Table，PAT）

PAT定义了TS中的所有节目，PID为0x0000，是PSI的信息节点，格式如下

![](./pic_TS/3.png)

程序在解析到N环部分的时候，会读取并保存节目列表及PID，PAT信息在TS流中隔一段时间就会传送，接收机在接收时，PAT表中列出所有节目和PID值，这个PID值表征该节目对应的PMT表的PID值，PAT与PMT表的关系如下

![](./pic_TS/4.png)

### 3.2 节目映射表（Program Map Table，PMT）

PMT提供一路节目包含的所有原始码流的PID映射表，其格式如下

![](./pic_TS/5.png)

程序读取N环的时候会读取节目所有的码流列表和其PID，解析的时候可以根据PID分离，N环描述符如下，节目参考时钟PCR的PID和视频的PID是相等的。由PAT得出所有节目列表，选定收看的节目后，筛选出所有对应节目PID的TS包，就可以得到该节目的所有码流的PID映射表，这样接收机就可以只接收PID等于节目码流的TS包

![](./pic_TS/6.png)

## 4. 视音频同步

后面再看[  链接  ](https://www.cnblogs.com/jiayayao/p/6832614.html)啦 

---

# 链接

[架构设计：生产者/消费者模式]( http://www.uml.org.cn/zjjs/200904161.asp)

好像找到了上面博客的原作者：program-think，地址就不写全啦。

---

# SVN命令行

调出命令帮助
```bash
svn help
```

将项目检出到本地，会跳出认证信息
```bash
svn checkout [path]
```

创建仓库，这个创建仓库的操作只应该出现在服务端，也就是说SVN这个东西，只应该是服务端为中心，其他用户先从服务端拉，然后改之后再提交上去，其实git似乎也差不多，但有些逻辑还是不大一样
```bash
svnadmin create [path]
```

一般操作流程

首先`svn checkout [path]`把项目拉下来，然后修改之后`svn commit –m ""`提交上去，如果有新增文件的话，需要`svn add [filename]`，然后再进行`commit`。如果要删除文件，再本地直接删是没用的，`commit`也没用，一个`svn update`就回来了。如果要删除文件的话，需要先`svn delete [filename]`，然后`commit`，就两边都没啦

然后`svn info`，是一些仓库的信息，`svn ls`可以看仓库的目录

---

# rsync配置

参考文档**新服务器-DM6467 平台开发说明.doc**

安装rsyncd这个软件**cwRsyncServer_4.1.0_Installer.exe**

配置安装目录下的rsyncd.conf

配置文件的头如下，前4项不考虑，后两项需要有，不然会报错
```
use chroot = false
strict modes = false
hosts allow = *
log file = rsyncd.log
uid = 0
gid = 0
```

配置项目，最主要就是项目名称和项目路径两个，关于项目路径，我觉得是软件实现的时候用到了cygwin，因此路径的写法会稍有不同。
```
[test]   # 项目名称
path = /cygdrive/d/work_dir/test_180724_win   # 项目路径
read only = false
transfer logging = yes
```

配置好之后需要任务管理器里面启动服务RsyncServer，不确定开机是否自启

在Linux下，使用命令
```bash
rsync -zvalR --delete 192.165.53.45::test  /home/jiangdongchao/work_dir
```

`--delete`参数是用来删除目标文件夹中多出来的文件的

后面的ip是Windows的ip，双冒号后面是要同步的项目名称，再后面的路径就是目标目录
```bsah
receiving incremental file list
deleting test_180724/
./
test.txt
```

类似上面这样，成功

---

# nalu type

## H.264

| nalu type                                                          || description  |
| ------------------------------------------------------------ | ---|---- |
| 0     ||没有定义|
|1-23 |  |NAL单元  单个 NAL 单元包|
|1  | NALU_TYPE_SLICE | 不分区，非IDR图像的片  |
|2    |NALU_TYPE_DPA| 片分区A  |
|3    |NALU_TYPE_DPB| 片分区B |
|4  | NALU_TYPE_DPC | 片分区C  |
|5  | NALU_TYPE_IDR |  IDR图像中的片  |
|6| NALU_TYPE_SEI |  补充增强信息单元（SEI）  |
|7 | NALU_TYPE_SPS |  SPS |
|8 | NALU_TYPE_PPS |  PPS  |
|9  |NALU_TYPE_AUD|  分界符  |
|10 |NALU_TYPE_EOSEQ|   序列结束  |
|11| NALU_TYPE_EOSTREAM | 码流结束 |
|12 |NALU_TYPE_FILL|   填充  |
|13-23| |保留 |

## H.265

| nalu type |                | description |
| --------- | -------------- | ----------- |
| 0         | NAL_TRAIL_N    |             |
| 1         | NAL_TRAIL_R    |             |
| 2         | NAL_TSA_N      |             |
| 3         | NAL_TSA_R      |             |
| 4         | NAL_STSA_N     |             |
| 5         | NAL_STSA_R     |             |
| 6         | NAL_RADL_N     |             |
| 7         | NAL_RADL_R     |             |
| 8         | NAL_RASL_N     |             |
| 9         | NAL_RASL_R     |             |
| 16        | NAL_BLA_W_LP   |             |
| 17        | NAL_BLA_W_RADL |             |
| 18        | NAL_BLA_N_LP   |             |
| 19        | NAL_IDR_W_RADL |             |
| 20        | NAL_IDR_N_LP   |             |
| 21        | NAL_CRA_NUT    |             |
| 32        | NAL_VPS        |             |
| 33        | NAL_SPS        |             |
| 34        | NAL_PPS        |             |
| 35        | NAL_AUD        |             |
| 36        | NAL_EOS_NUT    |             |
| 37        | NAL_EOB_NUT    |             |
| 38        | NAL_FD_NUT     |             |
| 39        | NAL_SEI_PREFIX |             |
| 40        | NAL_SEI_SUFFIX |             |

# 从Wireshark中拿rtp内容

ffmpeg发送

```sh
ffmpeg -re -stream_loop -1 -f s16be -ar 16K -i pink_16k_-18dBFS.pcm -acodec pcm_mulaw -f rtp -payload_type 0 rtp://192.165.152.241:4002
```



首先通过wireshark抓包，存到pcap文件，然后通过tshark

```shell
tshark -nr xxx.pcap -2R rtp -T fields -e rtp.payload > in.ascii
```

如果保存的pcap没有经过过滤，那么可以在-R后面进行端口及ip等过滤，需要的话再去找吧

```sh
tshark -nr all_11061443.pcap -2R "rtp and ip.dst == 192.165.152.241 and !icmp" -T fields -e rtp.payload > test.asc
```

-2R选项后面写在wireshark中进行过滤的时候一样的命令就行了

后面为什么用重定向？因为写文件的-w只会带上一个头，这个头由-F选项控制，然后再把-R过滤出来的写到文件里，没啥用，我实在是找了好久，找不到能用的选项

而重定向也有个问题，就是它写出来的是文本文件

所以用个py脚本来转换

```python
import binascii

prefix = "PC2D"

with open(prefix + ".g711mu", "wb") as hex_fd:
    with open(prefix + ".ascii", "r") as ascii_fd:
        lines = ascii_fd.readlines()
        for line in lines:
            #print(len(line[0:-1]))
            hex_fd.write(binascii.a2b_hex(line[0:-1]))
```

出来的是G711u编码的文件，可以使用

```sh
ffplay -f s16le -ar 8K -i out.711mu -acodec pcm_mulaw
```

进行播放

 可以使用

```sh
ffmpeg -f s16le -ar 8K -acodec pcm_mulaw -i PC2D.g711mu PC2D.wav
```

转换为wav（类似的，也可以转为pcm，但是没有必要）

# Wireshark抓到的包变成了双倍

主要是发出去的包在wireshark中全都变成了两个（实际上还是只发出去了一个），重启电脑不好使，删掉了其他ip不好使，重装了wireshark，也不好使

使用

```sh
netsh winsock reset catalog
netsh int ip reset reset.log
```

重启电脑，ip需要重新设置，然后就好了

# 离线实现远程调试（主机和虚拟机）

- 虚拟机网络配置为桥接（可能带来的问题：wireshark抓包时抓到发出的包变成两倍），给虚拟机配置IP与主机网段一致
- 安装openssh：**Windows的openssh** https://www.mls-software.com/opensshd.html
- 确认通过主机的openssh可以连接虚拟机（可能出现的问题，所有者或权限问题，可能需要修改openssh安装目录下面的一个.ssh/config文件）
- ok，打开主机的vscode，上网机下载然后离线安装remote development（可能需要remote ssh）
- 打开左面的小电视图标，ssh targets点加号，然后ssh xxx@xxx.xxx.xxx.xxx然后它可能就卡住了，因为它要下载一个东西但是下载不下来
- 此时打开虚拟机，查看~/.vscode-server/bin/xxxxxxxxxxxxxxxxxxxxxx，后面这串乱码很重要，到https://update.code.visualstudio.com/commit:xxxxxxxxxxxxxxxx/server-linux-x64/stable，中间就是那串乱码，这个是可以下得下来的，https://github.com/cdr/code-server/releases这里面也有，因为会重定向到亚马逊，就很难下载下来
- 好，下载完，扔到虚拟机的~/.vscode-server/bin/xxxxxxxxxxxxxxxxxxxxxx/里面，此时注意vscode中的打印，它有可能在等待一个锁，但由于我们是离线安装的，所以touch那个锁（名字我记不得了）其实这里直接手动解压出来好像也成
- 基本上就差不多了，这里我还遇到一个问题，就是它出现一个wget 127.0.0.1:xxx失败，重启了虚拟机就好了
- 关于插件，由于本机与远程机器不一定是一个平台，所以插件是要重新安装的，按理说通过网络下载就行了，但是我们还是要离线，这就要求在虚拟机上有一个本地的vscode，然后将.vscode/extensions中的东西（即虚拟机本地插件安装出来的东西）cp到.vscode-server/extensions中，重新连接远程设备即可

我真是熟悉各种东西的离线安装了

# speex交叉编译

```sh
./configure --host=arm-linux CC=aarch64-linux-gnu-gcc CXX=aarch64-linux-gnu-g++ prefix=/home/ippfcox/ -enable-fixed-point
```

编译出来的东西在libspeexdsp里面的.lib中，外面的testecho等都是sh脚本，我真是不太能理解干嘛这么干。。。

# webrtc交叉编译

```sh
proxychains ./build/linux/sysroot_scripts/install-sysroot.py --arch=arm64
gn gen out/arm64/ --args='target_os="linux" target_cpu="arm64"'
```

可能有些东西需要下载

# 虚拟机中的ubuntu扩容

参考：https://www.cnblogs.com/atuotuo/p/8934370.html

关闭虚拟机状态下，虚拟机→设置→硬盘→扩展，扩展到你想要的大小

开机，此时只是硬盘变大，ubuntu系统的空间还没有变大

于是先安装gparted

```sh
sudo gparted
```

一般会有一个主分区sda1，一个扩展分区sda2，扩展分区下面有个swap为sda5，删除sda5然后删除sda2，如果发现有个钥匙标记删除不了，右键swap点击swapoff，然后就可以删除了

右键sda1点击resize/move将其扩展到想要的大小，记得给swap留空间

然后剩下的创建一个扩展分区，在扩展分区中再创建swap即可（其实就是跟之前的结构一样），扩容之后重启出现一个挂在错误，要等待，其实是swap分区的uuid变了导致的，此时打开gparted可以看到swap分区的状态是not active，我们复制其uuid，放到/etc/fstab中对应的位置，重启，不再出错

# 交叉编译带有依赖的东西

编译speex-1.2.0beta3的时候遇到了ogg/ogg.h没有的问题，于是想办法找到了，并且下载了源码，此时编完输出的其实还是没法用，有两个办法，一个是在Makefile中直接link到我们编译ogg输出的include和lib中，这个不是不行，但是太不好了，尤其修改一个已发布的东西会很困难，优雅的解决方案：

```sh
aarch64-linux-gnu-gcc -print-sysroot
```

输出一个xxx/libc，我们将ogg编译的输出目录设置为xxx/libc/usr：

```sh
./configure --host=arm-linux CC=aarch64-linux-gnu-gcc CXX=aarch64-linux-gnu-g++ prefix=/home/ippfcox/aarch64_toolchain/gcc-linaro-7.4.1-2019.02-x86_64_aarch64-linux-gnu/bin/../aarch64-linux-gnu/libc/usr/
```

这样在make install的时候就会将编好的头和库放到那里，编译器自己就可以找到了，尝试编译，果然正常了

# 免密码登录ssh

网上的教程太注意没必要的细节，反而最重要的东西总是说不清楚

如果我们现在有机器A，想登陆服务器S，需要的操作是在A上产生ssh公钥私钥对

```bash
ssh-keygen -t rsa
```

然后连续回车，此时就在/home/xxx/.ssh/下面生成了id_rsa（私钥），id_rsa.pub（公钥）

然后将公钥想办法弄到服务器上，更名为authorized_keys（或者把里面的内容粘贴进去，一样的）

注意权限问题，.ssh目录应为700，authorized_keys的权限应为600

再登录就不再需要密码了

重点在于生成秘钥对的应该是我们自己拥有的设备，而非想登录的设备

# 定义数组大小不能超过栈空间大小

# 1604-1804

误操作升级了1804，跟着https://askubuntu.com/questions/140246/how-do-i-resolve-unmet-dependencies-after-adding-a-ppa基本还挺好用的

出现过一个说找不到https的问题，看了一下/usr/lib/apt/methods，里面确实只有http没有https，修改/etc/apt/sources.list，将所有条目修改为http，然后

```bash
sudo apt-get install apt-transport-https
```

然后再恢复sources.list，继续其他操作