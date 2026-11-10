---
lab: 00
title: Getting Started
status: complete
---

# Lab 0 · Getting Started

> **原始题目**: `SOURCE/CS61A-Website-Archive/lab/lab00/index.html`

---

## Starter Files

下载 lab00.zip：`SOURCE/CS61A-Website-Archive/lab/lab00/lab00.zip`

---

## Your First Assignment

### 1) What Would Python Do? (WWPD)

运行：`python ok -q python-basics -u`

#### Case 1

> 以下四个 `?` 是同一个 case 的连续交互，环境上下文共享。

##### 问题 1

```python
>>> x = 20
>>> x + 2
?
```

**答案：22**

x 绑着 20，表达式 `x + 2` 求值为 22。expression 只产出值，不改变环境。

---

##### 问题 2

```python
>>> x = 20
>>> x + 2
22
>>> x
?
```

**答案：20**

`x + 2` 没有修改环境的能力——它只是拿 x 的值算一下。x 始终绑着 20。这就是 Lec01&02 讲的 assignment 绑值不绑关系。

---

##### 问题 3

```python
>>> y = 5
>>> y = y + 3
>>> y * 2
?
```

**答案：16**

`y = y + 3` 是 statement，执行规则：先 evaluate 右边 `y + 3`（此时 y 还是 5，5 + 3 = 8），再把 y 重绑到 8。然后 `y * 2` 是 expression，用新值 8 乘 2 得 16。

---

##### 问题 4

```python
>>> x = 20
>>> y = 5
>>> y = y + 3
>>> y + x
?
```

**答案：28**

x = 20（从没被改），y = 8（被 `y = y + 3` 重绑）。8 + 20 = 28。

---

##### WWPD 考点（对接 Lec01&02）

**expression vs statement 的核心区别**

| | expression | statement |
|---|---|---|
| 走什么规则 | evaluate（求值） | execute（执行） |
| 产出什么 | value | 无返回值，改环境 |
| 本 case 例子 | `x + 2`、`y * 2`、`y + x` | `x = 20`、`y = y + 3` |

**assignment statement 两条规则**：
1. **先算右边再绑左边**——`y = y + 3` 算右边时 y 还是旧值 5
2. **绑值不绑关系**——`x + 2` 不改变 x，要改 x 必须再写一条 assignment

---

### 2) Implementing Functions

文件：`lab00.py`

#### twenty_twenty_four()

```python
def twenty_twenty_four():
    """Come up with the most creative expression that evaluates to 2024
    using only numbers and the +, *, and - operators.

    >>> twenty_twenty_four()
    2024
    """
    return ______
```

**要求**：只能用数字和 `+` `*` `-` 三种运算符，return 一个表达式值为 2024。

**参考答案**（多种写法之一）：

```python
return 2024           # 最直接
# 或者
return 2 * 1000 + 24
# 或者
return 2025 - 1
```

**考点**（对接 Lec01&02）

- **primitive expression**：数字 `2024` 是 primitive expression（叶子），直接求值为自身
- **call expression vs 算术表达式**：`2 * 1000 + 24` 是算术表达式（中缀记法），不是 call expression，但同样走 evaluate 规则，递归求值
- **Abstraction vs 没抽象**：这个函数本身没有 abstraction（return 里没有隐藏逻辑），但整个 lab00 是在铺垫"函数作为 abstraction means"——这是 Lec01&02 three means 的第三轴

---

### 3) Running Tests

```bash
python ok                          # 跑全部测试
python ok -q python-basics -u      # 单独跑 WWPD（带 -u 解锁）
python ok -q twenty_twenty_four    # 单独跑函数测试
python ok --score                  # 查看本地分数
```

Windows 下如果 `python3` 不通，用 `python` 或 `py`。

---

## Appendix: Useful Python Command Line Options

```bash
# 直接运行文件——执行完就退出
python3 lab00.py

# -i 交互模式——执行完进入 >>> 提示符，可调用函数测试
python3 -i lab00.py

# -m doctest——运行 docstring 里的 doctest 测试
python3 -m doctest lab00.py
```

**注意**：interactive 模式下修改了 .py 文件，需要 exit() 重新启动才能生效。
