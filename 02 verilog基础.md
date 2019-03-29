---
typora-root-url: D:\typora图片总
---

# 02 verilog基础

## 二、verilog基础

### 0、命名规则

|                                    |                                       |                                |
| ---------------------------------- | ------------------------------------- | ------------------------------ |
| 程序名称                           | 命名规则                              | 示例                           |
| 顶层文件                           | 对象+功能+top                         | video_oneline_top.v            |
| 逻辑控制文件，介于顶层与驱动层之间 | 对象+ctr                              | ddr_ctr.v                      |
| 驱动程序命名                       | 对象+功能+dri                         | lcd_dri.v、uart_rxd_dri.v      |
| 参数文件命名                       | 对象+para                             | lcd_para.v                     |
| 模块接口命名1                      | 文件名+u                              | lcd_dir_u(........)            |
| 模块接口命名2                      | 特征名+文件名+u                       | c3_mcb_read_u                  |
| 端口注释                           | input Video_vs_i //输入场同步入       |                                |
| 信号命名                           | 命名总体规则：对象+功能（+极性）+特性 |                                |
| 时钟信号                           | 对象+功能+特性                        | phy_txclk_i、sys_50mhz_i       |
| 复位信号                           | 对象+功能+极性+特性                   | phy_rst_n_i、sys_rst_n_i       |
| 特定功能计数器1                    | 对象+cnt                              | line_cnt、div_cnt0、div_cnt1   |
| 特定功能计数器2                    | 对象+功能+cnt                         | video_line_cnt、video_fram_cnt |
| 一般计数器                         | cnt+序号                              | cnt0、cnt1、cnt2               |
| 时序同步信号                       | 对象+功能+特性                        | line_sync_i、fram_sysc_i       |
| 使能信号1                          | 功能+en                               | wr_en、rd_en                   |
| 使能信号2                          | 对象+功能+en                          | fifo_wr_en、mcb_wr_en          |

### 1、技术背景

​       大规模集成电路设计制造技术和数字信号处理技术实质上是紧密相关的。因为数字信号处理系统往往要进行一些复杂的数学运算和数据的处理，并且又有实时响应的要求，它们通常是由高速专用数字逻辑系统或专用数字信号处理器所构成，电路是相当复杂的。因此只有在高速大规模集成电路设计制造技术进步的基础上，才有可能实现真正有意义的实时数字信号处理系统。现代专用集成电路的设计是借助于电子电路设计自动化（EDA）工具完成的。
学习和掌握硬件描述语言（HDL）是使用电子电路设计自动化（EDA）工具的基础。

​       对于单片机，程序代码是一条一条执行的。CPU 先通过总线，读取一条指令，然后解析这条指令，最后执行这条指令。C代码总是一条一条地执行。如果要同时处理多个子程序，那么CPU 只能一个个子程序来执行。对于一些实时性较高的功能，如扫描矩阵键盘，VGA 刷屏，都需要中断来实现。如果刷屏时间比较长，则会影响到按键的灵敏度。另外当用单片机的串口收发数据的同时，还要处理外部IO信号的时候，如果此时的IO信号非常快，并且有几百个信号，可能同一个时刻触发，很显然，此时单片机已经不能在满足这些要求。FPGA由于其良好的并行性，完全可以胜任上述任务。

​       如果并行控制是FPGA的优势，那么顺序执行就是其劣势。为了解决这一问题，通常有两种做法。

- ​       方法一：状态机设计

​       在verilog程序中，状态机无处不在。通过用verilog设计，可以控制状态机的功能和执行的快慢。

- ​       方法二：在FPGA中运行CPU

​       通过这种方法，可以把顺序控制逻辑用C代码来实现，而实时处理的部分仍然用verilog来实现，并且verilog代码部分可以用C代码控制。Xilinx FPGA 目前支持的CPU 有Microblaze，ARM9, POWERPC，CortexA9（zynq完美得将CortexA9 和FPGA 整合到一起）其中Microblaze 是一款软CPU，是软核。ARM9，CortexA9 和 POWERPC 是硬核。其中软核就是用代码实现的CPU核，这种核成本低廉，配置灵活，但是会占用FPGA的资源。硬核就是集成到FPGA内部的一块实际的电路，性能更高，功能更全面。例如Xilinx的DDR内存控制器，就是一种硬核，其运行速度非常高，只需要对其进行相应的配置，就可以方便使用。

### 2、verilog与C对比

关键词和结构对比：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/verilog%E4%B8%8Ec%E5%AF%B9%E6%AF%941.jpg)

运算符对比：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/verilog%E4%B8%8Ec%E5%AF%B9%E6%AF%942.jpg)



### 3、运算符

“;”分号用于每一句代码的结束，以表示结束，和C语言一样。

“：”冒号，用在数组、条件运算符以及case 语句结构中。

“<=”赋值符号，非阻塞赋值，在一个always 模块中，所有语句一起更新。它也可以表示小于等于，具体是什么含义根据当前编程环境判断，如果“<=”是用在一个if 判断里如：if(a <= 10)；当然就表示小于等于了。

“=”阻塞赋值，或者给信号赋值，如果在always 模块中，这条语句被立刻执行。

“+、-、*、/、% ”是加、减、乘、除运算符号，这些使用和C 语言基本是一样的，
当你用到这些符号时，编译后会自动生成或者消耗FPGA 原有的加法器或是乘法器等。
其中符号/、%会消耗大量的逻辑，谨慎使用。

“<”小于，比如A<B 含义就是A 和B 比较，如果A 小于B 就是TURE,否则为FALSE。

“<=”小于等于，比如A<=B 含义就是A 和B比较，如果A小于等于B 就是TURE,
否则为FALSE。

“>”大于，比如A>B 含义就是A 和B 比较，如果A 大于B 就是TURE,否则为FALSE。

“>=”大于等于，比如A>=B 含义就是A 和B比较，如果大于等于B 就是TURE,否则为FALSE。

“\==”等于等于，比如A==B 含义就是A和B比较，如果A 等于B就是TURE,否则为FALSE。

“!=”不等于，A!=B 含义是A 和B 比较，如果A不等于B就是TURE，否则为FALSE。

“>>”右移运算符，比如A>>2 表示把A 右移2位。

“<<”左移运算符，比如A<<2 表示把A左移2位。

“~”按位取反运算符，比如A=8'b1111_0000；则~A 的值为8'b0000_1111。

“&”按位于与，比如A=8'b1111_0000；B=8'b1010_1111；则A&B 结果为8'b1010_0000。

“^”异或运算符，比如A=8'b1111_0000；B=8'b1010_1111；则A^B 结果为8'b0101_1111。

“&&"逻辑与，比如A\==1,B\==2；则A&&B 结果为TRUE；如果A\==1,B==0,则A&&B 结果为FALSE，一般用于条件判断。

A = B ? C : D 是一个条件运算符，含义是如果B 为TRUE 则把C连线A,否则把D连线A。B 通常是个条件判断，用小括弧括起：

例如：assign C1_Clk = (C1==25'd24999999) ? 1 : 0 ;

**“｛｝”在Verilog 中表示拼接符，{a,b}这个的含义是将括号内的数连接在一起，比如：{1001,1110}表示的是10011110**，

拼接是Verilog 相对于其他语言的一大优势



### 4、模块结构

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/verilog%E6%A8%A1%E5%9D%97%E7%BB%93%E6%9E%84%E5%9B%BE.png)

每个Verilog程序主要包括4个主要部分：端口定义、I/O说明、内部信号声明和功能定义

#### **（1）模块的端口定义**

module 模块名（端口1，端口2，端口3，端口4）；

模块的端口名是模块的输入和输出口，是与别的模块联系端口的标识。

引用其他模块有两种方法：

a.   被调用的模块名 自定义模块名 （自定义信号名1，自定义信号名2，自定义信号名3）；

这种方法，引用时严格按照模块定义的端口顺序来连接，此方法不常用

b.  被调用的模块名 自定义模块名 （.被调用模块信号名1（自定义信号名1）, 

​                                                      .被调用模块信号名2（自定义信号名2））；

#### **（2）I/O说明**

输入口： input [ 信号位宽-1：0 ] 端口名 1；

​               input [ 信号位宽-1：0 ] 端口名 2；

