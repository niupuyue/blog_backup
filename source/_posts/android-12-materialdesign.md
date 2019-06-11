---
title: 重拾android路(十二) material design
date: 2016-07-20 23:29:10
tags:
 - Android
 - material design
---
在Google I/O 2015大会中，Google为Android开发者介绍了Design Support Library。这个library可以让开发者很容易地实现更多Material Design概念到他们的应用中，因为很多关键元素是不可用的在原来的框架外。首先就是很易于使用，Design Support Library向下兼容到API 7。Design Support Library可以引入到你的Android工程中通过导入Gradle依赖.
而在Google官网中的<a href='https://developer.android.com/reference/android/support/design/widget/package-summary.html'>android.support.design.widget</a>中，详细描述了这个几个组件(版本以24.1.0为准)
注释

<!--more-->

| 注释名称 | 原文描述 |
|---|---|
| CoordinatorLayout.DefaultBehavior	| Defines the default CoordinatorLayout.Behavior of a View class. |

类

| 组件名称 | 原文描述 |
|--------- |:----------------------- :|
| AppBarLayout | AppBarLayout is a vertical LinearLayout which implements many of the features of material designs app bar concept, namely scrolling gestures. |
| AppBarLayout.Behavior  | The default AppBarLayout.Behavior for AppBarLayout. |
| BottomNavigationView | Represents a standard bottom navigation bar for application. |
| BottomSheetBehavior<V extends View> | An interaction behavior plugin for a child view of CoordinatorLayout to make it work as a bottom sheet. |
| BottomSheetDialog | Base class for Dialogs styled as a bottom sheet. |
| CollapsingToolbarLayout | CollapsingToolbarLayout is a wrapper for Toolbar which implements a collapsing app bar. |
| CoordinatorLayout | CoordinatorLayout is a super-powered FrameLayout. |
| CoordinatorLayout.Behavior<V extends View> | Interaction behavior plugin for child views of CoordinatorLayout. |
| CoordinatorLayout.LayoutParams | Parameters describing the desired layout for a child of a CoordinatorLayout. |
| FloatingActionButton | Floating action buttons are used for a special type of promoted action. |
| FloatingActionButton.Behavior | Behavior designed for use with FloatingActionButton instances. |
| NavigationView | Represents a standard navigation menu for application. |
| Snackbar | Snackbars provide lightweight feedback about an operation. |
| TabLayout | TabLayout provides a horizontal layout to display tabs. |
| TabLayout.Tab	| A tab in this layout. |
| TabLayout.TabLayoutOnPageChangeListener | A ViewPager.OnPageChangeListener class which contains the necessary calls back to the provided TabLayout so that the tab position is kept in sync. |
| TabLayout.ViewPagerOnTabSelectedListener | A TabLayout.OnTabSelectedListener class which contains the necessary calls back to the provided ViewPager so that the tab position is kept in sync.|
| TextInputEditText | 	A special sub-class of EditText designed for use as a child of TextInputLayout |
| TextInputLayout | Layout which wraps an EditText (or descendant) to show a floating label when the hint is hidden due to the user inputting text. |

如果需要使用这些新引入的组件样式，需要加入新的依赖

```
compile 'com.android.support:design:22.2.0'
```

