---
title: android奇技淫巧 12 Android布局优化 include+merge+viewStub
date: 2019-08-13 14:34:10
tags:
  - android
---

在编写Android布局时总会遇到这样那样的痛点，如下：
<!--more-->
1. 有些布局在很多页面都会用到，而且样式都一样，每次都要复制粘贴大量的代码
2. 在解决1中的问题之后，发现复用的布局外面总是要额外的再套上一层布局，要知道布局嵌套会影响性能
3. 有些布局只有用到的时候才会显示，但是又必须提前写好，虽然设置了invisible或者gone，但是多多少少会占有内存

要解决这些痛点，我们可以使用include，merge和ViewStub三个标签，先来看一下实现的效果
![布局优化实现效果](/assets/tools/tools-merge-01.png)

## include

当我们在一个主页面中使用include标签是，会表示当前的主布局包含标签中的布局，这样一来，就能很好的起到复用布局的效果，在那些常用的布局比如标题栏或者分割线等上面使用它可以减少大量代码
主要属性有两个最重要的

1. layout:必填属性，为我们需要插入当前主布局的不具名称，通过R.layout.xx的方式引入
2. id：当我们想通过include添加进来的布局设置id的时候，可以使用该属性，他可以重写插入主布局的布局id

### 常规使用
我们先创建一个ViewOptimizationActivity，然后再创建一个layout_include.xml布局文件，它的内容非常简单，就一个TextView

```
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:gravity="center_vertical"
 android:textSize="14sp"
 android:background="@android:color/holo_red_light"
 android:layout_height="40dp">
</TextView>
```
现在我们就用include标签，将其添加到ViewOptimizationActivity的布局中：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical"
 tools:context=".ViewOptimizationActivity">
 <!--include标签的使用-->
 <TextView
 android:textSize="18sp"
 android:text="1、include标签的使用"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content" />
 <include
 android:id="@+id/tv_include1"
 layout="@layout/layout_include"/>
</LinearLayout>
```
这样我们就将include试用完了，只需要知名包含布局的id即可。除此之外，我们还给这个include标签设置了一个id，为了验证它就是layout_include.xml的根布局TextView的id，我们可以在ViewOptiomizationActivity中初始化TextView，并给他设置文字
```
TextView tvInClude1 = findViewById(R.id.tv_include1);
tvInclude1 = setText("常规下的include布局")
```
这样只要能显示相应的代码，就表示我们设置的layout和id是成功的。不过你可能会对这个id属性有疑问：id我们直接在TextView中设置，为什么还要重写他呢？别忘了，我们的作用是复用，那么如果我们在一个主布局中添加了两个include标签，添加两个以上的相同布局，id相同则会造成冲突，所以重写他可以让我们更好的调用它和它里面的空间，还有一种情况，假如你的主布局是RelateLayout，这是为了设置相对位置，我们也需要设置不同的id

### 重写根布局的布局属性

除了id之外，我们还可以重写宽高，边距和可见性这些布局属性。但是我们一定要注意，单单重写android:layout_height或者android:layout_width是不行的，必须两个同时重写才能起作用，包括边距也是这样。如果我们想要给一个include进来的布局添加右边距，完整的写法如下：
```
<include
  andorid:layout_width="math_parent"
  android:layout_height="match_parent"
  android:layout_marginEnd="40dp"
  android:id="@+id/tv_include2"
  layout="@layout/layout_include" />
```

### 空间ID相同时处理

在"常规使用"中我们知道id的属性可以重写include布局的根布局id，但对于根布局里面的布局和空间是无能为力的。如果这是一个布局中在主布局中多次include布局，怎么才能区分里面的子控件呢？

我们首先创建一个layout_include2.xml的布局，他的根布局时FrameLayout，里面有一个TextView，他的id是tv_same:
```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:background="@android:color/holo_orange_light"
 android:layout_height="wrap_content">
 <TextView
 android:gravity="center_vertical"
 android:id="@+id/tv_same"
 android:layout_width="match_parent"
 android:layout_height="50dp" />
