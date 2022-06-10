---
layout: post
title: Verilog编程-6. 同步FIFO
categories: Verilog
description: 讲清楚同步FIFO的原理和程序实现
keywords: Verilog
---

# Verilog编程-6. 同步FIFO
## 背景
    FIFO是First In First Out的缩写，即先进先出队列，FIFO根据读写时钟是否为同一时钟分为同步FIFO和异步FIFO，本文介绍的
是同步FIFO。FIFO与普通存储器的区别是没有外部读写地址线，这样使用起来非常方便，其缺点是只可顺序写入数据，顺序读出数据，其
数据地址由内部读写指针加1完成，不能像普通存储器那样可以由地址线决定读取或写入某个指定的地址。下图是FIFO的示意图，FIFO初始化的时候，读指针（rd_ptr）和写指针（wr_ptr）都指向0位置。当有数据写入的时候，写指针自动加1，直到FIFO达到满的状态；当需要读数据的时候，读数据控制器就会将读指针指向的位置的数据读出，并且读指针自动加1，直到FIFO达到空的状态。当两个指针指向`7`位置时，下一次会自动跳转到`0`位置。

![](/images/blog/picture19.jpg)

    FIFO一般用于不同时钟域之间的数据传输，比如FIFO的一端是AD数据采集，另一端是计算机的PCI总线。若其AD采集速率为16位
100kbps，那么每秒数据量就是100k × 16bit = 1.6Mbps，而PCI总线速度为33MHz，总线宽度为32bit，其最大传输速度为1056Mbps，在两个不同的时钟域之间就可以采用FIFO来作为数据缓冲。另外对于不同宽度的数据接口，也可以用FIFO，例如单片机8位数据输出，而DSP可能是16位数据输入，在单片机与DSP连接的时候就可以使用FIFO来达到数据匹配的目的。


## 设计思路
    FIFO设计需要考虑两个问题，一是FIFO的大小，即双端口RAM的大小；另一个是FIFO空满状态的判断，为保证数据正确的写入或读
出，而不发生溢出或者读空的状态出现，必须保证FIFO在满的情况下不可以写，在空的状态下不可以读。而怎样判断空满就成了FIFO设计的核心问题。对于同步FIFO来说，判断空满状态一般有两种方法。
（1）FIFO中的RAM一般是双端口RAM，有独立的读写地址，通过设立读写指针，比较读写指针的大小来确定空满状态。在设置读写指针的时候，有个细节需要注意，即读写指针的位数要多一位，这样可以通过最高位的比较，看当前FIFO是空还是满。
（2）设置一个计数器，当写使能有效的时候计数器加1；读使能有效的时候计数器减1，将计数器与RAM的size进行比较来判断FIFO的空满状态。这种方法设计比较简单，但需要额外的计数器，就会产生额外的资源，而且当FIFO比较大时，会降低FIFO最终可以达到的速度。
在本设计中，采用的就是计数器的方法来判断空满。

## 代码
整个工程主体为源文件`fifo_sync.v`，仿真文件`fifo_sync_tb.v`，Makefile文件`makefile`。编辑平台是 `vscode`, 仿真平台是 `vcs`, 使用 `verdi` 软件观察波形。具体如下所示：

