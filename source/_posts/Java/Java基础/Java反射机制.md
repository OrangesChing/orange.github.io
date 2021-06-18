---
title: Java反射机制
date: 2021-02-19 10:30:48
categories:
  - Java
  - Java基础
---

# 反射是什么？

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

反射就是把java类中的各种成分映射成一个个的Java对象，例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。



## Java反射机制主要功能

1. 在运行时判断任意一个对象所属的类
2. 在运行时构造任意一个类的对象
3. 在运行时判断任意一个类所具有的成员变量和方法
4. 在运行时调用任意一个对象的方法

 

## 对反射提供支持的类

- Class类：代表一个类，位于java.lang包下。
- Field类：代表类的成员变量（成员变量也称为类的属性）。
- Method类：代表类的方法。
- Constructor类：代表类的构造方法。
- Array类：提供了动态创建数组，以及访问数组的元素的静态方法。

## 反射的意义

**反射是框架设计的灵魂**

使用的前提条件：必须先得到代表的字节码的Class，Class类用于表示.class文件（字节码）

# 反射使用

## 示例类：学生类

学生类 继承 人 实现 学习接口

```java
package rejection;
@Component
public class Student extends Human implements Learn{
    private int grade;
    public String className;

    public Student(String name, int grade, int age) {
        super(name, age);
        this.name = name;
        this.grade = grade;
    }

    private Student(String name) {
        super();
        this.name = name;
    }

    public Student(){}

    private void show() {
        System.out.println("这是个名叫"+name+"的"+grade+"年级学生");
    }

    @Override
    public void read() {
        System.out.println("春眠不觉晓");
    }

    @Override
    public boolean remember(String message) {
        System.out.println("记忆消息："+message);
        return true;
    }
}
```
接口：学习

```java
package rejection;
public interface Learn {
    void read();
    boolean remember(String message);
}
```
父类：人

```java
package rejection;
public class Human {
    public String name;
    private int age;

    public Human(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public Human() {}

    public void speak () {
        System.out.println("Hello, My name is " + name);
    }
}
```



## 获取Class对象

三种方式：

1. `实例对象.getClass()`：因为Object类中的getClass方法、因为所有类都继承Object类。从而调用Object类来获取；<u>会先初始化静态块，接着执行非静态块的初始化，最后调用构造函数</u>
2. `类名.Class`：任何数据类型（包括基本数据类型）都有一个“静态”的class属性；<u>不会初始化的静态块，不会初始化参数，不会调用构造函数</u>
3. `Class.forName("全路径名")`**(常用)**；<u>会初始化类静态块，但不会初始化非静态的代码块，也不调用构造函数</u>

> 三种方式的区别
>
> 1 类名.class(也称类字面常量)  方式生成Class对象不会初始化的静态块，不会初始化参数，不会调用构造函数
> 3 Object.getClass()方式生成Class对象会先初始化静态块，接着执行非静态块的初始化，最后调用构造函数


>  在运行期间，一个类/接口，只有一个Class对象产生。 
>
> 三种方式常用**第三种**
> 第一种实例对象都有了还要反射干什么。
> 第二种需要导入类的包，依赖太强，不导包就抛编译错误。
> 第三种，一个字符串可以传入也可写在配置文件中等多种方法。

```java
public static Class<?> getClassByThreeMethod(int method){
        switch (method) {
            // 实例对象.getClass()
            case 1:
                Student student = new Student("Orange",1,6);
                return student.getClass();
                
            // 类名.Class
            case 2:
                return Student.class;
                
            // Class.forName("全路径名")
            case 3:
                try {
                    return Class.forName("rejection.Student");
                } catch (ClassNotFoundException e) {  // 需处理ClassNotFoundException异常
                    e.printStackTrace();
                }
        }
        return null;
    }
```



## 通过反射获取类信息

**获取类信息其实就是调用Class类中的方法**，以下为class类中常见方法

| 获取类信息 | 方法                                                         | 备注                                                     |
| ---------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| 类名       | `public String getName()`                                    | 获取类名                                                 |
| 包         | `public Package getPackage()`                                | 获取所属包                                               |
| 父类       | `public Class<? super T> getSuperclass()`                    | 获取父类class                                            |
| 实现接口   | `public Class<?>[] getInterfaces()`                          | 获取实现接口class                                        |
| 构造方法们 | `Constructor[] getConstructors()`                            | 获取所有public的构造方法（不包含父类构造方法）           |
| 构造方法们 | `Constructor[] getDeclaredConstructors()`                    | 获取所有构造方法（忽略访问控制符，不包含父类构造方法）   |
| 构造方法   | `Constructor getConstructor(Class... parameterTypes)`        | 获取某个构造方法（按参数类型定位，忽略访问控制符）       |
| 构造方法   | `Constructor getDeclaredConstructor(Class... parameterTypes)` | 获取某个构造方法（忽略访问控制符，无法获取父类构造方法） |
| 成员变量们 | `Field[] getFields()`                                        | 获取所有public的成员变量（包含父类属性）                 |
| 成员变量们 | `Field[] getDeclaredFields()`                                | 获取所有成员变量（忽略访问控制符，不包含父类属性）       |
| 成员变量   | `Field[] getField(String name)`                              | 获取某个成员变量（按变量名定位）                         |
| 成员变量   | `Field[] getDeclaredField(String name)`                      | 获取某个成员变量（忽略访问控制符，无法获取父类属性）     |
| 成员方法们 | `Method[] getMethods()`                                      | 获取所有public的成员方法（包含父类方法）                 |
| 成员方法们 | `Method[] getDeclaredMethods()`                              | 获取所有成员方法（忽略访问控制符，不包含父类方法）       |
| 成员方法   | `Method getMethod(String name, Class<?>... parameterTypes)`  | 获取某个成员方法（按变量名和参数类型定位）               |
| 成员方法   | `Method getDeclaredMethod(String name, Class<?>... parameterTypes)` | 获取某个成员方法（忽略访问控制符，无法获取父类方法）     |

