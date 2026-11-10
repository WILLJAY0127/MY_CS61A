---
disc: 01
title: Control, Environment Diagrams
status: in_progress
---

# Discussion 1: Control, Environment Diagrams

> **原始题目**: `SOURCE/CS61A-Website-Archive/disc/disc01/index.html`

---

## 总起

**一句话定位**：讨论课 1 用 6 道小题练 `while` / `if` 控制流；官网后半部分还有 Environment Diagrams，笔记暂未整理。

**覆盖知识点**

- `while` 循环：终止条件写的是「继续循环」的条件，循环变量每轮更新
- `if` 分支：Fizzbuzz 这类多条件判断的书写顺序
- 整除判据：`n % x == 0` 判断因数，`n % 2` 判断奇偶
- 数字拆解：`n % 10` 取末位、`n // 10` 砍末位
- 边界处理：循环结束后终止值是否还要额外处理
- 环境图：frame 创建时机、局部绑定不影响全局、first-class function 的重绑

**题目地图**

| 模块 | 题号 | 类型 |
|---|---|---|
| While and If | Q1 Race（找 bug）｜Q2 Fizzbuzz（写函数） | while/if 控制流 |
| Problem Solving | Q3 Is Prime｜Q4 Unique Digits（两个函数） | 质数判断 + 数字拆解 |
| Environment Diagrams | Q5 Bottles（3 道选择题）｜Q6 Double Trouble（画图） | 环境图概念 |

**对应讲义**：Lec02&03 · Control + Lec04&05 · Environment Diagrams

---

## disc01 完整结构（原始页面）

| 模块 | 内容 | 自学价值 |
|---|---|---|
| Ice Breaker | 破冰游戏 | ❌ |
| **While and If** | **Q1 Race ~ Q6 Double Trouble** | ✅ 全是 while/if 控制流练习 |

下面只保留 6 道题。

---

## Q1: Race

### 原版题目（英文）

The `race` function below sometimes returns the wrong value and sometimes runs forever.

```python
def race(x, y):
    """The tortoise always walks x feet per minute, while the hare repeatedly
    runs y feet per minute for 5 minutes, then rests for 5 minutes. Return how
    many minutes pass until the tortoise first catches up to the hare.

    >>> race(5, 7)  # After 7 minutes, both have gone 35 steps
    7
    >>> race(2, 4)  # After 10 minutes, both have gone 20 steps
    10
    """
    assert y > x and y <= 2 * x, 'the hare must be fast but not too fast'
    tortoise, hare, minutes = 0, 0, 0
    while minutes == 0 or tortoise - hare:
        tortoise += x
        if minutes % 10 < 5:
            hare += y
        minutes += 1
    return minutes
```

**任务要求**

> Find positive integers `x` and `y` (with `y` larger than `x` but not larger than `2 * x`) for which either:
> - `race(x, y)` returns the wrong value or
> - `race(x, y)` runs forever
>
> You just need to find one pair of numbers that satisfies either of these conditions to finish the question, but if you want to think of more you can.

**Notes**

> - `x += 1` is the same as `x = x + 1` when `x` is assigned to a number.
> - 0 is a false value and all other numbers are true values.

**官方 Hint**（toggle-1）

> The value of `race(x, y)` is incorrect when it is not the **first** time the tortoise passes the hare. Try some small numbers (below 5) to see if you can find a case where `tortoise` has become larger than `hare`, but the expression `tortoise - hare` was not zero when it happened.

Hint 点睛：循环条件是 `tortoise - hare != 0 就继续`——乌龟反超后不退出，只会等下一次整点相等。样例碰巧整点相等才跑对，找反超落在中间小数时刻的输入。

---

### 解法思路（已完成）

**Bug 根源**：两个坑叠加——① 循环条件用「相等才退出」（乌龟反超后继续）；② 一分钟一跳，丢了中间小数时刻。

**错答案 case**：`race(3, 5)` 返回 25（真实应是 25/3 ≈ 8.333）

```
t=9 时：tortoise=27, hare=25 → 乌龟反超但 27-25≠0，循环继续
t=25 时才凑到 tortoise=hare=75 → return 25  ← 错
```

**死循环 case**：`race(3, 4)` 跑 50 万步不退。平均速度乌龟更快，但再也碰不到恰好整点相等的时刻。

