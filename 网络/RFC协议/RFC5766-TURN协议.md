# RFC5766-TURN协议
[TOC]

## 摘要

如果一台主机处于NAT后面，那么在一定条件下两台主机无法之间进行通讯。在这种条件下，那么使用中继服务提供通讯是有必要的。
这个规范定义了一个名为TURN(使用中继穿越NAT)的协议，它允许一台主机使用中继服务与对端进行报文传输。TURN不同于其它中继协议在于它
允许客户机使用一个中继地址与多个对端同时进行通讯。

TURN协议也是ICE(交互式连接建立)协议的组成部分，也可以单独使用。

## 1、简介
一个处于NAT内的主机想要与其他主机进行通讯，其他主机有可能也处于NAT内。为了实现这个目的，
主机利用“打洞”的技术用于发现直接通讯的路径；然而，这个通讯路径能够直接穿越NAT而无需使用中继。

[RFC5128] and [RFC4787]定义的这种“打洞”技术不能够适用于任何NAT情况，例如某主机位于的NAT路由器使用的“地址依赖映射”
或“地址-端口依赖映射”，那么这种“打洞”技术通常会失败。

当无法找到一个直接通讯路径时，必须要使用到服务中继来进行互换报文。这种中继通常适用于在公网环境下，两个都处于NAT环境下的主机进行通讯场景。

本文定义了一个名为TURN的协议，它允许两个处于NAT环境的主机利用中继进行通讯。client能够在TURN Server上分配资源，与peer（对端）进行通讯，
也能够决定何时应该停止通讯。client需要关联一个TURN Server的地址作为中继，称为relayed server address。当客户发送报文给TURN Server，
TURN Server使用relayed server address作为源地址向其他peer进行中继转发报文。

client用到TURN服务必须通过某种方式获取peers的地址，这个超越了本文的范畴，可以使用email互换信息，另一种是可以参考[RFC5128]。

如果TURN被使用于ICE[RFC5245],那么relayed transport address和这些peers的地址都必须提供给ICE进行选择，
如果TURN和ICE被作为SIP协议的一部分...

通过TURN服务看起来能够很好的解决两个主机都位于NAT下的通讯问题，但是这对TURN Server是有高昂代价的，比如TURN Server必须在英特网上拥有大带宽，
如此，应该优先使用主机双方之间通讯的方式，只有在主机间无法直接通讯的情况下才使用TURN服务。
当client和peers使用ICE决定通讯路径时，ICE会优先考虑直接通讯，除非无法直接通讯才使用TURN。

TURN原来是设计用于支持多媒体会话通讯在SIP协议。从SIP协议派生后，TURN支持使用一个relayed transport address与多个peer通讯。
这个特写能够使TURN适应于不同的应用场景。

TURN被作为ICE（NAT穿越）框架的一个重要组成部分，然后TURN可以独立于ICE之外单独工作。

TURN是STUN(Session Traversal Utilities for NAT)的扩展协议，大多数情况下，TURN消息采用STUN的消息格式，读者应该的对STUN协议有所了解。

## 2、功能一览

这个节主要预览TURN协议功能，是非严格的描述。
在大多情况下TURN client处于私有网络下，并且通过一个或者多个NAT连接到公网。TURN Server是位于公网环境下。
client想要与其他peer进行通讯。这些peer可能位于同一个或者不同的NAT内。client利用TURN中继地址向peer发送报文，
也通过TURN的中继地址收集peer报文并发送给client。

```txt
                                       Peer A  
                                       Server-Reflexive    +---------+  
                                       Transport Address   |         |  
                                       192.0.2.150:32102   |         |  
                                           |              /|         |  
                         TURN              |            / ^|  Peer A |  
   Client's              Server            |           /  ||         |  
   Host Transport        Transport         |         //   ||         |  
   Address               Address           |       //     |+---------+  
  10.1.1.2:49721       192.0.2.15:3478     |+-+  //     Peer A  
           |               |               ||N| /       Host Transport  
           |   +-+         |               ||A|/        Address  
           |   | |         |               v|T|     192.168.100.2:49582  
           |   | |         |               /+-+  
+---------+|   | |         |+---------+   /              +---------+  
|         ||   |N|         ||         | //               |         |  
| TURN    |v   | |         v| TURN    |/                 |         |  
| Client  |----|A|----------| Server  |------------------|  Peer B |  
|         |    | |^         |         |^                ^|         |  
|         |    |T||         |         ||                ||         |  
+---------+    | ||         +---------+|                |+---------+  
               | ||                    |                |  
               | ||                    |                |  
               +-+|                    |                |  
                  |                    |                |  
                  |                    |                |  
            Client's                   |            Peer B  
            Server-Reflexive    Relayed             Transport  
            Transport Address   Transport Address   Address  
            192.0.2.1:7000      192.0.2.15:50000     192.0.2.210:49191  
```

图1显示一个典型的网络环境，在这张图中，TURN client处于NAT内，TURN server处于NAT外，
假设这个NAT是一个"bad NAT"; 例如其使用"地址-端口依赖映射"的NAT。

- client告诉服务的（IP和端口）组合称为client's HODST TRANSPORT ADDRESS。（也称为TRANSPORT ADDRESS)

- client发送TURN消息给TURN server的TURN SERVER TRANSPORT ADDRESS。client通过其他途径（比如配置）获得这一地址。

- client使用TURN命令进行创建和管理一个名为ALLOCATION的资源在TURN server上。一个allocation是TURN server里的一个数据结构。它包含但不限于RELAYED TRANSPORT ADDRESS。
  
- turn server发送从client获得要发送的数据后并通过这个relayed transport address发送给相关的peer。并且allocation可以通过relayed transport address作为唯一标示。

- 一旦一个allocation被创建，client能发送应用数据给server，并且通过server中继这个数据给相关的peer。client发送的数据还是TURN的消息；当server收到后将从消息中提取DATA部分，并通过原始UDP报文发送给对应的peer。反向，当server收到peer的UDP数据，会封住为TURN的消息格式发送个对应的client。消息中包含peer的相关信息，因此可以使用一个allocatoin与多个peer同时通讯。

