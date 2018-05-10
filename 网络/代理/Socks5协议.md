# socks5协议 (RFC1928的中文版本)
[TOC]

## 1、介绍

防火墙的使用，有效的隔离了机构内部网络和外部网络，这种类型的Internet架构变得越来越流行了。这种防火墙系统大都充当着网络之间的应用层网关的角色，通常提供经过控制的Telnet、FTP和SMTP访问。为了推动全球信息的交流，更多的新的应用层协议的推出，这就有必要提供一个总的架构使这些协议能够更明显和更安全的穿过防火墙，也有必要在实际上为它们穿过防火墙提供一个更强的认证机制。这种需要源于客户机-服务器联系在不同组织网络之间的实现，而这种联系需要得到控制并有安全的认证。

在这儿所描述的协议框架是为了让使用TCP和UDP的客户/服务器应用程序更方便地使用网络防火墙所提供的服务所设计的。这个协议从概念上来讲是介于应用层和传输层之间的“中介层”，因而不提供如传递ICMP信息之类由网络层网关的所提供的服务。

## 2、现有的协议

当前存在一个协议socks4[^1],它为Telnet、FTP、HTTP、WAIS和GOPHER等基于TCP协议的客户/服务器程序提供一个不安全的防火墙。而这个新协议扩展了socks4，以使其支持UDP框架规定的安全认证方案、地址解析方案中所规定的域名和IPV6。为了实现这个socks协议，通过需要重新编译或者重新链接基于TCP的客户端应用程序以使用socks库中相应的加密函数。

## 3、基于TCP协议的客户

当一个基于TCP协议的客户端希望与一个只能通过防火墙可以到达的目标（这个由实现决定的）建立连接，它必须先建立一个与socks服务器上socks端口的TCP连接。通常这个TCP端口是1080。当连接建立后，客户端进入协议的“握手”过程：认证方式的选择，根据选中的方式进行认证，然后发送转发的请求。sockds服务器检查这个请求，根据结果，或建立合适的连接，或拒绝。

除非特别注明，所有出现在数据包格式图中的十进制数字均以字节表示相应域的长度。如果某域需要给定一个字节的值，用X'hh'来表示这个字节中的值。如果某域中用到单词'Variable'，这表示该域的长度是可变的，且该长度定义在一个和这个域相关联（1-2个字节）的域中，或一个数据类型中。

客户端连到服务器后，然后就发送请求来协商版本和认证方法：

| VER | NMETHODS | METHODS  | 
| :-: | :------: | :------: |
|  1　| 　　1　　 | 1 to 255 | 

VER（版本）在这个协议版本中被设置为X'05'。NMETHODS（方法选择）中包含在METHODS（方法）中出现的方法标识的数据（用字节表示）。

服务器从METHODS给出的方法中选出一种，发送一个METHOD（方法）选择报文：

| VER | METHOD | 
| :-: | :----: | 
|  1　| 　 1　　| 

    
如果所选择的METHOD的值是X'FF'，则客户端所列出的方法是没有可以被接受的，客户机就必须关闭连接。

当前被定义的METHOD的值有：

 - **0x00**: 无验证需求
 - **0x01**: 通用安全服务应用程序接口（GSSAPI）
 - **0x02**: 用户名/密码(USERNAME/PASSWORD) 
 - **0x03 ~ 0x7F**: IANA 分配(IANA ASSIGNED) 
 - **0x80 ~ 0xFE**: 私人方法保留(RESERVED FOR PRIVATE METHODS) 
 - **0xFF**: 无可接受方法(NO ACCEPTABLE METHODS) 

其中，IANA是负责全球Internet上的IP地址进行编号分配的机构。

于是客户端和服务器进入方法细节的子商议。方法选择子商议描述于独立的文档中。

欲得到该协议新的METHOD支持的开发者可以和IANA联系以求得METHOD号。已分配号码的文档需要参考METHOD号码的当前列表和它们的通讯协议

如果想顺利的执行则必须支持GSSAPI和支持用户名/密码（USERAE/PASSWORD）认证方法。

## 4、需求

一旦方法选择子商议结束，客户机就发送请求细节。如果商议方法包括了完整性检查的目的或机密性封装，则请求必然被封在方法选择的封装中。 

SOCKS请求如下表所示： 

| VER | CMD | RSV  | ATYP | DST.ADDR | DST.PORT | 
| :-: | :-: | :--: | :--: | :------: | :------: | 
|  1  |  1  | 0x00 |  1   | Variable |     2　　| 

