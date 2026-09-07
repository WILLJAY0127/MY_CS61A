---
lecture: 01&02
title: Welcome & Functions
status: complete
---

# Lecture 01&02 · 两条线在 Abstraction 汇合

> **Core Question**: 教材为什么把"求值(expression)"和"执行(statement)"分两条线讲?它们在哪汇合?为什么教材结尾说 function、data、object、interpreter "are really not so distinct"?

## 📎 资料清单

- **PPT**: `assets/slides/01-Welcome_1pp.pdf`、`assets/slides/02-Functions_1pp.pdf`
- **Video**: Lec02 · https://www.youtube.com/watch?v=gNv81_4X0uU&list=PL6BsET-8jgYULSxiV2garZ0FxbnXR08MP&index=1
- **教材**(小节级,已读正文核实):
  - Ch. 1.1 Getting Started — `pages/11-getting-started.html`(1.1.1 / 1.1.3 / 1.1.4 / 1.1.5)
  - Ch. 1.2 Elements of Programming — `pages/12-elements-of-programming.html`(1.2.1-1.2.6)
  - Ch. 1.3 Defining New Functions — `pages/13-defining-new-functions.html`(1.3.1-1.3.4)
- **字幕**: 字幕 Lecture 02(Lec01 无独立字幕,内容落在 2.1) — 2.1 Welcome / 2.2 Names & Assignment / 2.3 Environment Diagrams / 2.4 Function Definition / 2.5 Print & None
- **代码**: `assets/slides/02.py`
- **配套**: Disc00 / Lab00(Due Wed 9/4)、HW01(Due Mon 9/9;独立闭环)

---

## 起点:程序的二元根本(教材 1.1.4)

教材开篇就定调:程序由两类指令组成——

> ① **Compute some value** ② **Carry out some action**

对应两种语言元素:
- **Expression(表达式)**:被 *evaluate*(求值),产出 value
- **Statement(语句)**:被 *execute*(执行),产生 change(改环境、改流程)

教材 1.2.5 原文把这条边界说得很硬:"statements are not evaluated but executed; they do not produce a value but instead make some change."

**这不是中途的张力,是整本书的二元骨架。** 后面所有的概念都长在这两条线上,并在某个点汇合。

---

## Expression 侧:从 primitive 到嵌套组合

### 第一步:primitive expressions(叶子)

教材 1.2.1 开宗明义:primitive expression 只有两类——
- **数字字面量** `42` —— 求值规则:值就是自身
- **名字** `pi`、`add` —— 求值规则:去 environment 查 binding

注意:名字绑的可以是 data,也可以是 function(`f = max`)。**这是 function vs data 张力的第一颗种子,在叶子层就埋下了。**

### 第二步:call expression(组合方式)

教材 1.2.2:表达式可以用运算符组合,但最重要的组合形式是 **call expression** —— `operator(operand1, operand2, ...)`。

**求值规则是 recursive**:
1. 先 evaluate operator 和所有 operand subexpressions
2. 再 apply function to arguments

每个 operand 自己可能又是 call expression,继续套同样规则。终止于 leaf(数字/名字)。

教材给出 expression tree 的可视化——自底向上求值,叶子先算,根最后算。

call expression 相比中缀的三个优势(教材 1.2.2 原文):① 任意参数数 `max(1,-2,3,-4)` ② 嵌套结构显式 `max(min(1,-2), min(pow(3,5),-4))` ③ 统一所有运算符为 named function。

### Expression 侧的矛盾

拼完就没了——`max(2,3)` 算完得到 3,这个 3 没有名字,下次要用还得重算。**combination 是临时的,无法保存复用。**

→ 教材 1.2.4 转向 statement 侧:**assignment**(给值命名,作为 abstraction 的最简形式)。

但这里有个关键:**assignment 是 statement,不是 expression**。教材 1.2.5 明确指出 `x=3` 不走 evaluate 规则,它走 execute 规则,目的是 bind name to value,不产出 value。

所以教材的结构是**两条线**:expression 侧讲完 combination,矛盾出现(没法保存),自然转到 statement 侧讲 abstraction。

---

## Statement 侧:从 assignment 到 def(两条 abstraction)

### Abstraction 第一层:assignment(教材 1.2.4,limited)

