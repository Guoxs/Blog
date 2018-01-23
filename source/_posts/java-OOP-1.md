---
title: Java 面向对象（一）
date: 2016-07-11 17:30:00
tags: Java
---

## 类与对象基础
### 基本语法
**定义语法：**
```java
class 类名称 {
    属性 (变量) ; 行为 (方法) ;
}
```
**类实例化：**
格式一：
```java
类名称 对象名称 = new 类名称();
```
格式二：
```java
//声明对象
类名称 对象名称 = null;
//实例化对象
对象名称 = new 类名称 ();
```
<!--more-->
**两种实例化方法的区别：**
>`new` 关键字表示开辟了新的堆内存。
>`Person per = null`，这个时候只声明了Person对象，但是并没有实例化Person对象（只有了栈内存，并没有对应的堆内存空间）。程序在编译的时候不会出现任何的错误，但是在执行的时候会出现`NullPointerException`（空指向异常）。

**堆内存**：保存对象的真正数据，都是每一个对象的属性内容。
**栈内存**：保存的是一块堆内存的空间地址。

**引用传递的精髓**：同一块堆内存空间，同时被多个栈内存所指向，不同的栈可以修改同一块堆内存的内容。

**垃圾**：指的是在程序开发之中没有任何对象所指向的一块**堆内存空间**，这块空间就成为垃圾，所有的垃圾将等待GC（垃圾收集器）不定期的进行回收与空间的释放。

### 封装
public、private、protect。

要保证属性的安全，又要有利于外部操作，需要定义`setter`与`getter`方法。
在定义类的时候，所有的属性都要编写private封装，封装之后的属性如果需要被外部操作，则编写setter、getter。

### 构造方法
构造方法本身的定义如下： 

- 构造方法的名称和类名称保持一致； 
- 构造方法不允许有返回值类型声明；
- 由于对象实例化操作一定需要构造方法的存在，所以如果在类之中没有明确定义构造方法的话，则会自动的生成一个无参的，无返回值的构造方法，供用户使用，如果一个类之中已经明确的定义了一个构造方法的话，则无参的什么都不做的构造方法将不会自动生成，也就是说，一个类之中至少存在一个构造方法。
- 构造方法在对象实例化的时候完成操作，而且一个对象的构造方法只会显式调用一次。

### 匿名对象
没名字的对象称为匿名对象，对象的名字按照之前的内存关系来讲，在栈内存之中，而对象的具体内容在堆内存之中保存，这样一来，**没有栈内存指向堆内存空间，就是一个匿名对象**。
```java
class Person { 
    private String name ;
    private int age ;
    public Person(String n,int a) {
        name = n ;
        age = a ;
    }
    public void tell() {
        System.out.println("姓名：" + name + "，年龄：" + age) ;
    }
}

public class TestDemo {
    public static void main(String args[]) {
        new Person("张三",20).tell() ; // 匿名对象
    }
}
```
>匿名对象由于没有对应的栈内存指向，所以只能使用一次，一次之后就将成为垃圾，并且等待被GC回收释放。

## 数组
**数组的静态初始化**
格式一：简写格式
```java
数据类型 数组名称 [] = {值,值,...} ;
数据类型 [] 数组名称= {值,值,...} ;
```
格式二：完整格式（推荐使用）
```java
数据类型 数组名称 [] = new 数据类型 [] {值,值,...} ;
数据类型 [] 数组名称 = new 数据类型 [] {值,值,...} ;
int data [] = new int [] {209,201,2,2,3,6,7} ;
```
>在开发之中使用何种数组初始化的方式并没有一个明确的定义，主要还是看功能，如果已经知道了所有的操作数据，那么使用静态合适，如果有一些数据需要单独配置，那么用动态合适。

**与数组有关的操作方法**
1、**数组排序**： `java.util.Arrays.sort(数组名称)`
>使用java.util.Arrays.sort()也可以排序。

2、**数组拷贝**： 从一个数组之中拷贝部分内容到另外一个数组之中
方法：`System.arraycopy(源数组名称，源数组开始点，目标数组名称，目标数组开始点，拷贝长度)` ;

**二维数组**
动态初始化：
>数据类型 数组名称 [][] = new 数组名称 [行数] [列数] ;

静态初始化
>数据类型 数组名称 [][] = new 数组名称 [] [] { {值,值,...},{值,值,...},{值,值,...},...} ;

## String 类
**String类的构造**：public String(String str)；

**字符串比较**
`==`：比较的是两个字符串内存地址的数值是否相等，属于数值比较；  `equals()`：比较的是两个字符串的内容，属于内容比较。

