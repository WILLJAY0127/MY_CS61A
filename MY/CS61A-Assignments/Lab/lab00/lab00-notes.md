---
lab: 00
title: Getting Started
status: complete
---

# Lab 0 · Getting Started

> **核心内容**: Python 环境配置 + Python Basics 入门题。
>
> **原始**: `SOURCE/CS61A-Website-Archive/lab/lab00/index.html`

## Python Basics · Suite 1 · Case 1

### 题目

```python
>>> x = 20
>>> x + 2
?
>>> x
?
>>> y = 5
>>> y = y + 3
>>> y * 2
?
>>> y + x
?
```

### 答案

```
22
20
16
28
```

### 逐行解释

```python
x = 20       # statement：把名字 x 绑定到值 20
x + 2        # expression：求值 20 + 2 = 22（不改变 x）
x            # expression：去 environment 查 x 的 binding → 20（没变！）

y = 5        # statement：y 绑到 5
y = y + 3    # statement：先 evaluate 右边 y+3 = 8，再把 y 重绑到 8
y * 2        # expression：8 * 2 = 16
y + x        # expression：8 + 20 = 28
```

### 考点（对接 Lec01&02）

**1. expression vs statement 的区别**

| | expression | statement |
|---|---|---|
| 走什么规则 | evaluate（求值） | execute（执行） |
| 产出什么 | value（值） | 无返回值，改环境或改流程 |
| 例子 | `x + 2`、`y * 2`、`y + x` | `x = 20`、`y = y + 3` |

**2. assignment 先算右边，再绑左边**

`y = y + 3` 不是"y 和 y+3 绑定关系"，而是：
1. **先** evaluate 右边 `y + 3` → 此时 y 还是 5，所以 5 + 3 = 8
2. **再** 把左边 y 重绑到 8

**3. assignment 绑值不绑关系（不同步）**

`x + 2` 不改变 x——x 始终绑着 20。这就是 Lec01&02 讲的 radius/area 不同步现象的极简版：赋值绑的是**值**，不是**关系**。
