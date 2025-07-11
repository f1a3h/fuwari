---
title: "CNATDA 第三章学习笔记"
published: 2023-11-22
description: 'Transport Layer'
image: ''
tags: [Computer Network, CNATDA]
category: 'Study Notes'
draft: false 
lang: ''
---

## Introduction and Transport-Layer Services

transport layer 将 application layer messages 封装成 *segments*，然后交给 network layer 进行传输，将 network layer 提供的 logical communication between hosts 扩展为 logical communication between processes。

最主要的两个 transport layer protocol 是 TCP 和 UDP，其中 UDP unreliable、connectionless 的，TCP reliable、connection-oriented 的。

UDP 仅仅提供 data delivery 和 error checking 的服务，不保证 datagram 是否送达、是否有序，是否完整、正确，也不会控制传输速度。

TCP 则提供 reliable data transfer、flow control、保证 datagram 正确、有序送达，还会提供 congestion control。

而 network layer protocol 最重要的 IP protocol 提供的是一个 best-effort delivery (unreliable) 服务。

## Multiplexing and Demultiplexing

在 transport layer 中，*multiplexing* 指 sender 的 transport layer 将不同 socket 的 message 收集并发送给 network layer 的过程，而 *demultiplexing* 指 receiver 的  transport layer 将 network layer 发送来的 datagrams 拆分后传递给对应的 socket 的过程。


multiplexing 和 demultiplexing 需要每个 socket 有 unique identifier。

UDP socket 的 identifier 为二元组 `(dest IP addr, dest port num)` ，它也会存有 source port number 等其他 field，但是不作为 identifier。

TCP socket 的 identifier 为四元组 `(src IP addr, src port num, dest IP addr, dest port num)`。

## Connectionless Transport: UDP

UDP 仅提供 multiplexing/demultiplexing 和 error checking。一些 application 选择 UDP 主要基于以下几点原因：

- 更好地控制发送什么数据、什么时候发送：TCP 有 congestion control，可以会延迟发送数据，而且会错误重传导致发送较慢。
- 不需要建立连接：建立连接的过程会产生 delay。
- 无连接状态：可以减小系统资源消耗。
- 较小的 header：TCP segment header 长度通常为 20 bytes，而 UDP 只有 8 bytes。

但是，在使用 UDP 传输大量数据时，会导致 packet overflow at routers，UDP 丢包严重，TCP 传输过慢。

UDP datagram header 共有 4 个 fields，每个长度都是 2 bytes。

- length：记录 UDP segment 的总长度。
- checksum：对所有 16-bit 的数据求和后使用 1s complement 检测是否全为 1。
- source port
- destination port

UDP 的 checksum 仅仅进行 error detecting，不会进行 error correcting，当检测到错误时可以选择丢弃/通知 application。

尽管很多 link layer protocol 也提供了 error checking，但是不能保证路上所有的 link layer 都提供了，同时 error 也不一定是 link-to-link 的过程中产生的，因此 UDP 仍然需要提供 checksum 的服务。

## Principles of Reliable Data Transfer

### Stop-And-Wait

![](./rdt3.JPG)

在 stop-and-wait 中，sender 每次需要收到 receiver 发送的对应的 ACK/NAK 才会发送下一个 packet。

- checksum：传输过程可能出错，需要进行 error detection。
- acknowledgment：receiver 需要告知 sender 是否丢包、出现错误。
- retransmission：丢包（超时）或者收到 NAK 时需要重传。
- timeout：出现丢包现象需要使用 timeout 来进行判断。timeout 的时间太长会影响性能，所以一般设置为有可能丢包但是不能完全确定是否是丢包的值。
- sequence number：sender 发送的 packet 与 receiver 发送的 ACK 都有相应的 sequence number。在传输过程中，ACK 也有可能出错，从而导致 duplicate packet，因此加上 sequence number 来识别 duplicate packet。另外，在 stop-and-wait 中，sequence number 仅由 0/1 组成，因此又被称为 *alternating-bit protocol*。

