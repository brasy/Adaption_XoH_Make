# 运行环境linux run Myexe, libprotobuf.so.9找不到
```
root@a:~/om >./Myexe &
root@a:~/om >./Myexe: error while loading shared libraries: libprotobuf.so.9: cannot open shared object file: No such file or directory
```

# solution
1. try to find this lib
```
root@a:/usr/lib >ll | grep libproto
lrwxrwxrwx  1 root root       25 Apr  5  2018 libprotobuf-lite.so.9 -> libprotobuf-lite.so.9.0.1
-rwxr-xr-x  1 root root   141848 Apr  5  2018 libprotobuf-lite.so.9.0.1
```
result: no this lib, `libprotobuf-lite.so.9.0.1` is total different with `libprotobuf.so.9`

2. 查看编译环境, Myexe代码的makefile，找到在哪里link的
找到外部结果的mk，PIA一般都是和外部模块的接口，刚才用到的是PIA里面的BMY模块
```
Make]$ cat  /var/fwork/xying/project/Application/OM/PIA/Make/BMYexe.mk
else ifeq ($(ARCH),arm_nrt)
EXTRA_LDFLAGS += -lprotobuf -lCCS -ldl -lpthread
```
这里确定用的是protobuf，而不是protobuf-lite，在这里有一个对外库依赖，会连接这个库。

3. 查看编译环境上，这个库libprotobuf.so.9 是如何link的，在哪里
既然代码能够编译成功，那意味着编译环境上一定有这个库，而且是配套的.
(1)找编译环境上的库
环境find，结果只找到libprotobuf.so.7.0.0，尝试匹配file
```
]$ find / > /tmp/xxxxx
]$ cd /linux_builds/ipalightsdkroot/FZC/SAPI/R_FPT_165.7.1.13/data/SS_Protobuf/build/protobuf_target/lib
```
比较MYexe和lib库文件的 file内容，两者不相干，看起来不是这个文件，不符合。
```
lib]$ file libprotobuf.so.7.0.0 
libprotobuf.so.7.0.0: ELF 64-bit MSB shared object, MIPS, MIPS64 rel2 version 1 (SYSV), dynamically linked, with unknown capability 0x410000000f676e75 = 0x1000000070403, not stripped
lib]$ file /var/fwork/xying/project/exec/arm_nrt/release/Myexe 
/var/fpwork/xying001/trunk/lteDo/exec/arm_cortexa15_nrt/release/Myexe: ELF 32-bit LSB executable, ARM, version 1 (GNU/Linux), dynamically linked (uses shared libs), for GNU/Linux 4.4.0, stripped
```
再看lib库的内容，可以找到，但是版本名不一样
```
lib]$ nm libprotobuf.so.7.0.0 | grep "ZN6google8protobuf15FieldDescriptor17"
00000000000e5f58 R _ZN6google8protobuf15FieldDescriptor17kTypeToCppTypeMapE
```
编译环境上，原有的linux上的不能用, 也没有tools-chain, 基本确定这个环境的用的是私建的tools-chain

(2) 那找gcc吧，编译成功，这个里面对应的lib库应该有这个库
Step1：找makefile,随便一个mk都可以。已经封装了, 没有gcc路径
`$(Q)$(MAKE) $(build)=Application/.../ makefile=ProtoGen.mk`

Step2：查看上次编译结果，详细的log，关键词gcc，找到如下内容
```
/build/.../os/archs/arm-linux-gnueabihf/bld-tools/x86_64-pc-linux-gnu/bin/arm-linux-gnueabihf-gcc -o……
]$ cd /build/.../os/archs/arm-linux-gnueabihf/
arm-linux-gnueabihf]$ ll
total 0
22 May  4 02:53 bld-tools -> ../../../sdk/bld-tools
22 May  4 02:53 dbg-tools -> ../../../sdk/dbg-tools
44 May  4 02:53 sys-root -> ../../sys-root/arm-linux-gnueabihf
arm-cortexa15-linux-gnueabihf]$ cd sys-root/lib
lib]$ file libprotobuf.so.9.0.1
libprotobuf.so.9.0.1: ELF 32-bit LSB shared object, ARM, version 1 (SYSV), dynamically linked, stripped
```
这个看起来很符合，arm的32bit的，和编译出来的MYexe一致, OK
`lib]$ cp libprotobuf.so.9.0.1 /home/xying/`

Step3: 将这个库libprotobuf.so.9.0.1弄到运行环境上的, 并link上, 因为私包, 所以要把自己依赖lib export进来，OK
```
root@a:~/om >ln -s /../libprotobuf.so.9.0.1 /../libprotobuf.so.9
root@a:~/om >export LD_LIBRARY_PATH=/user/toor4n/om
root@a:~/om >./MYexe &
[1] 4864
root@a:~/om >2018-02-23T12:21:16.791Z ::APPS_START :OM Interface is ready
```

4. 尝试重新编译个这个库libprotobuf.so.9
环境上有gcc，和protobuf源码，尝试交叉编译出这个库.
这个是实在不行的下策了，平常的项目中应该有统一管理资源的，会统一config这些dependence.
