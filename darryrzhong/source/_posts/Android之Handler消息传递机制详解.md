---
title: Android之Handler消息传递机制详解
date: 2019-03-27 11:42:29
tags: Handler
categories: Android
---


# 前言
* 在Android开发中,多线程应用是非常频繁的,其中`Handler`机制随处可见.
* 下面就本人对Handle的一些理解与大家一起分享，共同回顾下`Handle异步消息传递机制`。

------------------------------

## 1.Handler是什么？
* Handler是一套在 Android开发中 进行异步消息传递的机制。
-------------------

## 2.Handler在Android中的作用
* 在Android开发中多线程的应用中，将工作线程中需更新UI的操作信息 传递到 UI主线程，从而实现 工作线程对UI的更新处理，最终实现异步消息的处理。

---------------------
## 3. 我们为什么要使用Handler去处理更新UI操作呢？
* 在多个线程并发更新UI的同时 保证线程安全。

-------------------------
## 4.Handler异步消息传递所涉及的相关概念
* MainThread （主线程）`UI线程，程序启动时自动创建。`
* 工作线程，`开发者自己开启的线程，执行耗时操作等。`
* Handler（处理者） `UI线程与子线程通信的媒介，添加消息到消息队列（Message Queue）处理循环器分发过来的消息（Looper）。`
* Message （消息） `Handler接受&处理的对象，存储需要操作的消息。`
* Message Queue（消息队列） `数据存储结构，采用先进先出方式，存储Handler发过来的消息。 `
*  Looper（循坏器）`消息队列与处理者的媒介，从消息队列中循环取出消息并发送给Handler处理。`

--------------------------

<!--more-->



## 5.使用方式
* Handler的使用方式 因发送消息到消息队列的方式不同而不同（两种）
*  `使用Handler.sendMessage（）、使用Handler.post（）`

### 1.使用 Handler.sendMessage（）方式
```
/** 
  * 方式1：新建Handler子类（内部类）
  */

    // 步骤1：自定义Handler子类（继承Handler类） & 复写handleMessage（）方法
    class mHandler extends Handler {

        // 通过复写handlerMessage() 从而确定更新UI的操作
        @Override
        public void handleMessage(Message msg) {
         ...// 执行UI操作
            
        }
    }

    // 步骤2：在主线程中创建Handler实例
        private Handler mhandler = new mHandler();

    // 步骤3：创建所需的消息对象
        Message msg = Message.obtain(); // 实例化消息对象
        msg.what = 1; // 消息标识
        msg.obj = "AA"; // 消息内容存放

    // 步骤4：在工作线程中 通过Handler发送消息到消息队列中
        mHandler.sendMessage(msg);


/** 
  * 方式2：匿名内部类
  */
   // 步骤1：在主线程中 通过匿名内部类 创建Handler类对象
            private Handler mhandler = new  Handler(){
                // 通过复写handlerMessage()
                @Override
                public void handleMessage(Message msg) {
                        ...// 需执行UI操作
                    }
            };

  // 步骤2：创建消息对象
    Message msg = Message.obtain(); // 实例化消息对象
  msg.what = 1; // 消息标识
  msg.obj = "AA"; // 消息内容存放
  // 步骤3：在工作线程中 通过Handler发送消息到消息队列中
   mHandler.sendMessage(msg);

```

### 2.使用Handler.post（）
```
 new Thread() {
            @Override
            public void run() {
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                // 通过psot（）发送，传入1个Runnable对象
                mHandler.post(new Runnable() {
                    @Override
                    public void run() {
                        // 指定操作UI内容
                   
                    }

                });
            }
        }.start();

```

------------------------

## 6.Handler底层原理及源码分析

在源码分析前，先来了解Handler机制中的几个核心类
* 处理器 （Handler）
* 消息队列 （MessageQueue）
* 循环器 （Looper）
关于这几个类的具体作用前面已经介绍过了就不再过多阐述了。

下面开始源码分析，注意力集中了
上文中我们提到过Handler发送消息有两种方式，分别是
* Handler.sendMessage（）
* 使用Handler.post（）
下面先从第一种开始分析：
### 方式1：使用 Handler.sendMessage（）
```
  //通过匿名内部类 创建Handler类对象
    private Handler mhandler = new  Handler(){
        // 通过复写handlerMessage()从而确定更新UI的操作
        @Override
        public void handleMessage(Message msg) {
                ...// 需执行的UI操作
            }
    };

---------->>开始源码分析

public Handler() {

            this(null, false);
            // ->>此处this指代的就是当前的Handler实例，调用有参构造

    }

public Handler(Callback callback, boolean async) {

            ...// 无关代码我就不贴了

            // 1. 指定Looper对象
                mLooper = Looper.myLooper();
                if (mLooper == null) {
                    throw new RuntimeException(
                        "Can't create handler inside thread that has not called Looper.prepare()");
                }
                // Looper.myLooper()作用：获取当前线程的Looper对象；若线程无Looper对象则抛出异常

                // 可通过执行Loop.getMainLooper()方法获得主线程的Looper对象

            // 2. 绑定消息队列对象（MessageQueue）
                mQueue = mLooper.mQueue;
                // 获取该Looper对象中保存的消息队列对象（MessageQueue）
                // 至此，完成了handler 与 Looper对象中MessageQueue的关联
    }

```
* 从上面的源码来看，当我们创建Handler对象后，通过Handler的构造方法系统就已经帮我们自动绑定了looper和对应的MessageQueue消息队列。我们只需拿着这个Handler对象执行我们所需的操作就可以了
* 但是，你肯定有疑问了，当前线程的Looper对象 & 对应的消息队列对象（MessageQueue） 是哪来的呢？我既没有获取也没有创建啊？