**考点**：离散模拟丢失中间状态；`while tortoise - hare` 不是「乌龟落后才继续」，是「不相等就继续」。

---

## Q2: Fizzbuzz

### 原版题目（英文）

> Implement the classic [Fizz Buzz](https://en.wikipedia.org/wiki/Fizz_buzz) sequence. The `fizzbuzz` function takes a positive integer `n` and prints out a single line for each integer from 1 to `n`. For each `i`:
>
> - If `i` is divisible by both 3 and 5, print `fizzbuzz`.
> - If `i` is divisible by 3 (but not 5), print `fizz`.
> - If `i` is divisible by 5 (but not 3), print `buzz`.
> - Otherwise, print the number `i`.
>
> Try to make your implementation of `fizzbuzz` concise.

```python
def fizzbuzz(n):
    """
    >>> result = fizzbuzz(16)
    1
    2
    fizz
    4
    buzz
    fizz
    7
    8
    fizz
    buzz
    11
    fizz
    13
    14
    fizzbuzz
    16
    >>> print(result)
    None
    """
    "*** YOUR CODE HERE ***"
```

**官方 Hint**（toggle-2）

> Be careful about the order of your `if` and `elif` clauses: try first checking if the current number is divisible by both 3 and 5, then check for just divisibility by 3 and just divisibility by 5.

Hint 点睛：先判 15，再判 3 / 5，别让 15 被第一个分支吃掉。

（解法等你做完再补）

---

## Problem Solving（方法论）

官网在 Q2 和 Q3 之间插入了一段**通用解题方法论**，要求讨论课上大家一起走一遍：

> A useful approach to implementing a function is to:
> 1. Pick an example input and corresponding output.
> 2. Describe a process (in English) that computes the output from the input using simple steps.
> 3. Figure out what additional names you'll need to carry out this process.
> 4. Implement the process in code using those additional names.
> 5. Determine whether the implementation really works on your original example.
> 6. Determine whether the implementation really works on other examples. (If not, you might need to revise step 2.)
>
> Importantly, this approach doesn't go straight from reading a question to writing code.
>
> **Important**: It's highly recommended that you **don't** check your work using a computer right away. Instead, talk to your group and think to try to figure out if an answer is correct. On exams, you won't be able to guess and check because you won't have a Python interpreter. Now is a great time to practice checking your work by thinking through examples. You could even draw an environment diagram!

---

## Q3: Is Prime?

### 原版题目（英文）

> Write a function that returns `True` if a positive integer `n` is a prime number and `False` otherwise.
>
> A prime number n is a number that is not divisible by any numbers other than 1 and n itself. For example, 13 is prime, since it is only divisible by 1 and 13, but 14 is not, since it is divisible by 1, 2, 7, and 14.
>
> Use the `%` operator: `x % y` returns the remainder of `x` when divided by `y`.

```python
def is_prime(n):
    """
    >>> is_prime(10)
    False
    >>> is_prime(7)
    True
    >>> is_prime(1) # one is not a prime number!!
    False
    """
    "*** YOUR CODE HERE ***"
```

**官方 Hint**（toggle-3）

> Here's a `while` statement that goes through all numbers above 1 and below `n`:
>
> ```python
> i = 2
> while i < n:
>     ...
>     i = i + 1
> ```
>
> You can use `n % i == 0` to check whether `i` is a factor of `n`. If it is, `return False`.

**Presentation Time**（讨论课要求）

> Come up with a **one sentence description** of the process you implemented to solve `is_prime` that you think someone could understand without looking at your code. Share the sentence with your TA for feedback.

Hint 点睛：`1` 不是质数；while 从 `i = 2` 开始，只要 `n % i == 0` 就 return False。

（解法等你做完再补）

---

## Q4: Unique Digits

### 原版题目（英文）

> Write a function that returns the number of unique digits in a positive integer.

**内嵌 Hint block**（官网在题目里直接内嵌了一个引用块）

> <blockquote>
> <p><strong>Hints:</strong> You can use <code>//</code> and <code>%</code> to separate a positive integer into its one's digit and the rest of its digits.</p>
> <p>You may find it helpful to first define a function <code>has_digit(n, k)</code>, which determines whether a number <code>n</code> has digit <code>k</code>.</p>
> </blockquote>

```python
def unique_digits(n):
    """Return the number of unique digits in positive integer n.

    >>> unique_digits(8675309) # All are unique
    7
    >>> unique_digits(13173131) # 1, 3, and 7
    3
    >>> unique_digits(101) # 0 and 1
    2
    """
    "*** YOUR CODE HERE ***"

def has_digit(n, k):
    """Returns whether k is a digit in n.

    >>> has_digit(10, 1)
    True
    >>> has_digit(12, 7)
    False
    """
    assert k >= 0 and k < 10
    "*** YOUR CODE HERE ***"
```

**官方 Hint**（toggle-4）

> One approach is to loop through every digit from 0 to 9 and check whether `n` has the digit. Count up the ones it has.

Hint 点睛：两道题——`unique_digits` 调 `has_digit`；`has_digit` 用 `n % 10` + `n // 10` 拆数字。toggle Hint 直接遍历 0–9 更简单。

（解法等你做完再补）

---

## Environment Diagrams（环境图模块）

官网侧边栏导航显示 disc01 有三大模块：While and If → Problem Solving → **Environment Diagrams**。Q5 和 Q6 属于第三模块，不是写函数题，是**理解环境图的概念题**。

核心概念：**environment diagram** 用方框（frame）记录每个函数调用创建的名字-值绑定关系。

## Q5: Bottles（环境图选择题）

### 原版题目（英文）

> Answer the following questions with your group. Step through the diagram to check your answers.

```python
bottles = 99
take = 1

def pass_it(around):
    bottles = 98
    return take

remaining = bottles - pass_it(bottles)
bottles = remaining
```

**Q5.1** What determines how many different frames appear in an environment diagram?
- a) The number of functions defined in the code
- b) The number of call expressions in the code
- c) The number of return statements in the code
- d) The number of times user-defined functions are called when running the code

