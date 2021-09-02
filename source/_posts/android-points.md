---
title: android开发知识点
date: 2018-04-16 20:51:25
tags:
  - android
---
记录自己在开发中遇到的问题，以及解决方法
<!--more-->
1. 关于Fragment的引用到Activity，当横竖屏切换的时候，有时候会报空指针异常的原因分析
我们一般有两种方法创建当前的Fragment对象。第一种是通过new的方式，创建Fragment，另一种是通过在当前的Fragment中使用静态方法返回一个Fragment对象。这两种方法中，第二种方法是官方推荐的方法，这是因为，当我们对手机进行横竖屏的切换的时候，需要执行的操作是对Activity的生命周期的操作，而如果是使用new的方式创建Fragment，那么在重新调用Activity的生命周期时只会调用无参构造。所以我们在使用Fragment的时候，最好使用下面的方法：

```
public MyFragment(){
    //无参构造
}
//在这里传递的参数表示Activity想要传递给Fragment的数据，我们可以通过oncreate方法接收数据
Public static MyFragment newInstance(String ss){
    MyFragment myFragment = new MyFragment();
    Bundle bundle = new Bundle();
    Bundle.putString(“key”,ss);
    myFragment.setArguments(bundle);
    Return myFragment;
}
//在这里可以得到想要传递来的值
Public void onCreate(Bundle savedInstanceState){
    Super.oncreate(savedInstanceState);
    If(getArgument != null){
        this.ss = getArgument().getString(“key”);
    }
}
```
2. 关于Fragment使用getActivity()回报空指针异常
使用如下代码，在内存紧张的时候，会出现空指针异常
```
Private void showToash(String text){
    getActivity().runOnUiThread(()->
        Toast.makeText(getActivity(),text,Toast.LENGTH_SHORT).show();
    );
}
```
原因：在执行onAttach()时，Fragment和Activity实现了绑定，在调用getActivity()后就会得到于Fragment绑定的Activity对象，当内存较低时，系统会回收到Fragment所依附的Activity，所以在执行onDetach()方法时，Fragment已经实现了和Activitity的绑定。解决这个问题就是在执行onAttach()方法之后直接调用getActivity()方法获取当前的上下文对象，并保存为全局变量

3. Fragment实现懒加载操作
一般情况下我们实现懒加载的时候，可以通过onResume()和onPause()方法实现，但是在实际开发中我们发现Fragment并没有真的调用这两个方法，因为在源码中，官方已经说了，这两个方法的调用会被Fragment所依附的Activity代替，那么就是说，我们调用的应该是Activity中的onResume()和onPause()方法。我们可以通过Fragment的另外一个方法setUserVisibleHint(boolean isVisibleToUser)来实现懒加载的操作，其中isVisibleToUser这个变量表示当前页面对用户可以见。那么我们只需要判断当前的真假就可以执行响应的代码。但是这个还有另外一个问题，就是setUserVisibleHint()方法的调用时间是不固定的，因为他判断的是当前用户是否看见，那么就存在当前页面已经可见，但是空间初始化没有完成的操作，空指针异常。
解决办法：创建一个flag，当执行setUserVisibleHint的时候检测onViewCreated是否已经被调用了，如果是，在根据对用户可见，执行UI代码：
```
Protected void onViewCreate(View view,Bundle SavedInstanceState){
    isViewCreated = true;
    if(mIsVisibleToUser){
        onLazyLoad();
        isFirstVisible = false;
        onFragmentResume()
    }
}
Public void setUserVisibleHint(boolean isVisibleToUser){
    mIsVisibleToUser = isVisibleToUser;//当自动执行该方法的时候会将当前的是否可见的值传递给全局变量
    If(!isViewCreated) return;//如果视图还没有创建,直接返回
    If(isVisibleToUser){
        onFragmentResume();//对用户可见，执行加载操作
        if(isFirstVisible){
            onLoayLoad();//第一次对用户可见，执行懒加载
            isFirstVisible = false;
        }
    }else{
        onFragmentPause();
    }
}
```
4. content.getSharedPreference(),PackageManager.getSharedPreference()和PackageManager.getDefaultSharedpreference()
第一个是一般形式是
getSharedPreference(String name,int node) 创建一个名字是name.xml的偏好设置文件，然后权限就是mode
getSharedPreference(int mode)创建一个名字是当前Activity.xml的偏好设置文件，然后权限就是mode
第二个的源码中所呈现的方式是非常相似的。这个方法是一个普通方法，必须通过PreferenceManager的实例调用才可以，只不过他得到的是一个名字为”yourpackagename_preferences”的偏好文件，并且mode为默认私有
第三个是一个静态方法，因此我们可以直接调用，同和跟第二个最终的结果是一样的，只不过调用的方法不一样而已



















> 因为所有的都是自己在开发中记录的，大部分的代码都是伪代码，如果以后有时间，会将代码补全