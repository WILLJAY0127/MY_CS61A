---
type: disc
id: "04"
title: Tree Recursion
status: in_progress
source: SOURCE/CS61A-Website-Archive/disc/disc04/index.html
solution: SOURCE/CS61A-Website-Archive/disc/sol-disc04/index.html
code: none
lectures: Lec09
due: TBD
---

# Discussion 4: Tree Recursion

- **题目来源**：`SOURCE/CS61A-Website-Archive/disc/disc04/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/disc/sol-disc04/index.html`
- **关联讲次**：Lec09 · Tree Recursion（教材 Section 1.7）
- **Due**：TBD

***

## 总起

**一句话定位**：Disc 4 正式命名并练习「树递归」——函数在同一个调用里**调用自己两次或更多次**。从二维网格路径数（Q1），到列表上的「选或不选」（Q2），再到把递归结果拼成列表（Q3）。

**覆盖知识点**

- 树递归的定义：一次调用产生多个递归调用
- 二维状态与两个方向的分支（Q1）
- 「包含首元素 / 跳过首元素」的二元划分（Q2）
- 列表切片 `s[1:]`、`s[2:]` 与列表拼接 `[k] + s`
- 从「返回数字 / 布尔」升级到「返回列表的列表」（Q3）

**题目地图**

| 题号 | 题名 | 类型 | 核心 |
| -- | -- | -- | -- |
| Q1 | Insect Combinatorics | 写代码（树递归） | 二维网格，两个方向分叉 |
| Q2 | Max Product | 写代码（列表 + 树递归） | 用/不用首元素，`max` 聚合 |
| Q3 | Sum Fun | 写代码（列表 + 树递归） | 返回所有可行列表 |

**对应讲义**：Lec09 · Tree Recursion，教材 Section 1.7

***

## Getting Started

**Definition**: Tree recursive functions are functions that call themselves more than once.

> **中文**：树递归函数 = 在函数体里调用自己**不止一次**的函数。

In this discussion, don't use a Python interpreter to run code until you are confident your solution is correct. Figure things out and check your work by *thinking* about what your code will do. Not sure? Talk to your group!

> **中文**：本节不要急着用解释器跑代码。靠推理代码会做什么来验证，不确定就组内讨论。

**[New]** Recursion takes practice. Please don't get discouraged if you're struggling to write recursive functions. Instead, every time you do solve one (even with help or in a group), make note of what you had to realize to make progress. Students improve through practice and reflection.

> **中文**：递归靠练。每做出一道（哪怕是别人帮忙），都记下「自己当时必须想通什么才能推进」。

***

## Tree Recursion

For the following questions, don't start trying to write code right away. Instead, start by describing the recursive case in words. Some examples:

- In `fib` from lecture, the recursive case is to add together the previous two Fibonacci numbers.
- In `double_eights` from lab, the recursive case is to check for double eights in the rest of the number.
- In `count_partitions` from lecture, the recursive case is to partition `n-m` using parts up to size `m` **and** to partition `n` using parts up to size `m-1`.

> **中文**：别一上来就写代码，先用文字描述递归情况。三个例子：`fib` 是把前两个斐波那契数相加；`double_eights` 是去剩下的数位里找双 8；`count_partitions` 是**同时**算「用不超过 m 的部分凑 n-m」**和**「用不超过 m-1 的部分凑 n」。

### Q1: Insect Combinatorics

**原题（English）**

