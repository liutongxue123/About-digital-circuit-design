---
typora-root-url: D:\typora图片总
---

# 08 基于zynq平台的裸机工程

## 六、基于zynq平台的裸机工程

#### 1、zynq开发的一般流程( helloword )

这个hello word例程只会用到PS部分资源，包括ARM Cortex-A9，DDR3内存、一个UART串口，这是一个最小的系统。首先程序会加载到DDR内存中，然后CPU一条一条执行，执行的情况可以通过串口打印观察。

![](D:/typora图片总/zynq%E6%B5%81%E7%A8%8B2.png)

##### **（1）Vivado SOC硬件部分：** 

- 新建工程，输入工程名，工程存储路径
- 选择创建RTL Project
- 选择芯片类型（Parts和Boards均可）
- 单击Create Block Design，输入其名称为System
- 添加zynq IP，然后单击Run Block Automation ，直接单击OK
- 对Block中的zynq IP进行连线

![](D:/typora图片总/zynq%E6%B5%81%E7%A8%8B3.png)

- 此时可以双击zynq IP进行设置，使其对应正在使用的硬件（一般官方开发板不强制需要）
- 保存，然后点击Validata Design，验证工程的正确性
- 右击system.bd, 单击Generate Output Products，勾选Global

![](D:/typora图片总/zynq%E6%B5%81%E7%A8%8B4.jpg)

- 右击system.bd 选择Create HDL Wrapper 这步的作用是产生顶层的HDL 文件

![](D:/typora图片总/zynq%E6%B5%81%E7%A8%8B5.jpg)

- 选择 Let Vivado manager wrapper and auto-update 然后单击OK

![](D:/typora图片总/zynq%E6%B5%81%E7%A8%8B6.jpg)

- 点击Run Synthesis，先进行综合，然后进行实现

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E7%BB%BC%E5%90%88%E4%B8%8E%E5%AE%9E%E7%8E%B0.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E7%BB%BC%E5%90%88%E4%B8%8E%E5%AE%9E%E7%8E%B02.png)

- 点击Generate Bitstream生成比特流文件

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK5.jpg)

- File$\to$ Export$\to$ Export Hardware,导出SOC硬件到SDK

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E5%AF%BC%E5%87%BASOC%E7%A1%AC%E4%BB%B6%E5%88%B0SDK.jpg)

- 勾选Include bitstream 直接单击OK

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK1.jpg)



##### **（2）SDK软件部分：** 

- File->Launch SDK 加载到SDK，点击ok

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK2.jpg)

- 导出完成后如下图

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK4.jpg)

1、硬件部分，此部分即为从Vivado定制好的SOC硬件；2、这部分是硬件的地址空间分配

- 选择File->New->Application Project，创建新的工程，输入工程名hello wrld，点击Next，再选择hello world 工程模板

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK6.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK7.jpg)

- 工程创建完毕后，右键工程名hello world，然后选择Generate linker Script，可以查看所有内存的情况，代码、数据、堆栈运行所在内存的情况，这里不改动，关闭。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK8.png)

- 首先连接开发板，可以打开串口调试工具putty
- 右键工程名，选择Debug As，再选择Debug Configurations,进入调试配置界面。

（不调试，直接运行的话，选择Run As，再选择Launch on Hardware（System Debugger）运行）

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK9.png)

- 双击上图左侧的Xilinx C\C++ application(System Debugger)，然后进行如下设置

​      （勾选Reset entire system和Program FPGA，点击Apply和Debug）

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK10.jpg)

- 进入SDK调试界面（1、启动2、暂停3、停止4、代码5、信息控制台6、调试变量）

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK11.jpg)

- 启动系统自带的串口调试助手，进行相关设置

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SDK12.jpg)

- 单击运行输出结果







