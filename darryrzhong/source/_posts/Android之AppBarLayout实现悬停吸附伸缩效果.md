---
title: Android之AppBarLayout实现悬停吸附伸缩效果
date: 2019-03-27 11:47:26
tags: AppBarLayout
categories: Android
---


前几天看到这样一个UI效果,然后自己也仿照实现了下:

![开眼app个人中心.gif](https://upload-images.jianshu.io/upload_images/5549640-9e3cecfad8df9334.gif?imageMogr2/auto-orient/strip)


> 看着挺酷的,也有很多App都用到了这个UI效果,比如开眼App和沪江开心词场就用到了.
所以下面就来简单实现一下这个UI效果吧.

# 组合三剑客
 1.[AppBarLayout](https://developer.android.google.cn/reference/android/support/design/widget/AppBarLayout)
2.CoordinatorLayout
3.[CollapsingToolbarLayout](https://developer.android.google.cn/reference/android/support/design/widget/CollapsingToolbarLayout)

实现上面的UI效果需要将这三剑客的组合起来用,下面先介绍下这三个控件:

## AppBarLayout:
1.AppBarLayout简单介绍

AppBarLayout是`android.support:design`包中的支持的控件,继承自LinearLayout,实际上就是一个垂直分布的LinearLayout.父类视图结构如下:
```
public class AppBarLayout 
extends LinearLayout

java.lang.Object 
   ↳  android.view.View
      ↳ android.view.ViewGroup
           ↳  android.widget.LinearLayout  
              ↳ android.support.design.widget.AppBarLayout 

```

其中官方文档中有这么两句话尤为重要:
> This view depends heavily on being used as a direct child within a `CoordinatorLayout`. If you use AppBarLayout within a different `ViewGroup`, most of it's functionality will not work.

> `AppBarLayout ` also requires a separate scrolling sibling in order to know when to scroll. The binding is done through the `AppBarLayout.ScrollingViewBehavior` behavior class, meaning that you should set your scrolling view's behavior to be an instance of `AppBarLayout.ScrollingViewBehavior`.


意思就是说AppBarLayout 必须作为`CoordinatorLayout`的直接子类,否则很多功能是无法实现的.并且AppBarLayout 必须有一个能滚动的兄第ScrollView (实现了`NestedScrollView`,listview不可以哦),以此来通知AppBarLayout 何时进行滚动,兄弟View必须实现以下标识:

```
app:layout_behavior="@string/appbar_scrolling_view_behavior"
```


<!--more-->

官方给出的例子如下:
```
<android.support.design.widget.CoordinatorLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="match_parent">

     <android.support.v4.widget.NestedScrollView
             android:layout_width="match_parent"
             android:layout_height="match_parent"
             app:layout_behavior="@string/appbar_scrolling_view_behavior">

         <!-- Your scrolling content -->

     </android.support.v4.widget.NestedScrollView>

     <android.support.design.widget.AppBarLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent">

         <android.support.v7.widget.Toolbar
                 ...
                 app:layout_scrollFlags="scroll|enterAlways"/>

         <android.support.design.widget.TabLayout
                 ...
                 app:layout_scrollFlags="scroll|enterAlways"/>

     </android.support.design.widget.AppBarLayout>

 </android.support.design.widget.CoordinatorLayout>

```

2.AppBarLayout的具体使用

* AppBarLayout直接子View的几种响应方式
> AppbarLayout 可以指定当某个可滑动的兄弟View滑动手势改变时AppbarLayout 内部直接子View的响应动作,只要通过`app:layout_scrollFlags`属性来指定响应动作,`layout_scrollFlags`有5种响应动作,下面简单介绍下:

1. app:layout_scrollFlags="scroll"
当子view设置响应动作为`app:layout_scrollFlags="scroll"`时,子view会随ScrollView 的滚动而滚动,就相当于这时的子view变成了ScrollView 的item了,会跟随item一起滚动.
```
<android.support.v7.widget.Toolbar
                   android:layout_width="match_parent"
                   android:layout_height="?attr/actionBarSize"
                   app:title="AppbarLayout"
                   app:titleTextColor="@color/white"
                   app:layout_scrollFlags="scroll"
                   >

               </android.support.v7.widget.Toolbar>
```

2. app:layout_scrollFlags="scroll|enterAlways"
当子view设置响应动作为` app:layout_scrollFlags="scroll|enterAlways"`时,当ScrollView 向下滑动时，子View 将直接向下滑动，而不管ScrollView 是否在滑动。

```
<android.support.v7.widget.Toolbar
                   android:layout_width="match_parent"
                   android:layout_height="?attr/actionBarSize"
                   app:title="AppbarLayout"
                   app:titleTextColor="@color/white"
                   app:layout_scrollFlags="scroll|enterAlways"
                   />
```

3.app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
当子view设置响应动作为`app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed" `时,
当ScrollView 向下滑动的时候，子View（设置了enterAlwaysCollapsed 的子View）下滑至折叠的高度，当ScrollView 到达滑动范围的结束值的时候，滑动View剩下的部分开始滑动。这个折叠的高度是通过子View的minimum height （最小高度）指定的。

简单来说,就是第二种的加强版,当ScrollView 向下滑动时候,子view先露出半个头,当ScrollView 下滑到顶时,子view的头就全露出来了.

```
<android.support.v7.widget.Toolbar
                   android:layout_width="match_parent"
                   android:layout_height="200dp"
                   android:minHeight="?attr/actionBarSize"
                   app:title="AppbarLayout"
                   android:gravity="bottom"
                   android:layout_marginBottom="25dp"
                   app:titleTextColor="@color/white"
                   app:layout_scrollFlags="scroll|enterAlways|enterAlwaysCollapsed"
                   />
```

4.app:layout_scrollFlags="scroll|exitUntilCollapsed"
当子view设置响应动作为`app:layout_scrollFlags="scroll|exitUntilCollapsed"`时,
 当ScrollView 向上滑动时，子View先响应滑动事件，滑动至折叠高度，也就是通过minimum height 设置的最小高度后，就固定不动了，再把滑动事件交给 scrollview,然后 scrollview才开始滑动。

```
<android.support.v7.widget.Toolbar
                   android:layout_width="match_parent"
                   android:layout_height="200dp"
                   android:minHeight="?attr/actionBarSize"
                   app:title="AppbarLayout"
                   android:gravity="bottom"
                   app:titleTextColor="@color/white"
                   app:layout_scrollFlags="scroll|exitUntilCollapsed"
                   />
```

5.   app:layout_scrollFlags="scroll|snap"
当子view设置响应动作为`   app:layout_scrollFlags="scroll|snap"`时,当ScrollView 下滑到顶部时,如果子view只露出30%的话,子view就会自动折叠回去,如果露出60%的话,就会自动展开.

简单来说,就是具有弹性且遵守就近原则,露的小就干脆不露头了,露的大,就全部出来了.

```
<android.support.v7.widget.Toolbar
                   android:layout_width="match_parent"
                   android:layout_height="200dp"
                   android:minHeight="?attr/actionBarSize"
                   app:title="AppbarLayout"
                   android:gravity="bottom"
                   app:titleTextColor="@color/white"
                   app:layout_scrollFlags="scroll|snap"
                   />
```



##  CoordinatorLayout 
CoordinatorLayout 用来调节和控制子View的滚动，而这些子View 的具体响应动作是通过 behavior 属性来指定的,你也可以根据需求自定义自己的behavior, 简单使用如下:,

```
<android.support.design.widget.CoordinatorLayout
         xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:app="http://schemas.android.com/apk/res-auto"
         android:layout_width="match_parent"
         android:layout_height="match_parent">

     <android.support.v4.widget.NestedScrollView
             android:layout_width="match_parent"
             android:layout_height="match_parent"
             app:layout_behavior="@string/appbar_scrolling_view_behavior" >
 <!-- 使用此属性指定响应 -->

         <!-- Your scrolling content -->

     </android.support.v4.widget.NestedScrollView>

 </android.support.design.widget.CoordinatorLayout>

```

```
    app:layout_behavior="@string/appbar_scrolling_view_behavior" >
 <!-- 使用此属性指定响应 -->
```
 

##  CollapsingToolbarLayout:

CollapsingToolbarLayout也是`android.support:design`包中的支持的控件,继承自FrameLayout.主要用于实现ToolBar的伸缩效果,而且必须为`AppBarLayout`的直接子View;

继承结构图如下:
```
 java.lang.Object 
   ↳ android.view.View  
     ↳ android.view.ViewGroup  
        ↳ android.widget.FrameLayout   
           ↳  android.support.design.widget.CollapsingToolbarLayout 
```
主要使用到的方法如下:
1. setCollapsedTitleGravity

```
void setCollapsedTitleGravity（int gravity）
```

设置折叠标题和垂直重力的水平对齐方式，当折叠边界中有额外空间超出标题本身所需的空间时，将使用该对齐方式
相关的XML属性：
```
CollapsingToolbarLayout_collapsedTitleGravity
```
2.setExpandedTitleGravity
```
void setExpandedTitleGravity（int gravity）
```

设置展开标题和垂直重力的水平对齐方式，当扩展边界中有额外空间超出标题本身所需的空间时，将使用该对齐方式。
相关的XML属性：
```
CollapsingToolbarLayout_expandedTitleGravity
```

3.setExpandedTitleTextColor
```
void setExpandedTitleTextColor（ColorStateList colors）
```
设置展开标题的文本颜色。

4.setCollapsedTitleTextColor
```
void setCollapsedTitleTextColor（ColorStateList colors）
```
设置折叠标题的文本颜色。



5.setCollapsedTitleTypeface
```
void setCollapsedTitleTypeface（字体字体）
```
设置用于折叠标题的字体。

5.setExpandedTitleMarginBottom
```
void setExpandedTitleMarginBottom（int margin）
```
以像素为单位设置底部展开的标题边距。
相关的XML属性：
```
CollapsingToolbarLayout_expandedTitleMarginBottom
```

6. 固定Toolbar 
```
app:layout_collapseMode="pin"
```

6.[更多方法见文档](https://developer.android.google.cn/reference/android/support/design/widget/CollapsingToolbarLayout.html#setExpandedTitleMargin(int,%20int,%20int,%20int))


> 关于AppBarLayout的三剑客组合就介绍的差不多了,想进一步了解的可以去查阅官方文档,上面都给出了连接的.

### 特别说明:
三剑客配合使用，可以做出一些很炫酷的UI效果.
但是前提必须满足:AppbarLayout 要作为CoordinatorLayout 的直接子View，而CollapsingToolbarLayout 要作为AppbarLayout 的直接子View ，否则，上面展示的效果将实现不了.

# AppbarLayout实例展示

1.仿 [开眼App]个人中心效果

* .xml布局文件:

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout

    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:fresco="http://schemas.android.com/apk/res-auto"
    xmlns:imagetext="http://schemas.android.com/apk/res-auto"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:background="@color/colorWhite"

    >

<android.support.design.widget.AppBarLayout
    android:id="@+id/center_appbar_layout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:elevation="0dp"
    android:background="@color/colorWhite"
    android:fitsSystemWindows="true"

    >

    <android.support.design.widget.CollapsingToolbarLayout
        android:id="@+id/collapsing_toobar"
        android:layout_width="match_parent"
        android:layout_height="260dp"
        app:layout_scrollFlags="scroll|exitUntilCollapsed"
        app:contentScrim="@color/colorGraylight"
        fresco:expandedTitleTextAppearance="@style/style_textsize1"
        fresco:collapsedTitleTextAppearance="@style/style_textsize"
        fresco:collapsedTitleGravity="left"
       fresco:expandedTitleMarginTop="185dp"
        fresco:expandedTitleGravity="left"
        fresco:expandedTitleMarginStart="30dp"
        >
        <FrameLayout
            app:layout_scrollFlags="scroll"
            android:layout_width="match_parent"
            android:layout_height="180dp">

            <com.facebook.drawee.view.SimpleDraweeView
                android:id="@+id/avatar_max"
                android:layout_width="match_parent"
                android:layout_height="170dp" />


            <com.facebook.drawee.view.SimpleDraweeView
                android:layout_marginLeft="20dp"
                android:layout_gravity="bottom"
                android:id="@+id/avatar_min"
                android:layout_width="70dp"
                android:layout_height="70dp"
                fresco:actualImageScaleType="centerCrop"
                fresco:placeholderImageScaleType="centerCrop"
                fresco:roundedCornerRadius="50dp"
                />
        </FrameLayout>

        <FrameLayout
            android:layout_marginTop="180dp"
            app:layout_scrollFlags="scroll"
            android:layout_width="match_parent"
            android:layout_height="50dp"
            >

            <Button
                android:id="@+id/edit_btn"
                android:layout_width="60dp"
                android:layout_height="20dp"
                android:layout_marginRight="20dp"
                android:layout_gravity="right|center_vertical"
                android:background="@drawable/login_btn"
                android:gravity="center"
                android:text="编辑资料"
                android:textColor="@color/colorBlacklight"
                android:textSize="10sp" />
        </FrameLayout>

        <TextView
           android:layout_marginTop="230dp"
            app:layout_scrollFlags="scroll"
            android:textSize="10sp"
            android:id="@+id/date"
            android:layout_marginLeft="20dp"
            android:text="2018.07.08注册"
            android:textColor="@color/colorGraylight"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <android.support.v7.widget.Toolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            app:layout_collapseMode="pin"
            android:layout_height="?attr/actionBarSize"
            >
        </android.support.v7.widget.Toolbar>
    </android.support.design.widget.CollapsingToolbarLayout>







    <LinearLayout
        android:gravity="center_vertical"
        android:layout_marginTop="20dp"
        android:background="@color/colorGrayalpha"
        android:layout_width="match_parent"
        android:layout_height="70dp">

        <openeyes.dr.openeyes.widget.CustomImageTextView
            android:id="@+id/works"
            android:layout_marginLeft="40dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            imagetext:image="@drawable/works2"
            imagetext:text="@string/works"
            imagetext:textColor="@color/colorGraylight"
            >
        </openeyes.dr.openeyes.widget.CustomImageTextView>

        <openeyes.dr.openeyes.widget.CustomImageTextView
            android:id="@+id/attention"
            android:layout_marginLeft="90dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            imagetext:image="@drawable/attention6"
            imagetext:text="@string/attention"
            imagetext:textColor="@color/colorGraylight"
            >
        </openeyes.dr.openeyes.widget.CustomImageTextView>
        <openeyes.dr.openeyes.widget.CustomImageTextView
            android:id="@+id/fans"
            android:layout_marginLeft="90dp"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            imagetext:image="@drawable/works2"
            imagetext:text="@string/fans"
            imagetext:textColor="@color/colorGraylight"
            >
        </openeyes.dr.openeyes.widget.CustomImageTextView>
    </LinearLayout>

</android.support.design.widget.AppBarLayout>

    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycle_state"
        android:layout_width="match_parent"
        android:layout_height="match_parent"

        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        />




</android.support.design.widget.CoordinatorLayout>
```

* java文件:

```
/**
 * Created by darryrzhong on 2018/9/5.
 */

public class PersonPageActivity extends SwipeBackActivity {

    @BindView(R.id.avatar_max)
    SimpleDraweeView avatarBackground;
    @BindView(R.id.avatar_min)
    SimpleDraweeView avatarUser;
    @BindView(R.id.date)
    TextView registerDate;
    @BindView(R.id.works)
    CustomImageTextView works;
    @BindView(R.id.attention)
    CustomImageTextView attention;
    @BindView(R.id.fans)
    CustomImageTextView fans;
    @BindView(R.id.center_appbar_layout)
    AppBarLayout appBarLayout;
    @BindView(R.id.collapsing_toobar)
    CollapsingToolbarLayout collToobar;
    @BindView(R.id.toolbar)
    Toolbar toolbar;
    @BindView(R.id.recycle_state)
    RecyclerView recyclerView;
    private SharedPreferences sharedPreferences = MyApplication.sharedPreferences;
    private HistoryDB db;
    private List<HistoryDetails> details  = null;
    private DynamicStateRecyclerAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_person_page);
        ButterKnife.bind(this);
        initView();
        initData();
    }

    private void initData() {
        db = new HistoryDB();
        details= db.loadHistoryByUserId(sharedPreferences.getString("userId","000"));
        if (details==null||details.size()==0){

        }else {
            Collections.reverse(details);
            adapter = new DynamicStateRecyclerAdapter(details,this);
            LinearLayoutManager manager = new LinearLayoutManager(this);
            manager.setOrientation(LinearLayoutManager.VERTICAL);
            recyclerView.setLayoutManager(manager);
            recyclerView.setAdapter(adapter);
            recyclerView.setItemAnimator(new DefaultItemAnimator());//添加默认动画
            //设置RecyclerView的item监听事件
            setOnItemClickListener();
        }


    }

    private void setOnItemClickListener() {

        adapter.setOnItemClickListener(new DynamicStateRecyclerAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View itemview, DynamicStateRecyclerAdapter.MyViewHolder childview, int position) {
                initVideoDetail( position);//初始化视频详情界面数据并实现跳转
            }
        });
    }

    /**
     * 初始化视频详情界面数据,跳转至视频详情界面
     * */
    private void initVideoDetail(int position) {
        Intent intent = new Intent(PersonPageActivity.this, VideoDetailActivity.class);
        Bundle bundle = new Bundle();
        bundle.putString("title",details.get(position).getTitle());//获取标题
        bundle.putString("time", details.get(position).getDetail());
        bundle.putString("desc", details.get(position).getDesc());//视频描述
        bundle.putString("blurred", details.get(position).getBlurred());//模糊图片地址
        bundle.putString("feed", details.get(position).getPhoto());//图片地址
        bundle.putString("video", details.get(position).getVideo());//视频播放地址
        bundle.putInt("collect", details.get(position).getCollect());//收藏量
        bundle.putInt("share", details.get(position).getShare());//分享量
        bundle.putInt("reply", details.get(position).getReply());//回复数量
        intent.putExtras(bundle);
        startActivity(intent);
    }

    private void initView() {
        avatarBackground.setImageURI(Uri.parse(sharedPreferences.getString("userIcon","")));
        avatarUser.setImageURI(Uri.parse(sharedPreferences.getString("userIcon","")));
        toolbar.setTitle(sharedPreferences.getString("userName",""));//设置标题
        collToobar.setExpandedTitleColor(getResources().getColor(R.color.colorBlack));
        setSupportActionBar(toolbar);
        getSupportActionBar().setDisplayShowHomeEnabled(true);
        toolbar.setNavigationIcon(R.drawable.back_black);
        toolbar.setNavigationOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                finish();
            }
        });
    }

    @OnClick(R.id.attention)
    public void onClick(){
        startActivity(new Intent(this,MyAttentionActivity.class));
    }


}

