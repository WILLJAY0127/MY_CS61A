---
type: disc
id: "04"
title: Tree Recursion
status: complete
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
- <br />
- 列表拼接 `[k] + s`
- 从「返回数字 / 布尔」升级到「返回列表的列表」（Q3）

**题目地图**

| 题号 | 题名                   | 类型            | 核心               |
| -- | -------------------- | ------------- | ---------------- |
| Q1 | Insect Combinatorics | 写代码（树递归）      | 二维网格，两个方向分叉      |
| Q2 | Max Product          | 写代码（列表 + 树递归） | 用/不用首元素，`max` 聚合 |
| Q3 | Sum Fun              | 写代码（列表 + 树递归） | 返回所有可行列表         |

**对应讲义**：Lec09 · Tree Recursion，教材 Section 1.7

***

## Getting Started

**Definition**: Tree recursive functions are functions that call themselves more than once.

> **中文**：树递归函数 = 在函数体里调用自己**不止一次**的函数。

In this discussion, don't use a Python interpreter to run code until you are confident your solution is correct. Figure things out and check your work by *thinking* about what your code will do. Not sure? Talk to your group!

> **中文**：本节不要急着用解释器跑代码。靠推理代码会做什么来验证，不确定就组内讨论。

**\[New]** Recursion takes practice. Please don't get discouraged if you're struggling to write recursive functions. Instead, every time you do solve one (even with help or in a group), make note of what you had to realize to make progress. Students improve through practice and reflection.

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

```python
def paths(m, n):
    def path(a, b):
        if (a == m) and (b == n):
            return 1
        elif a == m:
            return path(a, b + 1)
        elif b == n:
            return path(a + 1, b)
        else:
            return path(a, b + 1) + path(a + 1, b)
    return path(1, 1)
```

**思路**

- 先用文字写递归情况：昆虫站在 `(a, b)`，还没到终点时只有两个动作——往上走 `(a+1, b)` 或往右走 `(a, b+1)`，两者互不重叠，所以「从 `(a, b)` 出发的路径总数」= 两个方向各自的路径数**相加**
- 这就是 Hint 要问的东西：走到上边界 `a == m` 后往上会出格，只剩往右一种走法；走到右边界 `b == n` 后只剩往上一种。**一旦只剩一条路，分支数从 2 降到 1**，所以写成直接 `return` 一个递归调用，而不是两个相加
- 终点判定 `(a == m) and (b == n)` 必须放在最前面：两个边界可能同时成立，若先判单边分支，终点会被当成「只剩一个方向」继续往下走，走出去就死循环
- 用嵌套 helper `path(a, b)` 表示「当前格子」，外层 `m`、`n` 是闭包里固定不变的目标，不必作为参数层层传递；起点是 `(1, 1)`，因为题目坐标从 1 开始，不是 0
- 每层必有一维 +1，且边界分支把另一维锁死，所以递归一定在有限步内撞到 `(m, n)`，不会无限展开

**关联知识点**

- 递归三段式（[[Discussion 3 - Recursion]]）：base case / 递归调用 / 组合返回值；这里组合符是 `+`（计数），与 Lab 3 Q8 的 `or`（存在性）、Q2 的 `max`（最优）同一骨架
- 教材 Section 1.7 Tree Recursion：本函数一次调用最多产生 **两个** 递归调用，是树递归的最小形态
- 计数型树递归：与 [[Homework 3 - Recursion, Tree Recursion]] Q4 `count_dollars` 同构——「每个分支代表一种选择」，叶子节点累计为答案
- 闭包：`path` 里直接读外层 `m`、`n`，属于 Lec 04 的高阶函数/闭包概念在递归里的复用

**易错点**

- `a == m & b == n`：`&` 是位运算符，优先级**高于** `==`，Python 实际解析成链式比较 `a == (m & b) == n`，语义完全变了。修法：写 `and`（优先级低于比较，可省括号），或给两个比较各加括号
- 起点写成 `(0, 0)`：坐标是 1-based，从 0 起算会多出一整行和一整列，`paths(2, 2)` 会返回 6 而不是 2
- 把边界分支也写成两条相加：往网格外走会永远撞不到 `(m, n)`，无限递归
- 递归调用忘了 `return`（单独一行 `path(a, b + 1)`）：返回值被丢弃，函数返回 `None`，外层相加时报 `TypeError`
- 用 `return paths(...)` 递归自己而不是 `path(...)`：每次调用都重置起点，且重新定义 helper，永远走不到终点

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
    if s == []:
        return 1
    return max(s[0] * max_product(s[2:]), max_product(s[1:]))
