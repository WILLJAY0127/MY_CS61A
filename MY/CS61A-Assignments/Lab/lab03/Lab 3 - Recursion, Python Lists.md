---
type: lab
id: "03"
title: Recursion, Python Lists
status: in_progress
source: SOURCE/CS61A-Website-Archive/lab/lab03/index.html
solution: SOURCE/CS61A-Website-Archive/lab/sol-lab03/index.html
code: MY/CS61A-Assignments/Lab/lab03/lab03.py
lectures: Lec08, Lec10
due: 2024-09-25
---

# Lab 3: Recursion, Python Lists

- **题目来源**：`SOURCE/CS61A-Website-Archive/lab/lab03/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/lab/sol-lab03/index.html`
- **Starter 基线**：`SOURCE/CS61A-Assignments_SOURCE/Lab/lab03/lab03.py`
- **我的代码**：`MY/CS61A-Assignments/Lab/lab03/lab03.py`
- **关联讲次**：Lec08 · Recursion、Lec10 · Sequences
- **Due**：2024-09-25（Wed 11:59pm）

***

## 总起

**一句话定位**：Lab 3 把「序列」和「递归」接起来——前半用列表、range、列表推导式做遍历，后半把遍历换成递归，最后一道 Q8 用递归搜索一棵「调用表达式」的树。

**覆盖知识点**

- 列表的索引（含负索引）、`len`、`in`、`+`、`*`、嵌套列表
- `for` 语句与 `range`（左闭右开）
- 列表推导式的两种形式：带 `if` 与不带 `if`
- 用递归遍历数位（Q7）与搜索组合空间（Q8）
- 高阶函数 + 递归：闭包里记住 `f`、`g`，递归里换着用

**题目地图**

| 题号 | 题名                        | 类型                 | 必做 |
| -- | ------------------------- | ------------------ | -- |
| Q1 | WWPD: Lists & Ranges      | WWPD               | 是  |
| Q2 | Print If                  | 编程（for + 高阶）       | 是  |
| Q3 | Close                     | 编程（range 索引）       | 是  |
| Q4 | WWPD: List Comprehensions | WWPD               | 是  |
| Q5 | Close List                | 编程（列表推导式）          | 是  |
| Q6 | Squares Only              | 编程（列表推导式 + `math`） | 是  |
| Q7 | Double Eights             | 编程（递归）             | 是  |
| Q8 | Making Onions             | 编程（递归 + 高阶）        | 是  |

**对应讲义**：Lec08 · Recursion，Lec10 · Sequences

***

## Starter Files

Download [lab03.zip](lab03.zip).

> **中文**：下载 `lab03.zip`，解压后得到 `lab03.py`、`ok`、`lab03.ok`、`tests/`。

***

## Topics

Consult this section if you need a refresher on the material for this lab. It's okay to skip directly to the questions and refer back here should you get stuck.

> **中文**：本节是复习材料，可以直接跳到题目，卡住再回来查。

### Lists

A list is a data structure that can hold an ordered collection of items. These items, known as elements, can be of any data type, including numbers, strings, or even other lists. A comma-separated list of expressions in square brackets creates a list:

```python
>>> list_of_values = [2, 1, 3, True, 3]
>>> nested_list = [2, [1, 3], [True, [3]]]
```

Each position in a list has an index, with the left-most element indexed `0`.

```python
>>> list_of_values[0]
2
>>> nested_list[1]
[1, 3]
```

A negative index counts from the end, with the right-most element indexed `-1`.

```python
>>> nested_list[-1]
[True, [3]]
```

Adding lists creates a longer list containing the elements of the added lists.

```python
>>> [1, 2] + [3] + [4, 5]
[1, 2, 3, 4, 5]
```

> **中文**：列表是有序容器，元素可以是任意类型（含列表）。索引从 0 开始，负索引从右往左数，`+` 是拼接。

### List Comprehensions

A list comprehension describes the elements in a list and evaluates to a new list containing those elements. There are two forms:

```python
[<expression> for <element> in <sequence>]
[<expression> for <element> in <sequence> if <conditional>]
```

Here's an example that starts with `[1, 2, 3, 4]`, picks out the even elements `2` and `4` using `if i % 2 == 0`, then squares each of these using `i*i`. The purpose of `for i` is to give a name to each element in `[1, 2, 3, 4]`.

```python
>>> [i*i for i in [1, 2, 3, 4] if i % 2 == 0]
[4, 16]
```

This list comprehension evaluates to a list of:

