---
title: Kernel
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# Kernel

```shell
$ gcc -c -o kernel/main.o kernel/main.c 
$ ls kernel/
main.c  main.o
$ file kernel/main.o
kernel/main.o: ELF 64-bit LSB relocatable, x86-64, version 1 (SYSV), not stripped
$ nm kernel/main.o
0000000000000000 T main
# -e
$ ld kernel/main.o -Ttext 0xc0001500 -e main -o kernel/kernel.bin
# no -e
$ ld kernel/main.o -Ttext 0xc0001500 -o kernel/kernel.bin
ld: warning: cannot find entry symbol _start; defaulting to 00000000c0001500

# main -> _start
$ gcc -c -o kernel/main.o kernel/main.c 
$ ld kernel/main.o -Ttext 0xc0001500 -o kernel/kernel.bin
$ file kernel/kernel.bin 
kernel/kernel.bin: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped


$ nm kernel/kernel.bin 
00000000c0202000 R __bss_start
00000000c0202000 R _edata
00000000c0202000 R _end
00000000c0001500 T main
$ nm tmp/main.bin 
0000000000601030 B __bss_start
0000000000601030 b completed.7594
0000000000601020 D __data_start
0000000000601020 W data_start
0000000000400410 t deregister_tm_clones
0000000000400490 t __do_global_dtors_aux
0000000000600e18 t __do_global_dtors_aux_fini_array_entry
0000000000601028 D __dso_handle
0000000000600e28 d _DYNAMIC
0000000000601030 D _edata
0000000000601038 B _end
0000000000400554 T _fini
00000000004004b0 t frame_dummy
0000000000600e10 t __frame_dummy_init_array_entry
0000000000400688 r __FRAME_END__
0000000000601000 d _GLOBAL_OFFSET_TABLE_
                 w __gmon_start__
0000000000400564 r __GNU_EH_FRAME_HDR
0000000000400390 T _init
0000000000600e18 t __init_array_end
0000000000600e10 t __init_array_start
0000000000400560 R _IO_stdin_used
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000600e20 d __JCR_END__
0000000000600e20 d __JCR_LIST__
                 w _Jv_RegisterClasses
0000000000400550 T __libc_csu_fini
00000000004004e0 T __libc_csu_init
                 U __libc_start_main@@GLIBC_2.2.5
00000000004004d6 T main
0000000000400450 t register_tm_clones
00000000004003e0 T _start
0000000000601030 D __TMC_END__
```







## 加载内核

```assembly
mov eax, KERNEL_START_SECTOR
mov ebx, KERNEL_BIN_BASE_ADDR
mov ecx, 200
```

## 初始化内核

```assembly
kernel_init:
	xor eax, eax
	xor ebx, ebx ; ebx 记录程序头表地址
	xor ecx, ecx ; ecx 记录程序头表中的 program header 数量
	xor edx, edx ; edx 记录 program header 尺寸 即 e_phentsize
	
	; 偏移到 e_phentsize 标识 program header 大小
	mov dx, [KERNEL_BIN_BASE_ADDR + 42]
	
	add ebx, KERNEL_BIN_BASE_ADDR
	; 偏移到 e_phnum 标识有几个 program header
	mov cx, [KERNEL_BIN_BASE_ADDR + 44]
	
.each_segment:
	cmp byte [ebx + 0], PT_NULL
	; p_type == PT_NULL 说明该 program header 未使用
	je .PTNULL
	
	; 为函数 memcpy 压入参数，参数从右至左依次压栈
	; 函数原型类似 memcpy(dst, src, size)
	; 偏移到 program header 中 p_filesz所在的位置
	; 即 memcpy 第三个参数 size
	push dword [ebx + 16]
	; 获取 p_offset 的值
	mov eax, [ebx + 4];
	; 加上 kernel.bin 被加载的物理地址，eax 为该段的物理地址
	add eax, KERNEL_BIN_BASE_ADDR
	; memcpy 第2个参数 : src
	push eax
    ; memcpy 第1个参数 : dest
	; 偏移 8 个字节位置是 p_vaddr 即目的地址
	push dword [ebx + 8]
	; 调用 mem_cpy 完成段复制
	call mem_cpy
	; 清理栈中压入的3个参数
	add esp, 12
	
.PTNULL:
	; edx 为 program header 大小 即e_phentsize
	; 此时 ebx 指向下一个 program header
	add ebx, edx
	
	loop .each_segment
	ret
	
; 逐词解拷贝 mem_cpy
mem_cpy:
	cld
	push ebp
	mov ebp, esp
	push ecx
	
	mov edi, [ebp + 8] ; dst
	mov esi, [ebp + 12] ; src
	mov ecx, [ebp + 16] ; size
	rep movsb
	
	; recover env
	pop ecx
	pop ebp
	ret
```











## Reference 

1. 
