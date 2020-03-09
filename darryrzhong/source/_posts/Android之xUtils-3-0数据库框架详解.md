---
title: Android之xUtils-3.0数据库框架详解
date: 2019-03-27 11:45:54
tags: xUtils
categories: Android
---


> 在Android 开发中,数据库模块是必不可少的.现在也有许多非常好用流行的数据库快速开发框架.今天主要介绍下xUtils下封装的数据库模块.

# Android 常用数据库开发框架
在这里列一下Android开发中常用的数据库框架,有情趣的可以自行了解下,顺便储备点知识.

1.[OrmLite](http://ormlite.com/)
对象关系映射Lite（ORM Lite）提供了一些简单，轻量级的功能，用于将Java对象持久保存到SQL数据库，同时避免了更多标准ORM包的复杂性和开销。

2.[LitePal](https://github.com/LitePalFramework/LitePal)
LitePal是一个开源的Android库，允许开发人员非常容易地使用SQLite数据库。您可以完成大部分数据库操作，而无需编写SQL语句，包括创建或升级表，crud操作，聚合函数等.LitePal的设置也非常简单，您可以在不到5分钟中将其集成到项目中。

3.[GreenDao](https://github.com/greenrobot/greenDAO)
greenDAO是一款轻巧快捷的Android版ORM，可将对象映射到SQLite数据库。greenDAO针对Android进行了高度优化，性能卓越，占用内存极少。

4.[DBFlow](https://github.com/Raizlabs/DBFlow)
一个功能强大，功能强大且非常简单的ORM android数据库库，带有注释处理功能。
该库建立在速度，性能和可接近性的基础之上。它不仅消除了大多数用于处理数据库的样板代码，而且还提供了一个强大而简单的API来管理交互。

5.[Realm](https://github.com/realm/realm-java)
Realm是一个直接在手机，平板电脑或可穿戴设备中运行的移动数据库。此存储库包含Realm的Java版本的源代码，该版本目前仅在Android上运行。

------

# xutils3.0数据库模块详解

## 项目加载
1.添加依赖
在app的build.gradle下添加以下依赖:
```
compile 'org.xutils:xutils:3.5.0'
```

`对于gradle3.0以上,compile已经被废弃了,需要使用api`
如下:
```
api 'org.xutils:xutils:3.5.0'
```

<!--more-->

2.配置权限
在AndroidManifest中添加权限:
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

```
3.初始化
在application的onCreate中初始化:
```
@Override
public void onCreate() {
    super.onCreate();
    x.Ext.init(this);
    x.Ext.setDebug(BuildConfig.DEBUG); // 是否输出debug日志, 开启debug会影响性能.
    ...
}
```

> 以上就是xutils3.0的使用配置了,配置好并且初始化好下面我们就可以愉快的使用了

##  xutils-DB使用

1.创建数据库帮助类:DatabaseOpenHelper
```

/**
 * 数据库帮助类
 * Created by darryrzhong on 2018/9/23.
 */

public class DatabaseOpenHelper {
  private DbManager.DaoConfig daoConfig;
  private static DbManager dbManager;
  private final String DB_NAME="history.db";//数据库名
  private final int VERSION = 1;


  private DatabaseOpenHelper(){
      //初始化数据库配置信息
      daoConfig = new DbManager.DaoConfig()
              .setDbName(DB_NAME)//设置数据库名称
              .setDbVersion(VERSION)//设置初始版本号
              .setDbOpenListener(new DbManager.DbOpenListener() {//监听数据库打开
                  @Override
                  public void onDbOpened(DbManager db) {
                     db.getDatabase().enableWriteAheadLogging();
                      //开启WAL, 对写入加速提升巨大(作者原话)
                  }
              })
              .setDbUpgradeListener(new DbManager.DbUpgradeListener() {//监听数据库更新
                  @Override
                  public void onUpgrade(DbManager db, int oldVersion, int newVersion) {

                  }
              });
      dbManager= x.getDb(daoConfig);//获取数据库管理类
  }

  //单例设计
  public static DbManager getInstance(){
     if (dbManager==null){
         DatabaseOpenHelper databaseOpenHelper = new DatabaseOpenHelper();
     }
     return dbManager;
  }
}

```

2.创建JavaBean数据实体类
先介绍下注解字段意思:
>注意必须添加注解字段,否则数据库无法识别,且实体类必须要有无参构造,否则数据库可能创建不成功
```
@Table(name = "details") 表名

 @Column(name = "userid") 字段名
```
实体类如下:

```

/**
 * 历史记录详情表
 * Created by darryrzhong on 2018/7/23.
 */


@Table(name = "details")//表名即实体类名
public class HistoryDetails {
    @Column(name = "id",isId = true,autoGen = true)//主键,自增
    private int id;
    @Column(name = "userid")
    private String userid;
    @Column(name = "photo")
    private String photo;//视频封面
    @Column(name = "title")
    private String title;//视频标题
    @Column( name = "detail")
    private String detail;//视频分类and时长
    @Column(name = "date")
    private String date;//观看日期
    @Column(name = "desc")
    private String desc;//视频详情
    @Column(name = "blurred")
    private String blurred;//模糊图片
    @Column(name = "video")
    private String video;//视频播放地址
    @Column(name = "collect")
    private int collect;//收藏量
    @Column(name = "share")
    private int share;//分享量
    @Column(name = "reply")
    private int reply;//回复量

    /**
     * 无参数构造法不可私有化，否则数据库表格创建异常
     * 若使用无参构造，容易引起数据插入只有一条。
     */
    public HistoryDetails(){

    }


    /**
    *
    * @param photo 视频封面
    * @param title 视频标题
    * @param detail 视频分类And时间
     *@param date 观看日期
    * @param desc 视频详情
    *@param blurred 迷糊图片
    * @param video 视频播放地址
    * @param collect 收藏量
    * @param share 分享量
    * @param reply 回复量
    * */

    public HistoryDetails(String photo, String title, String detail, String date, String desc, String blurred, String video, int collect, int share, int reply,String userid) {
        this.photo = photo;
        this.title = title;
        this.detail = detail;
        this.date = date;
        this.desc = desc;
        this.blurred = blurred;
        this.video = video;
        this.collect = collect;
        this.share = share;
        this.reply = reply;
        this.userid = userid;
    }

    public String getPhoto() {
        return photo;
    }

    public void setPhoto(String photo) {
        this.photo = photo;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getDetail() {
        return detail;
    }

    public void setDetail(String detail) {
        this.detail = detail;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public String getDesc() {
        return desc;
    }

    public void setDesc(String desc) {
        this.desc = desc;
    }

    public String getBlurred() {
        return blurred;
    }

    public void setBlurred(String blurred) {
        this.blurred = blurred;
    }

    public String getVideo() {
        return video;
    }

    public void setVideo(String video) {
        this.video = video;
    }

    public int getCollect() {
        return collect;
    }

    public void setCollect(int collect) {
        this.collect = collect;
    }

    public int getShare() {
        return share;
    }

    public void setShare(int share) {
        this.share = share;
    }

    public int getReply() {
        return reply;
    }

    public void setReply(int reply) {
        this.reply = reply;
    }

    public String getUserid() {
        return userid;
    }

    public void setUserid(String userid) {
        this.userid = userid;
    }
}

```
3.创建数据操作类

```
public class HistoryDB {

    private DbManager dbManager;
    private boolean succeed;//操作是否成功
    boolean idDesc = true;//是否倒序，默认false
    private List<HistoryDetails>  details =null;
    private HistoryDetails historyDetails;
    long count=0;

    //初始化的DbManager对象
    public HistoryDB(){
        dbManager=DatabaseOpenHelper.getInstance();
    }

    /**
     * 将HistoryDetails实例存进数据库
     * 保存新增
     * @param historyDetails
     *
     */

    public void saveHistory(HistoryDetails historyDetails){

        try {
            dbManager.save(historyDetails);
        } catch (DbException e) {
            e.printStackTrace();
        }
    }

    /**
     * 新增或更新
     *
     * @param historyDetails
     */

    public void saveOrUpdate(HistoryDetails historyDetails){
        try {
            dbManager.saveOrUpdate(historyDetails);
           // Log.e("log","save succeed!");
        } catch (DbException e) {
            e.printStackTrace();
        }
    }

    /**
     * 读取所有HistoryDetails信息
     *
     * @return
     */
  public List<HistoryDetails> loadHistoryAll(){
      try {
          details=dbManager.selector(HistoryDetails.class).findAll();
      } catch (DbException e) {
          e.printStackTrace();
      }
      return details;
  }

    /**
     * 根据视频标题读取HistoryDetails信息
     *
     * @return
     */

    public List<HistoryDetails> loadHistoryByTitle(String title){
        try {
            details=dbManager.selector(HistoryDetails.class).where("title","==",title).findAll();
        } catch (DbException e) {
            e.printStackTrace();
        }
        return details;
    }

    /**
     * 根据用户id读取HistoryDetails信息
     *
     * @return
     */

    public List<HistoryDetails> loadHistoryByUserId(String userid){
        try {
            details=dbManager.selector(HistoryDetails.class).where("userid","==",userid).findAll();
        } catch (DbException e) {
            e.printStackTrace();
        }
        return details;
    }




    /**
     * 删除对象数据
     *
     *
     * @return
     */
    public boolean delete(HistoryDetails historyDetails){
        try {
            dbManager.delete(historyDetails);
            succeed=true;
        } catch (DbException e) {
            succeed=false;
            e.printStackTrace();
        }
        return succeed;
    }


    /**
     * 删除表
     *
     * @throws DbException
     */

    private void delTable() throws  DbException{
        dbManager.dropTable(HistoryDetails.class);

    }

    //关闭数据库
    public void close(){
        try {
            dbManager.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}

```

4.使用数据库

创建以上操作类,就可以愉快的快速使用数据库了.

值得注意的是,xutils3并没有提供单独创建一张表的方法,他是在你调插入数据库操作的时候会判断是否存在这张表,如果不存在就会去创建,所以我们在取数据的时候需要判断表是否存在
```
 db = new HistoryDB();
        details= db.loadHistoryByUserId(sharedPreferences.getString("userId","000"));
       if (details==null||details.size()==0){//如果表为空的话则为null,
           tvHint.setVisibility(View.VISIBLE);
           compile.setVisibility(View.GONE);
           cleanHint.setVisibility(View.GONE);
       }
```

还有一点值得注意就是,xutils插入数据库时,相同数据是会重复插入的,
所以我们可以按自身需求去解决,例如我这里保存视频观看记录不需要重复插入数据.
所以我根据id判断数据是否存在,不存在则插入数据库保存:
```
  List<HistoryDetails> temp = hdb.loadHistoryByTitle(title);//同步默认账户观看记录
                   if (temp==null){
                       hdb.saveOrUpdate(historyDetails);//判断表是否存在
                   }else if (temp.size()==0){
                       hdb.saveOrUpdate(historyDetails);//判断数据是否存在
                   }
```

> xutils使用就这么简单的完成了数据库的编写,使用起来步骤简单明了,当然还有很多缺憾,所以数据库模块比较繁重的话还是推荐以上介绍的几种开发框架.


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)








