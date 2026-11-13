---
lab: 02
title: Higher-Order Functions, Lambda Expressions
status: in_progress
---

# Lab 2: Higher-Order Functions, Lambda Expressions

**原始题目**：`SOURCE/CS61A-Website-Archive/lab/lab02/index.html`

**官方解答参考**：`SOURCE/CS61A-Website-Archive/lab/sol-lab02/index.html`（仅卡死时对答案用）

---

## 总起

**一句话定位**：Lab 2 是「函数是一等值」的实战——先用 WWPD 把 and/or 短路、函数作参数/返回值、lambda 的求值规则过一遍，再用两道必做编程题（返回函数的函数）落地。

**覆盖知识点**

- 短路求值：`and` / `or` 不求值全部操作数，返回「最后一个被求值的操作数」，不一定是布尔值
- 函数作为参数：函数对象是值，可以传给别的函数
- 函数作为返回值：内层 def 只在函数调用帧里存在，要用返回值接住它
- lambda 表达式：`lambda <参数>: <返回表达式>`，是 expression（求值出函数对象、不改环境），def 是 statement（建函数对象并绑定名字）
- 环境图：函数对象的 parent 帧 = 定义它的帧；lambda 不产生新绑定

**题目地图**：Q1 WWPD 短路｜Q2 WWPD 高阶函数｜Q3 WWPD Lambda｜Q4 composite_identity｜Q5 count_cond｜Q6 环境图（纸笔）｜Q7 multiple（Optional）｜Q8 cycle（Optional）

**对应讲义**：Lec04 · Higher-Order Functions，教材 Section 1.6；环境图规则接 Lec01&02 / Disc02

---

## Starter Files

**Original**

Download lab02.zip.

> **中文**：下载 lab02.zip。

下载地址：`SOURCE/CS61A-Website-Archive/lab/lab02/lab02.zip`

我的代码：`MY/CS61A-Assignments/Lab/lab02/lab02.py`

---

## Topics（前置概念）

**Original**

Consult this section if you need a refresher on the material for this lab. It's okay to skip directly to the questions and refer back here should you get stuck.

> **中文**：需要复习就看这一节，否则直接跳到题目，卡住再回来。

### Short Circuiting

**Original**

What do you think will happen if we type the following into Python?

```python
1 / 0
```

Try it out in Python! You should see a ZeroDivisionError. But what about this expression?

```python
True or 1 / 0
```

It evaluates to True because Python's and and or operators short-circuit. That is, they don't necessarily evaluate every operand.

> **中文**：`1 / 0` 会 `ZeroDivisionError`，但 `True or 1 / 0` 直接得 `True`，因为 `and` / `or` 会短路——不一定求值每个操作数。

| Operator | Checks if: | Evaluates from left to right up to: | Example |
|---|---|---|---|
| AND | All values are true | The first false value | `False and 1 / 0` evaluates to `False` |
| OR | At least one value is true | The first true value | `True or 1 / 0` evaluates to `True` |

**Original**

Short-circuiting happens when the operator reaches an operand that allows them to make a conclusion about the expression. For example, and will short-circuit as soon as it reaches the first false value because it then knows that not all the values are true.

If and and or do not short-circuit, they just return the last value; another way to remember this is that and and or always return the last thing they evaluate, whether they short circuit or not. Keep in mind that and and or don't always return booleans when using values other than True and False.

> **中文**：短路 = 运算符碰到一个足以对整体下结论的操作数就停。`and` / `or` 永远返回**最后一个被求值的操作数**（不管有无短路），所以操作数不是布尔值时，结果也不一定是布尔值。

### Higher-Order Functions

**Original**

Variables are names bound to values, which can be primitives like 3 or 'Hello World', but they can also be functions. And since functions can take arguments of any value, other functions can be passed in as arguments. This is the basis for higher-order functions.

A higher order function is a function that manipulates other functions by taking in functions as arguments, returning a function, or both.

