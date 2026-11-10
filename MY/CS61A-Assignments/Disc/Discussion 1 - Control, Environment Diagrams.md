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
- 环境图：frame 与 name 的绑定关系（待补）

**题目地图**：Q1 Race（找 bug）｜Q2 Fizzbuzz｜Q3 Is Prime｜Q4 Unique Digits｜Q5 Bottles｜Q6 Double Trouble

**对应讲义**：Lec02&03 · Control

---

## disc01 完整结构（原始页面）

| 模块 | 内容 | 自学价值 |
|---|---|---|
| Ice Breaker | 破冰游戏 | ❌ |
| **While and If** | **Q1 Race ~ Q6 Double Trouble** | ✅ 全是 while/if 控制流练习 |

下面只保留 6 道题。

---

## Q1: Race

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

**问题**：找正整数 x 和 y（y > x 但 y ≤ 2x），使得 race 要么返回错误值，要么永远跑不完。找出 bug 并修复。

（解法等你做完再补）

---

## Q2: Fizzbuzz

Implement the classic Fizz Buzz sequence. The `fizzbuzz` function takes a positive integer `n` and prints out a single line for each integer from 1 to `n`. For each `i`:

- If `i` is divisible by both 3 and 5, print `fizzbuzz`.
- If `i` is divisible by 3 (but not 5), print `fizz`.
- If `i` is divisible by 5 (but not 3), print `buzz`.
- Otherwise, print the number `i`.

Try to make your implementation of `fizzbuzz` concise.

```python
def fizzbuzz(n):
    """Print the numbers from 1 to n, replacing the multiples of 3 with
    'fizz', multiples of 5 with 'buzz', and multiples of both with 'fizzbuzz'.

    >>> fizzbuzz(5)
    1
    2
    fizz
    4
    buzz
    """
    "*** YOUR CODE HERE ***"
```

（解法等你做完再补）

---

## Q3: Is Prime?

Write a function that returns `True` if `n` is a prime number and `False` otherwise.

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

（解法等你做完再补）

---

## Q4: Unique Digits

Write a function that returns the number of unique digits in a positive integer.

```python
def unique_digits(n):
    """Return the number of unique digits in positive integer n.

    >>> unique_digits(8675309) # All ten unique digits
    7
    >>> unique_digits(13173131) # Unique digits are 1 and 3
    2
    >>> unique_digits(1)
    1
    """
    "*** YOUR CODE HERE ***"
```

**提示**：`n % 10` 取最后一位，`n // 10` 砍掉最后一位。需要记录"哪些数字出现过"。

（解法等你做完再补）

---

## Q5: Bottles

Write a function that prints the lyrics to the song "99 Bottles of Beer on the Wall".

```python
def bottles(n):
    """Prints out the song starting at n bottles.

    >>> bottles(2)
    2 bottles of beer on the wall, 2 bottles of beer.
    Take one down and pass it around, 1 bottle of beer on the wall.

    1 bottle of beer on the wall, 1 bottle of beer.
    Take one down and pass it around, no more bottles of beer on the wall.

    No more bottles of beer on the wall, no more bottles of beer.
    Go to the store and buy some more, 2 bottles of beer on the wall.
    """
    "*** YOUR CODE HERE ***"
```

（解法等你做完再补）

---

## Q6: Double Trouble

Write a function that takes a positive integer `n` and returns the number of times you have to double `n` before it is greater than or equal to 1000.

```python
def double_trouble(n):
    """Return the number of times you have to double n before it is
    greater than or equal to 1000.

    >>> double_trouble(2)
    9
    >>> double_trouble(500)
    1
    >>> double_trouble(999)
    1
    >>> double_trouble(1000)
    0
    """
    "*** YOUR CODE HERE ***"
```

（解法等你做完再补）