### Pipelined Reliable Data Transfer

由于是 stop-and-wait 的机制，所以性能不高。

定义 sender 的 *utilization* 的大小为 sender 实际在传输的时间与总时间的比值，在 stop-and-wait protocol 中，大小约为 $U_{\mathrm{sender}}=\dfrac{L/R}{RTT+L/R}$ ，实际值非常小。

解决这个问题的办法就是让 sender 无需等待即可一次传输多个 packets，这种方法也叫 *pipelining*，使用这种办法也意味着：

- sequence number 数量增大。
- sender 和 receiver 都需要 buffer 更多的 packets。

有两种基础的实现方法：Go-Back-N 和 selective repeat。

### Go-Back-N (GBN)

![](./gbn_seqnum.jpeg)

window 的大小收到 packet 中 sequence number 的 field 的限制，若 sequence number field 的长度为 $k$，则 window size 为 $2^k$，sequence number 范围为 $[0,2^k-1]$。

![](./gbn_sender.jpeg)

sender:
- 使用一个 sliding window 表示当前有效的 sequence number 的范围。
- 如果 window 内所有 sequence number 都被使用了就不能发送新的 packet。
- 收到的 ACK 被称为 *cumulative acknowledgement*，即 sequence number 小于当前 ACK 的 number 的未被 ACK 的 packets 也会被视为 ACK'd 。
- 收到一个 ACK 就会将 window 滑到它对应 sequence number 后面。
- 所有的未被 ACK 的 packets 共用一个 timer。
- retransmit 时会重发所有未被 ACK 的 packets。

![](./gbn_receiver.jpeg)

receiver: 只接收顺序正确的 packet，否则会直接丢弃，而如果是顺序正确但是出现错误的 packet 则会通过发送上一个 packet 的 ACK 进行 NAK。

GBN 相对 selective repeat 的优势在于 receiver 不需要 buffer，但是相应的，一个 packet 出错就要重发所有未被 ACK 的 packets，造成了很多浪费。

### Selective Repeat (SR)

![](./sr.jpeg)

sender:

- window 除了第一个一定是未被 ACK 的其余都有可能是 ACK 或者未被 ACK 的。
- 每个 packet 的 ACK，timer，retransmission 都是独立的。
- 收到开头的 ACK 时 window 滑动到第一个未被 ACK 处。

receiver：

- 同样维护一个 sliding window，window size 可能与 sender 不同，第一个是尚未收到的最小的 packet。
- 收到非 window 开头的会 buffer 下来。
- 收到 window 开头的会将其与 buffer 中存储的连续的一整段传给 application layer，然后滑动 window。
- 收到比 window 开头小的 packet 时说明之前的 ACK 传输出了问题，仍然需要发送 ACK。
- 如果收到 packet 存在 error，则丢弃不管

需要注意的是 sender 和 receiver 的 window size 不一定相同，但是都不能超过 sequence number 大小范围的一半。

不管是 GBN 还是 SR，sequence number 的范围都是有限的，所以是循环使用的，如果一个 duplicate packet/ACK 的传输耗时过久导致占用了现有的 sequence number，实践中的解决办法是认定一个 packet/ACK 在传输几分钟之后就没了。

## Connection-Oriented Transport: TCP

### The TCP Connection

TCP 被称为 *connection-oriented* 是因为当两个 process 之间能够传输 data 之前需要先进行 handshake。

TCP 需要先建立 connection，这个 connection 只是 logical 的，是在两个 end system 上建立一些 state variable，没有 physical 上的连接。

TCP connection 是 *full-duplex* 的，即双方建立连接之后互相可以发送信息；是 *point-to-point* 的，不能进行 multicasting。

TCP connection 通过 *three-way handshake* 建立，在 handshake 过程中也会建立 *send/receive buffer*，发送/收到 message 时会先放入 send/receive buffer 然后再传递给 application/link layer。