- The value of `i*i`
- For each element `i` in the sequence `[1, 2, 3, 4]`
- For which `i % 2 == 0`

In other words, this list comprehension will create a new list that contains the square of every even element of the original list `[1, 2, 3, 4]`.

We can also rewrite a list comprehension as an equivalent `for` statement, such as for the example above:

```python
>>> result = []
>>> for i in [1, 2, 3, 4]:
...     if i % 2 == 0:
...         result = result + [i*i]
>>> result
[4, 16]
```

> **中文**：列表推导式描述「新列表里放什么」，两种形式（带不带 `if`）。任何推导式都能改写成等价的 `for` 循环，反之亦然。

### For Statements

A `for` statement executes code for each element of a sequence, such as a list or range. Each time the code is executed, the name right after `for` is bound to a different element of the sequence.

```python
for <name> in <expression>:
    <suite>
```

First, `<expression>` is evaluated. It must evaluate to a sequence. Then, for each element in the sequence in order,

1. `<name>` is bound to the element.
2. `<suite>` is executed.

Here is an example:

```python
for x in [-1, 4, 2, 0, 5]:
    print("Current elem:", x)
```

This would display the following:

```text
Current elem: -1
Current elem: 4
Current elem: 2
Current elem: 0
Current elem: 5
```

> **中文**：`for` 逐个绑定序列元素再执行循环体。`for x in s` 拿到的是**元素**，不是索引。

### Ranges

A range is a data structure that holds integer sequences. A range can be created by:

- `range(stop)` contains `0, 1, ..., stop - 1`
- `range(start, stop)` contains `start, start + 1, ..., stop - 1`

Notice how the range function doesn't include the `stop` value; it generates numbers up to, but not including, the `stop` value.

For example:

```python
>>> for i in range(3):
...     print(i)
...
0
1
2
```

While ranges and lists are both sequences, a range object is different from a list. A range can be converted to a list by calling `list()`:

```python
>>> range(3, 6)
range(3, 6)  # this is a range object
>>> list(range(3, 6))
[3, 4, 5]  # list() converts the range object to a list
>>> list(range(5))
[0, 1, 2, 3, 4]
>>> list(range(1, 6))
[1, 2, 3, 4, 5]
```

> **中文**：`range` 左闭右开，`range(3)` 是 0/1/2。`range` 对象不是列表，直接显示是 `range(3, 6)`，要 `list()` 才变成列表。

***

## Required Questions

### Lists

> **Important:** For all WWPD questions, type `Function` if you believe the answer is `<function...>`, `Error` if it errors, and `Nothing` if nothing is displayed.

> **中文**：所有 WWPD 题，`<function...>` 一律填 `Function`，报错填 `Error`，什么都不显示填 `Nothing`。

#### Q1: WWPD: Lists & Ranges

**原题（English）**

Use Ok to test your knowledge with the following "What Would Python Display?" questions:

```bash
python3 ok -q lists-wwpd -u
```

Predict what Python will display when you type the following into the interactive interpreter. Then try it to check your answers.

> **中文**：先预测解释器会显示什么，再实际跑一遍对答案。

```python
>>> s = [7//3, 5, [4, 0, 1], 2]
>>> s[0]
______
>>> s[2]
______
>>> s[-1]
______
>>> len(s)
______
>>> 4 in s
______
>>> 4 in s[2]
______
>>> s[2] + [3 + 2]
______
>>> 5 in s[2]
______
>>> s[2] * 2
______
>>> list(range(3, 6))
______
>>> range(3, 6)
______
>>> r = range(3, 6)
>>> [r[0], r[2]]
______
>>> range(4)[-1]
______
```

**答案**（unlock 全部通过）

```python
>>> s = [7//3, 5, [4, 0, 1], 2]
>>> s[0]
2
>>> s[2]
[4, 0, 1]
>>> s[-1]
2
>>> len(s)
4
>>> 4 in s
False
>>> 4 in s[2]
True
>>> s[2] + [3 + 2]
[4, 0, 1, 5]
>>> 5 in s[2]
False
>>> s[2] * 2
[4, 0, 1, 4, 0, 1]
>>> list(range(3, 6))
[3, 4, 5]
>>> range(3, 6)
range(3, 6)
>>> r = range(3, 6)
>>> [r[0], r[2]]
[3, 5]
>>> range(4)[-1]
3
```

**说明**

