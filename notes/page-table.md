---
title: Page Table
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# 页表

```assembly
; 创建页目录及页表并初始化页内存位图
call setup_page

; 将描述符表地址和偏移量写入内存 gdt_ptr
sgdt [gdt_ptr]
; gdt_ptr 中前 2 字节为偏移量 后 4 个字节为 GDT 基址
mov ebx, [gdt_ptr+2]
; 将原来的 dgt 描述符中的显存段基址 + 0xc0000000 （3GB~4GB的起始地址）
; 显存段为第 3 个段描述符，每个描述符 8 字节，故偏移 0x8 * 3 = 0x18
; 段描述符的高 4 字节 的高 1 字节为段基址的 31~24 位
; 0xc0000000 中仅最高 1 字节不为 0，以下 or 操作也只会修改最高 1 字节的值
or dword [ebx + 0x18 + 4], 0xc0000000

; 修改 gdt 的基址：加上 0xc0000000
add dword [gdt_ptr + 2], 0xc0000000

; 修改栈指针的基址
add esp, 0xc0000000

; 将 cr3（页目录基址寄存器） 赋值为页目录地址
mov eax, PAGE_DIR_TABLE_POS
mov cr3, eax

; 打开 cr0 的 pg 位（第 31 位）
mov eax, cr0
or eax, 0x80000000
mov cr0, eax

; 开启分页后，重新加载 gdt
lgdt [gdt_ptr]

; 测试代码，正常输出字符 'V' 则成功
mov byte [gs:160],'V'

jmp $
```



boot.inc

```assembly
; 定义目录表的物理地址
PAGE_DIR_TABLE_POS equ 0x100000

; 页表相关属性
; 存在位，1标识在内存中，0标识当前页不在内存中，触发 pagefault异常
PG_P equ 1b
; 读写位，1标识可读可写，0标识只读不可写
PG_RW_R equ 00b
PG_RW_W equ 10b
; 左边第1位标识 US 值
; 0 : 不能被特权级 3 的任务访问
; 1 : 所有任务都可以访问
PG_US_S equ 000b
PG_US_U equ 100b
```





```assembly
; 创建页目录及页表
setup_page:

; 功能：页目录占用的内存空间逐字节清 0
; ecx 使用来控制循环次数的，loop 每次循环会做两件事：
	; 1. [ecx] = [exc]-1
	; 2. [ecx] 是否为0，为0往下执行，不为0跳到 标号所在位置
; esi 当前清空的字节相对于页目录起始位置 PAGE_DIR_TABLE_POS 的偏移
	mov ecx, 4096
	mov esi, 0
.clear_page_dir:
	mov byte [PAGE_DIR_TABLE_POS + esi], 0
	inc esi
	loop .clear_page_dir
	
; create page directory entry
.create_pde:
	mov eax, PAGE_DIR_TABLE_POS
	add eax, 0x1000 ; 0x1000(4kb)偏移到第 0 个页表的位置
	mov ebx, eax
	
	; 所有特权级别都可访问，允许读写，在内存中
	; PG_US_U | PG_RW_W | PG_P = 0x7
	or eax, PG_US_U | PG_RW_W | PG_P
	
	; 现在 eax 中的值为 
	; (PAGE_DIR_TABLE_POS + 0x1000) | PG_US_U | PG_RW_W | PG_P
	; 即第一个页表的地址及其属性
	
	; 为以下几个页表目录项写入属性
	; 第 0 个目录项
	mov [PAGE_DIR_TABLE_POS + 0x0], eax 
	; 第768个目录项，0xc00 以上的目录项用于内核空间
	; 也就是页表的 0xc0000000 ~ 0xffffffff 共1G
	mov [PAGE_DIR_TABLE_POS + 0xc00], eax 
	
	
	; 最后1个目录项,指向页目录表自己的地址
	; 为了之后能动态的操作
	; eax 减去 0x1000 后为
	; (PAGE_DIR_TABLE_POS) | PG_US_U | PG_RW_W | PG_P
	sub eax, 0x1000
	mov [PAGE_DIR_TABLE_POS + 4092], eax
	
	; 1MB 低端内存 / 每页大小 4KB = 256
	; 将 1MB 的低端内存划分为 256 个页
	; 并将地址全部存入第 0 个页表中
	mov ecx, 256
	mov esi, 0
    ; 所有特权级别都可访问，允许读写，在内存中
	mov edx, PG_US_U | PG_RW_W | PG_P
	
; create page table entry
.create_pte:
	; ebx 当前为第 0 个页表的地址
	mov [ebx + esi * 4], edx
	add edx, 0x1000
	inc esi
	loop .create_pte
	
; 将 769 ~ 1022 的页目录项创建出来
; 并固定内核页目录项指向的页表地址
; 这么做的目的是共享内核
; 如果仅创建页目录项而固定769 ~ 1022的内存
; 当程序访问到 4MB 以外的内存之后，创建的新页表信息只会更新在呢一个程序，对其他程序不可见
	mov eax, PAGE_DIR_TABLE_POS
	add eax, 0x2000
	or eax, PG_US_U | PG_RW_W | PG_P
	mov ebx, PAGE_DIR_TABLE_POS
	mov ecx, 254 ; 1022 - 768 个页目录项
	mov esi, 769
	
; create kernel page directory entry
.create_kernel_pde:
	mov [ebx + esi * 4], eax
	inc esi
	add eax, 0x1000
	loop .create_kernel_pde
	ret
```













## Reference 

1. 