> **中文**：变量可以绑定函数；函数又能收任意值当实参，所以函数能作为参数传入——这就是高阶函数的基础。高阶函数 = 收函数作参数、返回函数，或两者都做。

**Functions as arguments**

**Original**

In Python, function objects are values that can be passed around.

> **中文**：Python 里函数对象是值，可以到处传。

```python
def square(x):
    return x * x

def scale(f, x, k):
    """ Returns the result of f(x) scaled by k. """
    return k * f(x)

>>> scale(square, 3, 2)   # Double square(3)
18
>>> scale(square, 2, 5)   # 5 times 2 squared
20
>>> scale(lambda x: x + 10, 5, 2)
30
```

**Original**

Note that in the body of the call to scale, the function object with the intrinsic name square is bound to the parameter f. Then, we call square in the body of scale by calling f(x).

> **中文**：调用 `scale` 时，intrinsic name 为 `square` 的函数对象被绑定到形参 `f`，函数体里通过 `f(x)` 调用它。

**Functions that return functions**

**Original**

Because functions are values, they are valid as return values!

> **中文**：函数是值，所以也能当返回值。

```python
def multiply_by(m):
    def multiply(n):
        return n * m
    return multiply

>>> multiply_by(3)
<function multiply_by.<locals>.multiply at ...>
>>> multiply(4)
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
NameError: name 'multiply' is not defined
```

**Original**

A call to multiply_by returns a function, as expected. However, calling multiply errors, even though that's the name we gave the inner function. This is because the name multiply only exists within the frame where we evaluate the body of multiply_by.

> **中文**：`multiply_by(3)` 返回函数，但直接调 `multiply(4)` 会 `NameError`——`multiply` 这个名字只存在于 `multiply_by` 的调用帧里。

**Original**

So how do we actually use the inner function? Here are two ways:

> **中文**：两种用法。

```python
>>> times_three = multiply_by(3) # Assign the result of the call expression to a name
>>> times_three(5) # Call the inner function with its new name
15
>>> multiply_by(3)(10) # Chain together two call expressions
30
```

**Original**

multiply_by would look like if we wrote it with a lambda expression:

> **中文**：lambda 写法。

```python
def multiply_by(m):
    return lambda n: n * m
```

### Lambda Expressions

**Original**

Lambda expressions are expressions that evaluate to functions by specifying two things: the parameters and a return expression.

```python
lambda <parameters>: <return expression>
```

> **中文**：lambda 表达式通过「参数 + 返回表达式」求值出一个函数。

**Original**

While both lambda expressions and def statements create function objects, there are some notable differences.

> **中文**：lambda 和 def 都能造函数对象，但有区别。

| | lambda | def |
|---|---|---|
| Type | Expression that evaluates to a value | Statement that alters the environment |
| Result of execution | Creates an anonymous lambda function with no intrinsic name. | Creates a function with an intrinsic name and binds it to that name in the current environment. |
| Effect on the environment | Evaluating a lambda expression does not create or modify any variables. | Executing a def statement both creates a new function object and binds it to a name in the current environment. |
| Usage | A lambda expression can be used anywhere that expects an expression, such as in an assignment statement or as the operator or operand to a call expression. | After executing a def statement, the created function is bound to a name. You should use this name to refer to the function anywhere that expects an expression. |

```python
# A lambda expression by itself does not alter
# the environment
lambda x: x * x

# We can assign lambda functions to a name
# with an assignment statement
square = lambda x: x * x
square(3)

# Lambda expressions can be used as an operator
# or operand
negate = lambda f, x: -f(x)
negate(lambda x: x * x, 3)
```

```python
def square(x):
    return x * x

# A function created by a def statement
# can be referred to by its intrinsic name
square(3)
```

### Environment Diagrams

**Original**

Environment diagrams are one of the best learning tools for understanding lambda expressions and higher order functions because you're able to keep track of all the different names, function objects, and arguments to functions. We highly recommend drawing environment diagrams or using Python tutor if you get stuck doing the WWPD problems below.