</FrameLayout>
```
在主布局中添加进去：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical"
 tools:context=".ViewOptimizationActivity">
 <!--include标签的使用-->
 ……
 <include layout="@layout/layout_include2"/>
 <include
 android:id="@+id/view_same"
 layout="@layout/layout_include2"/>
</LinearLayout>
```
为了区分，这里给第二个layout_include2设置了id，这就是我们要创建根布局的对象，然后再去初始化里面的控件
```
// 这是默认的第一个include的中的TextView
TextView tvSame = findViewById(R.id.tv_same);
// 这是id=view_same的include中的TextView
FrameLayout viewSame = findViewById(R.id.view_same);
TextView tvSame2 = viewSame.findViewById(R.id.view_same);
```

# merge
include标签虽然能够解决布局重用的问题，但是也会带来另一个问题：布局嵌套。因为把需要重用的布局放在一个自布局之后，就必须加上一个根布局，如果我们的主布局的根布局和我们需要include的根布局都一样，那么相当于在中间添加了一层多余的布局，这样会造成我们的布局嵌套越来越多。

使用merge标签要注意一点：必须是一个布局文件中的根节点。看起来和其他布局并没有什么两样，但是他的特别之处在于页面加载时他不会被绘制。如：它就像是布局或者控件的搬运工，把“货物”搬到主布局之后就会功成身退，不会占用任何空间，因此也就不会增加布局层级了。这正如它的名字一样，只起“合并”作用

### 常规使用

```
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:layout_height="wrap_content">
 <TextView
 android:id="@+id/tv_merge1"
 android:text="我是merge中的TextView1"
 android:background="@android:color/holo_green_light"
 android:gravity="center"
 android:layout_width="wrap_content"
 android:layout_height="40dp" />
 <TextView
 android:layout_toEndOf="@+id/tv_merge1"
 android:id="@+id/tv_merge2"
 android:text="我是merge中的TextView2"
 android:background="@android:color/holo_blue_light"
 android:gravity="center"
 android:layout_width="match_parent"
 android:layout_height="40dp" />
</merge>
```
这里我使用了一些相对布局的属性，原因后面你就知道了。我们接着在ViewOptimizationActivity的布局添加RelativeLayout，然后使用include标签将layout_merge.xml添加进去：
```
 <RelativeLayout
 android:layout_width="match_parent"
 android:layout_height="wrap_content">
 <include
 android:id="@+id/view_merge"
 layout="@layout/layout_merge"/>
 </RelativeLayout>
 ```

 ### merge标签对布局层级的影响

 在layout_merge.xml中，我们使用相对布局的属性android:layout_toEndOf将蓝色TextView设置到绿色TextView的右边，而layout_merge.xml的父布局是RelativeLayout，所以这个属性是起了作用，merge标签不会影响里面的空间，也不会增加布局层级

 ### merge的ID
 在学习include标签时我们知道，它的android:id属性可以重写被include的根布局id，但如果根节点是merge呢？前面说了merge并不会作为一个布局绘制出来，所以这里给它设置id是不起作用的。我们可以在它的父布局RelativeLayout中再加一个TextView，使用android:layout_below属性把设置到layout_merge下面:

 ```
<RelativeLayout
 android:layout_width="match_parent"
 android:layout_height="wrap_content">
 <include
 android:id="@+id/view_merge"
 layout="@layout/layout_merge"/>
 <TextView
 android:text="我不是merge中的布局"
 android:layout_below="@+id/view_merge"
 android:background="@android:color/holo_purple"
 android:gravity="center"
 android:layout_width="match_parent"
 android:layout_height="40dp"/>
 </RelativeLayout>
 ```
 运行之后你会发现新加的TextView会把merge布局盖住，没有像预期那样在其下方。如果把android:layout_below中的id改为layout_merge.xml中任一TextView的id（比如tv_merge1）

 # ViewStub

 在开发过程中我们会遇到这样的问题，页面中有些布局在初始化的时候，没有必要显示，但又不得不事先在布局文件中写好，虽然设置成了invisible或者gone，但在初始化的时候还是会加载，这样无疑会影响加载速度。针对这样的情况，Android为我们提供了一个利器--ViewStub。这是一个不可见，大小为0的视图，具有懒加载功能。它存在于视图层级中，到那时会在setVisibility()和inflate()方法调用才会填充视图，所以不会影响初始化加载速度，他有三个重要属性
 1. android:layout  ViewStub需要填充视图名称，为R.layout.xx的形式
 2. android:inflated    重写被填充的视图的父布局id

 与include标签不同，ViewStub的android:id属性是设置ViewStub本身的id，而不是重写布局的id，这一点不能搞错。另外ViewStub还提供了OnInflateListener接口用于监听布局是否已经加载

 ### 填充布局的正确方式

 这里我们创建一个layout_view_stub.xml，里面放置一个Switch开关

 ```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="match_parent"
 android:background="@android:color/holo_blue_dark"
 android:layout_height="100dp">
 <Switch
 android:id="@+id/sw"
 android:layout_gravity="center"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content" />
</FrameLayout>
 ```
