---
title: Java 面向对象（三）
date: 2016-07-31 20:18:58
tags: Java
---
## 继承
语法
```java
class 子类 extends 父类 {}
```
子类又被称为**派生类**，父类又被称为**超类**（Super Class）。

**继承的限制**

- 一个子类只能够继承一个父类，存在单继承局限（Java之中只允许多层继承，不允许多重继承）
- 在一个子类继承的时候，实际上会继承父类之中的**所有操作**（属性、方法），但是，对于所有的非私有（no private）操作属于显式继承（可以直接利用对象操作），而所有的私有操作属于隐式继承（间接完成）。
- 在继承关系之中，如果要实例化子类对象，会默认先调用父类构造，为父类之中的属性初始化，之后再调用子类构造，为子类之中的属性初始化（默认情况下，子类会找到父类之中的**无参构造方法**）。
  <!--more-->
```java
public class Test {
    public static void main(String args[]) {
        B b = new B() ; // 实例化子类对象
    }
}
```
上面代码实例化的是子类对象，但是会默认先执行父类构造，调用父类构造的方法体执行，而后再实例化子类对象，调用子类的构造方法。对于子类的构造而言，就相当于隐含了一个 `super()` 的形式。
```java
class B extends A {
    public B() { // 子类构造
        super() ; // 调用父类构造
        System.out.println("#########################");
    }
}
```
默认调用无参构造，如果父类没有无参构造，则子类必须通过 `super` 调用指定参数构造器。

**super调用父类构造的时候，一定要放在构造方法的首行上。**
>如果一个类之中有多个构造方法之间使用this()互相调用的话，那么至少要保留有一个构造方法作为出口，而这个出口就一定会去调用父类构造。一个简单Java类一定要保留有一个无参构造方法。

## 覆写
当子类定义了和父类在**方法名称**、**返回值类型**、**参数类型**及**个数**完全相同的方法的时候，称为方法的覆写。

```java
class A {
    public void print() {
        System.out.println("Hello World .") ;
    }
}
class B extends A {
    public void print() { // 方法名称、参数类型及个数、返回值全相同
        System.out.println("世界，你好！") ;
    }
}
public class Test {
    public static void main(String args[]) {
        B b = new B() ;
        b.print() ; // 方法从父类继承而来
    }
}
```
>当一个类之中的方法被覆写之后，如果实例化的是这个子类对象，则调用的方法就是被覆写过的方法。

**注意：**被子类所覆写的方法不能拥有比父类更严格的访问控制权限，如果此时父类之中的方法是 default 权限，那么子类覆写的时候只能是 default 或 public 权限，而如果父类的方法是 public，那么子类之中方法的访问权限只能是 public。

当一个子类覆写了一个父类方法的时候，那么在这种情况下，子类要想调用父类的被覆写过的方法，则在方法前要加 上“**super**”。

this 与 super 区别：

- **this.方法()**：先从本类查找是否存在指定的方法，如果没有找到，则调用父类操作； 
- **super.方法()**：直接由子类调用父类之中的指定方法，不再找子类。

