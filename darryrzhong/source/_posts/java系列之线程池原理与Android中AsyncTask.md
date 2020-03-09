---
title: java系列之线程池原理与Android中AsyncTask
date: 2019-09-15 15:07:50
tags: 线程池原理
categories: java

---



 

# 线程池原理与AsyncTask

## 什么是线程池？为什么要用线程池？

Java中的线程池是运用场景最多的并发框架，几乎所有需要异步或并发执行任务的程序都可以使用线程池。线程池就是将线程进行池化，需要运行任务时从池中拿一个线程来执行，执行完毕，线程放回池中。

在开发过程中，合理地使用线程池能够带来3个好处。

第一：降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。

第二：提高响应速度。当任务到达时，任务可以不需要等到线程创建就能立即执行。假设一个服务器完成一项任务所需时间为：T1 创建线程时间，T2 在线程中执行任务的时间，T3 销毁线程时间。   如果：T1 + T3 远大于 T2，则可以采用线程池，以提高服务器性能。线程池技术正是关注如何缩短或调整T1,T3时间的技术，从而提高服务器程序性能的。它把T1，T3分别安排在服务器程序的启动和结束的时间段或者一些空闲的时间段，这样在服务器程序处理客户请求时，不会有T1，T3的开销了。

第三：提高线程的可管理性。线程是稀缺资源，如果无限制地创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一分配、调优和监控。

## JDK中的线程池和工作机制

### 线程池的创建各个参数含义 

public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize,long keepAliveTime,TimeUnit unit,BlockingQueue<Runnable> workQueue,ThreadFactory threadFactory,RejectedExecutionHandler handler)

#### corePoolSize

线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize；

如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；

如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。

#### maximumPoolSize

线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize

#### keepAliveTime

线程空闲时的存活时间，即当线程没有任务执行时，继续存活的时间。默认情况下，该参数只在线程数大于corePoolSize时才有用

#### TimeUnit

keepAliveTime的时间单位

#### workQueue

workQueue必须是BlockingQueue阻塞队列。当线程池中的线程数超过它的corePoolSize的时候，线程会进入阻塞队列进行阻塞等待。通过workQueue，线程池实现了阻塞功能

##### 什么是阻塞队列 

**队列：**

队列是一种特殊的线性表，特殊之处在于它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作，和栈一样，队列是一种操作受限制的线性表。进行插入操作的端称为队尾，进行删除操作的端称为队头。队列中没有元素时，称为空队列。

队列的数据元素又称为队列元素。在队列中插入一个队列元素称为入队，从队列中删除一个队列元素称为出队。因为队列只允许在一端插入，在另一端删除，所以只有最早进入队列的元素才能最先从队列中删除，故队列又称为先进先出（FIFO—first in first out）线性表。

**阻塞队列：**

1）支持阻塞的插入方法：意思是当队列满时，队列会阻塞插入元素的线程，直到队列不满。

2）支持阻塞的移除方法：意思是在队列为空时，获取元素的线程会等待队列变为非空。

阻塞队列常用于生产者和消费者的场景，生产者是向队列里添加元素的线程，消费者是从队列里取元素的线程。阻塞队列就是生产者用来存放元素、消费者用来获取元素的容器。 ![img](java系列之线程池原理与Android中AsyncTask/clip_image002.jpg)

•抛出异常：当队列满时，如果再往队列里插入元素，会抛出IllegalStateException（"Queuefull"）异常。当队列空时，从队列里获取元素会抛出NoSuchElementException异常。

•返回特殊值：当往队列插入元素时，会返回元素是否插入成功，成功返回true。如果是移除方法，则是从队列里取出一个元素，如果没有则返回null。

•一直阻塞：当阻塞队列满时，如果生产者线程往队列里put元素，队列会一直阻塞生产者线程，直到队列可用或者响应中断退出。当队列空时，如果消费者线程从队列里take元素，队列会阻塞住消费者线程，直到队列不为空。

•超时退出：当阻塞队列满时，如果生产者线程往队列里插入元素，队列会阻塞生产者线程一段时间，如果超过了指定的时间，生产者线程就会退出。

##### 常用阻塞队列

•ArrayBlockingQueue：一个由数组结构组成的有界阻塞队列。

•LinkedBlockingQueue：一个由链表结构组成的有界阻塞队列。

•PriorityBlockingQueue：一个支持优先级排序的无界阻塞队列。

•DelayQueue：一个使用优先级队列实现的无界阻塞队列。

•SynchronousQueue：一个不存储元素的阻塞队列。

•LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。

•LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

#### threadFactory

创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名Executors静态工厂里默认的threadFactory，线程的命名规则是“pool-数字-thread-数字”

