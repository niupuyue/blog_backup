---
title: android奇技淫巧 01 Coordinatorlayout+appbarLayout
date: 2019-06-12 22:40:20
tags:
  - android
---

现在好多的App中都会有折叠式布局，给用户提供了很好的体验，其实就是通过Coordinatorlayout + AppbarLayout + CollapsingToolbarLayout三个部分组成的。以前自己学习这个技巧的时候，都是拿来主义，能用就好，对于其中的具体内容并不是非常关心，因为毕竟大部分时间还是要出效果，至于原理，项目是不会在乎的。但是别人不在乎，我们不能不在乎，学到的就是自己的。
<!--more-->
这里我会把这三个部分一一拆分，逐一分析，尽量一篇文章搞定

# Toolbar
从android3.0之后出现了ActionBar，但是这个效果不理想，颜色难看，布局也无法定制，还不如自己写一个ActionBar的好，

使用方式：
1. 首先在Activity主题里面将默认Actionbar改为NoActionbar
```
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
```
2. 绑定toolbar，通过setSupportActionBar(toolbar)设置toolbar为标题栏
3. 设置常用属性：
```
toolbar.setNavigationIcon(int resId);
toolbar.setLogo(int resId);
toolbar.setTitle("");
toolbar.setSubtitle("");
toolbar.setOnMenuItemClickListener(Toolbar.OnMenuItemClickListener listener);
```
4. 引用菜单
```
@override
public boolean onCreateOptionsMenu(Menu menu){
    // 引入options菜单
    getMenuInflater().inflate(R.menu.menu,menu);
    return true;
}
```
5. 在menu文件夹中设置菜单
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/menu_01"
            android:title="菜单1"
            android:icon="@mipmap/icon_menu_01"
            android:showAsAction="collapseActionView" />
    <item android:id="@+id/menu_02"
            android:title="菜单2"
            android:icon="@mipmap/icon_menu_02"
            android:showAsAction="collapseActionView" />
</menu>
```
对于showAsAction的可选参数如下：
- ifRoom 会显示在item中，如果空间不足会将后面的item收起来，如果已经有了4个或者4个以上的item则会隐藏在溢出列表中
- never 永远不会显示，只会在藏出列表中显示，而且只显示标题，所以在定义item的时候，最好把标题都带上
- always 无论是否超出空间都会显示
- withText withText值示意Action bar要显示文本标题。Action bar会尽可能的显示这个标题，但是，如果图标有效并且受到Action bar空间的限制，文本标题有可能显示不全
- collapseActionView 声明了这个操作视窗应该被折叠到一个按钮中，当用户选择这个按钮时，这个操作视窗展开。否则，这个操作视窗在默认的情况下是可见的，并且即便在用于不适用的时候，也要占据操作栏的有效空间
或者我们可以直接在布局文件中添加Toolbar标签
```
<android.support.v7.widget.Toolbar
android:id="@+id/toolbar"
android:layout_width="match_parent"
android:layout_height="?attr/actionBarSize"
app:layout_collapseMode="pin"
app:popupTheme="@style/ThemeOverlay.AppCompat.Light" >

<TextView
android:id="@+id/tv1"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:text="返回"
android:textSize="13sp"
android:textColor="@android:color/white" />

<TextView
android:id="@+id/tv2"
android:layout_width="wrap_content"
android:layout_height="match_parent"
android:layout_gravity="right"
android:layout_centerHorizontal="true"
android:layout_marginRight="6dp"
android:gravity="center"
android:padding="4dp"
android:textColor="#fff"
android:textSize="14sp"
android:text="菜单"/>
</android.support.v7.widget.Toolbar>
```
# Coordinatorlayout
作用：协调子view的相互关系，比如位置、大小，就像有几个调皮孩子的爸爸，要管管孩子的行为
![Behavior](/assets/tips/tip_coordinator_01.jpg)
开Coordinatorlayout看，Behavior是CoordinatorLayout的一个泛型抽象内部类,所以给子view添加layout_behavior属性是来自于它
```
<android.support.design.widget.CoordinatorLayout xmlns:tools="http://schemas.android.com/tools"
xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:id="@+id/coordinatorLayout"
xmlns:app="http://schemas.android.com/apk/res-auto"
tools:context="com.example.md.mdview.CoordinatorLayoutActivity">

<View
android:id="@+id/view_girl"
android:layout_width="70dp"
android:layout_height="70dp"
android:layout_marginLeft="200dp"
android:background="@mipmap/make_music_voice_changer_female" />

