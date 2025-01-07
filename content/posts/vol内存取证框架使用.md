+++
title = 'vol内存取证框架使用'
date = 2022-06-10T15:03:29+08:00
draft = false

+++
vol3之于vol2，很大的改变就是用symbol_tables(符号表)替换了profile(配置文件)，vol3带有一个广泛的符号表库，并且可以基于内存映像本身为大多数 Windows 内存映像生成新的[符号表](https://volatility3.readthedocs.io/en/latest/volatility3.framework.interfaces.symbols.html#volatility3.framework.interfaces.symbols.SymbolTableInterface)。



Linux确认内核版本：利用**banner**中关键词搜索“**Linux Version**”，从而再进一步选择**Volatility2**的**Profile**或者**Volatility3**的**Symbols**。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351908.png)



### Volatility3 [Symbol Tables](https://github.com/ffffffff0x/1earn/blob/master/1earn/Security/安全工具/Volatility.md#symbol-tables)

所有文件都以 JSON 数据的形式存储，它们可以是. json 的纯 JSON 文件，也可以是. json.gz 或. json.xz 的压缩文件。Volatility 会在使用时自动解压它们。使用时也会将它们的内容（压缩后）缓存起来，位于用户主目录下的. cache/volatility3 中，以及其他有用的数据。缓存目录目前无法更改。

符号表 JSON 文件默认位于 volatility/symbols 下，在操作系统目录下（目前是 windows、mac 或 linux 中的一种）。符号目录是可以在框架内配置的，通常可以在用户界面上设置。

这些文件也可以被压缩成 ZIP 文件，Volatility 将处理 ZIP 文件以定位符号文件。ZIP 文件必须以相应的操作系统命名（如 linux.zip、mac.zip 或 windows.zip）。在 ZIP 文件中，目录结构应与未压缩的操作系统目录一致。

- Windows 符号表对于 Windows 系统，Volatility 接受由 GUID 和所需 PDB 文件的 Age 组成的字符串。然后，它在 Windows 子目录下的已配置符号目录下搜索所有文件。与文件名模式 /-.json（或任何压缩变体）匹配的任何文件都会被使用。如果找不到这样的符号表，则将从 Microsoft 的 Symbol Server 下载关联的 PDB 文件，并将其转换为适当的 JSON 格式，并将其保存在正确的位置。Windows 符号表可以从适当的 PDB 文件手动构建。用于执行此操作的主要工具内置于 Volatility 3 中，称为 pdbconv.py。
- Mac / Linux 符号表对于 Mac / Linux 系统，两者都使用相同的识别机制。JSON 文件位于符号目录下的 linux 或 mac 目录下。生成的文件包含一个标识字符串（操作系统横幅），Volatility 的 automagic 可以检测到该字符串。易失性会缓存字符串和它们来自的符号表之间的映射，这意味着精确的文件名无关紧要，并且可以在操作系统目录下的任何必要层次结构下进行组织。可以使用称为 dwarf2json 的工具从 DWARF 文件生成 Linux 和 Mac 符号表。当前，带有调试符号的内核是恢复大多数 Volatility 插件所需的所有信息的唯一合适方法。找到具有调试符号 / 适当的 DWARF 文件的内核之后，dwarf2json 会将其转换为适当的 JSON 文件。
- 

一般情况Windows符号文件会自动从微软下载，Linux、mac需要自己生成。也可以自行到第三方下载：

开源的 profile 在线搜索列表

- https://isf-server.techanarchy.net/
- https://github.com/leludo84/vol3-linux-profiles/tree/main

符号文件PDB ，也可以在其他环境良好的机器中制作Profile、Symbols，需要保持下面3点与取证目标系统一致。

1、Linux 发行版

2、内核版本

3、CPU 架构（32 位、64 位等）



Volatility2版Linux常用插件

