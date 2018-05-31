
## filter-out
$(filter-out PATTERN...,TEXT) 
函数名称：反过滤函数—filter-out
函数功能：和"filter"函数实现的功能相反。过滤掉字串"TEXT"中所有符合模式"PATTERN"的单词，保留所有不符合此模式的单词.
可以有多个模式, 存在多个模式时, 模式表达式之间使用空格分割
返回值：空格分割的"TEXT"字串中所有不符合模式"PATTERN"的字串
函数说明: "filter-out"函数也可以用来去除一个变量中的某些字符串, (实现和"filter"函数相反). 
示例： 
objects=main1.o foo.o main2.o bar.o 
mains=main1.o main2.o 
$(filter-out $(mains),$(objects)) 
结果: 去除变量"objects"中"mains"定义的字串(文件名), 返回值为"foo.o bar.o"

## wildcard
在Makefile规则中，通配符会被自动展开。但在变量的定义和函数引用时，通配符将失效。
这种情况下如果需要通配符有效，就需要使用函数"wildcard"，它的用法是：$(wildcard PATTERN...) 在Makefile中，它被展开为已经存在的、使用空格分开的、匹配此模式的所有文件列表。如果不存在任何符合此模式的文件，函数会忽略模式字符并返回空
注意：这种情况下规则中通配符的展开和上一小节匹配通配符的区别

`"$(wildcard *.c)"` 获取工作目录下的所有的.c文件列表
`SRC = $(wildcard *.c inc/*.c ABC/*.c BCD/*.c) ` 包含子目录为inc... 

### 复杂用法
`"$(patsubst %.c,%.o,$(wildcard *.c))"`
首先使用"wildcard"函数获取工作目录下的.c文件列表, 之后将列表中所有文件名的后缀.c替换为.o. 得到在当前目录可生成的.o文件列表.
因此在一个目录下可以使用如下内容的Makefile来将工作目录下的所有的.c文件进行编译并最后连接成为一个可执行文件:
```
#sample Makefile
objects := $(patsubst %.c,%.o,$(wildcard *.c))

 foo : $(objects)

cc -o foo $(objects)
```
使用make的隐含规则来编译.c的源文件.对变量的赋值也用到了一个特殊的符号(:=).

### wildcard : 扩展通配符
### notdir ： 去除路径
### patsubst ：替换通配符

建立一个测试目录，在测试目录下建立一个名为sub的子目录
```
$ mkdir test
$ cd test
$ mkdir sub
```
在test下, 建立a.c和b.c2个文件, 在sub目录下，建立sa.c和sb.c2 个文件

建立一个简单的Makefile
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
执行结果分析:
第一行输出: a.c b.c ./sub/sa.c ./sub/sb.c
wildcard把 指定目录 ./ 和 ./sub/ 下的所有后缀是c的文件全部展开。

第二行输出: a.c b.c sa.c sb.c
notdir把展开的文件去除掉路径信息

第三行输出: a.o b.o sa.o sb.o
在`$(patsubst %.c,%.o,$(dir) )`中，patsubst把$(dir)中的变量符合后缀是.c的全部替换成.o, 
或者可以使用 `obj=$(dir:%.c=%.o)`

这里用到makefile里的替换引用规则，即用您指定的变量替换另一个变量。
它的标准格式是
$(var:a=b) 或 ${var:a=b}

特别注意：“a=b”中“=”两侧没有空格
它的含义是把变量var中的每一个值结尾用b替换掉a

#### 一个简化版本的万能驱动
```
CROSS_COMPILE = arm-none-linux-gnueabi-#教材编译器
LD = $(CROSS_COMPILE)gcc
CC = $(CROSS_COMPILE)gcc
TARGET = test_app#目标文件名

SRCS = $(wildcard ./src/*.c)
OBJS := $(SRCS:.c=.o)

INCLUDES =  -I ./inc #头文件路径
LIBS = -L ./lib -lpthread #库的路径及相关库
CCFLAGS = -g -Wall -O0#告警及优化等级

%.o : %.c
$(CC) -o $@ $(INCLUDES) $(CCFLAGS) -c $<  

all: $(TARGET)
$(TARGET) : $(OBJS)
$(CC) -o $@ $^ $(LIBS)


clean:
-rm $(OBJS) $(TARGET)
```

