---
type: disc
id: "03"
title: Recursion
status: in_progress
source: SOURCE/CS61A-Website-Archive/disc/disc03/index.html
solution: SOURCE/CS61A-Website-Archive/disc/sol-disc03/index.html
code: none
lectures: Lec06
due: TBD
---

# Discussion 3: Recursion

- **题目来源**：`SOURCE/CS61A-Website-Archive/disc/disc03/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/disc/sol-disc03/index.html`
- **关联讲次**：Lec06 · Recursion（教材 Section 1.7）
- **Due**：TBD

***

## 总起

**一句话定位**：Disc 3 是递归的第一场实战——从"先打印、再递归、后打印"的输出结构（Q1），到 base case 的设计（Q2），到用辅助函数把循环改写成递归（Q3），最后回到 HW 1 的 hailstone 做递归版（Q4）。

**覆盖知识点**

- 递归的三段式结构：base case / 递归调用 / 组合返回值
- 用递归替代 `while` 和 `for`
- 辅助函数（helper）的设计：少传参数、用闭包捕获外层变量
- 递归函数的"打印顺序"与"返回顺序"的区别
- 数字逐位拆解（`//` 和 `%`）在递归里的用法

**题目地图**

| 题号 | 题名                  | 类型             | 核心                  |
| -- | ------------------- | -------------- | ------------------- |
| Q1 | Swipe               | 写代码（递归 + 打印）   | 先 print、再递归、后 print |
| Q2 | Skip Factorial      | 写代码（base case） | 隔项阶乘的终止条件           |
| Q3 | Is Prime            | 写代码（递归改写循环）    | 内层 helper 替代 while  |
| Q4 | Recursive Hailstone | 写代码（递归）        | 偶数/奇数两条链 + 计步数      |
| Q5 | Sevens              | 选做（模拟）         | 用递归或循环模拟报数游戏        |
| Q6 | Karel the Robot     | 选做（Karel）      | 只用 if / 函数调用写 main  |

**对应讲义**：Lec06 · Recursion，教材 Section 1.7

***

## Getting Started

> 破冰（说名字 + 分享小时候喜欢的食物）与 Midterm 1 后的复习建议，无题目，省略。

***

## Recursion

Many students find this discussion challenging. Everything gets easier with practice. Please help each other learn.

VERY IMPORTANT: In this discussion, don't check your answers until your whole group is sure that the answer is right. Figure things out and check your work by thinking about what your code will do. Your goal should be to have all checks pass the first time you run them! If you need help, ask.

> **中文**：本节偏难，多练就顺了。**非常重要**：全组都确信答案正确之前不要对答案——靠推理代码会做什么来验证，目标是第一次跑就全过。

### Q1: Swipe

**原题（English）**

Implement `swipe`, which prints the digits of argument `n`, one per line, first backward then forward. The left-most digit is printed only once. Do not use `while` or `for` or `str`. (Use recursion, of course!)

> **中文**：实现 `swipe(n)`，把 n 的每一位数字逐行打印，先倒序再正序，最左边那位只打印一次。不能用 `while`、`for`、`str`，只能用递归。

**Starter**（原样摘录）

```python
def swipe(n):
    """Print the digits of n, one per line, first backward then forward.

    >>> swipe(2837)
    7
    3
    8
    2
    8
    3
    7
    """
    if n < 10:
        print(n)
    else:
        "*** YOUR CODE HERE ***"
```

> base case 官方已经写好了，你只需要补 `else` 分支。

**提示（Hint）**

First print the first line of the output, then make a recursive call, then print the last line of the output.

> **中文**：先打印输出的第一行，再递归调用，最后打印输出的最后一行。

**答案**

```python
def swipe(n):
    if n < 10:
        print(n)
    else:
        digit = n % 10
        print(digit)
        swipe(n // 10)
        print(digit)
```

**验证**

```python
>>> swipe(2837)
7
3
8
2
8
3
7
>>> swipe(5)
5
```

**思路**

利用**递归调用夹在两个 print 之间**制造"先倒序、再正序"的效果，`else` 分支里就三步：

1. `print(digit)` —— 先打印末位（倒序那一段）
2. `swipe(n // 10)` —— 递归处理剩下的高位
3. `print(digit)` —— 递归回来后再打印一次末位（正序那一段）

调用栈展开 `swipe(2837)`：

