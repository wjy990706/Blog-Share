---
  layout:       post
  title:        "在CentOS 7下Tomcat+JSP实现留言板"
  date:         2019-03-31
  author:       "王玉松"
  header-img:   "/Blog-Share/img/1903/04/walkerwyl/message.png"
  tags:
      - CentOS
      - Tomcat配置环境
      - JSP
      - MVC
---

# 在CentOS 7下Tomcat+JSP实现留言板

## Java配置环境

一、检查原有的Java环境

```textile
   yum list installed | grep java
   yum -y remove java-1.8.0-openjdk*        *表示卸载所有openjdk相关文件输入
```

二、下载稳定的openJDK

```textile
    yum search java | grep -i -color JDK
    yum install java-1.8.0-openjdk-devel.x86_64
   //查找yum库中的java，选择最新的版本安装，以java-1.8.0-openjdk-devel.x86_64为例
    //没有java-1.8.0-openjdk-devel.x86_64就没有javac命令
    //也可以用yum install java-1.8.0-openjdk*安装所有java1.8.0所有java程序
```

三、添加配置文件

```textile
    通过yum默认安装的路径是 /usr/lib/jvm
    选择比较安全的方式进行配置
    在/etc/profile.d文件夹下新建java.sh文件，而不是直接更改/etc/profile
```

```bash
    # java.sh 2019-03-31 openJDK-1.8.0
    export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-8.b10.el7_5.x86_64
    export CLASSPATH=.:$JAVA_HOME/jre/lib.rt.jar:$JAVA_HOME/lib/dt.jar\
            :$JAVA_HOME/lib/tools.jar
    export PATH=$PATH:$JAVA_HOME/bin
```

```textile
    初期相比于在/etc/profile文件（设置全局变量）中直接将Java环境添入，存在一定的风险
    保存在/etc/profile.d文件夹中，启动时被/etc/profile文件调用，方便查找替换，记录文件创建更改的时间
```

## Tomcat配置环境

一、下载稳定的Tomcat

```textile
    到官网复制下载的地址，使用wget下载

    wget http://mirrors.shu.edu.cn/apache/tomcat/tomcat-9/v9.0.17/bin/apache-tomcat-9.0.17.tar.gz
    下载压缩包后，使用tar解压缩，并移到/usr/local
    tar -zxvf apache-tomcat-9.0.17.tar.gz
    cp apache-tomcat-9.0.17 /usr/local/apache-tomcat-9.0.17
```

二、启动关闭Tomcat

```bash
    cd /usr/local/apache-tomcat-9.0.17/bin
    ./startup.sh
    ./shutdown.sh
```

三、简单配置Tomcat

```bash
    firewall-cmd --state；#查看防火墙的状态
    # 若firewalld处于关闭状态，则打开firewalld systemctl start firewalld.service
    # 开启8080端口
    firewall-cmd --zone=public --add-port=8080/tcp --permanent；
    #--zone=public：表示作用域为公共的；
    #--add-port=8080/tcp：添加tcp协议的端口8080；
    #--permanent：永久生效，如果没有此参数，则只能维持当前服务生命周期内，重新启动后失效；
    systemctl restart firewalld.service；#重启防火墙
    firewall-cmd --reload；#重新载入配置
```

## 实例：留言板的设计

show.jsp                用于输入留言的界面

showResult.jsp    用于展示输入结果的界面

Message.java        用于存储数据

