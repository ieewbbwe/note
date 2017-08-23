#概述
开发中通讯这块也是必不可少的，无论什么产品都离不开与后台的交互。在数据通信中我们经常使用的是Http，json，但是也有许多场景中需要用到长连接，广播包等需求，今天开始研究下项目中的数据通讯技术。

##通讯协议

先来简单说下通讯协议，TCP、udp、http、rtsp、xmpp、icmp、smtp等等这些都是协议，那么什么是协议？

就是大家普遍遵守的一套规范，通讯协议就是在数据传输过程中对数据格式，传输速度，传输步奏，校验方式等问题作出的统一约定，交互双方必须遵守。有个听起来比较牛逼的叫法叫链路控制规程。

举个通俗的例子，中国人和中国人讲普通话大家都可以沟通信息，中国人和美国人一个说普通话一个说英文就听不懂了。就和《宪法》一样，是大家都必须遵守的约定。

通讯协议有很多，那么我们通常使用的TCP/IP、Http关系看下图就能看明白了

- TCP，经过三次握手，可靠的链接，一般比较重要的信息会使用这个协议
- UDP，不可靠链接，但是效率比TCP高，一般视频，语音等数据会用这个协议

TCP、UDP是传输层协议，Http、rtsp、ftp这些是应用层协议。

有一个形象的例子帮助理解IP、tcp、http的关系，将IP理解为高速公路，那么tcp就是跑在高速路上的卡车，http就是这辆卡车的货物。

网络有一段比较容易理解的介绍：“我们在传输数据时，可以只使用（传输层）TCP/IP协议，但是那样的话，如果没有应用层，便无法识别数据内容，如果想要使传输的数据有意义，则必须使用到应用层协议，应用层协议有很多，比如HTTP、FTP、TELNET等，也可以自己定义应用层协议。WEB使用HTTP协议作应用层协议，以封装HTTP 文本信息，然后使用TCP/IP做传输层协议将它发到网络上。”

<table cellspacing="1" cellpadding="5">
<tbody>
<tr>
<td>7</td>
<td><strong>应用层</strong></td>
<td>例如<a title="HTTP" href="http://zh.wikipedia.org/wiki/HTTP">HTTP</a>、<a title="SMTP" href="http://zh.wikipedia.org/wiki/SMTP">SMTP</a>、<a title="SNMP" href="http://zh.wikipedia.org/wiki/SNMP">SNMP</a>、<a title="FTP" href="http://zh.wikipedia.org/wiki/FTP">FTP</a>、<a title="Telnet" href="http://zh.wikipedia.org/wiki/Telnet">Telnet</a>、<a title="SIP" href="http://zh.wikipedia.org/wiki/SIP">SIP</a>、<a title="SSH" href="http://zh.wikipedia.org/wiki/SSH">SSH</a>、<a title="NFS" href="http://zh.wikipedia.org/wiki/NFS">NFS</a>、<a title="RTSP" href="http://zh.wikipedia.org/wiki/RTSP">RTSP</a>、<a title="XMPP" href="http://zh.wikipedia.org/wiki/XMPP">XMPP</a>、<a title="Whois" href="http://zh.wikipedia.org/wiki/Whois">Whois</a>、<a title="ENRP" href="http://zh.wikipedia.org/w/index.php?title=ENRP&amp;action=edit&amp;redlink=1">ENRP</a></td>


</tr>
<tr>
<td>6</td>
<td><strong>表示层</strong></td>
<td>例如<a title="External Data Representation" href="http://zh.wikipedia.org/w/index.php?title=External_Data_Representation&amp;action=edit&amp;redlink=1">XDR</a>、<a title="Abstract Syntax Notation 1" href="http://zh.wikipedia.org/w/index.php?title=Abstract_Syntax_Notation_1&amp;action=edit&amp;redlink=1">ASN.1</a>、<a title="Server message block" href="http://zh.wikipedia.org/w/index.php?title=Server_message_block&amp;action=edit&amp;redlink=1">SMB</a>、<a title="Apple Filing Protocol" href="http://zh.wikipedia.org/w/index.php?title=Apple_Filing_Protocol&amp;action=edit&amp;redlink=1">AFP</a>、<a title="NetWare Core Protocol" href="http://zh.wikipedia.org/w/index.php?title=NetWare_Core_Protocol&amp;action=edit&amp;redlink=1">NCP</a></td>


</tr>
<tr>
<td>5</td>
<td><strong>会话层</strong></td>
<td>例如<a title="Aggregate Server Access Protocol" href="http://zh.wikipedia.org/w/index.php?title=Aggregate_Server_Access_Protocol&amp;action=edit&amp;redlink=1">ASAP</a>、<a title="Transport Layer Security" href="http://zh.wikipedia.org/wiki/Transport_Layer_Security">TLS</a>、<a title="SSH" href="http://zh.wikipedia.org/wiki/SSH">SSH</a>、ISO 8327 / CCITT X.225、<a title="Remote procedure call" href="http://zh.wikipedia.org/w/index.php?title=Remote_procedure_call&amp;action=edit&amp;redlink=1">RPC</a>、<a title="NetBIOS" href="http://zh.wikipedia.org/w/index.php?title=NetBIOS&amp;action=edit&amp;redlink=1">NetBIOS</a>、<a title="AppleTalk" href="http://zh.wikipedia.org/w/index.php?title=AppleTalk&amp;action=edit&amp;redlink=1">ASP</a>、<a title="Winsock" href="http://zh.wikipedia.org/w/index.php?title=Winsock&amp;action=edit&amp;redlink=1">Winsock</a>、<a title="Berkeley sockets" href="http://zh.wikipedia.org/wiki/Berkeley_sockets">BSD sockets</a></td>


