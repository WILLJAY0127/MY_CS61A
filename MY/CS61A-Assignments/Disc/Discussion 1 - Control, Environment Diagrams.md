---
type: disc
id: "01"
title: Control, Environment Diagrams
status: in_progress
source: SOURCE/CS61A-Website-Archive/disc/disc01/index.html
solution: SOURCE/CS61A-Website-Archive/disc/sol-disc01/index.html
code: none
lectures: Lec02, Lec03
due: TBD
---

# Discussion 1: Control, Environment Diagrams

- **题目来源**：`SOURCE/CS61A-Website-Archive/disc/disc01/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/disc/sol-disc01/index.html`
- **关联讲次**：Lec02–03 · Control + Lec04·05 · Environment Diagrams
- **Due**：TBD

---

## 总起

**一句话定位**：Disc 1 用 6 道小题练 `while` / `if` 控制流，后半部分用两道环境图题把「frame、绑定、parent」概念砸实。

**覆盖知识点**

- `while` 循环：终止条件写的是「继续循环」的条件，循环变量每轮更新
- `if` 分支：Fizzbuzz 多条件判断的书写顺序，特殊条件放前面
- 整除判据：`n % x == 0` 判断因数，`n % 2` 判断奇偶
- 数字拆解：`n % 10` 取末位、`n // 10` 砍末位
- frame 概念：每调用一次自定义函数就新建一个 frame，return 后销毁
- 函数内赋值只建局部名：与全局同名变量互不影响

**题目地图**

| 模块 | 题号 | 类型 |
|---|---|---|
| While and If | Q1 Race（找 bug）｜Q2 Fizzbuzz（写函数） | while/if 控制流 |
| Problem Solving | Q3 Is Prime｜Q4 Unique Digits | 质数判断 + 数字拆解 |
| Environment Diagrams | Q5 Bottles（选择题）｜Q6 Double Trouble（画图） | 环境图概念 |

**对应讲义**：Lec02&03 · Control + Lec04&05 · Environment Diagrams

---

## Ice Breaker

（破冰，省略）

## While and If

### Q1: Race

**原题（English）**

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

Find positive integers `x` and `y` (with `y` larger than `x` but not larger than `2 * x`) for which either:
- `race(x, y)` returns the wrong value or
- `race(x, y)` runs forever

> **中文**：乌龟每分钟走 x 步，兔子快 5 分钟慢 5 分钟反复跑 y 步。找一组 x/y 要么返回错误值，要么永远跑不完。

**答案**

**错答案 case**：`race(3, 5)` 返回 25（真实应是 25/3 ≈ 8.333）

```
t=9 时：tortoise=27, hare=25 → 乌龟反超但 27-25≠0，循环继续
t=25 时才凑到 tortoise=hare=75 → return 25  ← 错
```

**死循环 case**：`race(3, 4)` 平均速度乌龟更快，但再也碰不到恰好整点相等的时刻。

**思路**

循环条件是 `while tortoise - hare`——「乌龟和兔子不相等就继续」，不是「乌龟落后才继续」。样例碰巧整点相等才跑对，找反超落在中间小数时刻的输入就行。两个 bug 叠加：① 反超后不退出；② 一分钟一跳，丢了中间小数时刻。

**关联知识点**

- `while` 循环条件的精确语义：`tortoise - hare` 等于 0 时退出（Python 把 0 当假），不是负数时退出
- 离散模拟的精度丢失：一分钟一步的离散化会漏掉中间时刻

**易错点**

- 把循环条件理解成「乌龟落后才继续」→ 实际上是「不相等就继续」，乌龟反超后循环继续找下一次相等
- 忽略 assert 约束：`y > x` 且 `y <= 2 * x`，`race(1, 3)` 这种反例会被 assert 挡掉

---

### Q2: Fizzbuzz

**原题（English）**

