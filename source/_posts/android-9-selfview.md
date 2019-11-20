---
title: 重拾android路(九) 自定义View
date: 2016-05-30 23:25:45
tags:
  - android
---
自定义view
<!--more-->
自定义view所包含的内容非常多，打算分好几次去写，所以时间可能会拉的比较长

2019年11月20日16:42:04
之前写的内容全部删除，重新写，因为感觉自己之前写的比较杂乱，很多地方都没有说清楚。
最近一段时间，公司在做IM的聊天功能，其中很多地方都是用到了自定义view，那么也是趁着这个时间，将自己之前没有总结到位的再重新总结一下

目前我使用到的自定义view分为两个部分，第一个是通过引入新的布局，即在将布局引入到我们的自定义view中；第二种是通过onDraw等方法绘制新的view

# 引入布局
这种方法相对而言比较简单，我们只需要将相应的Layout布局文件引入即可，下面举一个例子，具体的操作不细说，直接看结果

![引入布局的自定义view](/assets/selfview/selfview-01.gif)

```
// 自定义view代码
/**
 * 弹幕
 */
public class BarrageView extends LinearLayout {

    private static final int HANDLER_MSG_CODE_STAR_ANIM = 0;

    private static final int SHOW_RECENT_ITEM_LIMIT = 5;
    private static final int SHOW_RECENT_ITEM_DURATION = 2000;
    private static final int SHOW_RECENT_ITEM_MARGIN_TOP = 30;

    private Context mContext;
    private View mRoot;
    private LinearLayout llJobDetailBarrageContainer;
    private List<RecentApplyModel> recentTipViews;
    private Pools.SimplePool<TextView> recentTipsView;
    private BarrageHandler mHandler;

    private int index;
    private int offset;

    public BarrageView(Context context) {
        this(context, null);
    }

    public BarrageView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public BarrageView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        this.mContext = context;
        initView();
    }

    // 初始化页面布局
    private void initView() {
        mRoot = LayoutInflater.from(mContext).inflate(R.layout.view_job_detail_barrage, this);
        llJobDetailBarrageContainer = mRoot.findViewById(R.id.llJobDetailBarrageContainer);

        // 设计开场动画
        LayoutTransition transition = new LayoutTransition();
        PropertyValuesHolder scaleY = PropertyValuesHolder.ofFloat("scaleY", 0, 1);
        PropertyValuesHolder scaleX = PropertyValuesHolder.ofFloat("scaleX", 0, 1);

        ObjectAnimator valueAnimator = ObjectAnimator.ofPropertyValuesHolder(null, new PropertyValuesHolder[]{scaleX, scaleY})
                .setDuration(transition.getDuration(LayoutTransition.APPEARING));
        valueAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationStart(Animator animation) {
                super.onAnimationStart(animation);
                ObjectAnimator objectAnimator = (ObjectAnimator) animation;
                View view = (View) objectAnimator.getTarget();
                view.setPivotX(0f);
                view.setPivotY(view.getMeasuredHeight());
            }
        });
        transition.setAnimator(LayoutTransition.APPEARING, valueAnimator);
        ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(null, "alpha", 0, 1).setDuration(LayoutTransition.DISAPPEARING);
        transition.setAnimator(LayoutTransition.DISAPPEARING, objectAnimator);
        llJobDetailBarrageContainer.setLayoutTransition(transition);

        // 初始化Handler对象
        mHandler = new BarrageHandler(this);
    }

    /**
     * 获取最近消息的布局文件
     *
     * @return
     */
    private View getRecentTipsItem() {
        View tipView = recentTipsView.acquire();
        if (tipView == null) {
            tipView = LayoutInflater.from(mContext).inflate(R.layout.view_recent_tip, null);
        }
        return tipView;
    }

    /**
     * Handler对象，在这个class中完成每隔2秒钟弹出一个消息
     */
    private class BarrageHandler extends Handler {
        WeakReference<BarrageView> mWeakReference;

        // 构造方式初始化弱引用对象
        public BarrageHandler(BarrageView mWeakReference) {
            this.mWeakReference = new WeakReference(mWeakReference);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            if (recentTipViews.isEmpty()) {
                return;
            }
            // 判断当前发送的消息类型是否正确
            if (msg.what == HANDLER_MSG_CODE_STAR_ANIM) {

                if (llJobDetailBarrageContainer.getChildCount() == recentTipViews.size() && index <= offset) {
                    // 判断如果容器中存在5个子元素，并且当前下标小于等于偏移数据，则将第一个子元素移除掉
                    llJobDetailBarrageContainer.removeViewAt(0);
                } else if (index >= SHOW_RECENT_ITEM_LIMIT && index <= offset) {
                    // 如果下标大于5 并且下标小于等于偏移数据，移除容器中的第一个子元素
                    llJobDetailBarrageContainer.removeViewAt(0);
                }

                if (index < recentTipViews.size()) { // 如果当前下标小于需要显示的数据内容，则需要向容器中添加一个数据
                    View recentItem = getRecentTipsItem();
                    ImageView rivTipLogo = recentItem.findViewById(R.id.rivTipLogo);
                    TextView tvContent = recentItem.findViewById(R.id.tvContent);
                    // 填充数据内容
                    tvContent.setText(recentTipViews.get(index).name + recentTipViews.get(index).content);
                    rivTipLogo.setImageResource(R.mipmap.ic_avatar_default);
                    // 将recentItem添加到容器中
                    llJobDetailBarrageContainer.addView(recentItem);
                    // 设置上外边距为30px
                    LinearLayout.LayoutParams lp = (LayoutParams) recentItem.getLayoutParams();
                    lp.setMargins(0, SHOW_RECENT_ITEM_MARGIN_TOP, 0, 0);
                    // 延迟2秒钟添加一个新数据
                    sendEmptyMessageDelayed(0, SHOW_RECENT_ITEM_DURATION);
                    // 下标自增
                    index++;
                    recentItem = null;
                } else if (index <= offset) { // 如果已经将数据内容全部展示，后面为了能够让recentItem向上移动，向容器中添加空数据
                    View recentItem = getRecentTipsItem();
                    recentItem.setVisibility(View.INVISIBLE);
                    llJobDetailBarrageContainer.addView(recentItem);
                    LinearLayout.LayoutParams lp = (LayoutParams) recentItem.getLayoutParams();
                    lp.setMargins(0, SHOW_RECENT_ITEM_MARGIN_TOP, 0, 0);
                    sendEmptyMessageDelayed(0, SHOW_RECENT_ITEM_DURATION);
                    index++;
                } else {
                    // 弹幕弹窗显示完毕，自动关闭
                    stop();
                    return;
                }
                // 设置透明效果(因为五个item透明度没有规则，所以只能写死)
                setContianerAlpha();
            }
        }
    }

    // 设置每个子view的透明度(此处数据是写死的)
    private void setContianerAlpha() {
        if (llJobDetailBarrageContainer.getChildCount() == 1) {
            llJobDetailBarrageContainer.getChildAt(0).getBackground().setAlpha(148);
        } else if (llJobDetailBarrageContainer.getChildCount() == 2) {
            llJobDetailBarrageContainer.getChildAt(0).getBackground().setAlpha(46);
            llJobDetailBarrageContainer.getChildAt(1).getBackground().setAlpha(148);
        }
        if (llJobDetailBarrageContainer.getChildCount() == 3) {
            llJobDetailBarrageContainer.getChildAt(0).getBackground().setAlpha(46);
            llJobDetailBarrageContainer.getChildAt(1).getBackground().setAlpha(122);
            llJobDetailBarrageContainer.getChildAt(2).getBackground().setAlpha(148);
        } else if (llJobDetailBarrageContainer.getChildCount() == 4) {
            llJobDetailBarrageContainer.getChildAt(0).getBackground().setAlpha(46);
            llJobDetailBarrageContainer.getChildAt(1).getBackground().setAlpha(122);
            llJobDetailBarrageContainer.getChildAt(2).getBackground().setAlpha(148);
            llJobDetailBarrageContainer.getChildAt(3).getBackground().setAlpha(174);
        } else if (llJobDetailBarrageContainer.getChildCount() == 5) {
            llJobDetailBarrageContainer.getChildAt(0).getBackground().setAlpha(46);
            llJobDetailBarrageContainer.getChildAt(1).getBackground().setAlpha(122);
            llJobDetailBarrageContainer.getChildAt(2).getBackground().setAlpha(148);
            llJobDetailBarrageContainer.getChildAt(3).getBackground().setAlpha(174);
            llJobDetailBarrageContainer.getChildAt(4).getBackground().setAlpha(174);
        }
    }

    ////////////////////////////////////////////////////////////////////////////对外开放方法--必须实现//////////////////////////////////////////////////////////////////////

    /**
     * 开启动画，展示RecentTips
     * 先设置数据，再调用start()方法
     */
    public void start() {
        if (mHandler != null) {
            mHandler.sendEmptyMessage(HANDLER_MSG_CODE_STAR_ANIM);
        }
    }

    /**
     * 关闭动画
     */
    public void stop() {
        if (mHandler != null) {
            mHandler.removeMessages(HANDLER_MSG_CODE_STAR_ANIM);
        }
    }

    /**
     * 设置数据，将RecentApplyModel的集合数据传递过来即可
     *
     * @param recentTips
     */
    public void setRecentTips(List<RecentApplyModel> recentTips) {
        this.recentTipViews = recentTips;
        recentTipsView = new Pools.SimplePool<>(recentTipViews.size());
        offset = recentTipViews.size() * 2;
    }
    ////////////////////////////////////////////////////////////////////////////对外开放方法--必须实现//////////////////////////////////////////////////////////////////////

    static class RecentApplyModel {
        public String name;
        public String photo;
        public String content;
    }
}
```

