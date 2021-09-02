---
title: JavaEE 04 JavaWeb基础(JSP)
date: 2019-10-16 21:37:03
tags:
  - JavaEE
---

JSP 全称Java Server Page，是一种动态网页开发技术。它是用JSP标签在HTML网页中插入Java代码，标签通常是以<%开头以%>结束

<!--more-->

JSP是一种JavaServlet，主要用于实现JavaWeb应用程序的用户界面部分。我们可以通过HTML，XHTML，XML和嵌入的Java代码操作和命令完成JSP代码的书写。
JSP通过网页表单获取用户输入的信息，访问数据库和其他资源，然后动态的创建网页
JSP标签有多种功能，比如访问数据库，记录用户选择信息，访问JavaBean组件，还可以在不同网页之间传递数据和共享信息

## JSP结构
网络服务器需要一个JSP引擎，也就是容器来处理JSP，容器负责截获对JSP的请求
JSP 容器与 Web 服务器协同合作，为JSP的正常运行提供必要的运行环境和其他服务，并且能够正确识别专属于 JSP 网页的特殊元素。

下图显示了 JSP 容器和 JSP 文件在 Web 应用中所处的位置。

![JSP结构示意图](/assets/JavaEE/javaweb_18.jpg)

### JSP处理
以下步骤表明了Web服务器如何通过JSP创建网页
1. 像往常一下，浏览器发送一个HTTP请求给服务器
2. Web服务器辨别出这是一个对JSP页面的请求，并且将该请求传递给JSP引擎，通过使用的URL或者.jsp文件来完成
3. JSP引擎从磁盘中载入JSP文件，然后将它转换成Servlet，这种转换只是简单的将所有模板文本改成println()语句输出，并且将所有的JSP元素转化成Java代码
4. JSP引擎将Servlet转换成可执行类，并将原始请求传递给Servlet引擎
5. Web服务器的某些组件将调用Servlet引擎，并将可执行类载入，执行过程中，Servlet产生HTML格式的输出，并将其内嵌入HTTP response中交给Web服务器
6. Web服务器以静态HTML的形式将HTTP response返回到浏览器中
7. 最终浏览器处理HTTP请求，动态产生HTML

## JSP语法

### 脚本程序

脚本程序可以包含任意量的Java语句，变量，方法或表达式，只要他们在脚本中是有效的。
形式可以如下
```
<% 代码片段 %>
```
或者
```
<jsp:scriptlet>
  代码片段
</jsp:scriptlet>
```

任何文本，HTML标签，JSP元素必须卸载脚本程序的外面
```
<html>
<head><title>Hello World</title></head>
<body>
Hello World!<br/>
<%
out.println("Your IP address is " + request.getRemoteAddr());
%>
</body>
</html>
```

在使用JSP页面的时候，会出现中文编码的问题，如果我们想要显示中文，可以在JSP文件头部添加如下代码
```
<%@ page language="java" contentType="text/html;charset=UTF-8" pageEncoding="UTF-8" %>
```

JSP的声明
一个声明语句可以声明一个或多个变量，方法，供后面的Java代码使用。在JSP文件中，我们必须先声明这些变量和方法，然后才能使用
```
<%! int i = 0; %> 
<%! int a, b, c; %> 
<%! Circle a = new Circle(2.0); %> 
```

JSP表达式
一个JSP表达式中包含的脚本语言表达式，先被转换成String，然后插入到表达时出现的地方。
由于表达式的值会被转换成String，所我们可以在一个文本杭中使用表达式而不用管它是否是HTML标签
表达式元素中可以包含任何符合Java语言规范的表达式，但是不能用分号结束表达式
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>菜鸟教程(runoob.com)</title>
</head>
<body>
<p>
   今天的日期是: <%= (new java.util.Date()).toLocaleString()%>
