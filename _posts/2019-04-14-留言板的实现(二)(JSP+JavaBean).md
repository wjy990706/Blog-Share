---
  layout:       post
  title:        "留言板的实现（二）（JSP+JavaBean）"
  date:         2019-04-14
  author:        "王玉松"
  header-img:   "img/1904/01/walkerwyl/windmill.jpg"
  tags:
      - CentOS7
      - Tomcat
      - JSP
      - JavaBean
---

# 留言板的实现（二）（JSP+JavaBean）<br>

## 一、JSP的布局<br>

1、首页+导航条<br>
<p>所有页面将包括一个导航条，由写留言，注册，登录，显示最新留言，回到首页组成。其他页面使用JSP的include标记将导航条文件head.txt嵌入。<br>
```JSP
    <!--index.jsp-->
    <%@ page contentType="text/html;charset=gb2312" %>
    <head><%@ include file="head.txt" %></head>
    <html><body></body></html>
```
```txt
    head.txt
    <%@ page contentType="text/html;charset=gb2312" %>
    <center><font size="5"><p>Message Port</font></center>
    <table cellSapcing="1" cellPadding="1" width="560" align="center" boder="0">
        <tr valign="bottom">
          <td><a href="writer.jsp"><font size="2">Writing Message</font></a></td>
          <td><a href="register.jsp"><font size="2">Register</font></a></td>
          <td><a href="login.jsp"><font size="2">Login in</font></a></td>
          <td><a href="show_message.jsp"><font size="2">The lastest message</font></a></td>
          <td><a href="index.jsp"><font size="2">Return to main page</font></a></td>
        </tr>
    </table>
```
<br>
2、留言板页面
<p>留言板页面使用Message作为数据存放的模板，将得到的usr和comtent封装起来交给write_mysql.jsp，即交给另一个页面将数据存入数据库。<br>
```JSP
  <!--writer.jsp-->
  <%@ page contentType="text/html;charset=gb2312"%>
  <%@ page import="mybean.data.Message" %>
  <jsp:useBean id="message" class="mybean.data.Message" scope="request" />
  <html><head><title>Message Port</title></head><body>
	<h1>Message Port</h1><br>
	<form action="write_mysql.jsp" method="post">
	       Your Name:<input type="text" name="usr">
	       Your Message:<textArea name="content" rows="15" cols="50"></textArea>
	       <input type="submit" name="Send">
		     <input type="reset" name="CLean Up">
	</form>
  </body></html>
```
<br>
```JSP
    <!--writer_mysql.jsp-->
    <%@ pageicontentType="text/html;charset=gb2312" %>
    <%@ page import="java.sql.*" %>
    <%@ page import="mybean.data.Message" %>
    <jsp:useBean id="message" class="mybean.data.Message" scope="request" />
    <jsp:setProperty name="message" property="*" />

    <html><body>
    <%
      request.setCharacterEncoding("gb2312");
      String usr=request.getParameter("usr");
      String subject=request.getParameter("subject");
      String content=request.getParameter("content");

      Connection con=null;
      Statement sql;
      ResultSet rs;
      try {   Class.forName("com.mysql.jdbc.Driver");
      }
      catch(Exception e) {
          out.println("This page can't find a jdbc.");
      }
      try {
          String uri="jdbc:mysql://127.0.0.1/MESSAGES";
          String user="root";
          String password="";
          con=DriverManager.getConnection(uri, user, password);
          sql=con.createStatement();
          sql.executeUpdate("INSERT INTO information values('"+usr+"','"+subject+"','"+content+"');");
          con.close();
      }
      catch(SQLException e) {
          out.print(e);
      }
%></body></html>
```
<br>
3、注册与登录界面
<p>注册与登录界面（register.jsp和login.jsp）分别用到Rigester.java和Login.java作为数据的载体，用户输入的信息将由交由另外的两个页面handleRigester.jsp二号handleLogin.jsp处理，从数据库中查找相应的信息进行判断。
```JSP
    <!--login.jsp-->
    <%@ page contentType="text/html;charset=gb2312" %>
    <%@ page import="mybean.data.Login" %>
    <jsp:useBean id="login" class="mybean.data.Login" scope="request" />

    <head><%@ include file="head.txt" %></head><html>
    <body background=bgpicture.jpg><title>Message Port Login Page</title>
    <h1><center>Login in Message Port</center></h1>
    <form action="login_mysql.jsp" method="post">
      <table align="center">
        <tr><td>Your name:</td>    <td><input type="text" name="usr"><br></td></tr>
        <tr><td>Your password:</td>  <td><input type="text" name="passwd"><br></td></tr>
        <tr><td><input type="submit" value="Send"><input type="reset" value="Clear"><br></td></tr>
      </table>
    </form>
    <jsp:setProperty name="login" property="*" />
    </body></html>
```
<br>
```JSP
    <!--register.jsp-->
    <%@ page contentType="text/html;charset=gb2312" %>
    <%@ page import="mybean.data.Register" %>
    <jsp:useBean id="register" class="mybean.data.Register" scope="request" />
    <head><%@ include file="head.txt" %></head><html>
    <body background=bgpicture.jpg><title>Message Port Sign Page</title>
    <h1><center>Sign Up Message Port</center></h1>
    <form action="sign_mysql.jsp" method="post">
      <table align="center">
        <tr><td>Your name:</td>    <td><input type="text" name="usr"><br></td></tr>
        <tr><td>Your password:</td>  <td><input type="text" name="passwd"><br></td></tr>
        <tr><td><input type="submit" value="Send"><input type="reset" value="Clear"><br></td></tr>
      </table>
    </form>
    <jsp:setProperty name="login" property="*" /></body></html>
```
<br>
4、显示最新留言界面<br>
<p>show_message.jsp专门用于连接数据库将information数据表中的数据展示在页面上。
```JSP
    <%@ page contentType="text/html;charset=gb2312" %>
    <%@ page import="java.sql.*" %>
    <head><%@ include file="head.txt" %></head><html>
    <body background=bgpicture.jpg><br>
    <table align="center">
      <tr>
        <td><h2>User</h2></td>
        <td><h2>Subject</h2></td>
        <td><h2>Message</h2></td>
      </tr>
    <%
      Connection con=null;
      Statement sql;
      ResultSet rs;
      try {   Class.forName("com.mysql.jdbc.Driver");
      }
      catch(Exception e) {
        out.println("This page can't find a jdbc.");
      }
      try {
        String uri="jdbc:mysql://127.0.0.1/MESSAGES";
        String user="root";
        String password="";
        con=DriverManager.getConnection(uri, user, password);
        sql=con.createStatement();
        rs=sql.executeQuery("SELECT * FROM information;");
        while(rs.next()) {
    %>
    <tr>
      <td><h3><%=rs.getString(1)%></h3></td>
      <td><h3><%=rs.getString(2)%></h3></td>
      <td><h3><%=rs.getString(3)%></h3><td>
    <tr>
   <%
      }
      con.close();
      }
      catch(SQLException e) {
          out.print(e);
      }
    %>
    </table><br></body></html>
```
<br>
## 二、JavaBean的设置<br>
<p>目前总共用到了三个JavaBean，分别是Message.java、Login.java、Register.java。用于JSP页面之间封装传递数据的模板。<br>
```Java
    package mybean.data;

    public class Message {
        String name;
        String content;

    public void setName(String n) {
        name=n;
    }
    public void setContent(String m) {
        content=m;
    }

    public String getName() {
        return name;
    }
    public String getContent() {
        return content;
    }

    }
```
<br>
```Java
    package mybean.data;

      public class Register {
          String logname="";
          String password="";

      public void setLogname(String name) {
          logname=name;
      }
      public void setPassword(String pw) {
          password=pw;
      }

      public String getLogname() {
          return logname;
      }
      public String getPassword() {
          return password;
      }
    }
```
<br>
```Java
    package mybean.data;

      public class Login {

      String usr;
      String passwd;

      public void setUsr(String new_usr) {
        usr=new_usr;
      }
      public void setPasswd(String new_passwd) {
        passwd=new_passwd;
      }

      public String getUsr() {
        return usr;
      }
      public String getPasswrd() {
        return passwd;
      }
    }
```
<br>
## 三、数据库的创建<br>
<p>Mariadb与MySQL大体上互相兼容。从官网上下载相应的COnnect/J放到系统环境中的JDK以及Tomcat的lib库文夹下。<br>
```SQL
    CREATE DATABASE MESSAGES;
    USE MESSAGES;
    CREATE TABLE usr_message(
      usr char(50) not null,
      passwd char(50),
      PRIMARY KEY(usr)
    );
    CREATE TABLE information(
      usr char(50) not null,
      subject char(50),
      content char(255),
      PRIMARY KEY(usr)
    );
```
## 四、收获<br>
1、Linux与Windows相结合，Windows更加常用，可以现在电脑上把各部分测试之后再放到服务器上，避免在不熟悉的环境下，花费大量时间。<br>
2、Linux下输入需要十分仔细，由于环境不熟悉，再加上自己的错误输入，会使得任务难上加难。<br>
3、JSP下，两个页面之间通过JavaBean传值时，接收方也要使用setProperty，否则只能收到空值。<br>
4、独立思考和寻求帮助两者不相互冲突，如果在一个问题浪费了太多时间，即使最终解决了，也有可能得不偿失，适时的寻求帮助可以更快地学习，不能害怕麻烦什么的。<br>

## 五、下一周将要完成的目标<br>
1、将留言板中的很大一部分操作提取出来，用Servlet来完成；并对现有的页面再次进行整理，期望达到MVC的标准。<br>
2、整理笔记，写成自己的博客。希望一周能多记录一点。<br>

## 参考资料
- [JSP实用教程(第三版)](https://baike.baidu.com/item/JSP%E5%AE%9E%E7%94%A8%E6%95%99%E7%A8%8B%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%EF%BC%89/17513286?fr=aladdin "")
- [MVC框架简易留言板实例](https://blog.csdn.net/qq_39145189/article/details/79839872 "MVC 留言板")
