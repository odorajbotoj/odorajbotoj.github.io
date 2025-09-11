---
layout: post
title: 如何优雅地制作 NOI Linux 2 裁切版
category: linux
---

# 如何优雅地制作 NOI Linux 2 裁切版

## 导言

打OI的时候被蒋炎岩博士写的JSOI系统震撼到了(膜)。老早就有这个想法了，正好蒋炎岩博士把自己的操作系统原理课分享到了哔哩哔哩上，又在知乎上贴出了自己[制作JSOI-Linux的历程](https://zhuanlan.zhihu.com/p/619237809)。我简单地学习了下，准备自己也搭建一个类似的系统。

## 0.准备

我的建议是，准备一个趁手的Linux就行。我使用的是在我家老爷机上安装的NOI-Linux 2.0。

*没错，我就是这么热爱NOI*

## 1.准备mini系统里需要的包

蒋炎岩博士已经贴心地帮我们列好了。（见他的知乎专栏）我通过解包 `ubuntu-noi-v2.0.iso` 获取了其中的一部分包，但还有一部分包找不到完全一样名称的文件。没办法，只能妥协。最终我准备的包列表如下：

```text
binutils_2.34-6ubuntu1_amd64.deb
binutils-common_2.34-6ubuntu1_amd64.deb
binutils-x86-64-linux-gnu_2.34-6ubuntu1_amd64.deb
cpp-9_9.3.0-10ubuntu2_amd64.deb
g++-9_9.3.0-10ubuntu2_amd64.deb
gcc-9_9.3.0-10ubuntu2_amd64.deb
libasan5_9.3.0-10ubuntu2_amd64.deb
libbinutils_2.34-6ubuntu1_amd64.deb
libc6_2.31-0ubuntu9_amd64.deb
libc6-dev_2.31-0ubuntu9_amd64.deb
libc-dev-bin_2.31-0ubuntu9_amd64.deb
libctf0_2.34-6ubuntu1_amd64.deb
libctf-nobfd0_2.34-6ubuntu1_amd64.deb
libexpat1_2.2.9-1ubuntu0.6_amd64.deb
libgcc1_10.3.0-1ubuntu1~20.04_amd64.deb
libgcc-9-dev_9.3.0-10ubuntu2_amd64.deb
libgmp10_6.2.0+dfsg-4ubuntu0.1_amd64.deb
libisl22_0.22.1-1_amd64.deb
libmpc3_1.1.0-1_amd64.deb
libmpfr6_4.0.2-1_amd64.deb
libstdc++6_10.3.0-1ubuntu1~20.04_amd64.deb
libstdc++-9-dev_9.3.0-10ubuntu2_amd64.deb
linux-libc-dev_5.4.0-148.165_amd64.deb
zlib1g_1.2.11.dfsg-2ubuntu1_amd64.deb
```

基本差不多，版本号没大问题就应该没啥大问题了。

接下来，该考虑下如何把这些包装进系统的filesystem了。

我写了个简单的Python3脚本，来批处理这些deb包：

```python
import os

fs = os.listdir()
for f in fs:
    if f.endswith(".deb"):
        os.system("dpkg -X " + f + " ./rtfs/")

input("\n\nfinished.")
```

包里的内容会被解到rtfs目录下。

## 2.装载Busybox

busybox，linux下的瑞士军刀！

通过busybox，我们可以便捷地使用一些诸如 `sh` ， `vi` ， `cd` ， `ls` 等常用命令。

去[busybox的下载站](https://busybox.net/downloads/)下载源码。我这里使用的是 `busybox-1.36.0.tar.bz2` 。解压缩。

```bash
tar -jxvf busybox-1.36.0.tar.bz2
```

接下来准备编译。

```bash
make menuconfig
```

日常报错： `curses.h: No such file or directory` 。原因是缺失ncurses库。我们手动安装下。

```bash
sudo apt-get install libncurses5-dev libncursesw5-dev
```

继续编译。我勾选了支持Unicode的相关选项。还有最重要的，静态编译。 `Settings--Build Options` 下的 `Build static binary (no shared libs)` 必选！！！

我的CPU（奔腾E5600，够老了吧）是双核的。于是我开两线程编译。

```bash
make -j 2
```

接下来就是等待，我顺便去洗了个碗。

*（其实编译没那么慢的，在我去洗碗之前就好了）*

*（当然，碗还是要洗的）*

## 3.准备镜像文件系统

*后面的操作以我自己的目录结构为准，大家可自行调整！！！*

粗略计算了下256MB大概够了。（事实上只用了200MB不到）

```bash
dd if=/dev/zero of=rootfs.img bs=1M count=256
```

然后格式化成ext4文件系统。

```bash
mkfs.ext4 rootfs.img
```

接下来挂载到fs找个目录下：

```bash
mkdir fs
sudo mount -t ext4 -o loop rootfs.img ./fs/
```

把busybox装进去。

```bash
cd busybox/busybox-1.36.0/
sudo make install CONFIG_PREFIX=../../fs/
```

新建相关文件夹，拷贝配置文件。

```bash
cd ../../fs/
sudo mkdir proc dev etc home mnt
sudo cp ../busybox/busybox-1.36.0/examples/bootfloppy/etc/* etc/ -r
```

注意 `etc/init.d/rsS` 文件最后要加上 `/bin/mount -o remount,rw /` 来使最终的文件系统可写。

所以我目前的目录结构是这样的：

```text
├── busybox
│   └── busybox-1.36.0
│       └── (something else)
├── deb
│   └── rtfs
│       └── (something else)
├── fs
│   ├── bin
│   ├── dev
│   ├── etc
│   │   └── init.d
│   ├── home
│   ├── lost+found [error opening dir]
│   ├── mnt
│   ├── proc
│   ├── sbin
│   └── usr
│       ├── bin
│       └── sbin
```

然后把之前解好的包拷贝过来。

```bash
sudo cp ../deb/rtfs/*. -r 

```

赋权，然后卸载这个盘：

```bash
cd ../
sudo chmod +777 -R fs/
sudo umount fs
```

## 4.准备内核

按理来说内核也是要我自己编译的，但我不知道该下载哪份源码来编译。于是我索性去Ubuntu的包网站下载了 `linux-image-5.4.0-42-generic_5.4.0-42.46_amd64.deb` 这个包，用之前的方法解开，在boot下找到了 `vmlinuz-5.4.0-42-generic` 这个内核镜像。

能用。

## 5.启动准备

我新建了一个文件夹，把rootfs和kernel全放了进去。

安装一下qemu。我这里只是测试用，最后的使用环境应该是win7x86。具体安装过程就不演示了。

记得编译时候要打开网卡支持。`--enable-slirp`

最后尝试启动。

```bash
qemu-system-x86_64 \
        -kernel vmlinuz-5.4.0-42-generic \
        -hda rootfs.img \
        -append "root=/dev/sda console=ttyS0" \
        -nographic \
```

闪过一些熟悉的启动日志后，我们进入了命令行界面。（欣喜）

输入一些指令看看。比如最重要的 `g++`。

诶，怎么没反应？（作者一度开始怀疑人生）

经过试验排查，原来我们需要输入 `g++-9` 才行。

```bash
g++-9 --version
```

输出

```text
g++-9 (Ubuntu 9.3.0-10ubuntu2) 9.3.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

看来是真的成功了（大喜）。

最后，使用 `poweroff` 关闭虚拟机。

## 6.其他

+ 感谢蒋炎岩博士！（超大声）

+ 我学了Golang，并且重新实现JSOI Web Client的项目已经在路上了。

+ 求助！我不知道如何配置qemu的网络选项（悲）

+ 2023-05-28 21:55 总第四次编辑