输出口： output [ 信号位宽-1：0 ] 端口名 1；

​               output [ 信号位宽-1：0 ] 端口名 2；

输入\输出口：inout [ 信号位宽-1：0 ] 端口名 1；

​                      inout [ 信号位宽-1：0] 端口名 2；

**注：可以将I/O说明写在端口定义语句中**

module 模块名（input 端口1，input 端口2，output 端口3，output 端口4）；

#### **（3）内部信号说明**

在模块内用到的和与端口有关的wire和reg类型变量的声明。

wire [width-1：0]  变量1，变量2；     **input，output和inout均属于wire型**。

reg [width-1：0]  变量1，变量2；     **和线信号不同，它可以在always中被赋值，经常用于时序逻辑中**。如reg[3:0]Led，表示一组寄存器。

#### **（4）功能定义**

模块中最重要的部分是逻辑功能定义部分。有3种方法可在模块中产生逻辑。

**a. 用assign声明语句：**

这个用来给output，inout 以及wire 这些类型进行连线。assign相当于一条连线，将表达式右边的电路直接通过wire(线)连接到左边，左边信号必须是wire 型（output 和inout 属于wire 型）。

当右边变化时左边立马变化，用来描述简单的组合逻辑，例如：

```verilog
wire a , b, y;
assign y = a & b;
```

**b. 用实例元件**

​       采用这种方法就像在电路图输入方式下调入库元件一样，输入元件的名字和相连的引脚即可。

​       例如：and #2 u1(q,a,b);

​       这表示在设计中用到一个跟与门一样的名为u1的与门，其输入为a、b，输出为q。输出延迟为2个单位时间。

**c. 用always@( )语句**

括号里面是敏感信号。

例如，always@(posedge Clk)，敏感信号是posedge Clk，其含义是上升沿有效。

always@( negedge Clk)，其含义是下降沿的时候有效。以上两种形式一般用于时序逻辑。

always@( * )，其表示一直是敏感的，一般用于组合逻辑。

always@（a），只要a信号发生变化，无论是高到低还是低到高，均发生触发。

### 5、参数使用

**（1）参数的有效定义**

用parameter来定义一个标志符代表一个常量，称作符号常量，他可以提高程序的可读性和可维护性。parameter是参数型数据的关键字，在每一个赋值语句的右边都必须是一个常数表达式。即该表达式只能包含数字或先前已经定义的参数。

例如：

parameter     msb=7;                 //定义参数msb=7

parameter     r=5.7;                   //定义r为一个实型参数5.7

parameter     byte_size=8,byte_msb=byte_size-1;        //利用常数表达式赋值

**参数型常量经常用于定义延迟时间和变量宽度。在模块和实例引用时，可以通过参数传递改变在被引用模块或实例中已经定义的参数。**

```verilog
module exam_prj
    #(parameter WIDTH=8) //端口内的参数只能在这使用
    (
        input [WIDTH-1:0] data1,
        input [WIDTH-1:0] data2,
        output reg [WIDTH:0] result
    );
    parameter counter_top = 4'd9;//用于代码部分的参数
```

**这里出现的两个参数parameter，第一个表示只在端口设置时使用，后面的是对于模块内部的使用。**

**（2）参数传递**

一般有三种传递方法：

1、module_name #( parameter1, parameter2) inst_name( port_map);

即 被调用的模块名 #（参数1，参数2）自定义模块名 （.被调用模块信号名1（自定义信号名1）, 

​                                                                                   .被调用模块信号名2（自定义信号名2））；

```verilog
module adder_16(sum,a,b);
    parameter time_delay=5,time_count=10;
              ......
endmodule
module top;
    wire[2:0] a1,b1;
    wire[3:0] a2,b2,sum1;
    wire[4:0] sum2;
    adder_16  #(4,8)  AD1(sum1,a1,b1);//time_delay=4,time_count=8
endmodule
```

2、module_name  #( .parameter_name(para_value), .parameter_name(para_value))  inst_name (port map);

```verilog
module exam_prj
    #(parameter WIDTH=8) //端口内的参数只能在这使用
    (
        input [WIDTH-1:0] data1,
        input [WIDTH-1:0] data2,
        output reg [WIDTH:0] result
    );
    parameter counter_top = 4'd9;//用于代码部分的参数
    
//module exam_prj_tb;
exam_prj #
    (
		.WIDTH(8),  
        .Conuter_Top(4'd5)
    )
    exam_prj_inst//------*注意例化时的名字在这个位置*
    (
		.dataa(dataa),     
        .datab(datab),  
        .result(sum)
    );
```

3、在多层次的模块中，改变参数需要使用defparam命令。     defparam   Test.T.B1.P=2；         //Test、T、B1分别是高层模块中的底层模块实例。参数需要写绝对路径来指定。

**（3）子模块调用子模块**

```verilog
module sub_sub_modu;
    parameter w1=4;
    ...
endmodule
module sub_modu;
    parameter w=8;
    ...
    sub_sub_modu #(w) ( );
    ...
endmodule

module main_modu;
    ...
    sub_modu #(16) ;  //这样w和w1都是16位
    ...
endmodule
```



### 6、数值表示

 如果要表示一个十进制是180 的数值，在Verilog 中的表示方法如下：

```verilog
二进制：8’b1010_1010; //其中“_”是为了容易观察位数，可有可无。
十进制：8’d180;
16 进制：8’hAA;
```

### 7、阻塞和非阻塞赋值

<= 非阻塞赋值

=   阻塞赋值

- 对于非阻塞赋值：

“非阻塞”是指在进程语句（initial和always）中，当前的赋值语句不会阻断其后的语句。非阻塞语句可以认为是分为两个步骤进行的：

a.计算等号右边的表达式的值，可以理解为，在进入进程后，所有的非阻塞语句的右端表达式同时计算，赋值动作只发生在顺序执行到当前非阻塞语句那一刻。

b.在本条赋值语句结束时，将等号右边的值赋给等号左边的变量。

例如：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E9%9D%9E%E9%98%BB%E5%A1%9E%E8%B5%8B%E5%80%BC.jpg)

当执行“x<=next_x;”时，并不会阻断语句“y<=x;”的执行。因此，语句“y<=x;”中的x的值与语句“x<=next_x;”中的x的值不同：语句“y<=x;”中的x是右边D触发器的初值（Q0）。而语句“x<=next_x;”中的x的值是左边D触发器经过一个同步脉冲后的输出值（Q1）。基于此这个进程产生了与阻塞赋值进程截然不同的结果，即：产生了移位寄存器的效果。

实际的例子：

```verilog
reg A;
reg B;
always @(posedge clk)
    begin
		A <= 1'b1;
		B <= 1'b1;
	end
```

```verilog
reg A;
reg B;
always @(posedge clk)
    begin
		A <= 1'b1;
	end
always @(posedge clk)
    begin
		B <= 1'b1;
	end
```

以上两段程序完全等价，两段程序综合出的逻辑也是完全相同的，A和B同时被赋值，体现了FPGA的并行性。

- 对于阻塞赋值：

“阻塞”是指在进程语句（initial和always）中，当前的赋值语句阻断了其后的语句，也就是说后面的语句必须等到当前的赋值语句执行完毕才能执行。而且阻塞赋值可以看成是一步完成的，即：计算等号右边的值并同时赋给左边变量。

例如：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E9%98%BB%E5%A1%9E%E8%B5%8B%E5%80%BC.jpg)

当执行“x=next_x;”时，x会立即的到next_x的值。而下一句“y=x;”必须等到“x=next_x;”执行完毕才能被执行。由于这两条语句都没有延迟（相当于导线），导致他们的等价语句为“y=next_x;”。

实际的例子：

```verilog
always @(posedge clk)
	begin
		A = 1'b1;
		B <= 1'b1;
	end
```

这个程序是阻塞和非阻塞的混合使用，这样写是为了更好得理解阻塞赋值。

当时钟上升沿来临的时刻，首先A会被置1，然后B寄存器再置1。区别就是A和B不再同时置1。A要比B提前零点几纳秒。

这样就出现了先后顺序。这个过程还是在一个时钟内完成的，但是数据到达B寄存器相比上面两段程序晚了那么零点几纳秒!