```plain
linux_apihooks             - 检查用户名apihooks
linux_arp                  - 打印ARP表
linux_aslr_shift           - 自动检测Linux aslr改变
linux_banner               - 打印Linux Banner信息
linux_bash                 - 从bash进程内存中恢复bash历史记录
linux_bash_env             - 恢复一个进程的动态环境变量
linux_bash_hash            - 从bash进程内存中恢复bash哈希表
linux_check_afinfo         - 验证网络协议的操作函数指针
linux_check_creds          - 检查是否有任何进程正在共享凭证结构
linux_check_evt_arm        - 检查异常向量表以查找系统调用表钩子
linux_check_fop            - 检查rootkit修改的文件操作结构
linux_check_idt            - 检查IDT是否被更改
linux_check_inline_kernel  - 检查内联内核挂钩
linux_check_modules        - 将模块列表与sysfs信息进行比较
linux_check_syscall        - 检查系统调用表是否已被更改
linux_check_tty            - 检查tty的钩子
linux_cpuinfo              - 打印有关每个活动处理器的信息
linux_dentry_cache         - 从dentry缓存收集文件
linux_dmesg                - 收集dmesg buffer
linux_dump_map             - 将选定的内存映射写入到磁盘
linux_dynamic_env          - 恢复进程的动态环境变量
linux_elfs                 - 在进程映射中找ELF二进制文件
linux_enumerate_files      - 列出文件系统缓存引用的文件
linux_find_file            - 列出并从内存中恢复文件
linux_getcwd               - 列出每个进程的当前工作目录
linux_hidden_modules       - Carves内存寻找隐藏的内核模块
linux_ifconfig             - 收集活动接口
linux_info_regs            - GDB中的“info寄存器”。它打印出所有的输出
linux_iomem                - 提供与/proc/iomem相似的输出
linux_kernel_opened_files  - 列出从内核中打开的文件
linux_keyboard_notifiers   - 解析键盘通知调用链
linux_ldrmodules           - 将proc映射的输出与libdl中的库列表进行比较
linux_library_list         - 将库加载到一个进程中
linux_librarydump          - 将进程内存中的共享库转储到磁盘
linux_list_raw             - 列出应用程序与混杂的套接字
linux_lsmod                - 收集加载内核模块
linux_lsof                 - 列出文件描述符及其路径
linux_malfind              - 查找可疑的过程映射
linux_memmap               - 转储用于linux任务的内存映射
linux_moddump              - 提取加载内核模块
linux_mount                - 收集挂载的fs/devices 
linux_mount_cache          - 收集从kmem_cache安装的fs/设备。
linux_netfilter            - 列出Netfilter钩子
linux_netscan              - 刻画网络连接结构
linux_netstat              - 列表打开的套接字
linux_pidhashtable         - 通过PID哈希表枚举进程
linux_pkt_queues           - 将每个进程的数据包队列写入磁盘
linux_plthook              - 扫描ELF二进制文件' PLT hooks
linux_proc_maps            - 收集进程内存映射
linux_proc_maps_rb         - 通过映射红黑树收集linux的进程映射
linux_procdump             - 将进程的可执行映像转储到磁盘
linux_process_hollow       - 检查是否有进程被挖空的迹象
linux_psaux                - 收集进程和完整的命令行和开始时间
linux_psenv                - 收集进程及其静态环境变量
linux_pslist               - 收集活动任务通过task_struct->task list
linux_pslist_cache         - 从kmem_cache中收集计划任务
linux_psscan               - 扫描进程的物理内存
linux_pstree               - 显示进程之间的父/子关系
linux_psxview              - 查找隐藏进程与各种各样的进程列表
linux_recover_filesystem   - 从内存中恢复整个缓存的文件系统
linux_route_cache          - 从内存中恢复路由缓存
linux_sk_buff_cache        - 从sk_buff kmem_cache中恢复数据包
linux_slabinfo             - 在一台正在运行的机器上模拟/proc/slabinfo。
linux_strings              - 将物理偏移量匹配到虚拟地址(可能需要一段时间，非常详细)
linux_threads              - 打印进程的线程
linux_tmpfs                - 从内存中恢复tmpfs文件系统。
linux_truecrypt_passphrase - 恢复缓存Truecrypt口令
linux_vma_cache            - 从vm_area_struct 缓存中收集VMAs
linux_volshell             - 内存映像中的shell 
linux_yarascan             - Linux内存映像中的一个shell

#列出文件列表
vol -f 1.mem --profile=LinuxUbuntu1804-5_4_0-84x64 linux_enumerate_files
#-i 选项的参数是linux_enumerate_files得到的偏移量
vol -f 1.mem --profile=LinuxUbuntu1804-5_4_0-84x64 linux_find_file -i 0xf5a4e568 -O file.txt
#bash历史命令
vol -f 1.mem --profile=LinuxUbuntu1804-5_4_0-84x64 linux_bash

扩展插件

aim4r/VolDiff - 利用 Volatility 框架来识别 Windows 7 内存中恶意软件威胁的 Python 脚本
JamesHabben/evolve - Web 界面版的 Volatility
kevthehermit/VolUtility - Web 界面版的 Volatility
andreafortuna/autotimeliner - 自动从 memory dump 中提取取证时间线
superponible/volatility-plugins

配置文件路径 volatility\plugins\overlays
```

