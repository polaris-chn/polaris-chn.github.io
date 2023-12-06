---
layout: post
title: Verilog编程-16. DFF的一些思考
categories: Verilog
description: 对于DFF功能的一些总结思考
keywords: Verilog
---

# DFF的功能和时序
DFF，即D触发器，是最常用的一种时序器件，但是长久以来对其了解不深刻，现在对此进行一些总结和思考。首先是DFF的程序：
```verilog
module dff(
    input clk,
    input rstn,
    input wire [3:0] d,
    output reg [3:0] q
);

    always@(posedge clk) begin
        if(!rstn)
            q <= 0;
        else
            q <= d;
    end
endmodule
```
下面只截取部分仿真代码进行说明：

```verilog
initial begin
    clk = 0;
    forever #5 clk = ~clk;
end

initial begin
    rstn = 0;
    d = 0;
    #15 rstn = 1; d = 1;
    #10 d = 2;
    #10 d = 3;
end
```

下面截取部分仿真波形进行说明：
![](/images/blog/picture34.png)
从上图中可以看出在15ns时刻，rstn信号有效且输入信号发生了变化，此时输出信号也发生了变化。即数据的变化沿与时钟的上升沿对齐的时候，对应的输出则在此时刻就发生了变化。DFF捕获时钟上升沿的D端数据，并<font color=red>同时刻</font>在Q端输出，一直维持到下一时钟上升沿到来之前。在此期间，D端的数据变化不会影响Q端的输出。


# 流水线
DFF经常插入到组合逻辑中去，形成流水线以改善电路性能。接下来讨论DFF的流水线功能。

```verilog
module dff(
    input clk,
    input rstn,
    input wire [3:0] d,
    output reg [3:0] q
);
    reg [3:0] tmp
    always@(posedge clk) begin
        if(!rstn) begin
            q <= 0;
        end
        else begin
            tmp <= d;
            q <= tmp;
        end
    end
endmodule

```
对于上述程序，可以形成如下的DFF链，输入信号d先进入DFF tmp，然后在下一时钟上升沿从tmp到达DFF q，从而形成在15ns时刻输入改变，而在25ns时刻才输出改变的效果。最后的波形图也可以说明这一点。
![](/images/blog/picture36.png)

![](/images/blog/picture35.png)