- 14 个空，首遍错 2 个概念点（见易错点）；`s[0]` 有一次空提交是误触，不算
- `range(3, 6)` 单独求值显示的是 **range 对象本身**（repr 就是 `range(3, 6)`），不是列表。它能索引（`r[0]`、`r[2]`、`range(4)[-1]` 都合法），但看不到元素，要 `list(r)` 才展开

**易错点**

- `5 in s[2]` 答成 `True`：`s[2]` 是 `[4, 0, 1]`，5 不在里面。做 `in` 判断先确认**容器是谁**（`s` 还是 `s[2]`），上一题的结果会带节奏
- `range(3, 6)` 卡了三次：`3`（以为显示首元素）→ `range(3,6)`（少个空格）→ `range(3, 6)` 才过。ok 对列表答案宽松（`[4,0,1,5]` 无空格也放行），对 range 卡格式——**按 Python 标准显示格式（带空格）写最保险**

#### Q2: Print If

**原题（English）**

Implement `print_if`, which takes a list `s` and a one-argument function `f`. It prints each element `x` of `s` for which `f(x)` returns a true value.

> **中文**：实现 `print_if(s, f)`，把 `s` 里所有满足 `f(x)` 为真的元素打印出来。返回 `None`。

**Starter**（原样摘录）

```python
def print_if(s, f):
    """Print each element of s for which f returns a true value.

    >>> print_if([3, 4, 5, 6], lambda x: x > 4)
    5
    6
    >>> result = print_if([3, 4, 5, 6], lambda x: x % 2 == 0)
    4
    6
    >>> print(result)  # print_if should return None
    None
    """
    for x in s:
        "*** YOUR CODE HERE ***"
```

**答案**

```python
def print_if(s, f):
    for x in s:
        if f(x):
            print(x)
```

**验证**

```bash
python3 ok -q print_if
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

`for x in s` 拿到的已经是**元素本身**，不是索引。所以这一题不需要 `range(len(s))`，直接对 `x` 调用 `f` 就行：

1. `f(x)` 为真 → 打印 `x`
2. `f(x)` 为假 → 什么都不做，继续下一个

关键是**不写 `return`**。函数的职责只有「打印」，打印完就该让循环继续下一个元素。一旦在循环里 `return`，第一个满足条件的元素就会终止整个函数。

**关联知识点**

- `for x in s` 遍历元素 vs `for i in range(len(s))` 遍历索引——本题用前者，Q3 用后者
- 高阶函数：`f` 是参数而不是写死的判断，同一个 `print_if` 能配任意筛选条件（对接 Lec04 / Lab 2）

**易错点**

- 循环体里写 `return` 或 `return x` → doctest 第三段期望 `print(result)` 是 `None`，会失败
- 写成 `for i in range(len(s)): if f(i)` → 判断的是下标而不是元素
- 漏掉 `if` 直接 `print(x)` → 全部打印，失去了筛选的意义

#### Q3: Close

**原题（English）**

Implement `close`, which takes a list of numbers `s` and a non-negative integer `k`. It returns how many of the elements of `s` are within `k` of their index. That is, the absolute value of the difference between the element and its index is less than or equal to `k`.

> Remember that list is "zero-indexed"; the index of the first element is `0`.

> **中文**：`close(s, k)` 返回 `s` 中「元素值与它的下标相差不超过 k」的元素个数。注意下标从 0 开始。

**Starter**（原样摘录）

```python
def close(s, k):
    """Return how many elements of s that are within k of their index.

    >>> t = [6, 2, 4, 3, 5]
    >>> close(t, 0)  # Only 3 is equal to its index
    1
    >>> close(t, 1)  # 2, 3, and 5 are within 1 of their index
    3
    >>> close(t, 2)  # 2, 3, 4, and 5 are all within 2 of their index
    4
    >>> close(list(range(10)), 0)
    10
    """
    count = 0
    for i in range(len(s)):  # Use a range to loop over indices
        "*** YOUR CODE HERE ***"
    return count
```

**答案**

```python
def close(s, k):
    count = 0
    for i in range(len(s)):  # Use a range to loop over indices
        if abs(s[i] - i) <= k:
            count += 1
    return count
```

**验证**

```bash
python3 ok -q close
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

这题必须走**索引**，因为要比较「元素值」和「它的下标」两个东西，而元素本身不知道自己的位置。所以循环用 `range(len(s))` 而不是 `for x in s`：

1. `s[i]` 取元素值
2. `i` 是它的下标
3. `abs(s[i] - i)` 是「值与下标相差多少」，去掉正负号
4. `<= k` 就计入一次

以 `t = [6, 2, 4, 3, 5]`、`k = 1` 为例：

