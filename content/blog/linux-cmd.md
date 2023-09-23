---
title: Linux-cmd
date: 2021-05-03 15:28:09.0
updated: 2022-07-30 15:52:58.539
url: /archives/linux-cmd
categories: 
- 技术
tags: 
- Linux
- Shell
---



记录学习

<!--more-->

# 文本三剑客

> Linux下处理文本数据非常强大的三个程序

## grep
grep是一个过滤器，通过指定条件过滤文本数据
egrep和greo -E 相等，都是开启正则表达式的支持。

语法：
`grep [option] [pattern] [file, ...]`
`command | grep [...] ...`

常用参数
`-o` 只显示匹配到的字符本身。
`-v` 不显示匹配行（反向匹配）
`-i` 匹配忽略大小写
`-n` 显示行号
`-r` 递归目录搜索
`-E` 支持正则表达式
`-F` 不按正则表达式匹配，按字符串字面意思匹配

了解参数
`-c` 只输出匹配行的数量，不输出匹配行
`-w` 匹配整词 不匹配包含的（精确匹配，前后有空格或tab）
`-x` 匹配整行 （同上）
`-l` 只列出匹配文件名，不显示匹配行数据（与-r搭配）

----------

## sed
sed是一个行（流）编辑器，非交互式的对文件进行增删改查操作。

使用者只能在命令行输入编辑命令，指定文件，然后再屏幕上查看输出。它和文本编辑器有本质区别

文本编辑器:编辑对象为文本文件
行编辑器:编辑对象为文本中的行

i> 前者一次处理一个文本整体，而后者是处理文本中的行。

sed数据处理流程

`文本行` -> `内存(缓存)` -> `屏幕`
i> 数据在内存中被处理，然后默认输出到屏幕
也就是说默认情况下sed是修改的只是读到内存的文本行数据，不会修改源文本行

### sed命令
语法：`sed [options] '{command}[flage]' [filename]`

### command  内部命令

`/abcd/` 匹配模式，在斜杠中写匹配字符
`sed '/root/i\addTest' filename`
将addTest插入到有root字符的行前。

- a

`a`  在匹配后面插入：
`sed 'aaddTest' filename`
`sed 'a\addTest' filename`
将addTest字符串追加到file文件每行的后面。
这个两种写法都对，sed认为参数的第一个是命令，可以在a参数后面加\转义，一便区分。

- i

`i`  在匹配前插入
`sed 'i\addTest' filename`
将addTest字符串插入到file文件每行的前面。

- p

`p`  打印
`sed 'p' filename`
打印file文件，但每一行会打印两次，因为加p会打印文件内容，但默认会打印缓存的行，所以每一行会打印两边。

- d

`d`  删除
`sed '3d' filename`
删除file文件的第三行
`sed '3,5d' filename`
删除file文件的第三行到第五行
`sed '/root/d' filename`
删除file文件的有root的一行

- s

`s`  查找替换
`sed 's/lisi/zhangsan/' filename`
将file文件行的lisi替换为zhangsan
`sed '/user/s/lisi/zhangsan/' filename`
将file文件行有user字段的lisi替换为zhangsan

- c

`c`  更改
`sed 'c\addTest' filename`
将file文件每行改为addTest
`sed '3c\addTest' filename`
将file文件第三行改为addTest
`sed '2,5c\addTest' filename`
!> 注意2，5在c参数里与其他参数有区别，它会将2到5行视为一行，也就是将2到5行全部删除然后只插入一条addTest。

- y

`y`  转换  N D P
`sed 'y/abcd/ABCD' filename`
将file文件行的a,b,c,d 转换为对于的A,B,C,D
!> 注意！前面的第一个字符会被后面的一个字符代替，并不是一定要有关系也可以用其他任意字符代替前面的字符，只需要位置顺序相同。

### flags

