---
title: 实验1：汇编指令编程
date: 2016-06-22 16:34:19
tags: 汇编语言
---
### debug的使用
Debug是DOS、Windows都提供的实模式（8086方式）程序的调试工具，使用它，可以查看CPU各种寄存器的内容，内存情况可机器码级跟踪程序的运行。
<!-- more -->
实验中用到的debug功能：

- -r 查看、改变CPU寄存器的内容
- -d 查看内存中的内容
- -e 改写内存中的内容
- -u 将内存中的机器指令翻译成汇编指令
- -t 执行一条机器指令
- -a 已汇编指令的格式在内存中写入一条机器指令

>注意：windows 7 以后就不提供带有debug的commond工具了，这里实验使用MS-DOS虚拟机完成实验。

① 用R查看、改变CPU寄存器的内容

![用R查看、改变CPU寄存器的内容][1]

注意CS和IP的值，CS=0CA2，IP = 0100， 也就是说，内存0CA2：0100处的指令为当前CPU要读取执行的指令。下方还可以看到其对应的机器码与汇编指令。

② 用R改变寄存器中的内容

![用R改变寄存器中的内容][2]

③ 用D来查看内存中的内容

![用D来查看内存中的内容][3]

使用“d 段地址：偏移地址”的格式命令将列出从指定单元开始的128个内存单元的内容。 使用“ f ”表示列出一行。

结果显示中，左边的为内存地址，中间为地址对应的值，右边为该值对应的ASCII码。

![一些其他查看格式][4]

>在使用“d 段地址：偏移地址”的格式命令后，接着使用D命令，可列出后续的内容。

④ 用debug的E来改写内存中的值

![用debug的E来改写内存中的值][5]

可以写入数字、字符和字符串，在右边可以看到写入的字符与字符串。

⑤ 用E命令向内存中写入机器码，用U命令查看机器码的含义，用T命令执行内存中的机器码

![E U][6]

**在执行机器码之前，需要修改CS和IP的值，使其指向当前代码的起始内存位置。**

![T][7]


⑥ 用A命令以汇编指令的形式在内存中写入机器指令

![用A命令以汇编指令的形式在内存中写入机器指令][8]

![执行][9]

最后，做一个有趣的实验，向内存块B810：0000写入：

![做一个有趣的实验][10]

仔细观察，屏幕上某些地方显示了一些奇怪的字符。这是怎么回事呢？留作大家思考。

[1]: http://static.zybuluo.com/guoxs/9a4p6ax934t80z9etspmgbsb/10.png
[2]: http://static.zybuluo.com/guoxs/dj2gwgty2q74z9kein1egyrb/11.png
[3]: http://static.zybuluo.com/guoxs/uu336hqn6gibqxwnb7vas22u/12.png
[4]: http://static.zybuluo.com/guoxs/ecmcub0a1tvxqkkw1yudz2q1/13.png
[5]: http://static.zybuluo.com/guoxs/o9w7qhxew2cwzct74473fm4k/14.png
[6]: http://static.zybuluo.com/guoxs/v7o2daoh4paur50a5qrpid0y/15.png
[7]: http://static.zybuluo.com/guoxs/sf4b0ejpglqb3wuykag3idnb/16.png
[8]: http://static.zybuluo.com/guoxs/mhjlgu0ndtai90onmstzv57e/17.png
[9]: http://static.zybuluo.com/guoxs/fknk4hodomzunqf6gwjyrwcf/18.png
[10]: http://static.zybuluo.com/guoxs/2ny7437gdklwqgich33lvid5/20.png