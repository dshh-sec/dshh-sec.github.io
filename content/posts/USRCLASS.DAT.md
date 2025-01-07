+++
title = 'USRCLASS.DAT取证'
date = 2024-08-06T15:03:29+08:00
draft = false

+++

ShellBags是一组用来记录文件夹（包括挂载网络驱动器文件夹和挂载设备的文件夹）的名称、大小、图标、视图、位置的注册表项，或称为BagMRU。每次对文件夹的操作，ShellBags的信息都会更新，而且包含时间戳信息。是Windows系统改善用户体验的功能之一。即使删除文件夹后，ShellBags仍然会保留文件夹的信息。因此可以用来揭示用户的活动。

ShellBags解析工具

ShellBagsExplorer加载USRCLASS.DAT

#### Windows 取证之ShellBags

https://zhuanlan.zhihu.com/p/585383290



- HKEY_CURRENT_USER\Software\Microsoft\Windows\ShellNoRoam
- HKEY_CURRENT_USER\Software\Microsoft\Windows\Shell
- HKEY_CURRENT_USER\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell（仅在 Windows Vista 中）



记录最近通过资源管理器访问的文件夹的信息：

USRCLASS.DAT：HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU

USRCLASS.DAT：HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\Bags



以时间线排列

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241353943.png)
