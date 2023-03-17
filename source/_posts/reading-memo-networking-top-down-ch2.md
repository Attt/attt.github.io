---
title: Computer Networking - Transport Layer
date: 2023-03-16 22:58:13
categories:
    - 基础姿势
    - 读书笔记
tags:
    - networking
    - tcp/ip
password: Transport Layer
---

>好记性不如流水账

## Introduction and Transport-Layer Services

![](reading-memo-networking-top-down-ch2/截屏2023-03-17%2010.42.56.png)

### Relationship Between Transport and Network Layers

IP提供了hosts间的逻辑沟通，TCP/UDP提供了processes间的逻辑沟通

### Overview of the Transport Layer in the Internet


## Multiplexing and Demultiplexing

![](reading-memo-networking-top-down-ch2/截屏2023-03-17%2011.07.13.png)

multiplexing: 从不同的sockets获取数据，将数据封装发送到network-layer

demultiplexing: 识别network-layer来的数据，分发到不同的socket

port number

**Connectionless Multiplexing and Demultiplexing**

相同二元组（dest IP + dest port）的segment会发送到相同的socket

**Connection-Oriented Multiplexing and Demultiplexing**

相同四元组（dest IP + dest port + src IP + src port）的segment会发送到相同的socket

**Web Servers and TCP**
## Connectionless Transport: UDP
### UDP Segment Structure
### UDP Checksum
## Principles of Reliable Data Transfer
### Building a Reliable Data Transfer Protocol
**Reliable Data Transfer over a Perfectly Reliable Channel: rdt1.0**
**Reliable Data Transfer over a Channel with Bit Errors: rdt2.0**
**Reliable Data Transfer over a Lossy Channel with Bit Errors: rdt3.0**
### Pipelined Reliable Data Transfer Protocols
### Go-Back-N (GBN)
### Selective Repeat (SR)
## Connection-Oriented Transport: TCP
### The TCP Connection
### TCP Segment Structure
**Sequence Numbers and Acknowledgment Numbers**
**Telnet: A Case Study for Sequence and Acknowledgment Numbers**
### Round-Trip Time Estimation and Timeout
**Estimating the Round-Trip Time**
**Setting and Managing the Retransmission Timeout Interval**
### Reliable Data Transfer
**A Few Interesting Scenarios**
**Doubling the Timeout Interval**
**Fast Retransmit**
**Go-Back-N or Selective Repeat?**
### Flow Control
### TCP Connection Management
## Principles of Congestion Control
### The Causes and the Costs of Congestion
**Scenario 1: Two Senders, a Router with Infinite Buffers**
**Scenario 2: Two Senders and a Router with Finite Buffers**
**Scenario 3: Four Senders, Routers with Finite Buffers, and Multihop Paths**
### Approaches to Congestion Control
## TCP Congestion Control
### Classic TCP Congestion Control
**Slow Start**
**Congestion Avoidance**
**Fast Recovery**
**TCP Congestion Control: Retrospective**
**TCP Cubic**
**Macroscopic Description of TCP Reno Throughput**
### Network-Assisted Explicit Congestion Notification and Delayed-based Congestion Control
**Explicit Congestion Notification**
**Delay-based Congestion Control**
### Fairness
**Fairness and UDP**
**Fairness and Parallel TCP Connections**
## Evolution of Transport-Layer Functionality
**QUIC: Quick UDP Internet Connections**