#### 带有依赖文件的makefile专业写法
```
CROSS_COMPILE = arm-none-linux-gnueabi-#教材编译器
LD = $(CROSS_COMPILE)gcc
CC = $(CROSS_COMPILE)gcc
TARGET = test_app#目标文件名

SRCS = $(wildcard ./src/*.c)
OBJS := $(SRCS:.c=.o)

INCLUDES =  -I ./inc #头文件路径
LIBS = -L ./lib -lpthread #库的路径及相关库
CCFLAGS = -g -Wall -O0#告警及优化等级

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

#### 再附上一个搜索到的万能驱动
```
#Genericmakefile　 
#　 
#byGeorgeFoot　 
#email:george.foot@merton.ox.ac.uk　 
#　 
#Copyright(c)1997　GeorgeFoot　 
#Allrightsreserved.　 
#　保留所有版權　 
#　 
#Nowarranty,noliability;　 
#youusethis　atyourownrisk.　 
#　沒保險，不負責　 
#　你要用這個，你自己擔風險　 
#　 
#Youarefreetomodifyand　 
#distributethis　withoutgiving　 
#credittotheoriginalauthor.　 
#　你可以隨便更改和散發這個文件　 
#　而不需要給原作者什榮譽。　 
#　 
###################################### 

###Customising 
# 
#Adjustthefollowingif　necessary;EXECUTABLEis　thetarget 
#executable'sfilename,andLIBSisalistoflibrariestolinkin 
#(e.g.alleg,stdcx,iostr,etc).Youcanoverride　theseonmake's 
#commandlineofcourse,if　youprefertodo　itthatway. 
#　 
#　如果需要，調整下面的東西。　EXECUTABLE　是目標的可執行文件名，　LIBS 
#　是一個需要連接的程序包列表（例如　alleg,stdcx,iostr　等等）。當然你 
#　可以在　make　的命令行覆蓋它們，你願意就沒問題。 
#　 

EXECUTABLE:=mushroom.exe 
LIBS:=alleg 

#Nowalteranyimplicit　rules'variablesifyoulike,e.g.: 
# 
#　現在來改變任何你想改動的隱含規則中的變量，例如 

CFLAGS:=-g-Wall-O3-m486 
CXXFLAGS:=$(CFLAGS) 

#Thenextbitcheckstoseewhetherrmis　in　yourdjgppbin 
#directory;if　notitusesdelinstead,butthis　cancause(harmless) 
#`Filenotfound'errormessages.IfyouarenotusingDOSatall, 
#set　thevariabletosomethingwhichwillunquestioninglyremove 
#files. 
# 
#　下面先檢查你的　djgpp　命令目錄下有沒有　rm　命令，如果沒有，我們使用 
#del　命令來代替，但有可能給我們　'Filenotfound'　這個錯誤信息，這沒 
#　什大礙。如果你不是用　DOS　，把它設定成一個刪文件而不廢話的命令。 
#　（其實這一步在　UNIX　類的系統上是多余的，只是方便　DOS　用戶。　UNIX 
#　用戶可以刪除這５行命令。） 

ifneq($(wildcard$(DJDIR)/bin/rm.exe),) 
RM-F:=rm-f 
else 
RM-F:=del 
endif 

#Youshouldn'tneedtochangeanythingbelowthispoint. 
# 
#　從這裡開始，你應該不需要改動任何東西

SOURCE:=$(wildcard*.c)$(wildcard*.cc) 
OBJS:=$(patsubst%.c,%.o,$(patsubst%.cc,%.o,$(SOURCE))) 
DEPS:=$(patsubst%.o,%.d,$(OBJS)) 
MISSING_DEPS:=$(filter-out　$(wildcard$(DEPS)),$(DEPS)) 
MISSING_DEPS_SOURCES:=$(wildcard$(patsubst%.d,%.c,$(MISSING_DEPS))\ 
$(patsubst%.d,%.cc,$(MISSING_DEPS))) 
CPPFLAGS+=-MD 

.PHONY:everythingdepsobjscleanverycleanrebuild 

everything:$(EXECUTABLE) 

deps:$(DEPS) 

objs:$(OBJS) 

clean: 
　　@$(RM-F)*.o 
　　@$(RM-F)*.d 

veryclean:clean 
　　@$(RM-F)$(EXECUTABLE) 

rebuild:verycleaneverything 

ifneq($(MISSING_DEPS),) 
$(MISSING_DEPS): 
　　@$(RM-F)$(patsubst%.d,%.o,$@) 
endif 

-include$(DEPS) 

$(EXECUTABLE):$(OBJS) 
　　gcc-o$(EXECUTABLE)$(OBJS)$(addprefix-l,$(LIBS)) 
###
```

今天的实例: 使用函数wildcard得到指定目录下所要文件
SRC = $(wildcard *Main.c)

增加子目录inc，则再增加一个wildcard函数
SRC = $(wildcard *.c) $(wildcard inc/*.c)
也可以指定汇编源程序：
ASRC = $(wildcard *.S)

过滤出来指定目录下的非main函数
BASEPATH = $(PROJECT_ROOT)/C_Application/BMA/Source
CFILES := $(filter-out $(rwildcard,$(BASEPATH)/,*Main.cpp), $(rwildcard,$(BASEPATH)/,*.cpp))