- 当peer处于NAT内，client必须指定peer的server-reflexive transport address，而不是它的host transport address，举例，在上图示例中，向PeerA发送数据时，在client必须指定192.0.2.150:32102 (Peer A's server-reflexive transport address)，而不是192.168.100.2:49582 (Peer A's host transport address)。

- 在server端，每个allocation都能够关联一个单独client,这样能保证relayed transport address收到数据后只会被传送给对应的client。

- client可以同时拥有多个allocation在服务端。

### 2.1 传输方式

TURN，在本文的定义中，server与peer间总是使用UDP通讯。然而，client与server间可以使用UDP，TCP,TLS任何一种协议。

```txt
+----------------------------+---------------------+  
| TURN client to TURN server | TURN server to peer |  
+----------------------------+---------------------+  
|             UDP            |         UDP         |  
|             TCP            |         UDP         |  
|        TLS over TCP        |         UDP         |  
+----------------------------+---------------------+  
```

如果TCP或TLS协议被用于client与server之间，那么server需要转为UDP协议与peer进行通讯。

TURN当前这个版本只允许peer与server之间使用UDP协议，估计大部分client也向使用UDP与server进行通讯，为何还要设计TCP和TLS的通讯方式呢？

TURN支持client与server使用TCP协议是因为有些防火墙完全阻止UDP协议，另外因为TCP对防火墙而且认为更加安全。
举例，TCP使用三步握手机制能够明确表明是用户所期望产生的行为。相对UDP而且，防火墙还需要花一定时间来判断连接是否失效，而TCP连接失效能够立刻反映出来。

TURN支持client与server间使用TLS是因为某些安全方面的需求，然而不推荐默认使用该协议，因为对server而且会带来加解密额外的性能开销。

### 2.2 Allocations

client使用allocate事务创建一个server端的allocation。client发送一个Allocate请求到server端，server回应allocation创建成功的相应给client，
并且携带相应的relayed transport address。client可以在Allocate请求中携带所期望的属性（例如：allocation的存活周期等）。
如果数据有安全的考虑，server必须要求client进行认证授权，通常使用STUN协议定义的long-term认证方式。

一旦relay transport address被创建，client必须保持它存活。那么client需要主动发起Refresh请求给server进行刷新。
TURN新定义Refresh方法而不复用Allocate方法进行刷新是为了明确Allocatoin失效是由于某些的原因。

Refresh事务的频率由allocation存活时间决定，allocation默认存活时长是10分钟--设置这么长的时间是为了减轻client段刷新的负担。
并且当client意外退出是allocation能及时的失效。然而，client可以在Allocation请求设置一个长时间的存活时间，或者在Refresh请求修改这个一时间。
server总是在Allocate或Refresh回应消息中携带一个真实的存活时间。如果client相应释放这个allocation，可以在Refresh请求中将存活时间设置为0,
server端就会删除这个allocation。

server和client都保存一个5元组。client端,5元组包括client's host transport address,server transport address，传输协议。
server端，5元组和client类似，唯一不同的是server使用的server-reflexive transport address，而不是client's host transport address。

在Allocate请求时server和client能够保存5元组。在随后的消息中都会使用到5元组。这样client和server都知道哪些allocation。
如果client相应申请一个新的allocation，则必须使用不同的5元组（比如client;t host transport address是不同的）

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |-- Allocate request --------------->|             |             |  
    |                                    |             |             |  
    |<--------------- Allocate failure --|             |             |  
    |                 (401 Unauthorized) |             |             |  
    |                                    |             |             |  
    |-- Allocate request --------------->|             |             |  
    |                                    |             |             |  
    |<---------- Allocate success resp --|             |             |  
    |            (192.0.2.15:50000)      |             |             |  
    //                                   //            //            //  
    |                                    |             |             |  
    |-- Refresh request ---------------->|             |             |  
    |                                    |             |             |  
    |<----------- Refresh success resp --|             |             |  
    |                                    |             |             |  
```

在图2中，

- client没有携带鉴权信息发送Allocate请求给server。但server要求所有请求必须使用STUN的long-term授权机制时，server会返回401错误，要求重新鉴权。
- client第二次携带鉴权信息重新发送Allocation请求，server回复allocation创建成功，并且携带relayed transport address。
- 稍后client发起Refresh请求给server,server回应Refresh请求成功。

### 2.3 Permissions (许可）

为了避免使用防火墙安全机制，TURN定义了许可的概念。TURN许可仿照了“NAT地址过滤限制[RFC4787]”机制。

一个allocation可以用0个或更多的许可。每个许可包含了IP地址和存活时间。当server的中继地址(relayed transport address)接收UDP报文后必须检查源地址是否包含在许可列表中。如果源地址被匹配成功，则将UDP报文中继给client，否则则丢弃该报文。

一个许可在未刷新的5分钟之后失效，并且无法显示的删除许可。这个行为是匹配NAT规范。

client可以使用CreatePermission或ChannelBind请求进行创建和刷新许可。一个CreatePermission请求里可以携带多个被安装或刷新的许可。

这点ICE里十分重要。处于安全考虑，许可只能创建和刷新，并且可以被验证。但是Send和ChannelData消息不需要安装和刷新任何许可。

注意，一个许可是携带allocation上下文的，因此一个allocation的许可失效是不会影响其他allocations的。

### 2.4 传输机制 Send+Data

这里有两种使用TURN服务让client与其peers进行交换数据的方式。

- 第一种是使用Send和Data方法
- 第二种是使用通道。这两种方法能够利用一个中继地址与多个peers进行通讯。也意味着server收到client的数据后发送给peers，收到peers数据后发送给client。

第一种方式下，client通过Send方法发送应用数据给server，而server使用Data方法发送数据给client。
使用这种发送机制时，client发送Send信令给server时需要包含XOR_PEER_ADDRESS属性，并且填写peer的NAT外地址（server-reflexiv）,而不是peer的主机地址。使用DATA属性包含应用数据。server收到Send信令提取DATA数据并用中继地址作为源地址向peer发送UDP报文。注意这里无需指定中继地址，因为Send信令已经隐含了5元组。

反向而言，当server的中继地址收到UDP报文，将其转化为Data信令发送给client，XOR_PEER_ADDRESS属性携带peer的NAT外地址，data包含在DATAT属性中。
因为中继地址是allocation的唯一标示，所有通过中继地址能够确定哪个client需要接收该数据。

Send和Data信令不能进行STUN的long-term认证。这对这套发送机制而言并不是大问题。
因为Send信令无法进行授权认证，为避免一个攻击者伪造send数据，因此在发送Send信令前需要安装许可。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |                                    |             |             |  
    |-- CreatePermission req (Peer A) -->|             |             |  
    |<-- CreatePermission success resp --|             |             |  
    |                                    |             |             |  
    |--- Send ind (Peer A)-------------->|             |             |  
    |                                    |=== data ===>|             |  
    |                                    |             |             |  
    |                                    |<== data ====|             |  
    |<-------------- Data ind (Peer A) --|             |             |  
    |                                    |             |             |  
    |                                    |             |             |  
    |--- Send ind (Peer B)-------------->|             |             |  
    |                                    | dropped     |             |  
    |                                    |             |             |  
    |                                    |<== data ==================|  
    |                            dropped |             |             |  
    |                                    |             |             |  
```

