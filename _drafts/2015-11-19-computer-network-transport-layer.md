---
layout: blog
categories: reading
title:  《计算机网络》笔记 - 传输层
tags: ALOHA LAN WAN TCP UDP Socket Wifi 蓝牙
---

## 传输服务

### 向上层提供的服务

* 传输实体：传输层内部，利用网络层提供的服务，向它的用户提供高效、可靠和性价比合理的服务的硬件/软件
* 传输服务提供者、传输服务用户

<!--more-->

## 信道分配问题

* LAN与MAN中静态信道分配方案：多用FDM，只能满足少数目拥有繁重任务的用户。
* LAN与MAN中动态信道分配方案：5个假设：站（终端）模型、单信道假设、冲突假设、持续时间/分槽时间、载波监测/无载波检测。

## 多路访问协议

### ALOHA

* 纯ALOHA：随时发送，检测到冲突（或未收到确认）则等待随机时间，尝试再次发送
* 分槽ALOHA：等待下一时槽开始时才发送

### 载波检测多路访问协议

* 1-持续CSMA（carrier sense multiple access）：持续检测信道，空闲则立即发送，冲突后等待随机时间后，再次检测和发送
* 非持续的CSMA：检测到信道空闲则发送，信道占用则等待随机的时间后重复算法
* p-持续CSMA：检测到空闲，p的可能性发送，1-p的可能性延迟到下一时槽，重复该算法
* 冲突检测的CSMA（CSMA/CD，CSMA with collision detection）：传送过程中检测到冲突则立即放弃任务

### 无冲突的协议

* 位图协议（一种预留协议，reservation protocol）：每个竞争周期包含N个时槽，j号站要发送则在j时槽传送1位（此时槽j号站专有），这是所有站按照此队列进行传送，都结束后开始另一个N位竞争周期。
* 二进制倒计数（binary countdown）协议：要传送的站以二进制传送自己地址（从高位开始），然后取或运算，如果一个站看到自己的高位被改为1则放弃传送。

### 有限竞争协议

* 负载轻时竞争方法更理想；负载重是无冲突协议更好，结合起来形成有限竞争协议（limited-contention protocol）
* 自适应树搜索协议（深度优先）：0号时槽，所有站尝试获得信道；若冲突则1号时槽只有左支可以竞争；直到没有冲突，左支某站获得信道，下一时槽只允许右枝竞争。

### 波分多路访问协议 WDMA（wavelength division multiple access）

* 一个固定波长接收器：监听控制信道
* 一个可调波长发送器：在其他站的控制信道发送信息
* 一个固定波长发送器：发送数据
* 一个可调波长接收器：选择监听一个数据发送器

### 无线LAN协议

* 隐藏站问题：A检测到B而检测不到C，错误滴认为可以与B通信
* 暴露站问题：A检测到介质中B与C的通信，错误的认为不能与D通信
* MACA（multiple access with collision avoidance，避免冲突的多路访问）：A给B发送RTS（request to send），B以CTS（clear to send）应答，这两帧均包含数据长度；此时听到RTS和CTS的站保持足够时间的沉默。
* MACAW（MACA for wireless）：引入ACK帧，及时重传。

## 以太网

### 以太网电缆

* 10Base5 粗同轴电缆（粗以太网），使用插入式分接头，在10Mbps上可支持500m
* 10Base2 细同轴电缆（细以太网），BNC连接器（T型接头），这两种通过时间域反射计检测故障。
* 10BaseT 双绞线，通过集线器连接
* 10BaseF 光纤，支持上千米。每一版本的网络都可以通过中继器（repeater）扩大范围。
* 100BaseT4 3类UTP
* 100BaseTX 全双工，5类UTP
* 100BaseFX 全双工，长距离，两根多模光纤
* 1000Base-SX 多模光纤
* 1000Base-LX 单模或多模光纤
* 1000Base-CX 两对STP（shield twisted pair，屏蔽双绞线）
* 1000Base-T 四对5类UTP


### 编码

* 曼彻斯特编码：1（高电压+低电压）、0（低电压+高电压）
* 差分曼彻斯特编码：间隔起始处没有相变为1，相变为0

### 太网的性能

* 二元指数后退法：第i次冲突后，随机等待0~2^i-1个时槽

* 交换式以太网：核心为交换机，每块插卡构成自己的冲突域

### 逻辑链路控制子层

* LLC（logical link control）：IEEE 802.2，与MAC构成数据链路层

## 无线LAN

### 802.11物理层

