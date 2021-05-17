# String和StringTable

> 说明：
>
> 1. 本文的阅读需要一定的JVM基础，如果您对JVM的底层结构不熟悉，建议跳过含有JVM结构的过程或直接看[总结](#总结)部分
> 2. 本文基于[尚硅谷JVM教程](https://www.bilibili.com/video/BV1PJ411n7xZ)，进行了一定省略与总结，仅作为String和StringTable的最基础介绍，为的是不牵涉出比较复杂的底层原理和因为jdk版本导致的变化，更加详细的信息推荐您观看视频或者查询更加详细的资料
> 3. “纸上得来终觉浅，绝知此事要躬行”，有关String的内容相对来说比较复杂，希望您能在自己的IDE上进行实操
> 4. 本人亲自尝试发现一些结果在main方法中运行和JUnit测试下得到的结果不一致，推荐使用main方法运行

## 前情提要

本人在浏览课程必读的[材料](http://web.mit.edu/6.031/www/fa20/classes/02-basic-java/#mutating_values_vs_reassigning_variables)时，发现这一块相当难以理解，主要问题在于材料中对”Unreassignable references”的说明不够充分，特别是对String类的说明不足。因为String类是编程时最常用的类，甚至可以说没有之一，因此选择对这个类以及StringTable进行一些说明。

## String的基本说明

* String使用一对引号`` " "           ``直接表示

* String的创建一般来说有两种方式：

  ```java
  String str1 = "I'm an HITer."; //字面量定义
  String str2 = new String("I'm an HITer."); //new对象的方式
  ```

* String实现了Comparable接口，可以直接进行大小的比较

* 在jdk9中，String的底层使用``byte[]``进行字符串数据存储。但是在jdk及之前，底层使用的是``final char value[]``

  > 为什么进行了修改？
  >
  > [官方文档](http://openjdk.java.net/jeps/254)的**Motivation**部分中有进行相关说明
  >
  > 简单来说，是在统计基础上进行的修改，为了节约空间
  >
  > 同时，对StringBuffer和StringBuilder类的底层也进行了同样的修改



## String的基本特性：不可变性

不可变性的说明：

- 当对字符串重新赋值时，需要重新指定内存区域赋值，不能使用原有的value进行赋值。
- 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
- 调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

简单来说，任何对String类对象的修改，都不是在原有的value基础上进行修改的，而是重新赋值



下面举出几个例子进行说明

例1：

```java
String s1 = "abc";
String s2 = "abc";
System.out.println(s1==s2); //true

s1="hello";
System.out.println(s1==s2); //false
```

例2：

```java
String s1 = "abc";
String s2 = "abc";
s2 += "def";
System.out.println(s2); //abcdef
System.out.println(s1); //abc
System.out.println(s1==s2); //false
```



## StringTable的基本信息

StringTable，也叫字符串常量池。如果您对**数据库连接池**有所了解，想必一定能很快理解字符串常量池的创建目的。如果没有了解也没有关系，这里还是首先要讲一下什么是**池**。

为了理解什么是池，首先我们要对常量进行考察：常量简单来说就是不会改变的信息，既然是不会改变的信息，就只需要存在一份就可以了，存在多份会浪费存储资源。然而在Java中讲究“一切皆对象”，对象的创建都会在堆上开辟一个新的空间用于存储，为了解决这个问题，JVM中才出现了字符串常量池的概念。也就是说，字符串常量池就是用来存储字符串信息，防止资源的浪费。

因此我们可以推理出StringTable最重要特征：**StringTable中不会存储内容相同的字符串**



## String的内存分配

在进行这一块内容的展开之前，我们不妨来看看JVM底层结构随jdk版本的变化，这里列出了从jdk6到jdk8的变化，从jdk8开始JVM底层没有太大结构性的调整，可以大致参考jdk8

<img src="https://github.com/Edmund-Lai/JavaBlog/blob/main/Blog01/appendix/%E9%85%8D%E5%9B%BE2.png" alt="配图2" style="zoom: 33%;" />

我们主要关注StringTable的位置变化：从jdk6到jdk8，它分别在永久代、堆、堆中

> 为什么进行调整？
>
> 原因简单来说有两点
>
> 1. 永久代空间较小，存储大量字符串很容易导致Full GC或者直接导致OOM
> 2. 堆空间足够大，触发Minor GC或者Old GC效率上也比Full GC高



String在进行内存分配时，有两种情况：

1. 如果以字面量形式定义，那么String对象会直接分配在StringTable中
2. 以new对象的形式定义的，会在堆中分配一块位置，然后如果StringTable中已经有了同样内容的字符串就不会再在StringTable中创建，否则会在其中创建



下面给出一个来自官方对StringTable的例子说明

```java
class Memory {
    public static void main(String[] args) {
        int i = 1;
        Object obj = new Object();
        Memory mem = new Memory();
        mem.foo(obj);
    }

    private void foo(Object param) {
        String str = param.toString();
        System.out.println(str);
    }
}
```

下图是对上述例子JVM中的堆、栈情况的说明（此图只是为了大致情况，栈帧局部变量表没有写全）

<img src="https://github.com/Edmund-Lai/JavaBlog/blob/main/Blog01/appendix/%E9%85%8D%E5%9B%BE1.png" alt="配图1" style="zoom:50%;" />

如果您对这一块还有很多疑问，请不要着急，文章下面的内容还会对您的一部分疑问进行回答



## String的拼接

String的拼接经常会带来十分疑惑的结果，这里首先总结String拼接的结论：

1. 常量与常量的拼接结果直接在常量池，因为编译时编译器会直接帮忙拼接上
2. 常量池中不会存在相同内容的变量
3. 拼接前后，只要其中有一个部分是以变量形式给出的，结果就在堆中
4. 如果拼接的结果调用intern()方法，根据该字符串是否在常量池中存在，分为：
   - 如果存在，则返回字符串在常量池中的地址
   - 如果字符串常量池中不存在该字符串，则在常量池中创建一份，并返回此对象的地址



下面给出例子进行说明：

例1：对上面结论1、2的说明

```java
String s1 = "ab" + "cd";
String s2 = "abcd";
System.out.println(s1==s2); //true
```

例2：对上面结论3、4的说明

```java
String s1 = "java";
String s2 = "script";
String s3 = "javascript";
String s4 = "java" + s2;
String s5 = s1 + "script";
String s6 = s1 + s2;

//如果拼接符号的前后出现了变量，则相当于在堆空间中new String()，拼接结果为javascript，但是这些对象在堆中的地址各不相同
System.out.println(s3==s5); //false
System.out.println(s3==s6); //false
System.out.println(s3==s7); //false
System.out.println(s4==s5); //false
System.out.println(s5==s6); //false
System.out.println(s6==s4); //false

//如果调用了对象.intern()，那么返回的就是这个字符串在StringTable中的地址
String s7 = s6.intern();
System.out.println(s3 == s7);//true
```

接下来对`intern()`进行必要的说明



## intern()的使用

对于``intern()``的基本说明：

1. intern()方法的原型：

   ```java
   public native String intern();
   ```

   这个方法是一个本地方法，不需要Java程序员对其进一步的细节进行考量

2. StringTable最初是空的，由String类进行维护。在调用intern方法时，如果池中已经包含了和调用它的String类中内容相等的字符串，则返回池中的字符串地址。否则，该字符串对象将被添加到池中，并返回对该字符串对象的地址

3. 如果不是用字面量形式声明的String对象，可以使用String提供的intern方法从字符串常量池中查询当前字符串是否存在，若不存在就会将当前字符串放入常量池中。比如：

   ```java
   String s = new String("Javase").intern();
   ```

4. 也就是说，使用任意String对象调用intern方法，其返回结果所指向的那个类实例，必须和直接以常量形式出现的字符串实例完全相同。因此，下列表达式的值必定是true：

   ```java
   ("a" + "b" + "c").intern()=="abc"; //true
   ```



在上述对intern的说明基础上，我们可以总结出如下的结论：

要想让变量s指向的是字符串常量池中的数据，而不是堆区中的数据，可以使用如下两种方法：

1. 使用字面量进行初始化

   ```java
   String s = "To be or not to be,that's a question.";
   ```

2. 使用intern()方法

   ```java
   String s = new String("To be or not to be,that's a question.").intern();
   ```



## 总结<a name="总结"> </a>

1. String最重要的特性是**不可变性**，任何的修改都会在StringTable中重新创建字符串
2. StringTable中**不会存储相同的字符串**
3. 在String类的拼接中：
   * 常量的拼接直接存储在StringTable中
   * 只要拼接元素之一是变量，就会在堆中创建
4. 通过`String类对象.intern()`可以返回StringTable中的字符串