图3中，client已经创建好了allocation。

- 首先client发送CreatePermission请求在XOR-PEER-ADDRESS属性中携带PeerA的NAT地址进行许可创建。如果这一步未完成，则无法进行中继。
- 然后client通过Send信令发送数据给server，server中继给PeerA。当server中继地址收到PeerA的UDP报文后，将其转化为Data信令中继给client。
- 最后，client相应中继给PeerB，但是之前没有安装许可因此server将其丢弃，PeerB发送给中继地址的UDP报文也同样被丢弃。

### 2.5 传输机制 Channels（通道）

某些应用场景下（例如VoIP),使用Send或Data信令需要36字节的负担，这样大大增加了client到server间的带宽。为解决这个问题，client利用server发送指定peers通讯方式有另一种选择。

这种方式是利用一种名为ChannelData的另一种消息格式。ChannelData消息不同与之前的消息格式，它不是使用STUN格式的消息。而是使用4字节的头部用于标示一个通道ID，每一个通道ID都对应指定的peer，可以使用peer host transport address对应。

为了将peer绑定为一个通道，client需要向server发送ChannelBind请求，并且包含通道ID和对端地址信息。一旦通道绑定成功，client可以使用ChannelData消息向peer传送数据。同样server的中继地址收到peer消息后也会转换为ChannelData消息发送给client。

通道绑定存活时间为10分钟长于许可的存活时间5分钟，通道绑定刷新请求需要重新发送ChannelBind请求。类似于许可，通道绑定也无法显示取消绑定，只能等待通道绑定超时失效。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |                                    |             |             |  
    |-- ChannelBind req ---------------->|             |             |  
    | (Peer A to 0x4001)                 |             |             |  
    |                                    |             |             |  
    |<---------- ChannelBind succ resp --|             |             |  
    |                                    |             |             |  
    |-- [0x4001] data ------------------>|             |             |  
    |                                    |=== data ===>|             |  
    |                                    |             |             |  
    |                                    |<== data ====|             |  
    |<------------------ [0x4001] data --|             |             |  
    |                                    |             |             |  
    |--- Send ind (Peer A)-------------->|             |             |  
    |                                    |=== data ===>|             |  
    |                                    |             |             |  
    |                                    |<== data ====|             |  
    |<------------------ [0x4001] data --|             |             |  
    |                                    |             |             |  
