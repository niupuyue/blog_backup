---
title: 重拾android路(十七) Android部分框架介绍
date: 2018-02-03 23:17:14
tags:
  - android
---

到目前为止，也写了一些项目，那么现在我要把之前写项目中，使用到的框架，好好的总结一下，因为我感觉总是使用别人的，没有自己的特点，那么之后我会花一段时间，总结完成。后面的时候会不断补充。
<!--more-->
先把框架罗列一下：

1. Gilde
2. OkHttp
3. GreenDao/Realm
4. RxJava/RxAndroid
5. Dagger2
6. LeakCanary

# Glide
Glide是一个图片加载框架，使用起来比较简单，只要我们配置好相应的设置，就可以将图片缓存到本地，显示到控件中。

```
//引入依赖
repositories {
  mavenCentral()
  google()
}

dependencies {
  implementation 'com.github.bumptech.glide:glide:4.8.0'
  annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'
}
```

```
//glide的混淆

-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
  **[] $VALUES;
  public *;
}
 
# for DexGuard only
-keepresourcexmlelements manifest/application/meta-data@value=GlideModule
```

```
//我们还需要获取网络请求权限和读取本地存储的权限,如果是5.0就动态获取
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.INTERNET" />
```

## 使用
首先来看一下官网的例子，这个没啥好说的
```
// For a simple view:
@Override 
public void onCreate(Bundle savedInstanceState) {
  ...
  ImageView imageView = (ImageView) findViewById(R.id.my_image_view);
 
  Glide.with(this).load("http://goo.gl/gEgYUd").into(imageView);
}
 
// For a simple image list:
@Override 
public View getView(int position, View recycled, ViewGroup container) {
  final ImageView myImageView;
  if (recycled == null) {
    myImageView = (ImageView) inflater.inflate(R.layout.my_image_view, container, false);
  } else {
    myImageView = (ImageView) recycled;
  }
 
  String url = myUrls.get(position);
 
  Glide
    .with(myFragment)
    .load(url)
    .centerCrop()
    .placeholder(R.drawable.loading_spinner)
    .crossFade()
    .into(myImageView);
 
  return myImageView;
}
```
### 加载网络图片
```
Glide.with(context).load(internetUrl).into(targetImageView);
```
### 从本地图片图片
```
File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES),"Test.jpg");
Glide.with(context).load(file).into(imageViewFile);

```
### 根据资源Id加载图片
```
int resourceId = R.mipmap.ic_launcher;
Glide.with(context).load(resourceId).into(imageViewResource);

```
### 从Uri加载图片
```
Glide.with(context).load(uri).into(imageViewUri);
```
### 加载Gif图片
```
String gifUrl = "xxxxx";
Glide.with( context ).load( gifUrl ).into( imageViewGif );
```
### 利用bitmap播放gif
```
Glide.with( context ).load( gifUrl ).asBitmap().into( imageViewGifAsBitmap );
```
### 强制转换gif
```
Glide.with( context ).load( gifUrl ).asGif().error( R.drawable.full_cake ).into( imageViewGif );
```
### 图片加载失败
```
Glide.with( context ).load( gifUrl ).placeholder( R.drawable.cupcake ).error( R.drawable.full_cake ).into( imageViewGif );
```
### 图片占位符
```
Glide.with( context ).load( imageUrl ).placeholder( R.drawable.cupcake )
```
### fallback
如果我们设置了图片url为null，这时候需要使用fallback
```
Glide.with(context)
     .load( null)//加载空指针的时候
     .fallback( R.drawable.wuyanzu)
     .into( imageViewNoFade );
```
### 设置动画
淡入淡出动画
```
Glide.with(context).load().placeholder(R.mipmap.ic_launcher) .error(R.mipmap.future_studio_launcher).crossFade().into(imageViewFade);
```
这里还有一个.fadeFade(int duration)，设置动画时间。如果你不想要动画可以加上.dontAnimate()
.animate(android.R.anim.slide_in_left)：Android系统提供，从左到右滑出加载动画
### 调整图片大小
单位是像素，裁剪你的图片大小。其实Glide已经会自动根据你ImageView裁剪照片来放在缓存中了。但是不想适应ImageView大小的时候，可以调用这个方法.override()为ImageView指定大小。
```
Glide.with(context).load(image).override(600, 200) .into(imageViewResize);
```
### 裁剪图片
fitCenter(),centerCrop()
Glide清楚在合适的ImageView中加载合适的Image.当需要裁剪大小时，有个.centerCrop方法，这个方法的裁剪会让你的ImageView周围不会留白，还有一个.fitCenter()方法，表示让你的Image完全显示，尺寸不对时，周围会留白

### 设置缩略图
.thumbnail()方法的目的就是让用户先看到一个低解析度的图，点开后，再加载一个高解析度的图。
```
//表示为原图的十分之一
Glide.with( context ).load(image).thumbnail( 0.1f ).into( imageView2 );
```
当缩略图也需要进行网络请求时
```
private void loadImageThumbnailRequest() {
    DrawableRequestBuilder<String> thumbnailRequest = Glide.with( context ).load( eatFoodyImages[2] );
    Glide.with( context ).load( UsageExampleGifAndVideos.gifUrl ).thumbnail( thumbnailRequest ).into( imageView3 );
}
```
### 设置图片显示效果
```
Glide.with(this).load(R.mipmap.ic_image_sample)
     //模糊
     .bitmapTransform(new BlurTransformation(this))
     //圆角
     .bitmapTransform(new RoundedCornersTransformation(this, 24, 0, RoundedCornersTransformation.CornerType.ALL))
     //遮盖
     .bitmapTransform(new MaskTransformation(this, R.mipmap.ic_launcher))
     //灰度
     .bitmapTransform(new GrayscaleTransformation(this))
     //圆形
     .bitmapTransform(new CropCircleTransformation(this))
     .into(mResultIv);
```
除此之后还有别的一些样式
- ToonFilterTransformation
- SepiaFilterTransformation
- ContrastFilterTransformation
- InvertFilterTransformation
- PixelationFilterTransformation
- SketchFilterTransformation
- SwirlFilterTransformation
- BrightnessFilterTransformation
- KuwaharaFilterTransformation
- VignetteFilterTransformation

## Glide的缓存
缓存是为了减少或者杜绝多的网络请求。为了避免缓存，Glide用了内存缓存和‘外存缓存机制’,并且 提供了相应的方法，完全封装，不需要处理细节。Glide会自动缓存到内存，除非调用.skipMemoryCache( true )。尽管调用了这个，Glide还是会缓存到外存，还有一种情形，就是有一张图片，但是这张图变化非常快，这个时候可能并不想缓存到外存中，就使用.diskCacheStrategy( DiskCacheStrategy.NONE )。如果你两种都不需要，可以两个方法组合着一起使用
### 自定义外存缓存机制
- DiskCacheStrategy.NONE 什么都不缓存
- DiskCacheStrategy.SOURCE 只缓存最高解析图的image
- DiskCacheStrategy.RESULT 缓存最后一次那个image,比如有可能你对image做了转化
- DiskCacheStrategy.ALL image的所有版本都会缓存
```
Glide.with( context ).load( image ).diskCacheStrategy( DiskCacheStrategy.SOURCE ).into( imageViewFile );
```