然后在Activity的布局中修改如下：
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 xmlns:app="http://schemas.android.com/apk/res-auto"
 xmlns:tools="http://schemas.android.com/tools"
 android:layout_width="match_parent"
 android:layout_height="match_parent"
 android:orientation="vertical"
 tools:context=".ViewOptimizationActivity">
 <!--ViewStub标签的使用-->
 <TextView
 android:textSize="18sp"
 android:text="3、ViewStub标签的使用"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content" />
 <ViewStub
 android:id="@+id/view_stub"
 android:inflatedId="@+id/view_inflate"
 android:layout="@layout/layout_view_stub"
 android:layout_width="match_parent"
 android:layout_height="100dp" />
 <LinearLayout
 android:orientation="horizontal"
 android:layout_width="match_parent"
 android:layout_height="wrap_content">
 <Button
 android:text="显示"
 android:id="@+id/btn_show"
 android:layout_weight="1"
 android:layout_width="0dp"
 android:layout_height="wrap_content" />
 <Button
 android:text="隐藏"
 android:id="@+id/btn_hide"
 android:layout_weight="1"
 android:layout_width="0dp"
 android:layout_height="wrap_content" />
 <Button
 android:text="操作父布局控件"
 android:id="@+id/btn_control"
 android:layout_width="wrap_content"
 android:layout_height="wrap_content" />
 </LinearLayout>
</LinearLayout>
```

在ViewOptimizationActivity中监听ViewStub的填充事件
```
viewStub.setOnInflateListener(new ViewStub.OnInflateListener(){
  @Override
  public void onInflate(ViewStb viewStub,View view){
    Toast.makeText(ViewOptimizationActivity.this,"viewStub加载了",Toast.LENGTH_SHORT).show();
  }
})
```
然后通过按钮事件来填充和显示layout_view_stub:
```
@Override
 public void onClick(View view) {
 switch (view.getId()) {
 case R.id.btn_show:
 viewStub.inflate();
 break;
 case R.id.btn_hide:
 viewStub.setVisibility(View.GONE);
 break;
 default:
 break;
 }
 }
```
运行之后，点击“显示”按钮，layout_view_stub显示了，并弹出"ViewStub加载了"的Toast；点击“隐藏”按钮，布局又隐藏掉了，但是再点击一下“显示”按钮，页面居然却闪退了，查看日志，发现抛出了一个异常：
```
java.lang.IllegalStateException: ViewStub must have a non-null ViewGroup viewParent
```

我们打开ViewStub的源码，看看是在哪里抛出的异常
```
public View inflate() {
 final ViewParent viewParent = getParent();
 if (viewParent != null && viewParent instanceof ViewGroup) {
 if (mLayoutResource != 0) {
 final ViewGroup parent = (ViewGroup) viewParent;
 final View view = inflateViewNoAdd(parent);
 replaceSelfWithView(view, parent);
 mInflatedViewRef = new WeakReference<>(view);
 if (mInflateListener != null) {
 mInflateListener.onInflate(this, view);
 }
 return view;
 } else {
 throw new IllegalArgumentException("ViewStub must have a valid layoutResource");
 }
 } else {
 throw new IllegalStateException("ViewStub must have a non-null ViewGroup viewParent");
 }
 }