```text
print 7
  swipe(283)
    print 3
      swipe(28)
        print 8
          swipe(2)
            print 2   ← base case，只打印、不再递归
          print 8
        print 3
      print 7
```

最左边那位（2）只打印一次，因为它在 base case 里，没有"递归回来后补打"这一步。

**关联知识点**

- 递归不必"先递归后组合"，**语句可以写在递归调用前后**，靠执行顺序构造对称输出
- base case 的判断对象是 `n`（还剩几位），不是 `n % 10`（一位数字恒小于 10）
- 与 Q4 hailstone 的 `print(n)` + 递归同源：都是"边递归边输出"

**易错点**

- 改动官方给的 base case（写成 `if left_point < 10` 或 `if digit < 10`）→ 条件恒真（`n % 10` 永远在 0–9），第一层就打印一位然后返回。判断对象必须是 `n`（还剩几位）
- `else` 里漏掉递归前的 `print(digit)` → 只正序打印 `2 8 3 7`，倒序那半段丢了
- `else` 里漏掉递归后的 `print(digit)` → 只倒序打印 `7 3 8 2`，正序那半段丢了
- 两个 `print` 都挤在递归的同侧 → 输出不再被递归"夹"住，变成单调的倒序或正序
- `swipe(n = n // 10)` 用关键字传参 → 能跑，但 CS61A 默认位置参数 `swipe(n // 10)`
- 用 `str(n)`、`for`、`while` → 题目明确禁止

***

### Q2: Skip Factorial

**原题（English）**

Define the base case for the `skip_factorial` function, which returns the product of **every other positive integer**, starting with `n`.

> **中文**：为 `skip_factorial` 定义 base case，它返回从 n 开始每隔一个取一个的正整数的乘积。

**Starter**（原样摘录）

```python
def skip_factorial(n):
    """Return the product of positive integers n * (n - 2) * (n - 4) * ...

    >>> skip_factorial(5) # 5 * 3 * 1
    15
    >>> skip_factorial(8) # 8 * 6 * 4 * 2
    384
    """
    if ___:
        return ___
    else:
        return n * skip_factorial(n - 2)
```

> `else` 分支的递归调用官方已经写好，你只填 `___` 两个空。

**提示（Hint）**

If `n` is even, then the base case will be `2`. If `n` is odd, then the base case will be `1`. Try to write a condition that handles both possibilities.

> **中文**：n 为偶数时 base case 是 2，为奇数时 base case 是 1。想一个能同时覆盖两种情况的条件。

**答案**

```python
def skip_factorial(n):
    if n <= 0:
        return 1
    else:
        return n * skip_factorial(n - 2)
```

**验证**

```python
>>> skip_factorial(5)
15
>>> skip_factorial(8)
384
```

**思路**

递归部分官方已给：`n * skip_factorial(n - 2)`，每次砍 2，所以只需要决定"什么时候停、返回什么"。base case 有两种等价设计：

| 设计 | 条件 | 返回值 | 递归链（n=8） |
| -- | -- | -- | -- |
| 链尾即停（官方答案） | `n <= 2` | `n`（链尾本身就是 1 或 2） | `8 * 6 * 4 * 2`，到 2 停 |
| 越过链尾（本答案） | `n <= 0` | `1`（乘法单位元） | `8 * 6 * 4 * 2 * 1`，到 0 停 |

关键观察：**1 是乘法单位元**，乘上它不改变结果，所以"多走一步、越界后返回 1"永远安全。本答案用的就是这个思路，比"链尾即停"多一层递归调用，但 base case 写法更统一（不用管链尾是 1 还是 2）。

**关联知识点**

- 单位元思想：乘法单位元是 1、加法单位元是 0。求和型递归的越界 base case 写 `return 0`，求积型写 `return 1`，不要写反
- base case 的两种通用模式：在最后一个有效值停（返回该值）vs 越过有效值（返回单位元）——后者常让代码更短
- 与 Q1 对照：Q1 是"输出型"递归（靠 print 顺序），Q2 是"累积型"递归（靠返回值逐层相乘）

**易错点**