An insect is inside an `m` by `n` grid. The insect starts at the bottom-left corner `(1, 1)` and wants to end up at the top-right corner `(m, n)`. The insect can only move up or to the right. Write a function `paths` that takes the height and width of a grid and returns the number of paths the insect can take from the start to the end. (There is a [closed-form solution](https://en.wikipedia.org/wiki/Closed-form_expression) to this problem, but try to answer it with recursion.)

In the `2` by `2` grid, the insect has two paths from the start to the end. In the `3` by `3` grid, the insect has six paths (only three are shown above).

> **中文**：昆虫在 m×n 网格里，从左下角 `(1, 1)` 走到右上角 `(m, n)`，每步只能向上或向右。写 `paths(m, n)` 返回路径总数。有闭式解，但这题要用递归做。

**Starter**（原样摘录）

```python
def paths(m, n):
    """Return the number of paths from one corner of an
    M by N grid to the opposite corner.

    >>> paths(2, 2)
    2
    >>> paths(5, 7)
    210
    >>> paths(117, 1)
    1
    >>> paths(1, 157)
    1
    """
    "*** YOUR CODE HERE ***"

```

**提示（Hint）**

What happens if the insect hits the upper or rightmost edge of the grid?

> **中文**：如果昆虫走到了网格的上边界或右边界，会怎样？

**答案**

（做完再补）

**验证**

```bash
python3 ok -q paths
```

（待跑）

### Q2: Max Product

**原题（English）**

Implement `max_product`, which takes a list of numbers and returns the maximum product that can be formed by multiplying together non-consecutive elements of the list. Assume that all numbers in the input list are greater than or equal to 1.

> **中文**：`max_product(s)` 从列表里挑**互不相邻**的元素相乘，返回能得到的最大乘积。假设所有元素都 ≥ 1。

**Starter**（原样摘录）

```python
def max_product(s):
    """Return the maximum product of non-consecutive elements of s.

    >>> max_product([10, 3, 1, 9, 2])   # 10 * 9
    90
    >>> max_product([5, 10, 5, 10, 5])  # 5 * 5 * 5
    125
    >>> max_product([])                 # The product of no numbers is 1
    1
    """

```

**提示（Hint）**

First try multiplying the first element by the `max_product` of everything after the first two elements (skipping the second element because it is consecutive with the first), then try skipping the first element and finding the `max_product` of the rest. To find which of these options is better, use `max`.

> **中文**：先试「首元素 × 跳过第二个元素之后的 `max_product`」（因为第二个和第一个相邻，不能同时选），再试「跳过首元素，对剩余部分求 `max_product`」，两者之间用 `max` 取大。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q max_product
```

（待跑）

### Q3: Sum Fun

**原题（English）**

Implement `sums(n, m)`, which takes a total `n` and maximum `m`. It returns a list of all lists:

1. that sum to `n`,
2. that contain only positive numbers up to `m`, and
3. in which no two adjacent numbers are the same.

Two lists with the same numbers in a different order should both be returned.

Here's a recursive approach that matches the template below: build up the `result` list by building all lists that sum to `n` and start with `k`, for each `k` from 1 to `m`. For example, the result of `sums(5, 3)` is made up of three lists:

- `[[1, 3, 1]]` starts with 1,
- `[[2, 1, 2], [2, 3]]` start with 2, and
- `[[3, 2]]` starts with 3.

> **中文**：`sums(n, m)` 返回所有满足三个条件的列表：和为 `n`、元素都是 1..m 的正数、相邻元素不相同。顺序不同的算不同答案。骨架的思路是按首元素分桶：先枚举首元素 `k`（1..m），再把「和为 `n-k` 的所有列表」前面拼上 `k`。

**Hint:** Use `[k] + s` for a number `k` and list `s` to build a list that starts with `k` and then has all the elements of `s`.

```python
>>> k = 2
>>> s = [4, 3, 1]
>>> [k] + s
[2, 4, 3, 1]
```

> **中文**：`[k] + s` 是把 `k` 放到列表 `s` 最前面的写法。

**Starter**（原样摘录）

```python
def sums(n, m):
    """Return lists that sum to n containing positive numbers up to m that
    have no adjacent repeats.

    >>> sums(5, 1)
    []
    >>> sums(5, 2)
    [[2, 1, 2]]
    >>> sums(5, 3)
    [[1, 3, 1], [2, 1, 2], [2, 3], [3, 2]]
    >>> sums(5, 5)
    [[1, 3, 1], [1, 4], [2, 1, 2], [2, 3], [3, 2], [4, 1], [5]]
    >>> sums(6, 3)
    [[1, 2, 1, 2], [1, 2, 3], [1, 3, 2], [2, 1, 2, 1], [2, 1, 3], [2, 3, 1], [3, 1, 2], [3, 2, 1]]
    """
    if n < 0:
        return []
    if n == 0:
        sums_to_zero = []     # The only way to sum to zero using positives
        return [sums_to_zero] # Return a list of all the ways to sum to zero
    result = []
    for k in range(1, m + 1):
        result = result + [ ___ for rest in ___ if rest == [] or ___ ]
    return result
```

**提示（Hint: First Blank）**

`k` is the first number in a list that sums to `n`, and `rest` is the rest of that list, so build a list that sums to `n`.

> **中文**：`k` 是目标列表的第一个数，`rest` 是它后面那一段，把两者拼成一个和为 `n` 的列表。

**提示（Hint: Second Blank）**

Call `sums` to build all of the lists that sum to `n-k` so that they can be used to construct lists that sum to `n` by putting a `k` on the front.

> **中文**：调用 `sums` 生成所有和为 `n-k` 的列表，再把 `k` 拼到前面。

**提示（Hint: Third Blank）**

Here is where you ensure that "no two adjacent numbers are the same." Since `k` will be the first number in the list you're building, it must not be equal to the first element of `rest` (which will be the second number in the list you're building).

> **中文**：这里保证「相邻不同」。`k` 是新列表的第一个数，所以它不能等于 `rest` 的第一个元素（那会成为新列表的第二个数）。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q sums
```

（待跑）

***

## Document the Occasion

> 出勤表与结论讨论，无题目，省略。

***

## 本次踩的坑

- （待补）

***

## 相关概念

- [[Discussion 3 - Recursion]] — 同一个递归三段式，Disc 3 是单分支链式，Disc 4 升级成多分支树
- [[Homework 3 - Recursion, Tree Recursion]] — Q4 `count_dollars` 与 Q2 `max_product` 都是「含首元素 / 不含首元素」的二元划分
- [[Lab 3 - Recursion, Python Lists]] — Q2/Q3 用到的 `s[1:]`、`[k] + s` 正是 Lab 3 前半段练的列表操作
