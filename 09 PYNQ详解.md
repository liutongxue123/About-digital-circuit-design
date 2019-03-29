---
typora-root-url: D:\typora图片总
---

# 09 PYNQ详解

## 六、PYNQ详解

### 0、参考链接

pynq-z2（一）启动板子连接jupyter(已修改)  https://blog.csdn.net/weixin_38438451/article/details/83474479

pynq官方教程：https://blog.csdn.net/weixin_38438451/article/details/83690719





### 1、PYNQ基础

通过PYNQ，用户可以使用python进行APSoC编程，可编程逻辑电路将作为**硬件库**导入并通过其API进行编程，其方式与导入和编程软件库基本相同，即PYNQ=python+ZYNQ，工作时直接调用Python库和FPGA硬件库进行功能的开发。

![](/PYNQ基础1.png)

整个FPGA部分的设计被称为Overlay，可面向多用户、多应用生成不同的Bitstream文件，支持通过软件API进行调用，动态的切换FPGA上的逻辑功能。



**官方对Overlay的描述：**

PYNQ提供了一个Python接口，允许通过PS中运行的Python控制PL中的Overlay。FPGA设计是一项需要硬件工程知识和专业知识的专业任务。PYNQ的Overlay层由硬件设计人员创建，并包含在此PYNQ Python API中。然后，软件开发人员可以使用Python接口来编程和控制专用硬件Overlay，而无需自己设计。

![](/QNN例程2.png)

结合上图，可以理解为：

**PYNQ的Overlay就是由硬件专家设计好的硬件协处理器。你现在可以在PYNQ中使用Python接口非常方便调用它！如果你有一定硬件知识，你也可以根据自己的需要自定义Overlay。**



另外，在PYNQ中已经提前安装好了很多库，例如open cv等。









### 2、PYNQ开发板使用流程

- 使用win32diskimager-1.0.0-install工具向SD卡中烧录镜像pynq_z1_v2.3.zip或pynq_z2_v2.3.zip（先解压）
- 连接串口和网线到PC
- 在PC中，断开其他wifi，打开网络共享中心 ->更改适配器设置 ->本地连接 ->右键属性 ->Internet协议版本4 ->属性，将PC的IP地址改为和开发板的IP地址在同一个网段下（即最后一部分可以不同）。

![](/../typora%E5%9B%BE%E7%89%87%E6%80%BB/PYNQ%E4%BD%BF%E7%94%A81.png)

- 也可以将pc和PYNQ板卡连接到相同的路由器下，这样，PYNQ就可以上网
- 启动板卡，稍等一会，当有蓝灯闪烁时，说明板卡已经配置完毕

- 打开浏览器，输入开发板的IP地址，打开jupyter notebook

- 若要传输文件到pynq，可以打开PC的磁盘界面，输入：192.168.2.99（板卡IP地址）\xilinx（**一定要反斜杠**）

- 可以通过SSH在任意linux系统中登陆板卡，命令为：ssh xilinx@192.168.2.99 ，密码为：xilinx 

  然后输入命令：su root ,密码：xilinx 取得权限

  （PYNQ共有两个账户，分别为root账户和普通账户xilinx）

**实例工程**（base）：

- 打开base工程，然后打开Block Design工程

    （注：创建工程选择板卡时，选择xc7z020clg400-1）

- 点击File ->export ->export Block Design，则会在文件夹中生成textAAA.tcl文件

- 在base文件下，选择base,runs ->impl_1 ->base_wrapper.bit文件，将其重命名为textAAA.bit，即保证其与 .tcl文件的名称相同。textAAA.tcl文件的作用是让python可以识别textAAA.bit文件。将这两个文件一起拷贝到开发板上。`拷贝xilinx主目录下`

- 打开PYNQ的终端，进行操作（参考overlay/mnist下的python程序）

    :one:   输入**python3**

​         :two:   输入 **from pynq import Overlay**   (用作在线烧写程序)

​	 :three:   输入 **ol=Overlay("textAAA.bit")**    （让系统找到textAAA.bit文件）

​                **以上两步的作用是：想要与IP进行交互，则要先加载包含IP的Overlay** 

​         :four:   输入 **ol.ip_dict**   （此命令可以输出工程中有哪些IP）

​         :five:   输入 **help(ol.axi_gpio_0)**     （此命令可以打开gpio的使用手册，里面有各种函数）

​         :six:   输入**ol.download()**                （此命令可以将textAAA.bit文件烧录到开发板上）

 	 :seven:   输入**gpio.write(0,15)**  可以给寄存器0赋值15，使LED灯全亮；输入 gpio.write(0,7)的话，此时亮三个灯

​	 :eight:   输入**dma=ol.axi_dma_0**，然后输入help(dma)，可以查看dma IP的函数

​       	 :nine:   关于开辟数组  （解决操作系统的虚拟地址和板卡实际的物理地址不匹配的问题，如下）​	

```python
from pynq import xlnk
import numpy as np
input_buffer = xlnk.cma_array(shape=(4000),cacheable=0,dtype=np.float32) //这样开辟出的数组，在物理空间也连续
                                                                                                                           //cacheable=0，表示不使用cache
input_buffer.physical_address   //此命令可以查看开辟的数组的真实物理地址
input_buffer.nbytes                   //此命令可以查看开辟的数组的大小
```



**关于从github下载项目报错的解决办法：**

```python
export GIT_SSL_NO_VERIFY=1
git clone --recursive https://github.com/Xilinx/QNN-MO-PYNQ.git
```





