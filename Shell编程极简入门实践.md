<h2>Shell编程极简入门实践</h2>

<h6>By StevenSLXie (Last updated: 31, Dec, 2014)</h6>

<h4>0. 写在前面</h4>

程序员多多少少都会和命令行打交道，一些常用的命令，比如`cd`、`ls`、`ping`等等，使用起来可能问题不大。但大多数人对Shell编程的了解程度，可能仅止于那几个最常用的命令。当需要更复杂的命令或者需要写一个脚本来进行批处理的时候，很多人可能感到头疼。而Unix/Linux复杂多变而又不太直观的命令常常让初学者望而却步。比如像这样的语句`echo "123 abc" | sed 's/[0-9][0-9]*/& &/'`的语句，再比如那些以`-`开头的参数（`-l -n -a`等等），即使从网上找到了可能符合自己要求的代码，也往往因为看不懂而无法修改化为己用。

这个极简教程，或者说笔记，针对的是正是这部分读者。具体地说，通过学习这篇文档，你将获得以下技能：
<ul>
<li>熟练掌握Unix/Linux下的最常用命令及其最常见用法；</li>
<li>能够编写脚本，对文件进行批处理，对一些网络任务进行自动化等等；</li>
<li>避免写脚本过程中的最常见错误；</li>
<li>(Hopefully)可以借此消除对命令行的恐惧；</li>
</ul>

这个教程的特点是：
<ul>
<li>不求全面，只求实用。只覆盖最常用的命令及其用法；</li>
<li>以大量例子为导向；</li>
<li>一边阅读一边动手写例程的话，大约只需要1.5-2.5小时的时间；</li>
</ul>

这篇文档假定你是在Linux/Unix环境下，比如Ubuntu, 比如Mac OS X。同时假定你至少了解一门其它的编程语言。这个教程的代码均在Mac OS下测试过，由于各种shell的标准差别很小，（有充足的理由相信）在别的平台应该也都能顺利运行。

<h4>1. Hello World</h4>

首先打开你用得最顺手的文本编辑器，在第一、二行分别打入

```
#!/bin/bash
echo "Hello, World!"
```

保存文件，文件可保存在你喜欢的文件夹，扩展名选择`.sh`，比如这样的文件：`tutorial.sh`。

接着，打开命令行工具Terminal，首先将工作目录改到你保存文件的文件夹，比如如果你将`tutorial.sh`放在`/Users/Steven/code`,则在命令行里执行以下操作

```
cd /Users/Steven/code
```

`cd`是change directory的意思，因为我们要执行`tutorial.sh`这个脚本，所以我们要先将工作目录转到这个脚本对应的文件夹下面。接着，在命令行继续输入

```
chmod +x tutorial.sh
```

`chmod`是change mode，`+x`的意思是将`tutorial.sh`变为一个可执行的文件。

接下来，我们就可以运行`tutorial.sh`这个脚本了。在命令行里打入

```
./tutorial.sh
```

如无意外，你将看到命令行里返回`Hello, World!`这个字符串！请注意，文件名前面的`./`是必不可少的，它告诉系统，就在当前的目录查找一个叫tutorial.sh的文件，如果没有`./`，系统会只在系统目录里面查找（准确来说是PATH变量定义的路径）。

我们回头来看看`tutorial.sh`里面的程序，目前它只有两行：

```
#!/bin/bash
echo "Hello, World!"
```

`#!`是一个标记，它告诉系统该去哪里去寻找能“解释”`tutorial.sh`的解释器。
`echo`是回响的意思，意思是说`echo`后面的那一串东西，都会在命令行显示出来。它和其它语言的`print`是类似的。

就这样，我们完成了一个最简单的bash scripting的程序编写。这里面有几点需要注意：

<ul>
<li>执行脚本文件前，先要cd到文件所在的目录；</li>
<li>执行脚本文件前，先要chmod +x tutorial.sh将其变为可执行程序；</li>
<li>脚本文件的第一行，记得写上#!/bin/bash。</li>
</ul>

<h4>2. 整数和字符串</h4>

变量的定义很简单，按照以下格式就可以了：

```
NAME=var
```

比如定义一个字符串：

```
NAME='Steven'
```

