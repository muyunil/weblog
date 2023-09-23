---
title: Shell-Script
date: 2021-05-03 15:25:16.0
updated: 2022-07-31 02:06:37.181
url: /archives/shell-script
categories: 
- 技术
tags: 
- Linux
- Shell
---



虽然使用linux很长时间来，但对于shell脚本都是随用随查随写，并没有一个成体系的知识，所以从头过一边并记录。

<!--more-->

# 简介

shell的种类有很多，bash，zsh，csh ... 还有我所用的fish，个有各的特色。
i> shell就是一个“壳”，用shell来与linux内核进行交互。

大多数使用场景下都是一行命令一行命令交给shell执行的。

shell脚本就是shell自身提供的语法与各个程序的命令进行配合可以完成复杂功能的实现，可以反复执行，达到省力。

虽然有很多种shell，但通常时shell脚本代码都是基于bash的，因为大部分linux但预装shell就是bash，非常通用，并且绝大部分情况下是可以胜任工作的。

# 环境变量

bash拥有的环境变量（配置文件）：

```bash
/etc/profile  #1 全局公有配置，用户登录时读取
/etc/bashrc  #2 全局公有配置，bash执行时读取
~/.profile  #3 
~/.bashrc  #4
~/.bash_profile  #5
~/.bash_login  #6 登录时读取
~/.bash_logout  #7 退出时读取
```

根据使用bash的不同方式，bash读取的配置文件顺序也不同

进入图像界面：1 -> 3
图形界面打开终端：2 -> 6

文本界面登陆2 -> 1 -> 5

切换用户 `su`
不用参数：2 -> 6
使用参数：2 -> 1 -> 5

# 快捷键

`Ctrl + a` 光标移到到开始
`Ctrl + e` 光标移动到结尾
`Ctrl + l` 清屏，同clear效果
`Ctrl + u` 剪切光标前
`Ctrl + k` 剪切光标后
`Ctrl + y` 粘贴u/k剪切到的字符
`Ctrl + r` 搜索历史命令
`Ctrl + c` 终止当前命令
`Ctrl + d` 退出终端
`Ctrl + z` 暂停，并且放入后台。
`Ctrl + s` 暂停输出
`Ctrl + q` 恢复输出

# 后台运行管理。

可理解为后台作业
可以将一条命令放到后台去运行。
`sl &` 或者 sl 执行时使用`Ctrl + z`暂停，放到后台。
`jobs` 查看后台运行列表
`fg %1` 将jobs显示id为1的后台进程放到前台。
`bg %1` 将后台id为1的进程运行（解除暂停）
`kill %1` 杀死进程
i> 当前终端的作业在另一个终端不看见。

---

# 重定向

|文件描述符|类型|设备|
|--|--|--|
|0|标准输入|键盘|
|1|标准输出|屏幕|
|2|错误输出|屏幕|
|--|--|--|

输出重定向 `>` 覆盖，`>>` 追加
命令输出重定向

```shell
ls / > log #将标准输出重定向到log文件并覆盖（可不存在，自动创建）
ls / 2> err #将错误输出重定向到err文件并覆盖源内容

ls / &> log 
ls / > log 2>&1 #将标准输出和错误输出重定向到log文件并覆盖。
```

输入重定向 将标准输入（键盘）改为其他输入

将命令的标准输入重定向为一个文件：
```bash
cmd < file
```
将开始标记`END`和结束标记`END`之间的内容作为输入：
```bash
cmd << END 
hsjska sjsjsj ksjs 
END 
```

# 管道

将管道符左边命令的输出做为右边的输入。

```bash
man ls | less
```

`tee`可以将管道内容同时输出在文件和屏幕上。

```shell
ls | tee txt
```

---

# 命令排序

`&&` 符号前命令执行成功才会执行符号后命令
`||` 符号前命令执行不成功才会执行符号后命令
`;` 单纯的分割命令，互不影响，都会执行。

---

# 通配符

`*` 匹配任意一个或多个字符
`?` 匹配任意一个字符
`[test]` 匹配test中任意一个字符
`[!test]` 匹配除test中任意一个字符
`[a-z]` 匹配任意（a到z）一个小写字母
`a{string1,string2,...}b`
a与b之间匹配字符串，这个字符串只能是string1或者string2，如：astring1b，或astring2b。

---

# 脚本调试

`bash -n back.sh`
不执行脚本，只进行语法检查

`bash -x back.sh`
效果与-v类似，是打印一行执行一行。

