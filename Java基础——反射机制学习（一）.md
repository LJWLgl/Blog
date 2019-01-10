---
title: "Java基础——反射机制学习（一）"
date: 2019-01-06 18:49:23
tags: Java反射
categories: 夯实基础
summary:  "本篇文章主要介绍Java反射的基本知识，以供自己日后查阅。" 
---
题主刚学Java的时候就了解过Java反射，但是在实践开发中使用的并不是很多，所以也一直未深入了解过，最近在看一些公司内部框架的源码，发现了很多功能都是通过Java反射来实现的。<!-- more --> 本篇文章主要介绍Java反射的基本知识，以供自己日后查阅。先介绍一下Java反射的定义
>  JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
Java反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它在编译期不需要知道运行的对象是谁，Java反射实际操作对象是.class文件（字节码文件）
### 获得Class对象
1.通过Class类的forName静态方法，JDBC开发中常用此方法加载驱动
```
Class<?> c1 = Class.forName("java.lang.Integer");
```
2.通过调用对象的getClass()方法
```
Integer i1 = 1;
Class<?> c2 = i1.getClass();
```
3.直接获取类的class
```
Class<?> c3 = Integer.class;
```
### 判断是否为某个类的实例
判断是否是某个类的实例一般用`instanceof`关键字，同时我们也可借助Class对象的`isInstance()`方法来判断，如下面的代码
```
Object obj = "Java";
// 使用instanceof关键字
System.out.println(obj instanceof  String);
// 使用isInstance方法
System.out.println(String.class.isInstance(obj));
```
`isInstance()`方法是一个native方法，它的方法签名如下：
```
public native boolean isInstance(Object obj);
```
### 创建实例
通过Java反射创建对象有两种方法：
1.通过Class对象的newInstance方法来创建Class对象对应的实例
```
Class<?> c = String.class;
Object s = c.newInstance();
```
2.先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance方法来创建对象的实例，**通过这种方法可以指定构造器来构造对象。**
```
Class<?> cc = String.class;
Constructor constructor =  cc.getConstructor(String.class);
Object ss = constructor.newInstance("Java");
System.out.println(ss);
```
### 获取类名及变量
直接调用Class对象的getName()即可获取类名（包含package）
```
Class<?> c = String.class;
System.out.println(c.getName());
```
获取类的变量有下面两种方法
1.通过Class对象的getField方法获取类的所有变量，需要注意是getField()会获取该类及其父类的全部公有变量
```
// Book是自定义的类
Class<?> c = Book.class;
Field[] fields = c.getFields();

```
2.如果还想获取对象的私有变量，可以通过getDeclaredFields()方法，该方法会返回该类的所有变量，不论访问权限
```
// Book是自定义的类
Class<?> c = Book.class;
Field[] fields = c.getDeclaredFields();
for (Field field : fields) {
    // 获取访问权限
    System.out.println(field.getModifiers());
    // 获取变量类型及名称
    System.out.println(field.getType() + " " + field.getName());
}
```
### 获取方法
获取某个Class对象的方法，主要有以下几种方法：
1.`getDeclaredMethods`方法返回类或接口声明的所有方法，包括公共、保护、默认访问和私有方法，但不包括继续的方法。
```
public Method[] getDeclaredMethods() throws SecurityException
```
2.`getMethods`，返回该类所有的公共的方法以及继续的类公用方法
```
public Method[] getMethods() throws SecurityException 
```
3.`getMethod`返回一个特定的方法，第一个参数为方法名称，后面的参数是方法参数对应的Class对象
```
public Method getMethod(String name, Class<?>... parameterTypes)
```
获取Class对象的具体列子
```
Class<?> cc = Book.class;
Method[] methods = cc.getMethods();
for (Method method : methods) {
    //获取并输出方法的访问权限（Modifiers：修饰符）
    int modifiers = method.getModifiers();
    System.out.print(Modifier.toString(modifiers) + " ");
    //获取并输出方法的返回值类型
    Class returnType = method.getReturnType();
    System.out.print(returnType.getName() + " " + method.getName() + "( ");
    //获取并输出方法的所有参数
    Parameter[] parameters = method.getParameters();
    for (Parameter parameter: parameters) {
        System.out.print(parameter.getType().getName() + " " + parameter.getName() + ",");
    }
    //获取并输出方法抛出的异常
    Class[] exceptionTypes = method.getExceptionTypes();
    if (exceptionTypes.length == 0){
        System.out.println(" )");
    } else {
        for (Class c : exceptionTypes) {
            System.out.println(" ) throws " + c.getName());
        }
    }
}
```
### 获取构造器
通过`getConstructor`方法可以获取Class对象的构造器，具体的例子在上文创建实例中已经介绍过，在此不再赘述
### 调用方法
当我们从一个类获取到方法之后，可以用`invoke()`方法来调用这个方法。invoke方法的签名如下：
```
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException,
InvocationTargetException
```
invoke具体的例子：
```
public class Client {
    public static void main(String[] args) throws Exception {
        Class<?> c = Calculation.class;
        Object obj = c.newInstance();
        Method method = c.getMethod("add", int.class, int.class);
        int res = (int) method.invoke(obj, 1, 2);
        System.out.println(res);
    }
}
class Calculation {
    public int add(int x, int y) {
        return x + y;
    }
}
```
### 获取注解
首先介绍一下注解，注解是Java5中的新特性，它是插入你代码中的一种注释或者说是一种元数据（meta data）。这些注解信息可以在编译期使用预编译工具进行处理，也可以在运行期使用Java反射机制进行处理。注解可以分为以下四种：
- 类注解
- 方法注解
- 变量注解
- 参数注解
由于篇幅限制，在此只介绍获取类的注解，其它注解的获取可以参考：[Java Reflection(八):注解](http://ifeve.com/java-reflection-8-annotation/)
```
public class Client {
    public static void main(String[] args) throws Exception {
        Class<?> c = Calculation.class;
        // 获取所有注解
        // Annotation[] annotations = c.getAnnotations();
        // 指定获取MyAnnotation注解
        Annotation annotation = c.getAnnotation(MyAnnotation.class);
        if (annotation instanceof MyAnnotation) {
            MyAnnotation myAnnotation = (MyAnnotation) annotation;
            System.out.println(myAnnotation.id());
            System.out.println(myAnnotation.desc());
        }
    }
}

@MyAnnotation(id ="1", desc = "calc")
class Calculation {
    public int add(int x, int y) {
        return x + y;
    }
}

// 自定义注解
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface MyAnnotation {
    public String id();
    public String desc();
}
```

### Java反射用途
上文都是介绍java反射的基本用法，在业务开发确实很少使用反射，但是优秀的框架必然会使用到反射来实现很多复杂的功能，比如Spring的IOC（控制反转），通过解析bean的xml文件，Spring在运行期根据xml配置，通过Java反射生成相应的bean，放入到Spring容器中。
### Java反射的功能
- 获取一个对象的类信息.
- 获取一个类的访问修饰符、成员、方法、构造方法以及超类的信息.
- 检获属于一个接口的常量和方法声明.
- 创建一个直到程序运行期间才知道名字的类的实例.
- 获取并设置一个对象的成员，甚至这个成员的名字是在程序运行期间才知道.
- 检测一个在运行期间才知道名字的对象的方法
### 参考资料
- [java.lang.reflect](https://docs.oracle.com/javase/8/docs/api/java/lang/reflect/package-summary.html)
- [Java 反射由浅入深 | 进阶必备](https://juejin.im/post/598ea9116fb9a03c335a99a4)
- [深入解析Java反射（1） - 基础](https://www.sczyh30.com/posts/Java/java-reflection-1/)

