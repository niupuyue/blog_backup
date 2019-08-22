---
title: 重拾android路(三) 网络请求
date: 2016-02-19 19:57:43
tags:
  - android
---

做Android开发已经有两年了，这两年来自己有收获，也有很多的挫折。自己是属于比较笨的那种人，所以，我想现在就开始试着把自己之前学过的东西总结一下，那么今天就开始搞一下网络请求的内容。
<!--more-->
先说一下，我从2015年开始做Android开发，先后经历过几个网络框架
1. Android-sync-http
2. volley
3. OkHttp
4. Retrofit(2017年补充)

# 前言
网络请求在现在的开发中越来越常见，我在今年年初的几家面试中，也多次被问到一些关于网络请求框架使用的情况。基本上我们在开发过程中所用的框架，每个公司都有在用的。那么我们有必要说一下，为什么要使用网络框架

## 原因
网络请求框架是将网络请求的相关功能封装成类库，并且对外提供API

> 在没有使用网络框架之前，Android开发中的网络请求是非常繁琐的，因为在Android中主线程不能去请求网络，只能自己去声明一个子线程，完成请求，在得到数据之后，在返回到主线程，然后让主线程去刷新UI，当时用到最多的就是Handler，这个也是非常好用的内容，当时确实帮了很大的忙，以后再说。。。

在我们使用网络请求之后，对于一些重复的，繁琐的操作不用我们一个个的去写，需要使用的时候，直接调用即可，所以可以大大提高我们的工作效率。

> 在使用这些工具之前，最好有一定的网络基础，对于我这个大三的时候挂了计算机网络的人来说，真的是好难，不过后面我也会把这个网络的相关知识总结一下。。。

# 网络请求开源库介绍

[网络请求框架介绍](/assets/android/android-network-framework01.png)

即目前的情况来说，前面两个的使用率已经下降的非常快了，而后面两个的使用率还是非常高的，所以我们把重点放在最后两个部分

# OkHttp简介和引入方式
OkHttp是目前较为流行的网络请求库，特点如下
1. 支持http2，对一台机器的所有请求共享同一个socket
2. 内置连接池，支持连接复用，减少延迟
3. 支持透明的gzip压缩响应体
4. 通过缓存避免重复的请求 
5. 请求失败时自动重试主机的其他ip，自动重定向
6. 灵活的拦截器，行为类似Java EE的Filter或者函数编程中的Map

配置OkHttp非常简单
```
compile 'com.squareup.okhttp3:okhttp:3.4.1'
compile 'com.squareup.okio:okio:1.6.0'
```
添加网络权限
```
    <uses-permission android:name="android.permission.INTERNET"/>
```
一个非常简单的例子
```
public void requestBlog() {
     String url = "https://api.douban.com/v2/movie/in_theaters";
     mHttpClient = new OkHttpClient();
     Request request = new Request.Builder().url(url).build();
         /* okhttp3.Response response = null;*/

         /*response = mHttpClient.newCall(request).execute();*/
     mHttpClient.newCall(request).enqueue(new Callback() {
         @Override
         public void onFailure(Call call, IOException e) {

         }

         @Override
         public void onResponse(Call call, Response response) throws IOException {
             String json = response.body().string();
             Log.d("okHttp", json);
         }
     });
 }
```
## OkHttp的post请求

### post请求提交json数据
```
private void postJson() throws IOException {
    String url = "https://api.douban.com/v2/movie/in_theaters";
    String json = "haha";

    OkHttpClient client = new OkHttpClient();

    RequestBody body = RequestBody.create(JSON, json);
    Request request = new Request.Builder()
            .url(url)
            .post(body)
            .build();
    client.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {

            Log.d(TAG, response.body().string());
        }
    });
}
```
### post请求提交键值对
有时候我们需要吧键值对通过post请求提交给服务器
```
private void post(String url, String json) throws IOException {
     OkHttpClient client = new OkHttpClient();
     RequestBody formBody = new FormBody.Builder()
             .add("name", "liming")
             .add("school", "beida")
             .build();

     Request request = new Request.Builder()
             .url(url)
             .post(formBody)
             .build();

     Call call = client.newCall(request);
     call.enqueue(new Callback() {
         @Override
         public void onFailure(Call call, IOException e) {

         }

         @Override
         public void onResponse(Call call, Response response) throws IOException {
             String str = response.body().string();
             Log.i(TAG, str);

         }

     });
 }
```
### 异步上传文件
上传文件本身也是一个post请求
- 定义上传文件类型
```
public static final MediaType MEDIA_TYPE_MARKDOWN
        = MediaType.parse("text/x-markdown; charset=utf-8");
```
- 将文件上传到服务器
```
private void postFile() {
    OkHttpClient mOkHttpClient = new OkHttpClient();
    File file = new File("/sdcard/demo.txt");
    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, file))
            .build();

    mOkHttpClient.newCall(request).enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            Log.i(TAG, response.body().string());
        }
    });
}
```
同时要添加权限
```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
### 提取响应头
典型的HTTP头像一个map
```
private final OkHttpClient client = new OkHttpClient();

