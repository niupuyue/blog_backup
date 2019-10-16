---
title: JavaEE 03 JavaWeb基础(Servlet)
date: 2019-10-01 21:37:03
tags:
  - JavaEE
---

内容可能较多，包括Servlet，Jsp，Maven，html，Js等内容

<!--more-->

# Servlet

## 简介
Servlet是运行在web服务器或者是应用服务器上的应用程序。它是作为web浏览器或者其他Http客户端请求和Http服务器上数据库或应用程序之间的中间件。
Servlet的架构图如下所示：
![Servlet架构图](/assets/JavaEE/javaweb_01.png)

Servlet的任务主要包括以下几个内容：
1. 读取客户端发送的显式数据，包括网页上的表单，或者是applet或自定义Http客户端程序的表单
2. 读取客户端发送的隐式数据，包括cookie，媒体类型，浏览器能理解的压缩格式
3. 处理数据并产生结果，这个过程可能要访问数据库，执行RMI或CORBA调用，调用web服务，或直接计算出对应的响应。
4. 发送显式数据(即文档)到客户端，该文档的样式是多样性的，可以是文本文件(HTML,XML),二进制文件(git图像)，Excel
5. 发送隐式HTTP响应到客户端，包括返回的文档类型，设置cookie和缓存参数，以及其他类似任务。

实现Servlet的方式有三种
#### 实现Serlvet接口
```
//Servlet的生命周期:从Servlet被创建到Servlet被销毁的过程
//一次创建，到处服务
//一个Servlet只会有一个对象，服务所有的请求
/*
 * 1.实例化（使用构造方法创建对象）
 * 2.初始化  执行init方法
 * 3.服务     执行service方法
 * 4.销毁    执行destroy方法
 */
public class ServletDemo1 implements Servlet {

    //public ServletDemo1(){}

     //生命周期方法:当Servlet第一次被创建对象时执行该方法,该方法在整个生命周期中只执行一次
    public void init(ServletConfig arg0) throws ServletException {
                System.out.println("=======init=========");
        }

    //生命周期方法:对客户端响应的方法,该方法会被执行多次，每次请求该servlet都会执行该方法
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("hehe");

    }

    //生命周期方法:当Servlet被销毁时执行该方法
    public void destroy() {
        System.out.println("******destroy**********");
    }
//当停止tomcat时也就销毁的servlet。
    public ServletConfig getServletConfig() {

        return null;
    }

    public String getServletInfo() {

        return null;
    }
}
```

#### 继承GenericServlet抽象类
```
public class ServletDemo2 extends GenericServlet {

    @Override
    public void service(ServletRequest arg0, ServletResponse arg1)
            throws ServletException, IOException {
        System.out.println("heihei");

    }
}
```

#### 继承HttpServlet抽象类
```
public class ServletDemo3 extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("haha");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        System.out.println("ee");
        doGet(req,resp);
    }

}
```

一般情况下我们使用第三种方式的情况比较多

#### 三者之间的关系
关于 HttpServlet、GenericServlet 和 Servlet 的关系
对于一个 Servlet 类，我们日常最常用的方法是继承自 HttpServlet 类，提供了 Http 相关的方法，HttpServlet 扩展了 GenericServlet 类，而 GenericServlet 类又实现了 Servlet 类和 ServletConfig 类。

Servlet

Servlet 类提供了五个方法，其中三个生命周期方法和两个普通方法，关于 Servlet 类的方法，不再赘述，我主要补充一下另外两个类的实现思路。

GenericServlet

GenericServlet 是一个抽象类，实现了 Servlet 接口，并且对其中的 init() 和 destroy() 和 service() 提供了默认实现。在 GenericServlet 中，主要完成了以下任务：

- 将 init() 中的 ServletConfig 赋给一个类级变量，可以由 getServletConfig 获得；
- 为 Servlet 所有方法提供默认实现；
- 可以直接调用 ServletConfig 中的方法；

基本结构如下所示：
```
abstract class GenericServlet implements Servlet,ServletConfig{
 
   //GenericServlet通过将ServletConfig赋给类级变量
   private trServletConfig servletConfig;
 
   public void init(ServletConfig servletConfig) throws ServletException {

      this.servletConfig=servletConfig;

      /*自定义init()的原因是：如果子类要初始化必须覆盖父类的init() 而使它无效 这样
       this.servletConfig=servletConfig不起作用 这样就会导致空指针异常 这样如果子类要初始化，
       可以直接覆盖不带参数的init()方法 */
      this.init();
   }
   
   //自定义的init()方法，可以由子类覆盖  
   //init()不是生命周期方法
   public void init(){
  
   }
 
   //实现service()空方法，并且声明为抽象方法，强制子类必须实现service()方法 
   public abstract void service(ServletRequest request,ServletResponse response) 
     throws ServletException,java.io.IOException{
   }
 
   //实现空的destroy方法
   public void destroy(){ }
}
```

HttpServlet

HttpServlet 也是一个抽象类，它进一步继承并封装了 GenericServlet，使得使用更加简单方便，由于是扩展了 Http 的内容，所以还需要使用 HttpServletRequest 和 HttpServletResponse，这两个类分别是 ServletRequest 和 ServletResponse 的子类。代码如下：

```
abstract class HttpServlet extends GenericServlet{
 
   //HttpServlet中的service()
   protected void service(HttpServletRequest httpServletRequest,
                       HttpServletResponse httpServletResponse){
        //该方法通过httpServletRequest.getMethod()判断请求类型调用doGet() doPost()
   }
 
   //必须实现父类的service()方法
   public void service(ServletRequest servletRequest,ServletResponse servletResponse){
      HttpServletRequest request;
      HttpServletResponse response;
      try{
         request=(HttpServletRequest)servletRequest;
         response=(HttpServletResponse)servletResponse;
      }catch(ClassCastException){
         throw new ServletException("non-http request or response");
      }
      //调用service()方法
      this.service(request,response);
   }
}
```

我们可以看到，HttpServlet 中对原始的 Servlet 中的方法都进行了默认的操作，不需要显式的销毁初始化以及 service()，在 HttpServlet 中，自定义了一个新的 service() 方法，其中通过 getMethod() 方法判断请求的类型，从而调用 doGet() 或者 doPost() 处理 get,post 请求，使用者只需要继承 HttpServlet，然后重写 doPost() 或者 doGet() 方法处理请求即可。

我们一般都使用继承 HttpServlet 的方式来定义一个 servlet。

## 实例
Servlet是服务HTTP请求并实现了Servlet接口的类，Web 应用程序开发人员通常编写 Servlet 来扩展 javax.servlet.http.HttpServlet，并实现 Servlet 接口的抽象类专门用来处理 HTTP 请求。

```
package com.paulniu.dao;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

/**
 * author:niupuyue
 * date: 2019/10/9
 * time: 22:51
 * version:
 * desc:
 **/
public class FirstServlet extends HttpServlet {

    private String message;

    public void init() throws ServletException {
        message = "初始化FirstServlet";
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 设置响应内容
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.write("<h1> " + message + " </h1>");
    }
}
public void destory(){

}
```
看一下map
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>FirstServlet</servlet-name>
        <servlet-class>com.paulniu.dao.FirstServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>FirstServlet</servlet-name>
        <url-pattern>/first</url-pattern>
    </servlet-mapping>
</web-app>
```

补充说明：
destory方法被调用后，servlet被销毁，但是并没有立即回收，再次请求时，并没有重新初始化
```
private String message;

@Override
public void init() throws ServletException {
    message = "Hello World , Nect To Meet You: " + System.currentTimeMillis();
    System.out.println("servlet初始化……");
    super.init();
}

@Override
public void doGet(HttpServletRequest req, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html");
    PrintWriter writer = response.getWriter();
    writer.write("<h1>" + message + "</h1>");
    destroy();
}

@Override
public void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    // TODO Auto-generated method stub
    super.doPost(req, resp);
}

