---
layout: post
title: "Windows 8/8.1 上的MotionJoy和DS3配置"
description: "Windows 8/8.1 上的MotionJoy和DS3配置"
tags: [motionjoy, ps3, ds3]
comments: false
---

最近更新了Windows 8.1系统，原来使用正常的MotionJoy没法正常安装了，google了一圈解决办法，记录在这里。  
首先还是要到MotionJoy的官网上去下载最新版本的安装包，然后安装：  

{% highlight sh %}
http://motioninjoy.en.uptodown.com/  
{% endhighlight %}

由于MotionJoy的官网已经被功夫网和谐，而这个广告巨多的软件需要运行时联网，所以还需要到下面的链接去下载一个本地支持的补丁：  

http://bbs.3dmgame.com/forum.php?mod=attachment&aid=MzA4MjMyNXw1MTkxNjYxYnwxNDMwNDc1NjMxfDB8NDQ3OTYwMw%3D%3D

将解压得到的文件覆盖到MotionJoy安装目录的DS3子目录下。  
接下来，由于Windows 8.1对于驱动签名采取了比较严格的验证机制，直接安装DS3驱动是会失败的，所以需要使用管理员权限的命令行提示符执行以下命令将验证关闭：  

{% highlight sh %}
bcdedit -set loadoptions DISABLE_INTEGRITY_CHECKS  
bcdedit -set TESTSIGNING ON  
bcdedit /set testsigning on  
{% endhighlight %}

运行完毕后重新启动计算机。  
然后插入DS3手柄，进入MotionJoy安装目录的DS3子目录，以管理员权限运行DS3_Tool_Local.exe，点击Driver Manager，选中识别到的手柄，选择Install All即可安装驱动。  
安装完毕后点击Profiles，可以看到手柄已经被成功识别。  

最后把验证再重新开启：

{% highlight sh %}
bcdedit /set testsigning off  
bcdedit -set TESTSIGNING OFF  
{% endhighlight %}

