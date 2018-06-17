---
title: 关于 core dump 的那些事
date: 2018-06-17 11:02:09
tags:
- Linux
- 编程
---

本文的目标是记录下来关于 `core dump` 你应该知道的*背景知识*和进行*诊断操作*的那些事情。

<!-- more -->

## core dump 是什么

`core dump` 这一词的来源是 `Magnetic core memory` 磁芯存储器中的 core，而不是我曾经认为的和 kernel 相关的那个含义。磁芯存储器是五十多年前古时候的内存了，可见这一 term 的悠久历史，也说明核心转储文件（自然这个翻译也不甚靠谱了）本质只是记录了事故现场的内存快照。

## 如何获取 dump 文件

### 如何产生 core dump 文件

主要的来源是用户态程序崩溃时 kernel 自动帮你存储的，这也是一般情况下最常见和有意义的来源。对于内核来讲，红帽有个黑科技在内存中运行两份内核，kernel panic 时候备用内核接管，把主内核 dump 出来。当然 gdb 等调试器也可以给跑得好好的进程直接做个 dump 出来。

```sh
gdb -p <pid>
gdb>generate-core-file
```

会生成文件名为 `core.<pid>` 的 core dump 文件。

值得一提的是 core dump 文件也是合法的 ELF 文件，所以可以用一些 ELF 相关的工具读取一些信息。

下一个问题是 kernel 如何设置是否进行 core dump 以及 dump 的结果如何处理，存放在何处。这些通过系统暴露出来的 `/proc` 文件系统进行设置。

```sh
/proc/sys/kernel/core_pattern
/proc/sys/kernel/core_pipe_limit
/proc/sys/kernel/core_uses_pid
```

在笔者的笔记本中的 Arch Linux 内，内容如下：

```sh
yzy@yzy-arch [12:30:51 PM] [/proc/sys/kernel] 
-> % cat core_pipe_limit 
0
yzy@yzy-arch [12:31:32 PM] [/proc/sys/kernel] 
-> % cat core_uses_pid 
1
yzy@yzy-arch [12:31:37 PM] [/proc/sys/kernel] 
-> % cat core_pattern 
|/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h %e
```

要注意的是 `systemd` 也会更改这里的设置，所以虚拟文件系统中生效的设置和某些配置文件对不上很正常，一定是你没找全配置文件。

而这些设置在一台 CentOS 7 的机器上则如下：

```sh
[yzy@mu00 kernel]$ cat core_pattern 
core
[yzy@mu00 kernel]$ cat core_pipe_limit 
0
[yzy@mu00 kernel]$ cat core_uses_pid 
1
```

### “吐出来的核” 存放在哪里

这取决于 `core_pattern` 这个变量的设置。

在我的本机上可以看到该变量设置为了管道符的后半段，这时它的内容借助 `coredumpctl` 来管理。

```sh
coredumpctl list  # 列出 core
coredumpctl info <match>
coredumpctl gdb <match>
```

至于这个变量后面的格式化字符串的含义等等，请参阅 http://man7.org/linux/man-pages/man5/core.5.html，也就是

```sh
man core
```

Since  kernel  2.6.19,  Linux  supports  an alternate syntax for the /proc/sys/kernel/core_pattern file.  If the first character of this file is a pipe symbol (|), then the remainder of the line is interpreted as the command-line for a user-space program (or script) that is to be executed.  Instead of being written to a disk file, the core dump is given as standard input to the program.

使用管道符的语法从 man page 中摘抄如上。

否则就如同上文中 CentOS 的默认设置，将会以文件的形式直接存放于文件系统中，猜测应该是在对应进程的工作目录下下（待考证）。

## 如何进行分析

最主要的方式还是 `gdb`，同时有一个名为 `crash` 的小工具可以帮助我们。

下面给出一个 core dump 发生时控制台输出的例子：