```
注意到if语句中有一个replaceSelfWithView()方法，听这名字就让人有一种不祥的预感了，点进去一看：
```
private void replaceSelfWithView(View view, ViewGroup parent) {
 final int index = parent.indexOfChild(this);
 parent.removeViewInLayout(this);
 final ViewGroup.LayoutParams layoutParams = getLayoutParams();
 if (layoutParams != null) {
 parent.addView(view, index, layoutParams);
 } else {
 parent.addView(view, index);
 }
 }
```
果然，ViewStub在这里调用了removeViewInLayout()方法把自己从布局移除了。到这里我们就明白了，ViewStub在填充布局成功之后就会自我销毁，再次调用inflate()方法就会抛出IllegalStateException异常了。此时如果想要再次显示布局，可以调用setVisibility()方法。

为了避免inflate()方法多次调用，我们可以采用如下三种方式

#### 捕获异常
我们可以通过捕获异常，同时调用setVisibility()方法显示布局
```
try {
 viewStub.inflate();
 } catch (IllegalStateException e) {
 Log.e("Tag",e.toString());
 view.setVisibility(View.VISIBLE);
 }
```

#### 通过监听ViewStub的填充事件
声明一个布尔值变量isViewStubShow,默认值是false，布局填充成功之后，在监听事件onInflate方法中将其设置为true
```
if (isViewStubShow){
 viewStub.setVisibility(View.VISIBLE);
 }else {
 viewStub.inflate();
 }
```

#### 直接调用setVisibility()方法
```
public void setVisibility(int visibility) {
 if (mInflatedViewRef != null) {
 View view = mInflatedViewRef.get();
 if (view != null) {
 view.setVisibility(visibility);
 } else {
 throw new IllegalStateException("setVisibility called on un-referenced view");
 }
 } else {
 super.setVisibility(visibility);
 if (visibility == VISIBLE || visibility == INVISIBLE) {
 inflate();
 }
 }
 }
```
这里我们可以看到，在inflate()初始化mInflatedVIewRef之前，如果设置visibility为VISIBLE的话，是会调用inflate()方法的，在mInflatedViewRef不为null之后就不会再去调用inflate()了

### viewStub.getVisibility()为什么一直为0
在显示ViewStub中的布局时，我们可能会采用如下的方法
```
if(viewStub.getVisibility() == View.GONE){
  viewStub.setVisibility(View.VISIBLE);
}else{
  viewStub.setVisibility(View.GONE);
}
```
这真的是是一个大坑。这样写你会发现点击“显示”按钮后ViewStub里面的布局不会再显示出来，也就是说if语句里面的代码没有执行。如果你将viewStub.getVisibility()的值打印出来，就会看到它始终为0，这恰恰是View.VISIBLE的值。奇怪，我们明明写了viewStub.setVisibility(View.GONE)，layout_view_stub也隐藏了，为什么ViewStub的状态还是可见呢？

重新回到"直接调用setVisibility()方法"中，看看ViewStub中的setVisibility()源码，首先判断弱引用对象mInflatedViewRef是否为空，不为空则取出存放进去的对象，也就是我们ViewStub中的View，然后调用了view的setVisibility()方法，mInflatedViewRef为空时，则判断visibility为VISIBLE或INVISIBLE时调用inflate()方法填充布局，如果为GONE的话则不予处理。这样一来，在mInflatedViewRef不为空，也就是已经填充了布局的情况下，ViewStub中的setVisibility()方法实际上是在设置内部视图的可见性，而不是ViewStub本身。这样的设计其实也符合ViewStub的特性，即填充布局之后就自我销毁了，给其设置可见性是没有意义的

### 操作布局控件

其实我们可以这样认为，ViewStub就是一个懒惰的include，我们需要他加载时才会加载，要操作布局里面的控件也跟include一样，我们可以先初始化ViewStub中的布局，然后在初始化控件

```
// 1. 初始化inflate的布局后在初始化其中的控件
FrameLayout frameLayout = findViewById(R.id.view_inflate);// android:inflatedId设置id
switch sw = frameLayout.findViewById(R.id.sw);
sw.toggle();
```

如果主布局中控件id没有冲突，可以直接初始化控件即可

```
// 2. 直接初始化控件
Switch sw = findViewById(R.id.sw);
sw.toggle();
```


