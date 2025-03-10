# 第1章、部署一台linux操作系统

本笔记以RHEL8(Red Hat Enterprise Linux 8)为例。

## 1.4、linux系统中如何安装软件

最早，想在 Linux 系统中安装软件，只能采取编译源码包的方式；而且大多数的服务程序仅仅提供自身的源代码，还需要运维人员编译代码后自行解决软件之间的依赖关系。

随之，RPM(红帽软件包管理器)诞生，RPM 有点像Windows 系统中的控制面板，会建立统一的数据库，详细记录软件信息并能够自动分析依赖关系。常用RPM软件包命令为：

```c++
rpm -ivh filename.rpm 安装软件
rpm -Uvh filename.rpm 升级软件
rpm -e filename.rpm 卸载软件
rpm -qpi filename.rpm 查询软件描述信息
rpm -qpl filename.rpm 列出软件文件信息
rpm -qf filename 查询文件属于哪个 RPM
```

尽管 RPM 能够帮助用户查询软件之间的依赖关系，但问题还是要运维人员自己来解决，而有些大型软件可能与数十个程序都有依赖关系，在这种情况下安装软件依然很繁琐。

此时，Yum软件仓库应运而生，进一步降低了软件安装难度和复杂度。Yum 软件仓库可以根据用户的要求分析出所需软件包及其相关的依赖关系，然后自动从服务器下载软件包并安装到系统。常见的Yum命令如下：

```
yum repolist all 列出所有仓库
yum list all 列出仓库中所有软件包
yum info 软件包名称 查看软件包信息
yum install 软件包名称 安装软件包
yum reinstall 软件包名称 重新安装软件包
yum update 软件包名称 升级软件包
yum remove 软件包名称 移除软件包
yum clean all 清除所有仓库缓存
yum check-update 检查可更新的软件包
yum grouplist 查看系统中已经安装的软件包组
yum groupinstall 软件包组 安装指定的软件包组
yum groupremove 软件包组 移除指定的软件包组
yum groupinfo 软件包组 查询指定的软件包组信息
```

Yum 虽然解决了软件的依赖关系问题，但仍然还是存在分析不准确、内存占用量大、不能多人同时安装软件等硬伤。

于是在2015年红帽又给了我们一个新选择——DNF。DNF 实际上就是解决了上述问题的 Yum 软件仓库的提升版，行业内称之为 Yum v4 版本。且DNF的命令跟Yum格式一模一样，只需要把yum关键字替换成dnf既可。

## 1.5、系统初始化进程

1、Linux系统的开机过程：
1️⃣先从BIOS开始
2️⃣然后进入Boot Loader
3️⃣再加载系统内核
4️⃣内核进行初始化
5️⃣最后启动初始化进程

2、执行命令 “systemctl status 服务名” 可以查看某个服务的运行状态。

## 1.6、重置root密码

1️⃣首先先查看Linux系统是否是RHEL8：

```
[root@linuxprobe~]# cat /etc/redhat-release
```

2️⃣重启linux系统，进入引导界面的时候按e进入内核编辑界面；

3️⃣在 linux 参数这行的最后面追加 rd.break 参数，然后按下 Ctrl + X 组合键运行修改过的内核程序，大约 30 秒过后，系统会进入紧急救援模式；

4️⃣然后依次输入以下命令：

```
switch_root:/# mount -o remount,rw /sysroot
switch_root:/# chroot /sysroot
sh-4.4# passwd
sh-4.4# touch /.autorelabel
```

5️⃣命令执行结束之后连续按两次ctrl+D退出并重启。等重启完毕之后即可使用新密码登录。



# 第2章、新手必须掌握的Linux命令

## 2.1、强大好用的shell

1、Linux 系统的内核负责完成对硬件资源的分配、调度等管理任务，对系统的正常运行起着十分重要的作用。

2、用户不太好直接调用内核控制硬件，所以有如下层次：

用户⇔服务程序⇔系统调用接口⇔ 内核⇔硬件。

服务程序负责将用户提出的需求转换成硬件能够接收的指令代码，然后再将处理结果反馈成用户能够读懂的内容格式。这样一来一回，用户就能使用硬件资源了。

硬件被内核、系统调用接口、服务程序一层层包裹起来，看起来就像蜗牛的壳(Shell)，于是我们将用户终端程序称之为 Shell。

Shell 就是终端程序的统称，它充当了人与内核（硬件）之间的翻译官，用户把一些命令“告诉”终端程序，它就会调用相应的程序服务去完成某些工作。

3、现在包括红帽系统在内的许多主流 Linux 系统默认使用的终端是 Bash（Bourne-Again SHell）解释器，这个 Bash 解释器主要有以下 4 项优势：
1️⃣通过上下方向键来调取执行过的 Linux 命令；
2️⃣命令或参数仅需输入前几位就可以用 Tab 键补全；
3️⃣具有强大的批处理脚本；
4️⃣具有实用的环境变量功能。

## 2.2、执行命令的必备知识

1、常见的执行Linux命令的格式如下：

```
命令名称  [命令参数]  命令对象
```

命令名称：就是语法中的“动词”，表达的是想要做的事情，例如创建用户、查看文件、重启系统等操作。

命令参数：用于对命令进行调整，让“修改”过的命令能更好地贴合工作需求，达到事半功倍的效果。参数可以使用长格式(--)和短格式(-)，分别表示使用完整名称和单个字母缩写名称。

```
man --help 长格式
man -h     短格式
```

命令对象：一般指要处理的文件、目录、用户等资源名称，也就是命令执行后的“承受方”.

命令名称、命令参数与命令对象之间要用空格进行分隔，且字母严格区分大小写。

命令行中，我们约定将可选择的、可加或可不加的、非必需的参数使用中括号引起来。

2、man命令可以查询查询某个命令的帮助信息

```
[root@linuxprobe～]# man man 查询man命令的帮助信息
[root@linuxprobe～]# man cd  查询cd命令的帮助信息
```

man命令执行之后会出现很多信息，出现信息后我们可以执行的操作有：

```
空格键 向下翻一页
PaGe down 向下翻一页
PaGe up 向上翻一页
home 直接前往首页
end 直接前往尾页
/ 从上至下搜索某个关键词，如“/linux”
? 从下至上搜索某个关键词，如“?linux”
n 定位到下一个搜索到的关键词
N 定位到上一个搜索到的关键词
q 退出帮助文档
```

在输入命令前就已经存在的“[root@linuxprobe～]#”这部分内容是终端提示符，它用于向用户展示一些基本的信息—当前登录用户名为 root，简要的主机名是 linuxprobe，所在目录是～（这里的～是指用户家目录)，#表示管理员身份，$表示普通用户(相应权限会变低)。

3、bash终端中四个快捷键/组合键小技巧

1️⃣Tab键：能够实现对命令、参数或文件的内容补全。

2️⃣Ctrl+C组合键：当同时按下键盘上的 Ctrl 和字母 C 的时候，意味着终止当前进程的运行。假如执行了一个错误命令，或者是执行某个命令后迟迟无法结束，这时就可以冷静地按下 Ctrl+C 组合键，命令行终端的控制权会立刻回到我们手中。(在终端中复制是ctrl+shift+c)

3️⃣Ctrl+D组合键：当同时按下键盘上的 Ctrl 和字母 D 的时候，表示键盘输入结束。(草，我试了下终端直接给我干没了，搜了下“Ctrl+D 键组合代表输入结束或退出。”)

4️⃣Ctrl+I组合键：当同时按下键盘上行的 Ctrl 和字母 l 的时候，会清空当前终端中已有的内容。（我试了下，不能清屏，搜了下，跟tab功能一样，是补全功能。清空还是用clear吧）

## 2.3、常用系统工作命令

### 1、echo命令

echo命令用于在终端设备上输出字符串或者变量提取后的值，语法格式为：

```
echo [字符串] [$变量]
```

它的操作却非常简单，执行“echo 字符串”或“echo $变量”就行，其中$符号的意思是提取变量的实际值，以便后续的输出操作。

例如，把指定字符串“LinuxProbe.com”输出到终端屏幕的命令为：

```
[root@linuxprobe~]# echo LinuxProbe.com
```

该命令会在终端屏幕上显示如下信息：

```
LinuxProbe.com
```

使用“$变量”的方式提取出变量 SHELL 的值，并将其输出到屏幕上：

```
[root@linuxprobe~]# echo $SHELL
/bin/bash
```

### 2、date命令

date 命令用于显示或设置系统的时间与日期，语法格式为：

```
date [+指定的格式]
```

date命令中的参数及其作用：

```
常用的：
%S 秒（00～59）
%M 分钟（00～59）
%H 小时（00～23）24小时制
%m 月份（1～12）
%Y 完整年份（例如，2020）
%d 本月中的第几天

不常用的：
%j 今年中的第几天
%n 换行符（相当于按下回车键）
%t 跳格（相当于按下 Tab 键）
%I 小时（00～12）12小时制
%p 显示出 AM 或 PM
%a 缩写的工作日名称（例如，Sun）
%A 完整的工作日名称（例如，Sunday）
%b 缩写的月份名称（例如，Jan）
%B 完整的月份名称（例如，January）
%q 季度（1～4）
%y 简写年份（例如，20）
```

如果不指定格式，就是默认格式查看系统时间：

```
[root@linuxprobe~]# date
Wed Sep 11 12:14:23 CST 2024
```

按照“年-月-日 小时:分钟:秒”的格式查看当前系统时间的date 命令如下所示：注意加号(+)不要忘记！

```
[root@linuxprobe~]# date "+%Y-%m-%d %H:%M:%S"
2024-09-11 12:20:13
```

使用-s参数将系统的当前时间设置为 2020 年 11 月 1 日 8 点 30 分的 date 命令如下所示：

```
[root@linuxprobe~]# date -s "20201101 8:30:00"
Sun Nov 1 08:30:00 CST 2020
```

### 3、timedatectl命令

timedatectl 命令用于设置系统的时间，英文全称为“time date control”，语法格式为：

```
timedatectl [参数]
```

timedatectl 命令中常见的参数格式及作用如下：

```
status 显示状态信息
list-timezones 列出已知时区
set-time 设置系统时间
set-timezone 设置生效时区
```

可以使用status关键字查看系统时间与时区：

```
[root@linuxprobe ~]# timedatectl status
               Local time: Wed 2024-09-11 12:26:03 CST
           Universal time: Wed 2024-09-11 04:26:03 UTC
                 RTC time: Wed 2024-09-11 04:25:59
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: no
              NTP service: inactive
          RTC in local TZ: no
```

可以使用set-timezone关键字手动设置时区：

```
[root@linuxprobe~]# timedatectl set-timezone Asia/Shanghai
```

如果不知道时区名字，可以使用list-timezones关键字查询：

```
[root@linuxprobe~]# timedatectl list-timezones
```

可以使用set-time关键字手动修改系统日期和时间：

```
[root@linuxprobe~]# timedatectl set-time 2024-09-11

[root@linuxprobe~]# timedatectl set-time 12:30
```

### 4、reboot命令

reboot 命令用于重启系统，输入该命令后按回车键执行即可。最好是以 root 管理员的身份来重启，普通用户在执行该命令时可能会被拒绝。命令如下：

```
[root@linuxprobe~]# reboot
```

### 5、poweroff命令

poweroff 命令用于关闭系统，输入该命令后按回车键执行即可。同样最好也是root管理员身份执行。

```
[root@linuxprobe~]# poweroff
```

### 6、wget命令

wget 命令用于在终端命令行中下载网络文件，英文全称为web get，语法格式为：

```
wget [参数] 网址
```

借助wget命令，可以无需打开浏览器，直接在命令行就可以下载文件。wget 命令中的参数以及作用如下：

```
-b 		后台下载模式
-P 		下载到指定目录
-t 		最大尝试次数
-c 		断点续传
-p 		下载页面内所有资源，包括图片、视频等
-r 		递归下载
```

尝试用wget从站点中下载书本LinuxProbe.pdf

```
[root@linuxprobe~]# wget https://www.linuxprobe.com/docs/LinuxProbe.pdf
```

尝试使用wget递归命令下载www.linuxprobe.com网站内的所有页面数据及文件，下载完后会自动保存到当前路径下一个名为 www.linuxprobe.com 的目录中。

```
[root@linuxprobe~]# wget -r -p https://www.linuxprobe.com
```

### 7、ps命令

ps 命令用于查看系统中的进程状态，英文全称为“processes”，语法格式为：

```
ps [参数]
```

ps 命令的常见参数以及作用如下：

```
-a 		显示所有进程（包括其他用户的进程）
-u 		用户以及其他详细信息
-x 		显示没有控制终端的进程
```

 Linux 系统中有 5 种常见的进程状态：

1️⃣R（运行）：进程正在运行或在运行队列中等待。

2️⃣S（中断）：进程处于休眠中，当某个条件形成后或者接收到信号时，则脱离该状态。

3️⃣D（不可中断）：进程不响应系统异步信号，即便用 kill 命令也不能将其中断。

4️⃣Z（僵死）：进程已经终止，但进程描述符依然存在, 直到父进程调用 wait4()系统函数后将进程释放。

5️⃣T（停止）：进程收到停止信号后停止运行。

在 Linux 系统中的命令参数有长短格式之分，长格式和长格式之间不能合并，长格式和短格式之间也不能合并，但短格式和短格式之间是可以合并的，合并后仅保留一个减号（-）即可。另外 ps 命令可允许参数省略减号（-），因此可直接写成 ps aux 的样子。

### 8、pstree命令

pstree 命令用于以树状图的形式展示进程之间的关系，英文全称为“process tree”，输入该命令后按回车键执行即可。

```
[root@linuxprobe~]# pstree
```

### 9、top命令

top 命令用于动态地监视进程活动及系统负载等信息，输入该命令后按回车键执行即可。可以将它看作是 Linux 中“强化版的Windows 任务管理器”。

```
[root@linuxprobe~]# top
```

### 10、nice命令

nice 命令用于调整进程的优先级，语法格式为：

```c++
nice 优先级数字 服务名称
```

数字越低，优先级越高，数字的取值范围是-20~19。

例如将bash服务优先级调整到最高：

```
[root@linuxprobe~]# nice -n -20 bash
```

### 11、pidof命令

pidof 命令用于查询某个指定服务进程的 PID 号码值(一个服务进程坑对应好几个PID号码值)，语法格式为：

```
pidof [参数] 服务名称
```

每个进程的进程号码值（PID）是唯一的，可以用于区分不同的进程。例如查询本机上sshd服务的PID：

```
[root@linuxprobe~]# pidof sshd
1182
```

### 12、kill命令

kill 命令用于终止某个指定 PID 值的服务进程，语法格式为：

```
kill [参数] PID值
```

例如我们要终止PID值为1182的进程：

```
[root@linuxprobe~]# kill 1182
```

有时候系统可能会提示无法终止，此时可以添加参数-9，表示最高级别的强制终止进程；

```
[root@linuxprobe~]# kill -9 1182
```

### 13、killall命令

killall 命令用于终止某个指定名称的服务所对应的全部进程，语法格式为：

```
killall [参数] 服务名称
```

通常来讲，复杂软件的服务程序会有多个进程协同为用户提供服务，如果用 kill 命令逐个去结束这些进程会比较麻烦，此时可以使用 killall 命令来批量结束某个服务程序带有的全部进程。以httpd服务程序为例：

```
[root@linuxprobe~]# pidof httpd
13581 13580 13579 13578 13577 13576
[root@linuxprobe~]# killall httpd
[root@linuxprobe~]# pidof httpd
[root@linuxprobe~]#
```

当你在执行一个命令时，在命令的末尾添加 ‘’&‘’ 符号会使该命令在后台运行，而不会阻塞当前终端，从而允许你继续输入其他命令。这个过程被称为将命令置于后台执行。

## 2.4、系统状态检测命令

### 1、ifconfig命令

ifconfig 命令用于获取网卡配置与网络状态等信息，英文全称为“interface config”，语法格式为：

```
ifconfig [参数] [网络设备]
```

使用举例：

```
[root@linuxprobe~]# ifconfig
```

### 2、uname命令

uname 命令用于查看系统内核版本与系统架构等信息，英文全称为“unix name”，语法格式为：

```
uname [-a]
```

在使用 uname 命令时，一般要固定搭配上-a 参数来完整地查看当前系统的内核名称、主机名、内核发行版本、节点名、压制时间、硬件名称、硬件平台、处理器类型以及操作系统名称等信息.

```
[root@linuxprobe~]# uname -a
Linux linuxprobe.com 4.18.0-80.el8.x86_64 #1 SMP Wed Mar 13 12:02:46 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

### 3、uptime命令

uptime 命令用于查看系统的负载信息，输入该命令后按回车键执行即可。

```
[root@linuxprobe~]# uptime
 17:03:10 up  4:53,  1 user,  load average: 0.00, 0.00, 0.00
```

### 4、free命令

free 命令用于显示当前系统中内存的使用量信息，语法格式为：

```
free [-h]
```

在使用 free 命令时，可以结合使用-h 参数以更人性化的方式输出当前内存的实时使用量信息，不然如果服务器有几百 GB 的内存，则换算下来就会是一大长串的数字，真不利于阅读。

### 5、who命令

who 命令用于查看当前登入主机的用户终端信息，输入该命令后按回车键执行即可。

```
[root@linuxprobe~]# who
root     tty2         2024-09-12 10:06 (tty2)
```

三条信息从左到右分别表示：登录的用户名、终端设备、登录到系统的时间

### 6、last命令

last 命令用于调取主机的被访记录，输入该命令后按回车键执行即可。

```
[root@linuxprobe~]# last
```

### 7、ping命令

ping 命令用于测试主机之间的网络连通性，语法格式为：

```
ping [参数] 主机地址
```

执行 ping 命令时，系统会使用 ICMP 向远端主机发出要求回应的信息，若连接远端主机的网络没有问题，远端主机会回应该信息。因此，ping 命令可用于判断远端主机是否在线并且网络是否正常。ping 命令的常见参数以及作用如下：

```
-c 		总共发送次数
-l 		指定网卡名称
-I 		每次间隔时间（秒）
-W 		最长等待时间（秒）
```

我们用ping测试IP为192.168.10.10的主机：

```
[root@linuxprobe~]# ping -c 4 192.168.10.10
```

这里的-c 4告诉 ping 命令发送4个回显请求。发送这些请求后，ping命令将显示有关每个请求的回显应答的统计信息。

### 8、tracepath命令

tracepath 命令用于显示数据包到达目的主机时途中经过的所有路由信息，语法格式为：

```
tracepath [参数] 域名
```

当两台主机之间无法正常 ping 通时，要考虑两台主机之间是否有错误的路由信息，导致数据被某一台设备错误地丢弃。这时便可以使用 tracepath 命令追踪数据包到达目的主机时途中的所有路由信息，以分析是哪台设备出了问题。

```
[root@linuxprobe~]# tracepath www.linuxprobe.com
```

### 9、netstat命令

netstat 命令用于显示如网络连接、路由表、接口状态等的网络相关信息，英文全称为“network status”，语法格式为：

```
netstat [参数]
```

netstat 命令的常见参数以及作用如下：

```
-a   显示所有连接中的 Socket(接口？)
-p   显示正在使用的 Socket 信息
-t   显示 TCP 协议的连接状态
-u   显示 UDP 协议的连接状态
-n   使用 IP 地址，不使用域名
-l   仅列出正在监听的服务状态
-i   现在网卡列表信息
-r   显示路由表信息
```

使用 netstat 命令显示详细的网络状况：

```
[root@linuxprobe~]# netstat -a
```

使用 netstat 命令显示网卡列表：

```
[root@linuxrpobe~]# netstat -i
```

### 10、history命令

history 命令用于显示执行过的命令历史，语法格式为：

```
history [-c]
```

执行 history 命令能显示出当前用户在本地计算机中执行过的最近 1000 条命令记录。

可以使用"!数字"来执行某次命令：

```
[root@linuxprobe~]# history
1 ifconfig
2 uname -a
3 cat /etc/redhat-release
4 uptime
……