**字符串常量是String的匿名对象**
```java
public class StringDemo {
    public static void main(String args[]) {
        String str = "Hello" ;
        // 通过字符串调用方法
        System.out.println("Hello".equals(str)) ;
    }
}
```
匿名对象可以调用类之中的方法与属性，以上的字符串可以调用了`equals()`方法，那么一定是一个对象。

### String类的两种实例化方式
1、分析直接赋值的情况：
```java
public class StringDemo {
    public static void main(String args[]) {
        String str1 = "Hello" ;
        String str2 = "Hello" ;
        String str3 = "Hello" ;
        System.out.println(str1 == str2) ; // true
        System.out.println(str1 == str3) ; // true
        System.out.println(str2 == str3) ; // true
    }
}
```
直接赋值操作之中，字符串比较都是相同的，原因是：
>在 String 类进行设计的时候采用了一种称为**共享设计模式**的概念，在每一个运行的 JVM 底层存在一个字符串的**对象池**（Object Pool），如果用户采用了直接赋值的方式，会将字符串的内容放入到池之中，以供其他继续使用直接赋值方式的 String 对象使用，如果新声明的字符串内容不再池之中，则会开辟一个新的，继续放到池，以供下次使用。

2、分析构造方法赋值的情况：
```java
public class StringDemo {
    public static void main(String args[]) {
        String str = new String("Hello") ;
        System.out.println(str) ;
    }
}
```
![构造方法赋值][1]

使用构造方法的方式开辟的字符串对象，实际上会开辟两块空间，其中有一块空间将称为垃圾。
```java
public class StringDemo {
    public static void main(String args[]) {
        String str1 = new String("Hello") ;
        String str2 = "Hello" ; // 入池
        String str3 = "Hello" ; // 使用池对象
        System.out.println(str1 == str2) ; // false
        System.out.println(str1 == str3) ; // false
        System.out.println(str2 == str3) ; // true
    }
}
```
，使用构造方法实例化的String对象，不会入池，所以，只能自己使用。可是在String类之中为了方便操作提供了一种称为手工入池的方法：`public String intern()`。
```java
public class StringDemo {
    public static void main(String args[]) {
        String str1 = new String("Hello").intern() ;
        String str2 = "Hello" ; // 入池
        String str3 = "Hello" ; // 使用池对象
        System.out.println(str1 == str2) ; // true
        System.out.println(str1 == str3) ; // true
        System.out.println(str2 == str3) ; // true
    }
}
```
**字符串的内容一旦声明则不可改变**
>字符串内容的更改，实际上改变的是字符串对象的引用过程，并且会伴随有大量的垃圾出现

### String类的常用方法
#### 字符串与字符
| No.  |                   方法名称                   |  类型  |                   描述                   |
| :--: | :--------------------------------------: | :--: | :------------------------------------: |
|  1   |       public String(char[] value)        |  构造  |            将全部的字符数组内容变为字符串             |
|  2   | public String(char[] value, int offset, int count) |  构造  | 将部分字符数组变为字符串，offset表示开始点，count表示要操作的长度 |
|  3   |      public char charAt(int index)       |  普通  |              取得指定索引位置上的字符              |
|  4   |       public char[] toCharArray()        |  普通  |              将字符串转换为字符数组               |

#### 字符串与字节
| No.  |                   方法名称                   |  类型  |      描述       |
| :--: | :--------------------------------------: | :--: | :-----------: |
|  1   |       public String(byte[] bytes)        |  构造  | 将全部的字节数组变为字符串 |
|  2   | public String(byte[] bytes, int offset, int length) |  构造  | 将部分的字节数组变为字符串 |
|  3   |         public byte[] getBytes()         |  普通  |  将字符串变为字节数组   |
|  4   | public byte[] getBytes(String charsetName) throws UnsupportedEncodingException |  普通  |    字符串转码操作    |

一般情况下，在程序之中如果要想操作字节数组只有两种情况：

- 需要进行编码的转换时
- 情况二：数据要进行传输的时候

#### 字符串比较
| No.  |                   方法名称                   |  类型  |      描述      |
| :--: | :--------------------------------------: | :--: | :----------: |
|  1   |  public boolean equals(String anObject)  |  普通  |  区分大小写的相等判断  |
|  2   | public boolean equalsIgnoreCase(String anotherString) |  普通  | 不区分大小写比较是否相等 |
|  3   | public int compareTo(String anotherString) |  普通  |  比较两个字符串的大小  |

要想比较两个字符串的大小关系，必须使用compareTo()方法完成，这个方法返回int型数据，而这个int型数据有三种结果：**大于（返回结果大于0）、小于（返回结果小于0）、等于（返回结果为0）**。