| i | s[i] | s[i] - i | abs | ≤ 1 ? |
|---|---|---|---|---|
| 0 | 6 | 6 | 6 | ✗ |
| 1 | 2 | 1 | 1 | ✓ |
| 2 | 4 | 2 | 2 | ✗ |
| 3 | 3 | 0 | 0 | ✓ |
| 4 | 5 | 1 | 1 | ✓ |

3 个，与 doctest 的 `3` 一致。

**关联知识点**

- `for x in s`（拿元素）vs `for i in range(len(s))`（拿下标）——需要位置信息时必须用后者
- `abs()` 是一个内建函数，返回绝对值；`abs(x)` 只接受一个参数
- 计数型循环的骨架：初始化 → 遍历 → 条件判断 → 累加 → 返回
- 与 Q5 是同一判定、不同产出：Q3 数个数，Q5 收集成列表

**易错点**

- 不写 `abs`，只写 `s[i] - i <= k` → 负数会全部被误判为「在上界内」，`close(t, 1)` 会多算
- 用 `for x in s` 后试图用 `s.index(x)` 找下标 → 重复元素会返回第一个匹配位置，`t` 里有两个索引不同的相同值时会算错
- 在循环里 `return count` → 第一次循环就返回，只算了第一个元素
- `count` 忘记 `+= 1` 而写成 `count += s[i]` → 数字变求和

### List Comprehensions

> **Important:** For all WWPD questions, type `Function` if you believe the answer is `<function...>`, `Error` if it errors, and `Nothing` if nothing is displayed.

#### Q4: WWPD: List Comprehensions

**原题（English）**

Use Ok to test your knowledge with the following "What Would Python Display?" questions:

```bash
python3 ok -q list-comprehensions-wwpd -u
```

Predict what Python will display when you type the following into the interactive interpreter. Then try it to check your answers.

> **中文**：预测列表推导式的输出。

```python
>>> [2 * x for x in range(4)]
______
>>> [y for y in [6, 1, 6, 1] if y > 2]
______
>>> [[1] + s for s in [[4], [5, 6]]]
______
>>> [z + 1 for z in range(10) if z % 3 == 0]
______
```

**答案**（unlock 全部通过）

```python
>>> [2 * x for x in range(4)]
[0, 2, 4, 6]
>>> [y for y in [6, 1, 6, 1] if y > 2]
[6, 6]
>>> [[1] + s for s in [[4], [5, 6]]]
[[1, 4], [1, 5, 6]]
>>> [z + 1 for z in range(10) if z % 3 == 0]
[1, 4, 7, 10]
```

**说明**

- 4 个空，首遍对 2 个、错 2 个
- 第 3 题：`s` 依次绑定 `[4]`、`[5, 6]`，`[1] + s` 是**列表拼接**，所以是 `[1, 4]` 和 `[1, 5, 6]`，不是数值加法
- 第 4 题：`range(10)` 里满足 `z % 3 == 0` 的是 `0, 3, 6, 9`，各加 1 得 `1, 4, 7, 10`

**易错点**

- 第 3 题多写一个 `]`：两个元素各闭合一次、外层再闭合一次，末尾只有两个 `]`。括号配对建议对着「外层 + 每个元素」数
- 第 4 题漏掉 `z = 0` → 答 `[4, 7, 10]`，少了 `1`。**`0 % 3 == 0` 成立**，0 是 `range(10)` 的第一个元素，不能默认从 3 开始
- 把 `[1] + s` 当成数值加法 → 会以为第 3 题输出 `[5, 6]` 之类的数字
- 带 `if` 的推导式，`if` 过滤的是**元素**（`y`、`z`），`for` 前面的表达式是在**过滤之后**才求值的，顺序别弄反

#### Q5: Close List

**原题（English）**

Implement `close_list`, which takes a list of numbers `s` and a non-negative integer `k`. It returns a list of the elements of `s` that are within `k` of their index. That is, the absolute value of the difference between the element and its index is less than or equal to `k`.

> **中文**：与 Q3 同一个判定，但这次要**返回列表**，用列表推导式写。

**Starter**（原样摘录）

```python
def close_list(s, k):
    """Return a list of the elements of s that are within k of their index.

    >>> t = [6, 2, 4, 3, 5]
    >>> close_list(t, 0)  # Only 3 is equal to its index
    [3]
    >>> close_list(t, 1)  # 2, 3, and 5 are within 1 of their index
    [2, 3, 5]
    >>> close_list(t, 2)  # 2, 3, 4, and 5 are all within 2 of their index
    [2, 4, 3, 5]
    """
    return [___ for i in range(len(s)) if ___]
```