- 条件 `n - 2 <= 0` 搭配返回 `1` → 条件在链尾（1 或 2）就命中，偶数链尾的 **2 被换成 1**，`skip_factorial(8)` 得 192 而非 384（第一版踩的坑：条件和返回值要配套，提前停就得返回链尾本身）
- 只考虑奇数链（如 `n == 1: return 1`）→ 偶数输入会跳过 1 直接到 0、负数，条件永不命中，无限递归
- base case 返回 `0` → 0 是乘法零元不是单位元，整个乘积归零
- 条件写 `n == 0` → 奇数输入从 1 直接跳到 -1，永远不等于 0，无限递归

***

### Q3: Is Prime

**原题（English）**

Implement `is_prime` that takes an integer `n` greater than 1. It returns `True` if `n` is a prime number and `False` otherwise. Try following the approach below, but implement it recursively without using a `while` (or `for`) statement.

```python
def is_prime(n):
    assert n > 1
    i = 2
    while i < n:
        if n % i == 0:
            return False
        i = i + 1
    return True
```

You will need to define another "helper" function (a function that exists just to help implement this one). Does it matter whether you define it within `is_prime` or as a separate function in the global frame? Try to define it to take as few arguments as possible.

> **中文**：实现 `is_prime(n)`（n > 1），是质数返回 True，否则 False。参照上面的 while 版本，但**不能用 while/for**，改成递归。你需要再定义一个辅助函数——它定义在 `is_prime` 内部还是全局，有区别吗？尽量让它少收参数。

**Starter**（原样摘录）

```python
def is_prime(n):
    """Returns True if n is a prime number and False otherwise.
    >>> is_prime(2)
    True
    >>> is_prime(16)
    False
    >>> is_prime(521)
    True
    """
    "*** YOUR CODE HERE ***"
```

**提示（Hint）**

Define an inner function that checks whether some integer between `i` and `n` evenly divides `n`. Then you can call it starting with `i=2`:

```python
def is_prime(n):
    def f(i):
        if n % i == 0:
            return ____
        elif ____:
            return ____
        else:
            return f(____)
    return f(2)
```

> **中文**：定义一个内层函数，检查 i 到 n 之间有没有整数能整除 n；从 `i=2` 开始调用。

**Documentation**

Come up with a one sentence docstring for the helper function that describes what it does. Don't just write, "it helps implement is\_prime." Instead, describe its behavior. If you need help, ask!

> **中文**：给辅助函数写一句话 docstring，描述它的**行为**，不要只写"它帮助实现 is\_prime"。

**答案**

```python
def is_prime(n):
    def f(i):
        """Check whether no number from i up to n evenly divides n."""
        if i >= n:
            return True
        elif n % i == 0:
            return False
        else:
            return f(i + 1)
    return f(2)
```

> `f` 的 docstring 就是 **Documentation** 那道小问的答案：描述它**检查了什么**，而不是"它辅助 is_prime"。

**验证**

```python
>>> is_prime(2)
True
>>> is_prime(16)
False
>>> is_prime(521)
True
```

**思路**

while 循环的每个部件在递归版里都有明确对应物，可以机械翻译：

| while 版 | 递归版 |
| -- | -- |
| 初始化 `i = 2` | 最外层调用 `helper(2)`（骨架已给） |
| 循环条件 `i < n` | 决定 base case 什么时候命中 |
| 每轮 `i += 1` | 递归调用时参数怎么变（最后一个空） |
| 循环体里 `if n % i == 0: return False` | 一个 base case 分支 |
| 循环正常结束 `return True` | 另一个 base case 分支 |

**分支顺序是这题唯一的陷阱**：`n % n` 恒等于 0，所以"检查是否走完"必须放在"能否整除"**前面**。while 版靠 `while i < n` 把 `i == n` 挡在循环体外，递归版没有这层外壳，只能靠顺序。

**为什么 helper 定义在内部**：`f` 定义在 `is_prime` 体内，通过闭包直接看到外层的 `n`，所以只需要一个参数 `i`；若定义在全局，就得传 `(n, i)` 两个参数。这正是题目"让参数尽量少"的意图，同时 `f` 不会污染全局名字空间。

**关联知识点**

- **循环 → 递归的机械映射**：初始化→入口实参、循环条件→base case、循环体→分支、步进→递归实参、循环后语句→合并进 base case
- **闭包减少参数**：内层函数能直接读外层调用帧的名字（这里读 `n`），于是参数只留"会变的那一个"
- **边界判断优先**：能被所有数整除的边界值（`i == n`）必须先于业务判断处理
- **辅助函数的作用域选择**：内部定义 = 私有 + 省参数；全局定义 = 可复用但污染名字空间

