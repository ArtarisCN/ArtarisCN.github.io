---
title: 自定义 Handler 的内存泄漏问题
date: 2017-06-23 15:51:54
tags: [Android]
categories:
- Android
---

Handler 内存泄漏一直是内存泄漏的一大常见原因。今天主要分析怎么解决这个问题。

<!-- more -->


- 设为静态内部类
静态内部类属于 GC Root 对象，不会持有外部 Activity 的引用
- 在 onDestroy 中让 Handler.removeMessage，removeRunnable handle = null
在 onDestroy 取消 Handle 执行的所有任务，释放对外的引用也是不错的办法，不过要注意的是 onDestroy 方法不一定会被调用，这种方法并不能万无一失
- 可以设 Handler 对 Activity 的弱引用
方法一虽然可以保证内部静态类不持有外部引用了，但是当自线程需要发消息更新主界面的时候还是需要Activity的对象引用，这时候使用弱引用可以避免内存的泄漏，当系统需要回收 Activity 的时候并不会因为 Handler 的持有而失败，使用代码如下：
    ```
    private static class StaticHandle extends Handler{
        WeakReference<Activity> mActivityReference;

        public StaticHandle(Activity activity) {
            mActivityReference= new WeakReference<Activity>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            Activity activity = mActivityReference.get();
            if(activity != null{
                mTextView.setText((String)msg.obj);
            }
        }
    }
    ```
