---
hw: 02
title: Higher-Order Functions
status: completed
---

# Homework 2: Higher-Order Functions

> **原始题目**: `SOURCE/CS61A-Website-Archive/hw/hw02/index.html`

---

## 总起

**一句话定位**：HW02 是高阶函数这一讲的收口——先写一个具体高阶函数，再把它抽象成通用骨架 `accumulate`，最后写「返回函数」的 `make_repeater`（闭包）。

**覆盖知识点**

- 函数作为参数：`term`（单参数）、`fuse`（双参数）在函数体里被调用
- 函数作为返回值：`make_repeater` 返回的内层函数要记住外层的 `f` 和 `n`（闭包）
- 抽象与复用：`summation` / `product` 只差 `fuse` 和 `start` 两个实参
- 单行约束：两个 `_using_accumulate` 封装必须只有一行 `return`

**题目地图**

| 题号 | 函数 | 核心动作 |
|---|---|---|
| Q1 | `product(n, term)` | 把 `term(i)` 当值参与累乘 |
| Q2 | `accumulate` ＋ 两个一行封装 | 把「怎么合并」抽成 `fuse` 参数 |
| Q3 | `make_repeater(f, n)` | 返回函数，对入参连续应用 `f` 共 n 次 |

**对应讲义**：Lec04 · Higher-Order Functions，教材 Section 1.6

---

## 前置准备：doctest 辅助函数

官网原话：Several doctests refer to these functions，hw02.py 顶部已定义好：

```python
from operator import add, mul
square = lambda x: x * x
identity = lambda x: x
triple = lambda x: 3 * x
increment = lambda x: x + 1
```

`add`/`mul` 当 `fuse`，其余四个当 `term`，**不需要重复写**。

---

## Required Questions

### Q1: Product

**原版题目（英文）**

> Write a function called `product` that returns the product of the first `n` terms of a sequence. Specifically, `product` takes in an integer `n` and `term`, a single-argument function that determines a sequence. (That is, `term(i)` gives the `i`th term of the sequence.) `product(n, term)` should return `term(1) * ... * term(n)`.

```python
def product(n, term):
    """Return the product of the first n terms in a sequence.

    n: a positive integer
    term:  a function that takes one argument to produce the term

    >>> product(3, identity)  # 1 * 2 * 3
    6
    >>> product(5, identity)  # 1 * 2 * 3 * 4 * 5
    120
    >>> product(3, square)    # 1^2 * 2^2 * 3^2
    36
    >>> product(5, square)    # 1^2 * 2^2 * 3^2 * 4^2 * 5^2
    14400
    >>> product(3, increment) # (1+1) * (2+1) * (3+1)
    24
    >>> product(3, triple)    # 1*3 * 2*3 * 3*3
    162
    """
    "*** YOUR CODE HERE ***"
```

**Ok 验证**：`python3 ok -q product`

**答案**

```python
def product(n, term):
    total = 1
    while n > 0:
        total = total * term(n)
        n = n - 1
    return total
```

**逐步思路**：初始值 1（乘法单位元），递减写法 `n > 0` 确保乘到 `term(1)`，`term` 必须调用 `term(n)` 不能当数值。

**关联知识点**

- **高阶函数（函数作参数）**：`term` 是函数对象，作为参数传入后在本函数体内被调用 `term(n)`，和传数字没有本质区别——只是"被调用"发生在循环里、调用次数取决于 `n`。这是 CS61A 从 Lec04 起的核心转折点：之前你写的所有循环里，"每一步做什么"都是写死的（加？乘？取反？）；现在 `term` 把"每一项是什么"也抽成了参数，同一个循环骨架能算阶乘（`identity`）、平方积（`square`）、等差数列积（`increment`、`triple`）——这就是代码复用的起点。
- **累乘的单位元**：循环累加/累乘必须从"什么都没做"的状态起步。加法的"什么都没加"是 0（`0 + x = x`），乘法的"什么都没乘"是 1（`1 × x = x`）。如果 `product` 从 0 起步，不管 term 是什么结果都会恒为 0——这不是小 bug，是对单位元概念不理解的直接后果。Q2 的 `start` 参数就是把这个单位元从写死改成可配置。
- **闭区间边界**：题目要求乘到 `term(n)`，所以 n 是闭区间 `[1, n]` 的右端点。递减写法 `while n > 0` 让 n 跑到 1 才停；如果写成 `while n > 1`，n 会从 3→2 就停，`term(1)` 永远不乘——但 `identity(1)=1`、`square(1)=1` 恰好乘不乘都不影响结果，前几个 doctest 完美掩盖了这个 bug，直到 `increment(1)=2`、`triple(1)=3` 才暴露。这就是「边界 case 被巧合掩盖」的经典坑。