示例，获取Student类的信息，整个Student可被拆解为以下部分存储在Class对象中供获取

![拆解Student](Java反射机制\image-20210219164148147.png)

```java
public static void getClassMessage(Class<?> clazz) {
    //获取所有方法并遍历
    System.out.println("获取所有public方法并遍历");
    Method[] methods = clazz.getMethods();
    for (Method method : methods) {
        System.out.println(method);
    }
    System.out.println("获取所有声明方法并遍历（忽略访问修饰符）");
    Method[] methods1 = clazz.getDeclaredMethods();
    for (Method method : methods1) {
        System.out.println(method);
    }

    //获取所有构造方法并遍历
    System.out.println("获取所有public构造方法并遍历");
    Constructor<?>[] constrctors = clazz.getConstructors();
    for (Constructor constrctor : constrctors) {
        System.out.println(constrctor);
    }
    System.out.println("获取所有声明构造方法并遍历（忽略访问修饰符）");
    Constructor<?>[] constrctors1 = clazz.getDeclaredConstructors();
    for (Constructor constrctor : constrctors1) {
        System.out.println(constrctor);
    }

    // 获取父类并打印父类名
    System.out.println("获取父类并打印父类名");
    Class<?> superClazz = clazz.getSuperclass();
    System.out.println(superClazz.getName());

    // 获取所有声明的属性（忽略访问修饰符）
    System.out.println("获取public的属性");
    Field[] declaredField = clazz.getFields();
    for (Field field : declaredField) {
        System.out.println(field);
    }
    System.out.println("获取所有声明的属性（忽略访问修饰符）");
    Field[] declaredField1 = clazz.getDeclaredFields();
    for (Field field : declaredField1) {
        System.out.println(field);
    }
}
```

输出：

```txt
获取所有public方法并遍历
public void rejection.Student.read()
public boolean rejection.Student.remember(java.lang.String)
public void rejection.Human.speak()
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public java.lang.String java.lang.Object.toString()
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
获取所有声明方法并遍历（忽略访问修饰符）
public void rejection.Student.read()
private void rejection.Student.show()
public boolean rejection.Student.remember(java.lang.String)
获取所有public构造方法并遍历
public rejection.Student(java.lang.String,int,int)
获取所有声明构造方法并遍历（忽略访问修饰符）
public rejection.Student(java.lang.String,int,int)
private rejection.Student(java.lang.String)
获取父类并打印父类名
rejection.Human
获取public的属性
public java.lang.String rejection.Student.className
public java.lang.String rejection.Human.name
获取所有声明的属性（忽略访问修饰符）
private int rejection.Student.grade
public java.lang.String rejection.Student.className
```



## 调用/修改所获取的类信息

### Constructor

使用Constructor调用构造方法相当于new

`Constructor对象.newInstance(参数)`

示例：

```
clazz.getConstructor().newInstance(); // 调用无参构造方法
等价于
Student stu = new Student()
```

**注意所有调用的前提是可访问的**

对于构造方法`private Student(String name)`可使用暴力访问来调用

```
Constructor constructor = clazz.getDeclaredConstructor(String.class);
constructor.setAccessible(true);  // 暴力设置为可访问然后调用
constructor.newInstance("Orange")
等价于
Student stu = new Student("Orange")
```

### Field

使用Field中的set为成员变量赋值

`Field对象.set(class对象, 值)`

```
// 获取成员变量
Field f = clazz.getField("className");
//获取一个对象
Object obj = clazz.getConstructor().newInstance();
//为字段设置值
f.set(obj, "一年三班");
//验证
Student stu = (Student)obj;
System.out.println(stu.className);
```

**注意所有调用的前提是可访问的**

对于构造方法`private int grade`可使用暴力访问来修改

```
// 获取成员变量
Field f = clazz.getDeclaredField("grade");
//获取一个对象
Object obj = clazz.getConstructor().newInstance();
//为字段设置值
f.set(obj, 1);
//验证
Student stu = (Student)obj;
System.out.println(stu.grade);
```

### Method

```
// 获取成员方法
Method m = clazz.getMethod("remember", String.class);
//实例化一个Student对象
Object obj = clazz.getConstructor().newInstance();
// 调用
m.invoke(obj, "Orange");
```

特别地，当调用方法为静态方法时`invoke(null, 参数)`

**同样注意所有调用的前提是可访问的，可使用修改访问信号来调用**

参考：https://blog.csdn.net/weixin_42724467/article/details/84311385?utm_medium=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-BlogCommendFromMachineLearnPai2-1.control