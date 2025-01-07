+++
title = 'windows下检测方法'
date = 2024-03-12T15:03:29+08:00
draft = false

+++
# 用户

 对于帐户，攻击者一般的利用方式有：

• 使用存在/默认的帐户

• 创建新的帐户 

• 删除/修改帐户 

• 账号克隆

 查看系统存在的用户：  

```powershell
net user	# 查看系统用户,但是无法查看
wmic useraccount list full
Get-LocalUser |Select * 		# powershell
# 注册表查看
```

 检查用户目录(可能不准确，第一次登录时才会创建) ，用户的目录显示的时间可能是用户实际创建的时间，也有情况实际时间可能比目录时间早。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241401541.png)

 对于用户帐户，多数情况下和业务需求用关，我们可能需要和管理员确认帐户的合法性。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241401742.png)

 AccountType，本地帐户和域帐户，类型都是512。

 SID和RID：每一个Windows帐户都有一个唯一的SID (Security Identifier)，格式如 S-R-I-S  ,   SID 以字符"S"开头，接着为版本标识 (通常设置为"1")，接着为Identifier-authority值（通常为 "5"），然后为一个或多个subauthority值。subauthority值最后为RID (Relative Identifier)， 表示主机上的特定对象。  与Linux类似，Windows用户的RID通常以500（174h）开始，administrator的RID就是500，系统新建用户从1001开始，如上图。RID在用户创建时被分配并+1，所以通过RID可以判断是否有用户被删除。

 RID 为5xx的帐户的状态为Degraded[Disabled]，这些帐户因为安全原因，默认是禁用的。在取证过程中，如果我们发现上面某个帐户被启用，尤其是内置的administrator或者guest，我们可以认为存在可疑，并进一步调查，我们一般使用事件ID 4722查看用户何时被启用以及为何启用。

```powershell
wevtutil qe security /f:text "/q:*[System[(EventID=4722)]]"
```

在20240809日用户admin启用guest账户

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241401561.png)

# 网络连接

对于网络连接，我们可以判断是否有进程外联恶意IP地址，或者不应该产生的网络连接，使用命令`netstat -anob 3`（每3秒查看一次所有连接情况）查看网络连接和对应进程。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241402097.png)

也可以使用`netstat -ano 3 | findstr 8.8.8.8`检测8.8.8.8相关的网络连接情况。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241402036.png)

对于cobaltstrike这种异步通信的C2，使用cports的log changes功能很容易记录到，建议在Advance Options中将自动刷新设置未2秒左右。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241402229.png)

同样我们可以是使用强大的微软SysinternalsSuite套件的监控工具Procmon，过滤Operation（操作）选择相关网络行为如tcp send、udp send。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241402771.png)

或者也可以快速选择只查看网络行为。

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409241402111.png)

如果网络中发现异常，我们可以记住pid，从进程分析。

# 进程

 👌进程进行分析，我们主要关注进程可执行程序对名称，可执行程序对位置，进程创建时间等信息。

 常见的恶意程序名称：

-  scvhost.exe
-  iexplore.exe
-  explorer.exe
-  随机字符串 

-  lsass.exe
-  win.exe 
-  winlogon.exe 
- **update.exe

常见的恶意程序位置： 

-  Windows\System32\ 
-  %Temp%
-  \Windows 
- WinSxS目录
-  \System Volume Information

-   $Recycle Bin
-  \Program files 
- Temorary Internet files  
-  .....

我们使用 tasklist和wmic对进程进行一个初步的分析：

tasklist	# 列出所有当前运行的进程

tasklist	#  列出所有当前运行的进程及加载的模块  

 tasklist /m [dll]  	#  列出加载了指定模块的进程  

wmic process list full	# 查看完整进程列表

wmic PROCESS WHERE Name="epr_update.exe" LIST FULL  	# 指定进程

咱们再扩展下wmic命令，全称Windows Management Instrumentation Command-line，是用于管理和监视Windows操作系统的命令行工具。它提供了丰富的功能，可以用来查询系统信息、执行管理任务和监控系统状态，示例：

```bash
基本参数		c:\> wmic [alias] [where clause] [verb clause] 
常用的[aliases]: 
    process         service 
    share           nicconfig 
    startup         useraccount 
[where 语句] 例子: 
    where name="nc.exe" 
    where (commandline like "%stuff") 
    where (name="cmd.exe" and parentprocessid!="[pid]") 
[verb 语句] 例子: 
    list [full|brief] 
    get [attrib1,attrib2] 
    call [method] 
    delete 

1、查看系统版本
wmic os get Caption, Version, OSArchitecture
2、查看账户信息
wmic useraccount get Name, FullName, Disabled
3、查看启动的服务
wmic service where "State='Running'" get Caption, StartName
```

 除了命令行工具，也可以使用图形界面工具procexp。我们可以在Options中选择Verify Image Signatures，在分析过程，我们可以优先查看没有签名，或者可执行程序路径异常的进程:  

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722851541280-80803f7a-6da3-40a9-a9ee-4cdf86e9ac87.png)

也可以使用procexp对比镜像文件与内存中的字符串来获取一些有用的信息

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722852653823-b1eda17f-a050-4b8f-a14c-3fc6a4cf2c6d.png)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722852752137-2927924a-98e9-4776-bccb-708de0987796.png)

 另外就是在取证过程，我们可能发现合法的进程发起了可疑的网络连接，此时需要检测是否存在代码注入或shellcode注入。我们可以使用pe-sieve或者hollows_hunter进行检测：  

pe-sieve：https://github.com/hasherezade/pe-sieve

hollows_hunter：https://github.com/hasherezade/hollows_hunter

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722852207340-6f8de78b-7157-4a1f-9b47-9361682a44e9.png)

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722934239761-1e1de8b9-722d-4253-bf7c-909bd0710683.png)

