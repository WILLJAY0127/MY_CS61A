---
lab: 00
title: Getting Started
status: complete
---

# Lab 0 · Getting Started

> **核心内容**: Python 环境配置 + expression/statement 入门 + 函数实现。
>
> **原始**: `SOURCE/CS61A-Website-Archive/lab/lab00/index.html`

---

## Part 1: What Would Python Do? (WWPD)

运行方式：`python ok -q python-basics -u`

### WWPD · Case 1

> 以下四个 `?` 是同一个 case 的连续交互，环境上下文共享。

#### 问题 1

```python
>>> x = 20
>>> x + 2
?
```

**答案：22**

x 绑着 20，表达式 `x + 2` 求值为 22。expression 只产出值，不改变环境。

---

#### 问题 2

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

#### 问题 3

```python
>>> y = 5
>>> y = y + 3
>>> y * 2
?
```

**答案：16**

`y = y + 3` 是 statement，执行规则：先 evaluate 右边 `y + 3`（此时 y 还是 5，5 + 3 = 8），再把 y 重绑到 8。然后 `y * 2` 是 expression，用新值 8 乘 2 得 16。

---

#### 问题 4

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

### WWPD 考点（对接 Lec01&02）

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

## Part 2: Implementing Functions

文件：`lab00.py`

### twenty_twenty_four()

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
return 2024  # 最直接，但不够"creative"
# 或者
return 2 * 1000 + 24
# 或者
return 2025 - 1
```

---

## Part 3: Running Tests

```bash
python ok              # 跑全部测试
python ok -q python-basics -u   # 单独跑 WWPD（带 -u 解锁）
python ok -q twenty_twenty_four  # 单独跑函数测试
```

Windows 下如果 `python3` 不通，用 `python` 或 `py`。