当我们的时钟跑的比较慢的时候，比如50M，一个周期有20ns，那么这么短暂的延时基本可以忽略不计，但是随着设计的复杂，以及时钟速度的提高，这样的语句就要小心。



### 9、常用语句（上）

#### （1）条件语句

##### 1、 if 语句：

```
一、if（表达式）//表达式为逻辑表达式或关系表达式，若为0,x,z，为假；若为1，为真
二、if（表达式）//表达式允许一定形式的简写，如if（expression）等价于 if (expression == 1 )
		语句1；
	else
		语句2；
三、if（表达式1）
		语句1；
	else if（表达式2）  语句2；
	else if（表达式3）  语句3；
	.....
	else 	语句n;
```

:one: 条件语句必须在过程块中使用，即只能在 initial 和 always 语句引导的 begin end块中，其他位置均不能编写使用。

:two: 注意 if 语句和 begin end 块的嵌套使用。

##### 2、case 语句

```
case（表达式）	< case 分支项>		endcase  //case是一种全等比较，必须每一位都相同
casez（表达式）	< case 分支项>		endcase  //casez语句用来处理不考虑高阻态z的比较过程
casex（表达式）	< case 分支项>		endcase  //casex语句用来处理不考虑高阻值Z和不定值x的情况

case分支项的一般格式为：
分支表达式1：		  语句1；
分支表达式2：		  语句2
default(默认)项	语句；
```

​                                                         case，casez和casex的真值表

| case | 0    | 1    | x    | z    |      | casez | 0    | 1    | x    | z    |      | casex | 0    | 1    | x    | z    |
| ---- | ---- | ---- | ---- | ---- | ---- | ----- | ---- | ---- | ---- | ---- | ---- | ----- | ---- | ---- | ---- | ---- |
| 0    | 1    | 0    | 0    | 0    |      | 0     | 1    | 0    | 0    | 1    |      | 0     | 1    | 0    | 1    | 1    |
| 1    | 0    | 1    | 0    | 0    |      | 1     | 0    | 1    | 0    | 1    |      | 1     | 0    | 1    | 1    | 1    |
| x    | 0    | 0    | 1    | 0    |      | x     | 0    | 0    | 1    | 1    |      | x     | 1    | 1    | 1    | 1    |
| z    | 0    | 0    | 0    | 1    |      | z     | 1    | 1    | 1    | 1    |      | z     | 1    | 1    | 1    | 1    |

##### 3、关于使用条件语句不当，综合时生成锁存器的问题：

```verilog
always @ (al or d)
    bengin
    	if(al) q=d;
	end
//以上代码，综合时会生成锁存器

always @ (al or d)
    begin
        if(al) q=d;
        else q=0;
    end
//以上代码，综合时不会生成锁存器
```

在第一段代码中，没有写出 al=0 时，q 的值。在always块内，如果在给定的条件下没有赋值，这个变量就会保持原值，也就是说会生成一个锁存器。如果希望 al=0时，q的值为0，则else项就是必不可少的。同理，在case语句中没有default项的话，也会造成综合生成锁存器这种情况，如下：

```verilog
always @ (sel[1:0] or a or b)
    case(sel[1:0])
        2'b00: q<=a;
        2'b11: q<=b;
    endcase
//以上代码会生成锁存器

always @ (sel[1:0] or a or b) 
	case(sel[1:0])
        2'b00: q<=a;
        2'b11: q<=b;
        default: q<='b0;
    endcase
//以上代码不会生成锁存器
```

结论：如果用到 if 语句，最好写上else；如果用case语句，最好加上default语句。



#### （2）循环语句

##### 1、forever语句

语句格式：

```verilog
forever 语句；
或者
forever
    begin
        语句
    end
```

forever循环语句是连续执行的语句，其常用于产生周期性的波形，用来作为仿真测试信号。它与always语句的不同在于其不能独立写在程序中，而必须写在initial语句中。

##### 2、repeat语句

连续执行一条语句n次

语句格式：

```verilog
repeat 语句；
或者
repeat
    begin
        语句
    end
```

示例：使用repeat循环语句及加法器和移位寄存器实现乘法器

```verilog
parameter size=8,longsize=16;
reg [size:1] opa, opb;
reg [longsize:1] result;
begin: mult
	reg [longsize:1] shift_opa, shift_opb；
	shift_opa = opa;
	shift_opb = opb;
	result = 0;
	repeat(size)
		begin
			if(shift_opb[1])
				result = result + shift_opa;

			shift_opa = shift_opa <<1;
			shift_opb = shift_opb >>1;
		end
end
```



##### 3、while语句

执行一条语句直到某个条件不满足。如果一开始条件即不满足，则语句一次也不执行。

语句格式：

```
while(表达式) 语句；
或者
while(表达式)
	begin
		多条语句;
	end	
```

示例：用while循环语句对rega这个八位二进制数中值为1的位进行计数

```verilog
begin:	count1s
	reg[7:0] tempreg;
	count=0;
	tempreg = rega;
	while(tempreg)
		begin
			if(tempreg[0])  count = count + 1;
			tempreg = tempreg>>1;
		end
end
```



##### 4、for语句

```
for（循环变量赋初值；循环是否结束条件；循环变量增值) 
	执行语句；

其等价于如下语句：
begin
	循环变量赋初值 ；
	while（循环是否结束条件） 
	begin
		执行语句
		循环变量增值 ；
	end
end
```

示例1：用for循环语句初始化memory

```verilog
begin:	init_mem
	reg[7:0] tempi;
	for(tempi=0;tempi<memsize;tempi=tempi+1)
		memory[tempi]=0;
end
```

示例2：用for循环来实现乘法器

```verilog
parameter  size = 8, longsize = 16;
reg[size:1] opa, opb;
reg[longsize:1] result;

begin:mult
	integer bindex;
	result=0;
	for( bindex=1; bindex<=size; bindex=bindex+1 )
		if(opb[bindex])
			result = result + (opa<<(bindex-1));
end
```

示例3：对rega这个八位二进制数中值为1的位进行计数

在for语句中，循环变量增值表达式可以不是一般的常规加法或者减法表达式

```verilog
begin: count1s
	reg[7:0] tempreg;
	count=0;
	for( tempreg=rega; tempreg; tempreg=tempreg>>1 )
		if(tempreg[0])  
			count=count+1;
end
```



#### （3）块语句

块语句包括两种类型：顺序块和并行块

##### 1、顺序块（过程块）

顺序块由关键字begin-end将多条语句组成，其具有以下特点：

a. 顺序块中的语句是一条接一条按顺序执行的，只有前面语句执行完成之后才能执行后面的语句（除了带有内嵌延迟控制的非阻塞赋值语句）。

b. 如果语句包括延迟或事件控制，那么延迟总是相对于  前面那条语句执行完成的仿真时间

示例1：

```verilog
reg x, y;
reg [1:0] z, w;
 
initial
begin
        x = 1'b0;  
        y = 1'b1; 
        z = {x, y};  
        w = {y, x}; 
end
//在仿真0时刻x,y,z,w的最终值分别为0,1,1,2
//执行这4个赋值语句有顺序，但不需要执行时间
```

示例2：

```verilog
reg x, y;
reg [1:0] z, w;

initial
begin
        x = 1'b0;     //在仿真时刻0 完成
        #5 y = 1'b1;   //在仿真时刻5 完成
        #10 z = {x, y};  // 在仿真时刻15 完成 
        #20 w = {y, x};  // 在仿真时刻35 完成 
end
//这4个变量的最终值也是0,1,1,2，但块语句完成时的仿真时刻为35，除第一句外，以后每条执行语句都需要等待
```

##### 2、并行块

并行块由关键字 fork-join 声明，其特点为：

a. 并行块内的语句并行执行

b. 语句执行的顺序是由各自语句内延迟或事件控制决定的

c. 语句中的延迟或事件控制是相对于块语句开始执行的时刻而言的

示例1：

```verilog
reg x, y;
reg [1:0] z, w;
 
initial
fork
        x = 1'b0;       // 在仿真时刻0 完成 
        #5 y = 1'b1;     // 在仿真时刻5 完成
        #10 z = {x, y};   // 在仿真时刻10 完成
        #20 w = {y, x};   // 在仿真时刻20 完成
join
//并行块中所有的语句同时开始执行，语句之间的先后顺序是无关紧要的
//此代码与上一个顺序块相比，执行结束的时间为第20个仿真时间单位，而不是第35个
```