`name = expression` 的 execute 规则:
1. 先把 `=` 右边**所有**表达式 evaluate 完,得到具体 value
2. 再把左边 name bind 到这个 value

**不同步现象**(教材 1.2.4 原例):
```python
radius = 10
area = pi * radius * radius   # area 绑 314.159 这个 value
radius = 11
area                          # 还是 314.159!不是 380.13
```

教材原文:"Changing the value of one name does not affect other names."——因为 assignment 绑的是**值**,不是**关系**。

**多赋值** `x, y = 3, 4.5`:all right evaluated before any left bound → 一行交换 `y, x = x, y`。

教材自己点明:assignment 是 "**simplest** means of abstraction"。**limited** 在哪?绑值不绑关系 → 不同步 → 改了 radius 想让 area 跟着走,assignment 做不到。

### Abstraction 第二层:def(教材 1.3,powerful)

```python
def square(x):
    return mul(x, x)
```

**def 与 assignment 本质同构**——教材 1.3 原文:"Both def statements and assignment statements bind names to values"。区别在 value 的粒度:assignment 绑数据值,def 绑**整个运算流程**(return expression 藏起来,调用时重算 → 解决不同步)。

def 执行时(教材 1.3 原文 "return expression is **not evaluated right away**; it is **stored** as part of the newly defined function"):
1. 创建新 function object(签名 = def 到冒号那行)
2. 把 return expression **藏起来**(squirrel)作为 function 的一部分
3. bind 函数名到 function object

调用 user-defined function 时(教材 1.3.2):
1. 新建 **local frame**
2. 形参 bind 到实参值
3. 在新 environment 里 execute 函数体

每次调用都重算 → area 定义成函数就能跟着 radius 走。**def 是更强的 abstraction,正好解决 assignment 的不同步。**

### Statement 侧的矛盾

函数体执行时,内部用什么 environment 查名字?形参名 `square` 和全局函数名 `square` 冲突怎么办?`f = max` 之后 `max = 5` 再调用 `max(2,3,4)` 为什么报错,但 `f(2,3,4)` 还能用?

→ 教材 1.3.1 被迫给出 **environment 的精确定义**。

---

## 枢纽:Environment(两条线在这里交汇)

Environment 不是独立一段,它是**被迫两次引出**的概念,连接 expression 侧和 statement 侧:

| 出处 | 为什么被迫定义 | 定义 |
|---|---|---|
| 1.2.4 | assignment 需要存 binding | interpreter 的"记忆",记录 names/values/bindings |
| 1.2.5 | 名字求值时要去哪查 | 给符号赋予含义(脱离环境,`add(x,1)` 毫无意义) |
| 1.3.1 | 形参名和全局名冲突怎么解决 | **sequence of frames**;每 frame 含 bindings;有 single global frame |
| 1.3.2 | def 调用内部用什么环境 | 新建 local frame + 形参绑实参 + 在新 environment execute 函数体 |

**两条线在 environment 交汇**:
- Expression 侧要 environment —— 名字求值靠它给含义(1.2.5)
- Statement 侧改 environment —— assignment 和 def 都改 binding(1.2.4、1.3.2)

### Name lookup rule(教材 1.3.1)

从当前帧开始,依次向前查找,找到第一个 binding 就用。

→ 解决 `def square(square): return mul(square, square)` + `square(2)` 为什么不出错:调用时新建 local frame,形参 square 绑到实参 2,查找 square 时 local 帧先找到 2,根本不看 global 帧的 square 函数。这就是 **local name 屏蔽 global**。

### Intrinsic name vs bound name(教材 1.3.1)

function 有 intrinsic name(函数自身名)和 bound name(帧里绑的名)。`f = max` 后 f 和 max 是两个 bound name 指向同一函数;**intrinsic name 不参与求值**。

→ 解决 `f = max; max = 5; f(2,3,4)` 为什么还能调用原 max:max 这个 bound name 被重绑到 5,但 f 还指着原来的 function object;intrinsic name 是函数自身的,查找时不用它。

---

## 汇合点:print(expression 调用产生 statement 效果)

教材 1.2.6 引入 print,**这是两条线的汇合点** —— 一个 expression 调用(call expression),却产生了 statement 的效果(side effect)。

### Pure vs non-pure(教材 1.2.6)

