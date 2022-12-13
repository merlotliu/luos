---
title: Print
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# 自定义打印函数

## 单个字符打印





## 字符串打印

C 语言定义的字符串，编译后会以 `\0` （ascii 为 0）结尾，因而我们可以通过 `\0` 的出现判断字符串的结束，反之循环调用 `put_char` 打印每个字符。

```assembly
[bits 32]
section .text

;----------------------------------------
; put_str 
; brief : print string whose end is '\0'
;----------------------------------------
; input : string
; output : none
global put_str
put_str:
	push ebx
	push ecx
	xor ecx, ecx ; clear ecx
	mov ebx, [esp + 12] ; string address
	
.goon:
	mov cl, [ebx] ; get char
	cmp cl, 0 ; is or not reach string tail ('\0')
	jz .str_over
	push ecx ; push arg for put_char
	call put_char
	add esp, 4 ; recycle arg space of stack
	inc ebx ; mov to next char
	jmp .goon

.str_over:
	pop ecx
	pop ebx
	ret
```

### 代码详解

```assembly
global put_str
```

导出 `put_str` 函数供外部其他文件调用。

```assembly
put_str:
    push ebx
    push ecx
```

将 `ebx`  和 `ecx` 压入栈中备份。`ebx` 用于存放字符地址，开始为字符串首地址，然后依次访问后面的字符。`ecx` 为记录当前字符。

为什么栈中记录的是字符串首地址而非字符？因为字符串长度并不确定，有可能导致栈溢出，回收参数空间也不确定。

```assembly
    xor ecx, ecx
    mov ebx, [esp + 12]
```

清空 `ecx` 寄存器。将字符串首地址记录在 `ebx`。

为什么是 `esp + 12` ？在函数 `put_str` 调用前先，将字符串首地址压入栈中，随后依次将函数返回地址（4）、寄存器 `ebx`（4）  和  `ecx`（4） 共 12 字节，压入栈中。

```assembly
.goon:
	mov cl, [ebx]
	cmp cl, 0
	jz .str_over
	push ecx
	call put_char
	add esp, 4
	inc ebx
	jmp .goon
```

读取 `ebx` 中地址所在字符，记录在 `cl` ，判断是否为 `'\0'`，是表示到达字符串尾部，跳转到 `str_over` ，否则继续向下执行。将 `ecx` ，即当前需要处理的字符作为参数压栈，随即调用 `put_char` 打印字符。处理完字符，回收栈空间，此时栈指针指向刚才打印的字符，仅 1 个参数（4 字节），故 `esp` 增加 4 字节。然后，保存字符地址的 `ebx` ++，即变为下一个字符地址，继续循环。

```assembly
.str_over:
	pop ecx
	pop ebx
	ret
```

字符串已到达末尾，恢复 `ecx` 和 `ebx` ，并返回。

## 整数打印

该函数的功能是将 32 位整型数字转化成字符后输出，输出的内容为 16 进制整数（不含 0x）。转化原理：按照 16 进制处理 32 位数字，每 4 位二进制表示 1 位 16 进制，再将各 16 进制转化为对应字符，共 8 个 16 进制数字。

```assembly
section .data
	put_int_buffer dq 0 ; buffer for converting number to char

;----------------------------------------
; put_int 
; brief : print 32 bits 16 based integer
;----------------------------------------
; input : 32 bits 16 based integer
; output : 32 bits 16 based integer string
global put_int
put_int:
	pushad
	mov ebp, esp
	mov eax, [ebp + 4 * 9]
	mov edx, eax
	mov ebx, put_int_buffer
	mov	edi, 7
	mov ecx, 8
	
    .16based_4bits:
    	and edx, 0x0000000f
    	cmp edx, 9
    	jg .is_a2f
    	add edx, '0'
    	jmp .store
    	
    	.is_a2f:
    		sub edx, 10
    		add edx, 'a'
            
        .store:
        	mov [ebx + edi], dl
        	dec edi
        	shr eax, 4
        	mov edx, eax
        	loop .16based_4bits
    
    .ready_to_print:
    	inc edi
        .skip_prefix_0:
            cmp edi, 8
            je .full0
            .go_on_skip:
                mov cl, [put_int_buffer + edi]
                inc edi
                cmp cl, '0'
                je .skip_prefix_0
                dec edi
                jmp .put_each_num
            .full0:
                mov cl, '0'
    .put_each_num:
		push ecx
		call put_char
		add esp, 4
		inc edi
		mov cl, [put_int_buffer + edi]
		cmp edi, 8
		jl .put_each_num
		popad
		ret
```

