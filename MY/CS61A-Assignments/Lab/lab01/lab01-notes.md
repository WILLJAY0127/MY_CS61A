---
lab: 01
title: Control
status: in_progress
---

# Lab 1 · Control

> **原始题目**: `SOURCE/CS61A-Website-Archive/lab/lab01/index.html`
>
> **官方解答参考**: `SOURCE/CS61A-Website-Archive/lab/sol-lab01/index.html`（仅作映射，不作为结构来源）

---

## Starter Files

下载 lab01.zip：`SOURCE/CS61A-Website-Archive/lab/lab01/lab01.zip`

---

## Review（前置概念）

### Using Python

三种运行方式：

```bash
# 1. 直接运行文件（无 -i）——执行完就退出
python3 lab00.py

# 2. -i 交互模式——执行完后进入 >>> 提示符，可调用函数测试
python3 -i lab00.py

# 3. -m doctest——运行 docstring 里的 doctest 测试
python3 -m doctest lab00.py
```

**注意**：interactive 模式下修改了 .py 文件，需要 exit() 重新启动才能生效。

---

### Using OK

```bash
# 测试单个函数
python3 ok -q FUNCTION

# WWPD 解锁模式
python3 ok -q FUNCTION -u

# 查看本地分数
python3 ok --score
```

如果函数里有调试用 print，前缀加 `DEBUG:` 会被 ok 忽略：

```python
print("DEBUG:", x)
```

---

### Division, Floor Div, and Modulo

| 运算符 | 名称 | 例子 | 结果 |
|---|---|---|---|
| `/` | True Division（带小数） | `25 / 4` | `6.25` |
| `//` | Floor Division（截断小数） | `25 // 4` | `6` |
| `%` | Modulo（取余） | `25 % 4` | `1` |

三者都不允许除以 0 → `ZeroDivisionError`。

**实用技巧**：`x % y == 0` 判断 x 能被 y 整除。例如判断偶数：`x % 2 == 0`。

**HW01/lab01 里反复用到的模式**：

| 需求 | 代码 |
|---|---|
| 取 n 的最后一位 | `n % 10` |
| 砍掉 n 的最后一位 | `n // 10` |
| 判断 n 是否被 k 整除 | `n % k == 0` |
| 取 n 的第 k 位（从右数） | `n // pow(10, k) % 10` |

---

### Return and Print

| | return | print |
|---|---|---|
| 作用 | 提供函数调用的返回值 | 显示值（副作用） |
| 终止函数？ | ✅ 立即终止 | ❌ 不终止，继续执行 |
| 解释器显示 | 带引号（如 `'hello'`） | 不带引号（如 `hello`） |
| 函数末尾没 return | 默认返回 `None` | — |

```python
def what_prints():
    print('Hello World!')        # 副作用：显示 Hello World!
    return 'Exiting this function.'  # 返回值，同时终止函数
    print('61A is awesome!')     # ← 这行永远不会执行

>>> what_prints()
Hello World!
'Exiting this function.'
```

print 是 non-pure function——它有副作用，不终止函数。return 是 statement——它终止函数并产出值。

---

## Required Questions

### Q1: WWPD — Return and Print

运行：`python ok -q return-and-print -u`

#### 问题 1

```python
>>> def welcome():
...     print('Go')
...     return 'hello'
>>> def cal():
...     print('Bears')
...     return 'world'
>>> welcome()
?
?
```

**答案（terminal 跑出来的）：**

```
Go
'hello'
```

**解释：**

函数调用时，**函数体从上到下执行**——print 先发生（副作用显示 Go），然后 return 终止函数，return 的值 `'hello'` 作为 expression 的值被解释器自动 print 出来（带引号）。

---

#### 问题 2

```python
>>> print(welcome(), cal())
?
?
?
```

**答案（terminal 跑出来的）：**

```
Go
Bears
hello world
```

**解释：**

call expression 求值规则（Lec01&02）：**先 evaluate 所有 operand，再 apply operator**。

```
1. evaluate welcome()  → print Go + 返回 'hello'
2. evaluate cal()      → print Bears + 返回 'world'
3. apply print('hello', 'world')  → print 用空格分隔 → hello world
```

副作用（Go、Bears）在求值阶段就输出了，比最终 print 的结果（hello world）先出现在屏幕上。

**考点**（对接 Lec01&02）

- **return vs print**：return 终止函数并产出值；print 是 non-pure function，有副作用但不终止函数
- **解释器输出规则**：expression 的值自动 print（带引号如 `'hello'`）；print 函数的输出不带引号
- **call expression 求值顺序**：先 evaluate 所有 operand（副作用先出），再 apply operator（结果后出）

---

### Q2: Debugging Quiz（concept）

运行：`python ok -q debugging-quiz -u`

参考 debugging article：`SOURCE/CS61A-Website-Archive/articles/debugging/index.html`

---

#### Case 1