**答案**

```python
def close_list(s, k):
    return [s[i] for i in range(len(s)) if abs(s[i] - i) <= k]
```

**验证**

```bash
python3 ok -q close_list
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

这就是 Q3 的列表推导式版本。把 Q3 的 for 循环逐块对到推导式上：

| Q3 的 for 循环 | Q5 的推导式 |
|---|---|
| `for i in range(len(s))` | `for i in range(len(s))` |
| `if abs(s[i] - i) <= k:` | `if abs(s[i] - i) <= k` |
| `count += 1` | 换成把 `s[i]` 放进新列表 |
| `return count` | `[...]` 本身就是返回值，不用 return 变量 |

关键差异：**Q3 累加的是“次数”，Q5 收集的是“元素本身”**。所以 `for` 前面必须是 `s[i]`，不能是 `i`（否则会得到下标列表）也不能是 `1`（否则会得到一堆 1）。

以 `t = [6, 2, 4, 3, 5]`、`k = 2` 为例，通过筛选的是 `i = 1, 2, 3, 4`，对应的 `s[i]` 依次是 `2, 4, 3, 5`，所以结果是 `[2, 4, 3, 5]`。注意这些值原样保留，没有排序，与 `[2, 3, 4, 5]` 不同。

**关联知识点**

- 列表推导式三要素：`[表达式 for 变量 in 序列 if 条件]`
- 同一套逻辑的两种产出：累计计数（Q3）vs 收集列表（Q5）。想清楚“我要的是数量还是元素”
- `for` 前面的表达式作用在**过滤之后**的元素上，顺序是“先筛、再变换”

**易错点**

- `for` 前面写 `i` 而不是 `s[i]` → 得到 `[1, 2, 3, 4]` 这种下标列表
- 忘了 `range(len(s))` 而写 `for i in s` → `i` 变成元素值，`s[i]` 会越界或取错
- 以为结果会排序 → doctest 里 `[2, 4, 3, 5]` 就是原顺序，不是 `[2, 3, 4, 5]`
- 在推导式里又想写 `count += 1` → 推导式不能执行语句，只能求值

#### Q6: Squares Only

**原题（English）**

Implement the function `squares`, which takes in a list of positive integers. It returns a list that contains the square roots of the elements of the original list that are perfect squares. Use a list comprehension.

> To find if `x` is a perfect square, you can check if `sqrt(x)` equals `round(sqrt(x))`.

> **中文**：`squares(s)` 返回列表，包含 `s` 中所有完全平方数的平方根，必须用列表推导式。判断方法：`sqrt(x) == round(sqrt(x))`。

**Starter**（原样摘录）

```python
from math import sqrt

def squares(s):
    """Returns a new list containing square roots of the elements of the
    original list that are perfect squares.

    >>> seq = [8, 49, 8, 9, 2, 1, 100, 102]
    >>> squares(seq)
    [7, 3, 1, 10]
    >>> seq = [500, 30]
    >>> squares(seq)
    []
    """
    return [___ for n in s if ___]
```

**答案**

```python
from math import sqrt

def squares(s):
    return [round(sqrt(n)) for n in s if sqrt(n) == round(sqrt(n))]
```

**验证**

```bash
python3 ok -q squares
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

和 Q5 同一个骨架，只是换成「判断完全平方数 + 取平方根」。这题 `round()` 出现了两次，干的是两件不同的事：

| 位置 | 作用 | 说明 |
|---|---|---|
| `if sqrt(n) == round(sqrt(n))` | **判断**是否完全平方 | 完全平方数的 `sqrt` 是整数，取整后不变 |
| `for` 前面的 `round(sqrt(n))` | **转类型** | `sqrt` 返回浮点，`round` 收成整数 |

为什么第二个 `round` 不能省：

```python
>>> sqrt(49)
7.0
>>> round(sqrt(49))
7
```

doctest 期望 `[7, 3, 1, 10]`，如果只写 `sqrt(n)` 会得到 `[7.0, 3.0, 1.0, 10.0]`，浮点与整数不相等，测试失败。

以 `seq = [8, 49, 8, 9, 2, 1, 100, 102]` 为例：

