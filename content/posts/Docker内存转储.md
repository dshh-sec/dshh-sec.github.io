+++
title = 'docker内存转储'
date = 2023-07-14T15:03:29+08:00
draft = false

+++
大多数docker为剪裁的nuix系统，很多的命令工具都没有，需要上传静编译好的busybox上传docker cp busybox-i486 8553e459dc85:\tmp与scp命令相似。

![image.png](1689300476991-3abd3884-202e-4d2f-ad2d-a4e9a15abcb9-1726652442112-95.webp)

1、方法一

![image.png](1689302239074-259deb79-d19d-410c-896a-05a42625fcfd.webp)



root也无权限，因为docker为了保证主机安全，docker开了很多安全设置，禁止ptrace，Docker 将gdb调试需要SYS_PTRACE属性被禁止掉了，所以gdb在调试的时候会显示ptrace被禁止。所以想在docker内部调试gdb解决办法就是create和run的时候带上以下参数。

\#采用超级权限模式

docker run --privileged  ......

\#关闭seccomp

docker run --security-opt seccomp=unconfined

\#仅开放ptrace限制 docker run --cap-add=sys_ptrace

docker安全机制中中包括ASLR（Address space layout randomization），即docker里的内存地址和主机内存地址是不一样的。ASLR会导致GDB这种依赖地址的程序无法正常运作。

使用gdb转储时应该要注意gdb调试中gdb默认需要关闭linux的地址随机化功能，可以通过gdb 命令set disable-randomization off关闭。

set disable-randomization on		# 开启

set disable-randomization off		# 关闭

show disable-randomization		# 显示

![image.png](1689304452847-09469603-8d59-46ec-9922-7002f7d081de-1726652442113-96.webp)



最后，通过超级权限模式进入，正常转储进程coredump文件，

![image.png](1689303425325-b08ee60a-5b27-4f08-a491-2d57b31ddd91.webp)



2、方法二

方法一中docker 运行时不能更改SYS_PTRACE且可能造成容器逃逸。所以可以才用以下方法，进入dockers，并且该 bash 并没有 SYS_PTRACE 权限的限制，可以非常方便的使用 gdb 了。



成功转储core

![image.png](1689315549944-96bea02a-df01-4056-b620-c9d0794e222f.webp)



3、疑问：转储宿主机pid是否正确？

从宿主机上找到对应的进程，然后在宿主机上执行gdb attach。

宿主机进程与容器进程有以下特点
1、docker容器内的一个进程对应于宿主机器上的一个进程。
2、容器内的进程，与相对应的宿主进程，由相同的uid、gid拥有。

![image.png](1689312378558-f11d4d2a-9898-4985-bd38-f254535f18ec.webp)



![image.png](1689312562296-f3c72e64-b840-4597-b0a6-22ceaa4a6df4.webp)







参考资料：

https://visualgdb.com/gdbreference/commands/set_disable-randomization?spm=a2c6h.12873639.article-detail.6.558714108Mgy0C

https://docs.docker.com.zh.xy2401.com/engine/security/seccomp/

https://www.zsythink.net/archives/4321
