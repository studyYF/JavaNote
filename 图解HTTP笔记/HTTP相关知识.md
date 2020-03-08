#### 1.网络模型 应用层、传输层、网络层、数据链路层、物理层
网络层：HTTP协议、FTP协议、DNS 协议
传输层：TCP协议、UDP协议
网络层： IP（internet protocol）协议
数据链路层：硬件
注意： TCP/IP协议是互联网各类协议族的总称
#### * 2.一个HTTP请求的过程

![5040e95ba7487bba24bdf83bdaef1a0d](HTTP相关知识.resources/1045569A-DFB9-4DE5-976F-EC77791A49CD.png)

* 首先作为发送端的客户端在应用层 (HTTP 协议)发出一个想看某个 Web 页面的 HTTP 请求。

接着，为了传输方便，在传输层(TCP 协议)把从应用层处收到的数 据(HTTP 请求报文)进行分割，并在各个报文上打上标记序号及端 口号后转发给网络层。
在网络层(IP 协议)，增加作为通信目的地的 MAC 地址后转发给链 路层。这样一来，发往网络的通信请求就准备齐全了。
接收端的服务器在链路层接收到数据，按序往上层发送，一直到应用 层。当传输到应用层，才能算真正接收到由客户端发送过来的 HTTP 请求。
TCP协议通信的三次握手
syn synchronize ack acknowledgement
![206635b33ac986eb95a4639ae7cf3331](HTTP相关知识.resources/5EDE3848-F48F-46C4-973D-B4D7507486BE.png)

#### 3.代理、网关、隧道
**代理**
代理是一种有转发功能的应用程序，它扮演了位于服务器和客户 端“中间人”的角色，接收由客户端发送的请求并转发给服务器，同时 也接收服务器返回的响应并转发给客户端。
每次通过代理服务器转发请求或响应时，会追加写入 Via 首 部信息
![2bd2bff074d907d41b3f15da0d4bac11](HTTP相关知识.resources/A5F7E36E-ED7D-4F95-9570-6291F009B025.png)

缓存代理
代理转发响应时，缓存代理(Caching Proxy)会预先将资源的副本 (缓存)保存在代理服务器上。
当代理再次接收到对相同资源的请求时，就可以不从源服务器那里获 取资源，而是将之前缓存的资源作为响应返回。
透明代理
转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理 (Transparent Proxy)。反之，对报文内容进行加工的代理被称为非 透明代理。
**网关**
网关是转发其他服务器通信数据的服务器，接收从客户端发送来的请 求时，它就像自己拥有资源的源服务器一样对请求进行处理。有时客 户端可能都不会察觉，自己的通信目标是一个网关。网关的工作机制和代理十分相似。而网关能使通信线路上的服务器提供非 HTTP 协议服务。
**隧道**
隧道是在相隔甚远的客户端和服务器两者之间进行中转，并保持双方 通信连接的应用程序。

#### 4,HTTP 首部字段（请求头）
*   通用首部字段
![a6549c53cce8a19ec20250d113a9c04e](HTTP相关知识.resources/F11F5BD4-B3F2-4010-B5B0-66128349D6C2.png)
*   请求首部字段
![3f18565a2b437213fa433309eb4f7896](HTTP相关知识.resources/45FF5A64-61A9-4A26-AF7A-FF741FFD2EB0.png)
*   响应首部字段
![3530a8b37e3d9659ba023638cfae0282](HTTP相关知识.resources/75EF701D-6246-43FC-9B34-EA3C57DDA73F.png)
*   实体首部字段
![c552041d563ef6411e19b4f2ebc38d41](HTTP相关知识.resources/FF12C53A-F402-4E18-A0E7-D771F68F9419.png)




 **Cache-Control 缓存指令** 
*   请求指令
![14b126abcb125b8a88dda3473bfd6806](HTTP相关知识.resources/8999D9C6-6B13-4210-BE1B-5674C84BBFA3.png)

*   响应指令
![16fb7d6b81f449624221a94b8f56d2e2](HTTP相关知识.resources/7A3ECCC2-23D6-433C-9821-6B819E5F48CA.png)

