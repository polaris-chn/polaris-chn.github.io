---
layout: post
title: Verilog编程-7. 异步FIFO
categories: Verilog
description: 讲清楚异步FIFO的原理和程序实现
keywords: Verilog
---

# Verilog编程-7. 异步FIFO

## 1. 背景
异步FIFO常用于缓冲不同速率时钟域之间的数据交换，可以使得异时钟域数据传输的时序变得宽松，也提高了不同时钟域之间数据传输的效率。对于异步FIFO来说，由于push和pop分别在不同的时钟域，那么最核心的问题就是空满状态的判断。本文将详细介绍异步FIFO的设计思路，并给出相应的代码。



## 2. 设计思路
对于空满状态的判断，最常见的思路就是将FIFO中的读指针和写指针进行比较，然而由于多比特数据在跨时钟域传输的时候，很容易由于亚稳态而产生传输错误，所以对于读写指针来说，不可以直接进行跨时钟域传输。格雷码由于相邻数之间只有1比特的差别，这样可以大大减小亚稳态所产生的影响，所以，异步FIFO基本的设计思路是将读写指针转化为格雷码，再进行跨时钟域传输。同时，在跨时钟域传输的过程中，对于跨时钟域信号进行两级寄存器同步，即打两拍，可以有效减少亚稳态产生的影响。综上可得，异步FIFO跨时钟域信号传输的基本思路是：两级寄存器同步加上格雷码。同时，我们还需要理清一个思路，那就是需要将写时钟域的写指针同步到读时钟域，将同步后的写指针与读时钟域的读指针进行比较产生空信号；将读时钟域的读指针同步到写时钟域，将同步后的读指针与写时钟域的写指针进行比较产生满信号。即在读时钟域产生读空信号，在写时钟域产生写满信号。

![](/images/blog/picture21.jpg)

上图是根据以上设计思路得到的FIFO设计结构图，其基本组成包括一个双端口RAM组成FIFO的存储主体，FIFO的读控制器，FIFO的写控制器。在读写控制器内，分别将读指针和写指针转化为格雷码，再通过sync_r2w和sync_w2r两个同步器，分别同步到写时钟域和读时钟域，这是整个设计的关键点。在格雷码分别同步到对方时钟域时，再和对方时钟域中的格雷码进行比较，则可以在写控制器中产生写满信号，在读控制器中产生读空信号。读写时钟域的时钟信号和复位信号则分别控制着两个同步器。读写控制器还分别产生读地址和写地址信号，进而在FIFO memory中读出数据和写入数据。

还需要注意一点的是，读写指针的实际宽度比FIFO的深度要多1个比特，通过这样的比特扩展，就可以很方便地进行格雷码的直接比较，进而得出FIFO的空满状态。

同时还需要注意到，FIFO memory中的地址是二进制，而跨时钟域进行比较的是该二进制比特扩展后再转化的格雷码。而具体的二进制转换为格雷码的方法，以及格雷码如何进行比较，可见下面的代码中。




## 3. 代码

整个工程主体为源文件`fifo_async.v`，仿真文件`fifo_async_tb.v`，Makefile文件`makefile`。编辑平台是 `vscode`, 仿真平台是 `vcs`, 使用 `verdi` 软件观察波形。具体如下所示：

### fifo_async.v
```verilog
module fifo_async #(
    parameter DATA_WIDTH = 32,
    // 此处深度为 2^6，即64个
    parameter FIFO_DEPTH = 6 
)
(
    input wr_clk, wr_rstn, rd_clk, rd_rstn, rq, wq,
    input [DATA_WIDTH-1:0] wr_data,
    output reg [DATA_WIDTH-1:0] rd_data,
    output full, empty
);

// 设置FIFO
reg [DATA_WIDTH-1:0] fifo [0:(1<<FIFO_DEPTH)-1];

// 读写指针，及其更新
reg [FIFO_DEPTH:0] wr_ptr, rd_ptr;
always @(posedge rd_clk or negedge rd_rstn) begin
    if (!rd_rstn)
        rd_ptr <= 0;
    else if (rq && !empty)
        rd_ptr <= rd_ptr + 1;
end

always @(posedge wr_clk or negedge wr_rstn) begin
    if (!wr_rstn)
        wr_ptr <= 0;
    else if (wq && !full)
        wr_ptr <= wr_ptr + 1;
end



wire [FIFO_DEPTH:0] wr_gray_ptr, rd_gray_ptr;
// 格雷码产生
assign wr_gray_ptr = (wr_ptr >> 1) ^ wr_ptr;
assign rd_gray_ptr = (rd_ptr >> 1) ^ rd_ptr;


// 空判断，在读时钟域中两级同步写指针
reg [FIFO_DEPTH:0] wr_gray_ptr_r1;
reg [FIFO_DEPTH:0] wr_gray_ptr_r2;
always @(posedge rd_clk or negedge rd_rstn) begin
    if (!rd_rstn)
        {wr_gray_ptr_r2, wr_gray_ptr_r1} <= 0;
    else 
        {wr_gray_ptr_r2, wr_gray_ptr_r1} <= {wr_gray_ptr_r1, wr_gray_ptr};
end

// 空判断，读写指针的格雷码相同
assign empty = (rd_gray_ptr == wr_gray_ptr_r2);


// 满判断，在写时钟域中两级同步读指针
reg [FIFO_DEPTH:0] rd_gray_ptr_r1;
reg [FIFO_DEPTH:0] rd_gray_ptr_r2;
always@(posedge wr_clk or negedge wr_rstn) begin
    if (!wr_rstn)
        {rd_gray_ptr_r2, rd_gray_ptr_r1} <= 0;
    else 
        {rd_gray_ptr_r2, rd_gray_ptr_r1} <= {rd_gray_ptr_r1, rd_gray_ptr};
end

// 满判断，读写指针的格雷码高两位相反，其他低位相同
assign full = (wr_gray_ptr[FIFO_DEPTH:FIFO_DEPTH-1] == ~rd_gray_ptr_r2[FIFO_DEPTH:FIFO_DEPTH-1]) &&
                (wr_gray_ptr[FIFO_DEPTH-2:0] == rd_gray_ptr_r2[FIFO_DEPTH-2:0]);


// 读数据，此处rd_addr为rd_ptr的低（FIFO_DEPTH-1）位
wire [FIFO_DEPTH-1:0] rd_addr;
assign rd_addr = rd_ptr[FIFO_DEPTH-1:0];
always @(posedge rd_clk or negedge rd_rstn) begin
    if (!rd_rstn)
        rd_data <= 0;
    else if (rq && !empty)
        rd_data <= fifo[rd_addr];
    else 
        rd_data <= 0;
end

// 写数据，此处wr_addr为wr_ptr的低（FIFO_DEPTH-1）位
wire [FIFO_DEPTH-1:0] wr_addr;
assign wr_addr = wr_ptr[FIFO_DEPTH-1:0];
always @(posedge wr_clk or negedge wr_rstn) begin
    if (wq && !full)
        fifo[wr_addr] = wr_data;
end

endmodule
```


