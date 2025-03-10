# Makefile介绍

当工程中的文件逐渐增多的时候，使用gcc命令编译就会力不从心，这时需要借助make工具帮助我们完成这个艰巨的任务。

make工具在构造项目的时候需要加载一个叫做makefile的文件，Makefile定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要重新编译等。

makefile带来的好处是--“自动化编译”，一旦写好，只需要一个make命令，整个工程完全自动编译，极大提高了软件开发的效率。

makefile文件有两种命名方式：makefile和Makefile。

在哪个目录下执行构建命令make，这个目录下的makefile文件就会被加载。

# Makefile的规则

makefile中每条规则由三个部分组成：

1️⃣目标(target)：用规则生成的目标。（可以理解成目标文件？）

2️⃣依赖(depend)：表示规则所必需的依赖条件，在规则的命令中可以使用这些依赖。（可以理解成源文件？）

3️⃣命令(command)：表示当前这条规则的动作，一般情况下这个动作就是一个shell命令。每个命令(command)前必须有一个Tab缩进并且独占一行。

语法格式为：

```
target1,target2... : depend1,depend2...
    command
    ...
    ...
```

例1，有源文件a.c，b.c，c.c，和head.h，需要生成可执行程序app：

```
app:a.c b.c c.c
	gcc a.c b.c c.c -o app
```

例2，多个目标，多个依赖，多个命令：

```
app,app1:a.c b.c c.c d.c
	gcc a.c b.c -o app
	gcc c.c d.c -o app1
```

例3，规则之间的嵌套，即可以允许有多条规则，但是一般情况下，从第二条一直到最后一条规则都是为第一条规则服务的。

```
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
a.o:a.c
	gcc -c a.c
b.o:b.c
	gcc -c b.c
c.o:c.c
	gcc -c c.c
```

# Makefile的工作原理

执行make命令之后，首先会去当前目录里面查询是否有makefile文件，如果没有make命令就会罢工。如果有makefile文件，make命令就会去执行其中的第一条规则，如果第一条规则中的依赖项没有被生成，就继续往下执行规则，直到第一条规则内的所有依赖项都被生成，然后第一条命令就可以基于这些依赖项生成对应的目标，make的任务就完成了。

make命令执行的时候也会根据文件的时间戳来判定是否执行makefile中的相关规则的命令，例如：

```
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
```

如果目标app的时间戳大于所有依赖项的时间戳，即app在所有依赖项之后生成，就不会执行命令；如果目标app的时间戳小于部分依赖项的时间戳，此时目标文件会通过规则中的命令重新生成。如果规则中的目标文件不存在，就一定会执行命令。

make命令还有自动推到的功能，例如：

```
calc:add.o div.o main.o mult.o sub.o
	gcc add.o div.o main.o mult.o sub.o -o -clac
```

即使makefile文件没有生成.o文件的后续规则，通过make执行的时候也不会报错，make命令会自动帮我们添加上，只要我们当前目录下有相关.c文件即可。

# Makefile中三中类型的变量

使用makefile进行规则定义的时候，为了写起来更加灵活，可以在里面使用变量，makefile中的变量没有类型；

makefile中的变量分为三种：自定义变量，预定义变量和自动变量。

## 自定义变量

自定义变量在定义的时候必需给其赋值，不然会报错：

```
变量名=变量值
```

定义好了之后使用$()取出变量的值，例如：

```
obj=add.o div.o main.o mult.o sub.o
$(obj)   #取出变量的值
```

## 预定义变量

预定义变量是值makefile已经给我们定义好的一些变量。常见的预定义变量如下：

```
CC: 指定要使用的C编译器的名称，默认是cc
CXX: 指定C++编译器的名称，默认是g++
CFLAGS：表示C编译器的编译选项，无默认值，例如-O3，-g等都是编译选项
```

-O0,-O1,-O2,-O3是代码优化的四个级别，其中-O3是优化级别最高的：

```
CFLAGS=-O3
obj=add.0 div.o main.o
target=calc
$(target):$(obj)
	$(CC) $(obj) -o $(target) $(CFLAGS)
```

## 自动变量

自动变量只能在规则的命令中使用，常见的自动变量如下：

```
$*	表示目标文件的名称，不包含文件扩展名
$@  表示目标文件的名称，包含文件扩展名
$^  表示依赖项中所有不重复的文件，这些文件之间以空格分开
$+  表示依赖项中的所有文件，可能会重复，这些文件之间以空格分开
$<  表示依赖项中第一个依赖文件的名称
```

例如：

```
calc:add.o div.o main.o sub.o
	gcc $^ -o $@
```

# 模式匹配

先看一个例子：

```
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
a.o:a.c
	gcc -c a.c
b.o:b.c
	gcc -c b.c
c.o:c.c
	gcc -c c.c
```

每生成一个.o文件都要写一条规则，太过冗余，可以使用模式匹配来完成：

```
app:a.o b.o c.o
	gcc a.o b.o c.o -o app
%.o:%.c
	gcc $< -c
```

通配符%匹配的是文件名称，实时发生变化，所以命令中依赖项的名称必须要使用自动变量。

# Makefile中的函数

makefile中所有的函数都是有返回值的，其格式为：

```
$(函数名 参数1,参数2,参数3, ....)
```

## wildcard函数

这个函数的作用主要是获取指定目录下类型的文件名，其返回值是以空格分割的。函数格式为：

```
$(wildcard PATTERN...)
```

可以同时搜索多个路径下的文件名称，例如：

```
$(wildcard *.c ./sub/*.c)
```

返回值的格式为：

```
a.c b.c c.c ./sub/aa.c ./sub/bb.c
```

## patsubst

函数的功能是按照指定的模式替换指定的文件名的后缀，函数格式为：

```
$(patsubst <pattern>,<replacement>,<text>)
```

例如把变量src中的所有文件名的后缀从.cpp替换为.o:

```
src=a.cpp b.cpp c.cpp e.cpp
obj=$(patsubst %.cpp,%.o,$(src))
```

最终obj的值为：a.o，b.o，c.o，e.o

# makefile中伪目标的声明

举例如下：

```
target=calc
src=$(wildcard *.c)
obj=$(patsubst %.c, %.o, $(src))
$(target):$(obj)
	gcc $(obj) -o $(target)
%.o:%.c
	gcc -c $^
	
clean:
	rm $(obj) $(target)
```

想让make执行对应的clean中的命令，需要将clean声明为伪目标，声明方式为：

```
.PHONY:伪文件名称
```

在clean上面添加一行代码即可：

```
.PHONY:clean
clean:
	rm $(obj) $(target)
```

声明为伪目标之后make就会忽略时间戳。

然后就可以使用make clean命令执行了。

```
.PHONY:clean
clean:
	-mkdir /abc
	rm $(obj) $(target)
```

-mkdir /abc命令前加一个-，表示即使命令没有执行成功，也会继续往下执行下面的命令。