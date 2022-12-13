---
title: File System
date: 2022-11-13
updated: 2022-11-13
tags: 
- [OS]
categories: 
- [OS]
comments: true
---

# File System

## 超级块、inode结点、目录项





## 创建文件系统





## 文件描述符

进程或线程中存在一个文件描述符表的属性，每个元素为全局的文件表中的元素，文件表中的元素再指向全局的inode队列（缓冲队列），inode最后指向文件块。





## 创建文件







### 2022.11.14

#### bug

读取的扇区超过最大扇区数

#### resolve

未打开根目录和初始化文件表





## 打开文件

成功返回文件描述符，否则返回 -1。





## Reference 

1. 
