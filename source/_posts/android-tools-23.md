---
title: android奇技淫巧 23 SpannableStringBuilder
date: 2019-11-20 20:38:10
tags:
  - android
---

最近一段时间一直在做聊天部分的内容，这部分中，让我对SpannableStringBuilder这个类有了一些新的认识，自己也总结了一下他的用法，记录下来，方便以后查阅

<!--more-->

> SpannableString和SpannableStringBuilder的关系类似于String和StringBuilder。前者不可变，后者可变

功能在于给遗传普通的字符串加上颜色，大小，背景等样式，以及一些点击事件
![SpannableStringBuilder的例子](/assets/tools/tools-span-01.png)

这里我们会发现其实就是一个简单的TextView.那么我们其实可以使用Html.fromHtml()的方法设置颜色，背景等样式。但是如果我们查看了Html.formHtml()方法的源码，就会发现，其实它的里面还是用SpannableStringBuilder实现的,如下
```
public Spanned convert() {

        mReader.setContentHandler(this);
        try {
            mReader.parse(new InputSource(new StringReader(mSource)));
        } catch (IOException e) {
            // We are reading from a string. There should not be IO problems.
            throw new RuntimeException(e);
        } catch (SAXException e) {
            // TagSoup doesn't throw parse exceptions.
            throw new RuntimeException(e);
        }

        // Fix flags and range for paragraph-type markup.
        Object[] obj = mSpannableStringBuilder.getSpans(0, mSpannableStringBuilder.length(), ParagraphStyle.class);
        for (int i = 0; i < obj.length; i++) {
            int start = mSpannableStringBuilder.getSpanStart(obj[i]);
            int end = mSpannableStringBuilder.getSpanEnd(obj[i]);

            // If the last line of the range is blank, back off by one.
            if (end - 2 >= 0) {
                if (mSpannableStringBuilder.charAt(end - 1) == '\n' &&
                    mSpannableStringBuilder.charAt(end - 2) == '\n') {
                    end--;
                }
            }

            if (end == start) {
                mSpannableStringBuilder.removeSpan(obj[i]);
            } else {
                mSpannableStringBuilder.setSpan(obj[i], start, end, Spannable.SPAN_PARAGRAPH);
            }
        }

        return mSpannableStringBuilder;
    }
```
上面是截取的Html类中关于数据解析的方法源码，这里我们会发现，我们频繁的使用了mSpannableStringBuilder这个对象，其实这个对象就是SpannableStringBuilder对象。所以说我们还是需要将SpannableStringBuilder和SpannableStringBuilder详细学习的。

## SpannableString
这里先看一个小例子
```
SpannableString ss = new SpannableString("这是测试内容简单的文字");
ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.RED);
ss.setSpan(colorSpan,0,5,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
tv.setText(ss);
```
这个方法和SpannableStringBuilder类似，只做简单介绍，不做过多解释

## SpannableStringBuilder

我们先来看一下SpannableStringBuilder的类的声明
```
/**
 * This is the class for text whose content and markup can both be changed.
 */
public class SpannableStringBuilder implements CharSequence, GetChars, Spannable, Editable,
        Appendable, GraphicsOperations {
    private final static String TAG = "SpannableStringBuilder";
```

这里需要注意，CharSequence和Spannable，继承自Spannable，赋予了它给文本设置样式的基础功能，实现接口CharSequence则代表他能在很多地方使用，比如TextView的setText方法

在源码中有两个比较重要的方法

```
// 该方法将文本添加到SpannableStringBuilder中，和StringBuild的append方法类似
public SpannableStringBuilder append(Charsequence text)
public void setSpan(Object what,int start,int end,int flags)
```
关键是setSpan方法，这个方法的参数如下

1. what:各种span，不同的Span对应不同的样式，具体如下
    - ForegroundColorSpan：设置文本前景色(文本颜色)
    - BackgroundColorSpan:设置文本背景色
    - AbsoluteSizeSpan:设置绝对的文字大小，px为单位
    - ClickableSpan:为文字添加点击事件(类似于微信朋友圈评论用户中的用户昵称或者@用户,在小例子中会介绍@的方式)
    - ImageSpan:文本添加图片，最近一段时间一直是在做跟这个相关的
    - RelativeSizeSpan：设置相对文字的大小，为倍数，相对于其他文字的大小
    - StrikethroughSpan：设置添加删除线
    - SubscriptSpan:设置下标文字
    - SuperscriptSpan：设置上标文字
    - URLSpan：文字设置超链接
    - UnderlineSpan：设置下划线
2. start：样式开始生效的实际位置，包括该位置
3. end:样式结束的位置，不包括该位置
4. flags：这个常量传入标志，一般有四个值
    - Spannable.SPAN_EXCLUSIVE_INCLUSIVE：在Span前面输入的字符不应用Span的效果，在后面输入的字符应用Span的效果
    - Spannable.SPAN_INCLUSIVE_EXCLUSIVE：在Span前面输入的字符应用Span效果，在后面输入的字符不应用Span效果
    - Spannable.SPAN_INCUJSIVE_INCLUSIVE:在Span前后输入的字符都要应用Span的效果
    - Spannable.SPAN_EXCLUSIVE_EXCLUSIVE:在Span前后输入的字符都不应用Span效果

文字叙述比较难理解，我们来看一下图，这里只针对输入框
![原始样式](/assets/tools/tools-spanstring-01.png)

