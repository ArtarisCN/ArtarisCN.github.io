---
title: int 与 Integer
date: 2017-03-19 19:49:55
tags: [Java]
categories:
- Java
---

* Integer是int的包装类，int则是java的一种基本数据类型
* Integer变量必须实例化后才能使用，而int变量不需要
* Integer实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值
* Integer的默认值是null，int的默认值是0

<!-- more -->


## 关于int和Integer的值的比较
* 两个通过new生成的Integer变量永远是不相等的。因为 new 出来的对象永远是两个对象，内存地址不同。

```
Integer i = new Integer(100);
Integer j = new Integer(100);
System.out.print(i == j); //false
```
* 非new生成的Integer变量和new Integer()生成的变量比较时，结果为false。因为非new生成的Integer变量指向的是java常量池中的对象，而new Integer()生成的变量指向堆中新建的对象，两者在内存中的地址不同。

```
Integer i = new Integer(100);
Integer j = 100;
System.out.print(i == j); //false
```
* 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间-128到127之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false。因为在Integer 自动装箱的时候会调用`Integer valueOf(int i)`函数，判断值是否在缓冲区内。自动装箱请看后文。


```
Integer i = 100;
Integer j = 100;
System.out.print(i == j); //true

Integer i = 128;
Integer j = 128;
System.out.print(i == j); //false
```

## int 和 Integer 应用中具体会产生哪些差异

int 是直接存储在堆内存中的，大量使用对象类型会消耗大量的系统资源。int 在性能极度敏感的场景往往具有比较大的优势。

## Integer 的源码分析

### 缓冲区的调整
JVM 提供了参数设置：

```
-XX:AutoBoxCacheMax=N
```

### Integer 的数值保存

Integer 保存相应的数值时的变量是一个 final 变量，这是为了保证在使用的时候内存地址不变的情况下值也不会发生变化。


## 关于 int 和 Integer 的问题区别分析

### 自动装箱/自动拆箱

- 自动装箱时编译器调用 valueOf() 将原始类型值转换成对象，同时自动拆箱时，编译器通过调用类似intValue(),doubleValue()这类的方法将对象转换成原始类型值。
- 自动装箱是将boolean值转换成Boolean对象，byte值转换成Byte对象，char转换成Character对象，float值转换成Float对象，int转换成Integer，long转换成Long，short转换成Short，自动拆箱则是相反的操作。

自动装箱/拆箱是发生在编译阶段，当接受的一个原始类型值，那么Java 会自动将这个原始类型值转换为与之对应的对象，反之拆箱亦然。

## 线程安全
原始数据类型并不能保证县城安全，要保证多线程安全必须使用类似 AtomicInteger、AtomicLong 这样的线程安全类。

### AtomicInteger 的实现

* AtomicInteger类中有有一个变量valueOffset，用来描述AtomicInteger类中value的内存位置 。
* 当需要变量的值改变的时候，先通过get（）得到valueOffset位置的值，也即当前value的值.给该值进行增加，并赋给next
* compareAndSet（）比较之前取到的value的值当前有没有改变，若没有改变的话，就将next的值赋给value，倘若和之前的值相比的话发生变化的话，则重新一次循环，直到存取成功，通过这样的方式能够保证该变量是线程安全的
* value使用了volatile关键字，使得多个线程可以共享变量，使用volatile将使得VM优化失去作用，在线程数特别大时，效率会较低。


## 泛型的使用

Java 的原始数据类型并不能与泛型搭配使用，因为Java 的泛型是一种在编译阶段会自动将类型转换为对应的特定类型，这就决定了使用泛型，必须保证相应类型可以转换为 Object。而原始数据类型并不是对象类型。