```javascript
      <!--show.jsp-->
      1 <%@ page language="java" import="java.util.*" pageEncoding="iso-8859-1"%>
      2 <%@ page import="mybean.data.Message"%>
      3 <jsp:useBean id="message" class="mybean.data.Message" scope="request"/>
      4 <%
      5     String path = request.getContextPath();
      6     String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
      7 %>
      8
      9 <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
     10 <html>
     11   <head>
     12     <base href="<%=basePath%>">
     13
     14     <title>Message Board</title>

     15         <meta http-equiv="pragma" content="no-cache">
     16         <meta http-equiv="cache-control" content="no-cache">
     17         <meta http-equiv="expires" content="0">
     18         <meta http-equiv="keywords" content="keyword1,keyword2,keyword3">
     19         <meta http-equiv="description" content="Message Board">
     20
     21
     22   </head>
     23
     24   <body>
     25         <form action="showResult.jsp" method="post">
     26         <table width="50" border="1">
     27                 <tr bgcolor="#999999">
     28                         <td height="42" colspan="3">
     29                                 <div align="center"><span>Message Board</span></div>
     30                         </td>
     31                 </tr>
     32                 <tr>
     33                         <td width="28%" height="36">Name:</td>
     34                         <td colspan="2"><input type="text" name="messName"/></td>
     35                 </tr>
     36                 <tr>
     37                         <td height="41">Subject:</td>
     38                         <td colspan="2"><input type="text" name="title"/></td>
     39                 </tr>
     40                 <tr>
     41                         <td height="93">Message:</td>
     42                         <td colspan="2"><textarea name="content" row="8" cols="30"></textarea></td>
     43                 </tr>
     44                 <tr bgcolor="#999999">
     45                         <td height="60" colspan="3">
     46                                 <label>
     47                                         <div align="center">
     48                                                 <input type="submit" value="Submit"/>    <input type="reset" value="Clean up"/>
     49                                         </div>
     50                                 </label>
     51                         </td>
     52                 </tr>
     53         </table>
     54     </form>
     55     <jsp:setProperty name="message" property="messName" param="name"/>
     56     <jsp:setProperty name="message" property="title" param="title"/>
     57     <jsp:setProperty name="message" property="content" param="content"/>
     58   </body>
     59 </html>
```

```javascript
      <!--showResult.jsp-->
      1 <%@ page contentType="text/html;charset=gb2312"%>
      2 <%@ page import="mybean.data.Message"%>
      3 <jsp:useBean id="message" class="mybean.data.Message" scope="request"/>
      4 <html>
      5 <body>
      6 <font>
      7 <jsp:setProperty name="message" property="messName" param="messName"/>
      8 <jsp:setProperty name="message" property="title" param="title"/>
      9 <jsp:setProperty name="message" property="content" param="content"/>
     10 Name:<jsp:getProperty name="message" property="messName" />
     11 <br>Subject:<jsp:getProperty name="message" property="title" />
     12 <br>Message:<jsp:getProperty name="message" property="content" />
     13 </font>
     14 </body>
     15 </html>
```

```java
      1 package mybean.data;
      2
      3 public class Message {
      4
      5     public int messID;
      6     public String messName;
      7     public String title;
      8     public String content;
      9
     10
     11     public int getMessID() {
     12       return this.messID;
     13     }
     14     public String getMessName() {
     15       return this.messName;
     16     }
     17     public String getTitle() {
     18       return this.title;
     19     }
     20     public String getContent() {
     21       return this.content;
     22     }
     23
     24
     25     public void setMessID(int newMessID) {
     26       this.messID=newMessID;
     27     }
     28     public void setMessName(String newMessName) {
     29       try {
     30         byte[] bb=newMessName.getBytes("iso-8859-1");
     31         this.messName=new String(bb);
     32       }
     33       catch(Exception e) {}
     34       this.messName=newMessName;
     35     }
     36     public void setTitle(String newTitle) {
     37       try {
     38         byte[] bb=newTitle.getBytes("iso-8859-1");
     39         this.title=new String(bb);
     40       }
     41       catch(Exception e) {}
     42       this.title=newTitle;
     43     }
     44     public void setContent(String newContent) {
     45       try {
     46         byte[] bb=newContent.getBytes("iso-8859-1");
     47         this.content=new String(bb);
     48       }
     49       catch(Exception e) {}
     50       this.content=newContent;
     51     }
     52 }
```

## 参考资料

- [CentOS 7用yum安装Java](https://blog.csdn.net/qq_18298439/article/details/80827880 "yum安装Java")

- [CentOS 7安装Tomcat9](https://www.cnblogs.com/moonsoft/p/9264883.html "CentOS 7 Tomcat 9")

- [CentOS 7开放8080端口](https://www.jb51.net/os/Ubuntu/617627.html "CentOS 7 firewalld 8080")

- [MVC框架简易留言板实例](https://blog.csdn.net/qq_39145189/article/details/79839872 "MVC 留言板")
