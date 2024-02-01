# 腾讯云轻量应用服务器配置IPSec-IKEv2 VPN

这个方法废了，你好不容易搭建好，直接给你把UDP端口给你封了；不过，还是将我搭建的过程分享一下

这个搭建的VPN是适用于IOS、Windows；Android适不适用还没试过

## 服务端配置

### 安装StrongSwan
    yum install strongswan
    systemctl enable strongswan
    systemctl start strongswan

### 生成证书

#### 生成 CA 根证书

生成一个CA私钥

    strongswan pki --gen --outform pem > ca.key.pem
    
基于这个私钥自己签一个CA根证书

    strongswan pki --self --in ca.key.pem --dn "C=CN, O=ITnmg, CN=服务器的URL或IP" --ca --lifetime 3650 --outform pem > ca.cert.pem
    
> –self 表示自签证书
>
> –in 是输入的私钥
>
> –dn 是判别名
>
>> * C 表示国家名，同样还有 ST 州/省名，L 地区名，STREET（全大写） 街道名
>> * O 组织名称
>> * CN 友好显示的通用名
>
> –ca 表示生成 CA 根证书
>
> –lifetime 为有效期, 单位是天

#### 生成服务器端证书

生成一个服务端私钥

    strongswan pki --gen --outform pem > server.key.pem
    
基于这个私钥生成公钥

    strongswan pki --pub --in server.key.pem --outform pem > server.pub.pem
    
用前面自签的CA根证书+CA私钥+服务器公钥来生成服务器证书

    strongswan pki --issue --lifetime 3600 --cacert ca.cert.pem --cakey ca.key.pem --in server.pub.pem --dn "C=CN, O=ITnmg, CN=服务器的URL或IP" --san="服务器的URL或IP" --flag serverAuth --flag ikeIntermediate --outform pem > server.cert.pem
    
> –issue, –cacert 和 –cakey 就是表明要用刚才自签的 CA 证书来签这个服务器证书。
>
> –dn, –san，–flag 是一些客户端方面的特殊要求：
>
> * iOS 客户端要求 CN 也就是通用名必须是你的服务器的 URL 或 IP 地址;
> * Windows 7 不但要求了上面，还要求必须显式说明这个服务器证书的用途（用于与服务器进行认证），–flag serverAuth;
> * 非 iOS 的 Mac OS X 要求了“IP 安全网络密钥互换居间（IP Security IKE Intermediate）”这种增强型密钥用法（EKU），–flag ikdeIntermediate;
> * Android 和 iOS 都要求服务器别名（serverAltName）就是服务器的 URL 或 IP 地址，–san。

### 安装证书

    cp -r ca.key.pem /etc/strongswan/ipsec.d/private/
    cp -r ca.cert.pem /etc/strongswan/ipsec.d/cacerts/
    cp -r server.cert.pem /etc/strongswan/ipsec.d/certs/
    cp -r server.pub.pem /etc/strongswan/ipsec.d/certs/
    cp -r server.key.pem /etc/strongswan/ipsec.d/private/
    
### 配置VPN

先修改主配置

    nano /etc/strongswan/ipsec.conf
    
添加如下配置

    config setup
        uniqueids=no
     
    conn %default
        compress = yes
        esp = aes256-sha256,aes256-sha1,3des-sha1!
        ike = aes256-sha256-modp2048,aes256-sha1-modp2048,aes128-sha1-modp2048,3des-sha1-modp2048,aes256-sha256-modp1024,aes256-sha1-modp1024,aes128-sha1-modp1024,3des-sha1-modp1024!
        keyexchange = ike
        keyingtries = 1
        leftdns = 8.8.8.8,8.8.4.4
        rightdns = 8.8.8.8,8.8.4.4
     
    conn ikev2-eap
        leftca = "C=CN, O=ITnmg, CN=服务器的URL或IP"
        leftcert = server.cert.pem
        leftsendcert = always
        rightsendcert = never
        leftid = 服务器的URL或IP
        left = %any
        right = %any
        leftauth = pubkey
        rightauth = eap-mschapv2
        leftfirewall = yes
        leftsubnet = 0.0.0.0/0
        rightsourceip = 10.1.0.0/16
        fragmentation = yes
        rekey = no
        eap_identity=%any
        auto = add
 
通过Ctrl+O建保存，然后退出，后面同理       
修改DNS配置

    nano /etc/strongswan/strongswan.d/charon.conf
    
添加如下配置

    charon {
        # 同时连接多个设备,把冗余检查关闭.
        redundancy {
            mode = none
        }
     
        # windows 公用 dns
        dns1 = 8.8.8.8
        dns2 = 8.8.4.4
    }
    
配置验证方式的用户名和密码

    #使用证书验证时的服务器端私钥
    #格式 : RSA <private key file> [ <passphrase> | %prompt ]
    : RSA server.key.pem
     
    #使用预设加密密钥, 越长越好
    #格式 [ <id selectors> ] : PSK <secret>
    %any : PSK "预设加密密钥"
     
    #EAP 方式, 格式同 psk 相同
    用户名 : EAP "密码"
     
    #XAUTH 方式, 只适用于 IKEv1
    #格式 [ <servername> ] <username> : XAUTH "<password>"
    用户名 : XAUTH "密码"
    
