---
typora-root-url: D:\typora图片总
---

# 10 基于HLS的手写识别网络的搭建

## 十、基于HLS的手写识别网络的搭建

**视频内容索引：** 

1、神经网络基础

2、手写识别网络的讲解

3、讲解卷积和池化运算代码，并实例演示HLS使用流程

4、讲解HLS中的不同Directive约束的使用，SOC平台的搭建

5、PYNQ开发板使用教程，书写卷积和池化函数的python驱动（参考HLS中卷积和池化的c语言版本驱动）

6、使用python实现MNIST网络代码讲解

### 1、手写识别网络MNIST概述

**关于图片：**

一张灰度图片相当于一个二维矩阵。

![](/灰度图片.png)

彩色图片是由红绿蓝（RGB）三色混合，即相当于一个三维矩阵。



此工程的源码，在/tensorflow MNIST实现（包含data转换成bin）/tensorflow/MNIST/中，是神经网络的实现：

num_check.py是没有卷积的神经网络代码：

```python
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
import tensorflow as tf
x = tf.placeholder(tf.float32, [None, 784])
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
y = tf.nn.softmax(tf.matmul(x, W) + b)
y_ = tf.placeholder(tf.float32, [None, 10])
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1])) 
#定义损失函数
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
#代价函数
sess = tf.InteractiveSession()
init = tf.global_variables_initializer()
sess.run(init)
for _ in range(1000):
    batch_xs, batch_ys = mnist.train.next_batch(100) #每次用100张图片进行反向传播
    sess.run(train_step, {x: batch_xs, y_: batch_ys})
#print(sess.run(tf.matmul(x, W) + b, {x: mnist.test.images}))
correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
print(sess.run(accuracy, {x: mnist.test.images, y_: mnist.test.labels}))
```

mnist_int16_test是手写数字识别**卷积**神经网络的实现，input_data.py是导入的包：

```python
# -*- coding: utf-8 -*-
import input_data
import tensorflow as tf
import numpy as np
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
sess = tf.InteractiveSession()

def Record_Tensor(tensor,name):
	print ("Recording tensor "+name+" ...")
	f = open('./record/'+name+'.dat', 'w')
	array=tensor.eval();
	#print ("The range: ["+str(np.min(array))+":"+str(np.max(array))+"]")
	if(np.size(np.shape(array))==1):
		Record_Array1D(array,name,f)
	else:
		if(np.size(np.shape(array))==2):
			Record_Array2D(array,name,f)
		else:
			if(np.size(np.shape(array))==3):
				Record_Array3D(array,name,f)
			else:
				Record_Array4D(array,name,f)
	f.close();

def Record_Array1D(array,name,f):
	for i in range(np.shape(array)[0]):
		f.write(str(array[i])+"\n");

def Record_Array2D(array,name,f):
	for i in range(np.shape(array)[0]):
		for j in range(np.shape(array)[1]):
			f.write(str(array[i][j])+"\n");

def Record_Array3D(array,name,f):
	for i in range(np.shape(array)[0]):
		for j in range(np.shape(array)[1]):
			for k in range(np.shape(array)[2]):
				f.write(str(array[i][j][k])+"\n");

def Record_Array4D(array,name,f):
	for i in range(np.shape(array)[0]):
		for j in range(np.shape(array)[1]):
			for k in range(np.shape(array)[2]):
				for l in range(np.shape(array)[3]):
					f.write(str(array[i][j][k][l])+"\n");

with tf.name_scope('input'): 
	x = tf.placeholder("float", shape=[None, 784])
	y_ = tf.placeholder("float", shape=[None, 10])

def weight_variable(shape):
	initial = tf.truncated_normal(shape, stddev=0.1);
	return tf.Variable(initial)

def bias_variable(shape):
	initial = tf.constant(0.1, shape=shape)
	return tf.Variable(initial)

def conv2d(x, W):
	return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
	return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2,1], padding='SAME')

#First Convolutional Layer
with tf.name_scope('1st_CNN'): 
	W_conv1 = weight_variable([3, 3, 1, 16])
	b_conv1 = bias_variable([16])
	x_image = tf.reshape(x, [-1,28,28,1])
	h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
	h_pool1 = max_pool_2x2(h_conv1)

#Second Convolutional Layer
with tf.name_scope('2rd_CNN'): 
	W_conv2 = weight_variable([3, 3, 16, 32])
	b_conv2 = bias_variable([32])
	h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)
	h_pool2 = max_pool_2x2(h_conv2)

#Densely Connected Layer
with tf.name_scope('Densely_NN'): 
	W_fc1 = weight_variable([ 7* 7* 32, 128])
	b_fc1 = bias_variable([128])
	h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*32])
	h_fc1=tf.nn.relu(tf.matmul(h_pool2_flat , W_fc1) + b_fc1)

#Dropout
with tf.name_scope('Dropout'):
	keep_prob = tf.placeholder("float")
	h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

#Readout Layer
with tf.name_scope('Softmax'):
	W_fc2 = weight_variable([128, 10])
	b_fc2 = bias_variable([10])
	y_conv=tf.nn.softmax(tf.matmul(h_fc1_drop, W_fc2) + b_fc2)

with tf.name_scope('Loss'):
	cross_entropy = -tf.reduce_sum(y_*tf.log(y_conv))

with tf.name_scope('Train'):
	train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

with tf.name_scope('Accuracy'):
	correct_prediction = tf.equal(tf.argmax(y_conv ,1), tf.argmax(y_,1))
	accuracy = tf.reduce_mean(tf.cast(correct_prediction , "float"))

merged = tf.merge_all_summaries();
writer = tf.train.SummaryWriter("logs/",sess.graph) 

tf.initialize_all_variables().run()

for i in range(10000):
	batch = mnist.train.next_batch(50);
	if i%20 == 0:
		train_accuracy = accuracy.eval(feed_dict={x:batch[0], y_: batch[1], keep_prob:1.0});
		print("step %d, training accuracy %g"%(i, train_accuracy));
	train_step.run(feed_dict={x: batch[0], y_: batch[1], keep_prob:0.5});

print("test accuracy %g"%accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))

Record_Tensor(W_conv1,"W_conv1")
Record_Tensor(b_conv1,"b_conv1")
Record_Tensor(W_conv2,"W_conv2")
Record_Tensor(b_conv2,"b_conv2")
Record_Tensor(W_fc1,"W_fc1")
Record_Tensor(b_fc1,"b_fc1")
Record_Tensor(W_fc2,"W_fc2")
Record_Tensor(b_fc2,"b_fc2")
sess.close()

```

