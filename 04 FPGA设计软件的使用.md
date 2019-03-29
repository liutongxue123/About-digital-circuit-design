---
typora-root-url: D:\typora图片总
---

# 04 FPGA设计软件的使用

## 一、vivado软件使用

​          Vivado 是Xilinx 公司于2012 推出的新一代集成设计环境，随着FPGA进入28nm时代，ISE开发环境稍显落后，所以Xilinx公司从2008开始筹划，历经4年推出了vivado开发环境。它能显著增加Xilinx的28nm 工艺的可编程逻辑器件的设计、综合与实现效率。（28nm是指集成电路工艺光刻所能达到的最小线条宽度 ，一般指半导体器件的最小尺寸，如MOS管沟道长度）。vivado目前只支持Xilinx的28nm工艺的7系列FPGA，包括ZYNQ、Virtex-7系列、Kintex-7 系列和Artix-7 系列，不支持其它系列的FPGA。

### 1、vivado中使用tcl脚本



==待编辑整理==  

https://blog.csdn.net/qq_31811537/article/details/71813077

http://blog.chinaaet.com/greatcause/p/5100051589



### 2、vivado hls中使用tcl脚本

​	此例程基于UG871，展示了如何基于已存在的Vivado HLS工程来创建一个TCL脚本命令和如何应用TCL接口(已存在的工程为"lab1"，使用TCL新建立的工程为“lab2”)

:one:   当创建一个vivado HLS工程时，TCL文件自动保存在项目层次结构中。在打开lab1用户界面中，打开solution1中Constraints文件夹，双击script.tcl文件，相关信息在信息窗口中呈现。

（1）script.tcl文件包含了tcl命令，这命令是为了创建项目在设定和运行综合过程中指定的文件项目。

（2）directives.tcl文件包含了一些应用在设计的优化项。

![](/hls_tcl1.png)

​	在这个实验练习中，lab1是一个已经建立的完善的工程（包含.c文件、.h文件、tb文件、和用来给.tb文件输出做对比的正确数据文件.dat），我们可以采用lab1的script.tcl文件（在lab1工程建立时自动生成的文件），来为lab2创建一个tcl文件。

:two:   在windows系统中，采用“Start>All Programs>Xilinx Design Tools>Vivado2014.2>Vivado HLS>Vivado HLS 2014.2 command prompt”打开HLS命令提示符（可能会存在一个问题，即当命令行窗口的默认路径不是要求的路径。此时，可以手动更改，再开始菜单找到“命令行提示符”右键属性，在“快捷方式”一栏中可以更改初始路径）。

![](/hls_tcl2.jpg)

:three:   在vivado HLS命令提示符中，采用以下命令为lab2创建新的tcl文件

（1）将目录更改为lab1工程所在的目录：C:\Vivado_HLS_Tutorial\Introduction

（2）用命令cp lab1\fir_pri\solution1\script.tcl lab2\run_hls.tcl 来拷贝存在tcl文件到lab2（在Windows命令提示符下支

持使用Tab键自动完成：按tab键多次看到新的选择），此时在lab2工程目录下已经出现run_hls.tcl文件，内容和lab1工程

目录中的script.tcl文件一样。

（3）用命令cd lab2 改变成lab2的目录：

![](/hls_tcl3.jpg)

（4）使用任何文本编辑器（比如notepad++），编辑在lab2目录中的文件run_hls.tcl。编辑如下：

![](/hls_tcl4.png)

具体修改如下：

- 添加-reset选项的open_project命令。因为你通常反复在同一个项目中运行Tcl的文件，最好是覆盖任何现有的项目信息。


- 添加-reset选项的open_solution命令。这消除了当Tcl的文件在同一解决方案中重新运行时任何现有解决方案的信息。


- 删除源命令，如果在以前的项目中包含您希望重新使用的任何指令，你可以从该项目中复制directives.tcl文件到本地路径，也可以直接复制指令到该文件中。


- 添加退出命令。

:four:   用tcl文件在批处理模式下运行vivado HLS（在不启动HLS软件的GUI界面的情况下）。

在vivado HLS命令提示符窗口中（目录精确到\lab2），键入**vivado_hls -f run_hls.tcl** 。

Vivado HLS执行建立工程所有的步骤（C仿真、C综合、C和RTL验证、封装IP）。完成后，结果可在lab2工程目录的fir_prj

目录内使用。

- 综合报告在fir_prj\solution1\ syn\report是可用的。


- 仿真结果在fir_prj\solution\sim\report是可用的。


- 输出包在fir_prj\solution1\impl\ IP是可用的。


- 最终输出的RTL在fir_prj\\solution1\impl，然后Verilog或VHDL是可用的。

:five:   补充

（1）在一个完整工程目录下，可以通过命令行窗口启动工程（打开Vivado HLS的GUI界面）。在本文的例子中，命令行窗

口键入红色字体C:\Vivado_HLS_Tutorial\Introduction\lab2>**vivado_hls -p fir_prj**，之后则会在GUI界面中启动lab2工程

（lab2本身也是由命令行窗口创建的工程）；

（2）当从Vivado HLS项目中复制RTL结果，必须使用在impl目录中的RTL

（3）使用浮点运算符或AXI4接口的设计中，在syn目录中的RTL文件是仅由综合输出的。需要经过HLS软件进行额外的处理

（export_design），在可以使用此RTL在其他设计工具中执行之前。





### 3、使用制定的代码编辑器（以Notepad++为例）

- （对于vivado 2016.4）选择Tools->Options->Text Editor->Current Editor

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/vivado%E4%BD%BF%E7%94%A81.png)

- 然后先选择Notepad++，查看其命令：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/vivado%E4%BD%BF%E7%94%A82.png)

- 然后选择Custom Editor，在上一步命令的基础上，再加上Notepad++的安装路径即可

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/vivado%E4%BD%BF%E7%94%A83.png)

- 使用别的编辑器的步骤同上