比如定义一个整型变量：

```
NUM=3
```

这里有几点要注意，一是变量的名字，虽然大小写不限，但按照惯例一般采用全大写的方式。第二点特别重要，让我们做一个小实验来说明一下。打开刚才的那个`tutorial.sh`文件，将之前的内容清空，并打入

```
#!/bin/bash
NAME = 'Steven'
echo $NAME
```

如果你是直接复制以上的代码段，那么命令行应该会出现以下错误信息：

```
./tutorial.sh: line 2: NAME: command not found
```

出现这个错误，是因为：定义变量时，`=`的前面和后面，都是不能有空格的！这一点可能和其它语言不一样，但请务必注意。因为出现这类错误时，报错信息定位的栏数（line 2），是指向你引用变量的那一段代码，而不是定义变量的那一行，因此debug起来可能不是那么直观。

于是，我们把代码改为：

```
#!/bin/bash
NAME='Steven'
echo $NAME
```

命令行将显示`Steven`。

从上面的例子我们也可以看到，当你定义了一个变量，要引用它时，要在前面加上`$`。变量名两边可以加上花括号，比如这样`${NAME}`。花括号不是必须的，但最好养成加上的习惯。因为在某些情况下，不加花括号可能引发歧义<a href="https://github.com/qinjx/30min_guides/blob/master/shell.md">[1]</a>:

```
#!/bin/bash
NAME='Steven'
echo "My name is ${NAME}SLXie."
```

可想而知，如果没有花括号，NAME和后面的SLXie就无法区分了。

对于字符串变量，既可以用单括号，比如`'Steven'`，也可以用双括号`"Steven"`，它们之间有微妙而繁杂的区别，在这里我们先记住一点，当字符串里面包含对某个变量的引用时，必须用双括号。比如上面的`"My name is ${NAME}SLXie."`。请试着将它的双括号改为单括号，并观察它的输出结果。

单括号会将被引用字符串中的几乎所有特殊字符当作普通字符处理，比如上面的`$`，单括号时只把它当作一个普通的美元符号输出。

再看一个例子，试着在脚本分别输入这两行，并观察它们的输出。
```
echo "Here is $50."
echo 'Here is $50.'
```

在这里插播一句，看这个教程的时候，最好是看到那里，就动手写到哪里。写的时候，不要直接复制粘贴，而是试着手打代码到编辑器里面。有时候代码里有一些微小而琐碎的东西，一定要自己打一遍才能记得牢。比如，一开始可能容易将`#!/bin/bash`打成`#/bin/bash`，或者`#!bin/bash`。

花括号`{}`也可以用来对字符串进行某些操作。比如下面这个例子：

```
#!/bin/bash
USERNAME='StevenSLXie'
echo "My name is ${USERNAME}. People usually call me ${USERNAME:0:6}."
```

它会输出：

`My name is StevenSLXie. People usually call me Steven.`

这时候，`${USERNAME:0:6}`的作用是取字符串的一部分。第一个数字0是指截取的起始部分，则第二个数字6则是指截取的长度。再比如`${#USERNAME}`则是获取字符串的长度。更多的字符串用法，我们将在后面的正则表达式哪一节看到。

<h4>3. 数组</h4>
数组可以这样简单粗暴地定义：

```
NAMES[0]='Steven'
NAMES[1]='Peter'
NAMES[2]='David'
```

当数组体量太大时，这样定义未免麻烦，在bash里面（01-Jan-2015更新，感谢@mzhboy提醒，其它的一些shell，如zsh、ash不能用这种方法。）我们也可以用一行声明的方式来定义：

```
declare -a NAMES=('Steven' 'Peter' 'David')
```

用`declare -a`来声明，后面一次性定义所有数组元素。请注意，在这里，整个数组用小括号括起来，而每个数组元素之间，是用空格来隔开的，而不是逗号或者其它。

访问数组的其中一个元素，和其它语言没什么不同。在你声明好数组之后，就可以访问数组元素了：

```
echo ${NAMES[0]}
echo ${NAMES[2]}
echo ${NAMES[*]}
```

`${NAMES[*]}`或者`${NAMES[@]}`表示访问所有元素。而`${#NAMES[*]}`则返回数组长度。请注意它和`${#NAMES}`的区别，后者是返回`NAMES`里面第一个元素的长度，相当于`${#NAMES[0]}`。