### fifo_sync.v
```verilog
module fifo_sync (
    input wire clk,
    input wire rstn,
    input wire [31:0] wr_data,      // 输入数据，宽度32bit
    input wire rq,                  // 读数据请求
    input wire wq,                  // 写数据请求

    output reg [31:0] rd_data,      // 输入数据，宽度32bit
    output full,
    output empty
);

    reg [31:0] fifo_ram [0:7];      // FIFO深度8，宽度32bit
    reg [2:0] counter;              // 计数器，用来判断空满；
    reg [2:0] rd_ptr;               // 读指针
    reg [2:0] wr_ptr;               // 写指针

    assign full = (counter==7);
    assign empty = (counter==0);

    always @(posedge clk or negedge rstn) begin
        if (!rstn)
            counter <= 0;
        else if ((wq && !full) && (rq && !empty))
            counter <= counter;
        else if (wq && !full)
            counter <= counter + 1;
        else if (rq && !empty)
            counter <= counter - 1;
        else
            counter <= counter;
    end

    // 读数据
    always @(posedge clk or negedge rstn) begin
        if (!rstn)
            rd_data <= 0;
        else if (rq && !empty)
            rd_data <= fifo_ram[rd_ptr];
    end

    // 写数据
    always @(posedge clk or negedge rstn) begin
        if (wq && !full)
            fifo_ram[wr_ptr] <= wr_data;
    end


    // 更新读指针和写指针
    always @(posedge clk or negedge rstn) begin
        if (!rstn) begin
            wr_ptr <= 0;
            rd_ptr <= 0;
        end

        else begin
            if (wq && !full) begin
                wr_ptr <= wr_ptr + 1;
            end

            if (rq && !empty) begin
                rd_ptr <= rd_ptr + 1;
            end
        end
        
    end

    
endmodule
```

### fifo_sync_tb.v
```verilog
module fifo_sync_tb;
    reg clk,rstn;
    reg wq,rq;
    reg [7:0] wr_data;             // data input to be pushed to buffer
    wire [7:0] rd_data;       // port to output the data using pop.
    wire empty,full;  // buffer empty and full indication

    fifo_sync fifo_inst(
        .clk(clk),
        .rstn(rstn),
        .wr_data(wr_data),
        .rq(rq),
        .wq(wq),
        .rd_data(rd_data),
        .full(full),
        .empty(empty)
        );

    always #10 clk = ~clk;

    reg [7:0] tempdata = 0;
    initial begin
        clk = 0;
        rstn = 0;
        wq = 0;
        rq = 0;
        wr_data = 0;
        #15;
        rstn = 1;

        push(1);
        fork
           push(2);
           pop(tempdata);
        join              //push and pop together
        push(10);
        push(20);
        push(30);
        push(40);
        push(50);
        push(60);
        push(70);
        push(80);
        push(90);
        push(100);
        push(110);
        push(120);
        push(130);

        pop(tempdata);
        push(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        push(140);
        pop(tempdata);
        push(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        pop(tempdata);
        push(5);
        pop(tempdata);
        #1000 $finish;
    end

    `ifdef FSDB
    initial
    begin
	    $fsdbDumpfile("fifo_sync.fsdb");
	    $fsdbDumpvars();
    end
    `endif 

    task push (input [7:0] data);
	    if(full)
		    $display("---Cannot push %d: Buffer Full---",data);
	    else begin
		    $display("Push",,data);
            wr_data = data;
            wq = 1;
            @(posedge clk);
            #5 wq = 0;
        end
    endtask

    task pop(output[7:0] data);
        if(empty)
            $display("---Cannot Pop: Buffer Empty---");
        else begin
            rq = 1;
            @(posedge clk);
            #3 rq = 0;
            data = rd_data;
            $display("------Poped:",,data);
        end
    endtask

    wire [2:0] wr_ptr;
    assign wr_ptr = fifo_inst.wr_ptr; 
endmodule
```

### Makefile
```makefile
.PHONY: sim, verdi, clean

PROJECT = fifo_sync

VCS =	vcs \
		-R \
		-timescale=1ns/1ps \
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

### fifo_sync.f
```
./fifo_sync.v
./fifo_sync_tb.v
```

## 仿真结果
仿真波形图如下所示，波形中主要显示出了读使能`rq`和写使能`wq`信号，以及空满信号`empty`和`full`，并且同时添加了读指针和写指针以及仿真文件中的`tempdata`中间变量。从波形图中可以看出，空满状态的判断是正确的

![](/images/blog/picture20.jpg)

