---
title: Shell
date: 2022-11-17
updated: 2022-11-17
tags: 
- [OS]
categories: 
- [OS]
comments: true
---

# SHELL







## 字符串解析

根据分隔符 delim 对字符串 s 分割，save_ptr 保存下一次分割的起始位置指针，返回值为本次分割的字符串指针。

如： ""

```c
char s[] = "-abc-def";
char *sp;
x = strtok(s, "-", &sp); // x = "abc", sp = "=def"
x = strtok(NULL, "-", &sp); // x = "def", sp = NULL
x = strtok(NULL, "-", &sp); // x = NULL
// s = "abc\0def\0"
```

### 函数签名

```c
char* strtok(char *s, const char delim, char **save_ptr);
```

### 实现

1. 首先跳过开头连续的 delim 字符；
2. 找到剩下部分中，delim 出现在最左边的位置，将该位置置为0，记录下一次开始的位置，返回本次截取到的字符串地址。





















## Reference 

1. 
