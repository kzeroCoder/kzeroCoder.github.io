---
title: 计算机网络(二)
author: Kzero Coder
date: 2020-12-02 15:29:00 +0800
categories: [Blogging, Computer Network]
tags: [writing]
math: true
---
<head>
    <script type="text/x-mathjax-config"> 
   		MathJax.Hub.Config({ TeX: { equationNumbers: { autoNumber: "all" } } }); 
   	</script>
    <script type="text/x-mathjax-config">
    	MathJax.Hub.Config({tex2jax: {
             inlineMath: [ ['$','$'], ["\\(","\\)"] ],
             processEscapes: true
           }
         });
    </script>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" 	type="text/javascript">
	</script>
</head>


# 第二章 应用层

## 2.1 应用层协议原理

### 2.1.1 网络应用程序体系结构

在应用程序研发者的角度看，网络体系结构式固定的，应用程序体系结构由应用程序研发者涉及。

- 客户-服务器体系结构(C/S)
  - 服务器需要
    - 7*24小时提供服务
    - 永久性访问地址/域名
    - 利用大量服务器实现可扩展性
  - 客户机需要
    - 与服务器通信，使用服务
    - 间歇性接入网络
    - 不与其他客户机直接通信
  - 如：Web、FTP、Telnet和电子邮件
- 对等体系结构(P2P)
  - 没有永远在线的服务器
  - 任意端系统之间直接通信
  - 结点可能改变IP地址
  - 如：文件共享(BitTorrent)、因特网电话、IPTV等
- 混合结构
  - 如Napster
    - 文件传输使用P2P结构
    - 文件检索采用C/S结构
    - ![napster](/assets/img/computerNetwork/computerNetworkCh2/image-20201201155655234.png)



### 2.1.2 进程通信

**进程**：运行在端系统中的一个程序。

- 同一主机，通过端系统操作系统确定。

- 不同主机，通过跨越计算机网络交换**报文**相互通信。



1. 客户和服务器进程
   - 将两个进程之一称为客户client，另一进程标识为服务器server
   - 更具体的说，发起通信进程为client，等待联系的进程式server
   - P2P中，一个进程既是客户也是服务器
2. 进程与计算机网络之间的接口
   - 进程通过**套接字socket**软件接口向网络发送报文和从网络接收报文。
   - 由于套接字是建立网络应用程序的可编程接口，也被称为应用程序和网络之间的**应用程序编程接口**
   - 应用开发者对于运输层的控制仅限于
     - 选择运输层协议
     - 设定几个运输层参数(如最大缓存和最大报文段长度)
3. 进程寻址
   - 为了标识接收进程，需要定义
     - 主机地址
     - 定义在目的主机中的接收进程的标识符
   - 因特网中，主机由**IP地址**标识，目的地**端口号**用于唯一标识网络应用
   - **进程的标识符 = IP地址 + 端口号**



### 2.1.3 可供应用程序使用的运输服务

大体能够从四个方面对应用程序服务要求进行分类：

1. 可靠数据传输
   - 如果一个协议提供了确保数据交付服务，就认为提供了**可靠数据传输**
   - 由发送进程发送的某些数据可能不能够到达接收进程，这可能能被**容忍丢失的应用**所接收
2. 吞吐量
   - 具有吞吐量要求的的应用程序被称为**带宽敏感的应用**
   - **弹性应用**能够根据情况或多或少的利用可供使用的吞吐量
3. 定时
   - **实时应用**要求较低时延
   - **非实时应用**对端对端的时延没有严格约束
4. 安全性
   - 加密/解密，提供机密性
   - 数据完整性、端点鉴别

![网络应用需求](/assets/img/computerNetwork/computerNetworkCh2/image-20201201162127609.png)



### 2.1.4 因特网提供的运输服务