使用命令：**python3 mnist_int16_test.py** ，开始进行训练，训练时，会从MNIST_data文件夹中读取训练数据，然后将训练的参数存储到record文件夹中，此时生成的权重参数的数据格式为xxx.dat文件，在硬件实现中，这种格式的权重不能被引用，需要将其转化成xxx.bin格式的文件。

dat2bin.c文件被用来转换权重文件的格式：

```c
#include <stdio.h>
#include <malloc.h>
#include <stdlib.h>

char* filename_to_bin(char *filename_i)
{
	int filename_length=0;
	while(filename_i[filename_length]!='\0') filename_length++;
	//printf("filename length=%d\n",filename_length);
	char *filename_bin=(char *)malloc(filename_length+1);
	int i=0;
	while(!(filename_i[i]=='.' && filename_i[i+1]=='d'&& filename_i[i+2]=='a' && filename_i[i+3]=='t' && filename_i[i+4]=='\0'))//not '.dat\0'
	{
		if(i==filename_length-1)
		{
			free(filename_bin);
			filename_bin=NULL;
			return filename_bin;
		}
		filename_bin[i]=filename_i[i];
		i++;
	}
	filename_bin[i]='.';filename_bin[i+1]='b';filename_bin[i+2]='i';filename_bin[i+3]='n';
	filename_bin[i+4]='\0';
	return filename_bin;
}

int main(int argc, char *argv[])
{
	for(int i=1;i<argc;i++)
	{
		char *filename_i=argv[i];
		char *filename_bin=filename_to_bin(filename_i);
		if(filename_bin==NULL)
		{
			printf("%s is not a dat file\n",filename_i);
			break;
		}

		FILE *fp_IN;
		if((fp_IN=fopen(filename_i,"r"))==NULL) 
		{  
			printf("File %s cannot be opened/n",filename_i);   
			break;   
		}

		FILE* fp_OUT = fopen(filename_bin,"wb");
		if (fp_OUT == NULL) 
		{
			printf("File %s cannot be created/n",filename_bin);  
			break;
		}

		char str[20];
		while(fgets(str,20,fp_IN))
		{
			//printf("%s=%f\n",str,atof(str));
			float tp=atof(str);
			fwrite(&tp,sizeof(float),1,fp_OUT);
		};

		fclose(fp_IN);
		fflush(fp_OUT);
		fclose(fp_OUT);
		free(filename_bin);
	}
	return 0;
}
```

