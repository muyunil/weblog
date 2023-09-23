---
title: Shell-Ex
date: 2021-05-03 15:26:46.0
updated: 2022-07-30 15:52:44.855
url: /archives/shellex
categories: 
- 技术
tags: 
- Linux
- Shell
---



学习笔记

<!--more-->

# 变量的替换与测试

`${变量#匹配规则}`
从变量开头匹配，将符合最短的数据删除
`${变量##匹配规则}`
从变量开头匹配，将符合最长的数据删除（贪婪模式）
`${变量%匹配规则}`
从变量尾部匹配，将符合最短的数据删除
`${变量%%匹配规则}`
从变量尾部匹配，将符合最长的数据删除（贪婪模式）
`${变量/旧字符串/新字符串}`
替换变量内旧字符串为新字符串（只替换最先匹配到的
`${变量//旧字符串/新字符串}`
替换变量内旧字符串为新字符串（替换全部匹配到的）

> 变量测试

str为变量，expr为字符串。

|变量声明方式|var没有配置（声明）|var为空字符串|var以声明且非空|
|--|--|--|--|
|v=${str-expr}|v=expr|v= |v=$str|
|v=${str:-expr}|v=expr|v=expr|v=$str|
|v=${str+expr}|v= |v=expr|v=expr|
|v=${str:+expr}|v= |v= |v=expr|
|v=${str=expr}|v=expr|v= |v=$str|
|v=${str:=expr}|v=expr|v=expr|v=$str|

----------

# 脚本内变量

脚本内的变量一旦声明赋值，即成为全局变量。

使用local关键字将变量限制为临时变量（局域性变量）
```bash
function test {
    local var=50
}
```


# 字符串处理

## 获取长度

方法一 
`${#string}`

方法二
使用expr命令
`expr length "$string"` 
string有空格则必须加双引号

## 获取字符索引位置

`expr index $string substr`

i> expr的起始下标为1，还需要注意，并不是拿substr这个字符串去string变量里找位置，而是将substr拆成单个字符，每个字符都会去匹配，返回最近的一个字符位置。

## 获取子串的长度

`expr match $string substr`

!> 只能从头匹配，从中间匹配会匹配不到。

## 抽取字符串中的子串。
方法一
i> 左往右起始下标为0，负数右往左其实下标为1
正
1. `${string:position}`
2. `${string:position:length}`
负
1. `${string:  -position}` 或 `${string:(position)}`
2. `${string:  -position:length}`

方法二
i> expr 起始下标为1

`expr substr  "var1" 10 5`

提取var1变量中下标为10开始，往后长度5的子串

----------

# 显示声明变量类型

shell也是支持声明有类型的变量

## declare

参数

`-r` 只读变量
`-i` 整数变量
`-a` 数组变量
`-f` 显示此脚本前定义过的所有函数及内容
`-F` 仅显示函数名
`-x` 环境变量

----------

# bc
支持浮点数运算
脚本列子
`echo scale=3; 23.4 / 4.5 | bc`
scale设置保留小数点后3位