> **中文**：环境图是理解 lambda 和高阶函数最好的工具；WWPD 卡住就画图或上 Python Tutor。

**Assignment Statements**

**Original**

1. Evaluate the expression on the right hand side of the = sign.
2. If the name found on the left hand side of the = doesn't already exist in the current frame, write it in. If it does, erase the current binding. Bind the value obtained in step 1 to this name.

If there is more than one name/expression in the statement, evaluate all the expressions first from left to right before making any bindings.

> **中文**：先算 `=` 右边；左边名字已存在就覆盖，否则新建绑定。多个名字/表达式时先从左到右全部求值，再统一绑定。

**def Statements**

**Original**

1. Draw the function object with its intrinsic name, formal parameters, and parent frame. A function's parent frame is the frame in which the function was defined.
2. If the intrinsic name of the function doesn't already exist in the current frame, write it in. If it does, erase the current binding. Bind the newly created function object to this name.

> **中文**：画函数对象（intrinsic name、形参、parent 帧 = 定义它的帧），再绑定名字。

**Call expressions**

**Original**

Note: you do not have to go through this process for a built-in Python function like max or print.

1. Evaluate the operator, whose value should be a function.
2. Evaluate the operands left to right.
3. Open a new frame. Label it with the sequential frame number, the intrinsic name of the function, and its parent.
4. Bind the formal parameters of the function to the arguments whose values you found in step 2.
5. Execute the body of the function in the new environment.

> **中文**：求 operator → 从左到右求 operands → 开新帧（编号 + intrinsic name + parent）→ 绑形参 → 在新环境执行函数体。

**Lambdas**

**Original**

Note: As we saw in the lambda expression section above, lambda functions have no intrinsic name. When drawing lambda functions in environment diagrams, they are labeled with the name lambda or with the lowercase Greek letter λ. This can get confusing when there are multiple lambda functions in an environment diagram, so you can distinguish them by numbering them or by writing the line number on which they were defined.

1. Draw the lambda function object and label it with λ, its formal parameters, and its parent frame. A function's parent frame is the frame in which the function was defined.

This is the only step. We are including this section to emphasize the fact that the difference between lambda expressions and def statements is that lambda expressions do not create any new bindings in the environment.

> **中文**：lambda 函数没有 intrinsic name，画图时标 `λ`（或 lambda），多个时可编号/标行号。只画函数对象，**不产生任何新绑定**——这就是 lambda 和 def 在环境图上的唯一区别。

---

## Required Questions

### Q1: WWPD: The Truth Will Prevail

**Original**

Important: For all WWPD questions, type Function if you believe the answer is `<function...>`, Error if it errors, and Nothing if nothing is displayed.

Use Ok to test your knowledge with the following "What Would Python Display?" questions:

> **中文**：WWPD 题——显示 `<function...>` 填 `Function`，报错填 `Error`，无输出填 `Nothing`。用 ok 检验你的判断。

```bash
python ok -q short-circuit -u
```

#### 问题 1

```python
>>> True and 13
13
>>> False or 0
0
>>> not 10
False
>>> not None
True
```

**说明**：`and` 从左往右求值，遇到第一个假值就停并返回它，全是真值则返回最后一个（`True and 13` → `13`，不是 `True`）。`or` 对称：返回第一个真值，全假则返回最后一个（`False or 0` → `0`）。`not` 是唯一保证返回布尔值的。

---

#### 问题 2

```python
>>> True and 1 / 0   # If this errors, just type Error.
Error
>>> True or 1 / 0    # If this errors, just type Error.
True
>>> -1 and 1 > 0     # If this errors, just type Error.
True
>>> -1 or 5
-1
>>> (1 + 1) and 1    # If this errors, just type Error. If this is blank, just type Nothing.
1
```

