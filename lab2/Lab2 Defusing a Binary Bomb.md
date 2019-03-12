# Introduction
>The nefarious Dr. Evil has planted a slew of “binary bombs” on our machines. A binary bomb is a program that consists of a sequence of phases. Each phase expects you to type a particular string on stdin. If you type the correct string, then the phase is defused and the bomb proceeds to the next phase. Otherwise, the bomb explodes by printing "BOOM!!!" and then terminating. The bomb is defused when every phase has been defused. 
There are too many bombs for us to deal with, so we are giving each student a bomb to defuse. Your mission, whom you have no choice but to accept, is to defuse your bomb before the due date. Good luck, and welcome to the bomb squad!

# Get Your Bomb
通过SVN获得，之前已经进行了连接，所以只需```$ svn update```即可获得lab2文件夹，包含以下三个文件：
* README: 主要是用于判断这个Bomb是不是你的，里面包含bomb的编号和你的学号
* bomb: 就是一个可执行文件，但是可以通过反汇编```objdump -d bomb > <object_file>```得到汇编文件
* bomb.c: bomb主线的源文件，其实没什么东西，但是看着挺有趣的，有一些作者自言自语的话，自带萌点（嘤）。主要的提示就是不要直接双击运行bomb文件，因为这样相当于什么参数都不给，所以它就会一直爆炸(T^T)。代码贴在下面：
```c
/***************************************************************************
 * Dr. Evil's Insidious Bomb, Version 1.0
 * Copyright 2002, Dr. Evil Incorporated. All rights reserved.
 *
 * LICENSE:
 *
 * Dr. Evil Incorporated (the PERPETRATOR) hereby grants you (the
 * VICTIM) explicit permission to use this bomb (the BOMB).  This is a
 * time limited license, which expires on the death of the VICTIM.
 * The PERPETRATOR takes no responsibility for damage, frustration,
 * insanity, bug-eyes, carpal-tunnel syndrome, loss of sleep, or other
 * harm to the VICTIM.  Unless the PERPETRATOR wants to take credit,
 * that is.  The VICTIM may not distribute this bomb source code to
 * any enemies of the PERPETRATOR.  No VICTIM may debug,
 * reverse-engineer, run "strings" on, decompile, decrypt, or use any
 * other technique to gain knowledge of and defuse the BOMB.  BOMB
 * proof clothing may not be worn when handling this program.  The
 * PERPETRATOR will not apologize for the PERPETRATOR's poor sense of
 * humor.  This license is null and void where the BOMB is prohibited
 * by law.
 ***************************************************************************/

#include <stdio.h>
#include "support.h"
#include "phases.h"

/* 
 * Note to self: Remember to erase this file so my victims will have no
 * idea what is going on, and so they will all blow up in a
 * spectaculary fiendish explosion. -- Dr. Evil 
 */

FILE *infile;

int main(int argc, char *argv[])
{
    char *input;

    /* Note to self: remember to port this bomb to Windows and put a 
     * fantastic GUI on it. */

    /* When run with no arguments, the bomb reads its input lines 
     * from standard input. */
    if (argc == 1) {  
	infile = stdin;
    } 

    /* When run with one argument <file>, the bomb reads from <file> 
     * until EOF, and then switches to standard input. Thus, as you 
     * defuse each phase, you can add its defusing string to <file> and
     * avoid having to retype it. */
    else if (argc == 2) {
	if (!(infile = fopen(argv[1], "r"))) {
	    printf("%s: Error: Couldn't open %s\n", argv[0], argv[1]);
	    exit(8);
	}
    }

    /* You can't call the bomb with more than 1 command line argument. */
    else {
	printf("Usage: %s [<input_file>]\n", argv[0]);
	exit(8);
    }

    /* Do all sorts of secret stuff that makes the bomb harder to defuse. */
    initialize_bomb();

    printf("Welcome to my fiendish little bomb. You have 6 phases with\n");
    printf("which to blow yourself up. Have a nice day!\n");

    /* Hmm...  Six phases must be more secure than one phase! */
    input = read_line();             /* Get input                   */
    phase_1(input);                  /* Run the phase               */
    phase_defused();                 /* Drat!  They figured it out!
				      * Let me know how they did it. */
    printf("Phase 1 defused. How about the next one?\n");

    /* The second phase is harder.  No one will ever figure out
     * how to defuse this... */
    input = read_line();
    phase_2(input);
    phase_defused();
    printf("That's number 2.  Keep going!\n");

    /* I guess this is too easy so far.  Some more complex code will
     * confuse people. */
    input = read_line();
    phase_3(input);
    phase_defused();
    printf("Halfway there!\n");

    /* Oh yeah?  Well, how good is your math?  Try on this saucy problem! */
    input = read_line();
    phase_4(input);
    phase_defused();
    printf("So you got that one.  Try this one.\n");
    
    /* Round and 'round in memory we go, where we stop, the bomb blows! */
    input = read_line();
    phase_5(input);
    phase_defused();
    printf("Good work!  On to the next...\n");

    /* This phase will never be used, since no one will get past the
     * earlier ones.  But just in case, make this one extra hard. */
    input = read_line();
    phase_6(input);
    phase_defused();

    /* Wow, they got it!  But isn't something... missing?  Perhaps
     * something they overlooked?  Mua ha ha ha ha! */
    
    return 0;
}
```

#  Defuse Your Bomb
* 一共有6个phase（各10分）加一个secret_phase（5分），从第四次爆炸开始一次bomb扣一分，所以可以白炸三次2333（为了少扣分，先把explode_bomb和phase_defused两个函数加上断点，发现即将爆炸就kill调试进程）
* 可以将已经知道的答案写在文档里面，在命令行运行bomb直接读取，例如：
	```c
	$./bomb psol.txt //psol.txt的每一行分别为一个phase的答案
	```

# Tools
1. gdb
The GNU debugger, this is a command line debugger tool available on virtually every platform. You can trace through a program line by line, examine memory and registers, look at both the source code and assembly code (we are not giving you the source code for most of your bomb), set breakpoints, set memory watch points, and write scripts. Here are some tips for using ```gdb```.
	* To keep the bomb from blowing up every time you type in a wrong input, you’ll want to learn how to set breakpoints.
	* For other documentation, type ```help```at the gdb command prompt, or type ```man gdb```, or ```info gdb```at a Unix prompt. Some people also like to run gdb under gdb-mode in emacs.
	* http://ipads.se.sjtu.edu.cn/courses/ics/tutorials/gdb-ref.txt