public void run() throws Exception {
    Request request = new Request.Builder()
            .url("https://api.github.com/repos/square/okhttp/issues")
            .header("User-Agent", "OkHttp Headers.java")
            .addHeader("Accept", "application/json; q=0.5")
            .addHeader("Accept", "application/vnd.github.v3+json")
            .build();

    Response response = client.newCall(request).execute();
    if (!response.isSuccessful()) throw new IOException("Unexpected code " + response);

    System.out.println("Server: " + response.header("Server"));
    System.out.println("Date: " + response.header("Date"));
    System.out.println("Vary: " + response.headers("Vary"));
}
```
### post提交string
使用HTTP POST提交请求到服务。这个例子提交了一个markdown文档到web服务，以HTML方式渲染markdown。因为整个请求体都在内存中，因此避免使用此api提交大文档（大于1MB）
```
private void postString() throws IOException {


    OkHttpClient client = new OkHttpClient();


    String postBody = ""
            + "Releases\n"
            + "--------\n"
            + "\n"
            + " * zhangfei\n"
            + " * guanyu\n"
            + " * liubei\n";

    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(RequestBody.create(MEDIA_TYPE_MARKDOWN, postBody))
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }
        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println(response.body().string());
        }
    });
}
```
### post提交流
以流的方式POST提交请求体。请求体的内容由流写入产生。这个例子是流直接写入Okio的BufferedSink。你的程序可能会使用OutputStream，你可以使用BufferedSink.outputStream()来获取
```
public static final MediaType MEDIA_TYPE_MARKDOWN
        = MediaType.parse("text/x-markdown; charset=utf-8");

private void postStream() throws IOException {
    RequestBody requestBody = new RequestBody() {
        @Override
        public MediaType contentType() {
            return MEDIA_TYPE_MARKDOWN;
        }

        @Override
        public void writeTo(BufferedSink sink) throws IOException {
            sink.writeUtf8("Numbers\n");
            sink.writeUtf8("-------\n");
            for (int i = 2; i <= 997; i++) {
                sink.writeUtf8(String.format(" * %s = %s\n", i, factor(i)));
            }
        }

        private String factor(int n) {
            for (int i = 2; i < n; i++) {
                int x = n / i;
                if (x * i == n) return factor(x) + " × " + i;
            }
            return Integer.toString(n);
        }
    };

    Request request = new Request.Builder()
            .url("https://api.github.com/markdown/raw")
            .post(requestBody)
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println(response.body().string());

        }

    });
}
```
### post提交表单
```
private void postForm() {
    OkHttpClient client = new OkHttpClient();

    RequestBody formBody = new FormBody.Builder()
            .add("search", "Jurassic Park")
            .build();

    Request request = new Request.Builder()
            .url("https://en.wikipedia.org/w/index.php")
            .post(formBody)
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println(response.body().string());

        }
    });
}
```
### post请求提交分块请求
MultipartBody 可以构建复杂的请求体，与HTML文件上传形式兼容。多块请求体中每块请求都是一个请求体，可以定义自己的请求头。这些请求头可以用来描述这块请求，例如他的Content-Disposition。如果Content-Length和Content-Type可用的话，他们会被自动添加到请求头中
```
private static final String IMGUR_CLIENT_ID = "...";
private static final MediaType MEDIA_TYPE_PNG = MediaType.parse("image/png");

