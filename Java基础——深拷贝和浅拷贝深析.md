---
title: "Java基础——深拷贝和浅拷贝深析"
date: 2018-10-13 17:23:21
tags: 深拷贝、浅拷贝
categories: 夯实基础
summary:  "最近在公司的项目中用到了Java的对象拷贝(Object Copy)"
---
最近在公司的项目中用到了Java的对象拷贝(Object Copy)，准备深入了解一下，试想一下如果我们不对对象拷贝，而是直接将实例赋值给新的实例会如何？
<!-- more -->
```java
Wheel wheel = new Wheel(20, "朝阳");
Car car = new Car(1, "宝马", wheel);
Car newCar = car;
System.out.println(car);
System.out.println(newCar);

// 输出
objectcopy.Car@7637f22
objectcopy.Car@7637f22
```
正如以上代码所示，car和newCar都是对堆区内同一个对象的引用，如下图所示，如果我们相对newCar的属性修改，car也会做相应的改动，如果想让newCar生成一个新对象，就要用到Java的对象拷贝。
![](http://tc.ganzhiqiang.wang/1537022939.png?imageMogr2/thumbnail/!55p)

Java的对象拷贝(Object Copy)，即把一个对象的属性拷贝到具有相同类型的对象，目的就是为了复用已有的对象的数据，对象拷贝分为深拷贝、浅拷贝和延迟拷贝，其中浅拷贝和深拷贝最为常用，首先了解一下浅拷贝和深拷贝的定义：
**浅拷贝**：使用一个已知实例对新创建实例的成员变量逐个赋值。
**深拷贝**:  当一个类的拷贝构造方法，不仅要复制对象的所有非引用成员变量值，还要为引用类型的成员变量创建新的实例，并且初始化为形式参数实例值。
简单的说了浅拷贝只是复制一个对象，传递引用，不能复制实例， 而深拷贝对对象内部的引用均复制。
### 浅拷贝
浅拷贝是按位拷贝对象（是直接将一个对象的成员值拷贝过来）。如果是基本类型，拷贝的就是基本类型的值；如果属性是引用类型，则会拷贝属性的内存地址。下面的代码是调用Car的clone实现的浅拷贝。
```java
Wheel wheel = new Wheel(20, "朝阳");
Car car = new Car(1, "宝马", wheel);
Car newCar = (Car) car.clone();
System.out.println(car == newCar);
System.out.println(car.getId() == car.getId());
System.out.println(car.getBrand() == car.getBrand());
System.out.println(car.getWheel() == newCar.getWheel());

// 输出
false
true
true
true
```
![](http://tc.ganzhiqiang.wang/1537195457.png?imageMogr2/thumbnail/!60p)

### 浅拷贝的实现
浅拷贝的可以通过实现了Clonable接口并重写Object类的clone()方法，然后在方法内部调用super.clone()方法来实现，如下面代码所示
```java
//浅拷贝重写clone方法
@Override
protected Object clone() throws CloneNotSupportedException {
    return (Wheel)super.clone();
}
```
### 深拷贝
深拷贝会拷贝所有的属性，并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝，深拷贝相对浅拷贝速度较慢并且开销会更大。深拷贝的示例代码如下：
```java
Wheel wheel = new Wheel(20, "朝阳");
Car car = new Car(1, "宝马", wheel);
// 对Car对象进行深拷贝
Car newCar = (Car) BeanUtil.depthClone(car);

System.out.println(car == newCar);
System.out.println(car.getId() == car.getId());
System.out.println(car.getBrand() == car.getBrand());
System.out.println(car.getWheel() == newCar.getWheel());

// 输出
false
true
true
false
```
从上面结果看与浅拷贝不一样的是wheel没有引用car的wheel对象，而是重新生成了wheel对象，而cat的“id”字段是由基本类型，所以它只是将“id”字段拷贝到newCar的“id”的字段，由此可以看出，深拷贝是完全拷贝原来对象的属性。
![](http://tc.ganzhiqiang.wang/33.jpg?imageMogr2/thumbnail/!70p)
### 深拷贝的实现
深拷贝的最常见的实现就是将原对象的每个属性都赋值到新对象的对应的属性中，如下代码重写clone方法
```java
@Override
protected Object clone() throws CloneNotSupportedException {
    Wheel nWheel = (Wheel) wheel.clone();
    return new Car(id, brand, nWheel);
}

@Override
protected Object clone() throws CloneNotSupportedException {
    return new Wheel(diameter, brand);
}
```
字段较少时，可通过通过逐个字段赋值的方式进行深拷贝，但是如果字段过多，这种方式不免有些臃肿。另一种实现深拷贝的方式就是序列化的方式来实现。如下代码所示，先把srcObj序列化成byte,再用stream去读取byte，再反序列化成cloneObj对象。需要注意的是被序列化的类都要实现Serializable接口
```java
public static Object depthClone(Object srcObj) {
        Object cloneObj = null;
        try {
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            ObjectOutputStream oo = new ObjectOutputStream(out);
            oo.writeObject(srcObj);
            ByteArrayInputStream in = new ByteArrayInputStream(out.toByteArray());
            ObjectInputStream oi = new ObjectInputStream(in);
            cloneObj = oi.readObject();
            return cloneObj;
        } catch (Exception ex) {
            return null;
        }
}
```
详细代码，可以参考代码[GitHub](https://github.com/LJWLgl/CommonUtil/blob/master/src/main/java/io/github/ljwlgl/serialize/BeanUtil.java)
### 延迟拷贝
延迟拷贝是浅拷贝和深拷贝的一个组合，实际上很少会使用。 当最开始拷贝一个对象时，会使用速度较快的浅拷贝，还会使用一个计数器来记录有多少对象共享这个数据。当程序想要修改原始的对象时，它会决定数据是否被共享（通过检查计数器）并根据需要进行深拷贝。 
延迟拷贝从外面看起来就是深拷贝，但是只要有可能它就会利用浅拷贝的速度。当原始对象中的引用不经常改变的时候可以使用延迟拷贝。由于存在计数器，效率下降很高，但只是常量级的开销。而且, 在某些情况下, 循环引用会导致一些问题。
### 如何选择
如果对象都是基本类型，就可以使用浅拷贝，如果对象有引用类型，这就要看具体的需求了，如果对象是不变的，就可以用浅拷贝，反之就需要深拷贝。