在使用并行块时，如果两条语句在同一时刻对同一个变量产生影响，那么将会引起隐含的竞争，需要避免这种情况。

示例2：

```verilog
reg x, y;
reg [1:0] z, w;
 
initial
fork
        x = 1'b0;  
        y = 1'b1; 
        z = {x, y};  
        w = {y, x}; 
join
//在此例中，所有的语句在仿真0时刻开始执行，但是实际的执行顺序是未知的
/*关于仿真器使用的问题：使用仿真器进行仿真时，理论上并行块中所有语句是一起执行的，但是实际上运行仿真程序的CPU在任一个时刻只能执行一条语句,而且不同的仿真器按照不同的顺序执行*/
//如果x=1'b0和y=1'b1这两条语句首先执行，那么变量z和w的值为1和2;
//如果这两条语句最后执行，那么z和w的值都是2'bxx
```

##### 3、块语句的特点

:one: 块语句的的嵌套

```verilog
initial
begin
        x = 1'b0; 
        fork 
                #5 y = 1'b1; 
                #10 z = {x, y}; 
        join
        #20 w = {y, x};
end

endmodule
```

:two: 命名块

块可以具有自己的名字，称为命名块，其特点是：

a. 命名块中可以声明局部变量

b. 命名块是设计层次的一部分，命名块中声明的变量可以通过层次名引用进行访问

c. 命名块可以被禁用

示例1：显示命名块和命名块的层次名引用

```verilog
module  top ;

initial
begin :  block1  //名字为block1的顺序命名块
	integer  i ;      //整型变量 i 是block1命名块的静态本地变量
                //可以用层次名top.block1.i 被其他模块访问
...
...
end

initial
fork :  block2  //名字为block2的并行命名块
	reg  i ;   //寄存器变量 i 是block2命名块的静态本地变量
                //可以用层次名top.block2.i 被其他模块访问
...
...
join
```

命名块的禁用

verilog通过关键字disable提供了一种中止命令块执行的方法。

disable可以用来从循环中退出、处理错误条件以及控制某些代码段是否被执行。

使用disable可以禁用设计中任意一个命名块，然后紧接着块语句的那条语句被执行。

示例：在一个标志寄存器中查找第一个不为零的位

```verilog
//在（矢量）标志寄存器的各个位中从低有效位开始找寻第一个值为1的位 
//从矢量标志寄存器的低有效位开始查找第一个值为1的位
reg [15:0] flag;
integer i; //用于计数的整数
 
initial
begin
  flag = 16'b 0010_0000_0000_0000;
  i = 0;
  begin: block1   //while循环声明中的主模块是命名块block1
  	while(i < 16) 
		begin
     		if (flag[i])
     			begin
        			$display("Encountered a TRUE bit at element number %d", i);
        			disable block1; // 在标志寄存器中找到了值为真（1）的位，禁用block1  
    			 end
 			i = i + 1;
		end
  end
end
```



#### （4）生成语句



##### 1、循环生成



##### 2、条件生成



##### 3、case生成





### 10、常用语句（下）

#### （1）结构语句

verilog中的任何过程模块都从属于以下4种结构的说明语句：

##### 1、initial语句

语句格式：

```
initial
	begin
		语句1；
		语句2；
		...
		语句n；
	end
```

示例1：用initial块对存储器变量赋初始值

```verilog
initial
	begin
		areg=0;	//初始化寄存器areg
		for(index=0;index<size;index=index+1)
			memory[index]=0;	//初始化一个memory
	end
/*在这个例子中用initial语句在仿真开始时对各变量进行初始化，这个初始化过程不需要任何仿真时间，即在0ns时间内，
便可以完成存储器的初始化工作*/
```

示例2：用initial语句来生成激励波形

initial块常用于测试文件和虚拟模块的编写，用来产生仿真测试信号和设置信号记录等仿真环境。

```verilog
initial
	begin
		inputs = 'b000000;	//初始时刻为0
		#10 inputs = 'b011001;	
		#10 inputs = 'b011011;	
		#10 inputs = 'b011000;	
		#10 inputs = 'b001000;	
	end
```



##### 2、always语句

一个程序模块可以有多个initial和always过程块。每个initial和always说明语句在仿真的一开始同时立即开始执行。initial语句只执行一次，而always语句则是不断地重复活动着，直到仿真过程结束。但always语句后的过程块是否运行，则要看它的触发条件是否满足，每满足一次，则运行一次过程块。

语句格式： always   <时序控制>  <语句>

always的时间控制可以是边沿触发（时序逻辑电路）也可以是电平触发（组合逻辑电路），可以单个信号也可以多个信号，中间需要用关键字or连接。

一个模块中可以有多个always块，他们是并行运行的。

示例1：

```verilog
always #half_period  areg = ~areg;
//这个例子生成了一个周期为:period(=2*half_period) 的无限延续的信号波形，
//常用这种方法来描述时钟信号，作为激励信号来测试所设计的电路。
```

示例2：

```verilog
reg[7:0] counter;
reg tick;
always @(posedge areg) 
	begin
    	tick = ~tick;
         counter = counter + 1;
    end
//在这个例子中，每当areg信号的上升沿出现时把tick信号相反，并且把counter增加1.
```

示例3：OR事件控制（敏感列表）

```verilog
//有异步复位的电平敏感锁存器
always @ ( reset  or  clock  or  d ) 
    //等待复位信号reset 或 时钟信号clock 或 输入信号d 的改变
begin                
   if ( reset )         //若 reset 信号为高，把q置零
      q = 1 'b0 ;
    else  if ( clock )   //若clock 信号为高，锁存输入信号d
      q = d ;
end
```

示例4：使用逗号的敏感列表

```verilog
//有异步复位的电平敏感锁存器
always @ ( reset , clock , d )
 //等待复位信号reset 或 时钟信号clock 或 输入信号d的改变

begin               
   if ( reset )         // 若 reset 信号为高，把q置零
      q = 1 'b0 ;
   else  if ( clock )   // 若clock 信号为高，锁存输入信号d
      q = d ;
end

//用reset异步下降沿复位，clk正跳变沿触发的D寄存器
always @ ( posedge clk , negedge reset )  //注意：使用逗号来代替关键字or
	if (! reset )
   		q <= 0 ;
  	else 
   		q <= d ;
```

示例5：@ * 操作符的使用

```
//用or 操作符的组合逻辑块
//编写敏感列表很繁琐并且容易漏掉一个输入
always @ ( a or b or c or d or e or f or g or h or p or m )
begin
out1 = a ?  b + c  :  d + e ;
out2 = f ?  g + h  :  p + m ;
end
//不用上述方法，用符号 @(*) 来代替，可以把所有输入变量都自动包括进敏感列表。
always @ ( * )
begin
out1 = a ?  b + c  :  d + e ;
out2 = f ?  g + h  :  p + m ;
end
```

示例6：电平敏感时序控制

```verilog
//verilog同时也允许使用另外一种形式表示电平敏感时序控制，即后面的语句和语句块需要等待某个条件为真才能执行
//verilog用关键字wait来表示等待电平敏感的条件为真
always
		wait (count_enable)		#20 count = count + 1;
//在这个例子中，仿真器连续监视count_enable，若其值为0，则不执行后面的语句，会停止运行
//若count_enable=1，则在20个时间单位之后执行count加1
```



##### 3、task和function说明语句

​	task和function说明语句分别用来定义任务和函数，利用任务和函数可以把一个很大的程序模块分解成许多较小的任务和函数便于理解和调试。 输入、输出和总线信号的值可以传人、传出任务和函数。 任务和函数往往还是大的程序模块中在不同地点多次用到的相同的程序段。

###### （1）task和function 说明语句的不同点

任务和函数有些不同，主要的不同有以下4点：

:black_medium_square:  函数只能与主模块共用同一个仿真时间单位，而任务可以定义自己的仿真时间单位。

:black_medium_square:  ​函数不能启动任务，而任务能启动其他任务和函数。

:black_medium_square:  函数至少要有一个输入变量，而任务可以没有或有多个任何类型的变量。

:black_medium_square:  ​函数返回一个值，而任务则不返回值。