private void postMultipartBody() {
    OkHttpClient client = new OkHttpClient();


    // Use the imgur image upload API as documented at https://api.imgur.com/endpoints/image
    MultipartBody body = new MultipartBody.Builder("AaB03x")
            .setType(MultipartBody.FORM)
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"title\""),
                    RequestBody.create(null, "Square Logo"))
            .addPart(
                    Headers.of("Content-Disposition", "form-data; name=\"image\""),
                    RequestBody.create(MEDIA_TYPE_PNG, new File("website/static/logo-square.png")))
            .build();

    Request request = new Request.Builder()
            .header("Authorization", "Client-ID " + IMGUR_CLIENT_ID)
            .url("https://api.imgur.com/3/image")
            .post(body)
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println(response.body().string());

        }

    });
}
```
### 相应缓存
为了缓存响应，你需要一个你可以读写的缓存目录，和缓存大小的限制。这个缓存目录应该是私有的，不信任的程序应不能读取缓存内容。 
一个缓存目录同时拥有多个缓存访问是错误的。大多数程序只需要调用一次new OkHttpClient()，在第一次调用时配置好缓存，然后其他地方只需要调用这个实例就可以了。否则两个缓存示例互相干扰，破坏响应缓存，而且有可能会导致程序崩溃。 
响应缓存使用HTTP头作为配置。你可以在请求头中添加Cache-Control: max-stale=3600 ,OkHttp缓存会支持。你的服务通过响应头确定响应缓存多长时间，例如使用Cache-Control: max-age=9600
```
    int cacheSize = 10 * 1024 * 1024; // 10 MiB
    Cache cache = new Cache(cacheDirectory, cacheSize);

    OkHttpClient.Builder builder = new OkHttpClient.Builder();
    builder.cache(cache);
    OkHttpClient client = builder.build();

    Request request = new Request.Builder()
        .url("http://publicobject.com/helloworld.txt")
        .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            String response1Body = response.body().string();
            System.out.println("Response 1 response:          " + response);
            System.out.println("Response 1 cache response:    " + response.cacheResponse()  );
            System.out.println("Response 1 network response:  " + response.networkResponse  ());
        }

    });
```
### 超时
没有响应时使用超时结束call。没有响应的原因可能是客户点链接问题、服务器可用性问题或者这之间的其他东西。OkHttp支持连接，读取和写入超时
```
private void ConfigureTimeouts() {

    OkHttpClient.Builder builder = new OkHttpClient.Builder();
    OkHttpClient client = builder.build();

    client.newBuilder().connectTimeout(10, TimeUnit.SECONDS);
    client.newBuilder().readTimeout(10,TimeUnit.SECONDS);
    client.newBuilder().writeTimeout(10,TimeUnit.SECONDS);

    Request request = new Request.Builder()
            .url("http://httpbin.org/delay/2") // This URL is served with a 2 second delay.
            .build();

    Call call = client.newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            System.out.println("Response completed: " + response);
        }

    });

}
```
## OkHttp的封装
新建一个工具类HttpUtils 
```
private static OkHttpClient client = null;

private HttpUtils() {}

public static OkHttpClient getInstance() {
    if (client == null) {
        synchronized (HttpUtils.class) {
            if (client == null)
                client = new OkHttpClient();
        }
    }
    return client;
}
```
get异步请求
```
    /**
    * Get请求
    *
    * @param url
    * @param callback
    */
