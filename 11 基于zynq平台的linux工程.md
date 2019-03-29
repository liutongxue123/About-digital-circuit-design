---
typora-root-url: D:\typora图片总
---

# 07 基于zynq平台的linux工程

## 七、基于zynq平台的linux工程

在ZYNQ基础上加入Python内核和Python编译环境的网络服务器以及FPGA硬件库。

FPGA硬件库目前有：PYNQ-API接口+overlay





**如何在ZedBoard上实现PYNQ（PYNQ Linux on ZedBoard）  https://blog.csdn.net/dia323/article/details/83021648



PYNQ Linux on ZedBoard https://superuser.blog/pynq-linux-on-zedboard/



Embedded Linux Tutorial - Zybohttps://www.instructables.com/id/Embedded-Linux-Tutorial-Zybo/ 



**PYNQ在ZCU102上的移植【PYNQ】 https://blog.csdn.net/vacajk/article/details/84728062 

需要先在虚拟机上安装 Xilinx 工具，包括PetaLinux, Vivado, and SDx



Zynq-Linux移植学习笔记之九-petalinux   https://blog.csdn.net/zhaoxinfan/article/details/57530627

ZYNQ跑系统 系列（二） petalinux方式移植linux https://blog.csdn.net/long_fly/article/details/78727813

PetaLinux安装教程，基于Ubuntu  https://blog.csdn.net/u013793399/article/details/53054734

ubuntu16.04安装Vivado 2017.4 教程  https://blog.csdn.net/wmyan/article/details/78926324

ubuntu16.04下安装petalinux_2017.4详细流程  https://blog.csdn.net/u012230668/article/details/84951299

zynq学习笔记之petalinux （1）安装Ubuntu16.04.1   https://blog.csdn.net/zhupingyang/article/details/80507602

zynq学习笔记之petalinux https://blog.csdn.net/zhupingyang?t=1



zc706教程   https://www.cnblogs.com/paladinzxl/p/8882141.html

Xilinx ZC706嵌入式开发和Petalinux小试 http://www.21ic.com/app/eda/201702/703561.htm

 https://download.csdn.net/download/l312361206/8969115

petalinux使用-终极教程 https://blog.csdn.net/xiang_shao344/article/details/83144126

Xilinx Petalinux及ZC702 BSP安装教程https://download.csdn.net/download/l312361206/8969115

petalinux开发文档http://www.360doc.com/content/15/0204/11/18525993_446160690.shtml

BSP包中各个文件的具体作用https://blog.csdn.net/leoliu0128/article/details/6532607



学习ZYNQ博客 https://blog.csdn.net/long_fly?t=1



关于ZCU102开发板和SDSoc开发环境 https://blog.csdn.net/weixin_37182342/article/details/80227429

ZC706开发板官网介绍（含技术文档）： https://china.xilinx.com/products/boards-and-kits/ek-z7-zc706-g.html#hardware



PYNQ的YOLO例程：https://steinslab.io/archives/1543





###  1、安装教程

#### （1）安装VMware和ubuntu16.04

- 首先安装好VMware，按照默认一路next即可
- 准备好ubuntu16.04的ios镜像文件
- 打开VMware，选择新建虚拟机，选择自定义安装

![](/ubuntu1.png)

- 直接下一步：

![](/ubuntu2.png)

- 先创建一个空白磁盘，稍后安装系统

![ubuntu3](/ubuntu3.png)

- 默认选项“linux”，下一步

![ubuntu4](/ubuntu4.png)

- 选择虚拟机的安装位置和名称，我的虚拟机就放在E:\Virtual_Machine目录下面，下一步；

![ubuntu5](/ubuntu5.png)

- 配置虚拟机的处理器数量，下一步；

![ubuntu6](/ubuntu6.png)

- 分配内存，内存根据自己电脑的配置进行分配，UG1144手册要求最小8G，这里我给虚拟机分配了2G内存。

![ubuntu7](/ubuntu7.png)

- 添加网络，选择NAT，下一步；

![ubuntu8](/ubuntu8.png)

- 选择IO控制器类型，默认LSI（logic）-L，下一步；

![ubuntu9](/ubuntu9.png)

- 选择磁盘类型，默认SCSI，下一步；

![ubuntu10](/ubuntu10.png)

