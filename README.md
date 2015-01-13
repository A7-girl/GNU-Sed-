#sed, a stream editor
---
##sed, a stream editor

这份文档基于GNU sed 4.2.1版本。

版权&copy;1998,1999,2001,2003,2003,2004 Free Software Foundation, Inc.

##1 介绍

sed是一种流编辑器。流编辑器用于对输入流（文件或者管道输入）进行基本的文本转换。虽然在某些方面和允许进行脚本编辑的编辑器类似（例如ed）,但是sed每次只处理一行，效率更高。和其它类型编辑器不同的是：sed可以过滤管道中的文本。

---

##2 使用

通常sed都这么使用：

	sed SCRIPT INPUTFILE...

完整的使用方式：
	
	sed OPTIONS... [SCRIPT] [INPUTFILE...]

如果不指定INPUTFILE，或者INPUTFILE是-，sed会从标准输入过滤内容。第一个非可选参数会被认为是脚本而非待编辑文件当且仅当其他选项未指定脚本，即没有指定-e或者-f选项。

sed可以指定以下命令行选项：

--version  
打印sed版本和版权信息。

--help  
打印命令行参数的用法以及bug报告地址。

-n  
--quiet  
--silent
sed在每个脚本执行周期结束会默认打印模式空间中的内容（参见sed如何工作）。这些选项可以阻止上述默认打印，并且只有在明确指定p命令的时候才会输出内容。

-e script  
--expression=script  
把scripts中的命令添加到处理输入的命令集中。

-f script-file  
--file=script-file  
把script-file中的命令添加到处理输入的命令集中。

-i[SUFFIX]  
--in-place[=SUFFIX]  
这个选项是指文件会被改动。GNU sed会创建一个临时文件并把输出指定到临时文件而不是标准输出。

该选项和-s含义相同。

当到达文件结尾时，临时文件会被重命名为原始文件的名称。如果提供了扩展名，那么原始文件会在重命名临时文件之前先改名，以做备份。

上述过程遵循以下原则：如果扩展名不包含任何一个\*号，该扩展名会被加到当前文件名称后作为后缀；例如：
如果扩展名包含一个或多个\*号，每个\*号都会被替换为当前文件名。这样你可以为备份文件添加前缀，甚至可以把备份文件放到其他目录（前提是该目录已存在）。

如果没有提供扩展名，那么原始文件会被覆盖并且没有备份。

-l N  
--line-length=N  
指定一条命令自动换行的长度。0表示不进行换行。默认值是70。

--posix  
GNU sed包含一些对POSIX sed的扩展。为了简化脚本，指定该选项会禁用本文档描述的所有扩展及一些额外的命令。大部分扩展可以处理POSIX语法外的sed程序，其中部分还和POSIX标准冲突（例如报告Bug中的命令N）。如果只想禁用冲突的扩展，需要把变量POSIXLY_CORRECT设置为一个非空值。

-b  
-binary  
这个选项任何平台上都适用，但只有在区分文本文件和二进制文件的操作系统上才会生效。在MS-DOS、Windows、Cygwin-text这些以回车换行作为行结束符的平台上，sed找不到结束的回车符。当指定这个选项后，sed会以二进制方式打开文件，并且会认为行以换行符结束。

--follow-symlinks  
这个选项只存在于支持软链接的平台上，并且需要指定-i。如果命令行中的文件是软链接，sed会编辑软链接的目标文件。如果不指定--follow-symlinks，那么链接会中断，目标文件不会被修改。
例如文件example中内容为：
	
	this is an example

通过如下命令创建软链接example_link
	
	ln -s example example_link

	-rw-r--r-- 1 root root 19 1月   9 05:23 example
	lrwxrwxrwx 1 root root  7 1月   9 05:24 example_link -> example

执行如下命令：
	
	sed -i --follow-symlinks 'p' example_link

example文件内容变为：
	
	this is an example
	this is an example

如果不指定--follow-symlinks，执行如下命令：

	sed -i 'p' example_link

example文件内容不变，example_link成为文件

    -rw-r--r-- 1 root root 19 1月   9 05:30 example
    -rw-r--r-- 1 root root 38 1月   9 05:30 example_link

内容为：
	
	this is an example
	this is an example

-r  
--regexp-extended  
使用扩展正则表达式，即egrep所支持的。扩展正则表达式通常使用较少的反斜线，因此更清晰。但由于它是GNU扩展所以不够方便。参见扩展正则表达式。

-s  
--separate  
默认情况下，sed会把命令行里的多个文件当成一个单独的长数据流来处理。使用这个GNU扩展选项，sed会当成单独的文件来处理，需要注意的是：指定的地址范围不能跨文件（例如：‘/abc/,/def/’）；行号是相对于每个文件的首行；$是指每个文件的尾行；R命令指定的文件会倒回到行首。