@Override
public void destroy() {
    System.out.println("servlet销毁！");
    super.destroy();
}
```
Log日志打印：
```
servlet初始化……
servlet销毁！
2017-7-6 19:48:52 org.apache.catalina.core.StandardContext reload
信息: Reloading Context with name [/myServlet] has started
servlet销毁！
2017-7-6 19:48:52 org.apache.catalina.core.StandardContext reload
信息: Reloading Context with name [/myServlet] is completed
servlet初始化……
servlet销毁！
servlet销毁！
servlet销毁！
servlet销毁！
servlet销毁！
servlet销毁！
servlet销毁！
```

Servlet浏览器访问路径配置的小问题
有两种方式配置路径
- java 类里的注解 —— @WebServlet("/HelloServlet") 对应浏览器路径
```
http://localhost:8080/TomcatTest/HelloServlet
```
- 配置文件（web.xml）里对应的浏览器访问路径：
```
http://localhost:8080/TomcatTest/TomcatTest/HelloServlet
```
两种配置使用一种即可，不然路径重名反而会让tomcat启动不了

## Servlet生命周期
Servlet生命周期可被定义为冲创建知道毁灭的过程，遵循以下过程
1. Servlet通过调用init()方法进行初始化
2. Servlet调用service()方法来处理客户端请求
3. Servlet通过调用destory()方法终止(结束)
4. 最后Servlet是由JVM的垃圾回收器进行垃圾回收的

### init()方法
init方法被设计成只调用一次，在第一次创建该Servlet时被调用，在后续每次用户请求时不再调用，因此它是用一次性初始化
Servlet创建于用户第一次调用欧冠对象与该Servlet的URL时，但是我们也可以指定Servlet在服务器第一次启动时被加载。当用户调用Servlet时，创建一个Servlet实例，每一个用户请求都会产生新的线程，适当的时候交给doGet或者doPost方法。init方法简单的创建或加载一些数据，这些数据将被用于Servlet的整个生命周期
init方法的定义如下
```
public void init() throws ServletException{
    // 初始化代码
}
```
### service()方法
service()方法是执行实际任务的主要方法.Servlet容器(即Web服务器)调用service()方法来处理来自客户端浏览器的请求，并把格式化的响应写回给客户端
每次服务器接收到一个Servlet请求时，服务器会产生一个新的线程并调用服务。service()方法检查HTTP请求类型(GET,POST,PUT,DELETE等)，并在适当的时候调用相应的方法
```
public void service(ServletRequest request, 
                    ServletResponse response) 
      throws ServletException, IOException{
}
```
service()方法由容器调用，service方法在适当的时侯调用doGet,doPost,doPut,doDelete等方法，所以，不用对service()方法做任何动作，只需要根据来自客户端的请求类型重写相应的方法即可
doGet和doPost方法是每次服务请求中最常用的方法，
#### doGet()方法
GET 请求来自于一个 URL 的正常请求，或者来自于一个未指定 METHOD 的 HTML 表单，它由 doGet() 方法处理。
```
public void doGet(HttpServletRequest request,
                  HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```
#### doPost()方法
POST 请求来自于一个特别指定了 METHOD 为 POST 的 HTML 表单，它由 doPost() 方法处理。
```
public void doPost(HttpServletRequest request,
                   HttpServletResponse response)
    throws ServletException, IOException {
    // Servlet 代码
}
```
### destory()方法
destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收。destroy 方法定义如下所示：
```
  public void destroy() {
    // 终止化代码...
  }
```
### 架构图
![Servlet生命周期架构图](/assets/JavaEE/javaweb_02.png)  

## 表单数据
在我们的开发过程中表单是非常常用的一个组件，不管是登录也好，还是搜索，等等。浏览器使用两种方法可将表单信息传递给web服务器，分别是GET和POST
GET方法是默认的从浏览器向web服务器传递信息的方法，它会产生一个很长的字符串，并且出现在浏览器的地址栏中。如果我们想要传递一些比较敏感的信息，不要使用GET方法，并且GET方法对字符串大小也有限制，请求字符串中最多只能有1024个字符
POST方法是向后台传递信息比较可靠的方法。POST方法打包信息的方式与GET方法基本相同。但是POST方法不是吧信息直接放在URL的文本字符串发送，而是将这些信息作为一个单独的消息进行传递。消息以标准输出的形式传到后台程序。

使用Servlet读取表单数据，这些数据会根据不同的情况使用不同的方法自动解析：
1. getParameter():可以调用request.getParameter()方法来获取表单参数的值
2. getParameterValues():如果参数出现一次以上，则调用该方法，并且返回多个值，比如复选框
3. getParameterNames():如果想要得到当前请求中的所有参数的完整列表，则调用该方法

一个例子：
```
public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        String title = "使用get方法读取表单内容";

        String name = req.getParameter("inputEmail");
        System.out.println(name);
        System.out.println(req.getParameter("inputEmail"));
        String password = req.getParameter("inputPassword");
        String remember = req.getParameter("remember");

        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
                "<html> \n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<ul>\n" +
                "  <li><b>姓名</b>："
                + name + "\n" +
                "  <li><b>密码</b>："
                + password + "\n" +
                "  <li><b>是否选择记住我</b>: "
                + remember + "\n" +
                "</ul>\n" +
                "</body></html>");
        System.out.println(getAllParameter(req));
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter out = resp.getWriter();
        String title = "使用post方法读取表单内容";

        String name = new String(req.getParameter("inputName").getBytes("ISO8859-1"), "UTF-8");
        String password = req.getParameter("inputPassword");

        String docType = "<!DOCTYPE html> \n";
        out.println(docType +
                "<html> \n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<ul>\n" +
                "  <li><b>姓名</b>："
                + name + "\n" +
                "  <li><b>密码</b>："
                + password + "\n" +
                "</ul>\n" +
                "</body></html>");
    }

    /**
     * 获取所有的参数
     */
    private HashMap<String, Object> getAllParameter(HttpServletRequest request) {
        HashMap<String, Object> result = new HashMap<>();
        if (request == null) {
            return result;
        }
        Enumeration paramNames = request.getParameterNames();
        while (paramNames.hasMoreElements()) {
            String paramName = (String) paramNames.nextElement();
            String[] paramValues = request.getParameterValues(paramName);
            if (paramValues.length == 1) {
                // 只有一个数据，就去当前值
                String paramValue = paramValues[0];
                if (paramValue.length() == 0) {
                    // 没有数据，不用执行
                } else {
                    // 有数据，将key/value值写入到Map中
                    result.put(paramName, paramValue);
                }
            } else {
                // 读取到了多个值
                String paramVlue = paramValues[paramValues.length - 1];
                result.put(paramName, paramVlue);
            }
        }
        return result;
    }
}
```

网页代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <link href="bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <script src="jquery/jquery.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
    <title>表单</title>
</head>
<body>
<div class="container">
    <form class="form-horizontal" action="login" method="get">
        <div class="control-group">
            <label class="control-label" for="inputEmail">邮箱</label>
            <div class="controls">
                <input type="text" name="inputEmail" id="inputEmail" placeholder="请输入邮箱">
            </div>
        </div>
        <div class="control-group">
            <label class="control-label" for="inputPassword">密码</label>
            <div class="controls">
                <input type="password" name="inputPassword" id="inputPassword" placeholder="请输入密码">
            </div>
        </div>
        <div class="control-group">
            <div class="controls">
                <label class="checkbox">
                    <input name="remember" type="checkbox"> 记住我
                </label>
                <button type="submit" class="btn">登录</button>
            </div>
        </div>
    </form>
</div>
</body>
</html>
```

## 客户端HTTP请求
这里主要介绍在浏览器向后台发送HTTP请求的时候发送的是哪些参数
浏览器端重要的头信息：