其中： 

 - VER protocol version：0x05
 - CMD 

   - CONNECT 0x01
   - BIND 0x02
   - UDP ASSOCIATE 0x03

 - RSV RESERVED 
 - ATYP address type of following address 

   - IP V4 address: 0x01
   - DOMAINNAME: 0x03
   - IP V6 address: 0x04

  - DST.ADDR desired destination address 
  - DST.PORT desired destination port in network octet order 

## 5、地址

在地址域(DST.ADDR,BND.ADDR)中，ATYP域详细说明了包含在该域内部的地址类型： 

- 0x01: 该地址是IPv4地址，长4个八位组。 
- 0x03: 该地址包含一个完全的域名。第一个八位组包含了后面名称的八位组的数目，没有中止的空八位组。 
- 0x04: 该地址是IPv6地址，长16个八位组。

注：我在应用该协议时，纠结了好久，当时选择0x03，填写相应字段时，到底该怎么填写呢？是写socks服务器的还是写请求的？现在想想，自己好无知啊。。如我们想发生HTTP请求：www.baidu.com，则DST.ADDR为： 13www.baidu.com0x000x50。其中，13是www.baidu.com的长度，然后接www.baidu.com，然后是两个字节的端口号，这里为80端口。

## 6、回应

到SOCKS服务器的连接一经建立，客户机即发送SOCKS请求信息，并且完成认证商议。服务器评估请求，返回一个回应如下表所示： 

| VER | REP | RSV  | ATYP | BND.ADDR | BND.PORT | 
| :-: | :-: | :--: | :--: | :------: | :------: | 
|  1  |  1  | 0x00 |   1  | Variable |     2    | 

其中： 

- VER protocol version: X'05' 
- REP Reply field: 

  - 0x00 succeeded 
  - 0x01 general SOCKS server failure 
  - 0x02 connection not allowed by ruleset 
  - 0x03 Network unreachable 
  - 0x04 Host unreachable 
  - 0x05 Connection refused 
  - 0x06 TTL expired 
  - 0x07 Command not supported 
  - 0x08 Address type not supported 
  - 0x09 ~ 0xFF unassigned 

- RSV RESERVED 
- ATYP address type of following address 

  - IP V4 address: 0x01
  - DOMAINNAME: 0x03
  - IP V6 address: 0x04

- BND.ADDR server bound address 
- BND.PORT server bound port in network octet order 

标志RESERVED(RSV)的地方必须设置为X'00'。
    
如果被选中的方法包括有认证目的封装，完整性和/或机密性的检查，则回应就被封装在方法选择的封装套中。 

### CONNECT 

在CONNECT的回应中，BND.PORT包括了服务器分配的连接到目标主机的端口号，同时BND.ADDR包含了关联的IP地址。此处所提供的BND.ADDR通常情况不同于客户机连接到SOCKS服务器所用的IP地址，因为这些服务器提供的经常都是多址的(muti-homed)。都期望SOCKS主机能使用DST.ADDR和DST.PORT,连接请求评估中的客户端源地址和端口。 

### BIND 

BIND请求被用在那些需要客户机接受到服务器连接的协议中。FTP就是一个众所周知的例子，它通过使用命令和状态报告建立最基本的客户机-服务器连接，按照需要使用服务器-客户端连接来传输数据。(例如：ls,get,put) 都期望在使用应用协议的客户端在使用CONNECT建立首次连接之后仅仅使用BIND请求建立第二次连接。都期望SOCKS主机在评估BIND请求时能够使用ST.ADDR和DST.PORT。 

有两次应答都是在BIND操作期间从SOCKS服务器发送到客户端的。第一次是发送在服务器创建和绑定一个新的socket之后。BIND.PORT域包含了SOCKS主机分配和侦听一个接入连接的端口号。BND.ADDR域包含了关联的IP地址。　　

客户端具有代表性的是使用这些信息来通报应用程序连接到指定地址的服务器。第二次应答只是发生在预期的接入连接成功或者失败之后。在第二次应答中，BND.PORT和BND.ADDR域包含了欲连接主机的地址和端口号。 

### UDP ASSOCIATE(不太懂) 

UDP连接请求用来建立一个在UDP延迟过程中操作UDP数据报的连接。DST.ADDR和DST.PORT域包含了客户机期望在这个连接上用来发送UDP数据报的地址和端口。服务器可以利用该信息来限制至这个连接的访问。如果客户端在UDP连接时不持有信息，则客户端必须使用一个全零的端口号和地址。 

