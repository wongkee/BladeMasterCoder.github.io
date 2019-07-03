---
layout: post
title: 代理模式
category: 技术
tags: [Java基础] [设计模式]
description: 
---







> 序列化在 Java、JavaScript 中都有使用到，虽然会简单的使用但是对其中原理不是很了解，今天对序列化操作的原理，以及在开发中的使用场景进行总结,以加深自己的理解。





> 代理模式在日常开发以及Spring框架中得到广泛的使用，本文尝试分析代理模式的原理，并通过实现和分析静态代理、动态代理、远程代理来加深理解



### 设计模式
#### 代理模式
何为代理？
这里引用《HeadFirst设计模式》一书中对代理模式的解释：“你是一个白脸，提供很好且很友善的服务，但是你不希望每个人都叫你做事，所以找了黑脸控制对你的访问。这就是代理要做的：控制和管理访问”。所以我们我们可以理解---代理就是帮你做你不想做的事，但是你自己需要完成的功能，代理仍会调用你的引用来完成。 下面我们从静态代理、动态代理、远程代理三个方面解释代理模式的实现原理。 


#### 静态代理  
以一个例子解释静态代理的原理  
电影院播放电影，那么我们就可以将电影院理解为电影的代理，那么它是怎么实现的呢？

![](F:\wangqi\image\staticProxy.jpg)

1、首先实现一个 Movie接口,定义一个play（）方法用于放映电影 

```java
public interface Movie {
    void play();
}
```
2、创建 RealMovie类实现 Movie接口，表示真正播放的电影
```java
public class RealMovie implements Movie {
    @Override
    public void play() {
        System.out.println("您正在观看电影《肖申克的救赎》");
    }
}
```
3、创建 Cinema 类实现Movie接口，用于播放电影和插播广告

```java
public class Cinema implements  Movie {
   RealMovie movie;
    public Cinema(RealMovie movie){
        super();
        this.movie=movie;
    }
    @Override
    public void play() {
        guanggao(true);
        movie.play();
        guanggao(false);
    }
    public void guanggao(boolean isStart){
        if(isStart){
            System.out.println("电影马上开始了，爆米花、可乐、口香糖..");
        }else {
            System.out.println("电影马上结束，快回家吧！、、");
        }
    }
}
```
5、测试静态代理

```java
public class StaticProxyTest {
    public static  void main(String args[]){
        RealMovie realMovie=new RealMovie(); //电影的真正实现
        Movie movie = new Cinema(realMovie); //电影院实现了movie接口 并且添加自己特有的方法，即电影院是电影的代理
        movie.play();
    }
}
```
这里不容易理解的是，为什么电影院也要实现 Movie接口？ 在第一节我们就解释过，代理和被代理这做的事情从本质上来说是一样的。即RealMovie与Cinema都是播放电影的，
只不过是电影向提高自己的播放量，找到了电影院这个平台。所以电影院也实现Movie接口，并且添加一些自己特有的方法，如插播广告等实现对实际播放电影的代理。

