---
title: Nim
date: 2021-05-03 15:29:25.0
updated: 2022-04-01 16:02:01.738
url: /archives/nim
categories: 
- 技术
tags: 
- Nim
- 编程语言
---



记录Nim的学习过程的知识，简单的记录下来，方便以后查询。
<!--more-->

# 简介
高效，富有表现力，优雅。
Nim是一种静态类型的编译系统编程语言。它结合了来自成熟语言（如Python，Ada和Modula）的成功概念。

----------

# 入门小程序
一段类`HelloWorld`代码。
```nim
echo("Hello World!!! What's your name? ")
var name: string = readLine(stdin)
echo("Hi, ", name, "!")
```
```shell
nim c -r hello.nim
```
执行结果：
```nim
Hello World!!! What's your name?
muyu  #命令行输入
Hi, muyu!
```

----------

# 转义字符
nim使用`\`为转义字符，与c等语言相同,`\n`,`\t` 等等
可以使用`r""`标识字符串为原始字符串，原始字符串内的`\`不会被转义
```nim
var path = r"C:\program files\nim"
```
# 大段文本
在开发程序的时候，往往需要写大段文本，比如HTML的模版。
Nim使用成对的三个分号包住大段文本。
文本里的反斜杠也不会被当作转义符。

```nim
var str = """Nim是一种静态类型的编译系统编程语言。
它结合了来自成熟语言（如Python，Ada和Modula）的成功概念。
"""
```
----------

# 声明变量/常量
关键字：`var`,`const`,`let`

## `var`
```nim
var name: string  #标准式
name2 = "muyu"
var name2:string = "muyu" 

var name3 = "muyu"  #值推理式

var x,y,z:int

var 
    chara = "a"
    p = 3
```

## const
const定义的变量不能被修改
因为编译器会把所有const变量换成他所对应的值，所以变量对应的值是表达式的话，在编译时一定要能对表达式求值才行
```nim
const 
    x = 4
    y = x+5
    z = "allen"
```
## let
用let定义的变量，赋值后也不能被修改，但用let定义的变量，可以在运行期赋值。
```nim
const input = readLine(stdin)
# Error: 运行期的值不能赋给const变量
let input = readLine(stdin)
# works：运行期的值可以赋给let定义的变量
```

----------

# if_elif_else
```nim
echo("Hello World!!! What's your name? ")
let name: string = readLine(stdin)
if name == "":
    echo "空名字？"
elif name == "name":
    echo "name = name?"
else:
    echo("Hi, ", name, "!")
```

----------

# when
when与if非常相似
```nim
when system.hostOS == "windows":
  echo("running on Windows!")
elif system.hostOS == "linux":
  echo("running on Linux!")
elif system.hostOS == "macosx":
  echo("running on Mac OS X!")
else:
  echo("unknown operating system")
```
不同点如下：
when关键词的每个分支所用的表达式，都必须能在编译期取值
when关键词内的每个分支并不开辟新的作用域
当第一个分支的条件为true的时候，编译器会对第一个分支的代码做词法分析，编译器不对其他分支进行分析
当你编写系统级代码的时候可以用when关键字来代替C语言中`#ifdef`
因为上面讲到的第三条特性，所以经常会写when false这样的代码，以避免编译器在编译期分析的效果（主要是因为很多东西只有在运行期才能确定）

----------

# case_of_else
```nim
echo("Hello World!!! What's your name? ")
let name: string = readLine(stdin)
case name
of "":
    echo "空名字？"
of "name":
    echo "name = name?"
else:
    echo("Hi, ", name, "!")
```

----------

# while
名字为空时再次循环
```nim
echo("What's your name? ")
var name = readLine(stdin)
while name == "":
  echo("Please tell me your name: ")
  name = readLine(stdin)
```

----------

# for
```nim
echo("Counting to ten: ")
for i in countup(1, 10):
  echo(i)
```
可以简写为：
```nim
echo("Counting to ten: ")
for i in 1..10:
  echo(i)
```

----------

# 作用域
关键字：`block` `break`
- block
不管我们用for还是用while，都会导致开辟一个新的作用域
这里提到的作用域，是内建的作用域，我们可以使用block关键字显示创建一个额作用域
```nim
block myblock:  #myblock名称可选
  var x = "hi" 
echo(x)
#这里没有缩进，已经跳出myblock作用域，所以这是错误的
```
- break
break可以迫使程序执行跳出当前作用域
像while、for和block关键字声明的作用域，它都能跳出
```nim
block myblock:
  echo("entering block")
  while true:
    echo("looping")
    break # 跳出while循环
  echo("现在仍然在myblock作用域中")
```
注意只是跳出当前的作用域，要想一下子多跳几层作用域，那么就要声明block关键字了
```nim
block myblock2:
  echo("entering block")
  while true:
    echo("looping")
    break myblock2 # 跳出while循环，而且也跳出myblock2作用域
```

----------

