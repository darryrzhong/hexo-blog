---
title: 'Android RxJava系列一:  基础常用详解'
date: 2019-07-13 14:48:01
tags: Rxjava
categories: Android
---


# 前言
本篇主要介绍Rxjava在 Android 项目中的基础使用和常用方法,旨在给对 RxJava 感兴趣的人一些入门的指引.对Rxjava不熟悉的朋友可以去看我之前写的一篇简单介绍 [Android RxJava：基础介绍与使用](https://www.jianshu.com/p/9cb743b98b84),下面就来我们一起来看看在项目中如何使用 Rxjava 吧!

# Rxjava是什么?为什么要用Rxjava?
>  首先我们要知道Rxjava到底是什么东西?为什么这么多人用它以及它在Android项目中所占的比重.

* Rxjava到底是什么呢?
RxJava 在 GitHub 主页上的自我介绍是 : `a library for composing asynchronous and event-based programs using observable sequences for the Java VM`（一个在 Java VM 上使用可观测的序列来组成异步的、基于事件的程序的库）。这就是 RxJava ，概括得非常精准。然而对于初学者来说,两个字,不懂.其实说白了,Rxjava就是一个用来实现异步操作的第三方库,而至于其他的拓展功能也只是在实现异步过程中提供了一些辅助功能罢了.所以总结一下就一句话:

> Rxjava是一个用来实现异步的、基于事件的第三方库(就把它理解成Android Handler 的升级版就行了)

* 为什么要用Rxjava?
这就到我们今天的重头戏了.相信很多初学者都是在以下场景初识Rxjava的

`1.201x年你必须知道的几个Android开源库: .......、Rxjava`
`这是一个基于 Rxjava+Retrofit+mvp+........的demo`
`Android 工作必回 Rxjava+Retrofit+.......几件套`

<!--more-->

起初本人也是因为看到这些字眼才接触的Rxjava的,也是因为这些原因我才使用的Rxjava. But,当你真正的去了解了Rxjava,真正的把Rxjava用到了你的项目中,你才真正知道,为啥这么多人说要用Rxjava,以及你为什么要用Rxjava ,因为Rxjava真的太好用太便捷了,用了一次你就离不开它了.

举个例子你就能明白了:
假如现在有这么一个需求:你需要从数据库中取出一组图片资源id,然后通过遍历将它们显示在imageView上面,实现方式有很多种

* 使用传统方式实现
```
 //操作数据库属于耗时操作,需要开辟一个新线程放在后台操作
        new Thread() {
            @Override
            public void run() {
                super.run();
                final int[] drawableRes = {}; //.......从数据库中取出id资源数组操作
                //将id对应 drawable显示在界面上,需要在UI线程操作
                imageView.post(new Runnable() {
                    @Override
                    public void run() {
                        imageView.setImageResource(drawableRes[0]);
                    }
                });
            }
        }.start();
```

-----------------------------

```
Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            //.....操作UI
            imageView.setImageResource(drawableRes[0]);
        }
    };

 new Thread(){
            @Override
            public void run() {
                super.run();
                final int[] drawableRes = {}; //.......从数据库中取出id资源数组操作
                //将id对应 drawable显示在界面上,需要在UI线程操作
                handler.sendEmptyMessage(0);//发送消息,通知主线程刷新UI

            }
        }.start();
```

* 使用Rxjava方式实现

```
 Observable.create(new ObservableOnSubscribe<List>() {
            @Override
            public void subscribe(ObservableEmitter<List> emitter) throws Exception {
               drawableRes = new ArrayList<>(); //.......从数据库中取出id资源数组操作
               emitter.onNext(drawableRes);
               emitter.onComplete();
            }
        }).flatMap(new Function<List, ObservableSource<Integer>>() {
            @Override
            public ObservableSource<Integer> apply(List list) throws Exception {
                return Observable.fromIterable(list);
            }
        }).subscribeOn(Schedulers.io())//在IO线程执行数据库处理操作
          .observeOn(AndroidSchedulers.mainThread())//在UI线程显示图片
          .subscribe(new Observer<Integer>() {
              @Override
              public void onSubscribe(Disposable d) {
                  Log.d("----","onSubscribe");
              }

              @Override
              public void onNext(Integer integer) {
                  imageView.setImageResource(integer);//拿到id,加载图片
                  Log.d("----",integer+"");
              }

              @Override
              public void onError(Throwable e) {
                  Log.d("----",e.toString());
              }

              @Override
              public void onComplete() {
                  Log.d("----","onComplete");
              }
          });      
```

诶,等一下,你不是说使用Rxjava实现代码会更简洁快捷嘛,我怎么看着实现还变复杂了,明明只要切换一下线程,你这咋写了这么多,看不懂?????

咳咳,看不懂了吧,看不懂就对了,看着的确是变复杂了,But 代码这一连串链式调用下来不是显得代码逻辑很清晰吗.而且随着业务需求的增多,你可能需要拿到图片id时还要加一层过滤呢,只需要在加一个.xxx()方法即可,而且还可以随意切换操作线程,代码依然还是这么清晰简洁.而且使用Android studio 打开时还会自动缩进和显示提示信息:

![Rxjava.jpg](https://upload-images.jianshu.io/upload_images/5549640-ce72ae31b9f7a255.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上面的例子省去了部分代码,也是我随手写的,只要是让你体会一下Rxjava的书写方式和对比一下传统方式的实现有什么不同,看不懂没关系,下面我会一一解释,搬好小板凳
总结一下:

> 为什么要用Rxjava: 因为随着程序逻辑变得越来越复杂，它依然能够保持代码的简洁和阅读性.

# 怎么用Rxjava?

关于Rxjava的简单集成和基础使用请查看我之前的介绍[Android RxJava：基础介绍与使用](https://www.jianshu.com/p/9cb743b98b84)

首先大概说一下Rxjava的原理:

1.概念: 观察者模式
`RxJava 的异步实现，是通过一种扩展的观察者模式来实现的。`至于观察者模式的原理实现大家肯定都已经很熟悉了,我就不再阐述了,不熟悉的可以自行搜索.

RxJava 有四个基本概念：Observable (可观察者，即被观察者)、 Observer (观察者)、 subscribe (订阅)、事件。Observable 和 Observer 通过 subscribe() 方法实现订阅关系，从而 Observable 可以在需要的时候发出事件来通知 Observer。

RxJava 的事件回调方法除了普通事件 onNext() （相当于 onClick() / onEvent()）之外，还定义了两个特殊的事件：onCompleted() 和 onError()。
* onCompleted(): 事件队列完结。RxJava 不仅把每个事件单独处理，还会把它们看做一个队列。RxJava 规定，当不会再有新的 onNext() 发出时，需要触发
 * onCompleted() 方法作为标志。
* onError(): 事件队列异常。在事件处理过程中出异常时，onError() 会被触发，同时队列自动终止，不允许再有事件发出。
* 在一个正确运行的事件序列中, onCompleted() 和 onError() 有且只有一个，并且是事件序列中的最后一个。需要注意的是，onCompleted() 和 onError() 二者也是互斥的，即在队列中调用了其中一个，就不应该再调用另一个。


![Rxjava的观察者模式.jpg](https://upload-images.jianshu.io/upload_images/5549640-2d02072359cd3d10.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



2. 基本实现

* 创建 Observer 观察者

```
  Observer observer = new Observer<String>(){

            @Override
            public void onSubscribe(Disposable d) {
                Log.d("----","onSubscribe" );
            }

            @Override
            public void onNext(String s) {
                Log.d("----", s);
            }

            @Override
            public void onError(Throwable e) {
                Log.d("----", "onError");
            }

            @Override
            public void onComplete() {
                Log.d("----", "onComplete");
            }
        };
```

*  创建 Observable 被观察者

```
 Observable observable =Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                    emitter.onNext("rxjava");
                    emitter.onComplete();
            }
        });

```

* Subscribe (订阅)
```
observable.subscribe(observer);
```
以上就是传统的使用方式,你也可以采用链式调用：
```
  Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("rxjava");
                emitter.onComplete();
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d("----", "onSubscribe");
            }

            @Override
            public void onNext(String s) {
                Log.d("----", s);
            }

            @Override
            public void onError(Throwable e) {
                Log.d("----", "onError");
            }

            @Override
            public void onComplete() {
                Log.d("----", "onComplete");
            }
        });

```

在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念： Scheduler 。

先贴代码,以下便可实现异步操作:

```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("rxjava");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.newThread())
          .observeOn(AndroidSchedulers.mainThread())      
          .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d("----", "onSubscribe");
            }

            @Override
            public void onNext(String s) {
                Log.d("----", s);
            }

            @Override
            public void onError(Throwable e) {
                Log.d("----", "onError");
            }

            @Override
            public void onComplete() {
                Log.d("----", "onComplete");
            }
        });

```

在这里我们使用了Scheduler 进行了线程的切换,接下来介绍一下Scheduler :

3. Scheduler  (线程切换)

在不指定线程的情况下， RxJava 遵循的是线程不变的原则，即：在哪个线程调用 subscribe()，就在哪个线程生产事件；在哪个线程生产事件，就在哪个线程消费事件。如果需要切换线程，就需要用到 Scheduler （调度器）。

RxJava 已经内置了几个 Scheduler :

* Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。

* Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。

* Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。

*   AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行

有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。

* subscribeOn(): 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。

* observeOn(): 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

简单使用如下:
```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1");
                emitter.onNext("2");
                emitter.onNext("3");
                emitter.onComplete();
            }
        }).subscribeOn(Schedulers.io()) //在io执行上述操作
          .observeOn(AndroidSchedulers.mainThread())//在UI线程执行下面操作
          .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d("----","开始了");
            }

            @Override
            public void onNext(String s) {
                Log.d("----", s);
            }

            @Override
            public void onError(Throwable e) {

            }

            @Override
            public void onComplete() {
                Log.d("----", "complete");
            }
        });
```

> 以上就是Rxjava的基本常用方法了,看到这里你就已经可以愉快的使用Rxjava代替AsyncTask / Handler了,赶紧去试试吧!


-------------------------------------------

* 参考文章: [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083#toc_1),这是扔物线很早之前写的一篇关于Rxjava 1.x的详解,虽然现在已经Rxjava已经发展到2.x版本了.但其中一些基本实现原理的讲解确实非常到位,通俗易懂,很适合初学者学习,强烈建议去看下

* [RxJava2 只看这一篇文章就够了](https://juejin.im/post/5b17560e6fb9a01e2862246f)这是玉刚说的一篇关于Rxjava常用API的介绍,基本囊括了Rxjava所用到的所有API,还有代码举例,也是强烈建议观看收藏

关于Rxjava系列一就到此结束啦,后面有时间我还会写写一些其他的常用拓展操作符和与retrofit2的结合使用,欢迎关注订阅!


欢迎关注作者[darryrzhong](http://www.darryrzhong.xyz),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)


