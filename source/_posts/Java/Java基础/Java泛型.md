---
title: Java泛型
date: 2021-02-19 10:30:48
tags:
  - 反射
categories:
  - Java
  - Java基础
---

# 泛型概述

## 什么是泛型

泛型，即**参数化类型**，将类型由原来的具体的类型替换为参数形式变量，然后在使用/调用时传入具体的类型（类型实参）；也就是说在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

泛型在面向对象编程及各种设计模式中有非常广泛的应用

## 为什么需要泛型

### 保证类型的安全性

以下述程序为例：

```
List arrayList = new ArrayList();
arrayList.add("aaaa");
arrayList.add(100);

for(int i = 0; i< arrayList.size();i++){
    String item = (String)arrayList.get(i);
    Log.d("泛型测试","item = " + item);
}
```

毫无疑问，程序的运行结果会以崩溃结束：

```
java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
```

ArrayList可以存放任意类型，例子中添加了一个String类型，添加了一个Integer类型，再使用时都以String的方式使用，因此程序崩溃了。

**为了解决在编译阶段就能发现类似这样的问题并解决，泛型应运而生**

我们将第一行声明初始化list的代码更改一下，编译器会在编译阶段就能够帮我们发现类似这样的问题。

```
List<String> arrayList = new ArrayList<String>();
...
//arrayList.add(100); 在编译阶段，编译器就会报错
```

### 避免不必要的装箱、拆箱操作，提高程序的性能

泛型变量固定了类型，使用的时候就已经知道是值类型还是引用类型，可以避免不必要的装箱、拆箱操作

例如：

使用泛型之前，我们使用object代替。

```
object a=1;//由于是object类型，会自动进行装箱操作。

int b=(int)a;//强制转换，拆箱操作。当次数多了以后会影响程序的运行效率。
```

使用泛型之后

```
public static T GetValue<T>(T a) {
　　return a;
}

public static void Main() {
　　int b=GetValue<int>(1);//使用这个方法的时候已经指定了类型是int，所以不会有装箱和拆箱的操作。
}
```

### 提高方法、算法的重用性



## 特性

**泛型只在编译阶段有效**。看下面的代码：

```java
List<String> stringArrayList = new ArrayList<String>();
List<Integer> integerArrayList = new ArrayList<Integer>();

Class classStringArrayList = stringArrayList.getClass();
Class classIntegerArrayList = integerArrayList.getClass();

// 输出结果：D/泛型测试: 类型相同
if(classStringArrayList.equals(classIntegerArrayList)){
    Log.d("泛型测试","类型相同");
}
```

通过上面的例子可以证明：**在编译之后程序会采取去泛型化的措施**。在编译过程中，正确检验泛型结果后，<u>会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法</u>。也就是说，泛型信息不会进入到运行时阶段。

**泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型**

# 泛型的使用

泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

##  泛型类

泛型类型用于类的定义中，被称为泛型类。通过泛型可以完成对一组类的操作对外开放相同的接口

最典型的就是各种容器类，如：List、Set、Map。

泛型类的最基本写法：

`class 类名称 <泛型标识>`

泛型标识符可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型



例：一个最普通的泛型类

```java
//在实例化泛型类时，必须指定T的具体类型
public class GenericClass<T> {
    private T t;
    public GenericClass(T t) {
        this.t = t;
    }
    public T get() {
        return t;
    }
    public void set(T t) {
        this.t = t;
    }
}

// 实例化
//泛型的类型参数只能是类类型（包括自定义类），不能是简单类型
GenericClass<String> genericClass = new GenericClass<String>("Orange");
GenericClass<Integer> genericClass2 = new GenericClass<Integer>(123);  
```

**注意**：在使用泛型的时候如果传入泛型实参，则会根据传入的泛型实参做相应的限制，此时泛型才会起到本应起到的限制作用。如果不传入泛型类型实参的话，在泛型类中使用泛型的方法或成员变量定义的类型可以为任何的类型。

例：

```java
GenericClass genericClass = new GenericClass("orange");
System.out.println(genericClass.get().getClass().getName());  // 输出：java.lang.String
genericClass.set(123);
System.out.println(genericClass.get().getClass().getName());  // 输出：java.lang.Integer
```

 **注意：**