| No.  |  区别  |             this              |              super               |
| :--: | :--: | :---------------------------: | :------------------------------: |
|  1   |  定义  |            表示本类对象             |              表示父类对象              |
|  2   |  使用  | 本类操作：this.属性、this.方法()、this() | 父类操作：super.属性、super.方法()、super(） |
|  3   | 调用构造 |         调用本类构造，要放在首行          |          子类调用父类构造，放在首行           |
|  4   | 查找范围 |        先从本类查找，找不到查找父类         |            直接由子类查找父类             |
|  5   |  特殊  |            表示当前对象             |                -                 |

重载与覆写的区别：

| No.  |  区别  |        重载         |             覆写             |
| :--: | :--: | :---------------: | :------------------------: |
|  1   | 英文单词 |    Overloading    |          Override          |
|  2   |  定义  | 方法名称相同、参数的类型及个数不同 |   方法名称、参数类型及个数、返回值类型完全相同   |
|  3   |  权限  |      没有权限要求       | 被子类所覆写的方法不能拥有比父类更严格的访问控制权限 |
|  4   |  范围  |     发生在一个类之中      |         发生在继承关系类之中         |

## final 关键字
在Java中，final关键字表示的是一个**终结器**的概念，使用 final可以定义类、方法、变量。

- 使用final定义的类不能有子类，太监类
- 使用final定义的方法不能被子类所覆写
- 如果说使用了public static来定义的常量，那么这个常量就称为全局常量。
>public static final String INFO = "hello world" ; // 全局常量

**定义final常量的时候每个单词的字母都要大写。**

## 构造方法私有化
```java
class Singleton { // 定义一个类
    private Singleton() {} // 构造方法私有化
    public void print() {
        System.out.println("Hello World .") ;
    }
}
```
使用private访问权限定义的操作只能被本类所访问，外部无法调用。

对于一个类之中的普通属性，默认情况下一定要在本类存在了实例化对象之后才可以进行调用，如果使用构造方法私有化，外部就无法产生实例化对象，也就不能调用方法。这是可以使用 **static** 关键字，static定义的属性特点：**由类名称直接调用，并且在没有实例化对象的时候也可以调用。**
```java
class Singleton { // 定义一个类
    static Singleton instance = new Singleton() ;
    private Singleton() {} // 构造方法私有化
    public void print() {
        System.out.println("Hello World .") ;
    }
}
public class Test {
    public static void main(String args[]) {
        Singleton inst = null ; // 声明对象
        inst = Singleton.instance ; // 实例化对象
        inst.print() ; // 调用方法
    }
}
```
类之中的全部属性都应该封装，所以以上的instance属性应该进行封装，而封装之后要想取得属性要编写getter方法，只不过这个时候的getter方法应该也由类名称直接调用，定义为static型。
```java
class Singleton { // 定义一个类
    private static Singleton instance = new Singleton() ;
    private Singleton() {} // 构造方法私有化
    public void print() {
        System.out.println("Hello World .") ;
    }
    public static Singleton getInstance() {
        return instance ;
    }
}
public class Test {
    public static void main(String args[]) {
        Singleton inst = null ; // 声明对象
        inst = Singleton.getInstance() ; // 实例化对象
        inst.print() ; // 调用方法
    }
}
```
>如果说现在一个类只希望有唯一的一个实例化对象出现，应该控制构造方法，如果构造方法对外部不可见了，那么现在肯定无法执行对象的实例化操作，必须将构造方法隐藏，使用**private**隐藏。

### 单例设计模式
但是，以上代码还有问题，比如：
```java
public static Singleton getInstance() {
    instance = new Singleton() ;
    return instance ;
}
```
这样就打破了一个类只实例化一个对象的设定。可以通过添加 final 关键字弥补：
```java
class Singleton { // 定义一个类
    private static final Singleton INSTANCE = new Singleton() ;
    private Singleton() {} // 构造方法私有化
    public void print() {
        System.out.println("Hello World .") ;
    }
    public static Singleton getInstance() {
        return INSTANCE ;
    }
}
public class Test {
    public static void main(String args[]) {
        Singleton inst = null ; // 声明对象
        inst = Singleton.getInstance() ; // 实例化对象
        inst.print() ; // 调用方法
    }
}
```
这样的设计在设计模式上讲就称为**单例设计模式（Singleton）**。

>单例设计模式特点：
>构造方法被私有化，只能够通过**getInstance()**方法取得Singleton类的实例化对象，这样不管外部如何操作，最终也只有一个实例化对象，在单例设计模式之中，一定会存在一个**static**方法，用于取得本类的实例化对象。

单例设计模式按照设计模式的角度而言，分为两种：

- **饿汉式**：之前写的程序就属于饿汉式，因为在类之中的**INSNTACE**属性是在定义属性的时候直接实例化；
- **懒汉式**：在第一次使用一个类实例化对象的时候才去实例化。

懒汉式：
```java
class Singleton { // 定义一个类
    private static Singleton instance ;
    private Singleton() {} // 构造方法私有化
    public void print() {
        System.out.println("Hello World .") ;
    }
    public static Singleton getInstance() {
        if (instance == null) { // 没有实例化
            instance = new Singleton() ; // 实例化
        }
        return instance ;
    }
}
```
### 多例设计模式
单例设计模式只留有一个类的一个实例化对象，而多例设计模式，会定义出多个对象。

例如：定义一个表示星期X的类，这个类的对象只有7个取值，定义一个表示性别的类，只有2个取值，定义一个表示颜色基色的操作类，颜色只有三个：红、绿、蓝，这种情况下，这样的类就**不应该由用户无限制的去创造实例化对象**，应该只使用有限的几个，这个就属于**多例设计**，但不管是单例设计还是多例设计，有一个核心不可动摇 —— **构造方法私有化**。
```java
class Sex {
    private static final Sex MALE = new Sex("男") ;
    private static final Sex FEMALE = new Sex("女") ;
    private String title ;
    private Sex(String title) { // 构造方法私有化
        this.title = title ;
    }
    public static Sex getInstance(String msg) {
        switch(msg) {
            case "male" :
                return MALE ;
            case "female" :
                return FEMALE ;
            default :
                return null ;
        }
    }
    public String getTitle() {
        return this.title ;
    }
}
public class Test {
    public static void main(String args[]) {
        Sex male = Sex.getInstance("male") ;
        System.out.println(male.getTitle()) ;
    }
}
```
## 多态性
多态性包括方法的多态性和对象的多态性。

- 方法的多态性：重载与覆写
  - **重载**：同一个方法名称，根据不同的参数类型及个数可以完成不同的功能；
  - **覆写**：同一个方法，根据操作的子类不同，所完成的功能也不同。
- 对象的多态性：父子类对象的转换。
  - **向上转型**：子类对象变为父类对象，格式：父类父类对象 = 子类实例，自动；
  - **向下转型**：父类对象变为子类对象，格式：子类子类对象= (子类) 父类实例，强制；

```java
A a = new B() ; // 向上转型
B b = (B) a ; // 向下转型 java.lang.ClassCastException: A cannot be cast to B
```

转型因素： 

- 在实际的工作之中，对象的向上转型为主要使用，80%，向上转型之后，所有的方法以父类的方法为主，但是具体的实现，还是要看子类是否覆写了此方法； 
- 向下转型，10%，因为在进行向下转型操作之前，**一定要首先发生向上转型**，以建立两个对象之间的联系，如果没有这种联系，是不可能发生向下转型的，一旦发生了运行中就会出现“ClassCastException”，当需要调用子类自己特殊定义方法的时候，才需要向下转型；
- 不转型，10%，在一些资源较少的时候，例如：移动开发。

可以使用 **instanceof** 关键字判断某一个对象是否是一个类的实例
```java
public class Test {
    public static void main(String args[]) {
        A a = new A() ;
        System.out.println(a instanceof A) ;
        System.out.println(a instanceof B) ;
        if (a instanceof B) {
            B b = (B) a ;
            b.getB() ;
        }
    }
}
```
子类操作的过程之中，尽量向父类靠拢。
```java
class A {
    public void print() {
        System.out.println("A、public void print(){}") ;
    }
}
class B extends A {
    public void print() { // 方法覆写
        this.getB() ;
        System.out.println("B、public void print(){}") ;
    }
    public void getB() {
        System.out.println("B、getB()") ;
    }
}
class C extends A {
    public void print() { // 方法覆写
        this.getC() ;
        System.out.println("C、public void print(){}") ;
    }
    public void getC() {
        System.out.println("C、getC()") ;
    }
}
public class Test {
    public static void main(String args[]) {
        fun(new B()) ;
        fun(new C()) ;
}
    public static void fun(A a) {
        a.print() ;
    }
}
```
**一个类不能去继承一个已经实现好的类，只能继承抽象类或实现接口。**

## 抽象类
普通类就是一个完善的功能类，可以直接产生对象并且可以使用，里面的方法都是带有方法体的。而
抽象类之中最大的特点是**包含了抽象方法**，**抽象方法是只声明而未实现（没有方法体）的方法**，抽象方法定义的时候要使用`abstract`关键字完成。抽象方法一定要在抽象类之中，抽象类也要使用`abstract`关键字声明。
定义一个抽象类
```java
abstract class A {
    private String info = "Hello World ." ;
    public void print() {
        System.out.println(info) ;
    }
    public abstract void get() ; // 只声明没有方法体
}
```
抽象类的实例化不能用 new 关键字，这是由于一个类的对象实例化之后，可以调用类中的属性和方法，但是抽象类之中的抽象方法没有方法体。

抽象类的使用原则：

- 抽象类必须有子类，使用 extends 继承，**一个子类只能继承一个抽象类**；
- 子类（如果不是抽象类）则**必须覆写抽象类之中的全部抽象方法**；
- 抽象类对象可以使用对象的向上转型方式，通过子类来进行实例化操作。

```java
abstract class A {
    private String info = "Hello World ." ;
    public void print() {
        System.out.println(info) ;
    }
    public abstract void get() ; // 只声明没有方法体
}
class Impl extends A {
    public void get() {
        System.out.println("Hello MLDN .") ;
    }
}
public class Test {
    public static void main(String args[]) {
        A a = new Impl() ; // 向上转型
        a.print() ; // 自己类定义
        a.get() ; // 子类负责实现
    }
}
```
抽象类可以没有抽象方法，但是即使没有抽象方法也不能直接实例化。

>定义的外部抽象类不能使用 static 关键字修饰，但是定义的是内部抽象类时，这个内部的抽象类使用了static声明之后，就表示是一个外部的抽象类。

### 模板设计模式
现在有三种类型：狗、机器人、人； 
狗具备三种功能：吃、睡、跑； 
机器人具备两个功能：吃、工作； 
人具备四个功能：吃、睡、跑、工作。
现在给出的三个类实际上并没有任何的联系，唯一的联系就是在于一些行为上。

可以使用模板设计模式：
```java
abstract class Action {
    public static final int EAT = 1 ;
    public static final int SLEEP = 3 ;
    public static final int WORK = 5 ;
    public static final int RUN = 7 ;
    public void order(int flag) {
        switch (flag) {
            case EAT :
                this.eat() ;
                break ;
            case SLEEP:
                this.sleep() ;
                break ;
            case WORK :
                this.work() ;
                break ;
            case RUN :
                this.run() ;
                break ;
            case EAT + SLEEP + RUN :
                this.eat() ;
                this.sleep() ;
                this.run() ;
                break ;
            case EAT + WORK :
                this.eat() ;
                this.work() ;
                break ;
            case EAT + SLEEP + RUN + WORK :
                this.eat() ;
                this.sleep() ;
                this.run() ;
                this.work() ;
                break ;
        }
    }
    public abstract void eat() ;
    public abstract void sleep() ;
    public abstract void run() ;
    public abstract void work() ;
}
class Dog extends Action {
    public void eat() {
        System.out.println("小狗在吃。") ;
    }
    public void sleep() {
        System.out.println("小狗在睡。") ;
    }
    public void run() {
        System.out.println("小狗在跑步。") ;
    }
    public void work() {}
}
class Robot extends Action {
    public void eat() {
        System.out.println("机器人喝油。") ;
    }
    public void sleep() {}
    public void run() {}
    public void work() {
        System.out.println("机器人在工作。") ;
    }
}
class Person extends Action {
    public void eat() {
        System.out.println("人在吃饭。") ;
    }
    public void sleep() {
        System.out.println("人在睡觉。") ;
    }
    public void run() {
        System.out.println("人在跑步。") ;
    }
    public void work() {
        System.out.println("人在工作。") ;
    }
}
public class Test {
    public static void main(String args[]) {
        Action act1 = new Dog() ;
        act1.order(Action.EAT + Action.SLEEP + Action.RUN) ;
        Action act2 = new Robot() ;
        act2.order(Action.EAT + Action.WORK) ;
    }
}
```
所有的子类如果要想正常的完成操作，必须按照指定的方法进行覆写才可以，而这个时候抽象类所起的功能就是一个类定义模板的功能。

## 接口
接口属于一种特殊的类，如果一个类定义的时候全部由**抽象方法**和**全局常量**所组成的话，那么这种类就称为接口，但是接口是使用`interface`关键字进行定义的。
```java
interface A { // 定义接口
    public static final String INFO = "Hello World ." ;
    public abstract void print() ;
}
interface B {
    public abstract void get() ;
}
```
接口中同样存在了抽象方法，接口的使用原则如下： 

- 每一个接口必须定义子类，子类使用`implements`关键字实现接口；
- 接口的子类（如果不是抽象类）则**必须**覆写接口之中所定义的**全部**抽象方法；
- 利用接口的子类，采用对象的向上转型方式，进行接口对象的实例化操作。

子类实现接口的语法：
```java
class 子类 [extends 父类] [implemetns 接口1,接口2,...] {}
```
每一个子类可以同时实现多个接口，但是只能继承一个父类。

如果一个类现在即要实现接口又要继承抽象类的话，则应该采用**先继承后实现**的方式完成。
```java
interface A { // 定义接口
    public static final String INFO = "Hello World ." ;
    public abstract void print() ;
}
interface B {
    public abstract void get() ;
}
abstract class C {
    public abstract void fun() ;
}
class X extends C implements A,B { // 同时实现了两个接口
    public void print() { // 方法覆写
        System.out.println("Hello World .") ;
    }
    public void get() {
        System.out.println(INFO) ;
    }
    public void fun() {
        System.out.println("世界，你好！") ;
    }
}
public class Test {
    public static void main(String args[]) {
        A a = new X() ;
        B b = new X() ;
        C c = new X() ;
        a.print() ;
        b.get() ;
        c.fun() ;
    }
}
```
接口之中的全部组成就是抽象方法和全局常量，故以下两种定义接口的最终效果是一样的：
完整定义
```java
interface A { // 定义接口
    public static final String INFO = "Hello World ." ;
    public abstract void print() ;
}
```
简化定义
```java
interface A { // 定义接口
    public String INFO = "Hello World ." ;
    public void print() ;
}
```
>接口之中的访问权限只有一种：**public**，即：定义接口方法的时候就算没有写上public，那么最终也是public。

在Java之中每一个抽象类都可以实现多个接口，但是反过来讲，**一个接口却不能继承抽象类**，可是Java之中，**一个接口却可以同时继承多个接口**，以实现接口的多继承操作。

```java
interface A {
    public void printA() ;
}
interface B {
    public void printB() ;
}
interface C extends A,B { // 一个接口继承了多个接口
    public void printC() ;
}
class X implements C {
    public void printA() {}
    public void printB() {}
    public void printC() {}
}
```
在开发过程中，内部类永远不会受到概念的限制。同样的，在定义内部接口的时候如果使用了**static**，表示是一个外部接口。
```java
interface A {
    public void printA() ;
    static interface B { // 外部接口
        public void printB() ;
    }
}
class X implements A.B {
    public void printB() {
        System.out.println("Hello World .") ;
    }
}
public class Test {
    public static void main(String args[]) {
        A.B temp = new X() ;
        temp.printB() ;
    }
}
```

在实际开发过程中，接口有三大功能：

- 制订操作标准； 
- 表示一种能力； 
- 将服务器端的远程方法视图暴露给客户端。

**抽象类和接口的区别**
| No.  |  区别   |                   抽象类                    |         接口          |
| :--: | :---: | :--------------------------------------: | :-----------------: |
|  1   | 定义关键字 |              abstract class              |      interface      |
|  2   |  组成   |           常量、变量、抽象方法、普通方法、构造方法           |      全局常量、抽象方法      |
|  3   |  权限   |                 可以使用各种权限                 |      只能是public      |
|  4   |  关系   |              一个抽象类可以实现多个接口               | 接口不能够继承抽象类，却可以继承多接口 |
|  5   |  使用   | 子类使用extends继承抽象类抽象类和接口的对象都是利用对象多态性的向上转型，进行接口或抽象类的实例化操作 | 子类使用implements实现接口  |
|  6   | 设计模式  |                  模板设计模式                  |    工厂设计模式、代理设计模式    |
|  7   |  局限   |              一个子类只能够继承一个抽象类              |    一个子类可以实现多个接口     |


### 使用接口定义标准
下面使用接口定义一个 USB 的标准
```java
interface USB { // 操作标准
    public void install() ;
    public void work() ;
}
```
定义 USB 设备
```java
class Phone implements USB {
    public void install() {
        System.out.println("安装手机驱动程序。") ;
    }
    public void work() {
        System.out.println("手机与电脑进行工作。") ;
    }
}

class Print implements USB {
    public void install() {
        System.out.println("安装打印机驱动程序。") ;
    }
    public void work() {
        System.out.println("进行文件打印。") ;
    }
}
```
在电脑上应用此接口
```java
class Computer {
    public void plugin(USB usb) {
        usb.install() ;
        usb.work() ;
    }
}
```
测试
```java
public class Test {
    public static void main(String args[]) {
        Computer c = new Computer() ;
        c.plugin(new Phone()) ; // USB usb = new Phone() ;
        c.plugin(new Print()) ;
    }
}
```
### 工厂设计模式（Factory）
```java
interface Fruit {
    public void eat() ;
}
class Apple implements Fruit {
    public void eat() {
        System.out.println("吃苹果。") ;
    }
}
class Orange implements Fruit {
    public void eat() {
        System.out.println("吃橘子。") ;
    }
}
class Factory {
    public static Fruit getInstance(String className) {
        if ("apple".equals(className)) {
            return new Apple() ;
        }
        if ("orange".equals(className)) {
            return new Orange () ;
        }
        return null ;
    }
}
public class Test {
    public static void main(String args[]) {
        Fruit f = Factory.getInstance(args[0]) ;
        f.eat() ;
    }
}
```
一个接口不和一个固定的子类绑在一起，中间经过一个过渡，降低接口与子类之间的耦合。
### 代理设计模式（Proxy）
```java
interface Subject { // 操作主题
    public void get() ; 
}
class RealSubject implements Subject {
    public void get() {
        System.out.println("真实业务主题") ;
    }
}
class ProxySubject implements Subject {
    private Subject sub = null ;
    public ProxySubject(Subject sub) {
        this.sub = sub ;
    }
    public void prepare() {
        System.out.println("准备操作。") ;
    }
    public void destroy() {
        System.out.println("收尾操作。") ;
    }
    public void get() {
        this.prepare() ;
        this.sub.get() ;
        this.destroy() ;
    }
}
public class Test {
    public static void main(String args[]) {
        Subject sub = new ProxySubject(new RealSubject()) ;
        sub.get() ;
    }
}
```
代理负责完成与真实业务有关的所有辅助性操作。