# Lab / HW / Disc / Project 笔记统一模板

> **唯一权威模板**。新建任何作业笔记都先复制本文件的对应骨架，不得自拟章节名与标题层级。
> 规则来源：[CLAUDE.md](file:///d:/myfile/MY_CS61A/CLAUDE.md) 的「HW/Lab/Disc 独立闭环」「Project 独立闭环」。
> 目标：让每一份笔记的 **Obsidian 大纲（Outline）长得一样**——同一层级永远是同一类内容。

---

## 1. 命名与存放位置

| 类型 | 文件名 | 位置 |
|---|---|---|
| Lab | `Lab <N> - <官方标题>.md` | `MY/CS61A-Assignments/Lab/labNN/` |
| HW | `Homework <N> - <官方标题>.md` | `MY/CS61A-Assignments/HW/hwNN/` |
| Disc | `Discussion <N> - <官方标题>.md` | `MY/CS61A-Assignments/Disc/` |
| Project | `Project <N> - <名称>.md` | `MY/CS61A-Assignments/Project/<name>/` |

- 标题必须取对应 `index.html` 的官方 H1
- Windows 非法字符 `:` 替换为 ` - `
- 例：`Lab 2 - Higher-Order Functions.md`

---

## 2. Frontmatter（固定字段）

```yaml
---
type: lab            # lab / hw / disc / project
id: "02"             # 两位编号
title: Higher-Order Functions, Lambda Expressions
status: in_progress  # in_progress / complete
source: SOURCE/CS61A-Website-Archive/lab/lab02/index.html
solution: SOURCE/CS61A-Website-Archive/lab/sol-lab02/index.html
code: MY/CS61A-Assignments/Lab/lab02/lab02.py
lectures: Lec04
due: 2024-09-11
---
```

- `solution`：Lab/HW/Disc 写官方解答路径；Project 写 `none`（没有官方参考实现）
- `code`：Disc 没有代码文件时写 `none`
- `lectures`：多个用逗号分隔，如 `Lec03, Lec04`

---

## 3. 标题层级（逐级照抄官方页面）

```
H1  官方标题（全文唯一）
└─ H2  官方一级区块：Topics / Required Questions / Check Your Score Locally …
   └─ H3  官方二级分组：What Would Python Display? / Coding Practice / Short Circuiting …
      └─ H4  题目：Q1、Q2……（官方放在 H3 层的题就直接用 H3）
         └─ 题内固定小节用加粗标签，不占标题
```

硬性要求：

- **标题层级逐级照抄官方 index.html**：官方是 H2 就 H2，是 H3 就 H3，是 H4 就 H4；顺序与名称都不改
- 允许加中文括注（如 `## Topics（前置概念）`），但括号外必须是官方原名
- **二级分组必须保留**：`What Would Python Display?`、`Coding Practice` 等不能省略
- 官方没有的区块只出现在文档首尾：开头 `## 总起`，结尾 `## 验收`、`## 本次踩的坑`、`## 相关概念`
- 纯行政区块（`Submit Assignment`、问卷链接）整块省略
- 全文只能有一个 H1
- 题内固定小节（原题/中文/答案/验证/思路…）只用加粗标签，不占标题层级
- 题号用官方编号，不重排

---

## 4. 通用骨架（Lab / HW / Disc）

````markdown
---
type: lab
id: "02"
title: <官方标题>
status: in_progress
source: SOURCE/CS61A-Website-Archive/lab/lab02/index.html
solution: SOURCE/CS61A-Website-Archive/lab/sol-lab02/index.html
code: MY/CS61A-Assignments/Lab/lab02/lab02.py
lectures: Lec04
due: 2024-09-11
---

# <官方标题>

- **题目来源**：`SOURCE/CS61A-Website-Archive/lab/lab02/index.html`
- **官方解答（仅卡死时看）**：`SOURCE/CS61A-Website-Archive/lab/sol-lab02/index.html`
- **Starter 基线**：`SOURCE/CS61A-Assignments_SOURCE/Lab/lab02/lab02.py`
- **我的代码**：`MY/CS61A-Assignments/Lab/lab02/lab02.py`
- **关联讲次**：Lec04 · Higher-Order Functions（教材 Section 1.6）
- **Due**：2024-09-11

## 总起

**一句话定位**：<这份作业在整条知识线上解决什么>

**覆盖知识点**

- <知识点 1>
- <知识点 2>

**题目地图**

| 题号 | 题名 | 类型 | 必做 |
|---|---|---|---|
| Q1 | <题名> | WWPD | 是 |
| Q4 | <题名> | 编程 | 是 |
| Q7 | <题名> | 编程 | 否 |

**对应讲义**：LecXX · <标题>，教材 Section X.X

## Starter Files

<官方区块原文：下载链接与需要下载的文件>

## Topics（前置概念）

<官方 Topics 原文；逐小节展开，H3 用官方小节名>

### <官方概念名>

**原文**

<官方英文原文，普通正文，任意长>

> **中文**：<一句话说明，不逐句翻译>

## Required Questions

### What Would Python Display?

#### Q1: <官方题名>

**原题（English）**

<官方英文原文，逐段保留，Notes / Hint / 限制条件一个不删>

> **中文**：<只翻要求与限制，一句话到两句话>

**Starter**

```python
<starter 原样代码；填空题或需要展示签名时保留，否则省略此段>
```

**答案**

```python
<完整、可直接运行的实现；删除 *** YOUR CODE HERE *** 等模板注释>
```

**验证**

```bash
python ok -q <测试名>
```

<通过结果，或列出 doctest 期望值>

**思路**

- <关键推导步骤 1>
- <关键推导步骤 2>

**关联知识点**

- <对接哪个讲次/教材概念，为什么>

**易错点**

- <这一题特有的坑>

### Coding Practice

#### Q4: <官方题名>

<结构同 Q1；同样按官方页面的分组与标题级别放置>

## Check Your Score Locally

```bash
python ok --score
```

<记录：已通过题号、未做题号、分数>

## Environment Diagram Practice

<仅题目页有该区块时保留，没有就整节省略>

### Q6: <官方题名>

<纸笔题可只保留「思路」「易错点」>

## Optional Questions

### Q7: <官方题名>

<结构同上；未做时写「（做完再补）」>

<记录：已通过题号、未做题号、分数>

## 本次踩的坑

- <通用坑 → 是否补进 `MY/Knowledge/Common_Pitfalls.md`>

## 相关概念

- [[<关联讲次笔记>]] — <一句话关系>
- [[<同主题 HW/Lab/Disc 笔记>]] — <一句话关系>
````

---

## 5. 题目级结构（Q 级）

所有题型共用同一套「题目标题 + 加粗标签」，按题型二选一，不新造结构。

### 5.1 写代码题

````markdown
#### Q4: <官方题名>

**原题（English）**

<官方英文原文，Notes / Hint / 限制条件不删>

> **中文**：<一句话>

**Starter**（可选）

```python
<starter 原样代码>
```

**答案**

```python
<完整实现>
```

**验证**

```bash
python ok -q <测试名>
```

<通过结果>

**思路**

- <推导过程；跨题的取舍也写在这里>

**关联知识点**

- <对接的讲次/教材概念>

**易错点**

- <这一题特有的坑>
````

### 5.2 WWPD 题

````markdown
#### Q1: WWPD: <官方题名>

**原题（English）**

<官方英文原文>

> **中文**：<一句话>

```bash
python ok -q <测试名> -u
```

```python
>>> True and 13
13
```

**说明**

- <为什么是这个结果>

**思路**（可选）

- <同类题的判断顺序>

**关联知识点**（可选）

- <对接概念>

**易错点**（可选）

- <容易误判的点>
````

- 答案写进代码块 `?` 位置，或用「表达式 | 输出 | 原因」表格
- `说明` 必写；`思路 / 关联知识点 / 易错点` 按需选写
- 跨题的「关键设计决策」不单列，统一并进对应题的 `**思路**`
- Project 的题内结构同 5.1，标题层级按官方项目页（Hog 是 Phase 1 / Phase 2），不另设骨架
- 题目标题级别跟随官方页面（在 `What Would Python Display?` 下就是 H4）
- 页面源码中被 HTML 注释掉的 WWPD 不写入笔记

---

## 6. 写作规范

1. **英文原文用普通正文，中文翻译用引用块**：`> **中文**：…`，避免英文被引用块缩小字号。
2. **不预先写答案**：用户没做完的题只保留题目与「（做完再补）」；做完一题补一题，答案必须来自用户实际写并通过 ok 的代码。
3. **只留自学价值内容**：破冰、签到、行政通知、问卷链接整段删除。
4. **结构以题目页为准**：`SOURCE/CS61A-Website-Archive/` 下对应 `index.html` 是唯一结构来源；`sol-*` 只用于卡死时对答案，不作为章节依据。
5. **官方内容不删减**：Notes、Hint、限制条件、doctest 全部保留。
6. **表格转义**：单元格里的 `|` 写成 `\|`，否则表格错位。
7. **代码块标语言**：Python 用 ` ```python `，命令用 ` ```bash `。
8. **通用坑才升级**：同一坑在多题出现，才补进 `MY/Knowledge/Common_Pitfalls.md`；单题特有的坑留在该题「易错点」。

---

## 7. 新会话开工检查

- [ ] 已读本模板，并按对应骨架建文件
- [ ] Frontmatter 字段齐全
- [ ] H1 唯一，H2/H3/H4 与官方页面同名同序（含二级分组）
- [ ] 每题都有「原题（English）」和「> **中文**：」
- [ ] 未做的题没有预填答案
- [ ] 验收结果已记录