#### 字符串查找
| No.  |                   方法名称                   |  类型  |            描述             |
| :--: | :--------------------------------------: | :--: | :-----------------------: |
|  1   |    public boolean contains(String s)     |  普通  | 查找指定的子字符串是否存在，JDK 1.5之后有  |
|  2   |      public int indexOf(String str)      |  普通  |   从头查找指定字符串的位置，找不到返回-1    |
|  3   | public int indexOf(String str, int fromIndex) |  普通  |  由指定位置向后查找字符串的位置，找不到返回-1  |
|  4   |    public int lastIndexOf(String str)    |  普通  |   由后向前查找字符串的位置，找不到返回-1    |
|  5   | public int lastIndexOf(String str, int fromIndex) |  普通  |        从指定位置由后向前查找        |
|  6   | public boolean startsWith(String prefix) |  普通  |       判断是否以指定的字符串开头       |
|  7   | public boolean startsWith(String prefix, int toffset) |  普通  | 从指定位置判断是否以指定字符串开头，JDK 1.7 |
|  8   |  public boolean endsWith(String suffix)  |  普通  |       判断是否以指定的字符串结尾       |

#### 字符串替换、截取与拆分
| No.  |                   方法名称                   |  类型  |     描述      |
| :--: | :--------------------------------------: | :--: | :---------: |
|  1   | public String replaceAll(String regex, String replacement) |  普通  |    全部替换     |
|  2   | public String replaceFirst(String regex, String replacement) |  普通  |    替换首个     |
|  3   | public String substring(int beginIndex)  |  普通  | 从指定位置截取到结尾  |
|  4   | public String substring(int beginIndex, int endIndex) |  普通  |  截取指定范围的内容  |
|  5   |   public String[] split(String regex)    |  普通  | 按照指定的字符串全拆分 |
|  6   | public String[] split(String regex, int limit) |  普通  |  拆分为指定的长度   |

>在进行字符串拆分的时候，如果遇见拆不开的问题，可以试试 `\\`

#### 其他方法
| No.  |               方法名称               |  类型  |      描述       |
| :--: | :------------------------------: | :--: | :-----------: |
|  1   |     public boolean isEmpty()     |  普通  | 判断是否为空字符串（""） |
|  2   |       public int length()        |  普通  |    取得字符串长度    |
|  3   |       public String trim()       |  普通  |    去掉左右空格     |
|  4   |   public String toLowerCase()    |  普通  |   将全部字符串转小写   |
|  5   |   public String toUpperCase()    |  普通  |   将全部字符串转大写   |
|  6   |      public String intern()      |  普通  |      入池       |
|  7   | public String concat(String str) |  普通  |     字符串连接     |

编写一个让首字母大写的 initcap() 方法：
```java
public static String initcap(String s) { 
    return s.substring(0,1).toUpperCase().concat(s.substring(1));
}
```

## this 关键字
在Java中this可以完成三件事情：表示**本类属性**、表示**本类方法**、**当前对象**。

**this.属性**表示本类属性

**调用本类方法**
对于一个类之中的方法现在是分为两种： 

- 普通方法：在之前强调过，如果现在要调用的是本类方法，则可以使用 `this.方法()` 调用； 
- 构造方法：调用其他构造使用 `this()` 调用。

>**注意**：在使用 this 调用构造方法的时候有以下一些问题： 1、所以的构造方法是在对象实例化的时候被默认调用，而且是在调用普通方法之前调用，所以使用 `this()` 调用构造方法的操作，**一定要放在构造方法的首行**；



>2、如果一个类之中存在了多个构造方法的话，并且这些构造方法都使用了 `this()` 互相调用，那么至少要保留一个构造方法没有调用其他构造，以作为程序的出口。

**this表示当前对象，**当前对象指的是当前正在调用本类方法的操作对象

## 对象比较
对象比较的操作一定是一个类自己本身所具备的功能，而且对象比较的操作特点：

- 本类接收自己的引用，而后与本类当前对象（this）进行比较；
- 首先为了避免NullPointerException的产生，应该增加一个null的判断；
- 为了防止浪费性能的情况出现（要判断的属性会多），可以增加地址数值的判断，因为相同的对象地址相同；
- 之后进行属性的依次比较，如果属性全部相同，则返回true，否则返回false。

```java
public boolean compare(Person per) { 
    if (per == null) {
        return false; // 直接返回false 
    }if (this == per) { // 地址相同
        return true;
    }// 这样一来在compare()方法之中有两个对象：传入的Person，另外一个是this 
    if (this.name.equals(per.name) && this.age == per.age) {
        return true; 
    }
    return false;
}
```
[1]: http://static.zybuluo.com/guoxs/txoxnpweak0408f07jppydr3/1.png

