---
title: "CNATDA 第四章学习笔记"
published: 2023-11-08
description: 'The Network Layer: Data Plane'
image: ''
tags: [Computer Network, CNATDA]
category: 'Study Notes'
draft: false 
lang: ''
---

你口安计网怎么内容又多课时又少考的还早啊（恼）

~~第三章讲的太快没来及看书，就先咕咕咕了。~~

现在补完第三章了

## Overview of Network Layer

Network layer 可以细分成两个部分：data plane 和 control plane。

data plane 的主要功能是 *forwarding*（也叫做 *switching* ），也就是由 router 决定将 input link 收到的数据转发到哪个 output link。

router 中使用一个 forwarding table，根据 packet header 中的某些 fields 来 index forwarding table，从而得到 outgoing link interface。

control plane 的主要功能是 *routing*，即决定从 sending host 到 receiving host 到路径。

计算 forwarding table 也是 control plane 的任务，它有两种实现方式：

- the traditional approach：router 之间通过 router protocol 进行通信，然后使用 routing algorithm 计算得到 forwarding table。
- the SDN approach：router 只进行 forwarding，routing 由一个 remote controller 完成——router 向 remote controller 发送信息，由 remote controller 计算得到 forwarding table 发送给 router。这个 remote controller 通常由软件实现，这种方法称为 *software-defined networking*。

## What's Inside a Router?

router  包含以下几个部分：

- input ports:
  - incoming link 的 physical layer 和 link layer
  - look up:
    - 从 forwarding table 查找对应的 output port
    - 将 control packets forward 到 routing processor
  - 存在 input queue
- switching fabric: 连接 input ports 和 output ports
- output ports:
  - outgoing link 的 physical layer 和 link layer
  - 存在 output queue
- routing processor: 执行 control-plane 的功能，计算 forwarding table，进行 network management
  - traditional: 执行 routing protocols
  - SDN: 与 remote controller 进行通信

input ports、output ports 和 switching fabric 通常使用硬件实现，这样使得 forwarding 的速率在 ns 级从而保证通信速率。而 control plane 的功能一般只需保持在 ms 或 s 级，可以用软件实现。

### Input Port Processing and Destination-Based Forwarding

forwarding table 一般由 routing processors 更新或者接收自一个 remote SDN controller。

多个 input ports 可以合并在一个 line card 上，forwarding table 会从 routing processor 给每个 line card 复制一份，从而可以在每个局部分别计算。

destination-based forwarding 的 forwarding table 一般以 IP 地址前缀为 index，link interface 为 value 进行 longest prefix matching。

lookup 需要在 ns 级别实现，而 forwarding table 一般很大，因此还需要使用特殊的算法或存储器。

input ports 的 processing 还需要包括 physical-layer 和 link-layer processing，对 packets 的 version number、checksum、TTL 的检查与更改，对 network management 的 couters 进行更新。

### Switching

switching 有多种形式：

- via memory: 从 input port 复制到 memory 再复制到 output port，如果使用集中的 memory 而不是每个 line card 单独的 memory 会导致传输速率收到 memory 的速率限制，而且一次只能传输一个 packet
- via a bus: 将 packet 加上一个 label 然后通过 bus 传输到所有的 output port，根据 label 决定是否留下，传输速率收到 bus 的速率限制，一次只能传输一个 packet
- via an interconnection network: 每个 input port 对应一个 bus，每个 output port 对应一个 bus，每对 input port bus 和 output port bus 之间都有 crosspoint，通过控制 crosspoint 来控制从哪传到哪。这是 non-blocking 的，只要两个 pakcet 的 output port 不同就能同时传输。

### Input Queuing

如果 switching fabric 的速率大于所有 input port 的速率之和，则一般不会发生 input queuing，但是如果它们的 output port 都相同的话还是会存在 queuing。

以 switching via interconnection network 为例，一个 packet 即使没有和它 output port 相同的 packet 也可能因为 input queue 中在它前面的其他 packet 而被 block，即 HOL blocking。

### Output Queueing

如果 packet 到达 output port 的速率超过了 output line 的速率则会发生 output queuing。