# Material Text Input
EditText自从最开始就已经在Android中了，并且使用很简单，他们一直没有什么改变。使用Design Support Library，Google已经介绍了新的包含容器叫作TextInputLayout。这个新的view添加功能到标准的EditText上，例如支持让你的用户界面弹出错误消息和动画提示
```
<android.support.design.widget.TextInputLayout
    android:id="@+id/textinput"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <EditText
        android:id="@+id/edittext"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="What is your name?" />

</android.support.design.widget.TextInputLayout>
```
# SnackBar
SnackBar通过在屏幕底部展示简洁的信息，为一个操作提供了一个轻量级的反馈，并且在Snackbar中还可以包含一个操作，在同一时间内，仅且只能显示一个 Snackbar，它的显示依赖于UI，不像Toast那样可以脱离应用显示。它的用法和Toast很相似，唯一不同的就是它的第一个参数不是传入Context而是传入它所依附的父视图，但是他比Toast更强大。
示例：
```
Snackbar.make(rootContainer,"提示信息",Snackbar.LENGTH_SHORT).show()
```
效果
![material design SnackBar效果](/assets/material/materialdesign01.png)
SnackBar的显示效果和Toas的样式基本相同.不同之处在于，我们可以为当前的SnackBar提供一个点击之后需要执行的Action。
`
Snackbar.make(mDrawerLayout, "SnackbarClicked", Snackbar.LENGTH_SHORT).setAction("Action", new View.OnClickListener() {
@Override
public void onClick(View v) {
Toast.makeText(MainActivity.this, "I'm a Toast", Toast.LENGTH_SHORT).show();
}
}).setActionTextColor(Color.RED).show();
`
这是我们为SnackBar设置了一个Action，并且设置了颜色是红色。并且为他设置了点击事件。
# FloatingActionButton
FloatingActionButton从名字可以看出它是一个浮动的按钮，它是一个带有阴影的圆形按钮，可以通过fabSize来改变其大小，主要负责界面的基本操作，这个按钮总体来说还是比较简单的。
1. 默认FloatingActionButton 的背景色是应用主题的 colorAccent（其实MD中的控件主题默认基本都是应用的这个主题），可以通过app:backgroundTint 属性或者setBackgroundTintList (ColorStateList tint)方法去改变背景颜色。
2. 上面提到 Floating action button 的大小尺寸，可以用过**app:fabSize **属性设置（normal or mini）
3. app:rippleColor 表示设置样式(如水波纹)
4. app:borderWidth: 设置边框宽度
5. app:elevation设置普通状态阴影的深度（默认是 6dp）
6. app:pressedTranslationZ设置点击状态的阴影深度（默认是 12dp）
7. android:src 属性改变 drawable
```
<android.support.design.widget.FloatingActionButton
android:id="@+id/fab_search"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
android:layout_gravity="bottom|end"
android:layout_margin="16dp"
android:src="@android:drawable/ic_dialog_email"
app:borderWidth="2dp"
app:fabSize="normal"
app:rippleColor="#ff0000" />
```
# coordinatorLayout
是一个增强版的frameLayout，他可以协调子View之间的交互，从而达到控制手势效果
效果如下图所示：
![coordinatorLayout的使用](/assets/material/simple_coordinator.gif)
在这个例子中我们可以看到View之间是如何相互配合的，View会根据其他View的变动做相应的变化。

下面是CoordinatorLayout的简单使用例子
```
<?xml version="1.0" encoding="utf-8"?>
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/background_light"
    android:fitsSystemWindows="true"
    >
    <android.support.design.widget.AppBarLayout
        android:id="@+id/main.appbar"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        android:fitsSystemWindows="true"
        >
        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/main.collapsing"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginStart="48dp"
            app:expandedTitleMarginEnd="64dp"
            >
            <ImageView
                android:id="@+id/main.backdrop"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                android:fitsSystemWindows="true"
                android:src="@drawable/material_flat"
                app:layout_collapseMode="parallax"
                />
            <android.support.v7.widget.Toolbar
                android:id="@+id/main.toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                app:layout_collapseMode="pin"
                />
        </android.support.design.widget.CollapsingToolbarLayout>
    </android.support.design.widget.AppBarLayout>
    <android.support.v4.widget.NestedScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textSize="20sp"
            android:lineSpacingExtra="8dp"
            android:text="@string/lorem"
            android:padding="@dimen/activity_horizontal_margin"
            />
    </android.support.v4.widget.NestedScrollView>
    <android.support.design.widget.FloatingActionButton
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:layout_margin="@dimen/activity_horizontal_margin"
        android:src="@drawable/ic_comment_24dp"
        app:layout_anchor="@id/main.appbar"
        app:layout_anchorGravity="bottom|right|end"
        />
</android.support.design.widget.CoordinatorLayout>
```
我们看一下这个layout结构，CoordinatorLayout包含3个子控件：
AppbarLayout， scrolleable view 和 anchoredFloatingActionBar。
```
<CoordinatorLayout>
    <AppbarLayout/>
    <scrollableView/>
    <FloatingActionButton/>
</CoordinatorLayout>
```
## AppBarLayout
AppBarLayout是一个继承于LinearLayout的ViewGroup的实现，默认AppBarLayout的方向是垂直方向，可以管理内部组件在页面滚动时的行为
可能语言叙述不是那么的清晰，看一下官方提供的gif图片
![AppBarLayout的官方实现动图](/assets/material/gif01.gif)
AppBarLayout在这个例子中时蓝色的View，在其下放置了一个可以缩放的图片，其中包含一个Toolbar，
一个LinearLayout（包含标题和副标题），以及一个TabLayout。
我们可以通过设置layout_scrollFlags参数，来控制AppBarLayout中的控件行为。
在我们的这个例子中，大部分View的layout_scrollFlags都设置为scroll，如果没有设置的话，
当可滚动的View进行滚动时，这些没设置为scroll的View位置会保持不变；

