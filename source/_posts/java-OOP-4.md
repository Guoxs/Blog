---
title: Java 面向对象（四）
date: 2016-07-31 20:19:05
tags: Java
---
## 匿名内部类
匿名内部类指的是没有名字的内部类。匿名内部类是在抽象类和接口的基础之上所发展起来的一种应用。
匿名内部类的语法为：
```java
new SuperType(construction parameters){
    inner class methods and data;
}
```
<!--more-->
```java
interface Message { // 定义了一个接口
    public void print() ; // 抽象方法
}
class Demo {
    public static void get(Message msg) { // 接收接口对象
        msg.print() ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        Demo.get(new Message() { // 匿名内部类
            public void print() {
                System.out.println("Hello World .") ;
            }
        }) ;
    }
}
```
## Object类
Object 类是所有类的超类。
在Java的定义之中，除了 Object 类之外，所有的类实际上都存在继承关系。如果定义一个类没有默认继承任何一个父类的话，则默认将继承 Object 类。
所以下面两种定义是等价的：
```java
class Person {}
class Person extends Object {}
```
Object类可以接收所有类的实例化对象:
```java
class Person {}
    public class TestDemo {
    public static void main(String args[]) { 
        Object obj = new Person() ;
        Person per = (Person) obj ; 
    }
}
```
对于任意的一个简单Java类而言，理论上讲应该覆写Object类之中的三个方法： 

- **取得对象信息**：public String toString()； 
- **对象比较**：public boolean equals(Object obj)； 
- **取得哈希码**：public int hashCode()。

### toString()
对对象直接调用 println() 函数的话得到的是一个对象的地址，而 toString() 方法默认也是输出对象的地址。也就是说执行 println() 函数是对象默认调用了 toString() 函数。
```java
class Person {}
public class TestDemo {
    public static void main(String args[]) {
        Person per = new Person() ;
        System.out.println(per) ; // Person@ 1f 6226 
        System.out.println(per.toString()) ; // Person@ 1f 6226 }
}
```
如果要输出对象的具体信息，可以对这个方法进行覆写：
```java
public String toString() { // 方法覆写 
    return "姓名：" + this.name + "，年龄：" + this.age ;
}
```
### equals()
在Object类之中，默认的 equals() 方法实现比较的是两个对象的**内存地址数值**，这并不符合于真正的对象比较需要，需要进行覆写：
```java
class Person { 
    private String name ; 
    private int age ; 
    public Person(String name,int age) { 
        this.name = name ; 
        this.age = age ; 
    }
    public String toString() { // 方法覆写 
        return "姓名：" + this.name + "，年龄：" + this.age ; 
    }   
    public boolean equals(Object obj) { 
        if (this == obj) { 
            return true ; 
        }
        if (obj == null) { 
            return false ; 
        }
        if (! (obj instanceof Person)) { // 不是本类对象
            return false ; 
        }
        // 因为name和age属性是在Person类中定义，而Object类没有
        Person per = (Person) obj ;
        if (this.name.equals(per.name) && this.age == per.age) {
            return true ; 
        }
        return false ; 
    }
}
```
### 使用Object接收所有的引用数据类型
Object是所有类的父类，所以 Object 类可以接收所有类的对象，但是在Java设计的时候，考虑到引用数据类型的特殊性，所以 Object 类实际上是可以接收**所有引用数据类型**的数据，这就包括了数组、接口、类。

使用Object类接收数组
```java
public class TestDemo { 
    public static void main(String args[]) { 
        Object obj = new int [] {1,2,3} ; // 接收数组 
        if (obj instanceof int []) { 
            int [] data = (int []) obj ; // 向下转型 
            for (int x = 0 ; x < data.length ; x ++) { 
                System.out.println(data[x]) ; 
            } 
        } 
    }
}
```
### hashCode
hash code 是对象导出的一个整数值。散列码是没有规律的。两个不同的对象其散列码基本上不会相同。
String 类使用下列算法计算散列码:
```java
int hash = 0;
for(int i = 0; i < length(); i++){
    hash = 31 * hash + charAt(i);
}
```
由于hashCode 方法定义在Object类中，故每个对象都有一个默认的散列码，其值为对象的存储地址。
如果重新定义了 equals() 方法，就一定要重新定义 hashCode() 方法，以便用户可以将对象插入到散列表中。

