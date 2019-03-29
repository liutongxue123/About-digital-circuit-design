---
typora-root-url: D:\typora图片总
---

# 03 FPGA基础

## 三、FPGA基础

### 1、What is FPGA

FPGA的全称为**Field Programmable Gate Array**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Gate%20array.jpg) ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA2.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

**关于programmable可编程：**

| 类型             | 使用次数 | 掉电数据是否丢失 |
| ---------------- | -------- | ---------------- |
| fuse融丝         | 一次性   | 是               |
| Antifuse反融丝   | 一次性   | 是               |
| Flash（如U盘）   | N次用    | 否               |
| SRAM（在FPGA中） | N次用    | 是               |

**SRAM-based FPGA**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/SRAM%20of%20FPGA.png)

**Programmable example**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/programmable%20example.png)

**Early programmable devices**

第一款可编程逻辑器件是PLAs（Programmable Loic Arrays），具有用户可编程连接的AND和OR门的两级结构，现在这种器件通常被称为PLDs（Programmable Logic Devices）。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/PLAs.png)

**Origin of FPGA**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/origin%20of%20FPGA.png)

**FPGA vs CPLD**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%20VS%20CPLD.png)

**FPGA vs CPU**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%20vs%20CPU.png)

**FPGA vs ASIC**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%20vs%20ASIC.png)

**Distinguish**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/distinguish.png)

a. 对于CPU等，其工作的一般流程为：指针命令 $\to$翻译命令$\to$取操作数$\to$处理操作数，一个cpu，每次执行，进行一次运算

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/CPU%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

对于FPGA和ASICs，有专门的加法器和乘法器等单元，可以进行并行计算。

b. 一般GPPs器件的功率在百瓦级，可编程器件和ASICs的功率在瓦级，功率和频率有关，CPU的频率一般是GHz级别，FPGA一般在MHz级别。

c. 对于performance，CPU<FPGA<ASICs

   对于time to market，CPU<FPGA<ASICs

   对于cost，ASICs<FPGA，$70 millions for 45nm ASIC

   对于reliability，FPGA>ASICs、CPU

   对于long-term maintenance，FPGA>ASICs

**FPGA在工程中基本应用**

当10%的代码占用90%运行时间（如for 循环），将这10%的代码放到FPGA中加速。

**FPGA companies**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%20companies.png)

![](/硬件架构图.png)

### 2、FPGA architecture

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%E6%9E%B6%E6%9E%84%E5%9B%BE.png)

​    ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%E6%9E%B6%E6%9E%84%E5%9B%BE2.jpg)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FPGA%E7%A1%AC%E4%BB%B6%E5%9B%BE.jpg)

#### **（1）Programmable logic blocks**

对于Xilinx，称为Configurable Logic Blocks(CLBs)；

对于Altera，称为Logic Array Blocks (LABs)

在Xilinx中，CLB的整体结构为：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E6%9E%B6%E6%9E%84CLB.png)

**a.  对于CLBs：**

它是时序和组合电路的主要逻辑资源，和开关矩阵Switch Matix（SM）相连接，每个CLBs中包含几个slices。

  如在Virtex-2中，每个CLBs中包含4个slices；在Virtex-6中，每个CLBs中包含2个slices

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/CLB%E5%86%85%E9%83%A8.png)

**b.  对于Slices：**

每个slice排布在列上并且含有一个单独的进位链；CLB中的slices从底部被标记。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/slices.png)

​                                          注：图中X表示column列，Y表示row行

**c.  在每个Slices中**

包含若干个Logic Cells（LCs）,如Virtex-2 中有2个, Virtex-6 中有4个。

   LCs的内部主要有4个组件，分别为LUT，MUX，Carry，REG。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LCs.png)

**d.  LCs中的四个组件：**

- **关于LUTs查找表：**

  LUT用来实现组合逻辑，其能力取决于输入的数量，也被叫做Function Generators。

也可以将 FPGA中的LUT灵活地配置成RAM、ROM和FIFO等结构。

**LUT的工作原理：**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT1.png) ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT2.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT3.png)

在上图中，逻辑式$\to$ 真值表$\to$ 查找表

由开发工具，逻辑代码经综合、实现等生成比特流文件，其中存放的是真值表中的数据。烧录时，将真值表数据存到SRAM中，通过LUT逻辑门，得到输出。

注：LUT中没有真正的与或非门等，逻辑都是由这种查找表的方式实现的。

LUT的能力取决于它的输入数量：如一个4输入LUT可以实现任何4输入逻辑功能。

而且可以把LUT混合级联使用：例如，要实现逻辑Y=a&b&c&d

​        可以用3个2-input LUT+12个存储单元（3级）；可以用个3-input LUT+16个存储单元（2级）；可以用1个4-input LUT

​         ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT%E7%BA%A7%E8%81%94.png)

**LUT的其他功能**：

LUT可以被配置为 同步synchronousRAMs、ROM、移位寄存器Shift registers、数据选择器MUX等

  ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT%E5%BA%94%E7%94%A81.png) 

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT%E5%BA%94%E7%94%A82.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT%E5%BA%94%E7%94%A83.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LUT%E5%BA%94%E7%94%A84.png)

