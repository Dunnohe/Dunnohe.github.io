---
layout:     post
title:      "Linux命令学习-lsof命令"
subtitle:   " \"了解lsof的用法\""
date:       2016-07-20 20:20:00
author:     "Dunno"
header-img: "img/post-bg-2015.jpg"
tags:
    - Linux
---

# 目录

- <a href="#js">介绍</a>
- <a href="#ckzl">参考资料</a>
- <a href="#xkd">先看懂top的输出吧</a>
- <a href="#tcy">lsof常用的一些命令</a>


# <a name="js">介绍</a>
<p>lsof（“list open files”）命令名称很容易记住，把它看成“ls + of”, 在linux下面，一切都是文件，lsof则可以查看这些文件。</p>

# <a name="ckzl">参考资料</a>
> http://www.tecmint.com/10-lsof-command-examples-in-linux/

> http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/lsof.html?highlight=lsof

# <a name="xkd">先了解输出吧</a>
<p>我们先运行一下lsof -R(加-R 可以打出ppid)命令</p>
<pre>
<code>
# lsof -R
COMMAND     PID  PPID      USER   FD      TYPE             DEVICE SIZE/OFF      NODE NAME
init          1     0      root  cwd   unknown                                       /proc/1/cwd (readlink: Permission denied)
init          1     0      root  rtd   unknown                                       /proc/1/root (readlink: Permission denied)
init          1     0      root  txt   unknown                                       /proc/1/exe (readlink: Permission denied)
init          1     0      root NOFD                                                 /proc/1/fd (opendir: Permission denied)
</code>
</pre>

- COMMAND 进程名称
- PID 进程id
- PPID 进程父id
- USER 进程所有者
- FD 文件描述符
	<pre>
	<code>
	（1）cwd：表示current work dirctory，即：应用程序的当前工作目录，这是该应用程序启动的目录，除非它本身对这个目录进行更改
	（2）txt ：该类型的文件是程序代码，如应用程序二进制文件本身或共享库，如上列表中显示的 /sbin/init 程序
	（3）lnn：library references (AIX);
	（4）er：FD information error (see NAME column);
	（5）jld：jail directory (FreeBSD);
	（6）ltx：shared library text (code and data);
	（7）mxx ：hex memory-mapped type number xx.
	（8）m86：DOS Merge mapped file;
	（9）mem：memory-mapped file;
	（10）mmap：memory-mapped device;
	（11）pd：parent directory;
	（12）rtd：root directory;
	（13）tr：kernel trace file (OpenBSD);
	（14）v86  VP/ix mapped file;
	（15）0：表示标准输出
	（16）1：表示标准输入
	（17）2：表示标准错误
	一般在标准输出、标准错误、标准输入后还跟着文件状态模式：r、w、u等
	（1）u：表示该文件被打开并处于读取/写入模式
	（2）r：表示该文件被打开并处于只读模式
	（3）w：表示该文件被打开并处于
	（4）空格：表示该文件的状态模式为unknow，且没有锁定
	（5）-：表示该文件的状态模式为unknow，且被锁定
	同时在文件状态模式后面，还跟着相关的锁
	（1）N：for a Solaris NFS lock of unknown type;
	（2）r：for read lock on part of the file;
	（3）R：for a read lock on the entire file;
	（4）w：for a write lock on part of the file;（文件的部分写锁）
	（5）W：for a write lock on the entire file;（整个文件的写锁）
	（6）u：for a read and write lock of any length;
	（7）U：for a lock of unknown type;
	（8）x：for an SCO OpenServer Xenix lock on part      of the file;
	（9）X：for an SCO OpenServer Xenix lock on the      entire file;
	（10）space：if there is no lock.
	</code>
	</pre>
- TYPE 文件类型
	<pre>
	<code>
	（1）DIR：表示目录
	（2）CHR：表示字符类型
	（3）BLK：块设备类型
	（4）UNIX： UNIX 域套接字
	（5）FIFO：先进先出 (FIFO) 队列
	（6）IPv4：网际协议 (IP) 套接字
	</code>
	</pre>	