2. objdump -t
This will print out the bomb’s symbol table. The symbol table includes the
names of all functions and global variables in the bomb, the names of all the functions the bomb calls, and their addresses. You may learn something by looking at the function names!
3. objdump -d
Use this to disassemble all of the code in the bomb. You can also just look at individual functions. Reading the assembler code can tell you how the bomb works. Although ```objdump -d```gives you a lot of information, it doesn’t tell you the whole story. Calls to system-level functions are displayed in a cryptic form. For example, a call to ```printf```might appear as:
	```c
	171d: e8 6e f8 ff ff callq f90 <printf@plt>
	```
	To determine that the call was to ```printf```, you would need to disassemble within ```gdb```.
4. strings
This utility will display the printable strings in your bomb.

# Special
```objdump -d```之后的地址并不一定是真的地址，但是相对偏移量是对的。所以可以先```gdb bomb```，在main函数这里设一个断点```break main```，然后运行程序```run```，程序会在断点处停止并显示main的真正地址，计算真正地址与反汇编后的地址之间的偏移量，将反汇编文件中的地址加上偏移量可以得到真正的地址，之后就可以进行正常的设置断点和调试了

# Phase 1
1. 代码如下：
	```c
	00000000000013b0 <phase_1>:
    13b0:	48 83 ec 08          	sub    $0x8,%rsp
    13b4:	48 8d 35 55 10 00 00 	lea    0x1055(%rip),%rsi        # 2410 <_IO_stdin_used+0x150>
    13bb:	e8 43 04 00 00       	callq  1803 <strings_not_equal>
    13c0:	85 c0                	test   %eax,%eax
    13c2:	75 05                	jne    13c9 <phase_1+0x19>
    13c4:	48 83 c4 08          	add    $0x8,%rsp
    13c8:	c3                   	retq   
    13c9:	e8 45 0c 00 00       	callq  2013 <explode_bomb>
    13ce:	eb f4                	jmp    13c4 <phase_1+0x14>
	```
2. 分析：
这个bomb比较简单，首先联网拿到对应的string
	```c
	lea    0x1055(%rip),%rsi  # 2410 <_IO_stdin_used+0x150>
	```
	然后调用函数，从名字上来看是判断两个字符串是否相等
	```c
	13bb:	e8 43 04 00 00       	callq  1803 <strings_not_equal>
	```
	用```test```判断返回值是否为0
	```c
	13c0:	85 c0                	test   %eax,%eax
	```
	不为0就爆炸
	```c
	13c2:	75 05                	jne    13c9 <phase_1+0x19>
	13c9:	e8 45 0c 00 00       	callq  2013 <explode_bomb>
	```
	所以这个时候猜都能猜到是要输入```%rsi```中的字符串，直接```x $rsi```或者一个字符一个字符的```print```出来也行。如果不放心，可以到```<strings_not_equal>```函数里面再去看一看，代码如下：
	```c
	0000000000001803 <strings_not_equal>:
    1803:	41 54                	push   %r12
    1805:	55                   	push   %rbp
    1806:	53                   	push   %rbx
    1807:	48 89 fb             	mov    %rdi,%rbx
    180a:	48 89 f5             	mov    %rsi,%rbp
    180d:	e8 d3 ff ff ff       	callq  17e5 <string_length>
    1812:	41 89 c4             	mov    %eax,%r12d
    1815:	48 89 ef             	mov    %rbp,%rdi
    1818:	e8 c8 ff ff ff       	callq  17e5 <string_length>
    181d:	ba 01 00 00 00       	mov    $0x1,%edx
    1822:	41 39 c4             	cmp    %eax,%r12d
    1825:	74 07                	je     182e <strings_not_equal+0x2b>
    1827:	89 d0                	mov    %edx,%eax
    1829:	5b                   	pop    %rbx
    182a:	5d                   	pop    %rbp
    182b:	41 5c                	pop    %r12
    182d:	c3                   	retq   
    182e:	0f b6 03             	movzbl (%rbx),%eax
    1831:	84 c0                	test   %al,%al
    1833:	74 27                	je     185c <strings_not_equal+0x59>
    1835:	3a 45 00             	cmp    0x0(%rbp),%al
    1838:	75 29                	jne    1863 <strings_not_equal+0x60>
    183a:	48 83 c3 01          	add    $0x1,%rbx
    183e:	48 83 c5 01          	add    $0x1,%rbp
    1842:	0f b6 03             	movzbl (%rbx),%eax
    1845:	84 c0                	test   %al,%al
    1847:	74 0c                	je     1855 <strings_not_equal+0x52>
    1849:	3a 45 00             	cmp    0x0(%rbp),%al
    184c:	74 ec                	je     183a <strings_not_equal+0x37>
    184e:	ba 01 00 00 00       	mov    $0x1,%edx
    1853:	eb d2                	jmp    1827 <strings_not_equal+0x24>
    1855:	ba 00 00 00 00       	mov    $0x0,%edx
    185a:	eb cb                	jmp    1827 <strings_not_equal+0x24>
    185c:	ba 00 00 00 00       	mov    $0x0,%edx
    1861:	eb c4                	jmp    1827 <strings_not_equal+0x24>
    1863:	ba 01 00 00 00       	mov    $0x1,%edx
    1868:	eb bd                	jmp    1827 <strings_not_equal+0x24>
    ```
    发现就是先比较两个字符串的长度，如果相等再逐个比较字符，并没有什么用
3. 结果：
	```c
	When I get angry, Mr. Bigglesworth gets upset.
	```