public static void doGet(String url, Callback callback) {
    Request request = new Request.Builder()
            .url(url)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```
post请求发送键值对
```
    /**
    * Post请求发送键值对数据
    *
    * @param url
    * @param mapParams
    * @param callback
    */
public static void doPost(String url, Map<String, String> mapParams, Callback callback) {
    FormBody.Builder builder = new FormBody.Builder();
    for (String key : mapParams.keySet()) {
        builder.add(key, mapParams.get(key));
    }
    Request request = new Request.Builder()
            .url(url)
            .post(builder.build())
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```
post请求发送json数据
```
    /**
    * Post请求发送JSON数据
    *
    * @param url
    * @param jsonParams
    * @param callback
    */
public static void doPost(String url, String jsonParams, Callback callback) {
    RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8")
            , jsonParams);
    Request request = new Request.Builder()
            .url(url)
            .post(body)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```
文件上传
```
    /**
    * 上传文件
    *
    * @param url
    * @param pathName
    * @param fileName
    * @param callback
    */
public static void doFile(String url, String pathName, String fileName, Callback callback) {
    //判断文件类型
    MediaType MEDIA_TYPE = MediaType.parse(judgeType(pathName));
    //创建文件参数
    MultipartBody.Builder builder = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart(MEDIA_TYPE.type(), fileName,
                    RequestBody.create(MEDIA_TYPE, new File(pathName)));
    //发出请求参数
    Request request = new Request.Builder()
            .header("Authorization", "Client-ID " + "9199fdef135c122")
            .url(url)
            .post(builder.build())
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}

    /**
    * 根据文件路径判断MediaType
    *
    * @param path
    * @return
    */
private static String judgeType(String path) {
    FileNameMap fileNameMap = URLConnection.getFileNameMap();
    String contentTypeFor = fileNameMap.getContentTypeFor(path);
    if (contentTypeFor == null) {
        contentTypeFor = "application/octet-stream";
    }
    return contentTypeFor;
}
```
下载文件
```
    /**
    * 下载文件
    * @param url
    * @param fileDir
    * @param fileName
    */
public static void downFile(String url, final String fileDir, final String fileName) {
    Request request = new Request.Builder()
            .url(url)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            InputStream is = null;
            byte[] buf = new byte[2048];
            int len = 0;
            FileOutputStream fos = null;
            try {
                is = response.body().byteStream();
                File file = new File(fileDir, fileName);
                fos = new FileOutputStream(file);
                while ((len = is.read(buf)) != -1) {
                    fos.write(buf, 0, len);
                }
                fos.flush();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (is != null) is.close();
                if (fos != null) fos.close();
            }
        }
    });
}
```
如果在下载的过程中需要获取进度，可以通过onResponse回调的内容
```
@Override
public void onResponse(Call call, Response response) throws IOException {
    InputStream is = null;
    byte[] buf = new byte[2048];
    int len = 0;
    FileOutputStream fos = null;
    try {
        is = response.body().byteStream();
        File file = new File(fileDir, fileName);
        fos = new FileOutputStream(file);
        //---增加的代码---
        //计算进度
        long totalSize = response.body().contentLength();
        long sum = 0;
        while ((len = is.read(buf)) != -1) {
            sum += len;
            //progress就是进度值
            int progress = (int) (sum * 1.0f/totalSize * 100);
            //---增加的代码---
            fos.write(buf, 0, len);
        }
        fos.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (is != null) is.close();
        if (fos != null) fos.close();
    }
}
```

> 上面的举例比较简单，因为现在又出了一个新的内容也就是retrofit2，而且他的使用方便度不亚于OkHttp，目前使用的较多，所以我们把重心放在这里

# Retrofit2 

## 简介
Retrofit是一个RESTful的网络请求框架，基于OkHttp
特点：
1. 通过注解的方式配置网络参数
2. 支持同步，异步网络请求
3. 支持多种数据解析和数据序列化
4. 提供对Rxjava的支持

> 注意：其实网络请求的本质工作是有OkHttp完成的，Retrofit只是完成了对数据的封装
>      APP通过Retrofit进行网络请求，其实只是使用了Retrofit接口封装请求参数，Header，URL等信息

## 使用介绍

使用Retrofit的步骤共有7步
1. 添加Retrofit库依赖
2. 创建接受服务器数据返回的类
3. 创建用于描述网络请求接口的类
4. 创建Retrofit实例
5. 创建网络请求接口实例并配置网络请求参数
6. 发送网络请求
7. 处理服务器返回数据

下面对以上的7个步骤，依次分析

### 添加依赖
不仅需要Retrofit，而且需要OkHttp
```
dependencies {
    compile 'com.squareup.retrofit2:retrofit:2.0.2'
    // Retrofit库
    compile 'com.squareup.okhttp3:okhttp:3.1.2'
    // Okhttp库
  }
```
网络请求权限
```
<uses-permission android:name="android.permission.INTERNET"/>
```
### 创建接受服务器数据返回的类  Reception
```
public class Reception {
    ...
    // 根据返回数据的格式和数据解析方式（Json、XML等）定义
    // 下面会在实例进行说明
        }

