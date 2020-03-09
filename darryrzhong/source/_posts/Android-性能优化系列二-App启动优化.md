---
title: 'Android 性能优化系列二 : App启动优化'
date: 2019-07-13 17:35:51
tags: 性能优化
categories: Android
---

#前言
本篇文章主要针对 Android性能优化 中App的启动优化

App启动,相信大家都是非常熟悉了,那为何我们需要对App启动做优化呢,这里就要先对我们Android 从开机到启动我们的App进入主页面这一流程做一个简单的阐述了.


## 一、Android启动流程


我们先来看一张流程图

![Android启动流程.png](https://upload-images.jianshu.io/upload_images/5549640-33fc4dabf7839c47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



首先呢,我们Android手机开机时是先加载一个Boot程序,有点类似Windows开机时的开机引导程序,然后通过Boot程序加载Lux内核,随后是调用Native的init()方法做一些初始化加载操作(加载一些系统需要的驱动程序),然后就进入我们的java Framework层,也就是创建我们的java虚拟机,然后通过java虚拟机创建我们的系统程序,最后才是调用我们App的application启动我们的App.

流程如下:
> Loader > Kernel > Native > Framework > Application

Android启动流程大致就是这样,我们不需要去深入,只需要大概知道是这么个流程就行了.

<!--more-->


所以说,其实我们手机的操作系统就是一个App,开机启动时先加载各种驱动程序(类似App初始化各种第三方SDK),然后加载系统标识（黑白屏问题），然后启动开机欢迎动画(App欢迎页动画),最后进入到桌面(App主页面).

## 二、App启动时黑白屏问题

基于以上的启动流程 ，那么App启动优化的第一步就是从系统标识入手，我们手机开机时一般最先出现的是手机厂商的logo标识，而App启动时会先调用一个预显示窗口，这个窗口的样式一般是黑色或者白色，所以也就出现了App启动时出现短暂的黑白屏问题，


流程如下：

>  Application > onCreate > MainActivity > onCreate > windows > setContentView> layout

在我们点击桌面App启动图标时，系统首先会给我们App分配一个进程，然后在调用我们的application入口，最后调用我们的mainActivity的setContentView方法加载布局文件，最后我们就能看到我们的主界面了。

然后在application 到MainActivity 之间，还会有一个预显示窗口，就是出现的黑白屏。

那我们怎么优化去除这个惹人厌的黑白屏呢？

我们先看一下这个黑白屏从哪蹦出来的。

![AppTheme.png](https://upload-images.jianshu.io/upload_images/5549640-2c04c3ec92759272.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/5549640-5feb833ca674709e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![image.png](https://upload-images.jianshu.io/upload_images/5549640-8735fd4be8a1fd79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看到了吧，最初我们可没有设置这么个东西，这是系统默认给我们App设置的，而且是白色的。

知道源头了，那就开始处理吧

### 1.可以将背景设置为透明

```
  <item name="android:windowBackground">@null</item>

```
So easy ,就是这么简单,你在运行下App,果然黑白屏没有了,But 是不是有哪不对劲了.
对的,被你发现了,虽然黑白屏没了,但是我们的App似乎是变迟钝了

你在仔细观察一下,点击App启动图标后,App似乎是顿了一下,然后加载了我们的欢迎页面,有点像ANR,只不过很短暂 ,但是用户还是能够发现的,所以用户体验只是比起黑白屏好了那么一点点而已.

那就,继续优化呗

### 2.给背景设置一张图片或者xml布局文件

```
 <item name="android:windowBackground">@layout/activity_main</item>
    
   或者

  <item name="android:windowBackground">@drawable/splash_bg</item>

```

So easy,又是这么简单.对的,就是这么简单,这也是目前最认同的方案,稍微有点规模的公司都是采用这种方案来优化的.

But,这里需要注意的是,放一张图片的话,需要注意图片的大小,如果有虚拟导航键的话可能会出线底部闪烁问题,解决办法就是压缩图片大小,将图片转化成.9.png格式,让其自适应拉伸.

## 三、onCreate()优化

一般我们都会重写自己的Application,然后在onCreate()方法内做一些初始化操作,
一般都是一些第三方SDK配置.

```
public class App extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        
        //......初始化第三方SDK
        
        
    }
}

```

在这里我们可以将一些较大的第三方库放在异步线程中进行初始化

```

 @Override
    public void onCreate() {
        super.onCreate();

        //懒加载
        new Thread(){
            @Override
            public void run() {
                super.run();
                //初始化数据库
                //初始化数据库 网络等操作
                //......初始化第三方SDK
            }
        }.start();
    }
```

需要注意的是,如果涉及到UI操作的话,就不要放在异步线程中去执行,否则可能出Null
那我们怎么判断哪些初始化能够放在异步线程中呢,这个就需要你自己去分辨了,实在不知道就直接丢异步里,报错了就再丢出来就行了.

不止是Application中,我们的activity也可以用这种方式来进行优化.

以上这种优化也称为真优化,对代码层的优化我们叫做真优化,而不对代码层直接操作的我们称为伪优化,一般我们的优化方案都是两种混合使用.

## 四、伪优化

在我们做完上述的优化后,成功进入到主界面后,还没完.

你可能会发现进入主界面也会出现部分显示加载问题,具体就需要看你布局层级的复杂度和界面业务的需求了.如果是复杂页面,可以先看看布局文件层级是否还可以进行优化,然后在看是否时请求网络数据太大,例如加载了大图等. 这时就可以进去一定的伪优化了.

例如和产品协商在进入页面时加载一个dialog进行缓冲一下,很多App也是进去这样的优化,

看看我们的简书App就是这样,我基本每次进去都会弹一个dialog

![image.png](https://upload-images.jianshu.io/upload_images/5549640-8215ca9d6d469d44.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/240)


> 至此,App启动优化方案就介绍完了.当然还有更多优化方案,具体的就要根据业务需求而定了


欢迎关注作者[darryrzhong](http://www.darryrzhong.xyz),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)




