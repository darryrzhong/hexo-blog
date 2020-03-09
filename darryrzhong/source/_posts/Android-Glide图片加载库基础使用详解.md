---
title: Android Glide图片加载库基础使用详解
date: 2019-03-31 00:12:23
tags: Glide
categories: Android
---

# 前言
图片加载在 Android开发项目中是必不可少的，为了降低开发周期和难度，我们经常会选用一些图片加载的开源库，而Android发展到现在图片加载开源库也越来越多了，下面介绍 Glide开源图片加载库.

## 简介
`Glide`是由Google开源的一个图片加载库,是一款快速高效的Android开源媒体管理和图像加载框架，它将媒体解码，内存和磁盘缓存以及资源池包装成简单易用的界面.

## 功能介绍以及基础使用
### 1.配置

* 在Project的gradle添加依赖
```
repositories {
  mavenCentral()
  google()
}
```

* 在Module的gradle添加依赖
```
dependencies {
  implementation 'com.github.bumptech.glide:glide:4.9.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.9.0'
}
```

<!--more-->


* 添加网络权限
```
<uses-permission android:name="android.permission.INTERNET"/>
```

* 基本使用
```
ImageView mImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http://***********";

        Glide .with(this)
                .load(Url)
                .into(targetImageView);
```

### 2.基本功能介绍&使用

* 图片的异步加载（基础功能）
```
ImageView mImageView = (ImageView) findViewById(R.id.ImageView);
        String Url = "http://**********";

//Glide使用了流式接口的调用方式
        Glide.with(context).load(Url).into(targetImageView);

```

* 设置图片加载尺寸
```
Glide.with(this).load(imageUrl).override(500, 500).into(imageView);
```

* 设置加载中以及加载失败图片
```
Glide
 .with(this)
  .load(imageUrl)
 .placeholder(R.mipmap.ic_launcher).error(R.mipmap.ic_launcher).into(imageView);
```

* 设置加载动画
```
Glide.with(this).load(imageUrl).animate(R.anim.item_alpha_in).into(imageView);
```

* 设置要加载的内容(图文混排)
```
Glide.with(this).load(imageUrl).centerCrop().into(new SimpleTarget<GlideDrawable>() {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> glideAnimation) {
                imageView.setImageDrawable(resource);
            }
        });
```

* 多样式的媒体加载
```
Glide
        .with(context)
        .load(imageUrl)；
        .thumbnail(0.1f)；//设置缩略图支持：先加载缩略图 (原图像的10%)然后在加载全图
素。
        .asBitmap()//显示gif静态图片 
        .asGif();//显示gif动态图片
        .into(imageView)；
```

* 设置磁盘缓存策略
```
Glide.with(this).load(imageUrl).diskCacheStrategy(DiskCacheStrategy.ALL).into(imageView);

// 缓存参数说明
// DiskCacheStrategy.NONE：不缓存任何图片，即禁用磁盘缓存
// DiskCacheStrategy.ALL ：缓存原始图片 & 转换后的图片（默认）
// DiskCacheStrategy.SOURCE：只缓存原始图片（原来的全分辨率的图像，即不缓存转换后的图片）
// DiskCacheStrategy.RESULT：只缓存转换后的图片（即最终的图像：降低分辨率后 / 或者转换后 ，不缓存原始图片
```

* 清理缓存
```
Glide.get(this).clearDiskCache();//清理磁盘缓存 需要在子线程中执行 
Glide.get(this).clearMemory();//清理内存缓存 可以在UI主线程中进行
```

* 生命周期集成
```
 Glide.with(Context context)// 绑定Context
        .with(Activity activity);// 绑定Activity
        .with(FragmentActivity activity);// 绑定FragmentActivity
        .with(Fragment fragment);// 绑定Fragment
```

-----------------------------

至此,Glide图片加载库基础使用就讲解完毕了,感谢阅读


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)


