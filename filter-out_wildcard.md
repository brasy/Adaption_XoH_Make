
## filter-out
$(filter-out PATTERN...,TEXT) 
�������ƣ������˺�����filter-out
�������ܣ���"filter"����ʵ�ֵĹ����෴�����˵��ִ�"TEXT"�����з���ģʽ"PATTERN"�ĵ��ʣ��������в����ϴ�ģʽ�ĵ���.
�����ж��ģʽ, ���ڶ��ģʽʱ, ģʽ���ʽ֮��ʹ�ÿո�ָ�
����ֵ���ո�ָ��"TEXT"�ִ������в�����ģʽ"PATTERN"���ִ�
����˵��: "filter-out"����Ҳ��������ȥ��һ�������е�ĳЩ�ַ���, (ʵ�ֺ�"filter"�����෴). 
ʾ���� 
objects=main1.o foo.o main2.o bar.o 
mains=main1.o main2.o 
$(filter-out $(mains),$(objects)) 
���: ȥ������"objects"��"mains"������ִ�(�ļ���), ����ֵΪ"foo.o bar.o"

## wildcard
��Makefile�����У�ͨ����ᱻ�Զ�չ�������ڱ����Ķ���ͺ�������ʱ��ͨ�����ʧЧ��
��������������Ҫͨ�����Ч������Ҫʹ�ú���"wildcard"�������÷��ǣ�$(wildcard PATTERN...) ��Makefile�У�����չ��Ϊ�Ѿ����ڵġ�ʹ�ÿո�ֿ��ġ�ƥ���ģʽ�������ļ��б�����������κη��ϴ�ģʽ���ļ������������ģʽ�ַ������ؿ�
ע�⣺��������¹�����ͨ�����չ������һС��ƥ��ͨ���������

`"$(wildcard *.c)"` ��ȡ����Ŀ¼�µ����е�.c�ļ��б�
`SRC = $(wildcard *.c inc/*.c ABC/*.c BCD/*.c) ` ������Ŀ¼Ϊinc... 

### �����÷�
`"$(patsubst %.c,%.o,$(wildcard *.c))"`
����ʹ��"wildcard"������ȡ����Ŀ¼�µ�.c�ļ��б�, ֮���б��������ļ����ĺ�׺.c�滻Ϊ.o. �õ��ڵ�ǰĿ¼�����ɵ�.o�ļ��б�.
�����һ��Ŀ¼�¿���ʹ���������ݵ�Makefile��������Ŀ¼�µ����е�.c�ļ����б��벢������ӳ�Ϊһ����ִ���ļ�:
```
#sample Makefile
objects := $(patsubst %.c,%.o,$(wildcard *.c))

 foo : $(objects)

cc -o foo $(objects)
```
ʹ��make����������������.c��Դ�ļ�.�Ա����ĸ�ֵҲ�õ���һ������ķ���(:=).

### wildcard : ��չͨ���
### notdir �� ȥ��·��
### patsubst ���滻ͨ���

����һ������Ŀ¼���ڲ���Ŀ¼�½���һ����Ϊsub����Ŀ¼
```
$ mkdir test
$ cd test
$ mkdir sub
```
��test��, ����a.c��b.c2���ļ�, ��subĿ¼�£�����sa.c��sb.c2 ���ļ�

����һ���򵥵�Makefile
```
src=$(wildcard *.c ./sub/*.c)
dir=$(notdir $(src))
obj=$(patsubst %.c,%.o,$(dir) ) 

all:
 @echo $(src)
 @echo $(dir)
 @echo $(obj)
 @echo "end"
```
ִ�н������:
��һ�����: a.c b.c ./sub/sa.c ./sub/sb.c
wildcard�� ָ��Ŀ¼ ./ �� ./sub/ �µ����к�׺��c���ļ�ȫ��չ����

�ڶ������: a.c b.c sa.c sb.c
notdir��չ�����ļ�ȥ����·����Ϣ

