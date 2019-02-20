---
title: Java 中单例的实现与线程安全
date: 2019-02-20 11:37:24
tags: [Java]
categories:
- Java
---

# Java 中单例的实现与线程安全
单例模式事一种常用的设计模式，其目的是为了该类只拥有一个实例，避免具有相同功能的多个对象消耗过多的资源，创建一个单例需要注意以下问题：
1. 单例功能：基本功能
2. 延迟加载：避免不必要的资源消耗
3. 线程安全：多个线程同时创建对象时为同一个对象
4. 没有性能问题：创建函数不能影响效率
5. 防止序列化产生新对象：安全
6. 防止反射攻击：安全

<!-- more -->

## 饿汉式（类加载时就完成初始化）
```
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
优点：简单粗暴，加载类的时候就初始化完成了，线程安全。
缺点：消耗资源，如果使用时机较晚或不使用，浪费内存。

## 懒汉式(线程不安全)

```
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
优点：延迟初始化，避免了不必要的内存开销；
缺点：线程不安全。
## 懒汉式（线程安全，同步方法）
使用`synchronized`修饰获取方法。
```
public class Singleton {

    private static Singleton instance;

    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
优点：延迟初始化，避免了不必要的内存开销；线程安全；
缺点：每次获取实例都需要同步，其实只需要第一次同步就可以了。
## 懒汉式（线程安全，同步代码块 DCL:double check lock）
使用`synchronized`同步 new 方法，内外加双重判断，volatile 修饰实例。
synchronized 能保证每次只有一个线程访问代码块，volatile 保证了 instance 创建的原子性。
```
public class Singleton {

    private static volatile Singleton instance;

    private Singleton() {
    }

    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

    /**
     * 如果实现了Serializable, 必须重写这个方法
     */
    private Object readResolve() throws ObjectStreamException {
        return instance;
    }
}
```
## 静态内部类

```
public class Singleton {

    private static Singleton instance;
    /**
     * 私有化构造方法
     */
    private Singleton() {
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    /**
     * 类级的内部类，也就是静态的成员式内部类，该内部类的实例与外部类的实例没有绑定关系，
     * 而且只有被调用到才会装载，从而实现了延迟加载
     */
    private static class SingletonHolder {
        /**
         * 静态初始化器，由JVM来保证线程安全
         */
        private static final Singleton INSTANCE = new Singleton();
    }

}
```

## 枚举

```
public enum Singleton {

    INSTANCE;

}
```
优点：简单粗暴，线程安全，高效，自动避免序列化/反序列化攻击、反射攻击(枚举类不能通过反射生成)
缺点：可读性差


关联阅读
> ## synchronized 关键字
> synchronized 是 Java 中的关键字，是利用锁的机制来控制多线程同步的关键字.
> 锁的性质：
> 1.  互斥性/原子性：同一时间只允许一个线程持有某个对象锁，通过对锁的持有来保证同一时间只有一个线程对需同步的代码块(复合操作)进行访问；
> 2. 可见性：当一个线程释放锁之后，对共享变量所做的更改对之后获取该锁的另一个线程可见；
> 锁的分类:
> 1. 对象锁: 在 Java 中，每个对象都会有一个 monitor 对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。
> 2. 类锁:在 Java 中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。
>
> |  | 修饰代码块 | 修饰代码块 |
> | :-: | :-: | :-: |
> | 获取对象锁 | synchronized(this\|object) {} | 修饰非静态方法 |
> | 获取类锁 | synchronized(类.class) {} | 修饰静态方法 |
>
> **对于同一种锁的访问是同步的**
