---
layout: post
title: "在My Cloud EX2上安装aria2和yaaw"
description: "在My Cloud EX2上安装aria2和yaaw"
tags: [嵌入式, kernel, my cloud, yaaw, aria2]
comments: false
---

### 0. 前言

之前一直在使用My Book Live，可以方便地使用自带的Debian系统进行应用扩展。配合最常用的aria2和yaaw，可以方便地进行百度网盘、115网盘、迅雷离线等网络存储的脱机下载。

但由于My Book Live在设计上是单盘不可更换硬盘的结构，所以存储满了之后想更换一台扩展性更好的NAS，使用习惯的原因促使我购买了My Book Live的后续产品My Cloud EX2。但没想到My Cloud EX2更换了内部的软件系统，底层的Linux操作系统不再基于Debian，而是一个没有包管理器的自行定制内核，shell使用了busybox进行代替。想要直接安装软件就困难了许多。

### 1. 前期准备

首先是需要将原来My Book Live的数据备份出来，最快的办法是使用暴力将其拆开，然后将硬盘挂载到主板的SATA接口上进行数据对拷。这样多出来的数据盘还可以继续利用。

拆机的攻略可以参照：

{% highlight sh %}
http://bbs.feng.com/read-htm-tid-5609125.html
{% endhighlight %}

另外，拆机完成，挂载硬盘到SATA接口上以后，Linux操作系统（比如Ubuntu）是不能直接识别大分区的，需要进行如下操作：

{% highlight sh %}
sudo apt-get install fuseext2 parted
sudo parted -l #找到磁盘标识，如/dev/sdb
sudo fuseext2 -o allow_other -o ro -o sync_read /dev/sdb4 /mnt/
{% endhighlight %}

其中allow_other参数很重要，否则会报没有权限的错误。

然后尽情备份即可。

### 2. 制作交叉编译工具链

由于没有包管理器，不可能直接直接安装软件，那么就只能自行使用交叉编译工具进行源码编译。

首先看一下系统的芯片类型：

{% highlight sh %}
# cat /proc/cpuinfo
Processor       : Marvell PJ4Bv7 Processor rev 1 (v7l)
BogoMIPS        : 1196.85
Features        : swp half thumb fastmult vfp edsp vfpv3 vfpv3d16 tls 
CPU implementer : 0x56
CPU architecture: 7
CPU variant     : 0x1
CPU part        : 0x581
CPU revision    : 1

Hardware        : Marvell Armada-370
Revision        : 0000
Serial          : 0000000000000000
{% endhighlight %}

芯片类型是Marvell Armada-370，也是定制的ARM芯片，那么可以使用crosstool-ng来进行交叉编译器构建。

首先在宿主机（Ubuntu 14.04 LTS）上面从源码编译安装crosstool-ng，下载地址：

{% highlight sh %}
http://crosstool-ng.org/download/crosstool-ng/
{% endhighlight %}

在编译安装crosstool-ng交叉工具之前，要安装与crosstool-ng工具相依赖的包文件：

{% highlight sh %}
# apt-get install gcc g++ gdb libncurses5-dev bison flex texinfo autoconf automake libtool patch gcj cvs cvsd gawk pkg-config
{% endhighlight %}

安装好以上依赖的包文件后，就可以编译安装crosstool-ng交叉编译工具了，在编译crosstool-ng交叉编译工具之前，要先配置crosstool-ng交叉编译工具的安装路径：

{% highlight sh %}
./configure --prefix=${PATH}
{% endhighlight %}

配置好安装路径后就要编译安装了：

{% highlight sh %}
make
make install
{% endhighlight %}

安装好crosstool-ng交叉编译工具后，就要用该工具构建自己的交叉编译链，构建自己的交叉编译工具链之前，要建立自己的工作目录，还有对应的源码文件，然后拷贝默认配置文件到工作目录，进入工作目录后：