| n | sqrt(n) | 是整数？ | 输出 |
|---|---|---|---|
| 8 | ≈2.83 | ✗ | 丢弃 |
| 49 | 7.0 | ✓ | 7 |
| 8 | ≈2.83 | ✗ | 丢弃 |
| 9 | 3.0 | ✓ | 3 |
| 2 | ≈1.41 | ✗ | 丢弃 |
| 1 | 1.0 | ✓ | 1 |
| 100 | 10.0 | ✓ | 10 |
| 102 | ≈10.10 | ✗ | 丢弃 |

**关联知识点**

- `from math import sqrt`：`sqrt` 来自标准库 `math`，不是内建函数
- 浮点与整数的区别：`7.0 == 7` 为 `True`，但 `repr` 显示不同，doctest 按显示比对
- `round(x)` 返回整数（对浮点输入），既是「四舍五入」也是「浮点转整数」的手段
- 列表推导式里，`if` 分支和表达式分支**可以调用同一个函数两次**，这题 `sqrt` 实际被算了两次

**易错点**

- 只写 `sqrt(n)` 不写 `round` → 输出 `[7.0, 3.0, 1.0, 10.0]`，与 doctest 不匹配
- 判断写成 `sqrt(n) == int(sqrt(n))` → 能跑，但 `int` 对负数/浮点误差的处理不如 `round`；题目明确建议用 `round`
- 忘了 `from math import sqrt` → `NameError`
- 把 `round` 只放在 `if` 里、表达式里忘了 → 同样输出浮点，测试失败

### Recursion

#### Q7: Double Eights

**原题（English）**

Write a **recursive** function that takes in a positive integer `n` and determines if its digits contain two adjacent `8`s (that is, two `8`s right next to each other).u

**Hint:** Start by coming up with a recursive plan: the digits of a number have double eights if either \*\* (think of something that is straightforward to check) or double eights appear in the rest of the digits.

**Important:** Use recursion; the tests will fail if you use any loops (for, while).

> **中文**：递归判断 `n` 的数位里有没有相邻的两个 8。Hint 的句式是「要么【某件一眼能查的事】，要么剩下那些数位里有双 8」。禁用所有循环。
>
> 注意两处官方笔误（保留原样）：原题句尾多一个 `u`；Hint 里加粗的部分**本来就是空的**——那个「一眼能查的事」要你自己想。又一个 Disc 3 Q3 式的手写 hint。

**Starter**（原样摘录）

```python
def double_eights(n):
    """Returns whether or not n has two digits in row that
    are the number 8.

    >>> double_eights(1288)
    True
    >>> double_eights(880)
    True
    >>> double_eights(538835)
    True
    >>> double_eights(284682)
    False
    >>> double_eights(588138)
    True
    >>> double_eights(78)
    False
    >>> # ban iteration
    >>> from construct_check import check
    >>> check(LAB_SOURCE_FILE, 'double_eights', ['While', 'For'])
    True
    """
    "*** YOUR CODE HERE ***"
```

**答案**

```python
def double_eights(n):
    if n < 10:
        return False
    right = n % 10
    next_right = n // 10 % 10
    if right == next_right == 8:
        return True
    else:
        return double_eights(n // 10)
```

**验证**

```bash
python3 ok -q double_eights
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

每层只看**末尾两位**，把其余部分交给 `n // 10`：

1. `n % 10` —— 末位
2. `n // 10 % 10` —— 倒数第二位
3. 两个都是 8 → 本层发现双 8，返回 `True`
4. 否则 → 相信 `double_eights(n // 10)` 能查完剩下的位

以 `588138` 为例：

| 调用 | 末两位 | 结果 |
|---|---|---|
| `588138` | 8, 3 | 不是，继续 |
| `58813` | 3, 1 | 不是，继续 |
| `5881` | 1, 8 | 不是，继续 |
| `5881 → 588` | 8, 8 | **双 8，返回 True** |

**为什么 base case 是 `n < 10` 而不是 `n == 0`**

这题要判断的是“**有没有一对相邻的数字**”。一位数连一对都没有，所以边界应该停在「不足两位」，即 `n < 10`。

对比 HW3 的 Q1 `num_eights`：那里判断的是“**每一位是不是 8**”，每一位都算数，所以边界要退到「一位都没有了」，即 `n == 0`。

**同一个 `n == 0`，在不同题里含义不同**，关键看它代表的是：

| 边界含义 | 条件 | 对应题目 |
|---|---|---|
| 数据用完了（不再有元素可处理） | `n == 0` | HW3 Q1 `num_eights` |
| 没有可比较的对象了（凑不出一对） | `n < 10` | Lab3 Q7 `double_eights` |

写递归前先问一句：「这题的最小有效单位是什么？」是一**位**还是**一对**？答案决定边界停在哪里。

