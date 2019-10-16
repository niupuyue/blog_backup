---
title: JavaEE 03 JavaWeb基础(JSP)
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