一个数组声明并定义后，我们仍可以二次定义它，比如下面的代码是在原来的数组基础上再添加一个人名。

```
declare -a NAMES=('Steven' 'Peter' 'David')
echo ${#NAMES[*]}
NAMES=("${NAMES[*]}" 'Nancy')
echo ${NAMES[*]}
```
命令行将返回：
```
3
Steven Peter David Nancy
```

<h4>4. 运算符</h4>
<h5>4.1 算术运算符</h5>
Shell编程里的算术运算符和大多数编程语言很类似，主要是这些`+ - * / %`等。如果你试着在命令行里执行运算的话，比如输入以下算式：
```
2 + 2
```

会得到：

```
-bash: 2: command not found
```

这条错误信息。这是因为命令行的逻辑是它会把一行命令的第一个词当作是命令，在系统中寻找与之匹配的执行语句，因为在这里它会认为2是一个命令，而显然它不可能找到这个命令。要想执行运算，我们在命令行里打入

```
expr 2 + 2
```

输出结果是`4`。`expr`是一个常用命令，evaluate an expression的意思。注意，这里数字和运算符之间，必须有一个空格。不然的话，如果你输入，
```
expr 2+2
```

则会输出
```
2+2
```

这种情况下，`expr`会把后面的`2+2`当成一个字符串，而evaluate一个字符串的结果，自然就是它本身了。算术运算当然也可以用变量，比如：
```
VAR=5
expr 2 + ${VAR}
```

其它的算术运算符大体类似，但有一个要特别注意，如果你进行乘法运算，比如:

```
expr 2 * 15
```

会输出：

```
expr: syntax error
```

这是因为`*`是一个特殊的字符（后面还会介绍到），而当要表达其原来的意思是，我们需要在它前面加上下划线`\`。就像这样：
```
expr 2 \* 15
```

(31-Dec-2014更新。感谢@mzhboy补充。)此外还有`let`也是挺常用的。`let`计算跟在它后面的表达式。和`let`极其类似的还有`(())`。看下面的例子：

```
A=3
B=4

let add=$A+$B
((sub=$B-$A))

echo $add
echo $sub
```

除此之外还有一个强大的计算命令`bc`，这里就暂不展开了。有兴趣的同学可以看<a href="http://www.basicallytech.com/blog/?/archives/23-command-line-calculations-using-bc.html">这里</a>。

<h5>4.2 关系判断运算符</h5>
Shell提供了丰富的关系判断运算符，先来看一个例子，在`tutorial.sh`加入以下代码：

```
A=10
B=15

if [ $A -eq $B ]
then
    echo 'True'
else
    echo 'False'
fi
```

这是一个`if`条件判别语句(后面再细讲)。`-eq`判断运算符左右两边是否相等，如果是，则返回`True`，不然就返回`False`。关系判断运算符的基本格式是`[ VAR1 OPERATOR VAR2 ]`，用一个中括号括起来，这里有一点细节要注意，中括号和变量之间，需要有空格隔开。所以像`[$A -eq $B]`是会报错的。（不如自己写段代码试一试？）

完整的关系判断运算符文档可以看这里：<a href="http://www.tutorialspoint.com/unix/unix-basic-operators.htm">Unix Basic Operator</a>

<h5>4.3 逻辑运算符</h5>

Shell编程里，与逻辑和或逻辑分别是用`-a`和`-o`来表示的(02-Jan-2015更新。感谢@星光 修正)。
看看下面这个例子：

```
A=10
B=15

if [ $A -lt 8 -a $B -gt 8 ]
then
    echo 'True'
else
    echo 'False'
fi
```

这是判断变量A是否小于8且变量B是否大于B。

(31-Dec-2014更新。感谢@mzhboy指出。)除了`-o`和`-a`，`&&`和`||`也可以来表示与和或，但在表达上稍稍有一点不一样：每一个条件就要用一个中括号括起来。比如：

```
if [ $A -lt 8 ] && [ $B -gt 8 ]
then
    echo 'True'
else
    echo 'False'
