+++
title = 'arm定时转储'
date = 2024-09-10T15:03:29+08:00
draft = false

+++

需求：每小时取一次内存，并传回本地笔记本中。



树莓派版本如下，内存1GB，系统版本Ubuntu，内核版本armv7l

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241120216.png)

<font style="color:rgb(79, 79, 79);">arm架构使用lime来转储内存，需要编译模块。浏览编写需要的资源，发现已经有内核src文件了，但缺少头文件，树莓派下的linux_headers安装于其他平台不同，需要使用</font>`sudo apt install raspberrypi-kernel-headers`命令安装



Windows（//192.168.1.10/mem）目录共享到树莓派，并挂载到/mnt/mem，取的内存将存入/mnt/mem

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241120725.png)



把lime命令写入转储脚本，使用crontab定时执行转储脚本，也可以命令直接写入coon，如每小时的30分的时候取一次`30 * * * * sudo insmod /tmp/lime-5.15.76-v7+-armv7l.ko "path=/mnt/mem/lime`date "+%Y-%m-%d_%H%m"` format=lime" && sodu rmmon lime`

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241120431.png)



计划任务结果，每一小时执行取一次内存

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241120042.png)