当一个含有UDP连接请求到达的TCP连接中断时，UDP连接中断。 

在UDP连接请求的回应中，BND.PORT和BND.ADDR域指明了客户端需要被发送UDP请求消息的端口号/地址。 

### 回应过程 

当一个回应(REP值非X'00')指明失败时，SOCKS主机必须在发送后马上中断该TCP连接。该过程时间必须为在侦测到引起失败的原因后不超过10秒。 

如果回应代码(REP值为X'00')时，则标志成功，请求或是BIND或是CONNECT，客户机现在就可以传送数据了。如果所选择的认证方法支持完整性、认证机制和/或机密性的封装，则数据被方法选择封装包来进行封装。类似，当数据从客户机到达SOCKS主机时，主机必须使用恰当的认证方法来封装数据。 

## 7．基于UDP客户机的程序 

一个基于UDP的客户端必须使用在BND.PORT中指出的UDP端口来发送数据报到UDP延迟服务器，而该过程是作为对UDP连接请求的回应而进行的。如果所选择的认证方法提供认证机制、完整性、和/或机密性，则数据报必须使用恰当的封装套给予封装。每一个UDP数据报携带一个UDP请求的报头(header): 

| RSV | FRAG | ATYP | DST.ADDR | DST.PORT |   DATA   | 
| :-: | :--: | :--: | :------: | :------: | :------: | 
|  2  |  1   |  1   | Variable |    2     | Variable | 

UDP请求报头是： 

- RSV Reserved X'0000' 
- FRAG Current fragment number 
- ATYP address type of following addresses: 

  - IP V4 address: X'01' 
  - DOMAINNAME: X'03' 
  - IP V6 address: X'04' 

- DST.ADDR desired destination address 
- DST.PORT desired destination port 
- DATA user data 

当一个UDP延迟服务器决定延迟一个UDP数据报时，它会按兵不动，对客户机无任何通报。类似的，它会将它不能或不打算延迟的数据报Drop？掉。当一个UDP延迟服务器接收到一个来自远程主机的延迟数据报，它必须使用上面的UDP请求报头来封装该数据报，和任何认证方法选择的封装。 

一个UDP延迟服务器必须从SOCKS服务器获得所期望的客户机的IP地址，而该客户机要发送数据报到BND.PORT--在至UDP连接的回应中已经给出。UDP延迟服务器还必须drop掉除了特定连接中的一条记录之外的其它的所有源IP地址。 

FRAG域指出了数据报是否为大量的数据片(flagments)中的一片。如果标明了，高序(high-order)位说明是序列的结束段，而值为X'00'则说明该数据报是独立的。值介于1-127之间片断位于数据片序列中间。每一个接收端都有一个和这些数据片相关的重组队列表(REASSEMBLY QUEUE)和一个重组时间表(REASSEMBLY TIMER)。重组队列必须被再次初始化并且相关联的数据片必须被丢掉，而无论该重组时间表是否过期，或者一个新的携带FRAG域的数据报到达，并且FRAG域的值要小于正在进行的数据片序列中的FRAG域的最大值。且重组时间表必须不少于5秒。无论如何最好避免应用程序直接与数据片接触(？)。 

数据片的执行是可选的，一个不支持数据片的执行必须drop掉任何除了FRAG域值为X'00'了数据报。 

一个利用SOCKS的UDP程序接口必须预设有效的缓冲区来装载数据报，并且系统提供的实际缓冲区的空间要比数据报大： 

- if ATYP is X'01' - 10+method_dependent octets smaller 
- if ATYP is X'03' - 262+method_dependent octets smaller 
- if ATYP is X'04' - 20+method_dependent octets smaller 

## 8、安全性考虑

这篇文档描述了一个用来透过IP网络防火墙的应用层协议。这种传输的安全性在很大程度上依赖于特定实现所拥有以及在SOCKS客户与SOCKS服务器之间经协商所选定的特殊的认证和封装方式。系统管理员需要对用户认证方式的选择进行仔细考虑。 

## 9、引用

[^1]: Koblas, D., "SOCKS", Proceedings: 1992 Usenix Security Symposium.

Author's Address

  Marcus Leech
  Bell-Northern Research Ltd
  P.O. Box 3511, Stn. C,
  Ottawa, ON
  CANADA K1Y 4H7
  Phone: (613) 763-9145
  EMail: mleech@bnr.ca

## 10、译者

Radeon（Radeon bise@cmmail.com）

