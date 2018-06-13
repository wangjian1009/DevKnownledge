# P2P网络技术概要

P2P网络是公链的必备组成部分，提供了最基础的组网能力

[TOC]

## NAT穿透技术

### NAT穿透概念

- **NAT（Network Address Translation，网络地址转换）**，也叫做网络掩蔽或者IP掩蔽。NAT是一种网络地址翻译技术，主要是将内部的私有IP地址（private IP）转换成可以在公网使用的公网IP（public IP）

- **NAT穿透**：在不同NAT网络内部的主机直接建立联系

### 常见NAT穿透技术

- **STUN**：Simple Traversal of User Datagram Protocol Through Network Address Translators，即简单的用UDP穿透NAT，是个轻量级的协议，是基于UDP的完整的穿透NAT的解决方案。[RFC3489](https://tools.ietf.org/html/rfc3489)

- **UPNP**: 通用即插即用
- **ALG**: 应用层网关
- **SBC**:
- **ICE**:
- **TURN**:
- **IKE**:
- **IPsec**:

### - NAT穿透参考资料

[《P2P技术详解》系列文章](http://www.52im.net/thread-50-1-1.html)

- [快速理解P2P技术中的NAT穿透原理](NAT穿透原理.md)
- [P2P技术详解(一)：NAT详解——详细原理、P2P简介](P2P技术详解1.md)
- [P2P技术详解(二)：P2P中的NAT穿越(打洞)方案详解](P2P技术详解2.md)
- [P2P技术详解(三)：P2P技术之STUN、TURN、ICE详解](P2P技术详解3.md)

### 相关RFC协议

|             Specification                |                   Title                |
| :--------------------------------------: | :------------------------------------: |
| RFC3489                                  | STUN - Simple Traversal of User Datagram Protocol (UDP)Through Network Address Translators (NATs) |
| [RFC5389](../RFC协议/RFC5389-STUN协议.md) | Session Traversal Utilities for NAT (STUN) |
| [RFC5766](../RFC协议/RFC5766-TURN协议.md) | Traversal Using Relays around NAT (TURN): Relay Extensions to Session Traversal Utilities for NAT (STUN) |
| [RFC5245](../RFC协议/RFC5245-ICE协议.md)  | Interactive Connectivity Establishment (ICE): A Protocol for Network Address Translator (NAT) Traversal for Offer/Answer Protocols |
| [RFC5780](../RFC协议/RFC5780.md)         | NAT Behavior Discovery Using STUN |
| [RFC5626](../RFC协议/RFC5626-SIP协议.md)  | Managing Client Initiated Connections in the Session Initiation Protocol (SIP) |
| [RFC5853](../RFC协议/RFC5853.md)         | Test vectors for STUN |
| [RFC6062](../RFC协议/RFC6062-TURN-TCP.md) | Traversal Using Relays around NAT (TURN) Extensions for TCP Allocations |
| [RFC6156](../RFC协议/RFC6156-TURN-IPV6.md) | Traversal Using Relays around NAT (TURN) Extension for IPv4/IPv6 Transition |
| RFC6679                                    | Explicit Congestion Notification (ECN) for RTP over UDP |

