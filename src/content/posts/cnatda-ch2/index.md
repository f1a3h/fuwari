---
title: "CNATDA 第二章学习笔记"
published: 2023-10-29
description: 'Principles of Network Applications'
image: ''
tags: [Computer Network, CNATDA]
category: 'Study Notes'
draft: false 
lang: ''
---

:::note[INFO]
「Computer Networking: A Top Down Approach」(8th Edition)

因为老师讲的太快没办法细读书上的内容，也很难有时间认真思考写一下一篇有深度的（虽然 CNATDA 好像介绍的深度也不是很大）学习笔记，所以这篇笔记很大程度上借鉴了 [ouuan](https://ouuan.moe/post/2023/06/cnatda-2) dalao 的笔记。
:::

## Principles of Network Applications

现代主要使用的 application architecture 有两种：client-server 和 peer-to-peer (P2P) .

*process* 可以认为是一个在 end system 上运行的 program ，不同 end system 上的 process 通过在网络上交换 *messages* (application layer packets) 进行通信。

process 与 transport layer 之间的 API 是 socket 。

host 使用 IP address 识别，process (socket) 之间通过 IP + port number 识别。

transport layer 中最主要的两个 protocol 是 TCP 和 UDP 。TCP 提供 connection-oriented service (通过 handshaking 建立 connection) 、reliable data transfer (data sent without error and in order) 、congestion-control (控制 sending rate 来减少 network layer 的拥塞以减少丢包重传) ，而 UDP 则是 connectionless (no handshaking) 、unreliable (不保证 message 送到和是否有序) ，并且 sending rate 是很随意的，也可以说 UDP 提供的是「best effort」service 。

application layer 在选择 transport layer protocol 的时候需要根据自身是否 loss-tolerant 、time-sensitive 来考虑。

application-layer protocol 决定了 process 之间交换的 messages 的 type, syntax, semantics, rules 。需要注意的是，application-layer protocols 只是 network application 中的一个部分。

## The Web and HTTP

HTTP (Hyper Text Transfer Protocol) 是 Web 的 application-layer protocol 。

HTTP 不维护 client 的信息，是一个 *stateless protocol* 。

HTTP/1.0 、HTTP/1.1 、HTTP/2 使用的都是 TCP ，而 HTTP/3 使用的是 UDP 。

由于 TCP 是 connection-oriented ，所以需要考虑每一对 req/res 需要建立在不同还是相同的 connection 上，从而分为了 *non-persistent connection* 和 *persistent connection* 两种。

- non-persistent: 每一次 req/res 都会建立一个新的 TCP connection ，结束之后都会关闭 connection 。需要的时间是 RTT * 2 (handshaking + req/res) + 文件传输用时。
- persistent: 同一对 client-server 之间的所有 req/res 共用一个 TCP connection ，并且 client 在收到 response 之前可以一次性发送多个 request (*pipelining*) ，省下了多次建立 TCP connection 的时间。

HTTP/1.0 使用的是 non-persistent connection ，HTTP/1.1 和 HTTP/2 使用的是 persistent connection 。

HTTP message 是 ASCII text 。

HTTP request:

![](./1.png)

```http
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

HTTP response:

![](./2.png)

```http
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
(data data data data data ...)
```

server 可以通过 cookies 来识别用户，通过在 response 的 header lines 中加入 `Set-cookie` 来发送用户的 cookies，之后 client 每次发送 request 则在 header lines 中加入 `cookie: 1678` 。

ISP 可以设置 Web Cache ，用户先向 Web Cache 发送 request ，如果 cache hit 则直接发送 response ，否则就由 web cache 先向 origin server 发送 request 然后再向用户发送 response 。Web Cache 的存在可以减少延迟，降低带宽压力。

client 通过在 header 中加入 `If-Modified-Since:<date>` 进行 *conditional GET* ，如果 server 检查对应的文件在 `<date>` 之前没有更改过，就发送 body 为空的 304 Not Modified 。

HTTP/2 使用了 request and response multiplexing、prioritization、server push 来提高 server 端发送的灵活性。

- multiplexing: 使用 persistent connection 虽然减小了 RTT 的次数，但是带来了 Head of Line (HOL) blocking (较小的文件需要花费大量时间等待较大文件传输完成才能传输) 。HTTP/1.1 尽管使用 persistent connection ，但是还是建立了多个 TCP connection 来避免 HOL ，而 HTTP/2 则是将文件分成较小的 *frames* ，交替发送不同文件的 frame (interleaving) 从而减少了小文件的等待时间。
- prioritization: client 发送多次 request 时，可以为不同的 response 设置优先级，server 通过优先级调整 response 的发送顺序，另外，client 还可以设置 response 之间的依赖关系。
- push: server 可以对一个 request 发送多个 response ，将当前页面引用的但是 unrequested 的其他资源也 push 过去。

HTTP/3 使用基于 UDP 的 QUIC protocol 替代了 TCP 。

## Electronic Mail in the Internet

mail system 主要由 user agent、mail servers 和 SMTP 组成。

当 Alice 发送 email 给 Bob 时，会将写好的邮件通过她的 user agent 传递给她的 mail server ，再由 mail server 发送给 Bob 的 mail server ，Bob 通过他的 user agent 从 mail server 中取出邮件进行阅读。

发送端的 mail server 会维护一个 message queue ，如果要发送的 mail server 在当时不可用，则会隔一段时间重新发送，多次失败后则会退回邮件。

发送端的 mail server 即为 SMTP client ，接收端的 mail server 则是 SMTP server 。

SMTP 是一个上古协议，因此整个 message 都只能是 7bit-ASCII 格式，同时，它使用的也是 TCP 协议，默认端口是 25 。

```plain text
S: 220 smtp.example.com ESMTP Postfix
C: HELO relay.example.org
S: 250 Hello relay.example.org, I am glad to meet you
C: MAIL FROM:<bob@example.org>
S: 250 Ok
C: RCPT TO:<alice@example.com>
S: 250 Ok
C: RCPT TO:<theboss@example.com>
S: 250 Ok
C: DATA
S: 354 End data with <CR><LF>.<CR><LF>
C: From: "Bob Example" <bob@example.org>
C: To: "Alice Example" <alice@example.com>
C: Cc: theboss@example.com
C: Date: Tue, 15 Jan 2008 16:02:43 -0500
C: Subject: Test message
C:
C: Hello Alice.
C: This is a test message with 5 header fields and 4 lines in the message body.
C: Your friend,
C: Bob
C: .
S: 250 Ok: queued as 12345
C: QUIT
S: 221 Bye
```

SMTP 在建立完 TCP connection (handshaking) 之后会先自我介绍一遍 (`HELO`) 再传输数据，他们之间的连接使用的是同一个 TCP connection 。

email 的传递是 user agent -> mail server -> mail server -> user agent ，多一个 mail server 作为中间媒介是因为 user agent 经常不在线，一个 mail server 的存在可以提高在线率并实现失败重发。

user agent 发送到 mail server 的过程可以使用 SMTP/HTTP ，mail server 到 mail server 使用 SMTP ，而从 mail server 拉取邮件到 user agent 使用的是 HTTP/IMAP ，不能使用 SMTP 是因为 SMTP 是一个 *push protocol* 。

## DNS——The Internet's Directory Service

host 可以由 hostname 和 IP 识别，其中 hostname 对人类更友好，而 IP 则对 routers 更友好。

DNS (Domain Name System) 的主要任务就是将 hostname 翻译成 IP 。

### Services Provided by DNS

DNS 是由不同层级的 DNS servers 共同构成的一个 distributed database ，也是一个让 hosts 可以请求这个 distributed database 的 application-layer protocol 。

DNS 使用 UDP 协议，默认端口为 53 .

DNS 在提供 hostname 到 IP 的翻译之外，还提供了以下功能：

- host aliasing: 一个 host 在拥有 canonical hostname 之外还能拥有多个 alias 。
- mail server aliasing: 同一个 hostname 在作为 Web server 和 mail server 时可以分别指向不同的 host 。
- load distribution: 同一个 hostname 可以指向多个 host ，返回查询结果的时候 rotate addresses 的顺序。

### Overview of How DNS Works

单点式的 DNS 会面临以下问题：

- a single point of failure
- traffic volume
- distant centralized database
- maintainance

因此，DNS 必须是分布式、多层级的。

DNS 一般分为以下几层：

- root DNS server: 分布在全球的 13 个 root servers 一共有超过 1000 个 copy ，用来查询 TLD server。
- top-level domain (TLD) servers: 每个 TLD 都有自己的 TLD server ，用来查询 authoritative DNS servers。
- authoritative DNS servers: 每个 subdomain 都有自己的 authoritative DNS server ，用来查询 hostname 到 IP 的映射。

除此之外，TLD servers 和 authoritative DNS servers 之间还可能有 intermediate DNS server (IXP) 。

另外，ISP 一般都会有一个 local DNS server (default name server) ，作为 proxy 代替 host 向 DNS server 进行查询。

在向 DNS server 查询时，分为 iterative query (分别向不同的 DNS server 查询) 和 recursive query (由当前查询的 DNS server 替你向其他 DNS server 查询) ，实际中，requesting host 到 local DNS server 的查询是 recursive 的，而 local DNS server 从 root DNS server 开始向下查询是 iterative 的。

为了减少查询的数量，DNS 采用了 caching 的办法。每个查询的发起者都会将查询结果保存一段时间，直到 cache miss 才会发起查询。

### DNS Records and Messages

DNS distributed database 的存储单元是 RR (resource record) 。

RR 是一个四元组 (Name, Value, Type, TTL) ，TTL 表示 cache 多久过期，Name 和 Value 的含义由 Type 决定。常见的 Type 有以下几种：

- A: name = hostname, value = IP address, 代表一个 hostname 到 IP 地址的映射。
- NS: name = domain, value = name server 的 hostname ，代表可以在这个 name server 进行这个 domain 的下一步查询。
- CNAME: name = alias hostname, value = canonical hostname ，用来使用 host aliasing 。
- MX: name = alias hostname, value = canonical hostname ，用来使用 mail server aliasing 。

hostname 对应的 authoritative DNS server 会包含对应的 A record ，而其他的 authoritative DNS servers 则会包含它的 NS record 。

下面的例子来自 [ouuan's blog](https://ouuan.moe/post/2023/06/cnatda-2#video-streaming-and-content-distribution-networks) 。

| type |           name            |           value           |
| :--: | :-----------------------: | :-----------------------: |
|  NS  |            `.`            |   `a.root-servers.net.`   |
|  A   |   `a.root-servers.net.`   |       `198.41.0.4`        |
|  NS  |          `moe.`           |    `ns1.dns.nic.moe.`     |
|  A   |    `ns1.dns.nic.moe.`     |     `156.154.144.114`     |
|  NS  |       `ouuan.moe.`        | `amos.ns.cloudflare.com.` |
|  A   | `amos.ns.cloudflare.com.` |      `172.64.35.120`      |
|  A   |       `ouuan.moe.`        |     `172.67.181.123`      |

DNS 最早只能通过更改配置文件等方式静态更新，现在可以使用 DDNS 通过 DNS message 进行动态更新。

DNS message 格式如下：

![](./3.png)

identification 表示这是 query 还是 reply ，由 client 设置。

flags 包括：

- query/reply: 表示这条 message 是 query 还是 reply 。
- authoritative or not: 表示返回的结果是否是最终结果。
- recursion desired: client 是否希望 server 进行 recursive query 。
- recursion available: 表示 server 是否可以进行 recursive query 。

## Peer-to-Peer File Distribution

P2P 架构的优势在于它是 self-scalable 的。

最流行的 P2P file distribution protocol 是 BitTorrent ，它以 chunk 为基本下载单位。在一个 peer 刚加入 torrent 时只能从其他 peer 下载 chunk，获取到一些 trunk 后就会上传给其他 peer 。

每个 torrent 都会有一个 tracker ，peer 的加入和离开都会通知 tracker ，并且会在过程中定期告知 tracker 自己仍在活动，tracker 会给每个 peer 其他 peer 的 IP 地址与端口。

## Video Streaming and Content Distribution Networks

上课的时候老师没讲，虽然我后来还是把这部分内容看完了，但是时间原因，学习笔记就先咕了，后面有空的时候再写。

~~CDN 还是挺重要的为什么不讲？~~