|头信息|描述|
|---|---|
|Accept	|这个头信息指定浏览器或其他客户端可以处理的 MIME 类型。值 image/png 或 image/jpeg 是最常见的两种可能值|
|Accept-Charset	|这个头信息指定浏览器可以用来显示信息的字符集。例如 ISO-8859-1。|
|Accept-Encoding	|这个头信息指定浏览器知道如何处理的编码类型。值 gzip 或 compress 是最常见的两种可能值。|
|Accept-Language	|这个头信息指定客户端的首选语言，在这种情况下，Servlet 会产生多种语言的结果。例如，en、en-us、ru 等。|
|Authorization	|这个头信息用于客户端在访问受密码保护的网页时识别自己的身份。|
|Connection	|这个头信息指示客户端是否可以处理持久 HTTP 连接。持久连接允许客户端或其他浏览器通过单个请求来检索多个文件。值 Keep-Alive 意味着使用了持续连接。|
|Content-Length	|这个头信息只适用于 POST 请求，并给出 POST 数据的大小（以字节为单位）。|
|Cookie	|这个头信息把之前发送到浏览器的 cookies 返回到服务器。|
|Host	|这个头信息指定原始的 URL 中的主机和端口。|
|If-Modified-Since	|这个头信息表示只有当页面在指定的日期后已更改时，客户端想要的页面。如果没有新的结果可以使用，服务器会发送一个 304 代码，表示 Not Modified 头信息。|
|If-Unmodified-Since	|这个头信息是 If-Modified-Since 的对立面，它指定只有当文档早于指定日期时，操作才会成功。|
|Referer	|这个头信息指示所指向的 Web 页的 URL。例如，如果您在网页 1，点击一个链接到网页 2，当浏览器请求网页 2 时，网页 1 的 URL 就会包含在 Referer 头信息中。|
|User-Agent	|这个头信息识别发出请求的浏览器或其他客户端，并可以向不同类型的浏览器返回不同的内容。|

读取HTTP头的方法

|方法|描述|
|---|---|
|Cookie[] getCookies()|返回一个数组，包含客户端发送该请求的所有Cookie对象|
|Enumeration getAttributeNames()|返回一个枚举，包含提供给该请求可用的属性名称|
|Enumeration getHeaderNames()|返回一个枚举，包含在该请求中包含的所有的头名|
|Enumeration getParameterNames()|返回一个String对象的枚举，包含在该请求中包含的参数的名称|
|HttpSession getSession()|返回与该请求相关联的当前session会话，如果没有请求session会话，则会创建一个|
|HttpSession getSession(boolean create)|返回与该请求相关的当前HttpSession，或者如果没有当前会话，且创建时真的，则返回新建的session会话|
|Locale getLocale()|基于Accept-Language头，返回客户端接收内容的首选的区域设置|
|Object getAttribute(String name)|一对象形式返回已命名属性的值，如果没有给定名称的属性存在，则返回null|
|ServletInputStream getIputStream()|使用ServletInputStream，以二进制数形式检索请求的主体|
|String getAuthType()|返回用于保护Servlet的身份验证方案的名称，如"BASIC"或"SSL"，如果JSP没有收到保护则返回null|
|String getCharacterEncoding()|返回请求主体中使用的字符编码的名称|
|String getContentType()|返回请求主体MIME类型，如果不知道类型则返回null|
|String getContextPath()|返回指示请求上下文的请求URI部分|
|String getHeader(String name)|以字符串形式返回指定的请求头的值|
|String getMethod()|返回请求的HTTP方法额名称，例如GET，POST，PUT|
|String getParameter(String name)|以字符串形式返回请求参数的值，或者如果参数不存在则返回null|
|String getPathInfo()|当请求发出时，返回与客户端发送的URL相关的任何额外的路径信息|
|String getProtocol()|返回请求协议的名称和版本|
|String getQueryString()|返回包含在路径后的请求URL中查询字符串|
|String getRemoteAddr()|返回发送请求的客户端的互联网协议(IP)地址|
|String getRemoteHost()|返回发送请求的客户端的完全限定名称|
|String getRemoteUser()|如果用户已通过身份验证，则返回发出请求的登录用户，或者如果用户未通过身份验证，则返回null|
|String getRequestURI()|从协议名称知道HTTP请求的第一行的查询字符串中，返回该请求的URL的一部分|
|String getRequestedSessionId()|返回有客户端指定的session会话id|
|String getSevletPath()|返回调用JSP的请求的URL的一部分|
|String[] getParameterValues(String name)|返回一个字符串对象的数组，包含所有给定的请求参数的值，如果参数不存在则返回null|
|boolean isSecure()|返回一个布尔值，指示请求是否使用安全通道，如HTTPS|
|int getCountentLength()|以字节为单位返回请求主体的长度，并提供输入流，如果长度位置则返回-1|
|int getIntHeader(String name)|返回指定的请求头的值为一个int值|
|int getServerPort()|返回接收到这个请求的端口号|
|int getParameterMap()|将参数封装成Map类型|

例子：
```
public class HTTPServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
// 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "HTTP Header 请求实例 ";
        String docType =
                "<!DOCTYPE html> \n";
        out.println(docType +
                "<html>\n" +
                "<head><meta charset=\"utf-8\"><title>" + title + "</title></head>\n"+
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<table width=\"100%\" border=\"1\" align=\"center\">\n" +
                "<tr bgcolor=\"#949494\">\n" +
                "<th>Header 名称</th><th>Header 值</th>\n"+
                "</tr>\n");

        Enumeration headerNames = request.getHeaderNames();

        while(headerNames.hasMoreElements()) {
            String paramName = (String)headerNames.nextElement();
            out.print("<tr><td>" + paramName + "</td>\n");
            String paramValue = request.getHeader(paramName);
            out.println("<td> " + paramValue + "</td></tr>\n");
        }
        out.println("</table>\n</body></html>");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```
网页代码：
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <link href="bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <script src="jquery/jquery.js"></script>
    <script src="bootstrap/js/bootstrap.min.js"></script>
</head>
<body>
<div class="container">
    <a class="btn btn-primary btn-danger" href="http">发送请求</a>
</div>
</body>
</html>
```
![Http请求header](/assets/JavaEE/javaweb_03.png)

## 客户端HTTP响应
和前面的你饿哦让一样，当一个web服务器响应一个HTTP请求时，响应通常包括一个状态行，一些响应报头，一个空行和文档。典型的如下所示
```
HTTP/1.1 200 OK
Content-Type: text/html
Header2: ...
...
HeaderN: ...
  (Blank Line)