#### RejectedExecutionHandler（饱和策略）

线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略：

（1）AbortPolicy：直接抛出异常，默认策略；

（2）CallerRunsPolicy：用调用者所在的线程来执行任务；

（3）DiscardOldestPolicy：丢弃阻塞队列中靠最前的任务，并执行当前任务；

（4）DiscardPolicy：直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。



<!--more-->

### 线程池的工作机制 

1）如果当前运行的线程少于corePoolSize，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。

2）如果运行的线程等于或多于corePoolSize，则将任务加入BlockingQueue。

3）如果无法将任务加入BlockingQueue（队列已满），则创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。

4）如果创建新线程将使当前运行的线程超出maximumPoolSize，任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。

## 合理配置线程池

要想合理地配置线程池，就必须首先分析任务特性

要想合理地配置线程池，就必须首先分析任务特性，可以从以下几个角度来分析。

•任务的性质：CPU密集型任务、IO密集型任务和混合型任务。

•任务的优先级：高、中和低。

•任务的执行时间：长、中和短。

•任务的依赖性：是否依赖其他系统资源，如数据库连接。

性质不同的任务可以用不同规模的线程池分开处理。CPU密集型任务应配置尽可能小的线程，如配置Ncpu+1个线程的线程池。由于IO密集型任务线程并不是一直在执行任务，则应配置尽可能多的线程，如2*Ncpu。混合型的任务，如果可以拆分，将其拆分成一个CPU密集型任务和一个IO密集型任务，只要这两个任务执行的时间相差不是太大，那么分解后执行的吞吐量将高于串行执行的吞吐量。如果这两个任务执行时间相差太大，则没必要进行分解。可以通过Runtime.getRuntime().availableProcessors()方法获得当前设备的CPU个数。

## AsyncTask 

### 为什么需要AsyncTask？

在Android当中，当一个应用程序的组件启动的时候，并且没有其他的应用程序组件在运行时，Android系统就会为该应用程序组件开辟一个新的线程来执行。默认的情况下，在一个相同Android应用程序当中，其里面的组件都是运行在同一个线程里面的，这个线程我们称之为Main线程。当我们通过某个组件来启动另一个组件的时候，这个时候默认都是在同一个线程当中完成的。

在Android当中，通常将线程分为两种，一种叫做Main Thread，除了Main Thread之外的线程都可称为Worker Thread。

当一个应用程序运行的时候，Android操作系统就会给该应用程序启动一个线程，这个线程就是我们的Main Thread，这个线程非常的重要，它主要用来加载我们的UI界面，完成系统和我们用户之间的交互，并将交互后的结果又展示给我们用户，所以Main Thread又被称为UI Thread。

Android系统默认不会给我们的应用程序组件创建一个额外的线程，所有的这些组件默认都是在同一个线程中运行。然而，某些时候当我们的应用程序需要完成一个耗时的操作的时候，例如访问网络或者是对数据库进行查询时，此时我们的UI Thread就会被阻塞。例如，当我们点击一个Button，然后希望其从网络中获取一些数据，如果此操作在UI Thread当中完成的话，当我们点击Button的时候，UI线程就会处于阻塞的状态，此时，我们的系统不会调度任何其它的事件，更糟糕的是，当我们的整个现场如果阻塞时间超过5秒钟(官方是这样说的)，这个时候就会出现 ANR (Application Not Responding)的现象，此时，应用程序会弹出一个框，让用户选择是否退出该程序。对于Android开发来说，出现ANR的现象是绝对不能被允许的。

另外，由于我们的Android UI控件是线程不安全的，所以我们不能在UI Thread之外的线程当中对我们的UI控件进行操作。因此在Android的多线程编程当中，我们有两条非常重要的原则必须要遵守：

 

l  绝对不能在UI Thread当中进行耗时的操作，不能阻塞我们的UI Thread

l  不能在UI Thread之外的线程当中操纵我们的UI元素

 既然在Android当中有两条重要的原则要遵守，那么我们可能就有疑问了？我们既不能在主线程当中处理耗时的操作，又不能在工作线程中来访问我们的UI控件，那么我们比如从网络中要下载一张图片，又怎么能将其更新到UI控件上呢？这就关系到了我们的主线程和工作线程之间的通信问题了。在Android当中，提供了两种方式来解决线程直接的通信问题，一种是通过Handler的机制，这个时候就很可能自己会去封装一下thread+handler了，正是因为这类需求很多，google就帮我们封装了一下。其实我们也可以自己封装，但是我相信99%程序员自己封装的东西比不上google的。所以另外一种就是今天要详细讲解的 AsyncTask 机制。

