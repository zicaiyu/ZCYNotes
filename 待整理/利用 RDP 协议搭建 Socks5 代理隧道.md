如今，在很多组织机构内部，针对 DMZ 或隔离网络区域内的计算机设备，为了限制其它接入端口风险，通常只允许这些设备开启 3389 端口，使用远程桌面来进行管理维护。那么我们能不能利用这个 3389 端口的 RDP 服务建立起一条通向内网的代理隧道呢？当然可以，下面就引出我们今天的主角 —— SocksOverRDP。

SocksOverRDP
Default
项目地址：https://github.com/nccgroup/SocksOverRDP

SocksOverRDP 可以将 SOCKS 代理的功能添加到远程桌面服务，它使用动态虚拟通道，使我们能够通过开放的 RDP 连接进行通信，而无需在防火墙上打开新的套接字、连接或端口。此工具在 RDP 协议的基础上实现了 SOCKS 代理功能，就像 SSH 的 -D 参数一样，在建立远程连接后，即可利用 RDP 协议实现代理功能。

该工具可以分为两个部分：SocksOverRDP-Plugin.dll和SocksOverRDP-Server.exe
第一部分是一个 .dll 文件，需要在 RDP 连接的客户端上进行注册，并在每次运行时将其加载到远程桌面客户端 mstsc 的上下文运行环境中。
第二部分是一个 .exe 可执行文件，它是服务端组件，需要上传到 RDP 连接的服务器并执行。

该工具的运作原理

当 SocksOverRDP-Plugin.dll 在 RDP 客户端上被正确注册后，每次启动远程桌面时都会由 mstsc 加载。接着，当 SocksOverRDP-Server.exe 被上传到 RDP 服务端上传并执行后 ，SocksOverRDP-Server.exe 会在动态虚拟通道上回连 SocksOverRDP-Plugin.dll，这是远程桌面协议的一个功能。虚拟通道设置完成后，SOCKS 代理将在 RDP 客户端计算机上启动，默认为 127.0.0.1:1080。此服务可用作任何浏览器或工具的 SOCKS5 代理。并且服务器上的程序不需要服务器端的任何特殊特权，还允许低特权用户打开虚拟通道并通过连接进行代理。

攻击端

在攻击机上需要安装注册 SocksOverRDP-Plugin.dll。首先我们将 SocksOverRDP-Plugin.dll 放置到攻击机的任何目录中，但是为了方便我们可以将其放置到 ％SYSROOT％\system32\ 或 ％SYSROOT％\SysWoW64\ 目录下。

然后使用以下命令对 SocksOverRDP-Plugin.dll 进行安装注册：
Default
regsvr32.exe SocksOverRDP-Plugin.dll    # 注册
 
# regsvr32.exe /u SocksOverRDP-Plugin.dll    取消注册

注册成功。但是由于 SocksOverRDP 建立的 SOCKS5 代理是默认监听在 127.0.0.1:1080 上的，所以只能从攻击机本地使用，为了让攻击者的 Kali 也能使用搭建在攻击机 Windows 10 上的 SOCKS5 代理，我们需要修改其注册表，将 IP 从 127.0.0.1 改为 0.0.0.0。注册表的位置为：KEY_CURRENT_USER\SOFTWARE\Microsoft\Terminal Server Client\Default\AddIns\SocksOverRDP-Plugin

然后启动远程桌面客户端 mstsc.exe 连接目标 Web 服务器
弹出了一个提示说 SocksOverRDP 成功启动，当服务端的可执行文件运行后即可在攻击机的 1080 端口上启动 SOCKS5 代理服务。

服务端

远程桌面连接成功后，将服务端组件 SocksOverRDP-Server.exe 上传到目标 Web 服务器
直接运行 SocksOverRDP-Server.exe 即可
此时便成功搭建了一个 SOCKS5 代理隧道，查看攻击机 的端口连接状态发现已经建立连接：
Default
netstat -ano | findstr "1080"

然后在攻击机 Kali 上配置好 proxychains：
Default
socket5 本地ip 1080

此时便可以通过代理访问到内网的主机
成功打开了 DC 的远程桌面：
Default
proxychains4 rdesktop 192.168.93.30

探测内网主机的端口开放情况
Default
proxychains4 nmap -sT -Pn 192.168.93.30 -p 445

该主机开启了 445 端口，我们可以直接用 smbexec.py 连接：
Default
proxychains4 python3 smbexec.py whoamianony/administrator:Whoami2021@192.168.93.30
其他方式利用参考
Default
https://mp.weixin.qq.com/s?__biz=MzU4NzU4MDg0Mw==&mid=2247486684&idx=1&sn=1ea7b050e0b10c1a75f2f8e5157e41ff&scene=21#wechat_redirect