**说明**：`and` 要判断「是否全真」，所以左边为真时**必须**继续求右边 → `True and 1 / 0` 触发 `ZeroDivisionError`。`or` 只要左边为真就能下结论 → `True or 1 / 0` 短路，右边根本不执行。`-1` 是真值，所以 `-1 and 1 > 0` 返回右边的 `True`，`-1 or 5` 直接返回 `-1`。

---

#### 问题 3

```python
>>> print(3) or ""
3
''
```

**说明**：两行来源不同——`3` 是 `print` 的副作用（先发生）；`''` 是整条表达式的值（解释器自动显示）。`print(3)` 返回 `None`，`None` 是假值，所以 `or` 继续求右边的 `""` 并返回它。

---

#### 问题 4

```python
>>> def f(x):
...     if x == 0:
...         return "zero"
...     elif x > 0:
...         return "positive"
...     else:
...         return ""
>>> 0 or f(1)
'positive'
>>> f(0) or f(-1)
'zero'
>>> f(0) and f(-1)
''
```

**说明**：`0` 假 → `or` 求 `f(1)` → `'positive'`。`f(0)` 返回 `'zero'` 为真 → `or` 短路，不再调 `f(-1)`。`and` 左边 `'zero'` 为真 → 必须求右边 `f(-1)` → 返回 `''`。注意 `f(-1)` 返回的是空字符串，不是 `None`。

---

### Q2: WWPD: Higher-Order Functions

**Original**

Important: For all WWPD questions, type Function if you believe the answer is `<function...>`, Error if it errors, and Nothing if nothing is displayed.

Use Ok to test your knowledge with the following "What Would Python Display?" questions:

> **中文**：同样是 WWPD，填 `Function` / `Error` / `Nothing`。

```bash
python ok -q hof-wwpd -u
```

```python
>>> def cake():
...    print('beets')
...    def pie():
...        print('sweets')
...        return 'cake'
...    return pie
>>> chocolate = cake()
beets
>>> chocolate
Function
>>> chocolate()
sweets
'cake'
>>> more_chocolate, more_cake = chocolate(), cake
sweets
>>> more_chocolate
'cake'
>>> def snake(x, y):
...    if cake == more_cake:
...        return chocolate
...    else:
...        return x + y
>>> snake(10, 20)
Function
>>> snake(10, 20)()
sweets
'cake'
>>> cake = 'cake'
>>> snake(10, 20)
30
```

**说明**：

- `chocolate = cake()` 先执行调用 → 打印 `beets`；赋值本身不显示，但 print 的副作用已经出现。`cake()` 返回 `pie`，所以 `chocolate` 绑定到 `pie` 函数 → 显示 `Function`。
- `chocolate()` 执行 `pie` 的函数体：先打印 `sweets`，再返回字符串 `'cake'`，解释器自动显示带引号的 `'cake'`。
- `more_chocolate, more_cake = chocolate(), cake`：右边只有 `chocolate()` 被调用（打印 `sweets`）；`cake` 没加括号，是函数对象本身。解包后 `more_chocolate = 'cake'`，`more_cake = <function cake>`。
- `snake(10, 20)` 时 `cake` 和 `more_cake` 指向**同一个函数对象**，`==` 为真 → 返回 `chocolate`（即 `pie` 函数）→ 显示 `Function`；再加 `()` 才调用它，打印 `sweets` 并返回 `'cake'`。
- 最后 `cake = 'cake'` 只改了 Global 里 `cake` 的绑定，`more_cake` 仍指向函数 → 两者不相等 → 走 `else`，返回 `10 + 20 = 30`。

---

### Q3: WWPD: Lambda

**Original**

Use Ok to test your knowledge with the following "What Would Python Display?" questions:

> **中文**：WWPD Lambda 题。注：页面源码里还有几段 WWPD 被 HTML 注释掉了，不在 ok 测试范围内。

```bash
python ok -q lambda -u
```