# continue
continue可以让本次循环跳过，立即执行下一次循环。
```nim
while true:
  let x = readLine(stdin)
  if x == "": continue
  echo(x)
```

----------

# 方法（func）
Nim给方法叫做Procedures。
列子：
```nim
proc yes(str: string): bool =
  echo(str, " (y/n)")
  while true:
    case readLine(stdin)
      of "y", "Y", "yes", "Yes": return true
      of "n", "N", "no", "No": return false
      else: echo("Please be clear: yes or no")

if yes("是否确定？"):
  echo("确定成功！")
else:
  echo("取消操作成功！")
```
定义了一个叫yes的函数，接受一个字符串参数名为str，根据stdin输入返回一个bool值。

----------

# result
只要一个方法被定义成有返回值的方法， 那么Nim会给你默认创建一个result变量，
你可以在方法中不用声明就使用这个result变量，不管你方法中有没有retun指令，到方法执行结束，都会返回这个result变量
```nim
proc add(a,b:int): int =
  result = a + b

echo add(5,5)
```
如果你自行var了一个result 那么这个效果将消失，就是一个普通变量。

----------

# 参数
为了执行效率，在方法体内部不能改变参数的值
如果你只是想在方法体内部使用与参数同名的变量，你可以在方法体内部，使用var重新定义一个同名参数，隐藏掉本身的方法参数，就可以当一个普通变量使用了。

如果你一定要在方法体内部改变参数的值，（这是十分常见的，因为这样做可以为调用者提供信息）那么你可以在声明参数的时候，使用var关键字，就像下面这样：
```nim
proc test(a, b: var int) =
  echo a,"  ",b #1, 1
  a = 10
  b = 10                                    
  echo a,"  ",b #10,10

var
  a = 1
  b = 1
test(a,b)
echo a,"  ",b #10, 10
```

----------

# 丢弃返回值
在主流编程语言中，一个方法存在返回值，如果我们只想调用这个方法，而不使用他的返回值，那我们不理会他的返回值就是了
在Nim中，默认是不允许的，你必须显示的丢弃掉他的返回值才行
```nim
discard yes("是否确定？")
```
使用diacard显示丢弃返回值。

也可以在声明方法的时候，就显示的声明，这个方法的返回值是可以丢弃的，就像下面这样：
```nim
proc p(x, y: int): int {.discardable.} =
  return x + y

p(3, 4) # 值可丢弃
```

----------

# 具名实参
有的时候一个方法包含很多参数，使用这个方法的人只记得参数的名字，但不记得参数的顺序了，那么你可以使用具名参数来解决这个问题，就像下面这样：
```nim
proc test(x, y, z: int; ok: bool): int =
   ...

var t = test(ok=true,x=1,y=2,z=3)
```
这样显示参数名给值就可以无视顺序。

----------

# 参数默认值
我们可以给proc中的参数设置默认值，在使用时如果没提供参数，则使用默认值执行。
```nim
proc add(a=0,b=0)=
  echo a+b

add()  #0
add(1)  #1
add(1,1)  #2
```

------

过程（方法）的重载
所有的高级语言几乎都具有方法重载，Nim也有重载。
```nim
proc toString(x: int): string =
  result =
    if x < 0: "negative"
    elif x > 0: "positive"
    else: "zero"

proc toString(x: bool): string =
  result =
    if x: "yep"
    else: "nope"

assert toString(13) == "positive" # 调用 toString(x: int) 方法

assert toString(true) == "yep"    
# 调用 the toString(x: bool) 方法
```
Nim并没有使用特别复杂的算法和机制，而是基于一种简单的技术实现的。
有歧义的方法重载编译时不会通过。

----------

# 操作符
在Nim的类库中大量的使用了方法的重载，主要的原因就是操作符的原理其实就是方法重载。
Nim的语法允许你使用中缀表示法（a+ b）、前缀表示法（+a）。
中缀表示法就是一个接收两个参数的方法重载。
前缀表示法就是一个接收一个参数的方法重载。
不允许使用后缀表示法，因为当你想表示a++b的时候，你到底是想表示(a)+(+b)呢，还是想表示（a+）+b的意思呢？

在Nim中因为禁用了后缀表示法，所以a++b的意思是(a)+(+b)

除了一些内置的关键词操作符（and or not）之外，操作符一般都是这些字符：+ - * \ / < > = @ $ ~ & % ! ? ^ . |
用户可以自定义自己的操作符，只要你自己顾及可读性就好
可以用两个单撇号来定义一个操作符，就像下面这样：
```nim
proc `$` (x: myDataType): string = ...
# 现在你可以使用$来操作你的myDataType类型的数据：$ myDataTypeValue
```
操作符就是方法的重载，当然也可以像调用方法一样使用操作符
```nim
if `==`( `+`(3, 4), 7): echo("True")
```
首先计算 `+`(3,4) 结果为7，此时式子为 `==`(7,7) 结果为相等 所以if执行echo 显示True。
