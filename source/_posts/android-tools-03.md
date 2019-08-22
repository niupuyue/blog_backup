---
title: android奇技淫巧 03 RecyclerView分组悬浮列表
date: 2019-06-13 22:40:20
tags:
  - android
---

列表展示是在开发中经常使用到的功能，通常通过ListView或RecyclerView控件来实现。但是在列表展示中可能会碰到这样的需求，要求对列表进行分组展示，魅族都有标题itemview和内容itemview两部分组成。下面的是效果图：
<!--more-->
LinearLayoutManager:
![LinearLayoutManager]()

GridLayoutManager:
![GridLayoutManager]()

具体实现

其实我们都知道在RecyclerView中有一个叫做RecyclerView.ItemDecoration的对象。这个对象一般是表示对RecyclerView的item的修饰。通过这个对象我们可以给每一个item添加修饰样式，比如padding，或者分割线等。这里我们其实就是通过RecyclerView.ItemDecoration来实现悬浮样式的。

首先我们来看一下在RecyclerView.ItemDecoration中的几个函数
```
/**
* 可以通过重写这个函数给RecyclerView绘制任意合适的decorations(装饰)
* 会在RecyclerView item绘制之前绘制。可以认为是绘制在RecyclerView的下面
* 会在RecyclerView类的onDraw()里面调用
*/
public void onDraw(Canvas c, RecyclerView parent, RecyclerView.State state) {
onDraw(c, parent);
}

/**
* deprecated掉的函数我们不管，忽视掉，不建议使用了
*/
@Deprecated
public void onDraw(Canvas c, RecyclerView parent) {
}

/**
* 可以通过重写这个函数给RecyclerView绘制任意合适的decorations(装饰)
* 会在RecyclerView item绘制之后绘制。可以认为是绘制在RecyclerView的上面(在上面在盖一层)
* 会在RecyclerView类的super.draw()之后调用,
*/
public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
onDrawOver(c, parent);
}

/**
* deprecated掉的函数不建议使用了，忽视掉
*/
@Deprecated
public void onDrawOver(Canvas c, RecyclerView parent) {
}


/**
* deprecated掉的函数不建议使用了，忽视掉
*/
@Deprecated
public void getItemOffsets(Rect outRect, int itemPosition, RecyclerView parent) {
outRect.set(0, 0, 0, 0);
}

/**
* 给RecyclerView　item对应的每个view增加一些offsets(你可以这么认为item对应的view外面还有一层布局，给这个布局增加padding)
*/
public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
getItemOffsets(outRect, ((RecyclerView.LayoutParams) view.getLayoutParams()).getViewLayoutPosition(), parent);
}
```
在这里我们发现ItemDecoration为我们提供了三个方法，分别是onDraw(),onDrawOver(),getItemOffsets()，这几个对象从字面上我们基本上也能猜出个大概。getItemOffsets()函数是在每一个子View测量的时候调用的，用来设定每个子View的offset(间距).onDraw()方法是在RecyclerView中的onDraw方法是会调用。onDrawOver()方法会在RecyclerView中的onDraw方法中调用。其中onDraw()方法和onDrawOver()都是在RecyclerView中的onDraw方法中调用，我们可以认为，onDraw是在RecyclerView的View绘制时调用，onDrawOver方法是在RecyclerView的内容绘制完成之后调用。相当于绘制RecyclerView的上一层视图。

通过对 RecyclerView.ItemDecoration 类的简单分析，再结合我们分组固定标题 View 的需求，我们是要把每个分组的标题 View 固定在顶部，恩，那肯定是在要绘制在RecyclerView层之上的吧，和RecyclerView.ItemDecoration里面的onDrawOver()函数正好对应上了

首先，既然有些标题是要固定的，那咱们一定要明确的知道哪些position位置对应的view是标题吧，只能通过adapter做文章了，所有我们就有了一个基础的PinnedHeaderAdapter，代码如下

```
public abstract class PinnedHeaderAdapter<VH extends RecyclerView.ViewHolder> extends RecyclerView.Adapter<VH> {

/**
* 判断该position对应的位置是要固定
*
* @param position adapter position
* @return true or false
*/
public abstract boolean isPinnedPosition(int position);

}
```

接下来，RecyclerView.ItemDecoration里面的onDrawOver()函数里面我们做好三件事情就好了:第一，找到当前界面要一直固定在顶部的 View、第二，把找到固定在顶部的 View 画在 RecyclerView 的顶部、第三，当将要到达顶部的标题 View 和已经画在顶部的 View 相遇的时候顶部 view 上移的问题。这三个问题实现起来也不复杂，所以这里我们就直接贴代码了，毕竟代码才是王道吗

```
/**
* 把要固定的View绘制在上层
*/
@Override
public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
super.onDrawOver(c, parent, state);
//确保是PinnedHeaderAdapter的adapter,确保有View
if (parent.getAdapter() instanceof PinnedHeaderAdapter && parent.getChildCount() > 0) {
PinnedHeaderAdapter adapter = (PinnedHeaderAdapter) parent.getAdapter();
//找到要固定的pin view
View firstView = parent.getChildAt(0);
int firstAdapterPosition = parent.getChildAdapterPosition(firstView);
int pinnedHeaderPosition = getPinnedHeaderViewPosition(firstAdapterPosition, adapter);
if (pinnedHeaderPosition != -1) {
RecyclerView.ViewHolder pinnedHeaderViewHolder = adapter.onCreateViewHolder(parent, adapter.getItemViewType(pinnedHeaderPosition));
adapter.onBindViewHolder(pinnedHeaderViewHolder, pinnedHeaderPosition);
//要固定的view
View pinnedHeaderView = pinnedHeaderViewHolder.itemView;
ensurePinnedHeaderViewLayout(pinnedHeaderView, parent);
int sectionPinOffset = 0;
for (int index = 0; index < parent.getChildCount(); index++) {
if (adapter.isPinnedPosition(parent.getChildAdapterPosition(parent.getChildAt(index)))) {
View sectionView = parent.getChildAt(index);
int sectionTop = sectionView.getTop();
int pinViewHeight = pinnedHeaderView.getHeight();
if (sectionTop < pinViewHeight && sectionTop > 0) {
sectionPinOffset = sectionTop - pinViewHeight;
}
}
}
int saveCount = c.save();
c.translate(0, sectionPinOffset);
c.clipRect(0, 0, parent.getWidth(), pinnedHeaderView.getMeasuredHeight());
pinnedHeaderView.draw(c);
c.restoreToCount(saveCount);
}

}
}
```