**易错点**

- `total` 未初始化就用 → `UnboundLocalError`（函数体内出现 `total = ...` 后 Python 不再去全局找它）
- `while n > 1` 漏掉 `term(1)`，被 `identity(1)=1`、`square(1)=1` 掩盖，直到 `increment`/`triple` 才暴露
- 初始值写成 0 → 结果永远 0
- IDE 自动补全误加 `from itertools import count` → 跑完要清理

---

### Q2: Accumulate

**原版题目（英文）**

> Let's take a look at how `product` is an instance of a more general function called `accumulate`, which we would like to implement:

```python
def accumulate(fuse, start, n, term):
    """Return the result of fusing together the first n terms in a sequence
    and start.  The terms to be fused are term(1), term(2), ..., term(n).
    The function fuse is a two-argument commutative & associative function.

    >>> accumulate(add, 0, 5, identity)  # 0 + 1 + 2 + 3 + 4 + 5
    15
    >>> accumulate(add, 11, 5, identity) # 11 + 1 + 2 + 3 + 4 + 5
    26
    >>> accumulate(add, 11, 0, identity) # 11 (fuse is never used)
    11
    >>> accumulate(add, 11, 3, square)   # 11 + 1^2 + 2^2 + 3^2
    25
    >>> accumulate(mul, 2, 3, square)    # 2 * 1^2 * 2^2 * 3^2
    72
    >>> # 2 + (1^2 + 1) + (2^2 + 1) + (3^2 + 1)
    >>> accumulate(lambda x, y: x + y + 1, 2, 3, square)
    19
    """
    "*** YOUR CODE HERE ***"
```

> `accumulate` has the following parameters:
>
> - `fuse`: a two-argument function that specifies how the current term is fused with the previously accumulated terms
> - `start`: value at which to start the accumulation
> - `n`: a non-negative integer indicating the number of terms to fuse
> - `term`: a single-argument function; `term(i)` is the `i`th term of the sequence
>
> Implement `accumulate`, which fuses the first `n` terms of the sequence defined by `term` with the `start` value using the `fuse` function.
>
> For example, the result of `accumulate(add, 11, 3, square)` is
>
> ```
> add(11,  add(square(1), add(square(2),  square(3)))) =
>     11 +     square(1) +    square(2) + square(3)    =
>     11 +     1         +    4         + 9            = 25
> ```
>
> Assume that `fuse` is commutative, `fuse(a, b) == fuse(b, a)`, and associative, `fuse(fuse(a, b), c) == fuse(a, fuse(b, c))`.
>
> Then, implement `summation` (from lecture) and `product` as one-line calls to `accumulate`.
>
> **Important:** Both `summation_using_accumulate` and `product_using_accumulate` should be implemented with a single line of code starting with `return`.

```python
def summation_using_accumulate(n, term):
    """Returns the sum: term(1) + ... + term(n), using accumulate.

    >>> summation_using_accumulate(5, square)
    55
    >>> summation_using_accumulate(5, triple)
    45
    >>> # This test checks that the body of the function is just a return statement.
    >>> import inspect, ast
    >>> [type(x).__name__ for x in ast.parse(inspect.getsource(summation_using_accumulate)).body[0].body]
    ['Expr', 'Return']
    """
    return ____

def product_using_accumulate(n, term):
    """Returns the product: term(1) * ... * term(n), using accumulate.

    >>> product_using_accumulate(4, square)
    576
    >>> product_using_accumulate(6, triple)
    524880
    >>> # This test checks that the body of the function is just a return statement.
    >>> import inspect, ast
    >>> [type(x).__name__ for x in ast.parse(inspect.getsource(product_using_accumulate)).body[0].body]
    ['Expr', 'Return']
    """
    return ____
```

**Ok 验证**：`python3 ok -q accumulate`、`python3 ok -q summation_using_accumulate`、`python3 ok -q product_using_accumulate`

**答案**

```python
def accumulate(fuse, start, n, term):
    total = start
    while n > 0:
        total = fuse(total, term(n))
        n = n - 1
    return total

def summation_using_accumulate(n, term):
    return accumulate(add, 0, n, term)

def product_using_accumulate(n, term):
    return accumulate(mul, 1, n, term)
```

**逐步思路**