```

图4展现了通道传输机制。在传输之前client已经创建了allocation。

- client发送ChannelBing请求给server,指定PeerA的地址和通道ID(0x4001)。

- 在稍后的ChannelData消息中必须携带相同的通道ID(0x4001).当server收到ChannelData消息将Data封装为UDP报文发送给PeerA。

- 反向而言，当server中继地址收到PeerA的UDP报文，通过中继地址能关联到一个allocation，server将其封住为ChannelData消息方式给client，并且填写通道ID为(0x4001)。

一旦一个通道绑定成功，client可以自由选择Send信令或ChannelData消息向peer发送数据。在这张图的最后，client选择Send信令向peer传输数据。

下一章节将介绍IP的是否允许分配。而且一旦一个通道绑定，client只会收到server发来的ChannelData消息。

注意，通道只能被用于被绑定的peer,上图只能适用于PeerA，而PeerB这无法利用这个通道。

### 2.6 TURN Server的限制

当前版本的TURN被定义于使用用户空间实现，而非操作系统内核，因此无法使用操作系统的特权级别。之所以如此设计，是希望TURN server能够很好的集成到其他应用之中来实现NAT穿越。

这种设计原则对中继服务有以下局限性。

* 无法选择不同的服务类型
* IP报文存活时间(Time to Live:TTL)字段无法被用户设置。
* 显示拥塞通知(Explicit Congestion Notification: ECN)字段无法被用户设置
* ICMP消息无法被中继。
* 没有end-to-end分段，当报文被重传时。

在未来可以考虑解决这些局限性。

### 2.7 避免IP分片

在某些特殊因素的应用中，尤其是某些需要发送大量数据的情况需要避免数据被IP分片。如果使用TCP协议可以忽略这点，因为在标准的TCP协议中已经考虑IP分片的因素。但是如果使用UDP协议，必须要考虑避免IP分片。

有以下两种避免IP分片的方法。

第一种避免IP片方法是避免client和peer直接传输大量的应用数据，这种方法比较适用于VoIP这种音频数据，IP报文的长度最后不要超过576字节。

这个方案需要首先考虑传输方式是TCP,UDP还是TLS，然后是使用Send/Data信令还是ChannelData消息进行数据传输。另一个因素是很难处理的，例如使用IP隧道技术时很难处理路径MTU逐渐减少的情况。

作一个参考，（client到server）的应用数据和(server到peer）的UDP报文的最大长度最多500字节，为了进一步减少IP分片的情况，优先使用ChannelData消息，因为其更为节省字节。

第二种方式是，client和peer各种使用路径MTU发现机制来决定其应用程序数据的大小上限。

不幸的是，由于TURN server无法对ICMP进行中继，因此经典的路径MTU发现机制（RFC1191）无法解决client到peer的路径MTU发现。

因此client和peer无法利用ICMP消息作为路径发现的原理，[RFC4821]定义路径MTU发现的算法。

具体如何使用[ rfc4821 ]算法依次仍在研究。然而，为实现这一目标的步骤，这个版本将支持dont-fragment属性。

## 2.8 RTP 协议支持

一个使用TURN进行音频或者视频进行实时数据的例子是需要使用RTP协议，为了实现这个目录，TURN包含了支持这协议的一些特性。

老版本RTP[RFC3550]要求RTP流和其RTCP流的端口是RTCP=RTP+1的关系，因此TURN允许client创建allocatoin时保留之后的一位端口，便于在随后的分配时提供。

### 2.9 Server的选播发现机制

这个版本TURN协议设计了一个基于UDP协议的server发现机制。

具体而言，一个TURN server可以拒绝一个Allocate请求，并且建议client尝试一个备选server。为了避免某些类型的攻击，client必须使用相同鉴权信息进行鉴权。

## 3、术语

本文档包含“必须”，”必须不"，”要求”，“应该”，“不应该”，“需要”，“不需要”，“推荐”，“可能”，“可选”等关键字已经被定义于RFC 2119中。

预计读者已经阅读RFC5389相关的定义。

本文的定义有：

- **TURN**: 这个协议是STUN协议的扩展，本协议允许client利用中继服务与相关peer通讯。
- **TURN client**: 本协议规范了TURN client的实现
- **TURN server**: 本协议规范了TURN server的实现，提供client中继数据给它的peers。
- **Peer**: 一个TURN client相应通讯的主机，peer与server直接的通讯并不使用本协议，直接使用UDP。
- **Transport Address**: IP地址和端口的组合
- **Host Transport Address**: client和peer的主机地址。译者注：NAT内地址。
- **Server-Reflexive Transport Address**: 一个client/peer在其公网NAT端的映射地址，这个地址有路由器和主机地址进行互换。译者注：NAT外地址。
- **Relayed Transport Address**: TURN server分配的与client和peer进行中继的地址。 译者注：中继地址。
- **TURN Server Transport Address**: TURN消息的交换地址。这个地址被用于client与server进行交互。译者注：server地址
- **Peer Transport Address**: Peer的地址，如果Peer位于NAT内，则该地址表示PeerNAT的地址。译者注:对端地址
- **Allocation**: 由client使用Allocate请求生成的中继地址，并伴随着许可和存活时间等状态。
- **5-tuple**: 5元组，client端是（client's NAT内地址，server地址，传输类型），server端（client's NAT外地址,server地址，传输类型）是allocation的唯一标识。
- **Channel**: 通道，一个通道ID绑定了一个对端地址，一旦通道绑定成功，client和peer可以使用通道进行更节省带宽的交互。
- **Permission**: 许可，由一个IP地址和协议（无端口）向关联的通道，是client向peer通讯和peer向client通讯的许可。
- **Realm**: 用于描述服务器上下文的字符串。该Realm将告知client,并被用于用户名和口令的校验。
- **Nonce**: 一个服务器随机选择的信息摘要，为了防止重复攻击，服务器应该定期进行更换。
- **Indications**: 译者注：一个Send/Data的机制，用于传输数据的信令。

## 4、通用行为

这节包含TURN的通用处理规则是相应所有的TURN消息。

TURN是STUN协议的扩展。所有TURN消息，除ChannelData消息外，都是STUN的消息格式。所有集成的处理规则被定义于RFC5389响应STUN消息。这意味着所有消息格式和消息处理规则都被定义于RFC5389。

[RFC5389]定义的鉴权机制称为long-term认证机制。TURN server和client“必须”实现这个机制。服务端"必须"对所有client的请求消息进行这种认证机制进行校验。或者使用一个同样强大的认证机制。

注意，long-term认证机制只针对于请求消息，而不校验indications(信令)，换言之indications在TURN server无需鉴权。如果server要求请求需要校验，server管理者选择一个realm值并且选择唯一的username和password组合给client使用，除非client使用多个server外。
server管理者“可能”选择不重复的username给不同的client使用，或者使用同样的username给所有client。

对于每个allocation,server“应该”生成新的随机nonce在第一个分配时。

在创建allocation后的所有请求都必须使用相同的username并且使用这个allocation,防止攻击则挟持client's allocation。
具体而且，如果server要求使用long-term认证机制，并且非Allocation请求通过鉴权机制，并且5元组标示存在于allocation，但是这个请求的username不同于创建于allocation时的username，则server必须拒绝这个请求，并且返回441（非法权限）的错误。

当server收到client的TURN消息后，server使用5元组来识别相关的allocation。对所有TURN消息而言（包括ChannelData消息），如果5元组没有找到关联的allocation，如果是请求消息（eg, Refresh,ChannelBind)将返回437（Allocation未找到的消息），如果是数据消息（eg Send/Data Ind，或者ChannelData)则被丢弃。如果client收到437消息后必须认为allocation已经不存在。

[RFC5389]定义一下属性，例如**DSOFTWARE**和**FINGERPRINT**属性。client需要在Allocate和Refresh消息中包含**SOFTWARE**属性，也可以在其他消息包含该属性。
server应该在Allocation/Refressh的响应消息中包含SOFTWARE消息（无论成功还是失败），也可以在其他消息中包含该属性。
client和server可能包含**FINGERPRINT**属性在STUN格式的消息中。

TURN不使用定义于[RFC5389]的向后兼容的机制。

TURN，定义在本文档中只支持IPv4。client地址，服务器地址和所有地址必须为IPv4地址。

默认情况下，TURN使用和STUN相同的端口，UDP和TCP使用5348，TURN的TLS使用5349。也应该有自己的服务名字，如TURN for UDP/TCP,TURN for TLS。
所有的服务记录名字和备选服务的机制都定义在第6节。TUNR也可以使用不同的端口。

为确保互操作性，TURN服务必须实现UDP协议进行client和server交互，也应该实现TCP和TLS的交互方式。

当client使用UDP传输协议和server进行交互时，必须实现重传机制当一定时间内未收到响应时。正是因此，server可能收到2个或者更多的请求携带着相同的5元组和事务号。
TURN要求识别这种情况，并参考[RFC5389]进行处理。一些实现可能要求所有的请求在40秒之内收到响应，......

当client使用TCP传输协议和server进行交互时，可以由于某些错误导致包长度字段受损，使得接收方丢失同步信息流，如果发现这种情况应该将连接断开，让其它端能够及时发现。

为了避免有意或无意的拒绝服务攻击，client应该提供有效的用户名和密码。建议server限制username同时可分配的allocaiton数量和带宽。
当server的allocation数量达到上限时，应该拒绝分配新的allcation，返回486(资源不足)的错误码，并且丢弃已经超过带宽限制的数据。

## 5、Allocations

所有TURN操作都围绕着allocations，并且TURN消息都关联了一个allocation。一个allocatoin伴随以下这些状态：

* 中继地址
* 5元组
* 鉴权信息
* 存活时间
* 许可列表
* 通道绑定列表

中继地址是server所分配的并用于server与peers进行通讯，5元组用于client和server通讯的标示。   
client端是（client's NAT内地址，server地址，传输类型），server端（client's NAT外地址,server地址，传输类型)。

中继地址和5元组都能唯一标识一个allocatoin。

鉴权信息（包括，用户名，口令，realm,nonce)被使用与校验随后的消息并且提供回应消息的完整信息。用户名、realm,nonce在Allocate请求中被确定。
nonce在存活周期内可以利用438（nonce改变）响应进行改变。注意，处于安全考虑，Nonce是realm,username,password的hash值。

存活时间是一个allocation是有效期，以秒为单位。每个Allocation和Refessh事务设置它的值。默认情况下，它的值为10分钟，但是client可以在Allocation和Refresh请求内携带一个指定值。Allocations只能用Refresh请求刷新存活时间。发送数据给peer并不代表刷新存活时间。
当allocatoin超时失效，它所伴随的状态都将被释放。

许可将在第8节进行讲解，通道将在第11节进行讲解。

## 6、创建Allocation

allocation是使用Allocate请求进行创建的。

### 6.1 发送Allocate请求

client发送Allocate请求需要有如下要求。

client首先获取到自己的主机地址。建议利用soket来选择一个未被使用的地址。

client选择一个协议用于与server进行通讯。传输协议必须是UDP,TCP,TSL之一。因为server与peers直接只能适用UDP，如果没有其他特殊原因应该优先使用UDP协议。
2.1节讲解选取TCP,TSL作为传输协议的原因。

client需要获取server地址（可能通过配置）需要做如下事情，通过域名获取server的地址。

client在Allocate请求中必须包含**REQUESTED-TRANSPORT**属性指示选择的传输协议。这个传输协议被用于server与peers之间（注意，这不是client的5元组的协议）。
目前这个值只能为UDP,但是未来可以扩展为其他协议。

如果client需要指定一个allocation的存活时间可以设置**LIFETIME**属性。这仅仅是请求值，server可以根据情况选择是否使用，还是使用默认值。

如果client想要在后续Send indication使用**DONT-FRAGMENT**属性，可以在请求中携带该属性。这个可以让client测试server是否支持该属性。

如果client响应保留一位中继地址，可以使用**EVEN_PORT**属性。这样要求server为其分配中继地址时，保留中继地址+1端口，以便在随后分配给client。

client可以使用**RESERVATION-TOKEN**属性来获得server保留的中继地址，此时不能再使用EVEN_PORT属性。

一旦构造，client发送Allocation请求确定5元组。

### 6.2 接收到Allocate请求

当server接收到Allocate请求，需要进行如下检查。

1. Server必须对请求使用long-term机制进行鉴权检查，除非client和server使用其他机制进行认证。

2. Server需要检查该5元组是否已经存在allocatoin，如果存在，则拒绝请求并返回437（Allocation不匹配）错误。

3. Server需要检查请求是否协议REQUESTED-TRANSPORT属性，如果属性不存在或者是非法的则返回400 (Bad Request)错误，否则如果不是UDP协议，则返回442 (Unsupported Transport Protocol) error。

4. 请求可能携带DONT-FRAGMENT属性，如果server不支持该属性，则忽略。

5. Server检查是否包含RESERVATION-TOKEN属性，如果是但是又包含EVEN-PORT属性，则拒绝并回复400 (Bad Request) error。否则则检查TOKEN是否有效，如果无效则拒绝并返回508 (Insufficient Capacity)error。

6. Server检查请求是否携带EVEN-PORT属性，则需要找到两个端口连续的中继地址，如果无法满足这种条件，则拒绝并回复508 (Insufficient Capacity)error。

7. 任何情况下，client如果已经分配到的上限，server可以拒绝并回复486 (Allocation Quota Reached)error。server应该要求检查client的用户名，而不是地址。

8. 在任何情况下，server可以回复一个300(Try Alternate)错误码，将client重定向到另一个server。这个错误码定义在[RFC5389]中。

如果一切都检查通过，这server创建allocation,5元组被设置在这个allocation中，此时许可列表和通道绑定列表为空。

server为allocation选择中继地址遵从下面事项：

* 如果请求中包含**RESERVATION-TOKEN**属性，server选择之前保留的中继地址（如果仍然可用）。注意这个保留是服务端的行为，这次Allocate请求的5元组应该不同于之前的5元组，可以使用不同的client地址或端口，甚至不同的server地址或端口。

* 如果请求包含**EVEN-PORT**属性，并且R bit设置为0，则server选择中继地址是一任意端口。

* 如果请求包含**EVEN-PORT**属性，并且R bit设置为1，则server选择一对端口N，N+1，Port N被用于当前allocation，端口N+1将被保留，用于后续的allocation，server必须保留端口30秒，甚至更长，例如这个allocation失效，server将这个token通过RESERVATION-TOKEN属性填写到回应消息中。

* 否则，server选择任意可用的中继地址。

上述所有情况，serve应该选择端口的范围是49152 - 65535，除非TUNR server知道哪些端口可以被使用。如论如何，TURN server都不能使用0-1023之间的端口。

server决定allocation存活时间的初始值如下。如果请求中携带**LIFETIME**属性，server选择这个值与server允许最大存活时间直接取最小值。

如果这个结果大于默认时间（10分钟），则使用这个值，否则使用默认时间。我们推荐server的设置最大存活时间不要超过1小时。因为当client由于某种原因断开连接时，server需要利用这个超时时间进行释放allocatoin。同样，这个规则也使用与Refresh请求，这个值将在第一次刷新是被重新计算。

一旦allocaton创建成功，server需要回复一个成功回应消息，应该包含：

* **XOR-RELAYED-ADDRESS** 关联的中继地址。
* **LIFETIME** server决定的存活时间
* **RESERVATION-TOKEN** （如果需要保留中继地址时才用）
* **XOR-MAPPED-ADDRESS** （ client的NAT外地址）

这个回应是（无论成功还是失败）都将发生给client通过5元组。
注意：当Allocate请求使用UDP传输时，因为重传原因server端可能收到相同的Allocate请求消息，server应该避免创建多个allocation。
可能需要使用一种叫”无状态协议栈的方法“来实现如下。
需要识别重传消息，通过事务号和五元组。一旦识别为重传消息，server无需解析直接回复成功。当构造回应消息时，这个存活时间和原始可能不同。

如果原始请求没有创建成功下，这个消息没有什么特殊的，但是注意的是client可能收到第一个请求创建失败，而由于重传收到第二个请求创建成功，
此时client可能无法使用这个回应，那么已经被创建的allocatoin只能等待超时，因为client不会进行刷新。

### 6.3 接收成功的回应

当client接收到成功回应时，必须校验MAP Address和Delayed Address是否是同一个协议簇。当前这个版本定义这个两个地址只能是IPV4，如果这两个地址不是这个协议簇，这需要删除这个allocation，并且不要创建另一个allocations直到server解决这个问题。

否则，client记录这些数据，特别是存活时间，不是请求中的，而是回应中的。client要记录5元组，并且保持username,password被用于鉴权。

client也可能利用MAP-Address与它peers进行通讯，它是ICE的一个过程。

### 6.4 接收失败的回应

当client收到错误Allocate错误回应时，它的处理依赖实际的错误码。

* （请求超时） 可能是server端错误，或者选择错误的server端地址，client考虑使用不同的传输协议（TCP)进行重试。

* 300 （重定向） server想要client使用另一个server进行通讯，在 ALTERNATE-SERVER属性中指定该服务的地址。client认为这次处理失败，
应该尝试连接这个服务。当client连接备选服务器是的过程被定义于 [RFC5389]。

* 400(错误请求消息） server认为client的请求是不合法的。client认为创建失败，并且通知给用户，不要再次进行请求除非确保server问题已经解决。

* 401 (未授权）如果client已经使用long-term鉴权机制，否则认为创建失败，并且通知用户。

* 403 (拒绝）这个请求是合法的，但是server无法处理，比如管理限制。client认为创建失败。

* 420（未知的属性）当client发送**DONT-FRAGMENT**属性给server,但server不支持该属性，回复绝交，并指明该属性为未知属性，client收到错误消息后可以认为server不支持该属性，并且去掉该属性重新发送Allocate请求。

* 437 (Allocatoin失效） client认为创建失败，client使用另一个地址重新请求，当重试3次之后还不成功，2分钟只能不能再次请求。

* 438 (过期的Nonce) : 参见[RFC5389]的long-term认证机制。

* 441 (错误的认证)： client认为创建失败，并且通知用户。

* 442 （不支持的传输地址）client可能填写非UDP传输类型导致server返回不支持。

* 486 （Allocation达到上限）client认为server已经无法分配资源，认为创建失败，并且提示用户。

* 508 （能力不足） client可能携带**EVEN-PORT**或者**RESERVATION-TOKEN**属性用于创建allocation，server无法满足client的条件会返回该错误码，client可以去除这个两个属性重新进行Allocate，否则应该等待1分钟以上再次创建allocation。

其他未知错误的处理行为参考[RFC5389]。

## 7、刷新Allocation

Refresh事务可用于刷新存活时间和删除Allocation两种用途。

client如果想继续使用Allocation需要在失效前进行刷新。建议在失效期前1分钟进行刷新。如果client不想使用这个allocatoin需要删除allocation。client可能由于某些原因在任何时间进行刷新。

### 7.1 发送Refresh请求

如果client需要立即删除allocation需要包含属性**LIFETIME**值设为0.

刷新事务用于更新allocation的存活时间。如果client需要指定存活时间可以携带LIFETIME属性。当这个属性值为0时，server立即删除这个allocation。

### 7.2 接收Refresh请求

当server收到刷新请求，处理规则如下：

server决定存活时间，如果第6节的一段所介绍一样。

如果请求包含LIFETIME属性并且为0，则为0，如果不为0，server选择这个值与server允许最大存活时间直接取最小值。如果这个结果大于默认时间（10分钟），则使用这个值，否则使用默认时间。

下面server决定存活时间有如下处理：

* 如果存活时间为0，那么server立即删除allocation。
* 如果存活时间非0，那么响应成功后更新allocation的存活时间。

如果请求成功，server发送回应消息包括如下属性：

* LIFETIME 包含server所决定的存活时间。

注意：关于使用UDP是会收到重传的处理。

### 7.3 接收Refresh请求回应

如果client收到存活时间为非0的刷新成功响应后，那么应该使用相应中存活时间值来更新allocatoin响应的存活时间数据结构。

当client删除一个allocation收到一个437 (Allocation Mismatch) 错误时，有可能是重传消息导致，也认为这个allocation已经不纯在了，事务执行成功。

## 8、Permissions(许可）

## 9、创建Permission

## 10、发送数据方式

## 11、Channels

## 12、IP头字段

## 13、STUN新方法

这节列举了本文档新定义的方法。

0x003  : Allocate
0x004  : Refresh
0x006  : Send
0x007  : Data
0x008  : CreatePermission
0x009  : ChannelBind

## 14、STUN新属性

这个Stun扩展协议定义如下新属性：

0x000C:  CHANNEL-NUMBER
0x000D:  LIFETIME
0x0010:  Reserved (保留，可用于带宽)
0x0012:  XOR-PEER-ADDRESS
0x0013:  Data
0x0016:  XOR-RELAYED_ADDRESS
0x0018:  EVEN-PORT
0x0019:  REQUESTED-TRANSPORT
0x001A:  DONT-FRAGMENT
0x0021:  Reserved (was TIMER-VAL)
0x0022:  RESERVATION-TOKEN

## 15、新错误码

403  (Forbidden):
437  (Allocation Mismatch): 
441  (Wrong Credentials):
442  (Unsupported Transport Protocol):
486  (Allocation Quota Reached)：
508  (Insufficient Capacity): 

参见6.4的描述

## 16、示例

下图的示例都基于图1的网络环境进行展示的，请回顾图1的网络环境。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |                                    |             |             |  
    |--- Allocate request -------------->|             |             |  
    |    Transaction-Id=0xA56250D3F17ABE679422DE85     |             |  
    |    SOFTWARE="Example client, version 1.03"       |             |  
    |    LIFETIME=3600 (1 hour)          |             |             |  
    |    REQUESTED-TRANSPORT=17 (UDP)    |             |             |  
    |    DONT-FRAGMENT                   |             |             |  
    |                                    |             |             |  
    |<-- Allocate error response --------|             |             |  
    |    Transaction-Id=0xA56250D3F17ABE679422DE85     |             |  
    |    SOFTWARE="Example server, version 1.17"       |             |  
    |    ERROR-CODE=401 (Unauthorized)   |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="adl7W7PeDU4hKE72jdaQvbAMcr6h39sm"      |             |  
    |                                    |             |             |  
    |--- Allocate request -------------->|             |             |  
    |    Transaction-Id=0xC271E932AD7446A32C234492     |             |  
    |    SOFTWARE="Example client 1.03"  |             |             |  
    |    LIFETIME=3600 (1 hour)          |             |             |  
    |    REQUESTED-TRANSPORT=17 (UDP)    |             |             |  
    |    DONT-FRAGMENT                   |             |             |  
    |    USERNAME="George"               |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="adl7W7PeDU4hKE72jdaQvbAMcr6h39sm"      |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
    |                                    |             |             |  
    |<-- Allocate success response ------|             |             |  
    |    Transaction-Id=0xC271E932AD7446A32C234492     |             |  
    |    SOFTWARE="Example server, version 1.17"       |             |  
    |    LIFETIME=1200 (20 minutes)      |             |             |  
    |    XOR-RELAYED-ADDRESS=192.0.2.15:50000          |             |  
    |    XOR-MAPPED-ADDRESS=192.0.2.1:7000             |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
```