### 代码详解

```assembly
section .data
	put_int_buffer dq 0
```

32 位数字将处理为 8 个 16 进制数，故定义了一个 8 字节缓冲区。`dq` 为编译器伪指令，q 为 quad 简写，表示 4 ，`dq` 即 8 字节。

```assembly
put_int:
	pushad
	mov ebp, esp
	mov eax, [ebp + 4 * 9]
	mov edx, eax
	mov ebx, put_int_buffer
	mov	edi, 7
	mov ecx, 8
```

首先 `pushad` 将所有寄存器压栈备份。

`ebx` 获取栈内参数，虽然 32 位系统支持 `esp` 栈内寻址，但仍建议使用 `ebp` 获取参数（`esp` 值会受到压栈出栈的影响，`ebp`只有手动修改才会改变）。

`eax` 记录参数，作为备份，为 `edx` 赋值。

`edx` 每次操作参数前，先获取 `eax` 的值，然后进行转化操作。

`ebx` 记录缓冲区首地址

`edi` 记录缓冲区 put_int_buffer 内的偏移。

`ecx` 记录需要处理的 16 进制个数。

```assembly
	.16based_4bits:
    	and edx, 0x0000000f
    	cmp edx, 9
    	jg .is_a2f
    	add edx, '0'
    	jmp .store

		.is_a2f:
    		sub edx, 10
    		add edx, 'a'
```

`.16based_4bits` 部分，从 32 位数字的低位开始，每次处理 4 位二进制数字，为 1 位 16 进制，并存储在缓冲区。

`edx` 与操作只保留当前处理位置的低 4 位。如果当前 16 进制数在 0-9之间，加上 `'0'`的 ascii，否则说明在 a-f，edx - 10 + 'a' ，得到的就是当前 16 进制数的字符。然后进入到 `.store` 将字符存储到缓冲区。

```assembly
		.store:
        	mov [ebx + edi], dl
        	dec edi
        	shr eax, 4
        	mov edx, eax
        	loop .16based_4bits
```

`dl` 为 `edx` 的低 4 位，即为当前处理的 16 进制数的字符表示，存储在缓冲区偏移 `edi` 位置的地址，随后偏移量 - 1。`edi` 初始为 7，即 8 字节缓冲的最后一个字节（即最后缓冲区的字节序为大端格式）， 这样后期打印字符时，从缓冲首地址逐字节偏移打印即可，当然这不是必须的。

接着，`eax` 右移 4 位将，接下来需要处理的 16 进制数的 4 位二进制移到低 4 位，并将值赋值给 `edx`，继续新一轮循环。

```assembly
	.ready_to_print:
    	inc edi
        .skip_prefix_0:
            cmp edi, 8
            je .full0
            .go_on_skip:
                mov cl, [put_int_buffer + edi]
                inc edi
                cmp cl, '0'
            je .skip_prefix_0    
            dec edi
            jmp .put_each_num
            .full0:
                mov cl, '0'
```

运行到这，说明 32 位数已经处理完毕。这一部分的工作主要是，丢弃所有前缀 0，打印剩余部分字符，当然如果全为 0 ，只需要打印 0 即可。

如果 `edi` 增加到 8，说明全部为 0，直接跳转到打印。否则，获取缓冲区当前字符，与 0 对比，如果为 0，偏移到下一个字符，继续循环判断，如果不为 0，说明前导 0 处理完全，跳转到字符打印部分 put_each_num。

```assembly
	.put_each_num:
		push ecx
		call put_char
		add esp, 4
		inc edi
		mov cl, [put_int_buffer + edi]
		cmp edi, 8
		jl .put_each_num
		popad
		ret
```

从左起第一个非 0 位置开始逐字节打印，直至末尾。最后将寄存器数据恢复，并返回。

## Reference 

1. 