<View
android:id="@+id/view_uncle"
android:layout_width="100dp"
android:layout_height="100dp"
android:background="@mipmap/make_music_voice_changer_uncle"
app:layout_behavior="com.paulniu.RunBehavior"/>
</android.support.design.widget.CoordinatorLayout>
```
这里提供了一个小例子，布局：两个子view，操作viewgirl，viewuncle也会相应跟着走，这就要写一个联动关系，用自定义Behavior实现
```
public class RunBehavior extends CoordinatorLayout.Behavior<View>{

public RunBehavior(Context context, AttributeSet attrs) {
super(context, attrs);
}

@Override
public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency) {
int top = dependency.getTop();
int left = dependency.getLeft();

ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) child.getLayoutParams();
params.topMargin = top - 400;
params.leftMargin = left;
child.setLayoutParams(params);
return true;
}

@Override
public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency) {
return true;
}
}
```
public boolean layoutDependsOn(CoordinatorLayout parent, View child, View dependency)  方法：
根据条件过滤判断返回值，返回true联动，返回flase不联动，即behavior不生效
public boolean onDependentViewChanged(CoordinatorLayout parent, View child, View dependency)
当 dependency这个哥哥发生变化时， 另一个child弟弟也要跟着去玩
一个view根据另一个view的变化而变化，  dependency被 child监听
功能是child的y值永远比dependency大400像素
```
app:layout_behavior="com.example.md.mdview.RunBehavior"
```
这里一定要写上带参数的构造方法，因为coordinatorlayout是根据反射（所以是包名.类名路径）获取这个behavior，是从这个构造方法获得对象的，否则会出错
```
@Override
public boolean onTouch(View v, MotionEvent event) {
switch (event.getAction()){
case MotionEvent.ACTION_DOWN:
params.leftMargin = (int) (event.getX() - viewGirl.getMeasuredWidth() / 2);
params.topMargin = (int) (event.getY() - viewGirl.getMeasuredHeight() / 2);
viewGirl.setLayoutParams(params);
break;
case MotionEvent.ACTION_MOVE:
params.leftMargin = (int) (event.getX() - viewGirl.getMeasuredWidth() / 2);
params.topMargin = (int) (event.getY() - viewGirl.getMeasuredHeight() / 2);
viewGirl.setLayoutParams(params);
break;
}
return true;
}
```
最后是在界面监听手指的位置，给viewGirl设置手指的位置，viewgril变化了，viewuncle也就随之变化了

好，在会了Coordinatorlayout的用法，最外层父布局有了，该添加两个子view了。这里里面分别加入AppbarLayout和NestedScrollView作子view，给NestedScrollView加上behavior,就可以让AppbarLayout跟随NestedScrollView的Behavior联动。Android已经自带了app:layout_behavior="@string/appbar_scrolling_view_behavior"，只要滚动发生，就会给自己的子view（if instance of Appbarlayout）添加滚动事件。不明白这俩控件紧接着看后面讲解。
当前布局变为:
```
<android.support.design.widget.CoordinatorLayout
android:layout_width="match_parent"
android:layout_height="match_parent"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:orientation="vertical"
xmlns:android="http://schemas.android.com/apk/res/android">

<android.support.design.widget.AppBarLayout
android:layout_width="match_parent"
android:layout_height="wrap_content">

<android.support.v7.widget.Toolbar
android:id="@+id/toolbar"
android:layout_width="match_parent"
android:layout_height="50dp"
android:background="#0e932e"
app:layout_collapseMode="pin"/>
</android.support.design.widget.CollapsingToolbarLayout>

</android.support.design.widget.AppBarLayout>

<android.support.v4.widget.NestedScrollView
android:layout_width="match_parent"
android:layout_height="wrap_content"
app:layout_behavior="@string/appbar_scrolling_view_behavior">

<TextView
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:textSize="20sp"
android:textColor="#000"
android:padding="10dp"
android:text=""/>
</android.support.v4.widget.NestedScrollView>