客户选择一个本机地址用于这个TURN会话；例如使用（10.1.1.2:49721)在图1网络环境所描述的。客户发送一个Allocate请求到服务端的监听地址。并使用一个随机分配的96位事务号;并且包含到消息头格式中。客户使用SOFTWARE属性表示客户的软件信息;这里使用”Example client,version 1.03”作为示例。客户包含的LIFETIME属性值为1小时，大于服务端存活时间的默认最大值10分钟。在Allocate请求中必须包含REQUESTED-TRANSPORT属性，并且仅允许值为17，表示server到peer之间使用UDP进行通信。客户也可以包含DONT-FRAGMENT属性，因为可能想要在随后的Send indications中被使用;

服务端要求请求是被鉴权过的。然而，当服务端接受到初始Allocate请求时，会拒绝请求因为没有携带鉴权信息。根据STUN[RFC5389]定义的长期鉴权机制，服务端返回401错误的响应，并且携带REALM属性表示server的realm（在这个例子中是“example.com”）,并且包含NONCE属性，也包含SOFTWARE属性表示服务端的软件信息。

客户端收到401错误需要重新发起Allocate请求，这次需要包含鉴权的属性。客户端选择一个新的事务号。客户端包含USERNAME属性并且包含401返回消息中所得到的REALM和NONCE属性，最终客户需要包含MESSAGE-INTEGRITY作为消息最后一个属性。它的值是通过加密算法(HMAC-SHA1)基于上面的几个属性和password计算得出。攻击者在没有password的条件下是无法计算出它的值的。

