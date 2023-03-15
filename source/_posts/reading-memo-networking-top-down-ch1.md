---
title: Computer Networking - Application Layer
date: 2023-03-13 23:21:31
categories:
    - 基础姿势
    - 读书笔记
tags:
    - networking
    - http
password: atpex24
---

>流水账笔记。

## Principles of Network Applications

应用层的程序编写不需要过多考虑网络的核心设备的运行，这是网络中各种各样应用发展快的原因。
![](reading-memo-networking-top-down-ch1/截屏2023-03-14%2021.10.56.png)

### Network Application Architectures

应用层程序架构
- client-server（中心化）
- peer-to-peer（无中心化）

client-server架构中，通常client互相不会通信，server处于持续运行状态且拥有固定的地址（域名、IP），因此所有的client都可以轻松访问获取内容，同时为了防止单个server出现单点故障，通常会在多个data center部署多个server。

P2P架构中，几乎不依赖data center中的server，不同的hosts间相互通信，称为peer。P2P架构的优势是成本低（不用专用的server）、self-scalability（自我扩展），与之相对的可靠性、安全性、性能也是其缺点。

### Processes Communicating

网络中讨论的process间的通讯指不同end system上的processes之间的通讯。*（这里可以复习一下本地processess间的通讯途径和实现）*

**Client and Server Processes**

如何定义client process和server process：
> In the context of a communication session between a pair of processes, the pro- cess that initiates the communication (that is, initially contacts the other process at the beginning of the session) is labeled as the client. The process that waits to be contacted to begin the session is the server.

为了简化模型client和server的分别只是看哪一方先发起communication session，因此在P2P架构中也有client和server，只不过peer可以扮演两种角色。（这里要区别“full-duplex”和“half-duplex”，这个概念指的是能否同时进行，而client process和server process讨论的是角色）

**The Interface Between the Process and the Computer Network**

process通过叫做“socket”的software interface发出消息到网络（对于process来说消息给到socket之后确实就属于“网络”了，因为process并不用关心细节），也通过socket来接受消息。

socket是software层面的定义，本质上是将消息转移到下一层（通常是transport-layer），一般来说下一层的实现都由OS来负责，因此也可以理解socket是OS获取打包好的application-layer消息并且进一步打包成transport-layer消息（传出）、拆解transport-layer消息取出并送达application-layer消息（传入）的API。（通常对于transport-layer，应用程序只能选择使用何种协议）

![](reading-memo-networking-top-down-ch1/截屏2023-03-14%2021.51.57.png)

**Addressing Processes**

通过IP address来确定目标host，通过port number来确定目标host上的目标process。

### Transport Services Available to Applications

transport-layer提供的服务分为四个方面：可靠数据传输、吞吐量、即时性、安全性。

**Reliable Data Transfer**

具体问题具体分析，比如音视频流可能不太需要过于可靠（有点数据丢失或者错误问题不大），文件传输或者邮件之类的可能就必须保证可靠性。

**Throughput**

对速率bitrate有保障，比如指定码率的直播推流，就需要有一定的速率保障。

**Timing**

即时性，即时通讯、在线游戏之类的都需要保障一定的即时性，但邮件这种肯定不需要。

**Security**

不只是加密，还包括了完整性验证和身份验证。

### Transport Services Provided by the Internet

互联网提供了TCP和UDP两种协议（不是只有这两种！是互联网the Internet提供了这两种）。

![](reading-memo-networking-top-down-ch1/截屏2023-03-14%2022.06.50.png)

**TCP Services**

TCP协议提供的服务：
- Connection-oriented service
- Reliable data transfer service

面向connection有个好处就是可以保存会话状态，通过这个connection的概念来认定分别在两个host上的两个process间的消息和这些消息的状态buffer。也是在transport-layer保证可靠性的基础，同时可靠性就是保证数据的有序有效正确。

congestion-control mechanism，拥堵控制是用来保证整个网络的可用性和公平性。（TCP协议的研究基本就是在这个上面做文章）

安全性方面TLS

**UDP Services**

无connection，最基础的传输层协议，除了把消息发出去，基本上只有校验数据包完整性的功能。

**Services Not Provided by Internet Transport Protocols**

> today’s Internet can often provide satisfactory service to time-sensitive applications, but it cannot provide any timing or throughput guarantees.

互联网不提供即时性和吞吐的保障。由于网络的复杂性，每次消息都会经过多个节点多个ISP无数条线路，基于成本很难保证这两个特性。不过应用程序（end system）可以曲线救国，通过增加线路、冗余发送、压缩等手段提高即时性和吞吐。

### Application-Layer Protocols

一个application-layer protocol定义了：

- 消息类型，比如说请求和响应
- 如何区分消息类型
- 消息中各个部分的含义
- process如何处理（接收、发送、解释）消息

举个例子，HTTP是RFC(Request for Comments)定义的公开协议，当然也可以定义私有协议。

