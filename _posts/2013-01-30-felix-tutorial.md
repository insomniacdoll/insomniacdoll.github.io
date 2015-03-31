---
layout: post
title: "Felix入门与实践"
description: "Felix入门与实践"
tags: [felix, OSGi, java, service]
comments: false
---

首先，我假设你已经了解OSGi相关的一些概念，如果没有，可以看看官方的文档。

我们从创建一个监听OSGi Service事件的bundle开始。这个小例子并不会包含太多的东西，只是打印出注册以及未注册的Service的详细信息而已。下一篇我才会开始介绍实现了Service的bundle，这次只是通过一个范例来加深对于创建bundle基础知识的理解。

OSGi框架通过与bundle唯一对应的BundleContext来访问一个bundle，而得到BundleContext必须实现BundleActivator接口。这个接口有两个方法，start()和stop()。当bundle的被start或者stop的时候，BundleContext将作为参数传入这两个方法，然后这两个方法才会被调用。下面的源代码是一个bundle，当然它实现了BundleContext接口，同时这个bundle添加了自身作为Service事件的监听器。

{% highlight java %}
package tutorial.example1;

import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.osgi.framework.ServiceListener;
import org.osgi.framework.ServiceEvent;

/**
 * This class implements a simple bundle that utilizes the OSGi
 * framework's event mechanism to listen for service events. Upon
 * receiving a service event, it prints out the event's details.
**/
public class Activator implements BundleActivator, ServiceListener
{
    /**
     * Implements BundleActivator.start(). Prints
     * a message and adds itself to the bundle context as a service
     * listener.
     * @param context the framework context for the bundle.
    **/
    public void start(BundleContext context)
    {
        System.out.println("Starting to listen for service events.");
        context.addServiceListener(this);
    }

    /**
     * Implements BundleActivator.stop(). Prints
     * a message and removes itself from the bundle context as a
     * service listener.
     * @param context the framework context for the bundle.
    **/
    public void stop(BundleContext context)
    {
        context.removeServiceListener(this);
        System.out.println("Stopped listening for service events.");

        // Note: It is not required that we remove the listener here,
        // since the framework will do it automatically anyway.
    }

    /**
     * Implements ServiceListener.serviceChanged().
     * Prints the details of any service event from the framework.
     * @param event the fired service event.
    **/
    public void serviceChanged(ServiceEvent event)
    {
        String[] objectClass = (String[])
            event.getServiceReference().getProperty("objectClass");

        if (event.getType() == ServiceEvent.REGISTERED)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " registered.");
        }
        else if (event.getType() == ServiceEvent.UNREGISTERING)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " unregistered.");
        }
        else if (event.getType() == ServiceEvent.MODIFIED)
        {
            System.out.println(
                "Ex1: Service of type " + objectClass[0] + " modified.");
        }
    }
}
{% endhighlight %}

完成bundle的Java代码之后还不够，我们还需要定义一个包含了meta-data信息的manifest文件，这样OSGi框架才能“操作”这个bundle。manifest文件要和编译好的Java Class打包成一个Jar，而这个Jar包实际上就是一个bundle。我们接下来创建一个名为manifest.mf的文件，内容如下：

{% highlight html %}
Bundle-Name: Service listener example
Bundle-Description: A bundle that displays messages at startup and when service events occur
Bundle-Vendor: Apache Felix
Bundle-Version: 1.0.0
Bundle-Activator: tutorial.example1.Activator
Import-Package: org.osgi.framework
{% endhighlight %}

上面的meta-data信息大部分只是为了维护方便，实际上只有Bundle-Activator属性以及Import-Package属性是OSGi框架必须的。Bundle-Activator属性为框架指明了实现了BundleActivator接口的类。当OSGi框架start某个bundle的时候，将创建一个该属性指定的类的实例，同时调用该实例的start()方法；同样框架stop该bundle的时候，实例的stop()方法将被调用。Import-Package属性指定了这个bundle所依赖的外部类。所有的bundle必须导入org.osgi.framework这个类，因为它包含了OSGi类的核心定义。所有的包依赖关系都会由OSGi框架来验证以及处理。（注意，manifest文件最后一行之后必须有一个回车符，否则最后一行的内容将会被框架忽略。）
 
然后就可以开始编译源代码了，我们需要把felix.jar添加到classpath中（Felix的bin目录中可以找到这个jar包），然后在命令行中运行：

{% highlight sh %}
javac -d c:\classes *.java
{% endhighlight %}

上面的命令将会编译classes目录下所有的源代码，并将class二进制字节码输出到指定的子文件夹tutorial\example1中。编译完成之后，就可以把class文件和之前写好的bundle的manifest文件打包成jar。接着在命令行中运行：

{% highlight sh %}
jar cfm example1.jar manifest.mf -C c:\classes tutorial\example1
{% endhighlight %}  

一个打包好的bundle就到手了，用Felix的Shell就可以安装并运行这个bundle，比如：

{% highlight sh %}
start file:/c:/tutorial/example1.jar 
{% endhighlight %}

上面的命令会自动安装并start这个bundle。当然我们也可以手动安装并且运行。Felix的Shell有install和start两个命令，分别运行一下就OK。另外stop命令可以stop指定的bundle，而shutdown则是关闭整个Felix环境。

-EOF-