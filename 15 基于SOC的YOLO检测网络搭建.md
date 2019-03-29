---
typora-root-url: D:\typora图片总
---

# 11 基于SOC的YOLO检测网络搭建

## 十一、基于SOC的YOLO检测网络搭建

### （0）链接

基于FPGA的深度学习CNN加速器设计：https://blog.csdn.net/xiuxin121/article/details/78999697

如何简单快捷地用小型Xiliinx FPGA加速卷积神经网络CNN https://blog.csdn.net/awai54st/article/details/78177351

​                                                                                                  https://github.com/awai54st/PYNQ-Classification

用FPGA实现深度卷积神经网络：https://blog.csdn.net/cj1343395571/article/details/80140242







### （1）QNN-MO-PYNQ--基于PYNQ的tiny-YOLO官方例程

官方例程的地址链接为：https://github.com/Xilinx/QNN-MO-PYNQ

QNN的全称为：Quantized Neural Network 量化神经网络

MO的全称为：Multi-Layer Offload

项目中使用的是修改后的Tiny YOLO模型，叫做Tinier-YOLO。

在VOC数据集上训练，使用了1bit的权重层和3bit的激活层。

**tiny-yolo模型如下（9个卷积层，6个最大池化层）：**

![](/QNN例程1.png)

YOLO的特点为，在一端将图像的像素信息输入，经过前向推理计算，就可以得到检测结果。

在前向推理的过程中，卷积层计算占用了绝大部分时间，因此需要使用硬件加速的overlay加速这些计算密集的部分。





**执行流程：**

**1、安装：**

```
sudo pip3 install git+https://github.com/Xilinx/QNN-MO-PYNQ.git
```

如果报错SSL证书问题，检查你PYNQ板上的时间！如果时间早于目前时间，可能造成SSL整数不生效导致Https拉取失败。

安装完成后，我们可以看到在jupyter中出现了我们刚才安装的库用例。

![](/QNN例程5.png)

**2、运行**

打开tiny-yolo-image-loop.ipynb即可运行我们的Tiny YOLO。看下其中的逻辑：

首先导入必要的库。我们在pip安装的时候，安装了qnn库和yolo算法的Darknet库。

![](/QNN例程3.png)

然后创建分类器，**初始化加速器。** （在这个过程中，我们的加速器Overlay被示例化进我们的FPGA中，成为了我们的硬件协加速器！该加速器已经由库设计好。当然，在以后的进阶开发中，我们可以创建自己的协加速器。）

下面的配置文件形式和权重形式，类似于Darknet框架

![](/QNN例程4.png)

软硬协同前向推理：在大循环中，不断地载入图片，进行前向推理，最终输出结果。

**值得注意的是，在这个DEMO中，第一层和最后一层是用CPU软件算的，其余层使用了FPGA硬件协加速器。** 

![](/QNN例程6.png)

输出的结果为：

**输出速度，大概是3秒/1帧。注意，不是1秒3帧，是3秒1帧。** 

![](/QNN例程7.png)

考虑到ZYNQ7020**不到5w**的功耗，和嵌入式ARM的计算能力，这个成绩符合我的预料。根据我之前Darknet框架的实验，Tiny YOLO在1GHz的ARM A9纯CPU计算需要70秒以上。虽然本实验的模型是“Tinier YOLO”，这个成绩已经获得了相较纯软件计算数十倍的加速。



在源码中，/qnn/src/network/make-hw.sh，利用这个可以在UNIX环境中，复制硬件架构，，另外此目录下的hls-syn.tcl等尝试还原。

### （2）yolov2_xilinx_fpga--github例程

工程链接：https://github.com/dhm2013724/yolov2_xilinx_fpga

此工程在PYNQ板卡上实现了yolov2模型。

**关于例程的复现：**

首先打开**vivado hls** 的命令行窗口，然后进入到yolov2_xilinx_fpga文件夹下的hls目录

然后执行命令：**vivado_hls –f script.tcl** 

（注意：vivado版本会影响结果，官方是2018.2，自己用2017.4也行）



















**关于加速器部分的架构：**

![](/yolo实现架构.png)

此加速器有两个AXI4主接口和一个AXI4-Lite从接口。

CPU通过AXI4-Lite接口读写配置数据和状态寄存器，进而对加速器进行控制。输入特征和权重通过两个AXI4主接口从DDR中同时被提取，输出特征通过AXI4接口再写回DDR。

右边两个Data Scatter模块用于生成对应的读地址（DDR中对应数据的地址），并将从DRAM（DDR）中读取的数据分配到片上缓冲区（Input Pixel Buffers和Weight Buffers）。

