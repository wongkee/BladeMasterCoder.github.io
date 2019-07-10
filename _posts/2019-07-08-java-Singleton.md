---
layout: post
title: 单例设计模式
category: 技术
tags: [Java基础]
description: 
---







> 单例设计模式在我们日常开发中经常用到，今天总结下单例设计模式的使用方法。

#### 何为单例设计模式

保证系统启动到系统停止，全过程只会产生一个实例，这就是所谓的单例。    

#### 哪些情况用到单例

1. 配置文件
   一方面配置文件创建多个实例会导致资源的浪费
    另一方面产生多个实例不好确定应以哪一个实例为准
2. 直接上级领导（不知道该听谁的）
3. 在应用中用到功能性冲突的时候会使用单例模式

#### 实现方法

综上，单例模式就是保证对象的唯一性，主要分三步
（1）不允许其它程序 new 对象
   解决方法：将默认构造方法设为private
（2）该类创建对象
将默认构造方法设为private会导致除了自己谁也不可以创建自己的对象，因此自己 new一个对象并将其返回。

（3） 对外提供一个可以让其它程序获取该对象的方法
因为对象是在本类中创建的，所以需要提供一个方法让其它的类获取这个对象。

有效的实现方法主要提供了种：   



1、饿汉式

即类加载时变实例化对象

```java
public class Singleton implements Serializable{
    private static  Singleton INSTANCE=new Singleton();
    private Singleton(){}
    public static Singleton getInstance(){
        return INSTANCE;
    }
}
```

2、懒汉式

等到调用getInstance()方法时再实例化对象。但是这样会存在线程安全问题：  

第一个线程 返现 INSTANCE 为null，进入 if语句并准备创建对象， 此时 第二个线程也判断INSTANCE 为null 进入if语句并创建对象， 这样就导致实例化了两个对象。

```java
public class Singleton1 implements Serializable {
    private static Singleton1 INSTANCE=null;
    private Singleton1(){};
    public static Singleton1 getInstance(){
        if (INSTANCE==null){
            INSTANCE = new Singleton1();
        }
        return  INSTANCE;
    }
}
```

3、解决懒汉式的线程不安全问题

getInstance()方法上加锁，但是会导致效率很低

```java
public class Singleton2 implements Serializable {
    private static Singleton2 instance =null;
    private Singleton2(){}
    public  static synchronized Singleton2 getInstance(){
       /*
       * 静态方法中智能引用静态属性
       * */
       if(instance==null){
           instance=new Singleton2();
       }
        return instance;
    }
}
```

4、双重校验锁

 volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，保证了每次读写变量都从主内存中读，跳过CPU cache这一步。当一个线程修改了这个变量的值，新值对于其他线程是立即得知的。 (此处涉及到JVM指令重排序优化，暂时不是很理解，回头再看)

```java
public class Singleton3 {
    private  static volatile  Singleton3 instance =null;
    private Singleton3(){}
    public  static Singleton3 getInstance(){
        if(instance==null){
            synchronized (Singleton3.class){
                if (instance==null){
                    instance=new Singleton3();
                }
            }
    }
        return  instance;
    }
}

```

5、内部类的使用

与饿汉式有几分相似，不同之处在于该方法是在内部类中进行实例化，有以下几点好处：  

1、只有在调用getInstance()方法的时候 SingletonHolder类才会被加载  

2、类的静态属性只会在第一次加载类的时候初始化  

3、在类的初始化时，别的线程是无法进入的，JVM帮我们保证了线程的安全   

```java
public class Singleton4 implements Serializable {
    private Singleton4(){}
    private static class  SingletonHolder{
        private static final Singleton4 instance=new Singleton4();
    }
    public static Singleton4 getInstance(){
        return SingletonHolder.instance;
    }
}

```



5、Spring中的单例实现方法  

```java
public class SpringSingleton implements Serializable {
    private static Map<String,SpringSingleton> map = new HashMap<String,SpringSingleton>();
    static {
        SpringSingleton single = new SpringSingleton();
        map.put(single.getClass().getName(), single); //将类名称和类的实例存储到map中
    }
    //保护的默认构造子
    protected SpringSingleton(){}
    //静态工厂方法,返还此类惟一的实例
    public static SpringSingleton getInstance(String name) {
        if(name == null) {
            name = SpringSingleton.class.getName();
        }
        if(map.get(name) == null) {
            try {
                //如果不存在该名称所对应的类的实例，则创建一个
                map.put(name, (SpringSingleton) Class.forName(name).newInstance());
            } catch (InstantiationException e) {
                e.printStackTrace();
            } catch (IllegalAccessException e) {
                e.printStackTrace();
            } catch (ClassNotFoundException e) {
                e.printStackTrace();
            }
        }
        return map.get(name);
    }
}

```



代码测试  

```java
public class SingletonTest {
    public static void main(String[] args){
        /*
        * 使用 ==比较 比较的是两个对象是否处于同一块内存，即 比较的是地址
        * */
        Singleton s1=Singleton.getInstance();
        Singleton s2=Singleton.getInstance();
        if(s1==s2){
            System.out.println("s1==s2");
        }else {
            System.out.println("s1!=s2");
        } //output: s1==s2


        Singleton1 s11=Singleton1.getInstance();
        Singleton1 s12=Singleton1.getInstance();
        if(s11==s12){
            System.out.println("s11==s12");//s11==s12
        }else {
            System.out.println("s11!=s12");
        }

        Singleton2 s21=Singleton2.getInstance();
        Singleton2 s22=Singleton2.getInstance();
        if(s21==s22){
            System.out.println("s21==s22");//s21==s22
        }else {
            System.out.println("s11!=s12");
        }

        Singleton3 s31=Singleton3.getInstance();
        Singleton3 s32=Singleton3.getInstance();
        if(s31==s32){
            System.out.println("s31==s32"); //s31==s32
        }else {
            System.out.println("s31!=s32");
        }


        Singleton4 s41=Singleton4.getInstance();
        Singleton4 s42=Singleton4.getInstance();
        if(s41==s42){
            System.out.println("s41==s42");////s41==s42
        }else {
            System.out.println("s41!=s42");
        }


        /*
        * 使用多线程进行测试
        * */
        int count=5000;
        CountDownLatch latch=new CountDownLatch(count);
        final Set<Singleton1> syncSet= Collections.synchronizedSet(new HashSet<Singleton1>());
        for (int i=0;i<count;i++){
            new Thread(){
                public void run(){
                    syncSet.add(Singleton1.getInstance());
                }
            }.start();
            latch.countDown();
        }

        try{
            latch.await();
            System.out.println(syncSet.size());  //output: 1  一般比较不出来结果
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

//要注意 java反射机制里面要使用全类名
        SpringSingleton springSingleton=SpringSingleton.getInstance("gupao.design.singleton.SpringSingleton");
        SpringSingleton springSingleton1=SpringSingleton.getInstance("gupao.design.singleton.SpringSingleton");

        if(springSingleton==springSingleton1){
            System.out.println("springSingleton==springSinglrton1");
        }else {
            System.out.println("springSingleton!=springSinglrton1");
        }

    }
}
```



#### 参考链接

[Java设计模式—单例设计模式(Singleton Pattern)完全解析](https://blog.csdn.net/dmk877/article/details/50311791)

[深入浅出单实例Singleton设计模式](https://blog.csdn.net/haoel/article/details/4028232)