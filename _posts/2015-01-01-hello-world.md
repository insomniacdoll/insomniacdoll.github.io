---
layout: post
title: "Windows下编译OpenJDK7 笔记"
description: "Windows下编译OpenJDK7 笔记"
tags: [openjdk, compile, windows, make, cygwin]
image:
  feature: abstract-6.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
---

### 0. 系统需求

Windows XP SP3 / Windows 7 （强烈推荐用英文版系统进行构建，原因后详）

### 1. 所需软件

{% highlight sh %}
Cygwin

Sun JDK 1.6 u14以上

Microsoft DirecxX SDK

Microsoft Visual Studio C++ 2010 （正式版或者Express版均可）

Apache Ant 1.7.1以上

Freetype-2.3.5-1

openjdk-7-fcs-src-b147-27_jun_2011.zip

Vim（可选，推荐编辑器）
{% endhighlight %}

### 2.构建步骤

***以下涉及安装软件的步骤，都建议将软件安装到没有空格非中文的路径，且安装的目录名精简一些。***

(1) 首先安装Cygwin，所需的包如下所示：

需要注意的是，安装的make.exe为3.82版，导致编译不能成功，需要从cygwin网站上下载3.81版本并覆盖到bin目录。也可以下载源码之后自己编译一个（需要gcc包）。最后将cygwin的bin目录添加到PATH环境变量。
 
(2) 安装JDK6，版本需要u14以上（官方文档中称作Bootrap JDK），我采用的是u31版本。建立ALT_BOOTDIR和ALT_JDK_IMPORT_PATH环境变量，指向JDK的目录。注意不要设置JAVA_HOME和CLASSPATH环境变量。
 
(3) 安装Apache Ant，版本1.7.1以上，我采用的是1.8.2。建立ANT_HOME环境变量。
 
(4) 安装Freetype-2.3.5-1，建立ALT_FREETYPE_LIB_PATH 和ALT_FREETYPE_HEADERS_PATH环境变量，分别指向lib目录和include目录。将bin目录下的freetype6.dll和zlib1.dll复制到lib目录下。
 
(5) 安装DirectX SDK。强烈建议此步在安装Visual Stuido 2010 (Express)之前进行，否则有可能DirectX SDK无法正常安装成功。建立ALT_DXSDK_PATH环境变量，指向安装目录。
 
(6) 安装Microsoft Visual Studio C++ 2010 ，我采用的是Express版本（免费）。安装过程中会自动安装Windows SDK，且不能指定路径，在C:\Program Files\下。建立ALT_COMPILER_PATH 环境变量，指向VC++的bin目录。同时用以下命令将Windows SDK的目录转换为short路径（需要安装cygwin）：

{% highlight sh %}
Cygpath –s –m “C:\Program Files\Microsoft SDKs\Windows\v7.0A”
{% endhighlight %}

之后建立WINDOWSSDKDIR 环境变量，指向该short路径。

将WINDOWSSDKDIR 下的lib目录和VC++下的lib目录添加到LIB环境变量。

将WINDOWSSDKDIR 下的include目录和VC++下的include目录添加到INCLUDE环境变量。

去下载一个msvcr100.dll，并建立ALT_MSVCRNN_DLL_PATH 环境变量指向该文件所在的目录。如果你安装了Visual Studio C++ 2010，那么在你的系统中应该能找到这个文件，但是我用系统中自带的这个文件编译出的JDK不能正常运行，用下载的替换之后才可以。
 
(7) 解压openjdk-7-fcs-src-b147-27_jun_2011.zip，下载jaxp-1_4_5-unittests.zip、jaxp145_01.zip、jdk7-jaf-2010_08_19.zip、jdk7-jaxws2_2_4-b03-2011_05_27.zip四个文件放在openjdk\java\devtools\share\jdk7-drops目录下。并建立ALT_DROPS_DIR 环境变量指向该目录。

(8) 完善并检查环境变量，我写了一个批处理来进行环境变量设置：

{% highlight sh %}
SET VSINSTALLDIR=C:/PROGRA~2/MICROS~1.0
SET WINDOWSSDKDIR=D:/jdkBuild/sdk
SET ALT_BOOTDIR=D:/jdkBuild/JDK16~1.0_X
SET ALT_JDK_IMPORT_PATH=D:/jdkBuild/JDK16~1.0_X
SET ANT_HOME=D:/jdkBuild/APACHE~1.4
SET ALT_MSVCRNN_DLL_PATH=D:/jdkBuild/msvcr100
SET ALT_DXSDK_PATH=D:/jdkBuild/msdxsdk
SET ALT_FREETYPE_HEADERS_PATH=D:/jdkBuild/freetype/include
SET ALT_FREETYPE_LIB_PATH=D:/jdkBuild/freetype/lib
SET ALT_COMPILER_PATH=%VSINSTALLDIR%/VC/bin/
SET ALT_DROPS_DIR= D:/jdkBuild/openjdk/java/devtools/share/jdk7-drops
SET PATH=%WINDOWSSDKDIR%/Bin/NETFX4~1.0TO;%WINDOWSSDKDIR%/Bin;%VSINSTALLDIR%/VC/bin;%VSINSTALLDIR%/Common7/IDE;E:/cygwin/bin;%PATH%
SET INCLUDE=%VSINSTALLDIR%/VC/INCLUDE;D:/jdkBuild/sdk/INCLUDE
SET LIB=%VSINSTALLDIR%/VC/Lib;D:/jdkBuild/sdk/Lib
SET ALT_CC_VER = 16.00.30319.01
SET ALT_MSC_VER_RAW = 16.00.30319.01
{% endhighlight %}

其中带~的路径均为cygpath所转化所得的short路径。

建议路径中的分割符采用斜线（/）而不是反斜线（\），Windows资源管理器的默认显示路径是后者。如果采用反斜线，有可能在编译过程中某些脚本识别不出路径，从而出错。

同时cygwin的bin目录在PATH中的位置应该在系统的System32目录之前（脚本中会用到cygwin的find命令，否则会用windows的find命令导致出错）。而VC++的bin目录应该在cygwin之前（同样的原因，编译需要用到的是VC++中的link.exe）。
 