​	函数的目的是通过返回一个值来响应输入信号的值。 任务却能支持多种目的，能计算多个结果值，这些结果值只能通过被调用的任务的输出或总线端口送出。 Verilog HDL模块使用函数时是把它当作表达式中的操作符，这个操作的结果值就是这个函数的返回值。例如：

```
定义一任务或函数对一个16位的字进行操作让其高字节与低字节互换，把它变为另一个字（假定任务或者函数名为switch_bytes）。
一、任务返回的新字是通过输出端口的变量，因此16位字字节互换任务的调用源码是：
switch_bytes(old_ word, new_ word); 
任务 switch_bytes 把输入old_word字的高、低字节互换放入 new_word端口输出。 
二、函数返回的新字是通过函数本身的返回值，因此16位字字节互换的调用源码为：
new_ word = switch_bytes (old_ word)
```

###### （2）关于task说明语句

​	如果传给任务的变量值和任务完成后接收结果的变量已定义，就可以用一条语句启动任 务，任务完成以后控制就传回启动过程。 如任务内部有定时控制，则启动的时间可以与控制返回的时间不同。 任务可以启动其他 的任务，其他任务又可以启动别的任务，可以启动的任务数是没有限制的。 不管有多少任务启动，只有当所有的启动任务完成以后，控制才能返回。

**1、任务的定义**

```
task  <任务名>
	＜端口及数据类型声明语句＞ 
	＜语句1>
	＜语句2>
	＜语句n>
endtask
```

这些声明语句的语法与模块定义中的对应声明语句的语法是一致的。

**2、任务的调用及变量的传递**

〈任务名〉（端口1，端口2，… ，端口n);

**3、举例**

```verilog
task  my_task;
	input a, b;
	inout  c;
	output d, e;
	…
	<语句>	//执行任务工作相应的语句
	…
	c = foo1;	//赋初始值
	d = foo2;	//对任务的输出变量赋值  
	e = foo3;
endtask

//任务调用：
my_task(v,w,x,y,z);
//任务调用变量(v,w,x, y,z）和任务定义的I/O变量（a,b,c,d,e）之间是一一对应的。
```

###### （3）关于function说明语句

**1、函数定义**

```
function ＜返回值的类型或范围＞（函数名）  //这里返回值得类型或范围是可选项，默认为寄存器类型数据
		＜端口说明语句＞
		＜变量类型说明语句＞
		begin 
		＜语句＞
		......
		end 
endfunction 
```

**2、函数的调用**

<函数名>   （<表达式1>,<表达式2>,...<表达式n>）

**3、举例**

```verilog
function [7:0] getbyte;
input [15:0] address;
	begin
		<说明语句>		//从地址字中提取低字节的程序
		getbyte = result_expression; //把结果赋予函数的返回字节
	end
endfunction
```

**4、函数使用规则**

- 函数的定义不能包含有任何的时间控制语句，即任何用＃、＠、或 wait 来标识的语句。
- 函数不能启动任务。
- 定义函数时至少要有一个输入参量。
- 在函数的定义中必须有一条赋值语句给函数中的一个内部变量赋以函数的结果值，**该内部变量具有和函数名相同的名字。** 

###### （4）特殊函数

**1、自动（递归）函数** 

​	Verilog 中的函数是不能够进行递归调用的。 设计模块中若某函数在两个不同的地方被同时并发调用，由于这两个调用同时对同一块地址空间进行操作，那么计算结果将是不确定的。
​	若在函数声明时使用了关键字 **automatic**，那么该函数将成为自动的或可递归的，即仿真器为每一次函数调用动态地分配新的地址空间，每一个函数调用对各自的地址空间进行操作。因此，自动函数中声明的局部变量不能通过层次名进行访问。 而自动函数本身可以通过层次名进行调用。

**示例：定义自动函数完成递归运算**

```verilog
//用函数的递归调用定义阶乘计算
module top ;
...
//定义自动（递归）函数
function  automatic  integer  factorial ;
input  [ 31 : 0 ]  oper ;
integer  i ;
begin
	if ( operand >= 2 )
   		 factorial = factorial ( oper - 1 ) * oper ;  //递归调用
	else 
   		 factorial = 1 ;
end
endfunction

//调用该函数
integer result ;
initial 
begin
  	result = factorial ( 4 ) ; // 调用4的阶乘
  	$display ( " Factorial of 4 is % 0d ", result ) ;   // 显示24
end
......
......
endmodule 

    
    
```

**2、常量函数**

常量函数实际上是一个带有某些限制的常规verilog函数。这种函数能够用来引用复杂的值，因而可用来代替常量。

**示例：计算模块中地址总线的宽度**

```verilog
module ram ( ... ... ...) ;
parameter  RAM_DEPTH = 256 ;
input  [ clog2( RAM_DEPTH) - 1 : 0 ]  addr_bus ;  //
... ... ... 
... ... ...
//
function integer clogb2 ( input  integer depth ) ;
begin
    for ( clogb2 = 0 ; depth > 0 ; clogb2 = clogb2 + 1 )
     	 depth = depth >> 1 ;
end
endfunction
... ...
... ...
endmodule 

```

**3、带符号函数**

带符号函数的返回值可以作为带符号数进行运算。

**示例：**

```verilog
module top ;
... ...
//
//
function  signed [ 63 : 0 ]  compute _signed (  input [ 63 : 0 ]  vector ) ;
... ...
... ...
endfunction

//
if ( compute_signed (vector)  <  - 3 )
begin
... ...
end
... ... 
endmodule
```



#### （2）系统任务



#### （3）函数语句



#### （4）显示系统任务



### 11、编译预处理语句

预处理命令：

```verilog
//--------------------------------
`include file1.v
//--------------------------------
`define X = 1;
//--------------------------------
`deine Y;
`ifdef Y
Z=1;
`else
Z=0;
`endif
//--------------------------------
```

​       有时候对于一些公共的宏参数，可以放在一个文件中，比如这个文件名字为xx.v 那么通过`include xx.v 就可以使用此文件中定义的一些宏参数。即verilog中的‘include与C语言中的include功能基本一样，区别是C语言中的include一般是包含一个文件，verilog中一般是一些参数定义，其中一般是'ifdef、'define、'endif等。

以下以VGA程序举例：

假设有个lcd_para.v 文件，内容如下：

```verilog
// 640 * 480
`ifdef VGA_640_480_60FPS_25MHz
`define H_FRONT 11'd16
`define H_SYNC 11'd96
`define H_BACK 11'd48
`define H_DISP 11'd640
`define H_TOTAL 11'd800
`define V_FRONT 11'd10
`define V_SYNC 11'd2
`define V_BACK 11'd33
`define V_DISP 11'd480
`define V_TOTAL 11'd525
`endif
// 800 * 600
`ifdef VGA_800_600_72FPS_50MHz
`define H_FRONT 11'd56
`define H_SYNC 11'd120
`define H_BACK 11'd64
`define H_DISP 11'd800
`define H_TOTAL 11'd1040
`define V_FRONT 11'd37
`define V_SYNC 11'd6
`define V_BACK 11'd23
`define V_DISP 11'd600
`define V_TOTAL 11'd666
`endif
//---------------------------------
`define H_Start (`H_SYNC + `H_BACK)
`define H_END (`H_SYNC + `H_BACK + `H_DISP)
`define V_Start (`V_SYNC + `V_BACK)
`define V_END (`V_SYNC + `V_BACK + `V_DISP)
```

这里为VGA 定义了两种分辨率，通过'define VGA_640_480_60FPS_25MHz 或`define VGA_800_600_72FPS_50MHz 来决定使用哪种分辨率。

比如，xxx.v 文件想调用lcd_para.h，那么在xxx.v中可以写到：

```verilog
`define VGA_800_600_60MHz //这句要放在"lcd_para.h"的上面，不然编译不通过
`include "lcd_para.v"
```

那么xxx.v 文件中就可以用lcd_para.v 中的参数了，且对应VGA_800_600_60MHz
下的参数。

另外，xxx.v 和lcd_para.v 必须在同一文件夹下，但是lcd_para.v不是必须添加到工程中。



### 12、状态机概述