[root@linuxprobe~]# !3
cat /etc/redhat-release
Red Hat Enterprise Linux release 8.0 (Ootpa)
```

历史命令会被保存到用户家目录中的.bash_history 文件中。Linux 系统中以点（.）开头的文件均代表隐藏文件，这些文件大多数为系统服务文件，可以用 cat 命令查看其文件内容：

```
[root@linuxprobe~]# cat ~/.bash_history
```

在使用 history 命令时，可以使用-c 参数清空所有的命令历史记录。

```
[root@linuxprobe~]# history -c
```

### 11、sosreport命令

sosreport 命令用于收集系统配置及架构信息并输出诊断文档，输入该命令后按回车键执行即可。

当 Linux 系统出现故障需要联系技术支持人员时，大多数时候都要先使用这个命令来简单收集系统的运行状态和服务配置信息。

```
[root@linuxprobe~]# sosreport
```

## 2.5、查找定位文件命令

### 1、pwd命令

pwd 命令用于显示用户当前所处的工作目录，英文全称为“print working directory”，输入该命令后按回车键执行即可。

```
[root@linuxprobe etc]# pwd
/etc
```

### 2、cd命令

cd 命令用于切换当前的工作路径，英文全称为“change directory”，语法格式为：

```
cd [参数] [目录]
```

cd的其他特殊用法：注意cd后有个空格

```
cd -   返回到上一次所处的目录
cd ..  进入上级目录
cd ~   切换到当前用户的家目录
cd ~ username 切换到其他用户的家目录
```

例如，使用cd命令切换进/etc目录中

```
[root@linuxprobe~]# cd /etc
[root@linuxprobe etc]#
```

此时[]里面的命令提示符也发生了改变。

### 3、ls命令

ls 命令用于显示目录中的文件信息，英文全称为“list”，语法格式为：

```
ls [参数] [文件名称]
```

使用 ls 命令的-a 参数可以看到全部文件（包括隐藏文件）。
使用-l 参数可以查看文件的属性、大小等详细信息。将这两个参数整合之后，再执行 ls 命令即可查看当前目录中的所有文件并输出这些文件的属性信息：

```
[root@linuxprobe~]# ls -al
```

如果想要查看目录属性信息，则需要额外添加一个-d 参数。例如，可使用如下命令查看/etc 目录的权限与属性信息：

```
[root@linuxprobe~]# ls -ld /etc
```

### 4、tree命令

tree 命令用于以树状图的形式列出目录内容及结构，输入该命令后按回车键执行即可。

```
[root@linuxprobe~]# tree
```

### 5、find命令

find 命令用于按照指定条件来查找文件所对应的位置，语法格式为：

```
find [查找范围] 寻找条件
```

“Linux系统中的一切都是文件。”

find命令的参数以及作用如下：

```
-name    匹配名称
-perm    匹配权限（mode 为完全匹配，-mode 为包含即可）
-user    匹配所有者
-group    匹配所属组
-mtime -n +n    匹配修改内容的时间（-n 指 n 天以内，+n 指 n 天以前）
-atime -n +n    匹配访问文件的时间（-n 指 n 天以内，+n 指 n 天以前）
-ctime -n +n    匹配修改文件权限的时间（-n 指 n 天以内，+n 指 n 天以前）
-nouser    匹配无所有者的文件
-nogroup    匹配无所属组的文件
-newer f1 !f2    匹配比文件 f1 新但比 f2 旧的文件
--type b/d/c/p/l/f    匹配文件类型（后面的字母依次表示块设备、目录、字符设备、管道、链接
文件、文本文件）
-size    匹配文件的大小（+50KB为查找超过50KB的文件，而-50KB为查找小于50KB
的文件）
-prune    忽略某个目录
-exec…… {}\;    后面可跟用于进一步处理搜索结果的命令（下文会有演示）
```

-exec参数用于把 find 命令搜索到的结果交由紧随其后的命令作进一步处理。

想要获得/etc目录中以host开头的文件列表，可以执行：

```
[root@linuxprobe~]# find /etc -name "host*"
```

想要在整个系统中搜索权限中包括SUID权限的所有文件，只需使用-4000即可：

```
[root@linuxprobe~]# find / -perm -4000 -print
```

### 6、locate命令

locate 命令用于按照名称快速搜索文件所对应的位置，语法格式为：

```
locate 文件名称
```

使用 find 命令进行全盘搜索虽然更准确，但是效率有点低。如果仅仅是想找一些常见的且又知道大概名称的文件，不如试试 locate 命令。

在使用locate命令前，先使用updatedb命令生成一个索引库文件，然后再使用locate进行时就是在该库中进行查找操作，速度会快很多。

```
[root@linuxprobe~]# updatedb
[root@linuxprobe~]# locate whereis
```

使用 locate 命令搜索出所有包含“whereis”名称的文件所在的位置。

### 7、whereis命令

whereis 命令用于按照名称快速搜索二进制程序（命令）、源代码以及帮助文件所对应的位置，语法格式为：

```
whereis 命令名称
```

whereis 命令也是基于 updatedb 命令所生成的索引库文件进行搜索，它与 locate命令的区别是不关心那些相同名称的文件，仅仅是快速找到对应的命令文件及其帮助文件所在的位置。

使用whereis命令分别查找ls和pwd命令所在的位置：

```
[root@linuxprobe~]# whereis pwd
[root@linuxprobe~]# whereis ls
```

### 8、which命令

which 命令用于按照指定名称快速搜索二进制程序（命令）所对应的位置，语法格式为：

```
which 命令名称
```

如果我们既不关心同名文件（find 与 locate），也不关心命令所对应的源代码和帮助文件（whereis），仅仅是想找到命令本身所在的路径，使用which比较合适。

查找locate和whereis命令所对应的路径：

```
[root@linuxprobe~]# which locate
/usr/bin/locate
[root@linuxprobe~]# which whereis
/usr/bin/whereis
```

## 2.6、文本文件编辑命令

### 1、cat命令

cat 命令用于查看纯文本文件（内容较少的），英文全称为“concatenate”，语法格式为：

```
cat [参数] 文件名称
```

如果想在查看文件内容顺便显式行号时，可以在cat命令后面加一个-n参数；

```
[root@linuxprobe~]# cat -n initial-setup-ks.cfg
```

### 2、more命令

more 命令用于查看纯文本文件（内容较多的），语法格式为：

```
more [参数] 文件名称
```

more命令会在最下面使用百分比的形式来提示您已经阅读了多少内容；还可以使用空格键或回车键向下翻页

```
[root@linuxprobe~]# more initial-setup-ks.cfg
```

### 3、head命令

head 命令用于查看纯文本文件的前 *N* 行，语法格式为：

```
head [参数] 文件名称
```

如果只想查看文本中前10行的内容：

```
[root@linuxprobe~]# head -n 10 initial-setup-ks.cfg
```

### 4、tail命令

tail 命令用于查看纯文本文件的后 *N* 行或持续刷新文件的最新内容，语法格式为：

```
tail [参数] 文件名称
```

例如我们想查看文本内容的最后10行：

```
[root@linuxprobe~]# tail -n 10 initial-setup-ks.cfg
```

tail 命令最强悍的功能是实时查看最新的文件内容，比如日志文件，此时的命令格式为 “ tail -f 文件名称 ”：

```
[root@linuxprobe~]# tail -f /var/log/messages
```

### 5、tr命令

tr 命令用于替换文本内容中的字符，英文全称为“translate”，语法格式为：

```
tr [原始字符] [目标字符]
```

如果需要处理大批量的内容，进行手工替换不太现实，这时，就可以先使用 cat 命令读取待处理的文本，然后通过管道符把这些文本内容传递给 tr 命令进行替换操作即可。例如，把某个文本内容中的小写字母全部替换成大写：

```
[root@linuxprobe~]# cat anaconda-ks.cfg | tr [a-z] [A-Z]
```

### 6、wc命令

wc 命令用于统计指定文本文件的行数、字数或字节数，英文全称为“word counts”，语法格式为:

```
wc [参数] 文件名称
```

wc 命令中的参数以及作用如下：

```
-l   只显示行数
-w   只显示单词数
-c   只显示字节数
```

在 Linux 系统中，/etc/passwd 是用于保存所有用户信息的文件，要统计当前系统中有多少个用户，可以使用下面的命令来进行查询：

```
[root@linuxprobe~]# wc -l /etc/passwd
45 /etc/passwd
```

### 7、stat命令

stat 命令用于查看文件的具体存储细节和时间等信息，英文全称为“status”，语法格式为：

```
stat 文件名称
```

Linux 系统中的文件包含 3 种时间状态：
1️⃣Access Time（内容最后一次被访问的时间，简称为 Atime）
2️⃣Modify Time（内容最后一次被修改的时间，简称为 Mtime）
3️⃣Change Time（文件属性最后一次被修改的时间，简称为 Ctime）

使用 stat 命令可以查看文件的这 3 种时间状态信息：

```
[root@linuxprobe ~]# stat anaconda-ks.cfg
```

### 8、grep命令

grep 命令用于按行提取文本内容，允许你在文件中搜索特定模式并输出包含该模式的行，这里的模式指的是文本字符串或者正则表达式，允许你更灵活地搜索。语法格式为：

```
grep [参数] 模式 文件名称
```

grep 命令中的参数及其作用：

```
-b    将可执行文件（binary）当作文本文件（text）来搜索   
-c    仅显示找到的行数
-I    忽略大小写
-n    显示行号
-v    反向选择—仅列出没有“关键词”的行
```

最常用的两个参数是-n和-v

查询anaconda-ks.cfg文件中包含字符串"disk"的文件行，并显示行号：

```
[root@linuxprobe ~]# grep -n disk anaconda-ks.cfg
2:ignoredisk --only-use=sda
```

在 Linux 系统中，/etc/passwd 文件保存着所有的用户信息，而一旦用户的登录终端被设置成/sbin/nologin，则不再允许登录系统，因此可以使用 grep 命令查找出当前系统中不允许登录系统的所有用户的信息：

```
[root@linuxprobe~]# grep /sbin/nologin /etc/passwd
```

这行代码的意思是查询/etc/passwd文件中包含字符串"/sbin/nologin"的行。

### 9、cut命令

cut 命令用于按“列”提取文本内容，语法格式为：

```
cut [参数] 文件名称
```

系统文件在保存用户数据信息时，每一项值之间是采用冒号来间隔的，可以先用head命令查看passwd文件的前两行内容：

```
[root@linuxprobe~]# head -n 2 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
```

如果按“列”搜索，使用-d 参数来设置间隔符号，使用-f 参数设置需要查看的列数，例如提取出passwd文件中以冒号(:)为间隔的第一列：

```
[root@linuxprobe~]# cut -d : -f 1 /etc/passwd
```

### 10、diff命令

diff 命令用于比较多个文件之间内容的差异，英文全称为“different”，语法格式为：

```
diff [参数] 文件名A 文件名B
```

在使用 diff 命令时，可以使用--brief 参数来确认两个文件是否相同:

```
[root@linuxprobe~]# diff --brief diff_A.txt diff_B.txt
Files diff_A.txt and diff_B.txt differ
```

使用带有-c 参数的 diff 命令来描述文件内容具体的不同：

```
[root@linuxprobe~]# diff -c diff_A.txt diff_B.txt
```

### 11、uniq命令

uniq 命令用于去除文本中连续的重复行，英文全称为“unique”，语法格式为：

```
uniq [参数] 文件名称
```

uniq命令的作用是用来去除文本文件中连续的重复行，中间不能夹杂其他文本行（非相邻的默认不会去重）—去除了重复的，保留的都是唯一的：

```
[root@linuxprobe~]# uniq uniq.txt
```

### 12、sort命令

sort 命令用于对文本内容进行再排序，语法格式为：

```
sort [参数] 文件名称
```

sort 命令中的参数及其作用如下：

```
-f    忽略大小写
-b    忽略缩进与空格
-n    以数值型排序
-r    反向排序
-u    去除重复行
-t    指定间隔符
-k    设置字段范围
```

sort命令默认会按照字母顺序从小大大进行排列。

与 uniq 命令不同，sort 命令是无论内容行之间是否夹杂有其他内容，只要有两个一模一样的内容行，立马就可以使用-u 参数进行去重操作：

```
[root@linuxprobe~]# sort -u sort.txt
```

使用-n对数字进行排序：

```
[root@linuxprobe~]# sort -n number.txt
```

一个小实验：下面内容节选自/etc/passwd 文件中的前 5个字段：

```
[root@linuxprobe~]# cat user.txt
polkitd:x:998:996:User for polkitd
geoclue:x:997:995:User for geoclue
rtkit:x:172:172:RealtimeKit
pulse:x:171:171:PulseAudio System Daemon
qemu:x:107:107:qemu user
usbmuxd:x:113:113:usbmuxd user
unbound:x:996:991:Unbound DNS resolver
rpc:x:32:32:Rpcbind Daemon
gluster:x:995:990:GlusterFS daemons
```

上面其实是 5 个字段，各个字段之间是用了冒号进行间隔，如果想以第 3 个字段中的数字作为排序依据，那么可以用-t 参数指定间隔符，用-k 参数指定第几列，用-n 参数进行数字排序来搞定：(注意要指定-n排序，不然编译器只会看第一个数字，即会认为32比107大)

```
[root@linuxprobe~]# sort -t : -k 3 -n user.txt
```

## 2.7、文件目录管理命令

### 1、touch命令

touch 命令用于创建空白文件或设置文件的时间，语法格式为：

```
touch [参数] 文件名称
```

使用touch命令创建一个名为linuxprobe的空白文本文件：

```
[root@linuxprobe~]# touch linuxprobe
```

touch 命令的参数及其作用如下：

```
-a   仅修改“访问时间”（Atime）
-m   仅修改“修改时间”（Mtime）
-d   同时修改Atime与Mtime  
```

可以用 touch 命令把文件的修改时间设置成别的时间：

```
[root@linuxprobe~]# touch -d "2020-05-04 15:44" anaconda-ks.cfg
[root@linuxprobe~]# ls -l anaconda-ks.cfg
-rw-------. 1 root root 1260 May 4 15:44 anaconda-ks.cfg
```

### 2、mkdir命令

mkdir 命令用于创建空白的目录，英文全称为“make directory”，语法格式为：

```
mkdir [参数] 目录名称
```

除了能创建单个空白目录外，mkdir 命令还可以结合-p 参数来递归创建出具有嵌套层叠关系的文件目录：

```
[root@linuxprobe~]# mkdir linuxprobe
[root@linuxprobe~]# cd linuxprobe
[root@linuxprobe linuxprobe]# mkdir -p a/b/c/d/e
[root@linuxprobe linuxprobe]# cd a
[root@linuxprobe a]# cd b
[root@linuxprobe b]#
```

### 3、cp命令

cp 命令用于复制文件或目录，英文全称为“copy”，语法格式为：

```
cp [参数] 源文件名称 目标文件名称
```

在Linux系统中，复制操作具体分为3种情况：

1️⃣如果目标文件是目录，则会把源文件复制到该目录中；
2️⃣如果目标文件也是普通文件，则会询问是否要覆盖它；
3️⃣如果目标文件不存在，则创建之并执行正常的复制操作。

复制命令基本不会出错，唯一需要记住的就是在复制目录时要加上-r 参数。

cp 命令的参数及其作用如下：

```
-p    保留原始文件的属性
-d    若对象为“链接文件”，则保留该“链接文件”的属性
-r    递归持续复制（用于目录）
-i    若目标文件存在则询问是否覆盖
-a    相当于-pdr（p、d、r 为上述参数）
```

使用 touch 命令创建一个名为 install.log 的普通空白文件，然后将其复制为一份名为 x.log 的备份文件，最后再使用 ls 命令查看目录中的文件：

```c++
[root@linuxprobe~]# touch install.log
[root@linuxprobe~]# cp install.log x.log
[root@linuxprobe~]# ls install.log x.log
```

### 4、mv命令

mv 命令用于剪切或重命名文件，英文全称为“move”，语法格式为：

```
mv [参数] 源文件名称 目标文件名称
```

剪切操作不同于复制操作，因为它默认会把源文件删除，只保留剪切后的文件。

如果在同一个目录中将某个文件剪切后还粘贴到当前目录下，其实也就是对该文件进行了重命名操作：

```
[root@linuxprobe~]# mv x.log linux.log
```

其实就是把x.log重命名为linux.log，如果linux.log文件已经存在的话，就会覆盖其内容，而且x.log不会被删除。

### 5、rm命令

rm 命令用于删除文件或目录，英文全称为“remove”，语法格式为：

```
rm [参数] 文件名称
```

在 Linux 系统中删除文件时，系统会默认向您询问是否要执行删除操作，如果不想总是看到这种反复的确认信息，可在 rm 命令后跟上-f 参数来强制删除。另外，要想删除一个目录，需要在 rm 命令后面加一个-r 参数才可以，否则删除不掉。

rm 命令的参数及其作用如下：

```
-f   强制执行
-i   删除前询问
-r   删除目录
-v   显示过程
```

删除install.log文件和linux.log文件：

```
[root@linuxprobe~]# rm install.log
[root@linuxprobe~]# rm -f linux.log
```

删除多层目录a/b/c: 注意在询问的时候要输入y，而不是回车

```
[root@linuxprobe ~]# rm -r a
rm: descend into directory 'a'? y
rm: descend into directory 'a/b'? y
rm: remove directory 'a/b/c'? y
rm: remove directory 'a/b'? y
rm: remove directory 'a'? y
[root@linuxprobe ~]# 
```

### 6、dd命令

dd 命令用于按照指定大小和个数的数据块来复制文件或转换文件，语法格式为：

```
dd if=输入的文件 of=输出的文件 count=块个数 bs=块大小
```

Linux系统中有一个名为/dev/zero 的设备文件，因为这个文件不会占用系统存储空间，但却可以提供无穷无尽的数据，因此常常使用它作为dd命令的输入文件，来生成一个指定大小的文件。

dd 命令的参数及其作用如下表所示：

```
if       输入的文件名称
of       输出的文件名称
count    设置要复制“块”的个数
bs       设置每个“块”的大小
```

例如，用 dd 命令从/dev/zero 设备文件中取出一个大小为 560MB 的数据块，然后保存成名为 560_file 的文件:

```
[root@linuxprobe~]# dd if=/dev/zero of=560_file count=1 bs=560M
```

dd 命令的功能也绝不仅限于复制文件这么简单。如果想把光驱设备中的光盘制作成 iso格式的镜像文件，在 Windows 系统中需要借助于第三方软件才能做到，但在 Linux 系统中可以直接使用 dd 命令来压制出光盘镜像文件，将它变成一个可立即使用的 iso 镜像：

```
[root@linuxprobe~]# dd if=/dev/cdrom of=RHEL-server-8.0-x86_64-LinuxProbe.Com.iso
```

### 7、file命令

file命令用于查看文件的类型，语法格式为：

```
file 文件名称
```

在 Linux 系统中，由于文本、目录、设备等所有这些一切都统称为文件，但是它们又不像 Windows 系统那样都有后缀，因此很难通过文件名一眼判断出具体的文件类型，这时就需要使用 file 命令来查看文件类型了。

```
[root@linuxprobe~]# file anaconda-ks.cfg
anaconda-ks.cfg: ASCII text
[root@linuxprobe~]# file /dev/sda
/dev/sda: block special
```

在 Windows 系统中打开文件时，一般是通过用户双击鼠标完成的，系统会自行判断用户双击的文件是什么类型，因此需要有后缀进行区别。而 Linux 系统则是根据用户执行的命令来调用文件，例如执行 cat 命令查看文本，执行 bash 命令执行脚本等，所以也就不需要强制让用户给文件设置后缀了。

### 8、tar命令

tar 命令用于对文件进行打包压缩或解压，语法格式为：

```
tar 参数 文件名称
```

在 Linux 系统中，压缩格式主要使用的是.tar或.tar.gz 或.tar.bz2格式，这些格式大部分都是由 tar 命令生成的。

tar 命令的参数及其作用如下：

```
-c    创建压缩文件
-x    解开压缩文件
-t    查看压缩包内有哪些文件
-z    用 gzip 压缩或解压
-j    用 bzip2 压缩或解压
-v    显示压缩或解压的过程
-f    目标文件名
-p    保留原始的权限与属性
-P    使用绝对路径来压缩
-C    指定解压到的目录
```

推荐使用-v 参数向用户不断显示压缩或解压的过程。
-f 参数特别重要，它必须放到参数的最后一位，代表要压缩或解压的软件包名称。

一般使用的打包压缩命令为：

```
tar -czvf 压缩包名称.tar.gz 要打包的目录
```

解压命令为：

```
tar -xzvf 压缩包名称.tar.gz
```

例如，使用 tar 命令把/etc 目录通过 gzip格式进行打包压缩，并把文件命名为 etc.tar.gz：

```
[root@linuxprobe~]# tar -czvf etc.tar.gz /etc
```

接下来将打包后的压缩包文件指定解压到/root/etc 目录中：

```
[root@linuxprobe~]# tar -xzvf etc.tar.gz -C /root/etc
```

注：-czvf和-xzvf前面的 - 可以省略。



# 第3章、管道符、重定向与环境变量

## 3.1、输入输出重定向

1、简而言之
输入重定向：把文件导入到命令中。
输出重定向：把原本要输出到屏幕的数据信息写入到指定文件中。

2、1️⃣标准输入重定向 ( STDIN,文件描述符为0 )：默认从键盘输入，也可以从其他文件或命令中输入。
2️⃣标准输出重定向 (STDOUT,文件描述符为1)：默认输出到屏幕。
3️⃣错误输出重定向 (STDERR,文件描述符为2)：默认输出到屏幕。

3、输入重定向中用到的符号及其作用如下：

```
命令 < 文件 	 将文件作为命令的标准输入
命令 << 分界符   从标准输入中读入，直到遇见分界符才停止
命令 < 文件1 > 文件2  将文件1作为命令的标准输入并将标准输出到文件2
```

输入重定向相对来说有些冷门，在工作中遇到的概率会小一点。输入重定向的作用是把文件直接导入到命令中。下面使用输入重定向统计文件readme.txt中的内容行数：

```
[root@linuxprobe~]# wc -l < readme.txt
2
```

与正常统计行数结果不太一样：没有了文件名称

```
[root@linuxprobe~]# wc -l /etc/passwd
38 /etc/passwd
```

因为“wc -l < readme.txt”只是将 readme.txt 文件中的内容通过操作符导入到命令中，没有被当作命令对象进行执行，因此 wc 命令只能读到信息流数据，而没有文件名称的信息。

4、输出重定向用到的符号及其作用如下：

```
命令 > 文件 将标准输出重定向到一个文件中（清空原有文件的数据）
命令 2> 文件 将错误输出重定向到一个文件中（清空原有文件的数据）
命令 >> 文件 将标准输出重定向到一个文件中（追加到原有内容的后面）
命令 2>> 文件 将错误输出重定向到一个文件中（追加到原有内容的后面）
命令 &>> 文件  将标准输出与错误输出共同写入到文件中（追加到原有内容的后面）
```

重定向中的标准输出模式可以省略1不写，错误输出模式的2则是不能省略的。

通过标准输出重定向将 man bash 命令原本要输出到屏幕的信息写入到文件 readme.txt 中：

```
[root@linuxprobe~]# man bash > readme.txt
```

注意覆盖写入和追加写入的区别：

```
[root@linuxprobe~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@linuxprobe~]# echo "Welcome to LinuxProbe.Com" > readme.txt
```

覆盖写入，最后只有一行内容：

```
[root@linuxprobe~]# cat readme.txt
Welcome to LinuxProbe.Com
```

通过追加写入文件：

```
[root@linuxprobe~]# echo "Quality linux learning materials" >> readme.txt
[root@linuxprobe~]# cat readme.txt
Welcome to LinuxProbe.Com
Quality linux learning materials
```

如果命令没有发生错误，那么使用错误输出重定向技术无法把内容写入文件中：

```
[root@linuxprobe~]# ls -l linuxprobe 2> /root/stderr.txt
-rw-r--r--. 1 root root 0 Mar 1 13:30 linuxprobe
```

因为linuxprobe文件存在，所以指令能正确执行，使用错误输出重定向(2>)并不能将信息写入文件，而是仍然会在屏幕上显示。

改为与之匹配的标准输出重定向(>)即可：

```
[root@linuxprobe~]# ls -l xxxxxx 2> /root/stderr.txt
```

查看文件内容：

```
[root@linuxprobe~]# cat /root/stderr.txt
ls: cannot access xxxxxx: No such file or directory
```

如果想不论正确或错误，都想输入到文件内，要使用&>>操作符：

```
[root@linuxprobe~]# ls -l linuxprobe &>> readme.txt
[root@linuxprobe~]# ls -l xxxxxx &>> readme.txt
```

查看文件内容：

```
[root@linuxprobe ~]# cat readme.txt
-rw-r--r--. 1 root root 0 Sep 13 11:21 linuxprobe
ls: cannot access 'xxxx': No such file or directory
```

## 3.2、管道命令符

1、管道命令符的格式为 “命令A | 命令B”，其作用可以用一句话概括为：把前一个命令原本要输出到屏幕的信息当做后一个命令的标准输入。

2、有了管道命令符，我们就很容易把指令结合到一起使用了，比如统计被限制登录系统的用户个数：

```
[root@linuxprobe~]# grep /sbin/nologin /etc/passwd | wc -l
40
```

将原本grep搜集到的信息传给wc -l命令进行了进一步加工。

用翻页的形式(more支持)查看/etc目录中的文件列表及属性信息：

```
[root@linuxprobe~]# ls -l /etc/ | more
```

使用ps和grep命令来搜索和bash相关的进程信息：

```
[root@linuxprobe~]# ps aux | grep bash
```

先使用ps查找当前正在运行的所有进程，再用grep过滤出包含关键词 "bash" 的进程。

3、一行指令中也可以包含多个管道运算符。

将bash进程相关的信息输出到屏幕并且同时写入到文件中，要使用tee命令：

```
[root@linuxprobe~]# ps aux | grep bash | tee result.txt
root 1070 0.0 0.1 25384 2324 ? S Sep21 0:00 /bin/bash /usr/sbin/ksmtuned
root 3899 0.0 0.2 26540 5136 pts/0 Ss 00:27 0:00 bash
root 4320 0.0 0.0 12112 1112 pts/0 S+ 00:51 0:00 grep --color=auto bash
```

文件中也被写入了内容：

```
[root@linuxprobe~]# cat result.txt
root 1070 0.0 0.1 25384 2324 ? S Sep21 0:00 /bin/bash /usr/sbin/ksmtuned
root 3899 0.0 0.2 26540 5136 pts/0 Ss 00:27 0:00 bash
root 4320 0.0 0.0 12112 1112 pts/0 S+ 00:51 0:00 grep --color=auto bash
```

## 3.3、命令行的通配符

1、Linux 系统中的通配符及含义如下：

```
*            任意字符
?            单个任意字符
[abc]        a、b、c三个字符中的任意一个字符
[a-z]        单个小写字母
[A-Z]        单个大写字母
[a-Z]        单个字母
[0-9]        单个数字
[[:alpha:]]  任意字母
[[:upper:]]  任意大写字母
[[:lower:]]  任意小写字母
[[:digit:]]  所有数字
[[:alnum:]]  任意字母加数字
[[:punct:]]  标点符号
```

查看所有在/dev目录中且以sda开头的文件：

```
[root@linuxprobe~]# ls -l /dev/sda*
```

想查看文件名以 sda 开头，且后面有且只有一个字符的文件：

```
[root@linuxprobe~]# ls -l /dev/sda?
```

2、通配符不一定非要放到结尾。
例如：查询/etc/目录中所有以.conf结尾的配置文件：

```
[root@linuxprobe~]# ls -l /etc/*.conf
-rw-r--r--. 1 root root 55 Feb 1 2019 /etc/asound.conf
-rw-r--r--. 1 root root 25696 Dec 12 2018 /etc/brltty.conf
```

3、通配符还可以与创建文件的命令结合，一口气创建出多个文件：

```
[root@linuxprobe~]# touch {AA,BB,CC}.conf
```

一口气创建了三个文件AA.conf、BB.conf、CC.conf

还可以一口气输出多个指定的信息：

```
[root@linuxprobe~]# echo file{1,2,3,4,5}
file1 file2 file3 file4 file5
```

## 3.4、常用的转义字符

1、四个最常用的转义字符：

1️⃣反斜杠（ \ ）：使反斜杠后面的一个变量变为单纯的字符。
2️⃣单引号（' '）：转义其中所有的变量为单纯的字符串。
3️⃣双引号（" "）：保留其中的变量属性，不进行转义处理。
4️⃣反引号（``）：把其中的额命令执行后返回结果。

例如，先定义一个名为 PRICE 的变量并赋值为 5，然后输出以双引号括起来的字符串与变量信息：

```
[root@linuxprobe~]# PRICE=5
[root@linuxprobe~]# echo "Price is $PRICE"
Price is 5
```

如果想要把美元符号$也输出，不能使用$$PRICE，因为$$的作用是显示当前程序的进程ID号码：

```
[root@linuxprobe~]# echo "Price is $$PRICE"
Price is 3767PRICE
```

此时需要反斜杠\进行转义，让$变成一个单纯的文本输出：

```
[root@linuxprobe~]# echo "Price is \$$PRICE"
Price is $5
```

单引号(' ')会阻止变量扩展，特殊字符也不会被解释：

```
[root@linuxprobe~]# name=Linux
[root@linuxprobe~]# echo '$name'
$name
[root@linuxprobe~]# echo 'Hello\nWorld'
Hello\nWorld
```

如果需要某个命令的输出值，用反引号`` ：

```
[root@linuxprobe~]# echo `uname -a`
Linux linuxprobe.com 4.18.0-80.el8.x86_64 #1 SMP Wed Mar 13 12:02:46 UTC 2019
x86_64 x86_64 x86_64 GNU/Linux
```

2、如果参数中出现了空格，就加双引号；如果参数中没有空格，那就不用加双引号。

```
[root@linuxprobe~]# echo AA BB CC
AA BB CC
[root@linuxprobe~]# echo "AA BB CC"
AA BB CC
```

虽然上述结果一样，但是有空格还是加双引号较好，防止引起歧义。

## 3.5、重要的环境变量

1、在 Linux 系统中，变量名称一般都是大写的，命令则都是小写的，这是一种约定俗成的规范。Linux 系统中的环境变量是用来定义系统运行环境的一些参数，比如每个用户不同的家目录、邮件存放位置等。

2、可以用 alias 命令来创建一个属于自己的别名命令，语法格式为：

```
alias 别名=命令
```

若要取消一个命令别名，则是用 unalias 命令，语法格式为：

```
unalias 别名
```

例如创建别名 ll 用于显示详细列表信息：

```
[root@linuxprobe~]# alias ll='ls -l'
```

取消这个别名：

```
[root@linuxprobe~]# unalias ll
```

之前在使用 rm 命令删除文件时，Linux 系统都会要求用户确认是否执行删除操作，其实这就是 Linux 系统为了防止用户误删除文件而特意设置的 rm 别名命令—“rm -i”。

3、用户执行一条命令之后，命令在linux中的执行分为4个步骤：
1️⃣判断用户是否以绝对路径或相对路径的方式输入命令（如/bin/ls），如果是绝对路径则直接执行，否则进入第 2 步继续判断。

2️⃣Linux 系统检查用户输入的命令是否为别名命令（即用一个自定义的命令名称来替换原本的命令名称）。

3️⃣Bash 解释器判断用户输入的是内部命令还是外部命令。内部命令是解释器内部的指令，会被直接执行；而用户在绝大部分时间输入的是外部命令，这些命令交由步骤 4 继续处理(可以使用 ' type 命令名称 ' 来查看命令类型)。

4️⃣系统在多个路径中查找用户输入的命令文件，而定义这些路径的环境变量叫作 PATH，作用是告诉 Bash 解释器待执行的命令可能存放的位置，然后 Bash 解释器就会乖乖地在这些位置中逐个查找。这就是我们在配置环境的时候需要把路径添加到环境变量PATH中的原因。

4、可以使用env命令查看Linux系统中的所有环境变量：

```
[root@linuxprobe~]# env
```

常用的十个环境变量：

```
HOME            用户的主目录（即家目录）
SHELL           用户在使用的 Shell 解释器名称
HISTSIZE        输出的历史命令记录条数
HISTFILESIZE    保存的历史命令记录条数
MAIL            邮件保存路径
LANG            系统语言、语系名称
RANDOM          生成一个随机数字
PS1 Bash        解释器的提示符
PATH            定义解释器搜索用户执行命令的路径
EDITOR          用户默认的文本编辑器
```

5、一个相同的变量会因为用户身份的不同而具有不同的值：

```
[root@linuxprobe~]# echo $HOME
/root
[root@linuxprobe~]# su - linuxprobe
[linuxprobe@linuxprobe~]$ echo $HOME
/home/linuxprobe
```

su是用于切换用户的命令。

6、变量的作用范围有限，默认情况下不能被其他用户使用：

```
[root@linuxprobe~]# WORKDIR=/home/workdir
[root@linuxprobe workdir]# su linuxprobe
```

在切换用户之后，变量WORKDIR的值就不能被使用了。

如果工作需要，可以使用export命令将其提升为全局变量，这样其他用户就可以使用了：

```
[root@linuxprobe~]# WORKDIR=/home/workdir
[root@linuxprobe~]# export WORKDIR
[root@linuxprobe~]# su linuxprobe
[linuxprobe@linuxprobe~] echo $WORKDIR
/home/workdir
```

后续要是不使用这个变量了，则可执行 unset 命令把它取消掉：

```
[root@linuxprobe~]# unset WORKDIR
```

7、直接在终端设置的变量能够立即生效，但在重启服务器后就会失效，因此我们需要将变量和变量值写入到.bashrc 或者.bash_profile 文件中，以确保永久能使用它们。



# 第4章、Vim编辑器与Shell命令脚本

## 4.1、Vim文本编辑器

1、Vim的发布最早可以追溯到 1991 年，英文全称为 Vi Improved，它是 Vi 编辑器的提升版本，其中最大的改进当属添加了代码着色功能，在某些编程场景下还能自动修正错误代码。Vim文本编辑器会默认会安装在当前所有的 Linux 操作系统上。

2、Vim 之所以能得到广大厂商与用户的认可，原因在于 Vim 编辑器中设置了 3 种模式：
1️⃣命令模式：控制光标移动，可对文本进行复制、粘贴、删除和查找等工作。
2️⃣输入模式：正常的文本录入。
3️⃣末行模式：保存或退出文档，以及设置编辑环境。

3、每次运行 Vim 编辑器时，默认进入命令模式，此时需要先切换到输入模式后再进行文档编写工作。

编写完文档后需要先返回命令模式，然后再进入末行模式，执行文档的保存或退出操作。

在 Vim 中，无法直接从输入模式切换到末行模式，需要命令模式这个 ‘中介’。

4、命令模式中的常用命令：

```
dd    删除（剪切）光标所在整行
5dd   删除（剪切）从光标处开始的 5 行
yy    复制光标所在整行
5yy   复制从光标处开始的 5 行
n     显示搜索命令定位到的下一个字符串
N     显示搜索命令定位到的上一个字符串
u     撤销上一步的操作
p     将之前删除（dd）或复制（yy）过的数据粘贴到光标后面
```

要想切换到末行模式，在命令模式中输入一个冒号（:）就可以了。

5、末行模式主要用于保存或退出文件、设置 Vim 编辑器的工作环境、让用户执行外部的 Linux 命令或跳转到所编写文档的特定行数。末行模式中常用的命令如下：

```
:w            保存
:q            退出
:q!           强制退出（放弃对文档的修改内容）
:wq!          强制保存退出
:set nu       显示行号
:set nonu     不显示行号
:命令          执行该命令
:整数          跳转到该行
:s/one/two    将当前光标所在行的第一个one替换成 two
:s/one/two/g  将当前光标所在行的所有 one 替换成 two
:%s/one/two/g 将全文中的所有 one 替换成 two
?字符串        在文本中从下至上搜索该字符串
/字符串        在文本中从上至下搜索该字符串
```

6、创建一个文件，使用vim对其进行文本编辑。
i、使用vim user.txt命令会打开一个名为user.txt的文件以进行编辑。如果该文件不存在，Vim会创建一个新文件并将其命名为user.txt。

```
[root@linuxprobe~]# vim practice.txt
```

ii、执行完该命令，会自动打开practice.txt文件，但是此时还是命令模式，可以使用a，i，o切换至输入模式：
1️⃣a：在光标后面一位切换到输入模式。
2️⃣i：在光标当前位置切换到输入模式。
3️⃣o：在光标的下面再创建一个空行。

iii、在输入模式中编写完内容之后，敲击esc即可返回命令模式。
iv、在命令模式中输入“:wq!”切换到末行模式并完成保存退出操作。
v、退回到终端之后，就可以使用cat命令查看文件内容了。

如果我们要继续编辑上面编辑了的文档practice.txt：

i、vim practice.txt打开Vim编辑器，同时进入命令模式。

ii、由于是追加内容，因此按‘o’键进入输入模式更加高效。

后面的步骤就跟上面的一样了。

7、使用Vim编辑器配置主机名称

在 Linux系统中，主机名大多保存在/etc/hostname 文件中，使用Vim编辑器将/etc/hostname 配置文件中的主机名称修改为“linuxprobe.com”：

1️⃣vim /etc/hostname打开Vim编辑器，同时进入命令模式。

2️⃣按a进入输入模式之后把原始主机名称改为“linuxprobe.com”。

3️⃣保存并退出文档，然后使用 hostname 命令检查是否修改成功。

```
[root@linuxprobe ~]# hostname
linuxprobe.com
```

8、使用Vim编辑器配置网卡信息

网卡 IP 地址配置的是否正确是两台服务器是否可以相互通信的前提，配置网络服务的工作其实就是在编辑网卡配置文件。

现在有一个名称为 ifcfg-ens160 的网卡设备，将其配置为开机自启动，并且 IP 地址、子网、网关等信息由人工指定，其步骤如下所示：

1️⃣首先切换到存放着网卡的配置文件的目录/etc/sysconfig/network-scripts 中。

2️⃣使用 Vim 编辑器修改网卡文件 ifcfg-ens160，逐项改写配置参数并保存退出。

3️⃣重启网络服务并测试网络是否连通。

9、使用Vim编辑器配置软件仓库

软件仓库是一种能进一步简化 RPM 管理软件的难度以及自动分析所需软件包及其依赖关系的技术。可以把 Yum 或 DNF 想象成是一个硕大的软件仓库，里面保存有几乎所有常用的工具，而且只需要说出所需的软件包名称，系统就会自动为您搞定一切。建议在 RHEL 8 中使用 dnf 作为软件的安装命令，因为它具备更高的效率，而且支持多线程同时安装软件。

搭建并配置软件仓库的大致步骤如下所示：

1️⃣进入存放着软件仓库的配置文件的目录/etc/yum.repos.d/中。

2️⃣使用 Vim 编辑器创建一个名为 rhel8.repo 的新配置文件（文件名称可随意，但后缀必须为.repo），逐项编辑配置参数并保存退出。

3️⃣按配置参数中所填写的仓库位置挂载光盘，并把光盘挂载信息写入/etc/fstab 文件中。

4️⃣使用“dnf install httpd -y”命令检查软件仓库是否已经可用。

## 4.2、编写shell脚本

1、脚本是一种轻量级程序，通常由解释器运行而不是编译。它用于自动化任务、简化操作和快速开发。例如，使用 Vim 编辑器把 Linux 命令按照顺序依次写入到一个文件中，就是一个简单的脚本了。

2、shell脚本命令的工作方式有两种：
1️⃣交互式：用户每输入一条命令就立即执行。

2️⃣批处理：由用户事先编写好一个完整的 Shell 脚本，Shell 会一次性执行脚本中诸多的命令。

3、Shell 脚本文件的名称可以任意，但为了避免被误以为是普通文件，建议将.sh 后缀加上，以表示是一个脚本文件。

例如，查看当前所在工作路径并列出当前目录下所有的文件及属性信息，首先先进入vim编辑器：

```
[root@linuxprobe~]# vim example.sh
```

然后编辑脚本：

```
#!/bin/bash
#For Example BY linuxprobe.com
pwd
ls -al
```

第一行的脚本声明（#!）用来告诉系统使用哪种 Shell 解释器来执行该脚本；
第二行的注释信息（#）是对脚本功能和某些命令的介绍信息，在日后看到这个脚本内容时，可以快速知道该脚本的作用或一些警告信息；
第三、四行的可执行语句就是我们平时执行的 Linux 命令。

编写完脚本程序之后可以执行查看结果：

```
[root@linuxprobe~]# bash example.sh
```

除了直接用Bash解释器命令运行Shell脚本文件之外，还可以通过输入完整路径的方式来执行：

```
[root@linuxprobe~]# ~/example.sh
```

但是此时直接执行会报错，因为权限不够，需要额外为脚本文件增加执行权限：

```
[root@linuxprobe~]# chmod u+x example.sh
```

之后在执行就ok了。

4、像上述的脚本执行的只是固定的命令，太过死板，linux系统中的Shell脚本语言考虑到了这些，设置了用于接收用户提供的参数的变量：

```
[root@linuxprobe~]# bash example.sh 参数1 参数2 参数3 参数4 ……
$0   当前shell脚本程序的名称
$#   表示总共有几个参数
$*   对应的是所有位置的参数值
$?   显示上一次命令的执行返回值
$1   第一个位置的参数值
$2   第二个位置的参数值 后面的以此类推
```

尝试使用参数编写一个脚本程序：
1️⃣首先使用vim编辑器进行编辑：

```
[root@linuxprobe~]# vim example.sh
```

编辑的内容为：

```
#!/bin/bash
echo "当前脚本名称为$0"
echo "总共有$#个参数，分别是$*。"
echo "第 1 个参数为$1，第 5 个为$5。"
```

2️⃣执行脚本文件：

```
[root@linuxprobe~]# bash example.sh one two three four five six
```

结果为：

```
当前脚本名称为 example.sh
总共有 6 个参数，分别是 one two three four five six。
第 1 个参数为 one，第 5 个为 five。
```

5、Shell 脚本中的条件测试语法可以判断表达式是否成立，若条件成立则返回数字 0，否则便返回非零值。语法格式为：

```
[ 条件表达式 ]
```

注意条件表达式两边均有一个空格。按照测试对象来分，条件表达式可以分为四种：

1️⃣文件测试语句。
2️⃣逻辑测试语句。
3️⃣整数值比较语句。
4️⃣字符串比较语句。

6、文件测试时使用参数运算符来判断文件是否满足某个条件，参数列表如下：

```
-d    测试文件是否为目录类型
-e    测试文件是否存在
-f    判断是否为一般文件
-r    测试当前用户是否有权限读取
-w    测试当前用户是否有权限写入
-x    测试当前用户是否有权限执行
```

例如，用参数-d判断/etc/fstab是否为一个目录类型的文件，然后通过shell解释器内设的变量$?显示上一条命令执行后的返回值，若返回0，那么表示它是目录；若不为0，则意味着不是目录，或者目录不存在：

```
[root@linuxprobe~]# [ -d /etc/fstab ]
[root@linuxprobe~]# echo $?
1
```

使用-f参数判断/etc/fstab 是否为一般文件，为0说明文件存在，且是一般文件：

```
[root@linuxprobe~]# [ -f /etc/fstab ]
[root@linuxprobe~]# echo $?
0
```

7、可以使用一条语句就能判断文件是否满足条件，要用到逻辑运算符&&，当前面的命令执行成功后才会执行它后面的命令，因此可以用来判断/dev/cdrom 文件是否存在，若存在则输出 Exist 字样：

```
[root@linuxprobe~]# [ -e /dev/cdrom ] && echo "Exist"
Exist
```

除了逻辑与还有逻辑或，运算符号为||，表示当前面的命令执行失败后才会执行它后面的命令。

可以用||和环境变量$USER来判定当前登录的是不是用户是不是管理员身份：

```
[root@linuxprobe~]# [ $USER = root ] || echo "user"
```

如果是root身份就不会输出user了。

逻辑非在linux系统中符号是“ ！”，表示把条件测试中的判断结果取相反值，还是以上例为例：

```
[root@linuxprobe~]# [ ! $USER = root ] || echo "user"
```

此时如果是root身份就输出user了。

可以将上述逻辑符号结合使用，使得无论此时是root还是user都能得到输出：

```
[root@linuxprobe~]# [ ! $USER = root ] && echo "user" || echo "root"
```

上述命令是先判断&&左侧是不是真的，若是真则执行&&右边的，此时||左侧是真的，右侧的root就不会被输出；若&&左侧是假的，则整个&&是假的，右侧的root就会被输出。

8、整数比较运算符仅是对数字的操作，不能将数字与字符串、文件等内容一起操作。而且不能想当然地使用日常生活中的=、>、<等来判断，因为=与赋值命令符冲突，>和<分别与输出重定向命令符和输入重定向命令符冲突。可用的整数比较运算符如下：

```
-eq   是否等于
-ne   是否不等于
-gt   是否大于
-lt   是否小于
-le   是否等于或小于
-ge   是否大于或等于
```

测试10与10之间的关系：

```
[root@linuxprobe~]# [ 10 -gt 10 ]
[root@linuxprobe~]# echo $?
1
[root@linuxprobe~]# [ 10 -eq 10 ]
[root@linuxprobe~]# echo $?
0
```

9、字符串比较语句用于判断测试字符串是否为空值(即是否未被定义)，或两个字符串是否相同。

字符串常见运算符如下：

```
=    比较字符串内容是否相同
!=   比较字符串内容是否不同
-z   判断字符串内容是否为空
```

例子：通过判断String变量是否为空值，进而判断是否定义了这个变量：

```
[root@linuxprobe~]# [ -z $String ]
[root@linuxprobe~]# echo $?
0
```

通过引入逻辑运算符，判断当前语言是不是英语：

```
[root@linuxprobe~]# echo $LANG
en_US.UTF-8
[root@linuxprobe~]# [ ! $LANG = "en.US" ] && echo "Not en.US"
Not en.US
```

## 4.3、流程控制语句

1、在Shell脚本中，fi是if语句的结束标记，用于结束一个条件语句块。

1️⃣if条件语句的单分支语法格式：

```
if 条件测试操作
	then 命令序列
