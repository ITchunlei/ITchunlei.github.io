---
title: Zircon内核x86架构源码分析之start.S
date: 2018/12/27
categories: 
- 操作系统
tags:
- 内核
- fuchsia
- zircon
---

start.S 是Zircon内核代码入口，位于/kernel/arch/x86 目录中

Zircon 内核遵守multiboot协议，可以使用符合mutliboot规范的引导器引导系统。

下面是multiboot引导系统的的入口代码：

<!-- more -->

```
// This section name is known specially to kernel.ld and gen-kaslr-fixups.sh.
// This code has relocations for absolute physical addresses, which do not get
// adjusted by the boot-time fixups (which this code calls at the end).
.section .text.boot, "ax", @progbits
.code32
FUNCTION_LABEL(_multiboot_start)
    cmpl $MULTIBOOT_BOOTLOADER_MAGIC, %eax  // 检测eax寄存器中是否是MULTIBOOT协议魔数
    je 0f
    xor %ebx,%ebx  //eax 寄存器模数不正确则清空ebx寄存器
0:  // multiboot_info_t pointer is in %ebx. // 如果正常从multiboot引导过来，ebx内容为multiboot_info_t 地址

    /* load our gdtr */
    load_phys_gdtr //此时还未完成内核分页，所有还不能使用内核的线性地址，只能临时加载gdt的物理地址

    /* load our data selectors */
    movw $DATA_SELECTOR, %ax
    movw %ax, %ds
    movw %ax, %es
    movw %ax, %fs
    movw %ax, %gs
    movw %ax, %ss

    // 加载内核自己的32位代码选择子，跳转到内核自己的32位代码段
    /* We need to jump to our sane 32 bit CS */
    pushl $CODE_SELECTOR
    pushl $PHYS(.Lfarjump)
    lret
```

上面代码很简单，只是加载内核自己的全局描述符，然后跳入内核的32位代码段，唯一需要注意的代码就是这个宏load_phys_gdtr,用于加载内核gdt的物理地址，代码如下：

```
//通过PHYS宏获gdtr的地址，然后放到内核堆栈上，在通过堆栈加载到全局描述符寄存器中
// Load our new GDT by physical pointer.
// _temp_gdtr has it as a virtual pointer, so copy it and adjust.
// Clobbers %eax and %ecx.
.macro load_phys_gdtr
    movw PHYS(_temp_gdtr), %ax
    movl PHYS(_temp_gdtr+2), %ecx
    sub $PHYS_ADDR_DELTA, %ecx
    movw %ax, -6(%esp)
    movl %ecx, -4(%esp)
    lgdt -6(%esp)
.endm
```
PHYS宏代码定义位于/kernel/arch/x86/include/arch/x86/asm.h 中，asm.h文件内容如下：
```
#pragma once

#include <asm.h>

/* x86 assembly macros used in a few files */

#define PHYS_LOAD_ADDRESS (KERNEL_LOAD_OFFSET)
#define PHYS_ADDR_DELTA (KERNEL_BASE - PHYS_LOAD_ADDRESS)
#define PHYS(x) ((x) - PHYS_ADDR_DELTA)
```