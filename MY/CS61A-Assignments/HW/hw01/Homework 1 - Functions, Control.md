---
hw: 01
title: Functions, Control
status: complete
---

# Homework 1 · Functions, Control

> **原始题目**: `SOURCE/CS61A-Website-Archive/hw/hw01/index.html`
>
> **官方解答参考**: `SOURCE/CS61A-Website-Archive/sol-hw01/`（仅作映射，不作为结构来源）

---

## Required Questions

### Q1: A Plus Abs B

> Return `a + abs(b)`, but without calling `abs`.
>
> **Hint**: 题目开头 `from operator import add, sub` 已经给你了两个名字。想清楚什么时候用 add，什么时候用 sub。

```python
def a_plus_abs_b(a, b):
    if b < 0:
        f = _____
    else:
        f = _____
    return f(a, b)
```

**答案**

```python
from operator import add, sub

def a_plus_abs_b(a, b):
    if b < 0:
        f = sub
    else:
        f = add
    return f(a, b)
```

**验证**

```python
a_plus_abs_b(2, 3)    # 5
a_plus_abs_b(2, -3)   # 5
a_plus_abs_b(-1, 4)   # 3
a_plus_abs_b(-1, -4)  # 3
```

**解题思路**

核心：f 要绑**函数本身**，不是绑函数算出来的结果。

写 `f = a - b` 会报 `TypeError: 'int' object is not callable`。因为 assignment 先求值右边，`a - b` 是 call expression，求值结果是 int，不是 function。而 `f = sub` 绑的是 function object，`f(a, b)` 就能正常调用。

数学关系：

| 情况 | a + abs(b) 等价于 |
|---|---|
| b < 0 | a - b → `sub(a, b)` |
| b ≥ 0 | a + b → `add(a, b)` |

**考点**（对接 Lec01&02）

- **function 是 first-class value**：可以像数字一样绑到名字上（`f = sub`）
- **assignment 先算右边再绑左边**：`f = a - b` 里的 `a - b` 会立刻求值
- **intrinsic name vs bound name**：`f` 是 bound name，`sub` 是它 intrinsic name，两者指向同一 function object

---

### Q2: Two of Three

> Return `m*m + n*n`, where `m` and `n` are the two smallest members of the positive numbers `i`, `j`, and `k`.
>
> **约束**：函数体**只能有一条 return 语句**（syntax_check 会检查 AST，不允许 if/赋值）。

```python
def two_of_three(i, j, k):
    return ____
```

**答案**

```python
def two_of_three(i, j, k):
    return i*i + j*j + k*k - max(i*i, j*j, k*k)
```

**验证**

```python
two_of_three(1, 2, 3)    # 5   = 1+4
two_of_three(5, 3, 1)    # 10  = 9+1
two_of_three(10, 2, 8)   # 68  = 4+64
two_of_three(5, 5, 5)    # 50  = 25+25
```

**解题思路**

把"选两个最小"转化成"总和减最大"——三个数的平方和减去最大的平方，剩下的正好是两个最小的平方和。不需要 if 判断谁是最大，直接靠 `max` 选出来减掉。这就是"用减法代替选择"的思维。

**考点**（对接 Lec01&02）

- **纯表达式组合**：不用 statement（if/赋值），只用 call expression 嵌套
- **pure function 可嵌套**：`max` 是纯函数，所以能安全嵌在算术表达式里
- **一条 expression 完成逻辑**：教材 1.2.2 讲的 call expression 组合能力

---

### Q3: Largest Factor

> Return the largest factor of `n` that is smaller than `n`.
>
> **Hint**: 从 n-1 往下找，第一个能整除 n 的就是最大因数。

```python
def largest_factor(n):
    """Return the largest factor of n that is smaller than n.

    >>> largest_factor(15)  # factors are 1, 3, 5
    5
    >>> largest_factor(80)  # 1, 2, 4, 5, 8, 10, 16, 20, 40
    40
    >>> largest_factor(13)  # prime
    1
    """
    "*** YOUR CODE HERE ***"
```