{% highlight sh %}
cp ../crosstool-ng-1.9.3/samples/arm-unknown-linux-gnueabi /* ./   （以1.9.3为例）
{% endhighlight %}

修改文件名：

{% highlight sh %}
mv crosstool.config .config
{% endhighlight %}


进入menuconfig，开始修改配置：

{% highlight sh %}
../crosstool-ng-1.9.3_install/bin/ct-ng menuconfig
{% endhighlight %}

编译选项需要注意的选项如下：

{% highlight sh %}
Paths and misc options  --->Local tarballs directory  
  (/work/tools/Armada-370/crosstool-ng/src) Local tarballs directory   保存源码包路径
Paths and misc options  --->Prefix directory   
  (/work/tools/Armada-370/x-tools/${CT_TARGET}) Prefix directory  交叉编译器的安装路径
Target options --->Architecture level  
Target options --->Emit assembly for CPU
{% endhighlight %}

上面两个根据实际目标板来填写。每一项都可以进去看看，写的都很明确。

这里还有个问题，就是源码包的版本在配置选项中没有的情况，可以通过打开工作目录下的vim .config文件手动修改。

配置好路径和工具的各个版本后，直接运行：

{% highlight sh %}
../crosstool_install/bin/ct-ng build
{% endhighlight %}

进行build前面的配置。最后就是漫长的等待（中间需要下载源码），等待之后，自己的交叉编译工具链就完成了。

此外，其实群晖的某个NAS也使用了Armada-370这个芯片进行产品定制，且放出了开发工具包，可以直接下载来使用：

{% highlight sh %}
http://sourceforge.net/projects/dsgpl/files/DSM%204.3%20Tool%20Chains/Marvell%20armada%20370%20Linux%203.2.40/
{% endhighlight %}

但是**需要注意**，这个现成的工具包gcc编译器版本是4.6，并不支持包含c++11特性的代码编译。后续文章讲到使用NAS来构建PS3镜像流文件服务器的时候，就必须要使用自行构建的工具链。

### 3. 编译aria2c

现在交叉编译器到手，就可以来直接编译aria2c了，能够进行脱机下载功能的aria2c使用到的第三方包包括c-ares、expat和zlib。

为了自动下载源码和编译，我编写了一个shell脚本来完成：

{% highlight sh %}
#!/bin/sh

CWD=$(pwd)
NJOB=4

# Local folder where we install built binaries and libraries.
LOCAL_DIR=$(readlink -f ./local)
mkdir -p ${LOCAL_DIR}

# Cross-compiler tools
TOOL_BIN_DIR=/opt/arm-marvell-linux-gnueabi/bin
TOOL_CC=${TOOL_BIN_DIR}/arm-marvell-linux-gnueabi-gcc
TOOL_CXX=${TOOL_BIN_DIR}/arm-marvell-linux-gnueabi-g++

PATH=${TOOL_BIN_DIR}:$PATH

# zlib
rm -rf  zlib*
wget http://zlib.net/zlib-1.2.8.tar.gz ./
tar xzf zlib*.tar.gz
cd zlib*/
prefix=${LOCAL_DIR} CC=${TOOL_CC} CFLAGS="-O4" ./configure --static
make -j${NJOB}
make install

cd ${CWD}

# expat
rm -rf expat*
wget http://downloads.sourceforge.net/expat/2.1.0/expat-2.1.0.tar.gz ./
tar xzf expat*.tar.gz
cd expat*/
./configure \
    --host=arm-marvell-linux-gnueabi \
    --build=i686-linux-gnu \
    --enable-shared=no \
    --enable-static=yes \
    --prefix=${LOCAL_DIR}
make -j${NJOB}
make install

cd ${CWD}

# c-ares
rm -rf c-ares*
wget http://c-ares.haxx.se/download/c-ares-1.10.0.tar.gz ./
tar xzf c-ares*.tar.gz
cd c-ares*/
./configure \
    --host=arm-marvell-linux-gnueabi \
    --build=i686-linux-gnu \
    --enable-shared=no \
    --enable-static=yes \
    --prefix=${LOCAL_DIR}
make -j${NJOB}
make install

cd ${CWD}