<!doctype ...>
<html>
<head>...</head>
<body>
...
</body>
</html>
```

状态行包括HTTP版本(HTTP/1.1)，一个状态码(200)和一个对应于状态码的短消息(OK)

下表总结了从Web服务器端返回到浏览器的最有用的HTTP1.1响应报头，我们可能会在开发中频繁使用:

|头信息	|描述|
|---|---|
|Allow	|这个头信息指定服务器支持的请求方法（GET、POST 等）。|
|Cache-Control	|这个头信息指定响应文档在何种情况下可以安全地缓存。可能的值有：public、private 或 no-cache 等。Public 意味着文档是可缓存，Private 意味着文档是单个用户私用文档，且只能存储在私有（非共享）缓存中，no-cache 意味着文档不应被缓存。|
|Connection	|这个头信息指示浏览器是否使用持久 HTTP 连接。值 close 指示浏览器不使用持久 HTTP 连接，值 keep-alive 意味着使用持久连接。|
|Content-Disposition	|这个头信息可以让您请求浏览器要求用户以给定名称的文件把响应保存到磁盘。|
|Content-Encoding	|在传输过程中，这个头信息指定页面的编码方式。|
|Content-Language	|这个头信息表示文档编写所使用的语言。例如，en、en-us、ru 等。|
|Content-Length	|这个头信息指示响应中的字节数。只有当浏览器使用持久（keep-alive）HTTP 连接时才需要这些信息。|
|Content-Type	|这个头信息提供了响应文档的 MIME（Multipurpose Internet Mail Extension）类型。|
|Expires	|这个头信息指定内容过期的时间，在这之后内容不再被缓存。|
|Last-Modified	|这个头信息指示文档的最后修改时间。然后，客户端可以缓存文件，并在以后的请求中通过 If-Modified-Since 请求头信息提供一个日期。|
|Location	|这个头信息应被包含在所有的带有状态码的响应中。在 300s 内，这会通知浏览器文档的地址。浏览器会自动重新连接到这个位置，并获取新的文档。|
|Refresh	|这个头信息指定浏览器应该如何尽快请求更新的页面。您可以指定页面刷新的秒数。|
|Retry-After	|这个头信息可以与 503（Service Unavailable 服务不可用）响应配合使用，这会告诉客户端多久就可以重复它的请求。|
|Set-Cookie	|这个头信息指定一个与页面关联的 cookie。|

设置HTTP响应报头的方法
这些方法可用于在Servlet程序中设置HTTP响应报头，通过HttpServletResponse对象可用

|方法|描述|
|---|---|
|String encodeRedorectURL(String url)|为sendRedirect方法中使用的指定的URL进行编码，或者如果编码不是必须的，则返回URL未改变|
|String encodeURL(String url)|对包含session会话ID的指定URL编码，或者如果编码不是必须的，则返回URL未改变|
|boolean containsHeader(String name)|返回一个布尔值，指示是否已经设置已命名的相应报头|
|boolean isCommitted()|返回一个布尔值，指示响应是否已经提交|
|void addCookie(Cookie coookie)|把置顶的cookie添加到响应|
|void addHeader(String name,String value)|添加一个带有给定的名称和值的响应报头|
|void addDateHeader(String name,long date)|添加一个带有给定名称和日期值的响应报头|
|void addIntHeader(String name,int value)|添加一个带有给定名称和整数值的响应报头|
|void flushBuffer()|强制任何在缓冲区中的内容被写入到客户端|
|void reset()|清除缓冲区中存在的任何数据，包括状态码和头|
|void resetBuffer()|清除响应中基础缓存区的内容，不清除状态码和头|
|void sendError(int sc)|使用指定的状态码发送错误响应到客户端，并清除缓冲区|
|void sendError(int sc,String msg)|使用指定的状态发送错误响应到客户端|
|void sendRedirect(String location)|使用指定的重定向位置URL发送临时重定向响应到客户端|
|void setBufferSize(int size)|为响应主体设置首选的缓冲区大小|
|void setCharacterEncoding(String charset)|设置被发送到客户端的响应的字符编码(MIME字符集),例如UTF-8|
|void setContentLength(int len)|设置在HTTPServlet响应中内容主体的长度，该方法设置HTTP content-Length头|
|void setContentType(String type)|如果响应还未被提交，设置被发送到客户端的响应内容的类型|
|void setDateHeader(String name,long date)|设置一个带有给定名称和日期值的响应报头|
|void setHeader(String name,String value)|设置一个带有给定名称和值的响应报头|
|void setIntHeader(String name,int value)|设置一个带有给定的名称和整数值的响应报头|
|void setLocale(Locale loc)|如果响应还未被提交，设置响应的区域|
|void setStatus(int sc)|为该响应设置状态码|

一个例子：
```
public class Refresh extends HttpServlet {

    // 处理 GET 方法请求的方法
      public void doGet(HttpServletRequest request,
                        HttpServletResponse response)
                throws ServletException, IOException
      {
          // 设置刷新自动加载时间为 5 秒
          response.setIntHeader("Refresh", 5);
          // 设置响应内容类型
          response.setContentType("text/html;charset=UTF-8");
         
          //使用默认时区和语言环境获得一个日历  
          Calendar cale = Calendar.getInstance();  
          //将Calendar类型转换成Date类型  
          Date tasktime=cale.getTime();  
          //设置日期输出的格式  
          SimpleDateFormat df=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");  
          //格式化输出  
          String nowTime = df.format(tasktime);
          PrintWriter out = response.getWriter();
          String title = "自动刷新 Header 设置 - 菜鸟教程实例";
          String docType =
          "<!DOCTYPE html>\n";
          out.println(docType +
            "<html>\n" +
            "<head><title>" + title + "</title></head>\n"+
            "<body bgcolor=\"#f0f0f0\">\n" +
            "<h1 align=\"center\">" + title + "</h1>\n" +
            "<p>当前时间是：" + nowTime + "</p>\n");
      }
      // 处理 POST 方法请求的方法
      public void doPost(HttpServletRequest request,
                         HttpServletResponse response)
          throws ServletException, IOException {
         doGet(request, response);
      }
}
```

## 过滤器
Servlet可以动态的拦截请求和响应，用于变化和使用包含在其中的信息
Servlet过滤器是可以用于Servlet编程的Java类，主要目的有以下几个
- 客户端的请求访问资源之前，拦截这些请求
- 服务器的响应范送给客户端之前，处理这些响应

Servlet过滤器的作用：
1. 查询请求并作出相应的行为
2. 阻塞请求-响应对，使其不能进一步传递
3. 修改请求头部和数据，用户可以提供自定义请求
4. 修改响应的头部和数据，用户可以通过提供定制的响应版本来实现
5. 与外部资源进行交互

根据规范建议的各类型过滤器：
1. 身份验证过滤器（Authentication Filters）。
2. 数据压缩过滤器（Data compression Filters）。
3. 加密过滤器（Encryption Filters）。
4. 触发资源访问事件过滤器。
5. 图像转换过滤器（Image Conversion Filters）。
6. 日志记录和审核过滤器（Logging and Auditing Filters）。
7. MIME-TYPE 链过滤器（MIME-TYPE Chain Filters）。
8. 标记化过滤器（Tokenizing Filters）。
9. XSL/T 过滤器（XSL/T Filters），转换 XML 内容

过滤器通过web部署文件(web.xml)中的XML标签，然后映射到应用程序的部署描述符中的 Servlet 名称或 URL 模式
当web容器启动web应用程序时，他会在部署描述符中声明的每一个过滤器创建一个实例。
Filter执行的顺序鱼仔web.xml文件中声明的顺序是一致的。一般我们将Filter的声明放在Servlet之前

Filter的常用方法如下：

|方法|描述|
|---|---|
|public void doFilter(ServletRequest request,ServletResponse response)|该方法完成实际的过滤操作，当客户端请求方法和过滤器设置的URL匹配时，Servlet容器会优先调用过滤器中的doFilter方法，FilterChain用户访问后续过滤器|
|public void init(FilterConfig config)|web应用程序启动时，web服务器将创建Filter对象实例，并调用init方法，读取web.xml中的配置信息，完成对象初始化。filter对象只会创建一次，init方法也只会执行一次，开发人员通过init方法获取参数，可获取当前代表filter配置信息的FilterConfig对象|
|public void destory()|Servlet在销毁过滤器实例之前调用的方法，在该方法中释放资源|

Servlet过滤器创建的步骤：
1. 实现Filter接口
2. 实现init方法，读取过滤器初始化参数
3. 实现doFilter方法，完成对请求或过滤的响应
4. 调用FilterChain接口对象的doFilter方法，向后续过滤器传递请求或响应
5. 销毁过滤器

例子：
web.xml文件
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--请求url日志记录过滤器-->
    <filter>
        <filter-name>logfilter</filter-name>
        <filter-class>com.paulniu.filter.LogFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>logfilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--编码过滤器-->
    <filter>
        <filter-name>encodingfilter</filter-name>
        <filter-class>com.paulniu.filter.EncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingfilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--测试Servlet-->
    <servlet>
        <servlet-name>DemoServlet</servlet-name>
        <servlet-class>com.paulniu.servlet.DemoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DemoServlet</servlet-name>
        <url-pattern>/demo</url-pattern>
    </servlet-mapping>
    
</web-app>
```
日志过滤器
```
public class LogFilter implements Filter {

    private FilterConfig config;

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("paulniu:start do the logging filter");
        this.config = config;
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("paulniu:before the log filter");
        // 将请求转换成HttpServletRequest
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        // 将响应转换成HttpServletResponse
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        // 记录日志
        System.out.println("Log Filter 已经截取到用户请求的地址=" + request.getServletPath());
        try {
            // Filter 只链式处理，请求依然转发到目的地址
            filterChain.doFilter(request, response);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println("paulniu:after the log filter");
    }

    /**
     * 销毁日志过滤器
     */
    @Override
    public void destroy() {
        this.config = null;
        System.out.println("paulniu:end do the logging filter");
    }
}
```
编码过滤器
```
public class EncodingFilter implements Filter {
    private String encoding;
    private HashMap<String, String> params = new HashMap<>();

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // 项目开始时就已经进行读取
        System.out.println("paulniu:before do the encoding filter");
        encoding = filterConfig.getInitParameter("encoding");
        for (Enumeration<?> e = filterConfig.getInitParameterNames(); e.hasMoreElements(); ) {
            String name = (String) e.nextElement();
            String value = filterConfig.getInitParameter(name);
            params.put(name, value);
        }
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        System.out.println("paulniu:before encoding " + encoding + " filter!");
        servletRequest.setCharacterEncoding(encoding);
        filterChain.doFilter(servletRequest, servletResponse);
        System.out.println("after encoding " + encoding + " filter！");
        System.err.println("----------------------------------------");
    }

    @Override
    public void destroy() {
        System.out.println("paulniu:end do the encoding filter");
        params = null;
        encoding = null;
    }
}
```
Servlet测试
```
@WebServlet(name = "DemoServlet")
public class DemoServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```
测试结果：
![过滤器demo](/assets/JavaEE/javaweb_04.png)