fi
```

完整的关系判断运算符文档可以看这里：<a href="http://www.tutorialspoint.com/unix/unix-basic-operators.htm">Unix Basic Operator</a>

<h4>5. if条件判断、while循环、for循环</h4>
`if | while | for`语句和其它主流语言很相似，因此用起来应该不是大问题。值得注意的可能是以下几个小点：
<ul>
<li>if语句的格式是if...then...elif...fi。注意，这里用fi来标记一个条件判断的结束。嗯，感觉是一种很调皮的设定。</li>
<li>所有的while和for语句，其执行的语句都始于do，终于done。</li>
<li>for的格式是for VAR in ARRAY，是Python的样式，和经典的C for循环可能稍有不同。</li>
</ul>

下面我们用一个例子来感受一下这三种语句。

```
echo 'Shall we begin the demo?(y/n)'
read ANS

A=0
declare -a B=(0)

if [ $ANS = 'y' ]
then
    echo 'Output the results of the while loop'
    while [ $A -lt 10 ]
    do
        echo $A
        A=`expr $A + 1`
        B=(${B[*]} $A)
    done
else
    echo 'Not ready for the demo yet.'
fi

echo 'Output the results of the for loop.'
for I in ${B[*]}
    do
        echo $I
    done
```

第二行的`read`在这个文档是第一次出现，它的功能是从Terminal里读取一行，并将这一行内容赋给`read`后面的那个变量。在这个例子里，我们询问用户是否要开始`demo`，当得到肯定回答后，用户的回答（假定是`y`），将被赋给`ANS`这个变量。

接着，我们用一个`if`判断语句，如果用户回答是`y`，则执行`while`循环。`while`循环的内容比较简单，在每一次循环里，先打印出`A`的值，接着将`A`的值加一，然后将每一个得到的新的`A`放进数组`B`里面。当`A`大于等于10的时候，退出循环。

最后是一个`for`循环，`for`的功能是将数组`B`里面的元素依次打印出来。

这里要补充一点，在对`A`加一的时候，我们使用的是```A=`expr $A + 1` ```，如果你试图使用类似`A=$A + 1`这样的赋值，将会报错。这是因为如前所述，每一个算术运算，都需要用`expr`来算出它的结果。那`expr`两边的``` ` ` ```又是怎么回事呢？这里的``` ` ` ```其实是命令输出的引用，我们知道```expr $A + 1```本身是一个命令语句，而当我们要引用这个语句的执行结果的时候，就要加上``` ` ` ```了。举一个比较容易理解的例子吧：如果你在命令行输入`date`，就会输出系统的当前时间，这时候`date`是一个命令，但如果你要将`date`这个命令的执行结果赋给一个变量`D`，就要```D=`date` ```。同样的，如果我们要将`5 + 25`的结果赋给一个变量`SUM`，则需要```SUM=`expr 5 + 25` ```。

``` ` ` ```在键盘上的位置是在`Tab`上面，`1`的左边。

还有一点要注意，当我们对一个已经定义过的变量进行重新赋值的时候，是不需要加`$`的，上面的例子```A=`expr $A + 1` ```和```B=(${B[*]} $A)```里，等式左边的变量都不需要加`$`。

(31-Dec-2014更新。感谢@mzhboy指出。)此外还有一种`for`的情况需要注意。我们看下面这段代码：

```
A="cat dog"

for I in $A
    do
        echo $I
    done
    
for I in "$A"
    do
        echo $I
    done
```

第一个`echo`的输出是：
```
cat
dog
```

第二个：

```
cat dog
```

也就是说当我们用双引号将`$A`括起来时，cat和dog中间的空格自动被忽视了，整个A被当做一个字符串看待。这一点要注意，通常这个情况下（因为我们用for来枚举），所以一般用第一种。



<h4>6. 函数</h4>
Shell脚本里当然可以定义函数。比如这样的：

```
hello(){
    echo "Hello World!"
}

```
函数可以直接调用，比如下面这个脚本：

```
#!/bin/bash
hello(){
    echo "Hello World!"
}

