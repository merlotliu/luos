---
title: Assert
date: 2022-11-4
updated: 2022-11-4
tags: 
- [OS]
categories: 
- [OS]
comments: true
---

# ASSERT

辅助程序调试，通常用在开发阶段。将程序运行到此处的正确条件状态作为参数传递给 ASSERT，条件不满足则报错挂起，反之继续正常执行。C 语言中使用 ASSERT:

```C
ASSERT(Condition)
```

C 语言中 ASSERT 使用宏定义，原理是判断传送给 ASSERT 的表达式是否成立，成立则继续执行，否则打印错误信息并停止执行。

## debug.h

```c
#ifndef __KERNEL_DEBUG_H
#define __KERNEL_DEBUG_H

void painc_spin(char* finename, int line, const char* func, const char* condition);

/* __VA_ARGS__ 是预处理器的专用标识符，代表所有与省略号对应的参数
 * '...' 表示可变参数
 */
#define PANIC(...) panic_spin(__FILE__, __LINE__, __func__, __VA_ARGS__)

#ifdef NDEBUG
	#define ASSERT(CONDITION) ((void)0)
#else
	#define ASSERT(CONDITION) \
    	/* 符号 # 让编译器将宏的参数转化为字符串字面量 */
		if(CONDITION) {} else { PANIC(#CONDITION); }
#endif /* __NDEBUG */

#endif /* __KERNEL_DEBUG_H */

```

ASSERT 应该仅在调试过程中使用，如果不再调试，应使该宏失效。所以当定义 NDEBUG 宏时，ASSERT 将被置空。通常我们使用 gcc 编译时可以指定宏，如 `gcc -D NDEBUG`。

## debug.c

```c
#include "debug.h"
#include "print.h"
#include "interrupt.h"

/* print filename, line no, function name, condition and hang out program  */
void panic_spin(char* filename, int line, const char* func, const char* condition) {
    disable_intr(); /* close interrupt, because sometimes we will individually call panic_spin */
    
	put_str("\n\n !!!!!! error !!!!!! \n");
    put_str("filename : ");
    put_str(filename);
    put_str("\n");
    put_str("line : 0x");
    put_str(line);
    put_str("\n");
    put_str("function : ");
    put_str((char*)func);
    put_str("\n");
    put_str("condition : ");
    put_str((char*)condition);
    put_str("\n");
    while(1);
}
```

## makefile

```makefile
BUILD_DIR 	= ./build
ENTRY_POINT = 0xc0001500

AS			= nasm
CC 			= gcc
LD 			= ld
LIB 		= -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/

ASFLAGS 	= -f elf
CFLAGS		= -m32 -Wall $(LIB) -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes
LDFLAGS		= -m elf_i386 -Ttext $(ENTRY_POINT) -e main -Map $(BUILD_DIR)/kernel.map

OBJS		= 	$(BUILD_DIR)/main.o \
				$(BUILD_DIR)/init.o \
				$(BUILD_DIR)/interrupt.o \
				$(BUILD_DIR)/timer.o \
				$(BUILD_DIR)/kernel.o \
				$(BUILD_DIR)/print.o \
				$(BUILD_DIR)/debug.o 

# C
$(BUILD_DIR)/main.o: kernel/main.c \
					lib/kernel/print.h lib/stdint.h kernel/init.h
	$(CC) $(CFLAGS) $< -o $@

$(BUILD_DIR)/init.o: kernel/init.c \
					kernel/init.h lib/kernel/print.h lib/stdint.h kernel/interrupt.h device/timer.h
	$(CC) $(CFLAGS) $< -o $@

$(BUILD_DIR)/interrupt.o: kernel/interrupt.c \
						kernel/interrupt.h lib/stdint.h kernel/global.h lib/kernel/io.h lib/kernel/print.h
	$(CC) $(CFLAGS) $< -o $@

$(BUILD_DIR)/timer.o: device/timer.c \
					device/timer.h lib/stdint.h lib/kernel/io.h lib/kernel/print.h
	$(CC) $(CFLAGS) $< -o $@

$(BUILD_DIR)/debug.o: kernel/debug.c \
					kernel/debug.h lib/kernel/print.h lib/stdint.h kernel/interrupt.h
	$(CC) $(CFLAGS) $< -o $@
	
# assembly	
$(BUILD_DIR)/kernel.o: kernel/kernel.S
	$(AS) $(ASFLAGS) $< -o $@

$(BUILD_DIR)/print.o: lib/kernel/print.S
	$(AS) $(ASFLAGS) $< -o $@
	
# link all of object file
$(BUILD_DIR)/kernel.bin: $(OBJS)
	$(LD) $(LDFLAGS) $^ -o $@

.PHONY : mk_dir hd clean all

mk_dir:
	if[[! -d $(BUILD_DIR)]]; then mkdir $(BUILD_DIR); fi

hd:
	dd if=$(BUILD_DIR)/kernel.bin of=/home/ml/bochs/hd60M.img bs=512 count=200 seek=9 conv=notrunc

clean:
	cd $(BUILD_DIR) && rm -rf ./*

build: $(BUILD_DIR)/kernel.bin

all: mk_dir build hd
```

### make

```c
$ make clean
cd ./build && rm -rf ./*
ml@sakura:~/bochs/los$ make build
gcc -m32 -Wall -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/ -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes kernel/main.c -o build/main.o
gcc -m32 -Wall -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/ -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes kernel/init.c -o build/init.o
gcc -m32 -Wall -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/ -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes kernel/interrupt.c -o build/interrupt.o
gcc -m32 -Wall -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/ -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes device/timer.c -o build/timer.o
nasm -f elf kernel/kernel.S -o build/kernel.o
nasm -f elf lib/kernel/print.S -o build/print.o
gcc -m32 -Wall -I lib/ -I lib/kernel -I lib/user/ -I kernel/ -I device/ -c -fno-builtin -W -Wstrict-prototypes -Wmissing-prototypes kernel/debug.c -o build/debug.o
ld -m elf_i386 -Ttext 0xc0001500 -e main -Map ./build/kernel.map build/main.o build/init.o build/interrupt.o build/timer.o build/kernel.o build/print.o build/debug.o -o build/kernel.bin
$ make hd
dd if=./build/kernel.bin of=/home/ml/bochs/hd60M.img bs=512 count=200 seek=9 conv=notrunc
16+1 records in
16+1 records out
8332 bytes (8.3 kB, 8.1 KiB) copied, 0.000157152 s, 53.0 MB/s
```













## Reference 

1. 