* 红外线：1M和2Mbps
* FHSS（frequency hopping spread spectrum，调频扩频），使用79个信道，停延时间小于400ms，对多径衰减有很好的抵抗能力
* DSSS（direct sequence spread spectrum，直接序列扩频）：使用巴克序列，每一位需要11个时间片
* OFDM（orthogonal frequency division multiplexing，正交频分多路复用）：802.11a，54Mbps，52个频率
* HR-DSSS（high rate direct sequence spread spectrum，高速率的直接序列扩频）：802.11b，支持1、2、5.5、11Mbps；802.11g为802.11b的增强版本，理论速度为54Mbps。

### 802.11 MAC子层

* DCF（distributed coordination function，分布式协调功能），使用CSMA/CA协议，支持两种操作方法
+ 信道监听：空闲则送出整个帧，冲突后采用二元指数后退法计算等待时间
+ 虚拟信道监听：以MACAW为基础，NAV（network allocation vector，网络分配向量）

* PCF（point coordination function，点协调功能）：周期性广播信标帧（调频和停延时间、时钟同步等）
* PCF与DCF同时运行，4种间隔：
+ SIFS（short interframe spacing，短帧间间隔）：允许一个会话中各部分有机会首先被送达
+ PIFS（PCF interframe spacing，PCF帧间间隔）：若SIFS后得到授权的站未开始传送，则基站可能广播信标帧或表决帧
+ DIFS（DCF interframe spacing，DCF帧间间隔）：若基站未有动作，任一站开始尝试得到信道，冲突采用二元指数后退法
+ EIFS（extended interframe spacing，扩展帧间间隔）：收到坏帧和未知帧的站使用这个间隔

### 服务

* 分发服务：基站提供
* 关联：连接到基站
* 分离：解除与基站的联系
* 重新关联：改变首选基站
* 分发：如何路由发送给基站的帧
* 融合：翻译非802.11网络的帧
* 站服务：单元内部进行
* 认证：联系基站，确认新进入的站通过了基站的认证
* 解除认证：离开网络
* 私密性：指定使用RC4加密算法
* 数据投递：参考以太网的模型，不保证可靠性，由上层处理检错和纠错

## 宽带无线网络

* 802.16：无线MAN，或无线本地回路，包括数据链路层与物理层
* 带宽分配：FDD（frequency division duplexing，频分双工制）；TDD（time division duplexing，时分双工制）

## 蓝牙技术 802.15

* 微微网：蓝牙系统的基本单元。微微网中有一个主节点，可以有7个活动的从节点，255个静观节点。
* 分散网：一组互相连接的微微网，通过当作桥的从节点连接
* 应用轮廓：一般访问、服务发现、串行端口、一般的对象交换、LAN访问……
* 协议栈：物理层（物理无线电、基带）->数据链路层（基带、链路控制）->中间件层（电话、服务发现）->应用程序（应用轮廓）

## 数据链路层交换

* 802.x到802.y的网桥：在LLC层进行翻译
* 本地网络互连：网桥使用扩散算法、逆向学习法
* 生成树网桥：，采用序列号生成树，低位变为根；解决并行的透明网桥产生的回路
* 远程网桥：采用PPP协议，将完整的MAC帧放入净荷域
* 交换设施区别：物理层（转发器、集线器）；数据链路层（网桥、交换机）；网络层（路由器）；传输层（传输网关）；应用层（应用网关）
*虚拟LAN（VLAN）
+ 通过网桥或交换机的配置表来路由
- 每个端口分配一个VLAN颜色：VLAN所有机器在统一端口才可以
- 每个MAC地址分配一个VLAN颜色：从帧中提出MAC地址进行匹配
- 每个3层协议或IP分配一个VLAN颜色：须检查净荷域
+ IEEE 802.1Q：改变以太网的帧头、可理解VLAN的交换机


### 传输服务原语

* LISTEN、CONNECT、SEND、RECEIVE、DISCONNECT
* TPDU（transport protocol data unit，传输协议数据单元）：从传输实体发送到另一个传输实体的消息。

### 伯克利套接字（berkeley socket）

* Berkeley UNIX使用的TCP socket原语：SOCKET、BIND、LISTEN、ACCEPT、CONNECT、SEND、RECV、CLOSE

## 传输协议的要素

### 编址

