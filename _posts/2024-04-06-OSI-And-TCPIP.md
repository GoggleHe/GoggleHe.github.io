---
layout: post
title: OSI七层模型与TCP/IP四层模型
categories: [网络编程]
description: 网络编程
keywords: IO,OSI,TCP,IP
---

## OSI七层模型与TCP/IP四层模型

### OSI七层模型

- 应用层（Application）
- 表示层（Presentation）
- 会话层（Session）
- 传输层（Transport）
- 网络层（Network）
- 数据链路层（Data Link）
- 物理层（Physical）

![OSI七层模型示意图](E:\repository\github\GoggleHe.github.io\images\posts\osi-model.png)

第 7 层应用层 (Application Layer)

**主要功能：** 针对特定应用的协议，如邮件协议、文件传输协议、远程登录协议(ssh)、网络请求协议(HTTP)
**典型设备：** 网关
**典型协议、标准和应用：** http(80)、ftp(20/21)、smtp(25)、pop3(110)、telnet(23)、dns(53)

第 6 层表示层 (Presentation Layer)

**主要功能：** 定义了数据的解码和编码，数据的加密和解密，数据的压缩和解压缩
**典型设备：** 网关
**典型协议、标准和应用：** ASCLL、PICT、TIFF、JPEG、 MIDI、MPEG

第 5 层会话层 (Session Layer)

**主要功能：** 建立、维护、管理应用程序之间的会话（如RPC协议）
**典型设备：** 网关
**典型协议、标准和应用：** RPC、SQL、NFS 、X WINDOWS、ASP

第 4 层传输层 (Transport Layer)

**主要功能：** 在网络层（如Mac地址和IP地址）的基础上，引入端口，以区别不同的计算机程序（如进程）
**典型设备：** 网关
**典型协议、标准和应用：** TCP、UDP、SPX

第 3 层网络层 (Network Layer)

**主要功能：** 定义与区分了不同的计算机地址，比如IP地址
**典型设备：** 路由器
**典型协议、标准和应用：** IP、IPX、APPLETALK、ICMP

第 2 层数据链接层 (Data Link Layer)

**主要功能：** 定义了物理层二级制数据的分组方式，如一段有意义的数据有多长，数据头从几位到几位，数据体几位到几位，有什么含义等
**典型设备：** 交换机、网桥、网卡
**典型协议、标准和应用：** 802.2、802.3ATM、HDLC、FRAME RELAY

第 1 层物理层 (Physical Layer)

**主要功能：** 利用传输介质为数据链路层提供物理连接，实现比特流的透明传输，即网线、电缆、无线电波等物理连接，这一层引入Mac地址，属于网卡设备的固有属性
**典型设备：** 集线器、中继器
**典型协议、标准和应用：** V.35、EIA/TIA-232

### TCP/IP 协议族

#### 常用协议

**应用层：** TFTP，HTTP，SNMP，FTP，SMTP，DNS，Telnet 等等
**传输层：** TCP，UDP
**网络层：** IP，ICMP，OSPF，EIGRP，IGMP
**数据链路层：** SLIP，CSLIP，PPP，MTU

#### 四层模型

![TCPIP四层模型](E:\repository\github\GoggleHe.github.io\images\posts\tcpip-model.png)



#### TCP生命周期

![TCP三次握手与四次挥手生命周期示意图](E:\repository\github\GoggleHe.github.io\images\posts\tcp-handshake-wavehand-lifecycle.png)

##### 三次握手

- **第一次握手：** Client 将标志位 syn 设置为 1，随机产生一个 Number 值 seq=100，并将数据发送给 Server，Client 进入 SYN_SENT 状态，等待 Server 确认；
- **第二次握手：** Server 收到数据包后 Client 设置的标志位 syn=1 知道 Client 要求建立连接，Server 将标志位 syn 和 ack 都置为 1，并且发送一个确认序号 ack=100+1，然后随机产生一个值 seq=130，并将该数据包发送给 CLient 以确认连接请求，Server 进入 SYN_RCVD 状态。
- **第三次握手：** Client 收到确认后，检查 ack 状态是否为 100+1，ACK 是否为 1，如果正确则将标志位 ACK 置为 1，ack=130+1，并将该数据包发送给 Server，Server 检查 ack 是否为 130+1，ACK 是否为 1，如果正确则连接建立成功，Client 和 Server 进入 ESTABLISHED 状态，完成三次握手，随后 Client 与 Server 之间可以开始传输数据了。