fi
```

例如，用if来判断/media/cdrom目录是否存在，若不存在就创建这个目录：

首先打开Vim文本编辑器：

```
[root@linuxprobe~]# vim mkcdrom.sh
```

然后编辑内容为：

```
#!/bin/bash
DIR="/media/cdrom"
if [ ! -d $DIR ]
then
		mkdir -p $DIR
fi
```

最后执行脚本文件并使用ls命令检测目录是否已经被创建：

```
[root@linuxprobe~]# bash mkcdrom.sh
[root@linuxprobe~]# ls -ld /media/cdrom
drwxr-xr-x. 2 root root 6 Oct 13 21:34 /media/cdrom
```

2️⃣if条件语句的双分支结构格式为：

```
if 条件测试操作
	then 命令序列1
	else 命令序列2
fi
```

例如，验证某台主机是否在线：

首先打开Vim编辑器：

```
[root@linuxprobe~]# vim chkhost.sh
```

然后编辑脚本：

```
#!/bin/bash
ping -c 3 -i 0.2 -W 3 $1 &> /dev/null
if [ $? -eq 0 ]
then
		echo "Host $1 is On-line."
else
		echo "Host $1 is Off-line."
fi
```

这里如果ping命令成功执行，则$?的值就是0

最后运行脚本文件检测结果：

```
[root@linuxprobe~]# bash chkhost.sh 192.168.10.10
Host 192.168.10.10 is On-line.
[root@linuxprobe~]# bash chkhost.sh 192.168.10.20
Host 192.168.10.20 is Off-line.
```

3️⃣if条件语句的多分支结构语法格式如下：

```
if 条件测试操作1
  then 命令序列1
elif 条件测试操作2
  then 命令序列2
else
  命令序列3
fi
```

例如，判断用户输入的分数在哪个成绩区间内，然后输出如Excellent、Pass、Fail 等提示信息：

首先打开vim编辑器：

```
[root@linuxprobe~]# vim chkscore.sh
```

然后编辑脚本文件，其中read 是用来读取用户输入信息的命令，能够把接收到的用户输入信息赋值给后面的指定变量，-p 参数用于向用户显示一些提示信息：

```
#!/bin/bash
read -p "Enter your score（0-100）：" GRADE
if [ $GRADE -ge 85 ] && [ $GRADE -le 100 ] ; then
echo "$GRADE is Excellent"
elif [ $GRADE -ge 70 ] && [ $GRADE -le 84 ] ; then
echo "$GRADE is Pass"
else
echo "$GRADE is Fail"
fi
```

最后运行脚本文件：

```
[root@linuxprobe~]# bash chkscore.sh
Enter your score（0-100）：88
88 is Excellent
[root@linuxprobe~]# bash chkscore.sh
Enter your score（0-100）：80
80 is Pass
[root@linuxprobe~]# bash chkscore.sh
Enter your score（0-100）：30
30 is Fail
[root@linuxprobe~]# bash chkscore.sh
Enter your score（0-100）：200
200 is Fail
```

2、for循环语句的语法格式如下：

```
for 变量名 in 取值列表
do
	命令列表
done
```

使用for循环从文件中读取多个用户名：

首先使用vim给文件添加多个用户名：

```
[root@linuxprobe~]# vim users.txt
andy
barry
carl
duke
eric
george
```

接下来使用vim编辑器编写shell脚本：

```
[root@linuxprobe~]# vim addusers.sh
```

脚本编辑为：

```
#!/bin/bash
read -p "Enter The Users Password : " PASSWD
for UNAME in `cat users.txt`
do
        id $UNAME &> /dev/null
        if [ $? -eq 0 ]
        then
        		echo "$UNAME , Already exists"
        else
                useradd $UNAME &> /dev/null
                echo "$PASSWD" | passwd --stdin $UNAME &> /dev/null
                echo "$UNAME , Create success"
        fi
done
```

id $UNAME表示查看用户的信息。/dev/null 是一个被称作 Linux 黑洞的文件，把输出信息重定向到这个文件等同于删除要输出到屏幕上的信息，可以让用户的屏幕窗口保持简洁。于是id $UNAME &> /dev/null的意思就是将本来要显示在屏幕上的用户“$UNAME”信息写入到黑洞文件中了，而且后面用户添加成功之后正常情况下屏幕窗口除了“用户账户创建成功”（Create success）的提示后不会有其他内容。

执行脚本文件：

```
[root@linuxprobe~]# bash addusers.sh
Enter The Users Password : linuxprobe
andy , Create success
barry , Create success
carl , Create success
duke , Create success
eric , Create success
george , Create success
```

将for与if结合，尝试让脚本读取主机列表，并自动判断这些主机是否在线：

首先还是用vim编辑器创建一个主机列表文件：

```
[root@linuxprobe~]# vim ipaddrs.txt
192.168.10.10
192.168.10.11
192.168.10.12
```

使用vim编辑器编写脚本文件：

```
[root@linuxprobe~]# vim CheckHosts.sh
```

脚本文件内容编辑为：

```
#!/bin/bash
HLIST=$(cat~/ipaddrs.txt)
for IP in $HLIST
do
        ping -c 3 -i 0.2 -W 3 $IP &> /dev/null
        if [ $? -eq 0 ]
        then
        		echo "Host $IP is On-line."
        else
        		echo "Host $IP is Off-line."
        fi
done
```

执行脚本文件检测结果：

```
[root@linuxprobe~]# ./CheckHosts.sh
Host 192.168.10.10 is On-line.
Host 192.168.10.11 is Off-line.
Host 192.168.10.12 is Off-line.
```

3、while循环语句的格式：

```
while 条件测试操作
do
	命令序列
done
```

编写一个用来猜测数值大小的脚本Guess.h：

```
[root@linuxprobe~]# vim Guess.sh
```

脚本内容编写为：while true时，只有碰到exit才会退出while循环

```
#!/bin/bash
PRICE=$(expr $RANDOM % 1000)
TIMES=0
echo "商品实际价格在0-999之间，猜猜看是多少"
while true
do
	read -p "请输入您猜测的价格数目: " INT
	let TIMES++
	if [ $INT -eq $PRICE ]; then
		echo "恭喜您答对了，实际价格是 $PRICE"
		echo "您总共猜测了 $TIMES次 "
		exit
	elif [ $INT -gt $PRICE ]; then 
		echo "太高了"
	else
		echo "太低了"
	fi
done
```

4、case条件测试语句的格式：

```
case 变量值 in
模式1)
	命令序列1
	;;
模式2)
	命令序列2
	;;
*)
	默认命令序列
esac
```

编写脚本CheckKeys.sh判断用户输入的值是字母、数字还是其他字符：

```
[root@linuxprobe~]# vim Checkkeys.sh
```

编辑脚本内容：

```
#!/bin/bash
read -p "请输入一个字符，并按Enter键确认：" KEY
case "$KEY" in
	[a-z] | [A-Z])
		echo "您输入的是字母"
		;;
	[0-9])
		echo "您输入的是数字"
		;;
	*)
		echo "您输入的是其他字符"