当 packet 到达 output port 的时候 buffer 满了则需要考虑丢弃哪个 packet。在 buffer 满之前进行 pakcet dropping 或者 marking 称为 active queue management (AQM)。

### How Much Buffer Is "Enough" ?

存在 $N$ 个 TCP connection flow 经过一个带宽为 $C$ 的 link 时，需要的 buffer 大小为 $\mathrm{B=RTT\cdot C/\sqrt{N}}$。

拥有更大的 buffer 虽然能减少 loss，但是也会带来更大的 queuing delay。TCP 可能会使得 buffer 一直不被清空，从而导致 queuing delay 是 constant 且 persistent 的，这被称为是 *bufferfloat*，可以通过采取 AQM 来减缓。

### Packet Scheduling

主要有以下几种：

- FIFO (First In First Out)
- priority queuing: 优先级高的先开始传输
- weight fair queuing (WFQ): 给每个 class 一个权重来依次传输，保证这个 class 的传输频率是 $\frac{w_i}{\sum w}$

## The Internet Protocol (IP)

现在使用最多的 Internet protocol 是 IPv4 ，同时 IPv6 也在逐渐取代 IPv4 。

### IPv4 Datagram Format

IPv4 的 datagram 格式如下。

![](./IPv4.png)

- version number: 长度为 4 bit ，表示 IP 协议的版本。
- header length: 由于非定长的 options field 的存在，需要用 header length 表示 header 的长度，但是一般 options 为空，所以 header length 一般为 20 byte 。
- type of service (TOS) : 用来区分不同类别的 datagram 。
- datagram length: header + data 的长度。
- identifier、flags、fragmentation offset: 较大的 datagram 会被分成一些较小的 datagram 并分别 forward ，这些是用来识别它们的。
- time-to-live: 每被一个 router 加工就会减少 1 ，成为 0 的时候就会被丢弃，用来防止死循环。
- protocol: 只会在到达终点的 network layer 时使用，表明应该交给哪个 transport layer 的协议。
- header checksum: 检验 bit errors ，header 部分每 2 bytes 累加，得到的结果再取反，检查是否与存储在这个 field 的值相同。若不同，则直接丢弃。
- source and destination IP address: 起点与终点的 IP 地址。
- options: 允许 IP header 得以拓展，但是很少被使用到。
- data (payload): 包括 transport layer 的 segment ，也可以是其他类型的 data ，如 ICMP messages 。

如果 options 为空，则一个 IP header 长度是 20 bytes ，如果使用 TCP 协议，则这个 datagram 的总的 header 长度为 40 bytes 。

### IPv4 Addressing

一个 host 和他的 physical link 之间的边界就是 *interface* ，类似的，一个 router 和它每一个连接的 link 之间的边界也是一个 interface 。也就是说，其实相比于一个 IP 地址识别的是一个 host ，它识别的是一个 interface 的说法更准确。

IP 地址长度为 32 bits ，用 *dotted-decimal notation* 表示。

host interfaces 和它们连接的 router 的 interface 共同组成一个 *subnet* ，也称为 *network* 。

IP 地址按照 `223.1.1.0/24` 的形式分配给 subnet ，其中 `24` 称为 *subnet mask* ，是 IP 地址匹配过程中至少需要匹配的 prefix 的长度，而 `223.1.1.0` 的前 24 bits 就是这个 subnet 的地址。

Internet 分配 IP 地址的策略是 *Classless Interdomain Routing (CIDR)* 。在 CIDR 被采用之前，最常采用的是 *classful addressing* ，subnet mask length 只能为 8、16、24 bits ，分别称为 class A, B, C network ，导致不同 subnet 可分配地址数量差异过大，难以按需分配。

longest prefix matching 使得 address aggregation 成为可能：拥有相同 prefix 的 subnet 可以合并成一个更大的 subnet 。

`255.255.255.255` 是一个特殊的 IP 地址，表示 broadcast 地址。destination address 为 broadcast 地址的 datagram 会被发送给 subnet 内的所有 host ，也可能发送给 neighboring subnet 。

#### Obtaining Addresses

##### Obtaining a Block of Addresses

IP address block 由 ICANN 管理并分配给 ISP 和一些 organizations ，它们得到之后还可以将 IP address block 更细分然后分配给用户。

