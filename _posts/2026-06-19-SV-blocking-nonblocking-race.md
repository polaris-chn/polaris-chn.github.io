---
layout: post
title: Verilog编程-18. SystemVerilog阻塞与非阻塞赋值竞争详解
categories: Verilog
description: 通过三种赋值写法对比，深入理解SystemVerilog中阻塞赋值与非阻塞赋值的语义差异与仿真竞争
keywords: SystemVerilog, blocking, nonblocking, race, 阻塞赋值, 非阻塞赋值
---

# Verilog编程-18. SystemVerilog阻塞与非阻塞赋值竞争详解

## 1. 背景

在数字电路设计中，阻塞赋值（`=`）与非阻塞赋值（`<=`）是最基础却也最容易踩坑的语法点。很多初学者在同一个 `always` 块中混用两种赋值，或者在阻塞赋值中忽略语句顺序，导致仿真结果与预期不符——这就是所谓的 **仿真竞争（race condition）**。

本文基于一个精心设计的测试用例 `design_race.sv`，通过三种不同的赋值写法，对比它们在 VCS 仿真中的行为差异，帮助理解赋值语义的本质。

## 2. 公共测试平台

三种写法共用同一套时钟/复位逻辑，仅 `else` 分支内的赋值方式不同：

```systemverilog
module design_race;
    logic clk;
    logic rstn;
    logic [3:0] a;
    logic [3:0] b;

    initial begin
        clk = 0;
        forever #5 clk = ~clk;      // 周期 10ps
    end

    initial begin
        $display("test start!");
        rstn = 0;
        #15 rstn = 1;               // 15ps 释放复位
        #40 $finish;                 // 55ps 结束仿真
    end

    always @(posedge clk or negedge rstn) begin
        if (rstn == 0) begin
            a <= 0;
            b <= 0;
        end
        else begin
            // ← 此处三种写法见下文
            $display("@%0t a=%0d, b=%0d", $time, a, b);
        end
    end
endmodule
```

**关键点**：`$display` 在赋值语句之后、同一 time step 的 **Active Region** 执行；而非阻塞赋值的更新发生在 **NBA Region**，因此打印的可能是更新前的旧值。

## 3. 三种写法与仿真结果

### 写法一：阻塞赋值，先加 a 再赋 b

```systemverilog
else begin
    a = a + 1;
    b = a;
    $display("@%0t a=%0d, b=%0d", $time, a, b);
end
```

仿真结果：

```
@15 a=1, b=1
@25 a=2, b=2
@35 a=3, b=3
@45 a=4, b=4
```

**分析**：阻塞赋值顺序执行、立即生效。`a = a + 1` 执行完毕后 `a` 已更新，接下来 `b = a` 读到的是**更新后的 a**，所以 `b` 与 `a` 始终相等。

### 写法二：阻塞赋值，先赋 b 再加 a

```systemverilog
else begin
    b = a;
    a = a + 1;
    $display("@%0t a=%0d, b=%0d", $time, a, b);
end
```

仿真结果：

```
@15 a=1, b=0
@25 a=2, b=1
@35 a=3, b=2
@45 a=4, b=3
```

**分析**：仅仅调换了语句顺序，结果完全不同。`b = a` 先执行，此时 `a` 尚未加 1，`b` 拿到的是旧值；随后 `a = a + 1` 才更新 `a`。结果：**b 始终比 a 小 1**。

### 写法三：非阻塞赋值

```systemverilog
else begin
    a <= a + 1;
    b <= a;
    $display("@%0t a=%0d, b=%0d", $time, a, b);
end
```

仿真结果：

```
@15 a=0, b=0
@25 a=1, b=0
@35 a=2, b=1
@45 a=3, b=2
```

**分析**：非阻塞赋值的右值（RHS）在本 time step 开始时**一起求值**，左值（LHS）在 NBA Region 才更新。两行的 RHS 都使用**本拍旧的 a**：`a` 更新为 `旧a + 1`，`b` 更新为 `旧a`。因此 **b 落后 a 一拍**，语义上与写法二等价。

注意 `@15` 打印的是 `0/0`——这是因为 `$display` 在 Active Region 执行，本拍的 NBA 更新尚未提交。

## 4. 对照总结

| 写法 | 赋值类型 | 语句顺序 | a 与 b 关系 |
|------|----------|----------|-------------|
| 一 | 阻塞 `=` | `a=a+1` → `b=a` | **b 等于 a** |
| 二 | 阻塞 `=` | `b=a` → `a=a+1` | **b 比 a 小 1** |
| 三 | 非阻塞 `<=` | `a<=a+1; b<=a` | **b 比 a 小 1**（落后一拍） |

核心结论：**阻塞赋值的结果取决于语句顺序，非阻塞赋值的结果与顺序无关（RHS 统一取旧值）。**

## 5. 常见误解澄清

### 误解一：「b = a 是不是 b 比 a 大 1？」

不是。`b = a` 仅仅是把 a 的当前值赋给 b，不做任何运算。只有写成 `b = a + 1` 时 b 才会比 a 大 1。

### 误解二：「非阻塞 a<=a+1; b<=a 中，b 会拿到加 1 后的 a？」

不会。非阻塞赋值的 RHS 统一使用本拍事件触发前的旧值，b 拿到的是旧 a。

### 误解三：「$display 为什么和非阻塞结果对不上？」

`$display` 在 Active Region 执行，早于 NBA 更新。如需打印更新后的值，可使用 `$monitor` 或将 display 放到下一拍。

### 误解四：「$finish 后面的 $display 为什么没打印？」

`$finish` 立即终止仿真，其后的语句不会执行。应将 `$display` 放在 `$finish` 之前。

## 6. RTL 设计建议

理解了上述竞争语义后，在实际 RTL 设计中应遵循以下原则：

- **时序逻辑**（`always_ff`）：对寄存器使用**非阻塞赋值 `<=`**
- **组合逻辑**（`always_comb`）：使用**阻塞赋值 `=`**
- **不要在同一 always 块中对同一信号混用 `=` 和 `<=`**
- Testbench 中 `initial`/`task` 驱动信号常用阻塞 `=`；DUT 时钟沿采样建模常用非阻塞 `<=`

## 7. 如何复现

```bash
vlogan -full64 -sverilog -timescale=1ps/1ps -nc design_race.sv
vcs -full64 -top design_race -o design_race.simv
./design_race.simv
```

修改 `else` 分支为上述三种写法之一，重新编译运行即可对比验证。
