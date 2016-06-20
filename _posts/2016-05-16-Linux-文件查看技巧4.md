---
layout:     post
title:      "linux下文件查看技巧"
subtitle:   " \"4-了解awk的用法\""
date:       2016-05-16 17:00:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
---

# 目录

- <a href="#bj">背景</a>
- <a href="#ckzl">参考资料</a>
- <a href="#aml">awk命令</a>
	- <a href="#qb">起步</a>
	- <a href="#nzbl">内置变量</a>
	- <a href="#zfcpp">字符串的匹配</a>
	- <a href="#cfwj">拆分文件</a>
	- <a href="#tj">统计</a>
	- <a href="#dybg">打印表格</a>
	- <a href="#hjbl">环境变量</a>

# <a name="bj">背景</a>
<p>在工作中我们经常需要查看日志来迅速的定位问题,同时发生故障时我们要经常的统计日志中相关的内容，那么掌握一些常用的日志操作命令就非常必要了。</p>

<p>之前了解了常用的几个统计读取日志的命令以及正则表达式,这节了解一下非常强大的两个命令,一个是sed,一个是awk,这两个命令能够完成绝大部分的关键词的取用和统计。</p>

# <a name="ckzl">参考资料</a>
> http://coolshell.cn/articles/9070.html#more-9070

> 大量的例子都是从这里搬运的，推荐大家看酷壳，个人觉得帮助到我很多。

# <a name="aml">awk命令</a>

### <a name="qb">起步</a>

功能：管道流编辑器,它可以读指定的文件或者标准输入流,通过一系列的命名可以去修改读到的输入,然后再将修改后的输入进行输出。它可以对匹配的每行都进行处理，比如同时加上什么东西，替换什么东西，减少什么东西。<br>
直接看例子：<br>
测试文本：
<pre><code>
→ cat test
1-2-3-4-5
2-2-3-4-5
3-2-3-4-5
4-2-3-4-5
5-2-3-4-5
6-2-3-4-5
7-2-3-4-5
</code></pre>
取出部分字段
<pre><code>
→ awk -F "-" '{print $1 " lalala " $4}' test 
1 lalala 4
2 lalala 4
3 lalala 4
4 lalala 4
5 lalala 4
6 lalala 4
7 lalala 4
</code></pre>
前面加上判断条件<br>
**其中的“==”为比较运算符。其他比较运算符：!=, >, <, >=, <=**
<pre><code>
→ awk -F "-" '$1==2 || $1==3 {print $1 " lalala " $4}' test 
2 lalala 4
3 lalala 4
</code></pre>
再来一个吧
<pre><code>
→ awk -F "-" '$1>3 {print $1 " lalala " $4}' test 
4 lalala 4
5 lalala 4
6 lalala 4
7 lalala 4
</code></pre>

### <a name="nzbl">内置变量</a>

我们可以来了解一下awk的一些内建变量,然后直接来使用看看:
<pre><code>
$0	当前记录（这个变量中存放着整个行的内容）
$1~$n	当前记录的第n个字段，字段间由FS分隔
FS	输入字段分隔符 默认是空格或Tab
NF	当前记录中的字段个数，就是有多少列
NR	已经读出的记录数，就是行号，从1开始，如果有多个文件话，这个值也是不断累加中。
FNR	当前记录数，与NR不同的是，这个值会是各个文件自己的行号
RS	输入的记录分隔符， 默认为换行符
OFS	输出字段分隔符， 默认也是空格
ORS	输出的记录分隔符，默认为换行符
FILENAME	当前输入文件的名字
</code></pre>
试试NR，把第一行打出来
<pre><code>
→ awk -F "-" '$1>3 || NR==1 {print $1 " lalala " $4}' test
1 lalala 4
4 lalala 4
5 lalala 4
6 lalala 4
7 lalala 4
</code></pre>
指定分隔符
<pre><code>
→ awk 'BEGIN{FS=":"} {print $1 " lalala " $4}' test
等价于
→ awk -F "-" '{print $1 " lalala " $4}' test
</code></pre>
如果你要指定多个分隔符
<pre><code>
→ awk -F '[;:]'
</code></pre>
OFS的使用
<pre><code>
→ awk -F "-" '$1>3 || NR==1 {print $1,$2,$3}' OFS="输出记录分隔符" test 
1输出记录分隔符2输出记录分隔符3
4输出记录分隔符2输出记录分隔符3
5输出记录分隔符2输出记录分隔符3
6输出记录分隔符2输出记录分隔符3
7输出记录分隔符2输出记录分隔符3
</code></pre>

