---
title: use-vim
date: 2021-05-03 15:23:45.0
updated: 2022-07-30 15:52:21.06
url: /archives/use-vim
categories: 
- 技术
tags: 
- Vim
---



>一些vim的使用技巧


<!--more-->

-  `:shell`,`:sh`, `fish`, `...`
即可进入对应的shell 退出时回到vim 适合复杂长时间情况

-  `:! go run %`
在编辑完go代码后 用！表示执行shell命令 % 表示当前编辑文件 执行完回车返回vim
适合简单非交互式命令

vim默认使用系统默认使用的shell
 `:set shell ?` 查看 vim 当前使用哪个shell
 `:set shell=/path/to/shell` 设置你想要用的shell


 - `cat test.txt | vim -`
 cat 输出test文件内容 然后通过管道传给vim


 - 光标跳转

`Ctrl + d` 前进1/2屏幕
`Ctrl + u` 后退1/2屏幕
`{` 上一个段落
`}` 下一个段落
 ps:段落以空行分割
`w` 前进单词
`W` 前进单词（视标点符号为一体）
`b` 后退单词
`B` 后退单词（视标点符号为一体）
`$` 跳到行首
`0` 跳到行尾
`gg` 跳到第一行
`G` 跳到最后一行