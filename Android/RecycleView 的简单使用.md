#RecycleView 的简单使用
最近在做一個電商的項目，有許多列表需要用來展示商品和其他一些條目，並且這些列表的佈局大不相同，於是乎就想到了早在5.0就發佈的RecycleView,當時只是了解了一些他是ListView與GridleView的升華版，支持多種佈局，封裝了ViewHolder，解決了複用問題，总之是被到处夸，于是乎决定就来试一下这个控件。
## 1 概念

A flexible view for providing a limited window into a large data set.
在限定的窗口中显示许多数据的一个灵活布局。哈哈，官方给的第一句话，尊重官方。简单点来说其实就是类似ListView 一样用来显示数据的一个控件。来看一下他的效果吧！


## 2 能干什么

 - 在列表上显示数据
 - 支持横向\竖向\瀑布流的布局
 - 更优的View复用机制
 - 支持对单条Item进行操作
 - 快速的切换布局
 - 支持多种Item动画，效果nice~

## 3 怎么用

第一步 在module的build.gradle 中添加依赖（EC用户需要去下jar包）
```
    compile 'com.android.support:recyclerview-v7:23.4.0'
```
第二步 布局文件中引用RecycleView

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    android:id="@+id/activity_recycle"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.webber.webberdemo.RecycleActivity">

    <android.support.v7.widget.RecyclerView
        android:id="@+id/load_more_rv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

</RelativeLayout>
```
第三步 在Activity中初始化控件

这一步代码比较多主要是分为 1findViewById 2 setLayoutManager 3 addItemDecoration 4 setAdapter

```
package com.webber.webberdemo;

import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import java.util.ArrayList;
import java.util.List;

public class RecycleActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_recycle);
        initRecycleView();
    }

    private void initRecycleView() {
        RecyclerView mLoadMoreRv = (RecyclerView) findViewById(R.id.load_more_rv);
        //创建布局管理器 LinearLayoutManager为线性布局，支持横向、纵向。
        //               GridLayoutManager()为网格布局。
        //               StaggeredGridLayoutManager为瀑布流布局
        RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(this);
        //设置布局管理器
        mLoadMoreRv.setLayoutManager(mLayoutManager);
        //添加条目间距 RecycleView 没有提供类似ListView的divider方法，间距需要自己画，也就意味着我们也可以使用各种自定义的间距样式
        mLoadMoreRv.addItemDecoration(new DividerItemDecoration(this, DividerItemDecoration.VERTICAL_LIST));

        //创建适配器及数据
        List<String> mData = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            mData.add("item" + i);
        }
        MyAdapter myAdapter = new MyAdapter(mData);
        mLoadMoreRv.setAdapter(myAdapter);
    }

    //RecycleView 适配器，泛型可以传自己定义的ViewHolder
    private class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {

        private List<String> list;

        MyAdapter(List<String> list) {
            this.list = list;
        }

        @Override
        public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
            return new MyViewHolder(LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.layout_simple_text, parent, false));
        }

        //这个方法拿到绑定的ViewHolder 在此处处理数据
        @Override
        public void onBindViewHolder(MyViewHolder holder, int position) {
            holder.mItemTv.setText(list.get(position));
        }

        @Override
        public int getItemCount() {
            return list == null ? 0 : list.size();
        }

        class MyViewHolder extends RecyclerView.ViewHolder {

            TextView mItemTv;

            MyViewHolder(View itemView) {
                super(itemView);
                mItemTv = (TextView) itemView.findViewById(R.id.item_recycle_tv);
            }
        }
    }
}

```

##4 一些基本操作 

####4.1 RecycleView添加item 点击事件

RecycleView 没有像ListView 一样的setOnItemClickListener，需要我们自己去实现，思路是在onBindViewHolder的时候 拿到itemView，设置点击监听。来看下效果先：

第一步 定义一个接口
```
 interface OnRVItemClickListener {
	  void onRVItemClickListener(RecyclerView recyclerView, View itemView, int pos);
    }
```

第二步 在MyAdapter添加setClick方法

```
 //RecycleView 适配器，泛型可以传自己定义的ViewHolder
    private class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {

        private List<String> list;

        private OnRVItemClickListener onRVItemClickListener;

        public void setOnRVItemClickListener(OnRVItemClickListener listener){
            this.onRVItemClickListener = listener;
        }
        
        ......
          
        //这个方法拿到绑定的ViewHolder 在此处处理数据
        @Override
        public void onBindViewHolder(MyViewHolder holder, final int position) {
            holder.mItemTv.setText(list.get(position));
            if (onRVItemClickListener != null) {
                holder.itemView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        onRVItemClickListener.onRVItemClickListener(v,position);
                    }
                });
            }
        }
        
        ......
