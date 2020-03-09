---
title: Android之Activity全面解析
date: 2019-03-27 11:40:43
tags: activity
categories: Android
---


 # 前言

> 基于最近要准备去面试,特意系统的复习了下Android基础,看到Activity这块时,发现很多都忘了,而且之前也没有系统的学习和写笔记记录,所以,特此写下这篇关于Activity的一些理解,旨在帮助大家更好的理解Activity.


#  Activity是什么?
`Activity`是一个Android应用程序组件(也称为Android四大组件之一)，它提供了一个屏幕，用户可以通过该屏幕进行交互以执行某些操作，例如拨打电话，拍照，发送电子邮件或查看地图。每个活动都有一个窗口，用于绘制其用户界面。窗口通常填满屏幕，但可能比屏幕小，并漂浮在其他窗口的顶部.

Android应用程序通常由多个彼此松散绑定的`Activity`组成。通常，应用程序中的一个`Activity`被指定为“主要”`Activity`，该`Activity`在首次启动应用程序时呈现给用户。然后，每个`Activity`可以启动另一个`Activity`以执行不同的操作。每次新`Activity`开始时，前一个`Activity`都会停止，但系统会将`Activity`保留在`后台堆栈中`（“后堆栈”）。当一个新的`Activity`开始时，它会被推到后面的堆栈上，并引起用户的注意。后栈遵循基本的“ `后进先出`”堆栈机制，因此，当用户完成当前活动并按下"后退按钮"时，它从堆栈弹出（并销毁），之前的活动恢复。（后台堆栈将在后面为大家详细介绍。）

# 如何创建Activity
要创建Activity，您必须创建`Activity`（或其现有子类）的子类。在子类中，您需要实现当Activity在其生命周期的各个状态之间转换时系统调用的回调方法，例如在创建，停止，恢复或销毁活动时。两个最重要的回调方法是：

```
public class ExampleActivity extends AppCompatActivity {

 @Override
    public void onCreate(@Nullable Bundle savedInstanceState, @Nullable PersistableBundle persistentState) {
        super.onCreate(savedInstanceState, persistentState);
//您必须实现此方法。系统在创建Activity时调用此方法。在您的实施中，您应该初始化Activity的基本组成部分。最重要的是，您必须在此处调用以定义Activity用户界面的布局。
        setContentView();
    }


//系统将此方法称为用户离开您的Activity的第一个指示（尽管并不总是意味着Activity正在被销毁）。这通常是您应该提交应该在当前用户会话之外保留的任何更改的地方（因为用户可能不会回来）。

@Override
    protected void onPause() {
        super.onPause();
        //在此处应该提交应该在当前用户会话之外保留的任何更改的地方
    }

}

```

<!--more-->


> 在访问Activity时,必须在manifest中声明此Activity,

```
<manifest ... >
 <application ... > 
<activity android：name = “.ExampleActivity” />       ..
. </ application ... >   .
.. </ manifest >
```

你也可以指定Activity的`<intent-filter>`过滤器,如下:
```
<activity android：name = “. ExampleActivity ” android：icon = “@ drawable / app_icon” >
 <intent-filter> <action android：name = “android.intent.action.MAIN” /> 
<category android：name = “ android.intent.category.LAUNCHER“ />
 </ intent-filter>
 </ activity>  
```

# 如何启动Activity
您可以通过调用启动另一个Activity,通过`startActivity()`方法，并将`Intent`传递给您要启动的Activity。`intent`指定要启动的确切Activity或描述您要执行的操作类型（系统为您选择适当的活动，甚至可以来自不同的应用程序）。`Intent`(意图)还可以携带少量数据以供启动的活动使用。

* 启动指定自建的Activity
```
Intent intent = new Intent(this, SignInActivity.class);
startActivity(intent);
```
此种启动又叫做显示Intent .

* 启动其他类型的Activity
```
Intent intent = new Intent(Intent.ACTION_SEND);
intent.putExtra(Intent.EXTRA_EMAIL, recipientArray);
startActivity(intent);
```
此种启动又叫做隐式Intent .

> 有时候,我们可能需要从上一个Activity接收返回数据结果,这时,我们就需要另外一种启动方式了.

在这种情况下，通过调用startActivityForResult()（而不是startActivity()）来启动Activity。然后，要从后续Activity接收结果，就需要实现onActivityResult()回调方法。完成后续Activity后，它会在您的onActivityResult() 方法中返回结果。

