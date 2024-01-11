APP加固主要是对APP中的dex文件、so文件、资源文件等进行保护

简单的APP加固原理的流程

    加固的代码先运行，进行初始化工作
    加固的代码开始解密被保护的核心代码
    加固的代码开始加载解密后的核心代码
    加固的代码把控制权转交给核心代码

第一代加固和脱壳

第一代加固技术主要做了进行对app的dex文件进行保护和做了一些简单的反调试保护

Dex保护：Dex文件整体加密、字符串加密、自定义DexClassLoader。
动态防护：ptrace反调试、TracePid值校验反调试。

第一代加固的出现也同时出现各种对加固技术的攻破。从而出现了各种对抗加固的脱壳方法。
最突出的是直接从内存dump出dex完整结构脱壳原理：
程序在启动过程中，要保证程序正常运行，那么加固壳会自动解密受保护的dex文件并完成加载，基于这个子解密的原理，我可以选择在dex加载完成这个时机点，将其dump下来。从而实现第一层防护壳、加密壳的脱壳。

下面罗列几个脱壳方案

缓存脱壳法:
第一代的某些加固产品，安装包是加密压缩的，安装后回在data/dalvik-cache目录下生成解密的odex文件，这时候只需要获取odex文件进行做为分析的突破点。

内存 dump脱壳法
通过工具：IDA Pro + dumpDEX
1、通过/proc/%d/maps获取内存映射
2、在内存中查找关键字 dex.035或dex.036
3、手动dump查找到的数据。

动态调试脱壳法
1、通过基于IDA的android_server的代理方式进行附加app。
2、再IDA中下dvmDexFileOpenPartial 断点，确认要dump的起始地址和大小。
3、用ida的脚本方式进行dump出原始数据。

HOOK脱壳法
Hook脱壳法一般都是基于frida和xposed这两个框架进行做hook操作的。
这种hook脱壳法：先需要进行分析app应用，找到可以进行hook的函数，然后在选择用的顺手的、适用的frida或xposed框架进行 hook。
（xposed是java编译，适用于java层hook；frida适用于java层和native层hook）。

通过对关键函数dvmDexFileOpenPartial进行hook实现脱壳。
也可以通过xposed框架hook ClassLoader的loadClass函数实现脱壳。

定制系统脱壳法
主要是通过修改系统源码中的关键函数，接着将修改后的源码重新编译并进行刷机。
例如通过修改系统dvmdexfileopenpartial函数的关键逻辑。在函数里面修改写入我们想要实现的功能。

第二代加固和脱壳

由于第一代加固是整体性加固的，因此只要dump到关键点后，就可以完整的获取到dex整个内容。由于第一代壳的缺点，随之而来的就是进行对APP中关键类的抽取技术。那么第二代加固主要进行如下的功能点。

    DEX保护: DEX类抽取、DEX动态加载、SO动态加载
    So保护:SO加密
    动态防护: 反调试，防HOOK
    资源文件保护:本地数据库保护、本地文件保护

脱壳技术方案

脱壳原理：
主动调用类中的每一个方法，并实现函数指令的还原常见的加固厂商的指令抽取的实现方式主要又两大类，一种是指令代码在dex文件原便宜位置处还原，另外一种就是随机分配，通过修订偏移的方式，使程序在执行过程中通过修定的便宜找到函数指令；android程序在执行过程，使用到类时候，都需要进行加载，而在加载过程中，函数指令就会进行指令还原或修订函数指令指向的位置，我们可以利用这个时机，将代码拷贝到原代码位置，进而实现类抽取壳的脱壳。

内存重组脱壳法

通过内存中dex文件的格式，找到完整的dex文件，将其组合到一起。

内存dump脱壳法

通过工具：IDA Pro + dumpDEX
1、通过/proc/%d/maps获取内存映射
2、在内存中查找关键字 dex.035或dex.036
3、手动dump查找到的数据。
也可以直接用frida中的dump脚本进行内存dump脱壳。

动态调试脱壳法

1、通过基于IDA的android_server的代理方式进行附加app。
2、再IDA中下dvmDexFileOpenPartial 断点，确认要dump的起始地址和大小。
3、用ida的脚本方式进行dump出原始数据。

HOOK脱壳法

Hook脱壳法一般都是基于frida和xposed这两个框架进行做hook操作的。
这种hook脱壳法：先需要进行分析app应用，找到可以进行hook的函数，
然后在选择用的顺手的、适用的frida或xposed框架进行 对关键函数hook。
（xposed是java编译，适用于java层hook；frida适用于java层和native层hook）。

通过对关键函数memcmp、dexFileParse进行hook实现脱壳。
也可以通过hook ClassLoader的loadClass函数实现脱壳。

定制系统脱壳法

主要是通过修改系统源码中的关键函数，接着将修改后的源码重新编译并进行刷机。
例如通过修改系统memcmp、dexFileParse函数的关键逻辑。在函数里面修改写入我们想要实现的功能。

第三代加固和脱壳

第三代加固方案主要进行对函数进行做抽取，并进行做动态加解密方式，下面罗列关键的加固技术点

    DEX保护: DEX Method代码抽取、Dex Method动态解密
    SO保护:SO加壳
    动态防护：防内存dump、防系统核心库HOOK
    资源文件保护：H5文件保护

APP函数抽取指令流程：

1、解析原始dex文件格式，保存所有方法的代码结构体信息。
2、通过传入需要置空指令的方法和类名，检索到其代码结构体信息。
3、通过方法的代码结构体信息获取指令个数和偏移地址，构造空指令集，然后覆盖原始指令。
4、重新计算dex文件的checksum和signature信息，回写到头部信息中。

脱壳方法：

HOOK脱壳法

通过利用frida框架进行hook关键函数DexFile，OpenFile、dexFindClass等关键函数实现脱壳。

定制系统脱壳法

通过修改系统源码中的关键函数如DexFile、OpenFile、dexFindClass函数的关键逻辑，然后进行重编系统。
在ART中通过修改定制dex2oat法进行脱壳。

第四代加固和脱壳

现在市面上强度最强加固方案实属代码虚拟化保护的方案。通过将程序的代码编译为虚拟机指令也就是虚拟机代码（也就是自定义的代码集），通过虚拟机cpu解释并执行的一种方式。

    DEX保护:代码VMP虚拟化保护
    SO保护:基于llvm的SO文件保护（ELF VMP）

脱壳方案

HOOK脱壳法：可以基于frida框架进行针对不同虚拟机（dalvik和art）实现进行hook。
定制系统脱壳法