Q: In the following traceback, what is the most recent function call?

```
Traceback (most recent call last):
    File "temp.py", line 10, in <module>
      f("hi")
    File "temp.py", line 2, in f
      return g(x + x, x)
    File "temp.py", line 5, in g
      return h(x + y * 5)
    File "temp.py", line 8, in h
      return x + 0
  TypeError: must be str, not int
```

0) `f("hi")`
1) `h(x + y * 5)`
2) `g(x + x, x)`

**答案：1**

traceback 底部是最近的调用，错误发生在 h 内部。

---

#### Case 2

Same traceback. Q: In the following traceback, what is the cause of this error?

0) the code attempted to add a string to an integer
1) there was a missing return statement
2) the code looped infinitely

**答案：0**

`h` 里 `return x + 0`，x 是 string（从 `f("hi")` 传进来的），和 int 相加 TypeError。

---

#### Case 3

Q: How do you write a doctest asserting that `square(2) == 4`?

0) `def square(x): '''input: 2 output: 4''' return x * x`
1) `def square(x): '''square(2) 4''' return x * x`
2) `def square(x): '''doctest: (2, 4)''' return x * x`
3) `def square(x): '''>>> square(2) 4''' return x * x`

**答案：3**

doctest 格式：`>>> 表达式` + 下一行预期输出。

---

#### Case 4

Q: When should you use print statements?

0) To investigate the values of variables at certain points in your code
1) For permanent debugging so you can have long term confidence in your code
2) To ensure that certain conditions are true at certain points in your code

**答案：0**

print 是临时调试工具，看变量值后删掉。选项 2 是 assertion 的用途。

---

#### Case 5

Q: How do you prevent the ok autograder from interpreting print statements as output?

0) You don't need to do anything, ok only looks at returned values, not printed values
1) Print with `#` at the front of the outputted line
2) Print with `'DEBUG:'` at the front of the outputted line

**答案：2**

ok 忽略以 `DEBUG:` 开头的 print。

---

#### Case 6

Q: What is the best way to open an interactive terminal to investigate a failing test for question sum_digits in assignment lab01?

0) `python3 ok -q sum_digits`
1) `python3 ok -q sum_digits -i`
2) `python3 ok -q sum_digits --trace`
3) `python3 -i lab01.py`

**答案：1**

`ok -i` 测试失败时自动停在出错行进入交互模式。

---

#### Case 7

Q: Which of the following is NOT true?

0) Debugging is not a substitute for testing
1) It is generally bad practice to release code with debugging print statements left in
2) Testing is very important to ensure robust code
3) It is generally good practice to release code with assertion statements left in
4) Code that returns a wrong answer instead of crashing is generally better as it at least works fine

**答案：4**

silent failure（返回错误答案）比 crashing 更危险——crashing 立刻告诉你有问题，silent failure 悄悄产生错误结果而你没察觉。

---

### Q3: Pick a Digit

```python
def digit(n, k):
    """Return the digit that is k from the right of n for positive integers n and k.

    >>> digit(3579, 2)
    5
    >>> digit(3579, 0)
    9
    >>> digit(3579, 10)
    0
    """
    return ____
```

**要求**：函数体只能有一条 return statement。

**提示**：Use `//` and `%` and the built-in `pow` function.

运行：`python ok -q digit`

**答案**

```python
def digit(n, k):
    return (n // pow(10, k)) % 10
```

**验证**

```python
digit(3579, 0)   # (3579 // 1) % 10 = 9
digit(3579, 2)   # (3579 // 100) % 10 = 35 % 10 = 5
digit(3579, 10)  # (3579 // 10000000000) % 10 = 0 % 10 = 0
```

**解题思路**

HW01 学的两个操作：`n % 10` 取最后一位，`n // 10` 砍掉最后一位。

第 k 位 = 先砍掉后 k 位（`n // 10^k`），再取剩下的最后一位（`% 10`）。用 `pow(10, k)` 得到 10 的 k 次方。

k=10 时 10^10 远大于 3579，`n // 10^10 = 0`，`0 % 10 = 0`，正好对上 doctest。

**考点**（对接 Lec03 Control）

- **`//` 和 `%` 的组合**：`// 10^k` 砍掉后 k 位，`% 10` 取剩下的最后一位——这是数字拆解的标准模式
- **`pow()` 函数**：`pow(10, k)` 比 `10 ** k` 更规范，内置函数比运算符优先级更清晰
- **一行 return 约束**：强制用纯表达式组合，对接 HW01 Q2 two_of_three 的同类型约束

---

### Q4: Middle Number

```python
def middle(a, b, c):
    """Return the number among a, b, and c that is not the smallest or largest.
    Assume a, b, and c are all different numbers.

    >>> middle(3, 5, 4)
    4
    >>> middle(30, 5, 4)
    5
    >>> middle(3, 5, 40)
    5
    >>> middle(3, 5, 40)
    5
    >>> middle(30, 5, 40)
    30
    """
    return ____
```