layout_scrollFlags设置上snap值则可以避免进入动画中间状态（ mid-animation-states），
这意味着动画会一直持续到View完全显示或完全隐藏为止。

LinearLayout其中包含了一个标题和一个副标题，当用户向上移动时LinearLayout是一直显示的，直到移出屏幕（enterAlways）;

TabLayout会一直是可见的，因为我们没有在TabLayout上设置任何flag。

正如你所见，AppbarLayout的强大管理能力是通过在View上设置不同scroll flags实现的。
```
<AppBarLayout>
    <CollapsingToolbarLayout
        app:layout_scrollFlags="scroll|snap"
        />
    <Toolbar
        app:layout_scrollFlags="scroll|snap"
        />
    <LinearLayout
        android:id="+id/title_container"
        app:layout_scrollFlags="scroll|enterAlways"
        />
    <TabLayout /> <!-- no flags -->
</AppBarLayout>
```
下面AppBarLayout的flags：

| 取值 | 说明 |
|------|------|
|SCROLL_FLAG_ENTER_ALWAYS | 当任何向下滚动事件发生时, View都会移入 , 不管scrolling view 是否正在滚动。|
|SCROLL_FLAG_ENTER_ALWAYS_COLLAPSED | ‘enterAlways’的附加标识，它使得returning view恢复到指定的最小高度后才开始显示，然后再慢慢展开。|
|SCROLL_FLAG_EXIT_UNTIL_COLLAPSED | 但向上移出屏幕时，View会一直收缩到最小高度后，再移出屏幕。|
|SCROLL_FLAG_SCROLL | View 会根据滚动事件进行移动。|
|SCROLL_FLAG_SNAP | 但滚动结束时，如果View只有部分可见，它将会自动滑动到最近的边界（完全可见或完全隐藏）|

## CoordinatorLayout behavior
### SwipeDismissBehavior
深入design support library的代码，我们会发现一个新的类：SwipeDismissBehavior，使用这个Behavior，
我们可以很容易的使用CoordinatorLayout实现滑动删除功能:
![演示](/assets/material/coordinator03.gif)
```
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_swipe_behavior);
    mCardView = (CardView) findViewById(R.id.swype_card);
    final SwipeDismissBehavior<CardView> swipe
        = new SwipeDismissBehavior();
        swipe.setSwipeDirection(
            SwipeDismissBehavior.SWIPE_DIRECTION_ANY);
        swipe.setListener(
            new SwipeDismissBehavior.OnDismissListener() {
            @Override public void onDismiss(View view) {
                Toast.makeText(SwipeBehaviorExampleActivity.this,
                    "Card swiped !!", Toast.LENGTH_SHORT).show();
            }
            @Override
            public void onDragStateChanged(int state) {}
        });
        LayoutParams coordinatorParams =
            (LayoutParams) mCardView.getLayoutParams();
        coordinatorParams.setBehavior(swipe);
    }
```
### custom Behavior
除了上面的这种之外，我们还可以自定义。首先我们应该搞清楚两个核心child和dependency
![演示](/assets/material/coordinator04.png)
#### child 和 dependency
child 是指需要应用behavior的View ，dependency 担任触发behavior的角色，并与child进行互动。
在这个例子中， child 是ImageView， dependency 是Toolbar，如果Toolbar发生移动，ImageView也会做相应的移动。
![](assets/android_material_design/coordinator05.gif)
现在我们已经知道概念了，接着我们看看怎么实现，
第一步我们需要继承CoordinatorLayout.Behavior，T是指某一个View，
在我们的例子中是ImageView， 继承后，我们必须实现以下2个方法:
- layoutDependsOn
- onDependentViewChanged
layoutDependsOn方法在每次layout发生变化时都会调用，我们需要在dependency控件发生变化时返回True，在我们的例子中是用户在屏幕上滑动时（因为Toolbar发生了移动），然后我们需要让child做出相应的反应。
```
@Override
  public boolean layoutDependsOn(     
     CoordinatorLayout parent,
     CircleImageView, child,
     View dependency) {
     return dependency instanceof Toolbar;
 }
 ```
 一旦layoutDependsOn返回了True，第二个方法onDependentViewChanged就会被调用，