TCP 会根据 MTU (maximum transmission unit，最大的能传输的 link layer frame size) 计算出合适的 MSS (maximum segment size，segment 包含 data 的最大 size) 使得 data 加上 TCP header 和 IP header 后不超过 MTU。

### TCP Segment Structure

![](./tcp_segment.jpeg)

TCP 的 header 长度通常是 20 bytes，但是 Options 是可变长度的，所以也不一定。

### Sequence Numbers and Acknowledgment Numbers

TCP 将 data 视为一个无结构但有序的 byte stream。*sequence number* 指的是当前 segment 的 data 的第一个 byte 在 byte stream 中的序号；*acknowledgment number* 指的是期待对方下一个传送来的 segment 的 sequence number。

由于 TCP 只会 acknowledge 它重新组成的 byte stream 中的第一个 missing byte，所以也被称为 cumulative acknowledgments。

当 TCP 收到 out-of-order segments 时，可以选择丢弃或者将其 buffer 下来并等待 missing bytes，显然后者对于网络带宽的利用更高效，也是实践中采用的办法。

为了尽可能减小收到的上一个已经关闭的 connection 中仍然留存在 network 中的 segment 的 sequence number 与现在合法的 sequence number 搞混，TCP 会随机选择一个初始的 sequence number。

### Round-Trip Time Estimation and Timeout

一个 segment 的 sample RTT ($s$) 指的是从发送到收到对应的 ACK 的用时。

TCP 一般在计算某个 segment 的 sample RTT 后不会再计算同时也在传输的其他 segment 的 sample RTT，而且只会计算一个就传输成功的 segment，不会计算 retransmitted segment 的 sample RTT。

由于 network 的拥堵情况会变化，所以另外还需要计算这些 sample RTT 的 EWMA (exponential weighted moving average) 作为 estimated RTT ($e$) 来平缓 RTT 的波动：$e=(1-\alpha)\cdot e+\alpha\cdot s$ 。

使用 dev RTT ($d$) 来衡量 RTT 的波动：$d=(1-\beta)\cdot d+\beta\cdot |s-e|$ 。

$\alpha$ 和 $\beta$ 的建议取值为 0.125 和 0.25 。

time interval ($t$) 在 estimated RTT 的基础上根据 dev RTT 提供了一定的冗余：$t=e+4\cdot d$ 。

### Reliable Data Transfer

TCP 一般只使用一个 timer，是用来给最早的未被 ACK 的 packet 计时的，并且每次 timeout 只会 retransmit 这一个 packet。

- doubling the timeout interval：每次发生 retransmission 时，TCP 会将下一个 time interval 翻倍，而不是由 estimated RTT 和 dev RTT 计算得到。但是一旦在收到 application 传来的 data 或者收到 ACK 后重启的 timer 的 time interval 的值仍然设置为 $t=e+4\cdot d$ 。

- fast retransmit：doubling the timeout interval 虽然能减小因顺序错误导致收到 duplicate ACK 的次数，但是增加了 end-to-end delay。由于多于 2 个 duplicate ACKs 可以表明错误原因不是顺序问题（较大 number 的 packet 先到达）而是丢包，因此 TCP 在收到 3 个 duplicate ACKs（相同 sequence number 的第四个 ACK）时会在 time out 前立即重发。

TCP 只维护最小的未被 ACK 的 sequence number (sender) 和顺序正确的 sequence number (receiver)，这点上看类似 GBN，但是 TCP 每次只会重传一个 packet，而 receiver 也会 buffer 顺序错误的 packets。

### Flow Control

sender 和 receiver 都会另外维护一个 buffer 存储 packets，sender 收到 application 的 data 会 buffer 下来不一定立即发送，receiver 收到 packet 也会 buffer 下来等待 application 读取。

为了防止 sender 发送过快导致 receiver 的 buffer overflow，TCP 提供了 flow control 的服务。

