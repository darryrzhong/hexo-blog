---
title: 'Android RxJava系列三:  与Retrofit2结合使用和封装处理'
date: 2019-07-13 14:51:58
tags: Rxjava
categories: Android
---


# 前言
本篇文章主要介绍Rxjava与Retrofit结合使用,对Rxjava和Retrofit不熟悉的可以去看我之前的两篇介绍

* [Android RxJava：基础介绍与使用](https://www.jianshu.com/p/9cb743b98b84)
* [Android RxJava系列二: 常用拓展操作符](https://www.jianshu.com/p/063a133ccdbf)
* [Android Retrofit 2.5.0使用基础详解](https://www.jianshu.com/p/a8cecb30be38)

## 基本使用

### 定义请求接口

```
public interface GetRequest_Interface {

    @POST("/app/auth/school/list")
    Observable<School> postSchool(@Body RequestBody route);//根据学校名获取学校

}
   
```

###  创建 Retrofit接口实例

```

GetRequest_Interface request = new Retrofit.Builder()
                .baseUrl(API.BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build().create(GetRequest_Interface.class);

```



###  构建请求参数

这里以请求体为Json 字符串为准

```
HashMap<String, Object> map = new HashMap<>();
        map.put(key, value);
        Gson gson=new Gson();
        String strEntity = gson.toJson(map);
        KLog.d("22222222RequestBody"+strEntity);
        RequestBody body = RequestBody.create(okhttp3.MediaType.parse("application/json;charset=UTF-8"),strEntity);

```

<!--more-->


###  开始网络请求

```
 Observable<School> observable = request.postSchool(body);

 observable.subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<School>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                       //此处做一些请求开始时的初始化事件,例如弹出一个dialog
                    }

                    @Override
                    public void onNext(School school) {
                        //此处处理请求成功业务(code == 200 )
                    }

                    @Override
                    public void onError(Throwable e) {
                       //此处处理请求失败业务(code != 200 )
                    }

                    @Override
                    public void onComplete() {
                     //请求完成处理业务,关闭请求队列,关闭dialog等
                    }
                });

```

> 至此,Rxjava 与 Retrofit 结合基本使用就结束了,基于以上就可以愉快的完成网络请求工作了,是不是很方便简洁.


当然了,对于我们的业务来说,不可能只有一次网络请求,基本处处都需要进去网络请求,而且也不可能如上面一样,如此简单. 一般我们的业务中都需要配置一些其他的参数信息,如果每一次网络请求都像上面那样写的话,当然是可以的,但是你的代码就太low了,也不符合编程规范. 

所以基于你会熟练的使用了的前提下,我们就需要将以上代码进行简单封装.

> 关于封装我想多说一句

对于封装,很多人都认为封装就是使代码足够简洁,逻辑足够清晰,符合开闭原则等,的确是这样的,但是需要使情况而定,如果你写的代码是服务广大人群,也就是开源项目,那就要考虑很多因素,做到足够开放.

 但如果只是用到自己的项目里,那我们必须要明确一点,那就是定制化的前提是符合自己业务的需求,而不要过于封装.所以也就有为什么我们常常需要对别人的开源框架做二次封装,就是这个道理,没有最好的封装,只有最合适的封装.

## 封装篇

###   Url统一存放

```
public interface API {

   //此处存放所有的BaseUrl
    String BASE_URL = "http://xx.xxx.xx.225:8080"; //核心后台API
    String BASE_SCHOOL_URL = "http://xx.xxx.xx.225:8081"; //学校API


}

```

### 存放所有请求api
```
public interface GetRequest_Interface {

    /*-------------------------------------所有网络请求 API-------------------------------------------------------*/


    @POST("/app/auth/captcha")
    Observable<Phone> postPhone(@Body RequestBody route);  //获取验证码
    @POST("/app/auth/login")
    Observable<RegistLogin> postRegist(@Body RequestBody route);//登录注册

}
```

###  封装处理网络请求所需的参数信息

*  初始化 配置 OkHttpClient

```
根据自己业务需求初始化OkHttpClient

 OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(DEFAULT_TIME_OUT, TimeUnit.SECONDS)  //l 连接超时时间
                .writeTimeout(DEFAULT_READ_TIME_OUT,TimeUnit.SECONDS)  //读写超时
                .readTimeout(DEFAULT_READ_TIME_OUT,TimeUnit.SECONDS)   //读取超时
                .retryOnConnectionFailure(true)  //失败重连
                .addNetworkInterceptor(tokenInterceptor)  //添加网络拦截器
                .addInterceptor(tokenRespInterceptor)
                //.authenticator(authenticator)  //授权认证
                .build();

```

> 这里需要用到OkHttp3的拦截器相关内容,不熟悉的可以先去了解

* 统一添加公共请求头

```
 Interceptor tokenInterceptor = new Interceptor() {  //全局拦截器，
            @Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                Request originalRequest = chain.request();//获取原始请求
                Request.Builder requestBuilder = originalRequest.newBuilder() //建立新的请求
                        .addHeader("Accept", "application/json")
                        .addHeader("Content-Type", "application/json; charset=utf-8")
                        .removeHeader("User-Agent")
                        .addHeader("User-Agent",BaseUtils.getUserAgent())
                        .method(originalRequest.method(),originalRequest.body());
                return chain.proceed(requestBuilder.build()); //重新请求

```

* 全局动态添加Token 

```
 Interceptor tokenInterceptor = new Interceptor() {  //全局拦截器，往请求头部添加 token 字段，实现了全局添加 token
            @Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                Request originalRequest = chain.request();//获取原始请求
                Request.Builder requestBuilder = originalRequest.newBuilder() //建立新的请求
                        .addHeader("Accept", "application/json")
                        .addHeader("Content-Type", "application/json; charset=utf-8")
                        .removeHeader("User-Agent")
                        .addHeader("User-Agent",BaseUtils.getUserAgent())
                        .method(originalRequest.method(),originalRequest.body());
//                Log.e("----------------",originalRequest.body().toString());
                Request tokenRequest = null; //带有token的请求
                if (StringUtils.isEmpty(App.mmkv.decodeString(BaseConfig.TOKEN,null))){
                    return chain.proceed(requestBuilder.build());
                }

                tokenRequest = requestBuilder
                        .header("Authorization","Bearer "+App.mmkv.decodeString(BaseConfig.TOKEN,null))
                        .build();
                return chain.proceed(tokenRequest);
            }
        };

```

这里使用了腾讯的MMKV框架进去本地存储Token,因为我们一开始是没有拿到Token的,所以需要进行动态添加

* 自动判断token是否过期,过期无感刷新

在进行网络交互的时候,服务端签发的Token是有时效性的而且一般比较短,过了有效期就需要重新请求,而这个过程我们不能让用户察觉到,所以需要实现用户无感知的情况下刷新请求新的Token.

```
  Interceptor tokenRespInterceptor = new Interceptor() { //拦截返回体  判断token是否过期.过期重写拉取新的token
            @Override
            public Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                Response response = chain.proceed(request);
//                KLog.d( response.body().string());
                if (isTokenExpired(response)){
                    KLog.d( "自动刷新Token,然后重新请求数据");
                    //同步请求方式，获取最新的Token
                    String newToken = getNewToken();
                   if (newToken != null){
                        //使用新的Token，创建新的请求
                        Request newRequest = chain.request()
                                .newBuilder()
                                .header("Authorization", "Bearer " + newToken)
                                .build();
                        //重新请求
                        return chain.proceed(newRequest);
                    }

                }
                return response.newBuilder().body(ResponseBody.create(MediaType.parse("UTF-8"),body)).build();
            }
        };


```

这里需要根据服务端约定好的过期规则进去判断,这里简单示范一下

```


    /**
     * 根据Response，判断Token是否失效
     *
     * @param response
     * @return
     */
    private boolean isTokenExpired(Response response) {
        try {
            body = response.body().string();
            JSONObject object = new JSONObject(body);
            message = object.getString("Message");
            code = object.getInt("Code");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (JSONException e) {
            e.printStackTrace();
        }
        if ("Token is expired".equals(message)&& code == 1) {
            return true;
        }
        return false;
    }

```

获取新的Token

```
  /**
     * 同步请求方式，获取最新的Token
     *
     * @return
     */
    private String getNewToken() {
        // 通过获取token的接口，同步请求接口
        GetRequest_Interface request = new Retrofit.Builder()
                .baseUrl(API.BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build().create(GetRequest_Interface.class);
//        KLog.e(Remember.getString("refresh_token",null));
        RequestBody body = BaseUtils.convertJson(BaseUtils.paramsMap("refresh_token",App.mmkv.decodeString(BaseConfig.REFRESH_TOKEN,null)));
        Call<RefreshToken> call = request.postRefreshToken(body);
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        KLog.e(response.body().getCode()+response.body().getMessage());

        if (response.code() == 200 &&response.body().getCode() ==0){
            newToken = response.body().getData().getToken();
            KLog.e("---------------"+newToken);
            App.mmkv.encode(BaseConfig.TOKEN,newToken);
            App.mmkv.encode(BaseConfig.SCHOOL_TOKEN,response.body().getData().getSchool_token());
            App.mmkv.encode(BaseConfig.EXPIRE_IN,response.body().getData().getExpire_in());
        }

        return newToken;
    }

```


*  初始化 配置 Retrofit

```
  retrofit = new Retrofit.Builder()
                .client(client)
                .baseUrl(API.BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();

```

> 至此,关于网络请求的相关参数信息就基本配置完成

将上述配置步骤进行封装

```

/**
 * Created by darryrzhong
 * 
 *
 * 统一的Retrofit入口
 */
public class RetrofitHelper {

    private static final int DEFAULT_TIME_OUT = 5;//超时时间 5s
    private static final int DEFAULT_READ_TIME_OUT = 10;
    private final Retrofit retrofit;
    private String body;
    private retrofit2.Response<RefreshToken> response;
    private String newToken;
    private String message;
    private int code;

    private RetrofitHelper(){


        Interceptor tokenInterceptor = new Interceptor() {  //全局拦截器，往请求头部添加 token 字段，实现了全局添加 token
            @Override
            public okhttp3.Response intercept(Chain chain) throws IOException {
                Request originalRequest = chain.request();//获取原始请求
                Request.Builder requestBuilder = originalRequest.newBuilder() //建立新的请求
                        .addHeader("Accept", "application/json")
                        .addHeader("Content-Type", "application/json; charset=utf-8")
                        .removeHeader("User-Agent")
                        .addHeader("User-Agent",BaseUtils.getUserAgent())
                        .method(originalRequest.method(),originalRequest.body());
//                Log.e("----------------",originalRequest.body().toString());
                Request tokenRequest = null; //带有token的请求
                if (StringUtils.isEmpty(App.mmkv.decodeString(BaseConfig.TOKEN,null))){
                    return chain.proceed(requestBuilder.build());
                }

                tokenRequest = requestBuilder
                        .header("Authorization","Bearer "+App.mmkv.decodeString(BaseConfig.TOKEN,null))
                        .build();
                return chain.proceed(tokenRequest);
            }
        };


        Interceptor tokenRespInterceptor = new Interceptor() { //拦截返回体  判断token是否过期.过期重写拉取新的token
            @Override
            public Response intercept(Chain chain) throws IOException {
                Request request = chain.request();
                Response response = chain.proceed(request);
//                KLog.d( response.body().string());
                if (isTokenExpired(response)){
                    KLog.d( "自动刷新Token,然后重新请求数据");
                    //同步请求方式，获取最新的Token
                    String newToken = getNewToken();
                    if (newToken != null){
                        //使用新的Token，创建新的请求
                        Request newRequest = chain.request()
                                .newBuilder()
                                .header("Authorization", "Bearer " + newToken)
                                .build();
                        //重新请求
                        return chain.proceed(newRequest);
                    }
                }
                return response.newBuilder().body(ResponseBody.create(MediaType.parse("UTF-8"),body)).build();
            }
        };


        OkHttpClient client = new OkHttpClient.Builder()
                .connectTimeout(DEFAULT_TIME_OUT, TimeUnit.SECONDS)  //l 连接超时时间
                .writeTimeout(DEFAULT_READ_TIME_OUT,TimeUnit.SECONDS)  //读写超时
                .readTimeout(DEFAULT_READ_TIME_OUT,TimeUnit.SECONDS)   //读取超时
                .retryOnConnectionFailure(true)  //失败重连
                .addNetworkInterceptor(tokenInterceptor)  //添加网络拦截器
                .addInterceptor(tokenRespInterceptor)
                //.authenticator(authenticator)  //授权认证
                .build();


        retrofit = new Retrofit.Builder()
                .client(client)
                .baseUrl(API.BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build();



    }


    /**
     * 同步请求方式，获取最新的Token
     *
     * @return
     */
    private String getNewToken() {
        // 通过获取token的接口，同步请求接口
        GetRequest_Interface request = new Retrofit.Builder()
                .baseUrl(API.BASE_URL)
                .addConverterFactory(GsonConverterFactory.create())
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build().create(GetRequest_Interface.class);
        RequestBody body = BaseUtils.convertJson(BaseUtils.paramsMap("refresh_token",App.mmkv.decodeString(BaseConfig.REFRESH_TOKEN,null)));
        Call<RefreshToken> call = request.postRefreshToken(body);
        try {
            response = call.execute();
        } catch (IOException e) {
            e.printStackTrace();
        }
        KLog.e(response.body().getCode()+response.body().getMessage());

        if (response.code() == 200 &&response.body().getCode() ==0){
            newToken = response.body().getData().getToken();
            KLog.e("---------------"+newToken);
            App.mmkv.encode(BaseConfig.TOKEN,newToken);
            App.mmkv.encode(BaseConfig.SCHOOL_TOKEN,response.body().getData().getSchool_token());
            App.mmkv.encode(BaseConfig.EXPIRE_IN,response.body().getData().getExpire_in());
        }

        return newToken;
    }


    /**
     * 根据Response，判断Token是否失效
     *
     * @param response
     * @return
     */
    private boolean isTokenExpired(Response response) {
        try {
            body = response.body().string();
            JSONObject object = new JSONObject(body);
            message = object.getString("Message");
            code = object.getInt("Code");
        } catch (IOException e) {
            e.printStackTrace();
        } catch (JSONException e) {
            e.printStackTrace();
        }
        if ("Token is expired".equals(message)&& code == 1) {
            return true;
        }
        return false;
    }


    private static class SingletonHolder{
        private static final RetrofitHelper INSTANCE = new RetrofitHelper();
    }

    /**
     * 获取RetrofitServiceManager
     * @return
     */
    public static RetrofitHelper getInstance(){
        return SingletonHolder.INSTANCE;
    }



   /**
     * 获取对应的Service
     * @param service Service 的 class
     * @param <T>
     * @return
     */
    public  <T> T create(Class<T> service){
        return retrofit.create(service);
    }
    

}

```

如果业务中有多个BaseUrl的话,可以直接写个方法暴露出去就好了.

### 统一处理请求结果的BaseObserver

* 首先创建一个请求结果的回调 ResponseCallBack

```
public interface ResponseCallBack<T> {
    void onSuccess(T t);

    void onFault(String errorMsg);
}

```

* 创建一个请求开始时的初始化回调  ProgressListener

```
public interface ProgressListener {
    void startProgress();

    void cancelProgress();
}


```

* 创建统一处理结果的BaseObserver

创建BaseObserver,在回调中进行业务处理
```
public  class BaseObserver<T> implements Observer<T> {
        private ResponseCallBack responseCallBack;
        private ProgressListener progressListener;
        private Disposable disposable;

public BaseObserver(ResponseCallBack responseCallBack,ProgressListener progressListener){
     this.responseCallBack = responseCallBack;
     this.progressListener = progressListener;
 }


}

```

在 onSubscribe () 方法中进行请求开始时的初始化操作

```
  @Override
    public void onSubscribe(Disposable d) {
         this.disposable = d;
         if (progressListener != null){
             progressListener.startProgress();
         }
    }

```

在 onNext () 方法中处理请求成功业务

```
@Override
    public void onNext(T t) {
         responseCallBack.onSuccess(t);
    }

```

在onError () 方法中统一处理请求失败信息

```
 @Override
    public void onError(Throwable e) {
        KLog.d(e.getMessage());
        try {

            if (e instanceof SocketTimeoutException) {//请求超时
                responseCallBack.onFault("请求超时,请稍后再试");
            } else if (e instanceof ConnectException) {//网络连接超时
                responseCallBack.onFault("网络连接超时,请检查网络状态");
            } else if (e instanceof SSLHandshakeException) {//安全证书异常
                responseCallBack.onFault("安全证书异常");
            } else if (e instanceof HttpException) {//请求的地址不存在
                int code = ((HttpException) e).code();
                if (code == 504) {
                    responseCallBack.onFault("网络异常，请检查您的网络状态");
                } else if (code == 404) {
                    responseCallBack.onFault("请求的地址不存在");
                } else {
                    responseCallBack.onFault("请求失败");
                }
            } else if (e instanceof UnknownHostException) {//域名解析失败
                responseCallBack.onFault("域名解析失败");
            } else {
                responseCallBack.onFault("error:" + e.getMessage());
            }
        } catch (Exception e2) {
            e2.printStackTrace();
        } finally {
            Log.e("OnSuccessAndFaultSub", "error:" + e.getMessage());
            if (disposable !=null && !disposable.isDisposed()){ //事件完成取消订阅
                disposable.dispose();
            }
            if (progressListener!=null){
                progressListener.cancelProgress();
            }
        }
    }

```

在  onComplete () 中处理请求成功结束后的业务

```
 @Override
    public void onComplete() {
        if (disposable !=null && !disposable.isDisposed()){ //事件完成取消订阅
            disposable.dispose();
        }
        if (progressListener!=null){
            progressListener.cancelProgress();
        }
    }

```

代码如下:

```
/**
 * Created by darryrzhong
 * 
 */


public  class BaseObserver<T> implements Observer<T> {

 private ResponseCallBack responseCallBack;
 private ProgressListener progressListener;
 private Disposable disposable;

 public BaseObserver(ResponseCallBack responseCallBack,ProgressListener progressListener){
     this.responseCallBack = responseCallBack;
     this.progressListener = progressListener;
 }

    @Override
    public void onSubscribe(Disposable d) {
         this.disposable = d;
         if (progressListener != null){
             progressListener.startProgress();
         }
    }


    @Override
    public void onNext(T t) {
         responseCallBack.onSuccess(t);
    }

    @Override
    public void onError(Throwable e) {
        KLog.d(e.getMessage());
        try {

            if (e instanceof SocketTimeoutException) {//请求超时
                responseCallBack.onFault("请求超时,请稍后再试");
            } else if (e instanceof ConnectException) {//网络连接超时
                responseCallBack.onFault("网络连接超时,请检查网络状态");
            } else if (e instanceof SSLHandshakeException) {//安全证书异常
                responseCallBack.onFault("安全证书异常");
            } else if (e instanceof HttpException) {//请求的地址不存在
                int code = ((HttpException) e).code();
                if (code == 504) {
                    responseCallBack.onFault("网络异常，请检查您的网络状态");
                } else if (code == 404) {
                    responseCallBack.onFault("请求的地址不存在");
                } else {
                    responseCallBack.onFault("请求失败");
                }
            } else if (e instanceof UnknownHostException) {//域名解析失败
                responseCallBack.onFault("域名解析失败");
            } else {
                responseCallBack.onFault("error:" + e.getMessage());
            }
        } catch (Exception e2) {
            e2.printStackTrace();
        } finally {
            Log.e("OnSuccessAndFaultSub", "error:" + e.getMessage());
            if (disposable !=null && !disposable.isDisposed()){ //事件完成取消订阅
                disposable.dispose();
            }
            if (progressListener!=null){
                progressListener.cancelProgress();
            }
        }
    }


    @Override
    public void onComplete() {
        if (disposable !=null && !disposable.isDisposed()){ //事件完成取消订阅
            disposable.dispose();
        }
        if (progressListener!=null){
            progressListener.cancelProgress();
        }
    }
}

```


> 至此,统一处理结果的BaseObserver封装完毕

## 其他 (请求传参,返回JSON)
这里根据服务端接收数据不同而有不同方式,想要了解更多传参方式,可以自行去了解Retrofit的注解,这里只介绍向服务端传递Json数据.

* Json 数据格式
首先我们看一下标准的Json数据格式

```
返回体:

{
    "Code": 0,
    "Data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVC*********",
        "refresh_token": "c9ced011-***************************",
        "expire_in": 1560330991,
        "student_id": 33
    },
    "Message": "登录成功"
}

请求体:

{"phone":"13145214436","id":"12345"}

```

返回体的数据解析就不说了,说说请求体怎么传递

首先我们把最外面的{ } json 对象当做是一个 map 对象,这样是不是一下子就知道怎么转化了,对的,就是你想的那样.
 
```
 HashMap<String, Object> map = new HashMap<>();
        map.put("phone", "13145214436");
        map.put("id", "12345");

```


然后把map对象转化成json字符串,传给服务端就行了

```
        Gson gson=new Gson();
        String strEntity = gson.toJson(map);
RequestBody body = RequestBody.create(okhttp3.MediaType.parse("application/json;charset=UTF-8"),strEntity);

```

至于更复杂的请求体也是一样的做法

```
{
  "school_id":1,
  "student_id":23,
  "start_time":"2019-05-10 15:04:05",
  "end_time":"2019-06-10 15:04:05",
  "points": [
                {
                    "longitude": 0,
                    "latitude": 0
                },
                {
                    "longitude": 0,
                    "latitude": 0
                },
                {
                    "longitude": 0,
                    "latitude": 0
                },
                {
                    "longitude": 0,
                    "latitude": 0
                }
            ]
  
}

```

上面的请求体中有个json数组,数组里面又嵌套了json对象,还是一样的做法
把json数组看成是一个list,对的,有和上面一样的套路了是不是很简单,


## 使用示例

这里以登录业务做个简单示范

```

  GetRequest_Interface request = RetrofitHelper.getInstance().create(GetRequest_Interface.class);  //request请求入口

  HashMap<String,Object> params = new HashMap();
            params.put("phone",phone);
            params.put("id",id);
  RequestBody body = BaseUtils.convertJson(params);

  Observable<RegistLogin> observable= request.postRegist(body);

 observable.subscribeOn(Schedulers.io())
                      .observeOn(AndroidSchedulers.mainThread())
                      .subscribe(new BaseObserver<RegistLogin>(new ResponseCallBack<RegistLogin>() {
                          @Override
                          public void onSuccess(RegistLogin registLogin) {
                          //此处处理 code == 200 
                          }

                          @Override
                          public void onFault(String errorMsg) {
                               BaseUtils.showToast(mContext,errorMsg);
                          }
                      }, new ProgressListener() {
                          @Override
                          public void startProgress() {
                              dialog = BaseUtils.showSpotsDialog(mContext,"登录中");
                              dialog.show();
                          }

                          @Override
                          public void cancelProgress() {
                             dialog.dismiss();
                          }
                      }));

```

这样一来,是不是代码明了简洁,代码质量明显提高了一个层次

> 至此,Rxjava 和 Retrofit 结合使用 与封装就基本完成了,再次说明一下,没有最完美的封装,只有最适合自己业务的封装,所以,如果需要请进行自己的业务定制,这里只提供思路.


欢迎关注作者[darryrzhong](http://www.darryrzhong.xyz),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)