### Servlet监听器
Servlet监听器用于监听一些重要的事情发生，监听器对象可以在事情发生前，发生后做一些必要的处理

ServletContextListener：用于监听web应用的启动和销毁事件，监听器类需要实现ServletContextListener接口
```
public class QuartzListener implements ServletContextListener {  
  
    private Logger logger = LoggerFactory.getLogger(QuartzListener.class);  
  
    public void contextInitialized(ServletContextEvent sce) {  
  
    }  
  
    /** 
     *在服务器停止运行的时候停止所有的定时任务 
     */  
    @SuppressWarnings("unchecked")  
    public void contextDestroyed(ServletContextEvent arg0) {  
        try {  
            Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();  
            List<JobExecutionContext> jobList = scheduler.getCurrentlyExecutingJobs();  
            for (JobExecutionContext jobContext : jobList) {  
                Job job = jobContext.getJobInstance();  
                if (job instanceof InterruptableJob) {  
                    ((InterruptableJob) job).interrupt();  
                }  
            }  
            scheduler.shutdown();  
        } catch (SchedulerException e) {  
            logger.error("shut down scheduler happened error", e);  
        }  
    }  
}
```

ServletContextAttributeListener:用于监听Web应用属性改变的事件，包括增加属性，删除属性，修改属性，监听器类需要实现ServletContextAttributeListener接口

HttpSessionListener：用于监听Session对象的创建和销毁，监听器类需要实现HttpSessionListener接口或者HttpSessionActivationListener接口
```
public class SessionListener implements HttpSessionListener {  
  
    @Override  
    public void sessionCreated(HttpSessionEvent arg0) {  
  
    }  
  
    @Override  
    public void sessionDestroyed(HttpSessionEvent event) {  
        HttpSession session = event.getSession();  
        User user = (BrsSession) session.getAttribute("currUser");  
        if (user != null) {  
            //TODO something  
        }  
    }  
  
}  
```

HttpSessionActivationListener:用于监听Session对象的钝化/活化事件，监听器类需要实现javax.servlet.http.HttpSessionListener接口或者javax.servlet.http.HttpSessionActivationListener接口，或者两个都实现。

HttpSessionAttributeListener：用于监听Session对象属性的改变事件，监听类需要实现

部署：监听器的部署在web.xml文件中配置，在配置文件中，它的位置应该在过滤器的后面Servlet的前面
```
<!-- Quartz监听器 -->  
<listener>  
    <listener-class>  
        com.flyer.lisenter.QuartzListener  
    </listener-class>  
</listener> 
```

## Servlet异常处理
当一个Servlet抛出一个异常，Web容器通过exception-type元素的web.xml中搜索和抛出异常类型相匹配的设置
必须在 web.xml 中使用 error-page 元素来指定对特定异常 或 HTTP 状态码 作出相应的 Servlet 调用

web.xml配置
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!--请求url日志记录过滤器-->
    <filter>
        <filter-name>logfilter</filter-name>
        <filter-class>com.paulniu.filter.LogFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>logfilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--编码过滤器-->
    <filter>
        <filter-name>encodingfilter</filter-name>
        <filter-class>com.paulniu.filter.EncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingfilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--error-code相关页面-->
    <error-page>
        <error-code>404</error-code>
        <location>/ErrorHandler</location>
    </error-page>
    <error-page>
        <error-code>403</error-code>
        <location>/ErrorHandler</location>
    </error-page>

    <!--exception-type相关页面-->
    <error-page>
        <exception-type>javax.servlet.ServletException</exception-type>
        <location>/ErrorHandler</location>
    </error-page>
    <error-page>
        <exception-type>java.io.IOException</exception-type>
        <location>/ErrorHandler</location>
    </error-page>

    <!--测试Servlet-->
    <servlet>
        <servlet-name>DemoServlet</servlet-name>
        <servlet-class>com.paulniu.servlet.DemoServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>DemoServlet</servlet-name>
        <url-pattern>/demo</url-pattern>
    </servlet-mapping>

    <!--错误处理Servlet-->
    <servlet>
        <servlet-name>ErrorHandler</servlet-name>
        <servlet-class>com.paulniu.servlet.ErrorHandler</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>ErrorHandler</servlet-name>
        <url-pattern>/ErrorHandler</url-pattern>
    </servlet-mapping>

</web-app>
```

异常处理页面：
```
public class ErrorHandler extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Throwable throwable = (Throwable)
                request.getAttribute("javax.servlet.error.exception");
        Integer statusCode = (Integer)
                request.getAttribute("javax.servlet.error.status_code");
        String servletName = (String)
                request.getAttribute("javax.servlet.error.servlet_name");
        if (servletName == null) {
            servletName = "Unknown";
        }
        String requestUri = (String)
                request.getAttribute("javax.servlet.error.request_uri");
        if (requestUri == null) {
            requestUri = "Unknown";
        }
        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");

        PrintWriter out = response.getWriter();
        String title = "JavaWeb Error/Exception 信息";

        String docType = "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n");
        out.println("<h1>JavaWeb异常信息实例演示</h1>");
        if (throwable == null && statusCode == null) {
            out.println("<h2>错误信息丢失</h2>");
            out.println("请返回 <a href=\"" +
                    response.encodeURL("http://localhost:8080/") +
                    "\">主页</a>。");
        } else if (statusCode != null) {
            out.println("错误代码 : " + statusCode);
        } else {
            out.println("<h2>错误信息</h2>");
            out.println("Servlet Name : " + servletName +
                    "</br></br>");
            out.println("异常类型 : " +
                    throwable.getClass().getName() +
                    "</br></br>");
            out.println("请求 URI: " + requestUri +
                    "<br><br>");
            out.println("异常信息: " +
                    throwable.getMessage());
        }
        out.println("</body>");
        out.println("</html>");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

测试结果：
在浏览器中输入错误的地址
![Servlet异常](/assets/JavaEE/javaweb_05.png)

## Servlet Cookie处理

Cookie是存储在客户端上的文本文件，由服务器生成，发送给User-Agent，浏览器会将Cookie的key/value保存到某个目录下的文本文件内，下次请求同一网站时，发送该Cookie给服务器。Cookie的key和value都是由服务器自己规定的，对于JSP而言可以直接写入JSESSIONID用于标记一个回话的Session，这样服务器可以知道该用户是否为合法用户或是否需要重新登录，服务器可以设置和读取Cookie中的信息，借此维护用户在服务器中的状态