顺带：这题写 `n == 0` 也不会错，因为它多走一层时 `next_right` 会变成 `0`，而 `(某位, 0)` 永远不等于 `(8, 8)`。但那是一次白跑的调用，属于“碰巧安全”。

**关联知识点**

- 数字逐位拆解：`n % 10` 取末位、`n // 10` 丢掉末位
- 链式比较：`right == next_right == 8` 等价于 `right == next_right and next_right == 8`
- `construct_check` 禁 `While`、`For`，但**没有禁 `Assign`**，所以 `right = ...` 这类赋值可以用（对比 HW3 Q1 连赋值都禁）
- 本节 Q7、Q8 与 Q1–Q6 无关：这份 Lab 前半是列表（遍历 + 推导式），后半是递归，只是被官方放在同一份作业里

**易错点**

- **`else` 分支漏写 `return`**：写成 `double_eights(n // 10)` 只是调用、不返回，函数会返回 `None`，所有“不含双 8”的用例全部失败。这是本题最容易踩的坑
- base case 用 `n == 0` 而不加思考 → 能跑但多一层无用调用；真正该问的是“最小单位是一位还是一对”
- 只检查 `n % 10 == 8` → 只能发现单个 8，发现不了相邻
- 忘了 `n // 10 % 10` 中 `% 10` 的优先级 —— Python 里 `//` 和 `%` 同级、从左到右，所以 `n // 10 % 10` 就是 `(n // 10) % 10`，正确；但写成 `n // (10 % 10)` 会除零
- 用 `str(n)` 转字符串判断 → 虽然能过测试，但违背题目“递归拆数字”的意图

#### Q8: Making Onions

**原题（English）**

Write a function `make_onion` that takes in two one-argument functions, `f` and `g`. It returns a function that takes in three arguments: `x`, `y`, and `limit`. The returned function returns `True` if it is possible to reach `y` from `x` using up to `limit` calls to `f` and `g`, and `False` otherwise.

For example, if `f` adds 1 and `g` doubles, then it is possible to reach 25 from 5 in four calls: `f(g(g(f(5))))`.

> **中文**：`make_onion(f, g)` 返回 `can_reach(x, y, limit)`：从 `x` 出发，最多套用 `limit` 次 `f` 或 `g`，能不能得到 `y`。每次只能选 `f` 或 `g`，所以是一棵二叉搜索树（这就是标题里 "onion" 的层层包裹）。

**Starter**（原样摘录）

```python
def make_onion(f, g):
    """Return a function can_reach(x, y, limit) that returns
    whether some call expression containing only f, g, and x with
    up to limit calls will give the result y.

    >>> up = lambda x: x + 1
    >>> double = lambda y: y * 2
    >>> can_reach = make_onion(up, double)
    >>> can_reach(5, 25, 4)      # 25 = up(double(double(up(5))))
    True
    >>> can_reach(5, 25, 3)      # Not possible
    False
    >>> can_reach(1, 1, 0)      # 1 = 1
    True
    >>> add_ing = lambda x: x + "ing"
    >>> add_end = lambda y: y + "end"
    >>> can_reach_string = make_onion(add_ing, add_end)
    >>> can_reach_string("cry", "crying", 1)      # "crying" = add_ing("cry")
    True
    >>> can_reach_string("un", "unending", 3)     # "unending" = add_ing(add_end("un"))
    True
    >>> can_reach_string("peach", "folding", 4)   # Not possible
    False
    """
    def can_reach(x, y, limit):
        if limit < 0:
            return ____
        elif x == y:
            return ____
        else:
            return can_reach(____, ____, limit - 1) or can_reach(____, ____, limit - 1)
    return can_reach
```

**答案**

```python
def make_onion(f, g):
    def can_reach(x, y, limit):
        if limit < 0:
            return False
        elif x == y:
            return True
        else:
            return can_reach(f(x), y, limit - 1) or can_reach(g(x), y, limit - 1)
    return can_reach
```

**验证**

