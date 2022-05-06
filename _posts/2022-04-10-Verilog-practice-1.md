---
layout: post
title: Verilog编程-1. 10010序列检测器
categories: Verilog
description: 对一个序列检测问题的解答
keywords: Verilog
---

# Verilog编程-1.10010序列检测器

## 1. 背景

​		序列检测器在数据通讯，雷达和遥测等领域中有着充分应用，使用verilog语言进行序列检测也是常见的编程练习题，本文对于数字序列 10010 进行检测，使用verilog语言，编辑器为vscode仿真平台为vcs。对于检测到的序列信号输出为 1 ；否则输出为 0 。



## 2. 设计思路

​		使用数字电路中常用的Mealy型有限状态机（FSM）进行设计，Mealy型有限状态机其输出不仅与当前状态有关，还取决于当前输入信号。下图呈现的是状态转移图。

![](/images/blog/picture1.jpg)

需要注意的是，某些状态在产生非序列需要数字的时候，其状态转移并不是保持或者跳转到idle状态，例如在s2状态时输入为`1`，此时状态会跳转到s1，因为此时s2输入时的`1`，可以作为下一个`10010`序列的s1状态，还要几个状态跳转也是如此。



## 3. 代码

​		所有程序编辑平台为vscode，仿真平台为ubuntu系统中的vcs工具，分别包括源文件`seq_det.v`，仿真文件`seq_det_tb.v`，路径名列表文件`seq_det.f`和`makefile`文件；波形显示工具为verdi。文件具体内容如下。

### 源文件

```verilog
module seq_det (
    input clk,
    input rstn,
    input data_in,
    output reg data_out
);
    parameter   idle    = 3'b000,
                s1      = 3'b001,
                s2      = 3'b010,
                s3      = 3'b011,
                s4      = 3'b100,
                s5      = 3'b101;

    reg [2:0] curr_state, next_state;
    always @(posedge clk or negedge rstn) begin
        if (!rstn)
            curr_state <= idle;
        else
            curr_state <= next_state;
    end

    always @(*) begin
        case (curr_state)
            idle: case (data_in)
                0 : next_state = idle;
                1 : next_state = s1;
                default : next_state = idle;
            endcase 

            s1: case (data_in)
                0 : next_state = s2;
                1 : next_state = s1;
                default : next_state = s1;
            endcase

            s2: case (data_in)
                0 : next_state = s3;
                1 : next_state = s1;
                default: next_state = s2;
            endcase

            s3: case (data_in)
                0 : next_state = idle;
                1 : next_state = s4; 
                default: next_state = s3;
            endcase

            s4: case (data_in)
                0 : next_state = s5;
                1 : next_state = s1; 
                default: next_state = s4;
            endcase

            s5: case (data_in)
                0 : next_state = s3;
                1 : next_state = s1; 
                default: next_state = s5;
            endcase

            default: next_state = idle;
        endcase
    end

    always @(posedge clk or negedge rstn) begin
        if (!rstn)
            data_out <= 0;
        else if (next_state==s5 & data_in==0)
            data_out <= 0;
        else
            data_out <= 0;
    end
    

endmodule
```



### 仿真文件

```verilog
module seq_det_tb;

    reg clk, rstn;
    reg data_in;
    wire data_out;

    seq_det inst1(
        .clk        (clk),
        .rstn       (rstn),
        .data_in    (data_in),
        .data_out   (data_out)
    );

    initial begin
        clk = 0; rstn = 0;
        #8 rstn = 1;
    end
    
    initial begin
        forever begin
            #5 clk = ~clk;
        end
    end


    initial begin
        data_in = 0;
        #23 data_in = 1;
        #10 data_in = 0;
        #10 data_in = 0;
        #10 data_in = 1;
        #10 data_in = 0;
        #10 data_in = 1;
        #10 data_in = 0;
        #10 data_in = 1;
        #10 data_in = 1;
        #10 data_in = 0;
        #10 data_in = 0;
        #10 data_in = 1;
        #10 data_in = 0;
        #30 $finish;
    end


    `ifdef FSDB
    initial begin
        $fsdbDumpfile("seq_det.fsdb");
        $fsdbDumpvars();      
    end
    `endif

endmodule
```



### 路径名列表文件

```verilog
./seq_det.v
./seq_det_tb.v
```



### makefile文件

```makefile
.PHONY: sim, verdi, clean

PROJECT = seq_det

VCS =	vcs -sverilog \
			+v2k \
			-timescale=1ns/1ns \
			-debug_all \
			-R \
			-fsdb \
			+define+FSDB \
			-full64 \

VERDI = verdi 	-sv \
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





## 4. 仿真结果

​		下图为使用vcs软件仿真得到波形图

![](/images/blog/picture2.png)

由上图可知，在data_in信号输入序列`10010`的时候，输出信号data_out会输出信号`1`，否则输出为`0` 。