### 原理分析

AsyncTask是个abstract类，所以在使用时需要实现一个AsyncTask的具体实现类，一般来说会覆盖4个方法，我们以前面所说的从网络中下载一张图片，然后更新到UI控件来说明：

（1）onPreExecute()：在执行后台下载操作之前调用，将下载等待动画显示出来，运行在主线程中；

（2）doInBackground()：核心方法，执行后台下载操作的方法，必须实现的一个方法，运行在子线程中；这个方法是执行在子线程中的。在onPreExecute()执行完后，会立即开启这个方法。

（3）onProgressUpdate()：在下载操作doInBackground()中调用publishProgress()时的回调方法，用于更新下载进度，运行在主线程中；

（4）onPostExecute()：后台下载操作完成后调用，将下载等待动画进行隐藏，并更新UI，运行在主线程中；

然后在主线程中

![img](java系列之线程池原理与Android中AsyncTask/clip_image004.jpg)

通过上面的分析，我们可以知道，AsyncTask的构造方法和execute方法是我们分析AsyncTask的重点。

#### 1）构造方法

AsyncTask的构造方法中![img](java系列之线程池原理与Android中AsyncTask/clip_image006.jpg)

mWorker代表了AsyncTask要执行的任务，是对Callable接口的封装，意味着这个任务是有返回值的

![img](java系列之线程池原理与Android中AsyncTask/clip_image008.jpg)

mFuture代表了AsyncTask要执行的任务的返回结果，其实就是个FutureTask，安装FutureTask标准用法，mWorker作为Callable被传给了mFuture，那么mFuture的结果就从mWorker执行的任务中取得。仔细看mWorker，return语句返回的结果就是我们前面所说的doInBackground()的执行结果：

![img](java系列之线程池原理与Android中AsyncTask/clip_image010.jpg)

#### 2）再看执行流程

查看源码execute()àexecuteOnExecutor(sDefaultExecutor, params)àexec.execute(mFuture)

到了这一步，将mFuture传递给了AsyncTask的执行器进行执行。AsyncTask的执行器缺省是sDefaultExecutor。

找到成员变量sDefaultExecutor，最终定位到

![img](java系列之线程池原理与Android中AsyncTask/clip_image012.jpg)

SerialExecutor是对JDK里Executor的一个实现，被声明为一个静态变量，我们仔细看SerialExecutor的实现，

![img](java系列之线程池原理与Android中AsyncTask/clip_image014.jpg)

内部声明了一个双端队列ArrayDeque类型的mTasks（双端队列中offer方法表示从队列尾插入，poll()表示从队列头获取元素）。

每次调用execute，就创建一个Runnable匿名内部类对象，这个对象存入mTasks，在匿名内部类的run函数里面调用传入参数r.run()。然后通过一个scheduleNext函数把mTasks里面的所有对象通过THREAD_POOL_EXECUTOR.execute(mActive)执行一遍。说穿了，也就是说SerialExecutor类会把所有的任务丢入一个容器，之后把容器里面的所有对象**一个一个的排队（串行化）**执行THREAD_POOL_EXECUTOR.execute(mActive);

至于这个THREAD_POOL_EXECUTOR，是这样定义的：

![img](java系列之线程池原理与Android中AsyncTask/clip_image016.jpg)

我们可以看到这个线程池，被声明为一个静态变量，同时初始化的参数是：

核心线程数量，这里取得是CPU个数 + 1， 第二个参数是最大线程数量，这里是CPU个数 * 2 + 1，第五个参数是缓冲区的队列，这里是个LinkedBlockingQueue，这个队列的最大容量是128。

#### 3)结果和进度的通知

AsyncTask的执行结果和进度是怎么通知给UI线程的呢？检视mFuture

![img](java系列之线程池原理与Android中AsyncTask/clip_image018.jpg)

和更新进度时我们会调用的publishProgress方法

![img](java系列之线程池原理与Android中AsyncTask/clip_image020.jpg)

我们可以看到都调用了sHandler

![img](java系列之线程池原理与Android中AsyncTask/clip_image022.jpg)

![img](java系列之线程池原理与Android中AsyncTask/clip_image024.jpg)

说明当子线程需要和UI线程进行通信时，其实就是通过这个handler，往UI线程发送消息。

#### 总结：

1）、线程池的创建：

在创建了AsyncTask的时候，会默认创建两个线程池SerialExecutor和ThreadPoolExecutor，SerialExecutor负责将任务串行化，ThreadPoolExecutor是真正执行任务的地方，且无论有多少个AsyncTask实例，两个线程池都会只有一份。