## The Web and HTTP
### Overview of HTTP

基于**TCP**协议的**无状态**的*HyperText Transfer Protocol*，可靠完整由TCP来保证。（HTTP/3虽然基于UDP，因此一些TCP的保证特性需要在上一层也就是application-layer的HTTP/3中实现）

![](reading-memo-networking-top-down-ch1/截屏2023-03-15%2010.57.16.png)

### Non-Persistent and Persistent Connections

**HTTP with Non-Persistent Connections**

初代HTTP，每个请求都发起一次新的TCP连接

**HTTP with Persistent Connections**

HTTP/1.1开始，多个请求复用TCP连接

### HTTP Message Format

RFC：
- [Hypertext Transfer Protocol -- HTTP/1.0](https://www.rfc-editor.org/rfc/rfc1945)
- [Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing](https://www.rfc-editor.org/rfc/rfc7230)
- [Hypertext Transfer Protocol Version 2 (HTTP/2)](https://www.rfc-editor.org/rfc/rfc7540)
- [HTTP/2](https://www.rfc-editor.org/rfc/rfc9113)
- [HTTP/3](https://www.rfc-editor.org/rfc/rfc9114)


**HTTP Request Message**

```lua
GET /somedir/page.html HTTP/1.1 
Host: www.someschool.edu 
Connection: close
User-agent: Mozilla/5.0 
Accept-language: fr
```
1. HTTP消息是由ASCII文字编写
2. 第一行是request line(请求行)：格式是 [method] [url] [protocol/version], 接下来的都是header line(头行?)
3. Host指明了目标host的位置（用于web代理缓存，都是application-layer的事情。这里看似已经通过TCP和目标host建立连接，实际上由于HTTP是无状态的不同的TCP连接可能处理完全相同的HTTP请求）
4. Connection，close表示请求完成之后关闭连接，具体定义参照RFC：
>  o  If the "close" connection option is present, the connection will
>      not persist after the current response; else,
>
>   o  If the received protocol is HTTP/1.1 (or later), the connection
>      will persist after the current response; else,
>
>   o  If the received protocol is HTTP/1.0, the "keep-alive" connection
>      option is present, the recipient is not a proxy, and the recipient
>      wishes to honor the HTTP/1.0 "keep-alive" mechanism, the
>      connection will persist after the current response; otherwise,
> 
>   o  The connection will close after the current response.
1. User-agent用户代理，提供当前client的信息
2. Accept-language表示client能够接受的内容语言

![](reading-memo-networking-top-down-ch1/截屏2023-03-15%2014.01.32.png)

7. entity body是POST用的，GET方法参数是通过URL携带
8. 最开始的RFC里URL没有规定长度，但是不同的浏览器最低限制在2000chars，且CDN也会限制URL的长度

![](reading-memo-networking-top-down-ch1/截屏2023-03-15%2014.22.39.png)

![](reading-memo-networking-top-down-ch1/截屏2023-03-15%2014.25.06.png)

**HTTP Response Message**

```lua
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT 
Content-Length: 6821
Content-Type: text/html
(data data data data data ...)
```

1. 第一行是status line(状态行)，格式[protocol/version] [status code] + [status]，接下来是header line，最后是entity body
2. Connection和request时意义一样，这里是server告诉client接下来会关闭连接
3. Date是server发送响应的时间
4. Server提供server的信息
5. Last-Modified是资源最后被修改的时间（包括创建），这个对缓存至关重要
6. Content-Length内容的长度
7. Content-Type返回响应的资源类型

![](reading-memo-networking-top-down-ch1/截屏2023-03-15%2014.35.29.png)

### User-Server Interaction: Cookies

RFC:

[HTTP State Management Mechanism](https://www.rfc-editor.org/rfc/rfc6265)

最初设计是想用来标识用户身份，后来用于保存会话状态，本质是提供了一个手段让无状态的HTTP带上那么点状态。

### Web Caching



#### The Conditional GET
### HTTP/2
#### HTTP/2 Framing
#### Response Message Prioritization and Server Pushing
#### HTTP/3
## Electronic Mail in the Internet
### SMTP
### Mail Message Formats
### Mail Access Protocols
## DNS—The Internet’s Directory Service
### Services Provided by DNS
### Overview of How DNS Works
#### A Distributed, Hierarchical Database
#### DNS Caching
### DNS Records and Messages
#### DNS Messages
#### Inserting Records into the DNS Database
## Peer-to-Peer File Distribution
#### Scalability of P2P Architectures
#### BitTorrent
## Video Streaming and Content Distribution Networks
### Internet Video
### HTTP Streaming and DASH
### Content Distribution Networks
#### CDN Operation
#### Cluster Selection Strategies
### Case Studies: Netflix and YouTube
#### Netflix
#### YouTube
## Socket Programming: Creating Network Applications
### Socket Programming with UDP
### Socket Programming with TCP