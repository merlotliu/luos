---
title: Malloc & Free
date: 2022-11-10
updated: 2022-11-10
tags: 
- [OS, malloc, free]
categories: 
- [OS, malloc, free]
comments: true
---

# Malloc & Free







## 2022.11.10

### bug

测试两个内核线程 sys_malloc & sys_free 出现了缺页中断的情况。

### resolve

尚未解决，目前发现，只有当申请页面大于 1024 的情况下才会异常。



















## Reference 

1. 
