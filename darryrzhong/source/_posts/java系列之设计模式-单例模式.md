---
title: java系列之设计模式(单例模式)
date: 2019-09-15 17:17:15
tags: 单例模式
categories: 设计模式
---



1.饿汉
如果应用程序总是创建并使用单例实例或在创建和运行时开销不大

```java
class Single {
    private Single(){}

    private static Single single= new Single();
    
    public static Single getInstance(){
        return single;
    }
```

2.懒汉
如果开销比较大，希望用到时才创建就要考虑延迟实例化
Singleton的初始化需要某些外部资源(比如网络或存储设备)

```java
class Single {
    private Single(){}

    private static Single single= null;
    
    public static Single getInstance(){
        if ( single == null ) {
            synchronized (Single.class) {
                if ( single == null ) {
                    single = new Single();
                }
            }
    
        }
        return single;
    }
}
```

3.静态内部类

```java
class Single {
    private Single(){}

    private static class SingleHandler{
        private static Single single = new Single();
    }
    
    public static Single getInstance(){
        return Single.SingleHandler.single;
    }
```



<!--more-->





4.枚举

```java
public class Single {

    private Single(){}
    
    public enum SingleEnum {
        singleHandler;
        private Single single;
        private SingleEnum () {
            single = new Single();
        }
    
        public Single getSingle() {
            return single;
        }
    }
    
    public static Single getInstacne() {
        return SingleEnum.singleHandler.getSingle();
    }
}
```