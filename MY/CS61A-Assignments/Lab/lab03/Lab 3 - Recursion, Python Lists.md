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

| 题号 | 题名 | 类型 | 必做 |
|---|---|---|---|
| Q1 | WWPD: Lists & Ranges | WWPD | 是 |
| Q2 | Print If | 编程（for + 高阶） | 是 |
| Q3 | Close | 编程（range 索引） | 是 |
| Q4 | WWPD: List Comprehensions | WWPD | 是 |
| Q5 | Close List | 编程（列表推导式） | 是 |
| Q6 | Squares Only | 编程（列表推导式 + `math`） | 是 |
| Q7 | Double Eights | 编程（递归） | 是 |
| Q8 | Making Onions | 编程（递归 + 高阶） | 是 |

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

（做完再补）

**验证**

```bash
python3 ok -q print_if
```

（待跑）

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

（做完再补）

**验证**

```bash
python3 ok -q close
```

（待跑）

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

**答案**

（做完再补）

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

（做完再补）

**验证**

```bash
python3 ok -q close_list
```

（待跑）

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

（做完再补）

**验证**

```bash
python3 ok -q squares
```

（待跑）

### Recursion

#### Q7: Double Eights

**原题（English）**

Write a **recursive** function that takes in a positive integer `n` and determines if its digits contain two adjacent `8`s (that is, two `8`s right next to each other).u

**Hint:** Start by coming up with a recursive plan: the digits of a number have double eights if either ** (think of something that is straightforward to check) or double eights appear in the rest of the digits.

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

（做完再补）

**验证**

```bash
python3 ok -q double_eights
```

（待跑）

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

（做完再补）

**验证**

```bash
python3 ok -q make_onion
```

（待跑）

***

## Check Your Score Locally

You can locally check your score on each question of this assignment by running

```bash
python3 ok --score
```

**This does NOT submit the assignment!** When you are satisfied with your score, submit the assignment to Gradescope to receive credit for it.

> **中文**：`ok --score` 只是本地计分，不会提交。

（结果：待跑）

***

## 本次踩的坑

- （待补）

## 相关概念

- [[Discussion 3 - Recursion]] — 同一套递归套路，Q7 与 Disc 3 Q1 同源（都是逐位拆解）
- [[Homework 3 - Recursion, Tree Recursion]] — Q8 的二叉搜索与 HW 3 Q4 的树递归结构一致
- [[Lab 2 - Higher-Order Functions]] — Q2、Q8 都在用高阶函数