# Phase 2
1. 代码如下：
	```c
	00000000000013d0 <phase_2>:
    13d0:	55                   	push   %rbp
    13d1:	53                   	push   %rbx
    13d2:	48 83 ec 28          	sub    $0x28,%rsp
    13d6:	48 89 e6             	mov    %rsp,%rsi
    13d9:	e8 71 0c 00 00       	callq  204f <read_six_numbers>
    13de:	83 3c 24 01          	cmpl   $0x1,(%rsp)
    13e2:	74 05                	je     13e9 <phase_2+0x19>
    13e4:	e8 2a 0c 00 00       	callq  2013 <explode_bomb>
    13e9:	48 89 e5             	mov    %rsp,%rbp
    13ec:	bb 01 00 00 00       	mov    $0x1,%ebx
    13f1:	eb 09                	jmp    13fc <phase_2+0x2c>
    13f3:	48 83 c5 04          	add    $0x4,%rbp
    13f7:	83 fb 06             	cmp    $0x6,%ebx
    13fa:	74 15                	je     1411 <phase_2+0x41>
    13fc:	83 c3 01             	add    $0x1,%ebx
    13ff:	89 d8                	mov    %ebx,%eax
    1401:	0f af 45 00          	imul   0x0(%rbp),%eax
    1405:	39 45 04             	cmp    %eax,0x4(%rbp)
    1408:	74 e9                	je     13f3 <phase_2+0x23>
    140a:	e8 04 0c 00 00       	callq  2013 <explode_bomb>
    140f:	eb e2                	jmp    13f3 <phase_2+0x23>
    1411:	48 83 c4 28          	add    $0x28,%rsp
    1415:	5b                   	pop    %rbx
    1416:	5d                   	pop    %rbp
    1417:	c3                   	retq   
    ```
2. 分析：
从下面这句汇编中调用的函数名就可以看出，是要输入六个数字
	```c
	13d9:	e8 71 0c 00 00       	callq  204f <read_six_numbers>
	```
	所以用不着看这个函数具体内容，直接看后面的代码即可
	```c
	13de:	83 3c 24 01          	cmpl   $0x1,(%rsp)
    13e2:	74 05                	je     13e9 <phase_2+0x19>
    13e4:	e8 2a 0c 00 00       	callq  2013 <explode_bomb>
    ```
    从上面的代码可以看出，第一个数字必须是1，否则就会爆炸，接着看下面的代码：
	```c
	13e9:	48 89 e5             	mov    %rsp,%rbp
    13ec:	bb 01 00 00 00       	mov    $0x1,%ebx
    13f1:	eb 09                	jmp    13fc <phase_2+0x2c>
    13f3:	48 83 c5 04          	add    $0x4,%rbp
    13f7:	83 fb 06             	cmp    $0x6,%ebx
    13fa:	74 15                	je     1411 <phase_2+0x41>
    13fc:	83 c3 01             	add    $0x1,%ebx
    13ff:	89 d8                	mov    %ebx,%eax
    1401:	0f af 45 00          	imul   0x0(%rbp),%eax
    1405:	39 45 04             	cmp    %eax,0x4(%rbp)
    1408:	74 e9                	je     13f3 <phase_2+0x23>
    140a:	e8 04 0c 00 00       	callq  2013 <explode_bomb>
    140f:	eb e2                	jmp    13f3 <phase_2+0x23>
    ```
    显然这是一个循环，遍历输入的每一个数字，检查是否合乎要求，当遇到不合要求的数字或者%ebx=6的时候停止循环，改成c代码如下：
    ```c
    %rbp = %rsp;
    for (%ebx = 1; %ebx < 6; %rbp + 4)
    {
    	%ebx++;
    	%eax = %ebx * 0x0(%rbp);
    	if (0x4(%rbp) == %eax) continue;
    	else bomb!!!;
    }
    ```
    即第i个数等于前一个数乘以i，结束
3. 结果：
	```c
	1 2 6 24 120 720
	```