Implement the classic Fizz Buzz sequence. The `fizzbuzz` function takes a positive integer `n` and prints out a single line for each integer from 1 to `n`. For each `i`:
- If `i` is divisible by both 3 and 5, print `fizzbuzz`.
- If `i` is divisible by 3 (but not 5), print `fizz`.
- If `i` is divisible by 5 (but not 3), print `buzz`.
- Otherwise, print the number `i`.

Try to make your implementation of `fizzbuzz` concise.

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

> **中文**：经典 Fizz Buzz 序列——被 3 和 5 整除印 `fizzbuzz`，只被 3 整除印 `fizz`，只被 5 整除印 `buzz`，否则印数字。注意 doctest 里 `print(result)` 输出 `None`——函数没有 return。

**答案**

```python
def fizzbuzz(n):
    i = 1
    while i <= n:
        if i % 15 == 0:
            print('fizzbuzz')
        elif i % 3 == 0:
            print('fizz')
        elif i % 5 == 0:
            print('buzz')
        else:
            print(i)
        i += 1
```

**思路**

`i % 15 == 0` 等价于同时被 3 和 5 整除，必须写在最前面——否则 15 会先被 `i % 3` 或 `i % 5` 吃掉。用 `elif` 而不是一串 `if`，保证每个 `i` 只打印一行。

**关联知识点**

- `if / elif` 的判断顺序：条件最「特殊」的分支必须放最前面，否则会被更宽的条件抢先命中
- 整除判断：`n % k == 0` 是固定写法
- print vs return：`fizzbuzz` 只 print、无 return，隐式返回 None

**易错点**

- 忘了把 `i % 15` 放最前面，结果 15 印成 `fizz` 或 `buzz`
- 用了 `if` 连写而不是 `elif` → 15 会连续印 `fizzbuzz`、`fizz`、`buzz` 三行
- 以为函数应该 return 什么值 → print-only 函数天然返回 None

---

## Problem Solving

官网在 Q2 和 Q3 之间插入了一段通用解题方法论：

> A useful approach to implementing a function is to:
> 1. Pick an example input and corresponding output.
> 2. Describe a process (in English) that computes the output from the input using simple steps.
> 3. Figure out what additional names you'll need to carry out this process.
> 4. Implement the process in code using those additional names.
> 5. Determine whether the implementation really works on your original example.
> 6. Determine whether the implementation really works on other examples. (If not, you might need to revise step 2.)

Disc 02 也重复强调了这套方法（见 `## Higher-Order Functions` 区块）。

### Q3: Is Prime?

**原题（English）**

Write a function that returns `True` if a positive integer `n` is a prime number and `False` otherwise. A prime number n is a number that is not divisible by any numbers other than 1 and n itself. For example, 13 is prime (divisible by 1 and 13), but 14 is not (divisible by 1, 2, 7, 14).

Use the `%` operator: `x % y` returns the remainder of `x` when divided by `y`.

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

> **中文**：写 `is_prime(n)` 返回质数判断结果。注意 **1 不是质数**。用 `%` 判断整除。

**答案**

```python
def is_prime(n):
    if n == 1:
        return False
    i = 2
    while i < n:
        if n % i == 0:
            return False
        i += 1
    return True
```

**思路**

`1` 必须单独排除（`1` 既不是质数也没有因数）。循环从 `i = 2` 开始只试到 `i < n`——因为 `n % n` 恒为 0。循环中途 return False 说明发现因数，循环正常走完没 return 说明没找到因数，返回 True。

**关联知识点**

- `while` 里提前 return 是「找到答案就退出」的标准模式，比设标志位再在循环外判断更简洁
- 循环边界 `i < n` 而非 `i <= n`：多走一步就会用 `n` 去除自己，所有数全被判成非质数
- 边界值单独处理：`1` 不满足 while 条件（`2 < 1` 为假），如果不提前 return，会错误返回 True

**易错点**