- 泛型的类型参数只能是类类型，不能是简单类型。
- 不能对确切的泛型类型使用instanceof操作。如下面的操作是非法的，编译时会出错。

　　`if(ex_num instanceof Generic<Number>){ }`

## 泛型接口

泛型接口与泛型类的定义及使用基本相同

泛型接口的最基本写法：

`interface 接口名称 <泛型标识>`

例如：

```java
//定义一个泛型接口
public interface GenericInterface<T> {
    T getMsg();
}
```

对于实现泛型接口的类：

- 未传入泛型实参时，与泛型类的定义相同，在声明类的时候，需将泛型的声明也一起加到类中；如果不声明泛型，如：`class FruitGenerator implements Generator<T>`，编译器会报错：`Unknown class`
- 传入泛型实参时，则所有使用泛型的地方都要替换成传入的实参类型


```java
// 未传入泛型实参时，申明类时要加入泛型声明
public class GeneralClass<T> implements GenericInterface{
    private T msg;

    public GeneralClass (T msg) {
        this.msg = msg;
    }

    @Override
    public T getMsg() { 
        System.out.println("GeneralClass get message " + msg);
        return msg;
    }
}

// 传入泛型实参时
public class GeneralClass implements GenericInterface<String> {

    private String msg = "orange";

    @Override
    public String getMsg() {
        System.out.println("GeneralClass get message " + msg);
        return msg;
    }
}
```

## 泛型方法

泛型类，是在实例化类的时候指明泛型的具体类型；

泛型方法，是在调用方法的时候指明泛型的具体类型 

泛型方法的最基本写法：

`访问控制符 <泛型标识> 返回类型 方法名(形参...)`

- 访问控制符 与 返回值中间的`泛型标识<T>`非常重要，可以理解为声明此方法为泛型方法。<u>只有声明了泛型标识`<T>`的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法</u>
- `泛型标识<T>`表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T
- 如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法
```java
public class GenericMethod<T>{     
   private T key;

   public Generic(T key) {
       this.key = key;
   }
      
   //虽然在方法中使用了泛型，但是这并不是一个泛型方法。只是类中一个普通的成员方法，只不过他的返回值是在声明泛型类已经声明过的泛型。
   public T getKey(){
        return key;
   }
            
   //这也不是一个泛型方法，这就是一个普通的方法，只是使用了Generic<Number>这个泛型类做形参而已。
   public void showKeyValue1(Generic<Number> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
   }
    
   //这也不是一个泛型方法，这也是一个普通的方法，只不过使用了泛型通配符?
   //同时这也印证了泛型通配符章节所描述的，?是一种类型实参，可以看做为Number等所有类的父类
   public void showKeyValue2(Generic<?> obj){
        Log.d("泛型测试","key value is " + obj.getKey());
   }
   
   // 泛型方法
   public <T> void print(T t) {
        System.out.println("GenericMethod print " + t);
   }
          
      
   /**
   * 这个方法是有问题的，编译器会为我们提示错误信息："UnKnown class 'E' "
   * 虽然我们声明了<T>,也表明了这是一个可以处理泛型的类型的泛型方法。
   * 但是只声明了泛型类型T，并未声明泛型类型E，因此编译器并不知道该如何处理E这个类型 
   public <T> T showKeyName(Generic<E> container){
        ...
   }
   * 这个方法也是有问题的，编译器会为我们提示错误信息："UnKnown class 'T' "
   * 对于编译器来说T这个类型并未项目中声明过，因此编译也不知道该如何编译这个类。
   * 所以这也不是一个正确的泛型方法声明
   public void showkey(T genericObj){
      ...
   }
   */
}
```

### 类中的泛型方法

当然这并不是泛型方法的全部，泛型方法可以出现杂任何地方和任何场景中使用。但是有一种情况是非常特殊的，当泛型方法出现在泛型类中时，我们再通过一个例子看一下