### 动态代理  
试想一下，如果此时有电影A、B、C需要电影院代理，他们只是play()方法不同，但是使用静态代理你必须手动为他们创建代理，而动态代理可以在程序运行时为它们生成代理
可以大大的减少编码量。这也就是问什么我们使用动态代理的原因。
下面将Cinema类该为静态代理（其它文件不变）
```java
public class Cinema implements InvocationHandler {
    private  Movie target;
    public Object getInstance(Movie target){
        this.target=target;
        Class clazz=target.getClass();
        return Proxy.newProxyInstance(clazz.getClassLoader(),clazz.getInterfaces(),this);
    }
    
    //通过该方法实现对委托类的代理的访问，是代理类完整逻辑的集中体现，包括要切入的增强逻辑和进行反射执行的真实业务逻辑
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("电影马上开始了，爆米花、可乐、口香糖..");
        this.target.play();
        System.out.println("电影马上结束，快回家吧！、、");
        return null;
    }
}
```
进行测试
```java
public class Test {
    public static void main(String[] args){
       RealMovie realMovie=new RealMovie();
        Movie movie=(Movie) new Cinema().getInstance(realMovie);
        System.out.println(movie.getClass()); //output:class com.sun.proxy.$Proxy0
        movie.play();
    }
}/*output
电影马上开始了，爆米花、可乐、口香糖..
您正在观看电影《肖申克的救赎》
电影马上结束，快回家吧！、、
*/
```
实现的效果与静态代理一致，那么动态代理是如何完成这个功能的呢？
Test.java文件中输出了动态代理对象的类名称 class com.sun.proxy.$Proxy0，可见jdk自动帮我们创建了代理类。现在将该类的字节码文件打印出来并反编译一下，看看它内部是如何实现的
1、 在Test.java中输出字节码文件
```java
public class Test {
    public static void main(String[] args){
       RealMovie realMovie=new RealMovie();
        Movie movie=(Movie) new Cinema().getInstance(realMovie);
        System.out.println(movie.getClass()); //output:class com.sun.proxy.$Proxy0
        byte[] data=ProxyGenerator.generateProxyClass("$Proxy0",new Class[]{movie.getClass()});
        FileOutputStream fos=null;
        try {
            fos=new FileOutputStream("E:/$Proxy0.class");
            fos.write(data);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                fos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        movie.play();
    }
}
```
2、 将生成的字节码文件拖入IDEA（它会自动帮你反编译），由于生成的字节码文件过长，我们只分析对自己有用的部分，原文件可以在项目的同级目录下找到。
如下，实现的代理类也包含Movie接口中的play()方法，并且使用final修饰，在运行过程中该方法不可以再被修改。在该方法中调用了父类中的 invoke()方法。
所以动态代理执行委托代理对象的实际位置是代理类的invoke方法
```java
    public final void play() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
    
    
    //m3的定义为
    m3 = Class.forName("com.sun.proxy.$Proxy0").getMethod("play");


```
父类中 invoke的实现
```java
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("电影马上开始了，爆米花、可乐、口香糖..");
        this.target.play();
        System.out.println("电影马上结束，快回家吧！、、");
        return null;
    }
```
结合源码和文档，invoke传入的参数分别为代理对象、调用方法名称、调用方法的参数
因此可以直接调用 method.invoke(this.target,args) 调用委托调用者的方法。
并将target的类型该为Object，这样就可以接收通用的委托代理对象，而不只是Movie，播放个电视剧还是可以的...
具体参考源码





远程代理（headfirst设计模式）  
需求提出：对糖果机有很好的监控，找一种方法给CEO一份库存和机器状态报告。  
解决方法：
创建一个GumballMonitor 对糖果机进行监控，并将报告打印出来      
创建一个糖果机类 GumballMachine,它记录自己的位置和自己的生产状态  
将被监控的机器实例传递给 GumballMonitor,将其信息打印出来  


需求变更：  
需要的是远程监控糖果机，上面的实现都是在同一个JVM虚拟机中...  
解决方法
不变化GumballMonitor,不将糖果机交给GumballMonitor,而是将一个远程对象的代理交给它。  
原理如图：
 ![](remoteproxy)
 java内置的远程调用功能，RMI  
 原理：  
 rmi.jpg  

 实现步骤：
 1、制作远程接口；定义出可以让客户远程调用的方法。Stub和实际的服务都实现此接口
 2、制作远程实现： 实际工作的类，为远程接口中定义的远程方法提供真正的实现。这就是客户真正想要调用方法的对象
 3、利用rmic产生的stub和skeleton；  客户和服务的辅助类。工具自动生成（注此步骤在jdk8中已经被抛弃，不再使用rmic静态生成）
 4、启动RMI registry： 就像电话博，客户可以从中查到代理的位置
 5、开启远程服务； 服务类实例化一个服务对象，并将这个服务注册到RMI registry。之后服务就可以供客户调用了

实现原理
rmi-2.gif


java.security.AccessControlException: access denied.... 权限问题的错误  
解决方案  jdk\jre\lib\security\java.policy  
grant{} 最后一行添加  
permission java.security.AllPermission;  
参考博客：https://blog.csdn.net/qq_14994863/article/details/80769487  
继而报错：
java.rmi.ServerException: RemoteException occurred in server thread; nested exception is: 
	java.rmi.UnmarshalException: error unmarshalling arguments; nested exception is: 
解决方法 在class文件、bin目录下、当前类所在文件夹某一个下（这个真的是不确定，我是在classes文件夹）下执行rmiregistry  
然后再打开服务端  
开启客户端进行测试  
代码地址 spring_note\src\main\java\gupao\design\proxy\headfirst\rmitutor

//获取字节码文件
ProxyGenerator.generateProxy("$Proxy0",new Class[]{obj.getClass()});
FileOutputStream os=new FileOutputStream("$Proxy0.class");
os.write(data)
os.close()

如果报错的话引入 jdk/jre下面的rt.jar
使用反编译工具查看生成的动态代理类的源码（直接将 class文件拖入IDEA可以直接查看源码--IDEA自动帮忙完成反编译工具）
method,invoke(this.tartget,args);