#### 概念题（Suite 1）

**Q1: Which of the following statements describes a difference between a def statement and a lambda expression?**

> **中文**：下面哪句话描述了 def 语句和 lambda 表达式的区别？

0) A lambda expression does not automatically bind the function that it returns to a name.
1) A lambda expression cannot return another function.
2) A lambda expression cannot have more than two parameters.
3) A def statement can only have one line in its body.

> **中文**：
> 0) lambda 表达式不会自动把返回的函数绑定到一个名字上
> 1) lambda 表达式不能返回另一个函数
> 2) lambda 表达式不能有两个以上参数
> 3) def 语句的函数体只能有一行

**答案：0**

**说明**：`def` 是 statement，执行后会把函数绑定到名字；`lambda` 是 expression，只产出函数对象，不绑定任何名字（要自己 `f = lambda ...`）。1 错（可以返回函数），2 错（参数数量不限），3 错（def 函数体可以多行）。

---

**Q2: How many formal parameters does the following lambda expression have?**

```python
lambda a, b: c + d
```

> **中文**：下面这个 lambda 有几个形参？

0) two
1) Not enough information
2) one
3) three

> **中文**：
> 0) 两个
> 1) 信息不足
> 2) 一个
> 3) 三个

**答案：0（two）**

**说明**：只看**冒号前**的部分，`a, b` 是形参；冒号后的 `c`、`d` 是返回表达式里的自由变量，不是参数（调用时会去外层环境找它们，找不到就 `NameError`）。

---

**Q3: When is the return expression of a lambda expression executed?**

> **中文**：lambda 的返回表达式在什么时候执行？

0) When you pass the lambda expression into another function.
1) When the lambda expression is evaluated.
2) When you assign the lambda expression to a name.
3) When the function returned by the lambda expression is called.

> **中文**：
> 0) 当你把 lambda 传给另一个函数时
> 1) 当 lambda 表达式被求值时
> 2) 当你把 lambda 赋值给一个名字时
> 3) 当 lambda 返回的那个函数被调用时

**答案：3**

**说明**：求值 lambda 只是**创建函数对象**，函数体（return expression）一行都不执行；直到调用这个函数对象时才求值并返回值。

---

#### WWPD 1（Suite 2 Case 1）

**Original**

As a reminder, the following two lines of code will not display any output in the interactive Python interpreter when executed:

```python
>>> x = None
>>> x
>>>
```

> **中文**：提醒——`x = None` 之后再打 `x`，交互解释器什么都不显示。

```python
>>> lambda x: x  # A lambda expression with one parameter x
Function
>>> a = lambda x: x  # Assigning the lambda function to the name a
>>> a(5)
5
>>> (lambda: 3)()  # Using a lambda expression as an operator in a call exp.
3
>>> b = lambda x, y: lambda: x + y  # Lambdas can return other lambdas!
>>> c = b(8, 4)
>>> c
Function
>>> c()
12
>>> d = lambda f: f(4)  # They can have functions as arguments as well.
>>> def square(x):
...     return x * x
>>> d(square)
16
```

**说明**：

- `lambda x: x` 是表达式，求值出一个函数对象 → `Function`（没有 intrinsic name，显示为 `<function <lambda> ...>`）
- `b(8, 4)` 返回内层 `lambda: x + y`，它记住 `b` 帧里的 `x=8, y=4` → `c` 是函数；`c()` 才执行 `8 + 4 = 12`
- `d` 把 `f` 当函数调用，`d(square)` → `square(4) = 16`

---

#### WWPD 2（Suite 2 Case 2）

**Original**

Try drawing an environment diagram if you get stuck!

> **中文**：卡住就画环境图。