hasCode 方法应该返回一个整型数值（也可能是负数），合理的组合实例域的散列码，可以使各个不同对象产生的散列码更加均匀。
## 包装类
在Java的设计之中，一直提倡一个原则：一切皆对象。但是基本数据类型不是对象，这就促使了包装类的诞生。
包装类基本是这样定义的：
```java
class Int { // 类 
    private int num ; // 基本型 
    public Int(int num) { 
        this.num = num ; 
    }
    public int intValue() { 
        return this.num ; 
    }
}
```
java 提供了八种包装类，这八种包装类又分为两大阵营：

- 数值型（Number子类）：Byte、Short、Integer、Float、Double、Long；
- 对象型（Object子类）：Boolean、Character。

针对数值型包装类定义的方法：byteValue()、intValue()、doubleValue()、shortValue()、floatValue()、doubleValue()，可以从包装的类之中取得所包装的数值。
### 装箱与拆箱
在基本数据类型和包装类之间的转换操作之中分为两个重要概念：

- 装箱操作：将基本数据类型变为包装类，称为装箱，包装类的构造方法；
- 拆箱操作：将包装类变为基本数据类型，称为拆箱，各个类的 xxValue()方法。

以double和Double为例
```java
public class TestDemo { 
    public static void main(String args[]) { 
        Double var = new Double(15.5) ; // 装箱 
        double result = var.doubleValue() ; // 拆箱 
        System.out.println(result * result) ; 
    }
}
```
JDK 1.5之后，Java提供了自动的装箱和拆箱机制，并且包装类的对象可以自动的进行数学计算
```java
public class TestDemo { 
    public static void main(String args[]) { 
        Integer var = 15 ; // 自动装箱 
        int result = var ; // 自动拆箱 
        System.out.println(++ var * result) ; 
    }
}
```
因为有了这样的自动装箱和拆箱的机制，所以Object也可以接收基本数据类型的数据。
```java
public class TestDemo { 
    public static void main(String args[]) { 
        Object obj = 15 ; // int --> 自动装箱 --> Object 
        int result = (Integer) obj ; // Object --> 包装类 --> 自动拆箱
        System.out.println(result * result) ; 
    }
}
```
使用包装类的时候还需要考虑equals()和==的区别
```java
public class TestDemo { 
    public static void main(String args[]) { 
        Integer x = new Integer(10) ; // 新空间 
        Integer y = 10 ; // 入池 
        Integer z = 10 ; 
        System.out.println(x == y) ; // false 
        System.out.println(x == z) ; // false 
        System.out.println(z == y) ; // true 
        System.out.println(x.equals(y)) ; // true 
    }
}
```
### 数据转型
包装类之中所提供的最大优点还在于可以将字符串变为指定的基本数据类型

- **Integer类**：public static int parseInt(String s)； 
- **Double类**：public static double parseDouble(String s)； 
- **Boolean类**：public static boolean parseBoolean(String s)；

但是 Character 这个包装类之中，并没有提供一个类似的 `parseCharacter()`，因为字符串 String 类之中提供了一个`charAt()`方法，可以取得指定索引的字符，而且一个字符的长度就是一位。

在执行这种转换的操作过程之中，字符串中的全部内容必须由数字所组成，如果有一位内容不是数字，则在转换的过程之中将出现如下的错误提示：**NumberFormatException**。

在使用Boolean型包装类的时候，如果字符串之中的内容不是true或者是false，统一都按照false处理。

**基本类型变为String字符串**
方式一：任何的基本类型数据遇到了String之后都变为String型数据；
```java
public class TestDemo { 
    public static void main(String args[]) { 
        int num = 100 ; 
        String str = num + "" ; // int --> String 
        System.out.println(str.length()) ; 
    }
}
```
方式二：利用String类的方法，public static String valueOf(数据类型b)
```java
public class TestDemo { 
    public static void main(String args[]) { 
        int num = 100 ; 
        String str = String.valueOf(num) ; // int --> String 
        System.out.println(str.length()) ; 
    }
}
```
