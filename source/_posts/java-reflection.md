---
title: Java中的反射
date: 2016-06-30 19:56:52
tags: Java
---
最近在项目中为了减少切换数据而带来的额外操作，需要用到反射的知识，于是了解了一下 Java 高级语法中的反射。
## Class类
在Java中万物皆对象，类也是对象。
>java中的静态成员、普通成员数据类型不是对象，但是普通数据类型有包装对象。

<!-- more -->
**类是 java.lang.Class 的实例对象。**
Class类的构造函数是一个私有对象，故不能通过 new 关键字实例化。
Class 类实例化的三种表示方法：
```java
 class Foo{}
 //Foo实例化对象表示
 Foo foo1 = new Foo();
 
 //Foo这个类是Class类的实例化对象
 //第一种表示方法（实际上说明每个类都有一个隐含的静态成员变量class）
 Class c1 = Foo.class;
 
 //第二种表达方式，通过getClass方法
 Class c2 = foo1.getClass();
 
 //第三种表达方法
 Class c3 = Class.forName("Foo的完整类名");
 
 //可以通过类的类类型创建该类的对象实例（通过c1 or c2 or c3创建Foo的实例对象）
 //需进行强制类型转换，应该做异常处理
 Foo foo = (Foo) c1.newInstance();//需要有无参数的构造方法
```

`类类型` ： Class类的实例对象称为该类的类类型。上面的c1 ,c2 表示了Foo类的类类型(class type)，一个类只可能是Class类的一个实例对象，故 c1 == c2 == c3。

## 动态加载类
**Class.forName("类的全称")** 不仅表示了，类的类类型，还代表了动态加载类。编译时刻加载类是静态加载类、运行时刻加载类是动态加载类。
`new` 创建的对象是静态加载类，在编译时就需要加载所有的可能使用到的类。
比如下面代码：
```java
class Office{
    public static void main(String[] args){
        if("Word".equal(args[0]){
            Word w = new Word();
            w.start();
        }
        if("Excel".equal(args[0]){
            Excel w = new Excel();
            w.start();
        }
    }
}
```
这个程序中，只有当 Word 类与 Excel 类都实现了才能通过编译，但是有时否则有一个有问题就不能编译。这是非常不友好的，比如说如果有100多个 if 语句分别对应着不同的功能，如果一个功能有问题那么所有的功能都不能实现，这是非常糟糕的一件事情。
如果用到**动态加载类**，这种情况可以很好地避免：
``` java
class OfficeBetter{
    public static void main(String[] args){
        try{
            //动态加载类，在运行时刻加载
            Class c = Class.forName(args[0]);
            //在类的是实例化中，由于需要用到类的类型转换，要提前知道类名，可通过接口统一
            OfficeAble oa = (OfficeAble) c.newInstance();
            oa.start();
        }
        catch(Exception e){
            e.printStackTrace();
        }
    }
}
//标准接口
interface OfficeAble{
    public void start();
}

//在Word类的实现中就需要继承接口OfficeAble
class Word implements OfficeAble{
    public void start(){
        System.out.println(".......");
    }
}
```
现在，有一个功能就能实现一个功能，通过动态加载类，以后如果需要添加其他功能，之前的类不需要修改，只需要实现相应的接口就行了。程序的许多在线快速升级也是这样的，不需要重新编译已有的类，只需要编译新加入的功能类。

