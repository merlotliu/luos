---
title: Inline Assembly
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# 内联汇编

## 基本内联汇编

### 基本格式

```c
asm [volatile] ("assembly code")
// asm 是 gcc 定义的宏： #define __asm__ asm
// volatile 是 gcc 定义的宏： #define __volatile__ volatile 指定该宏后，编译器不会在优化时修改这段代码
```

"assembly code"是编写的代码，必须位于圆括号中，且代码在双引号中。其基本规则：

- 指令必须在双引号中，不论时一条还是多条指令；
- 一对双引号不可换行，换行需使用 `\` 衔接；
- 指令之间用 `;` `\n` `\n\t` 分隔；

如：

```c
asm("movl $9, %eax;""pushl %eax”) 正确
asm("movl $9, %eax""pushl % eax) 错误
```

gcc 会将多个引号的内容合并，因此指令之间的分割符是必须的，当然最后一条指令可以没有。 

### 示例

```c
char *str = "hello,world\n";
int count = 0;

void main() {

asm("pusha; \
    movl $4, %eax; \
    movl $1, %ebx; \
    movl str, %ecx; \
    movl $12, %edx; \
    int $0x80; \
    mov %eax, count; \
    popa \
    "
);

}
```

### 编译运行

```shell
$ gcc -m32 -o inlineASM.bin inlineASM.c
$ ls
a  b  c  inlineASM.bin  inlineASM.c
$ ./inlineASM.bin 
hello,world
```

Notes

基本内联汇编中，引用使用 C 变量，必须定义为全局变量。

## 扩展内联汇编

### 基本格式

```c
asm [volatile] ("assemly code" : output : input : c lobber/modify)
```

在基本汇编的基础上，添加了 output 、 input 和 clobber/modify 三项。每一部分都可以省略，在 `:` 后的部分都省略的情况下，`:`也可以省略。

## Troubleshooting

```
g++: selected multilib '32' not installed


solve method:

sudo apt-get install g++-4.4-multilib
```

## Reference 

1. 