</android.support.design.widget.CoordinatorLayout>
```

# NestedScrollView
NestedScrolling机制能够让父View和子View在滚动式进行配合，其基本流程如下：
当子view开始滚动之前，可以通知父View，让其先于自己进行滚动；
子View自己进行滚动；子view滚动之后，还可以通知父view继续滚动。
而要实现这样的交互机制，首先父view要实现NestedScrollingParent接口，而子View需要实现NestedScrollingChild接口，在这套机制中子View是发起者，父view是接受回调并做出响应的
下面是几个关键的类和接口
```
/**
* NestedScrollView is just like {@link android.widget.ScrollView}, but it supports acting
* as both a nested scrolling parent and child on both new and old versions of Android.
* Nested scrolling is enabled by default.
*/
public class NestedScrollView extends FrameLayout implements NestedScrollingParent,
NestedScrollingChild, ScrollingView {
static final int ANIMATED_SCROLL_GAP = 250;

static final float MAX_SCROLL_FACTOR = 0.5f;

private static final String TAG = "NestedScrollView";

/**
* Interface definition for a callback to be invoked when the scroll
* X or Y positions of a view change.
*
* <p>This version of the interface works on all versions of Android, back to API v4.</p>
*
* @see #setOnScrollChangeListener(OnScrollChangeListener)
*/
public interface OnScrollChangeListener {
/**
* Called when the scroll position of a view changes.
*
* @param v The view whose scroll position has changed.
* @param scrollX Current horizontal scroll origin.
* @param scrollY Current vertical scroll origin.
* @param oldScrollX Previous horizontal scroll origin.
* @param oldScrollY Previous vertical scroll origin.
*/
void onScrollChange(NestedScrollView v, int scrollX, int scrollY,
int oldScrollX, int oldScrollY);
}

private long mLastScroll;

private final Rect mTempRect = new Rect();
private OverScroller mScroller;
private EdgeEffect mEdgeGlowTop;
private EdgeEffect mEdgeGlowBottom;
······
```
//主要接口
NestedScrollingChild
NestedScrollingParent
//帮助类
NestedScrollingChildHelper
NestedScrollingParentHelper

# AppbarLayout
继承自Linearlayout，且方向是vertical，它可以让你定制当某个可滚动View的滚动手势发生变化时，其内部的子View实现何种动作。
AppBarLayout子View的动作
内部的子View通过在布局中加app:layout_scrollFlags设置执行的动作
- ·scroll  :子view会跟随滚动事件一起滚动，相当于添加到scrollview头部
- ·enterAlways  :只要屏幕下滑，view就会立即拉下出来。
- ·snap ：这个属性让控件变得有弹性,如果控件下拉了75%的高度，就会自动展开，如果只有25%显示，就会反弹回去关闭。（去试试支付宝首页吧，就是加了弹性这个效果）
- ·exitUntilCollapsed  ：当scrollview滑到订部，再将子view折叠起来

可以给ViewPager设置行为，就不需要使用NestedScrollView的滑动，实现与AppBarLayout联动。
app:layout_behavior="@string/appbar_scrolling_view_behavior"
setExpande(boolean )  设置展开和关闭状态，默认有开关动画

# CollapsingToolbarLayout
CollapsingToolbarLayout作用是提供了一个可以折叠的Toolbar，它继承自FrameLayout。
CollapsingToolbarLayout属性   含义
app:title   设置标题
app:collapsedTitleGravity="center"  设置标题位置
app:contentScrim    设置折叠时toolbar的颜色，默认是colorPrimary的色值
app:statusBarScrim  设置折叠时状态栏的颜色 ，默认是colorPrimaryDark的色值
app:layout_collapseParallaxMultiplier   设置视差
app:layout_collapseMode="parallax"  视差模式，在折叠的时候会有个视差折叠的效果
app:layout_collapseMode="pin"   固定模式，在折叠的时候最后固定在顶端
使用示例：让图片折叠，让toolbar固定
```
<android.support.design.widget.AppBarLayout
android:layout_width="match_parent"
android:layout_height="wrap_content">

<android.support.design.widget.CollapsingToolbarLayout
android:layout_width="match_parent"
android:layout_height="wrap_content"
app:layout_scrollFlags="scroll|exitUntilCollapsed">

<ImageView
android:layout_width="match_parent"
android:layout_height="200dp"
android:background="@mipmap/bg"
app:layout_collapseMode="parallax"/>

<android.support.v7.widget.Toolbar
android:id="@+id/toolbar"
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:minHeight="50dp"
android:background="#000"
app:layout_collapseMode="pin"/>
</android.support.design.widget.CollapsingToolbarLayout>

</android.support.design.widget.AppBarLayout>
```
添加flags可以设置系统状态栏为透明，如果最顶上是背景这样用效果更佳
```
getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
setContentView(R.layout.activity_main);
```
实现toolbar渐变颜色：AppbarLayout提供了滑动偏移监听，偏移量除以appbar总高度可以得到当前滑动百分比。注意：这个verticalOffset是0或者负数，需要转绝对值
```
appbarLayout.addOnOffsetChangedListener(new AppBarLayout.OnOffsetChangedListener() {
@Override
public void onOffsetChanged(AppBarLayout appBarLayout, int verticalOffset) {
//verticalOffset始终为0以下的负数
float percent = (Math.abs(verticalOffset * 1.0f)/appBarLayout.getTotalScrollRange());
}
});
```
