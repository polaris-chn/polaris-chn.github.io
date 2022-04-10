<!-- ---
layout: post
title: 黑客马拉松参赛总结
categories: Blog
description: 和同事组队参加了搜狗首届黑客马拉松，一些个人总结。
keywords: Hackathon
---

![首届黑客马拉松](/images/blog/hackathon.jpg)

五月的最后一个周末，和同事一起组队参加了我司举办的首届黑客马拉松大赛，经过几十个小时的奋战，我和队友们一起完成度较高地准备好了演示 Demo、ppt 等，在大家一致看好、觉得在首轮评审表现出色的情况下止步首轮，说不遗憾肯定是假的，想想和小伙伴们为了完成 Demo 的通宵达旦，四个人短时间内提交的上万行代码，三十多个小时不曾睡觉的情况下，听到评审结果宣判的那一刻，倦意趁我的一丝失落情绪铺天盖地地袭来。

时间仍然欢快地往前流淌，我在沉睡十多个小时后也满血复活，过去的就让它过去吧，收拾收拾心情，总结总结经验教训，继续前行。

比赛中有幸被小伙伴们推举为我们小组的 CTO，作为需要在技术方面掌控全局的角色，回想一下自己的表现，算是中规中矩，有亮点，也暴露了很多问题。

**先臭屁一下自己的优点：**

1. 担负起了技术选择、模块划分、任务分配和接口制定等方面的工作。
2. 认真负责，尽自己的努力试图去掌控开发进度、完成我们的 Demo。
3. 在态度上应该算无可挑剔，对可能影响进度和体验的 Bug 从不轻易放过。
4. 花了一些时间封装了一些使用到的基础类库，让小伙伴们用得舒心，写得放心。

**然后说说自己暴露的问题：**

1. 模块划分与 UML 表达。

   平时工作过程中较多地画一些小流程与逻辑的流程图，但是对整个 APP 的模块划分未涉足过，临时抱了一下佛脚之后做了一个类 MVC 的分层设计，自己感觉并不专业，用于指导开发也并不科学。

   对此我倒没什么好找借口的，就是对此的理论基础太薄弱，后续需要大量的充电补全，自己平时做一些东西的过程中也最好将前期的总体设计工作重视起来，多实践。

2. 任务分配。

   这一点是否做好取决于上一点，模块划分不合理将会直接导致任务分配出现问题。比如本次的任务安排，早期版本出现了这样的情况：

   | 人员  | 任务1       | 任务2         | 任务3        | 任务4         |
   |:------|:------------|:--------------|:-------------|:--------------|
   | 成员1 | 第一阶段 UI | 第二阶段 UI   | 第三阶段 UI  | 第三阶段 逻辑 |
   | 成员2 | 空闲        | 第一阶段 逻辑 | 继续         | 继续          |
   | 成员3 | 空闲        | 空闲          | 第二阶段逻辑 | 继续          |

   因为对 UI 的工作量评估不足，觉得由一个人来快速完成就行，但是实际上 UI 开发后来成为了明显的瓶颈，持续了整个开发周期。其它两个人有较长的时间处于等待 UI 出来之后才能开始往工程里接入逻辑的状态，浪费了大量的劳动力。

   后来对此进行了一番思考，觉得相对来说这样可能会更好一点：
   * 各人负责一段逻辑流程或者功能模块，顺带简单把对应的界面先搭起来，后来如果需要美化，可以仍然分工给各人，或者由一个相对工作量不那么紧的人统一来调整和美化。
   * 前期做工作分配时需要有意识地提炼出不严格需要前置条件的任务，松耦合的任务，合理分配。比如可以先将任务做成粗细两种粒度的列表，然后理清它们的前置依赖关系，找出适合打包分给同一个人的集合。
   * 对工期的评估结合我的评估和对应的负责人的评估来做。

   当然这里头显然大有学问，后续还是要针对性地学习一些相关的知识，有理论支撑做起来才更科学更有底气。

3. 量化的缺失。

   在试图掌控进度的过程中，有较长的时间里对剩下多少活只能有个模糊印象，而不能明确地评估剩下百分之多少的工作量，究其原因，就是对工作任务的细分量化不够，若能做到以下两点想必会更好：
   * 将模块细分为可以明确评估工作量的子任务。
   * 定期沟通了解小伙伴的进展，更新任务完成情况列表。

总之，离 CTO 的水平还很遥远啊~骚年需努力！ -->

---
layout: post
title: Verilog编程-1.10010序列检测器
categories: Verilog
description: 对一个序列检测问题的解答
keywords: Verilog
---

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





