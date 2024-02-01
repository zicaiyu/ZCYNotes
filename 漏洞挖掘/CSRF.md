# CSRF

CSRF（ Cross-site request forgery 跨站请求伪造），是一种挟持用户在已登录的web应用上执行非本意操作的攻击手法
 
## 成因

这个网站Cookie没有过期且没有退出网站，那么只要是访问这个网站，都会携带上这个Cookie

## 防御绕过

### 基于CSRF Token防御绕过

#### 取决于请求方法的CSRF Token验证

某些应用程序在请求使用POST方式时会验证令牌，但在使用GET方法时跳过验证。

在这种情况下，攻击者可以切换到GET方法来绕过验证并进行CSRF攻击。

#### CSRF Token的验证取决于Token是否存在

某些应用程序在令牌存在时正确验证令牌，但如果省略令牌则跳过验证。

在这种情况下，攻击者可以删除包含令牌的整个参数（不仅仅是它的值）以绕过验证并进行CSRF攻击。

#### CSRF Token不绑定到用户会话

某些应用程序不会验证令牌与发出请求的用户是否属于同一会话，相反，应用程序维护它已发布的全局令牌池，并接受该池中出现的任何令牌。

在这种情况下，攻击者可以使用自己的账号登录应用程序，获取有效令牌，然后在CSRF攻击中将攻击令牌提供给受害用户。

举例

    比如：修改邮箱
    1.抓包，抓取修改邮箱的请求，单击右键生成CSRF POC
    2.在抓一次包，提取该请求的CSRF令牌，然后将该包丢弃
    3.将提取的CSRF令牌替换掉CSRF POC里的CSRF旧令牌，然后将邮箱修改为受害者的邮箱
    4.将CSRF HTML放到服务器上，然后发送给受害者
    
#### CSRF Token绑定到非会话cookie

在上述漏洞的一个变体中，一些应用程序确实将CSRF令牌绑定到一个cookie，但没有绑定到用于跟踪会话的同一个cookie。

当应用程序使用两个不同的框架时，很容易发生这种情况，一个用于会话处理，一个用于CSRF保护，但它们没有集成在一起：

这种情况更难利用，但仍然很脆弱。如果网站包含任何允许攻击者在受害者浏览器中设置cookie的行为，那么就有可能发生攻击。

攻击者可以使用自己的账户登录应用程序，获取有效的令牌和关联的cookie，利用cookie设置行为将其cookie放入受害者的浏览器，并在CSRF攻击中将其令牌提供给受害者。

举例

    比如，修改邮箱
    1.用另一个浏览器（隐私模式）登录账号B，通过抓包获取CSRF的key和value，然后替换到账号A上
    2.因为CSRF的key是在Cookie上的，想要替换得结合其他漏洞（CLRF注入）
    比如在查询接口，查询成功会有Set-Cookie，把查询的内容放入到Cookie里，这时就通过CLRF注入
    重新设置新的CSRF key
    3.生成CSRF POC
    在POC上加入下面代码用于更改受害者的CSRF key
    <img src="$cookie-injection-url" onerror="document.forms[0].submit()">
    $cookie-injection-url就是CRLF注入更改了CSRF key的URL
    
#### CSRF Token只是在cookie中复制

在上述漏洞的进一步变体中，一些应用程序不维护任何已发布令牌的服务器端记录，而是在cookie和请求参数中复制每个令牌。

验证后续请求时，应用程序只需验证请求参数中提交的令牌与cookie中提交的值是否匹配。（即Cookie里的CSRF和请求里的CSRF的值一致）

这有时被称为针对CSRF的“双重提交”防御，并且被提倡是因为它易于实现并且避免了对服务器端状态的需要

在这种情况下，如果网站包含任何cookie设置功能，攻击者可以再次执行CSRF攻击。

在这里，攻击者不需要获取自己的有效令牌，他们只是发明了一个令牌（格式上可能要求一致），

利用cookie设置行为将他们的cookie注入受害者的浏览器，并在他们的CSRF攻击中将他们的令牌提供给受害者。

与CSRF令牌绑定到非会话cookie的过程类似，都要结合CSRF注入

### 基于Referer的CSRF防御绕过

#### Referer的验证取决于是否存在标头

某些应用程序会在请求中存在Referer标头时对其进行验证，但如果标头被省略，则会跳过验证。

在这种情况下，攻击者可以制作他们的CSRF漏洞利用，导致受害者用户的浏览器在结果请求中删除Referer标头。

有多种方法可以实现这一点，但最简单的方法是在承载CSRF攻击的HTML页面中使用META标记：

    <meta name="referrer" content="no-referrer">
或者

    <meta name="referer" content="never">
    
生成新的CSRF HTML，另外增加一个<head>头内容来删除referer标头，(即上面的代码)
    
#### 可以绕过Referer的验证

一些应用程序以一种可以绕过的幼稚方式验证Referer标头。

例如，如果应用程序验证Referer中的域以期望值开头，那么攻击者可以将其作为自己域的子域：

    http://目标网站.自己的网站
    
同样，如果应用程序只是验证Referer包含自己的域名，那么攻击者可以将所需的值放在URL的其他位置：

http://自己的网站?目标网站

但当在浏览器中测试验证时，经常会发现这种方法无效。这是因为为了降低以这种方式泄露敏感数据的风险，许多浏览器现在默认从Referer标头中去除查询字符串。

不过可以通过漏洞利用响应设置Referrer-Policy:unsafe-url标头来覆盖这个行为，这样可以确保发送完整的URL，包括查询字符串。
     
抓包生成CSRF POC

在history.pushState()中再修改如下语句，在请求的域后面增加参数以绕过Referer的验证：

    history.pushState("","", "/?目标网站")
    
然后再Head部分添加

    Referrer-Policy:unsafe-url
    
### 基于Origin的CSRF防御

当域名校验不严格时

1.在后面加域名：qq.com => qq.com.abc.com

2.将域名拼接:qq.com => abc_qq.com

3.在前面或后面加字符

    qq.com => abcqq.com 
           => qq.comabc.com
           => abc.com/qq.com
           