执行此程序的命令为： **./dat2bin ./MNIST/record/* ** （后面的为xxx.bat文件的路径，表示使用路径下文件夹中的全部文件）

另外，可以在python代码中直接将权重文件输出成二进制bin文件格式，用函数tofile（待亲自尝试）

### 2、基于HLS的卷积和池化单元设计

（1）首先应该明确，应该设计通用的卷积和池化单元，而不是为每一层卷积和池化单独设计模块,即应该实现两个子函数，一个负责卷积和Relu，一个负责池化。   

（2）全连接层是卷积的特殊形式，所以这里可以用卷积函数实现全连接层

若输入7x7x32，即图片大小7x7，有32个通道。

在全连接层中，先将其展开为一个一维向量（比如行向量），大小为1568。设全连接层有128个单元，此时权重矩阵维度为128*1568（1568列，128行）。

![](/MNIST8.png) ![](/MNIST9.png)

全连接层的输出为[x,x,x,x,x.........x]，一共输出128个数据。

现在保持输入7x7x32不变，经过128个7x7x32的卷积核，输出为1x1x128，与全连接层相同。

![](/MNIST10.png)



（3）池化运算函数

其源码在目录/PYNQ/hls/下

**原理解析：** 

1、ap_uint是HLS中自带数据类型，可以自定义任意位宽的数据。在C语言中，int类型只能有8位或者16位；在verilog中，位宽可以自定义。​	

2、关于数组指针的索引

为了使卷积和池化函数具有通用性，对于其中的feature_in[ ]和feature_out[ ]等变量使用数组地址索引（在python中是将其写成三维数组的形式，如feature_in [ H] [W] [C]）,其索引的方式如下：

![](/MNIST11.png)                                       ![](/MNIST12.png)

- 在大小为4x4的二维数组中（即数组的H=4,W=4）：

假设x的坐标为：feature[i] [j]，若用数组索引的方式为：feature[ ixW+j ]。

例如，x的坐标为feature[2] [1]，则用数组指针的方式为：feature[2x4+1=9]

- 在大小为4x4，通道数为16的三维数组中（即数组的H=4，W=4，Cout=16）：

假设x的坐标为：feature[i] [j] [c]，若用数组地址索引的方式为：feature[ i x W x Cout + j x Cout + c ]，

例如，x的坐标为feature[2] [2] [3]，则用数组指针的方式为：feature[ 2x4x16+2x16+16=163 ]

注：这里三维数组中的索引顺序为：通道1行1列1，通道2行1列1,.......通道16行1列1，通道1行1列2，通道2行1列2，...



pool_core.cpp           

```c++
#include "pool_core.h"

#define max(a,b) ((a>b)?a:b)
#define min(a,b) ((a>b)?b:a)

void Pool(ap_uint<16> CHin,ap_uint<16> Hin,ap_uint<16> Win,
		ap_uint<8> Kx,ap_uint<8> Ky,ap_uint<2> mode,
		Dtype_f feature_in[],Dtype_f feature_out[]
	)
	//mode:池化的模式 0:MEAN, 1:MIN, 2:MAX
	//Kx，Ky表示池化窗口的大小
{
	#pragma HLS INTERFACE m_axi depth=4294967295 port=feature_out offset=slave
	#pragma HLS INTERFACE m_axi depth=4294967295 port=feature_in offset=slave
	#pragma HLS INTERFACE s_axilite port=Win
	#pragma HLS INTERFACE s_axilite port=Kx
	#pragma HLS INTERFACE s_axilite port=Hin
	#pragma HLS INTERFACE s_axilite port=mode
	#pragma HLS INTERFACE s_axilite port=Ky
	#pragma HLS INTERFACE s_axilite port=CHin
	#pragma HLS INTERFACE s_axilite port=return
	ap_uint<16> Hout,Wout;
	Wout=Win/Kx;
	Hout=Hin/Ky;

	for(int c=0;c<CHin;c++)  
		for(int i=0;i<Hout;i++)
			for(int j=0;j<Wout;j++)
			{
				Dtype_f sum;
				if(mode==0)
					sum=0;
				else
					if(mode==1)
						sum=99999999999999999;
					else
						sum=-99999999999999999;
				for(int ii=0;ii<Ky;ii++)
					for(int jj=0;jj<Kx;jj++)
					{
						ap_int<16> h=i*Ky+ii;
						ap_int<16> w=j*Kx+jj;
						switch(mode)
						{
							case 0:{sum+=feature_in[h*CHin*Win+w*CHin+c];break;}
							case 1:{sum=min(sum,feature_in[h*CHin*Win+w*CHin+c]);break;}
							case 2:{sum=max(sum,feature_in[h*CHin*Win+w*CHin+c]);break;}
							default:break;
						}
					}
				if(mode==0)
					sum=sum/(Kx*Ky);
				feature_out[i*Wout*CHin+j*CHin+c]=sum;
			}
}
```

对池化函数代码的解释：

![](/MNIST13.png)

如对上述特征图进行池化运算：

- 函数中的参数Kx和Ky表示池化窗口的大小，Wout=Win/Kx和Hout=Hin/Ky操作，是为了计算出对于Win x Hin大小的特征图在一行和一列上需要进行多少次池化运算，即上图中红色窗口的数量，同时也表示池化完成后特征图的大小为Wout x Hout x CHin。

- 一开始的三个for循环，是为了依次遍历整个数组的通道、行和列（即先选定第一个通道，再选定其每一行的每一列，然后再对第二个通道进行操作。。。）。

- （假设Kx=Ky=2）池化运算的具体操作为：对上图红色框中的四个数进行计算，得到他们的均值、最大值或者最小值。第一次进行池化运算的四个数的坐标应该为：[0] [0]， [0] [1]， [1] [0]， [1] [1] ，在函数中先通过循环for(int ii=0;ii<Ky;ii++)等来确定，池化运算需要选定几个数据进行计算。然后通过h和w两个变量来定位池化操作需要的数据的具体坐标， h=i*Ky+ii，很明显，数据的坐标，是根据池化的窗口大小决定的。

- 每进行完依次池化运算之后，要把结果输出一次，即feature_out[i\*Wout\*CHin+j\*CHin+c]=sum

  输出的顺序与三维数组索引的顺序一样。

![](/MNIST14.jpg)

pool_core.h

```c++
#ifndef __POOL_CORE_H__
#define __POOL_CORE_H__

#include <ap_int.h>
#include <iostream>

typedef float Dtype_ f;  

void Pool(ap_uint<16> CHin,ap_uint<16> Hin,ap_uint<16> Win,
		ap_uint<8> Kx,ap_uint<8> Ky,ap_uint<2> mode,
		Dtype_f feature_in[],Dtype_f feature_out[]
	);//mode: 0:MEAN, 1:MIN, 2:MAX

#endif
```

（4）卷积运算函数

conv_core.cpp 

```c++
#include "conv_core.h"

//Feature: [H][W][C]
//kernel: [Ky][Kx][CHin][CHout]

void Conv(ap_uint<16> CHin,ap_uint<16> Hin,ap_uint<16> Win,ap_uint<16> CHout,
		ap_uint<8> Kx,ap_uint<8> Ky,ap_uint<8> Sx,ap_uint<8> Sy,ap_uint<1> mode,ap_uint<1> relu_en,
		Dtype_f feature_in[],Dtype_w W[],Dtype_w bias[],Dtype_f feature_out[]
	)
	//Dtype_f和Dtype_w是宏定义的数据类型，属于float数据类型
	//各个参数的意义
	//CHin：输入通道数，Hin：输入图高，Win：输入图宽，CHout：输出通道数
	//Kx，Ky：卷积核大小，Sx，Sy：在x，y方向上的步长
	//mode：padding的模式， 0:VALID, 1:SAME
	//relu_en：使能激活
	//Dtype_f feature_in[]：对输入特征的索引，Dtype_w W[]：对输入权重的索引
	//Dtype_w bias[ ]：对输入偏置的索引，Dtype_f feature_out[ ]：对输出特征的索引
	
{
	//#pragma HLS PIPELINE enable_flush
	#pragma HLS INTERFACE m_axi depth=4294967295 port=feature_out offset=slave
	#pragma HLS INTERFACE m_axi depth=4294967295 port=bias offset=slave
	#pragma HLS INTERFACE m_axi depth=4294967295 port=W offset=slave
	#pragma HLS INTERFACE m_axi depth=4294967295 port=feature_in offset=slave
	#pragma HLS INTERFACE s_axilite port=relu_en
	#pragma HLS INTERFACE s_axilite port=CHout
	#pragma HLS INTERFACE s_axilite port=Sx
	#pragma HLS INTERFACE s_axilite port=Hin
	#pragma HLS INTERFACE s_axilite port=CHin
	#pragma HLS INTERFACE s_axilite port=Kx
	#pragma HLS INTERFACE s_axilite port=mode
	#pragma HLS INTERFACE s_axilite port=Sy
	#pragma HLS INTERFACE s_axilite port=Ky
	#pragma HLS INTERFACE s_axilite port=Win
	#pragma HLS INTERFACE s_axilite port=return

	ap_uint<8> pad_x,pad_y;
	if(mode==0)
	{
		pad_x=0;pad_y=0;  //VALID模式时，不需要补特征图
	}
	else
	{
		pad_x=(Kx-1)/2;pad_y=(Ky-1)/2;
                //padding为SAME模式时，补全数值的大小
	}
	ap_uint<16> Hout,Wout;
	Wout=(Win+2*pad_x-Kx)/Sx+1;
	Hout=(Hin+2*pad_y-Ky)/Sy+1;   //输出特征图大小

	for(int cout=0;cout<CHout;cout++)
		for(int i=0;i<Hout;i++)
			for(int j=0;j<Wout;j++)   //相当于遍历卷积输出，一个一个计算卷积后的值
			{
				Dtype_acc sum=0; 
				for(int ii=0;ii<Ky;ii++)
					for(int jj=0;jj<Kx;jj++)
					{
						ap_int<16> h=i*Sy-pad_y+ii;
						ap_int<16> w=j*Sx-pad_x+jj;
						if(h>=0 && w>=0 && h<Hin && w<Win)
						{
							for(int cin=0;cin<CHin;cin++)
							{
		//Feature [H][W][C]
		//kernel: [Ky][Kx][CHin][CHout]
		//Dtype_mul tp=feature_in[h][w][cin]*w[ii][jj][cin][cout];
		//std::cout<<"h:"<<h<<",w"<<w<<",cin"<<cin<<"\n";
             //std::cout<<"feature_in["<<h*CHin*Win+w*CHin+cin<<"]*W[" <<ii*Kx*CHin*CHout+jj*CHin*CHout+cin*CHout+cout<<"]\n";
         Dtype_mul tp=feature_in[h*CHin*Win+w*CHin+cin]*W[ii*Kx*CHin*CHout+jj*CHin*CHout+cin*CHout+cout];
								sum+=tp;
							}
						}
					}

				sum+=bias[cout]; //每个卷积核计算完之后，成为输出的一个通道，为其加一个偏置
				if(relu_en & sum<0)  //若relu_en=0，则将卷积输出归零
					sum=0;
				//feature_out[i][j][cout]=sum;
				feature_out[i*Wout*CHout+j*CHout+cout]=sum;
			}
}
```

对卷积函数代码的解释：

- 若特征图大小为nxn，卷积核大小为fxf，步长为s，

  当采用padding=SAME模式，则需要补的值为：p=$\frac{f-1}{2}$ 

  卷积之后输出的特征图大小为：$\lfloor$ $\frac{n+2xp-f}{s}$ x $\frac{n+2xp-f}{s}$  $\rfloor$ ，$\lfloor$  $\rfloor$ 是向下取整  

- 特征图矩阵为Feature [H] [W] [C]，其中H，W为特征图大小，C为通道数

  权重矩阵为kernel: [Ky] [Kx] [CHin] [CHout]，其中Kx和ky为卷积核的大小，CHin为卷积核通道数，其与特征图的通道数相同。CHout为卷积操作后输出特征图的通道数，同时也是卷积核的数量。

  在函数中，用for循环实现两个多维矩阵的相乘

  

conv_core.h

```c++
#ifndef __CONV_CORE_H__
#define __CONV_CORE_H__

#include <ap_int.h>
#include <iostream>

using namespace std;

//typedef ap_int<8> Dtype_f;
//typedef ap_int<8> Dtype_w;
//typedef ap_int<15> Dtype_mul;
//typedef ap_int<32> Dtype_acc;

typedef float Dtype_f;
typedef float Dtype_w;
typedef float Dtype_mul;
typedef float Dtype_acc;

void Conv(ap_uint<16> CHin,ap_uint<16> Hin,ap_uint<16> Win,ap_uint<16> CHout,
		ap_uint<8> Kx,ap_uint<8> Ky,ap_uint<8> Sx,ap_uint<8> Sy,ap_uint<1> mode,ap_uint<1> relu_en,
		Dtype_f feature_in[],Dtype_w W[],Dtype_w bias[],Dtype_f feature_out[]
	);//mode: 0:VALID, 1:SAME

#endif
```



（5）关于卷积和池化中的Directive约束

卷积函数中的Directive （池化函数与卷积类似）

![](/MNIST5.png)

**绿色框中的约束，可以使CPU控制函数的启停。** 

**蓝色框中的这些约束，可以为整个运算单元生成AXI4-Lite的slave接口，CPU可以通过这个接口，对卷积函数进行配置。**

**红色框中的这些约束，可以为整个运算单元生成AXI4的master接口，使得卷积函数可以主动从DDR中获取所需数据。**

（注：对于红色框中的depth参数，设置一个足够大的数即可）。

![](/hls_conv_ip.png)

### 3、MNIST网络的硬件搭建

**简单架构：** 

![](/MNIST4.png)

**基于ZYNQ的SOC平台搭建：**

在ZYNQ架构中,DDR在ARM一端，通过AXI4接口，运算单元可以访问属于ARM的DDR。

添加conv和pool的路径（/PYNQ/hls/conv_core），然后在Blockdesign中添加这两个IP

在zynq IP设置中，可以选择一个HP口，也可以选择两个，在下图中，用了两个HP口。

![](/MNIST7.png)



### 4、SOC实现的软件控制部分

#### （1）ZYNQ版本





#### （2）PYNQ版本

##### 1、卷积和池化函数的python驱动

实现的代码为：helloworld.py，另外conv.py和pool.py分别是卷积和池化的python驱动,这里的python驱动，是根据HLS为卷积和池化函数生成的c语言版本的驱动修改而来。

c语言驱动的路径为：PYNQ/hls/pool_core/solution1/impl/ip/drivers/Pool_v1_0/src/目录下的文件所确定

例如，在xpool.c中，有如下操作

```c++
void XPool_Set_CHin_V(XPool *InstancePtr, u32 Data) {
    Xil_AssertVoid(InstancePtr != NULL);
    Xil_AssertVoid(InstancePtr->IsReady == XIL_COMPONENT_IS_READY);

    XPool_WriteReg(InstancePtr->Axilites_BaseAddress, XPOOL_AXILITES_ADDR_CHIN_V_DATA, Data);
}
```

其在xpool.h中对应为：

```c++
void XPool_Set_CHin_V(XPool *InstancePtr, u32 Data);
```

它的意思是，向地址为XPOOL_AXILITES_ADDR_CHIN_V_DATA的寄存器写入1

XPOOL_AXILITES_ADDR_CHIN_V_DATA的具体值在xpool_hw.h文件中

```c++
#define XPOOL_AXILITES_ADDR_CHIN_V_DATA      0x10
```

对应到如下的python驱动中为：

```python
  pool.write(0x10,feature_in.shape[2]);
```

对其余寄存器的操作，均这样做

在下面的代码中，**def RunConv**和**def RunPool**都是按照上面的方法书写

```python

def RunConv(conv,Kx,Ky,Sx,Sy,mode,relu_en,feature_in,W,bias,feature_out):
    conv.write(0x10,feature_in.shape[2]);
    conv.write(0x18,feature_in.shape[0]);
    conv.write(0x20,feature_in.shape[1]);
    conv.write(0x28,feature_out.shape[2]);
    conv.write(0x30,Kx);
    conv.write(0x38,Ky);
    conv.write(0x40,Sx);
    conv.write(0x48,Sy);
    conv.write(0x50,mode);
    conv.write(0x58,relu_en);
    conv.write(0x60,feature_in.physical_address);
    conv.write(0x68,W.physical_address);
    conv.write(0x70,bias.physical_address);
    conv.write(0x78,feature_out.physical_address);
    conv.write(0, (conv.read(0)&0x80)|0x01 );
    tp=conv.read(0)
    while not ((tp>>1)&0x1):
        tp=conv.read(0);
    #print(tp);
#调用这个函数之后，先对参数进行配置，然后start（0地址寄存器）开始执行，然后运行IsDone函数，查看是否执行完毕
def RunPool(pool,Kx,Ky,mode,feature_in,feature_out):
    pool.write(0x10,feature_in.shape[2]); 
    #对于feature_in数组，有三个维度，第0维是高度，第一维是宽度，第2维是输入通道数
    pool.write(0x18,feature_in.shape[0]);
    pool.write(0x20,feature_in.shape[1]);
    pool.write(0x28,Kx);
    pool.write(0x30,Ky);
    pool.write(0x38,mode);
    pool.write(0x40,feature_in.physical_address);
    pool.write(0x48,feature_out.physical_address);
    pool.write(0, (pool.read(0)&0x80)|0x01 ); #完全按照c驱动改编
    while not ((pool.read(0)>>1)&0x1):   #轮询XPool_IsDone函数，看是否执行完毕
        pass;

```

##### 2、MNIST网络整体实现代码

将整个overlay文件夹复制到板卡上，然后先取得板卡的root权限 （命令：su root，密码为xilinx）

切换到/home/xilinx/overlay/mnist目录下，使用命令**python3 helloworld.py**执行程序。

==对以下代码的详细讲解，待续。。。。==

```python
import time
from pynq import Overlay
import numpy as np
from pynq import Xlnk
import struct
from scipy.misc import imread
import cv2

def readbinfile(filename,size):
    f = open(filename, "rb")
    z=[]
    for j in range(size):
        data = f.read(4)
        data_float = struct.unpack("f", data)[0]
        z.append(data_float)
    f.close()
    z = np.array(z)
    return z

def RunConv(conv,Kx,Ky,Sx,Sy,mode,relu_en,feature_in,W,bias,feature_out):
    conv.write(0x10,feature_in.shape[2]);
    conv.write(0x18,feature_in.shape[0]);
    conv.write(0x20,feature_in.shape[1]);
    conv.write(0x28,feature_out.shape[2]);
    conv.write(0x30,Kx);
    conv.write(0x38,Ky);
    conv.write(0x40,Sx);
    conv.write(0x48,Sy);
    conv.write(0x50,mode);
    conv.write(0x58,relu_en);
    conv.write(0x60,feature_in.physical_address);
    conv.write(0x68,W.physical_address);
    conv.write(0x70,bias.physical_address);
    conv.write(0x78,feature_out.physical_address);
    conv.write(0, (conv.read(0)&0x80)|0x01 );
    tp=conv.read(0)
    while not ((tp>>1)&0x1):
        tp=conv.read(0);
    #print(tp);

def RunPool(pool,Kx,Ky,mode,feature_in,feature_out):
    pool.write(0x10,feature_in.shape[2]);
    pool.write(0x18,feature_in.shape[0]);
    pool.write(0x20,feature_in.shape[1]);
    pool.write(0x28,Kx);
    pool.write(0x30,Ky);
    pool.write(0x38,mode);
    pool.write(0x40,feature_in.physical_address);
    pool.write(0x48,feature_out.physical_address);
    pool.write(0, (pool.read(0)&0x80)|0x01 );
    while not ((pool.read(0)>>1)&0x1):
        pass;

#Conv1
IN_WIDTH1=28
IN_HEIGHT1=28
IN_CH1=1

KERNEL_WIDTH1=3
KERNEL_HEIGHT1=3
X_STRIDE1=1
Y_STRIDE1=1

RELU_EN1=1
MODE1=1  #0:VALID, 1:SAME
if(MODE1):
    X_PADDING1=int((KERNEL_WIDTH1-1)/2)
    Y_PADDING1=int((KERNEL_HEIGHT1-1)/2)
else:
    X_PADDING1=0
    Y_PADDING1=0

OUT_CH1=16
OUT_WIDTH1=int((IN_WIDTH1+2*X_PADDING1-KERNEL_WIDTH1)/X_STRIDE1+1)
OUT_HEIGHT1=int((IN_HEIGHT1+2*Y_PADDING1-KERNEL_HEIGHT1)/Y_STRIDE1+1)


#Pool1
MODE11=2  #mode: 0:MEAN, 1:MIN, 2:MAX
IN_WIDTH11=OUT_WIDTH1
IN_HEIGHT11=OUT_HEIGHT1
IN_CH11=OUT_CH1

KERNEL_WIDTH11=2
KERNEL_HEIGHT11=2

OUT_CH11=IN_CH11
OUT_WIDTH11=int(IN_WIDTH11/KERNEL_WIDTH11)
OUT_HEIGHT11=int(IN_HEIGHT11/KERNEL_HEIGHT11)

#Conv2
IN_WIDTH2=OUT_WIDTH11
IN_HEIGHT2=OUT_HEIGHT11
IN_CH2=OUT_CH11

KERNEL_WIDTH2=3
KERNEL_HEIGHT2=3
X_STRIDE2=1
Y_STRIDE2=1

RELU_EN2=1
MODE2=1  #0:VALID, 1:SAME
if(MODE2):
    X_PADDING2=int((KERNEL_WIDTH2-1)/2)
    Y_PADDING2=int((KERNEL_HEIGHT2-1)/2)
else:
    X_PADDING2=0
    Y_PADDING2=0

OUT_CH2=32
OUT_WIDTH2=int((IN_WIDTH2+2*X_PADDING2-KERNEL_WIDTH2)/X_STRIDE2+1)
OUT_HEIGHT2=int((IN_HEIGHT2+2*Y_PADDING2-KERNEL_HEIGHT2)/Y_STRIDE2+1)

#Pool2
MODE21=2  #mode: 0:MEAN, 1:MIN, 2:MAX
IN_WIDTH21=OUT_WIDTH2
IN_HEIGHT21=OUT_HEIGHT2
IN_CH21=OUT_CH2

KERNEL_WIDTH21=2
KERNEL_HEIGHT21=2

OUT_CH21=IN_CH21
OUT_WIDTH21=int(IN_WIDTH21/KERNEL_WIDTH21)
OUT_HEIGHT21=int(IN_HEIGHT21/KERNEL_HEIGHT21)

#Fc1
IN_WIDTH3=OUT_WIDTH21
IN_HEIGHT3=OUT_HEIGHT21
IN_CH3=OUT_CH21

KERNEL_WIDTH3=7
KERNEL_HEIGHT3=7
X_STRIDE3=1
Y_STRIDE3=1

RELU_EN3=1
MODE3=0  #0:VALID, 1:SAME
if(MODE3):
    X_PADDING3=int((KERNEL_WIDTH3-1/2))
    Y_PADDING3=int((KERNEL_HEIGHT3-1)/2)
else:
    X_PADDING3=0
    Y_PADDING3=0

OUT_CH3=128
OUT_WIDTH3=int((IN_WIDTH3+2*X_PADDING3-KERNEL_WIDTH3)/X_STRIDE3+1)
OUT_HEIGHT3=int((IN_HEIGHT3+2*Y_PADDING3-KERNEL_HEIGHT3)/Y_STRIDE3+1)

#Fc2
IN_WIDTH4=OUT_WIDTH3
IN_HEIGHT4=OUT_HEIGHT3
IN_CH4=OUT_CH3

KERNEL_WIDTH4=1
KERNEL_HEIGHT4=1
X_STRIDE4=1
Y_STRIDE4=1

RELU_EN4=1
MODE4=0  #0:VALID, 1:SAME
if(MODE4):
    X_PADDING4=int((KERNEL_WIDTH4-1/2))
    Y_PADDING4=int((KERNEL_HEIGHT4-1)/2)
else:
    X_PADDING4=0
    Y_PADDING4=0

OUT_CH4=10
OUT_WIDTH4=int((IN_WIDTH4+2*X_PADDING4-KERNEL_WIDTH4)/X_STRIDE4+1)
OUT_HEIGHT4=int((IN_HEIGHT4+2*Y_PADDING4-KERNEL_HEIGHT4)/Y_STRIDE4+1)

xlnk=Xlnk();

ol=Overlay("ai.bit")
ol.ip_dict
ol.download()
conv=ol.Conv_0
pool=ol.Pool_0
print("Overlay download finish");

#input image
image=xlnk.cma_array(shape=(IN_HEIGHT1,IN_WIDTH1,IN_CH1),cacheable=0,dtype=np.float32)

#conv1
W_conv1=xlnk.cma_array(shape=(KERNEL_HEIGHT1,KERNEL_WIDTH1,IN_CH1,OUT_CH1),cacheable=0,dtype=np.float32)
b_conv1=xlnk.cma_array(shape=(OUT_CH1),cacheable=0,dtype=np.float32)
h_conv1=xlnk.cma_array(shape=(OUT_HEIGHT1,OUT_WIDTH1,OUT_CH1),cacheable=0,dtype=np.float32)
h_pool1=xlnk.cma_array(shape=(OUT_HEIGHT11,OUT_WIDTH11,OUT_CH11),cacheable=0,dtype=np.float32)

#conv2
W_conv2=xlnk.cma_array(shape=(KERNEL_HEIGHT2,KERNEL_WIDTH2,IN_CH2,OUT_CH2),cacheable=0,dtype=np.float32)
b_conv2=xlnk.cma_array(shape=(OUT_CH2),cacheable=0,dtype=np.float32)
h_conv2=xlnk.cma_array(shape=(OUT_HEIGHT2,OUT_WIDTH2,OUT_CH2),cacheable=0,dtype=np.float32)
h_pool2=xlnk.cma_array(shape=(OUT_HEIGHT21,OUT_WIDTH21,OUT_CH21),cacheable=0,dtype=np.float32)

#fc1
W_fc1=xlnk.cma_array(shape=(KERNEL_HEIGHT3, KERNEL_WIDTH3, IN_CH3, OUT_CH3),cacheable=0,dtype=np.float32)
b_fc1=xlnk.cma_array(shape=(OUT_CH3),cacheable=0,dtype=np.float32)
h_fc1=xlnk.cma_array(shape=(OUT_HEIGHT3,OUT_WIDTH3,OUT_CH3),cacheable=0,dtype=np.float32)

#fc2
W_fc2=xlnk.cma_array(shape=(KERNEL_HEIGHT4, KERNEL_WIDTH4, IN_CH4, OUT_CH4),cacheable=0,dtype=np.float32)
b_fc2=xlnk.cma_array(shape=(OUT_CH4),cacheable=0,dtype=np.float32)
h_fc2=xlnk.cma_array(shape=(OUT_HEIGHT4,OUT_WIDTH4,OUT_CH4),cacheable=0,dtype=np.float32)




#Initialize W, bias

w_conv1=readbinfile("/home/xilinx/overlay/mnist/data/W_conv1.bin",KERNEL_HEIGHT1*KERNEL_WIDTH1*IN_CH1*OUT_CH1)
w_conv1=w_conv1.reshape((KERNEL_HEIGHT1,KERNEL_WIDTH1,IN_CH1,OUT_CH1))
for i in range(KERNEL_HEIGHT1):
    for j in range(KERNEL_WIDTH1):
        for k in range(IN_CH1):
        	for l in range(OUT_CH1):
        		W_conv1[i][j][k][l]=w_conv1[i][j][k][l]
B_conv1=readbinfile("/home/xilinx/overlay/mnist/data/b_conv1.bin",OUT_CH1)
for i in range(OUT_CH1):
	b_conv1[i]=B_conv1[i]

w_conv2=readbinfile("/home/xilinx/overlay/mnist/data/W_conv2.bin",KERNEL_HEIGHT2*KERNEL_WIDTH2*IN_CH2*OUT_CH2)
w_conv2=w_conv2.reshape((KERNEL_HEIGHT2,KERNEL_WIDTH2,IN_CH2,OUT_CH2))
for i in range(KERNEL_HEIGHT2):
    for j in range(KERNEL_WIDTH2):
        for k in range(IN_CH2):
        	for l in range(OUT_CH2):
        		W_conv2[i][j][k][l]=w_conv2[i][j][k][l]
B_conv2=readbinfile("/home/xilinx/overlay/mnist/data/b_conv2.bin",OUT_CH2)
for i in range(OUT_CH2):
	b_conv2[i]=B_conv2[i]

w_fc1=readbinfile("/home/xilinx/overlay/mnist/data/W_fc1.bin",KERNEL_HEIGHT3*KERNEL_WIDTH3*IN_CH3*OUT_CH3)
w_fc1=w_fc1.reshape((KERNEL_HEIGHT3,KERNEL_WIDTH3,IN_CH3,OUT_CH3))
for i in range(KERNEL_HEIGHT3):
    for j in range(KERNEL_WIDTH3):
        for k in range(IN_CH3):
        	for l in range(OUT_CH3):
        		W_fc1[i][j][k][l]=w_fc1[i][j][k][l]
B_fc1=readbinfile("/home/xilinx/overlay/mnist/data/b_fc1.bin",OUT_CH3)
for i in range(OUT_CH3):
	b_fc1[i]=B_fc1[i]

w_fc2=readbinfile("/home/xilinx/overlay/mnist/data/W_fc2.bin",KERNEL_HEIGHT4*KERNEL_WIDTH4*IN_CH4*OUT_CH4)
w_fc2=w_fc2.reshape((KERNEL_HEIGHT4,KERNEL_WIDTH4,IN_CH4,OUT_CH4))
for i in range(KERNEL_HEIGHT4):
    for j in range(KERNEL_WIDTH4):
        for k in range(IN_CH4):
        	for l in range(OUT_CH4):
        		W_fc2[i][j][k][l]=w_fc2[i][j][k][l]
B_fc2=readbinfile("/home/xilinx/overlay/mnist/data/b_fc2.bin",OUT_CH4)
for i in range(OUT_CH4):
	b_fc2[i]=B_fc2[i]

print("Finish initial")

while(1):
	while(1):
		g=input("input enter to continue")
		break
	image1=cv2.imread("/home/xilinx/overlay/mnist/data/1.jpg",cv2.IMREAD_GRAYSCALE).astype(np.float32)
	print("Read image")
	#image1=image1.reshape((IN_HEIGHT1,IN_WIDTH1,IN_CH1))
	for i in range(IN_HEIGHT1):
		for j in range(IN_WIDTH1):
			for k in range(IN_CH1):
				image[i][j][k]=(255-image1[i][j])/255
	print("Finish reading image")
	#conv1
	RunConv(conv,KERNEL_WIDTH1,KERNEL_HEIGHT1,X_STRIDE1,Y_STRIDE1,MODE1,RELU_EN1,image,W_conv1,b_conv1,h_conv1)
	RunPool(pool, KERNEL_WIDTH11, KERNEL_HEIGHT11, MODE11, h_conv1, h_pool1)
	# conv2
	RunConv(conv, KERNEL_WIDTH2, KERNEL_HEIGHT2, X_STRIDE2, Y_STRIDE2, MODE2, RELU_EN2, h_pool1, W_conv2, b_conv2,
	        h_conv2)
	RunPool(pool, KERNEL_WIDTH21, KERNEL_HEIGHT21, MODE21, h_conv2, h_pool2)
	# fc1
	RunConv(conv, KERNEL_WIDTH3, KERNEL_HEIGHT3, X_STRIDE3, Y_STRIDE3, MODE3, RELU_EN3, h_pool2, W_fc1, b_fc1,
	        h_fc1)
	# fc2
	RunConv(conv, KERNEL_WIDTH4, KERNEL_HEIGHT4, X_STRIDE4, Y_STRIDE4, MODE4, RELU_EN4, h_fc1, W_fc2, b_fc2,
	        h_fc2)

	print("Hardware run finish")
	MAX = h_fc2[0][0][0]
	result=0
	for i in range(1,OUT_CH4):
	    if(h_fc2[0][0][i]>MAX):
	        MAX=h_fc2[0][0][i]
	        result=i
	print("The number you write is "+str(result))
```