```

第三步 在Activity中设置监听回调

```
  myAdapter.setOnRVItemClickListener(new OnRVItemClickListener() {
            @Override
            public void onRVItemClickListener(View itemView, int pos) {
                Toast.makeText(RecycleActivity.this, "点击了" + pos, Toast.LENGTH_SHORT).show();
            }
        });
```

要设置ItemLongClick或者是item里面单一View 的Click 方法也同理。


####4.2 RecycleView添加分割线

RecycleView 没有像ListView一样可以在xml中设置分割线，要为他设置分割线需要调用addItemDecoration(ItemDecoration decor)方法，这里可以去看下鸿阳大神写的方法。
http://blog.csdn.net/lmj623565791/article/details/45059587

```
package com.webber.webberdemo;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;

/**
 * RecycleView分割线
 */
public class DividerItemDecoration extends RecyclerView.ItemDecoration {

    private static final int[] ATTRS = new int[]{
            android.R.attr.listDivider
    };

    public static final int HORIZONTAL_LIST = LinearLayoutManager.HORIZONTAL;

    public static final int VERTICAL_LIST = LinearLayoutManager.VERTICAL;

    private Drawable mDivider;

    private int mOrientation;

    public DividerItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
        setOrientation(orientation);
    }

    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL_LIST && orientation != VERTICAL_LIST) {
            throw new IllegalArgumentException("invalid orientation");
        }
        mOrientation = orientation;
    }

    @Override
    public void onDraw(Canvas c, RecyclerView parent) {

        if (mOrientation == VERTICAL_LIST) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }

    }


    public void drawVertical(Canvas c, RecyclerView parent) {
        final int left = parent.getPaddingLeft();
        final int right = parent.getWidth() - parent.getPaddingRight();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            android.support.v7.widget.RecyclerView v = new android.support.v7.widget.RecyclerView(parent.getContext());
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin;
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    public void drawHorizontal(Canvas c, RecyclerView parent) {
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();

        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }

    @Override
    public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
        if (mOrientation == VERTICAL_LIST) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
}

```

####4.3 RecycleView切换布局模式

RecycleView 使用给的是LayoutManager 来进行布局管理的，并能够通过方法来设置这个属性，也就意味着我们可以自定义他的布局模式。官方给了三种比较常见的布局管理器。

 1. LinearLayoutManager  线性布局
 
 来看下他的效果，借用下hongyang大神的图：
 ![](http://img.blog.csdn.net/20161110170651090)
 
 2. GridLayoutManager 网格布局
 
 ![](http://img.blog.csdn.net/20150415150125431)

 3. StaggeredGridLayoutManager 流式布局
 
  ![](http://img.blog.csdn.net/20150415194114149)

来看下他的效果：

 切换不同的布局只需要setLayoutManager() 传入布局即可，但是切换过后的条目位置会发生改变，很明显的道理，以前是第10个位置的线性布局，切换为2列的网格布局，同样一屏显示的item数会变化，就会导致位置发生变化，这样体验不好，因此我们在改变布局的时候需要先找到当前第一个可见条目的position，并在setLayoutManager 之后滑动到这个位置。代码如下：

```
  public void switchLayoutManager(LayoutManager layoutManager) {
        int firstVisiblePosition = getFirstVisiblePosition();
        setLayoutManager(layoutManager);
        getLayoutManager().scrollToPosition(firstVisiblePosition);
    }

    /**
     * 获取第一条可见item在RecycleView中所处的位置
     *
     * @return
     */
    public int getFirstVisiblePosition() {
        int position;
        if (getLayoutManager() instanceof LinearLayoutManager) {
            position = ((LinearLayoutManager) getLayoutManager()).findFirstVisibleItemPosition();
        } else if (getLayoutManager() instanceof GridLayoutManager) {
            position = ((GridLayoutManager) getLayoutManager()).findFirstVisibleItemPosition();
        } else if (getLayoutManager() instanceof StaggeredGridLayoutManager) {
            StaggeredGridLayoutManager layoutManager = (StaggeredGridLayoutManager) getLayoutManager();
            int[] lastPositions = layoutManager.findFirstVisibleItemPositions(new int[layoutManager.getSpanCount()]);
            position = getMinPositions(lastPositions);
        } else {
            position = 0;
        }
        return position;
    }

```

####4.4 RecycleView数据源操作

RecyeleView 对数据源的操作可能是绝大多数人选择使用他的原因。单从他支持单条操作来看就太爽了，你不用在notifyDataSetChange了，可以只针对单个position的数据进行操作。

1. 常见的操作数据源方法