**易错点**

- 分支顺序反了（`i == n` 放 `elif`）→ 质数全被判成 `False`：`is_prime(2)` 在 `i = 2` 时就命中 `2 % 2 == 0`；`is_prime(521)` 在最后一步 `521 % 521 == 0` 命中
- **官方 Hint 骨架顺序有误**：页面 Hint 把 `n % i == 0` 写在第一个 `if`，照抄必错；官方 Solution 是 `i == n` 在前。看到 Hint 骨架先用最小用例（这里 `2`）手跑一遍
- 写成 `elseif`（C/JS/PHP 习惯）→ Python 只有 `elif`，直接 `SyntaxError`
- 内层 `f(i + 1)` 前面漏写 `return` → 递归调用结果被丢弃，函数返回 `None`
- helper 定义在全局却忘了传 `n` → `NameError`；传成 `f(n, i)` 又违背"参数尽量少"

***

### Q4: Recursive Hailstone

**原题（English）**

Recall the `hailstone` function from Homework 1. First, pick a positive integer `n` as the start. If `n` is even, divide it by 2. If `n` is odd, multiply it by 3 and add 1. Repeat this process until `n` is 1. Complete this recursive version of `hailstone` that prints out the values of the sequence and returns the number of steps.

> **中文**：回忆 HW 1 的 hailstone。从正整数 n 开始：偶数就除以 2，奇数就乘 3 加 1，重复直到 n 为 1。补完这个递归版本，打印整个序列并返回步数。

**Starter**（原样摘录）

```python
def hailstone(n):
    """Print out the hailstone sequence starting at n,
    and return the number of elements in the sequence.
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
    >>> b = hailstone(1)
    1
    >>> b
    1
    """
    print(n)
    if n % 2 == 0:
        return even(n)
    else:
        return odd(n)

def even(n):
    return ____

def odd(n):
    "*** YOUR CODE HERE ***"
```

> `hailstone` 主体和 `even` 的骨架官方已给，你补 `even` 的返回值表达式和 `odd` 的函数体。

**提示（Hint）**

An even number is never a base case, so even always makes a recursive call to `hailstone` and returns one more than the length of the rest of the hailstone sequence.

An odd number might be 1 (the base case) or greater than one (the recursive case). Only the recursive case should call `hailstone`.

> **中文**：偶数永远不是 base case，所以偶数分支一定递归，并在返回值上加 1。奇数可能是 1（base case）也可能大于 1（递归情况），只有递归情况才调用 hailstone。

**Recommended**

Once your group has converged on a solution, it's time to practice your ability to describe your own code. Nominate someone to describe how your solution works and have them present to the group for practice. Then, tell this description to your TA for feedback.

> **中文**：全组达成一致后，推一个人讲解自己的解法，练"讲代码"的能力，讲给助教听并获取反馈。

**答案**

```python
def hailstone(n):
    print(n)
    if n % 2 == 0:
        return even(n)
    else:
        return odd(n)

def even(n):
    return hailstone(n // 2) + 1

def odd(n):
    if n == 1:
        return 1
    else:
        return hailstone(n * 3 + 1) + 1
```

**验证**

```python
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
>>> b = hailstone(1)
1
>>> b
1
```

**思路**

`hailstone` 只做两件事：打印当前值、按奇偶转交给 `even` / `odd`。它自己不计数，也不判断终止。

`even` / `odd` 负责算下一跳（交给 `hailstone` 递归），并把后面的项数 `+ 1` 加回来。

`hailstone(10)` 的展开：

```text
hailstone(10)  →  print 10, 返回 even(10)
even(10)       →  hailstone(5) + 1
hailstone(5)   →  print 5,  返回 odd(5)
odd(5)         →  hailstone(16) + 1
hailstone(16)  →  print 16, 返回 even(16)
even(16)       →  hailstone(8) + 1
hailstone(8)   →  print 8,  返回 even(8)
even(8)        →  hailstone(4) + 1
hailstone(4)   →  print 4,  返回 even(4)
even(4)        →  hailstone(2) + 1
hailstone(2)   →  print 2,  返回 even(2)
even(2)        →  hailstone(1) + 1
hailstone(1)   →  print 1,  返回 odd(1)
odd(1)         →  1           ← base case
```