2）、任务的执行：

在execute中，会执行run方法，当执行完run方法后，会调用scheduleNext()不断的从双端队列中轮询，获取下一个任务并继续放到一个子线程中执行，直到异步任务执行完毕。

3）、消息的处理：

在执行完onPreExecute()方法之后，执行了doInBackground()方法，然后就不断的发送请求获取数据；在这个AsyncTask中维护了一个InternalHandler的类，这个类是继承Handler的，获取的数据是通过handler进行处理和发送的。在其handleMessage方法中，将消息传递给onProgressUpdate()进行进度的更新，也就可以将结果发送到主线程中，进行界面的更新了。

4）、使用AsyncTask的注意点

通过观察代码我们可以发现，每一个new出的AsyncTask只能执行一次execute()方法，多次运行将会报错，如需多次，需要新new一个AsyncTask。

![img](java系列之线程池原理与Android中AsyncTask/clip_image026.jpg)

### AsyncTask优缺点

**AsyncTask****：**

优点：AsyncTask是一个轻量级的异步任务处理类，轻量级体现在，使用方便、代码简洁上，而且整个异步任务的过程可以通过cancel()进行控制；

缺点：不适用于处理长时间的异步任务，一般这个异步任务的过程最好控制在几秒以内，如果是长时间的异步任务就需要考虑多线程的控制问题；当处理多个异步任务时，UI更新变得困难。

**Handler:**

优点：代码结构清晰，容易处理多个异步任务；

缺点：当有多个异步任务时，由于要配合Thread或Runnable，代码可能会稍显冗余。

**总之，**AsyncTask不失为一个非常好用的异步任务处理类，只要不是频繁对大量UI进行更新，可以考虑使用；而Handler在处理大量UI更新时可以考虑使用。

# 补充知识：CAS

## 什么是原子操作？如何实现原子操作？

假定有两个操作A和B，如果从执行A的线程来看，当另一个线程执行B时，要么将B全部执行完，要么完全不执行B，那么A和B对彼此来说是原子的。

实现原子操作可以使用锁，锁机制，满足基本的需求是没有问题的了，但是有的时候我们的需求并非这么简单，我们需要更有效，更加灵活的机制，synchronized关键字是基于阻塞的锁机制，也就是说当一个线程拥有锁的时候，访问同一资源的其它线程需要等待，直到该线程释放锁，这里会有些问题：首先，如果被阻塞的线程优先级很高很重要怎么办？其次，如果获得锁的线程一直不释放锁怎么办？（这种情况是非常糟糕的）。还有一种情况，如果有大量的线程来竞争资源，那CPU将会花费大量的时间和资源来处理这些竞争（事实上CPU的主要工作并非这些），同时，还有可能出现一些例如死锁之类的情况，最后，其实锁机制是一种比较粗糙，粒度比较大的机制，相对于像计数器这样的需求有点儿过于笨重

实现原子操作还可以使用当前的处理器基本都支持CAS()的指令，只不过每个厂家所实现的算法并不一样罢了，**每一个****CAS****操作过程都包含三个运算符：一个内存地址****V****，一个期望的值****A****和一个新值****B****，操作的时候如果这个地址上存放的值等于这个期望的值****A****，则将地址上的值赋为新值****B****，否则不做任何操作。**CAS的基本思路就是，如果这个地址上的值和期望的值相等，则给其赋予新值，否则不做任何事儿，但是要返回原值是多少。**循环****CAS****就是在一个循环里不断的做****CAS****操作，直到成功为止。**怎么实现线程安全呢？语言层面不做处理，我们将其交给硬件—CPU和内存，利用CPU的多处理能力，实现硬件层面的阻塞，再加上volatile变量的特性即可实现基于原子操作的线程安全。

## CAS实现原子操作的三大问题

1)  ABA问题。因为CAS需要在操作值的时候，检查值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加1，那么A→B→A就会变成1A→2B→3A。

举个通俗点的例子，你倒了一杯水放桌子上，干了点别的事，然后同事把你水喝了又给你重新倒了一杯水，你回来看水还在，拿起来就喝，如果你不管水中间被人喝过，只关心水还在，这就是ABA问题。如果你是一个讲卫生讲文明的小伙子，不但关心水在不在，还要在你离开的时候水被人动过没有，因为你是程序员，所以就想起了放了张纸在旁边，写上初始值0，别人喝水前麻烦先做个累加才能喝水。

2)  循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

3)  只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用循环的方式来保证原子操作，但是对多个共享变量操作时，循环就无法保证操作的原子性，这个时候就可以用锁。还有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量＝，，合并一下，然后用来操作。从开始，提供了类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行操作。