`bash -v back.sh`
执行脚本前先打印脚本内容（全部）

---

# echo

echo 显示文字
参数： -n , -e
-n 为不换行（默认换行）
-e 对特殊字符加以处理，而不是当成字符处理。

-n：

```bash
echo -n "Login:";read
```

-e

```bash
echo -e "\a\a\a" 
#发出警告声。
echo -e "\b"
#删除前一个字符
echo -e "\c"
#最后不加上换行符。
echo -e "\f"
#换行但光标停留原来的位置
echo -e "\n"
#换行且光标移至行首
echo -e "\r"
#光标移至行首，但不换行
echo -e "\t"
#插入tab
echo -e "\v"
#与\f相同
echo -e "\abc\def"
#插入字符
echo -e "\nnn"
#插入nnn (八进制)所代表的ASCII字符
```

倒计时示列：

```bash
#!/bin/bash
for time in `seq 9 -1 0`; do
    echo -n -e "\b$time"
    sleep 1
done
echo
```

---

# read

默认接受键盘的输入，回车符代表结束。
read 选项
-p  打印信息 （类似基础echo功能）
-t  限定时间 （超过时间退出）
-s  不回显（输出字符不显示）
-n  限制输入字符个数

```bash
#!/bin/bash
read -p "Login: " acc

echo -n  "Passwd: "
read -s -t50 -n6 pw

echo acc, pw
```

打印登录，将键盘输入存到acc变量，在打印密码，将键盘输入存到pw变量但设置为输入不显示，输入时间50s，输入支付限制为6位。
最后打印输入的数据。

---

# 基础数据类型

## 变量

变量赋值`=`两边不可以有空格，可以使用unset 取消临时变量,对于全局变量只是临时取消。

- 全局变量
  全局变量需要加上export

```bash
export NAME="Muyu"
```

- 永久变量
  定义永久变量需要需要写在配置文件中
  `/etc/profile`,`/etc/bashrc`

```bash
##/etc/bashrc
export NAME="Muyu"
```

- 特殊变量
  有一些预先定义的特殊变量如：

`$1`,`$2`,`$3`, ... `$9`
可以获取执行sh时 文件名后的 值

`$$`可以获取当前进程pid

`$?`可以获取上一个命令的返回值 0为成功

`$!` 可以获取上一个后台进程的pid

```bash
#!/bin/bash

echo $0，$1, $2, $3, $4, $5, $9

echo $@  # 显示所有位置变量
echo $*   #显示所有位置变量，同上。

sh t.sh 1 2 3 4 5 6 7 8 9
```

其中 $0 就是脚本文件名，$1 就是文件名后第一个，以此类推。

---

## 数组

变量的有序集合体，以下标索引来区分。

赋值语法：

```bash
ARRAY=("abc" "ok" "adc")
ARRAY[3]="def"
#可以继续追加新的值在末尾。
ARRAY[3]=8274
#可以修改对于下标的值
```

- 特殊赋值语法

```bash
ARRAY=(`cat /etc/passwd`)
#将/etc/passwd文件内容按行读入数组赋值
ARRAY2=(`ls ./`)
将查询结果读取赋值
```

读取语法：
`echo ${ARRAY[2]}`
读取指定下标元素

`echo ${ARRAY[@]}`
`echo ${ARRAY[*]}`
读取数组所有元素，`@`,`*`等同。

`echo ${#ARRAY[*]}`
统计元素个数

`echo ${!ARRAY[*]}`
获取数组的索引

`echo ${ARRAY[*]:1}`
从下标1开始。

`echo ${!ARRAY[*]:1:3}`
从下标一开始，读取三个元素。

删除元素
`unset array[index]`
清空整个数组
`unset array`
内容替换
${array[@]/an/AN}

## 关联数组

可以使用自定义的索引（下标）
类似于map 字典
命令行使用declare -A 可以显示已声明的关联数组。

关联数组使用时需要声明。

```bash
declare -A ass_array1
declare -A ass_array2

ass_array1[name]="Muyu"
ass_array1[age]=88

echo ${ass_array1[name]}
echo ${ass_array1[*]}

#可以写在一条赋值语句里。

ass_array2=([name]="Muyu" [age]=99)

echo ${ass_array2[name]}
echo ${ass_array2[*]}
```

----------

# 流程控制

## 运算比较符

各种适用于判断的运算符

### 数学比较运算（整形）