���������: a.o b.o sa.o sb.o
��`$(patsubst %.c,%.o,$(dir) )`�У�patsubst��$(dir)�еı������Ϻ�׺��.c��ȫ���滻��.o, 
���߿���ʹ�� `obj=$(dir:%.c=%.o)`

�����õ�makefile����滻���ù��򣬼�����ָ���ı����滻��һ��������
���ı�׼��ʽ��
$(var:a=b) �� ${var:a=b}

�ر�ע�⣺��a=b���С�=������û�пո�
���ĺ����ǰѱ���var�е�ÿһ��ֵ��β��b�滻��a

#### һ���򻯰汾����������
```
CROSS_COMPILE = arm-none-linux-gnueabi-#�̲ı�����
LD = $(CROSS_COMPILE)gcc
CC = $(CROSS_COMPILE)gcc
TARGET = test_app#Ŀ���ļ���

SRCS = $(wildcard ./src/*.c)
OBJS := $(SRCS:.c=.o)

INCLUDES =  -I ./inc #ͷ�ļ�·��
LIBS = -L ./lib -lpthread #���·������ؿ�
CCFLAGS = -g -Wall -O0#�澯���Ż��ȼ�

%.o : %.c
$(CC) -o $@ $(INCLUDES) $(CCFLAGS) -c $<  

all: $(TARGET)
$(TARGET) : $(OBJS)
$(CC) -o $@ $^ $(LIBS)


clean:
-rm $(OBJS) $(TARGET)
```

#### ���������ļ���makefileרҵд��
```
CROSS_COMPILE = arm-none-linux-gnueabi-#�̲ı�����
LD = $(CROSS_COMPILE)gcc
CC = $(CROSS_COMPILE)gcc
TARGET = test_app#Ŀ���ļ���

SRCS = $(wildcard ./src/*.c)
OBJS := $(SRCS:.c=.o)

INCLUDES =  -I ./inc #ͷ�ļ�·��
LIBS = -L ./lib -lpthread #���·������ؿ�
CCFLAGS = -g -Wall -O0#�澯���Ż��ȼ�

%.o : %.c
$(CC) -o $@ $(INCLUDES) $(CCFLAGS) -c $<  

%.d : %.c 
echo $@;\
set -e; rm -f $@; \
$(CC) -MM $(CCFLAGS) $(INCLUDES) $< > $@.
; \
sed 's/$(*F)\.o/$(subst /,\/,$(@:.d=.o))/g' < $@.
> $@; \
rm -f $@.
\

all: $(TARGET)
$(TARGET) : $(OBJS)
$(CC) -o $@ $^ $(LIBS)


clean:

-rm $(OBJS) $(TARGET) $(OBJS:.o=.d)

include $(OBJS:.o=.d)
```

