+++
title = 'Kylin国产麒麟系统取证'
date = 2024-07-23T12:03:29+08:00
draft = false

+++
>   Kylin系统，为目前中国领先的自主可控操作系统之一，于2001年由国防科技大学研发，旨在打破外国操作系统在中国市场的垄断地位，并增强国家信息安全。
>

  麒麟系统最开始基于FreeBSD改写，Kylin 3.0版之后改用Linux来改写。

  安装时可能使用全盘加密，影响取证

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357438.png)

  麒麟系统默认root用户是不开启的，使用sudo -i 可以切换到root

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357684.png)

  尝试dd命令是可以使用的

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357187.png)

  运行x64内存转储工具avml，能够正常转储内存

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357889.png)

  运行检测工具

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357272.png)

  运行检测脚本

  ![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241357786.png)