```bash
-eq    #等于(equal)
-gt    #大于(greater than)
-lt    #小于(less than)

-ge    #大于或等于
-le    #小于或等于
-ne    #不等于
```

浮点数比较小技巧

```bash
#!/bin/bash

NUM1=`echo "1.5*10" | bc | cut -d "." -f1`
NUM2=$((2*10))

test NUM1 -eq $NUM2;echo $?
```

`bash -x eq.sh`：

```
++ cut -d . -f1
++ bc
++ echo '1.5*10'
+NUM1=15
+NUM2=20
+test 15 -eq 20
+echo 1
```

### 字符串比较运算

`==` 等于比较
`!=`  不等于比较

`-z`  判断字符串为空
`-n`  判断字符串不为空
判断依据是字符串长度是否等于或大于0

### 文件类型判断

```bash
-d  #检查是否存在且为目录
-f  #检查是否存在且为文件
-e  #检查是否存在（不管是文件还是目录）

-r  #检查文件是否存在且可读
-w  #检查文件是否存在且可写
-x  #检查文件是否存在且可执行
-s  #检查文件是否存在且不为空

-O  #检查文件是否存在并且被当前用户拥有
-G  #检查文件是否存在并且默认组为当前用户组。

file1 -nt file2  #检查文件1是否比文件2新
file1 -ot file2  #检查文件1是否比文件2旧
file1 -ef file2  #检查文件1是否与文件2是同一个文件（硬链接效果）
```

### 逻辑运算

`&&`  逻辑与
`||`  逻辑或
`!`  逻辑非

与或运算需要两个或以上的条件。

---

## if

语句格式:

```bash
if [ ! -d /tmp/adc ]
then
    mkdir -v tmp/adc
    echo "create /tmp/adc ok"
else
    echo "file exists"
fi
```

检查目录是否存在，如果不存在就创建，如果存在就打印：file exists 说明目录已经存在。

- 扩展用法
  if 可以使用双圆括号来进行数值运算

```bash
#!/bin/bash
if ((100%3+1 > 1))
then
    echo "yes"
else
    echo "no"
```

还可以使用双[]进行字符串判断

```bash
for str in aaa bbb ccc ddd fff
do
    if [[ $str == ?a? ]];then
        echo "$str ok"

    elif [[ $str  == b* ]];then
        echo "$str ok"

    else
        echo $str no
    fi
done
```

多分枝判断可以使用`elif` 也可以嵌套`if`
执行结果：

```
aaa ok
bbb ok
ccc no
ddd no
fff no
```

---

## for

bash 的for循环有两种写法
`for i in` 写法，循环次数是由给i多少个值了确定的。

```bash
for i in `seq 1  9` ;do
    echo $i
done
```

seq是一个数数的命令，`seq 1 9`意为从1数到9，也就是给i赋值9次，所以可见的这个for循环需要循环9次，每次都会打印i的值
i> seq也可以倒着数数：seq 9 -1 1 意为从9数到1 步进是-1 也就是每次减1

---

- 第二种：
  这种写法与一些高级语言较为相似如c语法

```bash
#单变量
for (( i=0;i<10;i++ ));do
    echo $i
done

echo "--------"
#多变量
for (( i=0,j=10;i<10;i++,j-- ));do
    echo -e  "$i\t$j"
done
```

执行结果：

```
0
1
2
3
4
5
6
7
8
9
--------
0       10
1       9
2       8
3       7
4       6
5       5
6       4
7       3
8       2
9       1
```

i> for ((;;)) 这种写法为无条件循环，必须要在循环内部设置退出循环判断否则及为死循环。

### 例子-主机存活脚本

```bash
#!/bin/bash

for ((;;));do
    ping -c1 $1 &> /dev/null
    if [ $? -eq 0 ];then
        echo -e "`date +"%F_%H%M%S"`:$1 \033[32m UP \033[0m"
    else
        echo -e "`date +"%F_%H%M%S"`:$1 \033[31m Down \033[0m"
    fi

    #控制执行频率
    sleep 60                                      #在此睡眠60秒
done
```

## continue

使用continue跳过循环。

```bash
for i in `seq 1 9` ;do
    if [ $i -eq 4 ];then
        continue
    fi
    echo "$i"
done

echo "---------"

for (( j=1;j<10;j++)) ;do
    if [ $j -eq 4 ];then
        continue
    fi
    echo "$j"
done
```

两种for循环方法都使用continue跳过了4这个值，没有echo显示它。

---

## break

使用break直接退出循环