# Phase 3
1. 代码：
	```c
	0000000000001418 <phase_3>:
    1418:	48 83 ec 18          	sub    $0x18,%rsp
    141c:	48 8d 4c 24 07       	lea    0x7(%rsp),%rcx
    1421:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
    1426:	4c 8d 44 24 08       	lea    0x8(%rsp),%r8
    142b:	48 8d 35 34 10 00 00 	lea    0x1034(%rip),%rsi        # 2466 <_IO_stdin_used+0x1a6>
    1432:	b8 00 00 00 00       	mov    $0x0,%eax
    1437:	e8 24 fc ff ff       	callq  1060 <__isoc99_sscanf@plt>
    143c:	83 f8 02             	cmp    $0x2,%eax
    143f:	7e 1f                	jle    1460 <phase_3+0x48>
    1441:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp)
    1446:	0f 87 0c 01 00 00    	ja     1558 <phase_3+0x140>
    144c:	8b 44 24 0c          	mov    0xc(%rsp),%eax
    1450:	48 8d 15 29 10 00 00 	lea    0x1029(%rip),%rdx        # 2480 <_IO_stdin_used+0x1c0>
    1457:	48 63 04 82          	movslq (%rdx,%rax,4),%rax
    145b:	48 01 d0             	add    %rdx,%rax
    145e:	ff e0                	jmpq   *%rax
    1460:	e8 ae 0b 00 00       	callq  2013 <explode_bomb>
    1465:	eb da                	jmp    1441 <phase_3+0x29>
    1467:	b8 67 00 00 00       	mov    $0x67,%eax
    146c:	81 7c 24 08 74 01 00 	cmpl   $0x174,0x8(%rsp)
    1473:	00 
    1474:	0f 84 e8 00 00 00    	je     1562 <phase_3+0x14a>
    147a:	e8 94 0b 00 00       	callq  2013 <explode_bomb>
    147f:	b8 67 00 00 00       	mov    $0x67,%eax
    1484:	e9 d9 00 00 00       	jmpq   1562 <phase_3+0x14a>
    1489:	b8 70 00 00 00       	mov    $0x70,%eax
    148e:	81 7c 24 08 20 03 00 	cmpl   $0x320,0x8(%rsp)
    1495:	00 
    1496:	0f 84 c6 00 00 00    	je     1562 <phase_3+0x14a>
    149c:	e8 72 0b 00 00       	callq  2013 <explode_bomb>
    14a1:	b8 70 00 00 00       	mov    $0x70,%eax
    14a6:	e9 b7 00 00 00       	jmpq   1562 <phase_3+0x14a>
    14ab:	b8 69 00 00 00       	mov    $0x69,%eax
    14b0:	81 7c 24 08 2a 01 00 	cmpl   $0x12a,0x8(%rsp)
    14b7:	00 
    14b8:	0f 84 a4 00 00 00    	je     1562 <phase_3+0x14a>
    14be:	e8 50 0b 00 00       	callq  2013 <explode_bomb>
    14c3:	b8 69 00 00 00       	mov    $0x69,%eax
    14c8:	e9 95 00 00 00       	jmpq   1562 <phase_3+0x14a>
    14cd:	b8 68 00 00 00       	mov    $0x68,%eax
    14d2:	81 7c 24 08 77 01 00 	cmpl   $0x177,0x8(%rsp)
    14d9:	00 
    14da:	0f 84 82 00 00 00    	je     1562 <phase_3+0x14a>
    14e0:	e8 2e 0b 00 00       	callq  2013 <explode_bomb>
    14e5:	b8 68 00 00 00       	mov    $0x68,%eax
    14ea:	eb 76                	jmp    1562 <phase_3+0x14a>
    14ec:	b8 65 00 00 00       	mov    $0x65,%eax
    14f1:	81 7c 24 08 b8 03 00 	cmpl   $0x3b8,0x8(%rsp)
    14f8:	00 
    14f9:	74 67                	je     1562 <phase_3+0x14a>
    14fb:	e8 13 0b 00 00       	callq  2013 <explode_bomb>
    1500:	b8 65 00 00 00       	mov    $0x65,%eax
    1505:	eb 5b                	jmp    1562 <phase_3+0x14a>
    1507:	b8 62 00 00 00       	mov    $0x62,%eax
    150c:	81 7c 24 08 1c 01 00 	cmpl   $0x11c,0x8(%rsp)
    1513:	00 
    1514:	74 4c                	je     1562 <phase_3+0x14a>
    1516:	e8 f8 0a 00 00       	callq  2013 <explode_bomb>
    151b:	b8 62 00 00 00       	mov    $0x62,%eax
    1520:	eb 40                	jmp    1562 <phase_3+0x14a>
    1522:	b8 71 00 00 00       	mov    $0x71,%eax
    1527:	81 7c 24 08 9a 01 00 	cmpl   $0x19a,0x8(%rsp)
    152e:	00 
    152f:	74 31                	je     1562 <phase_3+0x14a>
    1531:	e8 dd 0a 00 00       	callq  2013 <explode_bomb>
    1536:	b8 71 00 00 00       	mov    $0x71,%eax
    153b:	eb 25                	jmp    1562 <phase_3+0x14a>
    153d:	b8 78 00 00 00       	mov    $0x78,%eax
    1542:	81 7c 24 08 56 01 00 	cmpl   $0x156,0x8(%rsp)
    1549:	00 
    154a:	74 16                	je     1562 <phase_3+0x14a>
    154c:	e8 c2 0a 00 00       	callq  2013 <explode_bomb>
    1551:	b8 78 00 00 00       	mov    $0x78,%eax
    1556:	eb 0a                	jmp    1562 <phase_3+0x14a>
    1558:	e8 b6 0a 00 00       	callq  2013 <explode_bomb>
    155d:	b8 65 00 00 00       	mov    $0x65,%eax
    1562:	3a 44 24 07          	cmp    0x7(%rsp),%al
    1566:	74 05                	je     156d <phase_3+0x155>
    1568:	e8 a6 0a 00 00       	callq  2013 <explode_bomb>
    156d:	48 83 c4 18          	add    $0x18,%rsp
    1571:	c3                   	retq   
	```
2. 分析：
	代码很长，首先可以看到调用了sscanf函数读取输入的数据，
	```c
	143c:	83 f8 02             	cmp    $0x2,%eax
    143f:	7e 1f                	jle    1460 <phase_3+0x48>
    1441:	83 7c 24 0c 07       	cmpl   $0x7,0xc(%rsp)
    1446:	0f 87 0c 01 00 00    	ja     1558 <phase_3+0x140>
    ```
    从上面这两个跳转可以看出输入需要满足输入个数大于2（当然这是根据后面的分析猜测sscanf函数返回的是输入数据的个数），某个东西小于等于7，否则就会跳转爆炸。
    ```c
    144c:	8b 44 24 0c          	mov    0xc(%rsp),%eax
    1450:	48 8d 15 29 10 00 00 	lea    0x1029(%rip),%rdx        # 2480 <_IO_stdin_used+0x1c0>
    1457:	48 63 04 82          	movslq (%rdx,%rax,4),%rax
    145b:	48 01 d0             	add    %rdx,%rax
    145e:	ff e0                	jmpq   *%rax
    ```
    上面这段先计算后跳转，显然很像switch语句，由于代码中没有给跳转表，所以只能采取“试”的方式，我一开始都是拿“1 2 3”试的，结果都不知道发生了什么直接bomb，哭了。后来怀疑第一个数有要求，所以选择了4进行尝试。之后用```stepi```一步一步执行观察跳转位置就可以了。
3. 结果：
	```c
	4 e 952
	```
	我选择了4为第一个数据，应该也是可以选择其他数据作为第一个的，这里只给出这一组解。

# Phase 4
1. 代码：
	```c
	00000000000015a0 <phase_4>:
    15a0:	48 83 ec 18          	sub    $0x18,%rsp
    15a4:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
    15a9:	48 8d 35 bc 0e 00 00 	lea    0xebc(%rip),%rsi        # 246c <_IO_stdin_used+0x1ac>
    15b0:	b8 00 00 00 00       	mov    $0x0,%eax
    15b5:	e8 a6 fa ff ff       	callq  1060 <__isoc99_sscanf@plt>
    15ba:	83 f8 01             	cmp    $0x1,%eax
    15bd:	74 1f                	je     15de <phase_4+0x3e>
    15bf:	e8 4f 0a 00 00       	callq  2013 <explode_bomb>
    15c4:	8b 7c 24 0c          	mov    0xc(%rsp),%edi
    15c8:	e8 a5 ff ff ff       	callq  1572 <func4>
    15cd:	3d c2 2a 00 00       	cmp    $0x2ac2,%eax
    15d2:	74 05                	je     15d9 <phase_4+0x39>
    15d4:	e8 3a 0a 00 00       	callq  2013 <explode_bomb>
    15d9:	48 83 c4 18          	add    $0x18,%rsp
    15dd:	c3                   	retq   
    15de:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
    15e3:	7f df                	jg     15c4 <phase_4+0x24>
    15e5:	eb d8                	jmp    15bf <phase_4+0x1f>
	```