## 获取对象方法
基本数据类型、void 关键字都有类类型。
```java
Class c1 = int.class;//int 的类类型
Class c2 = String.class;//java.lang.String String类的类类型  
Class c3 = double.class;
Class c4 = Double.class;
Class c5 = void.class;

System.out.println(c2.getName());
System.out.println(c2.getSimpleName());//不包含包名的类的名称
```
下面程序实现获取类的信息，获取类的任何信息得到第一步，就是拿到类的类类型：
```java
/**
 * 打印类的信息，包括类的成员函数、成员变量(只获取成员函数)
 * @param obj 该对象所属类的信息
 */
public static void printClassMethodMessage(Object obj){
	//要获取类的信息  首先要获取类的类类型
	Class c = obj.getClass();//传递的是哪个子类的对象  c就是该子类的类类型
	//获取类的名称
	System.out.println("类的名称是:"+c.getName());
	/*
	 * Method类，方法对象
	 * 一个成员方法就是一个Method对象
	 * getMethods()方法获取的是所有的public的函数，包括父类继承而来的
	 * getDeclaredMethods()获取的是所有该类自己声明的方法，不问访问权限
	 */
	Method[] ms = c.getMethods();//c.getDeclaredMethods()
	for(int i = 0; i < ms.length;i++){
		//得到方法的返回值类型的类类型
		Class returnType = ms[i].getReturnType();
		System.out.print(returnType.getName()+" ");
		//得到方法的名称
		System.out.print(ms[i].getName()+"(");
		//获取参数类型--->得到的是参数列表的类型的类类型
		Class[] paramTypes = ms[i].getParameterTypes();
		for (Class class1 : paramTypes) {
			System.out.print(class1.getName()+",");
		}
		System.out.println(")");
	}
}
```
## 获取类的成员变量与构造函数
获取成员变量的信息：
```java
/**
 * 获取成员变量的信息
 * @param obj
 */
public static void printFieldMessage(Object obj) {
	Class c = obj.getClass();
	/*
	 * 成员变量也是对象
	 * java.lang.reflect.Field
	 * Field类封装了关于成员变量的操作
	 * getFields()方法获取的是所有的public的成员变量的信息
	 * getDeclaredFields获取的是该类自己声明的成员变量的信息
	 */
	//Field[] fs = c.getFields();
	Field[] fs = c.getDeclaredFields();
	for (Field field : fs) {
		//得到成员变量的类型的类类型
		Class fieldType = field.getType();
		String typeName = fieldType.getName();
		//得到成员变量的名称
		String fieldName = field.getName();
		System.out.println(typeName+" "+fieldName);
	}
}
```
获取构造函数信息：
```java
/**
* 打印对象的构造函数的信息
* @param obj
*/
public static void printConMessage(Object obj){
Class c = obj.getClass();
/*
 * 构造函数也是对象
 * java.lang. Constructor中封装了构造函数的信息
 * getConstructors获取所有的public的构造函数
 * getDeclaredConstructors得到所有的构造函数
 */
//Constructor[] cs = c.getConstructors();
Constructor[] cs = c.getDeclaredConstructors();
for (Constructor constructor : cs) {
	System.out.print(constructor.getName()+"(");
	//获取构造函数的参数列表--->得到的是参数列表的类类型
	Class[] paramTypes = constructor.getParameterTypes();
	for (Class class1 : paramTypes) {
		System.out.print(class1.getName()+",");
	}
	System.out.println(")");
}
```
## 方法反射的基本操作
方法的名称和方法的参数列表才能唯一决定某个方法。方法反射的操作 ：`method.invoke(对象，参数列表)`
```java
public static void main(String[] args) {
	   //要获取print(int ,int )方法  1.要获取一个方法就是获取类的信息，获取类的信息首先要获取类的类类型
		A a1 = new A();
		Class c = a1.getClass();
		/*
		 * 2.获取方法 名称和参数列表来决定  
		 * getMethod获取的是public的方法
		 * getDelcaredMethod自己声明的方法
		 */
	    try {
	    	Method m = c.getMethod("print", int.class,int.class);
	    	//Method m =  c.getMethod("print", new Class[]{int.class,int.class});也可
	    	
	    	//方法的反射操作  
	    	//a1.print(10, 20);方法的反射操作是用m对象来进行方法调用 和a1.print调用的效果完全相同
	    	//Object o = m.invoke(a1,new Object[]{10,20});或
	    	Object o = m.invoke(a1, 10,20);
	    	
	    	//获取方法print(String,String)
             Method m1 = c.getMethod("print",String.class,String.class);
             //用方法进行反射操作
             o = m1.invoke(a1, "hello","WORLD");
             
            Method m2 = c.getMethod("print");
           //Method m2 = c.getMethod("print", new Class[]{});也可
                
            // m2.invoke(a1, new Object[]{});
            m2.invoke(a1);
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
     
	}
}
class A{
	public void print(){
		System.out.println("helloworld");
	}
	public void print(int a,int b){
		System.out.println(a+b);
	}
	public void print(String a,String b){
		System.out.println(a.toUpperCase()+","+b.toLowerCase());
	}
```
## 集合中的泛型
Java中集合的泛型，是防止错误输入的，只在编译阶段有效，绕过编译就无效了。一般来说，java中数组都是存储相同类型的元素，但是可以通过反射来绕过编译，存储不同类型的元素，但此时不能用 for……in 遍历。
```java
public static void main(String[] args) {
		ArrayList list = new ArrayList();
		
		ArrayList<String> list1 = new ArrayList<String>();
		list1.add("hello");
		//list1.add(20);错误的
		Class c1 = list.getClass();
		Class c2 = list1.getClass();
		System.out.println(c1 == c2);
		//反射的操作都是编译之后的操作
		
		/*
		 * c1==c2结果返回true说明编译之后集合的泛型是去泛型化的
		 * Java中集合的泛型，是防止错误输入的，只在编译阶段有效，
		 * 绕过编译就无效了
		 * 验证：可以通过方法的反射来操作，绕过编译
		 */
		try {
			Method m = c2.getMethod("add", Object.class);
			m.invoke(list1, 20);//绕过编译操作就绕过了泛型
			System.out.println(list1.size());   //2
			System.out.println(list1);          //[hello,20]
			/*for (String string : list1) {
				System.out.println(string);
			}*///现在不能这样遍历
		} catch (Exception e) {
		  e.printStackTrace();
		}
	}
```