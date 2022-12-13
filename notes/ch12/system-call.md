---
title: System Call
date: {{ date }}
updated: {{ date }}
tags: 
categories: 
comments: true
---

# System Call

添加系统调用的步骤：

1. 在枚举项 SYSCALL_NR 添加新的子功能号；
2. 添加系统调用的用户接口；
3. 定义子功能处理函数，并注册在 syscall_table （int 0x80 中断会调用该数组中对应子功能号的函数）；





## 2022.11.9

### bug

make 编译时出现以下错误

```shell
undefined reference to `__stack_chk_fail'
```

### resolve

原因是定义数组的栈操作引起栈溢出。

1. 添加 `-fno-stack-protector` （不需要栈保护）；
2. 修改数组的大小为合适的值；
3. 将数组定义为静态变量或动态在堆中申请；



















## Reference 

1. 