​       状态机是许多数字系统的核心部件，是一类重要的时序逻辑电路。通常包括三个部分：一是下一个状态的逻辑电路，二是存储状态机当前状态的时序逻辑电路，三是输出组合逻辑电路。通常，状态机的状态数量有限，称为有限状态机（FSM）。由于状态机所有触发器的时钟由同一脉冲边沿触发，故也称之为同步状态机。

​       根据状态机的输出信号是否与电路的输入有关分为Mealy 型状态机和Moore 型状态机。

​       电路的输出信号不仅与电路当前状态有关，还与电路的输入有关，称为Mealy 型状态机。

​       电路的输出仅仅与各触发器的状态，不受电路输入信号影响或无输入，称为Moore 型状态机。

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E7%8A%B6%E6%80%81%E6%9C%BA%E6%A8%A1%E5%9E%8B.jpg)

### 13、状态机设计方法

状态机的设计步骤：

• 根据需求和设计原则，确定是Moore 型还是Mealy 型状态机；

• 分析状态机的所有状态，对每一状态选择合适的编码方式，进行编码；

• 根据状态转移关系和输出绘出状态转移图；

• 构建合适的状态机结构，对状态机进行硬件描述。

状态机的描述通常包含以下四部分：

1) 利用参数定义语句parameter 描述状态机各个状态名称，即状态编码。状态编码
通常有很多方法包含自然二进制编码，One-hot 编码，格雷编码码等；

2）用时序的always 块描述状态触发器实现状态存储；

3）使用敏感表和case 语句（也采用if-else 等价语句）描述状态转换逻辑；

4）描述状态机的输出逻辑。

### 14、三种状态机示例

状态机的描述通常有三种方法，称为一段式状态机，二段式状态机和三段式状态机。

一段式状态机：

```verilog
module detect_1(
	input clk_i,
	input rst_n_i,
	output out_o
	);
reg out_r;
//状态声明和状态编码
reg [1:0] state;
parameter [1:0] S0=2'b00;
parameter [1:0] S1=2'b01;
parameter [1:0] S2=2'b10;
parameter [1:0] S3=2'b11;
always@(posedge clk_i)
begin
	if(!rst_n_i)begin
		state<=0;
		out_r<=1'b0;
	end
	else
		case(state)
		S0 :
		begin
			out_r<=1'b0;
			state<= S1;
		end
		S1 :
		begin
			out_r<=1'b1;
			state<= S2;
		end
		S2 :
		begin
			out_r<=1'b0;
			state<= S3;
		end
		S3 :
		begin
			out_r<=1'b1;
		end
		endcase
end
assign out_o=out_r;
endmodule
```

一段式状态机是应该避免使用的，该写法仅仅适用于非常简单的状态机设计，不符合组
合逻辑与时序逻辑分开的原则，整个结构代码也不清晰，不利用维护和修改。

二段式状态机：

```verilog
module detect_2(
	input clk_i,
	input rst_n_i,
	output out_o
	);
reg out_r;
//状态声明和状态编码
reg [1:0] Current_state;
reg [1:0] Next_state;
parameter [1:0] S0=2'b00;
parameter [1:0] S1=2'b01;
parameter [1:0] S2=2'b10;
parameter [1:0] S3=2'b11;
//时序逻辑：描述状态转换
always@(posedge clk_i)
begin
	if(!rst_n_i)
		Current_state<=0;
	else
		Current_state<=Next_state;
end
//组合逻辑:描述下一状态和输出
always@(*)
begin
	out_r=1'b0;
	case(Current_state)
		S0 :
			begin
			out_r=1'b0;
			Next_state= S1;
			end
		S1 :
			begin
			out_r=1'b1;
			Next_state= S2;
			end
		S2 :
			begin
			out_r=1'b0;
			Next_state= S3;
			end
		S3 :
			begin
			out_r=1'b1;
			Next_state=Next_state;
			end
	endcase
end
assign out_o=out_r;
endmodule
```

两段式状态机采用两个always 模块实现状态机的功能，其中一个always 采用同步时序
逻辑描述状态转移，另一个always 采用组合逻辑来判断状态条件转移。两段式状态机
是推荐的状态机设计方法。

三段式状态机：

```verilog
module detect_3(
	input clk_i,
	input rst_n_i,
	output out_o
	);
reg out_r;
//状态声明和状态编码
reg [1:0] Current_state;
reg [1:0] Next_state;
parameter [1:0] S0=2'b00;
parameter [1:0] S1=2'b01;
parameter [1:0] S2=2'b10;
parameter [1:0] S3=2'b11;
//时序逻辑：描述状态转换
always@(posedge clk_i)
begin
	if(!rst_n_i)
		Current_state<=0;
	else
		Current_state<=Next_state;
end
//组合逻辑：描述下一状态
always@(*)
begin
	case(Current_state)
	S0:
		Next_state = S1;
	S1:
		Next_state = S2;
	S2:
		Next_state = S3;
	S3:
		begin
		Next_state = Next_state;
		end
	default :
		Next_state = S0;
	endcase
end
//输出逻辑：让输出out，经过寄存器out_r 锁存后输出，消除毛刺
always@(posedge clk_i)
begin
	if(!rst_n_i)
		out_r<=1'b0;
	else
	begin
		case(Current_state)
		S0,S2:
			out_r<=1'b0;
		S1,S3:
			out_r<=1'b1;
		default :
			out_r<=out_r;
		endcase
	end
end
assign out_o=out_r;
```

三段式状态机在第一个always 模块采用同步时序逻辑方式描述状态转移，第二个always 模块采用组合逻辑方式描述状态转移规律，第三个always 描述电路的输出。通常让输出信号经过寄存器缓存之后再输出，消除电路毛刺。这种状态机也是比较推崇的，主要是由于维护方便，组合逻辑与时序逻辑完全独立。

### 15、testbench文件

​       一个完整的设计，除了好的功能描述代码，对于程序的仿真验证是必不可少的。而RTL逻辑设计中，学会根据硬件逻辑来写测试程序，即Testbench 是尤其重要的。Verilog 测试平台是一个例化的待测（MUT）模块，重要的是给它施加激励并观测其输出。逻辑模块与其对应的测试平台共同组成仿真模型，应用这个模型可以测试该模块能否符合自己的设计要求。

​       编写TESTBENCH 的目的是为了对使用硬件描述语言设计的电路进行仿真验证，测试设计电路的功能、性能与设计的预期是否相符。通常，编写测试文件的过程如下：

• 产生模拟激励（波形）；

• 将产生的激励加入到被测试模块中并观察其响应；

• 将输出响应与期望值相比较。

**书写testbench的几点建议：**

• 封装有用且常用的testbench，testbench 中可以使用task 或function 对代码进行封装，下次利用时灵活调用即可；

• 如果待测试文件中存在双向信号(inout)需要注意，需要一个reg 变量来表示输入，一个wire 变量表示输出；

• 单个initial 语句不要太复杂，可分开写成多个initial 语句，便于阅读和修改；

• Testbench 说到底是依赖PC 软件平台，必须与自身设计的硬件功能想搭配。

**（1）完整的Test bench文件结构**

```
module Test_bench();//通常无输入无输出
信号或变量声明定义
逻辑设计中输入对应reg 型
逻辑设计中输出对应wire 型
使用initial 或always 语句产生激励
例化待测试模块
监控和比较输出响应
endmodule
```

**（2）时钟激励设计**

下面列举出一些常用的封装子程序。

```verilog
/*----------------------------------------------------------------
时钟激励产生方法一：50%占空比时钟
----------------------------------------------------------------*/
parameter ClockPeriod=10;
initial
	begin
		clk_i=0;
		forever
			#(ClockPeriod/2) clk_i=~clk_i;
	end
/*----------------------------------------------------------------
时钟激励产生方法二：50%占空比时钟
----------------------------------------------------------------*/
initial
	begin
		clk_i=0;
		always #(ClockPeriod/2) clk_i=~clk_i;
	end
/*----------------------------------------------------------------
时钟激励产生方法三：产生固定数量的时钟脉冲
----------------------------------------------------------------*/
initial
	begin
		clk_i=0;
		repeat(6)
			#(ClockPeriod/2) clk_i=~clk_i;
	end
/*----------------------------------------------------------------
时钟激励产生方法四：产生非占空比为50%的时钟
----------------------------------------------------------------*/
initial
	begin
		clk_i=0;
		forever
			begin
			#((ClockPeriod/2)-2) clk_i=0;
			#((ClockPeriod/2)+2) clk_i=1;
			end
	end
```