- **Pure function**(`abs`):输入→输出,封闭管道,无 side effect,可嵌套组合 `max(min(...), min(...))`
- **Non-pure function**(`print`):返回 `None` + side effect(显示内容)

**`print(print(1), print(2))`** 推导(教材 1.2.6 原例):

求值规则没变(先 evaluate operator 和 operands 再 apply),但 operand 求值时 side effect 发生:

```
1. evaluate operator: print 函数
2. evaluate operand1 = print(1):
   - side effect: 显示 1
   - 返回值: None
3. evaluate operand2 = print(2):
   - side effect: 显示 2
   - 返回值: None
4. apply print(None, None):
   - side effect: 显示 "None None"
   - 返回值: None(交互解释器默认不显示 None)
```

屏幕依次出现:`1` `2` `None None`。最后那个 None 不显示,因为 interpreter 默认不显示 None。

教材点明:"Pure functions can be composed more reliably into compound call expressions"——**这是后面所有 function 设计的根**。print 不可靠嵌套,正因为它跨界:作为 expression 被求值,却产生 statement 的 side effect。

**statement vs expression 这条起点张力,在 print 这里爆发。**

---

## 收束:三机制堆出 abstraction,但 function/data/object/interpreter 终将统一

教材 1.2 开篇给全书骨架:任何强大语言有 **three means**——
1. **primitive expressions and statements** —— 最简构建块
2. **means of combination** —— 从简单构建复合
3. **means of abstraction** —— 给复合元素命名,作为单元操作

并处理 **two elements**(function & data),但"**Soon we will discover that they are really not so distinct.**"

本讲对应到 three means:
- Primitive:数字、名字、内置 function(`max`/`add`/`mul`)
- Combination:中缀表达式、call expression、嵌套、`import`
- Abstraction:assignment(limited)→ def(powerful)

但教材 1.3 结尾用 **"some of"** 预留口子——"We have identified in Python **some of** the elements that must appear in any powerful programming language"。这是阶段性小结,不是全集。后续会引入 boolean、sequence、object、control flow 等更多 primitive、combination、abstraction 手段。

**终极预告**(教材 1.1.4 结尾):"functions are objects, objects are functions, and interpreters are instances of both"——整本书都在论证 function、data、object、interpreter 的最终统一。本讲已经埋下种子:
- `f = max` —— function 像 data 一样是 value,可绑名字
- `max = 5` 后 max 不能调用 —— 名字绑 data 或 function 本质都是 binding
- def 与 assignment 同构 —— 都 bind name to value

后续 Lec04 HOF(function 当参数/返回值,function 作为 data 流动)、Lec05 environment 深入、Lec29-32 interpreter,都在展开这条统一主线。

---

## 我的理解(Jace 推导链,保留原话)

> 来源:`MY/思维片段/Week1&2/`。闭包/一等公民/语言流派对比属 Lec04-05,不收进本讲。

**1. 组合 ≠ 抽象(边界判据)**

Jace 原话:"组合方式是一种内部构建,抽象是一种外部命名么?""我觉得组合就抽象了""组合就是用表达式和语句来操作某某东西,这是基本。然后抽象就是对于某些需要的东西进行隔离,划定边界,进而才能进行复用"

推导:直觉"拼起来=抽象" → 修正为"组合是拼装,抽象是拼装后**加边界、隐藏细节**"。判据:**修改内部实现,外部调用代码要不要改?** 要改=没抽象,不改=有抽象。

**2. 叶子 ≠ 只是数字**

Jace 原话:"如何区分一个节点是不是最终不可求节点,是判断一个表达式的表达数是不是真正的数字吧"

推导:以为叶子=数字 → 修正为叶子有两类(数字字面量**和名字**)。`add`、`pow` 这种 function name 也是 leaf,绑的值是 function object。口诀:看见 `xxx(...)` 就是 internal node,其余都是 leaf。

**3. 环境根本作用是赋予含义**

Jace 原话:"环境似乎也是抽象隔离的一种?"

推导:洞见 environment 的隔离性(每次调用新建独立 local frame)→ 自己修正:environment 最根本是**给标识符赋予含义**(脱离环境 `add(x,1)` 毫无意义),隔离只是衍生能力。这条直接对接教材 1.2.5 和 1.3.1。

**4. 作者用 "some of" 预留口子**

Jace 原话:"组合方式有很多。抽象方式有很多 基本元素应该也不止他说的这些"