2. 分析：
	从下列代码分析可以知道，输入数据个数应为1，否则爆炸
	```c
	15b5:	e8 a6 fa ff ff       	callq  1060 <__isoc99_sscanf@plt>
    15ba:	83 f8 01             	cmp    $0x1,%eax
    15bd:	74 1f                	je     15de <phase_4+0x3e>
    15bf:	e8 4f 0a 00 00       	callq  2013 <explode_bomb>
	```
	之后跳转到```15de <phase_4+0x3e>```
	```c
	15de:	83 7c 24 0c 00       	cmpl   $0x0,0xc(%rsp)
    15e3:	7f df                	jg     15c4 <phase_4+0x24>
    15e5:	eb d8                	jmp    15bf <phase_4+0x1f>
	```
	猜测```0xc(%rsp)```应该是我输入的这个数据（毕竟这个时候只输入了一个数据），输入应大于0，之后跳转到```15c4 <phase_4+0x24>```
	```c
	15c4:	8b 7c 24 0c          	mov    0xc(%rsp),%edi
    15c8:	e8 a5 ff ff ff       	callq  1572 <func4>
	```
	将输入的这个数据作为参数，调用函数```func4```
	```c
	0000000000001572 <func4>:
    1572:	b8 01 00 00 00       	mov    $0x1,%eax
    1577:	83 ff 01             	cmp    $0x1,%edi
    157a:	7e 22                	jle    159e <func4+0x2c>
    157c:	55                   	push   %rbp
    157d:	53                   	push   %rbx
    157e:	48 83 ec 08          	sub    $0x8,%rsp
    1582:	89 fb                	mov    %edi,%ebx
    1584:	8d 7f ff             	lea    -0x1(%rdi),%edi
    1587:	e8 e6 ff ff ff       	callq  1572 <func4>
    158c:	89 c5                	mov    %eax,%ebp
    158e:	8d 7b fe             	lea    -0x2(%rbx),%edi
    1591:	e8 dc ff ff ff       	callq  1572 <func4>
    1596:	01 e8                	add    %ebp,%eax
    1598:	48 83 c4 08          	add    $0x8,%rsp
    159c:	5b                   	pop    %rbx 
    159d:	5d                   	pop    %rbp
    159e:	f3 c3                	repz retq 
	```
	发现这个函数里面出现了自己的名字，猜测是个递归函数，不要急，先把这个递归的逻辑梳理下来：
	```c
	func4(%rdi)
	{
		%rax = 1;
		if (%rdi <= 1) return %rax;
		%rbx = %rdi;
		%rdi--;
		%rax = func4(%rdi);
		%rbp = %rax;
		%rdi = %rbx - 2;
		%rax = func4(%rdi);
		%rax += %rbp;
		return %rax;
	}
	```
	递归分析起来还是比较麻烦，可以先暂时跳出这个函数，看一下主函数是怎么利用这个函数的返回值的
	```c
	15cd:	3d c2 2a 00 00       	cmp    $0x2ac2,%eax
    15d2:	74 05                	je     15d9 <phase_4+0x39>
    15d4:	e8 3a 0a 00 00       	callq  2013 <explode_bomb>
	```
	显然返回值需要等于```$0x2ac2```即```10946```，接下来可以正着从函数一步一步分析，也可以先大致猜个数试一试，我选择了后者。
3. 结果：
	```c
	20
	```

# Phase 5
1. 代码：
	```c
	00000000000015e7 <phase_5>:
    15e7:	53                   	push   %rbx
    15e8:	48 89 fb             	mov    %rdi,%rbx
    15eb:	e8 f5 01 00 00       	callq  17e5 <string_length>
    15f0:	83 f8 06             	cmp    $0x6,%eax
    15f3:	74 05                	je     15fa <phase_5+0x13>
    15f5:	e8 19 0a 00 00       	callq  2013 <explode_bomb>
    15fa:	48 89 d8             	mov    %rbx,%rax
    15fd:	48 8d 7b 06          	lea    0x6(%rbx),%rdi
    1601:	b9 00 00 00 00       	mov    $0x0,%ecx
    1606:	48 8d 35 93 0e 00 00 	lea    0xe93(%rip),%rsi        # 24a0 <array.3089>
    160d:	0f b6 10             	movzbl (%rax),%edx
    1610:	83 e2 0f             	and    $0xf,%edx
    1613:	03 0c 96             	add    (%rsi,%rdx,4),%ecx
    1616:	48 83 c0 01          	add    $0x1,%rax
    161a:	48 39 f8             	cmp    %rdi,%rax
    161d:	75 ee                	jne    160d <phase_5+0x26>
    161f:	83 f9 2f             	cmp    $0x2f,%ecx
    1622:	74 05                	je     1629 <phase_5+0x42>
    1624:	e8 ea 09 00 00       	callq  2013 <explode_bomb>
    1629:	5b                   	pop    %rbx
    162a:	c3                   	retq   
	```
2. 分析：
	首先，根据下面代码中函数名可以推测出输入字符串长度应该为6
	```c
	15eb:	e8 f5 01 00 00       	callq  17e5 <string_length>
    15f0:	83 f8 06             	cmp    $0x6,%eax
    15f3:	74 05                	je     15fa <phase_5+0x13>
    15f5:	e8 19 0a 00 00       	callq  2013 <explode_bomb>
	```
	根据下面这行语句可以发现，程序从远端读取了一个数组
	```c
	1606:	48 8d 35 93 0e 00 00 	lea    0xe93(%rip),%rsi        # 24a0 <array.3089>
	```
	然后，根据下面语句及结构推测应该是个循环
	```c
	160d:	0f b6 10             	movzbl (%rax),%edx
    1610:	83 e2 0f             	and    $0xf,%edx
    1613:	03 0c 96             	add    (%rsi,%rdx,4),%ecx
    1616:	48 83 c0 01          	add    $0x1,%rax
    161a:	48 39 f8             	cmp    %rdi,%rax
    161d:	75 ee                	jne    160d <phase_5+0x26>
	```
	可以发现，上述代码遍历了输入的六个数据，分别记为```x1,x2,x3,x4,x5,x6```，并且累加数组中下标为```xn```的元素，最后判断累加的结果是否为设定值```$0x2f```即```47```
	```c
	161f:	83 f9 2f             	cmp    $0x2f,%ecx
    1622:	74 05                	je     1629 <phase_5+0x42>
    1624:	e8 ea 09 00 00       	callq  2013 <explode_bomb>
	```
	所以我们就需要逐个查看给定数组中的元素，然后选择合适的输入让对应六个元素的和为47即可
