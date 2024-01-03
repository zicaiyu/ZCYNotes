# HTTP协议WAF检测绕过

## 请求方法

常见的请求方法GET、POST、PUT、DELETE

有些接口换了请求方法依然能执行，甚至乱写一个不存在的请求方法都能成功

而WAF对不同的请求方法的检测力度是不一样的

## 参数

### apache/php

请求参数相同取最后一个

Header里有相同的key，value合并

Cookie里有相同的key，取第一个

### tomcat/java

请求参数相同取第一个

Header里有相同的key，取第一个

Cookie里有相同的key，都会取，但不会合并

### IIS/asp

请求参数相同通过,合并（可以尝试url里传一个参数，body来再传同一个参数）

Header里有相同的key，value合并

Cookie里有相同的key，取第一个

> IIS传参允许传Unicode编码，或者在英文字母里插入%字符

## Content-Type

更换Content-Type进行干扰

常见的Content-Type

    Content-Type: application/x-www-form-urlencoded
    Content-Type: application/json
    Content-Type: text/xml
    Content-Type: multipart/form-data
    有时候会加个编码
    Content-Type: text/html; charset=utf-8

## 分块传输

分块之后可以在标示长度处加分号后增加垃圾数据，此处填充位置记为A1

也可以在标示长度的值前面加若干个0，此处填充位置记为A0

> 在apache/php中
> 
> A0+A1包括分号总长度最大为8188，如果有一行超出了长度限制，apache会报413，但前面正确的chunk依旧正确识别。
> 
> 但如果整个body长度超过33180，无法获取
> 
> 同理，如果有一个chunk和实际长度不符合，会报400错误但依旧能取得到对应的值。
>
> Transfer-Encoding: chunked还可以和Content-Length: 8217共存，这种情况常见于找请求走私漏洞，但也有可能干扰waf识别。

> 在tomcat/java中
> 
> A0处仅允许8字节。A1处最大长度8191，并且所有chunk共享这一长度。
> 
> 超出长度后所有chunk将不再解析

> 在IIS/aspx中
> 
> A0+A1，长度限制为16264，也是浮动的。不使用Content-Length的情况下，在A0+A1处填充还很容易发包被拒绝。

> 在flask/python中
> A0处无长度限制，但不支持A1和分号

> 在nginx/php中
> A0和A1处均无长度限制。

也可以对分块传输增加延迟发送

## 检测返回包

有的WAF会检测返回包，通过在请求中添加上请求头

    Except: 100-continue

让返回包加上100的状态码

## 对协议https://中的//进行拦截

url中去掉//依然能识别

    https:jammny.github.io	==> https://jammny.github.io

