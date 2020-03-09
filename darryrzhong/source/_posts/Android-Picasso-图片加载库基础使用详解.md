---
title: Android Picasso 图片加载库基础使用详解
date: 2019-03-31 00:11:08
tags: Picasso
categories: Android
---

# 前言
图片加载在 Android开发项目中是必不可少的，为了降低开发周期和难度，我们经常会选用一些图片加载的开源库，而Android发展到现在图片加载开源库也越来越多了，下面介绍 Picasso 开源图片加载库.

## 简介

`Picasso`中文翻译为'毕加索',由Square公司开源的一个适用于Android的强大图像下载和缓存库.

## 功能介绍以及基础使用
### 1.配置
* 在gradle添加依赖

```
implementation 'com.squareup.picasso:picasso:2.71828'
```

* 添加网络权限
```
<uses-permission android:name="android.permission.INTERNET"/>
```

<!--more--> 



* 基本使用
```
ImageView mImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http:/*********";

        Picasso .with(this)
                .load(Url)
                .into(targetImageView);
```

### 2.功能介绍以及基本使用

* 异步加载显示图片
```
ImageView targetImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http://**********";

//Picasso使用了流式接口的调用方式
        Picasso .with(context)
                .load(Url)
                .into(targetImageView);
```

* 图片转换
转换图片以适合所显示的ImageView，来减少内存消耗
```
Picasso.with(context)
  .load(url)
//裁剪图片尺寸
  .resize(50, 50)
//设置图片圆角
  .centerCrop()
  .into(imageView)
```

* 加载过程中和加载错误时显示对应图片
```
Picasso.with(context)
    .load(url)
//加载过程中的图片显示
    .placeholder(R.drawable.user_placeholder)
//加载失败中的图片显示
//如果重试3次还是无法成功加载图片，则用错误占位符图片显示。
    .error(R.drawable.user_placeholder_error)
    .into(imageView);
```
*  在Adapter中的回收不在视野的ImageView和取消已经回收的ImageView下载进程
 ```
@Override 
public void getView(int position, View convertView, ViewGroup parent) {
  SquaredImageView view = (SquaredImageView) convertView;
  if (view == null) {
    view = new SquaredImageView(context);
  }
  String url = getItem(position);
 
  Picasso.with(context).load(url).into(view);
}
```

* 加载多种不同数据源 网络、本地、资源、Assets 等
```
//加载资源文件
Picasso.with(context).load(R.drawable.landing_screen).into(imageView1);
//加载本地文件
Picasso.with(context).load(new File("/images/lunch_bees.gif")).into(imageView2);
```

* 默认配置自动添加磁盘和内存二级缓存功能

------------------------


至此,Picasso的基本功能和使用就介绍我完毕了,感谢阅读

欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)