```
### 创建用于描述网络请求接口的类
Retrofit将Http请求抽象为Java接口：采用注解的方式描述网络请求参数和配置网络请求信息
```
public class Reception {
    ...
    // 根据返回数据的格式和数据解析方式（Json、XML等）定义
    // 下面会在实例进行说明
        }
```
### 注解类型
[](/assets/android/android-network-note.png)
#### 网络请求方法
[](/assets/android/network-way.png)
说明：
1. @GET、@POST、@PUT、@DELETE、@HEAD 
以上方法分别对应 HTTP中的网络请求方式
```
public interface GetRequest_Interface {

    @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
    Call<Translation>  getCall();
    // @GET注解的作用:采用Get方法发送网络请求
    // getCall() = 接收网络请求数据的方法
    // 其中返回类型为Call<*>，*是接收数据的类（即上面定义的Translation类）
}
```
此处特意说明URL的组成：Retrofit把 网络请求的URL 分成了两部分设置
```
// 第1部分：在网络请求接口的注解设置
 @GET("openapi.do?keyfrom=Yanzhikai&key=2032414398&type=data&doctype=json&version=1.1&q=car")
Call<Translation>  getCall();

// 第2部分：在创建Retrofit实例时通过.baseUrl()设置
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") //设置网络请求的Url地址
                .addConverterFactory(GsonConverterFactory.create()) //设置数据解析器
                .build();

// 从上面看出：一个请求的URL可以通过 替换块 和 请求方法的参数 来进行动态的URL更新。
// 替换块是由 被{}包裹起来的字符串构成
// 即：Retrofit支持动态改变网络请求根目录
```
> 网络请求的完整 Url =在创建Retrofit实例时通过.baseUrl()设置 +网络请求接口的注解设置（下面称 “path“ ）
2. @HTTP
作用：替换@GET、@POST、@PUT、@DELETE、@HEAD注解的作用 及 更多功能拓展
具体使用：通过属性method、path、hasBody进行设置
```
public interface GetRequest_Interface {
    /**
     * method：网络请求的方法（区分大小写）
     * path：网络请求地址路径
     * hasBody：是否有请求体
     */
    @HTTP(method = "GET", path = "blog/{id}", hasBody = false)
    Call<ResponseBody> getCall(@Path("id") int id);
    // {id} 表示是一个变量
    // method 的值 retrofit 不会做处理，所以要自行保证准确
}
```
####  标记
1. @FormUrlEncoded
作用：表示发送form-encoded的数据
> 每个键值对需要用@Filed来注解键名，随后的对象需要提供值。

2. @Multipart
作用：表示发送form-encoded的数据（适用于 有文件 上传的场景） 
> 每个键值对需要用@Part来注解键名，随后的对象需要提供值。 
具体使用如下： 
GetRequest_Interface
```
public interface GetRequest_Interface {
        /**
         *表明是一个表单格式的请求（Content-Type:application/x-www-form-urlencoded）
         * Field("username") 表示将后面的 String name中name的取值作为 username 的值
         */
        @POST("/form")
        @FormUrlEncoded
        Call<ResponseBody> testFormUrlEncoded1(@Field("username") String name, @Field("age") int age);

        /**
         * {@link Part} 后面支持三种类型，{@link RequestBody}、{@link okhttp3.MultipartBody.Part} 、任意类型
         * 除 {@link okhttp3.MultipartBody.Part} 以外，其它类型都必须带上表单字段({@link okhttp3.MultipartBody.Part} 中已经包含了表单字段的信息)，
         */
        @POST("/form")
        @Multipart
        Call<ResponseBody> testFileUpload1(@Part("name") RequestBody name, @Part("age") RequestBody age, @Part MultipartBody.Part file);

}

// 具体使用
       GetRequest_Interface service = retrofit.create(GetRequest_Interface.class);
        // @FormUrlEncoded 
        Call<ResponseBody> call1 = service.testFormUrlEncoded1("Carson", 24);

        //  @Multipart
        RequestBody name = RequestBody.create(textType, "Carson");
        RequestBody age = RequestBody.create(textType, "24");

        MultipartBody.Part filePart = MultipartBody.Part.createFormData("file", "test.txt", file);
        Call<ResponseBody> call3 = service.testFileUpload1(name, age, filePart);
