---
title: 关于 Context 的二三事
date: 2018-11-03 10:03:46
tags: [Android]
categories:
- Android
---

Context 是Android 开发处处可见的一个对象，这些对象是怎么来的，有什么关系，怎么使用，这里学习记录一下。

<!-- more -->

![](http://img.artaris.cn/15564663949291.jpg)

Context是一个抽象基类，我们通过它访问当前包的资源（getResources、getAssets）和启动其他组件（Activity、Service、Broadcast）以及得到各种服务（getSystemService）。

对Context的理解可以来说：Context提供了一个应用的运行环境，在Context的大环境里，应用才可以访问资源，才能完成和其他组件、服务的交互，Context定义了一套基本的功能接口，我们可以理解为一套规范，而Activity和Service是实现这套规范的子类，这么说也许并不准确，因为这套规范实际是被ContextImpl类统一实现的，Activity和Service只是继承并有选择性地重写了某些规范的实现。

**Activity在创建的时候会new一个ContextImpl对象并在attach方法中关联它**


很明确，不同的Context得到的都是同一份资源。这是很好理解的，请看下面的分析，
在设备参数和显示参数不变的情况下，不同的ContextImpl访问到的是同一份资源。设备参数不变是指手机的屏幕和android版本不变，显示参数不变是指手机的分辨率和横竖屏状态。也就是说，尽管Application、Activity、Service都有自己的ContextImpl，并且每个ContextImpl都有自己的mResources成员，但是由于它们的mResources成员都来自于唯一的ResourcesManager实例，所以它们看似不同的mResources其实都指向的是同一块内存(C语言的概念)，因此，它们的mResources都是同一个对象（在设备参数和显示参数不变的情况下）。


![](http://img.artaris.cn/15564659341006.jpg)



**getApplication()和getApplicationContext()的区别？**
getApplication返回结果为Application，且不同的Activity和Service返回的Application均为同一个全局对象，在ActivityThread内部有一个列表专门用于维护所有应用的application
getApplicationContext返回的也是Application对象，只不过返回类型为Context，看看它的实现


```
@Override  
public Context getApplicationContext() {  
    return (mPackageInfo != null) ?  
            mPackageInfo.getApplication():mMainThread.getApplication();  
}  
```

上面代码中mPackageInfo是包含当前应用的包信息、比如包名、应用的安装目录等，原则上来说，作为第三方应用，包信息mPackageInfo不可能为空，在这种情况下，getApplicationContext返回的对象和getApplication是同一个。但是对于系统应用，包信息有可能为空，具体就不深入研究了。从这种角度来说，对于第三方应用，一个应用只存在一个Application对象，且通过getApplication和getApplicationContext得到的是同一个对象，两者的区别仅仅是返回类型不同。

**Context 导致的内存泄漏问题？**
ApplicationContext 会伴随应用的整个生命周期，不会泄漏。Context 泄漏基本都指ActivityContext 。
当把Context传入第三方类时，这个类就持有了这个 Activity 这个对象，当Activity销毁的时候，可能这个对象还在使用中（如工具类、Presenter 等）造成Activity无法正常回收，即内存泄漏。

**正确使用 Context 的姿势？**
1. 与视图有关的上下文，尽量使用Activity Context，不去使用ApplicationContext 去创建新的页面（setFlag(FLAG_NEW_TASK|FLAG_CLEAN_TOP)）会开辟一个新的页面栈，不方便管理。
2. 在初始化工具类的时候，使用getApplicaitonContext()去初始化，不会造成内存泄漏。
3. Application Context 在 attachBaseContext()之后才会初始化。执行顺序为`Constructor()`->`attachBaseContext()`->`onCreate()`