hello
```

注意，但函数没有参数时，调用只需要写上函数名，而不是`hello()`之类的。

Shell函数的特殊之处，在于它参数传递的形式，具体地说，参数并不是像别的语言一样，写在括号里面，而是类似下面这个例子：

```
intro(){
    echo "There are $1 people here. They are $2, $3."
}
```

Shell用`$1` `$2`这样的特殊记号来标记函数参数。而`$0`则是函数名，`$n`表示第`n`个参数。调用上面的`intro`函数，则是这个样子：
```
intro 2 'Steven' 'David'
```

命令行的输出则是：

```
There are 2 people here. They are Steven, David.
```

参数不是写在括号里，而是在函数名之后依次排列，并以空格隔开。

函数也可以有返回值，比如：
```
hello(){
    A=`expr 5 \* 10`
    return $A
}
```

返回变量`A`的值。请注意，这里获取返回值的方式和其它语言可能不太一样。并不是`A=hello`，而是：
```
hello
RET=$?
```

`$?`是一个特殊字符，它保留上一个命令的执行结果。因此，当我们要获取`hello`函数的返回值时，就在调用该函数后，紧接着将`$?`赋给存储返回值的变量`ret`。在这里，这两句命令是必须紧挨着的，如果中间还有其它语句，则`$?`会返回最近一次的命令执行结果，而不一定是`hello`的返回值。


<h4>7. sed和正则表达式</h4>
正则表达式是一种特殊的字符串，用来描述一串具有某种共同特征的字符串。在进行批处理的时候，正则表达式有着异常强大的应用。

sed则是一个流编辑器(stream editor)，它读入一个输出，并通过加工处理，输出经处理后的 文件/字符串 输出。下面我们通过一系列例子，来掌握sed的基本应用。

首先我们要来新建一些txt文件供sed处理。在命令行输入：

```
mkdir files
cd files
touch test-1.txt
touch test-2.txt
touch test-3.txt
cd ../
```

第一行`mkdir`是在当前目录下新建一个叫`files`的文件夹。然后我们`cd`到新的文件夹里，在里面用`touch`新建三个`txt`文档。接着，通过`cd ../`回到上级目录，也就是我们的`tutorial.sh`所在的目录里。

然后，分别打开那三个`txt`文件，将以下几行字符串拷贝到文件里。
```
This is a file with several lines
some of which are blank lines
for example, the line that follows is blank

But this line has several characters.

And this marks the end of the file.
```

接着，打开我们的`tutorial.sh`，将之前内容清空，只留下第一行的`#!/usr/bash`。在里面输入：
```
cd files
FILE=test-1.txt

sed -i.tmp "/^$/d" $FILE
```

保存后运行`./tutorial.sh`。然后打开`test-1.txt`，如无意外的话，你会看到文档变成这个样子：
```
This is a file with several lines
some of which are blank lines
for example, the line that follows is blank
But this line has several characters.
And this marks the end of the file.
```

所有的空行被删除了。我们来看看`sed -i.tmp "/^$/d" $FILE`这句命令。其中`"/^$/d"`描述了sed作用于什么模式(pattern)，以及操作(action)，它的基本结构是这样的"/pattern/action"。其中`pattern`里面是一个正则表达式，表面了我们要寻找什么样的字符串(pattern match)，找到之后，则是对这个字符串进行操作(action)。在我们这个例子里，我们的正则表达式是`"^$"`，其中`^`是标记一个字符串的开始，而`$`是标记一个字符串的结束。在这里，开始和结束连在一起，表示我们要寻找的是一个空字符串，而对它的操作是`d`，`delete`的意思。整个句子合起来就是，`sed`会打开`test-1.txt`，逐行扫描，找到空行，删除掉空行。

而`sed`前面的 `-i`则是`in place`的意思，就是说我们这个操作是直接对目标文件下手的。这样的修改通常是比较危险的（万一改错了呢），所以一个安全的办法是在`-i`后面加上`-i.tmp`，意思是每次改动，我们都会保留一个后缀为`.tmp`的备份文件。于是你打开文件夹的话，会看到多出了一个`test-1.txt.tmp`。如果你确认你的改动没有问题之后，就可以把这个备份文件删掉了。备份文件的扩展名可以随意，因为当你需要它的时候总是需要把它改回`txt`的。但最好扩展名不要和文件夹的已有文件重复。

接着你可以试试将上面的`sed`语句改为：
```
sed -e "/^$/d" test-2.txt
```

