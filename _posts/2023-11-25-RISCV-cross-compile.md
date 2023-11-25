---
layout: post
title: RISC-V交叉编译
categories: RISC-V
description: RISCV交叉编译的内容
keywords: RISC-V, 编译
---

# 编译过程
通常，将c/c++代码编译成可执行文件的过程包括预处理、编译、汇编、链接四步，其顺序如下。
![](/images/blog/picture33.png)

# 交叉编译
交叉编译的概念为：在当前环境下位目标环境进行程序编译。例如在x86_64机器上编译适用于RISC-V 64位机器的目标程序。此时需要使用交叉编译工具链。

在此提供在x86_64架构，linux系统环境下编译RISC-V 64位程序的工具链，包括源码压缩包和已经编译好的压缩包。在[下载链接](https://pan.baidu.com/s/1rjZspMg3J9O2Z102z23xPg?pwd=6zt5)中，`riscv-gnu-toolchain`是编译器源码，下载后进行编译安装则可以使用；`riscv-unknown-linux`和`riscv-unknown-elf`则已经编译好，可直接使用。对于已经编译好的两个压缩包，在解压缩之后，将其路径添加到`.bashrc`文件之后即可使用，如下所示：
```
export $PATH=/解压路径/riscv/bin:$PATH
export $PATH=/解压路径/riscv64/bin:$PATH
```

# 示例
以`riscv64-unknown-elf-`为例：
- `riscv64-unknown-elf-gcc` 为编译器/编译套装
- `riscv64-unknown-elf-as` 为汇编器
- `riscv64-unknown-elf-ld` 为链接器
- `riscv64-unknown-elf-objdump` 为反汇编器，将目标/可执行文件反汇编成汇编文件
- `riscv64-unknown-elf-objcopy` 为目标拷贝软件，可将目标/可执行文件转变成不同格式，例如binary, verilog格式等

下面以一小段汇编代码`sample.S`为例进行演示
```s
.org 0x0
 	.global _start
_start:
	ori x1, x0, 0x210 # x1 = h210
	ori x2, x1, 0x021 # x2 = h231
	slli x3, x2, 1  # x3 = h462
	andi x4, x3, 0x568 # x4 = h460
	ori x5, x0, 0x68a # x5 = h68a
```

使用make工具进行工程化管理
```makefile
CROSS_COMPILE = /riscv64-unknown-elf-

# 编译.S文件，产生目标文件.o
%.o: %.S
    $(CROSS_COMPILE)as $< -o $@ -march=rv64g

# 链接.o文件，产生可执行文件.out
%.out: %.o
    $(CROSS_COMPILE)ld $< -o $@

# 反汇编.o文件，产生.s文件
%.s: %.o
    $(CROSS_COMPILE)objdump $< -o $@_dump

# 产生.verilog文件，可在verilog仿真文件中通过readmemh直接读入，验证CPU功能
%.verilog: %.o
    $(CROSS_COMPILE)objcopy -O verilog $< $@

clean:
    rm -rf *.o *.out *.verilog
```

最后产生的`.verilog`文件如下所示，即可读入testbench进行仿真。   