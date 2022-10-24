---
title: Bochs
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# Bochs

## 下载安装

官方地址：[Bochs x86 PC emulator download | SourceForge.net](https://sourceforge.net/projects/bochs/)

版本：2.6.2

解压：`tar zxvf bochs-2.6.2.tar.gz`

编译：

`cd bochs-2.6.2` 切换到解压的目录下，开始 configure、make、make install 三部曲：

```shell
./configure \
--prefix=/<INSTALL-PATH>/bochs \ # 指定bochs安装路径，根据实际情况将 <INSTALL-PATH> 替换

--enable-debugger \ # 打开 bochs 的调试器 

--enable-disasm \ # 让 bochs 支持反汇编
--enable-iodebug \ # 启用 io 接口调试器
--enable-x86-debugger \ # 支持 x86 调试器
--with-x \ # 使用 x windows
--with-x11 # 使用 x11 图形用户接口

./configure --prefix=/<INSTALL-PATH>/bochs --enable-debugger --enable-disasm --enable-iodebug --enable-x86-debugger --with-x --with-x11
```

以上编译参数不支持 gdb 远程调试，如果需要使用 gdb 调试，需要将参数 `--enable-debugger` 替换为 `--enable-gdb-stub`。两个调试开关只能打开一个否则报错，可以指定不同的路径编译两版本。

```shell
--enable-gdb-stub #  gdb 远程调试
```

configure 之后会生成对应的 Makefile，使用 `vim Makefile`，在 92 行 LIBS 后面添加 `-lpthread` 。

先 `make` ，没有问题后，`make install` 即可。

## 配置 Bochs

Bochs 参考配置文件在 安装路径下的：`share/doc/bochs/bochsrc-sample.txt`。

在 531 行描述了启动顺序，参考后简单设置如下：

```shell
####################################
# Configuration file for Bochs
####################################

# 第一步 设置 Bochs 在运行过程中能够使用的内存，本例为 32 MB
# 关键字为：megs
megs: 32

# 第二步 设置对应真是机器的 BIOS 和 VGA BIOS
# 对应两个关键字为: romimage vgaromimage
romimage: file=/<INSTALL-PATH>/bochs/share/bochs/BIOS-bochs-latest
vgaromimage: file=/<INSTALL-PATH>/bochs/share/bochs/VGABIOS-lgpl-latest

# 第三步 设置 Bochs 所使用的磁盘，软盘的关键字为 floppy
# 若只有一个软盘使用 floppya
# 若有多个，则为 floppya floppyb ...
# floppya: l_44=a.img, status=inserted

# 第四步 选择启动盘符
# boot = floppy # 默认软盘启动，将其注释
boot: disk		# 改为硬盘启动。（我们的代码都写在 硬盘上，不会读取软盘）

# 第五步 设置日志输出
log: bochs.out

# 第六步 开启或关闭某些功能
mouse: enabled=0 # 关闭鼠标
# 打开键盘
keyboard_mapping: enabled=1, map=/<INSTALL-PATH>/bochs/share/bochs/keymaps/x11-pc-us.map

# 硬盘设置
ata0: enabled=1, ioaddr1=0x1f0, ioaddr2=0x3f0, irq=14

# 下面的是增加的 bochs 对 gdb 的支持，可以方便 gdb 远程连接到机器的 1234 端口调试
# configure 的时候配置了 --enable-gdb-stub 才需要配置 都则会报错
# gdbstub: enabled=1, port=1234, text_base=0, data_base=0, bss_base=0
```

将其存为 `bochsrc.disk` 放在bochs 安装路径下。事实上， bochs 配置文件位置并不固定，名字也不要求固定。

## 运行 Bochs

输入 `./bin/bochs` ，回车输出如下：

```shell
$ ./bin/bochs 
========================================================================
                       Bochs x86 Emulator 2.6.2
                Built from SVN snapshot on May 26, 2013
                  Compiled on Oct 24 2022 at 19:58:38
========================================================================
------------------------------
Bochs Configuration: Main Menu
------------------------------

This is the Bochs Configuration Interface, where you can describe the
machine that you want to simulate.  Bochs has already searched for a
configuration file (typically called bochsrc.txt) and loaded it if it
could be found.  When you are satisfied with the configuration, go
ahead and start the simulation.

You can also start bochs with the -q option to skip these menus.

1. Restore factory default configuration
2. Read options from...
3. Edit options
4. Save options to...
5. Restore the Bochs state from...
6. Begin simulation
7. Quit now

Please choose one: [2] 

```

默认选项为 [2]，回车输入 `bochsrc.disk`，如果有问题，根据提示修改配置文件，成功会弹以下一下信息：

```shell
What is the configuration file name?
To cancel, type 'none'. [none] bochsrc.disk
00000000000i[     ] reading configuration from bochsrc.disk
00000000000e[     ] bochsrc.disk:29: 'keyboard_mapping' will be replaced by new 'keyboard' option.
------------------------------
Bochs Configuration: Main Menu
------------------------------

This is the Bochs Configuration Interface, where you can describe the
machine that you want to simulate.  Bochs has already searched for a
configuration file (typically called bochsrc.txt) and loaded it if it
could be found.  When you are satisfied with the configuration, go
ahead and start the simulation.

You can also start bochs with the -q option to skip these menus.

1. Restore factory default configuration
2. Read options from...
3. Edit options
4. Save options to...
5. Restore the Bochs state from...
6. Begin simulation
7. Quit now

Please choose one: [6] 
```

默认选项为 [6]，也就是 Begin simulation ，开始模拟 x86 硬件平台。如果加载不到设备， bochs 不会往下执行。回车弹出白框，键入配置文件（如果不输入，会回到之前的界面）。

### bximage

使用 ./bin/bximage 创建虚拟硬盘：

```shell
-fd # 创建软盘
-hd # 创建硬盘
-mode # 创建硬盘类型: flat sparse growing
-size # 创建多大的硬盘 MB 为单位
-q # 静默模式创建，即创建过程中不会和用户交互
```

使用以下命令创建一个虚拟硬盘：

```shell
./bin/bximage -hd -mode="flat" -size=60 -q hd60M.img
```

`hd60M.img` 为创建的虚拟硬盘名称。

同时需要修改配置文件 bochsrc.disk:

```
# 硬盘设置
ata0: enabled=1, ioaddr1=0x1f0, ioaddr2=0x3f0, irq=14
ata0-master: type=disk, path="hd60M.img", mode=flat, cylinders=121, heads=16, spt=63
```

每次启动 bochs 都要选择选项是很麻烦的，可以通过以下命令直接指定配置文件：

```shell
./bin/bochs -f bochsrc.disk
```

回车后得到以下信息，并弹出黑窗口：

```shell
$ ./bin/bochs -f bochsrc.disk 
========================================================================
                       Bochs x86 Emulator 2.6.2
                Built from SVN snapshot on May 26, 2013
                  Compiled on Oct 24 2022 at 21:06:44
========================================================================
00000000000i[     ] reading configuration from bochsrc.disk
00000000000e[     ] bochsrc.disk:29: 'keyboard_mapping' will be replaced by new 'keyboard' option.
------------------------------
Bochs Configuration: Main Menu
------------------------------

This is the Bochs Configuration Interface, where you can describe the
machine that you want to simulate.  Bochs has already searched for a
configuration file (typically called bochsrc.txt) and loaded it if it
could be found.  When you are satisfied with the configuration, go
ahead and start the simulation.

You can also start bochs with the -q option to skip these menus.

1. Restore factory default configuration
2. Read options from...
3. Edit options
4. Save options to...
5. Restore the Bochs state from...
6. Begin simulation
7. Quit now

Please choose one: [6] 
00000000000i[     ] installing x module as the Bochs GUI
00000000000i[     ] using log file bochs.out
Next at t=0
(0) [0x0000fffffff0] f000:fff0 (unk. ctxt): jmp far f000:e05b         ; ea5be000f0
<bochs:1>
```

## Reference 

1. 