打印在**下去**的路上（10 → 1），加法在**回来**的路上（6 个 `+ 1` 依次加回，`1 + 6 = 7`）。

> 这题本质就是「把 HW 1 的 `while` 改成递归」，和 Q3 同一个技能。`even` / `odd` 的拆分是填空题的骨架形式，顺带让 `even` 变成没有 `if` 的一行（偶数本来就不可能终止）。不用往"设计意图"上过度解读。

**职责划分：循环版 vs 递归版**

HW 1 的 while 版（官方解答）：

```python
def hailstone(n):
    length = 1
    while n != 1:
        print(n)
        if n % 2 == 0:
            n = n // 2
        else:
            n = 3 * n + 1
        length = length + 1
    print(n)
    return length
```

| 职责 | 循环版（HW 1） | 递归版（Disc 3） |
| -- | -- | -- |
| 打印 | 循环体里 `print(n)`，出循环后补一次 | `hailstone` 开头一次 `print(n)` |
| 判断奇偶 | 循环体里的 `if n % 2 == 0` | `hailstone` 里的 `if n % 2 == 0` |
| 算下一跳 | `n = n // 2` / `n = 3 * n + 1` | `even` / `odd` 传给 `hailstone` 的实参 |
| 计数 | `length = length + 1` 每轮加一次 | `+ 1` 写在 `even` / `odd` 的返回值里 |
| 终止条件 | `while n != 1` | `odd(1)` 的 `if n == 1` |
| 谁管全部 | 一个函数全包 | 三个函数分工 |

三个关键差异：

1. **状态 vs 新帧**：循环版 `n = n // 2` 覆盖同一个 `n`，下一圈接着用；递归版 `hailstone(n // 2)` 开一个新帧，每个 `n` 各占一格。
2. **计数方向相反**：循环版 `length += 1` 是"先加，再继续"；递归版 `+ 1` 是"等后面结果回来再加"。
3. **print 的位置**：循环版必须先 print 再赋值（否则打的是下一跳）；递归版 `n` 不被覆盖，但 print 仍要放开头才保证顺序。

**关联知识点**

- 这题相对 Q3 唯一新增的技能：**累加器 → 返回值**。Q3 的 `while` 没有状态（直接 `return True/False`），Q4 的 `while` 用 `length` 累加，转递归时这个 `length` 就变成返回路上的 `+ 1`
- 递归里"打印"和"返回"是两条独立的链：打印下去（先序），加法回来（后序）
- 返回值是**项数**不是**步数**：`hailstone(10)` 返回 7 = 7 项（跳了 6 次）

**易错点**

- 写成 `elif:` 后面不带条件 → `SyntaxError`。`n == 1` 已拦住 base case，剩下的直接 `else:`
- `odd` 的 base case 里又写 `print(n)` → `hailstone(1)` 先打一次、`odd(1)` 再打一次，输出两行 `1`。**打印只归 `hailstone` 管**
- `even` 漏掉 `+ 1` → 少算一项，`hailstone(10)` 得 6
- `odd` 递归分支漏掉 `+ 1` → 同上
- 在 `even` / `odd` 里再写 `print` → 重复打印
- 把返回值当"步数"→ 以为 `hailstone(1)` 该返回 0；实际 base case 那层把自己也算一项，返回 1

***

## Document the Occasion

> 出勤表填写（每周每人提交一次），行政区块，省略。

***

## Extra Questions

The questions below are recommended but optional. Don't worry if you don't get to them (but at least read the first one).

> **中文**：下面这些题推荐但选做，做不完不用担心，但至少把第一题读一遍。

### Q5: Sevens

**原题（English）**

The Game of Sevens: Players in a circle count up from 1 in the clockwise direction. (The starting player says 1, the player to their left says 2, etc.) If a number is divisible by 7 or contains a 7 (or both), switch directions. Numbers must be said on the beat at 60 beats per minute. If someone says a number when it's not their turn or someone misses the beat on their turn, the game ends.

For example, 5 people would count to 20 like this:

```text
Player 1 says 1
Player 2 says 2
Player 3 says 3
Player 4 says 4
Player 5 says 5
Player 1 says 6  # All the way around the circle
Player 2 says 7  # Switch to counterclockwise
Player 1 says 8
Player 5 says 9  # Back around the circle counterclockwise
Player 4 says 10
Player 3 says 11
Player 2 says 12
Player 1 says 13
Player 5 says 14 # Switch back to clockwise
Player 1 says 15
Player 2 says 16
Player 3 says 17 # Switch back to counterclockwise
Player 2 says 18
Player 1 says 19
Player 5 says 20
```