TCP 的 header 需要包含 *rwnd (receive window)* 用来表示 buffer 的剩余空间，sender 则需要保证发出的未被 ACK 的 packets 数量不超过 rwnd。

对于 receiver：$\mathrm{rwnd=RcvBuffer-[lastByteRcvd-LastByteRead]}$

对于 sender：$\mathrm{LastByteSent-LastByteAcked\le rwnd}$

当 $\mathrm{rwnd}=0$ 时会导致 sender 无法发送 packet 即使 receiver 之后拥有了可用 buffer，因此需要 sender 在这之后继续发送大小为 1 byte 的 segment，而 receiver 有了可用的 buffer 后会 ACK 这个 segment。

### TCP Connection Management

three-way handshake：

1. client 发送 SYN bit 为 1 的 SYN segment，并且 data 为空，rand 一个 initial sequence number (client_isn) 。
2. server 接收到 SYN segment 后分配 TCP buffer 和 variables 给这个 connection，然后发送 SYN bit 为 1，acknowledge number 为 client_isn + 1 ，data 为空的 SYNACK segment，并且 rand 一个 initial sequence number (server_isn) 。
3. client 收到 SYNACK segment 后分配 buffer 和 variables 给这个 connection，然后发送 SYN bit 为 0，acknowledge number 为 sever_isn + 1 ，data 为空的 ACK segment，并且可以添加 data。

断开连接时，双方都要互发 FIN bit 为 1 的 FIN segment，收到之后都需要发送对应的 ACK，并且提出断开连接的一方会等待一段时间后再断开连接来让对方有机会在 ACK 丢包时重传 FIN segment。

如果尝试连接一个不接受 TCP connection 的端口，会收到 RST bit 为 1 的 RST segment。

## Principles of Congestion Control

congestion 会导致：

- large queue delay
- router buffer overflow 导致丢包和 retransmission
- large delay 导致 premature timeout 从而导致 unneeded retransmission
- 传输过程中一个 router 出现丢包，则前面所有 router 的工作都浪费了

congestion control 主要分为两大类：

- end-to-end: network layer 不提供任何有关 congestion 的信息，仅由 retransmission 和 delay 的增加来推断出 congestion
- network-assisted: router 提供 congestion feedback（例如用一个 bit 表示是否 congested，或者提供更复杂的信息，如 maximum host sending rate），可以是由 router 发送一个新的 packet，也可以是 router 修改正在传输的某个 packet 的相关 field，然后由 receiver 通知 sender。

## TCP Congestion Control

### Classic TCP Congestion Control

sender 会使用一个 *congestion window ($\mathrm{cwnd}$)* 来限制发送的速率，使得 $\mathrm{LastByteSent-LastByteAcked\le \min\{cwnd, rwnd\}}$，从而发送的速率大约为 $\mathrm{cwnd/RTT}$.

classic TCP congestion control 主要有以下几个原则：

- 丢包意味着 congestion，因而 sender 应该要减小 $\mathrm{cwnd}$
- ACK 表明没有 congestion，因而 sender 可以增大 $\mathrm{cwnd}$
- 可以不断增大 $\mathrm{cwnd}$ 来试探什么时候会发送 congestion

*TCP congestion-control algorithm* 主要有 3 个部分：

- slow start: 初始状态以及 timeout 后到达的状态。从 $\mathrm{cwnd=1\,MSS}$ 开始，每收到一个 ACK 就增大 $\mathrm{1\,MSS}$，相当于每个 RTT $\mathrm{cwnd}$ 翻倍，直到 $\mathrm{cwnd\ge ssthresh}$ 进入 congestion avoidance。
- congestion avoidance: 每个 ACK 增大 $\mathrm{MSS\cdot (MSS/cwnd)}$，相当于每个 RTT 增大 $\mathrm{1\,MSS}$。
- fast recovery: 收到 3 个 duplicate ACK 后会进行 fast retransmit，之后转移到 fast recovery。从 $\mathrm{cwnd=ssthresh+3\, MSS}$ 开始，每次再收到一个 duplicate ACK 就增大 $\mathrm{1\, MSS}$，收到 new ACK 则转移到 congestion avoidance。

