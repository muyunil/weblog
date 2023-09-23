+++
title = "Ansible"
date = 2021-05-09 10:33:10.0
[taxonomies]
tags = ["Linux", "运维"]
+++



服务器多起来还是需要借助工具的帮助。
<!--more-->

# 简介（抄）
Ansible是一种开源软件供应，配置管理和应用程序部署工具，可将基础架构作为代码。
它可以在许多类Unix系统上运行，并且可以配置类Unix系统以及Microsoft Windows。
它包含自己的描述性语言来描述系统配置。Ansible由Michael DeHaan撰写，并于2015年被Red Hat收购。
Ansible是无代理的，可以通过SSH或Windows远程管理临时连接。（允许远程PowerShell执行）来执行其任务。

反正大体来讲用于管理几十到几百台机器还是可以的，在多的话就有性能局限了，毕竟是Python写的。

# 准备使用

准备用于管理的主机地址，编辑/etc/ansible/hosts ：

```yml
[AlyEcs]
192.168.10.1
192.168.10.2:22
192.168.10.3:2222
192.168.10.[4:20]

[WebServer]
192.168.20.1
...
...
```
`[name] `表示以下所属ip的标签，使用标签代替ip集。
一行一ip，可以显示标注ssh端口，规律性地址可以使用`[1:100]`表示这一位数值是从1到100。

# 标签选择

标签中使用中可以进行与或非
```bash
或
ansible "A:B" --list-host
#A与B标签下的所有ip
与
ansible "A:&B" --list-host
#A与B标签下同时存在的ip
非
ansible "A:!B" --list-host
#在A里面不在B里面的ip
支持正则表达式（~开头开启）
ansible "~(web|db).*\.test\.com" --list-host
```

# 执行顺序

1. 读取配置文件
2. 加载模块文件（py）
3. 生成临时py文件传输到远程主机
4. 给文件执行权限
5. 执行并且返回结果
6. 删除临时文件

可用-v -vv -vvv 查看执行详情，v越多越详细。

# ansible-galaxy
别人写好的`角色`工具（类脚本）
用于下载[galaxy](https://galaxy.ansible.com)上相应的`roles`

```bash
ansible-galaxy list
#列出已安装的galaxy

ansible-galaxy install geerlingguy.mysql
#安装
anainle-galaxy remove geerlingguy.mysql
#删除
```

# ansible-pull
虽然写的pull但它实际上是将命令推送到远程主机，据说效率高但对运维要求较高
先放着，后面回头再看。

# ansible-playbook
此工具用于执行编写好的playbook任务,playbook是用yml格式写的，缩进严格。
例子：
`ansible-playbook hello.yml`
hello.yml:
```yml
#广播hello world
- hosts: Aly
  remote_user: root
  tasks:
    - name: hello world
      command: /usr/bin/wall hello world
```

# ansible-vault 
为了防止yml文件里的一些敏感数据泄露，可以使用ansible-vault加密playbook文件。
语法：
`ansible-vault encrypt hello.yml`
参数：`encrypt`:加密，`decrypt`：解密，`view`:查看,`edit`：编辑加密文件，`rekey`：修改口令，`create`:创建新文件。

# ansinle-console
提供一个命令行，对远程主机操作。
直接ansible-console进入
`root@all (5)[f:5]$`
默认是`all`标签，代表所有主机。
（5）代表all标签下有5台主机，[f:5]并发数是5，可通过forks n设置 
使用`cd`切换标签如 `cd AlyEcs`，`list`查看当前标签下的主机。

# 修改默认模块
因为默认的模块是command，但对于很多Linux命令支持不好，所以修改为shell模块代替。
编辑/etc/ansible/ansible.cfg
修改 `module_name = command`的值为`shell`

# 使用模块

ansinle选项：
`--version `
显示版本
`-m module`
指定模块
`-v `
详细过程，-vv，-vvv 更详细
`--list-hosts`
显示主机列表
`-k, --ask-pass`
提示输入密码，不加默认key验证
`-C, --check`
检查，不执行
`-T`, `--timeout=TIMEOUT`
执行命令超时时间（默认10s）
`-u`,  `--user=REMOTE_USER`
执行命令远程用户
`-b`, `--become`
代替旧版sudo切换
`--become-user=USERNAME`
指定sudo的runas用户（默认root）
`-K`, `--ask-become-pass`
提示输入sudo口令

语法:`ansible [标签/ip] -m 模块 -a 模块参数`

## shell模块
功能：在远程主机上使用shell命令
shell模块可以完成大部分较为简单的工作。
列子：
```shell
ansibe AlyEcs -m shell -a "mkdir -p /tmp/test"
#在AlyEcs标签下的所有主机上创建/tmp/test目录

ansible all -a "echo myPasswd | passwd --stdin root"
#`all` 是一个自带标签用于代表hosts里的所有主机
#因为将shell模块设置为了默认模块，所以可以省略不写
#效果为给hosts下所有主机的root用户密码设置为myPasswd
```
剩下的自由发挥，但并不是所有shell命令都支持，还有有局限性的，不然要那么多模块干什么。

## Script模块
功能：在远程主机上执行在ansible主键上写好的脚本。

```bash
anisble all -m script -a 'hello.sh'
````
将hello.sh推送到所有主机执行。

## copy模块
功能：从ansible主控端推送文件/文件夹到远程主机。
```bash
ansible dataSer -m copy -a "src=/data/data.txt dest=/tmp/data.txt owner=myuser mode=600"
```