服务端收到携带鉴权消息的Allocate请求后，如果一切正常，就会创建allocation。服务端返回成功回应。服务端返回LIFETIME属性告诉allocation的存活时间。这里返回的是20分钟比请求时的1个小时要小。服务端返回的XOR-RELAYED-ADDRESS属性携带中继地址。XOR-MAPPED-ADDRESS属性包含了客户的NAT外地址，这个值在TURN协议没有使用，但是返回给客户。MESSAGE-INTEGRITY属性用于保证返回消息的完整性，注意返回消息中没有USERNAME,REALM,NONCE属性。还是可以包含SOFTWARE属性。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |--- CreatePermission request ------>|             |             |  
    |    Transaction-Id=0xE5913A8F460956CA277D3319     |             |  
    |    XOR-PEER-ADDRESS=192.0.2.150:0  |             |             |  
    |    USERNAME="George"               |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="adl7W7PeDU4hKE72jdaQvbAMcr6h39sm"      |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
    |                                    |             |             |  
    |<-- CreatePermission success resp.--|             |             |  
    |    Transaction-Id=0xE5913A8F460956CA277D3319     |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
```

客户使用CreatePermission请求创建PeerA的许可。XOR-PEER-ADDRESS属性包含了PeerA的地址。注意端口是被忽略的。这里端口被设置为0.注意PeerA的地址应该是外网地址，而不是主机地址。客户使用和allocate请求中相同的username,realm,nonce属性。可以选择不包含SOFTWARE属性。

服务端接受到创建许可请求后进行创建，并且返回成功的消息。类似客户端，服务端在回应请求中也可以不包含SOFTWARE属性。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |--- Send indication --------------->|             |             |  
    |    Transaction-Id=0x1278E9ACA2711637EF7D3328     |             |  
    |    XOR-PEER-ADDRESS=192.0.2.150:32102            |             |  
    |    DONT-FRAGMENT                   |             |             |  
    |    DATA=...                        |             |             |  
    |                                    |-- UDP dgm ->|             |  
    |                                    |  data=...   |             |  
    |                                    |             |             |  
    |                                    |<- UDP dgm --|             |  
    |                                    |  data=...   |             |  
    |<-- Data indication ----------------|             |             |  
    |    Transaction-Id=0x8231AE8F9242DA9FF287FEFF     |             |  
    |    XOR-PEER-ADDRESS=192.0.2.150:32102            |             |  
    |    DATA=...                        |             |             |  ```

