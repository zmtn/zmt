+++
title = 'Makefile学习手册'
date = 2025-06-10T09:55:36+08:00
draft = false
+++

> Makefile相关的学习

---

<!--more-->
- [概述](#概述)
- [makefile介绍](#makefile介绍)
  - [makefile规则](#makefile规则)
  - [一个示例](#一个示例)
  - [make是如何工作的](#make是如何工作的)
  - [makefile中使用变量](#makefile中使用变量)
  - [让make自动推导](#让make自动推导)
  - [makefile的另一种风格](#makefile的另一种风格)
  - [清空目录的规则](#清空目录的规则)
  - [Makefile里有什么](#makefile里有什么)
- [书写规则](#书写规则)
- [书写命令](#书写命令)
  - [显示命令](#显示命令)
  - [命令执行](#命令执行)
  - [命令出错](#命令出错)
  - [嵌套执行make](#嵌套执行make)
  - [定义命令包](#定义命令包)
- [使用变量](#使用变量)
- [使用条件判断](#使用条件判断)
- [使用函数](#使用函数)
- [make的运行](#make的运行)
- [隐含规则](#隐含规则)
- [使用make更新函数库文件](#使用make更新函数库文件)
- [后续](#后续)

> MakeFile学习参考手册 [^1]

## 概述

什么是makefile？或许很多Windows的程序员都不知道这个东西，因为那些Windows的集成开发环境（integrated development environment，IDE）都为你做了这个工作，但我觉得要作一个好的和专业的程序员，makefile还是要懂。这就好像现在有这么多的HTML编辑器，但如果你想成为一个专业人士，你还是要了解HTML的标签的含义。特别在Unix下的软件编译，你就不能不自己写makefile了，会不会写makefile，从一个侧面说明了一个人是否具备完成大型工程的能力。

因为，makefile关系到了整个工程的编译规则。一个工程中的源文件不计其数，并且按类型、功能、模块分别放在若干个目录中，makefile定义了一系列的规则来指定，哪些文件需要先编译，哪些文件需要后编译，哪些文件需要重新编译，甚至于进行更复杂的功能操作，因为makefile就像一个Shell脚本一样，其中也可以执行操作系统的命令。

`makefile`带来的好处就是——“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大的提高了软件开发的效率。 make是一个命令工具，是一个解释makefile中指令的命令工具，一般来说，大多数的IDE都有这个命令，比如：Delphi的make，Visual C++的nmake，Linux下GNU的make。可见，makefile都成为了一种在工程方面的编译方法。

现在讲述如何写makefile的文章比较少，这是我想写这篇文章的原因。当然，不同产商的make各不相同，也有不同的语法，但其本质都是在 “文件依赖性”上做文章，这里，我仅对GNU的make进行讲述，我的环境是RedHat Linux 8.0，make的版本是3.80。毕竟，这个make是应用最为广泛的，也是用得最多的。而且其还是最遵循于IEEE 1003.2-1992标准的（POSIX.2）。

在这篇文档中，将以C/C++的源码作为基础，所以必然涉及一些关于C/C++的编译的知识。关于这方面的内容，还请各位查看相关的编译器的文档。这里所默认的编译器是UNIX下的GCC和CC

`关于程序的编译和链接`
在此，我想多说关于程序编译的一些规范和方法。一般来说，无论是C还是C++，首先要把源文件编译成中间代码文件，在Windows下也就是 .obj 文件，UNIX下是 .o 文件，即Object File，这个动作叫做编译（compile）。然后再把大量的Object File合成可执行文件，这个动作叫作链接（link）。

编译时，编译器需要的是语法的正确，函数与变量的声明的正确。对于后者，通常是你需要告诉编译器头文件的所在位置（头文件中应该只是声明，而定义应该放在C/C++文件中），只要所有的语法正确，编译器就可以编译出中间目标文件。一般来说，每个源文件都应该对应于一个中间目标文件（ .o 文件或 .obj 文件）。

链接时，主要是链接函数和全局变量。所以，我们可以使用这些中间目标文件（ .o 文件或 .obj 文件）来链接我们的应用程序。链接器并不管函数所在的源文件，只管函数的中间目标文件（Object File），在大多数时候，由于源文件太多，编译生成的中间目标文件太多，而在链接时需要明显地指出中间目标文件名，这对于编译很不方便。所以，我们要给中间目标文件打个包，在Windows下这种包叫“库文件”（Library File），也就是 .lib 文件，在UNIX下，是Archive File，也就是 .a 文件。

总结一下，源文件首先会生成中间目标文件，再由中间目标文件生成可执行文件。在编译时，编译器只检测程序语法和函数、变量是否被声明。如果函数未被声明，编译器会给出一个警告，但可以生成Object File。而在链接程序时，链接器会在所有的Object File中找寻函数的实现，如果找不到，那到就会报链接错误码（Linker Error），在VC下，这种错误一般是： Link 2001错误 ，意思说是说，链接器未能找到函数的实现。你需要指定函数的Object File。

好，言归正传，gnu的make有许多的内容，闲言少叙。

## makefile介绍

make命令执行时，需要一个makefile文件，以告诉make命令需要怎么样的去编译和链接程序。

首先，我们用一个示例来说明makefile的书写规则，以便给大家一个感性认识。这个示例来源于gnu 的make使用手册，在这个示例中，我们的工程有8个c文件，和3个头文件，我们要写一个makefile来告诉make命令如何编译和链接这几个文件。我们的规则是：

如果这个工程没有编译过，那么我们的所有c文件都要编译并被链接。

如果这个工程的某几个c文件被修改，那么我们只编译被修改的c文件，并链接目标程序。

如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的c文件，并链接目标程序。

只要我们的makefile写得够好，所有的这一切，我们只用一个make命令就可以完成，make命令会自动智能地根据当前的文件修改的情况来确定哪些文件需要重编译，从而自动编译所需要的文件和链接目标程序。

### makefile规则
在讲述这个makefile之前，还是让我们先来粗略地看一看makefile的规则。

```makefile
target ... : prerequisites ...
    recipe
    ...
    ...
```
`target`
可以是一个object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。对于标签这种特性，在后续的“伪目标”章节中会有叙述。

`prerequisites`
生成该target所依赖的文件和/或target。

`recipe`
该target要执行的命令（任意的shell命令）。

这是一个文件的依赖关系，也就是说，target这一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。说白一点就是说:

`prerequisites中如果有一个以上的文件比target文件要新的话，recipe所定义的命令就会被执行。`
这就是makefile的规则，也就是makefile中最核心的内容。

说到底，makefile的东西就是这样一点，好像我的这篇文档也该结束了。呵呵。还不尽然，这是makefile 的主线和核心，但要写好一个makefile还不够，我会在后面一点一点地结合我的工作经验给你慢慢道来。内容还多着呢。:)


### 一个示例

正如前面所说，如果一个工程有3个头文件和8个C文件，为了完成前面所述的那三个规则，我们的makefile 应该是下面的这个样子的。

```makefile
edit : main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
    cc -o edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h buffer.h
    cc -c display.c
insert.o : insert.c defs.h buffer.h
    cc -c insert.c
search.o : search.c defs.h buffer.h
    cc -c search.c
files.o : files.c defs.h buffer.h command.h
    cc -c files.c
utils.o : utils.c defs.h
    cc -c utils.c
clean :
    rm edit main.o kbd.o command.o display.o \
        insert.o search.o files.o utils.o
```

反斜杠（ \ ）是换行符的意思。这样比较便于makefile的阅读。我们可以把这个内容保存在名字为“makefile”或“Makefile”的文件中，然后在该目录下直接输入命令 make 就可以生成执行文件edit。如果要删除可执行文件和所有的中间目标文件，那么，只要简单地执行一下 make clean 就可以了。

在这个makefile中，目标文件（target）包含：可执行文件edit和中间目标文件（ *.o ），依赖文件（prerequisites）就是冒号后面的那些 .c 文件和 .h 文件。每一个 .o 文件都有一组依赖文件，而这些 .o 文件又是可执行文件 edit 的依赖文件。依赖关系的实质就是说明了目标文件是由哪些文件生成的，换言之，目标文件是哪些文件更新的。

在定义好依赖关系后，后续的recipe行定义了如何生成目标文件的操作系统命令，一定要以一个 Tab 键作为开头。记住，make并不管命令是怎么工作的，他只管执行所定义的命令。make会比较targets文件和prerequisites文件的修改日期，如果prerequisites文件的日期要比targets文件的日期要新，或者target不存在的话，那么，make就会执行后续定义的命令。

这里要说明一点的是， clean 不是一个文件，它只不过是一个动作名字，有点像C语言中的label一样，其冒号后什么也没有，那么，make就不会自动去找它的依赖性，也就不会自动执行其后所定义的命令。要执行其后的命令，就要在make命令后明显得指出这个label的名字。这样的方法非常有用，我们可以在一个makefile中定义不用的编译或是和编译无关的命令，比如程序的打包，程序的备份，等等。

### make是如何工作的

在默认的方式下，也就是我们只输入`make` 命令。那么，

1. `make`会在当前目录下找名字叫“Makefile”或“makefile”的文件。

2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“edit”这个文件，并把这个文件作为最终的目标文件。

3. 如果edit文件不存在，或是edit所依赖的后面的 .o 文件的文件修改时间要比 edit 这个文件新，那么，他就会执行后面所定义的命令来生成 edit 这个文件。

4. 如果 edit 所依赖的 .o 文件也不存在，那么make会在当前文件中找目标为 .o 文件的依赖性，如果找到则再根据那一个规则生成 .o 文件。（这有点像一个堆栈的过程）

5. 当然，你的C文件和头文件是存在的啦，于是make会生成 .o 文件，然后再用 .o 文件生成make的终极任务，也就是可执行文件 edit 了。

这就是整个`make`的依赖性，`make`会一层又一层地去找文件的依赖关系，直到最终编译出第一个目标文件。在找寻的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错，而对于所定义的命令的错误，或是编译不成功，make根本不理。`make`只管文件的依赖性，即，如果在我找了依赖关系之后，冒号后面的文件还是不在，那么对不起，我就不工作啦。

通过上述分析，我们知道，像clean这种，没有被第一个目标文件直接或间接关联，那么它后面所定义的命令将不会被自动执行，不过，我们可以显示要make执行。即命令——`make clean` ，以此来清除所有的目标文件，以便重编译。

于是在我们编程中，如果这个工程已被编译过了，当我们修改了其中一个源文件，比如 `file.c` ，那么根据我们的依赖性，我们的目标 `file.o` 会被重编译（也就是在这个依性关系后面所定义的命令），于是 `file.o` 的文件也是最新的啦，于是 `file.o` 的文件修改时间要比`edit`要新，所以 `edit` 也会被重新链接了（详见 edit 目标文件后定义的命令）。

而如果我们改变了`command.h` ，那么， `kdb.o` 、 `command.o` 和 `files.o` 都会被重编译，并且，`edit`会被重链接。

### makefile中使用变量

### 让make自动推导


### makefile的另一种风格


### 清空目录的规则

### Makefile里有什么

## 书写规则


## 书写命令

每条规则中的命令和操作系统Shell的命令行是一致的。`make`会一按顺序一条一条的执行命令，每条命令的开头必须以 `Tab` 键开头，除非，命令是紧跟在依赖规则后面的分号后的。在命令行之间中的空格或是空行会被忽略，但是如果该空格或空行是以Tab键开头的，那么`make`会认为其是一个空命令。

我们在UNIX下可能会使用不同的Shell，但是make的命令默认是被`/bin/sh` ——UNIX的标准Shell 解释执行的。除非你特别指定一个其它的Shell。Makefile中， # 是注释符，很像C/C++中的 `//` ，其后的本行字符都被注释。

### 显示命令

通常，make会把其要执行的命令行在命令执行前输出到屏幕上。当我们用 @ 字符在命令行前，那么，这个命令将不被make显示出来，最具代表性的例子是，我们用这个功能来向屏幕显示一些信息。如:

```makefile
@echo 正在编译XXX模块......
```

当make执行时，会输出“正在编译XXX模块……”字串，但不会输出命令，如果没有“@”，那么，make将输出:

```makefile
echo 正在编译XXX模块......
正在编译XXX模块......
```
如果make执行时，带入make参数 -n 或 --just-print ，那么其只是显示命令，但不会执行命令，这个功能很有利于我们调试我们的Makefile，看看我们书写的命令是执行起来是什么样子的或是什么顺序的。

而make参数 `-s` 或 `--silent` 或 `--quiet` 则是全面禁止命令的显示。


### 命令执行

### 命令出错

### 嵌套执行make

### 定义命令包


## 使用变量

## 使用条件判断


## 使用函数


## make的运行


## 隐含规则


## 使用make更新函数库文件

## 后续












[^1]: [how-to-write-makefile](https://seisman.github.io/how-to-write-makefile/overview.html)
