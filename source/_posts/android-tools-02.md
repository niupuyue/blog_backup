---
title: android奇技淫巧 02 无限循环RecyclerView
date: 2019-06-12 22:44:20
tags:
  - android
---

项目中要实现横向列表的无限循环滚动，自然而然想到了RecyclerView，但我们常用的RecyclerView是不支持无限循环滚动的，所以就需要一些办法让它能够无限循环
<!--more-->
# 方案一
对adapter进行修改
网上大部分博客的解决方案都是这种方案，对Adapter做修改。具体如下

首先，让 Adapter 的 getItemCount() 方法返回 Integer.MAX_VALUE，使得position数据达到很大很大；

其次，在 onBindViewHolder() 方法里对position参数取余运算，拿到position对应的真实数据索引，然后对itemView绑定数据

最后，在初始化RecyclerView的时候，让其滑动到指定位置，如 Integer.MAX_VALUE/2，这样就不会滑动到边界了，如果用户一根筋，真的滑动到了边界位置，再加一个判断，如果当前索引是0，就重新动态调整到初始位置

这个方案是挺简单，但并不完美。一是对我们的数据和索引做了计算操作，二是如果滑动到边界，再动态调整到中间，会有一个不明显的卡顿操作，使得滑动不是很顺畅。所以，直接看方案二。

# 方案二
自定义LayoutManager，修改RecyclerView的布局方式
这个算得上是一劳永逸的解决方案了，也是我今天要详细介绍的方案。我们都知道，RecyclerView的数据绑定是通过Adapter来处理的，而排版方式以及View的回收控制等，则是通过LayoutManager来实现的，因此我们直接修改itemView的排版方式就可以实现我们的目标，让RecyclerView无限循环。
## 自定义LayoutManager
1. 创建自定义LayoutManager
首先，自定义 LooperLayoutManager 继承自 RecyclerView.LayoutManager，然后需要实现抽象方法 generateDefaultLayoutParams()，这个方法的作用是给 itemView 设置默认的LayoutParams，直接返回如下就行
```
public class LooperLayoutManager extends RecyclerView.LayoutManager {
        @Override
    public RecyclerView.LayoutParams generateDefaultLayoutParams() {
        return new RecyclerView.LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT,
                ViewGroup.LayoutParams.WRAP_CONTENT);
    }
}
```
2. 打开滚动开关
接着，对滚动方向做处理，重写canScrollHorizontally()方法，打开横向滚动开关。注意我们是实现横向无限循环滚动，所以实现此方法，如果要对垂直滚动做处理，则要实现canScrollVertically()方法
```
 @Override
    public boolean canScrollHorizontally() {
        return true;
    }
```
3. 对RecyclerView进行初始化布局
以上两部是基础工作，接下来，重写 onLayoutChildren() 方法，开始对itemView初始化布局
```
@Override
    public void onLayoutChildren(RecyclerView.Recycler recycler, RecyclerView.State state) {
        if (getItemCount() <= 0) {
            return;
        }
        //标注1.如果当前时准备状态，直接返回
        if (state.isPreLayout()) {
            return;
        }
        //标注2.将视图分离放入scrap缓存中，以准备重新对view进行排版
        detachAndScrapAttachedViews(recycler);

        int autualWidth = 0;
        for (int i = 0; i < getItemCount(); i++) {
            //标注3.初始化，将在屏幕内的view填充
            View itemView = recycler.getViewForPosition(i);
            addView(itemView);
            //标注4.测量itemView的宽高
            measureChildWithMargins(itemView, 0, 0);
            int width = getDecoratedMeasuredWidth(itemView);
            int height = getDecoratedMeasuredHeight(itemView);
            //标注5.根据itemView的宽高进行布局
            layoutDecorated(itemView, autualWidth, 0, autualWidth + width, height);

            autualWidth += width;
            //标注6.如果当前布局过的itemView的宽度总和大于RecyclerView的宽，则不再进行布局
            if (autualWidth > getWidth()) {
                break;
            }
        }
    }
```
onLayoutChildren() 方法顾名思义，就是对所有的 itemView 进行布局，一般会在初始化和调用 Adapter 的 notifyDataSetChanged() 方法时调用。代码思路已经注释的很清楚了，其中有几个方法需要简单提下：

标注2处 detachAndScrapAttachedViews(recycler) 方法会将所有的 itemView 从View树中全部detach，然后放入scrap缓存中。了解过RecyclerView的同学应该知道，RecyclerView是有一个二级缓存的，一级缓存是 scrap 缓存,二级缓存是 recycler 缓存，其中从View树上detach的View会放入scrap缓存里，调用removeView()删除的View会放入recycler缓存中。

标注3处 recycler.getViewForPosition(i)  方法会从缓存中拿到对应索引的 itemView，这个方法内部会先从 scrap 缓存中取 itemView，如果没有则从 recycler 缓存中取，如果还没有则调用 adapter 的 onCreateViewHolder() 去创建 itemView。
标注5处 layoutDecorated() 方法会对 itemView 进行布局排版，这里可以看出来，我们是根据宽依次往父容器的右边排下去，直到下一个 itemView的顶点位置超过了RecyclerView 的宽度
4. 对RecyclerView进行滚动和回收itemView处理
对RecyclerView的子item进行排版布局后，运行一下效果就会出现了，不过这时候我们滑动列表会发现滑动后变成空白了，所以就该对滑动操作进行处理了。
前面说过,我们打开了横向滚动的开关，所以对应的，我们要重写 scrollHorizontallyBy()方法进行横向滑动操作。
```
@Override
    public int scrollHorizontallyBy(int dx, RecyclerView.Recycler recycler, RecyclerView.State state) {
        //标注1.横向滑动的时候，对左右两边按顺序填充itemView
        int travl = fill(dx, recycler, state);
        if (travl == 0) {
            return 0;
        }

        //2.滑动
        offsetChildrenHorizontal(-travl);

        //3.回收已经不可见的itemView
        recyclerHideView(dx, recycler, state);
        return travl;
    }
```
可以看到，滑动逻辑很简单，总结为三步：
- 横向滑动的时候，对左右两边按顺序填充itemView
- 滑动itemView
- 回收已经不可见的itemView