```
private void pickContact() {
    // Create an intent to "pick" a contact, as defined by the content provider URI
    Intent intent = new Intent(Intent.ACTION_PICK, Contacts.CONTENT_URI);
    startActivityForResult(intent, PICK_CONTACT_REQUEST);
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    // If the request went well (OK) and the request was PICK_CONTACT_REQUEST
    if (resultCode == Activity.RESULT_OK && requestCode == PICK_CONTACT_REQUEST) {
        // Perform a query to the contact's content provider for the contact's name
        Cursor cursor = getContentResolver().query(data.getData(),
        new String[] {Contacts.DISPLAY_NAME}, null, null, null);
        if (cursor.moveToFirst()) { // True if the cursor is not empty
            int columnIndex = cursor.getColumnIndex(Contacts.DISPLAY_NAME);
            String name = cursor.getString(columnIndex);
            // Do something with the selected contact's name...
        }
    }
}
```

# 关闭Activity

您可以通过调用其`finish()方`法来关闭活动。您还可以关闭之前通过调用启动的单独活动` finishActivity()。`


>接下来便是整个Activity最核心的地方了,只要搞清楚一下内容,Activity也就理解的差不多了



#  Activity生命周期详解

Activity之所以能够成为Android四大组件之一,原因便是其具有非常灵活的生命周期回调方法,通过实现回调方法来管理Activity的生命周期对于开发强大而灵活的应用程序至关重要。Activity的生命周期直接受其与其他Activity，其任务和后台堆栈的关联的影响。


* Activity基本上存在于三种状态：

1. 恢复 onResume() 
Activity位于屏幕的前景并具有用户焦点。

2.已暂停 onPause() 
另一项Activity是在前台并具有焦点，但这一项仍然可见。也就是说，另一个Activity在这个Activity的顶部可见，该Activity部分透明或不覆盖整个屏幕。暂停的Activity完全处于活动状态（Activity 对象保留在内存中，它保留所有状态和成员信息，并保持附加到窗口管理器），但可以在极低内存情况下被系统杀死。

3.停止  onStop() 
该Activity完全被另一个Activity遮挡（活动现在位于“背景”中）。停止的Activity也仍然存在（Activity 对象保留在内存中，它维护所有状态和成员信息，但不 附加到窗口管理器）。但是，它不再对用户可见，并且当其他地方需要内存时，它可能被系统杀死。
如果Activity暂停或停止，系统可以通过要求它完成（调用其finish()方法）或简单地终止其进程来从内存中删除它。当活动再次打开时（在完成或杀死之后），必须全部创建它。

##  Activity完整生命周期回调方法

```
public class ExampleActivity extends Activity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // The activity is being created.
    }
    @Override
    protected void onStart() {
        super.onStart();
        // The activity is about to become visible.
    }
    @Override
    protected void onResume() {
        super.onResume();
        // The activity has become visible (it is now "resumed").
    }
    @Override
    protected void onPause() {
        super.onPause();
        // Another activity is taking focus (this activity is about to be "paused").
    }
    @Override
    protected void onStop() {
        super.onStop();
        // The activity is no longer visible (it is now "stopped")
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        // The activity is about to be destroyed.
    }
}
```

> 注意：在执行任何工作之前，以上这些生命周期方法的实现必须始终调用超类实现，如上面的示例所示。

1. Activity中存在的三个循环生命周期

* Activity的整个生命周期
Activity的整个生命周期发生在呼叫onCreate()和呼叫之间onDestroy()。您的Activity的整个生命周期应该执行“全局”状态的设置（例如定义布局）onCreate()，并释放所有剩余资源onDestroy()。

* Activity的可见生命周期
Activity的可见生命周期发生在呼叫onStart()和呼叫之间onStop()。在此期间，用户可以在屏幕上看到Activity并与之交互。例如，onStop()在新Activity开始时调用，并且此Activity不再可见。在这两种方法之间，您可以维护向用户显示Activity所需的资源。例如，你可以注册一个 BroadcastReceiver在onStart()监视会影响您的用户界面的变化，并在注销它onStop()时，用户将不再能够看到你所显示的内容。该系统可能会调用onStart()和onStop()活动的整个生命周期内多次为可见和隐藏用户之间的活动交替。

* Activity的前景生命周期
Activity的前景生命周期发生在调用onResume()和调用之间onPause()。在此期间，Activity位于屏幕上的所有其他活动之前，并且具有用户输入焦点。Activity可以频繁地进出前台 - 例如，onPause()当设备进入睡眠状态或出现对话框时调用。因为这种状态可以经常转换，所以这两种方法中的代码应该相当轻量级，以避免使用户等待的缓慢转换。

