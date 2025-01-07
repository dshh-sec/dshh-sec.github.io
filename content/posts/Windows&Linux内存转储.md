+++
title = 'Windows&Linux内存转储'
date = 2022-12-10T15:03:29+08:00
draft = false

+++
内存（RAM）是易失性数据，表现在当系统断电时候擦除，内存取证对寻找是否存在恶意程序有重要帮助。

# 原则

1、无损原则
防止由于对设备、系统的操作而损毁某些电子证据，造成证据收集不充分，在取证的过程中，不能对涉案设备、系统进行任何修改操作，以维护涉案设备、运行环境等全部信息的完整状态。
2、完整原则
在取证与鉴定过程中，应该尽可能地全面调查取证，认真分析电子证据的来源并进行全方位、多角度的取证和分析，在确保证据与案件事实关联的基础上，将获得的所有电子证据结合案件的其他证据，相互印证，排除矛盾的电子证据，最终形成完整的证据链条。
3、及时原则
随着系统时间的推移，如日志覆盖、C&C存活时间等，信息都会或多或少产生变化，这些数据信息则不再能够如实反映案件事实。因而电子证据具有一定的时效性，在确定取证对象之后，应该尽早收集证据，保证相关电子数据没有受到任何破坏或损失，维持其与案件事实的关联性。

# 步骤

1、确认取证环境，主要包括cpu架构、内存大小、系统类型及版本。
确认需求：单独进程内存 or 系统物理内存；（如果目标物理内存小于4GB，格式化为fat32就可以了。如果大于4GB，需要使用ssh远程拷贝。）
2、根据取证环境准备相应的工具，磁盘等
3、验证文件

# Windows内存转储

#### 1）任务管理器转储文件

打开任务管理器，找打可疑进程，右键点击进程，创建转储文件。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333838.png)

#### 2）WinHex

通过使用内存winhex对目标机器的运行内存进行单个进程提取，如图所示：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333258.png)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333616.png)

#### 3）DumpIt

DumpIt用于生成windows 32位、64位计算机物理内存转储。DumpIt绿色免安装，可以部署在usb上，快速响应应急时间。DumpIt 网上找到的开源版本比较旧（现在叫做MAGNET RAM Capture

），DumpIt下载链接：https://www.downloadcrew.com/article/23854/dumpit

运行DumpIt.exe程序，即可看到整个物理内存信息，输入yes即可获取整个物理内存，回显success为制作成功。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333392.png)

#### 4)FTK Imager

FTK Imager 默认需要安装，这里我们可以自己制作一个便携式版本。具体过程也比较简单，在非应急 的目标主机安装FTK Imager，然后将整个安装目录(通常为"C:\Program Files\AccessData\FTK Im ager"或"C:\Program Files (x86)\AccessData\FTK Imager")复制出来即可。 要提取内存，以管理员运行FTK Imager.exe
Windows10及以上推荐使用ftk imager。
file->Capture Memory

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333865.png)

#### 5）WinPmem

 项目地址：https://github.com/Velocidex/WinPmem

WinPmem 是一个开源项目，通过直接控制设备接口，从而为获取设备内存数据提供了更多的可能性。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333799.png)

执行`winpmem_mini_x64_rc2.exe WINDOWS10.RAW`获取系统内存转储，在程序目录中生成WINODWS.RAW

```plain
WinPmem64
Extracting driver to C:\Users\admin\AppData\Local\Temp\pme1B4E.tmp
Driver Unloaded.
Loaded Driver C:\Users\admin\AppData\Local\Temp\pme1B4E.tmp.
Deleting C:\Users\admin\AppData\Local\Temp\pme1B4E.tmp
The system time is: 05:22:16
Will generate a RAW image
 - buffer_size_: 0x1000
CR3: 0x00001AE000
 6 memory ranges:
Start 0x00001000 - Length 0x0009E000
Start 0x00100000 - Length 0x7EAF8000
Start 0x7F4F8000 - Length 0x0E216000
Start 0x8FC4E000 - Length 0x00001000
Start 0x90200000 - Length 0x05D80000
Start 0x100000000 - Length 0x362800000
max_physical_memory_ 0x462800000
Acquitision mode PTE Remapping
Padding from 0x00000000 to 0x00001000
pad
 - length: 0x1000

00% 0x00000000 .
copy_memory
 - start: 0x1000
 - end: 0x9f000

00% 0x00001000 .
Padding from 0x0009F000 to 0x00100000
pad
 - length: 0x61000

00% 0x0009F000 .
copy_memory
 - start: 0x100000
 - end: 0x7ebf8000
..............................................
```