Play a few games. Post the highest score your group reached on Discord.

Then, implement `sevens` which takes a positive integer `n` and a number of players `k`. It returns which of the `k` players says `n`. You may call `has_seven`.

An effective approach to this problem is to simulate the game, stopping on turn `n`. The implementation must keep track of the final number `n`, the current number `i`, the player who will say `i`, and the current direction that determines the next player (either increasing or decreasing). It works well to use integers to represent all of these, with direction switching between `1` (increase) and `-1` (decreasing).

> **中文**：报数游戏——从 1 开始顺时针轮流报数；遇到 7 的倍数或数字里含 7 就反向。实现 `sevens(n, k)`，返回 n 个人里第几个玩家报了 n，可以调用 `has_seven`。推荐做法是**模拟**：一直模拟到第 n 个数，跟踪四个量——目标 n、当前数 i、该报 i 的玩家、当前方向（1 或 -1）。

**Starter**（原样摘录）

```python
def sevens(n, k):
    """Return the (clockwise) position of who says n among k players.

    >>> sevens(2, 5)
    2
    >>> sevens(6, 5)
    1
    >>> sevens(7, 5)
    2
    >>> sevens(8, 5)
    1
    >>> sevens(9, 5)
    5
    >>> sevens(18, 5)
    2
    """
    def f(i, who, direction):
        if i == n:
            return who
        "*** YOUR CODE HERE ***"
    return f(1, 1, 1)

def has_seven(n):
    if n == 0:
        return False
    elif n % 10 == 7:
        return True
    else:
        return has_seven(n // 10)
```

> `has_seven` 官方已给，`f` 的入口调用 `return f(1, 1, 1)` 也已给出，你只补 `f` 里递归那几行。

**提示（Hint）**

First check if `i` is a multiple of 7 or contains a 7, and if so, switch directions. Then, add the direction to `who` and ensure that `who` has not become smaller than 1 or greater than `k`.

> **中文**：先判断 i 是 7 的倍数或含 7，是就换方向；然后把方向加到 `who` 上，并保证 `who` 没有小于 1 或大于 k。

**答案**

（做完再补）

***

### Q6: Karel the Robot

**原题（English）**

Karel the robot starts in the corner of an `n` by `n` square for some unknown number `n`. Karel responds to only four functions:

- `move()` moves Karel one square forward if there is no wall in front of Karel and errors if there is.
- `turn_left()` turns Karel 90 degrees to the left.
- `front_is_blocked()` returns whether there is a wall in front of Karel.
- `front_is_clear()` returns whether there is no wall in front of Karel.

Implement a `main()` function that will leave Karel stopped halfway in the middle of the bottom row. For example, if the square is 7 x 7 and Karel starts in position (1, 1), the bottom left, then Karel should end in position (1, 4) (three steps from either side on the bottom row). Karel can be facing in any direction at the end. If the bottom row length is even, Karel can stop in either position (1, n // 2) or (1, n // 2 + 1).

**Important** You can only write `if` or `if/else` statements and function calls in the body of `main()`. You may not write assignment statements, `def` statements, lambda expressions, or `while`/`for` statements.

> **中文**：Karel 站在 n×n 方格的角落（n 未知），只会四个动作：前进、左转、前方有墙？、前方无墙？。实现 `main()`，让 Karel 停在底行正中间。**限制**：`main()` 里只能写 if / if-else 和函数调用，不能有赋值、def、lambda、while/for。

**提示（Hint）**

For every two steps forward, take one step back to end up in the middle.

> **中文**：每前进两步就后退一步，最终就会停在中间。

**答案**

（做完再补）

***

## 本次踩的坑

（做完再补）

## 相关概念

- \[\[Discussion 2 - Environment Diagrams, Higher-Order Functions]] — 递归函数调用也是环境图的栈帧叠加，前置知识
- \[\[Homework 1 - Functions, Control]] — Q4 hailstone 的原型在这里
- \[\[Lab 2 - Higher-Order Functions]] — 把函数当参数的思路，Q3 的 helper 会用到