左边的Data Scatter模块用于生成将数据写回DRAM的地址，并将输出缓冲区（Output Pixel Buffers）上的数据写回DRAM

红色模块分别是卷积层（Conv和Leaky ReLU），最大池化层（Maxpooling）和重组层（Reorg）。



​	由于数据输入和数据量非常大，loop tiling技术被用于加载数据并减少内存访问的时间，即将卷积循环R，C，M，N平铺为Tr，Tc，Tm，Tn。（参考论文Optimizing FPGA-based Accelerator Design for Deep Convolutional Neural Networks  ）



**关于权重分配（weight arrangement)，数据访存：**

​	由于数据输入和数据量非常大，loop tiling技术被用于加载数据并减少内存访问的时间，即将卷积循环R，C，M，N平铺为Tr，Tc，Tm，Tn。（参考论文：Optimizing FPGA-based Accelerator Design for Deep Convolutional Neural Networks  ）

​	FPGA的有效带宽随着burst长度的增加而增加，最终会在burst长度的阈值处变平。（参考论文：Caffeine: Towards Uniformed Representation and Acceleration for Deep Convolutional Neural Networks ）

​	data tiling（数据平铺）技术通常导致DRAM中row-major数据布局的不连续访问。

​	为了减少内存访问次数并增加有效内存带宽，我们将卷积核权重排列为连续块，以确保高效利用外部存储器的带宽。（参考论文：Going Deeper with Embedded FPGA Platform for Convolutional Neural Network ）



**关于卷积运算的并行加速：**

卷积层利用输入和输出并行来进行加速计算（参考论文：An Automatic RTL Compiler for High-Throughput FPGA Implementation of Diverse Deep Convolutional Neural Networks）

通过设计多个并行乘法单元并添加树来实现卷积计算中的输入并行性（Tn并行性）和输出并行性（Tm并行性）。

Tm * Tn乘法单元是并行计算的。Log2（Tn）深度的添加树由管道累积，并生成部分和。



**关于乒乓操作：** 

此设计实现了乒乓缓冲区，以重叠读取输入特征图、权重参数和输出特征图，极大得提高了计算引擎的动态利用率。













**SOC架构图：** 

![](/vivado_bd2.jpg)



**性能评估：**

**硬件资源消耗：** 

实验表明，HLS中的浮点加法需要3个DSP资源，浮点乘法需要2个DSP资源。16位定点乘法需要1个DSP，16位定点加法只能使用LUT实现。

此设计经过分配之后，采用16位定点（Tn = 2，Tm = 32，Tr = 26，Tc = 26）消耗的资源为：

| Resource        | DSP      | BRAM    | LUT        | FF         | Freq   |
| --------------- | -------- | ------- | ---------- | ---------- | ------ |
| Fixed-16(n2m32) | 153(69%) | 88(63%) | 35977(68%) | 36247(34%) | 150MHz |

在此设计中进一步降低了DSP（有许多位宽冗余乘法）和BRAM的消耗。（参考论文Shen[1]，许多BRAM是多余的）

**整个设计的性能：** 



| Performance                                                  |          |
| ------------------------------------------------------------ | -------- |
| CNN models                                                   | YOLO v2  |
| Board                                                        | PYNQ     |
| Clock(MHz)                                                   | 150      |
| Precision                                                    | Fixed-16 |
| Power (W)                                                    | 2.98     |
| Operations (GOP)      //衡量处理器计算能力的指标单位         | 29.47    |
| Performance(GOP/s)    //giga operations per second，每秒十亿次运算数 | 25.98    |
| Power Efficiency(GOP/s/W)                                    | 4.20     |

执行结果：

![](/yolo执行结果.png)





​    

### （3）参考文献

[1] Maximizing CNN Accelerator Efficiency Through Resource Partitioning
[2] PLACID: A Platform for FPGA-Based Accelerator Creation for DCNNs
[3] Going Deeper with Embedded FPGA Platform for Convolutional Neural Network
[4] DianNao A Small-Footprint High-Throughput Accelerator for Ubiquitous Machine-Learning
[5] **An Automatic RTL Compiler for High-Throughput FPGA Implementation of Diverse Deep Convolutional Neural Networks** 
[6] A Dynamic Multi-precision Fixed-Point Data Quantization Strategy for Convolutional Neural Network
[7] Caffeine: Towards Uniformed Representation and Acceleration for Deep Convolutional Neural Networks
[8] Optimizing FPGA-based Accelerator Design for Deep Convolutional Neural Networks









































