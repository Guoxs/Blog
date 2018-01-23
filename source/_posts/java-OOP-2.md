---
title: java 面向对象（二）
date: 2016-07-11 20:59:38
tags: Java
---
## static 关键字
static 关键字表示公共的概念：

- 使用 static 定义的属性不在堆内存之中保存，保存在全局数据区； 
- 使用 static 定义的属性表示类属性，类属性可以由类名称直接进行调用；
- static 属性虽然定义在类之中，但是其可以在没有实例化对象的时候进行调用（普通属性保存在堆内存里，而 static 属性保存在**全局数据区**之中）；
  <!--more-->
### 使用static定义方法
使用 static 定义的方法也可以在没有实例化对象产生的情况下由类名称直接进行调用。
```java
public static void setCountry(String c) {
    country = c ;
}
```
方法调用：
```java
Person.setCountry("燕京") ;
```
>static 定义的方法**不能**调用非static的方法或属性；而非 static 定义的方法可以调用 static 的属性或方法。
>这是因为使用 static 定义的属性和方法，可以在没有实例化对象的时候使用；而非 static 定义的属性和方法，**必须实例化对象之后才可以进行调用**。

### 主方法
如果一个方法在主类之中定义，并且由主方法直接调用的时候，那么前面必须有**public static**。
```java
public static 返回值类型 方法名称 (参数列表) {
    [return [返回值] ;]
}
```
```java
public class TestDemo {
    public static void main(String args[]) {
        print() ; // 直接调用
    }
    public static void print() {
        System.out.println("Hello World .") ;
    }
}
```
不用 static 修饰的调用方法：
```java
public class TestDemo {
    public static void main(String args[]) {
        new TestDemo().print() ; // 对象调用
    }
    public void print() { // 非static方法
        System.out.println("Hello World .") ;
    }
}
```
### static关键字的使用
在实际的工作之中，使用static的原因有二：

- 希望可以在没有实例化对象的时候可以轻松的执行类的某些操作；
- 现在希望表示出数据共享的概念。

## 代码块
代码块是在程序之中使用 “{}” 定义起来的一段程序，而根据代码块声明位置以及声明关键字的不同，代码块一共分为四种：**普通代码块、构造块、静态块、同步块**。

### 普通代码块
普通代码块是定义在方法之中的代码块。
```java
public class TestDemo {
public static void main(String args[]) {
    { // 普通代码块
        int x = 10 ; // 局部变量
        System.out.println("x = " + x) ;
    }
    int x = 100 ; // 全局变量
    System.out.println("x = " + x) ;
    }
}
```
几乎用不上。
### 构造块
普通代码块是定义在方法之中的，而构造块是定义在类之中的代码块。
```java
class Person {
    public Person() {
        System.out.println("构造方法。") ;
    }
    { // 构造块
        System.out.println("构造块。") ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        new Person() ; new Person() ; new Person() ;
    }
}
```
**构造块优先于构造方法执行**，而且每当有一个新的实例化对象产生的时候，就会出现构造块的执行。
### 静态块
静态块也是定义在类之中的，如果一个构造块上使用了static关键字进行定义的话，那么就表示静态块，但是静态块要考虑两种情况：

情况一：在非主类之中定义的静态块
```java
class Person {
    public Person() {
     System.out.println("构造方法。") ;
    }
    { // 构造块
        System.out.println("构造块。") ;
    }
    static {
        System.out.println("静态块。") ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        new Person() ; new Person() ; new Person() ;
    }
}
```
**静态块优先于构造块执行，而且不管有多少个实例化对象产生，静态块只调用一次。**

情况二：在主类中定义的静态块
```java
public class TestDemo {
    static {
        System.out.println("静态块。") ;
    }
    public static void main(String args[]) {
        System.out.println("Hello World .") ;
    }
}
```
**在主类之中的静态块优先于主方法执行。**
### 同步块
java 同步块（synchronized block）用来标记方法或者代码块是同步的。Java同步块用来**避免竞争**。

Java中的同步块用 **synchronized** 标记。同步块在Java中是同步在某个对象上。所有同步在一个对象上的同步块同时只能被一个线程进入并执行操作。所有其他等待进入该同步块的线程将被阻塞，直到执行该同步块中的线程退出。

**实例方法同步**
```java
public synchronized void add(int value){
    this.count += value;
 }
```
Java实例方法同步是同步在**拥有该方法的对象**上。这样，每个实例其方法同步都同步在不同的对象上，即该方法所属的实例。只有一个线程能够在实例方法同步块中运行。如果有多个实例存在，那么一个线程一次可以在一个实例同步块中执行操作。一个实例一个线程。

**静态方法同步**
```java
public static synchronized void add(int value){
     count += value;
 }
```
静态方法的同步是指**同步在该方法所在的类对象上**。因为在Java虚拟机中一个类只能对应一个类对象，所以同时只允许一个线程执行同一个类中的静态同步方法。
对于不同类中的静态同步方法，一个线程可以执行每个类中的静态同步方法而无需等待。不管类中的那个静态同步方法被调用，一个类只能由一个线程同时执行。

**实例方法中的同步块**
有时不需要同步整个方法，而是同步方法中的一部分。Java可以对方法的一部分进行同步:
```java
public void add(int value){
    synchronized(this){
       this.count += value;
    }
  }
```
Java同步块构造器用括号将对象括起来。在上例中，使用了“this”，即为**调用add方法的实例本身**。在同步构造器中用括号括起来的对象叫做**监视器对象**。上述代码使用监视器对象同步，同步实例方法使用调用方法本身的实例作为监视器对象。