```
#### 网络请求参数
[](/assets/android/network-params.png)
1. @Header & @Headers
作用：添加请求头 &添加不固定的请求头
```
// @Header
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)

// @Headers
@Headers("Authorization: authorization")
@GET("user")
Call<User> getUser()

// 以上的效果是一致的。
// 区别在于使用场景和使用方式
// 1. 使用场景：@Header用于添加不固定的请求头，@Headers用于添加固定的请求头
// 2. 使用方式：@Header作用于方法的参数；@Headers作用于方法
```
2. @Body
作用：以 Post方式 传递 自定义数据类型 给服务器
> 如果提交的是一个Map，那么作用相当于 @Field
不过Map要经过 FormBody.Builder 类处理成为符合 Okhttp 格式的表单
```
FormBody.Builder builder = new FormBody.Builder();
builder.add("key","value");
```
3. @Field & @FieldMap
作用：发送 Post请求 时提交请求的表单字段
```
public interface GetRequest_Interface {
        /**
         *表明是一个表单格式的请求（Content-Type:application/x-www-form-urlencoded）
         * <code>Field("username")</code> 表示将后面的 <code>String name</code> 中name的取值作为 username 的值
         */
        @POST("/form")
        @FormUrlEncoded
        Call<ResponseBody> testFormUrlEncoded1(@Field("username") String name, @Field("age") int age);

/**
         * Map的key作为表单的键
         */
        @POST("/form")
        @FormUrlEncoded
        Call<ResponseBody> testFormUrlEncoded2(@FieldMap Map<String, Object> map);

}

// 具体使用
         // @Field
        Call<ResponseBody> call1 = service.testFormUrlEncoded1("Carson", 24);

        // @FieldMap
        // 实现的效果与上面相同，但要传入Map
        Map<String, Object> map = new HashMap<>();
        map.put("username", "Carson");
        map.put("age", 24);
        Call<ResponseBody> call2 = service.testFormUrlEncoded2(map);
```
4. @Part & @PartMap
作用：发送 Post请求 时提交请求的表单字段
> 与@Field的区别：功能相同，但携带的参数类型更加丰富，包括数据流，所以适用于 有文件上传 的场景
```
public interface GetRequest_Interface {

          /**
         * {@link Part} 后面支持三种类型，{@link RequestBody}、{@link okhttp3.MultipartBody.Part} 、任意类型
         * 除 {@link okhttp3.MultipartBody.Part} 以外，其它类型都必须带上表单字段({@link okhttp3.MultipartBody.Part} 中已经包含了表单字段的信息)，
         */
        @POST("/form")
        @Multipart
        Call<ResponseBody> testFileUpload1(@Part("name") RequestBody name, @Part("age") RequestBody age, @Part MultipartBody.Part file);

        /**
         * PartMap 注解支持一个Map作为参数，支持 {@link RequestBody } 类型，
         * 如果有其它的类型，会被{@link retrofit2.Converter}转换，如后面会介绍的 使用{@link com.google.gson.Gson} 的 {@link retrofit2.converter.gson.GsonRequestBodyConverter}
         * 所以{@link MultipartBody.Part} 就不适用了,所以文件只能用<b> @Part MultipartBody.Part </b>
         */
        @POST("/form")
        @Multipart
        Call<ResponseBody> testFileUpload2(@PartMap Map<String, RequestBody> args, @Part MultipartBody.Part file);

        @POST("/form")
        @Multipart
        Call<ResponseBody> testFileUpload3(@PartMap Map<String, RequestBody> args);
}