3. 结果：
	```c
	12579K
	```
	应该是有多解的，此处只提供一组解

# Phase 6
1. 代码：
	```c
	0000000000001693 <phase_6>:
    1693:	48 83 ec 08          	sub    $0x8,%rsp
    1697:	ba 0a 00 00 00       	mov    $0xa,%edx
    169c:	be 00 00 00 00       	mov    $0x0,%esi
    16a1:	e8 8a f9 ff ff       	callq  1030 <strtol@plt>
    16a6:	89 05 a4 30 20 00    	mov    %eax,0x2030a4(%rip)        # 204750 <node0>
    16ac:	48 8d 3d 9d 30 20 00 	lea    0x20309d(%rip),%rdi        # 204750 <node0>
    16b3:	e8 73 ff ff ff       	callq  162b <fun6>
    16b8:	48 8b 40 08          	mov    0x8(%rax),%rax
    16bc:	48 8b 40 08          	mov    0x8(%rax),%rax
    16c0:	48 8b 40 08          	mov    0x8(%rax),%rax
    16c4:	8b 0d 86 30 20 00    	mov    0x203086(%rip),%ecx        # 204750 <node0>
    16ca:	39 08                	cmp    %ecx,(%rax)
    16cc:	74 05                	je     16d3 <phase_6+0x40>
    16ce:	e8 40 09 00 00       	callq  2013 <explode_bomb>
    16d3:	48 83 c4 08          	add    $0x8,%rsp
    16d7:	c3                   	retq   
	```
2. 分析：
	这个题做的很迷，因为显然可以看出使用了链表的结构```<node0>```而且调用了一个函数```func6```，代码如下：
	```c
	000000000000162b <fun6>:
    162b:	48 89 f8             	mov    %rdi,%rax
    162e:	4c 8b 47 08          	mov    0x8(%rdi),%r8
    1632:	48 c7 47 08 00 00 00 	movq   $0x0,0x8(%rdi)
    1639:	00 
    163a:	48 89 fa             	mov    %rdi,%rdx
    163d:	4d 85 c0             	test   %r8,%r8
    1640:	75 2e                	jne    1670 <fun6+0x45>
    1642:	f3 c3                	repz retq 
    1644:	48 89 d1             	mov    %rdx,%rcx
    1647:	48 8b 51 08          	mov    0x8(%rcx),%rdx
    164b:	48 85 d2             	test   %rdx,%rdx
    164e:	74 04                	je     1654 <fun6+0x29>
    1650:	39 32                	cmp    %esi,(%rdx)
    1652:	7f f0                	jg     1644 <fun6+0x19>
    1654:	48 39 d1             	cmp    %rdx,%rcx
    1657:	74 33                	je     168c <fun6+0x61>
    1659:	4c 89 41 08          	mov    %r8,0x8(%rcx)
    165d:	49 8b 48 08          	mov    0x8(%r8),%rcx
    1661:	49 89 50 08          	mov    %rdx,0x8(%r8)
    1665:	48 89 c2             	mov    %rax,%rdx
    1668:	49 89 c8             	mov    %rcx,%r8
    166b:	48 85 c9             	test   %rcx,%rcx
    166e:	74 21                	je     1691 <fun6+0x66>
    1670:	48 85 d2             	test   %rdx,%rdx
    1673:	74 12                	je     1687 <fun6+0x5c>
    1675:	41 8b 30             	mov    (%r8),%esi
    1678:	48 89 c1             	mov    %rax,%rcx
    167b:	39 32                	cmp    %esi,(%rdx)
    167d:	7f c8                	jg     1647 <fun6+0x1c>
    167f:	48 89 c2             	mov    %rax,%rdx
    1682:	4c 89 c0             	mov    %r8,%rax
    1685:	eb d6                	jmp    165d <fun6+0x32>
    1687:	48 89 c1             	mov    %rax,%rcx
    168a:	eb c8                	jmp    1654 <fun6+0x29>
    168c:	4c 89 c0             	mov    %r8,%rax
    168f:	eb cc                	jmp    165d <fun6+0x32>
    1691:	f3 c3                	repz retq 
	```
	很长，关键是还跳来跳去让人迷茫不知所措，所以我选择先跳过这个函数，看之后的代码：
	```c
	16b8:	48 8b 40 08          	mov    0x8(%rax),%rax
    16bc:	48 8b 40 08          	mov    0x8(%rax),%rax
    16c0:	48 8b 40 08          	mov    0x8(%rax),%rax
    16c4:	8b 0d 86 30 20 00    	mov    0x203086(%rip),%ecx        # 204750 <node0>
    16ca:	39 08                	cmp    %ecx,(%rax)
    16cc:	74 05                	je     16d3 <phase_6+0x40>
    16ce:	e8 40 09 00 00       	callq  2013 <explode_bomb>
	```
	可以看出```%rax```里面存放了一个地址，拿这个地址加24之后的位置处存放的数据与设定值```%ecx```比较，相等就算成功，所以我们可以先输入几次数猜一猜```func6```到底干了什么……
	经过几次尝试之后发现……```(%rax)```就是我们输入的那个数，而```%ecx```这个值又很容易拿到，所以直接输入即可
3. 结果：
	```c
	836
	```

