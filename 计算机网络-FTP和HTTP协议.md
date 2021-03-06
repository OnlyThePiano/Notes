## 目录

- [FTP 文件传输协议](#ftp-------)
    + [1. FTP 的基本工作原理](#1-ftp--------)
    + [2. TFTP 简单文件传送协议](#2-tftp---------)
- [HTTP 超文本传输协议](#http--------)
  * [一、万维网 WWW](#------www)
    + [1. HTTP 超文本传输协议](#1-http--------)
    + [2. 统一资源定位符 URL](#2---------url)
    + [3. 万维网的文档](#3-------)







# FTP 文件传输协议

**文件传输协议 FTP (File Transfer Protocol)** 是互联网上使用得最广泛的文件传送协议。FTP 提供交互式的访问，允许客户指明文件的类型与格式 (如指明是否使用`ASCII码` )，并允许文件具有存取权限(如访问文件的用户必须经过授权，并输入有效的口令)。FTP 屏蔽了各计算机系统的细节，因而适合于在异构网络中任意计算机之间传送文件。



### 1. FTP 的基本工作原理

网络环境中的一项基本应用就是将文件从一台计算机中复制到另一台可能相距很远的计算机中。初看起来，在两台主机之间传送文件是很简单的事情。其实这往往非常困难。原因是众多的计算机厂商研制出的文件系统多达数百种，且差别很大。

**经常遇到的问题是：**

1. 计算机存储数据的格式不同；
2. 文件的目录结构和文件名的规定不同；
3. 对于相同的文件存取功能，操作系统使用的命令不同；
4. 访问控制方法不同。



文件传输协议 FTP 只提供文件传送的一些基本的服务，它使用TCP可靠的运输服务。FTP 的主要功能是减少或消除在不同操作系统下处理文件的不兼容性。

---



### 2. TFTP 简单文件传送协议

TCP/IP 协议族中还有一个 **简单文件传送协议TFTP (Trivial File Transfer Protocol** ，它是一个很小且易于实现的文件传送协议。

 虽然 TFTP 也使用客户服务器方式，但它使用 UDP 数据报，因此 TFTP 需要有自己的差错改正措施。TFTP 只支持文件传输而不支持交互。TFTP 没有一个庞大的命令集，没有列目录的功能，也不能对用户进行身份鉴别。

**- TFTP 的主要优点有两个：**

* TFTP 可用于 UDP 环境。例如，当需要将程序或文件同时向许多机器下载时就往往需要使用 TFTP 。
* TFTP 代码所占的内存较小。这对较小的计算机或某些特殊用途的设备是很重要的。



# HTTP 超文本传输协议

## 一、万维网 WWW

`万维网 www (World Wide Web)` 并非某种特殊的计算机网络。万维网是一个大规模的、联机式的信息储藏所，英文简称为 `Web` 。万维网用链接的方法能非常方便地从互联网 (Internet)上的一个站点访问另一个站点 (也就是所谓的“链接到另一个站点”) ，从而主动地按需获取丰富的信息。

万维网是一个分布式的 `超媒体(hypermedia)系统` ，它是 `超文本(hypertext)系统` 的扩充。

> **超文本** 是指包含指向其他文档的链接的文本(text)。也就是说，一个超文本由多个信息源链接成，而这些信息源可以分布在世界各地，并且数目也是不受限制的。利用一个链接可使用户找到远在异地的另一个文档，而这又可链接到其他的文档(依此类推)。这些文档可以位于世界上任何一个接在互联网上的超文本系统中。超文本是万维网的基础。



### 1. HTTP 超文本传输协议

超文本是万维网的基础。

**用什么协议来实现互联网 (Internet) 上的各种链接 ？**

答：要使万维网客户程序与万维网服务器程序之间的交互遵守严格的协议，`超文本传送协议HTTP (HyperText Transfer Protocol)` 。HTTP 是一个应用层协议，它使用 TCP 连接进行可靠的传送。

**- HTTP 协议的主要特点：**

1. **HTTP 协议是无连接的：** HTTP 使用了面向连接的 TCP 作为运输层协议，保证了数据的可靠传输。HTTP不必考虑数据在传输过程中被丢弃后又怎样被重传。但是，HTTP 协议本身是无连接的。这就是说，虽然 HTTP 使用了 TCP 连接，但通信的双方在交换 HTTP 报文之前不需要先建立 HTTP 连接。
2. **HTTP 协议是无状态的：** HTTP 协议是无状态的(stateless)。也就是说，同一个客户第二次访问同一个服务器上的页面时，服务器的响应与第一次被访问时的相同(假定现在服务器还没有把该页面更新)，因为服务器并不记得曾经访问过的这个客户，也不记得为该客户曾经服务过多少次。**HTTP 的无状态特性简化了服务器的设计，使服务器更容易支持大量并发的HTTP请求。 ** 



**- 代理服务器：**

代理服务器(proxy server) 是一种网络实体， 它又称为 `万维网高速缓存(Web cache)` 。 代理服务器把最近的一些请求和响应暂存在本地磁盘中。当新请求到达时，若代理服务器发现这个请求与暂时存放的请求相同，就返回暂存的响应，而不需要按 URL 的地址再次去 `互联网 (Internet)` 访问该资源。代理服务器可在客户端或服务器端工作，也可在中间系统上工作。

**- HTTP 的报文结构：**

* 请求报文
* 响应报文

![image-20200726141748292](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200726141748292.png)



看 HTTP 的报文结构，响应报文中的 **状态码** 有五大类，且都是 3 位数字，如下图：

![image-20200726141649082](https://github.com/OnlyThePiano/Notes/blob/master/images/image-20200726141649082.png)



**HTTP 和 HTTPS 的区别 ？**

1. **端口：** 

   HTTP的URL由“http://”起始且默认使用端口80，而HTTPS的URL由“https://”起始且默认使用端口443。

2. **安全性和资源消耗：** 

   HTTP 协议运行在 TCP 之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。HTTPS 是运行在 `SSL/TLS` 之上的 HTTP 协议，`SSL/TLS` 运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源。

   - 对称加密：密钥只有一个，加密解密为同一个密码，且加解密速度快，典型的对称加密算法有DES、AES等；
   - 非对称加密：密钥成对出现（且根据公钥无法推知私钥，根据私钥也无法推知公钥），加密解密使用不同密钥（公钥加密需要私钥解密，私钥加密需要公钥解密），相对对称加密速度较慢，典型的非对称加密算法有RSA、DSA等。



---



### 2. 统一资源定位符 URL

**分布在整个 `互联网 (Internet)` 的万维网文档该怎么去标志呢 ？**

答：万维网使用 `统一资源定位符 URL (Uniform Resource Locator)` 来标
志万维网上的各种文档，并使每一个文档在整个互联网的范围内具有唯一的标识符URL。

`URL` 给资源的位置提供一种抽象的识别方法，并用这种方法给资源定位。只要能够对资源定位，系统就可以对资源进行各种操作，如存取、更新、替换和查找其属性。由此可见，URL 实际上就是在互联网上的资源的地址。只有知道了这个资源在互联网上的什么地方，才能对它进行操作。

**- HTTP 协议的 URL**

* http://<主机>:<端口>/<路径>
* HTTP 的默认端口是80，通常可以省略

* URL 里面的字母不分大小写，但为了便于阅读，有时故意使用一些大写字母。



**打开一个网页，即在浏览器输入 URL 地址，这整个过程会使用哪些协议 ？**

答：过程如下

1. 浏览器查找域名的 IP 地址（DNS解析过程）；
2. 浏览器向 web 服务器发送一个 HTTP 请求；
3. 服务器处理请求返回 HTTP 报文；
4. 浏览器解析渲染页面

---



### 3. 万维网的文档

要使任何一台计算机都能显示出任何一个万维网服务器上的页面，就必须解决页面制作的标准化问题。`超文本标记语言HTML (HyperText Markup Language)` 就是一 种制作万维网页面的标准语言，它消除了不同计算机之间信息交流的障碍。

