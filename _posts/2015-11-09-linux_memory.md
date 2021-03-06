---
date: 2015-10-29 04:46:31.000Z
layout: post
title: linux
tags: 测试 分类
categories: info
---

##### 一、free各列含义
- total：物理内存的总大小
- used：已经使用的物理内存大小
- free：空闲的物理内存大小
- shared：多个进程共享的内存大小
- buffers/cached：磁盘缓存的大小

##### 二、从内核角度看内存使用
- free 是指实际的空闲物理内存，**不包括buffers和cached,buffers**和cache是程序使用的
- 实际上，内核完全控制着内存的使用情况，Linux会在需要内存的时候，或在系统运行逐步推进时，将buffers和cached状态的内存变为free状态的内存，以供系统使用
- 个人这么理解:系统默认加载一部分内存到buffers和cached供程序读写用，如不够，再从free中调，
  如有多，则放回到free中，如free快用完都不够，则从swap中调
  
##### 三、从应用角度看内存使用
- 应用程序可用的物理内存值是Mem项的free值加上buffers和cached值之和，也就是说，**这个free值是包括buffers和cached项大小的**

**linux总是在力求缓存更多的数据和信息，这样再次需要这些数据时可以直接从内存中取，而不需要有一个漫长的磁盘操作**

表单重复提交\单点登录\cookies登录（换IP）
csrf
cookie/session/http请求过程/keepalived/传输加密

最近项目被第三方工具扫描出来有一个Http head xss cross scripting漏洞，为了修复这个，顺便研究了一下跨站脚本攻击的原理， 跨站脚本攻击基本上就是sql注入的html版， 核心内容就是把一段精心设计的脚本通过网页中的html漏洞由HTTP GET/POST传给服务器并执行. XSS主要有两种，一种是注入的链接需要骗人来点击，目的是劫持用户的cookie； 一种是该脚本已经通过此方法注入了DB，每次有人浏览正常的该网站链接都会执行该脚本，理论上java script能做的都可以做。


XSS 全称“跨站脚本”，是注入攻击的一种。**其特点是不对服务器端造成任何伤害**，而是通过一些正常的站内交互途径，例如发布评论，**提交含有 JavaScript 的内容文本**。这时服务器端如果没有过滤或转义掉这些脚本，作为内容发布到了页面上，其他用户访问这个页面的时候就会运行这些脚本


我们知道 AJAX 技术所使用的 XMLHttpRequest 对象都被浏览器做了限制，只能访问当前域名下的 URL，所谓不能“跨域”问题。这种做法的初衷也是防范 XSS，多多少少都起了一些作用


安全的cookie登录状态设计方案
我们知道web是基于HTTP协议传输的，明文传输是极其危险的，随便哪个抓包工具分析下数据包，就over啦，一个加密的传输过程应该包括两部分，一部分为身份认证，用户鉴别这个用户的真伪；另外一部分为数据加密，用于数据的保密。
我大概是这样做的：
（1）生成用户验证token
    用户登录后我会生成一个token，该token可能由如下信息组成：username+ip+expiration+salt【只是举例】,然后将组成信息用可逆加密函数加密得到token，并将该token保存到数据库，写入cookie；
（2）最后这样去校验信息，判断用户的登录状态
    将token解密，验证用户username,如果存在，继续；然后验证token是否和存入数据库的token相同，如果相同继续；验证cookie的有效期expiration，如果有效继续；验证ip是否变化，若变化跳入登录。。。。。。甚至还可以验证user agent.
 (3)可以做到单终端登录，可以将token放到数据库，每次登录操作必然会改变token的值，另外一端的用户就会token验证失败下线
最后说明：
1.上面保证了token每次登录都会不一样，这回导致之前的token【既cookie】失效
2.cookie的有效期最好不超过一周



Token，就是令牌，最大的特点就是随机性，不可预测。一般黑客或软件无法猜测出来。

那么，Token有什么作用？又是什么原理呢？

Token一般用在两个地方:

1)防止表单重复提交、
2)anti csrf攻击（跨站点请求伪造）。
两者在原理上都是通过session token来实现的。当客户端请求页面时，服务器会生成一个随机数Token，并且将Token放置到session当中，然后将Token发给客户端（一般通过构造hidden表单）。下次客户端提交请求时，Token会随着表单一起提交到服务器端。
然后，如果应用于“anti csrf攻击”，则服务器端会对Token值进行验证，判断是否和session中的Token值相等，若相等，则可以证明请求有效，不是伪造的。
不过，如果应用于“防止表单重复提交”，服务器端第一次验证相同过后，会将session中的Token值更新下，若用户重复提交，第二次的验证判断将失败，因为用户提交的表单中的Token没变，但服务器端session中Token已经改变了。

上面的session应用相对安全，但也叫繁琐，同时当多页面多请求时，必须采用多Token同时生成的方法，这样占用更多资源，执行效率会降低。因此，也可用cookie存储验证信息的方法来代替session Token。比如，应对“重复提交”时，当第一次提交后便把已经提交的信息写到cookie中，当第二次提交时，由于cookie已经有提交记录，因此第二次提交会失败。
不过，cookie存储有个致命弱点，如果cookie被劫持（xss攻击很容易得到用户cookie），那么又一次gameover。黑客将直接实现csrf攻击。

所以，安全和高效相对的。具体问题具体对待吧。

 

此外，要避免“加token但不进行校验”的情况，在session中增加了token，但服务端没有对token进行验证，根本起不到防范的作用。

还需注意的是，对数据库有改动的增删改操作，需要加token验证，对于查询操作，一定不要加token，防止攻击者通过查询操作获取token进行csrf攻击。但并不是这样攻击者就无法获得token，只是增大攻击成本而已。


session/分布式session/token

首先我猜想你所说的session是php中的session，那么这个session中的数据并不是储存在客户端上的，而是储存在服务器上的；而客户端使用cookie储存一个服务器分配的客户端序号，当客户端请求服务器时，会将这个cookie（即客户端序号）传递给服务器，服务器通过配对获取session内容。所以这个token是很难伪造的，而一般来说，大型项目并不会使用session来储存数据（因为这样会增加分布式架构的难度）。回到主要问题，如果说这个token是指的用户登录的凭据，并用以维持登录状态的话，也就是说一个用户必须要输入用户名密码并验证通过后，服务器才会分配一个cookie，传回并储存在客户端作为凭证（同时储存在服务器上）。因此并不是每个人都可以获得这个token，只有能提供正确用户密码的客户端才可以。至于你说的CSRF防御的话，需要说明的是：例如A网站中加入了网银网站B的伪造连接，而当前用户用户储存了B的cookie，则通过访问A网站可能会发送一些伪造请求。但是A网站本身无法获取你该用户的token，因此token是相对安全的。而为了防范CSRF需要做的是不要将http(s)请求的参数放在get中，而应该放在post/put/delete中。然后在页面上加上再一个变化的token（每次刷新页面后都会改变，不同于之前的token），请求服务器时都验证该token，就可以了，因为B网站中页面上的内容是A网站怎么也得不到的。