```bash
python3 ok -q make_onion
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

`can_reach(x, y, limit)` 问的是：从 `x` 出发，只用 `f` 和 `g`，**最多调用 `limit` 次**，能否得到 `y`。

三个参数的分工：

| 参数 | 含义 | 递归时 |
|---|---|---|
| `x` | 现在在哪（当前值） | **变** |
| `y` | 想去哪（目标值） | **不变** |
| `limit` | 还剩几次调用额度 | **变**（每次减 1） |

三个分支：

1. `limit < 0` → 额度用光了还没到，返回 `False`
2. `x == y` → 已经站在目标上，返回 `True`（一步都不用走也算）
3. 否则 → 下一步要么用 `f`，要么用 `g`，两条路用 `or` 连起来

以 `can_reach(5, 25, 4)` 为例，答案路径是 `25 = f(g(g(f(5))))`：

```text
5 → f → 6 → g → 12 → g → 24 → f → 25      共 4 次调用
```

树的前两层：

```text
can_reach(5, 25, 4)
├─ 用 f: can_reach(6, 25, 3)
│         └─ 用 g: can_reach(12, 25, 2)
│                   └─ …最终走到 25
└─ 用 g: can_reach(10, 25, 3)
          └─ …这条路走不通
```

`limit = 3` 时不够用，返回 `False`。

**和 Q4 `count_dollars` 同构**

| | `count_dollars` | `make_onion` |
|---|---|---|
| 两条分支 | 用一张 bill / 不用 | 下一步用 `f` / 用 `g` |
| 聚合符 | `+`（数方案） | `or`（找一条路） |
| 目标参数 | 不变 | `y` 不变 |
| 预算参数 | `total` 减少 | `limit` 减少 |

树递归的**聚合符可以是 `+`（计数）、`or`（存在性）、`max`（最优）**，骨架完全一样，只有最后组合那一步不同。

**为什么这里不能做算术**

这题同时支持数字和字符串（`add_ing` / `add_end`）。所以递归里**不能出现任何对 `x`、`y` 的加减乘除**——`"crying" - "cryinging"` 会直接 `TypeError`。只能做三件事：相等比较、调用 `f`/`g`、`limit - 1`。

**易错点**

- **把 `y` 也改了**（写成 `y - f(x)`）→ 判据是「两个值相等」，传参却用「差额」，两套模型混用；且字符串版本直接 `TypeError`
- `limit` 忘减 → 无限递归
- 两个分支写成 `and` → 要求两条路同时走通，几乎永远是 `False`
- base case 顺序写反（先判 `x == y` 再判 `limit < 0`）→ `can_reach(1, 1, 0)` 虽然仍对，但语义上「没额度了」应该优先于「恰好到达」
- 第二个分支写成 `f(x)` 而不是 `g(x)` → 两条路一样，等于只有 `f` 可用

***

## Check Your Score Locally

You can locally check your score on each question of this assignment by running

```bash
python3 ok --score
```

**This does NOT submit the assignment!** When you are satisfied with your score, submit the assignment to Gradescope to receive credit for it.

> **中文**：`ok --score` 只是本地计分，不会提交。

（结果）

```text
Point breakdown
    print_if:       1.0/1
    close:          1.0/1
    close_list:     1.0/1
    squares:        1.0/1
    double_eights:  1.0/1
    make_onion:     1.0/1

Score:
    Total: 6.0
```

六道编程题全部满分。Q1、Q4 两道 WWPD 由 `ok -u` 单独解锁（不计入 `--score`）。

***

## 本次踩的坑

- **Q4 WWPD 漏掉 `z = 0`**：`range(10)` 里 `0 % 3 == 0` 成立，`0 + 1 = 1` 是第一个输出，容易默认从 3 开始算
- **Q4 WWPD 括号数错**：`[[1, 4], [1, 5, 6]]` 末尾只有两个 `]`，多写一个就错
- **Q7 `else` 分支漏 `return`**：只调用不返回，函数返回 `None`，所有“不含双 8”的用例全崩。**递归调用必须被 `return` 或参与运算，否则结果被丢弃**
- **Q8 把 `y` 也改了**：`y - f(x)` 混用了「相等判据」和「差额传参」两套模型；且这题支持字符串，减号会直接 `TypeError`
- **base case 的两种含义**（Q7 vs HW3 Q1）：`n == 0` 表示「数据用完」，`n < 10` 表示「没有可比较的对象」。先问「最小有效单位是一位还是一对」
- 通用项：递归调用前必须确认「参数确实变小了」+「本分支要有返回值」，这两条能拦住这题 90% 的错

## 相关概念

- \[\[Discussion 3 - Recursion]] — 同一套递归套路，Q7 与 Disc 3 Q1 同源（都是逐位拆解）
- \[\[Homework 3 - Recursion, Tree Recursion]] — Q8 的二叉搜索与 HW 3 Q4 的树递归结构一致
- \[\[Lab 2 - Higher-Order Functions]] — Q2、Q8 都在用高阶函数