在这个方法里我们需要实现动画，转场等效果。
```
public boolean onDependentViewChanged(
      CoordinatorLayout parent,
      CircleImageView avatar,
      View dependency) {
      modifyAvatarDependingDependencyState(avatar, dependency);
   }
   private void modifyAvatarDependingDependencyState(
    CircleImageView avatar, View dependency) {
        //  avatar.setY(dependency.getY());
        //  avatar.setBlahBlat(dependency.blah / blah);
    }
```
整合之后的代码是：
```
public static class AvatarImageBehavior
   extends CoordinatorLayout.Behavior<CircleImageView> {
   @Override
   public boolean layoutDependsOn(
    CoordinatorLayout parent,
    CircleImageView, child,
    View dependency) {
    return dependency instanceof Toolbar;
  }
  public boolean onDependentViewChanged(   
    CoordinatorLayout parent,
    CircleImageView avatar,
    View dependency) {
      modifyAvatarDependingDependencyState(avatar, dependency);
   }
  private void modifyAvatarDependingDependencyState(
    CircleImageView avatar, View dependency) {
        //  avatar.setY(dependency.getY());
        //  avatar.setBlahBlah(dependency.blah / blah);
    }    
}
```
几个写好的小栗子🌰，供大家参考


# NavigationView
这个其实就是侧滑栏，之前有自己写过一个侧滑栏，但是功能比较单一，而且写了好多，感觉在实现回调等操作的时候，比较复杂。不过之后就是用了官方的NavigationView，确实比较好用
```
<android.support.v4.widget.DrawerLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:id="@+id/drawer_layout"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:fitsSystemWindows="true">

<!--内容区-->
<android.support.design.widget.CoordinatorLayout
    android:id="@+id/main_content"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include
        android:id="@+id/appbar"
        layout="@layout/toolbar" />

    <FrameLayout
        android:id="@+id/frame_content"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@+id/appbar"
        android:scrollbars="none"
        app:layout_behavior="@string/appbar_scrolling_view_behavior" />
</android.support.design.widget.CoordinatorLayout>

<!--左侧导航菜单-->
<android.support.design.widget.NavigationView
    android:id="@+id/navigation_view"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:layout_gravity="start"
    app:headerLayout="@layout/navigation_header"
    app:menu="@menu/drawer" />
</android.support.v4.widget.DrawerLayout>
```
可以看到这里我们是以DrawerLayout作为其父布局，对于DrawLayout他可以实现一种抽屉式的侧滑效果，这里不多做讲解,这里只简单说一点:DrawLayout中的第一个布局是内容布局，第二个是菜单布局。现在我们直接定位到NavigationView，我们看到这里有** app:headerLayout="@layout/navigation_header"、app:menu="@menu/drawer"**这两行代码，其中headerLayout是设置其头部的布局，这个布局我们可以随便写，就和写普通的布局文件一样的。对于menu就是菜单项的配置了，其配置文件如下
```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
tools:context=".MainActivity">
<group android:checkableBehavior="single">
<item
android:id="@+id/navigation_item_home"
android:icon="@drawable/iconfont_home"
android:title="首页" />
<item
android:id="@+id/navigation_item_blog"
android:icon="@drawable/iconfont_blog"
android:title="我的博客" />

    <item
        android:id="@+id/navigation_item_about"
        android:icon="@drawable/iconfont_about"
        android:title="关于" />
</group>
</menu>
```
就这么简单，我们要实现上述的效果就只要这些就足够了。但是如果我们想要对Item添加一个点击的事件怎么做呢？请看下面：
```
private void setNavigationViewItemClickListener() {
mNavigationView.setNavigationItemSelectedListener(new NavigationView.OnNavigationItemSelectedListener() {
@Override
public boolean onNavigationItemSelected(MenuItem item) {
switch (item.getItemId()) {
case R.id.navigation_item_home:
mToolbar.setTitle("首页");
switchFragment("MainFragment");
break;
case R.id.navigation_item_blog:
mToolbar.setTitle("我的博客");
switchFragment("BlogFragment");
break;
case R.id.navigation_item_about:
mToolbar.setTitle("关于");
switchFragment("AboutFragment");
break;
default:
break;
}
item.setChecked(true);
mDrawerLayout.closeDrawer(Gravity.LEFT);
return false;
}
});
}
```
上面的代码很清楚了，就是为NavigationView添加了一个OnNavigationItemSelectedListener的监听事件，然后我们就可以做我们想做的事了


# 参考资料
[android的](http://www.jcodecraeer.com/a/anzhuokaifa/developer/2015/0531/2958.html)
[掌握coordinatorLayout](https://appkfz.com/2015/11/12/mastering-coordinator/?hmsr=toutiao.io)
[名字很长不知道该怎么翻译😆](https://lab.getbase.com/introduction-to-coordinator-layout-on-android/)