Volatility插件

malfind插件：

Volatility寻找注入的代码是通过使用'malfind'功能完成的，可能包含注入代码的进程的列表，基于十六进制显示的头信息，权限和一些提取的[汇编代码](https://www.varonis.com/blog/how-to-use-x64dbg?hsLang=en)，需要关注红色区域，如果出现“MZ”值时，那么可能已经确定了一个恶意软件，它已经注入了另一个进程。

ps.Windows可执行文件的头在十六进制中总是以 "4D 5A "开始，在ASCII中表示为 "MZ"



![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351654.jpeg)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351652.png)



pypykatz插件：用于提取Windows操作系统凭据和敏感数据，如密码哈希、明文密码以及域名和本地用户的凭据等。（https://github.com/skelsec/pypykatz-volatility3）

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351720.png)

dumpcerts插件：扫描证书，可以从内存镜像中提取X.509证书。证书是一种数字文件，用于验证身份、加密通讯和识别网站等，广泛用于电子商务、安全认证和网络通信等领域。通过分析内存中的证书数据，我们可以了解系统中安装的证书类型、颁发机构、有效期以及其他相关信息。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351813.png)

prefetch插件：该插件正在扫描、提取和解析从 Windows XP 到 Windows 11 的 Windows Prefetch 文件。(https://www.forensicxlab.com/posts/prefetch/)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351901.png)

antorun：扫描Windows自启进程

(https://github.com/Telindus-CSIRT/volatility3-autoruns)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351128.png)

volatility-docker：用于检查内存中的dockers（https://github.com/amir9339/volatility-docker）

The Docker plugin has a few options:

- **detector** - When choosing this option the plugin will give the investigator a quick indication about the presence of Docker / Docker containers running on the machine.
- **ps** - When choosing this option the plugin will display a table, similar to docker ps command output, that shows the following details about running containers on the machine: container creation time, running command, container-id, is privileged, container process PID.
- **inspect-caps** - When choosing this option a list of running containers will be displayed and the plugin will enumerate the containers’ capabilities.
- **inspect-mounts** - When choosing this option a list of non-default mounts will be displayed with information about the associated container, mount paths, and mount options.
- **inspect-networks** - When choosing this option a list of Docker networks will be displayed by their IP segments and the containers that are related to them.



**mimikatz**

- https://github.com/RealityNet/hotoloti/blob/master/volatility/mimikatz.py

```plain
python2 -m pip install construct
cp mimikatz.py /volatility/plugins/
python vol.py  -f tmp.vmem --profile=Win7SP1x64 mimikatz
```

osinfos.py、recentdocs.py（https://github.com/svn0/volatility3-plugins）

从注册表读取系统版本信息、和最近打开文档

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351793.png)



https://github.com/f-block/volatility-plugins



快速dump：https://github.com/reverseame/modex

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241351224.png)



**官方仓库**

- [volatilityfoundation/community](https://github.com/volatilityfoundation/community) - Volatility plugins developed and maintained by the community
- [volatilityfoundation/community3](https://github.com/volatilityfoundation/community3) - Volatility3 plugins developed and maintained by the community



TODO

1、构建每个独特发行版和内核的Profile、Symbols、lime,ko





ps:VMware 注意,要复制 .vmem 和 .vmss/.vmsn 文件。在某些情况下，需要 .vmss 文件才能正确解析 .vmem 内存文件。



参考

[https://github.com/ffffffff0x/1earn/blob/master/1earn/Security/%E5%AE%89%E5%85%A8%E5%B7%A5%E5%85%B7/Volatility.md](https://github.com/ffffffff0x/1earn/blob/master/1earn/Security/安全工具/Volatility.md)
