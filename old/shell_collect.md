格式化成vfat

```bash
mkfs.vfat -F 32 -n boot /dev/mmcblk0p2
```

挂载

```bash
mount /dev/mmcblk0p1 /mnt
```

V4L2采集

```bash
v4l2-ctl -d /dev/video0 --set-fmt-video width=1920,height=1080,pixelformat=NV12 --stream-mmap=3 --stream-count=3 --stream-to=./test.yuv
```

fat文件系统不支持软链接

格式化成ext4，注意看`mke2fs -h

```bash
./mke2fs -t ext4 -L boot /dev/mmcblk0p2
```

有时在secureCRT中，sqlite3的那个`>`无法使用backspace，这里

```bash
stty erase ^H
```

查看已挂载的文件系统的格式

```bash
df -T
```

没有rc.local添加自启动，要执行的脚本放在/etc/init.d中，建立/etc/rc2.d，/etc/rc3.d，/etc/rc5.d中指向脚本的软链接，注意软链接的名字一般以K或者S开头，对于S开头的，启动时会传一个start进去，K开头的会传递一个stop进去，然后后面跟的数字好像与启动顺序有关系，但是这里没有确认。

参考链接：https://blog.csdn.net/yang0511hui/article/details/79866289

---

```bash
top -p $(pgrep xxx.out)
top -p `gprep xxx.out`
```

这两种方式的效果似乎是类似的都是先计算里面的值，然后替换到外面。

---

alsa音频

采集

```bash
./arecord -Dhw:0,1 -t raw -f S16_LE -r 48000 -c 2 test.pcm
```

播放

```bash
./aplay -Dhw:0,0 -t raw -f S16_LE -r 48000 -c 2 test.pcm
```

采集直接播放

```bsah
./arecord -Dhw:0,1 -f S16_LE -r 48000 -c 2|./aplay -Dhw:0,0 -f S16_LE -r 48000 -c 2
```

经过采样的

```bash
./arecord -Dmca1_cap -f S16_LE -r 48000 -c 2|./aplay -Dmca0_play -f S16_LE -r 48000 -c 2    
```

查看系统支持的字体

```sh
fc-list
fc-list :lang=zh
```

ffmpeg的一些操作

```sh
ffmpeg -re -stream_loop -1 -i i.wav -f rtp -payload_type 0 rtp://192.165.53.45:9002 #发音频流
ffmpeg -re -stream_loop -1 -f s16be -ar 16K -i pink_16k_-18dBFS.pcm -acodec pcm_mulaw -f rtp rtp://192.165.152.241:4002
ffmpeg -f s16le -ar 8K -acodec pcm_mulaw -i a.g711mu o.wav # 大概是这样的
ffmpeg -i in.wav -ar 44100 out.wav # 重采样
ffmpeg.exe -i pc2d_cssln.wav -f s16le -ar 8000 -acodec pcm_s16le pc2d.pcm # wav转pcm
ffmpeg -re -stream_loop -1 -i atrium.avi  -vcodec h264 -f rtp -payload_type 98 rtp://127.0.0.1:9000

```

