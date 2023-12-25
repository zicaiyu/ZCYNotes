# App渗透环境搭建

## PC端

抓包工具：[Yakit](https://www.yaklang.com/)

### 获取证书

>获取证书的原因
>
>抓包过程会有https协议，这类数据包需要使用到证书来进行许可

---
在Yakit中下载证书，格式是.pem的文件

下载[openssl](https://slproweb.com/products/Win32OpenSSL.html)

使用openssl来查看证书的MD5值

    openssl x509 -inform PEM -subject_hash_old -in yakit.crt.pem
    
然后重命名证书

    ren yakit.crt.pem MD5的值加上.0
    
 比如：
 MD5的值为10fb1fcc
    
    ren yakit.crt.pem 10fb1fcc.0
    
## 安卓端

使用夜神模拟器，安卓版本为12；设置分辨率为手机版，手机类型选择Redmi Note 9

### 安装证书

#### 传输证书

在夜神模拟器右侧按钮，打开电脑文件夹，会跳转到指定的目录，这是共享文件夹，将证书复制到ImageShare目录下

#### 移动证书

下载[MT管理器](https://coolapk.com/apk/bin.mt.plus)，然后通过夜神模拟器右侧安装该APP

下载好后打开，左侧进入到Pictures目录下，证书在这里

右侧进入到/system/etc/security/cacerts目录

长按左侧的证书，将证书复制到右边

点击左上角，打开终端模拟器，将证书文件设置为可读

    cd /storage/emulated/0/Pictures
    su    # 管理员权限
    mount -o remount,rw /system  # 这个命令的意思是重新挂载 /system 文件系统，并将其以读写模式重新挂载。
    cd /system/etc/security/cacerts
    chmod 777 10fb1fcc.0

重启下模拟器
  
### 设置网络

打开设置，进入连接WIFI的地方，断开连接，然后设置代理，和yakit上的代理一致（注意，不要使用localhost、127.0.0.1）

### 绕过APP检测

有时抓包返回包显示网络等问题，报错；可能是对反代理或者反证书进行检测，导致无法成功抓包。

#### 绕过反代理检测

工具：Proxifier

在工具栏找到`配置文件`>`代理服务器`>`添加`>`与yakit相同的代理ip地址`>`选择https`

在工具栏找到`配置文件`>`代理规则`>`添加`>`名称随便填`>`选择应用程序noxvmhandle.exe;nox.exe`>`动作选择前面配置的代理服务器`

> 选择应用程序，可以通过任务管理器，找到对应的程序，打开文件所在位置，复制所在位置来添加
>
> 在完全进入到模拟器后在使用Proxifier，不然可能会进入不到模拟器
>
> 使用了Proxifier就不需要设置网络的代理了

#### 绕过反证书

> [Python下载镜像地址](https://mirrors.huaweicloud.com/python/)
>
>[r0capture下载地址](https://github.com/r0ysue/r0capture)

使用PyCharm打开r0capture，安装爆红的地方所需要的库

> 我用了r0capture.py,软件就会闪退

安装frida

    pip install frida
    
安装frida-tools

    pip install frida-tools

查看夜神模拟器运行的地址和端口

    adb devices 
    
连接模拟器    

    adb connect 127.0.0.1:62001
    
查找模拟器模拟器的CPU类型

    adb shell getprop ro.product.cpu.abi
    
查看frida版本

    frida --version
    
根据CPU类型和frida版本下载对应的frida server

    https://github.com/frida/frida/releases
    
    比如：
    frida-server-16.1.9-android-x86_64.xz 

将压缩包解压缩为文件，然后上传文件到指定目录下

    adb push frida-server-16.1.9-android-x86_64 /data/local/tmp
    
进入手机端命令

    adb shell
    
切换root权限

    su
    
进入到上传文件的目录下

    cd /data/local/tmp

看是否成功上传

    ls -l
    
将该文件权限修改为777

    chmod 777 frida-server-16.1.9-android-x86_64
    
在模拟器上运行该文件

    ./frida-server-16.1.9-android-x86_64 &
    
电脑检查手机端服务是否开启成功

    frida-ps -U
    
Windows运行，将端口转发到PC

    adb forward tcp:27042 tcp:27042
    adb forward tcp:27043 tcp:27043
    
打开模拟器上要抓包的软件，并且打开MT管理器，点击左上角菜单，选择安装包提取，获取需要抓包的APP的包名

在r0capture.py当前目录下进入终端，输入命令

Spawn模式，直接抓包

显示的txt文件杂乱无章，所以官方推荐Attach模式

    python r0capture.py -U -f 包名
    
Attach模式，将抓包内容保存成.pcap格式文件
    
    python r0capture.py -U 包名 -p 文件名.pcap