```

**思路**

- 划分维度：**是否包含首元素 `s[0]`**。它只有两个取值（选 / 不选），所以是两支——分支数由维度决定，不是背出来的
- 支 1「选 `s[0]`」：`s[1]` 与它相邻，被迫不能选，剩下的 `s[2:]` 自由选 → `s[0] * max_product(s[2:])`
- 支 2「不选 `s[0]`」：`s[1]` 解禁，剩下 `s[1:]` 自由选 → `max_product(s[1:])`
- 两支的子问题起点不同（`s[2:]` vs `s[1:]`），这是「选首元素要连带牺牲第二个」的直接体现
- 聚合符用 `max`：题目要的是「最优」而不是计数或求和；三题对照见下方关联知识点
- base case 是空列表返回 `1`：`[]` 表示「一个都不选」，乘法单位元是 1（加法单位元才是 0）

**关联知识点**

- 递归三段式的组合符可换，骨架不变：`+` 计数（[[Homework 3 - Recursion, Tree Recursion]] Q4 `count_dollars`）、`or` 存在性（[[Lab 3 - Recursion, Python Lists]] Q8 `make_onion`）、`max` 最优（本题）
- 与 `count_dollars` 同构：那边的维度是「用不用一张当前面额」→ 2 支，这边是「用不用首元素」→ 2 支；参数递减方式不同，分支结构相同
- `s[2:]` / `s[1:]` 来自 Lec 10 · Sequences 的切片语法，Lab 3 已练过
- 教材 Section 1.7：一次调用产生两个递归调用，属于最小形态的树递归

**易错点**

- 漏 base case：`s[0]` 在空列表上直接 `IndexError`，而 `max_product([])` 恰恰是官方 doctest 的第三个用例
- base case 返回 `0` 而不是 `1`：空乘法的结果必须是乘法的单位元 1，返回 0 会让所有答案变成 0
- 支 1 写成 `max_product(s[1:])`：忘了 `s[1]` 与 `s[0]` 相邻不能选，会选出相邻元素；应是 `s[2:]`
- 递归调用名拼错（`max_prouct`）：运行时 `NameError`
- 用 `+` 或 `sum` 聚合两支：那是「计数/求和」的写法，本题要比较大小，必须 `max`

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
        result = result + [[k] + rest for rest in sums(n - k, m) if rest == [] or rest[0] != k]
    return result
```

**思路**

- 划分维度：**首元素 `k` 取 1..m 中的哪个值**。取值个数 = m，与输入规模无关但由参数 `m` 决定，写不成固定 if 链 → 用 `for` 把 m 条分支展开（对照 Q1/Q2 的固定 2 支用 if）
- 选定 `k` 后，`rest` 要有三条性质：和为 `n-k`、元素 ≤ `m`、内部相邻不同——这三条恰好是 `sums(n - k, m)` 的定义，直接拿它的返回值当候选
- 第四条约束是**拼上去才出现的**：`k` 会紧挨着 `rest[0]`，所以必须 `rest[0] != k`，递归自己不知道这件事
- `rest == []` 放在 `or` 左边是为了短路：`k == n` 时 `sums(0, m)` 返回 `[[]]`，rest 为空，若先算 `rest[0]` 会 `IndexError`。而这条分支正好给出 `[k]` 本身，如 `sums(5, 5)` 里的 `[5]`
- `sums(n - k, m)` 的 `m` 不变：`m` 是「允许的最大值」这个上限，不是剩余额度；每层只把 `n` 减小 `k`
- base case 必须是两段：`n < 0` 表示超过目标、无解 → `[]`；`n == 0` 表示恰好拆完、只有「什么都不加」这一种拆法 → `[[]]`。合并成 `n <= 0` 会把「无解」误判成「有解」
- `result + [...]` 是把 k=1、k=2、…、k=m 各组结果**首尾相接**，得到扁平的结果列表

**关联知识点**

