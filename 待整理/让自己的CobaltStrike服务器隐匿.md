服务器禁ping

当服务器禁ping后，从某种角度可以判定为主机为不存活状态

临时禁ping

将/proc/sys/net/ipv4/icmp_echo_ignore_all文件里面的0临时改为1，从而实现禁止ICMP报文的所有请求，达到禁止Ping的效果，网络中的其他主机Ping该主机时会显示“请求超时”，但该服务器此时是可以Ping其他主机的。
Default
#禁ping
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
#启用ping
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all

永久禁ping
Default
#编辑配置
vim /etc/sysctl.conf
#设置禁ping（如果有此配置就无需重复添加，仅更新值即可）
net.ipv4.icmp_echo_ignore_all = 1
#刷新配置
sysctl -p
 
#启用ping
net.ipv4.icmp_echo_ignore_all = 0

IPTABLES防火墙禁ping

注：使用以下方法的前提是内核配置是默认值，也就是内核没有禁ping
Default
设置禁ping
iptables -A INPUT -p icmp --icmp-type 8 -s 0/0 -j DROP
启用ping
iptables -D INPUT -p icmp --icmp-type 8 -s 0/0 -j ACCEPT
 
#参数备注
-A：添加防火墙规则.
INPUT：入站规则.
-p icmp：指定包检查的协议为ICMP协议.
--icmp-type 8：指定ICMP类型为8.
-s：指定IP和掩码，“0/0”表示此规则针对所有IP和掩码.
-j：指定目标规则，即包匹配则应到做什么，"DROP"表示丢弃，"ACCEPT"表示接受

ps：通过修改配置方式禁止ping后，内部ping也将失效，通过防火墙方式禁止ping后，可以ping自己内部网络。

修改端口

CobaltStrike的连接端口默认为50050，这是个很明显的特征，可以通过编辑cobalt strike目录下的teamserver文件，搜索50050，将其改为任意端口即可，这里改成10086：
再次启动后可以看到端口已经发生变化：

流量特征修改

Domain+CDN

需要用到一个域名，最好选择未备案域名，以防溯源。不建议freenom的免费域名，可以到godaddy或者hostinger注册便宜的域名，可以选择一些与项目相关或者高可信度的域名。
关于CDN不建议使用国内的，虽然便宜，但是不支持非80端口的代理，且有信息泄露的风险。建议使用cloudflare的免费CDN，够用而且支持其他端口加速。注意在域名提供商处修改DNS解析需要一段时间生效。
dns配置：在Cloudflare添加A类型记录，自定义二级域名指向你的服务器真实IP
缓存 –> 配置选项中需要以下两项为开启状态
此处开发模式会在三小时后自动关闭，可以在规则中添加网站缓存级别为绕过,这点很重要！！！

证书配置

首先修改 SSL/TLS 加密模式为完全，在源服务器中创建证书
私钥类型选择ECC，并保存.pem和.key文件并上传到服务器,先删除cobalstrike默认的cobalstrike.store并使用命令重新生成
Default
openssl pkcs12 -export -in server.pem -inkey server.key -out xxx.xxx.com.p12 -name xxx.xxx. -passout pass:123456
 
#将证书打包并生成store文件
 
keytool -importkeystore -deststorepass 123456 -destkeypass 123456 -destkeystore cobaltstrike.store -srckeystore xxx.xxx.com.p12 -srcstoretype PKCS12 -srcstorepass 123456 -alias xxx.xxx.com
 
curl https://127.0.0.1:10086 -v -k

C2.profile配置

可使用C2concealer项目动态生成
Default
git clone https://github.com/FortyNorthSecurity/C2concealer

或者修改下面c2.profile文件，其中client.header中Host的值要为我们申请的域名，其他的部分，根据个人情况去配置。
Default
https-certificate {
    set keystore "cobaltstrike.store";
    set password "123456";
}
http-stager {
    set uri_x86 "/api/1";
    set uri_x64 "/api/2";
    client {
        header "Host" "xxx.xxx.com";}
    server {
        output{
        print;
        }
    }
        }
http-get {
    set uri "/api/3";
    client {
        header "Host" "xxx.xxx.com";
        metadata {
            base64;
            header "Cookie";
        }
        }
    server {
        output{
        print;
        }
    }
        }
http-post {
    set uri "/api/4";
    client {
        header "Host" "xxx.xxx.com";
        id {
            uri-append;
        }
        output{
        print;
        }
    }
    server {
        output{
        print;
        }
    }
}

可使用命令检查c2.profile配置文件是否正确
Default
./c2lint c2.profile

最后可利用命令启动cobaltstrike
Default
./teamserver ip passwd ./c2.profile

通过curl https://127.0.0.1:443 -v -k
可以看到https上线的监听端口证书信息已经更改：

监听器

正常配置就好，只是注意cloudflare免费版本只支持解析少量的端口，具体端口如下
Default
http:
80、8080、8880、2052、2082、2086、2095
https:
443、2053、2083、2087、2096、8443

流量检测

正常上线，成功隐藏真实IP，公网IP都是CDN的IP地址

除了以上措施之外，还可以配置服务器防火墙，c2上线端口只能让CDN的IP段访问，这样可以避免被扫描器扫描到，将你的IP贴上威胁情报标签。实战过程中想要更好的隐藏还需要做好木马免杀及签名等措施。对于整个过程的可移植性，只需要打包CobaltStrike文件夹部署在不同的服务器，并更新DNS解析即可。