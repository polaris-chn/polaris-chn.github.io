---
layout: post
title: Verilog编程-3. 阻塞与非阻塞
categories: Verilog
description: 对于阻塞与非阻塞的详细解答
keywords: Verilog
---

# Verilog编程-3. 阻塞与非阻塞

## 1. 背景

​		在Verilog初学阶段，阻塞与非阻塞赋值是一个理解的难点，其关键在于没有将相关Verilog代码映射成为具体电路来进行思考。阻塞赋值模拟的是组合逻辑电路，其主要特点是信号在组合逻辑电路中进行串行传递，信号必须经过前一个组合逻辑单元才能传递到后一个组合逻辑单元，而阻塞赋值语句，在同一个块中前一句执行完毕才可以执行后一语句，与组合逻辑电路行为相同；非阻塞赋值模拟的是时序逻辑电路，时序逻辑电路最重要的一个特点是在时钟的驱动下，在每一个时钟周期前一个时序单元的数值都可以向后一个时序单元传递，即在单元与单元之间信号传递是并行的，而非阻塞赋值语句，在同一个块中可以并行执行，没有前后顺序之分，同时序逻辑电路特点相同。阻塞赋值使用`=`操作符来表示，非阻塞赋值使用`<=`操作符来表示。



## 2. 设计思路

​		在Verilog编程中，使用移位寄存器可以清晰表示出阻塞/非阻塞的区别，对于非阻塞电路，多次的移位表示每一个寄存器都会存在，则会综合成为一个寄存器链；而对于阻塞电路，综合器对于多次的移位则会综合成为一个触发器而不是一个寄存器链，因为没有时钟驱动，信号在一个链中的传递可以简化成在一个触发器中传递。在《Verilog HDL高级数字设计》（第二版）5.8.2节中有详细描述，但是在该节中表示阻塞赋值语句的排列顺序可以影响最终的电路综合结果，在本实验中结果却不是这样。在两个不同模块中阻塞赋值语句排列顺序不同，但是两者综合结果显示电路是相同的；但是对于非阻塞语句来说，排列顺序当然不会影响最终综合电路。



## 3. 代码

​		所有程序编辑平台为notepad++，综合平台为vivado套件，源文件分别为`test1.v`，`test2.v`，`test3.v`，`test4.v`。文件具体内容如下。

### test1.v

```verilog
`timescale 1ns / 1ps


module test1(
	input clk,
	input rstn,
	input e,
	output reg a
    );
	
	reg b, c, d;
	always@(posedge clk or negedge rstn) begin
		if (!rstn) begin
			a <= 0;
			b <= 0;
			c <= 0;
			d <= 0;
		end
		
		else begin
			d <= e;
			c <= d;
			b <= c;
			a <= b;
		end
	end
endmodule

```



### test2.v

```verilog
`timescale 1ns / 1ps


module test2(
	input clk,
	input rstn,
	input e,
	output reg a
    );
	
	reg b, c, d;
	always@(posedge clk or negedge rstn) begin
		if (!rstn) begin
			a <= 0;
			b <= 0;
			c <= 0;
			d <= 0;
		end
		
		else begin
			a <= b;
			b <= c;
			c <= d;
			d <= e;
		end
	end
endmodule

```



### test3.v

```verilog
`timescale 1ns / 1ps


module test3(
	input rstn,
	input e,
	output reg a
    );
	
	reg b, c, d;
	always@(*) begin
		if (!rstn) begin
			a = 0;
			b = 0;
			c = 0;
			d = 0;
		end
		
		else begin
			d = e;
			c = d;
			b = c;
			a = b;
		end
	end
endmodule

```



### test4.v

```verilog
`timescale 1ns / 1ps


module test4(
	input rstn,
	input e,
	output reg a
    );
	
	reg b, c, d;
	always@(*) begin
		if (!rstn) begin
			a = 0;
			b = 0;
			c = 0;
			d = 0;
		end
		
		else begin
			a = b;
			b = c;
			c = d;
			d = e;
		end
	end
endmodule

```



注意：

##### 1. 程序输入为端口e，输出端口为a；

##### 2. `test1.v`和`test2.v`为非阻塞赋值语句，两者区别在于赋值语句排序不同；`test3.v`和`test4.v`为阻塞赋值语句，两者区别在于赋值语句排序不同；



## 4. 综合结果

​		使用vivado开发套件进行综合，下图分别为`test1.v`，`test2.v`，`test3.v`，`test4.v`程序综合之后形成的电路图。

![](/images/blog/picture6.png)
![](/images/blog/picture7.png)


![](/images/blog/picture8.png)
![](/images/blog/picture9.png)



由前两个图可知，非阻塞赋值语句排序不会影响综合形成的电路；由后两个图可知，阻塞赋值语句排序也不会影响综合形成的电路。至于《Verilog HDL高级数字设计》（第二版）5.8.2节所述是不是正确，还有待讨论。