- 忘记单独处理 `n == 1` → 返回 True
- 循环写 `i <= n` 把所有数判成非质数
- 把 while 条件写成 `while i * i <= n`（这是更高效的版本，但这题不需要）——没加 `n > 1` 前置的话，会把 1 判成质数

---

### Q4: Unique Digits

**原题（English）**

Write a function that returns the number of unique digits in a positive integer.

**Hints**: You can use `//` and `%` to separate a positive integer into its one's digit and the rest of its digits. You may find it helpful to first define a function `has_digit(n, k)`, which determines whether a number `n` has digit `k`.

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

> **中文**：`unique_digits(n)` 数 n 里有几个不同数字；`has_digit(n, k)` 看 n 里有没有数字 k。Hint 建议先写 `has_digit`，`unique_digits` 调它。

**答案**

```python
def unique_digits(n):
    count = 0
    i = 0
    while i < 10:
        if has_digit(n, i):
            count += 1
        i += 1
    return count

def has_digit(n, k):
    assert k >= 0 and k < 10
    while n > 0:
        if n % 10 == k:
            return True
        n = n // 10
    return False
```

**思路**

十进制拆位三件套：`n % 10` 取末位、`n // 10` 砍末位、`n > 0` 做循环条件。

`has_digit` 从右往左砍，每砍一位就跟 k 比，找到就提前 return True，砍到 0 都没找到就 return False。

`unique_digits` 不直接拆 n（那样还要用集合去重），而是对 0–9 每个数字问一次「n 里有吗」，有就 count += 1——天然避免重复计数。这是**问题转化**的思路。

**关联知识点**

- 十进制拆位三件套是所有「按位处理整数」题的通用骨架
- 问题转化：把「数有几个不同的数字」转成「对 0–9 逐个问它在不在」
- 函数间调用：`unique_digits` 依赖 `has_digit`，职责拆分

**易错点**

- `has_digit` 循环条件写 `n >= 0` → `0 // 10` 仍是 0，死循环
- 忘记写 `has_digit` → `unique_digits` 里硬写拆位 + 去重，复杂度翻倍
- 拆位循环会**破坏原数**（`n = n // 10`），所以只能拿函数参数 n 来改

**验证**

```
unique_digits(8675309) → 7
unique_digits(13173131) → 3
unique_digits(101) → 2
```

---

## Environment Diagrams

### Q5: Bottles（环境图选择题）

**原题（English）**

Answer the following questions with your group. Step through the diagram to check your answers.

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

> **中文**：三道选择题，考 frame 数量、返回值去向、函数内赋值是否改全局。

**执行过程**

前三行定义完后：

```
Global frame
  bottles → 99
  take    → 1
  pass_it → func pass_it(around) [parent=Global]
```

`remaining = bottles - pass_it(bottles)`：

1. 左边 `bottles` → 查全局 → 99
2. 右边 `pass_it(bottles)` → **新建 pass_it frame**：

```
pass_it frame（parent: Global）
  around  → 99          ← 参数绑定
  bottles → 98          ← 局部名字，与全局的 bottles 无关
```

`return take`：本 frame 没有 `take`，沿 parent 向上找 → 全局 `take = 1` → 返回 1，frame 销毁。

3. `remaining = 99 - 1 = 98` → 全局 `remaining → 98`
4. `bottles = remaining` → 全局 `bottles → 98`

**答案**

- **Q5.1 d)** frame 数量 = 自定义函数被调用的次数（不是 call expression 数，不是函数定义数）
- **Q5.2 a)** 返回的 1 参与 `99 - 1`，最终成为全局 `remaining` 的值
- **Q5.3 c)** 函数内 `bottles = 98` 建的是局部名字，全局 `bottles` 全程没被碰过

**关联知识点**

- frame 的创建与销毁：每调用一次自定义函数就新建一个 frame，return 后销毁
- 函数内赋值只建局部名：函数体里的 `x = ...` 在当前 frame 里绑定新名字，同名全局变量完全不受影响
- 名字查找沿 parent 向上：`return take` 在本 frame 找不到，就沿 parent 箭头到全局去找
- 环境图三要素：frame（方框）、名字-值绑定（框内条目）、parent 指针（向上箭头）