`数字` ：表示新文本的替换模式
`sed 's/lisi/zhangsan/' filename`
我们在使用查找替换内部命令s的时候，如果这一行有两个lisi这条命令只会替换第一个lisi，后面的lisi并不会替换
`sed 's/lisi/zhangsan/2' filename`
此时就只会处理第二个lisi，而不会处理第一个或者其他位置的lisi

`g` ：表示用新文本替换现有外部的全部实列
`sed 's/lisi/zhangsan/g' filename`
此时就会处理替换行内所有的lisi

`p` ：表示打印内容
`sed '3s/lisi/zhangsan/p' filename`
将第三行替换的同时在打印一遍

`w` ：将替换结果写入到文件
`sed 's/lisi/zhangsan/w tmpfile' filename`
将替换的行写入到tmpfile文件保存，源文件不变




### options

`-n` 抑制输出，只打印我要求打印的
`sed -n '3s/lisi/zhangsan/p' filename`
只会打印我要求的（p参数）的行，其他行不打印

`-e script` 
将脚本中指定的命令添加到处理输入时执行的命令中,多条件，一行中要有多个操作
`sed -e 's/tmp/Tmp/;s/fish/bash/' filename`
以`;`区分多条命令。

`-f script`  将文件中指定的命令添加到处理时执行的命令当中。
在cmd文件中添加命令，一行一个，使用是指定即可执行
cmdfile内容：
```bash
3s/root/ooo/
/root/s/abc/bca/
```
使用命令：
`sed -f cmdfile filename`
即可使用cmd文件内的命令

`-i`  编辑文件内容（修改源文件）
`sed -i 's/root/ooo/g' filename` 
此时会修改源文件，并且不会打印到屏幕上

`-i.xxx` 先备份文件以.xxx结尾在编辑文件内容（修改源文件）
`sed -i.bak 's/root/ooo/g' filename` 
此时会先备份一份源文件文件名末尾是.bak,然后再修改源文件，并且不会打印到屏幕上。
i> 推荐使用 -i.xx 以防万一改错了还能恢复。