```python
>>> higher_order_lambda = lambda f: lambda x: f(x)
>>> g = lambda x: x * x
>>> higher_order_lambda(2)(g)  # Which argument belongs to which function call?
Error
>>> higher_order_lambda(g)(2)
4
>>> call_thrice = lambda f: lambda x: f(f(f(x)))
>>> call_thrice(lambda y: y + 1)(0)
3
>>> print_lambda = lambda z: print(z)  # When is the return expression of a lambda expression executed?
>>> print_lambda
Function
>>> one_thousand = print_lambda(1000)
1000
>>> one_thousand  # What did the call to print_lambda return?
Nothing
```

**说明**：

- 参数是**按位置**对应外层调用的：`higher_order_lambda(2)` 把 `f` 绑成 `2`，返回的 `lambda x: 2(x)` 再收 `g`，调用时执行 `2(g)` → `int` 不可调用 → `Error`
- 换成 `higher_order_lambda(g)(2)`：`f = g`，返回 `lambda x: g(x)`，再传 `2` → `g(2) = 4`
- `call_thrice` 把 `f` 连套三次：`f(f(f(0)))` → `1 → 2 → 3`
- `print_lambda(1000)` 执行 `print(1000)`，屏幕显示 `1000`；但 `print` 返回 `None`，所以 `one_thousand` 求值为 `None` → 解释器不显示 → `Nothing`

---

### Q4: Composite Identity Function

**Original**

Write a function that takes in two single-argument functions, f and g, and returns another function that has a single parameter x. The returned function should return True if f(g(x)) is equal to g(f(x)) and False otherwise. You can assume the output of g(x) is a valid input for f and vice versa.

> **中文**：写 `composite_identity(f, g)`，收两个单参数函数，返回一个单参数函数；返回的函数在 `f(g(x)) == g(f(x))` 时返回 `True`，否则 `False`。可假设 `g(x)` 是 `f` 的合法输入，反之亦然。

```python
def composite_identity(f, g):
    """
    Return a function with one parameter x that returns True if f(g(x)) is
    equal to g(f(x)). You can assume the result of g(x) is a valid input for f
    and vice versa.

    >>> add_one = lambda x: x + 1        # adds one to x
    >>> square = lambda x: x**2          # squares x [returns x^2]
    >>> b1 = composite_identity(square, add_one)
    >>> b1(0)                            # (0 + 1) ** 2 == 0 ** 2 + 1
    True
    >>> b1(4)                            # (4 + 1) ** 2 != 4 ** 2 + 1
    False
    """
    "*** YOUR CODE HERE ***"
```

**Original**

Use Ok to test your code:

```bash
python ok -q composite_identity
```

**我的写法**

```python
def composite_identity(f, g):
    def composite(x):
        if f(g(x)) == g(f(x)):
            return True
        return False

    return composite
```

**逐步思路**

- 外层只负责接住 `f`、`g`，把「一个参数 x」的操作推迟到内层 → 返回 `composite`（函数对象，不加括号）
- 内层 `composite(x)` 拿到 `x` 后，分别算 `f(g(x))` 和 `g(f(x))`，比较后返回布尔值
- `composite` 的 parent 帧是 `composite_identity` 的调用帧，`f`、`g` 通过闭包拿到

**可以更短**：`==` 本身就是布尔表达式，`if ... return True / return False` 是多余的，内层一行就够：

```python
def composite_identity(f, g):
    def composite(x):
        return f(g(x)) == g(f(x))
    return composite

# 或者直接返回 lambda：
def composite_identity(f, g):
    return lambda x: f(g(x)) == g(f(x))
```

**验证**：`b1(0)` → `square(add_one(0)) = 1`，`add_one(square(0)) = 1` → `True`；`b1(4)` → `square(5) = 25`，`add_one(16) = 17` → `False`。

**易错点**

- `return composite()` 加了括号 → 立刻调用且缺 `x`，报 `TypeError`
- 内层忘记收 `x`，直接在外层比较 → 外层根本没有 `x`
- `f(g(x))` 写成 `f(g) * x` 之类 → 把「调用」和「相乘」混淆

---

### Q5: Count Cond

**Original**