-u  
--unbuffered  
输入和输出都会尽可能小的缓冲。（比如输入来自“tail -f”，你希望尽可能快的看到结果）。

如果命令行中没指定-e、-f、--expression或者--file，那么第一个非可选参数会被认为是待执行的script。

如果任何参数跟在上述几个选项之后，这个参数会被认为是输入文件。如果文件名为‘-’或者空，sed都会处理标准输入。

##3 sed Programs

一段sed程序由一条或多条sed命令组成，这些命令可通过选项-e、-f、--expression或者--file指定，如果没指定上述任何选项，则通过第一个非选项参数指定。本文中，上述选项后面跟着的一条或多条命令称为“脚本”。

脚本中的命令以英文半角冒号(:)或者换行符(ASCII 10)分隔。某些命令由于自身语法原因后面不能跟冒号，需要改成换行符或者把该条命令放在整个脚本的最后。Commands can also be preceded with optional non-significant whitespace characters.

每一条sed命令包含一个可选的地址或地址段，后面是一个字符长度的地址名称及其他该命令指定的代码。

###3.1 How sed Works

sed维护两个数据缓存区：pattern space及辅助用的hold space，初始都为空。

对每行输入都按以下周期执行：首先，从输入流中读取一行，去掉行尾换行符，放入pattern space。然后执行命令；每一条命令都有一个关联地址：一种条件代码，只有当条件校验过命令才会被执行。

当执行完最后一条命令时，pattern space中的内容会被打印到输出流，如果之前去过换行符，此时会重新添加上换行符，如果指定了-n选项，则不会执行打印动作。之后是下一行的执行周期。

除非指定了特殊的命令（例如‘D’），pattern space在两个执行周期间会被清空，hold space则会保留数据。(参见命令‘h’，‘H’，‘x’，‘g’，‘G’)。

###3.2 Selecting lines with sed

sed中的地址可以是以下任意一种形式：

number  
指定输入中待匹配的行号。（需要注意的是，如果不指定-i或-s选项，sed会跨文件进行行号计数）。

first~step  
这是一个GNU扩展，匹配从first行开始的每step行。即存在一个非负数n使得行号=first+(n*step)的行被选中。例如：选择奇数行，1~2;选择从第二行开始的每三行，2~3;选择从第10行开始的每五行，10~5；选则第50行，50~0，一种更令人迷惑的写法。

$  
这个地址匹配最后一个输入文件的最后一行，如果指定了-i或者-s选项，则匹配每个文件的最后一行。

/regexp/  
选中匹配正则表达式regexp的所有行。如果regexp包含‘/’，则需要利用反斜线（\）进行转义。

空正则表达式‘//’重复上一个正则表达式匹配（指定s命令时效果相同）。 Note that modifiers to regular expressions are evaluated when the regular expression is compiled, thus it is invalid to specify them together with the empty regular expression. 

\%regexp%  
‘%’可替换为其他任意单个字符。

和上述正则表达式匹配作用相同，只是允许使用除了‘/’外不同的分隔符。在regexp包含大量‘/’时会非常有用， 避免了对每个‘/’繁琐的转义。如果regexp包含自定义的分隔符，也需要通过反斜线（\）转义。

/regexp/I  
\%regexp%I  
I修饰符是一个GNU扩展，使得正则匹配区分大小写。

/regexp/M  
\%regexp%M  
The M modifier to regular-expression matching is a GNU sed extension which causes ^ and $ to match respectively (in addition to the normal behavior) the empty string after a newline, and the empty string before a newline. 特殊的字符序列（\`和\'）通常用来匹配缓冲区的开始和结束。M含义为多行（multi-line）。

如果没指定address，所有行都会被匹配；如果指定了address，只有符合指定address的行会被匹配。

可以通过以逗号（,）分隔的两个地址指定一个地址段。地址段从第一个地址匹配的行开始匹配，到第二个地址匹配的行结束（包含）。

如果第二个地址是正则表达式，那么会从第一个地址匹配到的下一行开始检测，结果至少会包含两行（除非输入流已结束）。

如果第二个地址是一个数字并且小于等于第一个地址匹配的行号，那么只有一行被匹配到。

GNU sed也支持一些特殊的two-address形式（都是GNU扩展）:

0,/regexp/  
第一个地址为0，sed就会从第一行就行正则匹配。换句话说，0,/regexp/和1,/regexp/相似，不同点是：0,/regexp/这种形式正则会匹配输入的第一行；而1,/regexp/这种形式正则会从第二行开始匹配。

记住这是地址为0唯一可以使用的地方；没有第0行并且如果在其他地址指定地址为0会报一个错误。