esac
```

## 4.4、计划任务服务程序

1、Linux 可以在没有人为介入的情况下，在指定的时间段自动启用或停止某些服务或命令。计划任务分为两种：
1️⃣一次性计划任务：今晚23:30重启网站服务。
2️⃣长期性计划任务：每周一的凌晨3:25把/home/wwwroot目录打包备份为backup.tar.gz。

2、一次性计划任务只执行一次，一般用于临时的工作需求。可以用 at 命令实现，只需要写成“at 时间”的形式就行。at 命令中的参数及其作用如下：

```
-f   指定包含命令的任务文件
-q   指定新任务名称
-l   显示待执行任务的列表
-d   删除指定的待执行任务
-m   任务执行后向用户发邮件
```

执行at命令来设置一次性计划任务时，默认采用的是交互式方法。例如，设置今晚23:30重启网站服务：

```
[root@linuxprobe~]# at 23:30
warning: commands will be executed using /bin/sh
at> systemctl restart httpd
at> 此处请同时按下<Ctrl>+<d>组合键来结束编写计划任务
job 1 at Tue Sep 17 23:30:00 2024
```

设置好之后检查一下任务列表：

```
[root@linuxprobe~]# at -l
1 Tue Sep 17 23:30:00 2024 a root
```

如果想用非交互式的方式，直接创建一次性任务，需要使用到管道命令符：

```
[root@linuxprobe~]# echo "echo $SHELL" | at 23:00
warning: commands will be executed using /bin/sh
job 2 at Tue Sep 17 23:00:00 2024
```

代码中at 23:30接收的输入流是“echo $SHELL”

此时任务列表中就有两个任务了：

```
[root@linuxprobe~]# at -l
1	Tue Sep 17 23:30:00 2024 a root
2	Tue Sep 17 23:00:00 2024 a root
```

想要删除某个任务，可以使用atrm命令，格式为“atrm 命令序号”

```c++
atrm 1
```

此时再去看看任务列表：

```
[root@linuxprobe~]# at -l
2	Tue Sep 17 23:00:00 2024 a root
```

只剩下了命令序号为2的任务了。

at后面也可以不跟固定的时间，可以是从现在开始2分钟之后执行：

```
[root@linuxprobe~]# at now +2 MINUTE
warning: commands will be executed using /bin/sh
at> systemctl restart httpd
at> 此处请同时按下<Ctrl>+<d>键来结束编写计划任务
job 3 at Wed Oct 14 22:50:00 2020
```

也可以将MINUTE替换成HOUR、DAY、MONTH等词汇。

3、Linux 系统能够周期性地、有规律地执行某些具体的任务，Linux 系统中默认启用的 crond 服务非常合适，其使用的命令为crontab，crontab 命令中的参数及其作用如下：

```
-e    编辑计划任务
-u    指定用户名称
-l    列出任务列表
-r    删除计划任务
```

例如，在每周一、三、五的凌晨3:25，都需要使用tar命令把某个网站的数据目录进行打包处理：
首先使用-e参数创建计划任务：

```
[root@linuxprobe~]# crontab -e
```

执行完上述命令后会自动进入vim编辑器模式，任务的参数格式为："分钟 小时 日期 月份 星期 命令"，如果有些字段没有被设置，则需使用星号*占位，参数字段说明为：

```
分钟 		取值为 0～59 的整数
小时 		取值为 0～23 的任意整数
日期 		取值为 1～31 的任意整数
月份 		取值为 1～12 的任意整数
星期 		取值为 0～7 的任意整数，其中 0 与 7 均为星期日
命令 		要执行的命令或程序脚本		
```

且命令一定要用绝对路径来写，如果不知道命令的绝对路径，就使用whereis去查。

```
[root@linuxprobe ~]# whereis tar
tar: /usr/bin/tar /usr/share/man/man5/tar.5.gz /usr/share/man/man1/tar.1.gz /usr/share/info/tar.info-1.gz /usr/share/info/tar.info-2.gz /usr/share/info/tar.info.gz
```

在vim编辑器中编辑内容为：

```
25 3 * * 1,3,5 /user/bin/tar -czvf backup.tar.gz /home/wwwroot
```

编辑完后使用-l参数查看任务列表：

```
[root@linuxprobe~]# crontab -l
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
```

在某个参数的位置上可以使用逗号(,)分别表示多个时间段，例如上述的1,3,5表示周一周三周五。
还可以用减号(-)表示一段时间，例如12-15表示12到15号。
可以用除号(/)表示执行任务的时间间隔，例如*/2表示每隔两分钟执行一次任务。

如果crond服务中需要同时包含多条计划任务的命令语句，每行应该只写一条，例如我们再添加一条计划任务，功能是每周一至周五的凌晨 1 点自动清空/tmp 目录内的所有文件。

使用-e参数编辑，添加一条新命令并保存退出即可：

```
25 3 * * 1,3,5 /usr/bin/tar -czvf backup.tar.gz /home/wwwroot
0 1 * * 1-5 /usr/bin/rm -rf /tmp/*
```

总结一下上述计划服务的注意事项：

1️⃣在 crond 服务的配置参数中，一般会像 Shell 脚本那样以#号开头写上注释信息，这样在日后回顾这段命令代码时可以快速了解其功能、需求以及编写人员等重要信息。

2️⃣计划任务中的“分钟”字段必须有数值，绝对不能为空或是*号，而“日”和“星期”字段不能同时使用，否则就会发生冲突。

删除 crond 计划任务则非常简单，直接使用 crontab -e 命令进入编辑界面，删除里面的文本信息即可。也可以使用 crontab -r 命令直接删除所有的计划任务。



# 第5章、用户身份与文件权限

## 5.1、用户身份与能力

### 1、用户UID

在 Linux 系统中，UID 就像我们的身份证号码一样具有唯一性，因此可通过用户的 UID 值来判断用户身份。在 RHEL 8 系统中，用户身份有下面这：

1️⃣管理员UID为0：系统的管理员用户。

2️⃣系统用户UID为1~999：Linux 系统为了避免因某个服务程序出现漏洞而被黑客提权至整台服务器，默认服务程序会由独立的系统用户负责运行，进而有效控制被破坏范围。

3️⃣普通用户UID从1000开始：是由管理员创建的用于日常工作的用户。

为了方便管理属于同一组的用户，Linux 系统中还引入了用户组的概念。通过使用用户组号码（GID，Group IDentification），可以把多个用户加入到同一个组中，从而方便为组中的用户统一规划权限或指定任务。

在 Linux 系统中创建每个用户时，将自动创建一个与其同名的基本用户组，而且这个基本用户组只有该用户一个人。该用户以后可能会被归纳到其他用户组，这个其他用户组称之为扩展用户组。

一个用户只有一个基本用户组，但是可以有多个扩展用户组。

### 2、id命令

id 命令用于显示用户的详细信息，语法格式为：

```
id 用户名
```

id命令能够简单轻松地查看用户的基本信息，例如用户 ID、基本组与扩展组 GID，以便于我们判别某个用户是否已经存在，以及查看相关信息。

使用 id 命令查看一个名称为 linuxprobe 的用户信息：

```
[root@linuxprobe~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
```

### 3、useradd命令

useradd 命令用于创建新的用户账户，语法格式为：

```
useradd [参数] 用户名
```

使用该命令创建用户账户时，默认的用户家目录会被存放在/home 目录中，默认的 Shell 解释器为/bin/bash，而且默认会创建一个与该用户同名的基本用户组。如果不想默认，需要添加参数。useradd 命令中的参数以及作用如下：

```
-d    指定用户的家目录（默认为/home/username）
-e    账户的到期时间，格式为 YYYY-MM-DD.
-u    指定该用户的默认 UID
-g    指定一个初始的用户基本组（必须已存在）
-G    指定一个或多个扩展用户组
-N    不创建与用户同名的基本用户组
-s    指定该用户的默认 Shell 解释器
```

使用 useradd 命令创建一个名称为 linuxcool 的用户，并使用 id 命令确认信息：

```
[root@linuxprobe~]# useradd linuxcool
[root@linuxprobe~]# id linuxcool
uid=1001(linuxcool) gid=1001(linuxcool) groups=1001(linuxcool)
```

创建一个普通用户并指定家目录的路径、用户的 UID 以及 Shell 解释器:

```
[root@linuxprobe~]# useradd -d /home/linux -u 8888 -s /sbin/nologin linuxdown
```

其中/sbin/nologin是终端解释器中的一员，与 Bash 解释器有着天壤之别，一旦用户的解释器被设置为 nologin，则代表该用户不能登录到系统中。

使用 id 命令确认信息：

```
[root@linuxprobe~]# id linuxdown
uid=8888(linuxdown) gid=8888(linuxdown) groups=8888(linuxdown)
```

### 4、groupadd命令

groupadd 命令用于创建新的用户组，语法格式为：

```
groupadd [参数] 群组名
```

创建一个用户组ronny：

```
[root@linuxprobe~]# groupadd ronny
```

### 5、usermod命令

usermod 命令用于修改用户的属性，英文全称为“user modify”，语法格式为：

```
usermod [参数] 用户名
```

Linux 系统中的一切都是文件，因此在系统中创建用户也就是修改配置文件的过程。用户的信息保存在/etc/passwd 文件中，可以直接用文本编辑器来修改其中的用户参数项目，也可以用 usermod 命令修改已经创建的用户信息。

usermod命令的参数以及作用如下：

```
-c 	  填写用户账户的备注信息
-d -m 参数-m与参数-d连用，可重新指定用户的家目录并自动把旧的数据转移过去
-e    账户的到期时间，格式为 YYYY-MM-DD
-g    变更所属用户组
-G    变更扩展用户组
-L    锁定用户禁止其登录系统
-U    解锁用户，允许其登录系统
-s    变更默认终端
-u    修改用户的 UID
```

将用户 linuxprobe 加入到 root 用户组中，这样扩展组列表中则会出现 root 用户组的字样，而基本组不会受到影响：

```
[root@linuxprobe ~]# usermod -G root linuxprobe
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe),0(root)
```

用-u 参数修改 linuxprobe 用户的 UID 号码值：

```
[root@linuxprobe ~]# usermod -u 6666 linuxprobe
[root@linuxprobe ~]# id linuxprobe
uid=6666(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe),0(root)
```

用-s参数修改linuxprobe用户的终端：

```
[root@linuxprobe ~]# usermod -s /sbin/nologin linuxprobe
[root@linuxprobe ~]# su - linuxprobe
This account is currently not available.
```

将用户的终端设置成/sbin/nologin 后用户马上就不能登录了,但这个用户依然可以被某个服务所调用，管理某个具体的服务。

### 6、passwd命令

passwd 命令用于修改用户的密码、过期时间等信息，英文全称为“password”，语法格式为：

```
passwd [参数] 用户名
```

普通用户只能使用 passwd 命令修改自己的系统密码，而 root 管理员则有权限修改其他所有人的密码，而且root 管理员在 Linux 系统中修改自己或他人的密码时不需要验证旧密码。passwd 命令中的参数以及作用如下：

```
-l      锁定用户，禁止其登录
-u      解除锁定，允许用户登录
--stdin 通过标准输入修改用户密码，如 echo "NewPassWord" | passwd --stdin Username
-d      使该用户可用空密码登录系统
-e      强制用户在下次登录时修改密码
-S      显示用户的密码是否被锁定，以及密码所采用的加密算法名称
```

要修改自己的密码，只需要输入命令后敲击回车即可：

```
[root@linuxprobe~]# passwd
Changing password for user root.
New password: 输入新密码
Retype new password: 确认新密码
passwd: all authentication tokens updated successfully.
```

要修改其他人的密码，则需要先检查当前是否为 root 管理员权限，然后在命令后指定要修改密码的那位用户的名称：

```
[root@linuxprobe~]# passwd linuxprobe
Changing password for user linuxprobe.
New password: 输入新密码
Retype new password: 确认新密码
passwd: all authentication tokens updated successfully.
```

锁定用户linuxprobe，禁止其登入系统：

```
[root@linuxprobe~]# passwd -l linuxprobe
Locking password for user linuxprobe.
passwd: Success
```

查看是否被锁定：

```
[root@linuxprobe ~]# passwd -S linuxprobe
linuxprobe LK 1969-12-31 0 99999 7 -1 (Password locked.)
```

对用户进行解锁，并查看：

```
[root@linuxprobe ~]# passwd -u linuxprobe
Unlocking password for user linuxprobe.
passwd: Success
[root@linuxprobe ~]# passwd -S linuxprobe
linuxprobe PS 1969-12-31 0 99999 7 -1 (Password set, SHA512 crypt.)
```

### 7、userdel命令

userdel 命令用于删除已有的用户账户，英文全称为“user delete”，语法格式为：

```
userdel [参数] 用户名
```

在执行删除操作时，该用户的家目录默认会保留下来，此时可以使用-r 参数将其删除。

userdel 命令的参数以及作用如下：

```
-f   强制删除用户
-r   同时删除用户及用户家目录
```

在删除一个用户时，一般会建议保留他的家目录数据，以免有重要的数据被误删除，等未来确认不需要的时候再手动删除就行。所以在使用 userdel 命令时可以不加参数，写清要删除的用户名称就行：

```
[root@linuxprobe ~]# userdel linuxdown
[root@linuxprobe ~]# id linuxdown
id: ‘linuxdown’: no such user
```

确认不需要是删除家目录数据：

```
[root@linuxprobe~]# cd /home
[root@linuxprobe home]# ls
linuxprobe linuxcool linuxdown
[root@linuxprobe home]# rm -rf linuxdown
[root@linuxprobe home]# ls
linuxprobe linuxcool
```

## 5.2、文件权限与归属

1、对于一般文件来说，可读(cat)，可写(vim)，可执行(./script)比较好理解；
对于目录文件来说，可读(ls)表示能够读取目录内的文件列表，可写(touch)表示能够在目录内新增、删除、重命名文件，可执行(cd)表示能够进入该目录。

2、文件的可读、可写、可执行权限的英文全称分别是 read、write、execute，可以简写为 r、w、x，亦可分别用数字 4、2、1 来表示。例如若某个文件的权限是7，表示可读可写可执行；若是6，表示可读可写。

3、文件的权限分配又分为文件所有者，文件所属组，其他用户。
例如在有一个文件，其所有者拥有可读、可写、可执行的权限，其文件所属组拥有可读、可写的权限；其他用户只有可读的权限。那么这个文件的权限就是rwxrw-r--，数字法表示为764。注意这里的减号(-)是占位符，代表这里无权限。注意r、w、x的位置不能改变，一定要严格按照顺序。

4、使用ls -l命令查看install.log的详细信息：

```
[root@linuxprobe~]# ls -l install.log
-rw-r--r-- 1 root root 34298 04-02 00:23 install.log
```

排在权限前面的减号（-）是文件类型（减号表示普通文件）；rw-r--r--表示所有者权限为可读、可写（rw-），所属组权限为可读（r--），其他人只有可读权限（r--），1指文件的硬链接计数，第一个root表示所有者，第二个root表示所属组，34298表示文件的磁盘占用大小，单位是字节，最近一次的修改时间是4月2日的00:23，文件名称为install.log。

5、常见文件类型表示方法如下：

```
-  普通文件
d  目录文件
l  链接文件
p  管道文件
b  块设备文件
c  字符设备文件
```

普通文件(-)：纯文本信息、服务配置信息、日志信息以及 Shell 脚本等。
块设备文件(b)和字符设备文件(c)一般指硬件设备，如鼠标、键盘、光驱、硬盘等。

## 5.3、文件的特殊权限

### 1、SUID

SUID 是一种对二进制程序进行设置的特殊权限，能够让二进制程序的执行者临时拥有所有者的权限（仅对拥有执行权限的二进制程序有效）。

文件被赋予SUID权限后(貌似是用chomd u+s file命令赋予)，文件所有者的权限会从rwx变成rws；如果原先所有者的权限是rw-，那么被赋予SUID权限后所有者的权限会变为rwS。

千万不要将 SUID 权限设置到 vim、cat、rm 等命令上面，因为一旦某个命令文件被设置了 SUID 权限，就意味着凡是执行该文件的人都可以临时获取到文件所有者所对应的更高权限。

### 2、chmod 命令

chmod 命令用于设置文件的一般权限及特殊权限，英文全称为“change mode”，语法格式为：

```
chmod [参数] 文件名
```

例如，要把一个文件的权限设置成其所有者可读可写可执行、所属组可读可写、其他人没有任何权限，则相应的字符法表示为rwxrw----，其对应的数字法表示为 760：

```
[root@linuxprobe~]# chomd 760 anaconda-ks.cfg
```

针对目录进行操作时需要加上大写参数-R 来表示递归操作，即对目录内所有的文件进行整体操作。

### 3、chown命令

chown命令用于设置文件的所有者和所有组，英文全称为 change own，语法格式为：

```
chown 所有者:所有组 文件名
```

```
[root@linuxprobe~]# ls -l anaconda-ks.cfg
-rw-------. 1 root root 1407 Jul 21 05:09 anaconda-ks.cfg
```

使用chown修改文件anaconda-ks.cfg的所有者和所有组：

```
[root@linuxprobe~]# chown linuxprobe:linuxprobe anaconda-ks.cfg
[root@linuxprobe~]# ls -l anaconda-ks.cfg
-rwxrw----. 1 linuxprobe linuxprobe 1407 Jul 21 05:09 anaconda-ks.cfg
```

针对目录进行操作时需要加上大写参数-R 来表示递归操作，即对目录内所有的文件进行整体操作。

### 4、SGID

SGID 特殊权限有两种应用场景：

1️⃣当对二进制程序进行设置时，能够让执行者临时获取文件所属组的权限；
例如，由于在平时需要查看系统的进程状态，为了能够获取进程的状态信息，可在用于查看系统进程状态的 ps 命令文件上增加 SGID 特殊权限位，下面查看ps命令文件的属性信息为：

```
-r-xr-Sr-x 1 bin system 59346 Feb 11 2017 ps
```

这样一来，由于 ps 命令被增加了 SGID 特殊权限位，所以当用户执行该命令时，也就临时获取到了 system 用户组的权限，从而顺利地读取到了设备文件。

2️⃣当对目录进行设置时，则是让目录内新创建的文件自动继承该目录原有用户组的名称。
如果现在需要在一个部门内设置共享目录，让部门内的所有人员都能够读取目录中的内容，那么就可以在创建部门共享目录后，在该目录上设置 SGID 特殊权限位：

首先在/tem中创建一个目录testdir：

```
[root@linuxprobe~]# cd /tmp
[root@linuxprobe tmp]# mkdir testdir
```

为该目录设置777权限，确保普通用户可以向其中写入文件：

```
[root@linuxprobe tmp]# chomd -R 777 testdir
```

为该目录设置SGID特殊权限位：

```
[root@linuxprobe tmp]# chomd -R g+s testdir
```

查看该目录信息：

```
[root@linuxprobe tmp]# ls -ald testdir
drwxrwsrwx. 2 root root 6 Oct 27 23:44 testdir
```

然后切换到普通用户linuxprobe，在该目录中创建文件，检查该文件是否继承了其所在的目录的所属组名称：

```
[root@linuxprobe tmp]# su - linuxprobe
[linuxprobe@linuxprobe~]$ cd /tmp/testdir
[linuxprobe@linuxprobe testdir]$ echo "linuxprobe.com" > test
[linuxprobe@linuxprobe testdir]$ ls -al test
-rw-rw-r--. 1 linuxprobe root 15 Oct 27 23:47 test
```

### 5、SBIT

SBIT 特殊权限位可确保用户只能删除自己的文件，而不能删除其他用户的文件。

当目录被设置 SBIT 特殊权限位后，文件的其他用户权限部分的 x 执行权限就会被替换成 t 或者 T，若原本有 x 执行权限则会写成 t，原本没有 x 执行权限则会被写成 T。

RHEL 8 系统中的/tmp 作为一个共享文件的目录，默认已经设置了 SBIT 特殊权限位，因此除非是该目录的所有者，否则无法删除这里面的文件：

```
[root@linuxprobe~]# ls -ald /tmp
drwxrwxrwt. 17 root root 4096 Oct 28 00:29 /tmp
```

为了测试，在/tmp目录下创建一个test文件并且赋予文件最大的777权限：

```
[root@linuxprobe~]# cd /tmp
[root@linuxprobe tmp]# echo "Welcome to linuxprobe.com" > test
[root@linuxprobe tmp]# chmod 777 test
```

随后切换到一个普通用户身份，尝试删除这个由其他人创建的文件，此时由于/tmp目录SBIT特殊权限位的存在，依然无法删除test文件：

```
[root@linuxprobe tmp]# su - linuxprobe
[linuxprobe@linuxprobe~]$ cd /tmp
[linuxprobe@linuxprobe tmp]$ rm -f test
rm: cannot remove 'test': Operation not permitted
```

### 6、特殊权限的设置

使用 chmod 命令设置特殊权限的参数如下：

```
u+s    设置 SUID 权限
u-s    取消 SUID 权限
g+s    设置 SGID 权限
g-s    取消 SGID 权限
o+t    设置 SBIT 权限
o-t    取消 SBIT 权限
```

例如为目录testdir设置SBIT权限：

```
[root@linuxprobe tmp]# chomd -R o+s testdir
```

除此之外，特殊权限也有数字表示法，SUID、SGID、SBIT分别表示4、2、1，数字权限的组成是“特殊权限+一般权限”，因此最大权限应该是7777。

如果权限是rwsrwSrwt，第一个小写s表示rwx且有SUID权限，第二个大写S表示rw-且有SGID权限，第三个小写t表示rwx且有SBIT权限，于是rwsrwSrwt转换成数字表示法就是7767。

反过来，若数字表示法为5537,5=4+1说明有SUID和SBIT特殊权限，537表示一般权限为r-x-wxrwx，再结合特殊权限就是r-s-wxrwt。

## 5.4、文件的隐藏属性

### 1、chattr命令

chattr 命令用于设置文件的隐藏权限，英文全称为 change attributes，语法格式为：

```
chattr [参数] 文件名称
```

如果想要把某个隐藏功能添加到文件上，则需要在命令后面追加“+参数”；
如果想要把某个隐藏功能移出文件，则需要追加“-参数”。

chattr 命令中的参数及其作用如下：

```
i   无法对文件进行修改；若对目录设置了该参数，则仅能修改其中的子文件内容而不能新建或删除文件   
a   仅允许补充（追加）内容，无法覆盖/删除内容（Append Only）
S   文件内容在变更后立即同步到硬盘（sync）
s   彻底从硬盘中删除，不可恢复（用零块填充原文件所在的硬盘区域）
A   不再修改这个文件或目录的最后访问时间（Atime）
b   不再修改文件或目录的存取时间
D   检查压缩文件中的错误
d   使用 dump 命令备份时忽略本文件/目录
c   默认将文件或目录进行压缩
u   当删除该文件后依然保留其在硬盘中的数据，方便日后恢复
t   让文件系统支持尾部合并（tail-merging）
x   可以直接访问压缩文件中的内容
```

新建一个普通文件，并为其设置“不允许删除与覆盖”（+a 参数）权限，然后再尝试将这个文件删除：

```
[root@linuxprobe~]# echo "for Test" > linuxprobe
[root@linuxprobe~]# chattr +a linuxprobe
[root@linuxprobe~]# rm linuxprobe
rm: remove regular file‘linuxprobe’? y
rm: cannot remove‘linuxprobe’: Operation not permitted
```

由于添加了隐藏属性，所以本来能被删除的文件不能被删除了。

### 2、lsattr命令

1、lsattr 命令用于查看文件的隐藏权限，英文全称为“list attributes”，语法格式为：

```
lsattr [参数] 文件名称
```

在 Linux 系统中，文件的隐藏权限必须使用 lsattr 命令来查看，平时使用的 ls 之类的命令则看不出端倪：

```
[root@linuxprobe~]# ls -al linuxprobe
-rw-r--r--. 1 root root 9 Feb 12 11:42 linuxprobe
```

使用 lsattr 命令，文件上被赋予的隐藏权限马上就会原形毕露:

```
[root@linuxprobe~]# lsattr linuxprobe
-----a---------- linuxprobe
```

按照显示的隐藏权限的类型（a），使用 chattr 命令将其去掉：

```
[root@linuxprobe~]# chattr -a linuxprobe
```

再去查看隐藏权限就发现没有了：

```
[root@linuxprobe~]# lsattr linuxprobe
---------------- linuxprobe
```

一般会将-a 参数设置到日志文件（/var/log/messages）上，这样可在不影响系统正常写入日志的前提下，防止黑客擦除自己的作案证据。

如果希望彻底地保护某个文件，不允许任何人修改和删除它的话，不妨加上-i 参数试试。

## 5.5、文件访问控制列表

前面讲解的权限都是针对一类用户设定的，能对很多用户同时生效。如果希望针对某个特定的用户进行单独的权限控制，就需要用到文件的访问控制列表(ACL)。

基于普通文件或目录设置 ACL 其实就是针对指定的用户或用户组设置文件或目录的操作权限，更加精准地派发权限。

如果针对某个目录设置了 ACL，则目录中的文件会继承其 ACL 权限；若针对文件设置了 ACL，则文件不再继承其所在目录的 ACL 权限。

### 1、setfacl命令

setfacl 命令用于管理文件的 ACL 权限规则，英文全称为“set files ACL”，语法格式为：

```
setfacl [参数] 文件名称
```

ACL 权限提供的是在所有者、所属组、其他人的读/写/执行权限之外的特殊权限控制。使用 setfacl 命令可以针对单一用户或用户组、单一文件或目录来进行读/写/执行权限的控制。

setfacl 命令中的参数以及作用如下：

```
-m    修改权限
-M    从文件中读取权限
-x    删除某个权限
-b    删除全部权限
-R    递归子目录
```

普通用户原本是无法进入/root目录中的，现在为普通用户单独设置一下权限：

```
[root@linuxprobe~]# setfacl -Rm u:linuxprobe:rwx /root
```

然后再再切换到普通用户身份就可以正常进入root了：

```
[root@linuxprobe~]# su - linuxprobe
[linuxprobe@linuxprobe~]$ cd /root
[linuxprobe@linuxprobe root]$
```

可以在管理员状态下查看root文件的权限，若最后一个点(.)变成了加号(+)，说明该文件已经被设置成了ACL：

```
[root@linuxprobe ~]# ls -ld /root
dr-xrwx---+ 15 root root 4096 Sep 18 11:13 /root
```

### 2、getfacl命令

getfacl 命令用于查看文件的 ACL 权限规则，英文全称为“get files ACL”，语法格式为：

```
getfacl [参数] 文件名称
```

查看root管理员家目录上设置的所有ACL信息：

```
[root@linuxprobe ~]# getfacl /root
getfacl: Removing leading '/' from absolute path names
# file: root
# owner: root
# group: root
user::r-x
user:linuxprobe:rwx  #给用户linuxprobe设置的ACL
group::r-x
mask::rwx
other::---
```

使用setfacl命令允许某个组的用户都可以读写/etc/fstab文件：

```
[root@linuxprobe~]# setfacl -m g:linuxprobe:rw /etc/fstab
```

使用getfacl命令查看/etc/fstab上的ACL信息：

```
[root@linuxprobe ~]# getfacl /etc/fstab
getfacl: Removing leading '/' from absolute path names
# file: etc/fstab
# owner: root
# group: root
user::rw-
group::r--
group:linuxprobe:rw- #给linuxprobe用户组设置的ACL
mask::rw-
other::r--
```

删除某个特定的ACL权限用参数-x：

```
[root@linuxprobe~]# setfacl -x g:linuxprobe /etc/fstab
```

检查ACL是否被删除：

```
[root@linuxprobe~]# getfacl /etc/fstab
getfacl: Removing leading '/' from absolute path names
# file: etc/fstab
# owner: root
# group: root
user::rw-
group::r--
mask::r--
other::r--
```

ACL 权限的设置都是立即且永久生效的，不需要再编辑什么配置文件。但是如果我们不小心设置错了权限，就会覆盖掉文件原始的权限信息，并且永远都找不回来了。

因此在操作前提前备份一下，总是个好习惯。备份目录的时候最好使用-R 递归参数，这样不仅能够把目录本身的权限进行备份，还能将里面的文件权限也自动备份。
例如，将/home目录的权限进行备份，使用getfacl备份的时候不能用绝对路径，要先切换到该目录。

```
[root@linuxprobe~]# cd /
[root@linuxprobe/]# getfacl -R home > backup.acl
```

ACL权限的恢复使用--restore参数，不需要在命令里面再写一次home了，因为backup.acl文件能自动找到home：

```
[root@linuxprobe /]# setfacl --restore backup.acl
```

## 5.6、su命令与sudo服务

### 1、su命令

su 命令可以解决切换用户身份的需求，使得当前用户在不退出登录的情况下，顺畅地切换到其他用户，比如从 root 管理员切换至普通用户：

```
[root@linuxprobe~]# su - linuxprobe
[linuxprobe@linuxprobe~]$
```

su 命令与用户名之间有一个减号（-），这意味着完全切换到新的用户，即把环境变量信息也变更为新用户的相应信息，而不是保留原始的信息。

root切换到其他用户不需要密码，而其他用户切换到root则需要输入管理员密码：

```
[linuxprobe@linuxprobe~]$ su - root
Password: 此处输入管理员密码
[root@linuxprobe~]#
```

上述直接切换到管理员用户可能会暴露密码，不太安全。

可以使用sudo命令把特定命令的执行权限赋予指定用户，这样既可保证普通用户能够完成特定的工作，也可以避免泄露 root 管理员密码。

### 2、sudo命令

sudo 命令用于给普通用户提供额外的权限，语法格式为：

```
sudo [参数] 用户名
```

使用sudo命令可以给普通用户提供额外的权限来完成原本只有 root管理员才能完成的任务。

sudo 命令中的可用参数以及作用：

```
-h     列出帮助信息
-l     列出当前用户可执行的命令
-u     用户名或 UID 值 以指定的用户身份执行命令
-k     清空密码的有效时间，下次执行 sudo 时需要再次进行密码验证
-b     在后台执行指定的命令
-p     更改询问密码的提示语
```

如果担心直接修改配置文件会出现问题，则可以使用 sudo 命令提供的 visudo 命令来配置用户权限。

visudo 命令用于编辑、配置用户 sudo 的权限文件，语法格式为“visudo [参数]”。
这是一条会自动调用 vi 编辑器来配置/etc/sudoers 权限文件的命令，能够解决多个用户同时修改权限而导致的冲突问题。不仅如此，visudo 命令还可以对配置文件内的参数进行语法检查，并在发现参数错误时进行报错提醒。

使用visudo命令配置权限文件时，其操作方法与Vim编辑器中用到的方法完全一致，按照下面的格式，在101行(大约)填上指定的信息：

```
谁可以使用 允许使用的主机 = (以谁的身份) 可执行的命令列表
```

1️⃣谁可以使用：稍后要为哪位用户进行命令授权。

2️⃣允许使用的主机：可以填写 ALL 表示不限制来源的主机，亦可填写如 192.168.10.0/24这样的网段限制来源地址，使得只有从允许网段登录时才能使用 sudo 命令。

3️⃣以谁的身份：可以填写 ALL 表示系统最高权限，也可以是另外一位用户的名字。

4️⃣可执行命令的列表：可以填写 ALL 表示不限制命令，亦可填写如/usr/bin/cat 这样的文件名称来限制命令列表，多个命令文件之间用逗号（,）间隔。

先使用visudo命令打开编辑器：

```
[root@linuxprobe~]# visudo
```

在101行填写上述的信息，然后保存并退出：

```
99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) ALL
```

切换到linuxprobe用户之后，使用-l参数查看当前可执行的命令：

```
[root@linuxprobe~]# su - linuxprobe
[linuxprobe@linuxprobe~]$ sudo -l
```

接下来就是见证奇迹的时刻，正常普通用户并不能进入管理员的家目录/root的：

```
[linuxprobe@linuxprobe~]$ ls /root
ls: cannot open directory '/root': Permission denied
```

只要在想执行的命令前加上sudo命令即可：

```
[linuxprobe@linuxprobe~]$ sudo ls /root
anaconda-ks.cfg Documents initial-setup-ks.cfg Pictures Templates
Desktop Downloads Music Public Videos
```

不过基本上不会给某个用户root的所有指令权限，若给出某个指令权限，记得一定要给出指令的绝对路径，指令之间用逗号(,)隔开：

```
[root@linuxprobe~]# visudo
99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) /user/bin/cat,/user/sbin/reboot
```

在每次执行 sudo 命令后都会要求验证一下密码(当前登录用户的)，可以添加 NOPASSWD 参数，使得用户下次再执行 sudo 命令时就不用密码验证：

```
[root@linuxprobe~]# visudo
99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) NOPASSWD:/usr/bin/cat,/usr/sbin/reboot
```

另外，visudo命令只有root管理员才能执行。

问答小题：如何使用访问控制列表（ACL）来限制 linuxprobe 用户组，使得该组中的所有成员不得在/tmp目录中写入内容？

答：想要设置用户组的 ACL，则需要把 u 改成 g，即 setfacl -Rm g:linuxprobe:r-x /tmp。  把rwx中的w去掉了 即不能写了。



# 第6章、存储结构与管理硬盘

## 6.1、一切从“/”开始

1、在 Linux 系统中，目录、字符设备、套接字、硬盘、光驱、打印机等都被抽象成文件形式，即“Linux 系统中一切都是文件”。

Linux 系统中的一切文件都是从“根”目录（/）开始的，而且Linux 系统中的文件和目录名称是严格区分大小写的，文件名称中不得包含斜杠(/)。

2、在 Linux 系统中，最常见的目录以及所对应的存放内容如下：

```
/boot   开机所需文件—内核、开机菜单以及所需配置文件等
/dev    以文件形式存放任何设备与接口
/etc    配置文件
/home   用户主目录
/bin    存放单用户模式下还可以操作的命令
/lib    开机时用到的函数库，以及/bin 与/sbin 下面的命令要调用的函数
/sbin   开机过程中需要的命令
/media  用于挂载设备文件的目录
/opt    放置第三方的软件
/root   系统管理员的家目录
/srv    一些网络服务的数据文件目录
/tmp    任何人均可使用的“共享”临时目录
/proc   虚拟文件系统，例如系统内核、进程、外部设备及网络状态等
/usr/local    用户自行安装的软件
/usr/sbin Linux 系统开机时不会使用到的软件/命令/脚本
/usr/share    帮助与说明文件，也可放置共享文件
/var    主要存放经常变化的文件，如日志
/lost+found   当文件系统发生错误时，将一些丢失的文件片段存放在这里
```

## 6.2、物理设备的命名规则

1、Linux 系统中常见的硬件设备及其文件名称如下：

```
IDE设备        /dev/hd[a-d]
SCSI/SATA/U盘  /dev/sd[a-z]
Virtio设备     /dev/vd[a-z]
软驱           /dev/fd[0-1]
打印机         /dev/lp[0-15]
光驱           /dev/cdrom
鼠标           /dev/mouse
磁带机         /dev/st0 或/dev/ht0
```

由于现在的 IDE 设备已经很少见了，所以一般的硬盘设备都是以“/dev/sd”开头，而一台主机上可以有多块硬盘，因此系统采用 a～z 来代表 26 块不同的硬盘的硬盘序列号（默认从 a 开始分配）。
除此之外，硬盘的分区编号也很有讲究：
1️⃣主分区或扩展分区的编号从 1 开始，到 4 结束；
2️⃣逻辑分区从编号 5 开始。

2、两个知识误区：

1️⃣/dev 目录中 sda 设备之所以是 a，并不是由插槽决定的，而是由系统内核的识别顺序来决定的，只是恰巧很多主板的插槽顺序就是系统内核的识别顺序。

2️⃣ sda3 只能表示是分区编号为 3 的分区，而不能判断 sda 设备上已经存在了 3 个分区。

例如：/dev/sda5中：
/dev/是硬件设备文件所在的目录；
sd表示的是SCSI存储设备；
a表示硬盘的顺序号(以a,b,c……表示),a代表是系统中同类接口里第一个被识别到的设备；
5 表示分区的顺序号，顺序号大于等于5说明这个设备是一个逻辑分区；
一言蔽之，/dev/sda5”表示的就是“这是系统中第一块被识别到的硬件设备中分区编号为 5 的逻辑分区的设备文件”。

小问答：/dev/hdc8 代表着什么？
答：这是第 3 块 IDE 设备中编号为 8 的逻辑分区。

3、硬盘设备是由大量的扇区组成的，每个扇区的容量为 512 字节，对于第一个扇区，其512字节分配为：主引导记录需要占用 446 字节，分区表占用 64 字节，结束符占用 2 字节。而记录一个分区信息需要16字节，于是第一个扇区最多只能记录4个分区的信息。每个分区的信息包括关于分区的起始扇区、大小、类型等信息，并不是说分区只有16字节大小。
硬盘中除了第一个扇区外的其他扇区通常包含文件系统的数据和元数据，如文件、目录结构、文件属性等。只有第一个扇区的512字节才是上述那么分配的，remember。

这样一看，一个硬盘最多只有四个分区？no
一般解决方法是，四个分区中留三个做主分区，还有一个做扩展分区，扩展分区中16个字节的分区信息，指向了逻辑分区，逻辑分区的信息（通常是16字节）包含了该逻辑分区的起始位置和大小，以及指向下一个逻辑分区的指针，类似于单链表结构。扩展分区自身不能存储数据，用户需要在其指向的逻辑分区上进行操作。

## 6.3、文件系统与数据资料

1、用户在硬件存储设备中执行的文件建立、写入、读取、修改、转存与控制等操作都是依靠文件系统来完成的。

文件系统的作用是合理规划硬盘，以保证用户正常的使用需求。

Linux 系统支持数十种文件系统，如：Ext3，Ext4，XFS等。RHEL6使用Ext4，RHEL7/8使用XFS，

2、在拿到一块新的硬盘存储设备后，先需要分区，然后再格式化文件系统，最后才能挂载并正常使用。

就像拿到了一张未裁切的完整纸张那样，首先要进行裁切以方便使用（分区），接下来在裁切后的纸张上画格以便能书写工整（格式化），最后是正式的使用（挂载）。

3、为了使用户在读取或写入文件时不用关心底层的硬盘结构，Linux 内核中的软件层为用户程序提供了一个虚拟文件系统(VFS)接口，这样用户实际上在操作文件时就是统一对这个虚拟文件系统进行操作，而不用考虑不同文件系统(比如Ext4和XFS)操作起来的差异了。

## 6.4、挂载硬件设备

挂载的通俗解释：当用户需要使用硬盘设备或分区中的数据时，需要先将其与一个已存在的目录文件进行关联，而这个关联动作就是“挂载”。
只需使用mount 命令把硬盘设备或分区与一个目录文件进行关联，然后就能在这个目录中看到硬件设备中的数据了。

### 1、mount命令

mount 命令用于挂载文件系统，格式为：

```
mount 文件系统 挂载目录
```

挂载是在使用硬件设备前所执行的最后一步操作。
mount 命令中的参数以及作用如下：

```
-a    挂载所有在/etc/fstab 中定义的文件系统
-t    指定文件系统的类型
```

对于比较新的 Linux 系统来讲，一般不需要使用-t 参数来指定文件系统的类型，Linux 系统会自动进行判断.

mount 中的-a 参数则厉害了，它会在执行后自动检查/etc/fstab文件中有无被疏漏挂载的设备文件，如果有，则进行自动挂载操作。

例如，把设备/dev/sdb2挂在到/backup目录：

```
[root@linuxprobe ~]# mount /dev/sdb2 /backup
```

如果在工作中挂载一块网络存储设备，其设备名称可能会发生变化，这时候/dev/sdb2可能不对，可以使用UUID来进行挂载。

UUID 是一串用于标识每块独立硬盘的字符串，具有唯一性及稳定性，特别适合用来挂载网络设备。

可以使用blkid命令来现实设备的属性信息，进而查询到我们想要挂载设备的UUID：

```
[root@linuxprobe~]# blkid
/dev/sdb1: UUID="2db66eb4-d9c1-4522-8fab-ac074cd3ea0b" TYPE="xfs" PARTUUID="eb23857a-01"
/dev/sdb2: UUID="478fRb-1pOc-oPXv-fJOS-tTvH-KyBz-VaKwZG" TYPE="ext4" PARTUUID="e
b23857a-02"
```

查到了/dev/sdb2的UUID为478fRb-1pOc-oPXv-fJOS-tTvH-KyBz-VaKwZG，可以使用其进行网络设备的挂载：

```
[root@linuxprobe~]# mount UUID=478fRb-1pOc-oPXv-fJOS-tTvH-KyBz-VaKwZG /backup
```

上述挂载在重启之后就会失效，如果想让硬件设备和目录永久地进行自动关联，就必须把挂载信息按照指定的填写格式写入到/etc/fstab文件中，挂载信息的填写格式为：

```
设备文件 挂载目录 格式类型 权限选项 是否备份 是否自检
```

上述信息各字段的意义如下：

```
设备文件   一般为设备的路径+设备名称，也可以写通用唯一识别码（UUID）

挂载目录   指定要挂载到的目录，需在挂载前创建好

格式类型   指定文件系统的格式，比如 Ext3、Ext4、XFS、SWAP、iso9660（此为光盘设备）等

权限选项   若设置为 defaults，则默认权限为 rw、suid、dev、exec、auto、nouser、async

是否备份   若为 1 则开机后使用 dump 进行磁盘备份，为 0 则不备份

是否自检   若为 1 则开机后自动进行磁盘自检，为 0 则不自检
```

例如，想将文件系统为 Ext4 的硬件设备/dev/sdb2 在开机后自动挂载到/backup 目录上，并保持默认权限且无须开机自检，就需要在/etc/fstab 文件中写入下面的信息：

```
/dev/sdb2 /backup ext4 defaults 0 0
```

当然还是使用vim编辑器进行编辑添加内容。

把光盘设备/dev/cdrom挂载到/media/cdrom 目录中，光盘设备的文件系统格式是 iso9660，需要在/etc/fstab文件中写入下面的信息：

```
/dev/cdrom /media/cdrom iso9660 defaults 0 0
```

写入到/etc/fstab 文件中的设备信息并不会立即生效，需要使用 mount -a 参数进行自动挂载：

```
[root@linuxprobe~]# mount -a
```

挂载网络存储设备时，可以在/etc/fstab文件挂载信息中加上_netdev参数：

```
/dev/sdb2 /backup ext4 defaults,_netdev 0
0
```

加上后系统会等联网成功后再尝试挂载这块网络存储设备，从而避免了开机时间过长或失败的情况。

### 2、df命令

df 命令用于查看已挂载的磁盘空间使用情况，英文全称为“disk free”，语法格式为：

```
df -h
```

可以使用df命令查看当前系统中设备的挂载情况，它不仅能够列出系统中正在使用的设备有哪些，还可以用-h 参数便捷地对存储容量进行“进位”操作，例如如果遇到10240K，会自动显示成10M，便于阅读。

```
[root@linuxprobe~]# df -h 
```

### 3、umount命令

挂载文件系统的目的是为了使用硬件资源，而卸载文件系统则意味不再使用硬件的设备资源。

umount 命令用于卸载设备或文件系统，英文全称为“un mount”，语法格式为：

```
unmount [设备文件/挂载目录]
```

取消关联的设备文件或挂载目录的其中一项即可，一般不需要加其他额外的参数。

```
[root@linuxprobe~]# unmount /dev/sdb2
```

如果我们要umount我们当前所在的目录，系统会提示设备繁忙，此时需要切换到其他目录重新进行umount：

```
[root@linuxprobe cdrom]# umount /dev/cdrom
umount: /media/cdrom: target is busy.
[root@linuxprobe cdrom]# cd~
[root@linuxprobe~]# umount /dev/cdrom
```

设备文件和挂载目录被挂载之后，它们都不能重复挂载到其他地方。

如果硬盘特别多，分区特别多，可以使用lsblk命令以树状图的形式列举一下：

```
[root@linuxprobe~]# lsblk
```

## 6.5、添加硬盘设备

添加硬盘设备的操作思路：首先需要在虚拟机中模拟添加入一块新的硬盘存储设备，然后再进行分区、格式化、挂载等操作，最后通过检查系统的挂载状态并真实地使用硬盘来验证硬盘设备是否成功添加。

向虚拟机中添加一块硬盘之后，根据udev 服务命名规则，第二个被识别的 SATA 设备应该会被保存为/dev/sdb，这个就是硬盘设备文件了。

### 1、fdisk命令

fdisk 命令用于新建、修改及删除磁盘的分区表信息，英文全称为“format disk”，语法格式为：

```
fdisk 磁盘名称
```

fdisk的参数不是直接写到命令后面的形式，而是一问一答的形式，好处就是可以根据需求动态调整参数。

fdisk 命令中的参数以及作用如下：

```
m    查看全部可用的参数
n    添加新的分区
d    删除某个分区信息
l    列出所有可用的分区类型
t    改变某个分区的类型
p    查看分区表信息
w    保存并退出
q    不保存直接退出
```

使用fdisk命令来尝试管理/dev/sdb硬盘设备，根据提示信息输入参数p来查看硬盘设备内已有的分区信息：

```
[root@linuxprobe~]# fdisk /dev/sdb
  
    #省略

Command (m for help): p
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x88b2c2b0
```

同时我们也可以通过参数n自己添加新的分区，系统会询问创建的是主分区(p)还是扩展分区(e)，这里创建主分区：

```
Command (m for help): n
Partition type
p primary (0 primary, 0 extended, 4 free)
e extended (container for logical partitions)
Select (default p): p
```

确认主分区之后，会提示输入主分区的编号：

```
Partition number (1-4, default 1): 1
```

接下来系统会提示用户定义起始的扇区位置，这不需要改动，敲击回车键保留默认设置即可，系统会自动计算出最靠前的空闲扇区的位置：

```
First sector (2048-41943039, default 2048): 此处敲击回车键即可
```

最后，系统会要求用户定义分区的结束扇区位置，这其实就是要去定义整个分区的大小是多少。我们不用去计算扇区的个数，只需要输入+2G 即可创建出一个容量为 2GB的硬盘分区：

```
Last sector, +sectors or +size{K,M,G,T,P} (2048-41943039, default 41943039):+2G
```

此时会显示分区创建成功：

```
Created a new partition 1 of type 'Linux' and of size 2 GiB.
```

创建之后再用参数p产看分区信息，即可看到到一个名称为/dev/sdb1、起始扇区位置为 2048、结束扇区位置为 4196351 的主分区了。

```
Command (m for help): p
Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x88b2c2b0
Device Boot Start End Sectors Size Id Type
/dev/sdb1 2048 4196351 4194304 2G 83 Linux
```

然后再敲击参数w保存退出：

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

在上述步骤执行完毕之后，Linux 系统会自动把这个硬盘主分区抽象成/dev/sdb1 设备文件。此时使用file命令查看/dev/sdb1 设备文件可能会报错：

```
[root@linuxprobe ]# file /dev/sdb1
/dev/sdb1: cannot open `/dev/sdb1' (No such file or directory)
```

那是因为有些时候系统并没有自动把分区信息同步给 Linux 内核，需要我们手工输入partprobe命令将分区信息同步到内核，一般连续执行两次。

```
[root@linuxprobe ]# partprobe
[root@linuxprobe ]# partprobe
[root@linuxprobe ]# file /dev/sdb1
/dev/sdb1: block special
```

分区创建好之后，需要对硬件存储设备进行格式化操作，使用的命令为mkfs。

```
[root@linuxprobe~]# mkfs
```

输入mkfs命令之后连续敲击两下tab键会出现针对不同文件系统初始化的命令文件：

```
[root@linuxprobe~]# mkfs
mkfs mkfs.ext2 mkfs.ext4 mkfs.minix mkfs.vfat
mkfs.cramfs mkfs.ext3 mkfs.fat mkfs.msdos mkfs.xfs
```

例如要将分区为 XFS 的文件系统进行格式化：

```
[root@linuxprobe~]# mkfs.xfs /dev/sdb1
```

完成了存储设备的分区和初始化操作之后，接下来要来挂载并使用存储设备了：

1️⃣创建一个用于挂载设备的挂载点目录：

```
[root@linuxprobe~]# mkdir /newFS
```

2️⃣使用mount命令将存储设备与挂载点进行关联：

```
[root@linuxprobe~]# mount /dev/sdb1 /newFS
```

3️⃣使用df -h命令来查看挂载状态和磁盘使用量信息：

```
[root@linuxprobe~]# df -h
/dev/sdb1 2.0G 47M 2.0G 3% /newFS
```

### 2、du命令

du 命令用查看分区或目录所占用的磁盘容量大小，英文全称为“disk usage”，语法格式为：

```
du -sh 目录名称
```

查看Linux系统根目录下所有一级目录分别占用的空间大小：

```
[root@linuxprobe~]# du -sh /*
```

存储设备已经顺利挂载后，就可以尝试通过挂载点目录向存储设备中写入文件了。

先从某些目录中复制过来一批文件，然后查看这些文件总共占用了多大的容量：

```
[root@linuxprobe~]# cp -rf /etc/* /newFS
[root@linuxprobe~]# du -sh /newFS
39M /newFS/
```

为了使挂载永久有效，不要忘记把挂载信息写入配置文件/etc/fstab中：

```
/dev/sdb1 /newFS xfs defaults 0 0
```

## 6.6、添加交换分区

1、交换分区技术：在硬盘中预先划分一定的空间，把内存中暂时不常用的数据临时存放到硬盘中，以便腾出物理内存空间让更活跃的程序服务来使用。

其设计目的是让硬盘帮内存分担压力，解决真实物理内存不足的问题。

2、交换分区的创建过程与前文讲到的挂载并使用存储设备的过程非常相似。

这里取出一个大小为 5GB 的主分区作为交换分区资源，除了分配的主分区大小是5G，其他操作跟上述分区1的分配一样，下面只写关键代码：

```
[root@linuxprobe~]# fdisk /dev/sdb
Command (m for help): n
Select (default p): p
Partition number (2-4, default 2):敲回车即可
First sector (4196352-41943039, default 4196352):敲回车即可
Last sector, +sectors or +size{K,M,G,T,P} (4196352-41943039, default 41943039): +5G

Created a new partition 2 of type 'Linux' and of size 5 GiB
```

创建好新的分区之后，使用t参数将硬盘的标识码修改为82(Linux swap)，方便以后知晓其作用：

```
Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 82

Changed type of partition 'Linux' to 'Linux swap / Solaris'.
```

然后敲击w保存并退出：

```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

3、两个与分区交换相关的简单命令：

1️⃣mkswap命令，英文全称为“make swap”，用于对新设备进行交换分区格式化，语法格式为：

```
mkswap 设备名称
```

将我们刚刚新创建的交换分区格式化：

```
[root@linuxprobe~]# mkswap /dev/sdb2
```

2️⃣swapon 命令，英文全称为“swap on”，用于激活新的交换分区设备，语法格式为：

```
swapon 设备名称
```

将我们刚刚准备好的swap硬盘设备正式挂载到系统中：

```
[root@linuxprobe~]# swapon /dev/sdb2
```

4、上述步骤执行完之后，为了能够让新的交换分区设备在重启后依然生效，需要按照下面的格式将相关信息写入配置文件/etc/fstab中，并记得保存：

```
/dev/sdb2 swap swap defaults 0 0
```

## 6.7、磁盘容量配额

1、Linux 系统的设计初衷就是让许多人一起使用并执行各自的任务，从而成为多用户、多任务的操作系统。但是，硬件资源是固定且有限的，因此root管理员就需要使用磁盘容量配额服务来限制某位用户或某个用户组针对特定文件夹可以使用的最大硬盘空间或最大文件个数，一旦达到这个最大值就不再允许继续使用。

可以使用quota技术进行磁盘容量配额管理，quota技术还有软限制和硬限制的功能：

1️⃣软限制：当达到软限制时会提示用户，但仍允许用户在限定的额度内继续使用。

2️⃣硬限制：当达到硬限制时会提示用户，且强制终止用户的操作。

2、早期的 Linux 系统要想让硬盘设备支持 quota 磁盘容量配额服务，使用的是usrquota 参数，而 RHEL 7/8 系统使用的则是 uquota 参数，参数添加在/etc/fstab文件中defaults后面：

```
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot xfs defaults,uquota 1 2
```

然后重启系统，便可支持quota磁盘配额技术了。

3、最后创建一个个用于检查 quota 磁盘容量配额效果的用户 tom，并针对/boot 目录增加其他人的写权限，保证用户能够正常写入数据：

```
[root@linuxprobe~]# useradd tom
[root@linuxprobe~]# chmod -R o+w /boot
```

xfs_quota 命令用于管理设备的磁盘容量配额，语法格式为：

```
xfs_quota [参数] 配额 文件系统
```

其中：

-c参数用于以参数的形式设置要执行的命令；

-x参数是专家模式，让运维人员能够对 quota服务进行更多复杂的配置。

例如，使用 xfs_quota 命令来设置用户 tom 对/boot 目录的 quota 磁盘容量配额，其中硬盘使用量的软限制和硬限制分别为 3MB 和 6MB；创建文件数量的软限制和硬限制分别为 3 个和 6 个：

```
[root@linuxprobe~]# xfs_quota -x -c 'limit bsoft=3m bhard=6m isoft=3 ihard=6 tom' /boot
```

当配置好上述各种软硬限制后，尝试切换到用户tom，然后尝试创建一个体积为 5MB文件：

```
[tom@linuxprobe~]$ cd /boot
[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=5M count=1 
1+0 records in
1+0 records out
5242880 bytes (5.2 MB, 5.0 MiB) copied, 0.00298178 s, 1.8 GB/s
```

然后再创建8MB的文件：

```
[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=8M count=1
dd: error writing '/boot/tom': Disk quota exceeded
1+0 records in
0+0 records out
4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.00398607 s, 1.1 GB/s
```

此时创建失败了。

4、edquota 命令用于管理系统的磁盘配额，英文全称为“edit quota”，语法格式为：

```
edquota [参数] 用户名
```

edquota 命令中可用的参数以及作用如下：

```
-u    对某个用户进行设置
-g    对某个用户组进行设置
-p    复制原有的规则到新的用户/组
-t    限制宽限期限
```

edquota 命令会调用 Vi 或 Vim 编辑器来让 root 管理员修改要限制的具体细节，记得用wq 保存退出。下面把用户 tom 的硬盘使用量的硬限额提升到 8MB：

```
[root@linuxprobe~]# edquota -u tom
Disk quotas for user tom (uid 1001):
Filesystem blocks soft hard inodes soft hard
/dev/sda1 4096 3072 8192 1 3 6
```

然后再去创建8MB文件，即成功创建：

```
[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=8M count=1
1+0 records in
1+0 records out
8388608 bytes (8.4 MB, 8.0 MiB) copied, 0.0185476 s, 452 MB/s
```

## 6.8、VDO（虚拟数据优化）

1、VDO(virtual data optimize)是一种通过压缩或删除存储设备上的数据来优化存储空间的技术。

VDO技术的关键就是对硬盘内原有的数据进行删重操作，它有点类似于我们平时使用的网盘服务，在第一次正常上传文件时速度特别慢，在第二次上传相同的文件时仅作为一个数据指针，几乎可以达到“秒传”的效果，无须再多占用一份空间。

除了删重操作，VDO 技术还可以对日志和数据库进行自动压缩，进一步减少存储浪费的情况。

2、如果系统没有安装vdo，可以使用dnf命令进行安装：

```
[root@linuxprobe~]# dnf install kmod-kvdo vdo
```

首先创建一个全新的VDO卷，新添加进来的物理设备使用 vdo 命令来管理：

```
[root@linuxprobe~]# vdo create --name=storage --device=/dev/sdc --vdoLogicalSize=200G
```

name参数代表新的设备卷的名称；device参数代表参数由哪块磁盘进行制作；vdoLogicalSize 参数代表制作后的设备大小。

在创建成功后，使用 status 参数查看新建卷的概述信息：

```
[root@linuxprobe~]# vdo status --name=storage
```

接下来，对新建卷进行格式化操作并挂载使用。

新建的 VDO 卷设备会被存放在/dev/mapper 目录下，并以设备名称命名，对它操作就行。另外，挂载前可以用 udevadm settle 命令对设备进行一次刷新操作，避免刚才的配置没有生效：

```
[root@linuxprobe~]# mkfs.xfs /dev/mapper/storage
[root@linuxprobe~]# udevadm settle
[root@linuxprobe~]# mkdir /storage
[root@linuxprobe~]# mount /dev/mapper/storage /storage
```

如果想查看设备的实际使用情况，使用 vdostats 命令即可。human-readable 参数的作用是将存储容量自动进位，以人们更易读的方式输出（比如，显示 20G 而不是 20971520K）：

```
[root@linuxprobe~]# vdostats --human-readable
Device Size Used Available Use% Space saving%
/dev/mapper/storage 20.0G 4.0G 16.0G 20% 99%
```

这里的Size是实际物理存储的空间大小（即 20.0GB 是硬盘的大小），如果想看逻辑存储空间，可以使用 df 命令进行查看：

```
[root@linuxprobe~]# df -h
/dev/mapper/storage 200G 2.4G 198G 2% /storage
```

VDO 设备卷在创建后会一直存在，但需要手动编辑/etc/fstab 文件后才能在下一次重启后自动挂载生效，为我们所用。

## 6.9、软硬方式链接

1、在 Windows 系统中，快捷方式就是指向原始文件的一个链接文件，可以让用户从不同的位置来访问原始的文件；原文件一旦被删除或剪切到其他地方，会导致链接文件失效。

2、linux系统中存在软链接和硬链接两种不同的类型：

1️⃣软链接：也叫符号链接，仅仅包含所链接文件的名称和路径，很像一个记录地址的标签。它与 Windows 系统的“快捷方式”具有一样的性质。

2️⃣硬链接：在文件系统级别创建了一个新的链接，使得多个文件名指向同一个数据块。也就是说硬链接文件与原始文件其实是一模一样的，只是名字不同，因此即便原始文件被删除，依然可以通过硬链接文件来访问。每添加一个硬链接，该文件的 inode 个数就会增加 1，只有当该文件的 inode 个数为 0 时，才算彻底将它删除(有点像shared_ptr)。

### 3、ln命令

ln 命令用于创建文件的软硬链接，英文全称为“link”，语法格式为：

```
ln [参数] 原始文件名 链接文件名
```

ln命令中的可用参数以及作用如下：

```
-s    创建软连接(如果不带-s参数,则默认创建硬链接）
-f    强制创建文件或目录的链接
-i    覆盖前先询问
-v    显示创建链接的过程
```

是否添加-s 参数，将创建出性质不同的两种“快捷方式”。

创建一个文件old.txt：

```
[root@linuxprobe~]# echo "Welcome to linuxprobe.com" > old.txt
```

1️⃣为old.txt创建一个软链接：

```
[root@linuxprobe~]# ln -s old.txt new.txt
[root@linuxprobe~]# cat new.txt
Welcome to linuxprobe.com
```

原始文件名为 old，新的软链接文件名为 new，删除原始文件old之后，软链接文件立即无法读取了：

```
[root@linuxprobe~]# rm -f old.txt
[root@linuxprobe~]# cat new.txt
cat: readit.txt: No such file or directory
```

2️⃣为old.txt创建一个硬链接，即相当于针对原始文件的硬盘存储位置创建了一个指针。

```
[root@linuxprobe~]# ln old.txt new.txt
[root@linuxprobe~]# cat new.txt
Welcome to linuxprobe.com
```

删除原始文件old，new还是能读取数据：

```
[root@linuxprobe~]# rm -f old.txt
[root@linuxprobe~]# cat new.txt
Welcome to linuxprobe.com
```

与此同时，创建的硬链接文件竟然会让文件属性第二列的数字变成了2，这个数字2表示的就是文件的 inode 信息块的数量。

```
[root@linuxprobe~]# ls -l old.txt
-rw-r--r-- 2 root root 26 Jan 11 00:13 old.txt
```

即便删除了原始文件，新的文件也会一如既往地可以读取，因为只有当文件 inode 数量被“清零”时，才真正代表这个文件被删除了。

## 6.10、本章复习题

1、/home 目录与/root 目录内存放的文件有何相同点以及不同点？

答：这两个目录都是用来存放用户家目录数据的，但是，/root 目录存放的是 root 管理员的家目录数据。

2、如果硬盘中需要 5 个分区，则至少需要几个逻辑分区？

答：可以选用创建 3 个主分区+1 个扩展分区的方法，然后把扩展分区再分成 2 个逻辑分区，即有了 5 个分区。

3、在配置 quota 磁盘容量配额服务时，软限制数值必须小于硬限制数值么？

答：不一定，软限制数值可以小于等于硬限制数值。

4、VDO 技术能够提升硬盘的物理存储空间么？

答：不可以，VDO 是通过压缩或删重操作来提高硬盘的逻辑空间大小。



# 第7章、使用RAID与LVM磁盘阵列技术

## 7.1、RAID（独立冗余磁盘阵列）

1、RAID技术通过把多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个区段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果。

企业更看重的则是 RAID 技术所具备的冗余备份机制以及带来的硬盘吞吐量的提升，尽管它提高了成本支出。

也就是说，RAID 不仅降低了硬盘设备损坏后丢失数据的几率，还提升了硬盘设备的读写速度，所以它在绝大多数运营商或大中型企业中得到了广泛部署和应用。

2、RAID 0、1、5、10 方案技术对比，其中n代表硬盘总数：

| RAID级别 | 最少硬盘 | 可用容量 | 读写性能 | 安全性 |                             特点                             |
| :------: | :------: | :------: | :------: | :----: | :----------------------------------------------------------: |
|    0     |    2     |    n     |    n     |   低   |   追求最大容量和速度；但是任何一块硬盘损坏，数据将全部异常   |
|    1     |    2     |   n/2    |    n     |   高   |   追求最大安全性，只要阵列中有一块硬盘可用，数据就不受影响   |
|    5     |    3     |   n-1    |   n-1    |   中   | 在控制成本的前提下，追求硬盘的最大容量、速度及安全性，允许有一块硬盘出现异常，且数据不受影响 |
|    10    |    4     |   n/2    |   n/2    |   高   | 综合 RAID 1 和 RAID 0 的优点，追求硬盘的速度和安全性，允许有一半硬盘出现异常（不可发生在同一阵列中），且数据不受影响 |

### 7.1.1、RAID0

RAID 0 技术把多块物理硬盘设备（至少两块）通过硬件或软件的方式串联在一起，组成一个大的卷组，并将数据依次写入各个物理硬盘中，也就是说，数据是分开存放的，任意一块硬盘发生故障，将导致整个系统的数据都受到破坏。

RAID 0 技术能够有效地提升硬盘设备的读写性能，但是不具备数据备份和错误修复能力。

### 7.1.2、RAID1

如果生产环境对硬盘设备的读写速度没有要求，而是希望增加数据的安全性时，就需要用到 RAID1 技术了。

RAID1技术是将数据同时写入到多块硬盘设备上（可以将其视为数据的镜像或备份）。当其中某一块硬盘发生故障后，一般会立即自动以热交换的方式来恢复数据的正常使用。

RAID 1 技术虽然十分注重数据的安全性，但是因为是在多块硬盘设备中写入了相同的数据，因此使得硬盘设备的利用率下降。例如由 3 块硬盘设备组成的 RAID 1 磁盘阵列的可用率只有 33%左右。

### 7.1.3、RAID5

RAID5 技术是把硬盘设备的数据奇偶校验信息保存到其他硬盘设备中，且奇偶校验信息并不是单独保存到某一块硬盘设备中，而是存储到除自身以外的其他每一块硬盘设备上。

RAID 5 技术实际上没有备份硬盘中的真实数据信息，而是当硬盘设备出现问题后通过奇偶校验信息来尝试重建损坏的数据。兼顾了硬盘设备的读写速度、数据安全性与存储成本问题。

RAID 5 最少由 3 块硬盘组成，如果其中一块硬盘发生故障，数据仍然可以通过剩余的数据块和奇偶校验数据进行重建，而不会导致数据丢失。

### 7.1.4、RAID10

RAID 10 技术是 RAID 1+RAID 0 技术的一个“组合体”。需要至少 4 块硬盘来组建，其中先分别两两制作成两个RAID 1 磁盘阵列，以保证数据的安全性；然后再对两个 RAID 1 磁盘阵列实施 RAID 0 技术，进一步提高硬盘设备的读写速度。

理论上来讲，只要坏的不是同一阵列中的所有硬盘，硬盘数据都不会丢失。

由于 RAID 10 技术继承了 RAID 0 的高读写速度和 RAID 1 的数据安全性，在不考虑成本的情况下 RAID 10 的性能也超过了 RAID 5，因此当前成为广泛使用的一种存储技术。

### 7.1.5、部署磁盘阵列

mdadm 命令用于创建、调整、监控和管理 RAID 设备，英文全称为“multiple devices admin”，语法格式为：

```
mdadm 参数 硬盘名称
```

mdadm 命令中的常用参数及作用如下：

```
-a     添加新设备到RAID阵列中
-n     指定设备数量
-l     指定 RAID 级别
-C     创建
-v     显示过程
-f     模拟设备损坏
-r     移除设备
-Q     查看摘要信息
-D     查看详细信息
-S     停止 RAID 磁盘阵列
```

在虚拟机中添加四块硬盘设备，根据udev命名规则，分别是/dev/sdb，/dev/sdc，/dev/sdd，/dev/sde

下面使用mdadm命令创建由这四个硬盘组成的RAID10磁盘阵列，名称为“/dev/md0”：

```
[root@linuxprobe~]# mdadm -Cv /dev/md0 -n 4 -l 10 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

-C 参数代表创建一个 RAID 阵列；-v参数显示创建的过程；-n 4 参数代表使用 4 块硬盘来部署这个 RAID 磁盘阵列；-l 10 参数则代表 RAID10 方案；最后再加上 4 块硬盘设备的名称。

使用-Q参数查看简要信息：

```
[root@linuxprobe~]# mdadm -Q /dev/md0
/dev/md0: 39.97GiB raid10 4 devices, 0 spares. Use mdadm --detail for more detail.
```

 4 块 20GB 大小的硬盘组成的磁盘阵列组，可用空间只有39.97GB，因为RAID10通过两两一组硬盘组成的 RAID 1 磁盘阵列保证了数据的可靠性，其中每一份数据都会被保存两次，因此导致硬盘存在 50%的使用率和 50%的冗余率。这样一来，80GB 的硬盘容量也就只有一半了。

等两三分钟后，把制作好的 RAID 磁盘阵列格式化为 Ext4 格式：

```
[root@linuxprobe~]# mkfs.ext4 /dev/md0
```

随后，创建挂载点/RAID:

```
[root@linuxprobe~]# mkdir /RAID
```

将硬盘设备进行挂载：

```
[root@linuxprobe~]# mount /dev/md0 /RAID
```

挂载结束后可以使用df -h查询设备挂载情况：

```
[root@linuxprobe~]# df -h
```

使用-D参数查看/dev/md0磁盘阵列设备的详细信息，确认RAID级别，阵列大小，总硬盘数：

```
[root@linuxprobe~]# mdadm -D /dev/md0
Raid Level : raid10
Array Size : 41908224 (39.97 GiB 42.91 GB)
Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
Raid Devices : 4
Total Devices : 4
```

如果想让创建好的 RAID 磁盘阵列能够一直提供服务，不会因每次的重启操作而取消，那么一定要记得将信息添加到/etc/fstab 文件中，可以直接用vim编辑器编辑/etc/fstab，也可以使用追加符>>追加内容：

```
[root@linuxprobe~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
```

### 7.1.6、损坏磁盘阵列及修复

在确认有一块物理硬盘设备出现损坏而不能再继续正常使用后，应该使用 mdadm 命令将其移除。
首先先使用-f参数模拟硬盘损坏：

```
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0
```

然后查看 RAID 磁盘阵列的状态，可以发现状态已经改变:

```
[root@linuxprobe~]# mdadm -D /dev/md0
```

如果不模拟损坏，直接用-r清除会显示“busy”。所以需要先-f之后再用-r参数彻底清除：

```
[root@linuxprobe~]# mdadm /dev/md0 -r /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
```

在 RAID 10 级别的磁盘阵列中，当 RAID 1 磁盘阵列中存在一个故障盘时并不影响 RAID 10 磁盘阵列的使用。当购买了新的硬盘设备后再使用 mdadm 命令予以替换即可，在此期间可以在/RAID 目录中正常地创建或删除文件.由于我们是在虚拟机中模拟硬盘，所以先重启系统，然后再把新的硬盘添加到 RAID 磁盘阵列中。

使用-a参数进行添加新硬盘的操作，系统默认会自动开始数据的同步工作：

```
[root@linuxprobe~]# mdadm /dev/md0 -a /dev/sdb
mdadm: added /dev/sdb
```

使用-D参数可以查看整个过程和进度：

```
[root@linuxprobe~]# mdadm -D /dev/md0
```

### 7.1.7、磁盘阵列+备份盘

1、RAID 10 磁盘阵列中最多允许 50%的硬盘设备发生故障，但是同一 RAID 1 磁盘阵列中的硬盘设备若全部损坏，也会导致数据丢失。

可以使用RAID备份盘技术来预防这类事故，该技术的核心理念就是准备一块足够大的硬盘，这块硬盘平时处于闲置状态，一旦 RAID 磁盘阵列中有硬盘出现故障后则会马上自动顶替上去。

2、现在创建一个 RAID 5磁盘阵列+备份盘：

```
[root@linuxprobe~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sdd /dev/sde
```

参数-n 3代表创建这个 RAID 5 磁盘阵列所需的硬盘数；参数-l 5 代表 RAID 的级别；参数-x 1 则代表有一块备份盘。

将部署好的RAID5磁盘阵列格式化为Ext4文件格式：

```
[root@linuxprobe~]# mkfs.ext4 /dev/md0
```

创建挂载点：

```
[root@linuxprobe~]# mkdir /RAID
```

先将挂载信息写到/etc/fstab文件中，然后执行mount -a命令进行自动挂载：

```
[root@linuxprobe~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
[root@linuxprobe~]# mount -a
```

使用df命令检查是否挂载成功：

```
[root@linuxprobe~]# df -h
```

然后我们把硬盘设备/dev/sdb 移出磁盘阵列，然后迅速查看/dev/md0 磁盘阵列的状态，就会发现备份盘/dev/sde已经被自动顶替上去并开始了数据同步：

```
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0

[root@linuxprobe~]# mdadm -D /dev/md0
Number Major Minor RaidDevice State
3 8 64 0 active sync /dev/sde
1 8 32 1 active sync /dev/sdc
4 8 48 2 active sync /dev/sdd
0 8 16 - faulty /dev/sdb
```

之后如果再想添加一块备份盘进来，使用-a参数即可。

### 7.1.8、删除磁盘阵列

要删除已经部署好的RAID磁盘阵列，首先要解除挂载，然后使用-f参数将所有的磁盘都设置成停用状态：

```
[root@linuxprobe~]# umunt /RAID
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md0
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
```

然后再用-r参数将磁盘逐一移除出去；

```
[root@linuxprobe~]# mdadm /dev/md0 -r /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
[root@linuxprobe~]# mdadm /dev/md0 -r /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md0
[root@linuxprobe~]# mdadm /dev/md0 -r /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md0
[root@linuxprobe~]# mdadm /dev/md0 -r /dev/sde
mdadm: hot removed /dev/sde from /dev/md0
```

也可以将-f和-r两条合并成一条语句，例如：

```
[root@linuxprobe~]# mdadm /dev/md0 -f /dev/sdb -r /dev/sdb
```

只是早期的服务器不支持-f和-r一起使用 所以保守起见还是分开操作-f和-r。

全部移除之后，可以使用-D参数查询一下：

```
[root@linuxprobe~]# mdadm -D /dev/md0
```

接下来停用整个RAID磁盘序列：

```
[root@linuxprobe~]# mdadm --stop /dev/md0
```

如果是新版本服务器此时就彻底完成了删除；如果是老版本可能在使用--stop 参数后依然会保留设备文件，需要我们使用--remove参数清理文件：

```
[root@linuxprobe~]# mdadm --remove /dev/md0
```

## 7.2、LVM（逻辑卷管理器）

前面学习的硬盘设备管理技术虽然能够有效地提高硬盘设备的读写速度以及数据的安全性，但是在硬盘分好区或者部署为 RAID 磁盘阵列之后，再想修改硬盘分区大小就不容易了。这时就需要用到另外一项非常普及的硬盘设备资源管理技术——逻辑卷管理器(LVM)。LVM 允许用户对硬盘资源进行动态调整。

LVM的创建初衷是为了解决硬盘设备在创建分区后不易修改分区大小的缺陷。

LVM 技术是在硬盘分区和文件系统之间添加了一个逻辑层，它提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。

LVM中的几个基本概念：

1️⃣物理卷（PV）：这是LVM的最底层，可以是整个物理硬盘或硬盘上的分区或者RAID磁盘阵列。

2️⃣卷组（VG）：由一个或多个物理卷组成，形成一个存储池。如果卷组的剩余容量不足，可以随时将新的物理卷加入到里面。管理员可以在卷组上创建逻辑卷。

3️⃣逻辑卷（LV）：逻辑卷是用卷组中空闲的资源建立的，并且逻辑卷在建立后可以动态地扩展或缩小空间。

4️⃣物理区域（PE）：这是LVM中分配存储空间的基本单元。每个逻辑卷的大小必须是PE的倍数。

### 7.2.1、部署逻辑卷

在我们使用LVM进行部署时，需要逐个配置物理卷、卷组和逻辑卷，常用的部署命令如下表：

| 功能 |  物理卷   |   卷组    |  逻辑卷   |
| :--: | :-------: | :-------: | :-------: |
| 扫描 |  pvscan   |  vgscan   |  lvscan   |
| 建立 | pvcreate  | vgcreate  | lvcreate  |
| 显示 | pvdisplay | vgdisplay | lvdisplay |
| 删除 | pvremove  | vgremove  | lvremove  |
| 扩展 |           | vgextend  | lvextend  |
| 缩小 |           | vgreduce  | lvreduce  |

向虚拟机添加两块新的硬盘，然后开始部署：

1️⃣对这两块新硬盘进行创建物理卷的操作，或者可以说是让这两块硬盘设备支持 LVM 技术：

```
[root@linuxprobe~]# pvcreate /dev/sdb /dev/sdc
```

2️⃣对这两块硬盘进行卷组合并，卷组的名称定义为storage。

```
[root@linuxprobe~]# vgcreate storage /dev/sdb /dev/sdc
```

查看卷组的状态：

```
[root@linuxprobe~]# vgdisplay
```

3️⃣从卷组中切割出一个约为150MB的逻辑卷设备：

在切割时有两种单位：-L 150M表示生成一个大小为 150MB 的逻辑卷。
-l  37表示生成一个大小为 37×4MB=148MB 的逻辑卷，因为每个基本单元(PE)大小默认为4MB。

```
[root@linuxprobe~]# lvcreate -n vo -l 37 storage
```

-n vo表示生成逻辑卷的名称为vo

然后对生成的逻辑卷进行查看：

```
[root@linuxprobe~]# lvdisplay
```

Linux 系统会把 LVM 中的逻辑卷设备存放在/dev 设备目录中，同时会以卷组的名称storage来建立一个目录，里面保存了逻辑卷的相关文件。(即/dev/卷组名称/逻辑卷名称)

4️⃣把生成好的逻辑卷进行格式化：

```
[root@linuxprobe ~]# mkfs.ext4 /dev/storage/vo
```

如果使用了逻辑卷管理器，则不建议用 XFS 文件系统，因为 XFS 文件系统自身就可以使用 xfs_growfs 命令进行磁盘扩容，且LVM与XFS兼容性在有些服务器上也不好。这里使用的是Ext4。

创建挂载点：

```
[root@linuxprobe~]# mkdir /linuxprobe
```

挂载：

```
[root@linuxprobe~]# mount /dev/storage/vo /linuxprobe
```

5️⃣查看挂载状态：

```
[root@linuxprobe~]# df -h
```

挂载信息写入配置文件/etc/fstab :

```
[root@linuxprobe~]# echo "/dev/storage/vo /linuxprobe ext4 defaults 0 0" >> /etc/fstab
```

使用cat命令查看是否添加成功：

```
[root@linuxprobe~]# cat /etc/fstab
```

### 7.2.2、扩容逻辑卷

在前面的实验中，卷组是由两块硬盘设备共同组成的。用户在使用存储设备时感知不到设备底层的架构和布局，更不用关心底层是由多少块硬盘组成的，只要卷组中有足够的资源，就可以一直为逻辑卷扩容。

对逻辑卷的扩容分为以下几步：

1️⃣在扩容之前一定要记得卸载设备和挂载点的关联：

```
[root@linuxprobe~]# umount /linuxprobe
```

2️⃣把上一个实验的逻辑卷vo扩容到290MB：

```
[root@linuxprobe~]# lvextend -L 290M /dev/storage/vo
```

- `-L 290M` 表示将逻辑卷的大小设置为 290 MB。
- `-L +290M` 表示在当前大小的基础上增加 290 MB。

3️⃣检查硬盘的完整性，确认目录结构、内容和文件内容没有丢失：（一般只要不报错就是ok的）

```
[root@linuxprobe~]# e2fsck -f /dev/storage/vo
```

4️⃣ LV（逻辑卷）设备进行了扩容操作，但系统内核还没有同步到这部分新修改的信息，需要手动同步设备在系统中的容量(即通知系统内核)：

```
[root@linuxprobe~]# resize2fs /dev/storage/vo
```

5️⃣重新挂载硬盘设备，因为/etc/fstab里面已经有挂载信息了，所以直接用-a参数自动挂载即可：

```
[root@linuxprobe~]# mount -a
```

查看是否挂载成功：

```
[root@linuxprobe~]# df -h
```

### 7.2.3、缩小逻辑卷

相较于扩容逻辑卷，在对逻辑卷进行缩容操作时，数据丢失的风险更大。所以在生产环境中执行相应操作时，一定要提前备份好数据。

缩小逻辑卷的操作分如下几步：

1️⃣缩容之前先记得把文件系统卸载：

```
[root@linuxprobe~]# umount /linuxprobe
```

2️⃣Linux 系统规定，在对 LVM 逻辑卷进行缩容操作之前，要先检查文件系统的完整性：

```
[root@linuxprobe~]# e2fsck -f /dev/storage/vo
```

3️⃣先通知系统内核要将逻辑卷 vo 的容量减小到 120MB：

```
[root@linuxprobe~]# resize2fs /dev/storage/vo 120M
```

4️⃣再将逻辑卷的容量缩减为120M：

```
[root@linuxprobe~]# lvreduce -L 120M /dev/storage/vo
```

这时候有一个交互式的输入，输入y确认：

```
Do you really want to reduce storage/vo? [y/n]: y
```

注意缩容是先通知在缩容，扩容是先扩容再通知，二者反过来了。可以理解成，缩容风险大，需要先通知一下系统缩容的打算，如果系统没拒绝，说明这么做是可以的，才能去缩容。

5️⃣重新挂载文件系统：

```
[root@linuxprobe~]# mount -a
```

查看挂载状态：

```
[root@linuxprobe~]# df -h
```

### 7.2.4、逻辑卷快照

LVM 还具备有“快照卷”功能，就是对某一个逻辑卷设备做一次快照，如果日后发现数据被改错了，就可以利用之前做好的快照卷进行覆盖还原。

LVM 的快照卷功能有两个特点：

1️⃣快照卷的容量必须等同于逻辑卷的容量；

2️⃣快照卷仅一次有效，一旦执行还原操作后则会被立即自动删除。

实行“快照卷”功能分为如下几步：

1️⃣首先查看卷组中的容量是否还够用：

```
[root@linuxprobe~]# vgdisplay
Alloc PE / Size 30 / 120.00 MiB
Free PE / Size 10208 / <39.88 GiB
```

可以看到卷组中已经使用了 120MB 的容量，空闲容量还有39.88GB。这里的一个基本单位PE是4MiB。

2️⃣使用重定向(>)，向逻辑卷所挂载的目录中写入一个文件readme.txt:

```
[root@linuxprobe~]# echo "Welcome to linuxprobe.com" > /linuxprobe/readme.txt
```

3️⃣使用-s参数生成一个快照卷：

```
[root@linuxprobe~]# lvcreate -L 120M -s -n SNAP /dev/storage/vo
```

-n SNAP表示快照卷的名称为“SNAP” ；命令中的/dev/storage/vo表示是针对这个逻辑卷生成的快照卷，稍后还原数据也会还原到这个设备上。

查看快照卷是否生成成功：

```
[root@linuxprobe~]# lvdisplay
Allocated to snapshot  0.01%
```

此时存储空间的占用量还很低。

4️⃣在逻辑卷所挂载的目录中创建一个 100MB 的垃圾文件，从文件/dev/zero里面输入空字符，输出到文件/linuxprobe/files中：

```
[root@linuxprobe~]# dd if=/dev/zero of=/linuxprobe/files count=1 bs=100M
```

count=1表示只写入一个块，bs=100M表示块大小为100M。

再查看快照卷的状态：

```
[root@linuxprobe~]# lvdisplay
Allocated to snapshot 83.71%
```

发现存储空间的占用量上升了。

5️⃣利用快照卷将逻辑卷vo还原到添加100MB文件之前的状态，但是在此之前需要先卸载掉逻辑卷设备与目录的挂载：

```
[root@linuxprobe~]# umount /linuxprobe
```

使用lvconvert命令恢复逻辑卷到的快照：

```
[root@linuxprobe~]# lvconvert --merge /dev/storage/SNAP
```

恢复完之后快照卷就会被自动删除掉，并且刚刚创建的100MB的垃圾文件也没了。

6️⃣重新挂载：

```
[root@linuxprobe~]# mount -a
```

### 7.2.5、删除逻辑卷

确定要删除逻辑卷之前，需要提前备份好重要的数据信息，然后依次删除逻辑卷、卷组、物理卷设备，这个顺序不可颠倒。

删除逻辑卷分为如下几步：

1️⃣取消逻辑卷与目录的挂载关联：

```
[root@linuxprobe~]# umount /linuxprobe
```

2️⃣删除/etc/fstab中逻辑卷vo的挂载信息:

```
[root@linuxprobe~]# vim /etc/fstab
/dev/storage/vo /linuxprobe ext4 defaults 0 0 #这一行删掉
```

3️⃣删除逻辑卷，需要输入 y 来确认操作：

```
[root@linuxprobe~]# lvremove /dev/storage/vo
Do you really want to remove active logical volume storage/vo? [y/n]: y
```

4️⃣删除卷组，此处只用写卷组名称，不用写绝对路径：

```
[root@linuxprobe~]# vgremove storage
```

5️⃣删除物理卷：

```
[root@linuxprobe~]# pvremove /dev/sdb /dev/sdc
```

6️⃣执行完上述操作后，分别执行lvdisplay、vgdisplay、pvdisplay 命令，确认已经被彻底删除了。

```
[root@linuxprobe~]# lvdisplay
[root@linuxprobe~]# vgdisplay
[root@linuxprobe~]# pvdisplay
```

### 扩容和缩容的区别：

扩容：先扩容	再检查文件系统+通知内核

缩容：先检查文件系统+通知内核	再缩容

# 第8章、使用iptables与firewalld防火墙

## 8.1、防火墙管理工具

1、防火墙的主要功能是依据策略对穿越防火墙自身的流量进行过滤。防火墙策略可以基于流量的源目地址、端口号、协议、应用等信息来定制。若流量与某一条策略规则相匹配，则执行相应的处理，反之则丢弃。

2、从 RHEL 7 系统开始，firewalld 防火墙正式取代了 iptables 防火墙。

其实，iptables 与 firewalld 都不是真正的防火墙，它们都只是用来定义防火墙策略的防火墙管理工具而已；或者说，它们只是一种服务。
iptables 服务会把配置好的防火墙策略交由内核层面的 netfilter 网络过滤器来处理。

firewalld 服务则是把配置好的防火墙策略交由内核层面的 nftables 包过滤框架来处理。

3、当前在 Linux 系统中其实存在多个防火墙管理工具，旨在方便运维人员管理 Linux 系统中的防火墙策略，我们只需要配置妥当其中的一个就足够了。

## 8.2、iptables

### 8.2.1、策略与规则链

1、防火墙会从上到下来读取配置的策略规则，因此要把较为严格、优先级较高的策略规则放到前面，以免发生错误。在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果读完所有的策略规则没有找到匹配项，就去执行默认的策略。

一般而言，防火墙策略规则的设置有两种：“通”（即放行）和“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

2、iptables 服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

➢ PREROUTING：在进行路由选择前处理数据包；

➢ INPUT：处理流入的数据包；

➢ OUTPUT：处理流出的数据包；

➢ FORWARD：处理转发的数据包；

➢ POSTROUTING：在进行路由选择后处理数据包；

3、iptables中服务的术语有：

 ACCEPT（允许流量通过）、REJECT（拒绝流量通过）、LOG（记录日志信息）、DROP（拒绝流量通过）。

考虑DROP和REJECT的区别：就 DROP 来说，它是直接将流量丢弃而且不响应；REJECT 则会在拒绝流量后再回复一条“信息已经收到，但是被扔掉了”信息，从而让流量发送方清晰地看到数据被拒绝的响应信息。

工作中最好多用DROP进行拒绝，这样可以隐藏服务器的运行状态。

4、在我们执行命令“ping -c 4 192.168.10.10”的时候，如果把 Linux 系统中的防火墙策略设置为 REJECT 动作后，流量发送方会看到端口不可达的响应；
把 Linux 系统中的防火墙策略修改成 DROP 动作后，流量发送方会看到响应超时的提醒。

### 8.2.2、基本的命令参数

1、在OSI七层模型中，iptables属于数据链路层中的服务，所以可以根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配；一旦匹配成功，iptables 就会根据策略规则所预设的动作来处理这些流量。

iptables 中常用的参数以及作用如下：

```
-P           设置默认策略
-F           清空规则链
-L           查看规则链
-A           在规则链的末尾加入新规则
-I num       在规则链的头部加入新规则
-D num       删除某一条规则
-s           匹配来源地址 IP/MASK，加叹号“!”表示除这个IP外
-d           匹配目标地址
-i 网卡名称    匹配从这块网卡流入的数据
-o 网卡名称    匹配从这块网卡流出的数据
-p           匹配协议，如 TCP、UDP、ICMP
--dport num   匹配目标端口号
--sport num   匹配来源端口号
```

2、使用-L参数查看已有的防火墙规则链：

```
[root@linuxprobe~]# iptables -L
```

使用-F参数清空已有的防火墙规则链：

```
[root@linuxprobe~]# iptables -F
```

使用-P策略将INPUT规则链的默认策略设置为拒绝：

```
[root@linuxprobe~]# iptables -P INPUT DROP
```

设置完之后使用-L参数查看是否设置成功：

```
[root@linuxprobe~]# iptables -L
```

需要注意的是，规则链的默认拒绝只能设置成DROP，不能设置成REJECT。

把INPUT链的默认策略设置成拒绝之后，就要往里面写入允许策略了。

3、接下来向INPUT链中添加允许ICMP协议流量进入的策略规则：

```
[root@linuxprobe~]# iptables -I INPUT -p icmp -j ACCEPT
```

删除INPUT规则链中刚刚加入的策略:

```
[root@linuxprobe~]# iptables -D INPUT 1
```

把默认策略设置成允许：

```
[root@linuxprobe~]# iptables -P INPUT ACCEPT
```

4、设置允许来自 `192.168.10.0/24` 网段的 IP 地址通过 TCP 协议访问本机的 22 端口（通常是 SSH 端口）：注意这里是对网段进行匹配，所以要写成子网掩码的形式“192.168.10.0/24”；对某台主机进行匹配才直接写出它的IP地址

```
[root@linuxprobe~]# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT
```

再设置拒绝所有其他通过 TCP 协议访问本机 22 端口的连接：

```
[root@linuxprobe~]# iptables -A INPUT -p tcp --dport 22 -j REJECT
```

注意防火墙策略规则是按照从上到下的顺序匹配的，因此一定要把允许动作放到拒绝动作前面，否则所有的流量就将被拒绝掉，从而导致任何主机都无法访问我们的服务。这两条命令的组合效果是，只有来自 `192.168.10.0/24` 网段的 IP 地址可以通过 SSH 访问本机，其他所有 IP 地址的 SSH 访问请求都会被拒绝。

设置好之后使用IP地址在192.168.10.0/24网段内的主机192.168.10.10来访问服务器的22号端口，是允许的：

```
[root@Client A~]# ssh 192.168.10.10
```

然后在用192.168.10.0/24网段之外的IP地址来访问服务器的22端口，发现请求被拒绝了：

```
[root@Client A~]# ssh 192.168.20.10
ssh: connect to host 192.168.20.10 port 22: No route to host
```

5、向INPUT规则链中添加拒绝所有人访问本机12345端口的策略规则，无论是通过TCP还是UDP协议：

```
[root@linuxprobe~]# iptables -I INPUT -p tcp --dport 12345 -j REJECT
[root@linuxprobe~]# iptables -I INPUT -p udp --dport 12345 -j REJECT
```

设置好之后使用-L参数查询规则链：

```
[root@linuxprobe~]# iptables -L
```

6、向INPUT规则链中添加拒绝192.168.10.5主机访问本机80端口(Web服务)的策略规则：

```
[root@linuxprobe~]# iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
```

设置好之后使用-L参数查看：

```
[root@linuxprobe~]# iptables -L
```

7、向INPUT规则链中添加拒绝所有主机访问本机1000~1024端口的策略规则：（添加兜底的规则用的是-A参数）

```
[root@linuxprobe~]# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
[root@linuxprobe~]# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
```

设置好之后使用-L参数查看：

```
[root@linuxprobe~]# iptables -L
```

8、上述iptables命令配置的防火墙规则默认会在下一次重启时失效，如果想要设置永久生效，需要执行保存命令：

```
[root@linuxprobe~]# iptables-save
```

如果服务器是RHEL 5/6/7版本的话，保存命令为：

```
[root@linuxprobe~]# service iptables save
```

## 8.3、firewalld

1、firewalld是RHEL8默认的防火墙配置管理工具，它拥有基于CLI（命令行界面）和基于GUI（图形用户界面）的两种管理方式。

相较于传统的防火墙管理配置工具，firewalld 支持动态更新技术并加入了区域（zone）的概念。简单来说，区域就是 firewalld 预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。

firewalld 中常见的区域名称以及相应的策略规则如下：（默认为 public）

```
trusted   允许所有网络连接，不进行任何过滤
public    允许有限的入站连接，默认情况下只允许特定的服务（如 DHCPv6 客户端）
home      允许更多的入站连接，适合家庭环境中的设备互联
work      类似于 home 区域，但可能会有更严格的规则以保护工作数据
dmz       只允许选定的入站连接，通常用于托管公共服务的服务器
block     所有入站连接都会被拒绝，并返回 icmp-host-prohibited 消息
drop      所有入站连接都会被直接丢弃，不返回任何消息
```

### 8.3.1、终端管理工具

1、firewall-cmd 是 firewalld 防火墙配置管理工具的 CLI（命令行界面）版本，它的参数格式一般都是长格式的，但是可以用tab键补齐命令，所以还好~

常见的参数及作用如下：

```
--get-default-zone  查询默认的区域名称
--set-default-zone=<区域名称>  设置默认的区域，使其永久生效
--get-zones      显示可用的区域
--get-services   显示预先定义的服务
--get-active-zones   显示当前正在使用的区域与网卡名称
--add-source=    将源自此IP或子网的流量导向指定的区域
--panic-on    开启应急状况模式
--panic-off   关闭应急状况模式
--remove-source=   不再将源自此 IP 或子网的流量导向某个指定区域
--list-all  显示当前区域的网卡配置参数、资源、端口以及服务等信息
--list-all-zones 显示所有区域的网卡配置参数、资源、端口以及服务等信息
--add-service=<服务名> 设置默认区域允许该服务的流量
--add-port=<端口号/协议> 设置默认区域允许该端口的流量
--remove-service=<服务名>  设置默认区域不再允许该服务的流量
--remove-port=<端口号/协议>  设置默认区域不在允许该端口的流量
```

使用firewalld命令配置的防火墙策略默认为当前生效模式，又称为运行时(Runtime)模式，会随着系统的重启而失效。如果想让策略一直存在，需要使用永久(Permanent)模式，方法就是在用 firewall-cmd 命令正常设置防火墙策略时添加--permanent 参数。永久生效需要重启才能生效，不过可以使用firewall-cmd --reload 命令让其立即生效。总结来说就是：

```
Runtime：当前立即生效，重启后失效。
Permanent：当前不生效，重启后生效。
```

2、查看当前firewalld服务当前所使用的区域：

```
[root@linuxprobe~]# firewall-cmd --get-default-zone
public
```

在配置防火墙策略之前一定不要忘记先查看当前生效的是哪个区域。

3、查询指定网卡在firewalld服务中绑定的区域：

```
[root@linuxprobe~]# firewall-cmd --get-zone-of-interface=ens160
public
```

执行这条命令后，系统会返回网络接口ens160当前所属的区域名称，本例是public。

4、把网卡ens160的默认区域修改为external，并在系统重启后生效(即永久生效)：

```
[root@linuxprobe~]# firewall-cmd --permanent --zone=external --change-interface=ens160 
```

设置完之后再去查看ens的区域是否已经被修改：

```
[root@linuxprobe~]# firewall-cmd --permanent --get-zone-of-interface=ens160
external
```

发现修改成功。这里不加--permanent参数也能查出external。

5、默认区域也叫全局配置，指的是对所有网卡都生效的配置，优先级较低，下面把firewalld服务的默认区域设置成public：

```
[root@linuxprobe~]# firewalld-cmd --set-default-zone=public
```

但是由于优先级较低，所以前面已经设置好的网卡ens160的区域还是external：

```
[root@linuxprobe~]# firewall-cmd --get-zone-of-interface=ens160
external
```

6、使用--panic-on参数会立即切断一切网络连接，使用--panic-off可以恢复网络连接：

```
[root@linuxprobe~]# firewall-cmd --panic-on
success
[root@linuxprobe~]# firewall-cmd --panic-off
success
```

7、查询ssh和https协议的流量在public区域下是否会被放行：

```
[root@linuxprobe~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@linuxprobe~]# firewall-cmd --zone=public --query-service=https
no
```

8、把HTTPS协议的流量设置为永久允许放行：

```
[root@linuxprobe~]# firewall-cmd --permanent --zone=public --add-service=https
```

然后立马去查询是否允许放行https：

```
[root@linuxprobe~]# firewall-cmd --zone=public --query-service=https
no
```

发现不允许放行，那是因为需要永久性设置需要重启才能生效，这里使用--reload参数也行：

```
[root@linuxprobe~]# firewall-cmd --reload
```

然后再查看一次：

```
[root@linuxprobe~]# firewall-cmd --zone=public --query-service=https
yes
```

此时就允许放行了。

9、把http协议的流量设置为永久拒绝，并立即生效：

```
[root@linuxprobe~]# firewall-cmd --permanent --zone=public --remove-service=http
[root@linuxprobe~]# firewall-cmd --reload
```

10、把访问8080和8081端口的流量策略设置为允许，但仅限当前生效(即不加permanent)：

```
[root@linuxprobe~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
```

/tcp表示指定协议是TCP。

设置完之后查看允许流量访问的端口：

```
[root@linuxprobe~]# firewall-cmd --zone=public --list-ports
8081-8081/tcp
```

11、使用firewall-cmd命令实现端口转发技术，命令通用格式如下：

```
firewall-cmd --permanent --zone=<区域> --add-forward-port=port=<源端口号>:proto=<协议>:toport=<目标端口号>:toaddr=<目标IP地址>
```

将public区域中的 TCP 协议的 888 端口的流量转发到 IP 地址192.168.10.10的 22 端口(ssh)，且要求长期有效：

```
[root@linuxprobe~]# firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10
```

然后在客户端使用ssh命令尝试访问192.168.10.10主机的888端口，发现访问成功：

```
[root@client A~]# ssh -p 888 192.168.10.10
```

12、富规则也叫复规则，表示更细致、更详细的防火墙策略配置，它可以针对系统服务、端口号、源地址和目标地址等诸多信息进行更有针对性的策略配置。它的优先级在所有的防火墙策略中也是最高的。

例如，在firewalld服务中配置一条富规则，使其拒绝192.168.10.0/24网段的所有用户访问本机的ssh服务(22端口)：

```
[root@linuxprobe~]# firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"
```

然后使用--reload使其立即生效：

```
[root@linuxprobe~]# firewall-cmd --reload
```

在客户端使用ssh命令尝试访问192.168.10.10主机不的ssh服务（22端口）：

```
[root@client A~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection failed
```

此时连接失败。

### 8.3.2、图形管理工具

1、firewall-config是 firewalld 防火墙配置管理工具的 GUI（图形用户界面）版本，几乎可以实现所有以命令行来执行的操作。

2、使用dnf来安装firewall-config：

```
[root@linuxprobe~]# dnf install firewall-config
```

安装好firewall-config之后再终端直接输入名字即可打开图形管理工具：

```
[root@linuxprobe ~]# firewall-config
```

其余具体操作讲解见书本P255。

3、SNAT 是一种为了解决 IP 地址匮乏而设计的技术，它可以使得多个内网中的用户通过同一个外网 IP 接入 Internet。

## 8.4、服务的访问控制列表

1、Linux 系统中其实有两个层面的防火墙：

第一个层面是基于TCP/IP协议的流量过滤工具，通常指的是诸如iptables这样的防火墙软件，它们通过过滤网络流量来控制数据包的传输，以保护系统免受网络攻击。

第二个层面是TCP Wrapper服务，它实际上是一个能够根据配置文件中的规则允许或禁止特定服务访问的工具。通过配置TCP Wrapper，可以对系统提供的服务进行访问控制。

第一个在网络层，负责过滤和管理网络数据包的传输。
第二个在应用层，通过控制应用程序的访问来提高系统的安全性。

2、TCP Wrapper 服务的防火墙策略由两个控制列表文件所控制，用户可以编辑允许控制列表文件（/etc/hosts.allow）来放行对服务的请求流量，也可以编辑拒绝控制列表文件（/etc/hosts.deny）来阻止对服务的请求流量。

控制列表文件修改后会立即生效，系统将会先检查允许控制列表文件（/etc/hosts.allow），如果匹配到相应的允许策略则放行流量；如果没有匹配，则会进一步匹配拒绝控制列表文件（/etc/hosts.deny），若找到匹配项则拒绝该流量。如果这两个文件都没有匹配到，则默认放行流量。

不过TCP Wrapper是RHEL6/7系统中默认启用的流量监控程序，在RHEL8中已经被firewalld取代了。

3、配置TCP Wrapper 服务时需要遵循两个原则：

1️⃣编写拒绝策略规则时，填写的是服务名称，而非协议名称；

2️⃣建议先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果。

例如，禁止访问本机sshd服务的所有流量：

```
[root@linuxprobe~]# vim /etc/hosts.deny
```

打开拒绝文件之后在文件中添加：

```
sshd:*
```

接下来，在允许策略规则文件中宏添加一条规则，使其放行源自192.168.10.0/24网段且访问本机sshd服务的所有流量：

```
[root@linuxprobe~]# vim /etc/hosts.allow
```

打开允许文件之后在文件中添加：

```
sshd:192.168.10.
```

在控制列表文件中常添加的其他内容举例如下：

```
192.168.10.10   IP地址为192.168.10.10的主机
192.168.10.     IP段为192.168.10.0/24的主机
192.168.10.0/255.255.255.0    IP段为192.168.10.0/24的主机
.linuxprobe.com     所有DNS后缀为.linuxprobe.com的主机
www.linuxprobe.com     主机名称为 www.linuxprobe.com的主机
ALL     所有主机全部包括在内
```

## 8.5、Cockpit驾驶舱管理工具

1、Cockpit 是一个基于 Web 的图形化服务管理工具，它天然具备很好的跨平台性，因此被广泛应用于服务器、容器、虚拟机等多种管理场景。

2、Cockpit 在默认情况下就已经被安装到系统中。下面执行 dnf 命令对此进行确认：

```
[root@linuxprobe~]# dnf install cockpit
```

但是Cockpit 服务程序在 RHEL 8 版本中没有自动运行，需要我们手动它开启并加入到开机启动项中：

```
[root@linuxprobe~]# systemctl start cockpit
[root@linuxprobe~]# systemctl enable cockpit.socket
```

在 Cockpit 服务启动后，打开系统自带的浏览器，在地址栏中输入“本机地址:9090”，由于访问 Cockpit 的流量会使用 HTTPS 进行加密，而证书又是在本地签发的，因此还需要进行添加并信任本地证书的操作，进入 Cockpit 的登录界面后，输入 root 管理员的账号与系统密码，单击 Log In 按钮后即可进入。

## 8.6、本章复习题

1、在 RHEL 8 系统中，iptables 是否已经被 firewalld 服务彻底取代？

答：没有，iptables 和 firewalld 服务均可用于 RHEL 8 系统。

2、如何把 iptables 服务的 INPUT 规则链默认策略设置为 DROP？

答：执行命令 iptables -P INPUT DROP 即可。

3、怎样编写一条防火墙策略规则，使得 iptables 服务可以禁止源自 192.168.10.0/24 网段的流量访问本机的 sshd 服务（22 端口）？

答：执行命令 iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j REJECT 即可。

4、请简述 firewalld 中区域的作用。

答：可以依据不同的工作场景来调用不同的 firewalld 区域，实现大量防火墙策略规则的快速切换。

5、如何在 firewalld 中把默认的区域设置为 dmz？

答：执行命令 firewall-cmd --set-default-zone=dmz 即可。

6、使用 SNAT 技术的目的是什么？

答：SNAT 是一种为了解决 IP 地址匮乏而设计的技术，它可以使得多个内网中的用户通过同一个外网 IP 接入 Internet。

7、TCP Wrapper 服务分别有允许策略配置文件和拒绝策略配置文件，请问匹配顺序是怎么样的？

答：TCP Wrapper 会先依次匹配允许策略配置文件，然后再依次匹配拒绝策略配置文件；如果都没有匹配到，则默认放行流量。

# 第9章、使用ssh服务管理远程主机

## 9.1、配置网络服务

### 9.1.1、配置网卡参数

1、在RHEL 5、RHEL 6 系统及其他大多数早期的 Linux 系统中，网卡的名称一直都是 eth0、eth1、eth2、……；
在 RHEL 7 中则变成了类似于 eno16777736 这样的名字；
在 RHEL 8 系统中网卡的最新名称是类似于 ens160、ens192 。

2、使用nmtui命令运行网络配置工具：

```
[root@linuxprobe ~]# nmtui
```

打开之后具体怎么配置见教材P275.

3、如果配置好之后发现还是连不上，可以去查看网卡ens160是否已经被激活：

```
[root@linuxprobe~]# vim /etc/sysconfig/network-scripts/ifcfg-ens160
```

打开文件之后查看ONBOOT属性，如果不是yes就修改成yes即可激活：

```
ONBOOT=yes
```

当修改完 Linux 系统中的服务配置文件后，并不会对服务程序立即产生效果。要想让服务程序获取到最新的配置文件，需要手动重启相应的服务：

```
[root@linuxprobe~]# nmcli connection reload ens160
[root@linuxprobe~]# nmcli connection up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/ NetworkManager/ActiveConnection/6)
```

此时再检查，发现网络已经通畅了：

```
[root@linuxprobe~]# ping 192.168.10.10
```

### 9.1.2、创建网络会话

1、RHEL和CentOS系统默认使用NetworkManger来提供网络服务，这是一种动态管理网络配置的守护进程，能够让网络设备保持连接状态。

可以使用nmcli命令来管理NetworkManager服务程序。例如，查看网络信息或网络状态：

```
[root@linuxprobe~]# nmcli connection show
```

2、RHEL 8 系统支持网络会话功能，允许用户在多个配置文件中快速切换（非常类似于 firewalld 防火墙服务中的区域技术）。

假如我们在公司使用笔记本电脑时需要自己手动指定IP地址去连接网络，在家则是路由器给你分配一个IP地址。这两种不同的环境需要我们频繁的修改IP地址，可以使用nmcli命令分别给这两个环境创建专门的网络会话，这样只需要在不同的环境激活相应的网络会话即可。

例如，配置在公司的网络会话：

```
[root@linuxprobe~]# nmcli connection add con-name company ifname ens160 autoconnect no type ethernet ip4 192.168.10.10/24 gw4 192.168.10.1
```

其中connection add con-name company表示添加一个新的网络会话，使用con-name参数指定会话名称为company；
ifname ens160指定了网络接口的设备为ens160；
autoconnect no参数表示网络会话默认不被自动激活；
type ethernet指定了连接的类型为以太网连接；
ip4 192.168.10.10/24设置了IP地址为192.168.10.10，并且使用CIDR表示法指定了子网掩码为/24；
gw4 192.168.10.1指定了网关地址为192.168.10.1；

注意ip4设置的是ip地址，用来唯一标识网络中的某台设备，gw4设置的是网关地址，是设备连接到另一个网络的出口点。

配置在家的网络会话：

```
[root@linuxprobe~]# nmcli connection add con-name house type ethernet ifname ens160
```

DHCP（Dynamic Host Configuration Protocol）是一种网络协议，能够自动为设备分配IP地址、子网掩码、网关地址等网络配置信息，所以这里ip地址和网关地址都不需要指定了。

成功创建网络会话之后，可以使用nmcli命令查看创建的所有网络会话：

```
[root@linuxprobe~]# nmcli connection show
```

使用nmcli 命令配置过的网络会话是永久生效的，这样当我们上班后，顺手启动 company网络会话，网卡信息就自动配置好了：

```
[root@linuxprobe~]# nmcli connection up company
```

在虚拟机上想要使用家庭中的路由器设备，需要把虚拟机设置中的网络适配器调成“桥接模式”，然后就可以连接成功了：

```
[root@linuxprobe~]# nmcli connection up house
```

这里如果是wifi上网启用house会失败，正常。

后续不需要网络会话时，直接用delete命令就能删除：

```
[root@linuxprobe~]# nmcli connection delete house
[root@linuxprobe~]# nmcli connection delete company
```

### 9.1.3、绑定两块网卡

1、假设我们对两块网卡实施了绑定技术，这样在正常工作中它们会共同传输数据，使得网络传输的速度变得更快；而且即使有一块网卡突然出现了故障，另外一块网卡便会立即自动顶替上去，保证数据传输不会中断。

2、处于相同模式的网卡设备才可以进行网卡绑定，否则这两块网卡无法互相传送数据。

3、使用nmcli命令来配置网卡设备的绑定参数：

1️⃣首先创建一个bond网卡：

```
[root@linuxprobe~]# nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=balance-rr"
```

命令与参数的意思是创建一个类型为 bond（绑定）、名称为 bond0、网卡名为 bond0 的绑定设备，网卡绑定模式为 balance-rr。

其中balance-rr表示负载均衡模式，轮询方式（Round-Robin），即数据包会轮流通过每个从属接口发送，如果某个网卡发生故障，会马上切换到另一台网卡设备上，保证网络传输不被中断。

另外一种常见网卡绑定模式为active-backup(主备模式)，它的特点是平时只有一块网卡正常工作，另一个网卡随时待命，一旦工作中的网卡发生损坏，待命的网卡会自动顶替上。

二者的区别是一个是网卡轮流工作，一个是只有一个网卡工作。

2️⃣向bond0设备添加从属网卡：

刚刚创建的bond0设备仅仅是个名称，里面并没有真正能为用户传输数据的网卡设备，接下来使用下面的命令把 ens160 和 ens192 网卡添加进来：

```
[root@linuxprobe~]# nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname ens160 master bond0

[root@linuxprobe~]# nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname ens192 master bond0
```

nmcli connection add表示使用 NetworkManager 命令行工具 nmcli 来添加一个新的网络连接配置；

type ethernet 指定连接类型为以太网连接；

slave-type bond 指定这个以太网连接是bond接口的从属端口；

con-name bond0-port2为从属网卡指定一个名称为 bond0-port2；

ifname ens192表示这个网卡名称为 ens192；

master bond0表示将网卡添加到设备bond0中。

3️⃣配置bond0设备的网络信息，下面使用nmcli命令依次配置网络的 IP 地址及子网掩码、网关、DNS、搜索域、手动配置：

```
[root@linuxprobe~]# nmcli connection modify bond0 ipv4.addresses 192.168.10.10/24
[root@linuxprobe~]# nmcli connection modify bond0 ipv4.gateway 192.168.10.1
[root@linuxprobe~]# nmcli connection modify bond0 ipv4.dns 192.168.10.1
[root@linuxprobe~]# nmcli connection modify bond0 ipv4.dns-search linuxprobe.com
[root@linuxprobe~]# nmcli connection modify bond0 ipv4.method manual
```

4️⃣启动bond0：

```
[root@linuxprobe~]# nmcli connection up bond0
```

查看设备的详细列表：

```
[root@linuxprobe~]# nmcli device status 
```

当用户接下来访问主机 IP 地址 192.168.10.10 时，主机实际上是由两块网卡在共同提供服务。

可以在本地主机执行 ping 192.168.10.10 命令检查网络的连通性：

```
[root@linuxprobe~]# ping 192.168.10.10
```

## 9.2、远程控制服务

### 9.2.1、配置sshd服务

1、SSH是一种能够以安全的方式提供远程登录的协议，是目前远程管理Linux系统的首选方式。

想要使用SSH 协议来远程管理 Linux 系统，则需要配置部署 sshd 服务程序。sshd 是基于SSH协议开发的一款远程管理服务程序，不仅使用起来方便快捷，而且能够提供两种安全验证的方法：

1️⃣基于密码的验证：使用账户和密码来验证登录。

2️⃣基于密钥的验证：需要在本地生成密钥对，然后把密钥对中的公钥上传至服务器，并与服务器中的公钥进行比较；更安全

2、sshd服务的配置信息保存在/etc/ssh/sshd_config文件中，其中包含的参数以及作用如下：

```
Port 22   默认的 sshd 服务端口
ListenAddress 0.0.0.0   设定 sshd 服务器监听的IP地址
Protocol 2   SSH 协议的版本号
PermitRootLogin yes   设定是否允许root管理员直接登录
StrictModes yes   当远程用户的私钥改变时直接拒绝连接
MaxAuthTries 6   最大密码尝试次数
MaxSessions 10   最大终端数
PasswordAuthentication yes   是否允许密码验证
PermitEmptyPasswords no   是否允许空密码登录（很不安全）
```

3、接下来的实验会使用两台虚拟机，一台充当服务器，另外一台充当客户端：

| 主机地址      | 操作系统 | 作用           |
| ------------- | -------- | -------------- |
| 192.168.10.10 | Linux    | 服务器(Server) |
| 192.168.10.20 | Linux    | 客户端(Client) |

RHEL8系统中，默认安装并启用了sshd服务程序。

在客户端使用ssh命令远程连接服务器，格式为"ssh [参数] 主机IP地址"：

```
[root@Client~]# ssh 192.168.10.10
```

退出时需要使用exit命令：

```
[root@Server~]# exit
```

如果禁止以 root 管理员的身份远程登录到服务器，则可以大大降低被黑客暴力破解密码的概率，可以修改sshd服务的配置信息文件，把第 46 行#PermitRootLogin yes 参数前的井号（#）去掉，并把参数值 yes 改成 no即可：

```
[root@Server~]# vim /etc/ssh/sshd_config
略
45 #LoginGraceTime 2m
46 PermitRootLogin no
47 #StrictModes yes
略
```

想让修改后的配置文件立即生效，需要手动重启应用程序：

```
[root@Server~]# systemctl restart sshd
```

也可以将sshd服务程序直接加到开机自启动项：

```
[root@Server~]# systemctl enable sshd
```

此时当root管理员再尝试访问sshd服务程序时就会被拒绝：

```
[root@Client~]# sshd 192.168.10.10
root@192.168.10.10's password:此处输入密码
Permission denied, please try again.
```

当然后续实验还是需要root登录的，还是改回yes先。

### 9.2.2、安全密钥验证

1、为sshd服务配置密钥验证，具体步骤如下：

1️⃣在客户端主机中生成“密钥对”(注意是在客户端Client)：

```
[root@Client~]# ssh-keygen
```

按三次enter键直接跳过询问步骤就ok了。

2️⃣把客户端主机中生成的公钥文件传送至远程服务器：（注意10.10是服务器，10.20是客户端）

```
[root@Client~]# ssh-copy-id 192.168.10.10
略
root@192.168.10.10's password:此处输入服务器管理员密码
略
```

3️⃣对服务器进行设置，使其只允许密钥验证，拒绝密码验证方式：

```
[root@Server~]# vim /etc/ssh/sshd_config
73 PasswordAuthentication no
```

修改完之后重启sshd服务程序：

```
[root@Server~]# systemctl restart sshd
```

4️⃣在客户端尝试登录到服务器，此时由于用户已经将客户端主机中生成的公钥文件传送到了远程服务器，所以不需要输入密码也能直接登录，但是如果没有密钥信息有密码也登不进去。

### 9.2.3、远程传输命令scp

scp（secure copy）是一个基于 SSH 协议在网络之间进行安全传输的命令，其格式为：

```
scp [参数]本地文件 远程账户@远程IP地址:远程目录
```

使用scp命令可以实现通过网络传送数据，而且所有的数据都将进行加密处理。

scp 命令中可用的参数以及作用如下：

```
-v     显示详细的连接进度
-P     指定远程主机的 sshd 端口号
-r     用于传送文件夹内的所有数据
-6     使用 IPv6 协议    
```

例如，将当前目录下的名为"readme.txt"的文件复制到远程主机192.168.10.10的/home目录下：

```
[root@Client~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@Client~]# scp /root/readme.txt 192.168.10.10:/home
```

本地文件一定要用绝对路径来表示！如果要传送整个文件夹，则需要➕参数-r。

如果想要指定传送数据给远程主机192.168.10.10的哪一个用户，例如传给tom，可以写成tom@192.168.10.10

反过来，也可以将远程服务器上的文件下载到本地主机，格式就是反过来即可：

```
scp [参数]远程用户@远程IP地址:远程文件 本地目录
```

例如，将远程主机192.168.10.10的系统版本信息文件下载到本机的root目录中：

```
[root@Client~]# scp 192.168.10.10:/etc/redhat-release /root
```

## 9.3、不间断会话服务

当与远程主机的会话被关闭时，在远程主机上运行的命令也随之被中断。但是如果我们正在使用命令打包文件或者使用脚本安装某个服务程序，中途绝对不能关闭在本地打开的终端窗口或断开网络连接。我们可以使用Tmux创建不间断会话解决上述问题。

Terminal Multiplexer（终端复用器，简称为 Tmux）是一款能够实现多窗口远程控制的开源服务程序，能够实现如下功能：

1️⃣会话恢复：即便网络中断，也可让会话随时恢复，确保用户不会失去对远程会话的控制。

2️⃣多窗口：每个会话都是独立运行的，拥有各自独立的输入输出终端窗口，终端窗口内显示过的信息也将被分开隔离保存，以便下次使用时依然能看到之前的操作记录。

3️⃣会话共享：当多个用户同时登录到远程服务器时，便可以使用会话共享功能让用户之间的输入输出信息共享。

在 RHEL 8 系统中，默认没有安装 Tmux 服务程序，因此需要配置软件仓库来安装它：

```
[root@linuxprobe~]# dnf install tmux
```

### 9.3.1、管理远程会话

1、打开Tumx服务：

```
[root@linuxprobe~]# tmux
```

关闭Tumx服务：

```
[root@linuxprobe~]# exit
```

2、会话窗口的编号是从 0 开始自动排序(即 0,1,2..)

创建一个名称为backup的会话窗口：

```
[root@linuxprobe~]# tmux new -s backup
```

假设我们突然要去忙其他事情，但会话窗口中执行的进程还不能被中断，此时便可以用detach 参数将会话隐藏到后台:

```
[root@linuxprobe~]# tmux detach
```

也可以直接❌掉窗口，Tmux服务程序会帮我们自动保存进程，dont worry。但是在传统的远程控制中直接关闭会话窗口就啥都没有了……

关闭之后不放心可以查看一下后台：

```
[root@linuxprobe~]# tmux ls
backup: 1 windows (created Sun Sep 29 11:42:21 2024) [80x23]
```

发现还在运行的。

回归到backup会话：

```
[root@linuxprobe~]# tmux attach -t backup
```

在日常的生产环境中，其实并不是必须先创建会话，然后再开始工作。可以直接使用 tmux命令执行要运行的指令，这样命令中的一切操作都会被记录下来，当命令执行结束后，后台会话也会自动结束：

```
[root@linuxprobe~]# tmux new "vim memo.txt"
```

删除backup可以使用如下命令：

```
[root@linuxprobe~]# tmux kill-session -t backup
```

此时再查看后台 ：

```
[root@linuxprobe ~]# tmux ls
no server running on /tmp/tmux-0/default
```



### 9.3.2、管理多窗格

Tmux 服务有个多窗格功能，能够把一个终端界面按照上下或左右进行切割，从而使得能同时做多件事情，而且之间互不打扰，特别方便。

创建上下切割的多窗格终端界面：

```
[root@linuxprobe~]# tmux split-window
```

创建左右切割的多窗格终端界面：

```
[root@linuxprobe~]# tmux split-window -h
```

如果创建的窗口太多，窗口尺寸变得特别小，想要向右扩大一些，可以同时按下“Ctrl+B+右箭头键”即可。

可以使用如下命令切换到不同的窗格工作：

```
tmux select-pane -U  切换至上方的窗格
tmux select-pane -D  切换至下方的窗格
tmux select-pane -L  切换至左方的窗格
tmux select-pane -R  切换至右方的窗格
```

可以使用如下命令把窗格的位置进行互换：

```
tmux swap-pane -U   将当前窗格与上方的窗格互换
tmux swap-pane -D   将当前窗格与下方的窗格互换
tmux swap-pane -L   将当前窗格与左方的窗格互换
tmux swap-pane -R   将当前窗格与右方的窗格互换
```

除了敲命令的方式，上述操作还可以通过tmux配置的快捷键来实现，方法是先同时按下ctrl+B，然后送收回迅速按下后续按键，注意不是一起按，常见快捷键功能如下：

```
%        划分为左右两个窗格
"        划分为上下两个窗格
<方向键>  切换到上下左右相邻的一个窗格
;        切换至上一个窗格
o        切换至下一个窗格
{        将当前窗格与上一个窗格位置互换
}        将当前窗格与下一个窗格位置互换
x        关闭窗格
!        将当前窗格拆分成独立窗口，而不在与其他窗格同处一个界面
q        显示窗格编号
```

### 9.3.3、会话共享功能

1、会话共享功能是一件很酷的事情，当多个用户同时控制服务器的时候，每个用户都能够看到相同的内容，还能一起同时操作。

2、要实现会话共享功能，首先使用 ssh 服务将客户端 A 远程连接到服务器：

```
[root@client A~]# ssh 192.168.10.10
```

随后使用Tmux服务创建一个新的会话窗口，名称为share：

```
[root@client A~]# tmux new -s share
```

使用 ssh 服务将客户端 B 也远程连接到服务器：

```
[root@client B~]# ssh 192.168.10.10
```

执行获取远程会话的命令:

```
[root@client B~]# tmux attach-session -t share
```

操作完成后，两台客户端的所有终端信息都会被实时同步，它们可以一起共享同一个会话窗口。

## 9.4、检索日志信息

1、Linux 系统拥有十分强大且灵活的日志系统，用于保存几乎所有的操作记录和服务运行状态，并且按照“报错”、“警告”、“提示”和“其他”等标注进行了分类。

在 RHEL 8 系统中，默认的日志服务程序是 rsyslog。

2、为了便于日后的检索，不同的日志信息会被写入到不同的文件中，Linux系统中，常见的日志文件如下所示：

```
/var/log/boot.log    系统开机自检事件及引导过程等信息
/var/log/lastlog     用户登录成功时间、终端名称及 IP 地址等信息
/var/log/btmp        记录登录失败的时间、终端名称及 IP 地址等信息
/var/log/messages    系统及各个服务的运行和报错信息
/var/log/secure      系统安全相关的信息
/var/log/wtmp        系统启动与关机等相关信息
```

在日常工作中，/var/log/message 这个综合性的文件用得最多。

3、从理论上讲，日志文件分为下面 3 种类型：

1️⃣系统日志：主要记录系统的运行情况和内核信息。

2️⃣用户日志：主要记录用户的访问信息，包含用户名、终端名称、登入及退出时间、来源 IP 地址和执行过的操作等。

3️⃣程序日志：稍微大一些的服务一般都会保存一份与其同名的日志文件，里面记录着服务运行过程中各种事件的信息。

4、journalctl 命令用于检索和管理系统日志信息，英文全称为“journal control”，语法格式为：

```
journalctl 参数
```

例如查看系统中最后5条日志信息：

```
[root@linuxprobe~]# journalctl -n 5
```

使用-f 参数实时刷新日志的最新内容:

```
[root@linuxprobe~]# journalctl -f
```

journalctl命令常见参数如下：

```
-k           内核日志
-b           启动日志
-u           指定服务
-n           指定条数
-p           指定类型
-f           实时刷新（追踪日志）
--since      指定时间
--disk-usage 占用空间
```

使用--since参数按照指定时间范围查看日志:

仅查看今天的日志：

```
[root@linuxprobe~]# journalctl --since today
```

仅查询最近1小时的日志信息：

```
[root@linuxprobe~]# journalctl --since "-1 hour"
```

仅查询12点整到14点整的日志信息：

```
[root@linuxprobe~]# journalctl --since "12:00" --until "14:00"
```

仅查询从 2020 年 7 月 1 日至 2020 年 8 月 1 日的日志信息：

```
[root@linuxprobe~]# journalctl --since "2020-07-01" --until "2020-08-01"
```

可以使用-u参数查看某个服务的日志信息：

```
[root@linuxprobe~]# journalctl -u sshd
```

5、在 rsyslog 服务程序中，日志根据重要程度被分为 9 个等级，如下所示：

```
emerg      系统出现严重故障，比如内核崩溃
alert      应立即修复的故障，比如数据库损坏
crit       危险性较高的故障，比如硬盘损坏导致程序运行失败
err        危险性一般的故障，比如某个服务启动或运行失败
warning    警告信息，比如某个服务参数或功能出错
notice     不严重的一般故障，只是需要抽空处理的情况
info       通用性消息，用于提示一些有用的信息
debug      调试程序所产生的信息
none       没有优先级，不进行日志记录
```

可以在 journalctl 命令中用-p 参数指定查看某一等级的日志：

```
[root@linuxprobe~]# journalctl -p crit
```

## 9.5、本章复习题

1、在 Linux 系统中有多种方法可以配置网络参数，请列举几种。

答：配置网络参数可以使用 nmtui 命令、nmcli 命令、nm-connection-editor 命令或者直接编辑网络配置文件来实现对网络参数的修改。

2、在 RHEL 8 系统中使用网络会话技术的目的是什么？

答：使用 nmcli 命令来管理网络会话的目的是为了快速切换网络参数，以便适应不同的工作场景。例如公司和家的网络会话的切换。

3、请简述网卡绑定技术 balaner-rr 模式的特点。

答：平时两块网卡均工作，且自动备援，无须交换机设备提供辅助支持。

4、想要把本地文件/root/out.txt 传送到地址为 192.168.10.20 的远程主机的/home 目录下，且本地主机与远程主机均为 Linux 系统，最为简便的传送方式是什么？

答：执行命令 scp /root/out.txt root@192.168.10.20:/home，并在进行密码验证后即可开始传送。

5、Tmux 服务程序能够让用户实现远程控制的不间断会话，即便网络发生中断也不丢失对远程主机的会话控制。那么，当想要恢复一个名为 linux 的会话窗口时，应该怎么做呢？

答：执行命令 tmux attach -t linux 即可恢复这个会话窗口。

# 第10章、使用Apache服务部署静态网站

## 10.1、网站服务程序

1、Web 网络服务，一般是指允许用户通过浏览器访问互联网中各种资源的服务。

Web 网络服务是一种被动访问的服务程序，即只有接收到互联网中其他主机发出的请求后才会响应，最终用于提供服务程序的 Web 服务器会通过 HTTP（超文本传输协议）或 HTTPS（安全超文本传输协议）把请求的内容传送给用户。

目前能够提供 Web 网络服务的程序有 IIS、Nginx 和 Apache 等。其中IIS（Internet information service）是Windows 系统中默认的 Web 服务程序。

2、配置软件仓库：

1️⃣把系统镜像挂载到/media/cdrom 目录：

```
[root@linuxprobe~]# mkdir -p /media/cdrom

[root@linuxprobe~]# mount /dev/cdrom /media/cdrom
```

2️⃣使用Vim文本编辑器创建软件仓库的配置文件：

```
[root@linuxprobe~]# vim /etc/yum.repos.d/rhel8.repo
```

配置内容如下：

```
[BaseOS]
name=BaseOS
baseurl=file:///media/cdrom/BaseOS
enabled=1
gpgcheck=0
[AppStream]
name=AppStream
baseurl=file:///media/cdrom/AppStream
enabled=1
gpgcheck=0
```

3️⃣安装Apache服务程序，使用dnf时Apache 服务的软件包名称为 httpd：

```
[root@linuxprobe~]# dnf install httpd
```

4️⃣启用httpd服务程序:

```
[root@linuxprobe~]# systemctl start httpd
```

设置httpd程序为开机自启动：

```
[root@linuxprobe~]# systemctl enable httpd
```

## 10.2、配置服务文件参数

1、httpd 服务程序的主要配置文件及存放位置如下所示：

```
服务目录      /etc/httpd
主配置文件    /etc/httpd/conf/httpd.conf
网站数据目录  /var/www/html
访问日志     /var/log/httpd/access_log
错误日志     /var/log/httpd/error_log
```

主配置文件中保存的是最重要的服务参数，httpd 服务程序的主配置文件中存在 3 种类型的信息：注释行信息、全局配置、区域配置。

全局配置参数就是一种全局性的配置参数，可作用于所有的子站点，有效降低了频繁写入重复参数的工作量。

区域配置参数则是单独针对每个独立的子站点进行设置的。

2、配置httpd服务程序最常用的参数如下：

```
ServerRoot      服务目录
ServerAdmin     管理员邮箱
User            运行服务的用户
Group           运行服务的用户组
ServerName      网站服务器的域名
DocumentRoot    网站数据目录
Listen          监听的 IP 地址与端口号
DirectoryIndex  默认的索引页页面
ErrorLog        错误日志文件
CustomLog       访问日志文件
Timeout         网页超时时间，默认为 300 秒
```

默认情况下网站的数据保存在/var/www/html/，在这个目录中起始页名称为index.html，我们只需要向起始页中写入一段内容，即可替换掉httpd服务程序的默认首页面：

```
[root@linuxprobe~]# echo "Welcome To LinuxProbe.com" > /var/www/html/index.html
```

此时打开火狐浏览器，起始页就是Welcome To LinuxProbe.com：

```
[root@linuxprobe~]# firefox
```

3、默认情况下，网站数据保存在/var/www/html 目录中。

下面把保存网站数据的目录修改为/home/wwwroot 目录：

1️⃣建立网站数据的保存目录:

```
[root@linuxprobe~]# mkdir /home/wwwroot
```

创建首页文件：

```
[root@linuxprobe~]# echo "The new web directory" > /home/wwwroot/index.html
```

2️⃣打开httpd服务程序的主配置文件，将约第122行用于定义网站数据保存路径的参数 DocumentRoot 修改为/home/wwwroot：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

122 DocumentRoot "/home/wwwroot"

```

同时还需要将约第 127 行与第 134 行用于定义目录权限的参数 Directory 后面的路径也修改为/home/wwwroot：

```
127 <Directory "/home/wwwroot">

134 <Directory "/home/wwwroot">
```

3️⃣重启httpd服务程序：

```
[root@linuxprobe~]# systemctl restart httpd
```

打开浏览器验证效果：

```
[root@linuxprobe~]# firefox
```

咦，显示权限不足？这是因为没有禁用SELinux服务。

## 10.3、SELinux安全子系统

1、Linux系统使用SELinux技术的目的是为了让各个服务进程都受到约束，使其仅获取到本应获取的资源。

它能够从多方面监控违法行为：
1️⃣对服务程序的功能进行限制：SELinux 域限制可以确保服务程序做不了出格的事情；
2️⃣对文件资源的访问进行限制：SELinux 安全上下文确保文件资源只能被其所属的服务程序进行访问

2、SELinux 服务有 3 种配置模式，具体如下：

1️⃣enforcing：强制启用安全策略模式，将拦截服务的不合法请求。

2️⃣permissive：遇到服务越权访问时，只发出警告而不强制拦截。

3️⃣disabled：对于越权的行为不警告也不拦截。

可以使用 getenforce 命令获得当前 SELinux服务的运行模式：

```
[root@linuxprobe~]# getenforce
Enforcing
```

可以使用setenforce命令修改SELinux当前的运行模式（0是禁用，1是启用）,注意这种修改只是临时的，系统重启之后就会失效：

```
[root@linuxprobe~]# setenforce 0
[root@linuxprobe~]# getenforce
Permissive
```

禁用之后再去打开firefox，就会发现能看到正常的网页内容了：

```
[root@linuxprobe wwwroot]# firefox
```

之所以没有禁用SELinux服务之前不能正常显示，是因为/home 目录是用来存放普通用户的家目录数据的，而现在，httpd 提供的网站服务却要去获取普通用户家目录中的数据，这显然违反了 SELinux 的监管原则。

然后把SELinux服务恢复到强制启用安全策略模式：

```
[root@linuxprobe~]# setenforce 1
```

分别查看两个目录是否有不同的SELinux安全上下文值：

```
[root@linuxprobe~]# ls -Zd /var/www/html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html
```

```
[root@linuxprobe~]# ls -Zd /home/wwwroot
drwxrwxrwx. root root unconfined_u:object_r:home_root_t:s0 /home/wwwroot
```

ls命令中，-Z表示查看文件的安全上下文值，-d表示对象是个文件夹。

在文件上设置的 SELinux 安全上下文是由用户段、角色段以及类型段等多个信息项组成的。以第一个为例，用户段 system_u 代表系统进程的身份，角色段 object_r 代表文件目录的角色，类型段 httpd_sys_content_t 代表网站服务的系统文件。

可以使用semanage命令将/home/wwwroot的SELinux安全上下文修改为跟原始网站目录的一样就行了。

3、semanage 命令用于管理 SELinux 的策略，英文全称为“SELinux manage”，语法格式为：

```
semanage [参数] [文件]
```

semanage命令经常用到的几个参数及其作用如下所示：

```
-l    查询
-a    添加
-m    修改
-d    删除
```

例如，向新的网站数据目录中新添加一条 SELinux 安全上下文，让这个目录以及里面的所有文件能够被httpd服务程序访问到：

```
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/*
```

此时还不能立即访问网站，需要使用restorecon命令将设置好的SELinux安全上再问立即生效：

```
[root@linuxprobe~]# restorecon -Rv /home/wwwroot/
```

-Rv 参数表示对指定的目录进行递归操作，并显示SELinux安全上下文的修改过程。最后再次刷新页面即可看到网页内容了：

```
[root@linuxprobe~]# firefox
```

在日常生活中要养成将所需服务添加到开机自启动项中的习惯。

## 10.4、个人用户主页功能

1、httpd服务程序提供的个人用户主页功能可以让系统内所有的用户在自己的家目录中管理个人的网站。

2、httpd服务程序中，默认没有开启个人用户主页功能，开启的步骤如下：

1️⃣编辑下面的配置文件，在第 17 行的 UserDir disabled 参数前面加上#，表示让 httpd 服务程序开启个人用户主页功能，同时把第24行的UserDir public_html 参数前面的#去掉：

```
[root@linuxprobe~]# vim /etc/httpd/conf.d/userdir.conf

17 # UserDir disabled

24   UserDir public_html
```

2️⃣在用户家目录中建立用于保存网站数据的目录及首页面文件：

```
[root@linuxprobe home]# su - linuxprobe
[linuxprobe@linuxprobe~]$ mkdir public_html
[linuxprobe@linuxprobe~]$ echo "This is linuxprobe's website" > public_html/index.html
```

把家目录的权限修改为755，保证其他人也有权限读取里面的内容：

```
[linuxprobe@linuxprobe~]$ chmod -R 755 /home/linxuprobe
```

3️⃣重启httpd服务程序：

```
[linuxprobe@linuxprobe~]$ exit
logout
[root@linuxprobe~]# systemctl restart httpd
```

在浏览器中输入网址，格式为“网址/~用户名”，即可看到用户的个人网站了，例如输入：

```
192.168.10.10/~linuxprobe
```

但是此时直接访问网页还是forbidden，还是因为SELinux！

4️⃣使用getsebool命令查询并过滤出所有与http协议相关的安全策略，其中，off为禁止，on为允许：

```
[root@linuxprobe~]# getsebool -a | grep http

httpd_enable_homedirs --> off


```

可以使用setsebool命令将参数httpd_enabled_homedirs修改为on，从而开启httpd服务的个人用户主页功能：

```
[root@linuxprobe~]# setsebool -P httpd_enable_homedirs=on
```

加上-P参数可以让修改后的SELinux策略规则永久生效且立即生效。

然后刷新网页，发现已经可以显示了：

```
[root@linuxprobe~]# firefox
```

3、有时，网站的拥有者并不希望直接将网页内容显示出来，而只想让通过身份验证的用户看到里面的内容，这时就可以在网站中添加密码功能了：

1️⃣使用htpasswd命令生成密码数据库：

```
[root@linuxprobe~]# htpasswd -c /etc/httpd/passwd linuxprobe
```

-c表示第一次生成，/etc/httpd/passwd表示密码数据库要存放的文件，linuxprobe表示验证用到的用户名称。

2️⃣继续编辑个人用户主页功能的配置文件，把第 31～37 行的参数信息修改成下列内容，#后的内容可以忽略：

```
[root@linuxprobe~]# vim /etc/httpd/conf.d/userdir.conf

31 <Directory "/home/*/public_html">
32 AllowOverride all
#刚刚生成出的密码验证文件保存路径
33 authuserfile "/etc/httpd/passwd"
#当用户访问网站时的提示信息
34 authname "My privately website"
#验证方式为密码模式
35 authtype basic
#访问网站时需要验证的用户名称
36 require user linuxprobe
37 </Directory>