1. TCP服务
   - 面向连接的服务：通过握手过程，建立两个进程套接字之间的TCP连接，该连接全双工，结束后需要拆除该连接。
   - 可靠的数据传送服务：依赖TCP、无差错、按适当顺序交付所有发送数据，没有字节的丢失和冗杂。
   - 拥塞控制、不提供时延保障和最小带宽保障
2. UDP
   - 无连接、不可靠数据传输服务
   - 不保证报文确认到达，也不保证按序到达
   - 没有拥塞控制机制
3. 因特网运输协议所不提供的服务
   - 不提供吞吐量或定时的保证
   - ![应用层协议和支撑的运输协议](/assets/img/computerNetwork/computerNetworkCh2/image-20201201162937529.png)



### 2.1.5 应用层协议

**应用层协议**定义了运行在不同端系统上的应用程序进程如何相互传递报文。

- 交换的报文类型，如请求报文和响应报文
- 各种报文类型的语法，如报文中的各个字段及这些字段是如何描述的
- 字段的语义，即这些字段中包含的信息的含义
- 一个进程何时以及如何发送报文、对报文进行响应的规则

公开协议：由RFC定义，用于允许互操作，如HTTP，SMTP，……

私有协议：用于多数P2P文件共享应用



## 2.2 Web和HTTP

### 2.2.1 HTTP概况

Web的应用层协议是**超文本传输协议HTTP**，是Web的核心。

HTTP由两个程序实现：客户程序和服务器程序。

