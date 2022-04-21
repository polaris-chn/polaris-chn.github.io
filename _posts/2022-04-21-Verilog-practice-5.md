---
layout: post
title: Verilog编程-5. 异步复位同步释放
categories: Verilog
description: 对于异步复位同步释放原理的阐释以及程序代码
keywords: Verilog
---

# Verilog编程-5. 异步复位同步释放

## 1. 背景
对于复位信号来说，也需要遵守建立时间和保持时间。对于一般的同步复位来说，其缺点是复位信号依赖时钟，时钟信号出现问题，那么就无法完成复位；同时复位信号的脉冲宽度必须要大于指定的时钟周期长度。而对于异步复位来说，最大的问题是复位信号释放的随机性，易导致时序违例，产生亚稳态。在复位中如果可以采用异步复位同步释放的方法，就可以避免以上两种复位方式的缺点。


## 2. 复位电路设计思路
从对于异步复位同步释放的理解来看，电路结构需要在复位信号有效的任意时刻输出复位信号，而在复位信号撤销的时候要等上一拍使得时钟可以同步到复位信号的撤销。而下图可以满足这种设计要求：
![](/images/blog/picture13.jpg)
以 `d2` 寄存器的输出作为后续电路的复位信号。则当复位信号有效时，`d2` 的输出会立刻变为0，这样对于后续电路来说也可以立刻得到复位的有效信号从而进行复位；当撤销复位信号的时候，由于高电平 `1` 经过了 `d1,d2` 两级寄存器，相当于打了两拍，这样就消除了亚稳态的概率。

## 3. 代码
整个工程包括源文件 `nrsr.v`, 仿真文件 `nrsr_tb.sv`, 列表文件 `nrsr.f`以及 `makefile` 文件。编辑平台是 `vscode`, 仿真平台是 `vcs`, 使用 `verdi` 软件观察波形。

### nrsr.v
```verilog
module nrsr (
    input clk,
    input rstn,
    input a, b,
    output c, d
);
    reg d1, d2;
    always @(posedge clk or negedge rstn) begin
        // 同步释放
        if(!rstn) begin
            d1 <= 0;
            d2 <= 0;
        end

        // 异步复位
        // 从下面语句可以看出，异步复位，同步释放的本质是将释放信号打两拍，信号1首先经过了d1，再经过了d2
        // 这样就可以减少亚稳态风险
        else begin
            d1 <= 1;
            d2 <= d1;
        end
    end
    wire rstn_o;
    assign rstn_0 = d2;

    reg d3, d4;
    always @(posedge clk or negedge rstn_o) begin
        if (!rstn_0) begin
            d3 <= 0;
            d4 <= 0;
        end

        else begin
            d3 <= a;
            d4 <= b;
        end
    end
    assign c = d3;
    assign d = d4;
endmodule
```

### nrsr_tb.sv
```verilog
module nrsr_tb;
    reg clk;
    reg rstn;
    reg a, b;
    wire c, d;

    nrsr uut1 (
        .clk    (clk),
        .rstn   (rstn),
        .a      (a),
        .b      (b),
        .c      (c),
        .d      (d)
    );

    initial begin
        clk = 0;
        forever #5 clk = ~clk;
    end

    initial begin
        rstn = 0;
        #24 rstn = 1;
        #63 rstn = 0;
    end

    initial begin
        a = 0; b = 0;
        #24 a = 1; b = 1;
    end

    initial begin
        #200 $finish;
    end

    `ifdef FSDB
    initial begin
        $fsdbDumpfile("nrsr.fsdb");
        $fsdbDumpvars();
    end
    `endif

    wire e;
    assign e = uut1.rstn_o;

    
endmodule
```

### nrsr.f
```
./nrsr.v
./nrsr_tb.sv
```

### Makefile
```makefile
.PHONY: sim, verdi, clean

PROJECT = nrsr

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


## 4. 仿真结果
由下图可以看出，在撤销复位信号的时候，经过了时钟同步之后，寄存器c, d才有了正确的输出；而在复位信号有效时，则在下一个有效的上升沿，寄存器c, d就进行了复位，不需要时钟同步。这样就实现了异步复位，同步释放的效果。

![](/images/blog/picture14.jpg)
