+++
title = 'solaris&zfs取证常用命令'
date = 2022-12-08T15:03:29+08:00
draft = false

+++

# solaris

> Solaris是Sun Microsystems研发的计算机操作系统，采用SPARC架构或X86架构，主要用于工 作站、服务器上的操作系统。  
>
>  	SPARC全称为“可扩充处理器架构”（Scalable Processor ARChitecture），是RISC微处理器架 构之一，其指令集和X86有显著区别，并且有自己独有的窗口、延迟槽、过程调用特点。 SPARC架构的计算机一般用于工业、航天相关领域，其在类似IDC和一般IT场景的使用极为罕见。  



```
cat /etc/release    # 查看系统版本
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344151.png)

```
uname -r    # 内核版本
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344551.png)

```
ifconfig -a	# 查看网络地址
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344448.png)

```
ls -lh /etc/init.d/*	# 查看启动脚本
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344182.png)

```
crontab -l root	# root
ls -alht /etc/cron.*/*	# cron配置文件
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344930.png)

```
svcs    # 查看所有服务状态
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241344836.png)

```
df -h   # 磁盘空间使用情况
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345934.png)

```
ps -efc	# 列出所有进程
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345164.png)

```
netstat -anuv -P tcp -f inet		# 查看所有网络连接
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345627.png)

```
netstat -r 	# 查看网络路由信息
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345669.png)

```
pmap -x <进程ID> 		# 显示进程的内存映射
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345595.png)

```
gcore <pid> -o <output_file>    # 对可疑进程生成进程的core文件，可以用于gdb调试分析
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345954.png)

```
dd if=/dev/mem of=/path/to/output/file bs=1M count=1	# dd内存块（测试失败）
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345827.png)



磁盘镜像:

zpool list		# 查看zfs储存池，

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345851.png)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345941.png)

zfs list	# 查看zfs 文件系统

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345344.png)

```
format  # 查看磁盘，一般0号为系统盘，c2t0d0，zfs格式。
```

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345575.png)

\#存储设备可以是整个磁盘 (c1t0d0) 或单个分片 (c0t0d0s7)

对于每个一磁盘分片，通常以cxtxdxsx表示，其中c/t/d/s的含义如下：
*C代表控制器标号。控制器位于主板上面，所谓控制器，就是控制，发出命令的器件。
*T代表目标编号。   即分配给每个存储设备的一个唯一的硬件地址。
*D代表磁盘编号。   这个数字反映的是目标位置上的磁盘号，即具体的磁盘
*S代表分片编号。   通常从0～7，即分区的号码了

root@solaris:~# fstyp /dev/dsk/c2t0d0s1

# zfs

> `ZFS`，即 `Zettabyte File System`，常见于新版本的 Linux 系统，ZFS 是基于存储池的，与典型的映射物理存储设备的传统文件系统不同，ZFS 所有在存储池中的文件系统都可以使用存储池的资源。其支持自动校验数据的完整性，并对数据压缩提供了支持。ZFS 与 Raid 相组合，可以组成 Raid-z 和 Raid-z2. Raid-z 和 Raid-z2 常见于基于 Linux 系统的 NAS 设备中 ( 如 Freenas, Truenas ) ，相较于传统的几大 Raid 模式，其整合了 Raid 的优点，并将 ZFS 的高级特性也带入了 Raid 储存池。
>
> Solaris10默认的文件系统是[ufs](https://so.csdn.net/so/search?q=ufs&spm=1001.2101.3001.7020)（Unix Filesystem），当然也可以选装zfs；Solaris11默认的文件系统是zfs（Zettabyte Filesystem）。
>
> ZFS文件系统的英文名称为Zettabyte File System,也叫动态文件系统（Dynamic File System）,是第一个128位文件系统。最初是由Sun公司为Solaris 10操作系统开发的文件系统。作为OpenSolaris开源计划的一部分，ZFS于2005年11月发布，被Sun称为是终极文件系统。ZFS是基于存储池，与典型的映射物理存储设备的传统文件系ZFS统不同，ZFS所有在存储池中的文件系统都可以使用存储池的资源。
>
> ZFS 是一个组合文件系统和逻辑卷管理器。
>
> ZFS 的功能包括防止数据损坏、高存储容量 （256 ZiB）、快照和写入时复制克隆以及连续完整性检查等。





```
ssh root@192.168.253.138 "sudo dd if=/dev/dsk/c2t0d0s1" | dd of=./138-zfs.dd	# 远程dd到本地
```

rdsk	裸设备

dsk	块设备

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345572.png)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345101.png)



导出数据池

zpool export tank



iostat -En

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241345253.png)



磁盘挂载(失败)

zfs与通常文件系统ufs不同，自带类似软riad的储存池，磁盘挂载空间是由储存池分配。将要浏览文件，需要将raw转化为vmdk，由vmdk仿真查看文件。

系统盘仿真时引导失败可能需要光盘进入shell或修复。
