# ���л���linux run Myexe, libprotobuf.so.9�Ҳ���
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

2. �鿴���뻷��, Myexe�����makefile���ҵ�������link��
�ҵ��ⲿ�����mk��PIAһ�㶼�Ǻ��ⲿģ��Ľӿڣ��ղ��õ�����PIA�����BMAģ��
```
Make]$ cat  /var/fwork/xying/project/Application/OM/PIA/Make/BMYexe.mk
else ifeq ($(ARCH),arm_nrt)
EXTRA_LDFLAGS += -lprotobuf -lCCS -ldl -lpthread
```
����ȷ���õ���protobuf��������protobuf-lite����������һ�����������������������⡣

3. �鿴���뻷���ϣ������libprotobuf.so.9 �����link�ģ�������
��Ȼ�����ܹ�����ɹ�������ζ�ű��뻷����һ��������⣬���������׵�.
(1)�ұ��뻷���ϵĿ�
����find�����ֻ�ҵ�libprotobuf.so.7.0.0������ƥ��file
```
]$ find / > /tmp/xxxxx
]$ cd /linux_builds/ipalightsdkroot/FZC/SAPI/R_FPT_165.7.1.13/data/SS_Protobuf/build/protobuf_target/lib
```
�Ƚ�MYexe��lib���ļ��� file���ݣ����߲���ɣ���������������ļ��������ϡ�
```
lib]$ file libprotobuf.so.7.0.0 
libprotobuf.so.7.0.0: ELF 64-bit MSB shared object, MIPS, MIPS64 rel2 version 1 (SYSV), dynamically linked, with unknown capability 0x410000000f676e75 = 0x1000000070403, not stripped
lib]$ file /var/fwork/xying/project/exec/arm_nrt/release/Myexe 
/var/fpwork/xying001/trunk/lteDo/exec/arm_cortexa15_nrt/release/Myexe: ELF 32-bit LSB executable, ARM, version 1 (GNU/Linux), dynamically linked (uses shared libs), for GNU/Linux 4.4.0, stripped
```
�ٿ�lib������ݣ������ҵ������ǰ汾����һ��
```
lib]$ nm libprotobuf.so.7.0.0 | grep "ZN6google8protobuf15FieldDescriptor17"
00000000000e5f58 R _ZN6google8protobuf15FieldDescriptor17kTypeToCppTypeMapE
```
���뻷���ϣ�ԭ�е�linux�ϵĲ�����, Ҳû��tools-chain, ����ȷ������������õ���˽����tools-chain

(2) ����gcc�ɣ�����ɹ�����������Ӧ��lib��Ӧ���������
Step1����makefile,���һ��mk�����ԡ��Ѿ���װ��, û��gcc·��
`$(Q)$(MAKE) $(build)=Application/.../ makefile=ProtoGen.mk`

Step2���鿴�ϴα���������ϸ��log���ؼ���gcc���ҵ���������
```
/build/.../os/archs/arm-linux-gnueabihf/bld-tools/x86_64-pc-linux-gnu/bin/arm-linux-gnueabihf-gcc -o����
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
����������ܷ��ϣ�arm��32bit�ģ��ͱ��������MYexeһ��, OK
`lib]$ cp libprotobuf.so.9.0.1 /home/xying/`

Step3: �������libprotobuf.so.9.0.1Ū�����л����ϵ�, ��link��, ��Ϊ˽��, ����Ҫ���Լ�����lib export������OK
```
root@a:~/om >cp libprotobuf.so.9.0.1 /usr/lib/
root@a:~/om >ln -s /usr/lib/libprotobuf.so.9.0.1 /usr/lib/libprotobuf.so.9
root@a:~/om >export LD_LIBRARY_PATH=/user/toor4n/om
root@a:~/om >./MYexe &
[1] 4864
root@a:~/om >2018-02-23T12:21:16.791Z ::APPS_START :OM Interface is ready
```

4. �������±���������libprotobuf.so.9
��������gcc����protobufԴ�룬���Խ������������
