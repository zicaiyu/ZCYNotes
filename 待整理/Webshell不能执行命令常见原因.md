这篇文章搜集整理的不能执行命令的常见原因和一些解决方法

出现无法执行命令的原因也有很多，如常见的：
Default
PHP安全模式(disable_functions)；
cmd.exe被降权或删除；
命令执行组件被卸载；
组策略禁止执行cmd.exe；
安全狗、云锁、360等安全防护软件；
...SNIP...
执行命令组件、函数、类和方法
Default
Asp：
Wscript.shell、Shell.Application
 
Aspx：
ProcessStartInfo、Wscript.shell，Shell.Application
 
Php：
system、passthru、shell_exec、exec、popen、proc_open
 
Jsp：
Runtime.getRuntime().exec(command)
注册与卸载执行命令的高危组件
Default
可以通过执行以下命令或删除对应注册表项来注册和卸载WScript.Shell、Shell.Application命令组件。
 
WScript.Shell组件：
regsvr32 /u %windir%\system32\wshom.ocx
HKEY_CLASSES_ROOT\WScript.Shell
HKEY_CLASSES_ROOT\WScript.Shell.1
 
Shell.Application组件：
regsvr32 /u %windir%\system32\shell32.dll
HKEY_CLASSES_ROOT\Shell.Application
HKEY_CLASSES_ROOT\Shell.Application.1

Webshell常见命令执行报错小结
Default
序号	问题描述						原因和解决
1	[Err]							可能是D盾，不太好绕
2	ret=-1							常出现在PHP环境中
3	系统无法执行指定的程序					Windows Defender将EXP隔离了
4	DNS服务器对区域没有权威					换可读/写目录：C:\ProgramData\
5	ERROR://系统找不到指定的文件				上传cmd.exe或换ASPX、PHP脚本
6	拒绝访问。						执行EXP时如果出现两个“拒绝访
	拒绝访问。						问”，将EXP文件路径加上双引号
7	{Err]拒绝访问.[中文]					上传cmd.exe或换成ASPX、PHP脚本，可
	[Err] Access is denied.[英文]				能会有权限执行
8	[Err]没有权限[中文]					没权限运行cme.exe，ASPX、PHP脚本
	[Err] Permission denied[英文] 				可能会有权限执行
9	[Err] ActiveX不见不能创建对象[中文] 			WScript.Shell被卸载，换ASPX、PHP
	[Err] ActiveX component can not create object [英文] 	或Shell.Application、msf_stdapi
10	命令执行失败，可能是启用了安全模式！			Hatchet、0620菜刀或多函数脚本
								Bypass disable_function
11	fatal error JS2008：保存百编译状态时出错：未找到	网站安全狗禁止IIS执行程序防护引起，
	cvtres.exe						可利用安全狗白名单文件绕过
12	命令提示符已被系统管理员停用。				本地组策略中的“阻止访问命令提示符
								(脚本处理)”，可利用setp绕过
13	ERROR://配额不足，无法处理命令。			执行MS13-051后会出现这报错，稍等
	ERROR://系统资源不足，无法完成请求的服务。		一会再执行即可
14	指定的可执行文件不是此操作系统平台的有效应用程序。	上传的EXP文件不完整，用菜刀的“下
	The specified executable is not availd application for	载文件到服务器”绕过。或可能时EXP
	this OS platform.					位数不对所造成
15	组策略阻止了这个程序。要获取详细信息，请与系统管理	组策略禁止执行cmd.exe,这种情况不
	员联系。						常见，换其它可读/写目录即可
16	ERROR://由于一个软件限制策略的阻止，Windows无法		组策略设置方法：secpol.msc->软件限
	打开此程序。要获取更多信息，请打开时间查看器或与系	制策略->其他规则->新建哈希规则->
	统管理员联系。						选择cmd.exe文件
	[Err] Windows connot open this program because it
	has been prevented by a software restriction policy.	绕过方法：换其他的cmd.exe文件即可
	For more information,open Even Viewer or contact
	your system administrator.
17	ERROR://%1不是有效的Win32应用程序。			1.我们上传的cmd.exe文件与目标系统
	ERROR://映像文件%1有效，但不适用于此计算机类型。	版本、位数不对应。上传与目标系统对
	ERROR://指定的可执行文件不是有效Win32应用程序。		应的cmd.exe即可解决
	ERROR://该版本的%1与您运行的Windows版本不兼
	容。请查看计算机的系统信息，了解是否需要x86(32位)	2.服务器安装了云锁防护软件，可使用
	或x64(64位)版本的程序，然后联系软件发行者。		rar.exe解压缩来绕过
	系统无法再消息文件为Application找到消息号为
	0x2331的消息文本。

Windows各版本操作系统cmd路径

大部分管理员只会给默认System32、SysWOW64目录下的cmd.exe文件做降权处理，这时我们就可以尝试使用以下对应操作系统版本的cmd.exe来执行系统命令
Win2k8：
Default
C:\Windows\winsxs\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_6.1.7601.17514_none_e932cc2c30fc13b0\cmd.exe
C:\Windows\winsxs\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_6.1.7601.17514_none_f387767e655cd5ab\cmd.exe
Win2k12：
Default
C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_6.3.9600.16384_none_7bcb26c7ee538fe3\cmd.exe
C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_6.3.9600.16384_none_861fd11a22b451de\cmd.exe
Win2k16：
Default
C:\Windows\WinSxS\amd64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.14393.0_none_b8813238310f2dd6\cmd.exe
C:\Windows\WinSxS\wow64_microsoft-windows-commandprompt_31bf3856ad364e35_10.0.14393.0_none_c2d5dc8a656fefd1\cmd.exe
