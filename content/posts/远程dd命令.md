+++
title = '远程dd命令'
date = 2023-03-07T00:00:20+08:00
draft = false

+++
一般ssh远程执行命令

`ssh root@121.5.106.25 -p 2333 "df -h"`

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1689321748984-6e07ce8d-63bf-4867-a159-b23c93e6cbde.png)

想要dd磁盘镜像到本地只需要加“|”管道符号，举个栗子  镜像/tmp/1.dd

`ssh root@192.168.253.145 -p 2333 "sudo dd if=/tmp/1.dd" | dd of=/root/1.image`

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1689321786209-5bb22927-9e7b-4a93-9b37-56c8748e73fe.png)

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1718259086439-2de5ca12-58df-4af1-9678-70396cb721f3.png)



带压缩的命令

远程主机上`dd if=/dev/sda | gzip -1 - | ssh user@local dd of=image.gz`

本地主机上`ssh user@remote "dd if=/dev/sda | gzip -1 -" | dd of=image.gz`

<font style="color:rgb(82, 82, 82);">#gzip -1 压缩级别 最高到9 默认是6级别</font>

#可以pv用来监视大型dd操作的进度，`dd if=/dev/sda | gzip -1 - | pv | ssh user@local dd of=image.gz`

<font style="color:rgb(82, 82, 82);">#</font><font style="color:rgb(33, 37, 41);">较新的dd版本也可以用</font><font style="color:rgb(33, 37, 41);background-color:rgb(239, 240, 241);">status=progress</font>来查看进度

