防护策略

同源检测

浏览器在发送请求的时候，会带上origin和referer这两个 HTTP 头部字段，一般情况下，我们可以在收到请求的时候校验这两个字段是否来自可信的域名。

但是，虽然浏览器不允许修改origin和referer，但是有可能由于浏览器本身的机制，或者是攻击者刻意隐藏，导致请求里并没有这两个字段的信息，对于这种情况，可以考虑拦截掉。

Samesite Cookie

cookie 有个SameSite属性可以防 CSRF 攻击，使用SameSite=Lax后，从别的网站提交 POST 表单或发送 ajax 请求时都不会附上 cookie ，这样就从根本上解决 CSRF 攻击。但这个SameSite属性只在现代浏览器上生效， IE 上是不支持的。

CSRF Token

除了传统的 cookie 以外，我们可以添加一个额外的 token 作为用户凭证。
token 的生成

    全局共享：每次登录的时候生成一个随机数写到 cookie 里；以后每次请求接口的时候，前端就从 cookie 中取出这个随机数作为请求参数；后端校验的时候只需要比对接口参数和 cookie 里的值就可以了，也不需要存 session 了（分布式 session 成本较高）。这种方法被称为“双重 cookie”。
    每个接口/表单独立生成：针对每个接口/表单都生成一个独立的 token ，“验证码”就属于这种方案。在这种方案下，生成和储存 token 的成本都比较高，建议只用于敏感接口。

双重 cookie 验证

所谓的双重 cookie ，指的是在请求接口时，除了常规带上 cookie 中的用户凭证信息，如 session_id 外，还把 cookie 中的用户凭证信息读出来附在接口请求参数里；这种方案对比起 CSRF Token 方案来说，好在不需要生成额外的 token ，也同样能够起到防御 CSRF 攻击的效果。

在 LocalStorage 里存放登录凭证

CSRF 的关键是：cookie 是自动附在请求里的，那如果登录凭证不是放在 cookie 而是 LocalStorage 里的话，比如使用 jwt 方案，那就从根本上破解了 CSRF 攻击了，不过这样的话，就需要防止 XSS 攻击了。