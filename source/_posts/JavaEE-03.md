---
title: JavaEE 03 JavaWeb基础
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