---
typora-root-url: D:\typora图片总
---

# 08 ZYNQ详解

## 五、ZYNQ详解

### 1、7系列FPGA简介

​	在7系列芯片以前，Xilinx的两个最重要的产品分别是面向高性能应用场景下的${Virtex}^{TM}$ 系列和面向低成本低功耗应用场景下的${Spartan}^{TM}$ 系列。这两个系列从芯片内部的布局布线、时钟管理单元，到内部的BRAM等硬件模块，都有着明显的不同。开发人员在不同平台间切换时，因为硬件模块的定义不同，往往需要做一定的代码修改，从而减慢了项目开发的速度。因此在最新的7系列芯片中，Xilinx采用了统一的架构，FPGA内硬件例化模块都采用统一的定义，以帮助用户快速完成设计的迁移。7系列FPGA采用了最新的28nm工艺，其应用范围更是涵盖了所有的系统要求，从低功耗、小型化、成本敏感、大批量应用到超高连接带宽、逻辑能力强和高性能，各种应用需要的信号处理能力其都具备。

**Xilinx 7系列FPGA包括：**

- ${Artix}^{TM}$-7系列：为最低的成本和功耗做了优化，为大批量应用的小型化封装设计。
- ${Kintex}^{TM}$-7系列：为最大的性价比做了优化，和前一代相比，提高了一倍的性能。
- ${Virtex}^{TM}$-7系列：为最高的系统性能做了优化，通过硅片堆叠技术（SSI）提供高性能、高容量的芯片。

**根据以上的系列分布，可以看出两个显著的变化：** 

- 低功耗、低成本的产品被称为Artix，而不是原来的Spartan；
- 在原来高性能和低成本的产品中间，增加了实现最大性价比的Kintex产品线。

**7系列FPGA的主要特征：**

- 基于6输入查找表（LUT）技术的先进的高性能FPGA逻辑，并且可配置为分布式存储器。
- 带有内建FIFO的36Kb双端口BRAM，可作为片内数据缓存。
- 支持速度高达 1866Mb/s 的 DDR3 接口的高性能${SelectIO}^{TM}$技术。
- 内建多个吉比特（Gb）收发器的高速串行连接器，速度从600Mb/s到最高28.05Gb/s，提供低功耗模式，优化了芯片到芯片的接口。
- 用户可配置的模拟接口（XADC），包括两个12位1MSPS的ADC和片上温度、电压传感器。
- DSP Slice包括25x18的乘法器、48位累加器和预加器，可用于高性能滤波，其中包括优化的对称滤波。
- 强大的时钟管理模块CMT ，连接相位锁相环PLL和混合模式时钟管理器 MMCM ，可用于高精度和低抖动的应用。
- 集成了用于x8 Gen3端点和根端口设计的PCIe硬核。
- 丰富的可配置选项，包括支持商用存储器 ，带有 HMAC/SHA-256 授权的256位AES加密及内建的SEU检测和纠错单元。
- 高性能低功耗的 28nm工艺。HKMG、HPL 处理 ，1.0 v 核心电压处理技术和 0.9V 核心电压选项用于更低的功耗应用。

7系列FPGA和Zynq AP SoC平台具有千丝万缕的联系。Zynq 内部的FPGA部分，就是基于7系列的FPGA 。

**两个面向低端应用的Zynq芯片Zynq-7010和Zynq-7020使用Artix-7，两个面向高端应用的Zynq芯片Zynq-7030和Zynq-7045使用Kintex-7。**

| 7系列FPGA资源对比                                      |                                      |                                                           |                                 |
| ------------------------------------------------------ | ------------------------------------ | --------------------------------------------------------- | ------------------------------- |
|                                                        | Arix-7系列                           | Kintex-7系列                                              | Virtex-7系列                    |
| Logic Cells/逻辑单元                                   | 215K                                 | 478K                                                      | 1955K                           |
| Block RAM/BRAM块                                       | 13Mb                                 | 34Mb                                                      | 68Mb                            |
| DSP Slices                                             | 740                                  | 1920                                                      | 3600                            |
| Peak DSP Performance/DSP性能峰值                       | 929GMAC/s                            | 284GMAC/s                                                 | 5335GMAC/s                      |
| Transceivers/串行收发器                                | 16                                   | 32                                                        | 96                              |
| Peak Transceiver Speed/串行收发器峰值                  | 6.6Gb/s                              | 12.5Gb/s                                                  | 28.05Gb/s                       |
| Peak Serial Bandwith（Full Duplex）/串行收发器带宽峰值 | 211Gb/s                              | 800Gb/s                                                   | 2784Gb/s                        |
| PCIe Interface/PCIe接口                                | x4 Gen2                              | x8 Gen2                                                   | x8 Gen3                         |
| Memory Interface/存储器接口                            | 1066Mb/s                             | 1866Mb/s                                                  | 1866Mb/s                        |
| I/O Pins/I/O引脚数量                                   | 500                                  | 500                                                       | 1200                            |
| I/O Voltage/I/O电压范围                                | 1.2V,1.35V,1.5V, 1.8V,2.5V,3.3V      | 1.2V,1.35V,1.5V, 1.8V,2.5V,3.3V                           | 1.2V,1.35V,1.5V, 1.8V,2.5V,3.3V |
| Package Options/封装选择                               | Low-Cost,Wire-Bond,Lidless Flip-Chip | Low-Cost,Lidless Flip-Chip and High-Performance Flip-Chip | Highest Performance Flip-Chip   |



**7系列关键技术：**

**（1）硅片堆叠技术（SSI）**