总布局页面 view_job_detail_barrage.xml
```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:id="@+id/llJobDetailBarrageContainer"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom"
        android:animateLayoutChanges="true"
        android:layoutAnimation="@anim/anim_barrage_bottom_to_top"
        android:orientation="vertical"
        android:padding="15dp"
        android:showDividers="middle" />

</FrameLayout>
```

其中动画anim_barrage_bottom_to_top.xml
```
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:animation="@anim/anim_bottom_to_top"
    android:delay="200"
    android:animationOrder="normal">

</layoutAnimation>
```

其中的动画文件anim_bottom_to_top.mxl
```
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000">
    <translate
        android:fromXDelta="-50%p"
        android:toXDelta="0" />
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0" />
</set>
```

布局文件view_recent_tip.xml
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:background="@drawable/bg_recent_tip"
    android:gravity="center_vertical"
    android:orientation="horizontal"
    android:padding="5dp">

    <ImageView
        android:id="@+id/rivTipLogo"
        android:layout_width="25dp"
        android:layout_height="25dp"
        android:scaleType="centerCrop" />

    <TextView
        android:id="@+id/tvContent"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="5dp"
        android:textColor="#ffffff"
        android:textSize="13sp" />

</LinearLayout>
```

使用方法，逻辑代码
```
public class LayoutViewActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_layout_view);

        BarrageView jobDetailBarrageView = findViewById(R.id.jobDetailBarrageView);
        jobDetailBarrageView.setRecentTips(MainActivity.getBarrageData());
        jobDetailBarrageView.start();
    }
}
```
使用方式，布局文件activity_layout_view.xml
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.paulniu.selfview_demo.BarrageView
        android:id="@+id/jobDetailBarrageView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="50dp"
        android:visibility="visible" />

</RelativeLayout>
```

> 这里没什么好说的，引入不同的布局文件，在布局文件中对特定的组件执行相应的操作.

下面我们来看一下，如何绘制一个自定义view

# 绘制自定义view

在我们的实际开发中，可能会遇到UI设计师设计出来的样式非常复杂，并且这种复杂导致我们直接使用系统为我们提供的组件远远不能满足我们的需要。这是我们就需要使用绘制的方式完成自定义view

再开始自定义view之前，先来学习一下基础知识，这部分的内容，可能介绍的不是很详细，大部分会通过代码和结果的方式演示





# 参考资料
[HenCoder大神的自定义View教程](http://hencoder.com/ui-1-1/)
[一个来自二次元的“萌妹”](http://www.gcssloop.com/customview/CustomViewIndex/)