- 从 Q1 出发：把写死的 `1 → start`、`* → fuse(total, term(n))` 两处改成参数
- `n = 0` 时循环不进，直接返回 `start`（doctest `accumulate(add, 11, 0, identity) → 11`）
- 两个封装：累加 `fuse=add, start=0`；累乘 `fuse=mul, start=1`；函数体只能一行 `return`

**关联知识点**

- **高阶函数（双参数 `fuse`）**：Q1 只有一个函数参数 `term`（控制"每一项是什么"），Q2 多了 `fuse` 这个双参数函数参数——它控制"怎么合并两项"。`add`（合并 = 相加）、`mul`（合并 = 相乘）、甚至 `lambda x, y: x + y + 1`（合并 = 相加再加 1），都是合法的 fuse。到这里你应该能感受到高阶函数的真正威力：**把"操作本身"也当数据传进来**，函数不再是写死的行为，而是可以组合的组件。后面 MapReduce、SQL 聚合、Stream reduce 全是这个思路。
- **抽象与复用的正确姿势**：`summation` 和 `product` 两个函数体几乎一模一样，只差初始值 0/1 和合并方式 +/*。第一次看会觉得"写两个也行"，但当你再要写 `product_of_squares`、`sum_of_cubes`、`gcd_all`……每加一个就多一份几乎相同的循环代码——accumulate 就是把这些重复代码抽成一个骨架，`summation_using_accumulate` / `product_using_accumulate` 变成一行参数替换。这就是 DRY（Don't Repeat Yourself）原则的最朴素实现。
- **单位元（identity element）与 `start` 参数**：Q1 里你已经知道累加单位元 0、累乘单位元 1。但为什么 accumulate 里要把它做成参数 `start`，而不是写死？因为不是所有合并都有标准单位元，或者你想从非零/非一起步——比如 `accumulate(add, 11, 5, identity)` 从 11 开始加，`start=11` 让它灵活。官网特地标注了 `fuse` 必须 commutative 和 associative，就是为了保证"从哪一头开始合并不影响结果"，这也是单位元概念的延伸。
- **单行 return 约束的真实意图**：那个 AST doctest 检查 body 只有 `['Expr', 'Return']`，看着像形式主义，其实在倒逼你识别"这里没有新逻辑"。如果你写 `result = accumulate(add, 0, n, term)` 再 `return result`，就是把"换两个参数"当成了新逻辑来对待——实际上它就是一行透传。能一眼认出"这里就是换参数"的能力，对后面写 decorator、adapter、factory 都很重要。
- **边界 `n=0`**：accumulate 有个 Q1 没有的边界 case——`n=0` 时 fuse 一次都不调用、直接返回 `start` 原值。这跟 Q1 里 `n>0` 循环条件是同一个根因：你不能假设题目给的输入一定是"正常数"，必须考虑 0、负数、空序列这些极端情况。

**易错点**

- `fuse` 参数顺序：`fuse(total, term(n))` 累积值在前（虽然 commutative 不影响数值，但语义要一致）
- 把 Q1 代码搬过来忘了改 `total = 1` → 累加也从 1 开始
- 循环条件 `n >= 0` 会死循环（`n=0` 时 fuse 不调用，应直接返回 start）
- 一行 return 加了中间变量会被 AST doctest 拒绝
- 参数顺序搞反 → `TypeError`
- IDE 误加 import（HW01/HW02 两次中招）

---

### Q3: Make Repeater

**原版题目（英文）**

> Implement the function `make_repeater` which takes a one-argument function `f` and a positive integer `n`. It returns a one-argument function, where `make_repeater(f, n)(x)` returns the value of `f(f(...f(x)...))` in which `f` is applied `n` times to `x`. For example, `make_repeater(square, 3)(5)` squares 5 three times and returns 390625, just like `square(square(square(5)))`.

```python
def make_repeater(f, n):
    """Returns the function that computes the nth application of f.

    >>> add_three = make_repeater(increment, 3)
    >>> add_three(5)
    8
    >>> make_repeater(triple, 5)(1) # 3 * (3 * (3 * (3 * (3 * 1))))
    243
    >>> make_repeater(square, 2)(5) # square(square(5))
    625
    >>> make_repeater(square, 3)(5) # square(square(square(5)))
    390625
    """
    "*** YOUR CODE HERE ***"
```

**Ok 验证**：`python3 ok -q make_repeater`

**答案**

```python
def make_repeater(f, n):
    def do_repeat(x):
        result = x
        i = 0
        while i < n:
            result = f(result)
            i = i + 1
        return result
    return do_repeat
```

**逐步思路**

- 先造内层函数 `do_repeat(x)`：它接收最终那个 x，内部循环 n 次套 f
- 内层用自己的循环变量 `i` 递增到 `n`，外层的 `f` 和 `n` **只读不写**（闭包）
- 最后 `return do_repeat`，**不要加括号**（加括号变成调用了）

trace `make_repeater(increment, 3)(5)`：

| 轮次 | i | result |
|---|---|---|
| 初始 | — | 5 |
| 1 | 0 | increment(5) = 6 |
| 2 | 1 | increment(6) = 7 |
| 3 | 2 | increment(7) = 8 |
| 结束 | 3 | return 8 |

**关联知识点**

- **闭包（closure）的本质是环境图里的 parent 指针**：这题和 disc01 Q5/Q6 是同一块知识点的递进。disc01 Q6 让你画 `hat = double; double = triple` 的环境图，那时你画的是「Global frame 里三个名字指向两个函数对象」。Q3 升级了——`make_repeater` 执行时会创建一个 frame，里面有 `f` 和 `n` 两个局部名字；`do_repeat` 这个函数对象的 `parent` 箭头**指向 `make_repeater` 当时的那个 frame**，而不是 Global。当 `make_repeater` 返回后，那个 frame 本来应该被回收，但因为 `do_repeat` 还握着指向它的箭头，它就活下来了——这就是闭包"记住了外层变量"的真实机制，不是魔法，是 Python 垃圾回收器不会回收"还被活着的函数对象 parent 指向"的 frame。
- **闭包里"只读"和"读写"的区别**：这是你实际踩到的坑。`do_repeat` 里想循环，最直觉的写法是 `while n > 0: ... n = n - 1`——但只要函数体里出现 `n = ...`，Python 在**编译时**就把 `n` 判定为 `do_repeat` 的局部变量，闭包的 `n` 查找链直接断裂（Local → Enclosing 变成了只在 Local 找），运行到 `while n > 0` 时 Local 的 `n` 还没初始化，直接 `UnboundLocalError`。解决方法不是用 `nonlocal`（这题还没教），而是内层**开自己的循环变量** `i`，外层的 `f` 和 `n` 只做只读访问。这个教训后面做 decorator、generator、async context 时都会反复用到。
- **函数作为一等值（first-class citizen）的完整图景**：Q1/Q2 你见的是「函数当参数传进去」，Q3 见的是「函数当结果吐出来」——合起来就是一等值的完整定义：函数可以像整数/字符串一样，被赋给变量、作为参数、作为返回值、放进列表、存进字典。disc01 Q6 的 `hat = double` 是最朴素的一等值赋值，Q3 是最有代表性的一等值返回值。后面 `map`、`filter`、装饰器、回调函数全是这个能力的延伸。
- **`return do_repeat` 和 `return do_repeat()` 的区别**：前者返回"这个函数对象本身"，后者返回"调用这个函数的结果"。在环境图里看：`return do_repeat` 就是把 frame 里 `do_repeat → func do_repeat(x)` 这个绑定的值（那个函数框）返回出去；`return do_repeat()` 会先新建一个 `do_repeat` 的 frame（参数 x 还没传！会报错），执行完再返回结果。这是区分"把函数当数据"和"调用函数"的最基础语法，看似简单但新手极容易多打一对括号。

**易错点**

- **内层里给 n 赋值会炸**：`n = n + 1` 让 Python 把 n 判定为内层局部变量，闭包链断裂，报 `UnboundLocalError: cannot access local variable 'n' where it is not associated with a value`（和 Q1 `count = count * term(n)` 同根因）
- **循环方向反了**：`while n > 0:` + `n = n + 1` 是死循环（n 越 +1 越大）；内层要开自己的循环变量 `i`
- **内层多了参数 f**：写成 `def do_repeat(f, x)` 会导致调用方必须 `add_three(increment, 5)` 传两个参数
- **return 加了括号**：`return do_repeat()` 变成立刻调用并返回结果，不是返回函数对象
- 验证四个 doctest：`8, 243, 625, 390625`

---

## Check Your Score Locally

```
python3 ok --score
```

只算分，不提交。要提交得把改过的文件传到 Gradescope。

---

## [Optional] Exam Practice

官网列出的同主题真题（选做，无法提交）：

1. Fall 2019 MT1 Q3：[You Again](https://cs61a.org/exam/fa19/mt1/61a-fa19-mt1.pdf#page=4)［Higher-Order Functions］
2. Fall 2021 MT1 Q1b：[tik](https://cs61a.org/exam/fa21/mt1/61a-fa21-mt1.pdf#page=4)［Functions and Expressions］
