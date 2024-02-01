# JWT攻击

## 认证方式基础

### session认证

传统session认证有很多问题，因为session是存在服务端的，所以一个是当用户多的时候，服务端压力会很大，

再一个用户认证记录如果保存在内存中，那么意味着该用户下次请求时还必须是这台服务器，这种情况就不适用于分布式的应用。

### token认证

现在使用token认证的也有很多，客户端输入用户名密码，服务器验证通过后会给一个token，客户端存储下这个token，以后每次请求时带上就可以了。

### jwt认证

jwt也类似token，由三段加密信息组成，每段用点连接，格式如下：

eyBhbGcgOiBIUzI1NiwgdHlwIDogSldUIH0K.eyB1c2VyX25hbWUgOiBhZG1pbiB9Cg.4Hb/6ibbViPOzq9SJflsNGPWSk6B8F6EqVrkNjpXh7M

第一段是标头(header)部分，header部分主要声明了类型（一般就是jwt），和加密算法（常用的就是HMAC、SHA256），使用base64编码，所以直接解码即可查看明文。

    { alg : HS256, typ : JWT }

base64url encoded string: eyBhbGcgOiBIUzI1NiwgdHlwIDogSldUIH0K

第二段是有效载荷(payload)部分，一般请求的相关信息都会放在这个段中，比如用户ID、用户名等等，使用base64编码，所以也可以直接查看明文。

    {
     "user_name" : "admin",
    }

base64url encoded string: eyB1c2VyX25hbWUgOiBhZG1pbiB9Cg

第三段是签名(signature)部分，签名是用于验证令牌未被篡改的部分。会将header和payload进行base64编码，然后以header中的加密方式+secret_key盐组合加密，看不了明文。

    signature = HMAC-SHA256(base64urlEncode(header) + '.' + base64urlEncode(payload), secret_key)

对于这个特定的令牌，字符串“eyBhbGcgOiBIUzI1NiwgdHlwIDogSldUIH0K.eyB1c2VyX25hbWUgOiBhZG1pbiB9Cg”使用HS256算法和密钥“key”进行签名。

生成的字符串是 4Hb/6ibbViPOzq9SJflsNGPWSk6B8F6EqVrkNjpXh7M。

通过将每个部分（标头、有效载荷和签名）与每个部分之间“.”连接起来，可获得完整的令牌，如下所示：

eyBhbGcgOiBIUzI1NiwgdHlwIDogSldUIH0K.eyB1c2VyX25hbWUgOiBhZG1pbiB9Cg.4Hb/6ibbViPOzq9SJflsNGPWSk6B8F6EqVrkNjpXh7M

## jwt解码

官网可以实现解码的效果: jwt.io

## 攻击方式

### 算法为none

jwt的header部分中，alg字段指定了加密算法，可以尝试将其设置为None，把signature签名设置为空，导致任何token都有效。

在程序开发过程中，如果开发为了方便调试开启了运行加密为None，上线又忘记了关闭，则存在该问题。

### 算法修改

还有一种是修改加密算法，jwt中常用的一个是RS256非对称，一个是HS256对称，对称加密只有一个密钥，非对称有两个，即私钥加密，公钥解密。

如果程序使用的是RS256，而我们把它改为HS256，即由非对称改为了对称，那么这时后端就会使用公钥作为解密密钥，然后使用HS256算法来验证签名。

即攻击者将header算法改为HS256，使用RS256的公钥对数据进行签名。当后端程序允许同时使用这两种算法时，就会产生这个问题。

### 签名失效问题

如果后端程序没有对签名做校验，那么攻击者就可以修改payload中的内容，达到绕过等目的，例如user是test，则可以改为admin：

### 暴力破解

当程序使用了对称加密，例如HS256，那么就可以尝试去暴破密钥，这里使用c-jwt-cracker

    https://github.com/brendan-rius/c-jwt-cracker

命令如下：

    git clone git://github.com/brendan-rius/c-jwt-cracker
    cd c-jwt-cracker
    make
    ./jwtcrack xxxxxxxxxxx.xxxxxxxxxxxxxxxx.xxxxxxxxxxxxxxxxxx

比如暴破的密钥结果为123456，那么就可以利用密钥来修改jwt的内容，jwt官网提供了修改：

### 密钥泄露

泄露方面，比如git泄露，目录遍历、文件读取等等获取到密钥。

### 令牌刷新

比如程序使用了jwt，且设置了jwt的失效时间，这时jwt失效后再次请求，为了不让用户再输入账号密码，则添加了一个refersh_token机制，即刷新token字段，

该字段一般都是一串随机的字符串，比如请求jwt，返回了如下包内容：

    {"code":0,"data":{"access_token":"XXX.YYY.ZZZ","access_token_expiration":"Thursday, November 9th, 2017, 10:27:33 PM","refresh_token":"ABC123"}}

危害在于后端如果没有对refresh_token和access_token做检验匹配，那么我们就可以拿着自己的refresh_token去获取别人的access_token值。

### KID

KID是header中的一个可选字段，key id的缩写，字面理解即key的序号，用来表示使用密钥几来验证，比如下面kid为1，则代表使用密钥1来做验证：

    {
        "alg": "HS256",
        "typ": "JWT",
        "kid": "1"
    }

危害在于我们可以修改kid达到一些攻击目的，例如后端接口kid后会放到sql语句中进行查询获取相关密钥，那么就可以尝试下sql注入问题：

    "kid":"aaaaaaa' UNION SELECT 'key';--"

如果值为文件路径，则可以尝试文件读取：

    "kid":"/etc/passwd"

如果是文件名，那么后端可能会使用一些打开文件的函数，有些函数支持命令执行：

    "key_file" | whoami