---
title: 浅析DNS协议
date: 2019-02-17 20:48:02
tags:
    - DNS
categories: 技术
clearReading: true
thumbnailImage: https://static001.geekbang.org/resource/image/ff/f2/ff7e8f824ebd1f7e16ef5d70cd79bdf2.jpg
thumbnailImagePosition: bottom
autoThumbnailImage: true
metaAlignment: center
coverMeta: in
coverSize: partial
comments: false
meta: false
actions: false
---

在网络中，网站的IP地址很难让人去记住，一般都是使用域名去访问，因此就有了DNS服务器，根据IP查询域名的地址簿。

每台计算器上都有相应的DNS客户端，也称作DNS解析器。通过DNS查询IP地址的操作称为域名解析，DNS解析器实际上是一段代码，包含在了操作系统的Socket库中，Socket是C语言实现的调用网络功能的库。

<!-- excerpt -->

## DNS解析流程

浏览器调用解析器时，切换到了解析器的上下文执行环境切，也就是Socket中的一段代码 ```gethostbyname``` ,这段代码会生成发送给DNS服务器的查询消息，与浏览器生成http请求的过程类似，然后将消息发送给DNS服务器。发送这个过程并不是解析器来执行的，而是委托给了操作系统内部的协议栈去做处理，协议栈通过网卡将消息发送给DNS服务器。

![调用解析器时计算机内部工作流程](https://user-images.githubusercontent.com/9363528/54486364-5d703200-48c2-11e9-9731-669e4b63cfa9.jpg)
  
> 与DNS服务器交互时，他的IP地址不需要查询，而是在操作系统中去设置好的，比如Mac的话，就在 ```系统偏好设置 -> 网络 -> 高级 -> DNS```中去设置。

## DNS的接受和响应消息

来自客户端的查询消息包含3中信息
- 域名
> 服务器、邮件服务器的名称
- Class
> 用来识别网络信息，值为IN，代表了Internet。
- 记录类型
> 表示域名对应何种类型的记录。A: 域名对应IP地址；MX: 域名对应邮件服务器。

DNS服务器中保存了这三种信息对应的数据，类似这样子:


域名 | Class | 记录类型 | 响应数据
---|---|---|---
wdst2.superboss.cc | IN | A | 121.43.69.90
imap.exmail.qq.com | IN | MX | 14.17.57.217
... | ... | ... | ...


根据查询请求返回符合的响应，比如查询wdst2.superboss.cc时，发送DNS服务器的查询信息就包含以下内容:
- 域名:wdst2.superboss.cc
- Class:IN
- 记录类型:A

然后DNS服务器根据存储查找对应返回IP给客户端。  

## 域名层次结构

每个人上网，都需要访问DNS，上网的人分布在全世界各地，如果大家都去同一个地方访问某一台服务器，时延将会非常大。因而，DNS 服务器，一定是设置成**高可用、高并发和分布式**的。

所以DNS的层次是树状结构:

![DNS树状结构](https://user-images.githubusercontent.com/9363528/54486374-6c56e480-48c2-11e9-8669-b952a617b98b.jpg)

> 分配给根DNS服务器的IP全世界仅有13个，不用于域名解析，只用于指引DNS请求。

## 寻找相应的DNS并获取IP

1. 客户端发送一个DNS请求，比如说访问[http://www.superboss.cc/](http://www.superboss.cc/)，这个请求会发送到最近的DNS服务器。

2. 最近的DNS服务器收到来自客户端的请求，会去缓存中找，如果找到了www.superboss.cc，直接返回。没有的话，该DNS会去访问根DNS服务器。

3. 根DNS服务器中根据域名结构判断，属于cc域，返回cc域中的DNS服务器的IP地址。

4. 接着最近DNS服务器又会向cc域的DNS发送请求，cc域判断他的缓存中有么有www.superboss.cc的ip，没有的话返回下面superboss.cc的DNS服务器的IP。

以此类推，直到找到拥有目标域名的IP的DNS服务器，然后返回IP。

总结一下就是下图这么一个流程

![image](https://static001.geekbang.org/resource/image/ff/f2/ff7e8f824ebd1f7e16ef5d70cd79bdf2.jpg)

## 负载均衡
以上这个过程是一次递归DNS服务器，这个过程除了查出映射的IP地址，还起到了负载均衡的作用。

### 内部负载均衡

例如，一个应用要访问数据库，在这个应用里面应该配置这个数据库的 IP 地址，还是应该配置这个数据库的域名呢？显然应该配置域名，因为一旦这个数据库，因为某种原因，换到了另外一台机器上，而如果有多个应用都配置了这台数据库的话，一换 IP 地址，就需要将这些应用全部修改一遍。但是如果配置了域名，则只要在 DNS 服务器里，将域名映射为新的 IP 地址，这个工作就完成了，大大简化了运维。

在这个基础上，我们可以再进一步。例如，A应用要访问B应用，如果配置另外一个应用的 IP 地址，那么这个访问就是一对一的。但是当B应用撑不住的时候，我们其实可以部署多个。但是，A应用，如何在多个之间进行负载均衡？只要配置成为域名就可以了。在域名解析的时候，我们只要配置策略，返回不同的IP，就可以实现负载均衡了。

### 全局负载均衡

为了保证我们的应用高可用，往往会部署在多个机房，每个地方都会有自己的 IP 地址。当用户访问某个域名的时候，这个 IP 地址可以轮询访问多个数据中心。如果一个数据中心因为某种原因挂了，只要在 DNS 服务器里面，将这个数据中心对应的 IP 地址删除，就可以实现一定的高可用。

另外，我们肯定希望北京的用户访问北京的数据中心，上海的用户访问上海的数据中心，这样，客户体验就会非常好，访问速度就会超快。这就是全局负载均衡的概念。

## 参考
> [浏览器的、本地的、hosts的各种DNS缓存里查找后，最后访问的DNS服务器是哪个服务器？](https://segmentfault.com/q/1010000007713951)
> 
> [DNS协议：网络世界的地址簿](https://time.geekbang.org/column/article/9895)
>
>《网络是怎么连接的》