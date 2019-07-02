---
layout: post
title: Java中的序列化操作
category: 技术
tags: [Java基础]
description: 
---







> 序列化在 Java、JavaScript 中都有使用到，虽然会简单的使用但是对其中原理不是很了解，今天对序列化操作的原理，以及在开发中的使用场景进行总结,以加深自己的理解。

##### 1、序列化基础知识

###### （1）何为序列化

百度百科：“序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。”  

维基百科：“**序列化**（serialization）在[计算机科学](https://zh.wikipedia.org/wiki/計算機科學)的数据处理中，是指将[数据结构](https://zh.wikipedia.org/wiki/資料結構)或对象状态转换成可取用格式（例如存成文件，存于缓冲，或经由网络中发送），以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始对象相同语义的副本。”    

计算机科学中的定义：在数据储存与发送的部分是指将一个[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))存储至一个[存储介质](https://zh.wikipedia.org/w/index.php?title=儲存媒介&action=edit&redlink=1)，例如[文件](https://zh.wikipedia.org/wiki/檔案)或是[存储器缓冲](https://zh.wikipedia.org/w/index.php?title=記憶體緩衝&action=edit&redlink=1)等，或者透过网络发送数据时进行编码的过程，可以是[字节](https://zh.wikipedia.org/wiki/字节)或是[XML](https://zh.wikipedia.org/wiki/XML)等格式。而[字节](https://zh.wikipedia.org/wiki/字节)的或[XML](https://zh.wikipedia.org/wiki/XML)编码格式可以还原完全相等的[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))。这程序被应用在不同[应用程序](https://zh.wikipedia.org/wiki/應用程式)之间发送[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))，以及服务器将[对象](https://zh.wikipedia.org/wiki/对象_(计算机科学))存储到[文件](https://zh.wikipedia.org/wiki/檔案)或[数据库](https://zh.wikipedia.org/wiki/資料庫)。相反的过程又称为[反序列化](https://zh.wikipedia.org/w/index.php?title=反序列化&action=edit&redlink=1)。  

简而言之，序列化是用某种方式对对象的状态进行保存和读取。  

###### （2）何时使用到序列化

- 经由电信线路传输数据的方法（通信）。

- 存储数据的方法（在数据库或硬盘）。

- 远程程序调用的方法，例如在[SOAP](https://zh.wikipedia.org/wiki/SOAP)中。

- 在以组件为基础，例如[COM](https://zh.wikipedia.org/wiki/组件对象模型)，[CORBA](https://zh.wikipedia.org/wiki/CORBA)的软件工程中，是对象的分布式方法。

- 检测随时间数据变动的方法。

  

###### （3）序列化格式

1. XML（可扩展标记语言）；优点：可读性高；缺：失去了扎实的扎实的编码字节流。
2. 二进制XML被提议作为一种妥协方式，它不能被纯文本编辑器读取，但比一般XML更为扎实。在二十一世纪的Ajax技术网页中，XML经常应用于结构化数据在客端和服务端之间的异步传输。
3. [JSON](https://zh.wikipedia.org/wiki/JSON)是一种轻量级的纯文字替代，也常用于网页应用中的客端－服务端通信。
4. [YAML](https://zh.wikipedia.org/wiki/YAML)，它包含加强序列化的功能，更“人性化”而且更扎实。这些功能包括标记数据类型，支持非层次结构式数据结构，缩进结构化数据的选项以及多种形式的标量数据引用的概念。

##### 2、Java中的序列化

###### 1、实现原理

上述介绍了一些序列化的基本知识，那么Java中是如何实现对象的序列化的呢？  

Java 序列化技术可以使你将一个对象的状态写入一个Byte 流里（系列化），并且可以从其它地方把该Byte 流里的数据读出来（反序列化）。  



###### 2、基本步骤

1. 实现 Serializable 接口，使得类可序列化；

   ```java
   class Person implements Serializable{	
   	private static final long serialVersionUID = 1L; //一会就说这个是做什么的
   	String name;
   	int age;
   	public Person(String name,int age){
   		this.name = name;
   		this.age = age;
   	}	
   	public String toString(){
   		return "name:"+name+"\tage:"+age;
   	}
   }
   ```

   

2. 通过ObjectOutputStream 的writeObject()方法把这个类的对象写到一个地方（文件）；

   ```java
   File file = new File("file"+File.separator+"out.txt");
   	
   	FileOutputStream fos = null;
   	try {
   		fos = new FileOutputStream(file);
   		ObjectOutputStream oos = null;
   		try {
   			oos = new ObjectOutputStream(fos);
   			Person person = new Person("tom", 22);
   			System.out.println(person);
   			oos.writeObject(person);			//写入对象
   			oos.flush();
   		} catch (IOException e) {
   			e.printStackTrace();
   		}finally{
   			try {
   				oos.close();
   			} catch (IOException e) {
   				System.out.println("oos关闭失败："+e.getMessage());
   			}
   		}
   	} catch (FileNotFoundException e) {
   		System.out.println("找不到文件："+e.getMessage());
   	} finally{
   		try {
   			fos.close();
   		} catch (IOException e) {
   			System.out.println("fos关闭失败："+e.getMessage());
   		}
   	}
   							
   ```

   

3. 通过ObjectInputStream 的readObject()方法把这个对象读出来。

   ```java
   FileInputStream fis = null;
   	try {
   		fis = new FileInputStream(file);
   		ObjectInputStream ois = null;
   		try {
   			ois = new ObjectInputStream(fis);
   			try {
   				Person person = (Person)ois.readObject();	//读出对象
   				System.out.println(person);
   			} catch (ClassNotFoundException e) {
   				e.printStackTrace();
   			} 
   		} catch (IOException e) {
   			e.printStackTrace();
   		}finally{
   			try {
   				ois.close();
   			} catch (IOException e) {
   				System.out.println("ois关闭失败："+e.getMessage());
   			}
   		}
   	} catch (FileNotFoundException e) {
   		System.out.println("找不到文件："+e.getMessage());
   	} finally{
   		try {
   			fis.close();
   		} catch (IOException e) {
   			System.out.println("fis关闭失败："+e.getMessage());
   		}
   	}
   ```

   

###### 3、注意事项

1、序列化时，只对对象的状态进行保存，而不管对象的方法；

2、当一个父类实现序列化，子类自动实现序列化，不需要显式实现Serializable接口；相反，如果一个子类实现序列化，但是父类没有实现序列化，那么父类中必须有可访问的无参构造函数。这是因为一个 Java 对象的构造必须先有父对象，才有子对象，反序列化也不例外。在反序列化时，为了构造父对象，只能调用父类的无参构造函数作为默认的父对象。因此当我们取父对象的变量值时，它的值是调用父类无参构造函数后的值。在这种情况下，在序列化时根据需要在父类无参构造函数中对变量进行初始化，否则的话，父类变量值都是默认声明的值，如 int 型的默认是 0，string 型的默认是 null。

```java
class People{
	int num;
	public People(){}  			//默认的无参构造函数，没有进行初始化
	public People(int num){		//有参构造函数
		this.num = num;
	}
	public String toString(){
		return "num:"+num;
	}
}
class Person extends People implements Serializable{	
	
	private static final long serialVersionUID = 1L;
	
	String name;
	int age;
	
	public Person(int num,String name,int age){
		super(num);				//调用父类中的构造函数
		this.name = name;
		this.age = age;
	}
	public String toString(){
		return super.toString()+"\tname:"+name+"\tage:"+age;
	}
}
//写入对象
	Person person = new Person(10,"tom", 22); //调用带参数的构造函数num=10,name = "tim",age =22
	System.out.println(person);
	oos.writeObject(person);	

//读取对象
	Person person = (Person)ois.readObject(); //反序列化，调用父类中的无参构函数。
	System.out.println(person);

//output:
num:0	name:tom	age:22
```



3、当一个对象的实例变量引用其他对象，序列化该对象时也把引用对象进行序列化；

4、并非所有的对象都可以序列化，至于为什么不可以，有很多原因了，比如：

（1）安全方面的原因，比如一个对象拥有private，public等field，对于一个要传输的对象，比如写到文件，或者进行RMI传输等等，在序列化进行传输的过程中，这个对象的private等域是不受保护的；  

（2）资源分配方面的原因，比如socket，thread类，如果可以序列化，进行传输或者保存，也无法对他们进行重新的资源分配，而且，也是没有必要这样实现；  

5、声明为static和transient类型的成员数据不能被序列化。因为static代表类的状态，transient代表对象的临时数据。

```java
transient String name; //反序列化时输出 name:null
```



6、序列化运行时使用一个称为 serialVersionUID 的版本号与每个可序列化类相关联，该序列号在反序列化过程中用于验证序列化对象的发送者和接收者是否为该对象加载了与序列化兼容的类。为它赋予明确的值。显式地定义serialVersionUID有两种用途（参考[serialVersionUID的作用以及设置方法](https://blog.csdn.net/kakaxi_77/article/details/8129070)）：

（1）在某些场合，希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有相同的serialVersionUID；  

（2）在某些场合，不希望类的不同版本对序列化兼容，因此需要确保类的不同版本具有不同serialVersionUID。    

7、Java有很多基础类已经实现了serializable接口，比如String,Vector等。但是也有一些没有实现serializable接口的；  

8、如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因；  

##### 参考资料

[我对java中Serializable接口的理解](https://blademastercoder.github.io/2015/01/29/java-Serializable.html)

[JDK文档](https://docs.oracle.com/javase/7/docs/api/java/io/Serializable.html)

[您不知道的 5 件事…… Java 对象序列化](https://www.ibm.com/developerworks/cn/java/j-5things1/index.html)

[序列化和反序列化的底层实现原理是什么？](https://blog.csdn.net/xlgen157387/article/details/79840134)

[serialVersionUID的作用以及设置方法](https://blog.csdn.net/kakaxi_77/article/details/8129070)