**静态方法中的同步块**
```java
public class MyClass {
    public static synchronized void log1(String msg1, String msg2){
       log.writeln(msg1);
       log.writeln(msg2);
    }
    public static void log2(String msg1, String msg2){
       synchronized(MyClass.class){
          log.writeln(msg1);
          log.writeln(msg2);
       }
    }
  }
```
这两个方法不允许同时被线程访问。
如果第二个同步块不是同步在MyClass.class这个对象上。那么这两个方法可以同时被线程访问。
## 内部类
内部类指的是在一个类的内部定义了其他类的情况
```java
class Outer { // 外部类
    private String msg = "Hello World " ; // 普通属性
    class Inner { // 内部类
        public void print() {
        System.out.println(msg) ;
    }
}
    public void fun() {
        Inner in = new Inner() ;
        in.print() ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        Outer out = new Outer() ;
        out.fun() ;
    }
}
```
内部类属于在一个类内部定义的新的结构，不过按照类的基本组成来讲，一个类之中应该由属性和方法所组成，但是这个时候又多了一个类，这样的结构并不好，所以**内部类本身最大的缺点破坏了程序的结构**。但是内部类本身也有自己的优点，而这个优点如果要想发现，最好的做法是**将内部类拿到外面来**，变为两个类。
```java
class Outer { // 外部类
    private String msg = "Hello World " ; // 普通属性
    public void fun() {
        Inner in = new Inner(this) ;
        in.print() ;
    }
    public String getMsg() {
    return this.msg ;
    }
}
class Inner { // 内部类
    private Outer out = null ;
    public Inner(Outer out) {
    this.out = out ;
    }
    public void print() {
    System.out.println(this.out.getMsg()) ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        Outer out = new Outer() ;
        out.fun() ;
    }
}
```
内部类的最大优点**：可以方便的访问外部类的私有操作，或者是由外部类方便的访问内部类的私有操作。**

但有一个问题，只要是访问类中的属性前面都要有 `this.`，但是在内部类使用`this`表示的只是**内部类的当前对象**，如果要想表示出外部类的当前对象，使用`外部类.this`来表示。

### 在不同的类操作内部类
一般内部类的`*.class` 文件名称为 `Outer$Inner.class`，作为Java的标识符，`$`也是一个标识符的组成元素，但是对于这样的元素，一直从未使用过，而`$`是在文件中的表示，但是换到了程序之中，每一个`$`表示一个 “.”，即：如果换到了程序里面，内部类的类名称就是：`Outer.Inner`。

所以在外部实例化内部类对象的操作格式：
>外部类.内部类内部类对象 = new 外部类().new 内部类() ; 

之所以实例化外部类对象，主要是因为内部类需要访问外部类之中的普通属性，那么普通属性只有在对象实例化之后才会被实例化（可以使用）。

```java
class Outer { // 外部类
    private String msg = "Hello World " ; // 普通属性
    class Inner { // 内部类
        public void print() {
            System.out.println(Outer.this.msg) ;// 内部类访问外部类
        }
    }
}
public class TestDemo {
    public static void main(String args[]) {
        Outer.Inner in = new Outer().new Inner() ; // 这种格式几乎不会出现
        in.print() ;
    }
}
```
如果内部类不希望被外面看见,可以这样：
```
class Outer { // 外部类
    private String msg = "Hello World " ; // 普通属性
    private class Inner { // 内部类
        public void print() {
            System.out.println(Outer.this.msg) ;// 内部类访问外部类
        }
    }
}
```
### 使用static定义内部类
使用 static 定义的属性和方法，是**独立于类之外**的，可以在没有实例化对象的时候调用，而 static 也同样可以进行内部类的定义，而使用了 static 定义的内部类，则就表示为“**外部类**”，并且**只能访问外部类之中 static 类型的操作**。

```java
class Outer { // 外部类
    private static String msg = "Hello World " ; // 普通属性
        static class Inner { // 内部类= ”外部类“
        public void print() {
            System.out.println(Outer.msg) ;// 内部类访问外部类
        }
    }
}
public class TestDemo {
    public static void main(String args[]) {
        Outer.Inner in = new Outer.Inner() ;
        in.print() ;
    }
}
```

### 在方法中定义内部类
内部类理论上可以在类的任意位置上进行定义，这就包括代码块之中，或者是普通方法之中。在普通方法里面定义内部类的情况是最多的。

但是一个内部类如果要定义在方法之中，并且要访问方法的参数或者是方法中定义变量的时候，这些参数或变量前一定要增加一个“**final**”关键字。
```java
class Outer { // 外部类
    private String msg = "Hello World " ; // 普通属性
    public void fun(final int x) { // 参数
        final String info = "Hello MLDN" ; // 变量
        class Inner { // 方法中定义的内部类
            public void print() {
                System.out.println(Outer.this.msg) ;
                System.out.println(x) ;
                System.out.println(info) ;
            }
        }
        Inner in = new Inner() ; // 产生内部类对象
        in.print() ;
    }
}
public class TestDemo {
    public static void main(String args[]) {
        new Outer().fun(30) ;
    }
}
```