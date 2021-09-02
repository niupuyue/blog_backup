---
title: android奇技淫巧 22 Viewpager小功能
date: 2019-11-18 20:38:10
tags:
  - android
---

最近一段时间，公司在赶项目，各种需求也是层出不穷。自己在实现一个viewpager的作用滑动的时候，有一个功能竟然自己不知道该如何实现。先来看一下UI

<!--more-->

![UI图](/assets/tools/tools-viewpager-01.png)

这里面我们会发现，在viewpager的数量较多时，可以实现左右滑动效果，并且，可以有一定的预览功能。其次，如果页面中只有一个，那么就要求宽度和屏幕宽度一致

![UI图2](/assets/tools/tools-viewpager-02.png)

而且需要注意的是，在右侧，是需要从最右侧屏幕出来的。

我的实现方法如下所示


#### 逻辑代码
```
// TODO 设置假数据
        final List<String> joinGroups = new ArrayList<>();
        joinGroups.add("邀请1");
        joinGroups.add("邀请2");
        vpPubTabMessage.setPageMargin(LibUtility.dp2px(10)); //显示viewpager间距
        vpPubTabMessage.setAdapter(new PagerAdapter() {
            @Override
            public int getCount() {
                return joinGroups.size();
            }

            @Override
            public boolean isViewFromObject(View view, Object object) {
                return view == object;
            }

            @Override
            public Object instantiateItem(ViewGroup container, int position) {
                View childView = LayoutInflater.from(container.getContext()).inflate(R.layout.view_invite_join_group_item, null, false);
                RoundedImageView groupHeader = childView.findViewById(R.id.rivInviteJoinGroupHeader);
                TextView groupName = childView.findViewById(R.id.tvInviteJoinGroupName);
                TextView groupMsg = childView.findViewById(R.id.tvInviteJoinGroupMsg);
                TextView groupJoin = childView.findViewById(R.id.tvInviteJoinGroupJoin);
                View leftView = childView.findViewById(R.id.viewLeft);
                View rightView = childView.findViewById(R.id.viewRight);
                UtilitySecurity.resetVisibility(leftView, position == 0);
                UtilitySecurity.resetVisibility(rightView, position == getCount() - 1);
                // 设置群头像
//                UtilitySecurity.setBackground(groupHeader, ContextCompat.getDrawable(getActivity(), bgImages[position]));
                // 设置群名称和消息
                UtilitySecurity.setText(groupName, joinGroups.get(position));
                UtilitySecurity.setText(groupMsg, joinGroups.get(position));
                childView.setOnClickListener(new View.OnClickListener() {
                    @Override
                    public void onClick(View v) {
                        // 设置整个view的点击事件，跳转到群聊待确认页面
                        Intent intent = ChatGroupConfirmActivity.getIntent(getContext(), "123");
                        startActivity(intent);
                    }
                });
                container.addView(childView);
                return childView;
            }

            @Override
            public void destroyItem(@NonNull ViewGroup container, int position, @NonNull Object object) {
                container.removeView((View) object);
            }

            @Override
            public float getPageWidth(int position) {
                if (getCount() == 1) {
                    return super.getPageWidth(position);
                } else {
                    return 0.84f;
                }
            }
        });
```

#### 布局文件

view_invite_join_group_item.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="@color/_ebeff2"
    android:orientation="horizontal">

    <View
        android:id="@+id/viewLeft"
        android:layout_width="10dp"
        android:layout_height="match_parent" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:layout_marginBottom="10dp"
        android:layout_weight="1"
        android:background="@drawable/shape_7_white"
        android:orientation="horizontal"
        android:paddingLeft="8dp"
        android:paddingTop="17dp"
        android:paddingRight="17dp"
        android:paddingBottom="13dp">

        <com.renrui.libraries.widget.RoundedImageView
            android:id="@+id/rivInviteJoinGroupHeader"
            android:layout_width="52dp"
            android:layout_height="52dp"
            android:layout_gravity="center_vertical"
            android:src="@mipmap/ic_avatar_default" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:orientation="vertical">

            <RelativeLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content">

                <TextView
                    android:id="@+id/tvInviteJoinGroupName"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_alignParentStart="true"
                    android:layout_centerVertical="true"
                    android:text="开发蹦迪小分队"
                    android:textColor="@color/gray_3333"
                    android:textSize="16sp"
                    android:textStyle="bold" />

                <TextView
                    android:id="@+id/tvInviteJoinGroupJoin"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_alignParentEnd="true"
                    android:background="@drawable/bg_corner_big_border_red"
                    android:paddingLeft="15dp"
                    android:paddingTop="4dp"
                    android:paddingRight="15dp"
                    android:paddingBottom="4dp"
                    android:text="进入看看"
                    android:textColor="#f15a5f" />

            </RelativeLayout>

            <TextView
                android:id="@+id/tvInviteJoinGroupMsg"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="12dp"
                android:maxLines="1"
                android:singleLine="true"
                android:text="Hi，你来了。这里或许有你未来并肩作战的同事。实名备注，文明用语"
                android:textColor="@color/gray_9999"
                android:textSize="14sp" />

        </LinearLayout>

    </LinearLayout>

    <View
        android:id="@+id/viewRight"
        android:layout_width="10dp"
        android:layout_height="match_parent" />

</LinearLayout>
```

其中viewpager所在的布局文件如下所示
```
<android.support.v4.view.ViewPager
        android:id="@+id/vpPubTabMessage"
        android:layout_width="match_parent"
        android:layout_height="110dp"/>
```

这个部分的内容比较简单，就不做过多介绍

其实这些代码我们都会写，但是有时候就是老忘，每次都要重新再去找相关的资料。记录下来，方便自己以后借鉴


2021年06月21日17:33:13 补充

### Viewpager2的使用