- 创建新的虚拟磁盘，下一步；

![ubuntu11](/ubuntu11.png)

- 指定磁盘大小，手册要求100G以上，下一步（**越大越好，最少120G**）；

![ubuntu12](/ubuntu12.png)

- 指定磁盘文件，默认就行，下一步；

![ubuntu13](/ubuntu13.png)

- 点击完成

![ubuntu14](/ubuntu14.png)

- 点击“编辑虚拟机设置”

![ubuntu15](/ubuntu15.png)

- 选择“CD/DVD(SATA)”，选择“使用ISO镜像文件”，然后指向准备好的Ubuntu镜像文件，点击“确定”。

![ubuntu16](/ubuntu16.png)

- 点击“开启此虚拟机”，启动镜像文件的安装； 点击“InstallUbuntu”，开始安装（注意这里选English）；

![ubuntu17](/ubuntu17.png)

- 选择“Downloadupdates while install Ubuntu”（需要主机能够联网），点击“Continue”；

![ubuntu18](/ubuntu18.png)

- 选择“Erase diskand install Ubuntu”，开始“Install now”，然后点击“Continue”

![ubuntu19](/ubuntu19.png)

![ubuntu20](/ubuntu20.png)

- 时区选择“shanghai”，选择语言“chinese”，设置用户名，密码；点击“Continue”，开始安装；

![ubuntu21](/ubuntu21.png)

![ubuntu22](/ubuntu22.png)

![ubuntu23](/ubuntu23.png)

- 安装完成，先关机；

![ubuntu24](/ubuntu24.png)

- 在“虚拟机设置”中，取消“使用ISO镜像文件”，改为“使用物理驱动器”，点击“确定”

![ubuntu25](/ubuntu25.png)



**接下来安装VMware tools，可以让文件直接在windows和linux之间拖拽：**

- 点击：虚拟机---安装VMware tools；弹出VMware Tools的安装文件 
- 在/home目录下，新建文件夹vmware_tool
- 将VMware Tools的安装文件复制到建立得到文件夹vmware_tool下
- 在文件夹vmware_tool下解压安装文件
- 创建管理员身份，（vmtools需要管理员权限安装），并切换到管理员身份

```
sudo passwd root
[sudo] password for robin:
enter new UNIX password:
retype new UNIX password:
passwd:password update succeddfully
su
```

- 安装VMware Tools

在vmware_tool文件夹下，进入解压出来的vmware-tools-distrib 文件夹，然后运行安装程序：

```
cd ~/home/vmware_tool/vmware-tools-distrib 
./vmware-install.pl     
```

然后一路回车和yes即可。



#### （2）在ubuntu16.04中安装petalinux 2017.4

**官方的软件安装指南**： https://china.xilinx.com/support/documentation/sw_manuals/xilinx2018_2/ug1144-petalinux-tools-reference-guide.pdf

**petalinux2017.4安装包下载地址：** https://china.xilinx.com/support/download/index.html/content/xilinx/zh/downloadNav/embedded-design-tools/2017-4.html

**安装环境要求：** 

8 GB RAM (recommended minimum for Xilinx tools)

2 GHz CPU clock or equivalent (minimum of 8 cores)

100 GB free HDD space

Supported OS:

\- RHEL 7.2/7.3 (64-bit)

\- CentOS 7.2/7.3 (64-bit)

\- Ubuntu 16.04.1 (64-bit)

**安装流程：** 

- 此处不需要更换ubuntu的apt-get源为阿里云的源，直接使用默认源即可
- 根据UG1144的列表，安装petalinux的依赖库

![](/UG1144.png)

```
$ sudo apt-get install tofrodos
$ sudo apt-get install iproute
$ sudo apt-get install gawk
$ sudo apt-get install xvfb
$ sudo apt-get install git
$ sudo apt-get install make
$ sudo apt-get install net-tools
$ sudo apt-get install libncurses5-dev
$ sudo apt-get install tftpd
$ sudo apt-get install zlib1g:i386
$ sudo apt-get install libssl-dev
$ sudo apt-get install flex
$ sudo apt-get install bison
$ sudo apt-get install libselinux1
$ sudo apt-get install gnupg
$ sudo apt-get install wget
$ sudo apt-get install diffstat
$ sudo apt-get install chrpath
$ sudo apt-get install socat
$ sudo apt-get install xterm
$ sudo apt-get install autoconf
$ sudo apt-get install libtool
$ sudo apt-get install tar
$ sudo apt-get install unzip
$ sudo apt-get install texinfo
$ sudo apt-get install zlib1g-dev
$ sudo apt-get install gcc-multilib
$ sudo apt-get install build-essential
$ sudo apt-get install libsdl1.2-dev
$ sudo apt-get install libglib2.0-dev
$ sudo apt-get install screen
$ sudo apt-get install pax
$ sudo apt-get install gzip

```