​	要设计出高容量的FPGA ，是有很多挑战的 ，Xilinx 的解决方案是使用 SSI 技术。SSI技术允许使用多个超级逻辑区域（SLR） 组成被动式内插层 （ Passive Interposer Layer） ，使用经过行业领导者验证的制造和装配技术。在单 FPGA 上集成了多达 10000 个 SLR 连接 ，提供具有低延迟和低功耗的超高带宽。

**（2）逻辑资源（CLB，Slice和LUT）**

​	7 系列 FPGA 的LUT ( Look Up Table ，查找表） 可以被配置为一个6 输入 1 输出的 LUT，或者两个5输入具有相同地址或逻辑输入的独立输出的LUT 。每一个 LUT的输出可以有选择性地在 FF( Flip-Flop）寄存。4 个这样的LUT 和与之相关的8 个FF再加上一些多路选择器和算术进位逻辑组成了Slice ，两个Slice组成了CLB 。25%-50% 的Slice可以用LUT作为分布式64 位 RAM或者32位移位寄存器 。

​                                                  ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/LCs.png) 

​                                      ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/slices.png)

CLB的一些关键特性如下：

- 真正6输入LUT
- LUT具有存储功能
- 具有寄存器和移位寄存器功能
- 详细的CLB配置情况，可以参考**UG474**

**（3）时钟管理**

​	7系列FPGA拥有多达24个时钟管理通道（CMT），每个CMT都由一个混合模式时钟管理器（ MMCM ） 和一个锁相环 （ PLL ） 组成。7系列FPGA提供六种不同类型的时钟线（BUFG 、BUFR 、BUFIO 、BUFH、BUFMR和高性能时钟）来解决不同的需求，包括高扇出 、短传输延迟和低抖动等需求。这一部分是 FPGA 的关键部分，可参考官方的**UG472**。 

**（4）BRAM（块存储器）**

​	大多数的FPGA都具有内嵌的BRAM，这加强了FPGA的灵活性。BRAM可用于片内数据缓冲 、FIFO缓冲等。7系列的FPGA提供了30-1880个双端口BRAM，每一个存储36Kb的数据。每个BRAM都有两个共享数据的独立端口。

BRAM的特性包括 ：

- 双端口36Kb的BRAM，最大位宽是72位
- 可编程的FIFO逻辑
- 内建可选的纠错电路

**（5）DSP Slice**

7系列的FPGA集成了专用的、充分定制的、低功耗的DSP Slice。其增强的功能主要包括：

- 25x18的补码乘法器/累加器，高分辨率48位的信号处理器
- 用于对称滤波器应用的低功耗预加法器
- 一些高级特性（可选的流水线、可选ALU和用于级联的专用总线）

**（6）I/O块**

​	7系列的FPGA的I/O块使用不同的封装 ，满足物理和逻辑级别上不同的要求。物理级上，I/O块支持一个范围内的驱动电压和驱动强度 ，以及接收功能接口的不同I/O标准。7系列的FPGA有高性能HP和高范围HR两种类型I/O。逻辑级上，所有的I/O都能被配置为组合或者寄存方式，都支持DDR模式。

**（7）低功耗吉比特收发器**

7系列的FPGA提供的吉比特收发器具有以下特征：

- 提供高达6.6Gb/s（GTP）、12.5Gb/s（GTX）、13.1Gb/s（GTH）或28.05Gb/s（GTZ）线速率，是业内第一个实现400G数据吞吐的单芯片。
- 支持芯片到芯片接口的低功耗模式。
- 支持高级预发送、后加重、接收器线性（CTLE）和判决反馈均衡（DFE），包括用于额外余量的自适应均衡。

**（8）集成PCIe模块**

7系列的FPGA集成的PCIe模块包括如下重要的特性：