```

修改好之后保存退出，重启httpd服务程序即可生效：

```
[root@linuxprobe~]# systemctl restart httpd
```

此后，当用户再想访问某个用户的个人网站时，就必须输入账户和密码才能正常访问了。

## 10.5、虚拟主机功能

利用虚拟主机功能，可以把一台处于运行状态的物理服务器分割成多个“虚拟的服务器”。

Apache 的虚拟主机功能是服务器基于用户请求的不同 IP 地址、主机域名或端口号，提供多个网站同时为外部提供访问服务的技术。

### 10.5.1、基于IP地址

1、如果一台服务器有多个 IP 地址，而且每个 IP 地址与服务器上部署的每个网站一一对应，这样当用户请求访问不同的 IP 地址时，会访问到不同网站的页面资源。

2、使用基于第九章的网络配置方法，配置三个IP地址，确保都可以访问，使用ping来检查三个IP的连通性：

```
[root@linuxprobe~]# ping -c 4 192.168.10.10
[root@linuxprobe~]# ping -c 4 192.168.10.20
[root@linuxprobe~]# ping -c 4 192.168.10.30
```

1️⃣分别在/home/wwwroot 中创建用于保存不同网站数据的 3 个目录，并向其中分别写入网站的首页文件：

```
[root@linuxprobe~]# mkdir -p /home/wwwroot/10
[root@linuxprobe~]# mkdir -p /home/wwwroot/20
[root@linuxprobe~]# mkdir -p /home/wwwroot/30
[root@linuxprobe~]# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html
[root@linuxprobe~]# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html
[root@linuxprobe~]# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
```

2️⃣从httpd服务的配置文件中大约第132行处开始，分别追加写入 3 个基于 IP 地址的虚拟主机网站参数，然后保存并退出：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

132 <VirtualHost 192.168.10.10>

139 </VirtualHost>
140 <VirtualHost 192.168.10.20>

147 </VirtualHost>
148 <VirtualHost 192.168.10.30>

155 </VirtualHost>

```