**要求**：函数体只能有一条 return statement。

**提示**：Try combining all the numbers and then taking away the ones you don't want to return.

运行：`python ok -q middle`

**答案**

```python
def middle(a, b, c):
    return a + b + c - max(a, b, c) - min(a, b, c)
```

**验证**

```python
middle(3, 5, 4)    # 12 - 5 - 3 = 4
middle(30, 5, 4)   # 39 - 30 - 4 = 5
middle(3, 5, 40)   # 48 - 40 - 3 = 5
middle(30, 5, 40)  # 75 - 40 - 5 = 30
```

**解题思路**

和 HW01 Q2 two_of_three 同思路：**总和减去不要的**。三个数总和减最大减最小，剩下的就是中间值。一行 return，完美符合约束。

**考点**（对接 Lec01&02 + HW01 Q2）

- **用减法代替选择**：不需要 if 判断谁是最大最小，直接用 max/min 选出来减掉——这种"补集思维"在算法里反复出现
- **纯表达式组合**：一行 return 完成逻辑，对接 HW01 Q2 two_of_three 完全同构的约束和思路
- **max/min 作为 building blocks**：内置函数是强大的 primitive，组合它们比自己写 if 分支更简洁

---

### Q5: Syllabus Quiz

填问卷：https://go.cs61a.org/syllabus-quiz

---

## Optional Questions（Lecture 3 后做）

### Q6: Falling Factorial

```python
def falling(n, k):
    """Compute the falling factorial of n to depth k.

    >>> falling(6, 3)  # 6 * 5 * 4
    120
    >>> falling(4, 3)  # 4 * 3 * 2
    24
    >>> falling(4, 1)  # 4
    4
    >>> falling(4, 0)
    1
    """
    "*** YOUR CODE HERE ***"
```

Falling factorial：`n * (n-1) * (n-2) * ... * (n-k+1)`，k=0 时返回 1。

运行：`python ok -q falling`

（解法等你做完再补）

---

### Q7: Divisible By k

```python
def divisible_by_k(n, k):
    """
    >>> a = divisible_by_k(10, 2)  # 2, 4, 6, 8, and 10 are divisible by 2
    2
    4
    6
    8
    10
    >>> a
    5
    >>> b = divisible_by_k(3, 1)  # 1, 2, and 3 are divisible by 1
    1
    2
    3
    >>> b
    3
    >>> c = divisible_by_k(6, 7)  # There are no integers up to 6 divisible by 7
    >>> c
    0
    """
    "*** YOUR CODE HERE ***"
```

打印 1 到 n 中所有能被 k 整除的数，返回个数。

运行：`python ok -q divisible_by_k`

（解法等你做完再补）

---

### Q8: Sum Digits

```python
def sum_digits(y):
    """Sum all the digits of y.

    >>> sum_digits(10) # 1 + 0 = 1
    1
    >>> sum_digits(4224) # 4 + 2 + 2 + 4 = 12
    12
    >>> sum_digits(1234567890)
    45
    >>> a = sum_digits(123) # make sure that you are using return rather than print
    >>> a
    6
    """
    "*** YOUR CODE HERE ***"
```

**提示**：Using floor division and modulo might be helpful here!

运行：`python ok -q sum_digits`

（解法等你做完再补）

---

### Q9: WWPD — What If?（if-statements）

运行：`python ok -q if-statements -u`

**提示**：print (unlike return) does **not** cause the function to exit.

#### 问题 1

```python
>>> def ab(c, d):
...     if c > 5:
...         print(c)
...     elif c > 7:
...         print(d)
...     print('foo')
>>> ab(10, 20)
?
?
```

**答案**（terminal 跑出来的，待你验证）：

```
10
foo
```

---

#### 问题 2

```python
>>> def bake(cake, make):
...     if cake == 0:
...         cake = cake + 1
...         print(cake)
...     if cake == 1:
...         print(make)
...     else:
...         return cake
...     return make
>>> bake(0, 29)
?
?
?
```

**答案**（terminal 跑出来的，待你验证）：

```
1
29
29
```

---

#### 问题 3

```python
>>> bake(1, "mashed potatoes")
?
?
```

**答案**（terminal 跑出来的，待你验证）：

```
mashed potatoes
'mashed potatoes'
```

---

### Q10: Double Eights

```python
def double_eights(n):
    """Return true if n has two eights in a row.
    >>> double_eights(8)
    False
    >>> double_eights(88)
    True
    >>> double_eights(2882)
    True
    >>> double_eights(880088)
    True
    >>> double_eights(12345)
    False
    >>> double_eights(80808080)
    False
    """
    "*** YOUR CODE HERE ***"
```

判断数字里有没有相邻的两个 8。

运行：`python ok -q double_eights`

（解法等你做完再补）