这时你会发现改动后的结果，在命令行显示出来了，而原文件并不会改动。

再来看下面的语句：

```
sed -e "1,3d" test-2.txt
```

这个语句的意思，就是删除`test-2.txt`中的第一到第三行。观察输出可以看到，前三行被删除了。类似地，`sed -e "2,4!d" test-2.txt`则是指，除了2-4行，其他都删掉。`sed -e "d" test-2.txt` 则是删除文档里面的全部内容。请注意，为了避免频繁改动文档，以上几个命令都是用的`-e`，改动是体现在命令行的输出的。如果要直接对文档进行改动，请用`-i.tmp`，或者分开写为`-i '.tmp'`。在Mac OS X的Bash Shell里面，似乎提供一个备份文件的扩展名是必须的，而在Linux平台则似乎是可选的。

更为常见的`sed`的应用，是用它来进行替换。看看下面的例子：
```
sed -e 's/But/but/' test-2.txt
```

命令行的输出：

```
This is file with several lines
some of which are blank lines
for example, the line that follows is blank

but this line has several characters.

And this marks the end of the file.
```

可以观察到，首字母大写的`But`被替换成`but`。替换的基本格式是`'s/old/new/'`，`old`和`new`分别代表替换前和替换后，而且一般用单引号，除非需要shell变量展开到sed命令中才用双引号（31-Dec-2014，感谢@mzhboy补充）。我们再来多看几个例子，首先将`test-2.txt`文档的内容改为：

```
1.This is file with several lines
2.some of which are blank lines
 for example, the line that follows is blank

7.But this line has several characters.

 And this marks the end of the file.
```

第三行和最后一行前面有一个空格。接着我们在`tutorial.sh`里面加入以下命令：
```
sed -i '.tmp' 's/^[ 1-3]//' test-2.txt
```

你会看到以下输出：

```
.This is file with several lines
.some of which are blank lines
for example, the line that follows is blank

7.But this line has several characters.

And this marks the end of the file.
```

在这个例子里，正则表达式`^[ 1-3]`的`^`表示字符串的开始，而中括号表示匹配任意一个在中括号里面的字符。我们的中括号里面有空格以及数字1-3，而之后`*`表示零到任意多个任意字符。于是，`sed`根据正则表达式的要求去逐行扫描，找出“以空格或者数字1-3开头的行”。

找到之后干什么呢？这就要看第二个`/`后面的内容了，而我们发现第二个`/`和第三个`/`之间并没有内容。这是说，找到了符合要求的这个字符串，就将其替换为空字符。于是你可以看到以上的输出了。第一行第二行的数字被移除，第三行和最后一行的空格被移除，而第五行的数字7则不受影响。

保留新的`test-2.txt`，我们继续执行以下命令：

```
sed -i '.tmp' 's/[!.]$/;/' test-2.txt
```

这个命令则是找出以`!` `.`结尾的行，并一律改为以分号结尾。

我们继续在原有的文档操作，输入以下命令：

```
sed -i '.tmp' 's/^[. 1-9]*//;s/[;.!]$/ ENDING/' test-2.txt

```

这是两个替换命令一起来。首先，我们找出以`.` 或者数字1-9,或者空格开头的行，然后将其删掉。请注意，这里有一个`*`，它代表了任意多个（包括0）前一个字符的重复。比如如果一个行是以三个空格开头的，则加上`*`可以加之一并铲除，如果不加的话就只会匹配并删除第一个空格。两个合并命令之间以空格隔开。第二个命令是找出以分号句号或者感叹号结尾的行，代之以` ENDING`。

输出结果应该是：

```
This is file with several lines
some of which are blank lines
for example, the line that follows is blank

But this line has several characters ENDING

And this marks the end of the file ENDING
```

回头看这个命令，`sed -i '.tmp' 's/^[. 1-9]*//;s/[;.!]$/ ENDING/' test-2.txt`。两个小时之前，如果你看到这么复杂的命令行的时候，很有可能因为看起来过于复杂而崩溃掉吧。但现在，你也能读、写这样复杂的命令了。恭喜！

让我们继续来看看`sed`的一些其它应用。