开启内核转发

    nano /etc/sysctl.conf
    
添加如下配置

    # VPN
    net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding=1
    
保存退出后，执行命令

    sysctl -p
 
 ### 配置防火墙
 
 查看所有网卡信息，找出公网网卡
 
    ip addr
    
编辑网卡配置

    nano /etc/sysconfig/network-scripts/ifcfg-eth0
    
在最尾处添加如下配置

    ZONE=public 
    
保存退出，重启服务

    systemctl restart network
    systemctl restart firewalld
    
然后新建一个服务

    nano /etc/firewalld/services/strongswan.xml

填入以下内容

    <?xml version="1.0" encoding="utf-8"?>
    <service>
      <short>Strongswan</short>
      <description>Strongswan VPN</description>
      <port protocol="udp" port="500"/>
      <port protocol="udp" port="4500"/>
    </service>

#### 保存退出，添加服务到当前区域，开启转发

> 以下命令没有指定 --zone 参数, 都是针对默认区域 public

为区域添加服务

    firewall-cmd --permanent --add-service=strongswan
    
启用ip伪装

    firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" masquerade'
    
添加nat转发

    firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" forward-port port="4500" protocol="udp" to-port="4500"'
    firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.1.0.0/16" forward-port port="500" protocol="udp" to-port="500"'

重新加载防火墙配置

    firewall-cmd --reload

配置完防火墙后，重启StrongSwan服务

    strongswan stop
    systemctl start strongswan
    
> 这里不使用 strongswan restart 命令的原因是, 使用这条命令后, 再用 systemctl status strongswan 命令得不到正确的运行状态.

查看StrongSwan的运行状态

     systemctl status strongswan -l
     
## 客户端配置

### IOS

将之前创建的 ca.cert.pem 用 ftp 导出 , 写邮件以附件的方式发到邮箱（或者在邮箱中保存到文件中转站）, 在 ios 浏览器登录邮箱, 下载附件, 安装 ca 证书。

#### 使用 IKEv2 + EAP 认证

找到手机上 “设置->VPN->添加配置”, 选 IKEv2

> * 描述: 随便填
> * 服务器: 填url或ip
> * 远程ID: ipsec.conf 中的 leftid
> * 本地ID: ipsec.conf 中的 leftid
> * 用户鉴定: 用户名
> * 用户名: EAP 项用户名
> * 密码: EAP 项密码

### Windows 11

#### 导入证书

> * 将 CA 根证书 ca.cert.pem 重命名为 ca.cert.crt
> * 双击 ca.cert.crt 开始安装证书
> * 点击安装证书
> * “存储位置” 选择 “本地计算机”, 下一步
> * 选择 “将所有的证书都放入下列存储区”, 点浏览, 选择 “受信任的根证书颁发机构”, 确定, 下一步, 完成.

#### 建立连接

> * “控制面板”-“网络和共享中心”-“设置新的连接或网络”-“连接到工作区”-“使用我的 Internet 连接”
> * Internet 地址写服务器 IP 或 URL。
> * 描述随便写。
> * 用户名密码写之前配置的 EAP 的那个
> * 确定
> * 转到 控制面板网络和 Internet网络连接
> * 在新建的 VPN 连接上右键属性然后切换到“安全”选项卡
> * VPN 类型选 IKEv2
> * 数据加密选“需要加密”
> * 身份认证这里需要说一下，如果想要使用 EAP 认证的话就选择“Microsoft:安全密码(EAP-MSCHAP v2)”; 想要使用私人证书认证的话就选择“使用计算机证书”。
> * 再切换到 “网络” 选项卡, 双击 “Internet 协议版本 4” 以打开属性窗口, 这里说一下, 如果你使用的是老版本的 win10, 可能会打不开属性窗口, 这是已知的 bug, 升级最新版本即可解决.
> * 点击 “高级” 按钮, 勾选 “在远程网络上使用默认网关”, 确定退出.

配置完能连上，但是发现，除了自己的服务器，其它的网站都连接不上，接下来进行排查，发现是能ping的通那些连接不上的网站的

最终排查下来发现是MTU 设置问题。MTU 指的是最大传输单元，它指定数据包的最大大小。如果您的 MTU 设置不正确，则可能会阻止您访问某些网站。

可以通过在命令提示符中使用以下命令来测试 MTU 值

    ping -f -l 1500 [目标网站的 IP 地址]
    
如果发现ping不通，减少100继续ping，我这里是到1300后才能ping通

查看自己的电脑设置的是多少

    netsh interface ipv4 show subinterfaces
    
发现我配置的VPN设置的是1400，所以接下来需要将这个值修改为1300

    netsh interface ipv4 set subinterface "adapter_name" mtu=MTU_size store=persistent
    
> adapter_name替换为实际的适配器名称，MTU_size替换为所需的MTU值
>
> 设置完MTU后需要管理员身份运行CMD，且设置完后需要重启电脑才能生效