至于列表中的python3.4，其实ubtunu16.04中同时安装了python2和python3(两者不兼容)，ubuntu默认的是python2，暂时不用管。

- 在安装petalinux可能会警告：

```
No tftp server found - please refer to "PetaLinux SDK Installation Guide" for its impact and solution
```

解决办法是：（启动tftp服务端）

```
$ sudo apt-get install tftp
$ sudo apt-get install openbsd-inetd
$ sudo gedit /etc/inted.conf
#在文件中增加以下内容：
tftp        dgram    udp    wait    nobody    /usr/sbin/tcpd    /usr/sbin/in.tftpd  /tftpboot
#保存并退出
$ sudo mkdir /tftpboot
$ sudo chmod 777 /tftpboot    #修改文件夹权限
$ /etc/init.d/openbsd-inetd  restart 
$ netstat -an | more | grep udp
#看到有如下输出，表示tftp安装成功   
udp        0      0 0.0.0.0:69              0.0.0.0:*
```

如果按照上述操作，还没解决这个问题：

首先安装tftp服务端：**sudo apt-get install tftpd-hpa**        //tftpd-hpa是服务器端
安装好后配置服务器的设置：**sudo vim /etc/default/tftpd-hpa**

编辑内容为：

![](/tftp.png)

其中第二项是TFTP的传送目录，将传输的文件放在该文件夹下即可，在这里新建一个/tftpboot目录用于存放文件。

而且要注意的是要将/tftpboot文件夹的权限修改成777，命令如下：chmod 777 /tftpboot

第三项tftp地址是主机的ip地址，设置好保存即可。

然后重新启动tftp服务：**sudo service tftpd-hpa restart**

到这步，主机上的tftp服务就算配置完成    

- 修改/bin/sh，因为ubuntu默认的“/bin/sh”是dash，需要修改成bash

```
$ ls -al /bin/sh                                   #查看未修改之前的/bin/sh
lrwxrwxrwx 1 root root 4 5月  29 16:40 /bin/sh -> dash
$ sudo dpkg-reconfigure dash                       #修改，弹出的对话框选择“否”
$ ls -al /bin/sh                                   #查看修改后的/bin/sh
lrwxrwxrwx 1 root root 4 5月  30 15:14 /bin/sh -> bash
```

- 接下来是安装petalinux2017.4

```
$ mkdir -p /home/robin/petalinux
$ chmod 755 /home/robin/petalinux         //注意：petalinux必须在普通用户权限下安装！！！
```

- 直接将安装包（petalinux-v2017.4-final-installer.run）拖拽到linux下，这里将其拖到桌面

```
$ cd ~/Desktop/
$ ./petalinux-v2017.4-final-installer.run /home/robin/petalinux/
INFO: Checking installer checksum... 
INFO: Extracting PetaLinux installer...
.........                                #开始安装
```

- 安装过程中，需要查看证书等，直接回车+:q+y    一共重复3次，再继续等。。。。直到安装成功
- 验证是否安装成功

```
$ source /home/robin/petalinux/settings.sh  #首先要设置petalinux的环境变量
PetaLinux environment set to '/home/zhupy/petalinux'
INFO: Checking free disk space
INFO: Checking installed tools
INFO: Checking installed development libraries
INFO: Checking network and other services
$  echo $PETALINUX
/home/zhupy/petalinux         #安装成功
```

- 到这里的话，以后每次重新启动系统后都需要重新执行：

```
$ source /home/zhupy/petalinux/settings.sh
```

- 设置打开终端，自动执行