- **关于REG**：

  ​     这是LCs中的Storage elements存储单元，其可以作为边沿触发的D型触发器或者电平敏感锁存器，主要用来实现时序逻辑。时钟（CK），使能（CE）和置位/复位（SR）对于一个slice中的所有FF是公共的。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/REG.png)

- **关于Carry Chain**

​      在slice中进行快速加法和减法；

​      用于宽逻辑功能的级联函数发生器;

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/carry%20chain.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Detailed%20slice.png)

​                                                            其中的AX和BX信号，用来级联两个LUT。

#### **（2）Programmable interconnect**

可配置互联资源，共有三部分：

a. CLBs access to channels through **Connection Blocks**

b. Channels are connected by **Switch Blocks**

c. Channels are comprised of **Wires**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/interconnect1.png) ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/interconnect2.png)



一个详细的互联资源的实例：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/interconnect3.png)

注：FC的数量表示CLB和wire的连接点个数；

​      FS的数量表示从一个CLB来的信号，经switch block可以去往多少个地方（一般是3的倍数）；

​      W表示线wire的数量。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/interconnect4.png)

线的长度取决于横跨多少个CLB

段长度会影响FPGA的速度和面积：

​       如果线短，则需要经过更多的switch block，会导致长的latency时延；

​       如果线长，会增加通道的宽度，导致FPGA面积大。

#### **（3）Embedded dedicated blocks**

嵌入式专用单元主要有以下几种：

**a. Block RAMs**

​       FPGA可以调用分布式RAM和块RAM两种RAM，当我们编写verilog代码的时候可以使我们想要的RAM被综合成BRAM（Block RAM）或者分布式RAM（Distributed RAM），其中BRAM是block ram，是存在FPGA中的大容量的RAM，分布式RAM是FPGA中由LUT（look-up table 查找表）组成的。当使用的容量较小会综合成分布式RAM，容量大的时候综合成BRAM。**这两种RAM都属于SRAM器件存储方式，即FPGA可配置**

对于Block RAMs：

实际为双端口；由两个18kb组成36kb；排布在列上；可以被配置成单口RAM，双口RAM，ROM，FIFOs；可以级联成更深的RAM

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Block%20RAMs.png)

**b. DSP blocks**

Block RAMs和DSP blocks就可以实现一个处理器。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/DSP%20blocks1.png)

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/DSP%20blocks2.png)

**c. Multi-Gigabit transceivers**

一般提高数据传输的速率，有两种方法：

一是提高频率，此方法容易产生串扰，解决方法是可以采用差分信号；

二是可以增加线宽（实际上是增加线的数量），这种方法的缺陷是不容易布线。

FPGA中的Multi-Gigabit transceivers可以用来传输串行信号：

例如Virtex-6 GTX: 600Mb/s to 6.6Gb/s

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Multi-Gigabit%20transceivers.png)

**d. Processor cores**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Processor%20cores.png)

FPGA内嵌的处理器可以分成两类：分别是软核和硬核。

- Hard core，有实际硬件

PowerPC （Virtex-2 Pro）

ARM core (Now Zynq-7000)

- Soft core，用LUT等实现，没有实际硬件

MicroBlaze (Xilinx)

Nios® II (Altera)

#### **（4）Clock resources**

- Clock trees drive FFs

​      在FPGA中有多种时钟资源，包括Global clock全局时钟、Region clock区域时钟、IO时钟

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Clock%20trees.png)

- Clock buffers

共有32个global clock buffers，例如：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/clock%20buffers1.png) 此结构可以在两个时钟中选择一个，在实际中可以用于高低性能转换（如变频空调）。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/clock%20buffers2.png) 此结构可以对一个时钟进行使能控制，在实际中可以用来降低功耗。

注：FPGA的功耗计算公式为P=a${V_{dd}}^2$ f，可以看出，功耗与频率有关。

- Digital Clock Manager (DCM)

  ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/DCM_BASE.png)

其功能为：Clock deskew；Frequency synthesis；Phase shifting；Dynamic reconfiguration；Drive region clocks

关于Clock deskew：在下图中，时钟到达两个触发器的时间不同步，则时钟管理器可以解决这种情况。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Clock%20deskew.png)

#### **（5）IO blocks**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/IO%20blocks.png) ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/IO%20blocks2.png)

输入路径：两个DDR寄存器

输出路径：两个DDR寄存器+两个三态使能DDR寄存器

对于输入和输出，分别使用单独的时钟驱动，但置位和复位信号是共享的。

IO资源被聚集到banks中，每个banks被单独配置。

支持差分信号标准：LVDS, BLVDS, ULVDS, LDT, LVPECL

支持单端I/O标准：LVTTL, LVCMOS(3.3V, 2.5V, 1.8V, 1.5V)

​                             PCI-X at 133MHz, PCI(3.3V at 33 MHz, 66MHz)

​                             GTL, GTLP and more…

数字控制阻抗Digital controlled impedance：即与外部的电路自动做阻抗匹配，

​                     其优点为：允许直接连接到不同电压和阈值的外部信号；优化速度/噪声均衡等

**Select devices based on resources**

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/Select%20devices%20based%20on%20resources.png)

### 3、HDL-based FPGA design flow



### 4、Logic synthesis



### 5、FPGA physical implementation



### 6、Post layout timing simulation



### 7、High level language based design flow



### 8、FPGA Reconfiguration



### 9、FPGA Design Case Study