客户使用Send indication向PeerA发送应用层数据，XOR-PEER-ADDRESS属性包含了PeerA的NAT外地址，Data属性包含应用数据。如果client在之前的Allocate请求中包含了DONT-FRAGMENT属性，server向peerA发送报文时应该设置DF标记。Send indicate不需要使用Stun定义的长期认证机制，因此是不包含MESSAGE-INTEGRITY属性。应用程序只能在应用层确保数据没有被篡改或伪造。

服务端收到客户的Send indication后，使用中继地址作为源地址向PeerA发送UDP数据报，并且确定DF标记。注意，如果没有PeerA地址的许可，那么服务端收到客户的Send indication后会丢弃报文。

PeerA发送UDP报文给服务端，服务端中继地址收到报文后，会构造Data indication，将UDP报文的源地址放到XOR-PEER-ADDRESS属性中，UDP报文数据放到DATA属性中发送给客户。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |--- ChannelBind request ----------->|             |             |  
    |    Transaction-Id=0x6490D3BC175AFF3D84513212     |             |  
    |    CHANNEL-NUMBER=0x4000           |             |             |  
    |    XOR-PEER-ADDRESS=192.0.2.210:49191            |             |  
    |    USERNAME="George"               |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="adl7W7PeDU4hKE72jdaQvbAMcr6h39sm"      |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
    |                                    |             |             |  
    |<-- ChannelBind success response ---|             |             |  
    |    Transaction-Id=0x6490D3BC175AFF3D84513212     |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