- 兼容PCIe 2.1和3.0基本规范，提供端点和根端口的能力
- 根据芯片类别，支持Gen1 ( 2.5Gb/s ) , Gen2 ( 5Gb/s） 和 Gen3(8Gb/s）。
- 支持高级配置选项 、高级错误报告（AER ) （包括端到端的（ ECRC ） 高级错误报告）和ECRC特性。
- 具有多重功能和单个启动I/O虚拟化（SR-IOV）支持

**（9）XADC模块**

​	全部的Xilinx 7系列FGPA都集成了新的灵活模拟接口 XADC。XADC结合了7 系列FPGA的可编程能力，满足板级数据采集和监控的需求。这种独特的组合被称为敏捷的混合信号的模拟和可编程逻辑。XADC模块的主要特性包括：

- 双12位 1 MSPS模拟到数字转换（ADC）
- 高达17个灵活的用户可配置逻辑输入
- 可选片内或者片外参考电压
- 片内温度（最大误差上下4℃）和电压（最大误差上下1℃）传感器



### 2、Zynq-7000 AP SoC体系简介

 **SoC**：单个硅芯片就可以用来实现整个系统的功能，而不是需要通过介个不同的物理芯片来实现。通常一块SoC上可以包括数字、模拟、数模混合的元件。通常是指的基于ASIC的SoC,其缺点是：（一）开发时间和成本；（二）缺乏灵活性

 **APSoC（全可编程SoC）**:由ARM核构成的处理系统和一个等价于一片FPGA的可编程逻辑（PL）部分，它还具有集成的存储器、各种外设和高速通信接口。

**PL部分**：实现高速逻辑、算术和数据子系统

**PS部分**：支持软件和操作系统



**Zynq-7000系列芯片资源对比：**

|                 | Z-7010 | Z-7015 | Z-7020 | Z-7030 | Z-7035 | Z-7045 | Z-7100 |
| --------------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| 逻辑单元（K）   | 28     | 74     | 85     | 125    | 275    | 350    | 444    |
| Block RAM（Mb） | 2.1    | 3.3    | 4.9    | 9.3    | 17.6   | 19.1   | 26.5   |
| DSP Slice       | 80     | 160    | 220    | 400    | 900    | 900    | 2020   |
| 最大I/O引脚     | 100    | 150    | 200    | 250    | 362    | 362    | 400    |
| 最大收发器数量  | -      | 4      | -      | 4      | 16     | 16     | 16     |



### 3、Zynq-7000系列开发板资源

#### （1）Zedboard开发板



![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/zedboard%E6%A1%86%E5%9B%BE.png)

#### （2）ZYBO开发板

**1、开发板示意图：** 

![](D:/typora图片总/ZYBO%E5%BC%80%E5%8F%91%E6%9D%BF%E7%A4%BA%E6%84%8F%E5%9B%BE.jpg)

![](D:/typora图片总/ZYBO%E8%B5%84%E6%BA%90.png)

**2、ZYBO开发板板载资源说明：**

- 512MB 大小的 DDR3 内存，1066Mbps 带宽
- 双向 HDMI 端口
- 16 位VGA 输出
- 6 个按键开关 (2 个连接至PS—MIO 按钮)
- 4 个拨码开关
- 1G 以太网PHY
- 外部 EEPROM（出厂时该EEPROM 里会预配置1 个48 位的EUI-48TM 以及兼容EUI-64TM 的ID 号）
- micro SD 卡槽
- 音频编解码器（耳机输出、麦克风及音频线输入插口）
- USB OTG 2.0 接口
- USB JTAG 和 USB-UART 共享接口(PROG-UART 接口)
- 128Mb Quad-SPI 闪存
- 5 个Pmod 接口 (1 个PS 专用——MIO Pmod, 1 个为数字/模拟复用口——XADC Pmod)

**3、开发板主芯片为Z-7010，其基本特征为：**

- 667Mhz 双核 Cortex-A9 处理器
- 8 通道DMA DDR3 存储控制器
- 高速外设控制器: 1G Ethernet, USB 2.0, SDIO
- 低速外设控制器: SPI, UART, CAN, I2C
- 可重复编程的逻辑资源：28K logic cells
      240KB Block RAM
      80 DSP slices
      12 位, 1 MSPS XADC

### 4、  zynq启动及配置过程

#### （1）zynq启动过程简介

​	Zynq-7000 AP SoC 器件有效地利用了片上的CPU来帮助配置。 在没有外部JTAG的情况下，处理器系统（PS）与可编程逻辑（PL）都是必须依靠PS 来完成芯片的初始化配置的。

​	配置的控制模块由以下两部分组成：第一部分是**启动存储区（BootROM）** ，这是一块由ARM核上电启动时执行的静态存储区；第二部分是**芯片配置单元** ， 这个模块控制着 JTAG debug 的进入点并为**PL 配置与数据解密**提供AES、HMAC、PCAP 模块的接口。 这也是Zynq的两种启动模式：从BootROM 主动启动， 从JTAG被动启动。

​	无论在是否在安全模式下， PS与PL都由PS端来负责配置。 但是**在有外部主机连接的情况下， 也可以使用 JTAG来进行配置**。 与其他Xilinx 7系列的器件不同的是， Zynq-7000 AP SoC并不支持从此端直接进行启动配置。

​	Zynq-7000 AP SoC 的启动配置是分多级进行的。 配置过程最少需要两步， 但通常是按如下三个阶段进行的：

​	**阶段0（Stage 0）**：这个阶段简称为 **BootROM**  ，控制着整个芯片的初始化过程 。 放在BootROM中的代码是固化的， 不可修改的， 处理器核在上电或者热启动时自动执行这部分代码。

​	**阶段1( Stage 1）**：第一阶段的启动加载器（First Stage Boot Loader, **FSBL**）， 也可以由用户的代码来控制。

​	**阶段2(Stage 2）**：这一阶段中通常可以是用户的PS端的设计代码， 当然也可以是第二阶段的启动加载器（Second Stage Boot Loader, **SSBL**）。 这个阶段可以完全由用户来控制， 是可选的。

#### （2）外部启动条件

Zynq-7000 AP SoC 器件必须在满足电源、 时钟、 复位等外部条件后才能够成功进行BootROM 的启动。启动条件如下图所示：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E5%A4%96%E9%83%A8%E5%90%AF%E5%8A%A8%E6%9D%A1%E4%BB%B6.png)

##### 1、电源要求

在 Stage 0 **BootROM**  状态时 PS 与 PL 的电源要求如下表所示：

| 启动选项                                                     | 安全性 | PS电源 | PL电源 |
| ------------------------------------------------------------ | ------ | ------ | ------ |
| 由NAND,NOR,SD,or Quad-SPI配置启动（注：安全模式下不允许JTAG配置） | 是     | 需要   | 需要   |
| 由NAND,NOR,SD,or Quad-SPI配置启动                            | 否     | 需要   | 不需要 |
| 由JTAG配置启动                                               | 否     | 需要   | 需要   |

在 Stage FSBL 时 PS 与 PL 都是必须上电的， 因为 PL 在这个阶段将进行配置， 而 PS 将负责配置的过程。

##### 2、时钟要求

​	器件在启动时在拥有了稳定的电压以后， 还必须保证 PS_POR_B 引脚被拉高前， PS_CLK已经拥有稳定的时钟输入。 系统中关于时钟的部分都是由 PS_CLK 输入的， 经过 PLL后所提供的不同频率的时钟。 通常 PS_CLK 采用 33. 3MHz 或者 50MHz 的时钟。

##### 3、复位要求

复位信号中有两个主要的外部复位源将影响 BootROM 的执行

:one:  PS_POR_B (电源复位信号)：这个信号的引脚会一直拉低PS端保持在复位状态，直到外部供电已经处于适合的电压范围内。PS_POR_B信号是由电源模块的供电正常信号产生的。

:two:  PS_SRST_B（系统复位信号）：这个信号的引脚用于系统的强制复位，当PS的电源未达到稳定状态时，PS_SRST_B会一直保持高电平。需要注意的是， 当系统处于电源复位状态时， 系统复位信号不会有效， 它会等待 BootROM 执行完毕， 并触发一个 lock-dowm 事件。

##### 4、启动引脚设置

Zynq-7000 AP SoC 拥有五个模式配置引脚 mode[4: 0 J ， 来管理启动源的选择、 JTAG 模式以及 PLL 通路选择。除此之外， 还拥有两个电压模式引脚 vmode [ 1: OJ， 来管理在多路复用情况下的 I/O banks的电压模式。

模式引脚 MIO[6: 2］与电压模式引脚 MIO[8:7］表示如下意思：
:one:  MIO [2］用于选择 JTAG 模式；
:two:  MIO [5:3］用于选择启动方式；
:three:  MIO [6］用于使能PLL;
:four:  MIO [ 8: 7〕用于配置 I/O bank的电压。
因此 Zynq-7000 AP SoC 在启动前需要配置好这些引脚才能正确的选择启动方式。

#### （3）启动时的三个阶段详解

##### 1、BootRom

###### （1）BootROM的作用

​	上电复位以后， PS 端即开始进行配置。 在不使用 JTAG 的情况下， ARM 将在片上的BootROM 中开始执行代码。 BootROM 中的代码将对 NAND、 NOR、 Quad- SPI, SD 与 PCAP 的基本外设控制器进行初始化， 使得 ARM 核可以访问使用这些外设。而 DDR 等其他外设将在 Stage 1或者之后进行初始化。 出于安全考虑， 在 PS 所有主要模块当中， CPU 总是最先启动的。

​	BootROM 中的代码还负责加载 Stage 1的启动镜像。 Zynq-7（则 AP SoC 在硬件层面上支持用户启动镜像的多级加载， 在 Stage 1之后加载的所有镜像都由用户来负责。所以， 当 BootROM将控制权交给了 Stage 1以后， 用户代码将拥有整个系统的控制权。如果想要重新执行 BootROM代码， 只有重启系统这一种办法。

正如前面所提到的， PS 的启动源是由外部模式引脚的高低电平来选择的， 这是指 **BootROM将根据外部配置引脚的设置来从不同的外部存储中加载 Stage 1 的启动镜像**， 当然也支持在线性Flash ( NOR QSPI）上直接运行。 BootROM加载 FSBL 的过程如下图所示：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/BootROM%E5%8A%A0%E8%BD%BDFSBL%E8%BF%87%E7%A8%8B.png)

​	BootROM 将根据外部模式引脚的配置来确定是否使用安全模式来运行。 如果选择了禁用安全模式（通常都禁用安全模式）, BootROM 将关闭所有的安全启动特性， 包括在 PL 中的 AES加密引擎， 然后再进入到 OCM RAM 或者线性Flash中运行用户代码。 如果 Stage 1 使用安全模式启动的， 那之后的用户程序可以自由选择是否使用安全模式继续运行；而如果使用非安全模式启动， 那之后的用户程序就将只能使用非安全模式运行。
​	需要注意的是， PL 的配置并不在 BootROM中完成， BootROM 只为配置 PL 做好准备。

###### **（2）关于BootROM的特点：**

BootROM 是系统启动上电的第一步， 将完成必要外设的初始化， 以及芯片的配置工作。它拥有如下配置特点：

:red_circle:  PS 端的三种不同的配置模式， 包括两种主动模式和一种被动模式：

:one:  安全， 加密镜像的主动模式

:two:  禁用安全模式的主动模式

:three:  基于JTAG禁用安全模式的被动模式

:red_circle:  四种主动启动方式：

:one:   Quad-SPI flash

:two:   NANO flash

:three:   NOR flash

:four:   SD

:red_circle:   采用 AES-256 与 HMAC(SHA-256）加密的安全配置模式

:red_circle:   SoC debug 安全

:red_circle:   在 NOR QSPI 中直接运行

==**总结： 在四种主动启动方式中， CPU 都是主动的从外部非易失性存储中加载启动镜像的。 而JTAG启动只能使用被动启动模式， 外部的计算机将充当主机通过JTAG直接加载启动镜像到OCM 当中， PS 端的 CPU 将保持在空闲状态， 等待启动镜像的加载。**==

###### （3）BootROM后的状态

​	BootROM 的运行时间受到很多因素的影响， 比如电源进入稳定供电的时间、 配置模式的选择、 外部非易失性存储器的使用、 St鸣e I 启动镜像的大小、 加载镜像到 OCM 中所需的时间等。
​	执行完 BootROM 后系统所处的状态依赖于很多方面， 比如是否启用安全模式、 引导配置 引脚的设置、 运行时是否遇到问题。但是通常， CPU 在执行完 BootROM 以后将处于如下状态：

:black_medium_square:   MMU 、数据高速缓存、 指令高速缓存、 L2 高速缓存都处于禁用状态。

:black_medium_square:   所有处理器都处于超级用户模式。

:black_medium_square:   ROM 中的代码是被屏蔽的， 无法访问。

:black_medium_square:   OCM 中的 192KB 空间将从地址 0x0 开始， 另外 64KB 空间将从地址 0xFFFF0000开始。

:black_medium_square:   如果 BootROM 中没有错误发生， CPU0 将会进入 Stage 1 状态。

:black_medium_square:    CPU1 将处于等待事件（Wait For Event, WFE）状态， 并且可执行的代码位于0xFFFFFF00 到0xFFFFFFF0的地址上。

**需要注意的是， 在BootROM 阶段时， 需要拷贝的 FSBL 必须小于192KB， 当 FSBL 开始执行以后OCM 所有的 256KB 空间才全部可用。对于 OCM 的技术细节可以查阅 UG585。**



##### 2、FSBL

First Stage Boot Loader ( FSBL）是在BootROM之后启动的引导程序，由 BootROM 加载到 OCM 或者直接在线性Flash 上运行。 其启动流程如图所示：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/FSBL%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.png)

由 BootROM 加载到 OCM 或者直接在线性Flash 上运行， FSBL主要完成以下几项工作：

:black_medium_square:   根据XPS中的配置， 完成PS端的初始化。

:black_medium_square:   使用比特流文件（Bitstream）对 PL 进行配置。

:black_medium_square:    **加载第二阶段引导程序（Second Stage Boot Loader, SSBL）或者裸跑程序（直接在ARM核上运行的元操作系统的程序）到内存空间。**

:black_medium_square:  跳转执行 SSBL 或者裸跑程序

**注**： FSBL 在跳转到 SSBL 或者裸跑程序之前， 并不使能 MMU。这是因为许多操作系统（例如Linux）假设MMU在启动时是禁用的。

​	Xilinx提供了简单的FSBL 启动代码， 帮助我们完成PS当中必要外设的初始化工作， 当然 我们也可以根据实际的需要进行裁剪， 使用 SDK 工具来查看修改 FSBL 的代码， 观察 FSBL 是如何初始化 CPU 以及外设的。

​	在vivado 中，我们可以通过配置界面对 Zynq 平台进行自定义的配置。在SDK中，可以关注以下文件：

:fu: ps7_init. c 和 ps7 _iniL h， 这些代码用于初始化 CLK, DDR 与MIO模块。

:fu:  ps7_init. tcl， 这个文件的初始化效果与ps7_init. c完全一致。

**注：**

1、在 FSBL 阶段 PL 的配置并不是必需的， 也就是说， 我们可以不配置 PL 部分， 而将 Zynq 平台完全当作两个ARM核来使用。 在 FSBL 阶段， 我们可以根据实际项目的需求来修改FSBL 的代码， 例如在 FSBL 阶段就完成 USB、 STDIO、Ethernet模块的初始化。

2、PL的比特流文件、SSBL、裸跑的应用程序等都必须组织成Flash分区镜像。 FSBL 将会遍历分区头表来寻找比特流文件、 SSBL、裸跑程序所在的位置， 然后进行加载或配置。

下面是裸跑程序的Flash分区镜像结构：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E8%A3%B8%E8%B7%91%E7%A8%8B%E5%BA%8F%E7%9A%84flash%E5%88%86%E5%8C%BA%E9%95%9C%E5%83%8F%E7%BB%93%E6%9E%84.png)



##### 3、SSBL

​	第二阶段的启动加载这一部分是可选的，如果在ARM 核上运行的是一个裸跑的程序， 那么可以省略。 通常对于操作系统而言， 这一阶段是为操作系统的运行做初始化工作的。
​	CPU 可以直接在内存中运行程序， 例如FSBL 就是直接在内存空间中运行的。 而操作系统通常需要更大的存储空间， 例如将硬盘、CD-ROM、 USB Disk、 网络服务器等作为存储载体。**当CPU 上电运行时， 操作系统并不在内存当中，所以通常需要另一个小程序来初始化CPU ，并将操作系统加载到内存当中。 这个小程序称为Bootloder，也就是Zynq平台中的SSBL。** 
​	对于运行在Zynq平台的Linux系统而言，**U-Boot就是SSBL**。U-Boot是一个在Linux社区当中广泛使用的开源的引导程序。Xilinx以往的Power PC、 MicroBlaze处理器都是使用U-Boot来引导Linux的， 因此Zynq中的ARM处理器也不例外。
​	在Zynq平台中，FSBL帮助我们加载U-Boot到RAM 来运行。 U-Boot拥有非常强大的功能，提供了许多用户指令、 读写内存、Flash、USB设备。 **U-Boot会帮助我们完成Linux内核启动之前所必需的硬件初始化， 例如串口、 DDR控制器等， 并且支持从本地、 网络中加载Linux内核、设备树镜像到内存中。之后U-Boot将控制权交给Linux内核， 从这里我们可以看出U-Boot的功能与PC上的BIOS 程序非常相似。** 

#### （4）Linux启动过程

​	嵌人式Linux从软件层面考虑可以分为以下四个部分：**引导程序 （Bootloader）、Linux内核、 文件系统、 应用程序。** 对于Zynq平台来说， 这些部分都是在ARM核上运行的应用程序。

**1、引导程序（Bootloader）:** Zynq平台中的嵌入式Linux引导程序就是我们上一节中介绍的U-Boot。FSBL帮助我们引导U-Boot 到内存当中。U-Boot 将为Linux内核初始化内存， 初始化必要的外设， 设置好启动参数。在Zynq平台中， 我们采用设备树来传递驱动部分的参数， 所以U-Boot还将为内核拷贝设备树( Device Tree） 镜像文件到内存当中。 由U-Boot传递给内核的启动参数中通常包含了设备树的地址， 文件系统的类型、 地址等信息。 而后U-Bool将把系统控制权交给Linux内核。

**2、Linux内核：**当Linux内核拥有系统的控制权后， 将先进行初始化， 建立起内核的运行环境。Linux内核在完成了虚拟地址到物理地址的映射后，还有一个主要的工作就是驱动设备的初始化与挂载。在Zynq平台中， 我们采用设备树的方式传递设备驱动的信息， 如果我们自己为PL 中的IP核编写了驱动程序， 也可以使用模块化的方式， 待Linux启动完成以后再进行挂载。Linux内核将根据U-Boot传递过来的参数信息， 选择挂载文件系统的格式与挂载点。最后Linux内核将运行init（）函数，这是内核引导的终点，也是系统所有进程的起点。

**3、文件系统：**文件系统是一种对存储设备上的数据和元数据进行组织的机制。 文件系统不仅包含着文件中的数据， 还有文件系统的结构。 在Linux操作系统当中， 一切都是文件， 我们可以通过统 一的接口来访问包括文件系统在内的所有数据。文件系统拥有许多格式， Linux也运行同时挂载多个不同结构的文件系统， Linux内核会在启动时挂载文件系统到指定的挂载点上， 之前提到的init（）函数所作的工作当中有一个重要的部分就是读取文件系统/etc/inittab 脚本， 然后根据该脚本中的信息来继续初始化系统的工作。 嵌入式Linux的文件系统大多针对Flash做出优化， 较为常用的有Jffs、yaffs2、ubifs 等。

**4、应用程序：** 这里我们提到的应用程序指的都是存放在文件系统当中、 运行在嵌入式Linux上的应用程序。对于Zynq平台而言， 所有的U-Boot、Linux内核等都是运行在ARM上的应用程序。 对于运行在Linux上的应用程序， 我们可以通过修改文件系统当中的启动脚本来实现开机自启动。也可以在嵌人式Linux启动完成以后， 通过终端来控制应用程序的启动。

 Zynq平台中U-Boot、Linux内核等文件都需要组织成镜像分区的方式进行启动，下图是Linux启动镜像的分区结构：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/linux%E5%90%AF%E5%8A%A8%E9%95%9C%E5%83%8F%E5%88%86%E5%8C%BA%E7%BB%93%E6%9E%84.png)









### 5、zynq系列开发板启动操作

#### （1）Zedboard开发板

​	通过Zynq芯片的启动过程可以看到上电或者复位后片内处理器先启动，然后根据MODE引脚判断启动方式，Zedboard的启动可以通过QSPI Flash、SD卡或者JTAG接口，如下表：

![](D:/typora图片总/zedboard%E5%90%AF%E5%8A%A8%E5%BC%95%E8%84%9A%E9%85%8D%E7%BD%AE.png)

:one: **从QSPI Flash启动：** 

板载QSPI Flash中预置了一个非常简单的程序，启动过程可以分为：

- 上电后，片上ROM程序执行，初始化后判断从QSPI Flash启动
- 从QSPI Flash拷贝FSBL到片上RAM执行
- FSBL执行，处理器从QSPI Flash读取比特流（bitstream）配置Zynq的PL部分
- PL配置完成后执行，点亮LED

MODE跳线选择为：

![](D:/typora图片总/zedboard%20flash%E5%90%AF%E5%8A%A8.png)

接通Zedboard电源，程序启动，因为是一个非常简单程序，所以启动配置过程非常快，蓝色LED（LD12）变亮说明Zynq芯片配置完成，程序执行后按照（10101010）逻辑点亮用户LED。

![](D:/typora图片总/zedboard%20flash%E5%90%AF%E5%8A%A82.png)

:two:  **从SD卡启动：**

这里假设SD卡中预装的是Linaro系统（ubuntu桌面版）。

首先搭好Zedboard平台，连接12V电源（J20）、USB-UART接口（J14）到计算机、插上SD卡（J12）、VGA（J19）或者HDMI（J9）。

![](D:/typora图片总/zedboard%20SD%E5%90%AF%E5%8A%A81.png)

上图看到跳线J6必须短接，MODE跳线选择SD启动。

![](D:/typora图片总/zedboard%20SD%E5%90%AF%E5%8A%A82.png)

用putty来监视串口，串口参数设为波特率115200、数据位8、停止位1、校验无。

准备好以后，可以接通电源，从SD卡启动过程可以分为：

- 上电后，片上ROM程序执行，初始化后判断从SD卡启动。
- 从SD卡拷贝FSBL到片上RAM执行，FSBL配置FPGA，蓝色LED（LD12）变亮说明配置完成。
- 从SD读取SSBL，开始Uboot过程（启动Linux），启动过程中VGA输出了一个Demo演示图像。
- 串口监视程序会显示Linux启动过程（需要上电前打开putty窗口），启动完成后，板上OLED会显示一个Digilent demo图像。

**注：linux系统简单实用教程**

**1、控制GPIO** 

在/usr/bin目录下有一些脚本文件用来控制或读取一些外设的状态。

脚本read_sw用来读取板上8个开关的状态，在命令行输入read_sw会显示输出开关的状态值（16位进制和10进制）。

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A8.jpg)

脚本write_led用来控制板载8个用户LED灯（LD0~LD7）的显示，输入write_led 后面加一个数值（可以是16位进制或者10位进制数，最大255），例如write_led 0xFF与write_led 255效果一样，都是点亮8个LED。

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A82.jpg)

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A83.jpg)

**2、OLED操作**

系统启动时在OLED有一个默认的Digilent公司logo显示，可以通过脚本unload_oled和load_oled挂载和关闭OLED显示。

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A84.jpg)

**3、网口**

Linux系统启动是初始化了Zedboard上的网口，设定固定IP192.168.1.10。演示系统的网络参数不能修改，因此并不能真正连到互联网远程访问。输入ifconfig查看Zedboard此时网络设置：

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A85.jpg)

可以通过网线连接一台主机，然后修改主机的网络设置：

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A86.jpg)

设置好了，在主机浏览器输入192.168.1.10，可以看到一个Http页面：

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A87.jpg)

前面我们看到Linux启动了ssh服务，这里也可以通过ssh访问Zedboard，打开putty：

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A88.jpg)

确定后进入登陆窗口，用户root，密码root，这样我们就能够访问Zedboard上的Linux了。

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A89.jpg)

现在和串口访问模式一样，也可以输入脚本控制Zedboard上的外设了。

![](D:/typora图片总/zynq%20linux%E5%BA%94%E7%94%A810.jpg)



#### （2）ZYBO开发板

**:one:  ZYBO的3种编程模式配置：**

可以通过选择跳线JP5来配置ZYBO的编程模式：

![](D:/typora图片总/ZYBO%E7%BC%96%E7%A8%8B%E6%A8%A1%E5%BC%8F%E9%85%8D%E7%BD%AE.png)

- **JTAG编程模式：**上电启动时，默认从PROG-UART USB接口编程
- QSPI编程模式：上电启动时，默认从板载QSPI Flash中读取配置文件来编程
- SD卡编程模式：上电启动时，默认从SD卡中读取配置文件来编程

**:two:  ZYBO的2种供电模式：**

可以通过跳线JP7来配置ZYBO的供电模式：

![](D:/typora图片总/ZYBO%E4%BE%9B%E7%94%B5%E6%A8%A1%E5%BC%8F.png)

- **USB供电模式：** 将ZYBO与PC 机通过micro USB 数据线连接(连接到ZYBO 的PROG-UART 接口)。PC机通过USB
  线来给ZYBO 供电。
- **外接电源供电模式：** 使用标配的5V 3A 电源适配器，接到ZYBO的电源接头来供电。

注：ZYBO外接电源接头处(J15)，有“**5V ONLY**”的字样。使用外接电源适配器时，只能使用5V电源。一定不要超过5V，不然会导致板卡损毁。低于5V的话也可能会由于欠压导致板卡损毁。

**:three: ZYBO上电过程：** 

![](D:/typora图片总/ZYBO%E7%94%B5%E6%BA%90%E6%8C%87%E7%A4%BA%E7%81%AF.png)

​	在电源接口(J15)左侧的拨码开关(SW4)是电源开关。在选择供电模式之后，将其由off 拨到on，观察电源状态指示灯
(PGOOD, LD11)。电源状态指示灯亮起，并且稳定(没有闪烁的情况)，说明ZYBO 供电正常。

**注：**ZYBO 的很多接口是不支持热插拔的，所以每次要通过板载接口连接外部设备时(例如通过VGA 接口接显示器，通过USB-OTG 接口接USB 键盘、鼠标等)，请关闭电源后再进行操作。否则很有可能会导致板卡损毁。

**:four:  PROG-UART 接口功能验证：**

 	ZYBO的板载micro-USB接口（PROG-UART接口）集供电、编程于UART功能为一体。该接口需要安装相应的驱动才能正常使用。Vivado 软件工具中已经集成了ZYBO 的PROG-UART 接口的驱动，如果已经安装了Vivado工具(2013.4 或更新的版本)，那么不需要再手动安装驱动。

​	先把ZYBO的PROG-UART端连接到PC机打开电源开关，确认电源供电正常。如果是第一次将ZYBO与PC相连，那么PC 端会自动识别该USB 设备并安装相应驱动。ZYBO的板载PROG-UART 接口集供电、编程与UART 功能为一体，PC 端识别该设备时会有2部分：**一部分是对应UART 功能的COM 端口；另一部分是对应USB 编程功能。**

进入PC 的设备管理器，如下图，“COM38”对应UART 功能；“USB Serial Converter A/B”对应USB 编程功能。

![](D:/typora图片总/ZYBO%E7%AB%AF%E5%8F%A3.png)

**:five: ZYBO启动linux的两种方式：** 

​	ZYBO有两种启动linux的方法，一种是从QSPI Flash中启动**厂家预装的** Open Linux（无桌面显示），另一种是从SD卡（或者为TF卡）启动，下面是两种启动方式的具体步骤：

**[1] 从QSPI启动**

- 设置跳线JP5来将编程模式设置为QSPI模式，设置跳线JP7来选择USB供电模式，连接PC机和PROG-UART串口。
- 上电，ZYBO会自动运行QSPI中预装的Open Linux，zynq芯片被配置完成后，指示灯LD10会亮起，如图：

![](D:/typora图片总/ZYBO%20QSPI%E5%90%AF%E5%8A%A8.png)

- 用putty等终端软件，打开对应的COM口，串口属性设置如图，等linux完全启动后，可以进行和linux系统的交互。

![](D:/typora图片总/ZYBO%20%E4%B8%B2%E5%8F%A3%E9%85%8D%E7%BD%AE.jpg)



**[2] 从SD卡启动**

​	从SD卡中可以启动两种linux系统：一种是digilent官网上开放的Linaro系统（ubuntu桌面版），其可以支持键盘鼠标操作。另一种是Open Linux系统（不显示桌面，只能用命令行的方式进行操作）。

​	从SD卡启动linux系统和从QSPI启动一样，设置跳线JP5来选择SD卡启动模式，设置跳线JP7来选择USB供电模式，使用putty等超级终端软件来打开对应的COM口。

挂载SD卡到linux系统：

**mount dev/mmcblk0p1 /mnt**  mmcblk0p1是SD卡在linux系统下的设备名，此命令是将SD卡挂载到mnt目录下。

**下面是两种linux系统SD卡的制作过程：** 

1、对于Open Linux系统

准备一个micro sd卡，要求是第一个分区是FAT32格式（**可以只有这一个分区** ），将文件夹（**ZYBO_openlinux启动文件** ）下的内容拷贝到SD卡中。

2、对于Linaro系统：

- 找到文件夹（**Linaro文件系统** ）下的**linaro-precise-ubuntu-desktop-20120923-436.tar.gz** 文件和**sd_image.rar**  文件。
- 打开虚拟机下的ubuntu，先将linaro-precise-ubuntu-desktop-20120923-436.tar.gz文件复制到当前用户目录下。
- 将SD卡（TF）连接到电脑，点击ubuntu桌面左上角白色圆形图标，输入disk，ubuntu 系统自动查找与disk 相关程序，点
  击Disk Utility 运行。

![](D:/typora图片总/ubuntu%20SD%E5%88%B6%E4%BD%9C1.jpg)

- 在Disk Utility 左侧列表中找到你的SD卡，USB Mass Storage Device，点击选中。在右侧Volumes 图表中会显示已经存在的分区，如果有已经存在的分区，点击它进行选择，再点Delete Partition 删除分区。当SD卡不再存在分区后，点Create Partition 创建分区。

![](D:/typora图片总/ubuntu%20SD%E5%88%B6%E4%BD%9C2.jpg)

- Size 是新分区大小，创建约SD卡总容量四分之一的分区，分区类型Type 选FAT，分区名称Name 写FAT，点Create 创建新分区。

![](D:/typora图片总/ubuntu%20SD%E5%88%B6%E4%BD%9C3.jpg)

- 继续点Create Partition 创建新分区，Size 使用默认值表示使用全部剩余空间，分区类型Type 为Ext4，分区名称写EXT，点Create 创建分区。

![](D:/typora图片总/ubuntu%20SD%E5%88%B6%E4%BD%9C4.jpg)

- 当是FAT 类型的第一个分区和Ext4 类型的第二个分区创建完成后，点Safe Removal 安全移除SD卡。

![](D:/typora图片总/ubuntu%20SD%E5%88%B6%E4%BD%9C5.jpg)

- 运行ubuntu 命令行，输入**lsblk**，插入 TF 卡后再次输入lsblk，比较两次结果可以知道TF 卡对应的设备名，这里以sdd1、sdd2 为例。  
- 输入**mkdir –p /tmp/linaro** 创建临时文件夹，输入
  **sudo cp linaro-precise-ubuntu-desktop-20120923-436.tar.gz /tmp/linaro/fs.tar.gz** 复制文件到临时文件夹。
- 输入**cd /tmp/linaro/**移动到临时文件夹，输入**sudo tar zxvf fs.tar.gz** 解压缩，输入**mkdir –p /tmp/sd_ext4**
  建立SD卡临时目录，输入**sudo mount /dev/sdd2/tmp/sd_ext4/** 加载SD卡
- 输入**cd binary/boot/filesystem.dir** 进入linaro 文件系统，输入**sudo rsync –a --progress ./ /tmp/sd_ext4** 同步文件系统到SD卡
- 输入**sudo umount /tmp/sd_ext4** 移除SD卡，等SD卡读卡器指示灯停止闪烁后，文件全部写入完成。再安全移除SD卡。
- 在Windows 下解压缩sd_image.rar，把包含的四个文件复制到SD卡的FAT 分区，安全移除TF 卡，这样Linaro Demo 的SD卡就准备好了。

:six:  **在Vivado中建立基于ZYBO开发板的工程**

**在创建工程选择板子时，有两种方式：**

1、选择Parts，然后选择Zynq-7000 系列，**Package 是clg400，Speedgrade 是-1，选择xc7z010clg400-1**

![](D:/typora图片总/ZYBO%20part%E9%80%89%E6%8B%A9.jpg)

对于这种方法，还需要在Block Design中引入ZYNQ IP时，点击Import XPS Settings，找到并选择ZYBO_zynq_def.xml。

2、将board_files文件夹里的内容拷贝到Vivado安装目录下的：D:\xilinx\Vivado\2016.4\data\boards\board_files

（此文件在**vivado-boards-master** 文件下，其中old文件对应于Vivado 14版本之前，new文件对应Vivado15版本之后。文档链接为：https://github.com/liutongxue123/Project-summary/tree/master/vivado-boards-master）

**对ZYNQ IP的一般配置：**

- 进入MIO Configuration，取消USB0、SD0 和I2C0的选择。

![](D:/typora图片总/ZYBO%20ZYNQ%20MIO%E8%AE%BE%E7%BD%AE.jpg)

- 进入PS-PL Configuration,取消 FCLK_RESET0_N

![](D:/typora图片总/ZYBO%20ZYNQ%20IP%20%E8%AE%BE%E7%BD%AE2.jpg)

- 进入Clock Configuration，取消FCLK_CLK0 选择

![](D:/typora图片总/ZYBO%20ZYNQ%20IP%20%E8%AE%BE%E7%BD%AE3.jpg)

- 进入PS-PL Configuration，取消M_AXI_GP0选择

![](D:/typora图片总/ZYBO%20ZYNQ%20IP%20%E8%AE%BE%E7%BD%AE4.jpg)