# 历史命令

cmd历史命令只记录当前shell窗口，关闭窗口后就清理了，使用`**doskey /history**`查看

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722851638629-8caf89a8-a8d1-469b-926f-f0e51b4c8b28.png)

powershell 保存完整的历史命令，位置如下

%userprofile%\AppData\\Roaming\\Microsoft\\Windows\\PowerShell\\PSReadLine\\ConsoleHost_history.txt

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722318374777-98f2d643-8792-453a-baa7-6cf19e9ad229.png)

# 文件及文件时间

对于文件系统的分析，我们可以先查看在入侵发生的时间点附近新增的文件。这里可以使用 Everything 的便携式版本。  

everything只支持ntfs磁盘文件格式，fat需要在选项中重新扫描：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722935134055-72434357-72f4-49d0-b34d-c6fe421cd1f1.png)

everything是创建文件索引库来完成快速检索，我们可以将文件索引库导出，Everything 的便携式版本生成数据库中程序目录下，安装版生成数据库位置在%LOCALAPPDATA%\everything\everything.db，如下：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722935663260-3bfaba17-d4f0-4b9f-a1a8-84bbccb7fdb8.png)

我们可以使用脚本读取取证回来的数据库文件

```bash
::读取everything数据库
@echo off
cd /d %~dp0
start /min .\Everything.exe -read-only -db .\Everything.db 
```

也可以在gui中导出

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722935257022-1fc3f140-5b88-44c7-a740-c633bc0b26cb.png)

根据文件创建时间查找：`dc:20240801`、根据文件修改时间查找：`dm:20240801`

示例

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722934901162-01630d06-4536-48f8-b5b4-6cfa182dbc9c.png)

搜索指定目录指定日期新增ext、exe、com后缀的文件 `c:\users\admin\Desktop\ datecreated:2024-05-10T08:00-2024-05-10T18:00 file:ext:exe;com`

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1723082798384-db75c273-5e4a-4164-a4f7-d1f4ee249ad2.png)

 除了搜索exe，dll外，也可以用来搜索webshell，如php，asp，aspx，jsp，jspx。  

# 隐藏文件

使用cmd快速查看重要目录的隐藏文件

cmd.exe /C dir /S /B /AHD C:\Windows\*

cmd.exe /C dir /S /B /A:H C:\Windows\System32\*

dir c:\Windows\System32\* /a:h

dir c:\Windows\System32\* /s /b /a:h

示例：

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1722841367780-8eb69011-36de-4791-ac6b-e4a0eccbec25.png)

# 其他

## 签名检测

使用Sysinternals套件中的sigcheck对程序签名检测

用法：sigcheck [-a][-h][-i][-e][-l][-n][[-s]|[-c|-ct]|[-m]][-q][-p <policy GUID>][-r][-u][-vt][-v[r][s]][-f catalog file] [-w file] <file or directory>

用法：sigcheck -d [-c|-ct] [-w file] <file or directory>

用法：sigcheck -o [-vt][-v[r]] [-w file] <sigcheck csv file>

用法：sigcheck -t[u][v] [-i] [-c|-ct] [-w file] <certificate store name|*>

-a 显示扩展版本信息。报告的熵度量是文件内容的每字节信息位数。

-c 带逗号分隔符的 CSV 输出

-ct 带制表符分隔符的 CSV 输出

-nobanner 以避免将横幅输出到 CSV

-d 转储目录文件的内容

-e 仅扫描可执行映像（无论其扩展名是什么）

-f 在指定的目录文件中查找签名

-h 显示文件哈希

-i 显示目录名称和签名链

-l 遍历符号链接和目录连接

-m 转储清单

-n 仅显示文件版本号

-o 使用 -h 选项时，对先前由 Sigcheck 捕获的 CSV 文件中捕获的哈希执行 Virus Total 查找。此用法适用于扫描离线系统。

-u 只显示未签名程序

-p 根据指定策略（由其 GUID 表示）或存储在指定策略文件中的自定义代码完整性策略验证签名。

-r 禁用证书吊销检查

-s 递归子目录

-vt:在 VirusTotal 上检查文件的哈希值，需要联网并且已配置 API 密钥。

-w 将输出写入指定文件。

`sigcheck -i c:\busybox.exe`		# 单个检测

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1723083826860-6d7ccfaf-a7bc-40e1-9193-b8e2afaf0dd4.png)

因为多数恶意程序是没有签名的，所以我们可以先对用户目录和C:\Windows目录可执行程序的签名进 行检测，优先分析未签名的程序：  `sigcheck  -u -s -c -e -h -w d:\sig.csv c:\Users\admin\` 结果保存在d:\sig.csv

![img](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/1723086253642-b213226f-01ec-4ef1-ac42-43c771b51089.png)

检查签名不足之处是整个扫描过程花费时间较长，建议作为最后一个手段。对于结果，我们可以基于 文件位置，签名，文件时间，hash等综合判断是否异常，可以将md5在威胁情报中判定，下面为几个常用的威胁情报：

微步在线威胁情报社区：https://x.threatbook.cn

virustotal：https://www.virustotal.com/gui/home/search

奇安信威胁情报中心：[https://ti.qianxin.com](https://ti.qianxin.com/)

## Windows 使用文件列表

**OpenSaveMRU**:这个注册表路径存储了用户最近使用的文件打开操作的记录

**OpenSavePidlMRU**:也是用来存储最近使用的文件路径记录，但是它记录的是通过文件系统对象标识符（PIDL，Persistent Item Identifier List）访问的文件路径

```powershell
reg query HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSaveMRU
reg query HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePidlMRU
```