Cookie的原理：
1. 首先浏览器想服务器发送请求
2. 服务器会根据需求生成一个Cookie对象，并且把数据保存在该对象中
3. 然后把Cookie对象放在响应头中，并发送会给浏览器
4. 浏览器收到服务器响应之后，提出该Cookie保存在浏览器端
5. 当浏览器再次访问那个服务器，会把这个Cookie放在请求头并发送给服务器
6. 服务器从请求头中提取Cookie，判别数据执行相应的操作

### Cookie剖析
Cookie通常设置在HTTP头信息中(虽然JavaScript也可以直接在浏览器设置一个Cookie)。设置Cookie的Servlet会发送如下的头信息
![图片是我拷贝的](/assets/JavaEE/javaweb_06.png)

如图所示，Set-Cookie头包含了一个名称值对，一个GMT日期，一个路径和一个域。名称和值会被URL编码。expires字段是一个指令，告诉浏览器在给定的时间和日期之后忘记Cookie
如果浏览器被配置为存储Cookie，他将会保留此信息直到到期日期。如果用户的浏览器指向任何匹配该Cookie的路径和域的页面，他会重新发送Cookie到服务器，浏览器的头信息可能如下：
![图片是我拷贝的](/assets/JavaEE/javaweb_07.png)

Cookie方法：

|方法|描述|
|---|---|
|public void setDomain(String pattern)|该方法设置cookie使用的域|
|public String getDomain()|该方法获取cookie适用的域|
|public void setMaxAge(int expiry)|设置cookie过期的时间，以秒为单位，如果不设置，则cookie会在当前Session会话中持续有效|
|public int getMaxAge()|该方法返回cookie的最大生存周期，以秒为单位，默认情况下为-1，表示cookie会持续下去，知道浏览器关闭|
|public String getName()|该方法返回cookie名称，名称在创建后不能改变|
|public void setValue(String newValue)|该方法设置与cookie关联的值|
|public String getValue()|方法获取与cookie相关联的值|
|public void setPath(String uri)|该方法设置cookie使用的路径，如果不指定路径，与当前页面相同目录下的(包括子目录)所有URL都会返回cookie|
|public String getPath()|该方法获取Cookie使用的路径|
|public void setSecure(boolean flag)|该方法设置布尔值，表示cookie是否应该只在加密的(即SSL)连接上发送|
|public void setComment(String purpose)|设置cookie的注解，该注解在浏览器向用户呈现cookie时非常有用|
|public String getComment()|获取cookie的注释，如果cookie没有注释则返回null|

### Cookie的使用
创建一个Cookie
```
public class CookieServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Cookie cookie = new Cookie("name", "value");
        resp.addCookie(cookie);

        // 得到浏览器传递的cookie数据
        Cookie[] cookies = req.getCookies();
        if (cookies != null) {
            for (Cookie c : cookies) {
                String name = c.getName();
                String value = c.getValue();
                System.out.println(name + "=" + value);
            }
        } else {
            System.out.println("没有Cookie信息！");
        }
    }
}
```

运行结果：
![Cookie使用](/assets/JavaEE/javaweb_08.png)

这里我们可以模拟一个登陆的例子，因为里面需要使用到Session的内容，所以我们等Session的内容完成之后一块写一个例子

## Servlet中的Session

HTTP是一种无状态的协议，这意味着每次客户端检索网页时，客户端打开一个单独的链接到服务器，服务器会自动不保留之前客户端的任何请求记录。但是仍然以下面三种方式维持客户端和服务器之间的session回话
- Cookie:一个web服务器可以分配一个唯一的SessionID作为每一个客户端的Cookie，对于客户端的后续请求，可以使用接收到的Cookie来识别。(但是很多浏览器可能不支持Cookie，所以这个并不是最优的解决方案)
- 隐藏的表单字段：一个Web服务器可以发送一个隐藏的HTML表单字段，以及唯一的一个Session回话ID，如
```
<input type="hide" name="sessionid" value="123456">
```
,这意味着当表单被提交的时候，指定的名称和值会自动包含在GET和POST请求中。每当浏览器发送请求时，session_id值可以可以用于保持不同浏览器的追踪。这个方式相对而言比较合理，但是对于一些超链接标签，我们没有发送网络请求，所以不会提交表单，所以也不是非常好的解决办法
- URL重定向：我们可以为每一个URL的末尾追加一些额外的数据来标识session会话。服务器会将传递过来session标识和已存储的相关session会话数据相关联，如：http://www.paulniu.com/login.jsp;sessionid=123456 URL重定向是一种更好的方式来维持Session会话，他在浏览器不支持Cookie时可以很好的完成维持Session会话的工作，缺点是会动态的生成每个URL来为页面分配一个sessionid，即使是很简单的静态页面也是如此。

除了上面的三种方式，Servlet还提供了HttpSession接口，该接口提供了一种跨多个页面请求或访问网站时识别用户以及存储用户相关信息的方法。
Servlet 容器使用这个接口来创建一个 HTTP 客户端和 HTTP 服务器之间的 session 会话。会话持续一个指定的时间段，跨多个连接或页面请求。
我们会通过调用 HttpServletRequest 的公共方法 getSession() 来获取 HttpSession 对象
```
HttpSession session = request.getSession();
```

Session常用方法：

|方法|描述|
|---|---|
|public Object getAttribute(String name)|该方法返回在该Session会话中具有指定名称的对象，如果没有指定名称的对象，返回null|
|public Enumeration getAttributeNames()|该方法返回String类型对象的枚举，String对象包含所有绑定到该Session会话的对象的名称|
|public long getCreationTime()|该方法返回Session会话被创建的时间,以毫秒为单位|
|public String getId()|该方法返回一个包含Session会话唯一标识符的字符串|
|public long getLastAccessedTime()|该方法返回客户端最后一次发送与Session会话相关的请求的时间，以毫秒为单位|
|public int getMaxInactiveInterval()|该方法返回Servlet容器在客户端访问时保持session会话打开的最大时间间隔，以秒为单位|
|public void invalindate()|指示该Session会话无效，并解除绑定到它上面的任何对象|
|public boolean isNew()|如果客户端还不知道该Session会话，或者客户端选择不参与该Session会话，返回true|
|public void removeAttribute(String name)|从Session会话中移除指定名称对象|
|public void setAttribute(String name,Object value)|该方法使用指定的名称绑定一个对象到该Session会话|
|public void setMaxInactiveInterval(int interval)|该方法在Servlet中指示该Session会话无效之前，指定客户端请求之前的时间，以秒为单位|

Session追踪实例：
```
public class SessionTrackServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse response) throws ServletException, IOException {
        // 如果不存在Session会话，则创建一个Session会话
        HttpSession session = req.getSession(true);
        // 获取Session创建的时间
        Date createTime = new Date(session.getCreationTime());
        // 获取该网页最后一次访问的时间
        Date lastAccessTime = new Date(session.getLastAccessedTime());

        // 设置日期输出格式
        SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

        String title = "Servlet session 实例";
        Integer visitCount = new Integer(0);
        String visitCountKey = new String("visitCount");
        String userIDKey = new String("userID");
        String userID = new String("paulniu");
        if (session.getAttribute(visitCountKey) == null) {
            session.setAttribute(visitCountKey, new Integer(0));
        }

        // 检查网页上是否有新的访问者
        if (session.isNew()) {
            title = "Servlet Session 实例";
            session.setAttribute(userIDKey, userID);
        } else {
            visitCount = (Integer) session.getAttribute(visitCountKey);
            visitCount = visitCount + 1;
            userID = (String) session.getAttribute(userIDKey);
        }
        session.setAttribute(visitCountKey, visitCount);

        // 设置响应内容类型
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();

        String docType = "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<h2 align=\"center\">Session 信息</h2>\n" +
                "<table border=\"1\" align=\"center\">\n" +
                "<tr bgcolor=\"#949494\">\n" +
                "  <th>Session 信息</th><th>值</th></tr>\n" +
                "<tr>\n" +
                "  <td>id</td>\n" +
                "  <td>" + session.getId() + "</td></tr>\n" +
                "<tr>\n" +
                "  <td>创建时间</td>\n" +
                "  <td>" + df.format(createTime) +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>最后访问时间</td>\n" +
                "  <td>" + df.format(lastAccessTime) +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>用户 ID</td>\n" +
                "  <td>" + userID +
                "  </td></tr>\n" +
                "<tr>\n" +
                "  <td>访问统计：</td>\n" +
                "  <td>" + visitCount + "</td></tr>\n" +
                "</table>\n" +
                "</body></html>");
    }
}
```
处理结果：
![session追踪](/assets/JavaEE/javaweb_09.png)