一个完整的三次握手也就是**请求 — 应答 — 再次确认**

##### 四次挥手

- **第一次挥手：** Client 发送一个 FIN，用来关闭 Client 到 Server 的数据传送，Client 进入 FIN_WAIT_1 状态。
- **第二次挥手：** Server 收到 FIN 后，发送一个 ACK 给 Client，确认序号为 ack=100+1（与 SYN 相同，一个 FIN 占用一个序号），Server 进入 CLOSE_WAIT 状态。
- **第三次挥手：** Server 发送一个 FIN，用来关闭 Server 到 Client 的数据传送，Server 进入 LAST_ACK 状态。
- **第四次挥手：** Client 收到 FIN 后，Client 进入 TIME_WAIT 状态，接着发送一个 ACK 给 Server，确认序号为 131+1，Server 进入 CLOSED 状态，完成四次挥手。

#### Q/A

##### 1. 为什么建立连接是三次握手，而关闭连接却是四次挥手呢？

这是因为服务端在 LISTEN 状态下，收到建立连接请求的 SYN 报文后，把 ACK 和 SYN 放在一个报文里发送给客户端。而关闭连接时，当收到对方的 FIN 报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即 close，也可以发送一些数据给对方后，再发送 FIN 报文给对方来表示同意现在关闭连接，因此，己方 ACK 和 FIN 一般都会分开发送。

##### 2. 为什么建立连接要三次握手？

**目的：** 防止已经失效的连接请求到达服务端，创建无效的连接，浪费资源。
**说明：** 当客户端发出的第一个连接请求在网络上的某个节点被滞留了（网络会存在许多不可靠的因素），过一段时间后突然又到达了服务端，服务端误以为这是一个新的建立连接的请求，于是就会向客户端发出确认包并建立连接。
实际上客户端当前并没有发出创建连接的请求，就会丢弃服务端的确认包。而服务端却创建了连接并等待客户端发送数据，浪费了相关的资源。

##### 3. SYN 攻击

在三次握手过程中，服务器`发送SYN-ACK之后，收到客户端的ACK之前`的 TCP 连接称为半连接 (half-open connect)。此时服务器处于 SYN_RECV 状态，当收到 ACK 后，服务器转入 ESTABLISHED 状态.

SYN 攻击就是：攻击客户端在短时间内伪造大量不存在的 IP 地址，向服务器不断地发送 SYN 包，服务器回复 ACK 确认包，并等待客户的确认从而建立连接。由于源地址是不存在的，不会再发送 ACK 确认包，所以服务器需要不断的重发直至超时，这些伪造的 SYN 包将长时间占用未连接队列，正常的 SYN 请求被丢弃，目标系统运行缓慢，严重者引起网络堵塞甚至系统瘫痪。

SYN 攻击是一个典型的 DDOS 攻击。检测 SYN 攻击非常的方便，当你在服务器上看到大量的半连接状态时，特别是源 IP 地址是随机的，基本上可以断定这是一次 SYN 攻击

##### 4. 为什么 TIME_WAIT 状态需要经过 2MSL (最大报文段生存时间) 才能返回到 CLOSE 状态？

虽然按道理，四个报文都发送完毕，我们可以直接进入 CLOSE 状态了，但是我们必须假象网络是不可靠的，有可以最后一个 ACK 丢失。所以 TIME_WAIT 状态就是用来重发可能丢失的 ACK 报文。

[OSI 七层模型与 TCP/IP 四层模型]: https://wsgzao.github.io/post/osi/