* 为监听连接请求的进程定义相应的传输地址，Internet：端口（port）；ATM：AAL-SAP；一般术语：TSAP（transport service access point，传输服务访问点）
* 同样，网络层的端点称为NSAP（network service access point，网络服务访问点），IP为一个特例。
* 初始连接协议：采用进程服务器为那些较少使用的服务器提供代理功能（监听一组端口），请求到达时启动相应的服务器，继承与客户的连接。
* 名字服务器（目录服务器）：在特殊硬件运行，不能在用户通话时临时创建

### 建立连接

* 分组生存周期：受限制的子网设计、分组设置跳计数器、分组时间戳
* TPDU编号，相等编号的TPDU不会同时有效，序列号空间应足够大，序列号回绕时同样编号的TPDU都已经消失

### 释放连接

* 两军队问题

### 流控制和缓冲

### 多路复用

* 向上多路复用：传输实体将TPDU交给上层指定的进程
* 向下多路复用：将流量分布到多个网络连接

### 崩溃恢复

## 有限状态机描述传输协议

## Internet 传输协议-UDP

### UDP介绍

* UDP（User Datagram Protocol，用户数据报协议）数据段由8字节头和净荷域构成。
* UDP头：源端口、目标端口、UDP长度、UDP校验和

### 远过程调用

* RPC（remote procedure call，远过程调用）：让远程调用像本地过程调用一样，客户程序需要绑定一个小的库过程（客户存根），服务器程序绑定一个服务器存根。
* 参数包装：列集（marshaling）

### 实时传输协议

* RTP（real-time transport protocol，实时传输协议）：RFC 1889
* 用户空间（多媒体应用->RTP）->套接字接口->操作系统内核（UDP->IP->以太网）
* RTCP（realtime transport control protocol，实时传输控制协议）：处理反馈、同步和用户界面等。

## Internet 传输协议-TCP

* TCP（transport control protocol，传输控制协议）：在不可靠的互联网络提供一个可靠的端到端字节流。
* TCP服务模型：16位端口，1024以下为知名端口
* 守护进程（unix中称inetd，internet daemon）：同时关联到多个端口，连接进入时fork出新的进程
* 紧急数据：URGENT标记
* TCP数据段（TCP segment）；MTU（maximum transfer unit，最大传输单元），以太网通常为1500字节净荷。
* TCP数据段（包括20字节的头）：源端口、目标端口、序列号、确认号、TCP长度、URG、ACK、PSH、PST、SYN、FIN、窗口大小、校验和、紧急指针、可选项、数据
* TCP连接的建立：SYN(SEQ=x)->SYN(SEQ=y, ACK=x+1)->SYN(SEQ=x+1, ACK=y+1)
* TCP传输策略：Nagle算法，数据进入发送方后只发送第一字节，其后的字节缓冲起来，直到第一字节被确认为止。
* TCP拥塞控制：网络容量（拥塞窗口）、接收方容量（接收方准许窗口）、慢启动法、阈值
* TCP定时器管理：重传定时器、持续定时器（确认超时的探询消息）、保活定时器（一段时间后发送，对方不应答则终止连接）
* 无线TCP和UDP：间接TCP、同质的网络
* 事务型TCP：用UDP来实现RPC的高效率，用TCP实现可靠性，T/TCP（transaction TCP，事务型TCP）；SCTP（stream control transmission protocol，流控制传输协议）

## 性能问题
广播风暴、带宽-延迟乘积

### 网络性能的测量

* 确保样本空间足够大
* 确保样本具有代表性
* 使用粗粒度时钟时要谨慎
* 确保在测试过程中不会发生不可预知的事情
* 缓存机制可能会破坏测量的正确性
* 理解所测量的指标
* 在往外推广结果时要谨慎

### 具有更好性能的系统设计

* CPU速度比网络速度更加重要
* 减少分组的数量可以减少软件开销
* 使环境切换的次数减到最少
* 使复制操作的次数减到最少
* 可以购买更多的带宽，但无法要求更低的延迟
* 应该想办法避免拥塞，而不是从拥塞中恢复
* 避免超时

### 快速的TPDU处理

* 分离出正常的发送操作病对它们作特殊处理

### 针对千兆网络的协议

旧的协议面临的问题

* 32位序列号：对于1Gbps的以太网，一轮序列号回绕时间为34s，大于internet上最大分组生存周期120s
* 通信速度的提高比计算速度的提高快得多
* 如果一条线路的带宽-延迟乘积非常大，则回退n步协议的性能会非常差
* 千兆位线路本质上不同于1兆位线路，因为长的千兆位线路的主要制约因素是延迟，而不是带宽
* 对于许多千兆位应用，分组到达时间的偏差与平均延迟本身一样重要


