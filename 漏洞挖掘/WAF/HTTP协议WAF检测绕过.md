# HTTP协议WAF检测绕过

## 请求方法

常见的请求方法GET、POST、PUT、DELETE

有些接口换了请求方法依然能执行，甚至乱写一个不存在的请求方法都能成功

而WAF对不同的请求方法的检测力度是不一样的

## 参数

## 分块传输

## 检测返回包

有的WAF会检测返回包，通过在请求中添加上请求头

    Except: 100-continue

让返回包加上100的状态码

## 对协议https://中的//进行拦截

url中去掉//依然能识别

    https:jammny.github.io	==> https://jammny.github.io

