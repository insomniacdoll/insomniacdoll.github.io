---
layout: post
title: "cubietruck安装Cubieez/Lubuntu笔记"
description: "cubietruck安装Cubieez/Lubuntu笔记"
tags: [嵌入式, ubuntu, cubietruck, cubieez]
comments: false
---

工具：

{% highlight sh %}
PhoneixSuite(Windows)
SecureCRT
{% endhighlight %}

固件下载链接：

{% highlight sh %}
http://dl.cubieboard.org/software/a20-cubietruck/
{% endhighlight %}

按照需求下载对应的固件，解压后使用PhoneixSuite刷入cubietruck即可。

罗技无线键鼠没法使用的解决办法：

因为内核中的驱动支持HIDRAW默认是关闭的，重新编译一次内核即可。

烧录好的系统，ssh登录进去，到/proc/目录下找到config.gz，是当前内核的配置文件，拷贝出来到内核源码目录，解压重命名为.config。这么做是为了最大兼容当前运行内核。

内核源码的github地址：
{% highlight sh %}
https://github.com/linux-sunxi/linux-sunxi
{% endhighlight %}
或者可以去固件目录下载。

当前交叉编译系统的版本必须是ubuntu 12.04 lts，保证编译器版本为gcc 4.6，高版本编译的内核无法在开发板上运行。

交叉编译环境准备：
{% highlight sh %}
sudo apt-get install build-essential u-boot-tools uboot-mkimage
sudo apt-get install gcc-arm-linux-gnueabihf gcc-arm-linux-gnueabi
sudo apt-get install libusb-1.0-0 libusb-1.0-0-dev git wget fakeroot kernel-package zlib1g-dev
sudo apt-get install libncurses5-dev
{% endhighlight %}
其他包按照提示安装即可。

编译选项打开HIDRAW：
{% highlight sh %}
make ARCH=arm menuconfig
{% endhighlight %}
编译：
{% highlight sh %}
make -j4 ARCH=arm CROSS_COMPILE=arm-none-linux-gnueabi- uImage modules
{% endhighlight %}
安装modules：
{% highlight sh %}
make STRIP=arm-none-linux-gnueabi-strip INSTALL_MOD_PATH=output ARCH=arm INSTALL_MOD_STRIP=1 modules_install
{% endhighlight %}
然后把output目录下的modules替换到开发板的/lib/modules下，/arch/arm/boot/uImage替换到开发板/boot目录下，内核就升级完成了，重启即可。

-EOF-