# Secret Phase
1. 从源文件中明显可以看出这个程序有一个隐藏的关卡，当然从反汇编的文件中也很容易找到下面这一串代码：
	```c
	0000000000001715 <secret_phase>:
    1715:	53                   	push   %rbx
    1716:	e8 75 09 00 00       	callq  2090 <read_line>
    171b:	ba 0a 00 00 00       	mov    $0xa,%edx
    1720:	be 00 00 00 00       	mov    $0x0,%esi
    1725:	48 89 c7             	mov    %rax,%rdi
    1728:	e8 03 f9 ff ff       	callq  1030 <strtol@plt>
    172d:	48 89 c3             	mov    %rax,%rbx
    1730:	8d 40 ff             	lea    -0x1(%rax),%eax
    1733:	3d e8 03 00 00       	cmp    $0x3e8,%eax
    1738:	77 2b                	ja     1765 <secret_phase+0x50>
    173a:	89 de                	mov    %ebx,%esi
    173c:	48 8d 3d 2d 2f 20 00 	lea    0x202f2d(%rip),%rdi        # 204670 <n1>
    1743:	e8 90 ff ff ff       	callq  16d8 <fun7>
    1748:	83 f8 04             	cmp    $0x4,%eax
    174b:	74 05                	je     1752 <secret_phase+0x3d>
    174d:	e8 c1 08 00 00       	callq  2013 <explode_bomb>
    1752:	48 8d 3d e7 0c 00 00 	lea    0xce7(%rip),%rdi        # 2440 <_IO_stdin_used+0x180>
    1759:	e8 f2 f7 ff ff       	callq  f50 <puts@plt>
    175e:	e8 2e 0a 00 00       	callq  2191 <phase_defused>
    1763:	5b                   	pop    %rbx
    1764:	c3                   	retq   
    1765:	e8 a9 08 00 00       	callq  2013 <explode_bomb>
    176a:	eb ce                	jmp    173a <secret_phase+0x25>
	```
	找到这个phase以及附带的fun7并不难，难的是怎么进入这个phase
2. 正确进入secret_phase：
	首先我们通过观察main函数可以发现，每次成功通过一个phase之后，都会有一句```callq  2191 <phase_defused>```，虽然知道这个函数可以将我们拆炸弹的情况实时同步到网站上，但是也可能有其他的功能，猜测入口可能在这里，所以进去看一下这个函数：
	```c
	0000000000002191 <phase_defused>:
    2191:	48 83 ec 68          	sub    $0x68,%rsp
    2195:	bf 01 00 00 00       	mov    $0x1,%edi
    219a:	e8 b1 f8 ff ff       	callq  1a50 <send_msg>
    219f:	83 3d 56 26 20 00 06 	cmpl   $0x6,0x202656(%rip)        # 2047fc <num_input_strings>
    21a6:	74 05                	je     21ad <phase_defused+0x1c>
    21a8:	48 83 c4 68          	add    $0x68,%rsp
    21ac:	c3                   	retq   
    21ad:	48 8d 4c 24 10       	lea    0x10(%rsp),%rcx
    21b2:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
    21b7:	48 8d 35 8f 06 00 00 	lea    0x68f(%rip),%rsi        # 284d <array.3089+0x3ad>
    21be:	48 8d 3d 4b 27 20 00 	lea    0x20274b(%rip),%rdi        # 204910 <input_strings+0xf0>
    21c5:	b8 00 00 00 00       	mov    $0x0,%eax
    21ca:	e8 91 ee ff ff       	callq  1060 <__isoc99_sscanf@plt>
    21cf:	83 f8 02             	cmp    $0x2,%eax
    21d2:	74 1a                	je     21ee <phase_defused+0x5d>
    21d4:	48 8d 3d c5 03 00 00 	lea    0x3c5(%rip),%rdi        # 25a0 <array.3089+0x100>
    21db:	e8 70 ed ff ff       	callq  f50 <puts@plt>
    21e0:	48 8d 3d e9 03 00 00 	lea    0x3e9(%rip),%rdi        # 25d0 <array.3089+0x130>
    21e7:	e8 64 ed ff ff       	callq  f50 <puts@plt>
    21ec:	eb ba                	jmp    21a8 <phase_defused+0x17>
    21ee:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
    21f3:	48 8d 35 59 06 00 00 	lea    0x659(%rip),%rsi        # 2853 <array.3089+0x3b3>
    21fa:	e8 04 f6 ff ff       	callq  1803 <strings_not_equal>
    21ff:	85 c0                	test   %eax,%eax
    2201:	75 d1                	jne    21d4 <phase_defused+0x43>
    2203:	48 8d 3d 36 03 00 00 	lea    0x336(%rip),%rdi        # 2540 <array.3089+0xa0>
    220a:	e8 41 ed ff ff       	callq  f50 <puts@plt>
    220f:	48 8d 3d 52 03 00 00 	lea    0x352(%rip),%rdi        # 2568 <array.3089+0xc8>
    2216:	e8 35 ed ff ff       	callq  f50 <puts@plt>
    221b:	b8 00 00 00 00       	mov    $0x0,%eax
    2220:	e8 f0 f4 ff ff       	callq  1715 <secret_phase>
    2225:	eb ad                	jmp    21d4 <phase_defused+0x43>
    2227:	66 0f 1f 84 00 00 00 	nopw   0x0(%rax,%rax,1)
    222e:	00 00 
	```
	发现真的有提到隐藏关卡
	```c
	2220:	e8 f0 f4 ff ff       	callq  1715 <secret_phase>
	```
	所以倒着看一下是在哪里跳到这里的，有一些从数组中拿到值然后调用puts函数的可以直接忽略，因为明显是在输出他写的一些话，所以整理下来就变成了下面这样：
	```c
	219f:	83 3d 56 26 20 00 06 	cmpl   $0x6,0x202656(%rip)        # 2047fc <num_input_strings>
    21a6:	74 05                	je     21ad <phase_defused+0x1c>
	21ad:	48 8d 4c 24 10       	lea    0x10(%rsp),%rcx
    21b2:	48 8d 54 24 0c       	lea    0xc(%rsp),%rdx
    21b7:	48 8d 35 8f 06 00 00 	lea    0x68f(%rip),%rsi        # 284d <array.3089+0x3ad>
    21be:	48 8d 3d 4b 27 20 00 	lea    0x20274b(%rip),%rdi        # 204910 <input_strings+0xf0>
    21c5:	b8 00 00 00 00       	mov    $0x0,%eax
    21ca:	e8 91 ee ff ff       	callq  1060 <__isoc99_sscanf@plt>
    21cf:	83 f8 02             	cmp    $0x2,%eax
    21d2:	74 1a                	je     21ee <phase_defused+0x5d>
    21ec:	eb ba                	jmp    21a8 <phase_defused+0x17>
    21ee:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
    21f3:	48 8d 35 59 06 00 00 	lea    0x659(%rip),%rsi        # 2853 <array.3089+0x3b3>
    21fa:	e8 04 f6 ff ff       	callq  1803 <strings_not_equal>
    21ff:	85 c0                	test   %eax,%eax
    2201:	75 d1                	jne    21d4 <phase_defused+0x43>
    221b:	b8 00 00 00 00       	mov    $0x0,%eax
    2220:	e8 f0 f4 ff ff       	callq  1715 <secret_phase>
	```
	第一行应该是判断一些是否六个关卡全部通过，然后将远端数组中的一个值拿出来放在```%rsi```里，将预先存的一个字符串拿出来放在```%rdi```里面，跟另外两个参数一起，调用sscanf函数。单步调试可以发现前两个参数分别是关卡号和读取格式，即在第四关用```%d %s```的格式读入数据，后两个参数其实就是我的输入值。
	之后判断输入数据是否为两个，并且判断一下第二个数据（也就是特定字符串）是否和设定的关键字一致，不相同就爆炸，相同的话，恭喜进入隐藏关卡。
	```c
	21ca:	e8 91 ee ff ff       	callq  1060 <__isoc99_sscanf@plt>
    21cf:	83 f8 02             	cmp    $0x2,%eax
    21d2:	74 1a                	je     21ee <phase_defused+0x5d>
    21ee:	48 8d 7c 24 10       	lea    0x10(%rsp),%rdi
    21f3:	48 8d 35 59 06 00 00 	lea    0x659(%rip),%rsi        # 2853 <array.3089+0x3b3>
    21fa:	e8 04 f6 ff ff       	callq  1803 <strings_not_equal>
    21ff:	85 c0                	test   %eax,%eax
    2201:	75 d1                	jne    21d4 <phase_defused+0x43>
	```
	当然这个关键字可以在单步调试中得到，我的是```austinpowers```（奥斯汀的力量，中二感十足的key word）而我们只需要在第四关不仅输入那一关的密码，同时也输入这个key word即可进入隐藏关卡