Consider the following implementations of count_fives and count_primes which use the sum_digits and is_prime functions from earlier assignments:

> **中文**：先看两个结构几乎一样的实现（用到了之前作业里的 `sum_digits` 和 `is_prime`）。

```python
def count_fives(n):
    """Return the number of values i from 1 to n (including n)
    where sum_digits(n * i) is 5.
    >>> count_fives(10)  # Among 10, 20, 30, ..., 100, only 50 (10 * 5) has digit sum 5
    1
    >>> count_fives(50)  # 50 (50 * 1), 500 (50 * 10), 1400 (50 * 28), 2300 (50 * 46)
    4
    """
    i = 1
    count = 0
    while i <= n:
        if sum_digits(n * i) == 5:
            count += 1
        i += 1
    return count

def count_primes(n):
    """Return the number of prime numbers up to and including n.
    >>> count_primes(6)   # 2, 3, 5
    3
    >>> count_primes(13)  # 2, 3, 5, 7, 11, 13
    6
    """
    i = 1
    count = 0
    while i <= n:
        if is_prime(i):
            count += 1
        i += 1
    return count
```

**Original**

The implementations look quite similar! Generalize this logic by writing a function count_cond, which takes in a two-argument predicate function condition(n, i). count_cond returns a one-argument function that takes in n, which counts all the numbers from 1 to n that satisfy condition when called.

Note: When we say condition is a predicate function, we mean that it is a function that will return True or False.

> **中文**：两者几乎一样——把逻辑泛化成 `count_cond(condition)`：收一个双参数谓词 `condition(n, i)`，返回一个收 `n` 的单参数函数，数出 1 到 n 中满足条件的 `i` 的个数。谓词 = 返回 `True` / `False` 的函数。

starter 已提供辅助函数（不用重写）：

```python
def sum_digits(y):
    """Return the sum of the digits of non-negative integer y."""
    total = 0
    while y > 0:
        total, y = total + y % 10, y // 10
    return total

def is_prime(n):
    """Return whether positive integer n is prime."""
    if n == 1:
        return False
    k = 2
    while k < n:
        if n % k == 0:
            return False
        k += 1
    return True
```

```python
def count_cond(condition):
    """Returns a function with one parameter N that counts all the numbers from
    1 to N that satisfy the two-argument predicate function Condition, where
    the first argument for Condition is N and the second argument is the
    number from 1 to N.

    >>> count_fives = count_cond(lambda n, i: sum_digits(n * i) == 5)
    >>> count_fives(10)   # 50 (10 * 5)
    1
    >>> count_fives(50)   # 50 (50 * 1), 500 (50 * 10), 1400 (50 * 28), 2300 (50 * 46)
    4

    >>> is_i_prime = lambda n, i: is_prime(i) # need to pass 2-argument function into count_cond
    >>> count_primes = count_cond(is_i_prime)
    >>> count_primes(2)    # 2
    1
    >>> count_primes(3)    # 2, 3
    2
    >>> count_primes(4)    # 2, 3
    2
    >>> count_primes(5)    # 2, 3, 5
    3
    >>> count_primes(20)   # 2, 3, 5, 7, 11, 13, 17, 19
    8
    """
    "*** YOUR CODE HERE ***"
```

**Original**

Use Ok to test your code:

```bash
python ok -q count_cond
```

（解法等你做完再补）

---

## Check Your Score Locally

**Original**

You can locally check your score on each question of this assignment by running:

```bash
python3 ok --score
```

This does NOT submit the assignment! When you are satisfied with your score, submit the assignment to Gradescope to receive credit for it.

> **中文**：只算本地分，不提交；要交分得传到 Gradescope。

---

## Environment Diagram Practice

**Original**

There is no Gradescope submission for this component.

However, we still encourage you to do this problem on paper to develop familiarity with Environment Diagrams, which might appear in an alternate form on the exam. To check your work, you can try putting the code into PythonTutor.

