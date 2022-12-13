---
title: Interrupt Handler
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# 中断处理程序







## 创建中断描述符表

```c
#include "interrupt.h"
#include "stdint.h"
#include "global.h"

#define IDT_DESC_CNT 0x21 // 目前支持的中断总数

struct gate_desc {
    uint16_t func_offset_low_word;
    uint16_t selector;
    uint8_t dcount; 

    uint8_t attribute;
    uint16_t func_offset_high_word;
}

static void make_idt_desc(struct gate_desc* p_gdesc, uint8_t attr, intr_handler function);
static sturct gate_desc idt[IDT_DESC_CNT];

extern intr_handler intr_entry_table[IDT_DESC_CNT];

/* create idt descriptor */
static void make_idt_desc(struct gate_desc* p_gdesc, uint8_t attr, intr_handler function) {
    
}
```









编译

```shell
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -c -fno-builtin -o build/main.o kernel/main.c
$ nasm -f elf -o build/print.o lib/kernel/print.S
$ nasm -f elf -o build/kernel.o kernel/kernel.S
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -c -fno-builtin -o build/interrupt.o kernel/interrupt.c
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -c -fno-builtin -o build/init.o kernel/init.c
$ ld -m elf_i386 -Ttext 0xc0001500 -e main -o build/kernel.bin build/main.o build/init.o build/interrupt.o build/print.o build/kernel.o
$ dd if=build/kernel.bin of=/home/ml/bochs/hd60M.img bs=512 count=200 seek=9 conv=notrunc
12+1 records in
12+1 records out
6152 bytes (6.2 kB, 6.0 KiB) copied, 0.034197 s, 180 kB/s
```



运行

```
master: ICW1: single mode not supported
master: ICW1：level sensitive mode not supported.
```

是往0x20 和 0xA0 端口写的值bit3（第四个位）为1 所导致的。

bochs 部分源码

```c++
void bx_pic_c::write(Bit32u address, Bit32u value, unsigned io_len)
{
	.........
    switch (address) {
        ......
        case 0x20: 
            if (value & 0x08) {
            BX_PANIC(("master: ICW1: level sensitive mode not supported"));
            }
            .....
        case 0x80: 
            if (value & 0x08) {
            BX_PANIC(("master: ICW1: level sensitive mode not supported"));
        }
    	.....
	}
    .........
}
```



## Reference 

1. 