### fifo_async_tb.v
```verilog
module fifo_async_tb;
reg wr_clk,wr_rstn;
reg rd_clk,rd_rstn;
reg wq,rq;
reg [31:0] wr_data;             // data input to be pushed to buffer
wire [31:0] rd_data;       // port to output the data using pop.
wire empty, full;  // buffer empty and full indication

fifo_async #(	.DATA_WIDTH(8),  
					.FIFO_DEPTH(3)) 
fifo_inst(
	.rd_clk				(rd_clk),
	.rd_rstn			(rd_rstn),
	.rq					(rq),
	.wr_clk				(wr_clk),
	.wr_rstn			(wr_rstn),
	.wq					(wq),
	.wr_data			(wr_data),
	.rd_data			(rd_data),
	.full				(full),
	.empty				(empty)
	
	);

always #10 wr_clk = ~wr_clk;
always #20 rd_clk = ~rd_clk;

reg [7:0] tempdata = 0;
    initial begin
	    rd_clk = 0;
	    wr_clk = 0;
	    #5000 $finish;
    end
    initial begin
	    wr_rstn = 1;
	    #2;
	    wr_rstn = 0;
	    #60;
	    wr_rstn = 1;
    end

    initial begin
	    rd_rstn = 1;
	    #2;
	    rd_rstn = 0;
	    #120;
	    rd_rstn = 1;
    end

    always  @(posedge wr_clk or negedge wr_rstn)
    begin
	    if(wr_rstn==1'b0)
		    wq <= 0;
	    else
		    wq <= $random;
    end

    always  @(posedge rd_clk or negedge rd_rstn)
    if(rd_rstn==1'b0)
	    rq <= 0;
    else
	    rq <= $random;

   always@(*)
     if(wq == 1)
        wr_data= $random;
     else
        wr_data = 0;
		
   initial
   begin
     $fsdbDumpfile("fifo_async.fsdb");
     $fsdbDumpvars();
   end

   // 用于观察FIFO中的数值
   wire [7:0] fifo_0; 
   assign fifo_0 = fifo_inst.fifo[0];

   wire [7:0] fifo_1; 
   assign fifo_1 = fifo_inst.fifo[1];

   wire [7:0] fifo_2; 
   assign fifo_2 = fifo_inst.fifo[2];

   wire [7:0] fifo_3; 
   assign fifo_3 = fifo_inst.fifo[3];

   wire [7:0] fifo_4; 
   assign fifo_4 = fifo_inst.fifo[4];

   wire [7:0] fifo_5; 
   assign fifo_5 = fifo_inst.fifo[5];

   wire [7:0] fifo_6;
   assign fifo_6 = fifo_inst.fifo[6];

   wire [7:0] fifo_7; 
   assign fifo_7 = fifo_inst.fifo[7];


endmodule

```

### Makefile
```makefile
.PHONY: sim, verdi, clean

PROJECT = fifo_async

VCS =	vcs \
		-R \
		-timescale=1ns/1ns \
		-debug_all \
		-fsdb \
		+define+FSDB \
		-full64 \
		+v2k \
		-sverilog \

VERDI =	verdi \
		-sv \
		-nologo \
		-ssf ${PROJECT}.fsdb \


sim:
	make clean
	${VCS} -f ${PROJECT}.f


verdi:
	${VERDI} -f ${PROJECT}.f &

clean:
	rm -rf ./csrc ./DVEfiles *.daidir *.log simv* *.key *.vpd \
				verdi* novas* *.fsdb

```


### fifo_async.f
```
./fifo_async.v
./fifo_async_tb.v

```



## 4. 仿真结果

下图是仿真产生的波形图。由图中可知，FIFO memory中可以顺利写入和读出数据，空满信号产生正常。整体设计功能正常。

![](/images/blog/picture22.jpg)

![](/images/blog/picture23.jpg)


