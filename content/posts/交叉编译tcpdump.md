+++
title = '交叉编译tcpdump'
date = 2023-07-10T15:03:29+08:00
draft = false

+++
![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409191513414.webp)

> 交叉编译tcpdump，在树莓派上抓包，用户层交叉编译比较简单。交叉编译器有很多，我们本次采用linaro([http://releases.linaro.org](http://releases.linaro.org/))交叉编译工具。
>



系统版本：centos7

编译工具：arm-linux-gnueabi



wget http://releases.linaro.org/components/toolchain/binaries/4.9-2017.01/arm-linux-gnueabi/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabi.tar.xz

wget<font style="color:rgb(77, 77, 77);"> </font>http://www.tcpdump.org/release/libpcap-1.8.1.tar.gz

<font style="color:rgb(77, 77, 77);">wget </font>http://www.tcpdump.org/release/tcpdump-4.9.1.tar.gz



1、下载好之后进行解压到任意目录

```shell
tar -xvf gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabi.tar.xz
tar -zxvf libpcap-1.10.3.tar.gz
tar -zxvf tcpdump-4.99.3.tar.gz
```

2、进入到编译工具的bin目录后，执行`./arm-linux-gnueabi-gcc -v `查看版本信息。

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409191513599.png)

3、配置编译器环境变量

vim /etc/profile

```bash
export ARCH=arm 
export CROSS_COMPILE=arm-linux-gnueabihf- export 
PATH=$PATH:/root/tcpdump/gcc-linaro-4.9.4-2017.01-x86_64_arm-linux-gnueabi/bin
```

source ~/.bashrc

4、编译libpcap和tcpdump

```bash
tar -zxvf libpcap-1.10.1.tar.gz
cd libpcap-1.10.1/
../configure CC=arm-linux-gnueabihf-gcc --host=arm-linux --prefix=/root/tcpdump/install
make
make install
```

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409191513361.png)

```bash
tar -zxvf tcpdump-4.99.1.tar.gz
cd tcpdump-4.99.1/
./configure CC=arm-linux-gnueabihf-gcc --host=arm-linux --prefix=/root/tcpdump/install
make
make install
```

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409191513223.png)

<font style="background-color:#FDE6D3;">ps：这里需要说明的是CC参数指定了交叉编译器, 编译两者时需要指定相同的目录, 否则在编译tcpdump时需要指定libpcap的路径。</font>

5、在安装目录下找到arm架构的tcpdump，结果如下

![](https://image-1253434595.cos.ap-chengdu.myqcloud.com/img/202409191513968.png)

