Windows Server 2008 中的组策略首选项，它允许系统管理员设置特定配置。可在机器上创建用户名和密码。但是通过这个功能，可能会因为首选项中的密码泄漏而威胁到整个域的安全。

什么是 GPP（组策略首选项）

组策略首选项简称 GPP，它允许管理员配置和安装以前无法使用组策略的 Windows 和应用程序设置。组策略首选项 (GPP) 最有用的功能之一是能够存储，此外，这些策略可以对机器进行各种配置更改，例如：

    映射驱动器
    创建本地用户
    数据源
    打印机配置
    注册表设置
    创建/更新服务
    计划任务
    更改本地管理员密码

配置 GPP（组策略）

1、首先在命令提示符输入：gpmc.msc
2、选择 saulgoodman.cn –>右键组策略对象–>新建，这里新建一个 GPPVuln 的组策略对象：
3、之后选中新建的组策略对象，右键点击编辑：
4、点击编辑后来到这个组策略管理编辑器：右键本地用户和组–>新建–>本地用户
5、我们将域中每个计算机的本地 Administrator 管理员用户更名为 admin，并且设置新的密码 Admin12345:
6、回到新建的 GPPVuln，点击添加，将 Domain Computers 添加到组策略应用的组中：
7、最后运行命令 gpupdate，强制更新组策略设置：
8、最后我们查看 GPPVuln 的详细信息，并且在 C:\Windows\SYSVOL\domain\Policies 目录下有 ID 相对应的文件夹目录即可查看到文件：
9、其实 C:\Windows\SYSVOL\domain\Policies\{F776977C-E982-4662-8970-B528C214E5B7}\Machine\Preferences\Groups\Groups.xml 文件里就是我们刚刚设置的本地用户 admin 的用户名和加密的密码：

GPP 漏洞利用

我们知道由于密码存储在 SYSVOL 中的首选项目中。SYSVOL 是所有经过身份验证的用户访问的 Active Directory 中的域扩展共享文件夹，也就是说只要你是域用户，你就可以访问这个首选项共享文件夹。
所有域组策略都存储在这里：\\<DOMAIN>\SYSVOL\<DOMAIN>\Policies\
当为用户或组帐户创建新的 GPP 时，它将与在 SYSVOL 中创建的 Group.XML 文件相关联，其中包含相关的配置信息，并且密码是 AES-256 位加密的。因此，密码并不安全，且微软已经公布了密钥，我们只要获取到 cpassword 就能获取到明文。

实验环境：
域控：AD-2008（10.10.0.8），saulgoodman\administrator:Admin12345
域机器：work-2008（10.10.0.9），saulgoodman\saul:S!@#456

1、首先域机器 work-2008 可以直接 dir 查看域控的 sysvol 共享目录：
Default
dir \\10.10.0.8\sysvol

2、我们可以直接一步步的来到 \\10.10.0.8\sysvol\saulgoodman.cn\Policies\{F776977C-E982-4662-8970-B528C214E5B7}\Machine\Preferences\Groups\目录下找到 Groups.xml 文件：

3、当然我们还可以通过 for 循环来搜索 xml 文件，因为手动去一个个目录翻很浪费时间！
Default
 for /r \\AD-2008/sysvol %i in (*.xml) do @echo %i

4、通过得到 Groups.xml 文件后，我们得到了以下信息：
Default
 newName="admin" 
 cpassword="A48HwlVXS/3M2Asazld/d7Fvvt42DD7pOJGn/ut+z7I"

5、通过拿到 cpassword 的密码，我们可以拿到 Kali 里使用自带的工具 gpp-decryp 去解密：
Default
gpp-decrypt "A48HwlVXS/3M2Asazld/d7Fvvt42DD7pOJGn/ut+z7I"

至此得到了密码 Admin12345，但是我们用得到的本地管理员用户 admin，密码 Admin12345 是登陆不了目标机器的：因为目标域控根本没有这个用户：

其实通过 GPP 这个漏洞，我们只能通过在域内进行信息搜集凭证，我们可以拿到这个密码去横向移动，看看其他机器的密码是不是这个，实际内网渗透中还是有很大的几率能够横向起来。
并不是说你拿到这个用户和密码就能够直接 wmi 或者 psexec，也需要一些运气成分在里面，若目标域管密码和这个 GPP 解密的密码是一样的，那么就可以直接 WMI 拿到域控了！