2.Activity生命周期循环图
![activity_lifecycle.png](https://upload-images.jianshu.io/upload_images/5549640-916e91d7d4c6f38b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 图中说明了这些循环以及活动在状态之间可能采用的路径。矩形表示当活动在状态之间转换时可以实现的回调方法。

3.Activity生命周期的回调方法摘要

|Method|Description|Killable after?|Next|
|-|-|:-:|-|
|onCreate()|在第一次创建活动时调用。这是您应该进行所有常规静态设置的地方 - 创建视图，将数据绑定到列表等等。如果捕获了该状态，则此方法将传递包含活动先前状态的Bundle对象|No|onStart()|
|onRestart()|在Activity停止后，再次启动之前调用。|No|onStart()|
|onStart()|在活动变得对用户可见之前调用。|No|onResume()或onStop()|
|onResume()|在活动开始与用户交互之前调用。此时，活动位于活动堆栈的顶部，用户输入转到活动堆栈。|No|onPause()|
|onPause()|当系统即将开始恢复另一个活动时调用。此方法通常用于将未保存的更改提交到持久数据，停止动画以及可能消耗CPU的其他事物，等等。它应该做很快的事情，因为下一个Activity在返回之前不会恢复。|Yes|onResume()或onStop()|
|onStop()|当活动不再对用户可见时调用。这可能是因为它正在被销毁，或者是因为另一个活动（现有的或新的活动）已经恢复并且正在覆盖它。|Yes|onRestart()或onDestroy()|
|onDestroy()|在活动被销毁之前调用。这是活动将收到的最后一个电话。可以调用它，因为活动正在完成（有人调用finish()它），或者因为系统暂时销毁此活动实例以节省空间。您可以使用该isFinishing()方法区分这两种情况。|Yes|	nothing|

其中`Killable after?`表示系统是否可以在方法返回后随时终止托管Activity的进程，而不执行Activity代码的下一个回调方法.

因为onPause()是三个允许`Killable after`方法中第一个执行的，一旦活动被创建，`onPause()是在回调过程中能够保证执行的最后一个方法`，如果系统必须在紧急情况下恢复记忆，是会调用onPause()方法的,但是onStop()和onDestroy()可能不被调用。因此，您应该使用onPause()将关键的持久性数据（例如用户编辑）写入存储。而不是onStop()或onDestroy().

> 所以在此处特别声明,要保存用户输入数据,需要在onPause()中执行,而不是在onStop()和onDestroy()中.

生命周期回调的顺序是明确定义的，特别是当两个Activity在同一个进程中而其中一个正在启动另一个时。以下是ActivityA启动ActivityB时发生的操作顺序：


```
1.ActivityA的onPause()方法执行。
2.ActivityB的onCreate()，onStart()和onResume() 方法执行顺序。（ActivityB现在具有用户关注点。）
3.然后，如果ActivityA在屏幕上不再可见，则onStop()执行其方法。
```
这种可预测的生命周期回调序列允许您管理从一个Activity到另一个Activity的信息转换。例如，如果您必须在第一个Activity停止时写入数据库以便以下Activity可以读取它，那么您应该在onPause()期间而不是在onStop()期间写入数据库。

3.保存Activity活动状态
当Activity暂停或停止，该Activity的活动状态会被保持,因为Activity对象在暂停或停止时仍保留在内存中 - 有关其成员和当前状态的所有信息仍然存在。因此，Activity会默认保留用户在活动中所做的任何更改，以便当活动返回到前台时（当它“恢复”时），那些更改仍然存在。

但是，当系统销毁Activity以恢复内存时，Activity对象被破坏，因此系统不能完好无损地恢复它的活动状态。在这种情况下，您可以通过实施额外的回调方法来确保保留有关活动状态的重要信息，该方法允许您保存有关活动状态的信息：

```
onSaveInstanceState()  //用来保存用户状态信息
```
onSaveInstanceState() 在Activity受破坏之前，系统会自动调用。系统会传递一个Bundle对象,您可以使用诸如putString()和putInt()之类的方法将关于Activity的状态信息保存为`名称 - 值`的键值对.

然后，如果系统终止您的应用程序进程,系统将重新创建活动并将其Bundle传递给Activity的onCreate()和onRestoreInstanceState()。使用这些方法之一，您可以从中提取已保存的状态Bundle并恢复活动状态。如果没有要恢复的状态信息，则Bundle传递给您的是null（这是第一次创建活动时的情况）。

如下图所示:
![restore_instance.png](https://upload-images.jianshu.io/upload_images/5549640-dd6569a60a33bcf2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


活动返回到用户焦点且状态完好无损的两种方式：活动被销毁，然后重新创建，活动必须恢复先前保存的状态，或者活动停止，然后恢复，活动状态仍然存在完整。

> 注意：onSaveInstanceState()在您的活动被销毁之前不能保证会被调用，因为在某些情况下不需要保存状态（例如当用户使用“ 返回”按钮离开您的活动时，因为用户是明确的关闭活动）。如果系统调用onSaveInstanceState()，则会在onPause()和onStop()之前执行.

再者就是,即使不执行任何操作并且未实现onSaveInstanceState(),只要Activity没有被销毁,Activity也会默认实现onSaveInstanceState()来恢复某些活动状态.例如EditText输入的文字,但前提是控件需要有`android:id `,否则不会默认保存状态.


4. onSaveInstanceState()常用场景

某些设备配置可能会在运行时更改（例如屏幕方向，键盘可用性和语言）。发生此类更改时，Android会重新创建正在运行的活动（系统调用onDestroy()，然后立即调用onCreate()）.处理此类重新启动的最佳方法是使用onSaveInstanceState()和onRestoreInstanceState()（或onCreate()）来保存和恢复Activity活动状态.


# Activity进出后台任务和后台堆栈详解

每个应用程序通常包含多个Activitys,每个Activity都应围绕用户可以执行的特定操作进行设计，并可以启动其他Activity,甚至可以启动设备上其他应用程序中存在的Activity.

Android设备主屏幕是大多数任务的起始位置。当用户触摸应用程序启动器中的图标（或主屏幕上的快捷方式）时，该应用程序的任务将到达前台。如果应用程序不存在任务（最近未使用该应用程序），则会创建一个新任务，该应用程序的“主”Activity将作为堆栈中的根Activity打开。

1.后台堆栈
后台堆栈指的是Activitys在后台任务中的排列规则,按"后进先出"排列.

例如当前Activity启动另一个Activity时，新Activity将被推到堆栈顶部并获得焦点。之前的Activity仍在堆栈中，但已停止。当活动停止时，系统将保留其用户界面的当前状态。当用户按下“ 返回” 按钮时，当前Activity将从堆栈顶部弹出（活动被销毁），之前的Activity将恢复（其UI的先前状态将恢复）。堆栈中的活动永远不会重新排列，只能在当前Activity启动时从堆栈推送和弹出到堆栈中，并在用户使用Back按钮退出时弹出.因此，后台堆栈作为“后进先出”对象结构操作。

如下图所示:
![diagram_backstack.png](https://upload-images.jianshu.io/upload_images/5549640-8a59e39c40d5d8f5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

图中表示任务中的每个新Activity如何将项添加到后台堆栈。当用户按下“ 返回”按钮时，将破坏当前Activity并恢复先前的Activity。

如果用户继续按Back，则弹出堆栈中的每个Activity以显示前一个Activity，直到用户返回主屏幕（或任务开始时运行的任何活动）。从堆栈中删除所有活动后，该任务不再存在。

2.后台任务
任务是一个内聚单元，当用户开始新任务或通过主页按钮进入主屏幕时，可以移动到“后台背景” 。在后台，任务中的所有Activitys都会停止，但任务的后台堆栈保持不变 - 任务在发生另一项任务时完全失去焦点.

如下图所示:
![diagram_multitasking.png](https://upload-images.jianshu.io/upload_images/5549640-20fe15469305d588.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两个任务：任务B在前台接收用户交互，而任务A在后台，等待恢复,这就是Android上的多任务处理的一个示例。

> 注意：可以在后台同时保存多个任务。但是，如果用户同时运行许多后台任务，系统可能会开始销毁后台活动以恢复内存，从而导致活动状态丢失。

由于后台堆栈中的活动永远不会重新排列，如果您的应用程序允许用户从多个Activitys启动特定Activity，则会创建该Acticity的新实例并将其推送到堆栈（而不是引入任何先前的活动实例）到顶部。因此，应用程序中的一个Acticity可能会被多次实例化（甚至来自不同的任务）.

如下图所示:

![diagram_multiple_instances.png](https://upload-images.jianshu.io/upload_images/5549640-6d33af4fe0be32ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果不希望多次实例化Acticity，则可以修改此行为。通常来说关于这部分内容是不太需要我们去更改的,所以这里我就不介绍了,有兴趣的同学可以自行翻阅`[官方文档]`.


3.定义Activity启动模式

启动模式允许您定义Activity的新实例与当前任务的关联方式。您可以通过两种方式定义不同的启动模式：

* 使用 manifest file(manifest 清单文件)

在清单文件中声明Activity时，可以使用<activity> 元素的`launchMode`属性指定Activity在启动时应如何与任务关联。

> launchMode 属性分配四有种不同的启动模式 ：

|modes|description|
|-|-|
|"standard" （默认模式）|默认。系统在启动它的任务中创建Activity的新实例，并将意图路由到该实例。Activity可以多次实例化，每个实例可以属于不同的任务，一个任务可以有多个实例。|
|"singleTop"|如果Activity的实例已存在于当前任务的顶部，则系统通过调用其onNewIntent()方法将意图路由到该实例，而不是创建Activity的新实例。Activity可以多次实例化，每个实例可以属于不同的任务，一个任务可以有多个实例（但只有当后端堆栈顶部的Activity不是Activity的现有实例时）。|
|"singleTask"|系统创建新任务并在新任务的根目录下实例化Activity。但是，如果Activity的实例已存在于单独的任务中，则系统会通过调用其onNewIntent()方法将意图路由到现有实例，而不是创建新实例。一次只能存在一个Activity实例。|
|"singleInstance"|相同"singleTask"，区别在于：系统不启动任何其他Activity纳入控制实例的任务。Activity始终是其任务的唯一成员; 任何由此开始的Activity都在一个单独的任务中打开。|

> 无论Activity是在新任务中启动还是在与启动它的Activity相同的任务中启动，“ 返回”按钮始终会将用户带到上一个Activity。但是，如果启动指定singleTask启动模式的活动，则如果后台任务中存在该Activity的实例，则将整个任务带到前台。此时，后端堆栈现在包括堆栈顶部提出的任务中的所有Activitys。

如下图所示:
![diagram_backstack_singletask_multiactivity.png](https://upload-images.jianshu.io/upload_images/5549640-99cea11671255a7d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


* 使用 Intent flags(意图传递)

当您调用时startActivity()，您可以在其中包含一个标志Intent，声明新Activity应如何（或是否）与当前任务相关联。

|Intent flags|description|
|-|-|
|FLAG_ACTIVITY_NEW_TASK|与上述"singleTask" 效果相同|
|FLAG_ACTIVITY_SINGLE_TOP|与上述"singleTop"效果相同|
|FLAG_ACTIVITY_CLEAR_TOP|如果正在启动的Activity已在当前任务中运行，则不会启动该Activity的新实例，而是销毁其上的所有其他Activitys，并将此意图传递给Activity的恢复实例|

> 注意：如果指定Activity的启动模式是 "standard"，它也将从堆栈中删除，并在其位置启动新实例以处理传入的意图。这是因为在启动模式下，总是为新意图创建一个新实例"standard"。


4.清理后堆栈

如果用户长时间离开任务，系统将清除除根Activity之外的所有Activitys的任务。当用户再次返回任务时，仅还原根Activity。系统以这种方式运行，因为在很长一段时间之后，用户可能已经放弃了之前正在做的事情并返回任务以开始新的事情。

可以使用一些Activity属性来修改此行为：

|attribute|description|
|-|-|
|alwaysRetainTaskState|如果任务的根Activity将此属性设置为"true"，则不会发生刚才描述的默认行为。即使经过很长一段时间，任务仍会保留堆栈中的所有Activity|
|clearTaskOnLaunch|与"alwaysRetainTaskState"正好相反,即使在离开任务片刻之后，用户也始终以初始状态返回任务|
|finishOnTaskLaunch|此属性类似clearTaskOnLaunch，但它在单个活动上运行，而不是在整个任务上运行。它还可以导致任何活动消失，包括根活动。当它设置为时"true"，活动仍然是当前会话的任务的一部分。如果用户离开然后返回任务，它将不再存在|


> 深呼一口气,不知不觉写了这么多,关于Activity的基本解析到这里就告一段落了,写完这些我对Activity又有了更深刻的理解和认识了,本文不止对Activity表面解析,更是深入到了一些底层驱动,希望看完这些能对你有所帮助.


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)