##### Obtaining a Host Address: DHCP

router 的 IP 地址一般是手动设置的，而 host 的地址则是通过 Dynamic Host Configuration Protocol (DHCP) 获得的。

每个 subnet 都会有至少一个 DHCP server 或者一个知道 DHCP server address 的 DHCP relay agent (一个 router ) ，由它们来给 host 提供**临时**的 IP 地址。

DHCP 获取 IP 地址的步骤如下：

- DHCP server discovery: 新来的 host 通过 UDP 发送给 `255.255.255.255` ，port 67 ，使用 transaction ID 作为 identifier 。
- DHCP server offer(s): DHCP server 收到 discover message 后发送 offer message 给 `255.255.255.255` ，port 68 ，包含 yiaddr 、transaction IP 、DHCP server IP 、Lifetime 。
- DHCP request: host 收到 DHCP offer 后选择其中一个作为 IP 地址，然后以 broadcast 的方式发送 DHCP request，与 DHCP offer 具有相似的结构。
- DHCP ACK: DHCP server 收到 DHCP request 后发送 DHCP ACK 。

### Network Address Translation (NAT)

subnet 需要一段连续的 IP 地址，如果设备数量超过了原先分配的 IP address block 的数量就难以分配新的地址，于是引入了 Network Address Translation (NAT) 的方法来解决这个问题。

使用 NAT 时，subnet 内部使用 IP address space reserved for private network （10.0.0.0/8、172.168.0.0/16、192.168.0.0/16），用一个 router 与外界连接并进行 NAT ，这个 router 对外界表现为 a single device with a single IP address，通过 NAT translation table 在 private address + port 与 WAN-side address + port 之间进行转换（给每个 host 以及 application port 分配一个 NAT port 进行区分）。这个 router 的 public address 从 ISP 获得，同时它作为 DHCP server 为 subnet 内部提供 private address 。

NAT 同时也带来了一些问题，比如 subnet 内的 port 一般会变，而某些时候需要特定的不变的 port ，这个问题可以通过 NAT traversal 来解决。

### IPv6

为了解决 IPv4 地址即将被耗尽的问题，人们发明了 IPv6 ，除了将地址长度从 32 bits 扩展到了 128 bits ，IPv6 还解决了 IPv4 中其他一些问题。

IPv6 datagram 的主要格式如下：

![](./iShot_2023-11-05_17.21.02.png)

主要变化有：

- 在 unicast 和 broadcast 的基础上引入了 anycast address，允许了向多个地址之一发送 datagram。
- 使用定长为 40 bytes 的 header。
- 引入 flow table 使得 router 可以对 flow 进行特殊处理。
- 删除 fragmentation 提高性能，如果 datagram 太大则直接丢弃并发送相应的 ICMP error message。
- 删除 header checksum 冗余功能提高了效率。
- 删除 options ，其中 next header 不一定是 transport layer protocol ，也可以是 option。
- TOS 改为 traffic class ，TTL 改为 hop limit ，datagram length 改为 payload length（不包含 header length），protocol 改为 next header。

由于 IPv4 到 IPv6 的转换仍在进行中，为了兼容旧的设备，采用 *tunneling* 将 IPv6 的 datagram 作为 IPv4 的 payload 进行传输，其中两个 IPv6 router 之间的 IPv4 router 被称为 *tunnel* 。

## Generalized Forwarding and SDN

Generalized Forwarding 基于 “match-plus-action” 的原则，相对于 destination-based forwarding，“match” 时还可以考虑 IP header field 中 destination 以外的 fields ，也可以考虑 link-layer header、transport-layer header 等，而 “action” 除了 forward 还可以 drop、修改 header field 等。

OpenFlow 是一个 generalized forwarding 的协议，规定了 match 时可以/不能使用哪些 field，以及可以采取哪些 action。

## Middleboxes

network 中除了 forwarding 之外用来实现一些其他功能的设备称为 *middlebox*，它们主要提供一下几种功能：

- NAT translation
- security services，如 firewall
- performance enhancement，如 Web cache

middlebox 在一定程度上破坏了 network 层的 layered architecture ，也就是 abstraction 的概念。