- DEVICE 指定磁盘的名称
- SIZE 文件大小
- NODE 索引节点
- NAME 打开文件的确切名称

# <a name="tcy">lsof常用的一些命令</a>
	
- 查询指定用户打开的文件
<pre>
<code>
lsof -u heliang05
COMMAND  PID      USER   FD      TYPE             DEVICE SIZE/OFF      NODE NAME
sshd    3126 heliang05  cwd   unknown                                       /proc/3126/cwd (readlink: Permission denied)
sshd    3126 heliang05  rtd   unknown                                       /proc/3126/root (readlink: Permission denied)
sshd    3126 heliang05  txt   unknown                                       /proc/3126/exe (readlink: Permission denied)
sshd    3126 heliang05 NOFD                                                 /proc/3126/fd (opendir: Permission denied)
</code>
</pre>
- 找到在特定TCP端口上运行的进程
<pre>
<code>
# lsof -i TCP:22
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sshd    1471    root    3u  IPv4  12683      0t0  TCP *:ssh (LISTEN)
sshd    1471    root    4u  IPv6  12685      0t0  TCP *:ssh (LISTEN)
</code>
</pre>
- 找到特定TCP端口范围上的进程
<pre>
<code>
# lsof -i TCP:0-1024
COMMAND    PID    USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
Google     258 lianghe  153u  IPv4 0x8bc54911385f362b      0t0  TCP 172.30.30.61:57712->10.32.255.116:http (ESTABLISHED)
Google     258 lianghe  156u  IPv4 0x8bc5491133dc4d23      0t0  TCP 172.30.30.61:57819->103.37.152.63:https (CLOSE_WAIT)
</code>
</pre>
- 查询所有的网络连接
<pre>
<code>
# lsof -i
COMMAND    PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind   1203     rpc    6u  IPv4  11326      0t0  UDP *:sunrpc
rpcbind   1203     rpc    7u  IPv4  11330      0t0  UDP *:954
rpcbind   1203     rpc   11u  IPv6  11336      0t0  TCP *:sunrpc (LISTEN)
avahi-dae 1241   avahi   13u  IPv4  11579      0t0  UDP *:mdns
avahi-dae 1241   avahi   14u  IPv4  11580      0t0  UDP *:58600
</code>
</pre>
- 查询所有的ipv4连接
<pre>
<code>
# lsof -i 4
COMMAND    PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind   1203     rpc    6u  IPv4  11326      0t0  UDP *:sunrpc
rpcbind   1203     rpc    7u  IPv4  11330      0t0  UDP *:954
rpcbind   1203     rpc    8u  IPv4  11331      0t0  TCP *:sunrpc (LISTEN)
avahi-dae 1241   avahi   13u  IPv4  11579      0t0  UDP *:mdns
avahi-dae 1241   avahi   14u  IPv4  11580      0t0  UDP *:58600
</code>
</pre>
- 查询所有的ipv6连接
<pre>
<code>
# lsof -i 6
COMMAND    PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind   1203     rpc    9u  IPv6  11333      0t0  UDP *:sunrpc
rpcbind   1203     rpc   10u  IPv6  11335      0t0  UDP *:954
rpcbind   1203     rpc   11u  IPv6  11336      0t0  TCP *:sunrpc (LISTEN)
rpc.statd 1277 rpcuser   10u  IPv6  11858      0t0  UDP *:55800
</code>
</pre>
- 根据指定的pid查找 	
<pre>
<code>
# lsof -p 1
COMMAND PID USER   FD   TYPE     DEVICE SIZE/OFF   NODE NAME
init      1 root  cwd    DIR      253,0     4096      2 /
init      1 root  rtd    DIR      253,0     4096      2 /
init      1 root  txt    REG      253,0   145180 147164 /sbin/init
</code>
</pre>
- 列出谁在使用某个端口 	
<pre>
<code>
# lsof -i :3306
</code>
</pre>

这些参数也可以互相组合使用









 