// 具体使用
 MediaType textType = MediaType.parse("text/plain");
        RequestBody name = RequestBody.create(textType, "Carson");
        RequestBody age = RequestBody.create(textType, "24");
        RequestBody file = RequestBody.create(MediaType.parse("application/octet-stream"), "这里是模拟文件的内容");

        // @Part
        MultipartBody.Part filePart = MultipartBody.Part.createFormData("file", "test.txt", file);
        Call<ResponseBody> call3 = service.testFileUpload1(name, age, filePart);
        ResponseBodyPrinter.printResponseBody(call3);

        // @PartMap
        // 实现和上面同样的效果
        Map<String, RequestBody> fileUpload2Args = new HashMap<>();
        fileUpload2Args.put("name", name);
        fileUpload2Args.put("age", age);
        //这里并不会被当成文件，因为没有文件名(包含在Content-Disposition请求头中)，但上面的 filePart 有
        //fileUpload2Args.put("file", file);
        Call<ResponseBody> call4 = service.testFileUpload2(fileUpload2Args, filePart); //单独处理文件
        ResponseBodyPrinter.printResponseBody(call4);
}
```
5. @Query和@QueryMap
作用：用于 @GET 方法的查询参数（Query = Url 中 ‘?’ 后面的 key-value）
> 如：url = http://www.println.net/?cate=android，其中，Query = cate
```
 @GET("/")    
   Call<String> cate(@Query("cate") String cate);
}

// 其使用方式同 @Field与@FieldMap，这里不作过多描述
```
6. @Path
作用：URL地址的缺省值
```
public interface GetRequest_Interface {

        @GET("users/{user}/repos")
        Call<ResponseBody>  getBlog（@Path("user") String user ）;
        // 访问的API是：https://api.github.com/users/{user}/repos
        // 在发起请求时， {user} 会被替换为方法的第一个参数 user（被@Path注解作用）
    }
```
7. @Url
作用：直接传入一个请求的 URL变量 用于URL设置
```
public interface GetRequest_Interface {

        @GET
        Call<ResponseBody> testUrlAndQuery(@Url String url, @Query("showAll") boolean showAll);
       // 当有URL注解时，@GET传入的URL就可以省略
       // 当GET、POST...HTTP等方法中没有设置Url时，则必须使用 {@link Url}提供

}
```
### 创建Retrofit实例
```
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") // 设置网络请求的Url地址
                .addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava平台
                .build();
```
> 关于数据解析
> Retrofit支持多种数据解析

| 数据解析器 |	Gradle依赖 |
|---------- |:--------------------------------------------------
| Gson |	com.squareup.retrofit2:converter-gson:2.0.2|
| Jackson	| com.squareup.retrofit2:converter-jackson:2.0.2|
| Simple XML |	com.squareup.retrofit2:converter-simplexml:2.0.2|
| Protobuf |	com.squareup.retrofit2:converter-protobuf:2.0.2|
| Moshi	| com.squareup.retrofit2:converter-moshi:2.0.2|
| Wire |	com.squareup.retrofit2:converter-wire:2.0.2|
| Scalars	| com.squareup.retrofit2:converter-scalars:2.0.2|

> 关于网络适配器
> Retrofit支持多种网络请求适配器方式：guava、Java8和rxjava
> 使用时如使用的是 Android 默认的 CallAdapter，则不需要添加网络请求适配器的依赖，否则则需要按照需求进行添加     Retrofit 提供的 CallAdapter

### 创建网络请求接口实例
```
// 创建 网络请求接口 的实例
        GetRequest_Interface request = retrofit.create(GetRequest_Interface.class);

        //对 发送请求 进行封装
        Call<Reception> call = request.getCall();
```
### 发送网络请求
```
/发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                //请求处理,输出结果
                response.body().show();
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<Translation> call, Throwable throwable) {
                System.out.println("连接失败");
            }
        });

// 发送网络请求（同步）
Response<Reception> response = call.execute();
```
### 数据返回处理
在response类中的body()里处理
```
//发送网络请求(异步)
        call.enqueue(new Callback<Translation>() {
            //请求成功时回调
            @Override
            public void onResponse(Call<Translation> call, Response<Translation> response) {
                // 对返回数据进行处理
                response.body().show();
            }

            //请求失败时候的回调
            @Override
            public void onFailure(Call<Translation> call, Throwable throwable) {
                System.out.println("连接失败");
            }
        });

// 发送网络请求（同步）
  Response<Reception> response = call.execute();
  // 对返回数据进行处理
  response.body().show();
```
在开发中我们一般会使用RxJava和Retrofit，以及Gson完成网络请求的基本操作，以后再写

# 参考资料

> 尊重他人劳动成果，也是对自己的尊重

[Android主流网络请求开源库的对比](http://blog.csdn.net/carson_ho/article/details/52171976)
[深入解析OkHttp3](http://blog.csdn.net/u012124438/article/details/54236967)

