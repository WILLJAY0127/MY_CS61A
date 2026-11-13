---
type: disc
id: "00"
title: Getting Started
status: complete
source: SOURCE/CS61A-Website-Archive/disc/disc00/index.html
solution: SOURCE/CS61A-Website-Archive/disc/sol-disc00/index.html
code: none
lectures: Lec01
due: TBD
---

# Discussion 0: Getting Started

- **题目来源**：`SOURCE/CS61A-Website-Archive/disc/disc00/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/disc/sol-disc00/index.html`
- **关联讲次**：Lec01 · Functions
- **Due**：TBD

---

## 总起

**一句话定位**：Disc 0 是破冰课，四个 Part 里只有 Part 2 有自学价值——只用 `f` / `g` / `h` 和数字 5 拼出最短表达式，结果等于 2024。

**覆盖知识点**

- call expression 的嵌套：每个 `f` / `g` / `h` 调用都是一个 call expression，自底向上求值
- primitive expression 与复合表达式：`5` 是叶子，`h(g(g(5)), ...)` 是内部节点
- 函数作为 building block：把 `f` / `g` / `h` 当作最小积木，组合出任意目标数
- 逆向分解：从目标 2024 往回拆，而不是从 5 正向枚举

**题目地图**：Part 0 / 1 / 3 破冰行政（略）｜Part 2「5 拼 2024」（唯一自学价值）

**对应讲义**：Lec01 · Functions

---

## Part 0: Meet your group and learn their names

（破冰，省略）

## Part 1: Learn about each other

（破冰，省略）

## Part 2: Solve a problem together

### Q1: Small Expression — 用 f/g/h + 5 拼 2024

**原题（English）**

Imagine you can call only the following three functions:

- `f(x)`: Subtracts one from an integer `x`
- `g(x)`: Doubles an integer `x`
- `h(x, y)`: Concatenates the digits of two different positive integers `x` and `y`. For example, `h(789, 12)` evaluates to `78912` and `h(12, 789)` evaluates to `12789`.

**Definition**: A *small expression* is a call expression that contains only `f`, `g`, `h`, the number 5, and parentheses. All of these can be repeated. For example, `h(g(5), f(f(5)))` is a small expression that evaluates to 103.

What's the shortest *small expression* you can find that evaluates to 2024?

> **中文**：只能调三个函数 `f`（减 1）/ `g`（翻倍）/ `h`（拼接十进制串），加上数字 5 和括号，拼出最短表达式等于 2024。`h(789, 12) = 78912`。

**答案**

```
h(g(g(5)),h(f(f(f(5))),f(5)))
```

**验证**

```python
f(5)                        # 4
f(f(f(5)))                  # 2
g(g(5))                     # 20
h(f(f(f(5))), f(5))         # h(2, 4) = 24
h(g(g(5)), h(f(f(f(5))), f(5)))  # h(20, 24) = 2024 ✓
```

**思路**

关键观察：f 只能减 1、g 只能翻倍，两者都无法直接得到 2024——2024 的最后一步一定是 h 拼接两个较小的数。

h 把两个数的十进制串拼起来，所以 2024 可以拆成：

```
h(20, 24)     → 20 和 24 都要从 5 构造（优先选两位数 + 两位数）
h(202, 4)     → 202 本身构造起来太长，不如两位数方案
h(2, 024)     → 不行，前导零无意义
```

优先选两位数 + 两位数的原因：h 拼接位数多的数会让总表达式更长。

构造 building blocks：

| 值 | 表达式 | 备注 |
|---|---|---|
| 4 | `f(5)` | f 一次 |
| 2 | `f(f(f(5)))` | f 三次（5→4→3→2） |
| 20 | `g(g(5))` | g 两次 |

拼 24 也得用 h：`24 = h(2, 4) = h(f(f(f(5))), f(5))`

最终：`2024 = h(20, 24) = h(g(g(5)), h(f(f(f(5))), f(5)))`

**延伸：计算机如何找最短表达式**

Disc 结尾思考题：如何用程序自动搜索？这会在后续 Lec 讲到。

思路框架：
- **状态空间搜索**：每个整数是节点，f/g/h 是操作
- **BFS 最短路径**：从 5 出发逐层应用，第一个到达 2024 的就是最短
- 但 h 可以产生极大数，需要限制搜索空间（只搜目标附近，或剪枝）

这和 Lec 树递归、HW 的 `count_partitions` 是同一种"从简单 building blocks 组合出目标"的思维模式。

**关联知识点**

- call expression 嵌套：每个 f/g/h 调用都是 call expression，递归求值规则自底向上算
- primitive expression：5 是 primitive（叶子），f/g/h 是名字（叶子），`h(g(g(5)), ...)` 是内部节点
- 逆向分解思维：从目标往回拆，而不是从 5 正向枚举——算法里反复出现这种模式

**易错点**

- 忘记 h 拼接后**没有分隔符**：`h(20, 24) = 2024` 不是 `20 24`
- 以为 h 能产生前导零：`h(2, 024)` 里 024 不是合法正整数
- 从 5 正向枚举而不是从 2024 逆向拆——会陷入组合爆炸

---

## Part 3: Document the occasion

（行政，省略）

---

## 本次踩的坑

（这题太简单，没踩坑）

## 相关概念

- [[Lab 0 - Getting Started]] — call expression 嵌套入门，Part 2 的同源知识点
- Lec01 · Functions（`MY/Weekly/`，待写）— expression vs statement、primitive vs compound