3. 分析secret_phase
	首先读入数据，并且不能超过特定值```$0x3e8```，否则bomb
	```c
	1728:	e8 03 f9 ff ff       	callq  1030 <strtol@plt>
    172d:	48 89 c3             	mov    %rax,%rbx
    1730:	8d 40 ff             	lea    -0x1(%rax),%eax
    1733:	3d e8 03 00 00       	cmp    $0x3e8,%eax
    1738:	77 2b                	ja     1765 <secret_phase+0x50>
    1765:	e8 a9 08 00 00       	callq  2013 <explode_bomb>
	```
	然后将设定值```<n1>```和输入值作为参数调用```fun7```，判断输出值是否等于4
	```c
	 173a:	89 de                	mov    %ebx,%esi
    173c:	48 8d 3d 2d 2f 20 00 	lea    0x202f2d(%rip),%rdi        # 204670 <n1>
    1743:	e8 90 ff ff ff       	callq  16d8 <fun7>
    1748:	83 f8 04             	cmp    $0x4,%eax
    174b:	74 05                	je     1752 <secret_phase+0x3d>
    174d:	e8 c1 08 00 00       	callq  2013 <explode_bomb>
	```
	看看```fun7```里面有什么：
	```c
	00000000000016d8 <fun7>:
    16d8:	48 85 ff             	test   %rdi,%rdi
    16db:	74 32                	je     170f <fun7+0x37>
    16dd:	48 83 ec 08          	sub    $0x8,%rsp
    16e1:	8b 17                	mov    (%rdi),%edx
    16e3:	39 f2                	cmp    %esi,%edx
    16e5:	7f 1b                	jg     1702 <fun7+0x2a>
    16e7:	b8 00 00 00 00       	mov    $0x0,%eax
    16ec:	39 f2                	cmp    %esi,%edx
    16ee:	74 0d                	je     16fd <fun7+0x25>
    16f0:	48 8b 7f 10          	mov    0x10(%rdi),%rdi
    16f4:	e8 df ff ff ff       	callq  16d8 <fun7>
    16f9:	8d 44 00 01          	lea    0x1(%rax,%rax,1),%eax
    16fd:	48 83 c4 08          	add    $0x8,%rsp
    1701:	c3                   	retq   
    1702:	48 8b 7f 08          	mov    0x8(%rdi),%rdi
    1706:	e8 cd ff ff ff       	callq  16d8 <fun7>
    170b:	01 c0                	add    %eax,%eax
    170d:	eb ee                	jmp    16fd <fun7+0x25>
    170f:	b8 ff ff ff ff       	mov    $0xffffffff,%eax
    1714:	c3                   	retq
	```
	显然又是一个递归，照例把结构梳理一下：
	```c
	fun7(%rdi, %rsi)
	{
		if (%rdi == 0)
		{
			%rax = -1;
			return %rax;
		}
		%rdx = (%rdi);
		if (%rdx > %rsi)
		{
			%rdi += 8;
			%rax = fun7(%rdi, %rsi);
			%rax *= 2;
			return %rax;
		}
		%rax = 0;
		if (%rdx == %rsi) return %rax;
		%rdi += 16;
		%rax = fun7(%rdi, %rsi);
		%rax = 2 * %rax + 1;
		return %rax;
	}
	```
	因为我想得到的返回值是4，所以考虑```4 = 2 * (2 * (2 * 0 + 1))```这样的组合方式，即迭代四次，前两次让```(%rdi) > %rsi```，第三次让```(%rdi) < %rsi```，第四次让```(%rdi) == %rsi```。通过单步调试可以得到这三次迭代中```(%rdi)```的值，发现第二次和第三次分别为8和6，而第四次正好为7，得到答案，收工！
4. 结果：
	```c
	7
	```

>小结：总结起来题型无外乎是循环、递归和跳转表，难度呈递增态，凡是涉及到jmp都让人头疼。关键是要冷静分析，在纸上把结构梳理一下，写成c语言或者其他随便什么语言都要比看汇编语言要舒适的多ORZ