```
sed -i '.tmp' 's/^some.*//' test-2.txt
```
删除以`some`开头的行。其中`.`可以指代任意字符。

```
sed -i '.tmp' 's/....$//' test-2.txt
```

删除每行末尾的四个字符。

```
sed -i '.tmp' 's/But/but/g' test-2.txt
```

这个看起来有点熟悉对不对？如果你比较之前的语句，会发现多了一个`g`。在这里，如果不加`g`则只会替换每行的第一个`But`，而加`g`则会替换所有的`But`。如果你只是要替换某一个位置的`But`，比如第三个，则可以`sed -i '.tmp' 's/But/but/3' test-2.txt`。

除了删除和替换，`sed`还支持插入(insert/append)新的字符串。

比如下面的例子：

```
sed -i '.tmp' '3 a\
just some random text' test-2.txt
```

它的功能是往`test-2.txt`的第三行后面添加`just some random text`这一新行。注意`a\`之后要另起一行，这里`a`是`append`的意思。如果`a\`改为`i\`，则是在前面加新行。`i`是`insert`的意思。

另外一个例子：

```
sed -i '.tmp' '$ a\
just some random text' test-2.txt
```

在文件末尾处加上新行。

当然我们也可以用正则表达式要判别，比如：

```
sed -i '.tmp' '/text/ i\
INSERT THIS BEFORE EVERY LINE CONTAINING TEXT' test-2.txt
```

在每一行包含`text`的行前面加入`INSERT THIS BEFORE EVERY LINE CONTAINING TEXT`。

更多的`sed`用法，可以参考：http://www.grymoire.com/Unix/Sed.html#uh-1 。

<h4>9. grep</h4>
grep相当于Unix/Linux命令行的Google，可以快速地找出包含某个字符串的文件。让我们先将当前目录移到`files`子文件夹，并显示该目录下的所有文件名。

```
cd files
ls
```

如果你按照这个教程一路走下来，现在这个文件夹里面应该有`test-1.txt test-2.txt test-3.txt`几个文件，以及可能的备份文件。如果不是也不要紧。接下来我们执行一个grep，来找出`test-1.txt`里面包含`'file'`的那些行。

```
grep "file" test-1.txt
```

正常的话会输出：
```
This is file with several lines
And this marks the end of the file.
```

这两行包含着`file`。

`grep`可以同时搜索多个文件，比如这样：

```
grep "file" test-1.txt test-2.txt
```

输出则是：

```
test-1.txt:This is file with several lines
test-1.txt:And this marks the end of the file.
test-2.txt:This is file with several li
test-2.txt:And this marks the end of the file END
```

格式是`文件名:字符串`。当然，罗列所有的文件名，有时候很不方便。这时候可以用上模糊搜索。比如这样：

```
grep "file" test-*.txt
```

有时候你只想知道哪些文件包含了某个字符串，而对那一行的具体内容是什么并不重要，那么可以这样：

```
grep -l "file" test-*.txt
```

系统会打印出包含`file`的文件名。有的时候你想多知道一点：不想打印出整行字符串，但想知道每个文件有几行包含`file`，那么可以：

```
grep -c "file" test-*.txt
```

而有时候你想知道的非常多，不仅是文件名，出现了几行，而且具体的行数也要知道，那么可以：

```
grep -n "file" test-*.txt
```

有的时候你想知道输出相反的结果：那些不包含`file`的行，那么可以：

```
grep -vn "file" test-*.txt
```

`grep`当然也支持正则表达式。要知道，`grep`的全称就是`global regular expression print`。所以“你问我支持不支持，他的名字叫全局正则表达式打印器，怎么能不支持？”（请忽略一个蛤丝的老梗~）。

我们来看几个例子：

```
grep "END$" test-*.txt
```

输出结尾为`END`的那些行。
```
grep "file\|But" test-*.txt
```

输出含有`file`或者`But`的行。注意`|`前面需要用`\`。

如果我们要搜索包含`characters.`的行，正确的命令是这样的：

```
grep 'characters\.' test-*.txt
```

同时试一试`grep 'characters.' test-*.txt`，看看结果有什么不同。


<h4>10. 写一个脚本吧</h4>

前面九节基本上覆盖了Shell编程最基本的内容，这一节我们来动手写一个脚本，一方面是把我们之前学到的东西复习一下，串联起来。另一方面，是将之前没有覆盖到的几个常用命令介绍一下。

我们的任务是在当前目录下，新建一个脚本`final-script.sh`。然后在脚本里新建一个文件夹`final`，在新文件夹里批量新建十个`.txt`文件，命名规则为`final-test-n.txt`。每个文件的文档内容`"This is a test file...!!"`。

接着，将十个文件的文件名从`final-test-n.txt`改为`Final-test-n.txt`。

然后，删除文件内容里面的标点符号。

接着，将文件内容全部变成大写。

将改动后的文件，拷贝一份到任意一个新的文件夹。

请注意，这当中会出现一些我们没学到的知识，如果你发现现有的知识不足以解决问题的话，请Google之。

下面是我写的脚本，为了便于理解，我将各个命令都分开写了。


```
#!/bin/bash
mkdir final
cd final