# aria2
rm -rf aria2*
wget http://downloads.sourceforge.net/aria2/aria2-1.18.10.tar.xz ./
tar xJf aria2*.tar.xz
cd aria2*/
./configure \
    --host=arm-marvell-linux-gnueabi \
    --build=i686-linux-gnu \
    --disable-nls \
    --disable-ssl \
    --disable-epoll \
    --without-gnutls \
    --without-openssl \
    --without-sqlite3 \
    --without-libxml2 \
    --with-libexpat --with-libexpat-prefix=${LOCAL_DIR} \
    --with-libcares --with-libcares-prefix=${LOCAL_DIR} \
    --with-libz --with-libz-prefix=${LOCAL_DIR} \
    --prefix=${LOCAL_DIR} \
    --enable-static=yes \
    CXXFLAGS="-Os -g" \
    CFLAGS="-Os -g" \
    LDFLAGS="-L${LOCAL_DIR}/lib" \
    PKG_CONFIG_LIBDIR="${LOCAL_DIR}/lib/pkgconfig" \
    ARIA2_STATIC="yes"
make -j${NJOB}
make install
{% endhighlight %}

上面的shell脚本可以直接自动从网络上下载截止博文发出的最新版本的aria2及其依赖的源码，并进行编译（全部使用静态编译，避免运行环境找不到库的情况出现）。

需要注意的是，TOOL_BIN_DIR是交叉工具链的安装目录，需要按照实际情况进行修改。

***另外这个脚本应该在一个干净的目录进行执行。***

### 4. 安装aria2c和yaaw

将编译得到的aria2上传到My Cloud EX2的目录：

{% highlight sh %}
/mnt/HD/HD_a2/Nas_Prog
{% endhighlight %}

并建立一个配置文件，内容如下：

{% highlight sh %}
daemon=true  #以daemon方式启动
dir=/mnt/HD/HD_a2/Private/Download  #下载目录
disable-ipv6=true #禁用IPv6
enable-rpc=true #启用RPC，对于yaaw的配合使用非常重要
rpc-allow-origin-all=true 
rpc-listen-all=true
rpc-listen-port=6800 #监听端口
continue=true #开启断点续传
input-file=/mnt/HD/HD_a2/Nas_Prog/aria2/session/aria2.session #session保存地址，用于记录任务信息
save-session=/mnt/HD/HD_a2/Nas_Prog/aria2/session/aria2.session
file-allocation=falloc #文件分配方式
max-concurrent-downloads=1 #最大并行下载任务数量
max-connection-per-server=5 #每个服务器的并发连接数
min-split-size=10M #最大文件分割大小
split=10 #文件分割块数
log=/mnt/HD/HD_a2/Nas_Prog/aria2/log/aria2.log #日志路径
log-level=warn #日志级别
{% endhighlight %}

然后用命令启动：

{% highlight sh %}
cd /mnt/HD/HD_a2/Nas_Prog/aria2
./aria2c --conf-path=./conf/aria2.conf
{% endhighlight %}

然后安装yaaw，下载地址：

{% highlight sh %}
https://github.com/binux/yaaw
{% endhighlight %}

解压后也放到/mnt/HD/HD_a2/Nas_Prog目录，然后建立一个指向web服务器根目录的软连接：

{% highlight sh %}
ln -s /mnt/HD/HD_a2/Nas_Prog/yaaw /var/www/yaaw
{% endhighlight %}

这样就可以直接从NAS的IP访问到yaaw了。

### 5. 后续配置

使用上，可以配合百度云盘、115网盘的chrome插件进行。

Chrome Web Store的地址：

{% highlight sh %}
115网盘： https://chrome.google.com/webstore/detail/115exporter/ojafklbojgenkohhdgdjeaepnbjffdjf

百度网盘： https://chrome.google.com/webstore/detail/baiduexporter/mjaenbjdjmgolhoafkohbhhbaiedbkno
{% endhighlight %}

配置上，将json-rpc的地址指向【NAS的IP:6800】即可（默认情况），然后下载的时候选中需要下载的文件，按“导出到RPC”。

此外，【http://NAS的IP/yaaw】这个地址可以查看aria2的运行情况，同时也可以使用Web的方式来使用Aria2，当然下载都是My Cloud EX2的任务了。

-EOF-