## ForegroundColorSpan
使用的方式如下
```
public class ForgegroundColorActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_foreground_color);

        TextView tvColor = findViewById(R.id.tvColor);
        SpannableStringBuilder sb = new SpannableStringBuilder();
        sb.append("测试字体颜色");
        ForegroundColorSpan redSpan = new ForegroundColorSpan(Color.RED);
        sb.setSpan(redSpan,0,2, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        ForegroundColorSpan greenSpan = new ForegroundColorSpan(Color.GREEN);
        sb.setSpan(greenSpan,2,4,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        ForegroundColorSpan blueSpan = new ForegroundColorSpan(Color.BLUE);
        sb.setSpan(blueSpan,4,6,Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        tvColor.setText(sb);
    }
}
```

效果如下所示：
![颜色字体](/assets/tools/tools-spanstring-03.png)

## ImageSpan

文字添加一个图片。我们都知道可以在TextView中通过drawableLeft等操作添加一个图片，但是这样的图片有时候不符合我们的要求，比如我们希望将图片放在文字的末尾(注意是文字的末尾)
看一下例子

```
public class ImageSpanActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_imagespan);
        TextView tvImage = findViewById(R.id.tvImage);

        SpannableStringBuilder sb = new SpannableStringBuilder();
        String ss = "这是测试文字a";
        sb.append(ss);
        ImageSpan span = new ImageSpan(this, R.drawable.aliwx_s001);
        sb.setSpan(span, ss.length() - 1, ss.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        tvImage.setText(sb);
    }
}
```

看一下效果图
![图片字体](/assets/tools/tools-spanstring-04.png)

OK,能够显示图片到textview中了，但是目测好像图片的大小比较大，怎么才能够让图片的大小和文字的大小想接近内，好吧，先看一下代码
这里我们需要写一个类，继承自ImageSpan类，并且在类中重写getSize方法重新绘制图片大小，重写onDraw方法，重新绘制图片
```
public class ResizeImageSpan extends ImageSpan {

    public ResizeImageSpan(Context context, int res) {
        super(context, res);
    }

    @Override
    public int getSize(Paint paint, CharSequence text, int start, int end, Paint.FontMetricsInt fm) {
        try {
            // 获取默认的drawable对象
            Drawable drawable = getDrawable();
            // 默认drawable所占有的矩形空间
            Rect rect = drawable.getBounds();
            // 判断当前尺寸大小
            if (fm != null) {
                Paint.FontMetricsInt fmPaint = paint.getFontMetricsInt();
                int fontHeight = fmPaint.bottom - fmPaint.top;
                int drHeight = rect.bottom - rect.top;
                int top = drHeight / 2 - fontHeight / 4;
                int bottom = drHeight / 2 + fontHeight / 4;
                // 设置尺寸
                fm.ascent = -bottom;
                fm.top = -bottom;
                fm.bottom = top;
                fm.descent = top;
            }
            return rect.right;
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return 0;
    }

    @Override
    public void draw(Canvas canvas, CharSequence text, int start, int end, float x, int top, int y, int bottom, Paint paint) {
        try {
            Drawable b = getDrawable();
            Paint.FontMetricsInt fm = paint.getFontMetricsInt();
            int transY = (y + fm.descent + y + fm.ascent) / 2 - b.getBounds().bottom / 2;
            canvas.save();
            canvas.translate(x, transY);
            b.draw(canvas);
            canvas.restore();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

并且当我们调用的时候，不再使用上面的简单调用，而是使用如下的方式
```
SpannableStringBuilder sb = new SpannableStringBuilder();
        String ss = "这是测试文字a";
        sb.append(ss);
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(),R.drawable.aliwx_s001);
        Drawable drawable = new BitmapDrawable(bitmap);
        if (drawable != null){
            drawable.setBounds(0,0,drawable.getIntrinsicWidth(),drawable.getIntrinsicHeight());
        }
        ResizeImageSpan span = new ResizeImageSpan(drawable);
        sb.setSpan(span, ss.length() - 1, ss.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        tvImage.setText(sb);
```

优化图片大小
![图片字体](/assets/tools/tools-spanstring-05.png)

## ClickableSpan
可以点击的文字
```
public class ClickableSpanActivity extends AppCompatActivity {

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_clickable);

        TextView textView = findViewById(R.id.tvClick);

        SpannableStringBuilder sb = new SpannableStringBuilder();
        String ss = "这是测试文字";
        sb.append(ss);
        ClickableSpan span = new ClickableSpan() {
            @Override
            public void onClick(View widget) {
                Toast.makeText(ClickableSpanActivity.this, "点击了文字", Toast.LENGTH_LONG).show();
            }
        };
        sb.setSpan(span, sb.length() - 2, sb.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        ForegroundColorSpan colorSpan = new ForegroundColorSpan(Color.RED);
        sb.setSpan(colorSpan, sb.length() - 2, sb.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);

        textView.setText(sb);
        textView.setMovementMethod(LinkMovementMethod.getInstance());
    }
}
```

效果如下所示
![点击文字](/assets/tools/tools-spanstring-06.gif)

# 参考资料
[强大的SpannableStringBuilder](https://www.jianshu.com/p/f004300c6920)
[SpannableString和SpannableStringBuilder的使用](https://juejin.im/post/5b927af16fb9a05d2b6d9918)