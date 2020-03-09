---
title: 'Android RxJava系列二:  常用拓展操作符'
date: 2019-07-13 14:50:27
tags: Rxjava
categories: Android
---

# 前言
本篇文章主要介绍Rxjava 2.x的一些常用的操作符,对Rxjava不熟悉的朋友可以先去看下我之前的两篇介绍

* [Android RxJava：基础介绍与使用](https://www.jianshu.com/p/9cb743b98b84) 
* [Android RxJava系列一: 基础常用详解](https://www.jianshu.com/p/143d3ae8d1c6)

## 创建操作符

* create()  创建一个被观察者
```
public static <T> Observable<T> create(ObservableOnSubscribe<T> source)

```

```
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {
        e.onNext("This is Observer"); //通过 ObservableEmitter 发射器向观察者发送事件。
        e.onComplete();
    }
});

```

<!--more-->

* just()  创建一个被观察者，并发送事件，发送的事件不可以超过10个以上。

```
public static <T> Observable<T> just(T item) 
......
public static <T> Observable<T> just(T item1, T item2, T item3, T item4, T item5, T item6, T item7, T item8, T item9, T item10)

```

```
Observable.just(1, 2, 3)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "-------onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "-------onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "-------onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "-------onComplete ");
    }
});

```
使用just()方法创建Observable对象,Observable会将事件逐个发送

* From 操作符 
     *  fromArray()  这个方法和 just() 类似，只不过 fromArray 可以传入一个数组

     * fromCallable()   Callable 和 Runnable 的用法基本一致，只是它会返回一个结果值
    
    * fromIterable()    直接发送一个 List 集合数据给观察者

```
public static <T> Observable<T> fromArray(T... items)

Integer array[] = {1, 2, 3, 4};
Observable.fromArray(array)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "--------------onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "--------------onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "--------------onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "--------------onComplete ");
    }
});

```



```
public static <T> Observable<T> fromCallable(Callable<? extends T> supplier)

Observable.fromCallable(new Callable < Integer > () {

    @Override
    public Integer call() throws Exception {
        return 1;
    }
})
.subscribe(new Consumer < Integer > () {
    @Override
    public void accept(Integer integer) throws Exception {
        Log.d(TAG, "--------------accept " + integer);
    }
});


```




```
public static <T> Observable<T> fromIterable(Iterable<? extends T> source)


List<Integer> list = new ArrayList<>();
list.add(0);
list.add(1);
list.add(2);
list.add(3);
Observable.fromIterable(list)
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "--------------onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d(TAG, "--------------onNext " + integer);
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "--------------onError ");
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "--------------onComplete ");
    }
});


```

* empty()   直接发送 onComplete() 事件

```
Observable.empty()
.subscribe(new Observer < Object > () {

    @Override
    public void onSubscribe(Disposable d) {
        Log.d(TAG, "---------------------onSubscribe");
    }

    @Override
    public void onNext(Object o) {
        Log.d(TAG, "---------------------onNext");
    }

    @Override
    public void onError(Throwable e) {
        Log.d(TAG, "---------------------onError " + e);
    }

    @Override
    public void onComplete() {
        Log.d(TAG, "==================onComplete");
    }
});

```

##  转换操作符

* map()  map 可以将被观察者发送的数据类型转变成其他的类型,是一对一的转换

```
public final <R> Observable<R> map(Function<? super T, ? extends R> mapper)


//将 Integer 类型的数据转换成 String。
Observable.just(1, 2, 3)
.map(new Function < Integer, String > () {
    @Override
    public String apply(Integer integer) throws Exception {
        return  integer+"rxjava";
    }
})
.subscribe(new Observer < String > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.e(TAG, "----------------------onSubscribe");
    }

    @Override
    public void onNext(String s) {
        Log.e(TAG, "----------------------onNext " + s);
    }

    @Override
    public void onError(Throwable e) {
      Log.d(TAG, "---------------------onError " + e);
    }

    @Override
    public void onComplete() {
 Log.d(TAG, "---------------------onComplete" );
    }
});



```


* flatMap()  这个方法可以将事件序列中的元素进行整合加工，返回一个新的被观察者。

```
public final <R> Observable<R> flatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)

```

flatMap() 其实与 map() 类似，但是 flatMap() 返回的是一个 Observerable。现在用一个map()的例子和flatMap()的例子来对比说明 flatMap() 的用法。

需求:我们现在需要通过学校拿到院系列表,然后在每个院系中拿到学生的信息.
传统的实现方式有很多种,我就不举例了,直接使用Rxjava实现:

```
//学校
 class School{
        private String name;
        private List<Department> departments;

        public School(){}
        public School(String name, List<Department> departments) {
            this.name = name;
            this.departments = departments;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public List<Department> getDepartments() {
            return departments;
        }

        public void setDepartments(List<Department> departments) {
            this.departments = departments;
        }
    }

```


```

//院系
 class Department{
        private String name;
        private List<Student> students;

        public Department(){}
        public Department(String name, List<Student> students) {
            this.name = name;
            this.students = students;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public List<Student> getStudents() {
            return students;
        }

        public void setStudents(List<Student> students) {
            this.students = students;
        }
    }


```

```

//学生
class Student {
        private String name;
        private String school;

        public Student(){}
        public Student(String name, String school) {
            this.name = name;
            this.school = school;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public String getSchool() {
            return school;
        }

        public void setSchool(String school) {
            this.school = school;
        }
    }


```


使用map()方法实现:

```
 //使用map()实现方式
        Observable.fromIterable(departments)
                .map(new Function<Department, List<Student>>() {
                    @Override
                    public List<Student> apply(Department department) throws Exception {
                        return department.getStudents();
                    }
                })
                .subscribe(new Observer<List<Student>>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(List<Student> students) {
                       for (Student student : students){
                           Log.d("----------", student.getName()+student.getSchool() );
                        //如果还需要获取学生所有课程以及成绩
                       ......................
                       }
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });

```




```
  //使用flatMap()实现
        Observable.fromIterable(departments)
                .flatMap(new Function<Department, ObservableSource<Student>>() {
                    @Override
                    public ObservableSource<Student> apply(Department department) throws Exception {
                        return Observable.fromIterable(department.getStudents());
                    }
                })
                .flatMap() //如果还需要获取学生所有课程以及成绩操作
                .subscribe(new Observer<Student>() {
                    @Override
                    public void onSubscribe(Disposable d) {
                        
                    }

                    @Override
                    public void onNext(Student student) {
                       Log.d("---------",student.getName()+student.getSchool());
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onComplete() {

                    }
                });

```

以上代码中map()方法实现中,可以看到我们在onNext()方法中使用了for循环.如果代码逻辑在复杂一些,就可能需要嵌套for循环来实现,那就真的迷之缩进了,而使用flatMap()方法实现,只需要实现一个flatMap()转换一下就好了,随着代码逻辑增加,代码依然清晰,这就是flatMap()的强大之处,也是很多人喜欢使用Rxjava的原因所在.

* concatMap()    
 concatMap() 和 flatMap() 基本上是一样的，只不过 concatMap() 转发出来的事件是有序的，而 flatMap() 是无序的。

```

public final <R> Observable<R> concatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper)
public final <R> Observable<R> concatMap(Function<? super T, ? extends ObservableSource<? extends R>> mapper, int prefetch)


  Observable.fromIterable(departments)
                .concatMap(new Function<Department, ObservableSource<Student>>() {
                    @Override
                    public ObservableSource<Student> apply(Department department) throws Exception {
                        return Observable.fromIterable(department.getStudents());
                    }
                })


```

## 功能操作符

* delay()  延迟一段事件发送事件。


```
相当于handler的延迟发送事件

handler.sendEmptyMessageDelayed(0,2000);
```


```
public final Observable<T> delay(long delay, TimeUnit unit)

Observable.just(1, 2, 3)
.delay(2, TimeUnit.SECONDS) //延迟两秒再发送事件
.subscribe(new Observer < Integer > () {
    @Override
    public void onSubscribe(Disposable d) {
        Log.d("------------onSubscribe");
    }

    @Override
    public void onNext(Integer integer) {
        Log.d("------------"+integer);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {
        Log.d(TAG, "----------------onComplete");
    }
});

```

* doOnSubscribe()     Observable 每发送 onSubscribe() 之前都会回调这个方法。
 此方法通常用来做准备工作,例如弹一个ProgressDialog提示用户, But,这里有一个小坑,特别拿出来说明一下:

> 前方有坑,请集中注意力

Observable.doOnSubscribe()方法是在subscribe() 调用后而且在事件发送前执行。默认情况下， doOnSubscribe() 执行在 subscribe() 发生的线程；而如果在 doOnSubscribe() 之后有 subscribeOn() 的话，它将执行在离它最近的 subscribeOn() 所指定的线程。


```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("1");
                emitter.onNext("2");
                emitter.onNext("3");
                emitter.onComplete();
            }
        })
          .subscribeOn(Schedulers.io()) //在io执行上述操作
          .doOnSubscribe(new Consumer<Disposable>() {
              @Override
              public void accept(Disposable disposable) throws Exception {
                  dialog.show(); //显示dialog
              }
          })
          .subscribeOn(AndroidSchedulers.mainThread()) //在UI线程执行上述准备操作
          .observeOn(AndroidSchedulers.mainThread())//在UI线程执行下面操作
          .subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                Log.d("----","开始了");
            }

            @Override
            public void onNext(String s) {
                Log.d("----", s);
                dialog.dismiss();
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

* subscribeOn()  指定被观察者的线程，要注意的时，如果多次调用此方法，只有第一次有效。

```
public final Observable<T> subscribeOn(Scheduler scheduler)

```

* observeOn()  指定观察者的线程，每指定一次就会生效一次。

```
public final Observable<T> observeOn(Scheduler scheduler)

Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
    .subscribeOn(Schedulers.io())
    .observeOn(Schedulers.newThread())
    .map(mapOperator) // 新线程，由 observeOn() 指定
    .observeOn(Schedulers.io())
    .map(mapOperator2) // IO 线程，由 observeOn() 指定
    .observeOn(AndroidSchedulers.mainThread) 
    .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定

```

> 以上就是Rxjava常用的一些操作符介绍和使用方法实例了


----------------------------

* [RxJava2 只看这一篇文章就够了](https://juejin.im/post/5b17560e6fb9a01e2862246f)这是玉刚说的一篇关于Rxjava常用API的介绍,基本囊括了Rxjava所用到的所有API,还有代码举例,也是强烈建议观看收藏

关于Rxjava系列二就到此结束啦,后面有时间我还会写写与retrofit2的结合使用,欢迎关注订阅!


欢迎关注作者[darryrzhong](http://www.darryrzhong.xyz),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)