**Connecttion 持久连接  不再转发给代理**

  * Connection: 不再转发的首部字段名
  * Connection: close 请求后断开链接
  * connection: Keep-Alive HTTP/1.1默认链接是持久连接
  
 **Pragma 向后兼容缓存的**
 HTTP/1.1中直接使用Cache-Control 
 
 **Trailer**
 
 **Transfer-Encoding**
 规定了传输报文主体采用的编码，仅对分块传输编码有效
 
 **Upgrade**
 检测HTTP协议及其他协议是否可以使用更高的版本进行通信
 
 **Via**
 追踪客户端与服务器之间的请求和响应报文的传输路径
 ![9b0478c39d3725e172babc1761181993](HTTP相关知识.resources/3F0C6628-3E91-4D74-9EE0-2E2ABE5C1923.png)

**Content-Type 报文主体的对象类型**  
* text/html 

**Accept**
用户代理能够处理的媒体类型及媒体 类型的相对优先级

**Accept-Charset**
通知服务器用户代理支持的字符集及 字符集的相对优先顺序

**Accept-Encoding**
告知服务器用户代理支持的内容编码及 内容编码的优先级顺序

**Accept-Language**
用来告知服务器用户代理能够处理的自然 语言集(指中文或英文等)，以及自然语言集的相对优先级。可一次 指定多种自然语言集。

**Host**
首部字段 Host 会告知服务器，请求的资源所处的互联网主机名和端 口号。Host 首部字段在 HTTP/1.1 规范内是唯一一个必须被包含在请 求内的首部字段。

**If-Match**
首部字段 If-Match，属附带条件之一，它会告知服务器匹配资源所用 的实体标记(ETag)值

*形如 If-xxx 这种样式的请求首部字段，都可称为条件请求。服务器接 收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。*

如果在**If-Modified-Since** 字段指定的日期时间后，资源发生了 更新，服务器会接受请求

**Max-Forward:**
请求被转发的次数，每次减1，到0后直接返回

**Proxy-Authorization** 
接收到从代理服务器发来的认证质询时，客户端会发送包含首部字段 Proxy-Authorization 的请求，以告知服务器认证所需要的信息。

**Range**
Range: bytes=5001-10000

**Referer**
首部字段 Referer 会告知服务器请求的原始资源的 URI。

**ETag**
服务器返回的实体标识，将资源以字符串形式做唯一性标识的方式

**Location: http://www.test.com**
重定向指定的url

**Retry-After： 120**
服务端告知客户端在多久之后再次发送请求

**Server: Apache/2.2.17 (Unix)**
服务器上HTTP服务器应用程序的信息

**Vary**
当代理服务器接收到带有 Vary 首部字段指定获取资源的请求时，如果使用的 Accept-Language 字段的值相同，那么就直接从缓存返回响应。反之，则需要先从源服务器端获取资源后才能作为响应返回

**Allow: GET, HEAD**
告知客户端服务器支持的HTTP请求方式

**Content-Encoding: gzip**
告知客户端服务器对实体的主体部分的编码方式

**Content-Location: http://abc.html**
返回的资源对应的URI

**Content-Type: text/html; charset=UTF-8**
实体主体内对象的媒体类型 和 Accept 字段使用形式一样
				
**Expires: Wed, 04 Jul 2020 08:33:22 GMT**
资源失效日期，超过该时间后，请求会转向源服务器 优先级低于Cache-Control

**Set-Cookie: status=enable; expires=Tue, 04 Jul 2020 07:22:22**
![137a0aeca30b18beb7e4a2474aefbd8f](HTTP相关知识.resources/22D0E694-01F6-4D4A-9E91-BBC8F86A33A5.png)

**Cookie: status=enable**
首部字段 Cookie 会告知服务器，当客户端想获得 HTTP 状态管理支 持时，就会在请求中包含从服务器接收到的 Cookie。接收到多个 Cookie 时，同样可以以多个 Cookie 形式发送。


#### 5.HTTP请求的安全机制

1.**加密通道** 
SSL(Secure Socket Layer) 安全套接层
TLS （Transport Layer Security) 安全层传输协议

2.**加密报文内容**

3.**HTTPS**:

* HTTP加上加密处理和认证以及完整性保护后即是HTTPS（HTTP secure）
* 通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通 信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP。


* 和使用 HTTP 相比，网络负载可能会变慢 2 到 100 倍。除去和 TCP 连接、发送 HTTP 请求 • 响应以外，还必须进行 SSL 通信， 因此整体上处理通信量不可避免会增加。
* 另一点是 SSL 必须进行加密处理。在服务器和客户端都需要进行 加密和解密的运算处理。因此从结果上讲，比起 HTTP 会更多地 消耗服务器和客户端的硬件资源，导致负载增强。