```

效果展示:

![开眼app个人中心.gif](https://upload-images.jianshu.io/upload_images/5549640-7e47a81560bdce78.gif?imageMogr2/auto-orient/strip)


2.仿[开眼App]搜索悬停界面:

* .xml布局文件:
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="50dp">
        <openeyes.dr.openeyes.widget.SearchEditText
            android:id="@+id/search_editext"
            android:layout_marginLeft="20dp"
            android:background="@drawable/shape_search"
            android:layout_width="300dp"
            android:layout_height="30dp"
            android:paddingLeft="15dp"
            android:paddingRight="15dp"
            android:gravity="start|center_vertical"
            android:imeOptions="actionSearch"
            android:singleLine="true"
            android:hint="搜索视频、作者、用户及标签"
            android:textSize="13sp"

            />

        <TextView
            android:id="@+id/cancle_main"
            android:textColor="@color/colorBlacklight"
            android:gravity="center"
            android:text="取消"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    </LinearLayout>

    <FrameLayout
        android:id="@+id/history_fl"
        android:visibility="gone"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        >

        <TextView
            android:layout_gravity="center_vertical"
            android:layout_marginLeft="20dp"
            android:textSize="20sp"
            android:text="搜索历史"
            android:textColor="@color/colorBlack"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />
        <TextView
            android:id="@+id/delete_history"
            android:layout_marginRight="20dp"
            android:layout_gravity="center_vertical|right"
            android:textColor="@color/colorAniBtns"
            android:gravity="center"
            android:text="清空"
            android:textSize="13sp"
            android:layout_width="50dp"
            android:layout_height="wrap_content" />
    </FrameLayout>

    <android.support.design.widget.CoordinatorLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        android:background="@color/colorWhite"
        >


        <android.support.design.widget.AppBarLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:background="@color/colorWhite"

            >
            <android.support.v7.widget.RecyclerView
                android:id="@+id/search_history_rv"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                app:layout_scrollFlags="scroll"
                >

            </android.support.v7.widget.RecyclerView>


            <TextView
                android:layout_marginLeft="20dp"
                android:text="热搜关键词"
                android:textSize="20sp"
                android:textColor="@color/colorBlack"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />

        </android.support.design.widget.AppBarLayout>



        <android.support.v7.widget.RecyclerView
            android:id="@+id/hot_search_rv"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_behavior="@string/appbar_scrolling_view_behavior"
            >

        </android.support.v7.widget.RecyclerView>

    </android.support.design.widget.CoordinatorLayout>


</LinearLayout>
```

