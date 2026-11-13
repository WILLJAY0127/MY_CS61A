---
type: lab
id: "00"
title: Getting Started
status: complete
source: SOURCE/CS61A-Website-Archive/lab/lab00/index.html
solution: SOURCE/CS61A-Website-Archive/lab/sol-lab00/index.html
code: MY/CS61A-Assignments/Lab/lab00/lab00.py
lectures: Lec01
due: TBD
---

# Lab 0: Getting Started

- **题目来源**：`SOURCE/CS61A-Website-Archive/lab/lab00/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/lab/sol-lab00/index.html`
- **Starter 基线**：`SOURCE/CS61A-Website-Archive/lab/lab00/lab00.zip`
- **我的代码**：`MY/CS61A-Assignments/Lab/lab00/lab00.py`
- **关联讲次**：Lec01 · Functions
- **Due**：TBD

---

## 总起

**一句话定位**：Lab 0 是环境搭建课，核心只有一件事——分清 expression 和 statement，以及 assignment 是怎么改环境的。

**覆盖知识点**

- expression vs statement：一个走 evaluate 产出值，一个走 execute 改变环境
- assignment 两条规则：先算右边再绑左边；绑值不绑关系
- primitive expression 与算术表达式：`2024`、`2 * 1000 + 24` 的求值方式
- 工具链：`python ok` 各参数、`python -i`、`python -m doctest`

**题目地图**

| 区块 | 内容 |
|---|---|
| Starter Files | 下载 lab00.zip |
| Introduction / Setup | Python 安装（略） |
| Your First Assignment | 1) WWPD｜2) twenty_twenty_four()｜3) Running Tests |
| Task C | 提交 Gradescope（略） |
| Appendix | Python 命令行选项 |

**对应讲义**：Lec01 · Functions

---

## Starter Files

下载 lab00.zip：`SOURCE/CS61A-Website-Archive/lab/lab00/lab00.zip`

---

## Your First Assignment

### 1) What Would Python Do? (WWPD)

```bash
python ok -q python-basics -u
```

**Case 1（四个问题共用同一个环境）**

```python
>>> x = 20
>>> x + 2
22                      ← 问题 1：expression 只产出值，不改环境
>>> x + 2
22
>>> x
20                      ← 问题 2：x 没变，expression 没有修改环境的能力
>>> y = 5
>>> y = y + 3
>>> y * 2
16                      ← 问题 3：assignment 先算右边（y 还是旧值 5→8），再绑左边
>>> x = 20
>>> y = 5
>>> y = y + 3
>>> y + x
28                      ← 问题 4：x=20（没被改），y=8（被重绑），加起来 28
```

**说明**

- `x + 2` 是 expression，走 evaluate 规则产出 22，**不修改任何名字绑定**
- `y = y + 3` 是 assignment statement，走 execute 规则：先算右边得 8，再把 y 重绑到 8
- 两条核心规则交叉验证：expression 不改环境，statement 改环境

**关联知识点**

**expression vs statement 的核心区别**

| | expression | statement |
|---|---|---|
| 走什么规则 | evaluate（求值） | execute（执行） |
| 产出什么 | value | 无返回值，改环境 |
| 本 case 例子 | `x + 2`、`y * 2`、`y + x` | `x = 20`、`y = y + 3` |

**assignment statement 两条规则**

1. **先算右边再绑左边**——`y = y + 3` 算右边时 y 还是旧值 5
2. **绑值不绑关系**——`x + 2` 不改变 x，要改 x 必须再写一条 assignment

**易错点**

- 以为 `y = y + 3` 算右边时 y 已经是新值 → 错，evaluate operands 时不触发 execute
- 以为 `x + 2` 会改 x → 只有 assignment 才改环境
- 把 `y + x` 里的 y 当成 5 而不是 8 → `y = y + 3` 已经把 y 重绑到 8

---

### 2) Implementing Functions

#### Q1: twenty_twenty_four

**原题（English）**

Come up with the most creative expression that evaluates to 2024 using only numbers and the +, *, and - operators.

> **中文**：只用数字和 `+` `*` `-` 三种运算符，return 一个值为 2024 的表达式。

**Starter**

```python
def twenty_twenty_four():
    """Come up with the most creative expression that evaluates to 2024
    using only numbers and the +, *, and - operators.

    >>> twenty_twenty_four()
    2024
    """
    return ______
```

**答案**

```python
return 2024                   # 最直接
# 或者
return 2 * 1000 + 24
# 或者
return 2025 - 1
```

**思路**

这题没有"正确"解法——只要 evaluate 到 2024 就行。数字 `2024` 是 primitive expression（直接求值为自身），`2 * 1000 + 24` 是算术表达式（同样走 evaluate 规则递归求值）。

**关联知识点**

- primitive expression：数字 `2024` 是 primitive（叶子），直接求值为自身
- 算术表达式 vs call expression：`2 * 1000 + 24` 是算术中缀记法，不是 call expression，但同样走 evaluate 规则

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
python lab00.py                    # 直接运行文件，执行完退出
python -i lab00.py                 # -i 交互模式，执行完进入 >>> 提示符
python -m doctest lab00.py         # 运行 docstring 里的 doctest 测试
```

**注意**：interactive 模式下修改了 .py 文件，需要 `exit()` 重新启动才能生效。

---

## Check Your Score Locally

```bash
python ok --score
```

（全对）

---

## 本次踩的坑

（这题太基础，没踩坑）

## 相关概念

- [[Lab 1 - Functions]] — return vs print、doctest 用法、digit 提取
- Lec01 · Functions（`MY/Weekly/`，待写）— expression vs statement、assignment 规则
