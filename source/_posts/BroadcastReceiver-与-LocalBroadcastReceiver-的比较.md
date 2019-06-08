---
title: BroadcastReceiver 与 LocalBroadcastReceiver 的比较
date: 2018-12-19 16:40:11
tags: [Android]
categories:
- Android
---

LocalBroadcastReceiver 和 BroadcastReceiver 均为常用的广播方法。这里比较一下他们之间的区别与使用。

<!-- more -->

## LocalBroadcastReceiver
LocalBroadcastReceiver 是本地广播，只能在应用使用和接收。
1. 获取一个localBroadcastManager实例
2. 使用localBroadcastManager.sendBroadcast(intent)方法发送广播
3. 写好广播接收器
4. 注册好广播接收器的要接收的广播地址，然后使用localBroadcastManager.registerReceiver(mBroadcastReceiver,intentFilter);方法进行注册
5. 记得在onDestroy()中取消注册

## BroadcastReceiver
BroadcastReceiver 是针对应用间、应用与系统间、应用内部进行通信的一种方式
LocalBroadcastReceiver 仅在自己的应用内发送接收广播，也就是只有自己的应用能收到，数据更加安全广播只在这个程序里，而且效率更高。
**BroadcastReceiver 使用**
1. 制作intent（可以携带参数）
2. 使用sendBroadcast()传入intent;
3. 制作广播接收器类继承BroadcastReceiver重写onReceive方法（或者可以匿名内部类啥的）
4. 在java中（动态注册）或者直接在Manifest中注册广播接收器（静态注册）使用registerReceiver()传入接收器和intentFilter
5. 取消注册可以在OnDestroy()函数中，unregisterReceiver()传入接收器
LocalBroadcastReceiver 使用
1. LocalBroadcastReceiver不能静态注册，只能采用动态注册的方式。
在发送和注册的时候采用，LocalBroadcastManager的sendBroadcast方法和registerReceiver方法

1. 可以明确的知道正在发送的广播不会离开我们的程序，因此不用担心机密数据泄露问题
2. 其他程序无法将广播发送到我们程序的内部，因此不需要担心会有安全漏洞的隐患
3. 发送本地广播要比全局广播更加高效

* BroadcastReceiver 是跨应用广播，利用Binder机制实现，支持动态和静态两种方式注册方式。
* LocalBroadcastReceiver 是应用内广播，利用Handler实现，利用了IntentFilter的match功能，提供消息的发布与接收功能，实现应用内通信，效率和安全性比较高，仅支持动态注册。


**广播细分为三种:**
- 普通广播
- 有序广播
- 本地广播

**普通广播是什么？**
调用sendBroadcast()发送

**有序广播是什么？**
调用sendOrderedBroadcast()发送
广播接收者会按照priority优先级从大到小进行排序
优先级相同的广播，动态注册的广播优先处理
广播接收者还能对广播进行截断和修改

**本地广播的优点?**
效率更高。
发送的广播不会离开我们的应用，不会泄露关键数据。
其他程序无法将广播发送到我们程序内部，不会有安全漏洞。

现在BroadcastReceiver也不推荐使用静态注册了，8.0之后限制了绝大部分广播只能使用动态注册

> 应用内广播现在还有人用吗？已经被EventBus类似的事件替换了吧？
> emmmm……
