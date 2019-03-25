# HTTP—>HTTPS (SSL/TLS)

Author: iyin

Date: 2019/3/23

HTTP（HyperText Transfer Protocol）作为一种无状态的、简单快速的、基于 TCP的”可靠“传输协议。它是否足够安全呢？与HTTPS又有什么关系呢？

## HTTP与HTTPS

### 1. 什么是HTTP？

* 大部分网站都是通过 HTTP 协议来传输 Web 页面以及 Web 页面上包含的图片、CSS 样式、JS 脚本等信息的，HTTP协议可以说是万维网的数据通信的基础。

* 但是，近几年，Google、Baidu、Facebook 等互联网巨头开始大力推行 HTTPS， 国内外的很多大型互联网公司很多也都已经启用了全站 HTTPS，这是为什么呢？

  HTTP明文传输带来了很大的**安全隐患**，主要有三大风险：

  > 窃听风险：第三方可以获知通信内容
  >
  > 篡改风险：第三方可以修改通信内容
  >
  > 冒充风险：第三方可以冒充他人身份参与通信

  ps: 抓包可看到使用http传输的图片信息（第四行）：

  > ![1553445596865](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553445596865.png)

  把他的图片给改了：

  ![1553452062737](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553452062737.png)

### 2. 给HTTP加一把锁—>HTTPS

* HTTPS（HyperText Transfer Protocol over Secure Socket Layer) 开发的主要目的，是提供对网站服务器的身份认证，保护交换数据的隐私与完整性。(from wiki)

* **HTTPS经由HTTP进行通信，利用SSL/TLS来加密数据包。**可以将它理解为HTTP+SSL/TLS， 即 HTTP 下加入 SSL/TLS 层。其层次结构及位置如图：

  > ![1553446309418](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553446309418.png)
  >
  > (from 简书)

* 针对HTTP的问题，它提供了以下**解决方案**：

  > 所有信息都是加密传播，第三方无法窃听
  > 具有校验机制，一旦被篡改，通信双方会立刻发现
  > 配备身份证书，防止身份被冒充

那么，SSL/TLS是如何承载HTTP，实现以上方案的呢？

## SST/TLS的工作

### 1. 工作机制简介

- 传输层安全性协议（Transport Layer Security，TLS）及其前身安全套接层（Secure Sockets Layer，SSL）是一种安全协议。

  **它指定了一种在应用程序协议（如Http、Telenet、NMTP和FTP等）和TCP/IP协议之间提供数据安全性分层的机制。** 示意图：

  > ![1553446400295](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553446400295.png)
  >
  > (from csdn)

### 2. 结构及功能

+ **整体结构**

  **TLS协议是由TLS记录协议和TLS握手协议这两层协议叠加而成的。**

  位于底层的TLS记录协议负责进行加密，而位于上层的TLS握手协议负责除了加密以外的其他各种操作，上层又可以分为4个子协议。如下图所示：

  ![1553447868551](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553447868551.png)

  (from wiki)

* **具体功能**

  * **TLS握手协议**最重要的作用是**进行身份认证与加密信息的协商**，4个子协议功能如下：

    > **握手协议，是TLS握手协议的一部分（注意区分二者），负责在客户端和服务器之间协商决定密码算法和共享密钥。基于证书的认证操作也是在这个协议中完成。**
    >
    > 密码规格变更协议，负责向通信对象传达变更密码方式的信号。
    >
    > 警告协议，负责在发生错误时将错误传达给对方。
    >
    > 应用数据协议，负责将TLS上面承载的应用数据传达给通信对象。

    其中，著名的客户端与服务器**握手过程**如下图所示（本图为个人理解的顺序和过程，其中，服务器必须认证客户端，但是客户端可以不要求服务器发送证书，如果要求，就是双向认证。ps:其实交换密钥的过程也是变相的身份认证了吧）：

    ![1553448792562](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553448792562.png)

    简单来说，本过程结束后，客户机与服务器选择了双方都支持的密码算法，生成了共享密钥，完成了身份认证，从而可以建立加密TSL连接。

  * **TLS 记录协议**，位于TLS握手协议的下层，是用来封装上层协议数据的协议，负责使用协商好的加密算法及产生的共享密钥对消息进行加密通信。

    所有的传输数据都被封装在记录中，该协议可完成以下工作：分组、组合；压缩、解压缩；消息认证；消息传输等。部分流程如下图所示：

    > 其中添加的MAC是消息认证码，用于接收方验证消息是否真的是对方产生的。
    >
    > ![1553449674281](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553449674281.png)
    >
    > (from csdn)

* **TLS协议与应用层**

  TLS协议对于应用层来说是透明的，在编写基于TSL的HTTPS应用时，无论客户端还是服务端都不需要考虑SSL的存在。

### 3. 过程模拟输出示意

用open ssl库进行了基于TSL的socket通讯过程模拟，部分输出：

> ![1553451193044](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553451193044.png)

> ![1553451209002](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553451209002.png)

> ![1553451232281](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553451232281.png)

> ![1553451240335](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553451240335.png)

## 补充与总结

1. HTTP默认使用80端口，Https使用443端口。

2. 关于安全性

   HTTPS相比HTTP会更加安全，但是由于需要额外加解密操作，效率没有那么高。在登录Https站点和Http站点时，可以明显感觉到性能差异。

   但是HTTPS当然也不是绝对安全，在设计机制、实现、信任模型等方面都可以寻找漏洞进行攻击，比如前些年著名的open ssl心脏滴血漏洞。

3. 可以利用Wireshark等抓包工具验证一下握手等过程，还是挺有意思的，HTTP账号和密码完全没有加密，明文传送，可以很容易截获到账号和密码。

4. TLS非常好地利用了密码技术，篇幅限制，握手的详细过程没有在此介绍，计科密码学的课程由于其过程偏繁琐也没有介绍，感兴趣的话可以进一步了解。

   对于在TLS中使用的密码技术总结如下：

   > - 公钥密码：加密预备主密码
   > - 单项散列函数：构成伪随机数生成器
   > - 伪随机数生成器：生成预备主密码；根据主密码生成密钥（密码参数）；生成初始化向量。

   > * 对称密码（CBC模式）：确保片段的机密性
   > * 消息认证码：确保片段的完整性并进行认证 
   > * 认证加密（AEAD）：确保片段的完整性和机密性并进行认证

5. 知乎上有老哥对握手过程的理解...

   ![1553451931548](C:\Users\iyin\AppData\Roaming\Typora\typora-user-images\1553451931548.png)

