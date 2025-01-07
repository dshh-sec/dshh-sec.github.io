+++
title = 'Jffs2镜像挂载过程'
date = 2023-03-10T15:03:29+08:00
draft = false

+++

# 记录一次jffs2镜像挂载过程

在某IOT设备上取得磁盘镜像文件，jffs2常用在嵌入式Linux系统。现在需要挂载到本地Linux系统，取出文件进行分析。

环境：ubuntu20.4   for ARM       # 不建议使用centos

工具：apt install mtd-utils        # 也可以源码编译

##### 1、查看iot设备文件镜像类型：

`file txt.dd`

为大端的jffs文件系统：

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241121315.png)

> ​	JFFS2的全名为JournallingFlashFileSystemVersion2（闪存日志型文件系统第2版），其功能就是管理在MTD设备上实现的日志型文件系统。与其他的存储设备存储方案相比，JFFS2并不准备提供让传统文件系统也可以使用此类设备的转换层。它只会直接在MTD设备上实现日志结构的文件系统。JFFS2会在安装的时候，扫描MTD设备的日志内容，并在RAM中重新建立文件系统结构本身。
>
> ​	除了提供具有断电可靠性的日志结构文件系统，JFFS2还会在它管理的MTD设备上实现“损耗平衡”和“[数据压缩](https://baike.baidu.com/item/数据压缩?fromModule=lemma_inlink)”等特性。																						--引用《百度百科jffs2》

因为本地系统只支持小端的文件系统，我们可以使用jffs2dump转换工具进行转换。命令如下：

`jffs2dump -c -v -b -e`

##### 2、格式50M的空间，加载mtd插件；并且使用dd命令将镜像填充至/dev/mtdblock1

`modprobe mtdram total_size=50720      # 单位默认KB`

`modprobe mtdblock`

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241121608.png)

##### 3、使用dd填充结果

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241121673.png)

##### 4、mount挂载结果

`mkdir /mnt/mtd`

`mount -t jffs2 /dev/mtdblock0 /mnt/mtd`

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241121966.png)


##### 5、打开挂载目录/mnt/mtd，成功获取到磁盘中文件：

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241121004.png)


##### 6、最后

`rmmod mtdblock`