![](./tcp_congestion_control.jpeg)

正常情况下会在 congestion avoidance 和 fast recovery 之间反复切换，此时 cwnd 的增长是线性的，减少则是减半，称为 *additive-increase, multiplicateive-decrease (AIMD)*，AIMD 的 congestion control 会导致 cwnd 呈现锯齿状的变化。

上面描述的所有内容加起来是 *TCP Reno*，如果用 slow start 替代 fast recovery 则是更古老的 *TCP Tahoe*。

TCP Reno 的上升曲线是线性的，不能快速恢复到接近 congestion 的临界值，对带宽的利用不是很高效，*TCP CUBIC* 则是在此基础上进行优化后的版本。

记 $W_{max}$ 为 TCP 检测到丢包时的 cwnd 的大小，$k$ 为 cwnd 重新增长到 $W_{max}$ 的大概时间，当当前时间距离 $k$ 较远的时候则快速增加 cwnd，接近 $k$ 的时候则缓慢增加，即便超过了 $W_{max}$ 但是没有检测到丢包时也遵循同样的规则。

![](./tcp_reno_vs_cubic.jpeg)

根据 TCP Reno 的线性增长的特性可以计算出大致的平均 throughput 为 $\dfrac{0.75\cdot W}{RTT}$。

### Network-Assisted Explicit Congestion Notification and Delayed-based Congestion Control

*Explicit Congestion Control (ECN)* 是一种 network-assisted congestion control，它对 TCP 和 IP 的 header 都进行了扩展。

ECN 在 IP datagram header 中使用了两个 bit，一个给 router 来表示是否 congested（一般在实际发生前设置），另一个给 sender 设置来告诉 router 自己和 receiver 是否是 ECN-capable 的。

当 receiver 收到 router 的 congestion 信息后，会在 ACK 中设置 ECE (explicit congestion notification echo) flag，sender 收到后会将 cwnd 减半，然后在下一个 segment 中设置 CWR (congestion window reduced) flag。

### Delayed-based Congestion Control

delay-based congestion control 也能在丢包发生前就检测到 congestion。

TCP Vegas 会维护历史最大的 throughput ($\mathrm{cwnd/RTT}$)，如果当前的 throughput 小于这个值，则说明发生了 congestion。

### Fairness

假设当前有 $K$ 个 TCP connection 共用一个 transmission rate 为 $R$ 的 bottleneck link，并且没有其他数据传输。如果每个 connection 的平均 throughput 都是 $R/K$，则称使用的 congestion control mechanism 是 fair 的。

在各方的 RTT 相同的情况下，AIMD 是 fair 的，如下图所示。

![](./tcp_fairness.jpeg)

但是当各方 RTT 不同时，RTT 更小的往往可以获得更大的 throughput。

UDP 是没有 congestion control 的，会导致 unfairness。另外，application layer 可能会使用多个 parallel TCP connection 也会导致 unfairness。

## Evolution of Transport-Layer Functionality

现在有许多种不同的 TCP/UDP 版本，丢弃了某些原有的功能/新增加一些功能来适应新的环境。

QUIC 是一个基于 UDP 的 application layer protocol，具有以下 feature:

- connection-oriented and secure。需要 handshake 来建立连接，并且所有数据加密。connection-establishment handshake 和 authentication and encryption handshake 被合并在了一起，从而比 TLS 更快。
- stream。以 stream 为单位传输 application data，而多个 stream 可以放在单个 packet 中传输。
- reliable, TCP-friendly congestion-controlled data transfer。其中 in-order delivery 是对每个 stream 分别保序，而一个 stream 可以放在不同的 packet 中，所以不同 stream 之间不会带来阻塞 (HOL blocking)。另外也采取了与 TCP 类似的 congestion control。