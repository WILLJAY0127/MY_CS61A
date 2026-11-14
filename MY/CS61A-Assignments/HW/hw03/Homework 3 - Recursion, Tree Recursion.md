---
type: hw
id: "03"
title: Recursion, Tree Recursion
status: in_progress
source: SOURCE/CS61A-Website-Archive/hw/hw03/index.html
solution: SOURCE/CS61A-Website-Archive/hw/sol-hw03/index.html
code: MY/CS61A-Assignments/HW/hw03/hw03.py
lectures: Lec08, Lec09
due: 2024-09-26
---

# Homework 3: Recursion, Tree Recursion

- **题目来源**：`SOURCE/CS61A-Website-Archive/hw/hw03/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/hw/sol-hw03/index.html`
- **Starter 基线**：`SOURCE/CS61A-Website-Archive/hw/hw03/hw03.zip`
- **我的代码**：`MY/CS61A-Assignments/HW/hw03/hw03.py`
- **关联讲次**：Lec08 · Recursion / Lec09 · Tree Recursion，教材 Section 1.7
- **Due**：2024-09-26 23:59

---

## 总起

**一句话定位**：HW03 是递归从「线性」走到「树形」的一整套练习——前两题只沿一条链递归（拆数位），Q3 用辅助函数控制奇偶交替，Q4 进入树递归（同时做「用这张钞票」和「不用这张钞票」两个选择），Q5 把同一棵树从反方向走一遍。

**覆盖知识点**

- 线性递归：每次只递归一次，结果沿一条链累加（Q1、Q2）
- 辅助函数：用一个「你确定知道性质」的起点（1 是奇数）代替 `%` 判断奇偶（Q3）
- 树递归：同一层展开两个递归调用，返回两者之和（Q4、Q5）
- 计数型树递归：`count_partitions` 的变体，用「当前面额」当作第二维状态
- 递归中打印：不收集结果，靠先序/后序决定输出顺序（Q6）
- 匿名递归：不用 `def`、不用赋值，用 Y 组合子的思路写阶乘（Q7）

**题目地图**

| 题号 | 题名 | 类型 | 必做 |
|---|---|---|---|
| Q1 | Num Eights | 线性递归（数位拆解） | 是 |
| Q2 | Digit Distance | 线性递归（相邻数位） | 是 |
| Q3 | Interleaved Sum | 递归 + 高阶函数（禁 `%`） | 是 |
| Q4 | Count Dollars | 树递归（计数） | 是 |
| Q5 | Count Dollars Upward | 树递归（反方向） | 否 |
| Q6 | Towers of Hanoi | 树递归 + 打印 | 否（Just For Fun） |
| Q7 | Anonymous Factorial | 匿名递归（lambda） | 否（Just For Fun） |

**对应讲义**：Lec08 · Recursion / Lec09 · Tree Recursion，教材 Section 1.7

---

## Instructions

Download `hw03.zip`. Inside the archive, you will find a file called `hw03.py`, along with a copy of the `ok` autograder.

**Submission:** When you are done, submit the assignment by uploading all code files you've edited to Gradescope. You may submit more than once before the deadline; only the final submission will be scored. Check that you have successfully submitted your code on Gradescope. See Lab 0 for more instructions on submitting assignments.

**Using Ok:** If you have any questions about using Ok, please refer to this guide.

**Readings:** You might find the following references useful: Section 1.7

**Grading:** Homework is graded based on correctness. Each incorrect problem will decrease the total score by one point. **This homework is out of 2 points.**

> **中文**：下载 `hw03.zip`（含 `hw03.py` 和 ok 评分器）；完成后在 Gradescope 上传改过的文件，截止前可多次提交但只算最后一次；参考教材 Section 1.7；作业满分 2 分，每题错扣 1 分。

`hw03.py` 顶部有 `HW_SOURCE_FILE = __file__`，是 `construct_check` 判断「有没有用循环/赋值」时要读的路径，不要删。

---

## Required Questions

### Q1: Num Eights

**原题（English）**

Write a recursive function `num_eights` that takes a positive integer `n` and returns the number of times the digit 8 appears in `n`.

