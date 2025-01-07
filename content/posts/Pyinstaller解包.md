+++
title = 'PYinstaller解包'
date = 2022-01-10T15:03:29+08:00
draft = false

+++
Pyinstaller用于打包python程序到二进制文件，win下打包为exe，linux下打包为elf，mac下打包为app。x86与arm架构均支持。

https://github.com/extremecoders-re/pyinstxtractor

pe

使用脚本解包，原先打包环境为python3.7

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241330688.png)

反编译pyc文件  

pip install uncompyle6

uncompyle6 -o xx.py xx.pyc

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241330809.png)

elf

 readelf -s erlfile 先查看pythondata数据地址

解包

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241330164.png) 







https://bbs.kanxue.com/thread-277811.htm