```
public static void main(String[] args) {
            ... // 无关的代码

            Looper.prepareMainLooper(); 
            // 1. 为主线程创建1个Looper对象，同时生成1个消息队列对象（MessageQueue）

            ActivityThread thread = new ActivityThread(); 
            // 2. 创建主线程

            Looper.loop(); 
            // 3. 自动开启 消息循环 

        }

```
* 我们可以看到，其实在Android应用进程启动时，会默认创建1个主线程（ActivityThread，也叫UI线程） ，创建ActivityThread的时候，会自动调用ActivityThread的1个静态的main（）方法 = 应用程序的入口，而main（）方法内则会自动调用Looper.prepareMainLooper()为主线程生成1个Looper对象和MessageQueue队列。

* 而Handler对象创建时若不指定looper则默认绑定主线程的looper，从而可以执行主线程的UI更新操作。

* 若是在子线程中创建Handler实例，则需要指定looper了，所以就用上了Loop.getMainLooper()方法来获得主线程的Looper对象。


### 方式1： 使用Handler.post（）

```
 public void dispatchMessage(Message msg) {

    // 1. 若msg.callback属性不为空，则代表使用了post（Runnable r）发送消息
    // 则执行handleCallback(msg)，即回调Runnable对象里复写的run（）
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }

            // 2. 若msg.callback属性为空，则代表使用了sendMessage（Message msg）发送消息，即回调复写的handleMessage(msg)
            handleMessage(msg);

        }
    }


   public void handleMessage(Message msg) {  
          ... // 创建Handler实例时复写
   } 
```
从以上源码来看，使用Handler.post（）时，系统会自动回调Runnable对象里复写的run（）方法，将其打包成msg对象， 实际上和sendMessage（Message msg）发送方式相同。


>至此，关于Handler的异步消息传递机制的解析就完成了。

------------------------------


## 7.关于Handler 内存泄露的原因

* 在Android开发中，内存泄露是 十分常见的
* 其中一种情况就是在Handler中发生的内存泄露

* 为什么会发生内存泄漏？
1.Handler的一般用法 ： 新建Handler子类（内部类） 、匿名Handler内部类，而在我们编写代码的时候，其实编译器就会提示我们这种操作可能会发生内存泄漏，在android studio中就是这块代码会变黄。
2.提示的原因是
* 该Handler类由于未设置为 静态类，从而导致了内存泄露
* 最终的内存泄露发生在Handler类的外部类：XXXActivity类中

3.内存泄漏的原因
首先我们先要了解一些其他的知识点。
* 主线程的Looper对象的生命周期 = 应该应用程序的生命周期
* 在Java中，非静态内部类 & 匿名内部类都默认持有 外部类的引用，

而在Handler处理消息的时候，Handler必须处理完所有消息才会与外部类解除引用关系，如果此时外部Activity需要提前被销毁了，而Handler因还未完成消息处理而继续持有外部Activity的引用。由于上述引用关系，垃圾回收器（GC）便无法回收MainActivity，从而造成内存泄漏。

-------------------------------

## 8.如何解决Handler内存泄漏
### 1.静态内部类+弱引用
将Handler的子类设置成 静态内部类，同时，还可加上 使用WeakReference弱引用持有Activity实例。
原因：弱引用的对象拥有短暂的生命周期。在垃圾回收器线程扫描时，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。
```

    // 设置为：静态内部类
    private static class FHandler extends Handler{

        // 定义 弱引用实例
        private WeakReference<Activity> reference;

        // 在构造方法中传入需持有的Activity实例
        public FHandler(Activity activity) {
            // 使用WeakReference弱引用持有Activity实例
            reference = new WeakReference<Activity>(activity); }

        // 复写handlerMessage() 
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case 1:
                 //更新UI
                    break;
                case 2:
                //更新UI
                    break;

            }
```

### 2.当外部l类结束生命周期时，清空Handler内消息队列

```
@Override
    protected void onDestroy() {
        super.onDestroy();
        mHandler.removeCallbacksAndMessages(null);
        // 外部类Activity生命周期结束时，同时清空消息队列 & 结束Handler生命周期
    }
```

> 推荐使用上述解决方法一，以保证保证Handler中消息队列中的所有消息都能被执行

---------------------------------------

## 总结
本文主要讲述了Handler的基本原理和使用方法，以及造成内存泄漏的原因和解决方案。

----------------------------


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)