**Q5.2** What happens to the return value of `pass_it(bottles)`?
- a) It is used as the new value of `remaining` in the global frame
- b) It is used as the new value of `bottles` in the global frame
- c) It is used as the new value of `pass_it` in the global frame
- d) None of the above

**Q5.3** What effect does the line `bottles = 98` have on the global frame?
- a) It temporarily changes the value bound to `bottles` in the global frame.
- b) It permanently changes the value bound to `bottles` in the global frame.
- c) It has no effect on the global frame.

PythonTutor 链接：[在代码里直接看](https://pythontutor.com/iframe-embed.html#code=bottles+%3D+99%0Atake+%3D+1%0A%0Adef+pass_it%28around%29%3A%0A++++bottles+%3D+98%0A++++return+take%0A%0Aremaining+%3D+bottles+-+pass_it%28bottles%29%0Abottles+%3D+remaining%0A&codeDivHeight=460&codeDivWidth=350&cumulative=true&curInstr=1&heapPrimitives=nevernest&origin=composingprograms.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false)

**答案**

- **Q5.1 d)** frame 数量 = 自定义函数被调用的次数
- **Q5.2 a)** `pass_it(...)` 返回 1，`99 - 1 = 98` 赋给全局 `remaining`
- **Q5.3 c)** 函数内 `bottles = 98` 只影响 pass_it frame，全局 bottles 仍是 99

（详解见原文）

---

## Q6: Double Trouble（环境图绘制题）

### 原版题目（英文）

> Draw the environment diagram on paper or a whiteboard (without having the computer draw it for you)! Then, check your work by stepping through the diagram.

```python
def double(x):
    return x * 2

def triple(x):
    return x * 3

hat = double
double = triple
```

PythonTutor 链接：[在代码里直接看](https://pythontutor.com/iframe-embed.html#code=def+double%28x%29%3A%0A++++return+x+%2A+2%0A%0Adef+triple%28x%29%3A%0A++++return+x+%2A+3%0A%0Ahat+%3D+double%0Adouble+%3D+triple%0A&codeDivHeight=460&codeDivWidth=350&cumulative=true&curInstr=1&heapPrimitives=nevernest&origin=composingprograms.js&py=3&rawInputLstJSON=%5B%5D&textReferences=false)

**结果**

```
Global frame:
  double → function triple
  triple → function triple
  hat    → function double
```

关键点：`hat = double` 绑的是当时的值（function double 对象），之后 `double = triple` 只重新绑名字，不影响 hat 的绑定。`hat(3)` = 6，`double(3)` = 9。