此时访问网站，则会看到 httpd 服务程序的默认首页面中显示“权限不足”。显然还是SELinux的原因。由于当前的/home/wwwroot 目录及里面的网站数据目录的 SELinux 安全上下文与网站服务不吻合，因此 httpd 服务程序无法获取到这些网站数据目录。

此时我们需要手动修改新的网站数据目录的SELinux安全上下文：

```
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30/*
```

使用restorecon命令让新设置的SELinux安全上下文立即生效：

```
[root@linuxprobe~]# restorecon -Rv /home/wwwroot
```

之后就可以基于IP地址访问虚拟主机网站了，即在浏览器搜索栏中输入192.168.10.10这些。

### 10.5.2、基于主机域名

当服务器无法为每个网站都分配一个独立 IP 地址的时候，可以尝试让 Apache 自动识别用户请求的域名，从而根据不同的域名请求来传输不同的内容。此时只需要保证位于生产环境中的服务器上有一个可用的 IP 地址即可。

1️⃣手动定义 IP 地址与域名之间对应关系的配置文件，保存并退出后会立即生效：

```
[root@linuxprobe~]# vim /etc/hosts

192.168.10.10 www.linuxprobe.com www.linuxcool.com www.linuxdown.com
```

可以通过过分别 ping 这些域名来验证域名是否已经成功解析为 IP 地址：