# Linux下取内存转储

#### 1）dd命令

在2.4系及以下的内核可以使用dd命令拷贝内存
本地：dd if=/dev/mem of=/tmp/mem.raw bs=1M
远程：dd if=/dev/men | ssh x.x.x.x dd of=/tmp/xxx.raw

#### 2）AVML

微软的Linux的便携式内存采集工具，在GitHub上有编译好的可执行文件 （https://github.com/microsoft/avml）。
支持x86_64以下版本: 
Ubuntu: 12.04, 14.04, 16.04, 18.04, 18.10, 19.04, 19.10, 20.04, 21.04, 22.04
Centos: 6.5, 6.6, 6.7, 6.8, 6.9, 6.10, 7.0, 7.1, 7.2, 7.3, 7.4, 7.5, 7.6, 7.9 
RHEL: 6.7, 6.8, 6.9, 7.0, 7.2, 7.3, 7.4, 7.5, 7.7, 8.5
Debian: 8, 9, 10, 11
Oracle Linux: 6.8, 6.9, 6.10, 7.3, 7.4, 7.5, 7.6, 7.9, 8.5
CBL-Mariner: 1.0
使用方法：
1.对avml添加可执行权限，avml -help可以查看帮助
2.在目标主机执行avml xxx.lime。# 默认为lime格式

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241333472.png)

#### 3）GDB

使用gdb实时取证，不影响当前进程运行，取证完成后进程可以继续执行。
1.准备取证介质，格式化为ext2文件系统格式（如果可以识别nfts就不需要格式化），并把脚本和gdb文件拷贝到取证介质上。
2.使用root用户登录
2.挂载取证介质，进入挂载的目录，假设我们挂载在/mnt/qz下面，对文件添加执行权限chmod +x -R /mnt/qz。
操作流程如下图：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334504.png)

#### 4）Lime+lmg：

Lime最初用于 Android 设备的内存提取工作，也可以获取完整的Linux物理内存镜像, 弥补进程内存gdb获取的不足,比如内核级驱动进程dump无法获取到。结合开源内存取证分析工具Volatility，可以做很多的取证操作。
lmg是一个自动化脚本，可以利用lime、Volatility生成内存镜像和profile符号表文件。
Lmg脚本使用：
取证前查看一下内存使用情况，当swap使用率为0时取证，可以确保所有运行内存都被捕获。使用命令free -m查看：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334072.png)

1.    准备lmg-master文件包，并进入目录。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334331.png)

2.解压static-dwarfdump.tgz，并将dwarfdump设置为临时变量。或者采取可以编译安装dwarfdump、apt install、yum install。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334915.png)

3.进入~/lmg-master/volatility-2.3/tools/linux，编译执行make命令编译模块module.dwarf。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334090.png)

4.运行./lmg -y 如下图操作：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334061.png)

5.结果如下

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334229.png)

错误常发生在，目标机没有安装内核编译环境，导致驱动编译不过。最好取证前需要准备一台虚拟机安装和目标机一致的操作系统，并安装好内核编译环境，将取证介质挂载到虚拟机上，并在取证介质的/lime/src目录下编译好lime驱动。

#### VMware虚拟机内存转储

在windows中，计算机进入休眠状态后会产生hiberfil.sys内存文件，在继续使用是将文件拷贝到内存中。VMware中则是在我们虚拟机配置文件夹中生成VMEM文件。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241334495.png)

