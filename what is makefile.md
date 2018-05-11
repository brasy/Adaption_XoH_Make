# MakeFile notes
  ```bash
  Reference: http://blog.csdn.net/lhf_tiger/article/category/954650 
  ```

## 什么是makefile？
* Windows的IDE
* Unix下的软件编译，会写makefile，具备完成大型工程的能力。
* makefile关系到了整个工程的编译规则。
  1.定义了一系列的规则来指定编译顺序。
  2.makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。 
  3.makefile带来的好处就是——“自动化编译”，一个make命令，整个工程完全自动编译
  4.make是解释makefile中指令的命令工具

## 程序的编译和链接 
———————
程序编译的一些规范和方法，首先要把源文件编译成中间代码文件，在Windows下是 `.obj` 文件，UNIX下是 `.o` 文件，即 Object File，这个动作叫做编译`compile`。然后再把大量的Object File合成执行文件，这个动作叫作链接`link` 
* 编译时，编译器需要的是语法的正确，函数与变量的声明的正确。每个源文件都应该对应于一个中间目标文件`O文件或是OBJ文件`
* 链接时，主要是链接函数和全局变量，使用这些中间目标文件`O文件或是OBJ文件`来链接我们的应用程序。中间目标文件太多，打个包，在Windows下这种包叫`库文件``Library File`，是 `.lib` 文件，在UNIX下，是Archive File，是 `.a` 文件
总结一下，源文件->中间目标文件->执行文件。

## Makefile 介绍
——————— 
make命令执行时，需要一个 Makefile 文件，以告诉make命令需要怎么样的去编译和链接程序。 
### Makefile的书写规则
1. 如果这个工程没有编译过，那么我们的所有C文件都要编译并被链接。
2. 如果这个工程的某几个C文件被修改，那么我们只编译被修改的C文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的C文件，并链接目标程序。
只用一个make命令就可以完成，make命令会自动智能地根据当前的文件修改的情况来确定哪些文件需要重编译，从而自己编译所需要的文件和链接目标程序。

### Makefile的规则 
在讲述这个Makefile之前，还是让我们先来粗略地看一看Makefile的规则、依赖关系。 
  ```bash
  target ... : prerequisites ... 
  command 
  ... 
  ... 
  clean :
  rm target ……
  ```
target一个目标文件，可以是Object File，也可以是执行文件。还可以是一个标签`Label`
prerequisites就是，要生成那个target所需要的文件或是目标
command也就是make需要执行的命令。（任意的Shell命令）.一定要以Tab键作为开头

**Makefile**的规则：target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中; prerequisites中如果有一个以上的文件比target文件要新的话，command所定义的命令就会被执行,也是Makefile中最核心的内容
notes: 反斜杠`\`是换行符的意思
clean不是一个文件，只是一个动作名字，像C语言中的lable一样，其冒号后什么也没有
我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等

### make是如何工作的 
在默认的方式下，只输入make命令
1、	make在当前目录下找名字叫“Makefile”或“makefile”的文件。
2、	如果找到，它会找文件中的第一个目标文件（target），上例子中，找到“edit”这个文件，并把这个文件作为最终的目标文件。
3、	如果edit文件不存在，或是edit所依赖的后面的 .o 文件的文件修改时间要比edit这个文件新，那么，他就会执行后面所定义的命令来生成edit这个文件。
4、	如果edit所依赖的.o文件也存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（这有点像一个堆栈的过程）
5、	当然，C文件和H文件是存在的，于是make会生成 .o 文件，然后再用 .o 文件生命make的终极任务，也就是执行文件edit了。 
这就是整个make的依赖性，make会逐层地找文件的依赖关系，直到最终编译出第一个目标文件。make只管文件的依赖性。

### makefile中使用变量 
为了makefile的易维护，在makefile中可以使用变量。makefile的变量也就是一个字符串，理解成C语言中的宏。
声明一个变量，叫objects。在makefile一开始就这样定义： 
`objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o`
于是，在makefile中以`“$(objects)”`的方式来使用这个变量了，改良版makefile如下面：
```
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o
clean : 
rm edit $(objects) 
```
于是如果有新的 `.o` 文件加入，我们只需简单地修改一下 objects 变量就可以了 

### make自动推导 
GNU的make很强大，它可以自动推导文件依赖及命令。
make看到一个[.o]文件，它就会自动的把[.c]文件加在依赖关系中
```
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o 
.PHONY : clean
clean : 
rm edit $(objects)
```
这种方法，也就是make的`隐晦规则`, 其中，`“.PHONY”`表示，`clean`是个伪目标文件。 

### 另类风格的makefile 
即然make可以自动推导命令，清除多的重复的[.h]， make提供了自动推导命令和文件的功能呢
```
objects = main.o kbd.o command.o display.o insert.o search.o files.o utils.o 
edit : $(objects) 
cc -o edit $(objects) 
$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h 
.PHONY : clean 
clean : 
rm edit $(objects) 
```
makefile变得很简单，但文件依赖关系就显得有点凌乱。鱼和熊掌不可兼得。一是文件的依赖关系看不清楚，二是如果文件一多，要加入几个新的.o文件，那就理不清楚了。 

七、清空目标文件的规则 
每个Makefile中都应该写一个清空目标文件（.o和执行文件）的规则，这不仅便于重编译，也很利于保持文件的清洁。一般的风格都是： 
```
clean: 
rm edit $(objects) 
```
更为稳健的做法是：
```
.PHONY : clean 
clean : 
-rm edit $(objects)
```
.PHONY意思表示clean是一个“伪目标”，。而在rm命令前面加了一个小减号的意思就是，也许某些文件出现问题，但不要管，继续做后面的事。clean的规则不要放在文件的开头，不然，这就会变成make的默认目标。不成文的规矩是——“clean从来都是放在文件的最后”。 