**易错点**

- 把 `bottles = 98` 当成改了全局 → 会算成 `remaining = 98 - 1 = 97`，全错
- Q5.3 选 a「临时修改」→ 错在它连临时修改都没有，局部名字从创建起就是独立绑定
- 忽略 `return take` 的名字查找：本 frame 找不到就往 parent 找

---

### Q6: Double Trouble（环境图绘制题）

**原题（English）**

Draw the environment diagram on paper or a whiteboard (without having the computer draw it for you)! Then, check your work by stepping through the diagram.

```python
def double(x):
    return x * 2

def triple(x):
    return x * 3

hat = double
double = triple
```

> **中文**：在纸上画这段代码的环境图。注意——整段代码**没有任何函数调用**，所以只有一个全局 frame。

**最终状态**（核心产出）

| 帧 | 名字 | 绑定值 |
|---|---|---|
| Global | double | function triple（intrinsic name: triple, parent=Global） |
| Global | triple | function triple |
| Global | hat | function double（intrinsic name: double, parent=Global） |

> 整段没有函数调用，只有 Global 一个 frame；两个函数对象、三个名字绑定。

**关键追踪步骤**

- `def double(x):` → 创建 function double（parent=Global），Global 里 `double → func double`
- `def triple(x):` → 创建 function triple（parent=Global），Global 里 `triple → func triple`
- `hat = double` → 先求右边得 **函数对象本身**（不是调用结果），绑定给 `hat`；此时 `hat` 和 `double` 指向同一个对象
- `double = triple` → 求右边得 func triple，把名字 `double` 重新绑过去；`hat` 指向的 func double 纹丝不动——**绑值，不绑关系**

**考点清单**

- **函数是一等值**：函数对象可以像数字一样赋给别的名字
- **赋值绑的是对象，不是名字**：`hat = double` 让 hat 直接指向函数对象；之后重新绑定 `double` 不影响 `hat`
- **两类箭头不能混**：名字 → 对象的绑定箭头（可随时改指向），函数对象 → 定义环境的 parent 箭头（def 时确定，不再改变）
- **没有调用就没有新 frame**：这段代码 0 次调用，所以整张图只有 Global 一个 frame

**验证**

| 调用 | 实际执行 | 结果 |
|---|---|---|
| `double(3)` | func triple | 9 |
| `triple(3)` | func triple | 9 |
| `hat(3)` | func double | 6 |

**关联知识点**

- 对接 Lec04·05 环境图规则
- 闭包机制里也用到「函数对象记住自己的 parent 帧」——这道题的 parent 箭头是 def 时就锁定的，跟名字重绑无关

**易错点**

- 误以为 `hat = double` 让 hat 跟随 double → `hat(3)` 会算成 9，实际是 6
- 把 hat 画成指向 `double` 这个名字（中间有虚线箭头）→ 错，hat 直接指向函数对象
- 数错 frame 个数 → 这段没有调用，只有 Global

---

## 本次踩的坑

- Q1 Race 里容易把 `minutes % 10 < 5` 当成「兔子前 5 分钟跑」——实际上 rabbit 的时间轴和 tortoise 是同一条 minutes，每 10 分钟一个周期（5 跑 + 5 歇）
- Q3 `is_prime` 的 `i < n` 写成 `i <= n` → 多走一步 `n % n == 0` 把所有数判成非质数

## 相关概念

- [[Lab 1 - Functions]] — 数字拆解三件套、while 控制流同源
- [[Lab 2 - Higher-Order Functions]] — Q6 Double Trouble 的「绑值不绑关系」是闭包的前置理解
- Lec02 · Control（`MY/Weekly/`，待写）— while 循环边界、提前 return 模式