## 登录demo
使用Cookie登录
```
public class LoginCookieServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    public LoginCookieServlet(){

    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
        // 获取表达中的username和password
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        // 定义一个名为username
        Cookie cookie = new Cookie(username,password);
        cookie.setPath("");
        cookie.setComment("这是一个cookie");
        response.addCookie(cookie);
        request.getRequestDispatcher("loginSuccess.jsp").forward(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.getWriter().append("serverd ad:").append(request.getServletPath());
    }
}
```
jsp页面
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录demo</title>
</head>
<body>
<form action="login" method="post">
    <input type="text" name="username" />
    <input type="password" name="password">
    <input type="submit" value="登录">
</form>
</body>
</html>
```

![登陆成功之后将cookie对象发送到浏览器中](/assets/JavaEE/javaweb_10.png)

使用Session登录
```
public class LoginSessionServlet extends HttpServlet {

    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().append("served at: ").append(req.getServletPath());
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String username = req.getParameter("username");
        String password = req.getParameter("password");
        User user = new User(username,password);

        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateTime = dateFormat.format(new Date());
        HttpSession session = req.getSession();
        session.setAttribute("user",user);
        session.setAttribute("loginTime",dateTime);
        session.setAttribute("sessionId",session.getId());
        // 请求转发
        req.getRequestDispatcher("loginSuccess.jsp").forward(req,resp);
    }
}
```
jsp页面：
```
// login.jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录demo</title>
</head>
<body>
<form action="loginsession" method="post">
    <input type="text" name="username"/>
    <input type="password" name="password">
    <input type="submit" value="登录">
</form>
</body>
</html>

// loginSuccess.jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@page import="java.text.DateFormat" %>
<jsp:directive.page import="com.paulniu.bean.User"/>
<% User user = (User) session.getAttribute("user");
    String dateTime = (String) session.getAttribute("loginTime");
    String sessionId = (String) session.getAttribute("sessionId");
%>
<html>
<head>
    <title>登录结果页面</title>
</head>
<body>
Hello,this is an index page.<p>
    用户名：<%=user.getUserName()%><p>
    密码：<%=user.getPassword()%><p>
    登陆时间：<%=dateTime %><p>
    SessionId：<%=sessionId %><p>
</body>
</html>
```
User对象
```
public class User {

    private String userName;
    private String password;

    public User(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }

    public User() {
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    @Override
    public String toString() {
        return "User{" +
                "userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                '}';
    }
}
```

在没有提交表单之前
![原始SessionID](/assets/JavaEE/javaweb_11.png)

表单提交之后
![从服务器端返回的SessionID](/assets/JavaEE/javaweb_12.png)

## Servlet 数据库
准备测试数据
```
CREATE TABLE `websites` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` char(20) NOT NULL DEFAULT '' COMMENT '站点名称',
  `url` varchar(255) NOT NULL DEFAULT '',
  `alexa` int(11) NOT NULL DEFAULT '0' COMMENT 'Alexa 排名',
  `country` char(10) NOT NULL DEFAULT '' COMMENT '国家',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8;

INSERT INTO `websites` VALUES ('1', 'Google', 'https://www.google.cm/', '1', 'USA'), ('2', '淘宝', 'https://www.taobao.com/', '13', 'CN'), ('3', '菜鸟教程', 'http://www.runoob.com', '5892', ''), ('4', '微博', 'http://weibo.com/', '20', 'CN'), ('5', 'Facebook', 'https://www.facebook.com/', '3', 'USA');
```

访问数据库
```
public class DataBaseServlet extends HttpServlet {

    private static final long serialVersionUID = 1l;

    //JDBC驱动名以及数据库URL
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://localhost:3306/javawebdb";

    // 数据库的用户名和密码
    static final String USER = "root";
    static final String PASS = "root";

    public DataBaseServlet() {

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Connection conn = null;
        Statement stmt = null;
        // 设置响应内容类型
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        String title = "Servlet Mysql 测试";
        String docType = "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n");
        try {
            // 注册 JDBC 驱动器
            Class.forName("com.mysql.jdbc.Driver");

            // 打开一个连接
            conn = DriverManager.getConnection(DB_URL, USER, PASS);

            // 执行 SQL 查询
            stmt = conn.createStatement();
            String sql;
            sql = "SELECT id, name, url FROM websites";
            ResultSet rs = stmt.executeQuery(sql);

            // 展开结果集数据库
            while (rs.next()) {
                // 通过字段检索
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String url = rs.getString("url");

                // 输出数据
                out.println("ID: " + id);
                out.println(", 站点名称: " + name);
                out.println(", 站点 URL: " + url);
                out.println("<br />");
            }
            out.println("</body></html>");

            // 完成后关闭
            rs.close();
            stmt.close();
            conn.close();
        } catch (SQLException se) {
            // 处理 JDBC 错误
            se.printStackTrace();
        } catch (Exception e) {
            // 处理 Class.forName 错误
            e.printStackTrace();
        } finally {
            // 最后是用于关闭资源的块
            try {
                if (stmt != null)
                    stmt.close();
            } catch (SQLException se2) {
            }
            try {
                if (conn != null)
                    conn.close();
            } catch (SQLException se) {
                se.printStackTrace();
            }
        }
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

运行结果
![运行结果](/assets/JavaEE/javaweb_13.png)

如果向数据库中插入数据
```
//编写预处理 SQL 语句
String sql= "INSERT INTO websites1 VALUES(?,?,?,?,?)";

//实例化 PreparedStatement
ps = conn.prepareStatement(sql);

//传入参数，这里的参数来自于一个带有表单的jsp文件，很容易实现
ps.setString(1, request.getParameter("id"));
ps.setString(2, request.getParameter("name"));
ps.setString(3, request.getParameter("url"));
ps.setString(4, request.getParameter("alexa"));
ps.setString(5, request.getParameter("country"));

//执行数据库更新操作，不需要SQL语句
ps.executeUpdate();
sql = "SELECT id, name, url FROM websites1";
ps = conn.prepareStatement(sql);

//获取查询结果
ResultSet rs = ps.executeQuery();
```

## Servlet文件的上传和下载
文件上传：
```
public class UploadServlet extends HttpServlet {

    private static final long serialVersionUID = 1l;

    // 上传文件存储的路径
    private static final String UPLOAD_DIRECTORY = "upload";
    // 上传配置
    private static final int MEMORY_THRESHOLD = 1024 * 1024 * 3;
    private static final int MAX_FILE_SIZE = 1024 * 1024 * 40;
    private static final int MAX_REQUEST_SIZE = 1024 * 1024 * 50;

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 上传数据及保存文件
        // 检测是否为多媒体上传
        if (!ServletFileUpload.isMultipartContent(request)){
            // 如果不是则停止
            PrintWriter writer = response.getWriter();
            writer.println("Error: 表单必须包含 enctype=multipart/form-data");
            writer.flush();
            return;
        }
        // 配置参数
        DiskFileItemFactory factory = new DiskFileItemFactory();
        // 设置内存临界值 - 超过该值之后将产生临时文件并存储于临时目录中
        factory.setSizeThreshold(MEMORY_THRESHOLD);
        // 设置临时存储目录
        factory.setRepository(new File(System.getProperty("java.io.tmpdir")));