#### �ٸ���һ������������������
```
#Genericmakefile�� 
#�� 
#byGeorgeFoot�� 
#email:george.foot@merton.ox.ac.uk�� 
#�� 
#Copyright(c)1997��GeorgeFoot�� 
#Allrightsreserved.�� 
#���������а�ࡡ 
#�� 
#Nowarranty,noliability;�� 
#youusethis��atyourownrisk.�� 
#���]���U����ؓ؟�� 
#����Ҫ���@�������Լ����L�U�� 
#�� 
#Youarefreetomodifyand�� 
#distributethis��withoutgiving�� 
#credittotheoriginalauthor.�� 
#��������S����ĺ�ɢ�l�@���ļ��� 
#��������Ҫ�oԭ����ʲ�s�u���� 
#�� 
###################################### 

###Customising 
# 
#Adjustthefollowingif��necessary;EXECUTABLEis��thetarget 
#executable'sfilename,andLIBSisalistoflibrariestolinkin 
#(e.g.alleg,stdcx,iostr,etc).Youcanoverride��theseonmake's 
#commandlineofcourse,if��youprefertodo��itthatway. 
#�� 
#�������Ҫ���{������Ė|������EXECUTABLE����Ŀ�˵ĿɈ����ļ�������LIBS 
#����һ����Ҫ�B�ӵĳ�����б����硡alleg,stdcx,iostr���ȵȣ�����Ȼ�� 
#�������ڡ�make���������и��w�����������͛]���}�� 
#�� 

EXECUTABLE:=mushroom.exe 
LIBS:=alleg 

#Nowalteranyimplicit��rules'variablesifyoulike,e.g.: 
# 
#���F�ځ��׃�κ�����Ąӵ��[��Ҏ�t�е�׃�������� 

CFLAGS:=-g-Wall-O3-m486 
CXXFLAGS:=$(CFLAGS) 

#Thenextbitcheckstoseewhetherrmis��in��yourdjgppbin 
#directory;if��notitusesdelinstead,butthis��cancause(harmless) 
#`Filenotfound'errormessages.IfyouarenotusingDOSatall, 
#set��thevariabletosomethingwhichwillunquestioninglyremove 
#files. 
# 
#�������șz����ġ�djgpp������Ŀ����Л]�С�rm���������]�У��҂�ʹ�� 
#del���������棬���п��ܽo�҂���'Filenotfound'���@���e�`��Ϣ���@�] 
#��ʲ��K������㲻���á�DOS���������O����һ���h�ļ������UԒ����� 
#�����䌍�@һ���ڡ�UNIX���ϵ�y���Ƕ���ģ�ֻ�Ƿ��㡡DOS���Ñ�����UNIX 
#���Ñ����Ԅh���@��������� 

ifneq($(wildcard$(DJDIR)/bin/rm.exe),) 
RM-F:=rm-f 
else 
RM-F:=del 
endif 

#Youshouldn'tneedtochangeanythingbelowthispoint. 
# 
#�����@�e�_ʼ���㑪ԓ����Ҫ�Ą��κΖ|��

SOURCE:=$(wildcard*.c)$(wildcard*.cc) 
OBJS:=$(patsubst%.c,%.o,$(patsubst%.cc,%.o,$(SOURCE))) 
DEPS:=$(patsubst%.o,%.d,$(OBJS)) 
MISSING_DEPS:=$(filter-out��$(wildcard$(DEPS)),$(DEPS)) 
MISSING_DEPS_SOURCES:=$(wildcard$(patsubst%.d,%.c,$(MISSING_DEPS))\ 
$(patsubst%.d,%.cc,$(MISSING_DEPS))) 
CPPFLAGS+=-MD 

.PHONY:everythingdepsobjscleanverycleanrebuild 

everything:$(EXECUTABLE) 

deps:$(DEPS) 

objs:$(OBJS) 

clean: 
����@$(RM-F)*.o 
����@$(RM-F)*.d 

veryclean:clean 
����@$(RM-F)$(EXECUTABLE) 

rebuild:verycleaneverything 

ifneq($(MISSING_DEPS),) 
$(MISSING_DEPS): 
����@$(RM-F)$(patsubst%.d,%.o,$@) 
endif 

-include$(DEPS) 

$(EXECUTABLE):$(OBJS) 
����gcc-o$(EXECUTABLE)$(OBJS)$(addprefix-l,$(LIBS)) 
###
```

�����ʵ��: ʹ�ú���wildcard�õ�ָ��Ŀ¼����Ҫ�ļ�
SRC = $(wildcard *Main.c)

������Ŀ¼inc����������һ��wildcard����
SRC = $(wildcard *.c) $(wildcard inc/*.c)
Ҳ����ָ�����Դ����
ASRC = $(wildcard *.S)

���˳���ָ��Ŀ¼�µķ�main����
BASEPATH = $(PROJECT_ROOT)/C_Application/BMA/Source
CFILES := $(filter-out $(rwildcard,$(BASEPATH)/,*Main.cpp), $(rwildcard,$(BASEPATH)/,*.cpp))