推导:觉得教材列的要素不全 → 理解原文 "some of" 是阶段性小结,不是全集。后续会引入 boolean、sequence、object、control flow。

**5. Python 是"通用通道"流派**

Jace 原话:"我现在觉得 py 在故意混淆俩个东西,如果我设计一个新的语句定义,我是不是就能给函数作为入参和数据作为入参给剥离开了"

推导:感觉 Python 用同一套语法通道承载 data 和 function 两种语义 → 理解这是语言设计主动取舍(Python/JS/Scheme 选通用通道,C/Java 选隔离通道)。这条对接 Lec04 HOF 和 Lec29 Scheme。

---

## Java 桥梁

| Python | Java | 桥梁说明 |
|---|---|---|
| `radius=10; area=pi*r*r` 不同步 | `int r=10; int area=r*r;` | Java 一模一样,绑值不绑关系。但 Java 没画 environment diagram |
| `def square(x): return mul(x,x)` 函数体不立即执行 | `int square(int x){return x*x;}` 方法体也不立即执行 | 都是"定义时藏起来,调用时才算"。Python 用 environment diagram 把 local frame 可视化,Java 没有 |
| Function 是 first-class(`f=max`) | Java 8 前方法必须依附 class;Lambda 是 FunctionalInterface 的语法糖 | Python/Scheme 通用通道流派,Java 隔离通道流派。这解释 Java 写 HOF 要包接口 |
| `print` non-pure(返回 None + 副作用) | `System.out.println` 返回 void + 副作用 | 都是非纯函数。Java 显式 `void`,Python 用 `None` 隐式 |
| Environment(帧 + binding) | JVM stack frame + 局部变量表 | 本质同源,都是 name→value 绑定表。Python 把 environment 作为一等概念教,Java 藏在字节码层 |

---

## 纠错与突破

**误区1:组合 = 抽象** —— 组合只是拼装,抽象是拼装后加边界隐藏细节。判据:改内部实现,外部要不要改。来源:Jace `1.md`、`想法3.md`

**误区2:叶子 = 数字** —— 叶子有两类:数字字面量和名字。function name 也是 leaf。来源:Jace `1.2.5.md`

**误区3:作者在穷举语言要素** —— 原文 "some of",是阶段性小结。来源:Jace `1.3.md`

**突破** —— Jace 自己推导出的"修改内部实现,外部要不要改"这条抽象判据,比教材讲得更准。直接对应 Lec04 function design 和后续所有 design pattern 的"依赖倒置"。

---

## 相关概念

- [[Lec03 - Control]] — 新的 combination means(if/while:分支/迭代),扩展本讲的 expression combination
- [[Lec04 - Higher-Order Functions]] — function 作为 combination 的对象,正是 function vs data 张力的展开
- [[Lec05 - Environments]] — environment 枢纽的深入,多帧环境模型
- [[HW01 - Functions, Control]] — 配套作业,独立闭环
- [[Lab00 - Getting Started]]、[[Disc00 - Getting Started]] — 配套,独立闭环
- [[Common_Pitfalls]] — 本讲通用坑:组合≠抽象、leaf≠数字、assignment 绑值不绑关系、def 不立即执行

---

## 复习卡

### R1 三机制是什么
<details><summary>primitive / combination / abstraction 分别指什么?教材为什么说 function & data "not so distinct"?</summary>
primitive = 最简构建块(数字、名字、内置函数);combination = 从简单构建复合(call expression、嵌套、import);abstraction = 给复合元素命名作为单元(assignment、def)。"not so distinct" 因为 function 也能绑名字(f=max)、当 value 流动,def 和 assignment 本质同构(都 bind name to value)。
</details>

### R2 call expression 求值规则
<details><summary>evaluate `sub(pow(2, add(1,10)), pow(2,5))` 的步骤?为什么是 recursive?</summary>
先 evaluate operator 和所有 operands,再 apply。每个 operand 自己可能是 call expression,继续套同样规则,所以 recursive。leaf(数字/名字)终止递归:数字值是自身,名字去 environment 查 binding。自底向上算 expression tree。
</details>

