+++
title = 'Iot取证'
date = 2022-09-19T10:00:00+08:00
draft = false

+++
常见的IoT路由器固件一般是保存在ROM flash芯片中，flash芯片是一种只读的固态半导体存储器，无法被改变或者删除，并且不会因为电源关闭而丢失数据

路由器固件提取，基本分为2种方法，一种是直接接触flash芯片、一种是从登录运行的路由器拷贝文件。



获取到flash

使用编程器（类似于读卡器）读取flash









获取到shell

1、proc/mtd文件保存着系统的磁盘分区信息

2、rootfs为固件文件系统（使用dd拷贝出来）

3、然后binwalk解包



获取到控制台（同上）











参考文章

[https://blog.csdn.net/Freedom_hzw/article/details/104216532](https://blog.csdn.net/Freedom_hzw/article/details/104216532)

[https://paper.seebug.org/2024/](https://paper.seebug.org/2024/)