**答案**

```python
def largest_factor(n):
    x = n - 1
    while n % x != 0:
        x = x - 1
    return x
```

**验证**

```python
largest_factor(15)  # 5
largest_factor(80)  # 40
largest_factor(13)  # 1
```

**解题思路**

第一关是数学概念：x 是 n 的因数 ⟺ `n % x == 0`。

第二关是搜索方向：从大到小找，第一个命中的必然最大（因为要"小于 n 的最大因数"）。

循环方向最容易写反：

```python
while n % x != 0:   # ✓ x 不是因数就继续往下挪
while n % x == 0:   # ✗ x 是因数才继续 → n=15 时一次都不进,直接返回 14
```

trace（n=15）：

```
x=14 → 15%14=1 ≠0 继续
x=13 → 15%13=2 ≠0 继续
...
x=6  → 15%6=3  ≠0 继续
x=5  → 15%5=0  → 停,return 5 ✓
```

质数 n=13：一路试到 x=1，`13%1=0` 停，return 1 ✓

**考点**（对接 Lec03 Control）

- **while 循环的终止条件**：条件写的是"继续循环"的条件，不是"停止"的条件
- **循环不变式**：每轮 x 递减 1，始终在 [1, n-1] 范围内
- **% 取余判断整除**：`n % x == 0` 是整除判定的标准写法

---

### Q4: Hailstone

> Print the hailstone sequence starting at `n` and return its length.
>
> 规则：偶数 → ÷2，奇数 → ×3+1，到 1 停止。

```python
def hailstone(n):
    """Print the hailstone sequence starting at n and return its
    length.

    >>> a = hailstone(10)
    10
    5
    16
    8
    4
    2
    1
    >>> a
    7
    """
    "*** YOUR CODE HERE ***"
```

**答案**

```python
def hailstone(n):
    count = 0
    while n != 1:
        print(n)
        count = count + 1
        if n % 2 == 0:
            n = n // 2
        else:
            n = n * 3 + 1
    print(n)
    count = count + 1
    return count
```

**验证**

```python
hailstone(10)   # 打印 10 5 16 8 4 2 1,返回 7
hailstone(1)    # 打印 1,返回 1
```

**解题思路**

函数要做两件事：print（副作用）+ 返回长度（返回值）。所以既要 `print` 又要 `count`。

为什么 1 要单独处理：`while n != 1` 退出时 n 已经是 1，但 1 也是序列的一员，doctest 里确实打印了 1，也要计入长度。所以循环外补一次 `print(n)` 和 `count += 1`。

trace（n=10）：

```
count=0, n=10
print 10, count=1, n→5
print 5,  count=2, n→16
print 16, count=3, n→8
print 8,  count=4, n→4
print 4,  count=5, n→2
print 2,  count=6, n→1
循环退出 → print 1, count=7 → return 7 ✓
```

**考点**（对接 Lec01&02 + Lec03）

- **pure vs non-pure**：`hailstone` 有 print 副作用（non-pure），同时又有返回值
- **状态用变量管理**：n 和 count 都是循环变量，每轮更新——迭代式思维
- **`//` 整数除法**：`n // 2` 保证结果是 int，不会变 float
- **循环外补处理**：终止值也要计入，边界处理常见坑

---

## Check Your Score Locally

```bash
python ok --score
```

---

## 易错点汇总

| # | 错误 | 原因 | 修正 |
|---|---|---|---|
| 1 | `f = a - b` → `TypeError: 'int' object is not callable` | 把函数调用的结果绑给了 f，而不是函数本身 | `f = sub` 绑 function object |
| 2 | `while n % x == 0` 返回错误值 | 循环条件方向反了 | `while n % x != 0`（不是因数才继续） |
| 3 | `from ast import While` 误导入 | IDE 自动补全把 `while` 当成符号导入 | 删掉这行 |
| 4 | hailstone 少打印最后的 1 | 循环体只在 n≠1 时执行 | 循环外补 `print(n)` |
