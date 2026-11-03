---
disc: 00
title: Getting Started
status: complete
---

# Discussion 0 · Getting Started

> **核心内容**: 三个函数 f/g/h + 常量 5，拼最短表达式得 2024。配套 Lec01&02 call expression、嵌套组合的理解。
>
> **原始**: `SOURCE/CS61A-Website-Archive/disc/disc00/disc00.pdf`

## 原版题目（Part 2）

> Imagine you can call only the following three functions:
>
> - `f(x)`: Subtracts one from an integer `x`
> - `g(x)`: Doubles an integer `x`
> - `h(x, y)`: Concatenates the digits of two different positive integers `x` and `y`. For example, `h(789, 12)` evaluates to `78912` and `h(12, 789)` evaluates to `12789`.
>
> **Definition**: A *small expression* is a call expression that contains only `f`, `g`, `h`, the number 5, and parentheses. All of these can be repeated. For example, `h(g(5), f(f(5)))` is a small expression that evaluates to 103.
>
> What's the shortest *small expression* you can find that evaluates to 2024?

## 答案（一行）

```
h(g(g(5)),h(f(f(f(5))),f(5)))
```

## 验证

```python
f(5)       # 4
f(f(f(5))) # 2
g(g(5))    # 20
h(f(f(f(5))), f(5))  # h(2, 4) = 24
h(g(g(5)), h(f(f(f(5))), f(5)))  # h(20, 24) = 2024 ✓
```

## 解题思路

### 关键观察：2024 的最后一步一定是 h

f 只能减 1、g 只能翻倍，两者都无法直接得到 2024。所以 2024 的最后一步一定是 h 拼接两个较小的数。

### 逆向分解 2024

h 把两个数的十进制串拼起来，所以 2024 可以拆成：

```
h(20, 24)     → 20 和 24 都要从 5 构造
h(202, 4)     → 202 和 4
h(2, 024)     → 不行，024 不是正整数（前导零无意义）
```

**优先选两位数 + 两位数**。原因：h 拼接位数多的数会让表达式更长（比如 202 本身就需要很多 f/g 来构造）。

### 构造 building blocks

| 值 | 表达式 | 备注 |
|---|---|---|
| 4 | `f(5)` | f 一次 |
| 2 | `f(f(f(5)))` | f 三次（5→4→3→2） |
| 10 | `g(5)` | g 一次 |
| 20 | `g(g(5))` | g 两次 |

### 拼 24

24 还得用 h，24 = h(2, 4)：

```
24 = h(f(f(f(5))), f(5))
```

### 拼 2024

```
2024 = h(20, 24)
     = h(g(g(5)), h(f(f(f(5))), f(5)))
```

去空格就是最终答案。

## 计算机如何找最短表达式

Disc00 结尾的思考题：如何用程序自动搜索？这会在 Lecture 2 讲到。

思路框架：

- **状态空间搜索**：每个整数是一个节点，f/g/h 是操作
- **BFS 最短路径**：从 5 出发，逐层应用 f/g/h，第一个到达 2024 的就是最短
- **但 h 可以产生极大数**，需要限制搜索空间（只搜目标附近的数，或剪枝）

这和 Lec07 树递归、HW01 的 `count_partitions` 是同一种"从简单 building blocks 组合出目标"的思维模式。

## 关联 Lec01&02 的点

1. **call expression 嵌套**：每个 f/g/h 调用都是 call expression，递归求值规则自底向上算
2. **primitive expression**：5 是 primitive（叶子），f/g/h 是名字（叶子），`h(g(g(5)), ...)` 是内部节点
3. **嵌套组合能力**：h 把两个子表达式的结果"粘"在一起，这就是教材 1.2.2 说的 call expression 嵌套的威力