```
[root@linuxprobe~]# ping -c 4 www.linuxprobe.com
[root@linuxprobe~]# ping -c 4 www.linuxcool.com
[root@linuxprobe~]# ping -c 4 www.linuxdown.com
```

2️⃣分别在/home/wwwroot 中创建用于保存不同网站数据的 3 个目录，并向其中分别写入网站的首页文件：

```
[root@linuxprobe~]# mkdir -p /home/wwwroot/linuxprobe
[root@linuxprobe~]# mkdir -p /home/wwwroot/linuxcool
[root@linuxprobe~]# mkdir -p /home/wwwroot/linuxdown
[root@linuxprobe~]# echo "www.linuxprobe.com" > /home/wwwroot/linuxprobe/index.html
[root@linuxprobe~]# echo "www.linuxcool.com" > /home/wwwroot/linuxcool/index.html
[root@linuxprobe~]# echo "www.linuxdown.com" > /home/wwwroot/linuxdown/index.html
```

3️⃣从 httpd 服务的配置文件中大约第 132 行处开始，分别追加写入 3 个基于主机名的虚拟主机网站参数，然后保存并退出：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

132 <VirtualHost 192.168.10.10>
133 Documentroot /home/wwwroot/linuxprobe

139 </VirtualHost>
140 <VirtualHost 192.168.10.10>
141 Documentroot /home/wwwroot/linuxcool