Web页面由对象组成，如：HTML文件、JPEG图片、视频文件、动态脚本等，且可通过一个统一资源定位器URL地址寻址(格式为Scheme://host:port/path)。

HTTP是**无状态的**：服务器不维护任何有关客户端过去所发请求的信息。



### 2.2.2 非持续连接和持续连接

1. 非持续连接的HTTP

   - 每个TCP连接最多允许传输一个对象

   - ![image-20201201164515585](/assets/img/computerNetwork/computerNetworkCh2/image-20201201164515585.png)

     ![image-20201201164523900](/assets/img/computerNetwork/computerNetworkCh2/image-20201201164523900.png)

   - **RTT（Round Trip Time）**：从客户端发送一个很小的数据包到服务器并返回所经历的时间。
     **响应时间（Response time）**=2RTT+文件发送时间

2. 采用持续连接的HTTP

   - 每个TCP连接允许传输多个对象
   - 类型：
     - 无流水的持久性连接
       - 客户端只有收到前一个响应后才发送新的请求，每个被引用的对象耗时1个RTT
     - 带流水机制的持久性连接
       - 客户端只要遇到一个引用对象就尽快发出请求；理想情况下，收到所有引用对象只需耗时约1个RTT



### 2.2.3 HTTP报文格式

HTTP报文由两种：请求报文和响应报文

1. HTTP请求报文
   - ![image-20201201165139571](/assets/img/computerNetwork/computerNetworkCh2/image-20201201165139571.png)
   - 用ASCII文本书写，人为可读，每行由一个回车和换行符结束"\r\n"
   - 第一行叫做**请求行**，包含方法字段、URL字段和HTTP版本字段
     - 方法字段可以取**GET、POST、DELETE、PUT(HTTP/1.1，上传)、HEAD(HTTP/1.1，类似GET，但是不返回请求对象)**
   - 后继行为**首部行**
   - ![HTTP请求报文通用格式](/assets/img/computerNetwork/computerNetworkCh2/image-20201201165604204.png)
2. HTTP响应报文
   - ![image-20201201165918840](/assets/img/computerNetwork/computerNetworkCh2/image-20201201165918840.png)
   - **状态行**：响应消息的第一行
     - 200 OK
     - 301 Moved Permanently
     - 400 Bad Request
     - 404 Not Found
     - 505 HTTP Version Not Supported
   - ![HTTP响应报文通用格式](/assets/img/computerNetwork/computerNetworkCh2/image-20201201170210958.png)



### 2.2.4 用户与服务器的交互：cookie

由于HTTP是无状态的，但是某些时候希望将内容与用户身份联系起来，为此某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据（通常加密），即cookie。

cookie技术由4个组件：

- HTTP响应报文中一个cookie首部行
- HTTP请求报文中一个cookie首部行
- 用户端系统中保留一个cookie文件，由浏览器管理
- 位于Web站点的一个后端数据库
- ![用cookie跟踪用户状态](/assets/img/computerNetwork/computerNetworkCh2/image-20201201170550254.png)

用于**身份认证、购物车、推荐、Web email**等，但具有**隐私问题**



### 2.2.5 Web缓存

**Web缓存器**也叫**代理服务器**，既作为客户也作为服务器。用于**缩短客户请求的响应时间、减少机构组织的流量、在大范围内实现有效的内容分发**。

![客户通过Web缓存器请求对象](/assets/img/computerNetwork/computerNetworkCh2/image-20201201171011316.png)

Web缓存器通常由**ISP**购买并安装。



### 2.2.6 条件GET方法

HTTP协议通过**条件GET方法**证实对象为最新的。

如果

1. 请求报文使用GET方法
2. 请求报文中包含"If-Modified-Since:"首部行

那么这个HTTP请求报文就是条件GET请求报文。

如果缓存版本是最新的，则响应消息包含状态码304 Not Modified，不包含对象。

![条件GET](/assets/img/computerNetwork/computerNetworkCh2/image-20201201171549937.png)



## 2.3 文件传输协议：FTP

![FTP过程](/assets/img/computerNetwork/computerNetworkCh2/image-20201201192425699.png)

HTTP和FTP都是文件传输协议，都运行在TCP上，但是FTP使用了两个并行的TCP连接来传输文件，一个是**控制连接**，一个是**数据连接**。

- 控制连接用于在两主机之间传输控制信息，如用户标识、口令、该百年远程目录的命令以及"存放put"和"获取get"文件的命令。
  - 一般是21端口
- 数据连接用于实际发送一个文件。
  - 一般是20端口
- 由于FTP使用一个独立的控制连接，所以也称FTP控制信息是**带外**传送的。
- ![控制连接和数据连接](/assets/img/computerNetwork/computerNetworkCh2/image-20201201192816779.png)
- 对会话的每一次文件传输都需要建立一个新的数据连接(即非持续连接)，并且FTP服务器需要保留用户的**状态**。



FTP的命令和回答

- 每个命令由4个大写字母ASCII字符组成：
  - USER username：用于向服务器传送用户标识
  - PASS password：用于向服务器发送用户口令
  - LIST：用于请求服务器在数据连接上回送当前远程目录中的所有文件列表
  - RETR filename：用于从远程主机get文件
  - STOR filename：用户在远程主机上put文件
- 每个命令对应一个回答，是一个3位数字：
  - 331 Username OK, Password required
  - 125 Data connection already open; transfer strating
  - 425 Can't open data connetction
  - 452 Error writing file



## 2.4 因特网中的电子邮件

因特网电子邮件系统由3个主要组成部分：**用户代理、邮件服务器、简单邮件传输协议SMTP**

![image-20201201193501868](/assets/img/computerNetwork/computerNetworkCh2/image-20201201193501868.png)

**典型邮件发送过程：**发送方用户代理$\rightarrow$发送方的邮件服务器$\rightarrow$接收方的邮件服务器$\rightarrow$接收方的邮箱



### 2.4.1 SMTP

SMTP限制所有邮件报文的体部分只能采用简单的**7比特ASCII**表示，所以在用SMTP传送邮件之前，需要**将二进制多媒体数据编码为ASCII码**(HTTP不用！！)

发送报文过程：

1. Alice调用邮件代理程序并提供Bob的邮件地址，撰写报文，然后指示代理发送报文
2. Alice的代理把报文发给邮件服务器，将报文放在报文队列中
3. 运行在Alice的邮件服务器上的SMTP客户端发现该报文，创建一个到Bob的邮件服务器上的SMTP服务器的TCP连接
4. 在Bob的邮件服务器上，SMTP服务器端接收报文，放入Bob的邮箱
5. Bob方便的时候，调用代理阅读该报文

![发送报文过程](/assets/img/computerNetwork/computerNetworkCh2/image-20201201194431297.png)

> 例子：
>
> ![Alice通话历史](/assets/img/computerNetwork/computerNetworkCh2/image-20201201194542066.png)

每个报文由**CRLF.CRLF**结束，其中CR和LF分别表示回车和换行。

SMTP采用**持续连接**



### 2.4.2 与HTTP的对比

- 同：

  - 从一台主机向另一台主机传送文件
  - 持续的HTTP和SMTP都使用持续连接

- 异：

  - HTTP是一个**拉pull协议**，在Web服务器上装在信息，用户通过HTTP拉取

    SMTP是一个**推协议**，发送邮件服务器把文件推向接收邮件服务器

  - SMTP需要每个报文使用7比特ASCII字符

    HTTP没有限制

  - HTTP把每个对象封装到自己的HTTP响应报文中

    SMTP把所有报文对象放在一个报文中



### 2.4.3 邮件报文格式和MIME

![Email消息格式](/assets/img/computerNetwork/computerNetworkCh2/image-20201201195323196.png)

头部行Header必须要有To、From，可能有Subject

消息体body包含消息本身(ASCII码)

**MIME**：**多媒体邮件扩展 RFC 2045****，2056**（用于新增多媒体内容）

- 通过在邮件头部增加额外的行以声明MIME的内容类型
- ![MIME例子](/assets/img/computerNetwork/computerNetworkCh2/image-20201201195531220.png)



### 2.4.4 邮件访问协议

取报文是一个拉操作，不能使用SMTP协议。

因此引入一些邮件访问协议来完成取报文操作：**第三版的邮局协议 POP3、因特网邮件访问协议 IMAP、HTTP**

![电子邮件协议及其通信实体](/assets/img/computerNetwork/computerNetworkCh2/image-20201201195746787.png)

1. POP3

   - 当用户代理打开一个到邮件服务器端口**110**上的TCP连接，POP3就开始工作

   - POP3有三个阶段：

     - 特许
       - 用户代理明文发送用户名和口令，以鉴别用户
       - 命令：user, pass
     - 事务处理
       - 用户代理取回报文
       - 同时，对报文做删除标记，取消报文删除标记，以及获取邮件的统计信息
       - 命令：list、retr、dele、quit
     - 更新
       - 在用户发出quit命令后，结束该POP3会话
       - 此时，邮件服务器删除那些被标记为删除的报文

   - 回答：+OK，-ERR

   - “下载并删除”模式（更换客户端软件，无法重读）

     “下载并保持”模式（不同客户端都可以保留消息的拷贝）

2. IMAP

   - 为了实现建立邮件文件夹，在邮件文件夹中删除报文、移动报文、查询报文的功能，需要IMAP提供船舰远程文件夹并为报文指派文件夹的方法。
   - IMAP具有允许用户代理获取报文组件的命令，例如指读取一个报文的首部，或指示一个多部分MIME报文的一部分。

3. 基于Web的电子邮件



## 2.5 DNS：因特网的目录服务

### 2.5.1 DNS提供的服务

人们喜欢便于记忆的主机名标识方式，而路由器则喜欢定长的、有层次结构的IP地址$\rightarrow$需要一种能进行主机名到IP地址转换的目录服务，即**域名系统DNS**

DNS：

1. 一个由分层的DNS服务器实现的分布式数据库
2. 一个使得主机能够查询分布式数据库的应用层协议
   - DNS服务器通常是运行在BIND软件的UNIX机器，DNS协议运行在UDP之上，使用**53**号端口



> 考虑某个用户主机上的一个浏览器请求URL www.someschool.edu/index.html：
>
> - 同一台用户主机上运行着DNS应用的客户端
> - 浏览器从URL中抽取出主机名 www.someschool.edu，并传给DNS应用的客户端
> - DNS客户向DNS服务器发送一个包含主机名的请求
> - DNS客户最终会收到一份回答报文，其中含有对应该主机名的IP地址
> - 一旦浏览器接收到来自DNS的该IP地址，向位于该IP地址80端口的HTTP服务器进程发起TCP连接



DNS的其他服务：

- **主机别名**：拥有复杂主机名的主机能够拥有一个或者多个别名。
- **邮件服务器别名**
- **负载分配**：繁忙的站点被冗杂分布在多台服务器上，每台服务器均运行在不同端系统上，每个都有着不同的IP地址。DNS数据库中存储所有IP地址集合，客户发出DNS请求时，该服务器用IP地址的整个集合进行相应，但在每个回答中循环这些地址次序，客户通常总是向IP地址排在最前面的服务器发送HTTP请求报文。



### 2.5.2 DNS工作机理概述

DNS的一种简单设计时在因特网上只使用一个DNS服务器，包含所有的映射，但是有以下问题：

- **单点故障**。DNS服务器崩溃会造成整个因特网瘫痪
- **通信容量**。需要处理所有的DNS查询
- **远距离的集中式数据库。**对部分地区远距离
- **维护**。为所有因特网主机保留数据

实际上DNS是分布式的：

![DNS分布式数据库结构](/assets/img/computerNetwork/computerNetworkCh2/image-20201201202810562.png)



1. 分布式、层次数据库

   - 根DNS服务器

     - 本地域名服务器解析服务器无法解析域名时，当问根域名服务器

       根域名服务器如果不知道映射，访问权威域名服务器；否则获取映射，并向本地域名服务器返回

   - 顶级域名(TLD) DNS服务器

     - 负责**com, org, net, edu**等顶级域名和国家顶级域名，如**cn, uk, fr**等

   - 权威DNS服务器

     - 权威域名服务器是组织的域名解析服务器，提供组织内部服务器的解析服务
     - 组织和服务提供商负责维护

   **本地DNS服务器**严格来说不数据服务器的层次结构，但每个ISP都有一个。当主机进行DNS查询时，查询被发送到本地域名服务器。

   > **查询**
   >
   > - 迭代查询：
   >
   >   ![迭代查询](/assets/img/computerNetwork/computerNetworkCh2/image-20201201203719875.png)
   >
   > - 递归查询：
   >
   >   ![递归查询](/assets/img/computerNetwork/computerNetworkCh2/image-20201201203745421.png)

2. DNS缓存

   - 为了改善时延性能并减少在因特网上到处传输的DNS报文数量
   - 原理：本地DNS服务器暂时(一般2天)缓存DNS回答信息在本地存储器中



### 2.5.3 DNS记录和报文

所有DNS服务器存储了四元组**资源记录 RR**：(Name, Value, Type, TTL)

- Type = A，Name是主机名，Value是主机名对应的IP地址
- Type = NS，Name是一个域，Value是知道如何获取该域中主机IP地址的权威DNS服务器的主机域名
- Type = CNAME，Name是真实域名的别名，Value是真实域名
- Type = MX，Value是Name对应的邮件服务器



1. DNS报文

   - DNS有查询和回答报文，报文格式如下：

     ![DNS报文格式](/assets/img/computerNetwork/computerNetworkCh2/image-20201201204818975.png)

   - 前12个字节是**首部区域**

     - **标识符**：16比特的数，用于标识该查询，匹配请求和回答
     - **标志**：含有若干标志
       - 1比特的"查询/回答"标志位指出报文是查询报文(0)还是回答报文(1)
       - 1比特的"权威的"标志位
       - 1比特的"希望递归"标志位

   - **问题区域**包含正在查询的信息

     - **名字字段**：被查询主机的名字
     - **类型字段**：指出有关改名字的被询问的问题类型

   - **回答区域**包含对最初请求的名字的资源记录，可以有多条RR

   - **权威区域**包含了其他权威服务器的记录

   - **附加区域**包含了其他有帮助的记录

2. 在DNS数据库中插入记录

   1. 向注册登记机构注册域名时，需要提供基本和辅助权威DNS服务器的名字和IP地址
   2. 注册登记机构确保将一个类型NS和一个类型A的记录输入TLD com服务器
   3. 确保用于Web服务器的类型A资源记录域用于邮件服务器的类型MX资源记录被输入权威DNS服务器

   ![例子](/assets/img/computerNetwork/computerNetworkCh2/image-20201201205912299.png)

## 2.6 P2P应用

### 2.6.1 P2P文件分发

1. P2P体系结构的扩展性

   > ![文件分发问题的示例图](/assets/img/computerNetwork/computerNetworkCh2/image-20201201210327928.png)
   >
   > $u_s$表示服务器接入链路上的上载速率，$u_i$表示第$i$对等方接入链路的上载速率，$d_i$表示了第$i$对等放接入链路的下载速率，$F$表示被分发的文件长度，$N$表示要获得的该文件副本的对等方的数量。
   >
   > 假设以C/S结构进行传输：
   > $$
   > t=\max\{NF/u_s, F/\min_\limits{i}(d_i)\}
   > $$
   > 假设以P2P结构进行传输：
   > $$
   > t=\max\{F/u_s, F/\min_\limits{i}(d_i), NF/(u_s+\sum_{i}u_i)\}
   > $$
   > 以$u_s=10u,\ u_i=u,\ F/u=1h,\ d_{min}\ge u_s$条件下以$N$为变量画$N-t$图：
   >
   > ![N-t图](/assets/img/computerNetwork/computerNetworkCh2/image-20201201211153080.png)

2. BitTorrent

   > 参与一个也顶文件分发的所有对等方的集合被称为一个洪流 torrent。在一个洪流中的对等方彼此下载等长度的文件块 chunk (典型的块长度为256KB)。
   >
   > 当一个对等方首次加入一个洪流时，没有块。随着时间流逝，累积越来越多的块。
   >
   > 当它下载块时，也为其他对等方上载了多个块。
   >
   > 该对等方获得整个文件后，可以离开或者留在该洪流。
   >
   > 
   >
   > ![BitTorrent分发文件](/assets/img/computerNetwork/computerNetworkCh2/image-20201201211853688.png)
   >
   > 当一个新的对等方Alice加入洪流时，追踪器随机地从参与对等方的集合中选择对等方的一个子集(设50个)，并将这50个对等方的IP地址发送给Alice。
   >
   > Alice试图域该列表上的所有对等方创建并行的TCP连接
   >
   > - Alice成功创建一个TCP连接的对等方为"邻近对等方"(图中为3个)
   >
   > 随着对等方中某些可能离开，其他对等方(最初50个以外的)可能试图与Alice创建TCP连接(对等方的邻近对等方将随时间而波动)
   >
   > 
   >
   > **获取chunk**
   >
   > - 给定任一时刻，不同节点持有文件不同chunk集合
   > - 节点（Alice）定期查询每个邻居所持有的chunk列表
   > - 节点发送请求，请求获取缺失的chunk**（以稀缺优先）**
   >
   > 
   >
   > **发送chunk**
   >
   > - Alice向四个邻居发送chunk**（选取正在向其发送chunk中速率最快的4个，每10秒重新评估）**
   > - 每30s随机选择一个其他节点，向其发送chunk**（新选择节点可能加入top4）**
   > - 上传速率高，则更能找到更好的交易伙伴，从而更快的获取文件

3. 索引技术

   索引：信息到节点位置（IP地址+端口号）的映射

   <left><img src="/assets/img/computerNetwork/computerNetworkCh2/image-20201201213113942.png" alt="image-20201201213113942" style="zoom:100%;" /></left>

   - **集中式索引**

     - 内容和文件传输时分布式的，但是内容定位时高度集中式的。
     - 存在**单点失效问题、性能瓶颈、版权问题。**
     - ![集中式索引](/assets/img/computerNetwork/computerNetworkCh2/image-20201201213308884.png)

   - **洪泛式查询：Query flooding**

     - 完全分布式系统。每个节点对它共享的文件进行索引，且只对它共享的文件进行索引。
     - ![泛洪式查询](/assets/img/computerNetwork/computerNetworkCh2/image-20201201213417098.png)

   - **层次式覆盖网络**

     - 每个节点或者是一个**超级节点**，或者被**分配一个超级节点**

       节点和超级节点间、某些超级节点对间维持TCP连接

       超级节点负责跟踪子节点的内容

     - ![层次式覆盖网络](/assets/img/computerNetwork/computerNetworkCh2/image-20201201213500699.png)

     - 例如**Skype**：

       ![Skype](/assets/img/computerNetwork/computerNetworkCh2/image-20201201213540307.png)

     - 本质是P2P的：用户/节点对之间直接通信

       应用层协议是**私有**的

       **层次式覆盖网络**

        索引负责**维护用户名与****IP****地址间的映射**

       索引分布在**超级节点**上



### 2.6.2 分布式散列表

构造一个C/S结构的数据库，以在一个中心服务器中存储的所有(key, value)对。在该P2P系统中，每个对等方将保持(key, value)对仅占总体的一个小子集。我们允许任何对等方用一个特别的键来查询该分布式数据库。分布式数据库将定位拥有该相应(key, value)对的对等方，然后向查询的对等方返回该(key, value)对。任何对等方也将允许在数据库中插入新键-值对。这样一种分布式数据库被称为**分布式散列表 DHT**

1. 环形DHT
   - 考虑将对等方组织成一个环，每个对等方仅与它的直接后继和直接前任联系
   - ![确定最邻近键](/assets/img/computerNetwork/computerNetworkCh2/image-20201202102903808.png)
     - 现在3要确定谁负责键11，绕环顺时针发送报文，直到12
     - 通过捷径可以大大减少用于处理查询的报文数量
2. 对等方扰动
   - 为了处理对等方扰动，要求每个对等方联系其第一个和第二个后继。
   - 要求每个对等方周期性地验证它地两个后继是存活的。
   - 这样当某个对等方离开，其前继会寻找前继的第二后继进行连接建立
   - 加入的时候从1开始顺时针询问前继后继



## 2.7 TCP套接字编程

### 2.7.1 UDP套接字编程

![使用UDP的C/S应用程序](/assets/img/computerNetwork/computerNetworkCh2/image-20201202103614601.png)



### 2.7.2 TCP套接字编程

TCPServer进程有两个套接字，一个用于获取连接请求，另一个用于连接：

![TCPServer的两个套接字](/assets/img/computerNetwork/computerNetworkCh2/image-20201202103750425.png)

![使用TCP的C/S应用程序](/assets/img/computerNetwork/computerNetworkCh2/image-20201202103821535.png)



### 2.7.3 一些C语言Socket函数补充

> WSAStartup：初始化socket库
>
> WSACleanup：清楚/终止socket库的使用
>
> socket：创建套接字
>
> connect："连接"远程服务器(客户端用)
>
> closesocket：释放/关闭套接字
>
> bind：绑定套接字的本地IP地址和端口号(通常客户端不需要)
>
> listen：置服务器端TCP套接字为监听状态，并设置队列大小(仅用于服务器端TCP套接字)
>
> accept：接收/提取一个连接请求，创建新套接字(仅用于服务器端TCP套接字)
>
> recv：接收数据(用于TCP套接字或连接模式的客户端UDP套接字)
>
> recvfrom：接收数据报(用于非连接的UDP套接字)
>
> send：发送数据(用于TCP套接字或连接模式的客户端UDP套接字)
>
> sendto：发送数据报(用于非连接的UDP套接字)
>
> setsocketopt：设置套接字选项参数
>
> getsocketopt：获取套接字选项参数