首先第一步，滑动的时候调用自定义的 fill() 方法，对左右两边进行填充。还没忘了，我们是来实现循环滑动的，所以这一步尤其重要，先看代码
```
/**
     * 左右滑动的时候，填充
     */
    private int fill(int dx, RecyclerView.Recycler recycler, RecyclerView.State state) {
        if (dx > 0) {
            //标注1.向左滚动
            View lastView = getChildAt(getChildCount() - 1);
            if (lastView == null) {
                return 0;
            }
            int lastPos = getPosition(lastView);
            //标注2.可见的最后一个itemView完全滑进来了，需要补充新的
            if (lastView.getRight() < getWidth()) {
                View scrap = null;
                //标注3.判断可见的最后一个itemView的索引，
                // 如果是最后一个，则将下一个itemView设置为第一个，否则设置为当前索引的下一个
                if (lastPos == getItemCount() - 1) {
                    if (looperEnable) {
                        scrap = recycler.getViewForPosition(0);
                    } else {
                        dx = 0;
                    }
                } else {
                    scrap = recycler.getViewForPosition(lastPos + 1);
                }
                if (scrap == null) {
                    return dx;
                }
                //标注4.将新的itemViewadd进来并对其测量和布局
                addView(scrap);
                measureChildWithMargins(scrap, 0, 0);
                int width = getDecoratedMeasuredWidth(scrap);
                int height = getDecoratedMeasuredHeight(scrap);
                layoutDecorated(scrap,lastView.getRight(), 0,
                        lastView.getRight() + width, height);
                return dx;
            }
        } else {
            //向右滚动
            View firstView = getChildAt(0);
            if (firstView == null) {
                return 0;
            }
            int firstPos = getPosition(firstView);

            if (firstView.getLeft() >= 0) {
                View scrap = null;
                if (firstPos == 0) {
                    if (looperEnable) {
                        scrap = recycler.getViewForPosition(getItemCount() - 1);
                    } else {
                        dx = 0;
                    }
                } else {
                    scrap = recycler.getViewForPosition(firstPos - 1);
                }
                if (scrap == null) {
                    return 0;
                }
                addView(scrap, 0);
                measureChildWithMargins(scrap,0,0);
                int width = getDecoratedMeasuredWidth(scrap);
                int height = getDecoratedMeasuredHeight(scrap);
                layoutDecorated(scrap, firstView.getLeft() - width, 0,
                        firstView.getLeft(), height);
            }
        }
        return dx;
    }
```
首先分为两部分，往左填充或是往右填充，dx为将要滑动的距离，如果 dx > 0，则是往左边滑动，则需要判断右边的边界，如果最后一个itemView完全显示出来后，在右边填充一个新的itemView。

看标注3，往右边填充的时候需要检测当前最后一个可见itemView的索引，如果索引是最后一个，则需要新填充的itemView为第0个，这样就可以实现往左边滑动时候无限循环了。然后将需要新填充的itemView进行测量布局操作，将填充进去了。

同理，往右滑动的逻辑跟往左滑动相似,就不一一再阐述了。

第二步：填充完新的itemView后，就开始进行滑动了，这里直接调用 LayoutManager 的 offsetChildrenHorizontal() 方法滑动-travl 距离，travl 是通过fill方法计算出来的，通常情况下都为 dx，只有当滑动到最后一个itemView，并且循环滚动开关没有打开的时候才为0，也就是不滚动了。

```
 //2.滚动
        offsetChildrenHorizontal(travl * -1);
```
第三步：回收已经不可见的itemView。只有对不可见的itemView进行回收，才能做到回收利用，防止内存爆增。
```
/**
     * 回收界面不可见的view
     */
    private void recyclerHideView(int dx, RecyclerView.Recycler recycler, RecyclerView.State state) {
        for (int i = 0; i < getChildCount(); i++) {
            View view = getChildAt(i);
            if (view == null) {
                continue;
            }
            if (dx > 0) {
                //标注1.向左滚动，移除左边不在内容里的view
                if (view.getRight() < 0) {
                    removeAndRecycleView(view, recycler);
                    Log.d(TAG, "循环: 移除 一个view  childCount=" + getChildCount());
                }
            } else {
                //标注2.向右滚动，移除右边不在内容里的view
                if (view.getLeft() > getWidth()) {
                    removeAndRecycleView(view, recycler);
                    Log.d(TAG, "循环: 移除 一个view  childCount=" + getChildCount());
                }
            }
        }

    }
```
遍历所有添加进 RecyclerView 里的item，然后根据 itemView 的顶点位置进行判断，移除不可见的item。移除 itemView 调用 removeAndRecycleView(view, recycler) 方法，会对移除的item进行回收，然后存入 RecyclerView 的缓存里。

至此，一个可以实现左右无限循环的LayoutManager就实现了，调用方式跟通常我们用RrcyclerView没有任何区别，只需要给 RecyclerView 设置 LayoutManager 时指定我们的LayoutManager，如下
```
 recyclerView.setAdapter(new MyAdapter());
        LooperLayoutManager layoutManager = new LooperLayoutManager();
        layoutManager.setLooperEnable(true);
        recyclerView.setLayoutManager(layoutManager);
```

# 参考资料
[无限循环RecyclerView完美实现](https://juejin.im/post/5cfa198ff265da1b8c197c2f)