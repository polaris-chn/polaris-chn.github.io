<!-- ---
layout: post
title: Verilog编程-1.10010序列检测器
categories: Verilog
description: 对一个序列检测问题的解答
keywords: Verilog
--- -->

# Verilog编程-1.10010序列检测器



## 1. 背景

​		序列检测器在数据通讯，雷达和遥测等领域中有着充分应用，使用verilog语言进行序列检测也是常见的编程练习题，本文对于数字序列 10010 进行检测，使用verilog语言，编辑器为vscode仿真平台为vcs。对于检测到的序列信号输出为 1 ；否则输出为 0 。



## 2. 设计思路

​		使用数字电路中常用的Mealy型有限状态机（FSM）进行设计，Mealy型有限状态机其输出不仅与当前状态有关，还取决于当前输入信号。下图呈现的是状态转移图。

![image-20210801110047720](https://github.com/polaris-chn/polaris-chn.github.io/blob/main/%E5%9B%BE/%E5%9B%BE1.png)

需要注意的是，某些状态在产生非序列需要数字的时候，其状态转移并不是保持或者跳转到idle状态，例如在s2状态时输入为`1`，此时状态会跳转到s1，因为此时s2输入时的`1`，可以作为下一个`10010`序列的s1状态，还要几个状态跳转也是如此。



## 3. 代码

​		所有程序编辑平台为vscode，仿真平台为ubuntu系统中的vcs工具，分别包括源文件`seq_det.v`，仿真文件`seq_det_tb.v`，路径名列表文件`seq_det.f`和`makefile`文件。文件具体内容如下。

### 源文件

```verilog
module seq_det (
    input       clk,
    input       rstn,
    input       data_in,
    output reg  data_out
);

    parameter idle = 0;
    parameter s1   = 1;
    parameter s2   = 2;
    parameter s3   = 3;
    parameter s4   = 4;
    parameter s5   = 5;

    reg [2:0] st_cur, st_next;
    always @(posedge clk or negedge rstn) begin
        if (!rstn)
            st_cur <= idle;
        else
            st_cur <= st_next;
    end

    always @(*) begin
        case (st_cur)
            idle: case(data_in)
                0 : st_next = idle;
                1 : st_next = s1;
                default : st_next = idle;
            endcase

            s1: case (data_in)
                0 : st_next = s2;
                1 : st_next = s1; 
                default: st_next = s1;
            endcase

            s2: case (data_in)
                0 : st_next = s3;
                1 : st_next = s1; 
                default: st_next = s2;
            endcase

            s3: case (data_in)
                0 : st_next = idle;
                1 : st_next = s4; 
                default: st_next = s3;
            endcase

            s4: case (data_in)
                0 : st_next = s5;
                1 : st_next = s1; 
                default: st_next = s4;
            endcase
            
            s5: case (data_in)
                0 : st_next = s3;
                1 : st_next = idle; 
                default: st_next= s5;
            endcase

            default: st_next = idle;
        endcase
    end

    always @(posedge clk or negedge rstn) begin
        if (!rstn) 
            data_out <= 0;
        else if (st_cur==s4 && data_in==0)
            data_out <= 1;
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
endmodule
```



### 路径名列表文件

```verilog
./seq_det.v
./seq_det_tb.v
```



### makefile文件

```makefile
.PHONY: com sim clean

VCS = vcs 	-sverilog +v2k -timescale=1ns/1ns \
			-debug_all

SIM = ./simv -gui

com:
		${VCS} -f seq_det.f

sim:
		${SIM}

clean:
		rm -rf ./csrc ./DVEfiles *.daidir *.log simv* *.key *.vpd
```





## 4. 仿真结果

​		下图为使用vcs软件仿真得到波形图

![image-20210801111426740](https://github.com/polaris-chn/polaris-chn.github.io/blob/main/%E5%9B%BE/%E5%9B%BE2.png)

由上图可知，在data_in信号输入序列`10010`的时候，输出信号data_out会输出信号`1`，否则输出为`0` 。





