---
title: "使用EasyN2N进行联机"
date: 2023-10-15T16:56:58+08:00
draft: false
tags: ["计算机"]
description: 使用EasyN2N进行联机教程
katex: false
toc: false
summary: EasyN2N Cilent and Server
---

# How to Use EasyN2N 

## Linux Server

### Install

参照官网[^1]进行配置：

``` bash
gresces@lscpcf9xLh:~/TEMP$ wget https://github.com/ntop/n2n/archive/refs/tags/3.0.tar.gz
--2023-10-15 14:57:18--  https://github.com/ntop/n2n/archive/refs/tags/3.0.tar.gz
Resolving github.com (github.com)... 20.205.243.166
Connecting to github.com (github.com)|20.205.243.166|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://codeload.github.com/ntop/n2n/tar.gz/refs/tags/3.0 [following]
--2023-10-15 14:57:18--  https://codeload.github.com/ntop/n2n/tar.gz/refs/tags/3.0
Resolving codeload.github.com (codeload.github.com)... 20.205.243.165
Connecting to codeload.github.com (codeload.github.com)|20.205.243.165|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [application/x-gzip]
Saving to: ‘3.0.tar.gz’

3.0.tar.gz                        [                                              <=> ] 467.10K  26.8KB/s    in 13s

2023-10-15 14:57:32 (36.7 KB/s) - ‘3.0.tar.gz’ saved [478311]
```

当然这些软件是必不可少的：`autoconf automake libtool git kernel-headers`

运行解压指令 `tar xzvf 3.0.tar.gz`，之后进入目录`n2n-3.0`中

``` bash
gresces@lscpcf9xLh:~/TEMP/n2n-3.0$ ls
autogen.sh      config.guess      doc      legacy       packages   supernode.1     win32
CHANGELOG.md    configure.seed    edge.8   LICENSE      README.md  tests           wireshark
CMakeLists.txt  contributors.txt  include  Makefile.in  scripts    tools
community.list  COPYING           INSTALL  n2n.7        src        uncrustify.cfg
```

按照教程运行指令

``` bash
./autogen.sh
./configure
make
```

``` bash
gresces@lscpcf9xLh:~/TEMP/n2n-3.0$ cat Makefile | grep SBINDIR
export SBINDIR
SBINDIR=$(PREFIX)/local/sbin
SBINDIR=$(PREFIX)/sbin
        $(MKDIR) $(SBINDIR) $(MAN1DIR) $(MAN7DIR) $(MAN8DIR)
        $(INSTALL_PROG) supernode $(SBINDIR)/
        $(INSTALL_PROG) edge $(SBINDIR)/
        $(MAKE) -C tools install SBINDIR=$(abspath $(SBINDIR))
```

这里显示的是`supernode`和`edge`的安装路径：`/usr/sbin`

之后运行`sudo make install`

安装完成后检查是否正确安装

``` bash
gresces@lscpcf9xLh:/usr/sbin$ ls /usr/sbin/ | grep "supernode\|edge"
edge
supernode
```

显示上面的结果表明安装完成

### Use

新建一个文件夹（刚刚的TEMP文件夹可以删除），新建脚本文件`run.sh`，写入如下内容

可修改项：

- 端口 9527 自己在服务器上放通 TCP以及UDP端口
- IP段 172.16.0.0-172.16.1.0/24 可以控制连接数
- log文件 run.txt 将输出保存于此
- 标准错误信息输出 2>&1 错误信息一同输出到log文件中，建议使用

``` bash
nohup supernode -p 9527 -v -f -a 172.16.0.0-172.16.1.0/24 > ./run.txt 2>&1 &
```

运行指令

``` bash
gresces@lscpcf9xLh:~/n2n$ sudo ./run.sh

gresces@lscpcf9xLh:~/n2n$ cat run.txt
15/Oct/2023 15:30:08 [supernode.c:268] the network range for community ip address service is '172.16.0.0...172.16.1.0/24'
15/Oct/2023 15:30:08 [supernode.c:588] WARNING: using default federation name; FOR TESTING ONLY, usage of a custom federation name (-F) is highly recommended!
15/Oct/2023 15:30:08 [sn_utils.c:120] started shared secrets calculation for edge authentication
15/Oct/2023 15:30:08 [sn_utils.c:136] calculated shared secrets for edge authentication
15/Oct/2023 15:30:08 [supernode.c:604] supernode is listening on UDP 9527 (main)
15/Oct/2023 15:30:08 [supernode.c:613] supernode opened TCP 9527 (aux)
15/Oct/2023 15:30:08 [supernode.c:620] supernode is listening on TCP 9527 (aux)
15/Oct/2023 15:30:08 [supernode.c:629] supernode is listening on UDP 5645 (management)
15/Oct/2023 15:30:08 [supernode.c:641] dropping privileges to uid=65534, gid=65534
15/Oct/2023 15:30:08 [sn_utils.c:807] successfully created resolver thread
15/Oct/2023 15:30:08 [supernode.c:659] supernode started
```

检查运行情况，记得放通端口

``` bash
gresces@lscpcf9xLh:~/n2n$ ps -ef | grep 9527
nobody   15193     1  0 15:30 pts/0    00:00:00 supernode -p 9527 -v -f -a 172.16.0.0-172.16.1.0/24
gresces  15250 12024  0 15:30 pts/0    00:00:00 grep 9527
```

## Windows Cilent

### Use

下载软件[^2]，解压运行软件，需要安装虚拟网卡

> **注意**  本作者不保证该软件的绝对安全，包括客户端以及服务端

在程序主界面输入好友（服务器主）提供的`IP`地址，格式为 `IP:Port`，该端口为服务器配置中放通的端口

填入小组名称，并勾选自动分配（按需），点击启动即可

一般房主可以选择一个固定的端口以便于服务器连接，比如使用terraria这类IP联机需要房主IP的游戏

### For Server

当成员连接后，可以在服务器后台查看log文件

``` bash
15/Oct/2023 15:41:00 [sn_utils.c:2077] new community: Minecraft
15/Oct/2023 15:41:00 [sn_utils.c:1295] assigned sub-network 172.16.0.0/24 to community 'Minecraft'
15/Oct/2023 15:41:00 [sn_utils.c:1218] assign IP 172.16.0.54/24 to tap adapter of edge
15/Oct/2023 15:41:00 [sn_utils.c:1097] created edge  00:FF:12:5E:00:05 ==> 111.42.148.211:63111
15/Oct/2023 15:45:52 [sn_utils.c:1218] assign IP 172.16.0.53/24 to tap adapter of edge
```

这样就可以正常游玩了

[^1]:[ 使用N2N搭建虚拟局域网联机游戏（服务端） ](https://bugxia.com/336.html)

[^2]:[ EasyN2N（N2N启动器） v3.1.2 ](https://bugxia.com/357.html)