        ServletFileUpload upload = new ServletFileUpload(factory);

        // 设置最大文件上传值
        upload.setFileSizeMax(MAX_FILE_SIZE);
        // 设置最大请求值(包含文件和表单数据)
        upload.setSizeMax(MAX_REQUEST_SIZE);
        // 中文处理
        upload.setHeaderEncoding("UTF-8");

        // 构造临时路径存储上传的文件
        // 这个路径相对于当前应用的目录
        String uploadPath = request.getServletContext().getRealPath("./") + File.separator+ UPLOAD_DIRECTORY;

        // 如果目录不存在则创建
        File uploadDir = new File(uploadPath);
        if (!uploadDir.exists()){
            uploadDir.mkdirs();
        }

        try {
            // 解析请求的内容提取文件数据
            List<FileItem> formItems = upload.parseRequest(request);
            if (formItems != null && formItems.size() > 0){
                // 迭代表单数据
                for (FileItem item:formItems){
                    // 处理不在表单中的字段
                    if (!item.isFormField()){
                        String fileName = new File(item.getName()).getName();
                        String filePath = uploadPath + File.separator + fileName;
                        File storeFile = new File(filePath);
                        // 在控制台输出文件的上传路径
                        System.out.println(filePath);
                        // 保存文件到硬盘
                        item.write(storeFile);
                        request.setAttribute("message","上传文件成功");
                    }
                }
            }
        }catch (Exception ex){
            ex.printStackTrace();
            request.setAttribute("message","错误信息"+ex.getMessage());
        }
        // 跳转到message.jsp文件中
        request.getServletContext().getRequestDispatcher("/message.jsp").forward(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```
文件上传upload.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>文件上传</title>
</head>
<body>
<h1>文件上传实例</h1>
<form method="post" action="upload" enctype="multipart/form-data">
    选择一个文件：
    <input type="file" name="uploadFile">
    <br/><br/>
    <input type="submit" value="上传">
</form>
</body>
</html>
```
文件上传结果页message.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>文件上传结果</title>
</head>
<body>
<center>
    <h2>${message}</h2>
</center>
</body>
</html>
```

上传文件结果
![上传成功](/assets/JavaEE/javaweb_14.png)
保存文件路径
![保存路径](/assets/JavaEE/javaweb_15.png)

文件下载
首先需要声明一个页面，当访问这个页面的时候，将所有可以下载的内容展示在网页上
```
public class ListFilesServlet extends HttpServlet {

    // 上传文件存储的路径
    private static final String UPLOAD_DIRECTORY = "upload";

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        // 获取上传文件的目录
        String uploadPath = request.getServletContext().getRealPath("./") + File.separator+ UPLOAD_DIRECTORY;
        // 存储要下载的文件名称
        Map<String,String> fileNameMap = new HashMap<>();
        // 递归遍历filePath目录下的所有文件和目录
        listfile(new File(uploadPath),fileNameMap);
        // 将Map集合发送listFile.jsp文件中
        request.setAttribute("fileNameMap",fileNameMap);
        request.getRequestDispatcher("/listfile.jsp").forward(request,response);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }

    /**
     * 遍历给定文件夹下的所有文件
     * @param file
     * @param map
     */
    private void listfile(File file,Map<String,String> map){
        // 如果file代表的不是一个文件而是一个目录，则需要递归调用
        if (!file.isFile()){
            // 列出该目录下所有的文件和目录
            File[] files = file.listFiles();
            for (File f:files){
                listfile(f,map);
            }
        }else {
            // 获取到的文件名和对象直接放在Map集合中
            map.put(file.getName(),file.getName());
        }
    }
}
```

listfile.jsp
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<html>
<head>
    <title>已经上传的文件列表</title>
</head>
<body>
<c:forEach var="me" items="${fileNameMap}">
    <c:url value="download" var="downurl">
        <c:param name="filename" value="${me.key}"></c:param>
    </c:url>
    ${me.value}<a href="${downurl}">下载</a>
    <br/>
</c:forEach>
</body>
</html>
```

下载的Servlet
```
public class DownloadServlet extends HttpServlet {
    // 上传文件存储的路径
    private static final String UPLOAD_DIRECTORY = "upload";

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request,response);
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String fileName = request.getParameter("filename");
        fileName = new String(fileName.getBytes("iso8859-1"),"UTF-8");
        // 上传的文件保存在固定的目录里
        String uploadPath = request.getServletContext().getRealPath("./") + File.separator+ UPLOAD_DIRECTORY;
        // 得到需要下载的文件
        File file = new File(uploadPath +File.separator+fileName);
        // 如果文件不存在
        if (!file.exists()){
            request.setAttribute("message","你想要下载的资源已被删除");
            request.getRequestDispatcher("/message.jsp").forward(request,response);
            return;
        }
        // 设置响应头
        response.setHeader("content-disposition","attachment;filename="+ URLEncoder.encode(fileName,"UTF-8"));
        // 读取要下载的文件，保存到文件输入流中
        FileInputStream fis = new FileInputStream(uploadPath + File.separator+fileName);
        // 创建输出流
        OutputStream os = response.getOutputStream();
        // 设置缓冲区
        byte[] buffer = new byte[1024];
        int len = 0;
        while ((len = fis.read(buffer)) > 0){
            // 输出缓冲区的内容到浏览器，实现文件下载
            os.write(buffer,0,len);
        }
        // 关闭文件输入流
        fis.close();
        // 关闭输出流
        os.close();
    }
}
```

![文件列表](/assets/JavaEE/javaweb_16.png)

## Servlet处理日期

Servlet可以使用Java中大多数的方法，这也是Servlet的优势之一。
日期相关的常用方法：

|方法|描述|
|---|---|
|boolean after(Date date)|如果调用的Date对象中包含的日期在date指定日期之后返回true|
|boolean before(Date date)|如果调用的Date对象中包含的日期在date指定日期之前返回true|
|Object clone()|重复调用Data对象|
|int compareTo(Date date)|把调用对象的值与date值进行比较，如果调用对象在date之前返回负数，如果调用对象在datte之后返回正数，两者相同返回0|
|int compareTo(Object obj)|如果obj是Date类型，使用的方式同上|
|boolean equals(Object date)|如果调用的Date对象中包含的时间和日期与date指定的相同，返回true|
|long getTime()|返回毫秒数|
|int hashCode()|未调用返回哈希代码|
|void setTime(long time)|设置time指定的时间和日期，毫秒为单位|

例子：
```
public class DateServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        String title = "显示当前的日期和时间";
        Date date = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        String docType = "<!DOCTYPE html>\n";
        out.println(docType +
                "<html>\n" +
                "<head><title>" + title + "</title></head>\n" +
                "<body bgcolor=\"#f0f0f0\">\n" +
                "<h1 align=\"center\">" + title + "</h1>\n" +
                "<h2 align=\"center\">" + sdf.format(date) + "</h2>\n" +
                "</body></html>");
    }
}
```

![时间servlet运行结果](/assets/JavaEE/javaweb_17.png)

## Servlet页面重定向
当文档移动到新的位置，我们需要向客户端发送这个新位置，我们需要用到网页重定向。当然也有可能是负载均衡，或者为了简单的随机，这些情况都有可能使用到网页重定向。

例子：
```
public class PageRedirectServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req,resp);
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        // 设置响应内容
        resp.setContentType("text/html;charset=UTF-8");
        // 设置重定向的位置
        String site = new String("http://www.paulniu.com");
        resp.setStatus(resp.SC_MOVED_TEMPORARILY);
        resp.setHeader("Location",site);
    }
}
```

# JSP

本来想写在一个页面里面的，但是东西有点太多，所以分开多个博客书写

# 参考文档
[Cookie和Session](https://www.cnblogs.com/vmax-tam/p/4130589.html)
[登录Demo](https://blog.csdn.net/weixin_36146275/article/details/55673211)