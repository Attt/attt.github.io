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

#### Client and Server Processes

如何定义client process和server process：
> In the context of a communication session between a pair of processes, the pro- cess that initiates the communication (that is, initially contacts the other process at the beginning of the session) is labeled as the client. The process that waits to be contacted to begin the session is the server.

为了简化模型client和server的分别只是看哪一方先发起communication session，因此在P2P架构中也有client和server，只不过peer可以扮演两种角色。（这里要区别“full-duplex”和“half-duplex”，这个概念指的是能否同时进行，而client process和server process讨论的是角色）

#### The Interface Between the Process and the Computer Network

process通过叫做“socket”的software interface发出消息到网络（对于process来说消息给到socket之后确实就属于“网络”了，因为process并不用关心细节），也通过socket来接受消息。

socket是software层面的定义，本质上是将消息转移到下一层（通常是transport-layer），一般来说下一层的实现都由OS来负责，因此也可以理解socket是OS获取打包好的application-layer消息并且进一步打包成transport-layer消息（传出）、拆解transport-layer消息取出并送达application-layer消息（传入）的API。（通常对于transport-layer，应用程序只能选择使用何种协议）

![](reading-memo-networking-top-down-ch1/截屏2023-03-14%2021.51.57.png)

#### Addressing Processes

通过IP address来确定目标host，通过port number来确定目标host上的目标process。

### Transport Services Available to Applications

transport-layer提供的服务分为四个方面：可靠数据传输、吞吐量、即时性、安全性。

#### Reliable Data Transfer

具体问题具体分析，比如音视频流可能不太需要过于可靠（有点数据丢失或者错误问题不大），文件传输或者邮件之类的可能就必须保证可靠性。

#### Throughput

对速率bitrate有保障，比如指定码率的直播推流，就需要有一定的速率保障。

#### Timing

即时性，即时通讯、在线游戏之类的都需要保障一定的即时性，但邮件这种肯定不需要。

#### Security

不只是加密，还包括了完整性验证和身份验证。

### Transport Services Provided by the Internet

互联网提供了TCP和UDP两种协议（不是只有这两种！是互联网the Internet提供了这两种）。

![](reading-memo-networking-top-down-ch1/截屏2023-03-14%2022.06.50.png)

#### TCP Services

TCP协议提供的服务：
- Connection-oriented service
- Reliable data transfer service

面向connection有个好处就是可以保存会话状态，通过这个connection的概念来认定分别在两个host上的两个process间的消息和这些消息的状态buffer。也是在transport-layer保证可靠性的基础，同时可靠性就是保证数据的有序有效正确。

congestion-control mechanism，拥堵控制是用来保证整个网络的可用性和公平性。（TCP协议的研究基本就是在这个上面做文章）

安全性方面TLS

#### UDP Services

无connection，最基础的传输层协议，除了把消息发出去，基本上只有校验数据包完整性的功能。

#### Services Not Provided by Internet Transport Protocols



### Application-Layer Protocols
## The Web and HTTP
### Overview of HTTP
### Non-Persistent and Persistent Connections
#### HTTP with Non-Persistent Connections
#### HTTP with Persistent Connections
### HTTP Message Format
#### HTTP Request Message
#### HTTP Response Message
### User-Server Interaction: Cookies
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