</tr>
<tr>
<td>4</td>
<td><strong>传输层</strong></td>
<td>例如<a title="TCP" href="http://zh.wikipedia.org/wiki/TCP">TCP</a>、<a title="User Datagram Protocol" href="http://zh.wikipedia.org/wiki/User_Datagram_Protocol">UDP</a>、<a title="Real-time Transport Protocol" href="http://zh.wikipedia.org/w/index.php?title=Real-time_Transport_Protocol&amp;action=edit&amp;redlink=1">RTP</a>、<a title="Stream Control Transmission Protocol" href="http://zh.wikipedia.org/w/index.php?title=Stream_Control_Transmission_Protocol&amp;action=edit&amp;redlink=1">SCTP</a>、<a title="Sequenced packet exchange" href="http://zh.wikipedia.org/w/index.php?title=Sequenced_packet_exchange&amp;action=edit&amp;redlink=1">SPX</a>、<a title="AppleTalk" href="http://zh.wikipedia.org/w/index.php?title=AppleTalk&amp;action=edit&amp;redlink=1">ATP</a>、<a title="IL Protocol" href="http://zh.wikipedia.org/w/index.php?title=IL_Protocol&amp;action=edit&amp;redlink=1">IL</a></td>


</tr>
<tr>
<td>3</td>
<td><strong>网络层</strong></td>
<td>例如<a title="网际协议" href="http://zh.wikipedia.org/wiki/%E7%BD%91%E9%99%85%E5%8D%8F%E8%AE%AE">IP</a>、<a title="ICMP" href="http://zh.wikipedia.org/wiki/ICMP">ICMP</a>、<a title="IGMP" href="http://zh.wikipedia.org/wiki/IGMP">IGMP</a>、<a title="IPX" href="http://zh.wikipedia.org/wiki/IPX">IPX</a>、<a title="BGP" href="http://zh.wikipedia.org/wiki/BGP">BGP</a>、<a title="OSPF" href="http://zh.wikipedia.org/wiki/OSPF">OSPF</a>、<a title="RIP" href="http://zh.wikipedia.org/wiki/RIP">RIP</a>、<a title="IGRP" href="http://zh.wikipedia.org/wiki/IGRP">IGRP</a>、<a title="EIGRP" href="http://zh.wikipedia.org/wiki/EIGRP">EIGRP</a>、<a title="ARP" href="http://zh.wikipedia.org/wiki/ARP">ARP</a>、<a title="RARP" href="http://zh.wikipedia.org/wiki/RARP">RARP</a>、&nbsp;<a title="X.25" href="http://zh.wikipedia.org/wiki/X.25">X.25</a></td>


</tr>
<tr>
<td>2</td>
<td><strong>数据链路层</strong></td>
<td>例如<a title="以太网" href="http://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E7%BD%91">以太网</a>、<a title="令牌环" href="http://zh.wikipedia.org/wiki/%E4%BB%A4%E7%89%8C%E7%8E%AF">令牌环</a>、<a title="HDLC" href="http://zh.wikipedia.org/wiki/HDLC">HDLC</a>、<a title="帧中继" href="http://zh.wikipedia.org/wiki/%E5%B8%A7%E4%B8%AD%E7%BB%A7">帧中继</a>、<a title="ISDN" href="http://zh.wikipedia.org/wiki/ISDN">ISDN</a>、<a title="异步传输模式" href="http://zh.wikipedia.org/wiki/%E5%BC%82%E6%AD%A5%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%BC%8F">ATM</a>、<a title="IEEE 802.11" href="http://zh.wikipedia.org/wiki/IEEE_802.11">IEEE 802.11</a>、<a title="FDDI" href="http://zh.wikipedia.org/wiki/FDDI">FDDI</a>、<a title="PPP" href="http://zh.wikipedia.org/wiki/PPP">PPP</a></td>


</tr>
<tr>
<td>1</td>
<td><strong>物理层</strong></td>
<td>例如<a title="线路" href="http://zh.wikipedia.org/w/index.php?title=%E7%BA%BF%E8%B7%AF&amp;action=edit&amp;redlink=1">线路</a>、<a title="无线电" href="http://zh.wikipedia.org/wiki/%E6%97%A0%E7%BA%BF%E7%94%B5">无线电</a>、<a title="光纤" href="http://zh.wikipedia.org/wiki/%E5%85%89%E7%BA%A4">光纤</a>、<a title="信鸽" href="http://zh.wikipedia.org/wiki/%E4%BF%A1%E9%B8%BD">信鸽</a></td>

</tr>

</tbody>

</table>

此表格转自：[TCP/UDP 协议，和 HTTP、FTP、SMTP，区别及应用场景](http://www.cnblogs.com/duanxz/p/5127561.html)

## Socket

### 概念

那么我们说的Socket又是什么？

是一种抽象层，应用程序通过它来发送和接收数据，简单来说，Socket提供了程序内部与外界通信的端口并为通信双方的提供了数据传输通道。

- http连接使用的是“请求—响应方式”，即在请求时建立连接通道，当客户端向服务器发送请求后，服务器端才能向客户端返回数据。
- Socket通信则是在双方建立起连接后就可以直接进行数据的传输，在连接时可实现信息的主动推送，而不需要每次由客户端想服务器发送请求。 

### Socket的使用



