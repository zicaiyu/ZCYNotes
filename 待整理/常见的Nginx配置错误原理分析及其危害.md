根目录位置丢失

危害：在某些情况下，访问者可能会访问其他配置文件、访问日志甚至 HTTP 基本身份验证的加密凭据

示例：
server {
listen 5000;
server_name ~^(.+)$;

location /cats {
alias /usr/share/nginx/html/;
}
}
root指令指定 Nginx 的根文件夹。nginx的根文件夹是/etc/nginx，这意味着我们可以访问该文件夹中的文件。上面的配置没有针对/(location / {...})的位置，只有/cats的位置。因此，root 指令会被设置为全局，这意味着对/的请求会将你带到本地路径/etc/nginx。
像GET /nginx.conf这样简单的请求都能显示存储在/etc/nginx/nginx.conf中Nginx 配置文件的内容。如果将根设置为/etc，则对/nginx/nginx.conf的GET请求将显示配置文件。
root指令指定/etc/nginx文件夹位置，可访问该文件夹下所有文件
Nginx 配置文件中，最常见的根路径如下
Default
/usr/share/nginx/html
/path/to/app/current/public
/var/www/vhosts/yoursite.com
/var/www/html
/app/frontend/web
/app/backend/web
/var/www/
/var/www
/www
/usr/share/nginx/www
/home/you/app/current/public
/app/public
/home/USER/www/html
/usr/local/www/nginx-dist
/var/www/public
/app
/var/www/example.com/htdocs
/var/www/maxdegterev/public
/

off-by-slash

危害
这可能导致服务器状态通过 URL公开，或者可能让不希望公开访问的路径可访问
结合一条缺少尾斜杠的location指令与一条alias指令，来读取 Web 应用程序的源代码。鲜为人知的是，
它还可以与其他指令（例如proxy_pass）一起使用。

示例：
location /cats {
alias /usr/share/nginx/html/;
}
路径指向的是 /usr/share/nginx/html/
一条缺少尾斜杠的location指令与一条alias指令
Nginx处理完 /cats_anything 后，其转发（到后端服务器）的请求格式为
http://ip/usr/share/nginx/html/_anything

在cats后加.. 可越权访问 nginx下的文件
http://192.168.240.129/cats../flag.txt

它还可以与其他指令（例如proxy_pass）一起使用

location /image {
proxy_pass http://apache:80/catpicctures/;
}

如果一个 Nginx 服务器运行能在 server 访问的以下配置，则可以假定访问者只能访问http://apache:80/catpictures/下的路径
当请求http://192.168.240.129:5000/image时，Nginx将首先规范化 URL。然后，它会查看前缀/images是否与URL 匹配，本例中是匹配的
然后，服务器从 URL 中删除该前缀，保留/1.jpg路径。再将此路径添加到proxy_pass URL 中，从而得到最终 URL http://apache:80/images//1.jpg
请求http://192.168.240.129:5000/image../可以利用这种错误配置，这将导致 Nginx 请求
http://apache:80/catpictures../，其标准化为http://apache:80/

不安全的变量使用

1、使用 $uri 可导致 CRLF 注入

与Nginx 变量有关的另一个错误配置是使用$uri或$document_uri代替$request_uri。$uri和$document_uri包含标准化的 URI，而 Nginx 中的normalization包括对 URI 解码的 URL。在 Nginx 配置中创建重定向时经常会使用$uri，结果导致 CRLF 注入

一个易受攻击的 Nginx 配置
location /image-credits{
return 302 https://placekitten.com/attribution.html?originalPath=$uri;
}
HTTP 请求的换行符为\r（回车）和\n（换行）。对换行符进行 URL 编码将导致以下字符表示形式％0d％0a。如果将这些字符包含在对配置错误的服务器的一个请求中
（例如http://192.168.240.129/images-credits％0d％0aDetectify:％20clrf），则该服务器将使用一个名为Detectify的新标头进行响应，因为 $uri 变量包含URL 解码的换行符。

2、SCRIPT_NAME

有可能发生 XSS
像下面这样的配置
Default
location ~ \.php$ {
                include fastcgi_params;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_pass 127.0.0.1:9000;
        }

主要问题是 Nginx 会将所有 URL 发送到以.php结尾的 PHP 解释器，即使该文件在磁盘上不存在。这是 Nginx 创建的“陷阱和常见错误”文档中提到的，在许多 Nginx 配置中都常见的错误。如果这个 PHP 脚本试图基于SCRIPT_NAME定义一个基本 URL，则将发生 XSS。
Default
<?php
 
if(basename($_SERVER['SCRIPT_NAME']) ==
basename($_SERVER['SCRIPT_FILENAME']))
   echo dirname($_SERVER['SCRIPT_NAME']);
 