```bash
i=0
for ((;;));do
    if ((i > 10));then
        echo "end"
        break  #跳出循环
    else
        echo "$i"
        (( i++ ))
    fi
done
```

我们使用了for ((;;)) 这种死循环写法，这种写法必须要控制循环的执行，我们使用了if判断当i大于10时打印end表示结束执行break跳出循环。在不大于10时打印i的值，每次循环i++（计数器），我们的for循环依赖此计数器才能正常退出。

i> 如果是多层for，默认只跳出当前循环，可以使用break n 跨级跳出n循环。

```bash
#!/bin/bash

for (( i=1;i<10;i++ ));do  #1层循环 对mybreak来说是它的第三层
    echo $i
    echo "for1..."

    for ((;;));do  # 2层循环 对mybreak来说是2
        echo "for2..."

        for ((;;));do  # 3层循环 对mybreak来说是1
            echo "for3..."
            break 3 # 1为默认值跳出当前循环，2为跳出它上面的第二层循环，3为跳出它上面的第三层循环
            #暂时叫它为mybreak # 3为跳出对它来说的第三层循环
        done

        break  
    done
    sleep 1

done
```

执行结果:
1
for1...
for2...
for3...
可以看到直接跳出了最外面的循环。

---

## while

while循环

```bash
read -p "login name: " USER
read -s -p  "login passwd: " PW

while [ $USER != "root" ] || [ $PW != "123" ];do
    echo -e "\nNo!"
    read -p "login name: " USER
    read -s -p  "login passwd: " PW
done

echo -e "\nHello!"
```

如果输入的用户及密码不对就一直循环，直到全部正确然后循环退出，打印hello

### 99乘法表

使用for和while循环打印乘法表

```bash
#!/bin/bash

echo "----for-for------"
for (( i=1;i<10;i++ ));do
    for (( j=1;j<=i;j++ ));do
        echo -n  " $j*$i=" $(( $j*$i ))
    done
    echo 
done

echo "----while-for------"
wi=1
while [ $wi -lt 10 ];do
    for (( wj=1;wj<=wi;wj++ ));do
        echo -n " $wj*$wi=" $(( $wj*$wi ))
    done
    echo
    (( wi++ ))
done

echo "----while-while------"
wwi=1
while [ $wwi -lt 10 ];do
    wwj=1
    while [ $wwj -le $wwi ];do
        echo -n " $wwj*$wwi=" $(( $wwj*$wwi ))
        if [ $wwj -eq 9 ] ;then
            (( wwj=1 ))
            break
        else 
            (( wwj++ ))
        fi

    done
    echo
    (( wwi++ ))
done
```

----------

## until
until循环和while循环除了判断条件相反之外其他完全一致。
while循环是条件为真循环，until循环是条件为假循环。
```bash
#!/bin/bash
num=1
while [ $num -lt 10 ];do
    echo $num
    num=$(($num+1))

    until [ $num -lt 10 ];do
        echo $num
        if [ $num -eq 20 ];then
            break
        fi
        num=$(($num+1))
    done
done
```
while循环打印1-9 until循环打印10-20。

----------

## case
case语法类似于一种预案，根据值执行不同的代码
```bash
#!/bin/bash

case $1 in
a)
    echo "A"
;;

b)
    echo "B"
;;

c)
    echo "C"
;;

*)
    echo "USAGE: $0 a|b|c"
;;

esac
```

----------

## 函数
bash里面也有函数，并且为我们提供了一个函数文件，里面定义了很多函数供我们使用，可以导入使用
```bash
#如果有就导入使用，`•`的作用相当于`source`
if [ -f /etc/init.d/function ];then
    . /etc/init.d/function
else
    echo "not found file /etc/init.d/function"
    exit 2
fi

```
函数的声明有两种格式都可以使用
```bash
#!/bin/bash
function hello {
    echo -n "Hello"
}

world () {
    echo "World"
}

hello
world

hello; world
```
执行结果：
`HelloWorld`
`HelloWorld`

### 函数返回值

> return 返回

```bash
function hello {
    if [ $a -gt $b ];then
        return 0
        #可以省略0不写，默认返回0
    else
        return 1
    fi
}
a=10
b=2

hello $a $b
echo $?
```

!> return 在shell大部分用于返回函数执行状态，因为它只能返回0~255的值

> echo 返回

```bash
function get_user {
    users=$(cat /etc/passwd | awk -F ":" '{print $1}')
    echo $users
}

users_list=$(get_user)

for i in $users_list;do
    echo $i
done
```