### <a name="zfcpp">字符串匹配</a>
匹配第一列有1的这一行
<pre><code>
→ awk -F "-" '$1 ~ /1/ ' test 
1-2-3-4-5
</code></pre>
匹配第一列没有1的这一行
<pre><code>
→ awk -F "-" '$1 !~ /1/ ' test 
2-2-3-4-5
3-2-3-4-5
4-2-3-4-5
5-2-3-4-5
6-2-3-4-5
7-2-3-4-5
</code></pre>
匹配这一行有1的数据
<pre><code>
→ awk '/1/' test  
1-2-3-4-5
</code></pre>
匹配这一行没有1的数据
<pre><code>
→ awk '!/1/' test 
2-2-3-4-5
3-2-3-4-5
4-2-3-4-5
5-2-3-4-5
6-2-3-4-5
7-2-3-4-5
</code></pre>

### <a name="cfwj">拆分文件</a>
匹配第一列有1的这一行
<pre><code>
→ awk -F"-" '{print > $1}' test

→ ll
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 1
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 2
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 3
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 4
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 5
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 6
-rw-r--r--   1 lianghe  staff         10  6 16 12:20 7

→ cat 1
1-2-3-4-5

→ cat 2
2-2-3-4-5

→ cat 3
3-2-3-4-5
</code></pre>
你也可以把指定的列输出到文件：
<pre><code>
→ awk -F"-" '{print $1,$2 > $1}' test
</code></pre>

### <a name="tj">统计</a>
下面的命令计算所有的C文件，CPP文件和H文件的文件大小总和。
<pre><code>
$ ls -l  *.cpp *.c *.h | awk '{sum+=$5} END {print sum}'
2511401
</code></pre>

### <a name="dybg">打印表格</a>

在上面我们可以看到一个END关键字。END的意思是“处理完所有的行的标识”，即然说到了END就有必要介绍一下BEGIN，这两个关键字意味着执行前和执行后的意思，语法如下：

BEGIN{ 这里面放的是执行前的语句 }
END {这里面放的是处理完所有的行后要执行的语句 }
{这里面放的是处理每一行时要执行的语句}
为了说清楚这个事，我们来看看下面的示例：

假设有这么一个文件（学生成绩表）：
<pre><code>
$ cat score.txt
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62
</code></pre>

我们的awk脚本如下（我没有写有命令行上是因为命令行上不易读，另外也在介绍另一种用法）：
<pre><code>
$ cat cal.awk
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
 
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}
</code></pre>

我们来看一下执行结果：（也可以这样运行 ./cal.awk score.txt）
<pre><code>
$ awk -f cal.awk score.txt
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
</code></pre>

### <a name="hjbl">环境变量</a>
即然说到了脚本，我们来看看怎么和环境变量交互：（使用-v参数和ENVIRON，使用ENVIRON的环境变量需要export）
<pre><code>
$ x=5
 
$ y=10
$ export y
 
$ echo $x $y
5 10
 
$ awk -v val=$x '{print $1, $2, $3, $4+val, $5+ENVIRON["y"]}' OFS="\t" score.txt
Marry   2143    78      89      87
Jack    2321    66      83      55
Tom     2122    48      82      81
Mike    2537    87      102     105
Bob     2415    40      62      72
</code></pre>







 