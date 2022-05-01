# Internet 控制消息协议 Internet Control Message Protocol
内容：是一个错误处理和消息处理协议，是Tcp/Ip协议族不可或缺的一部分，是IP的核心协议。             
作用：提供了有关网络可连接性和路由行为的信息，这些事基于数据报的、无连接协议（IP和UDP）无法传输的。   
ICMP消息用来让主机报告网络状况和问题，也可使主机能够选择网络上的最佳路径。（当数据报不能到达预定接收方时，会有一条ICMP消息告知发送方。当网关或路由器能够把主机引导到一条更好的网络路由上时，将发送一条重定向消息）    
提供有关IP路由行为、可达性、两个特定主机之间的路由、传输错误等各种信息   
运行在网络层   
## 5.1
Reachability可达性：对于任何网络节点，如果要与另一个网络节点进行通信和数据交换，一定存在从发送方到接收方转发数据的某种方法，这一概念称为可达性     
正常情况下，在位于发送发与接收方之间的各种中间设备的本地IP路由表中，可以找到可用的转发路径。   

ICMP如何工作：   
ICMP利用特殊类型的消息，提供了一种将信息返回给发送发的方法，这些信息是关于数据包在转发过程中所经历的路由，包括可达性信息。     
ICMP还提供了一种在因为路由或可达性问题而阻止了IP数据报交付时返回错误信息的方法。   
IP数据报：TCP/IP协议定义了一个在因特网上传输的包，称为IP数据报。IP数据报就是IP协议传输的数据，IP数据报包含地址、路由选择信息和其它为将数据的分组从源地发送到目的地的分组头信息。 这是一个与硬件无关的虚拟包，由首部和数据两部分组成。   

这种能力很好的补充了IP的数据包交付服务，因为ICMP提供了IP自身不能提供同的东西，路由、可达性、控制信息以及交付错误报告。
事实上，ICMP消息不过是特殊格式的IP数据报，它有一个8字节的首部，与一般网络流量中其他IP数据报受到相同的限制。      
### ICMP在网络中的作用


## 5.2 ICMPv4
ICMP是IP的核心协议，计算机操作系统使用ICMPv4，主要用于发送某些错误消息给其他网络节点。      
尽管ICMPv4被认为是一种传输协议，但是它与TCP和UDP不同，并不携带有效载荷，不能被计算机应用程序使用，其支持一系列的网络测试和错误消息。    
ping    
tracert、traceroute，可跟踪从源计算机到达目的地计算机之间的IP数据包，计算数据包在传输过程中经过跳数，以及每跳所需的时间。     
ICMP消息类型：Echo请求、Echo应答、目的地不可达、路由器公告等。
## 5.21 RFC 792

## 5.22 ICMPv4的首部
IP首部协议字段的值1表示ICMP首部紧跟在该IP首部后面。    
ICMP首部由两部分组成：固定部分和可变部分。    
固定部分：   
IP首部后面，ICMP数据包仅包含3个必须字段：类型字段Type、编码字段Code、校验和字段Checksum。    
单，在某些ICMP数据包中，还有一些附加字段，他们提供了消息的信息或细节，或提供了某些特定消息的信息。如：ICMP重定向数据包需要包含用于数据报重定向的网关地址，一旦接收到这样的数据包，主机应该在其路由表中添加一个动态路由项，并立即开始使用新的路由信息。    

校验和字段：仅用于ICMP首部的错误检测，跟在校验和后面的字段依据所发送的特定ICMP消息而变化。    

## 5.23 ICMPv4消息的类型
ICMP的消息类型分为，错误消息和信息消息。    
所有ICMPv4消息都使用一种常用消息格式，并使用一组协议规则来发送和接收，但其消息的详细内容因特定的消息类型而不同。    
### 目的地不可达消息
由于IPv4是不可靠协议，不能保证发送的数据包肯定能到达目的地，如果数据包不能到达目的地，那么目的地不可达消息将会与无法交付的部分数据报一起返回给发送节点。
### 源节点抑制消息
作用：告诉源节点，降低往目的地节点发送数据报的速率。
### 超时消息
一、在数据包到达其目的地之前，网络上的路由器吧数据包的生存时间TTL字段减少为0了。   
二、当结点的数据包重组计时器为0时，还有一些消息段没有到达目的地结点。 



