#RecyclerView粘性标签库
一个强大的粘性标签库，实现思路来源于「[pinned-section-item-decoration](https://github.com/takahr/pinned-section-item-decoration)」,感觉有用的话star个呗（＾∀＾）
## 功能
- 大粘性标签支持垂直方向的线性、网格、瀑布流布局管理器
- 小粘性标签支持垂直方向的线性和网格一行只有一列网格布局管理器
- 支持标签的单击、双击和长按事件
- 支持标签内部子控件的单击、双击和长按事件
- 可以绘制线性、网格、瀑布流布局的分隔线，支持自定义分割线样式(PS:垂直瀑布流布局需要Item高度固定，不能随机变化导致Item位置切换，可参考「[MainActivity 99-115行](https://github.com/oubowu/PinnedSectionItemDecoration/blob/master/app%2Fsrc%2Fmain%2Fjava%2Fcom%2Foushangfeng%2Fpinneddemo%2FMainActivity.java)」)

## 效果图
- 不带分隔线<br>
![大标签线性布局](/pic/big_header_linearlayout.gif) 
![大标签网格布局](/pic/big_header_gridlayout.gif) 
![大标签瀑布流布局](/pic/big_header_staggeredgridlayout.gif) 
![小标签线性布局](/pic/small_header_linearlayout.gif) 
![股市Demo](/pic/stock_demo.gif) 
- 带分隔线<br>
![大标签线性布局带分隔线](/pic/big_header_linear_divider.png) 
![大标签网格布局带分隔线](/pic/big_header_grid_divider.png) 
![大标签瀑布流布局带分隔线](/pic/big_header_staggeredgrid_divider.png) 
![小标签线性布局带分隔线](/pic/small_header_linear_divider.png) 

## 它能做什么？

首先在dependencies添加
```groovy
compile 'com.oushangfeng:PinnedSectionItemDecoration:1.1.0'
```

RecyclerView的Adapter需要继承PinnedHeaderNotifyer接口，重写方法告诉ItemDecoration哪种类型是粘性标签类型和某个位置粘性标签的信息(用于点击标签事件)「[供参考的RecyclerAdapter](https://github.com/oubowu/PinnedSectionItemDecoration/blob/master/app%2Fsrc%2Fmain%2Fjava%2Fcom%2Foushangfeng%2Fpinneddemo%2Fadapter%2FRecyclerAdapter.java)」
```
    @Override
    public boolean isPinnedHeaderType(int viewType) {
        // TYPE_SECTION代表是粘性标签类型
        return viewType == TYPE_SECTION;
    }
    
    @Override
    public T getPinnedHeaderInfo(int position) {
        return mData == null ? null : mData.get(position).getData();
    }
    
    
```
Adapter记得要实现对网格布局和瀑布流布局的标签占满一行的处理
```
    @Override
    public void onAttachedToRecyclerView(RecyclerView recyclerView) {
        // 如果是网格布局，这里处理标签的布局占满一行
        final RecyclerView.LayoutManager layoutManager = recyclerView.getLayoutManager();
        if (layoutManager instanceof GridLayoutManager) {
            final GridLayoutManager gridLayoutManager = (GridLayoutManager) layoutManager;
            final GridLayoutManager.SpanSizeLookup oldSizeLookup = gridLayoutManager.getSpanSizeLookup();
            gridLayoutManager.setSpanSizeLookup(new GridLayoutManager.SpanSizeLookup() {
                @Override
                public int getSpanSize(int position) {
                    if (getItemViewType(position) == TYPE_SECTION) {
                        return gridLayoutManager.getSpanCount();
                    }
                    if (oldSizeLookup != null) {
                        return oldSizeLookup.getSpanSize(position);
                    }
                    return 1;
                }
            });
        }
    }

    @Override
    public void onViewAttachedToWindow(RecyclerViewHolder holder) {
        // 如果是瀑布流布局，这里处理标签的布局占满一行
        final ViewGroup.LayoutParams lp = holder.itemView.getLayoutParams();
        if (lp instanceof StaggeredGridLayoutManager.LayoutParams) {
            final StaggeredGridLayoutManager.LayoutParams slp = (StaggeredGridLayoutManager.LayoutParams) lp;
            slp.setFullSpan(getItemViewType(holder.getLayoutPosition()) == TYPE_SECTION);
        }
    }
```

实现大粘性标签RecyclerView只需要添加一个PinnedHeaderItemDecoration，由于参数太多，现在只支持使用创建者模式创建，注意大标签所在的最外层布局不能设置marginTop，暂时没想到方法解决
往上滚动遮不住真正的标签「[供参考的StockActivity](https://github.com/oubowu/PinnedSectionItemDecoration/blob/master/app%2Fsrc%2Fmain%2Fjava%2Fcom%2Foushangfeng%2Fpinneddemo%2FStockActivity.java)」
``` 

     final OnHeaderClickAdapter<StockEntity.StockInfo> clickAdapter = new OnHeaderClickAdapter<StockEntity.StockInfo>() {

         @Override
         public void onHeaderClick(int id, int position, StockEntity.StockInfo data) {
             switch (id) {
                 case OnItemTouchListener.HEADER_ID:
                     Toast.makeText(StockActivity.this, "点击了标签: " + mAdapter.getData().get(position).getPinnedHeaderName(), Toast.LENGTH_SHORT).show();
                     break;
                 case R.id.iv_more:
                     Toast.makeText(StockActivity.this, "点击了标签的更多按钮", Toast.LENGTH_SHORT).show();
                     break;
             }
         }

     };

     mRecyclerView.addItemDecoration(
             new PinnedHeaderItemDecoration.Builder<StockEntity.StockInfo>()
             // 设置分隔线资源ID
             .setDividerId(R.drawable.divider)
             // 开启绘制分隔线，默认关闭
             .enableDivider(true)
             // 通过传入包括标签和其内部的子控件的ID设置其对应的点击事件
             .setClickIds(R.id.iv_more)
             // 开启标签点击事件(不包括标签里面的子控件)，默认开启
             .disableHeaderClick(false)
             // 设置标签和其内部的子控件的监听，若设置点击监听不为null，并且开启标签的点击监听，那么标签的点击回调返回的id为ItemTouchListener.HEADER_ID
             .setHeaderClickListener(clickAdapter)
             .create());
    
```
![大标签布局](/pic/big_pinned_header.png) 

实现小粘性标签稍微复杂点，比如这个是数据的布局A
```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
              xmlns:tools="http://schemas.android.com/tools"
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              android:layout_marginBottom="2dp"
              android:layout_marginLeft="2dp"
              android:layout_marginRight="2dp"
              android:background="#70E593">

    <ImageView
        android:id="@+id/iv_animal"
        android:layout_width="match_parent"
        android:layout_height="120dp"
        tools:src="@mipmap/panda0"/>

    <TextView
        tools:text="1"
        android:id="@+id/tv_pos"
        android:textColor="#000000"
        android:textSize="18dp"
        android:padding="8dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

</FrameLayout>
```
![布局A](/pic/item-data.png) 

这个是带有小标签的布局B
```
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             xmlns:tools="http://schemas.android.com/tools"
             android:layout_width="match_parent"
             android:layout_height="wrap_content"
             android:paddingBottom="2dp"
             android:paddingLeft="2dp"
             android:paddingRight="2dp"
             android:paddingTop="2dp">

    <ImageView
        android:id="@+id/iv_animal"
        android:layout_width="match_parent"
        android:layout_height="120dp"
        android:background="#70E593"
        tools:src="@mipmap/panda0"/>

    <TextView
        android:id="@+id/tv_small_pinned_header"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#5A5A5A"
        android:padding="8dp"
        android:textColor="#ffffff"
        android:textSize="18dp"
        tools:text="熊猫"/>

</FrameLayout>
```
![布局B](/pic/small_pinned_header.png) 

布局B就相当于在原来A的基础上放上个小标签，然后实现小粘性标签RecyclerView只需要添加一个SmallPinnedHeaderItemDecoration，只支持使用创建者模式创建，注意标签不能设置marginTop，
因为往上滚动遮不住真正的标签「[供参考的SecondActivity](https://github.com/oubowu/PinnedSectionItemDecoration/blob/master/app%2Fsrc%2Fmain%2Fjava%2Fcom%2Foushangfeng%2Fpinneddemo%2FSecondActivity.java)」
```
     OnHeaderClickAdapter<String> headerClickAdapter = new OnHeaderClickAdapter<String>() {

         @Override
         public void onHeaderClick(int id, int position, String data) {
             if (id == R.id.tv_small_pinned_header) {
                 Toast.makeText(SecondActivity.this, "点击了标签: " + data, Toast.LENGTH_SHORT).show();
             }
         }
     };
     mRecyclerView.addItemDecoration(
             // 构造方法需要传入小标签的ID
             new SmallPinnedHeaderItemDecoration.Builder<String>(R.id.tv_small_pinned_header)
             // 开启绘制分隔线，默认关闭
             .enableDivider(true)
             // 设置分隔线资源ID
             .setDividerId(R.drawable.divider)
             // 关闭标签点击事件(不包括标签里面的子控件)
             .disableHeaderClick(true)
             // 通过传入包括标签和其内部的子控件的ID设置其对应的点击事件
             // 这里我虽然关闭了标签点击事件，但是又传入了标签的ID，所以点击事件仍旧会发生
             .setClickIds(R.id.tv_small_pinned_header)
             // 设置标签和其内部的子控件的监听，若设置点击监听不为null，并且开启标签的点击监听，那么标签的点击回调返回的id为ItemTouchListener.HEADER_ID
             .setHeaderClickListener(headerClickAdapter)
             .create());
    
```

## 后续
- 解决不能设置marginTop的问题
- 解决设置marginBottom位置不对的问题

#### License
```
Copyright 2016 oubowu

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```