```java
class GenerateTest<T>{
	public void show_1(T t){
		System.out.println(t.toString());
	}

	//在泛型类中声明了一个泛型方法，使用泛型E，这种泛型E可以为任意类型。可以类型与T相同，也可以不同。
	//由于泛型方法在声明的时候会声明泛型<E>，因此即使在泛型类中并未声明泛型，编译器也能够正确识别泛型方法中识别的泛型。
	public <E> void show_3(E t){
		System.out.println(t.toString());
	}

	//在泛型类中声明了一个泛型方法，使用泛型T，注意这个T是一种全新的类型，可以与泛型类中声明的T不是同一种类型。
	public <T> void show_2(T t){
		System.out.println(t.toString());
	}
}
```

### 静态方法与泛型

静态方法有一种情况需要注意一下，那就是在类中的静态方法使用泛型：<u>静态方法无法访问类上定义的泛型</u>；如果静态方法操作的引用数据类型不确定的时候，必须要将泛型定义在方法上。

即：如果静态方法要使用泛型的话，必须将静态方法也定义成泛型方法 。

```
public class StaticGenerator<T> {
    ....
    ....
    /**
     * 如果在类中定义使用泛型的静态方法，需要添加额外的泛型声明（将这个方法定义成泛型方法）
     * 即使静态方法要使用泛型类中已经声明过的泛型也不可以。
     * 如：public static void show(T t){..},此时编译器会提示错误信息：
          "StaticGenerator cannot be refrenced from static context"
     */
    public static <T> void show(T t){

    }
}
```

### 泛型方法总结

泛型方法能使方法独立于类而产生变化，以下是一个基本的指导原则：

<u>无论何时，如果你能做到，你就该尽量使用泛型方法。也就是说，如果使用泛型方法将整个类泛型化，那么就应该使用泛型方法</u>

## 泛型通配符

`Ingeter`是`Number`的一个子类，前面验证过`Generic<Ingeter>`与`Generic<Number>`实际上是相同的一种基本类型。但在使用`Generic<Number>`作为形参的方法中，不能使用`Generic<Ingeter>`的实例传入，测试案例如下：

```
public void showKeyValue1(Generic<Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}

Generic<Integer> gInteger = new Generic<Integer>(123);
Generic<Number> gNumber = new Generic<Number>(456);

showKeyValue(gNumber);

// showKeyValue这个方法编译器会为我们报错：Generic<java.lang.Integer> 
// cannot be applied to Generic<java.lang.Number>
// showKeyValue(gInteger);
```

通过提示信息我们可以看到`Generic<Integer>`不能被看作为``Generic<Number>`的子类。由此可以看出:同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。

这时可使用类型通配符`?`表示：

```
public void showKeyValue1(Generic<?> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
```

类型通配符一般是使用`?`代替具体的类型实参，注意了，此处`?`是类型实参，而不是类型形参 。此处的`?`和Number、String、Integer一样都是一种实际的类型，可以把`?`看成所有类型的父类。是一种真实的类型。

可以解决当具体类型不确定的时候，这个通配符就是`?`。当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用`?`通配符来表未知类型。

##  泛型上下边界

在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：

类型实参只准传入某种类型的父类`<? super 类名>`

类型实参只准传入某种类型的子类`<? extends 类名>`

```
public void showKeyValue1(Generic<? extends Number> obj){
    Log.d("泛型测试","key value is " + obj.getKey());
}
Generic<String> generic1 = new Generic<String>("11111");
Generic<Integer> generic2 = new Generic<Integer>(2222);
Generic<Float> generic3 = new Generic<Float>(2.4f);
Generic<Double> generic4 = new Generic<Double>(2.56);

//这一行代码编译器会提示错误，因为String类型并不是Number类型的子类
//showKeyValue1(generic1);

showKeyValue1(generic2);
showKeyValue1(generic3);
showKeyValue1(generic4);
```

注意：在泛型方法中添加上下边界限制的时候，必须在权限声明与返回值之间的<T>上添加上下边界，即在泛型声明的时候添加

```
//public <T> T showKeyName(Generic<T extends Number> container)，编译器会报错："Unexpected bound"
public <T extends Number> T showKeyName(Generic<T> container){
    System.out.println("container key :" + container.getKey());
    T test = container.getKey();
    return test;
}
```

通过上面的两个例子可以看出：泛型的上下边界添加，必须与泛型的声明在一起 。

转载整理至：https://blog.csdn.net/s10461/article/details/53941091