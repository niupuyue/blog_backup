---
title: 重拾android路(二十四) Android自定义键盘
date: 2019-5-08 11:27:11
tags:
  - android
---

目前公司项目集成的是阿里云旺的IM通信系统，这套系统现在已经过时了，而且项目组已经不再维护了。所以决定将IM模块重新设置一遍。然后在这个过程中，我负责了聊天键盘的操作，这个过程让我对键盘和语音播放的内容有了进一步的认识，写篇博客，记录一下
<!--more-->
# IMChatPanel
聊天系统中的软键盘，这个模块的重点在于显示键盘和显示扩展模块。想说一下我的思路:
我们想完成一个移植性比较高的聊天键盘，所以我们需要将键盘中的操作都已接口的形式暴露出来，包括表情的点击事件，语音录制，拍照等功能。
除此之外，我们还要想着将键盘的高度和系统软键盘的高度保持一致。那么下面我们一步一步的实现我们的功能

## 创建输入框基本布局页面
首先我们创建一个名为IMChatPanelView的类作为我们的软键盘自定义view。因为这里我们想要实现的效果是上面有一个输入框，下面是软键盘的扩展，所以这里我们直接采用线性布局，让IMChatPanelView继承自LinearLayout。通过LayoutInflate引入一个xml文件作为基本布局，布局的文件比较简单，我们直接看代码
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <include
        android:id="@+id/includeChatKeyboardBar"
        layout="@layout/view_chat_keyboard_bar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_above="@+id/llFcExpandContainer" />

    <!--更多页面的容器，用来放置表情，更多，语音Fragment-->
    <RelativeLayout
        android:id="@+id/rlVckExpandContainer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#FFFFFF"
        android:visibility="gone" />

</LinearLayout>
```
这里我们会发现，外层是一个LinearLayout，里面有两个子view，一个是```include```另外一个是```RelativeLayout```。其中```RelativeLayout```是专门用来存放软键盘扩展的，比如表情，更多的页面，我们通过Fragment引入到这个RelativeLayout中。再来看一下我们引入的```include```布局中有的内容
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/llVckbContainer"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="#FFFFFF"
    android:gravity="center_vertical"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <!--常用语-->
        <TextView
            android:id="@+id/tvVckbPhrase"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="7dp"
            android:layout_marginLeft="7dp"
            android:layout_marginStart="7dp"
            android:layout_marginTop="7dp"
            android:background="@drawable/bg_button_square_rededge_selector"
            android:gravity="center_vertical"
            android:paddingBottom="7dp"
            android:paddingLeft="5dp"
            android:paddingRight="5dp"
            android:paddingTop="7dp"
            android:text="@string/chat_keyboard_phrase_txt"
            android:textColor="#f04d52"
            android:textSize="12sp" />

        <!--输入框-->
        <com.renrui.job.widget.im.chatkeyboard.IMChatPanelEditTextView
            android:id="@+id/etVckbInput"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_centerVertical="true"
            android:layout_marginBottom="7dp"
            android:layout_marginLeft="7dp"
            android:layout_marginTop="7dp"
            android:layout_weight="1"
            android:background="@drawable/bg_chatting_keyboard_input"
            android:maxLength="1024"
            android:maxLines="4"
            android:minHeight="32dp"
            android:padding="4dp"
            android:textSize="16sp"
            android:visibility="visible" />

        <!--显示和隐藏表情-->
        <CheckBox
            android:id="@+id/cbVckbSmily"
            android:layout_width="27dp"
            android:layout_height="27dp"
            android:layout_gravity="right|center_vertical"
            android:layout_marginLeft="7dp"
            android:background="@drawable/chatting_expand_bar_smily_selector"
            android:button="@null" />

        <FrameLayout
            android:id="@+id/flVckbAddSendContainer"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="7dp"
            android:layout_marginRight="7dp">

            <!--显示和隐藏更多-->
            <CheckBox
                android:id="@+id/cbVckbMore"
                android:layout_width="27dp"
                android:layout_height="27dp"
                android:background="@drawable/chatting_expand_bar_more_selector"
                android:button="@null"
                android:visibility="visible" />

            <!--发送按钮-->
            <Button
                android:id="@+id/btnVckbSendBtn"
                android:layout_width="50dp"
                android:layout_height="30dp"
                android:background="@drawable/chatting_expand_bar_send_btn"
                android:clickable="true"
                android:gravity="center"
                android:singleLine="true"
                android:text="@string/button_send"
                android:textColor="#ffffff"
                android:textSize="12sp"
                android:visibility="gone" />
        </FrameLayout>

    </LinearLayout>

    <View
        android:layout_width="match_parent"
        android:layout_height="1dp"
        android:background="#C4C7C9" />

</LinearLayout>
```
在这段代码中，除了输入框之后，都是我们在日常开发中会使用到的控件，里面的布局和使用方法，我们略过，只看一下输入框。这个输入框里我们其实做的东西比不多，只是监听了系统的复制粘贴等操作。具体代码如下：
```
/**
 * Desc: 自定义输入框，为了实现监听剪切板复制粘贴操作
 */
public class IMChatPanelEditTextView extends AppCompatEditText {

    /**
     * 剪切板操作接口
     */
    public interface OnClipKeyboardDelegateCallback {

        void onPaste();
    }

    private Context mContext;
    private OnClipKeyboardDelegateCallback mClipKeyboardCallback;

    public IMChatPanelEditTextView(Context context) {
        super(context);
    }

    public IMChatPanelEditTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public IMChatPanelEditTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public void setClipKeyboardCallback(OnClipKeyboardDelegateCallback callback) {
        this.mClipKeyboardCallback = callback;
    }

    /**
     * 输入框菜单操作 包括了复制 粘贴等
     *
     * @param id
     * @return
     */
    @Override
    public boolean onTextContextMenuItem(int id) {
        if (id == android.R.id.paste) {
            // 粘贴
            if (mClipKeyboardCallback != null) {
                mClipKeyboardCallback.onPaste();
            }
        }
        return super.onTextContextMenuItem(id);
    }
}
```
有了布局文件之后，我们只需要在initView的时候，通过```mRoot = LayoutInflater.from(mContext).inflate(R.layout.view_chat_keyboard, this);```这句代码引入，就可以获取当前页面的rootView了。至于后面的比如控件实例化，添加点击事件，我们就不再一一赘述。

## 设置RelativeLayout
上面我们说了，RelativeLayout主要是用来切换不同的场景的，以方便可以显示表情，另外一方面可以完成动态添加Fragment。
主要代码：



