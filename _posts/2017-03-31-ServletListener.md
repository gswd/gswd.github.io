---
layout: post
title:  "Servlet监听器(Listener)"
categories: servlet
tags:  Servlet监听器
excerpt: 当web应用在web容器中运行时，Web应用内部会不断的发生各种事件：如Web应用被启动、Web应用停止、用户请求到达等，通常这些Web事件对开发者来说是透明的。ServletAPI提供了大量的监听器来监听Web应用的内部事件，从而允许当Web内部事件发生时回调事件监听器内的方法。
---

* content
{:toc}

![](http://7xvdkv.com1.z0.glb.clouddn.com/img/servlet/servletListener.jpg)

## Listener介绍

> 当web应用在web容器中运行时，Web应用内部会不断的发生各种事件：如Web应用被启动、Web应用停止、
> 用户请求到达等，通常这些Web事件对开发者来说是透明的。
> ServletAPI提供了大量的监听器来监听Web应用的内部事件，
> 从而允许当Web内部事件发生时回调事件监听器内的方法。


## Servelt中常用的监听器

* ServletContextListener：监听web应用的启动和关闭。
* ServletContextAttributeListener：监听ServletContext范围内属性的改变。
* ServletRequestListener：监听用户请求。
* ServletRequestAttributeListener：监听ServletRequest范围内属性的改变。
* HttpSessionListener：监听用户session的开始和结束。
* HttpSessionAttributeListener：监听ServletSession范围内的属性改变。


## 实现和配置监听器

使用Listener只需要两个步骤：

1. 实现上面介绍的某种或多种监听器接口，实现对应方法，方法作用显而易见
2. 在web.xml中或者使用注解方式注册监听器。

```xml
<listener>
    <listener-class>com.hm707.demo.xxxListener</listener-class>
</listener>
```

```
@WebListener
public class OnlineListener implements ServletRequestListener{
    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        //用户请求到达、被初始化时调用该方法
        HttpServletRequest request = (HttpServletRequest)sre.getServletRequest();
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        //用户请求结束、被销毁时调用该方法
        HttpServletRequest request = (HttpServletRequest)sre.getServletRequest();
    }
}

```

## 监听器的实现原理

Servlet 中的监听器使用了和AWT相似的委托事件模型(DEM)，
具体实现和原理请看我的另一篇博客[观察者模式和委托事件模型](http://www.hm707.com/2017/03/28/Observer/)


## 应用监听器的例子

使用`ServletRequestListener`监控在线用户状态。

实现思路：

1. 实现一个`ServletRequestListener`用来负责监听每个用户的请求，当请求到达时，系统将请求的Session ID、用户名、用户IP、正在访问的资源、访问时间等都记录下来。
2. 启动一个后台线程，每隔一段事件就去检查上面的每条在线记录，如果在线记录的访问时间与当前的时间相差超过了指定值，将这条在线记录删除即可。该线程可以考虑使用`ServletContextListener`来启动。