- 列表推导式（Lec 10 · Sequences）在这里的角色：**展开参数化的 m 条分支**，同时做筛选（`if` 部分）和变换（`[k] + rest`）
- 列表拼接：`[k] + rest` 造新列表，`result + group` 拼接列表的列表；两者都是 `+`，但操作对象层级不同
- 与 Q1/Q2 的分支数对照：Q1「上/右」2 支（if）、Q2「用/不用首元素」2 支（if + max）、Q3「首元素取哪个」m 支（for + 推导式）——**分支数由维度决定，表达方式由分支数是否固定决定**
- 与 [[Homework 3 - Recursion, Tree Recursion]] Q4 `count_dollars` 同源：都是「按第一个选择分类」，那边累加计数（`+`），这边拼接具体方案（`+` 列表）

**易错点**

- 把两个 base case 合并成 `n <= 0`：`sums(5, 1)` 会返回 `[[]]` 而不是 `[]`，因为「拆过头」被当成了「拆完」
- `rest == []` 写在 `or` 右边：rest 为空时先算 `rest[0]` → `IndexError`
- 漏掉 `rest[0] != k`：会出现 `[2, 2, 1]` 这类相邻重复，`sums(5, 3)` 的 `k=2` 组会多出 `[2,2,1]`
- 用 `result.append([[k] + rest ...])`：`append` 把整组当成一个元素塞进去，结果多一层嵌套；要拼接子列表应写 `result = result + [...]` 或 `result.extend([...])`
- 递归写成 `sums(n - k, k)`：把上限也缩小了，会漏掉「后面出现比 k 大的数」的所有方案（如 `sums(5, 3)` 里 `k=1` 之后的 `3`）
- 忘记 `return result`：循环跑完返回 `None`

***

## Document the Occasion

> 出勤表与结论讨论，无题目，省略。

***

## 验收

| 题号 | 状态 |
| -- | -- |
| Q1 | 完成（手推 4 个 doctest 通过，Disc 无 ok） |
| Q2 | 完成（手推 3 个 doctest 通过，Disc 无 ok） |
| Q3 | 完成（doctest 5/5 通过：`failed=0, attempted=5`） |

***

## 本次踩的坑

- **`&` 当 `and` 用**：`a == m & b == n` 解析成 `a == (m & b) == n`，因为 `&` 优先级高于 `==`。比较条件一律用 `and`/`or`，`&`/`|` 只用于位运算 → 可升级进 `Common_Pitfalls.md`（Python 运算符优先级）
- **起点写成 `(0, 0)`**：坐标 1-based 的题，递归起点必须跟题目叙述一致；用 `paths(2, 2) == 2` 这种小样例自查一步就能发现
- **递归启动名写错**：嵌套 helper 时 `return path(1, 1)`，别写成 `return paths(1, 1)`——后者会重新定义 helper 并重置状态
- **乘法递归的 base case 是 1 不是 0**：空乘法的单位元是 1（`1` 乘任何数不变），空加法的单位元才是 0。写 `return 0` 会让 `max_product` 全返回 0 → 可升级进 `Common_Pitfalls.md`
- **分支数由划分维度决定**：Q1「上/右」2 支、Q2「用/不用首元素」2 支、Q3「首元素取 1..m」m 支。写代码前先问「维度是什么、有几个取值」
- **`or` 的短路顺序要用上**：`rest == [] or rest[0] != k` 里，空列表判断必须在前，否则 `rest[0]` 先求值直接 `IndexError`
- **`append` 和 `+` 的层级差**：`result.append(group)` 会把整组塞成一个元素（多一层嵌套），`result + group` 才是首尾相接
- **base case 不能贪合并**：`n < 0 → []` 和 `n == 0 → [[]]` 语义相反，写成 `n <= 0` 会把「无解」当成「空解」

***

## 相关概念

- \[\[Discussion 3 - Recursion]] — 同一个递归三段式，Disc 3 是单分支链式，Disc 4 升级成多分支树
- [[Homework 3 - Recursion, Tree Recursion]] — Q4 `count_dollars` 与 Q2 `max_product` 都是「含首元素 / 不含首元素」的二元划分
- [[Lab 3 - Recursion, Python Lists]] — Q2/Q3 用到的 `s[1:]`、`[k] + s` 正是 Lab 3 前半段练的列表操作