```
npc: /home/yzy/repos/naive-pascal-compiler/src/utils/../ast/../utils/ast_utils.hpp:37: typename std::enable_if<std::is_base_of<npc::AbstractNode, TNode>::value, std::shared_ptr<_Tp> >::type npc::cast_node(const std::shared_ptr<npc::AbstractNode>&) [with TNode = npc::ExprNode; typename std::enable_if<std::is_base_of<npc::AbstractNode, TNode>::value, std::shared_ptr<_Tp> >::type = std::shared_ptr<npc::ExprNode>]: Assertion `is_a_ptr_of<TNode>(node)' failed.
[1]    16042 abort (core dumped)  ./npc < ../test/basic.pas
```

上面非常长的那一行应该是标准库 assert 宏断言失败的输出，然后这个宏中断了程序运行，内核对此进程进行 dump 输出下面那一行。

对于保存成文件的，我们可以使用

```sh
gdb -c <coredump-file>
```

进行调试，更多用法请参考https://stackoverflow.com/questions/5115613/core-dump-file-analysis，本例中我们使用

```sh
coredumpctl gdb 16042  # 要调试的 core 文件 PID 为 16042 且在我的记录中唯一，可以 match
```

进入 gdb 查看。

```
           PID: 16042 (npc)
           UID: 1000 (yzy)
           GID: 100 (users)
        Signal: 6 (ABRT)
     Timestamp: Sun 2018-06-17 10:52:15 +08 (7h ago)
  Command Line: ./npc
    Executable: /home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc
 Control Group: /user.slice/user-1000.slice/session-c2.scope
          Unit: session-c2.scope
         Slice: user-1000.slice
       Session: c2
     Owner UID: 1000 (yzy)
       Boot ID: 6d54757c981742b6984d6d89a59ba2a3
    Machine ID: adbc8ab8645546ffb9ef66ec15b02081
      Hostname: yzy-arch
       Storage: /var/lib/systemd/coredump/core.npc.1000.6d54757c981742b6984d6d89a59ba2a3.16042.1529203935000000.lz4
       Message: Process 16042 (npc) of user 1000 dumped core.
                
                Stack trace of thread 16042:
                #0  0x00007f8df8f4986b raise (libc.so.6)
                #1  0x00007f8df8f3440e abort (libc.so.6)
                #2  0x00007f8df8f342e0 __assert_fail_base.cold.0 (libc.so.6)
                #3  0x00007f8df8f42112 __assert_fail (libc.so.6)
                #4  0x0000563a871686f3 n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)
                #5  0x0000563a87166c3c n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)
                #6  0x0000563a8716afe3 n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)
                #7  0x0000563a8716582b n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)
                #8  0x0000563a8715e499 n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)
                #9  0x00007f8df8f3606b __libc_start_main (libc.so.6)
                #10 0x0000563a8715e36a n/a (/home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc)

GNU gdb (GDB) 8.1
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-pc-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from /home/yzy/Repos/naive-pascal-compiler/cmake-build-debug/npc...done.
[New LWP 16042]
Core was generated by `./npc'.
Program terminated with signal SIGABRT, Aborted.
#0  0x00007f8df8f4986b in raise () from /usr/lib/libc.so.6
(gdb)
```

打印了如上信息并且进入了 gdb 的 REPL。

使用 `backtrace` 命令查看事故现场的调用栈：

```
(gdb) bt
#0  0x00007f8df8f4986b in raise () from /usr/lib/libc.so.6
#1  0x00007f8df8f3440e in abort () from /usr/lib/libc.so.6
#2  0x00007f8df8f342e0 in __assert_fail_base.cold.0 () from /usr/lib/libc.so.6
#3  0x00007f8df8f42112 in __assert_fail () from /usr/lib/libc.so.6
#4  0x0000563a871686f3 in npc::cast_node<npc::ExprNode> (node=std::shared_ptr<npc::AbstractNode> (use count 3, weak count 1) = {...})
    at /home/yzy/repos/naive-pascal-compiler/src/utils/../ast/../utils/ast_utils.hpp:37
#5  0x0000563a87166c3c in npc::BinopExprNode::BinopExprNode (this=0x563a87be6a80, _Op=npc::BinopExprNode::OP::ge, _lhs=std::shared_ptr<npc::AbstractNode> (use count 3, weak count 1) = {...}, 
    _rhs=std::shared_ptr<npc::AbstractNode> (use count 2, weak count 1) = {...}) at /home/yzy/repos/naive-pascal-compiler/src/utils/../ast/type_nodes.hpp:65
#6  0x0000563a8716afe3 in npc::make_node<npc::BinopExprNode, npc::BinopExprNode::OP, std::shared_ptr<npc::AbstractNode>&, std::shared_ptr<npc::AbstractNode>&> (
    args#0=@0x7fff62efc900: npc::BinopExprNode::OP::ge, args#1=std::shared_ptr<npc::AbstractNode> (use count 3, weak count 1) = {...}, 
    args#2=std::shared_ptr<npc::AbstractNode> (use count 2, weak count 1) = {...}) at /home/yzy/repos/naive-pascal-compiler/src/utils/../ast/../utils/ast_utils.hpp:44
#7  0x0000563a8716582b in yyparse () at src/parse.y:271
#8  0x0000563a8715e499 in main (argc=1, argv=0x7fff62efd888) at /home/yzy/repos/naive-pascal-compiler/src/main.cpp:20
```

皮了一下，这里敲下 `run` 命令居然还能继续执行……好吧对我的场景没啥用。

调用栈最近发生的在最上面，函数名和模板参数都打了出来，对照源代码进行阅读，阅读打印的类型信息可以得出我们执行了错误类型的 `dynamic_cast` 之前，assert 报错。`cast_node` 函数是在 `BinopExprNode` 的构造函数前初始化成员时被调用的。试图将子节点 cast 为 `ExprNode` 时出错。

可见比起 assertion error 的单行信息，没有调用栈现场的话我们无从确定事故发生在 `BinopExprNode` 构造时，进行排查就大海捞针了很多。

## 参考资料

https://en.wikipedia.org/wiki/Core_dump

https://en.wikipedia.org/wiki/Magnetic-core_memory

https://wiki.archlinux.org/index.php/Core_dump

http://man7.org/linux/man-pages/man5/core.5.html