**（3）复位信号设计**

```verilog
/*----------------------------------------------------------------
复位信号产生方法一：异步复位
----------------------------------------------------------------*/
initial
	begin
		rst_n_i=1;
		#100;
		rst_n_i=0;
		#100;
		rst_n_i=1;
	end
/*----------------------------------------------------------------
复位信号产生方法二：同步复位
----------------------------------------------------------------*/
initial
	begin
		rst_n_i=1;
		@（negedge clk_i)
		rst_n_i=0;
		#100;  //固定时间复位
		repeat(10) @（negedge clk_i); //固定周期数复位
		@（negedge clk_i)
		rst_n_i=1;
	end
/*----------------------------------------------------------------
复位信号产生方法三：复位任务封装
----------------------------------------------------------------*/
task reset;
	input [31:0] reset_time; //复位时间可调，输入复位时间
	RST_ING=0; //复位方式可调，低电平或高电平
	begin
		rst_n=RST_ING; //复位中
		#reset_time; //复位时间
		rst_n_i=~RST_ING; //撤销复位，复位结束
	end
endtask    
```

**（4）特殊信号设计**

```verilog
/*----------------------------------------------------------------
特殊激励信号产生描述一：输入信号任务封装
----------------------------------------------------------------*/
task i_data;
input [7:0] dut_data;
begin
	@(posedge data_en); send_data=0;
	@(posedge data_en); send_data=dut_data[0];
	@(posedge data_en); send_data=dut_data[1];
	@(posedge data_en); send_data=dut_data[2];
	@(posedge data_en); send_data=dut_data[3];
	@(posedge data_en); send_data=dut_data[4];
	@(posedge data_en); send_data=dut_data[5];
	@(posedge data_en); send_data=dut_data[6];
	@(posedge data_en); send_data=dut_data[7];
	@(posedge data_en); send_data=1;
	#100;
end
endtask
//调用方法：i_data(8'hXX);
/*----------------------------------------------------------------
特殊激励信号产生描述二：多输入信号任务封装
----------------------------------------------------------------*/
task more_input;
input [7:0] a;
input [7:0] b;
input [31:0] times;
output [8:0] c;
begin
	repeat(times) //等待times 个时钟上升沿
	@(posedge clk_i)
	c=a+b; //时钟上升沿a，b 相加
end
endtask
//调用方法：more_input(x,y,t,z); //按声明顺序
/*----------------------------------------------------------------
双向信号描述一：inout 在testbench 中定义为wire 型变量
----------------------------------------------------------------*/
//为双向端口设置中间变量inout_reg 作为inout 的输出寄存，其中inout 变
//量定义为wire 型，使用输出使能控制传输方向
//inout bir_port;
wire bir_port;
reg bir_port_reg;
reg bi_port_oe;
assign bi_port=bi_port_oe ? bir_port_reg : 1'bz;
/*----------------------------------------------------------------
双向信号描述二：强制force
----------------------------------------------------------------*/
//当双向端口作为输出口时，不需要对其进行初始化，而只需开通三态门
//当双向端口作为输入时，只需要对其初始化并关闭三态门，初始化赋值需
//使用wire 型数据，通过force 命令来对双向端口进行输入赋值
//assign dinout=(!en) din :16'hz; 完成双向赋值
initial
	begin
		force dinout=20;
		#200
		force dinout=dinout-1;
	end
/*----------------------------------------------------------------
特殊激励信号产生描述三：输入信号产生,一次SRAM 写信号产生
----------------------------------------------------------------*/
initial
	begin
		cs_n=1; //片选无效
		wr_n=1; //写使能无效
		rd_n=1; //读使能无效
		addr=8'hxx; //地址无效
		data=8'hzz; //数据无效
		#100;
		cs_n=0; //片选有效
		wr_n=0; //写使能有效
		addr=8'hF1; //写入地址
		data=8'h2C; //写入数据
		#100;
		cs_n=1;
		wr_n=1;
		#10;
		addr=8'hxx;
		data=8'hzz;
	end
/*----------------------------------------------------------------
Testbench 中@与wait
----------------------------------------------------------------*/
//@使用沿触发
//wait 语句都是使用电平触发
initial
	begin
		start=1'b1;
		wait(en=1'b1);
		#10;
		start=1'b0;
	end
```

**（5）仿真控制语句及系统任务描述**

```verilog
/*----------------------------------------------------------------
仿真控制语句及系统任务描述
----------------------------------------------------------------*/
$stop //停止运行仿真，modelsim 中可继续仿真
$stop(n) //带参数系统任务，根据参数0,1 或2 不同，输出仿真信息
$finish //结束运行仿真，不可继续仿真
$finish(n) //带参数系统任务，根据参数0,1 或2 不同，输出仿真信息
//0:不输出任何信息
//1:输出当前仿真时刻和位置
//2:输出当前仿真时刻、位置和仿真过程中用到的memory 以及CPU 时间的
统计
$random //产生随机数
$random % n //产生范围-n 到n 之间的随机数
{$random} % n //产生范围0 到n 之间的随机数


/*----------------------------------------------------------------
仿真终端显示描述
----------------------------------------------------------------*/
$monitor //仿真打印输出,大印出仿真过程中的变量，使其终端显示
/*
$monitor($time,,,"clk=%d reset=%d out=%d",clk,reset,out);
*/
$display //终端打印字符串,显示仿真结果等
/*
$display(” Simulation start ! ");
$display(” At time %t,input is %b%b%b,output is %b",$time,a,b,en,z);
*/
$time //返回64 位整型时间
$stime //返回32 位整型时间
$realtime //实行实型模拟时间


/*----------------------------------------------------------------
文本输入方式：$readmemb/$readmemh
----------------------------------------------------------------*/
//激励具有复杂的数据结构
//verilog 提供了读入文本的系统函数
$readmemb/$readmemh("<数据文件名>",<存储器名>);
$readmemb/$readmemh("<数据文件名>",<存储器名>,<起始地址>);
$readmemb/$readmemh("<数据文件名>",<存储器名>,<起始地址>,<结束地址>);
$readmemb:/*读取二进制数据，读取文件内容只能包含：空白位置，注释行，二进制数,数据中不能包含位宽说明和格式说明，每个数字必须是二进制数字。*/
$readmemh:/*读取十六进制数据，读取文件内容只能包含：空白位置，注释行，十六进制数数据中不能包含位宽说明和格式说明，每个数字必须是十六进制数字。*/
/*当地址出现在数据文件中，格式为@hh...h,地址与数字之间不允许空白位置，可出现多个地址*/

module
	reg [7:0] memory[0:3];//声明8 个8 位存储单元
	integer i;
	initial
		begin
			$readmemh("mem.dat",memory);//读取系统文件到存储器中的给定地址
			//显示此时存储器内容
			for(i=0;i<4;i=i+1)
				$display("Memory[%d]=%h",i,memory[i]);
		end
endmodule
/*mem.dat 文件内容
@001
AB CD
@003
A1
*/
//仿真输出为
Memory[0] = xx;
Memory[1] = AB;
Memory[2] = CD;
Memory[3] = A1;
```

**（6）加法器仿真文件示例**

加法器程序：

```verilog
module add(a,b,c,d,e);// 模块接口
input [5:0] a; // 输入信号a
input [5:0] b; // 输入信号b
input [5:0] c; // 输入信号a
input [5:0] d; // 输入信号b
output[7:0] e; // 求和输出信号
wire [6:0]outa1,outa2; // 定义输出网线型
assign e = outa2+outa1; // 把两部分输出结果合并
/*
通常，我们模块的调用写法如下：
被调用的模块名字- 自定义的名字- 括号内信号
括号内的信号类似于.ina(ina1)
这种写法最常用，信号的顺序可以调换
另外还有一种写法没可以直接这样写
adder u1 (ina1,inb1,outa1);
这种写法必须确保信号的顺序一致，这种写法几乎没有人采用
*/

/*被调用的模块名 自定义模块名 （.被调用模块信号名1（自定义信号名1）,
                             .被调用模块信号名2（自定义信号名2））;*/
adder u1 (.ina(a),.inb(b),.outa(outa1)); // 调用adder 模块，自定义名字为u1
adder u2 (.ina(c),.inb(d),.outa(outa2)); // 调用adder 模块，自定义名字为u2
endmodule
//adder 子模块
module adder(ina,inb,outa );// 模块接口
input [5:0] ina; // ina-输入信号
input [5:0] inb; // inb-输入信号
output [6:0] outa; // outa-输入信号
assign outa = ina + inb; // 求和
endmodule // 模块结束
```