```

客户发生bind请求绑定PeerB，使用一个可用的Channel号放在CHANNEL-NUMBER属性里，PeerB的地址放在XOR-PEER-ADDRESS属性里。在这里需要携带username,realm,nonce等鉴权信息。
服务端收到请求，将Channel号与PeerB绑定好，回复成功响应。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |--- ChannelData ------------------->|             |             |  
    |    Channel-number=0x4000           |--- UDP datagram --------->|  
    |    Data=...                        |    Data=...               |  
    |                                    |             |             |  
    |                                    |<-- UDP datagram ----------|  
    |                                    |    Data=... |             |  
    |<-- ChannelData --------------------|             |             |  
    |    Channel-number=0x4000           |             |             |  
    |    Data=...                        |             |             |  
```

现在客户发送ChannelData消息给服务端目的发网Peer B。ChannelData不是STUN消息格式，没有事务号，只有三个字段，分别是通道号，消息长度，消息。通道号就是上面所指定的值（这里0x4000)。服务端收到消息后检查下通道号，并使用UDP报文发给Peer B。


随后，Peer B发送UDP报文到服务端中继地址，服务端创建ChannelData消息发给客户，服务端上知道通道号，因为根据Peer B的地址可以查找之前绑定时的通道号，如果之前没有绑定，那么服务根据是否创建许可，如果有使用Data indication 发往客户端。

```txt
TURN                                 TURN           Peer          Peer  
  client                               server          A             B  
    |--- Refresh request --------------->|             |             |  
    |    Transaction-Id=0x0864B3C27ADE9354B4312414     |             |  
    |    SOFTWARE="Example client 1.03"  |             |             |  
    |    USERNAME="George"               |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="adl7W7PeDU4hKE72jdaQvbAMcr6h39sm"      |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
    |                                    |             |             |  
    |<-- Refresh error response ---------|             |             |  
    |    Transaction-Id=0x0864B3C27ADE9354B4312414     |             |  
    |    SOFTWARE="Example server, version 1.17"       |             |  
    |    ERROR-CODE=438 (Stale Nonce)    |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="npSw1Xw239bBwGYhjNWgz2yH47sxB2j"       |             |  
    |                                    |             |             |  
    |--- Refresh request --------------->|             |             |  
    |    Transaction-Id=0x427BD3E625A85FC731DC4191     |             |  
    |    SOFTWARE="Example client 1.03"  |             |             |  
    |    USERNAME="George"               |             |             |  
    |    REALM="example.com"             |             |             |  
    |    NONCE="npSw1Xw239bBwGYhjNWgz2yH47sxB2j"       |             |  
    |    MESSAGE-INTEGRITY=...           |             |             |  
    |                                    |             |             |  
    |<-- Refresh success response -------|             |             |  
    |    Transaction-Id=0x427BD3E625A85FC731DC4191     |             |  
    |    SOFTWARE="Example server, version 1.17"       |             |  
    |    LIFETIME=600 (10 minutes)       |             |             |  
```

这20分钟之内，客户端发起刷新请求给服务端，有时候服务端认为Nonce信息已经过期，会回应一个438端错误吗给客户端，并且携带新的nonce属性，注意nonce属性有变化，客户端需要再次使用新的nonce发起刷新，服务端收到新的刷新后回复成功响应。当然438错误不是必须的，根据服务端的策略而定。