</p>
</body> 
</html> 
```

JSP注释分为以下几种

|语法|描述|
|---|---|
|<%-- 注释内容 --%>|JSp注释，注释内容不会被发送到浏览器甚至不会被编译|

JSP指令
JSP指令用来设置与整个JSP页面相关的属性
这里有三种指令标签

|指令|描述|
|---|---|
|<%@ page ... %>|定义页面中的依赖属性，比如脚本语言，error页面，缓存需求等|
|<%@ include ... %>|包好其他文件|
|<%@ taglib ... %>|引入标签库的定义，可以是自定义标签|

JSP行为
JSP行为标签使用XML语法结构来控制Servlet引擎，它能够懂她插入一个文件，重用JavaBean组件，引导用户去另一个页面，为Jav插件产生英冠的HTML等
应为标签只有一种语法格式，而且必须严格遵守XML标准
```
<jsp:action_name attribute="value" />
```
行为标签基本上是一些预先定义好的函数，下面是一些可用的JSP行为标签

|语法|描述|
|---|---|
|jsp:include|用于在当前页面中包含鼎泰或动态资源|
|jsp:useBean|寻找和初始化一个JavaBean组件|
|jsp:setProperty|设置JavaBean组件的值|
|jsp:getProperty|将JavaBean组件的值插入到output中|
|jsp:forward|从一个JSP文件向另一个文件传递一个包含用户请求的request对象|
|jsp:plugin|用于在生成的HTML页面中包含Applet和JavaBean对象|
|jsp:element|动态创建一个XML元素|
|jsp:attribute|定义动态创建的XML元素属性|
|jsp:body|定义动态创建的XML元素的主体|
|jsp:text|用于封装模板数据|

JSP隐含对象
JSP支持九个自动定义的变量，如下表所示

|对象名称|描述|
|---|---|
|request|HttpServletRequest类实例|
|response|HttpServletResponse类实例|
|out|PrintWriter类实例，用于把结果输出至网页上|
|session|HttpSession类实例|
|application|ServletContext类实例，与应用上下文有关|
|config|ServletConfig类实例|
|pageContext|PageContext类实例，提供对JSP页面所有对象以及命名空间的访问|
|page|类似于Java中的this关键字|
|Exception|Exception类对象，代表发生错误的JSP页面中对应的异常对象|

JSP字面量
- 布尔值：boolean
- 整型：int
- 浮点型：float
- 字符串：String
- Null：null

### JSP的控制流语句

#### 判断语句
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%! int day = 3; %> 
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>判断语句</title>
</head>
<body>
<h3>IF...ELSE 实例</h3>
<% if (day == 1 | day == 7) { %>
      <p>今天是周末</p>
<% } else { %>
      <p>今天不是周末</p>
<% } %>
</body> 
</html>
```

switch...case语句
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%! int day = 3; %> 
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>判断语句</title>
</head>
<body>
<h3>SWITCH...CASE 实例</h3>
<% 
switch(day) {
case 0:
   out.println("星期天");
   break;
case 1:
   out.println("星期一");
   break;
case 2:
   out.println("星期二");
   break;
case 3:
   out.println("星期三");
   break;
case 4:
   out.println("星期四");
   break;
case 5:
   out.println("星期五");
   break;
default:
   out.println("星期六");
}
%>
</body> 
</html> 
```

 #### 循环语句

 ```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%! int fontSize; %> 
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>循环语句</title>
</head>
<body>
<h3>For 循环实例</h3>
<%for ( fontSize = 1; fontSize <= 3; fontSize++){ %>
   <font color="green" size="<%= fontSize %>">
    循环语句测试
   </font><br />
<%}%>
</body> 
</html> 
 ```

 # JSP指令
 JSP指令用来设置整个页面相关的属性，比如网页的编码格式和脚本语言等
 语法格式如下：
 ```
<%@ directive attribute="value" %>
 ```
 指令可以有多个属性，他们以键值对的形式存在，并用逗号隔开

 |指令|描述|
 |---|---|
 |<%@ page ...%>|定义网页依赖属性，比如脚本语言，error页面，缓存请求等|
 |<%@ include ...%>|包含其它文件|
 |<%@ taglib ...%>|引入标签库的定义|

 ## Page指令
 Page指令为容器提供当前页面的使用说明，一个JSP页面可以包含多个page指令
 属性包括以下内容
 
 |属性|描述|
 |---|---|
 |buffer|指定out对象使用缓冲区的大小|
 |autoFlush|控制out对象的缓冲区|
 |contentType|指定当前JSP页面的MIME类型和字符编码|
 |errorPage|指定当前JSP页面发生异常时需要转向的错误处理页面|
 |isErrorPage|指定当前页面是否可以作为另一个JSP的错误处理页面|
 |extends|指定servlet从哪一个类继承|
 |import|导入要使用的java类|
 |info|定义JSP页面的描述信息|
 |isThreadSafe|指定对JSP页面的访问是否为线程安全|
 |language|定义JSP页面所使用的脚本语言，默认是Java|
 |session|指定JSP页面是否使用session|
 |isELlgnored|指定是否执行EL表达式|
 |isScriptingEnabled|确定脚本元素能否被使用|

 ## include指令
 JSP可以通过include指令来包含其它文件，被包含的文件可以是JSP页面，HTML页面或者文本文件，包含的文件就好像是JSP文件的一部分，会被同时编译执行


 ## Tablib指令
 JSP API允许用户自定义标签，一个自定义标签库就是自定义便签的集合
 Tablib指令引入一个自定义便签合集的定义，包括库路径，自定义标签
 ```
<jsp:directive.taglib uri="uri" prefix="prefixOfTag" />
 ```

 