declare -a NAME
NAME=(1 2 3 4 5 6 7 8 9 10)


# 创建新文件。
for I in ${NAME[*]}
    do
        touch final-test-${I}.txt
    done

# 往文档写入。这里使用的是echo，通过`>`改变其默认输出。不妨思考一下如果用sed来实现会有什么问题？
for F in final-test-*.txt
    do
        echo 'This is a test file...!!' > $F
    done

# 文件名首字母大写。注意echo和sed的连用，以及我们引用命令的一种新方法$(command)。还有mv这个新命令。
for F in final-test-*.txt
    do
        NEW=$(echo "$F" | sed -e 's/^./F/')
        mv "$F" "$NEW"
    done

# 删除标点。
for F in *.txt
    do
        sed -i.tmp 's/...!!$//' $F
    done

# 大写。注意tr的使用。
for F in *.txt
    do
        tr '[:lower:]' '[:upper:]' < $F > FILE2
        mv FILE2 $F
    done

cd ../
mkdir repo

cd final

# 复制文件。
for F in *.txt
    do
        cp $F ../repo/$F
    done
```

当中新出现的东西，就留待读者自己解决啦。如果你也完成了这个任务，不妨将你的代码发给我。相信你一定能写出比我更简洁的代码！


<h4>11. 后记</h4>

Happy Coding!


<h4>参考资源</h4>
<ul>
<li><a href="http://www.thegeekstuff.com/2010/06/bash-array-tutorial/">1. Bash Array Tutorial</a></li>
<li><a href="https://github.com/qinjx/30min_guides/blob/master/shell.md">2. Shell脚本编程三十分钟入门</a></li>
<li><a href="http://www.tutorialspoint.com/unix/index.htm">3. Unix Tutorial Point</a></li>
<li><a href="http://linux.about.com/">4. about.com</a></li>
<li><a href="http://www.grymoire.com/Unix/Sed.html#uh-1">5. Sed - An Introduction and Tutorial by Bruce Barnett</a></li>
<li><a href="http://www.thegeekstuff.com">6. The Geek Stuff</a></li>
<li><a href="http://www.panix.com/~elflord/unix/grep.html">7. Grep Tutorials</a></li>
<li><a href="http://www.uccs.edu/~ahitchco/grep/">8. Drew's Grep Tutorial</a></li>
</ul>

<h4>勘误和交流</h4>
匆忙写下的一个笔记，出错简直是一定的。如果您发现了任何错误或者有关于本文的任何建议，麻烦发邮件给我（stevenslxie at gmail.com）或者在GitHub上直接交流，不胜感激。

<h4>转载声明</h4>
如果你喜欢这篇文章，可以随意转载。但请
<ul>
<li>标明原作者StevenSLXie;</li>
<li>标明原链接(https://github.com/StevenSLXie/Tutorials-for-Web-Developers/blob/master/Shell%E7%BC%96%E7%A8%8B%E6%9E%81%E7%AE%80%E5%85%A5%E9%97%A8%E5%AE%9E%E8%B7%B5.md);</li>
<li>在可能的情况下请保持文本显示的美观。比如，请不要直接一键复制到博客之类，因为代码的显示效果可能非常糟糕;</li>
<li>请将这个转载声明包含进来；</li>
</ul>







