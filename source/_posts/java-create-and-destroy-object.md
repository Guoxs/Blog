---
title: Java 创建与销毁对象
date: 2016-10-04 23:03:23
tags: Java
---
本文《Effective Java》第一章的读书笔记，主要内容是：在java程序升设计中，何时以及如何创建对象，何时以及如何避免创建对象，如何确保对象及时地销毁以及销毁对象后的清理工作。

## 静态工厂方法代替构造器
创建类的实例，有两种方法：
① 利用共有构造器
② 类提供一个共有的静态工厂方法，它返回类的实例

第二种方法类似于这样：
```java
public static Boolean valueOf(boolean b){
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```
<!--more-->
### 静态工厂方法的优点
**① 静态工厂方法有名字**

一个类只能有一个带有指定签名的构造器，如果要提供多个构造器，则只能在参数上做文章。这样不利于编写文档。由于静态方法有名称，故可以很好的区分不同的实例化静态方法。

**② 静态方法不必在每次调用时都创建一个新的对象**

这个特点可以使不可变类提前创建好实例然后缓存起来重复利用，从而避免创建不必要的对象。这类方法类似于**享元（Flyweight）模式**，避免大量拥有相同内容的小类的开销(如耗费内存),使大家共享一个类(元类)。
静态工厂方法能够为重复的调用返回相同的对象，这样有助于类总能严格控制在某个时刻哪些实例应该存在，这种类称为**实例受控的类（instance-contrilled）**。

**③ 静态工厂方法可以返回原返回类型的任何子类型对象**

使用这种方法在返回对象的类时有很大的灵活性，API可以返回对象，同时又不会使对象的类变为公有的。以这种方法隐藏实现类会使API变得十分简洁。这项技术适用于基于接口的框架。使用这种静态方法时，甚至要求客户端通过接口来引用被访问的对象，而不是通过它的实现类来引用被返回的对象。

**④ 创建参数化类型实例的时候可以使代码变得更加简洁**
对于参数化类的构造器，即使类型参数很明显，在调用时也必须指明：
```java
Map<String, List<String>> m = new HashMap<String, List<String>>();
```
如果类型参数很长，那么这种说明将是很难受的。
如果使用静态工厂方法，编译器就可以替你找到类型参数，这种被称作**类型推导（type inference）**，例如，假设 HashMap 提供了这个静态工厂：
```java
public static <K,V> HashMap<K,V> newInstance(){
    return new HashMap<K,V>();
}
```
那么上面那句繁琐的声明就可以写成这样：
```java
Map<String, List<String>> m = HashMap.newInstance();
```

### 静态工厂方法的缺点

**① 类如果不含共有或者受保护的构造器，就不能被子类化**

**② 静态工厂方法与其他静态方法实际上没有任何区别**
在API文档中，没有任何标志说明某个静态方法可用于实例化类，因此，对于那些没有构造器的类来说，想要查明如何实例化一个类将是很困难的。当然，也许以后 Javadoc 会注意到这一点。

一些静态工厂方法的惯用名称：

- valueOf
- of
- getInstance
- newInstance
- getType
- newType

**切记第一反应就提供共有的构造器，而不考虑静态工厂。**

## 遇到多个构造器参数时要考虑用构建器
静态工厂和构造器有个共同的局限性：**它们都不能很好地扩展到大量的可选参数。**
**重叠构造器模式（telescoping constructor）**可以很好地解决这个问题。
```java
public class NutritionFacts {
    private final int servingSize; // (mL) required
    private final int servings; // (per container) required
    private final int calories; // optional
    private final int fat; // (g) optional
    private final int sodium; // (mg) optional
    private final int carbohydrate; // (g) optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this(servingSize, servings, calories, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this(servingSize, servings, calories, fat, sodium, 0);
    }
    
    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```
不过，这种模式当需要传递许多参数的时候，代码也会变得难以编写。要仔细弄清楚每个参数的含义才能正确地使用构造器。

另一种替代方法为**JavaBeans模式**，在这种模式下，调用一个无参数构造器创建对象，然后调用setter方法来设置每个必要的参数，以及每个相关的可选参数。这种模式虽然弥补了重叠构造器模式的不足，但是也有自身的缺陷。因为构造过程被分到几个调用中，在构造过程中JavaBeans可能处于不一致的状态，类无法仅仅通过检验构造器参数来保证一致性。

比较完美地替代方法是**Builder模式**：
```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
       
       // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int carbohydrate = 0;
        private int sodium = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val)
            { calories = val; return this; }
        public Builder fat(int val)
            { fat = val; return this; }
        public Builder carbohydrate(int val)
            { carbohydrate = val; return this; }
        public Builder sodium(int val)
            { sodium = val; return this; }
            
        public NutritionFacts build() {
            return new NutritionFacts(this);}
    }
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```
这种方法参数传递是一个链式操作：
```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240,8).calories(100).sodium(35).carbohydrate(27).build();
```
Builder模式非常之灵活，可以利用单个builder构建多个对象，builder参数可以在创建对象期间进行调整，也可以随着不同的对象而改变。
带有builder实例的方法通常利用有限制的通配符类型来约束构建器的类型参数：
```java
Tree buildTree(Builder<? extends Node> nodeBuilder){...}
```
**如果累个构造器或者静态工厂方法中具有多个参数，设计这种类时，Builder模式就是中不错的选择，特别是当大多数参数都是可选的时候。**

## 用私有构造器或者枚举类型强化Singleton属性
Singleton指的是仅仅被实例化一次的类，它通常代表那些本质上唯一的系统组件。

