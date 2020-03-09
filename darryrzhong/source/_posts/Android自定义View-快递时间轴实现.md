---
title: 'Android自定义View:快递时间轴实现'
date: 2019-03-28 00:15:56
tags: 自定义view
categories: Android
---

# 前言
* 在Android开发中，时间轴的 UI非常常见，如下图：
![TIM图片20190327232833.jpg](https://upload-images.jianshu.io/upload_images/5549640-d7336ab6e9d578b9.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 储备知识:
1.自定义view基础
2.RecyclerView的使用
3.自定义RecyclerView.ItemDecoration

## 具体实现
1.最终效果如下:
![TIM截图20190327231820.png](https://upload-images.jianshu.io/upload_images/5549640-7dcc8332e13a4132.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<!--more-->

2.实现思路
* 使用RecyclerView,自定义RecyclerView.ItemDecoration
* 复习ItemDecoration中getItemOffsets()方法,重写onDraw()方法
* 实现RecyclerView.Adapter,绑定数据

3.详细设计

![TIM截图20190327235039.png](https://upload-images.jianshu.io/upload_images/5549640-4cbf9b5b5d1c47a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![TIM截图20190327235010.png](https://upload-images.jianshu.io/upload_images/5549640-da8c7743a4115fff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.具体实现

* 引入RecyclerView依赖包
```
dependencies {
     ..........
    api 'com.android.support:recyclerview-v7:28.0.0'
}
```

* 在布局文件中使用
```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/my_recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scrollbars="horizontal"
        />


</RelativeLayout>
```

* 设置item布局
```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >

    <TextView
        android:id="@+id/item_title"
        android:text="New Text"
        android:textSize="15sp"
        android:layout_marginLeft="30dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="New Text"
        android:layout_marginLeft="30dp"
        android:textSize="15sp"
        android:id="@+id/item_text"
        android:layout_below="@+id/item_title"
        />

</LinearLayout>
```

* 实现RecyclerView.Adapter
```
public class MyAdapter extends RecyclerView.Adapter {
    private LayoutInflater inflater;
    private ArrayList<HashMap<String,Object>> listitem;

    //构造函数,传入数据
    public MyAdapter(Context context,ArrayList<HashMap<String, Object>> listitem) {
        this.inflater = LayoutInflater.from(context);
        this.listitem = listitem;
    }

    class ViewHolder extends RecyclerView.ViewHolder{
        private TextView title,text;

        public ViewHolder(@NonNull View itemView) {
            super(itemView);
            title = itemView.findViewById(R.id.item_title);
            text = itemView.findViewById(R.id.item_text);
        }

        public TextView getTitle() {
            return title;
        }

        public TextView getText() {
            return text;
        }


    }



    @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
        return new ViewHolder(inflater.inflate(R.layout.list_cell,null));
        //绑定item布局
    }

    @Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder viewHolder, int i) {
          //绑定数据到ViewHolder
        ViewHolder  vh = (ViewHolder) viewHolder;
        vh.title.setText((CharSequence) listitem.get(i).get("ItemTitle"));
        vh.text.setText((CharSequence) listitem.get(i).get("ItemText"));
    }

    @Override
    public int getItemCount() {
        return listitem.size();
    }
}
```

* 自定义RecyclerView.ItemDecoration
```
public class DividerItemDecoration extends RecyclerView.ItemDecoration {

    //轴点画笔
    private final Paint mPaint;
    //时分画笔
    private final Paint mPaint1;
    //年月画笔
    private final Paint mPaint2;
    //itemView 左 上 偏移量
    private int itemView_leftinterval;
    private int itemView_topintervarl;
    //轴点半径
    private  int circle_radius;
    private final Bitmap mIcon;


    //在构造函数里初始化需要属性
    public DividerItemDecoration(Context context){
        mPaint = new Paint();
        mPaint.setColor(Color.RED);//设置画笔颜色为红色

        mPaint1 = new Paint();
        mPaint1.setColor(Color.BLUE);
        mPaint1.setTextSize(30);//设置绘制字体大小

        mPaint2 = new Paint();
        mPaint2.setColor(Color.BLUE);
        mPaint2.setTextSize(15);

        itemView_leftinterval = 200; //左偏移长度200
        itemView_topintervarl = 50; //上偏移长度50

        circle_radius = 10;//轴点半径为10
        mIcon = BitmapFactory.decodeResource(context.getResources(),R.mipmap.logo);

    }

    @Override
    public void getItemOffsets(@NonNull Rect outRect, @NonNull View view, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        super.getItemOffsets(outRect, view, parent, state);

        //设置itemview的左上偏移量,即为onDraw可绘制的区域
        outRect.set(itemView_leftinterval,itemView_topintervarl,0,0);

    }

    @Override
    public void onDraw(@NonNull Canvas c, @NonNull RecyclerView parent, @NonNull RecyclerView.State state) {
        super.onDraw(c, parent, state);

        //获取RecyclerView的Child的个数
        int childCount = parent.getChildCount();
        //遍历每个item,分别获取他们的位置信息,然后在绘制对应的分割线
        for (int i=0;i<childCount;i++){
            View view = parent.getChildAt(i);//获取每个item对象

            /**
             * 绘制轴点
             */
            // 轴点 = 圆 = 圆心(x,y)

            float centerX = view.getLeft() - itemView_leftinterval/3;
            float centerY = view.getTop() - itemView_topintervarl+(itemView_topintervarl+view.getHeight()/2);
            // 绘制轴点圆
            //c.drawCircle(centerX,centerY,circle_radius,mPaint);
            c.drawBitmap(mIcon,centerX-circle_radius,centerY-circle_radius,mPaint);

            /**
             * 绘制上半轴线
             */
            // 上端点坐标(x,y)
            float upLine_up_x = centerX;
            float upLine_up_y =view.getTop()-itemView_topintervarl;

            // 下端点坐标(x,y)
            float upLine_down_x = centerX;
            float upLine_down_y = centerY-circle_radius;

            c.drawLine(upLine_up_x,upLine_up_y,upLine_down_x,upLine_down_y,mPaint);//绘制下半轴线

            /**
             * 绘制下半轴线
             */
            // 上端点坐标(x,y)
            float bottomLine_up_x = centerX;
            float bottom_up_y = centerY + circle_radius;

            // 下端点坐标(x,y)
            float bottomLine_bottom_x = centerX;
            float bottomLine_bottom_y = view.getBottom();

            //绘制下半部轴线
            c.drawLine(bottomLine_up_x, bottom_up_y, bottomLine_bottom_x, bottomLine_bottom_y, mPaint);


            /**
             * 绘制左边时间文本
             */
          int index =  parent.getChildAdapterPosition(view);
          //绘制时间文本起始位置
          float Text_x = view.getLeft()-itemView_leftinterval*5/6;
          float Text_y = upLine_down_y;

          //根据item位置设置时间

            switch (index){
                case 0:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                case 1:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                case 2:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                case 3:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                case 4:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                case 5:
                    //设置绘制日期
                    c.drawText("13:40",Text_x,Text_y,mPaint1);
                    c.drawText("2018.4.03",Text_x+5,Text_y+20,mPaint2);
                    break;
                    default:
                        c.drawText("已签收",Text_x,Text_y,mPaint1);

            }





        }

    }

}

```

* 初始化数据,绑定RecyclerView
```
public class MainActivity extends AppCompatActivity {

    private ArrayList<HashMap<String, Object>> itemlist;
    private RecyclerView rl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        initData();
        initView();
    }

    private void initData() {
        itemlist = new ArrayList<HashMap<String, Object>>();
        HashMap<String, Object> map1 = new HashMap<String, Object>();
        HashMap<String, Object> map2 = new HashMap<String, Object>();
        HashMap<String, Object> map3 = new HashMap<String, Object>();
        HashMap<String, Object> map4 = new HashMap<String, Object>();
        HashMap<String, Object> map5 = new HashMap<String, Object>();
        HashMap<String, Object> map6 = new HashMap<String, Object>();


        map1.put("ItemTitle", "中国广州公司已发出");
        map1.put("ItemText", "发件人:妙卡迪文化公司");
        itemlist.add(map1);

        map2.put("ItemTitle", "国际顺丰已收入");
        map2.put("ItemText", "等待中转");
        itemlist.add(map2);

        map3.put("ItemTitle", "国际顺丰转件中");
        map3.put("ItemText", "下一站中国");
        itemlist.add(map3);

        map4.put("ItemTitle", "中国顺丰已收入");
        map4.put("ItemText", "下一站江苏理工大学");
        itemlist.add(map4);

        map5.put("ItemTitle", "中国顺丰派件中");
        map5.put("ItemText", "等待派件");
        itemlist.add(map5);

        map6.put("ItemTitle", "江苏理工大学已签收");
        map6.put("ItemText", "收件人:darryrzhong");
        itemlist.add(map6);

    }

    private void initView() {
        rl = findViewById(R.id.my_recycler_view);
        LinearLayoutManager manager = new LinearLayoutManager(this);
        rl.setLayoutManager(manager);
        //当知道Adapter内Item的改变不会影响RecyclerView宽高的时候，可以设置为true让RecyclerView避免重新计算大小。
        rl.setHasFixedSize(true);
        rl.addItemDecoration(new DividerItemDecoration(this));//设置自定义分割线
        MyAdapter adapter =  new MyAdapter(this,itemlist);
        rl.setAdapter(adapter);
    }
}
```

至此,自定义RecyclerView就实现完成了.
![TIM截图20190327231820.png](https://upload-images.jianshu.io/upload_images/5549640-97c34967e42aef5f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 参考文章:
[Android 自定义View实战系列 ：时间轴](https://www.jianshu.com/p/655ea359e3db)


欢迎关注作者[darryrzhong](http://www.darryrzhong.site),更多干货等你来拿哟.

### 请赏个小红心！因为你的鼓励是我写作的最大动力！
>更多精彩文章请关注
- [个人博客:darryrzhong](http://www.darryrzhong.xyz)
- [掘金](https://juejin.im/user/5a6c3b19f265da3e49804988)
- [简书](https://www.jianshu.com/users/b7fdf53ec0b9/timeline)
- [SegmentFault](https://segmentfault.com/u/darryrzhong_5ac59892a5882/articles)
- [慕课网手记](https://www.imooc.com/u/6733207)