### R3 assignment 不同步
<details><summary>`radius=10; area=pi*radius*radius; radius=11` 后 area 是多少?为什么?</summary>
还是 314.159。assignment 规则:先把右边所有表达式算完得到具体值,再把名字绑到值。area 绑的是 314.159 这个值,不是"π×r²"这个关系。改 radius 不影响已绑定的 area。
</details>

### R4 def 执行 vs 调用
<details><summary>`def square(x): return mul(x,x)` 执行时乘法算了吗?调用 `square(-2)` 时发生什么?</summary>
def 执行时没算。def 做三件事:①创建 function object ②把 return expression 藏起来(squirrel)作为 function 的一部分 ③bind 名字到 function object。调用时:①新建 local frame ②形参 x 绑到实参 -2 ③在新 environment execute 函数体 → mul(-2,-2)=4。
</details>

### R5 local name 屏蔽
<details><summary>`def square(square): return mul(square,square)` + `square(2)` 为什么不出错?</summary>
调用时新建 local frame,形参 square 绑到实参 2。Name lookup rule:从当前帧开始找,local 帧先找到 square=2,就用它,根本不看 global 帧的 square 函数。这就是 local name 屏蔽 global。
</details>

### R6 statement vs expression
<details><summary>两者的根本区别?各举三例。为什么说这是整本书的二元骨架?</summary>
Expression:evaluate(求值),产出 value。例:`42`、`pi`、`max(2,3)`。Statement:execute(执行),产生 change。例:`radius=10`(改环境绑定)、`import`(绑名字)、`def`(绑函数名)。教材 1.1.4:程序由 compute value 和 carry out action 两类指令组成;1.2.5 原文 "statements are not evaluated but executed"。后续所有概念都长在这两条线上,在 print 处汇合。
</details>

### R7 pure vs non-pure
<details><summary>`print(print(1), print(2))` 输出?为什么最后 None 不显示?为什么这是两条线的汇合点?</summary>
依次输出 `1` `2` `None None`。求值规则没变(先算 operator 和 operands 再 apply),但 operand 求值时 side effect 发生:print(1) 副作用显示 1 + 返回 None;print(2) 副作用显示 2 + 返回 None;外层 print(None,None) 副作用显示 "None None" + 返回 None。最后那个 None 不显示因为 interpreter 默认不显示 None。汇合点:print 作为 expression 被求值,却产生 statement 的 side effect,两条线在这里跨界。
</details>

### R8 environment 四次递进定义
<details><summary>environment 在 1.2.4 / 1.2.5 / 1.3.1 / 1.3.2 分别被定义成什么?为什么说它是两条线的交汇?</summary>
1.2.4:interpreter 的"记忆"(names/values/bindings 的存储);1.2.5:给符号赋予含义(脱离环境 add(x,1) 毫无意义);1.3.1:sequence of frames(每 frame 含 bindings,有 single global frame);1.3.2:调用 user-defined function = 新建 local frame + 形参绑实参 + 在新 environment execute 函数体。交汇:expression 侧要 environment(名字求值靠它给含义),statement 侧改 environment(assignment 和 def 都改 binding)。
</details>

### R9 组合 vs 抽象判据
<details><summary>一段代码拼在一起但变量全局可见,算抽象吗?判断标准?</summary>
不算,只是 combination。判据:修改内部实现,外部调用代码要不要改?要改=没抽象,不改=有抽象。Abstraction = Combination + 边界 + 隐藏细节。
</details>

### R10 assignment vs def
<details><summary>两者都属于 abstraction means,本质区别?为什么 def 是 powerful 而 assignment 是 limited?</summary>
都 bind name to value,本质同构(教材 1.3 原文)。区别在 value 的粒度:assignment 绑的是**数据值**(limited,绑值不绑关系→不同步);def 绑的是**整个运算流程**(powerful,return expression 藏起来,调用时重算→解决不同步)。def 解决了 assignment 的局限。
</details>

### R11 intrinsic name vs bound name
<details><summary>`f = max; max = 5; f(2,3,4)` 为什么还能调用原 max?</summary>
function 有 intrinsic name(函数自身名)和 bound name(帧里绑的名)。f 和 max 是两个 bound name 指向同一 function object。max = 5 后 max 这个 bound name 重绑到 5,但 f 还指着原来的 function object。intrinsic name 不参与求值(查找时用 bound name),所以 f(2,3,4) 还能调用原 max,max(1,2) 报错 "int object is not callable"。
</details>
