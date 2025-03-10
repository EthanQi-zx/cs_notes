GDB是一套字符界面的程序集，可以使用命令 gdb 加载要调试的程序。

项目程序如果是为了进行调试而编译时，必须要打开调试选项-g。

-Wall选项表示打开所有的warning；-O0选项表示关掉编译器的优化。例如：

```
gcc arg.c -g -Wall -O0 -o app
```

生成了可用于调试的文件app之后，就可以启动gdb进行调试了：

```
gdb app
```

退出gdb，可以用q或者quit：	

```
(gdb) quit
```

可以给app设置命令行参数1-6：

```
(gdb) set args 1 2 3 4 5 6
```

这其实就等价于app在执行的时候在命令行传入的参数：

```
./app 1 2 3 4 5 6
```

查看设置的参数：

```
(gdb) show args
```

使用start启动可执行程序app，但是只会执行main中的一行：

```
(gdb) start
```

如果想让程序继续执行，需要执行c：

```
(gdb) c 
```

使用run或者简写r也可以启动可执行程序app，它会一直往下执行，直到执行完毕或者遇到断点：

```
(gdb) run
```

使用list或者l查看当前main函数代码信息：(默认情况下只显示10行，如果想要继续显示需要继续敲回车)

```
(gdb) l
```

l后面加上行号可以查询某一行的代码，最终显示的是这一行的上下文，默认也是只有10行代码：

```
(gdb) l 行号
```

l后面加上函数名可以查看某一个函数对应的代码：(默认情况下只显示10行)

```
(gdb) l 函数名
```

还可以查看别的文件某一行或者某个函数的代码：

```
(gdb) l 文件名:行号
(gdb) l 文件名:函数名
```

执行完代码之后，gdb就会切换到这个文件。

可以修改默认显示的行数：

```
(gdb) set listsize 行数
```

listsize可以写成list，即：

```
(gdb) set list 行数
```

修改好之后查看当前list一次显示的行数：

```
(gdb) show list
```



GDB中断点的设置有两种方式：常规断点和条件断点。调试程序的断点可以设置到某个具体的行上，也可以设置到某个函数上。

设置断点使用break和b都行：

```
(gdb) b 行号
(gdb) b 函数名  #停止在函数体的第一行
```

也可以在其他文件(假设为insert.cpp)中设置断点：

```
(gdb) b insert.cpp:行号
(gdb) b insert.cpp:函数名
```

条件断点的设置：

```
(gdb) b 行数 if 条件(比如i==5)
```

查看设置的断点，用info和i都可以：

```
(gdb) i b
```

删除创建的断点，用delete/del/d都行：

```
(gdb) d 断点编号
```

例如：

```
(gdb) d 1
(gdb) d 2 3  #删除编号为2和3的断点
(gdb) d 4-7  #删除编号4~7的断点
```

如果不想删除断点，可以将断点设置为无效，disable和dis等价：

```
(gdb) dis 断点编号
```

例如：

```
(gdb) dis 1
(gdb) dis 2 3
(gdb) dis 4-7
```

设置为无效之后，GDB就不会停在这个断点了。

如果想让断点重新生效，使用enable或者缩写ena：

```
(gdb) ena 断点编号
```

语法格式跟上面的dis完全一样：

```
(gdb) ena 1
(gdb) ena 2 3
(gdb) ena 4-7
```

GDB调试的时候，可以通过print(或者简写为p)来打印变量的值：

```
(gdb) p 变量名
```

如果变量是整形，则默认是以十进制格式进行输出，如果想要以其他格式输出，需要添加参数，例如：

```
(gbd) p i     #默认十进制格式打印i
(gdb) p/x i   #十六进制格式打印i
(gdb) p/o i   #八进制格式打印i
(gdb) p/c i   #以字符形式打印i
(gdb) p/f i   #以浮点数形式打印i
```

可以使用ptype打印变量类型：

```
(gdb) ptype 变量名
```

 display也可以用于调试阶段查看某个变量的值，但是每当程序暂停执行时，display都回重新帮我们打印变量的值，而print不会，因此，当我们想频繁查看某个变量或表达式的值从而观察它的变化情况时，使用display命令可以一劳永逸：

```
(gdb) display 变量名
```

如果变量是整形，可以在display后面执行参数，表示想要以何种格式打印变量，方法同print。

可以使用info display查看自动跟踪了哪些变量：

```
(gdb) info display
```

使用undisplay可以取消相应编号变量的自动跟踪，也是单个、多个、范围的方法：

```
(gdb) undisplay 2
(gdb) undisplay 1 3
(gdb) undisplay 4-7
```

也可以使用delete：

```
(gdb) delete display 2
(gdb) delete display 1 3
(gdb) delete display 4-7
```

如果不想删除也可以使用disable禁用：

```
(gdb) disable display 1
```

使用enable启用：

```
(gbd) enable display 1
```

step(s)和next(n)都可以进行单步调试，二者的区别是：如果当前行是函数的调用，使用step会进入到函数体内部，而使用next会直接跳过当前函数。如果使用step进入函数体内部之后，想要跳出，则需要使用finish命令，跳出的前提是必须要保证函数体的内部不能有有效断点，否则无法跳出。

通过until命令可以直接跳出某个循环体(即直接让循环体一次性执行完)，如果要想跳出某个循环体，必须也要满足循环体内部不能有有效的断点，且要在循环体的开始或者结束处，执行until命令。

GDB调试时可以使用set设置某个变量的值来进行调试：

```
(gdb) set var 变量名=值
(gdb) set var i=5
```