**Important:** Use recursion; the tests will fail if you use any assignment statements or loops. (You can define new functions, but don't put assignment statements there either.)

> **中文**：写递归函数 `num_eights(n)`，返回 n 的十进制表示里数字 8 出现了几次。禁止循环和任何赋值语句（新定义的函数里也不能有赋值）。

**Starter**（原样摘录）

```python
def num_eights(n):
    """Returns the number of times 8 appears as a digit of n.

    >>> num_eights(3)
    0
    >>> num_eights(8)
    1
    >>> num_eights(88888888)
    8
    >>> num_eights(2638)
    1
    >>> num_eights(86380)
    2
    >>> num_eights(12345)
    0
    >>> num_eights(8782089)
    3
    >>> from construct_check import check
    >>> # ban all assignment statements
    >>> check(HW_SOURCE_FILE, 'num_eights',
    ...       ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr', 'For', 'While'])
    True
    """
    "*** YOUR CODE HERE ***"
```

> 测试里 `check(...)` 会扫描源码，出现 `Assign` / `AugAssign` / `For` / `While` 直接判 `False`——连 `count = 0` 这种初始化都不行。

**答案**

```python
def num_eights(n):
    if n == 0:
        return 0
    else:
        return (1 if n % 10 == 8 else 0) + num_eights(n // 10)
```

**验证**

```bash
python3 ok -q num_eights
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

递归沿「去掉最后一位」的方向走：`n → n // 10`。每层只做两件事——看末位 `n % 10` 是不是 8，再把「剩下那几位」的结果加回来。

`num_eights(2638)` 的展开：

```text
num_eights(2638) = 1 + num_eights(263)   # 2638 % 10 == 8 → 1
num_eights(263)  = 0 + num_eights(26)    # 263 % 10 == 3 → 0
num_eights(26)   = 0 + num_eights(2)     # 26 % 10 == 6 → 0
num_eights(2)    = 0 + num_eights(0)     # 2 % 10 == 2 → 0
num_eights(0)    = 0                      # base case
```

加起来是 1，和 doctest 一致。

**base case 为什么是 `n == 0` 而不是 `n < 10`**：如果停在最后一位，就得在 base case 里再判一次「它是不是 8」，和递归分支里的判断重复。退到「一位都没有了」，判断就只在 `(1 if n % 10 == 8 else 0)` 这一处出现。代价是多一层 `num_eights(0)`，但它立刻返回 0，不做任何事。

**关联知识点**

- 数位拆解的两件套：`n % 10` 取末位、`n // 10` 去掉末位（与 Disc 3 Q1 `swipe` 同源）
- 计数型递归的合并方式是**加法**，base case 返回加法的单位元 0
- **禁止赋值**意味着没有累加变量，「计数」只能靠返回值一层层加起来
- base case 的两种等价设计：停在最后一位（返回该位的贡献）vs 越过所有位（返回单位元）；后者能让逻辑只写一遍

**易错点**

- 用 `count = 0` / `count += 1` → `construct_check` 扫到 `Assign` / `AugAssign` 直接判 `False`，不是运行报错，是测试文件返回 `False`
- base case 停 `n < 10` 又在里面判一次 8 → 逻辑写两遍，`88888888` 这种全 8 的数最容易看出重复
- base case 返回 1 而不是 0 → 每个数结果都多 1，`num_eights(3)` 得 1
- `n // 10` 写成 `n % 10` → 参数不变，无限递归
- 用 `str(n)` 或转成 list 数 8 → 题目要求「递归拆数位」，`construct_check` 不拦但违背题意

### Q2: Digit Distance

**原题（English）**

For a given integer, the *digit distance* is the sum of the absolute differences between consecutive digits. For example:

- The digit distance of `61` is `5`, as the absolute value of `6 - 1` is `5`.
- The digit distance of `71253` is `12` (`abs(7-1) + abs(1-2) + abs(2-5) + abs(5-3)` = `6 + 1 + 3 + 2`).
- The digit distance of `6` is `0` because there are no pairs of consecutive digits.

Write a function that determines the digit distance of a positive integer. You must use recursion or the tests will fail.

> **中文**：digit distance = 相邻两位数字之差的绝对值求和（`61` → 5，`71253` → 12，一位数 → 0）。写函数求正整数的 digit distance，必须用递归。

**Starter**（原样摘录）

```python
def digit_distance(n):
    """Determines the digit distance of n.

    >>> digit_distance(3)
    0
    >>> digit_distance(777) # 0 + 0
    0
    >>> digit_distance(314) # 2 + 3
    5
    >>> digit_distance(31415926535) # 2 + 3 + 3 + 4 + ... + 2
    32
    >>> digit_distance(3464660003)  # 1 + 2 + 2 + 2 + ... + 3
    16
    >>> from construct_check import check
    >>> # ban all loops
    >>> check(HW_SOURCE_FILE, 'digit_distance',
    ...       ['For', 'While'])
    True
    """
    "*** YOUR CODE HERE ***"
```

**答案**

```python
def digit_distance(n):
    if n // 10 == 0:
        return 0
    else:
        right = n % 10
        other = n // 10
        next_right = other % 10
        return abs(right - next_right) + digit_distance(other)
```

**验证**

```bash
python3 ok -q digit_distance
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

这题的难点是：**每层递归只能直接看到末位一个数字**，但要做差的是相邻两位。

解法是让「左边那位」通过参数传下去——把 `n // 10` 交给下一层，于是**这一层的左数，正好成为下一层的右数**：

```text
digit_distance(314)
  right = 4, other = 31, next_right = 1  →  abs(4 - 1) = 3
digit_distance(31)
  right = 1, other = 3,  next_right = 3  →  abs(1 - 3) = 2
digit_distance(3)
  n // 10 == 0  →  0
总 = 3 + 2 + 0 = 5
```

`31` 这一层的 `right = 1`，正是上一层算过的 `next_right = 1`。同一个数字被两层各算了一次，但读起来最直观。

**为什么边界是 `n // 10 == 0` 而不是 `n == 0`**：一组「相邻对」需要两个数字。只剩一位时就没有对可算了，此时 `n // 10 == 0`。如果等到 `n == 0` 才停，一位数会被补上一个凭空出现的左邻居 `0`，`digit_distance(3)` 就算成 `abs(3 - 0) = 3`。

**关联知识点**

- 数位链上的「相邻关系」靠**传参**保持：`n // 10` 交给下一层，跨层的两个数字就在同一层相遇
- 边界由数据的**最小结构**决定：相邻对需要 2 位，所以停在「不足 2 位」
- `construct_check` 的禁止列表决定写法自由度：Q1 禁 `Assign`（计数只能靠 `return`），这题只禁 `For`/`While`，所以 `right = ...` 这类临时变量完全允许
- 与 Q1 对比：Q1 的边界是「一位都没有」（`n == 0`），Q2 的边界是「不足两位」（`n // 10 == 0`）——**边界跟着你要比较的结构走，不是跟着 0 走**

**易错点**

- `abs(right, next_right)` → `TypeError: abs() takes exactly one argument`，要先做减法再取绝对值
- 边界写成 `n == 0` → 一位数被补上左邻居 0，`digit_distance(3)` 得 3（而不是 0）
- 漏掉 `abs` → 出现负数，`71253` 这种「前大后小」的数结果会偏小甚至为负
- `next_right` 写成 `next_right = n // 10` → 拿「整个高位」和「末位」比，量纲不对；应该是 `other % 10`
- 用 `for` / `while` → `construct_check` 直接判 `False`

### Q3: Interleaved Sum

**原题（English）**

Write a function `interleaved_sum`, which takes in a number `n` and two one-argument functions: `odd_func` and `even_func`. It applies `odd_func` to every odd number and `even_func` to every even number from 1 to `n` *inclusive* and returns the sum.

For example, executing `interleaved_sum(5, lambda x: x, lambda x: x * x)` returns `1 + 2*2 + 3 + 4*4 + 5 = 29`.

**Important:** Implement this function without using any loops or directly testing if a number is odd or even (no using `%`). Instead of directly checking whether a number is even or odd, start with 1, which you know is an odd number.

**Hint:** Introduce an inner helper function that takes an odd number `k` and computes an interleaved sum from `k` to `n` (including `n`).

> **中文**：`interleaved_sum(n, odd_func, even_func)` 对 1..n 每个数交替套用 `odd_func`（奇数位）和 `even_func`（偶数位）再求和。**禁止循环，也禁止用 `%` 判断奇偶**——改从 1 这个已知的奇数出发。Hint：写一个内层辅助函数，参数是「某个奇数 k」，计算 k..n 的交错和。

**Starter**（原样摘录）

```python
def interleaved_sum(n, odd_func, even_func):
    """Compute the sum odd_func(1) + even_func(2) + odd_func(3) + ..., up
    to n.

    >>> identity = lambda x: x
    >>> square = lambda x: x * x
    >>> triple = lambda x: x * 3
    >>> interleaved_sum(5, identity, square) # 1   + 2*2 + 3   + 4*4 + 5
    29
    >>> interleaved_sum(5, square, identity) # 1*1 + 2   + 3*3 + 4   + 5*5
    41
    >>> interleaved_sum(4, triple, square)   # 1*3 + 2*2 + 3*3 + 4*4
    32
    >>> interleaved_sum(4, square, triple)   # 1*1 + 2*3 + 3*3 + 4*3
    28
    >>> from construct_check import check
    >>> check(HW_SOURCE_FILE, 'interleaved_sum', ['While', 'For', 'Mod']) # ban loops and %
    True
    >>> check(HW_SOURCE_FILE, 'interleaved_sum', ['BitAnd', 'BitOr', 'BitXor']) # ban bitwise operators, don't worry about these if you don't know what they are
    True
    """
    "*** YOUR CODE HERE ***"
```

> 除了 `For` / `While`，这里还禁了 `Mod`（`%`）和位运算 `&` `|` `^`——所有「判奇偶」的捷径都被堵死，只剩「用两个交替的辅助函数」这一条路。

**答案**

```python
def interleaved_sum(n, odd_func, even_func):
    def odd(k):
        if k > n:
            return 0
        return odd_func(k) + even(k + 1)

    def even(k):
        if k > n:
            return 0
        return even_func(k) + odd(k + 1)

    return odd(1)
```

**验证**

```bash
python3 ok -q interleaved_sum
```

```text
Test summary
    1 test cases passed! No cases failed.
```

**思路**

先把这题拆成「会的部分」和「新的部分」：

**会的部分**是普通递归求和——`total(k) = k + total(k + 1)`，`k > n` 时停。**新的部分**只有一处：当前这个 `k` 该交给 `odd_func` 还是 `even_func`。

答案是不判断，让**调用关系**携带这个信息：

```text
odd(1)  = 1  + even(2)      # 1 是奇数 → odd_func
even(2) = 4  + odd(3)       # 2 是偶数 → even_func
odd(3)  = 3  + even(4)
even(4) = 16 + odd(5)
odd(5)  = 5  + even(6)
even(6) = 0                 # 6 > 5，停
总 = 1 + 4 + 3 + 16 + 5 = 29
```

`odd` 只被 `even(k + 1)` 和入口 `odd(1)` 调用，所以它手里的 `k` **一定**是奇数；`even` 同理。这就是题目禁掉 `%` 之后仍然能做的原因——**奇偶不是算出来的，是由谁调用谁决定的**。

边界 `k > n` 同时收得住两种 `n`：`n` 为奇数时是 `even(n + 1)` 停下，`n` 为偶数时是 `odd(n + 1)` 停下。

**关联知识点**

- **互递归**：两个函数互为对方的「下一层」，不需要任何显式状态变量
- 「状态编码在结构里」：奇偶不靠值判断，靠调用链的位置；和 Q1 的「靠返回值累加」是两种不同的传递方式
- **闭包**：`odd` / `even` 直接读外层的 `n`、`odd_func`、`even_func`，所以参数只留会变的 `k`
- 与 Disc 3 Q4 对照：`hailstone` → `even` / `odd` 也是互递归，但那题靠**函数名**区分「这一跳是偶还是奇」；这题靠**调用顺序**交替

**易错点**

- 想用 `k % 2` 判奇偶 → `Mod` 被 ban，`check` 返回 `False`（不是报错，是测试项判失败）
- 边界写成 `k >= n` → 把第 `n` 项也跳过了；要算到 `n`，所以是 `k > n` 才停
- 两个函数调用同一个 func（`odd` 里也调 `even_func`）→ 交替错位，结果偏大或偏小
- 下一层写成 `odd(k + 1)`（`odd` 调自己）→ 从未进入 `even`，且 `k` 只增不减条件不满足会无限递归
- 忘了 `return odd(1)` 或从 `0` 开始 → 整个序列错位，`odd_func` 套到了偶数上
- 用 `for` / `while` 收集 → 直接被 `construct_check` 拦下

### Q4: Count Dollars

**原题（English）**

Given a positive integer `total`, a set of dollar bills makes change for `total` if the sum of the values of the dollar bills is `total`. Here we will use standard US dollar bill values: 1, 5, 10, 20, 50, and 100. For example, the following sets make change for `15`:

- 15 1-dollar bills
- 10 1-dollar, 1 5-dollar bills
- 5 1-dollar, 2 5-dollar bills
- 5 1-dollar, 1 10-dollar bills
- 3 5-dollar bills
- 1 5-dollar, 1 10-dollar bills

Thus, there are 6 ways to make change for `15`. Write a **recursive** function `count_dollars` that takes a positive integer `total` and returns the number of ways to make change for `total` using 1, 5, 10, 20, 50, and 100 dollar bills.

Use `next_smaller_dollar` in your solution: `next_smaller_dollar` will return the next smaller dollar bill value from the input (e.g. `next_smaller_dollar(5)` is `1`). *The function will return `None` if the next dollar bill value does not exist.*

**Important:** Use recursion; the tests will fail if you use loops.

**Hint:** Refer to the implementation of `count_partitions` for an example of how to count the ways to sum up to a final value with smaller parts. If you need to keep track of more than one value across recursive calls, consider writing a helper function.

> **中文**：用面额 1/5/10/20/50/100 凑出 `total`，返回方案数（`15` 有 6 种）。必须用递归，并且要用官方给的 `next_smaller_dollar`（`100 → 50 → 20 → 10 → 5 → 1`，到 1 之后返回 `None`）。Hint：参考教材里 `count_partitions` 的实现——每次在「用一个当前面额」和「不用当前面额、换更小面额」之间分叉；需要跨层记住多个值时写辅助函数。

**Starter**（原样摘录）

```python
def next_smaller_dollar(bill):
    """Returns the next smaller bill in order."""
    if bill == 100:
        return 50
    if bill == 50:
        return 20
    if bill == 20:
        return 10
    elif bill == 10:
        return 5
    elif bill == 5:
        return 1

def count_dollars(total):
    """Return the number of ways to make change.

    >>> count_dollars(15)  # 15 $1 bills, 10 $1 & 1 $5 bills, ... 1 $5 & 1 $10 bills
    6
    >>> count_dollars(10)  # 10 $1 bills, 5 $1 & 1 $5 bills, 2 $5 bills, 10 $1 bills
    4
    >>> count_dollars(20)  # 20 $1 bills, 15 $1 & $5 bills, ... 1 $20 bill
    10
    >>> count_dollars(45)  # How many ways to make change for 45 dollars?
    44
    >>> count_dollars(100) # How many ways to make change for 100 dollars?
    344
    >>> count_dollars(200) # How many ways to make change for 200 dollars?
    3274
    >>> from construct_check import check
    >>> # ban iteration
    >>> check(HW_SOURCE_FILE, 'count_dollars', ['While', 'For'])
    True
    """
    "*** YOUR CODE HERE ***"
```

> 注意 `next_smaller_dollar` 对 `1` 没有分支 → 隐式返回 `None`，这正是 Hint 里说的「没有更小面额了」的信号。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q count_dollars
```

---

## Check Your Score Locally

You can locally check your score on each question of this assignment by running

```bash
python3 ok --score
```

**This does NOT submit the assignment!** When you are satisfied with your score, submit the assignment to Gradescope to receive credit for it.

> **中文**：`python3 ok --score` 只在本地看分，不等于提交；满意后要自己去 Gradescope 上交。

**记录**

（做完再补）

---

## Optional Questions

These questions are optional. If you don't complete them, you will still receive credit for this assignment. They are great practice, so do them anyway!

> **中文**：选做，不做也不影响得分，但值得练。

### Q5: Count Dollars Upward

**原题（English）**

Write a **recursive** function `count_dollars_upward` that is just like `count_dollars` except it uses `next_larger_dollar`, which returns the next larger dollar bill value from the input (e.g. `next_larger_dollar(5)` is `10`). *The function will return `None` if the next dollar bill value does not exist.*

**Important:** Use recursion; the tests will fail if you use loops.

> **中文**：和 Q4 完全一样，只是面额从 1 开始往上走（`1 → 5 → 10 → 20 → 50 → 100`，`next_larger_dollar(100)` 返回 `None`）。必须用递归。

**Starter**（原样摘录）

```python
def next_larger_dollar(bill):
    """Returns the next larger bill in order."""
    if bill == 1:
        return 5
    elif bill == 5:
        return 10
    elif bill == 10:
        return 20
    elif bill == 20:
        return 50
    elif bill == 50:
        return 100

def count_dollars_upward(total):
    """Return the number of ways to make change using bills.

    >>> count_dollars_upward(15)  # 15 $1 bills, 10 $1 & 1 $5 bills, ... 1 $5 & 1 $10 bills
    6
    >>> count_dollars_upward(10)  # 10 $1 bills, 5 $1 & 1 $5 bills, 2 $5 bills, 10 $1 bills
    4
    >>> count_dollars_upward(20)  # 20 $1 bills, 15 $1 & $5 bills, ... 1 $20 bill
    10
    >>> count_dollars_upward(45)  # How many ways to make change for 45 dollars?
    44
    >>> count_dollars_upward(100) # How many ways to make change for 100 dollars?
    344
    >>> count_dollars_upward(200) # How many ways to make change for 200 dollars?
    3274
    >>> from construct_check import check
    >>> # ban iteration
    >>> check(HW_SOURCE_FILE, 'count_dollars_upward', ['While', 'For'])
    True
    """
    "*** YOUR CODE HERE ***"
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q count_dollars_upward
```

---

## Exam Practice

Homework assignments will also contain prior exam-level questions for you to take a look at. These questions have no submission component; feel free to attempt them if you'd like a challenge!

- Fall 2017 MT1 Q4a: Digital（`exam/fa17/mt1/61a-fa17-mt1.pdf` 第 5 页）
- Fall 2019 Final Q6b: Palindromes（`exam/fa19/final/61a-fa19-final.pdf` 第 6 页）

> **中文**：往届考试题，不用提交，想挑战可以做。

---

## Just For Fun Questions

The questions below are out of scope for 61A. You can try them if you want an extra challenge, but they're just puzzles that are not required for the course. Almost all students will skip them, and that's fine. We will **not** be prioritizing support for these questions on Ed or during Office Hours.

> **中文**：超出课程范围，纯益智，绝大多数人会跳过，助教也不会优先答疑。

### Q6: Towers of Hanoi

**原题（English）**

A classic puzzle called the Towers of Hanoi is a game that consists of three rods, and a number of disks of different sizes which can slide onto any rod. The puzzle starts with `n` disks in a neat stack in ascending order of size on a `start` rod, the smallest at the top, forming a conical shape.

The objective of the puzzle is to move the entire stack to an `end` rod, obeying the following rules:

- Only one disk may be moved at a time.
- Each move consists of taking the top (smallest) disk from one of the rods and sliding it onto another rod, on top of the other disks that may already be present on that rod.
- No disk may be placed on top of a smaller disk.

Complete the definition of `move_stack`, which prints out the steps required to move `n` disks from the `start` rod to the `end` rod without violating the rules. The provided `print_move` function will print out the step to move a single disk from the given `origin` to the given `destination`.

**Hint:** Draw out a few games with various `n` on a piece of paper and try to find a pattern of disk movements that applies to any `n`. In your solution, take the recursive leap of faith whenever you need to move any amount of disks less than `n` from one rod to another. If you need more help, see the following hints.

> **中文**：三根杆、n 个大小不同的盘，全部从 `start` 移到 `end`；一次只能移一个盘，大盘不能压小盘。补完 `move_stack(n, start, end)` 打印每一步（用官方给的 `print_move`）。Hint：先手画几个 n 找规律，需要移动少于 n 个盘时直接「信任递归」。

**Hint 1**：官方给了一个 6 盘解法的动画链接（Wikimedia，见页面）。

**Hint 2**：The strategy used in Towers of Hanoi is to move all but the bottom disc to the second peg, then moving the bottom disc to the third peg, then moving all but the second disc from the second to the third peg.

> **中文**：先把最底下那个盘上面的 n-1 个盘移到「另一个杆」，把最底下的盘移到目标杆，再把那 n-1 个盘移到目标杆。

**Hint 3**：One thing you don't need to worry about is collecting all the steps. `print` effectively "collects" all the results in the terminal as long as you make sure that the moves are printed in order.

> **中文**：不用把步骤收集起来返回，只要保证按顺序 `print` 就行。

**Starter**（原样摘录）

```python
def print_move(origin, destination):
    """Print instructions to move a disk."""
    print("Move the top disk from rod", origin, "to rod", destination)

def move_stack(n, start, end):
    """Print the moves required to move n disks on the start pole to the end
    pole without violating the rules of Towers of Hanoi.

    n -- number of disks
    start -- a pole position, either 1, 2, or 3
    end -- a pole position, either 1, 2, or 3

    There are exactly three poles, and start and end must be different. Assume
    that the start pole has at least n disks of increasing size, and the end
    pole is either empty or has a top disk larger than the top n start disks.

    >>> move_stack(1, 1, 3)
    Move the top disk from rod 1 to rod 3
    >>> move_stack(2, 1, 3)
    Move the top disk from rod 1 to rod 2
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 2 to rod 3
    >>> move_stack(3, 1, 3)
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 1 to rod 2
    Move the top disk from rod 3 to rod 2
    Move the top disk from rod 1 to rod 3
    Move the top disk from rod 2 to rod 1
    Move the top disk from rod 2 to rod 3
    Move the top disk from rod 1 to rod 3
    """
    assert 1 <= start <= 3 and 1 <= end <= 3 and start != end, "Bad start/end"
    "*** YOUR CODE HERE ***"
```

> 三根杆的位置只有 1/2/3，`start` 和 `end` 之间那根「过渡杆」可以用 `6 - start - end` 算出来。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q move_stack
```

### Q7: Anonymous Factorial

**原题（English）**

This question demonstrates that it's possible to write recursive functions without assigning them a name in the global frame.

The recursive factorial function can be written as a single expression by using a conditional expression.

```python
>>> fact = lambda n: 1 if n == 1 else mul(n, fact(sub(n, 1)))
>>> fact(5)
120
```

However, this implementation relies on the fact (no pun intended) that `fact` has a name, to which we refer in the body of `fact`. To write a recursive function, we have always given it a name using a `def` or assignment statement so that we can refer to the function within its own body. In this question, your job is to define `fact` recursively without giving it a name!

Write an expression that computes `n` factorial using only call expressions, conditional expressions, and `lambda` expressions (no assignment or `def` statements).

**Note:** You are not allowed to use `make_anonymous_factorial` in your return expression.

The `sub` and `mul` functions from the `operator` module are the only built-in functions required to solve this problem.

> **中文**：要求「不给函数起名字」也能递归。只能用调用表达式、条件表达式、`lambda`，不能用赋值和 `def`（也不能在返回表达式里调用自己）。只允许用 `operator` 的 `sub` 和 `mul`。思路是让函数把自己作为参数传给自己（Y 组合子）。

**Starter**（原样摘录）

```python
from operator import sub, mul

def make_anonymous_factorial():
    """Return the value of an expression that computes factorial.

    >>> make_anonymous_factorial()(5)
    120
    >>> from construct_check import check
    >>> # ban any assignments or recursion
    >>> check(HW_SOURCE_FILE, 'make_anonymous_factorial',
    ...     ['Assign', 'AnnAssign', 'AugAssign', 'NamedExpr', 'FunctionDef', 'Recursion'])
    True
    """
    return 'YOUR_EXPRESSION_HERE'
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q make_anonymous_factorial
```

---

## 本次踩的坑

- （做完再补；通用坑再升级进 `MY/Knowledge/Common_Pitfalls.md`）

---

## 相关概念

- [[Discussion 3 - Recursion]] — 同一批递归手法（拆数位、辅助函数、打印顺序）
- [[Lab 2 - Higher-Order Functions]] — `lambda` 与函数作为参数/返回值，Q3、Q7 会用到