* java文件:

```
package openeyes.dr.openeyes.view.activity;

/**
 * Created by darryrzhong on 2018/9/10.
 */

public class SearchActivity extends AppCompatActivity {

    @BindView(R.id.search_editext)
    SearchEditText searchEditText;
    @BindView(R.id.cancle_main)
    TextView cancleMain;
    @BindView(R.id.history_fl)
    FrameLayout hintLayout;
    @BindView(R.id.delete_history)
    TextView deleteHistory;
    @BindView(R.id.search_history_rv)
    RecyclerView recyclerSearch;
    @BindView(R.id.hot_search_rv)
    RecyclerView recyclerHot;
    private String [] hotKeyWord = {"美食","旅行","生活小技巧","健身","汽车","广告","动画",
            "创意灵感","当下乱码","一条","日食记","视知TV"};
    private List<SearchHistory> searchHistories;
    private List<SearchHistory> hotKeyWords = new ArrayList<>();;
    private SearchRecyclerAdapter adapter;
    private SearchDB db;
    private SearchRecyclerAdapter adapter1;


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_search);
        ButterKnife.bind(this);
        initData();
        setListener();
    }

    private void initData() {
        db = new SearchDB();
        hotKeyWords.clear();
        for (String keyWord:hotKeyWord){
            hotKeyWords.add(new SearchHistory(keyWord));
        }

        LinearLayoutManager manager = new LinearLayoutManager(this);
        manager.setOrientation(LinearLayout.VERTICAL);
        recyclerHot.setLayoutManager(manager);
        adapter = new SearchRecyclerAdapter(hotKeyWords);
        recyclerHot.setAdapter(adapter);
        recyclerHot.setItemAnimator(new DefaultItemAnimator());

        searchHistories = db.loadSearchHistoryAll();
        if (searchHistories==null||searchHistories.size()==0){
            hintLayout.setVisibility(View.GONE);
            recyclerSearch.setVisibility(View.GONE);
        }else {
            hintLayout.setVisibility(View.VISIBLE);
            Collections.reverse(searchHistories);
            LinearLayoutManager manager1 = new LinearLayoutManager(this);
            manager.setOrientation(LinearLayout.VERTICAL);
            recyclerSearch.setLayoutManager(manager1);
            adapter1 = new SearchRecyclerAdapter(searchHistories);
            recyclerSearch.setAdapter(adapter1);
            recyclerSearch.setItemAnimator(new DefaultItemAnimator());//添加默认动画
            adapter1.setOnItemClickListener(new SearchRecyclerAdapter.OnItemClickListener() {
                @Override
                public void onItemClick(View itemview, SearchRecyclerAdapter.MyViewHolder childview, int position) {
                    Intent intent = new Intent(SearchActivity.this,SearchResultActivity.class);
                    Bundle bundle = new Bundle();
                    bundle.putString("keyWord",searchHistories.get(position).getKeyWord());
                    intent.putExtras(bundle);
                    startActivity(intent);
                }
            });
        }

    }

    private void setListener() {
        searchEditText.setOnKeyListener(new View.OnKeyListener() {
            @Override
            public boolean onKey(View view, int keyCode, KeyEvent keyEvent) {
                if (keyCode==KeyEvent.KEYCODE_ENTER&&keyEvent.getAction()==KeyEvent.ACTION_DOWN){
                       String keyWord = searchEditText.getText().toString().trim();
                       if ("".equals(keyWord)){
                           ToastUtil.showToast(SearchActivity.this,"请输入搜索内容");
                       }else {
                           SearchHistory searchHistory = new SearchHistory(keyWord);
                           List<SearchHistory> temp = db.loadSearchByKeyWord(keyWord);
                           if (temp==null){
                               db.saveOrUpdate(searchHistory);
                           }else if (temp.size()==0){
                               db.saveOrUpdate(searchHistory);
                           }
                           Intent intent = new Intent(SearchActivity.this,SearchResultActivity.class);
                           Bundle bundle = new Bundle();
                           bundle.putString("keyWord",keyWord);
                           intent.putExtras(bundle);
                           startActivity(intent);
                       }
                }
                return false;
            }
        });

        adapter.setOnItemClickListener(new SearchRecyclerAdapter.OnItemClickListener() {
            @Override
            public void onItemClick(View itemview, SearchRecyclerAdapter.MyViewHolder childview, int position) {
                Intent intent = new Intent(SearchActivity.this,SearchResultActivity.class);
                Bundle bundle = new Bundle();
                bundle.putString("keyWord",hotKeyWords.get(position).getKeyWord());
                intent.putExtras(bundle);
                startActivity(intent);
            }
        });


    }

    @OnClick({R.id.cancle_main,R.id.delete_history,R.id.search_editext})
    public void onClick(View v){
        switch (v.getId()){
            case R.id.cancle_main:
                finish();
                break;
            case R.id.delete_history:
                try {
                    db.delTable();
                } catch (DbException e) {
                    e.printStackTrace();
                }
                initData();

                break;
            case R.id.search_editext:
                break;
        }

    }

    @Override
    protected void onResume() {
        super.onResume();
        initData();
    }
}

```

效果展示:

![开眼app搜索记录.gif](https://upload-images.jianshu.io/upload_images/5549640-1e6bdf17608d33ae.gif?imageMogr2/auto-orient/strip)


> 好了,到这里关于AppBarLayout三剑客的基本使用就介绍完了,使用三剑客能够实现许多炫酷的UI效果,感兴趣的朋友可以自行自定义.


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)