仿真文件：

```verilog
`timescale 1ns / 1ps
module add_tb();
reg [5:0] a;
reg [5:0] b;
reg [5:0] c;
reg [5:0] d;
wire[7:0] e;
reg [5:0] i; //中间变量
// 调用被仿真模块模块
add uut (.a(a), .b(b),.c(c),.d(d),.e(e));
initial// initial 是仿真用的初始化关键词
begin 
	a=0;b=0;c=0;d=0; // 必须初始化输入信号
	for(i=1;i<31;i=i+1)
	begin
		#10 ;
		a = i;
		b = i;
		c = i;
		d = i;
	end
end
initial
begin
	$monitor($time,,,"%d + %d + %d + %d ={%d}",a,b,c,d,e); // 信号打印输出
	#500 $finish;
end
endmodule
```

仿真结果：

![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E5%8A%A0%E6%B3%95%E5%99%A8%E4%BB%BF%E7%9C%9F.jpg) ![](D:/typora%E5%9B%BE%E7%89%87%E6%80%BB/%E5%8A%A0%E6%B3%95%E5%99%A8%E4%BB%BF%E7%9C%9F2.jpg)

### 16、简单verilog例程

#### （1）加法器

```verilog
/*带有异步复位，同步计数使能和可预置型十进制计数器*/
module CNT10 (CLK,RST,EN,LOAD,COUT,DOUT,DATA);
    input CLK,EN,RST,LOAD;
	input [3:0] DATA;//4位并行加载数据
	output [3:0] DOUT;//计数数据输出
	output COUT;
	reg [3:0] Q1; reg COUT;
	assign DOUT=Q1;//将内部寄存器的计数结果输出至DOUT
	always@(posedge CLK or negedge RST) begin
	        if(!RST) Q1<=0;//异步清零
			//RST下降沿时，清零，
			//之后，若RST一直低电平，则一直清零状态
			//当CLK上升沿，并且RST=1，EN=1时，看LOAD的值
			//LOAD=0时，置数，LOAD=1时，累加1
	   else if(EN) begin //同步使能EN=1，此时允许加载或计数
	             if(!LOAD) Q1<=DATA;//当LOAD=0，向内部寄存器加载数据
			else if(Q1<9)  Q1<=Q1+1;//当Q1小于9时，允许累加，此时为十进制计数器
			else Q1<=4'b0000; end
			end 
	
	always@ (Q1)//组合逻辑电路
	    if（Q1==4'h9) COUT=1'b1;  //当Q1=1001时，COUT输出进位标志1
		   else     COUT=1'b0;  //否则，输出进位标志0
endmodule
```

#### （2）D触发器

```verilog
/*D触发器基本模块*/
module DFF1 (CLK,D,Q);
   output Q;
   input  CLK,D;
   reg Q;
   always@ (posedge CLK)
      Q<=D;
endmodule
/*时序模块的简单识别，
  1、always的敏感信号列表含有posedge或者negedge
  2、always语句中有不完整的条件语句，例如if(CLK) Q<D;
  3、门逻辑电路图中有输出到输入的反馈
  */
 //一般，组合电路用阻塞式赋值，时序电路用非阻塞式赋值 
  
  
/*含异步清零和时钟同步使能的D触发器*/
module DFF2 （CLK,D,Q,RST,EN);//同步，依赖时钟；异步，不依赖时钟
   output Q;
   input CLK,D,RST,EN;
    always@(posedge CLK or negedge RST)
	//异步清零，该复位信号出现在always中，与时钟在一起，说明不依赖时钟，为异步
	  begin
	    if(!RST) Q<=0;
	    else if(EN) Q<=D;
	  end
endmodule
```

#### （3）数据选择器

```verilog
 /*四选一多路选择器 */
 module MUX41a(a,b,c,d,s1,s0,y);
 input a,b,c,d;
 input s0,s1;
 output y;
 
 reg y;//变量有两种，寄存器类型（reg)和线型(wire),没有特意定义的，一般默认为wire类型
 //只能对寄存器类型端口赋值
 
 always@(a，b ，c ，d ，s1 ，s0)//必须列入所有输入端口
   begin : MUX41
     case({s1,s0})
	     2'b00: y<=a;
		 2'b01: y<=b;
		 2'b10: y<=c;
		 2'b11: y<=d;
		 default: y<=a;
	 endcase
   end 
//case是一种全等比较，必须每一位都相同
//casez语句用来处理不考虑高阻态z的比较过程
//casex语句用来处理不考虑高阻值Z和不定值x的情况
```

#### （4）移位寄存器

```verilog
/*移位模式可控的8位移位寄存器*/
module SHFT2(CLK,C0,MD,D,QB,CN);
    output CN;//进位输出
	output [7:0] QB;//移位数据输出
	input [7:0] D;//待加载移位的数据输入
	input [2:0] MD;//移位模式控制字
	input  CLK,C0;//时钟和进位输入
	reg [7:0] REG; reg CY;
	
  always@（posedge CLK) begin
	case(MD)
	1:begin REG[0]<=C0；REG[7:1]<=REG[6:0]; CY<=REG[7]；end //带进位循环左移
	2:begin REG[0]<=REG[7]; REG[7:1]<=REG[6:0]; end //自循环左移
	3:begin REG[7]<=REG[0]; REG[6:0]<=REG[7:1]; end //自循环右移
	4:begin REG[7]<=C0; REG[6:0]<=REG[7:1]; CY=REG[0]; end //带进位循环右移
	5:begin REG<=D; end //加载待移数
	default:begin REG<=REG;CY<=CY;end //过程结束
	endcase end
	  assign QB=REG;//移位数据并行输出
	  assign CN=CY;//左移高位输出
endmodule
```

#### （5）FIFO

```verilog
/*同步FIFO*/
module sc_fifo(
input clk,rst,
input [7:0] din,
input wr,
output full,
output reg [7:0] dout,
input rd,
output empty
);

//1，需要一个buff双口存储器做存储载体，当地址有n位，就有2^N个存储单元，标号从0到（2^N-1）。三位地址存储地址就是从0到7。
reg [7:0] buff[0:7];
//2，需要wr_ptr指针，与地址位数相同，指向下个可以写的地址。
reg [2:0] wr_ptr;
//3，需要rd_ptr指针，与地址位数相同，指向下个可以读的地址。
reg [2:0] rd_ptr;
//4，需要fifo_cntr计数器，比地址位数多一位，指示当前可读的数据个数。
reg [3:0] fifo_cntr;//用以表示有效可读数据的个数
//5，full和empty由fifo_cntr生成。
assign full=fifo_cntr==8;
assign empty=fifo_cntr==0;
//6，区别读写操作是否有效。
wire valid_rd=~empty &rd;
wire valid_wr=~full&wr;
//7,实现以上功能
always@(posedge clk) if(rst) wr_ptr<=0;else if (valid_wr) wr_ptr<=wr_ptr+1;
always@(posedge clk) if(rst) rd_ptr<=0;else if (valid_rd) rd_ptr<=rd_ptr+1;

    always@(posedge clk) if(rst) fifo_cntr<=0;else
if ((valid_rd==0)&&(valid_wr==1)) fifo_cntr<=fifo_cntr+1;
else if ((valid_rd==1)&&(valid_wr==0)) fifo_cntr<=fifo_cntr-1;
/*用case语句写
always@(posedge clk) 
casex ({rst,valid_wr,valid_rd})
3'b1xx : fifo_cntr<=0;
3'b010 : fifo_cntr<=fifo_cntr+1;
3'b001 : fifo_cntr<=fifo_cntr-1;
3'b011 ,3'b000 :fifo_cntr<=fifo_cntr ;
endcase  */


always@(posedge clk) if(valid_wr) buff[wr_ptr]<=din;
always@(posedge clk) if(valid_rd) dout<=buff[rd_ptr];

endmodule
```

