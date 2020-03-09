---
title: Android Fresco图片加载库基础使用详解
date: 2019-03-31 00:13:33
tags: Fresco
categories: Android
---

# 前言
图片加载在 Android开发项目中是必不可少的，为了降低开发周期和难度，我们经常会选用一些图片加载的开源库，而Android发展到现在图片加载开源库也越来越多了，下面介绍 Fresco开源图片加载库.

## 简介
`Fresco`是由Facebook开源的一个图片加载库,Fresco是一个功能强大的系统，用于在Android应用程序中显示图像.

## 功能介绍以及基础使用
### 1.配置
* 在 build.gradle 中配置:
```
dependencies {
  // 其他依赖
  compile 'com.facebook.fresco:fresco:0.12.0'
}
```

* 下面的依赖需要根据需求添加：
```
dependencies {
  // 在 API < 14 上的机器支持 WebP 时，需要添加
  compile 'com.facebook.fresco:animated-base-support:0.12.0'

  // 支持 GIF 动图，需要添加
  compile 'com.facebook.fresco:animated-gif:0.12.0'

  // 支持 WebP （静态图+动图），需要添加
  compile 'com.facebook.fresco:animated-webp:0.12.0'
  compile 'com.facebook.fresco:webpsupport:0.12.0'

  // 仅支持 WebP 静态图，需要添加
  compile 'com.facebook.fresco:webpsupport:0.12.0'
}
```


<!--more-->


* Application中初始化Fresco
```
[MyApplication.java]
public class MyApplication extends Application {
	@Override
	public void onCreate() {
		super.onCreate();
		Fresco.initialize(this);
	}
}
```

* 在 AndroidManifest.xml 中指定你的 Application 类
```
  <manifest
    ...
    >
    <uses-permission android:name="android.permission.INTERNET" />
    <application
      ...
      android:label="@string/app_name"
      android:name=".MyApplication"
      >
      ...
    </application>
    ...
  </manifest>
```

* 添加网络权限
```
<uses-permission android:name="android.permission.INTERNET"/>
```

* 在xml布局文件中, 加入SimpleDraweeView：
```
<com.facebook.drawee.view.SimpleDraweeView
    android:id="@+id/my_image_view"
    android:layout_width="130dp"
    android:layout_height="130dp"
    fresco:placeholderImage="@drawable/my_drawable"
  />
```

* 开始加载图片
```
Uri uri = Uri.parse("https://raw.githubusercontent.com/facebook/fresco/gh-pages/static/logo.png");
SimpleDraweeView draweeView = (SimpleDraweeView) findViewById(R.id.my_image_view);
draweeView.setImageURI(uri);
```

剩下的，Fresco会替你完成:

显示占位图直到加载完成；
下载图片；
缓存图片；
图片不再显示时，从内存中移除；
等等等等。

---------------------------------

###  2.基本功能介绍&使用

* 可配置的所有选项
```
<com.facebook.drawee.view.SimpleDraweeView
  android:id="@+id/my_image_view"
  android:layout_width="20dp"
  android:layout_height="20dp"
  fresco:fadeDuration="300"  //淡入淡出动画持续时间
  fresco:actualImageScaleType="focusCrop"  //实际图片缩放类型
  fresco:placeholderImage="@color/wait_color"  //占位符
  fresco:placeholderImageScaleType="fitCenter"  //占位符图片缩放类型
  fresco:failureImage="@drawable/error"  //加载失败显示图片
  fresco:failureImageScaleType="centerInside"  //缩放类型
  fresco:retryImage="@drawable/retrying"  //重新加载图片
  fresco:retryImageScaleType="centerCrop"  
  fresco:progressBarImage="@drawable/progress_bar"   //正在加载图片
  fresco:progressBarImageScaleType="centerInside"   //缩放类型
  fresco:progressBarAutoRotateInterval="1000"    //正在加载图片自动旋转的时间间隔,直到图片加载成功停止旋转
  fresco:backgroundImage="@color/blue"    //背景图片
  fresco:overlayImage="@drawable/watermark"  // 叠加图
  fresco:pressedStateOverlayImage="@color/red"
  fresco:roundAsCircle="false" //圆形图
  fresco:roundedCornerRadius="1dp"   //圆角图&半径
  fresco:roundTopLeft="true"  //左上角
  fresco:roundTopRight="false"//右上角圆
  fresco:roundBottomLeft="false"//左下角
  fresco:roundBottomRight="true"//右下角
  fresco:roundWithOverlayColor="@color/corner_color"  // 圆形&圆角边框颜色
  fresco:roundingBorderWidth="2dp"  // 圆形&圆角边框宽度
  fresco:roundingBorderColor="@color/border_color"     //圆形或圆角图像底下的叠加颜色
/>
```

> 必须声明 android:layout_width 和 android:layout_height。如果没有在XML中声明这两个属性，将无法正确加载图像。

> Drawees 不支持 wrap_content 属性。

所下载的图像可能和占位图尺寸不一致，如果设置出错图或者重试图的话，这些图的尺寸也可能和所下载的图尺寸不一致。

如果大小不一致，假设使用的是 wrap_content，图像下载完之后，View将会重新layout，改变大小和位置。这将会导致界面跳跃。

----------------------------


至此,Fresco的基本功能介绍&使用就讲解完毕了,感谢阅读

## 参考文章
* [官方文档](https://www.fresco-cn.org/)
* [Android图片加载神器之Fresco-加载图片基础](https://blog.csdn.net/y1scp/article/details/49245535)


欢迎关注作者[darryrzhong](http://www.darryrzhong.xyz),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)