147 </VirtualHost>
148 <VirtualHost 192.168.10.10>
149 Documentroot /home/wwwroot/linuxdown

155 </VirtualHost>
```

4️⃣因为当前的网站数据目录还是在/home/wwwroot 目录中，因此还是必须要正确设置网站数据目录文件的 SELinux 安全上下文，使其与网站服务功能相吻合：

```
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown/*
```

然后使用restorecon命令让新配置的SELinux安全上下文立即生效：

```
[root@linuxprobe~]# restorecon -Rv /home/wwwroot
```

之后就可以基于主机域名访问虚拟主机网站了，即在浏览器搜索栏中输入www.linuxprobe.com这些。

### 10.5.3、基于端口号

基于端口号的虚拟主机功能可以让用户通过指定的端口号来访问服务器上的网站资源，这种方式的配置是最复杂的，因为我们不仅要考虑 httpd 服务程序的配置因素，还需要考虑到 SELinux 服务对新开设端口的监控。

1️⃣分别在/home/wwwroot 中创建用于保存不同网站数据的 3 个目录，并向其中分别写入网站的首页文件：

```
[root@linuxprobe~]# mkdir -p /home/wwwroot/6111
[root@linuxprobe~]# mkdir -p /home/wwwroot/6222
[root@linuxprobe~]# mkdir -p /home/wwwroot/6333
[root@linuxprobe~]# echo "port:6111" > /home/wwwroot/6111/index.html
[root@linuxprobe~]# echo "port:6222" > /home/wwwroot/6222/index.html
[root@linuxprobe~]# echo "port:6333" > /home/wwwroot/6333/index.html
```

2️⃣在 httpd 服务配置文件的第 46 行～48 行分别添加用于监听 6111、6222 和 6333端口的参数：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

46 Listen 6111
47 Listen 6222
48 Listen 6333

```

3️⃣从 httpd 服务的配置文件中大约第 134 行处开始，分别追加写入 3 个基于端口号的虚拟主机网站参数，然后保存并退出：

```
134 <VirtualHost 192.168.10.10:6111>

141 </VirtualHost>
142 <VirtualHost 192.168.10.10:6222>

149 </VirtualHost>
150 <VirtualHost 192.168.10.10:6333>

157 </VirtualHost>
```

记得需要重启 httpd 服务，这些配置才生效：

```
[root@linuxprobe~]# systemctl restart httpd
```

4️⃣因为我们把网站数据目录存放在/home/wwwroot 目录中，因此还是必须要正确设置网站数据目录文件的 SELinux 安全上下文，使其与网站服务功能相吻合：

```
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222/*
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333
[root@linuxprobe~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333/*
```

然后用restorecon 命令让新配置的 SELinux 安全上下文立即生效：

```
[root@linuxprobe~]# restorecon -Rv /home/wwwroot
```

此时还是不完全ok，因为 SELinux 服务检测到 6111、6222 和 6333 端口原本不属于 Apache 服务应该需要的资源，但现在却以 httpd 服务程序的名义监听使用了，所以 SELinux 会拒绝使用Apache 服务使用这 3 个端口。

可以使用 semanage 命令查询并过滤出所有与 HTTP 协议相关且SELinux 服务允许的端口列表：

```
[root@linuxprobe~]# semanage port -l | grep http
http_port_t tcp 80, 81, 443, 488, 8008, 8009, 8443, 9000
```

此时我们只需要手动将这三个端口号添加进去就行：

```
[root@linuxprobe~]# semanage port -l | grep http
http_port_t tcp 6333, 6222, 6111, 80, 81, 443, 488, 8008,8009, 8443, 9000
```

然后再重启httpd，打开网页：

```
[root@linuxprobe~]# systemctl restart httpd
[root@linuxprobe~]# firefox
```

之后就可以使用类似于192.168.10.10:6111的端口号去访问虚拟主机网站了。

## 10.6、Apache的访问控制

可以通过修改httpd服务的配置文件来控制对服务器的访问：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

161 <Directory "/var/www/html/server">
162 SetEnvIf User-Agent "Firefox" ff=1
163 Order allow,deny
164 Allow from env=ff
165 </Directory>

[root@linuxprobe~]# systemctl restart httpd
[root@linuxprobe~]# firefox
```

这段代码表示只允许使用firefox浏览器才能访问服务器的首页文件。其中“Order allow, deny”表示先将源主机与允许规则进行匹配，若匹配成功则允许访问请求，反之则拒绝访问请求。

还可以只允许 IP 地址为 192.168.10.20 的主机访问网站资源：

```
[root@linuxprobe~]# vim /etc/httpd/conf/httpd.conf

161 <Directory "/var/www/html/server">
162 Order allow,deny
163 Allow from 192.168.10.20
164 </Directory>

[root@linuxprobe~]# systemctl restart httpd
[root@linuxprobe~]# firefox
```

## 10.7、本章复习题

1、什么是 Web 网络服务？

答：一种允许用户通过浏览器访问互联网中各种资源的服务。

2、相较于 Nginx 服务程序，Apache 服务程序最大的优势是什么？

答：Apache 服务程序具备跨平台特性、安全性，而且拥有快速、可靠、简单的 API 扩展。

3、httpd 服务程序没有检查到首页文件，会提示报错信息吗？

答：不会，httpd 服务在未找到网站首页文件时，会向访客显示一个默认页面。

4、简述 SELinux 服务的作用。

答：为了让各个服务进程都受到约束，使其仅获取到本应获取的资源。

5、要想查询并过滤出所有与 HTTP 协议相关的 SELinux 域策略有哪些，应该怎么做呢？

答：可以结合管道符来实现，即执行 getsebool -a | grep http 命令。
