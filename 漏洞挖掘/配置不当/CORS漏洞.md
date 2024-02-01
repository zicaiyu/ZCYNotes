# CORS漏洞

CORS漏洞是服务端配置的规则不当，服务器端没有配置Access-Control-Allow-Origin等字段，导致攻击者可以构造恶意脚本，诱导用户点击获取用户敏感数据

一般有CORS漏洞的地方都有CSRF

## 挖掘思路

在请求包中加上或修改Origin字段，发现响应包的Access-Control-Allow-Origin会随之改变，则存在CORS漏洞

以下情况的任意一种存在则不存在CORS漏洞

* 响应包为HTML格式的就没有CORS，要为json传输
* 对于响应包中的Access-Control-Allow-Origin值一直为*，不随着请求包中的origin改变时，也不存在CORS漏洞
* 数据包中必须包含个人的信息才能响应的，也不存在漏洞，如数据包中必须包含电话号码
* 有token进行验证、有进行签名也不存在漏洞