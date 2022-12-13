---
title: Timer
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# Timer



```shell
$ gcc -m32 -I lib/kernel/ -c -o build/timer.o device/timer.c
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -c -fno-builtin -o build/main.o  kernel/main.c 
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -I device/ -c -fno-builtin -o build/init.o  kernel/init.c 
$ gcc -m32 -I lib/kernel/ -I lib/ -I kernel/ -c -fno-builtin -o build/interrupt.o  kernel/interrupt.c 

$ nasm -f elf -o build/kernel.o kernel/kernel.S 
$ nasm -f elf -o build/print.o lib/kernel/print.S

$ ld -m elf_i386 -Ttext 0xc0001500 -e main -o build/kernel.bin build/main.o build/init.o build/interrupt.o build/print.o build/kernel.o build/timer.o 
$ dd if=build/kernel.bin of=../hd60M.img bs=512 count=200 seek=9 conv=notrunc
15+1 records in
15+1 records out
7688 bytes (7.7 kB, 7.5 KiB) copied, 0.00170896 s, 4.5 MB/s
ml@sakura:~/bochs/los$ 

```





















## Reference 

1. 
