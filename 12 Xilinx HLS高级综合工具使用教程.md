---
typora-root-url: D:\typora图片总
---

# 09 Xilinx HLS高级综合工具使用教程

## 九、Xilinx HLS高级综合工具使用教程

### 1、HLS简介

​	vivado-HLS可以实现直接使用 C，C++ 以及 System C 语言对Xilinx的FPGA器件进行编程。用户无需手动创建 RTL，通过高层次综合生成HDL级的IP核，从而加速IP创建。例如：神经网络的卷积层，用HDL语言实现是较复杂的，而用C代码描述是相对较简单的。用户用高层次语言将卷积层描述后，HLS工具再完成从高层次语言到HDL语言的转化。用户不用直面HDL级设计，提高了开发效率。同时，工具里的一些优化工具可以优化RTL的资源消耗和数据吞吐速率。

HLS的官方文档为：**ug871-vivado-high-level-synthesis-tutorial**，所用代码来自于Xilinx官方例程：**ug871-design-files\2016.1\Introduction\lab1**

**HLS的开发阶段：**

![](/HLS开发阶段.png)

Scheduling（时序安排）和 Control Logic Extraction（逻辑控制）：解决每个时钟周期完成什么操作，不同的操作放在哪个时钟周期执行

Binding：负责将代码映射到硬件

![](/2.4.png)

![](/2.5.png)

![](/2.6.png)

**传统RTL开发流程为：**

![](/../typora%E5%9B%BE%E7%89%87%E6%80%BB/3.1.png)

**HLS开发流程：**

![](/../typora%E5%9B%BE%E7%89%87%E6%80%BB/3.2.png)

此时关注的指标为：**Latency，Timing，Resource**

**使用HLS生成IP并使用：**

![](/../typora%E5%9B%BE%E7%89%87%E6%80%BB/3.3.png)



### 2、使用HLS开发流程

![](/3.4.png)

**首先添加source文件和test bench文件**

![](/3.5.png)

:one:  C仿真：此操作会在solution下生成csim文件

:two:  C综合：此操作会生成综合报告，同时在solution下会生成syn文件，其中包括：report,system c,verilog,vhdl

**注：在综合前，可以对不同的solution添加不同的directive来进行优化**

:three:   C/RTL 联合仿真：生成simulation报告，在solution下生成sim文件，此时可以打开vivado查看仿真波形

:four:    输出RTL实现结果，在solution下生成impl文件

**总流程图：** 

![](/4.1.png)



### 3、数据类型和数据操作



### 4、基于C++的运算符



### 5、C test bench



### 6、关于综合Synthesis



### 7、关于数组的实现Implement



### 8、对for循环的优化



### 9、对于数组的优化



### 10、对函数的优化





### 11、实例演示（矩阵乘法）

工程名为：hls_test

:one:  新建工程，一路next，然后选择芯片型号和period：

这里芯片型号选择为：xc7z020clg400-1，Period选择为10ns，即此设计需要工作在100MHz下

 ![](/hls_test1.png)

:two:  右键source，新建xxx.cpp文件（这里为Matrix_Mul.cpp）

```c++
void Matrix_Mul(float A[4][4],float B[4][4],float C[4][4])
{
	for (int i=0; i<4; i++)
		for (int j=0; j<4;j++)
		{
			C[i][j]=0;
			for (int k=0; k<4; k++)
			{
				C[i][j]+=A[i][k]*B[k][j];
			}
		}
}
```

以上代码存在问题，因为c语言中的变量没有输入输出的概念，而C[i] [j]+=A[i] [k]*B[k] [j] 语句对C变量进行了读和写操作，在综合的时候，容易将C变量综合成inout接口，**即尽量不要直接对变量进行操作**。 

```c++
void Matrix_Mul(float A[4][4],float B[4][4],float C[4][4])
{
	for (int i=0; i<4; i++)
		for (int j=0; j<4;j++)
		{
			double tp;
			for (int k=0; k<4; k++)
			{

				tp+=A[i][k]*B[k][j];
			}
			C[i][j]=tp;
		}
}
//此时相当于在硬件映射中添加了一个寄存器，在综合报告中可以看到，可以使latency减少
```



编写完成后，记得保存代码。

:three:  右键solution1-->选择solution settings-->这里选择默认，不做改动，直接默认确定退出。（可以改synthesis等设置）

:four:  右键工程名-->选择Project Settings-->Synthesis-->Top Function，选择要综合的代码