> **中文**：这部分不用提交，但在纸上画，考试可能会变体考；用 PythonTutor 对答案。

### Q6: HOF Diagram Practice

**Original**

Draw the environment diagram that results from executing the code below on paper or a whiteboard. Use tutor.cs61a.org to check your work.

> **中文**：在纸上/白板画下面代码执行后的环境图，用 tutor.cs61a.org 检查。

```python
n = 7

def f(x):
    n = 8
    return x + 1

def g(x):
    n = 9
    def h():
        return x + 1
    return h

def f(f, x):
    return f(x + n)

f = f(g, n)
g = (lambda y: y())(f)
```

（推导做完再补）

---

## Optional Questions

**Original**

These questions are optional. If you don't complete them, you will still receive credit for this assignment. They are great practice, so do them anyway!

> **中文**：选做，不做也满分，但建议做。

### Q7: Multiple

**Original**

Write a function that takes in two numbers and returns the smallest number that is a multiple of both.

> **中文**：收两个数，返回同时是两者倍数的最小数（最小公倍数）。

```python
def multiple(a, b):
    """Return the smallest number n that is a multiple of both a and b.

    >>> multiple(3, 4)
    12
    >>> multiple(14, 21)
    42
    """
    "*** YOUR CODE HERE ***"
```

**Original**

Use Ok to test your code:

```bash
python ok -q multiple
```

（解法等你做完再补）

---

### Q8: I Heard You Liked Functions...

**Original**

Define a function cycle that takes in three functions f1, f2, and f3, as arguments. cycle will return another function g that should take in an integer argument n and return another function h. That final function h should take in an argument x and cycle through applying f1, f2, and f3 to x, depending on what n was. Here's what the final function h should do to x for a few values of n:

- n = 0, return x
- n = 1, apply f1 to x, or return f1(x)
- n = 2, apply f1 to x and then f2 to the result of that, or return f2(f1(x))
- n = 3, apply f1 to x, f2 to the result of applying f1, and then f3 to the result of applying f2, or f3(f2(f1(x)))
- n = 4, start the cycle again applying f1, then f2, then f3, then f1 again, or f1(f3(f2(f1(x))))
- And so forth.

Hint: most of the work goes inside the most nested function.

> **中文**：`cycle(f1, f2, f3)` 返回 `g`，`g(n)` 返回 `h`，`h(x)` 按 n 循环套用 f1→f2→f3（n=0 返回 x，n=4 重新从 f1 开始）。提示：大部分工作在最内层函数里。

```python
def cycle(f1, f2, f3):
    """Returns a function that is itself a higher-order function.

    >>> def add1(x):
    ...     return x + 1
    >>> def times2(x):
    ...     return x * 2
    >>> def add3(x):
    ...     return x + 3
    >>> my_cycle = cycle(add1, times2, add3)
    >>> identity = my_cycle(0)
    >>> identity(5)
    5
    >>> add_one_then_double = my_cycle(2)
    >>> add_one_then_double(1)
    4
    >>> do_all_functions = my_cycle(3)
    >>> do_all_functions(2)
    9
    >>> do_more_than_a_cycle = my_cycle(4)
    >>> do_more_than_a_cycle(2)
    10
    >>> do_two_cycles = my_cycle(6)
    >>> do_two_cycles(1)
    19
    """
    "*** YOUR CODE HERE ***"
```

**Original**

Use Ok to test your code:

```bash
python ok -q cycle
```

（解法等你做完再补）

---

## 本次踩的坑 → 是否要补进 Common_Pitfalls

（做完再补）

## 相关概念

- [[Homework 2 - Higher-Order Functions]] — 函数作参数 / 闭包 / make_repeater，本 Lab 编程题的同源知识
- [[Lab 1 - Functions]] — return vs print、ok 用法、环境图入门
- Lec04 · Higher-Order Functions（`MY/Weekly/Week2/`，待写）