实现Singleton有三种方法：

**① 共有静态成员是一个final域**
```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```
私有构造器仅被调用一次，用来实例化公有的静态final域Elvis.INSTANCE。

公有域方法的主要好处在于，组成类的成员声明很清楚地表明了这个类是一个Singleton: 共有的静态域是final的，所以该域将总是包含相同的对象引用。
**② 公有的成员是个静态工厂方法**
```java
// Singleton with static factory
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }
    public void leaveTheBuilding() { ... }
}
```
对于静态方法Elvis.getInstance的所有调用，都会返回同一个对象引用，所以永远不会创建其他Elvis实例。

>**注意：** 享有特权的客户端可以借助 AccessibleObject.setAccessible 方法，通过反射机制调用私有构造器。要抵御这种攻击，可以修改构造器，让它在被要求创建第二次的时候抛出异常。

**③ 使用单个元素枚举类型**
```java
// Enum singleton - the preferred approach
public enum Elvis {
    INSTANCE;
    public void leaveTheBuilding() { ... }
}
```
这种方式在功能上和公有域方法相近，但是它更加简洁，无偿提供了序列化机制，绝对泛指多次实例化，即使面对复杂的序列化或者反射攻击的时候。**单元素的枚举类已经成为实现Singleton的最佳方案。**

## 通过私有构造器强化不可实例化的能力

有时候我们需要编写只包含静态方法和静态域的类，这种时候我们不希望类被实例化。但是即使在不提供构造器的时候，编译器也会自动提供一个共有的、无参的缺省构造器。
这个时候可以使用私有构造器，使类不能被实例化：
```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
    throw new AssertionError();
    }
    ... // Remainder omitted
}
```
由于只有类中不包含显式的构造器时编译器才会生成默认构造器，且私有的构造器外部不可以访问，所以这样就完全限制了类不可实例化。

>AssertionError不是必需的，但是它可以避免不小心在类的内部调用构造器。

## 避免创建不必要的对象
一般来说，最好能重用对象而不是每次需要的时候就创建一个相同功能的新对象，重用方式既快速又流行。如果对象是不可变的，它始终可以被重用。

对于同时提供静态工厂方法与构造器的不可变类，通常可以使用静态工厂方法而不是构造器，以避免创建不必要的对象。

除了可重用不可变的对象外，还可以重用那些一直不会改变的可变对象，例如：
```java
public class Person {
    private final Date birthDate;
    // Other fields, methods, and constructor omitted
    // DON'T DO THIS!
    public boolean isBabyBoomer() {
        // Unnecessary allocation of expensive object
        Calendar gmtCal =
            Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomStart = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        Date boomEnd = gmtCal.getTime();
        return birthDate.compareTo(boomStart) >= 0 &&
                birthDate.compareTo(boomEnd) < 0;
    }
}
```
isBabyBoomer 每次调用都会新建一个Calendar, TimeZone和两个 Date对象，这是不必要的。可以使用**静态的初始化器（initiallizer）**避免这种低效的情况：
```java
class Person {
    private final Date birthDate;
    // Other fields, methods, and constructor omitted
    /**
    * The starting and ending dates of the baby boom.
    */
    private static final Date BOOM_START;
    private static final Date BOOM_END;
    
    static {
        Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
        gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_START = gmtCal.getTime();
        gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
        BOOM_END = gmtCal.getTime();
    }
    public boolean isBabyBoomer() {
        return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
    }
}
```
改进后的Person类只在初始化的时候创建Calendar, TimeZone 和 Date 对象，而不是每次调用都重新创建。

>如果想进一步优化，比如在第一次调用时才初始化，这种可以通过**延迟初始化（lazily initializing）**实现。但是不建议这么做，因为会使方法的实现变得更加复杂。

另外，也要注意**自动装箱**所创建的不必要的对象。

## 消除过期的对象引用
Java虽然有自动垃圾回收机制，但是也会存在**内存泄漏**的情况。比如：
```java
    public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }
    
    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }
    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }
    
    /**
    * Ensure space for at least one more element, roughly
    * doubling the capacity each time the array needs to grow.
    */
    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```
这段程序有 “内存泄漏”，如果一个栈先增长，后收缩，那么从栈中弹出的对象将不会当做垃圾回收。这是因为**栈每部维护着这些对象的过期引用（obsolete reference）**。过期引用是永远不会再被解除的引用，本例中凡是在elements数组的“活动部分”之外的任何引用都是过期的，活动部分是指elements中下标小于size的那些元素。

这类问题的处理很简单,字需要按如下方式修改pop()函数：
```java
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference
    return result;
}
```
>一旦数字元素变成非活动元素的一部分，程序员就手工清空这些数组元素。

**内存泄漏的另一个常见来源是缓存**。当所要的缓存项的生命周期是由该键的外部引用而不是由值来决定，可以使用 WeakHashMap 来代表缓存，当缓存项过期之后，它们会被自动删除。

**内存泄漏的第三个常见来源是监听器和其他回调。**确保回调立即被当作垃圾回收的最佳方法是只保存它们的弱引用，例如，只将它们保存为 WeakHashMap 的键。

## 避免使用finalizer
终结方法通常是不可预测的，也是很危险的，一般情况下是不必要的。使用终结方法会导致行为不稳定，降低性能，以及可移植性问题。

**终结方法的缺点**

- 从一个对象变得不可达开始，到它的终结方法被执行，所花费的这段时间是任意长的。
- 如果未捕获的异常在终结过程中被抛出，那么这种异常可以被忽略，并且该对象的终结过程也会终止。
- 终结方法非常严重的性能损失。