`-r`  使用扩展的正则表达式
`sed -n -r '/^(root)(.*)(fish)$/p' /etc/passwd`
在匹配模式中使用正则表达式，打印以root开头中间除回车以外的任意字符(该字符不出现或出现多个 以fish结尾的行
内容：
`root:x:0:0::/root:/bin/fish`


`!`  取反，跟在模式条件后面（与shell有所区别）

### 命令总结

增: `a`（行后），`i`（行前）
`i`（将后面指定文件的内容追加到匹配的行后）
`w`（将结果另存文件）

删：`d`

改（部分）：
`s/pattern/string/`（查找pat替换为string（默认只配一个））
`s/patern/string/g`（g表示行内全部匹配）
`s/patern/string/2`（2表示同行之匹配2个）
`s/patern/string/ig`（ig表示匹配时忽略大小写）

查：`p`（将结果打印）

### 小技巧
统计一个文件有多少行:
`sed -n '$=' filename`

打印file内容时加上行号
`sed '=' filename`

----------

## awk

> 简介
awk其实也是一种行编辑器，可以把数据截取，处理(运算)，输出，功能十分强大。

awk的工作方式是读取数据将每一行数据视为一条记录（record）每条记录以字段分割符分成若干字段，然后输出各个字段的值。

> 语法
`awk [options] [BEGIN]{program} [END][file]`
优先级：BEGIN{} -> {program} -> END{}

program和END需要数据源不能脱离数据源执行。
BEGIN是在program之前处理的，它不需要数据源也可以执行如：
`awk 'BEGIN{print "HelloWorld"}'`
没有制定文件名有没有通过管道传输数据，就可以执行。

### 提取列
awk可以用与确认列的参数符

|参数|含义|
--|--
|$0|全部的列|
|$n|指定第n列如:$3（第三列|
|$NF|最后一列|

awk默认`列`在行数据里是以空格分隔的
提取test文件全部列
```shell
awk '{print $0}' test
```
提取/etc/passwd文件每一行的第一列
因为此文件里列是以`:`分隔的所以指定分隔符使用 -F
`awk -F ":" '{print $1}' test`
还可以对列进行处理：
`awk -F ":" '{print "User:" $1, "Shell:" $NF}' test`

如果只想处理要第一行，可以使用NR==n来指定行。
`awk -F ":"  'NR==1{print "User:" $1, "Shell:" $NF}' test`

----------

### awk高级用法
awk就是一门语言，它拥有语言的特性，可以定义变量，数组，可以进行运算，流程控制。

定义变量：
`awk 'NR==1{user=$1,print user}' passwd`
结果：root
还可以使用 -v 定义变量
`awk -v num=10 'BEGIN{print num}'`

定义一个数组：
```shell
awk 'BEGIN{array[0]=root;array[1]=200;print array[0],array[1]}'
#输出
root 200
```
#### awk运算符

赋值运算 `=`
比较运算 `>` `>=` `==` `<` `<=` `!=`
数学运算 `+` `-` `*` `/` `%` `**` `++` `--`
逻辑运算 `&&` `||` 
匹配运算 `~` `!~`

运算例子：
```shell
awk 'BEGIN{print 100>=2 && 1<=100}'
1   #打印
``` 
为真打印1，为假打印0
匹配例子：
```shell
#精确匹配
awk -F ":" '$1 == "root"{print $0}' passwd

精确不匹配
awk -F ":" '$1 != "root"{print $0}' passwd

模糊匹配
awk -F ":" '$1 ~ "ro"{print $0}' passwd

模糊不匹配
awk -F ":" '$1 !~ "ro"{print $0}' passwd
```

#### awk 环境变量

--|--
|FIELDWIDTHS|自定义列的宽度以确认列的分隔|
|FS|输入行中的列分隔符（与-F相同）|
|OFS|输出行中的列分隔符用于显示效果（默认空格）|
|RS|输入行的分隔符(默认行分隔符\n）|
|ORS|输出行的分隔符(默认分隔符\n）|

例子：
```shell
awk'BEGIN{FIELDWIDTHS="1 2 3"}NR==1{print $1,$2,$3}' file
#自定义列宽，以列宽确认列的分隔，第一列长度为1字符，第二列为2字符，第三列为3字符。

awk 'BEGIN{FS=":"}NR==1{print $1,$2,$3}'
#效果和-F ":"相同
awk 'BEGIN{OFS="-"}NR==1{print $1,$2,$3}' file
#$1，$2，$3之间不在是空格分隔而是“-”

awk 'BEGIN{RS=" "}{print $1,$2,$3}' file
#此时输出效果为[第1行  第2行  第3行]
#行与行之间不在进行换行
#都在一行中显示以自定义的空格来区分。

awk 'BEGIN{ORS}{print $0}' file
#现在输出在屏幕上的行末尾不在换行。
```

----------

#### awk 流程控制
awk支持以下流程控制语句
`if` `for` `while` `do ... while` `break`

> if

源式子：
`seq 1 9 > num`
`awk '$1>5{print $0}'` num
会打印$1大于5的行

用if写：
`awk '{if($1>5)print $0}' num`
注意if后面紧跟需要执行的语句，否则不认为和if有关联。

else:
`awk '{if($1<5)print $1*2;else print $1/2}' num`
如果小于5就乘2打印，如果大于5就除2打印

> for

`seq 1 10 > num`
`awk -v num=0 '{num=num+$1}END{print num}' num`
将每一行相加，最后打印结果为55

用for写：
cat num2
10 82 26
39 24 42
39 28 99

`awk '{num=0;for(i=1;i<4;i++)num+=$i;print num}'  num2`
每行的列相加打印：
118
105
166
> while

cat num2
10 82 26
39 24 42
39 28 99

`awk '{num=0;i=1;while(i<4){num+=$i;i++}print num}'  num2`
打印：
118
105
166

> do ... while

cat num2
10 82 26
39 24 42
39 28 99
其实与while一样，只不过这种写法是先执行一次在判断，while是先判断在执行。

`awk '{num=0;i=1;do{num+=$i;i++}while(i<4)print num}'  num2`
打印：
118
105
166

> break

可在循环中搭配if跳出循环
cat num2
10 82 26
39 24 42
39 28 99
在循环中判断当num大于50时跳出循环


- while-break

`awk '{num=0;i=1;while(i<4){num+=$i;i++;if(num>50)break}print num}'  num2`
输出：
92
63
67

- for-break

`awk '{num=0;for(i=1;i<4;i++){num+=$i;if(num>50)break}print num}'  num2`
输出：
92
63
67

----------

### awk字符串处理函数

|函数名|解释|函数返回值|
--|--
|length(str)|计算字符串长度|整数长度值|
|index(str1,str2)|在str1中查找str2的下标|索引，从1计数|
|tolower(str)|转小写|转换后的str|
|toupper(str)|转大写|转换后的str|
|substr(str,m,n)|截取从str的m开始长度n|截取后的字符串|
|split(str,arr,fs)|按fs切割str结果存为arr|切割后的子串的个数|
|match(str,RE)|在str中查找RE位置|位置索引|
|sub(RE,RepStr,str)|将str中符合re的替换为repstr(只替换一个）|替换的个数|
|gsub(RE,RepStr,str)|将str中符合re的全部替换为repstr|替换个数|

----------

### awk小技巧
打印test文本的行数：
`awk 'END{print NR}' test`

打印test文本最后一行内容：
`awk 'END{print $0}' test`

打印test文本最后行的列数：
`awk 'END{print NF}' test`
打印指定行的列数：
`awk 'NR==n{print NF}' num2`


----------

# find 文件查找
find语法 ：`find  ./ -name test.txt`
find常用选项

常用选项：
`-name` 根据文件名搜索
`-iname` 根据文件名搜索（不区分大小写）
`-user` 搜索文件属主（用户名）的所有文件
`-group` 搜索文件属组（组名）的所有文件
`-type` :  
`find . -type f` 文件
`find . -type d` 目录
`find . -type c` 字符设备文件
`find . -type b` 块设备文件
`find . -type l` 链接文件
`find . -type p` 管道文件

`-size` : 
`find . -size -nc` 小于n字节的文件
`find . -size +nM` 大于1M的文件
`find . -size nk` 等于nKb的文件
!> 不建议使用精准匹配大小，有很多问题。

`-mtime` :
`find . -mtime -n` n天以内修改的文件
`find . -mtime +n` n天以外修改的文件
`find . -mtime n` 正好n天修改的文件

`-mmin` 效果同上，不过单位是分钟。

`-mindepth  n` 查找n目录层级
查找2级子目录下的普通的文件。
`find . -mindepth 2 -type f`
!> 当多参数时mindepth必须要在第一个位置

`-maxdepth n` 表示最多搜索到n-1级子目录，不会遍历全部目录。

了解选项：
`-nouser` 搜索没有属主的
`-nogroup` 搜索没有属组的
`-perm 655` 搜索指定权限的

`-prune` 排除文件
搜索文件时排除test_1目录
`find . -path ./test_1 -prune -o -type f`
排除多个目录
`find . -path ./test_1 -prune -o -path ./test_2 -prune -o -type f`

i> -prune语法较为怪异。

`-newer Afile` 查找比Afile新的文件

文件操作
`-print` 将查询结果显示，默认开启。
`-exec` 对结果执行命令:
`find . -name '*.bak' -exec cp {} /tmp/ \;`
`{}`代表搜索到的结果， `\;`为特定格式。
`-ok` 与`-exec`功能相同但每次操作会给用户提示

逻辑运算符
`-a` 与 搜索条件之间的默认使用的运算符
`-o` 或
`-not` `!` 非

----------

# whereis 
查命令二进制文件，命令man文档，命令源码。
参数：
`-b` 只返回二进制文件
`-m` 只返回帮助文件
`-s` 只返回源码文件（无则不返回）

i>无参数使用时默认使用全部参数。

# which
将一个默认参数`-b`只返回唯一二进制文件。