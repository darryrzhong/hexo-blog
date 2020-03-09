---
title: Android Retrofit 2.5.0使用基础详解
date: 2019-03-28 19:37:41
tags: Retrofit
categories: Android
---

# 前言
* 在Andrroid开发中，网络请求必不可少
* 而在Android所有网络请求库中，`Retrofit`是最受开发者欢迎的一个网络请求库

[retrofit:2.5.0 官方文档](https://square.github.io/retrofit/)

[retrofit:2.5.0 - github](https://github.com/square/retrofit)

## 简介
* `Retrofit`是Square公司开发的一款针对Android网络请求的框架,遵循Restful设计风格,底层基于OkHttp.

## 功能
* 支持同步/异步网络请求
* 支持多种数据的解析&序列化格式(Gson、json、XML等等)
* 通过注解配置网络请求参数
*  提供对Rxjava的支持
* 高度解耦,使用方便


## 对比其他网络请求框架
* 性能最好,速度最快
* 高度封装导致扩展性差
*  简洁易用,代码简化
* 解耦彻底,职责细分
* 易与其他框架联用(Rxjava)

<!--more--> 

## 使用场景
* 任何场景下建议优先使用

## 网络请求流程
* App应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、Url 等信息，之后由 OkHttp 完成后续的请求操作
* 在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit根据用户的需求对结果进行解析

![请求流程](https://upload-images.jianshu.io/upload_images/5549640-93219ec2c71748f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 具体使用

### 1.添加Retrofit库的依赖

```
dependencies {
      implementation 'com.squareup.retrofit2:retrofit:2.5.0'
     api 'com.squareup.retrofit2:converter-gson:2.0.2'
}
```

### 2. 添加 网络权限

```
<uses-permission android:name="android.permission.INTERNET"/>
```

### 3. 创建 装载服务器返回数据 的类
```
public class ResultData{
    ...
    // 根据返回数据的格式和数据解析方式（Json、XML等）定义

        }

```

### 4. 创建 用于配置网络请求 的接口

* Retrofit将 Http请求 抽象成 Java接口：采用 注解 描述网络请求参数 和配置网络请求参数
```
public interface GetRequestInterface {

    @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
    Call<ResultData>  getCall();
    // @GET注解的作用:采用Get方法发送网络请求
 
    // getCall() = 接收网络请求数据的方法
    // 其中返回类型为Call<*>，*是接收数据的类（即上面定义的Translation类）
    // 如果想直接获得Responsebody中的内容，可以定义网络请求返回值为Call<ResponseBody>
}
```

#### 注解说明
* 网络请求方法:@GET、@POST、@PUT、@DELETE、@HEAD(常用)
* 网络请求标记: @FormUrlEncoded、@Multipart、@Streaming
* 网络请求参数:  @Header &、@Headers、 @Body、@Field 、 @FieldMap、@Part 、 @PartMap、@Query、@QueryMap、@Path、@Url

具体作用以及解释请自行前往官方文档查看,这里就不一一解释了

### 5. 创建 Retrofit 实例
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") // 设置网络请求的公共Url地址
                .addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava平台
                .build();
```
* 数据解析器说明
 Retrofit支持多种数据解析方式,使用时需要在Gradle添加依赖

数据解析器
Gradle依赖

Gson
com.squareup.retrofit2:converter-gson:2.0.2

----------------------

Jackson
com.squareup.retrofit2:converter-jackson:2.0.2
----------------------

Simple XML
com.squareup.retrofit2:converter-simplexml:2.0.2

------------------------
Protobuf
com.squareup.retrofit2:converter-protobuf:2.0.2

--------------------
Moshi
com.squareup.retrofit2:converter-moshi:2.0.2
--------------

Wire
com.squareup.retrofit2:converter-wire:2.0.2

---------------------
Scalars
com.squareup.retrofit2:converter-scalars:2.0.2
------------------------

* 网络适配器说明
Retrofit支持多种网络请求适配器方式：guava、Java8和rxjava
Android 提供默认的 CallAdapter，不需要添加网络请求适配器的依赖，若要使用其他网络适配器,则需要按照需求在Gradle添加依赖

网络请求适配器
Gradle依赖

guava
com.squareup.retrofit2:adapter-guava:2.0.2
----------------------

Java8
com.squareup.retrofit2:adapter-java8:2.0.2

-----------------------
rxjava
com.squareup.retrofit2:adapter-rxjava:2.0.2
--------------------------

### 6.创建 网络请求接口实例
```
// 创建 网络请求接口 的实例
        GetRequestInterface request = retrofit.create(GetRequestInterface.class);

        //对 发送请求 进行封装
        Call<ResultData> call = request.getCall();
```

### 7.  发起网络请求（异步 / 同步）

```
/发送网络请求(异步)
        call.enqueue(new Callback<ResultData>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<ResultData> call, Response<ResultData> response) {
                //处理结果
         
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<ResultData> call, Throwable throwable) {
             //提示失败
            }
        });

// 发送网络请求（同步）
Response<ResultData> response = call.execute();
```


### 8. 处理返回数据

```
//发送网络请求(异步)
        call.enqueue(new Callback<ResultData>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<ResultData> call, Response<ResultData> response) {
                // 对返回数据进行处理
                response.body();//拿到ResultData对象进行数据操作
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<ResultData> call, Throwable throwable) {
                System.out.println("连接失败");
            }
        });

// 发送网络请求（同步）
  Response<ResultData> response = call.execute();
  // 对返回数据进行处理
  response.body().blabla;

```

------------------------

## 总结
* Retrofit 是一个 restful 的 HTTP 网络请求框架的封装。
* 网络请求的工作本质上是 OkHttp 完成，而 Retrofit 仅负责 网络请求接口的封装
* App应用程序通过 Retrofit 请求网络，实际上是使用 Retrofit 接口层封装请求参数、Header、Url 等信息，之后由 OkHttp 完成后续的请求操作
* 在服务端返回数据之后，OkHttp 将原始的结果交给 Retrofit，Retrofit根据用户的需求对结果进行解析
* 相对其他开源库而言代码简洁使用更加方便.

> 关于Retrofit 2.5的简单介绍到这里就结束了,感谢阅读.

## 参考文章:
* [Android Retrofit 2.0 的详细 使用攻略（含实例讲解）](https://www.jianshu.com/p/a3e162261ab6)
* [Android：手把手带你 深入读懂 Retrofit 2.0 源码](https://www.jianshu.com/p/0c055ad46b6c)


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)