```
$ gedit ~/.bashrc
#修改~/.bashrc文件 
#这个.bashrc 是终端的初始化配置脚本，每次打开新的终端的时候，都会执行这个脚本，把环境变量配置脚本加在里面，就能在打开的时候自动配置了。
#在最后一行增加
source /home/robin/petalinux/settings.sh
```

- 关闭终端，再打开

![](/petalinux启动.png)

#### （3）在ubuntu16.04中安装vivado 2017.4

1、上官网下载vivado hlx 版本2017.4，和 链接如下：

[Vivado Design Suite - HLx Editions - 2017.4 Full Product Installation](https://www.xilinx.com/support/download.html)

选择版本：Vivado Design Suite - HLx Editions - 2017.4  Full Product Installation  大约16G.

2、直接使用指令  （也可以直接右键压缩包解压）  

```
$ tar  xvzf   xxx(你下载的文件名).tar.gz
```

3、进入解压缩之后的文件夹  然后执行

```
$ sudo ./xsetup
```

4、进入安装界面，根据需求选择， 安装即可，这里可以改安装路径

5、验证是否安装成功，与petalinux一样

```
$ source ~/home/robin/Desktop/xilinx/Vivado/2017.4/settings64.sh
$ vivado          #这时，系统会自动打开vivado
```

6、设置打开终端，自动执行，与petalinux一样

```
$ gedit ~/.bashrc
#修改~/.bashrc文件 
#这个.bashrc 是终端的初始化配置脚本，每次打开新的终端的时候，都会执行这个脚本，把环境变量配置脚本加在里面，就能在打开的时候自动配置了。
#在最后一行增加
source /home/robin/Desktop/xilinx/Vivado/2017.4/settings64.sh
```



#### （4）使用petalinux教程

**一般设计流程：** 

![](/petalinux使用教程8.png)

**一、对于特定的硬件平台（开发板），petalinux可以通过下载官方BSP来直接创建文件。**

1、首先将下载的bsp包复制到想要建立工程的目录下（以ZC702为例）

2、创建工程

```
petalinux-create -t project -s xilinx-zc702-v2018.2-final.bsp
#若已经cd到bsp的同级目录下，则可以直接按上述命令执行
#若在其他的目录，则需要添加bsp的路径
```

之后会创建一个文件夹在与bsp同级的目录下：目录名为xilinx-zc702-2018.2/

3、cd到目录xilinx-zc702-2018.2下，执行命令

```
petalinux-build
```

等待一段时间，会生成响应镜像。



**二、对于没有官方BSP文件的情况下，可以创建一个自定义的工程。**



1、在vivado上创建硬件平台，生成***.hdf文件。

Create Project：

![](/petalinux使用教程1.png)

Create Block Design：

![](/petalinux使用教程2.png)

Create IP：

![](/petalinux使用教程3.png)

根据UG1144中对硬件的要求设置IP（一般是默认）：

![](/petalinux使用教程4.png)

```
1.TTC模块（必须） ，如果有多个，Linux内核将会使用第一个。 
2.外部32MB存储空间（必须） 
3.UART模块（必须），控制台打印信息用，若用IP核的话，需中断信号连到PS 
4.非易失存储器（可选），如：QSPI Flash，SD/MMC 
5.以太网接口（可选），若用IP核或外部PHY的话，需中断信号连到PS
```

（在这里，可以选择启动TTC、UART、SD以及之后可能用到的USB、Ethernet（网口0在bank1，bank1电压要选1.8V，否则报错） 

![](/petalinux使用教程9.png)

Generate output products -->Create HDL Wrapper -->Generate Bistream -->Export Hardware

生成的hdf文件在：<工程目录>\Miz_sys.sdk文件夹内 

2、在linux中创建一个工程目录，可以放在桌面上

```
$ cd ~/Desktop/
$ mkdir petalinux_proj
```

3,  在工程目录中创建工程

```
$ cd ~/Desktop/petalinux_proj/
$ petalinux-create --type project --template zynq --name my_first_proj
INFO: Create project: my_first_proj
INFO: New project successfully created in /home/zhupy/Desktop/petalinux_proj/my_first_proj
```

4、将步骤1中生成的hdf文件从windows中复制到my_first_proj这个文件夹中

5、导入硬件配置文件

```
$ cd ~/Desktop/petalinux_proj/my_first_proj/
$ petalinux-config --get-hw-description=./        #这里的./是hdf文件的存放位置
```

![](/petalinux使用教程5.png)

设置项的内容直接默认，save后退出。（细节可以参考UG1144的附录B和附录C）开始config。。。。完成后出现以下信息：

```
[INFO] generating u-boot configuration files                                                                         
[INFO] generating kernel configuration files
[INFO] generating kconfig for Rootfs
Generate rootfs kconfig
[INFO] oldconfig rootfs
[INFO] generating petalinux-user-image.bb
```

此时在~/Desktop/petalinux_proj/my_first_proj/project-spec/hw-description文件目录下，可以看到：

```
$ ls
config.project  ps7_init_gpl.c  ps7_init.html  system_wrapper.bit
metadata        ps7_init_gpl.h  ps7_init.tcl
ps7_init.c      ps7_init.h      system.hdf
```

6、生成镜像文件

```
$ cd ~/Desktop/petalinux_proj/my_first_proj/   #切换到工程文件夹
$ petalinux-build
[INFO] building project
[INFO] sourcing bitbake
INFO: bitbake petalinux-user-image
Parsing recipes: 100% |##########################################| Time: 0:00:38
Parsing of 2466 .bb files complete (0 cached, 2466 parsed). 3259 targets, 226 skipped, 0 masked, 0 errors.
NOTE: Resolving any missing task queue dependencies
Initialising tasks: 100% |#######################################| Time: 0:00:07
Checking sstate mirror object availability:   3% |               | ETA:  0:00:03
..........
INFO: Copying Images from deploy to images
INFO: Creating images/linux directory
NOTE: Successfully copied built images to tftp dir:  /tftpboot
[INFO] successfully built project
```

此时，工程目录内多出一个images文件夹

```
$ cd ./images/linux/
$ ls
image.ub               rootfs.ext4      system.dtb          vmlinux
rootfs.cpio            rootfs.ext4.gz   System.map.linux    zImage
rootfs.cpio.gz         rootfs.jffs2     system_wrapper.bit  zynq_fsbl.elf
rootfs.cpio.gz.u-boot  rootfs.manifest  u-boot.bin
rootfs.ext3            rootfs.tar.gz    u-boot.elf
```

7、若linux中没有安装vivado，所以不能用petalinux-package命令，需要把以下四个文件拷贝到windows中备用：

![](/petalinux使用教程6.png)

​	在Windows中打开SDK Xilinx-->Create Boot Image，`按顺序`加入 zynq_fsbl.elf--->IFC_TOP_wrapper.bit--->u-boot.elf 三个文件。点击 Create Image 按钮，生成BOOT.bin文件

![](/petalinux使用教程7.png)

8、若linux中有vivado，对zynq7000系列生成引导镜像：

```
petalinux-package --boot --fsbl <FSBL image> --fpga <FPGA bitstream> --u-boot
说明：
petalinux-package --boot：是一条命令，生成BOOT.bin的引导文件，详情见UG1157
 
--fsbl：磁盘/SD卡上到达FSBL elf二进制文件的路径，默认：<plnx-proj-root>/images/linux.
• zynqmp_fsbl.elf for Zynq UltraScale+MPSoC
• zynq_fsbl.elf for Zynq-7000
• fs-boot.elf for MicroBlaze.
 
--fpga：磁盘上bit二进制流文件的路径，也就是vivado生成的bit文件路径，无默认，由用户指定
 
--u-boot:可选的，磁盘上U-Boot二进制文件的路径， 默认：<plnx-projroot>/images/linux
• u-boot.elf for Zynq family device
• u-boot-s.bin for MicroBlaze.
 
实用例子：
petalinux-package --boot --fsbl ./images/linux/zynq_fsbl.elf --fpga <PATH-TO-BITSTREAM> --u-boot
说明：--fpga无默认路径，必须由用户指定其路径：例如：~/XXX.bit
```

之后会在linux目录下生成BOOT.bin文件

9、将生成的BOOT.bin和之前拷贝的image.ub复制到SD卡上。

​      此时SD卡只需要一个FAT32分区即可，不需要再做文件系统（分区）。

10、将SD卡插入开发板上电，打开Putty，查看putty上的log信息。

​	用户名密码默认都为root。



























  