?>
 
GET /index.php/<script>alert(1)</script>/index.php
SCRIPT_NAME  =  /index.php/<script>alert(1)</script>/index.php

3、Any 变量

用户可以在其中打印 Nginx 变量的值
在某些情况下，用户提供的数据可以视为 Nginx 变量。目前尚不清楚为什么会发生这种情况，但如这份H1报告所示，这种情况并不罕见或不容易测试。如果搜索错误消息，我们可以看到它是在SSI过滤器模块中找到的，表明这是由 SSI 引起的。
一种测试方法是设置一个引用标头值：
Default
$ curl -H ‘Referer: bar’ http://localhost/foo$http_referer | grep ‘foobar’

原始后端响应读取

隐藏内部错误消息和标头以便 Nginx 处理

使用 Nginx 的proxy_pass，可以拦截后端创建的错误和 HTTP 标头。如果你这个方法会非常有用。如果后端回答一个错误，Nginx 将自动提供一个自定义错误页面。但如果 Nginx 无法理解这是一个 HTTP 响应怎么办？
如果一个客户端向 Nginx 发送了一个无效的 HTTP 请求，则该请求将按原样转发到后端，后端将使用其原始内容来应答。然后，Nginx 将无法理解无效的 HTTP 响应，而将其转发给客户端。想象一个这样的 uWSGI 应用程序：
Default
def application(environ, start_response):
   start_response('500 Error', [('Content-Type',
'text/html'),('Secret-Header','secret-info')])
   return [b"Secret info, should not be visible!"]

并在 Nginx 中使用以下指令：
Default
http {
   error_page 500 /html/error.html;
   proxy_intercept_errors on;
   proxy_hide_header Secret-Header;
}

如果后端的响应状态大于 300，proxy_intercept_errors将提供一个自定义响应。在上面的 uWSGI 应用程序中，我们将发送一个500 Error，Nginx 将拦截该错误。
proxy_hide_header可以自解释；它将从客户端隐藏任何指定的 HTTP 标头。
如果我们发送一个普通的 GET 请求，则 Nginx 将返回：
Default
HTTP/1.1 500 Internal Server Error
Server: nginx/1.10.3
Content-Type: text/html
Content-Length: 34
Connection: close

但是，如果我们发送一个无效的 HTTP 请求
Default
GET /? XTTP/1.1
Host: 127.0.0.1
Connection: close

我们将收到以下答复：
Default
XTTP/1.1 500 Error
Content-Type: text/html
Secret-Header: secret-info
Secret info, should not be visible!

merge_slashes 设置为 off

此配置项表示是否合并相邻的“/”，例如，//test///a.txt，在配置为on时，会将其匹配为location /test/a.txt；如果配置为off，则不会匹配，URI将仍然是//test///a.txt。
如果 Nginx 用作反向代理，并且被代理的应用程序容易受到本地文件包含内容的影响，则在请求中使用额外的斜杠可能会留出恶意利用空间。

空白符的妙用（nginx + gunicorn）

前提：
nginx < 1.21.1

nginx会将传入的路径进行翻译，如传入
/aaa/bbb/../../ccc
nginx实际看到的是访问
/ccc

这个案例的Poc为
/private HTTP/1.1/../../public
根据上面所述，nginx看到的是在访问
/public

然而gunicorn解析path时会解析成
/private HTTP/1.1
从而访问到/private目录

截断导致前后端解析不一致（Nginx + Weblogic）
环境中的nginx配置
Default
location /console/ {
  deny all;
  return 403;
}
 
location / {
  proxy_pass http://backend:7001;
}

可以看到，禁止访问 /console，访问 / 会转发到后端的weblogic服务器

nginx会将传入的path进行解析，所以传入的poc是
Default
/console/login/LoginForm.jsp;/../../../

nginx解析后会认为传入的是
/

所以绕过了对 /console的封禁，并转发请求到后端weblogic服务器;weblogic服务器有一个特性是遇到 ; 会进行截断操作，所以实际解析的请求是
Default
/console/login/LoginForm.jsp

weblogic中 # 的妙用（nginx + weblogic）
环境中的nginx配置
Default
location /console/ {
  deny all;
  return 403;
}
 
location / {
  proxy_pass http://backend:7001;
}

禁止访问 /console，访问 / 会转发到后端的weblogic服务器

Weblogic把#作为有效成分，所以可以构造
Default
/#/../console/

Nginx处理请求时，它无视了#后面的所有东西，这样可以绕过访问/console/的限制，并转发原始的/#/../console/给Weblogic。Weblogic根据规范处理这个路径，得到的解析结果是
Default
/console/

所以进入了/console/目录