![](/hls_test2.png)

:five:  Run C Synthesis ![](/hls_test3.png)

:six:  分析综合报告：

![](/hls_test4.png)

开始设置的Period=10ns，表示设定电路的最长路径延时为10ns，这里评估的路径延时为8.09ns，满足要求

![](/hls_test5.png)

Latency表示，执行完整个运算需要多少个**时钟周期**    

![](/hls_test6.png)

Utilization Estimates是对消耗的硬件资源进行评估

![](/hls_test7.png)

Interface是生成的硬件接口。

:seven:  给设计添加directive，对电路进行优化

![](/hls_test8.png)

**1、对for循环进行优化**

右键for Statement-->Insert Directive，如下图：

![](/hls_test9.png)

首先选择Directive选项中的UNROLL，将循环展开；

对于Destination：

- 若选择Source File，则会将directive添加到源码中（建议选择此种方式）；
- 若选择Directive File，则会单独生成一个关于directive的文件

设置完成后，重新进行C综合，可以看到latency有明显得减少。

![](/hls_test10.png)

将上一步生成的directive语句复制到其他两个循环下面，可以对其余两个循环也进行优化

```c++
void Matrix_Mul(float A[4][4],float B[4][4],float C[4][4])
{
	for (int i=0; i<4; i++)
	{
		#pragma HLS UNROLL
		for (int j=0; j<4;j++)
		{
			#pragma HLS UNROLL
			double tp;
			for (int k=0; k<4; k++)
			{
				#pragma HLS UNROLL
				tp+=A[i][k]*B[k][j];
			}
			C[i][j]=tp;
		}
	}
}
```

![](/hls_test11.png) ![](/hls_test12.png)

**2、对函数参数位宽优化**

除了展开循环可以加快电路的执行速度，给输入快速赋值也可以，即给函数的参数添加directive

由上面右图可以看到，A,B,C的数据位宽为32位，可以对其进行增加

在Directive界面，右键A-->Insert Directive，Directive选择ARRAY RESHAPE；

对于dimension选项，可以指定展开哪个维度，若选择1，则展开二维数组的第一维；若选择2，则展开第二维。

![](/hls_test13.png)

这里将所有参数的所有维度全部展开：

```c++
void Matrix_Mul(float A[4][4],float B[4][4],float C[4][4])
{
#pragma HLS ARRAY_RESHAPE variable=C dim=1
#pragma HLS ARRAY_RESHAPE variable=B dim=1
#pragma HLS ARRAY_RESHAPE variable=A dim=1
#pragma HLS ARRAY_RESHAPE variable=C dim=2
#pragma HLS ARRAY_RESHAPE variable=B dim=2
#pragma HLS ARRAY_RESHAPE variable=A dim=2
	for (int i=0; i<4; i++)
	{
		#pragma HLS UNROLL
		for (int j=0; j<4;j++)
		{
			#pragma HLS UNROLL
			double tp;
			for (int k=0; k<4; k++)
			{
				#pragma HLS UNROLL
				tp+=A[i][k]*B[k][j];
			}
			C[i][j]=tp;
		}
	}
}

```

重新进行综合：

![](/hls_test14.png) ![](/hls_test15.png)

可以看到，latency减少不多，这可能是设置的时钟周期太短，频率太高，此时可以尝试增加时钟周期；

对于A，B，C三个变量，其数据位宽为512位，说明数组整个展开了，一个数组的512个数可以同时赋值

**3、对函数名进行优化**

对函数名添加directive，实际上是改变**函数的启停工作方式**。 

当不添加directive时，函数的启停由start、done等信号进行控制，此时需要引出GPIO来对函数进行控制。

添加diorective之后，可以将函数挂载到AXI总线上，即可以通过CPU对函数进行控制。

具体实现方法：右键函数名-->Insert Directive，在Directive选项中选择**INTERFACE** ，

然后在Options-->mode中选择**s_axilite**

**4、通过AXI总线配置数据**

从操作可以让CPU通过AXI总线对函数进行赋值。

先将对函数参数位宽的优化删除，右键A,B,C（按住CTRL键可以同时选中多个），Directive选项选择INTERFACE，Option选项下的mode选择**m_axi**，offset选择**slave** （目的是为了使CPU可以通过s_axilite总线向卷积和池化单元配置数据存储的地址）。

![](/hls_test16.png)

可以看到，生成了AXI总线接口。

 











