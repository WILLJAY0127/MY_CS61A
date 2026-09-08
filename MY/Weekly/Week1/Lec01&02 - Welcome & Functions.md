---
lecture: 01&02
title: Welcome & Functions
status: complete
---

# Lecture 01&02 · Three Means —— 从组合到抽象的框架

> **Core Question**: 教材 1.2 开篇说"every powerful language has three mechanisms"。这三机制如何串起整章?它们在哪交汇?为什么教材结尾说 function、data、object、interpreter "are really not so distinct"?

## 📎 资料清单

- **PPT**: `assets/slides/01-Welcome_1pp.pdf`、`assets/slides/02-Functions_1pp.pdf`
- **Video**: Lec02 · <https://www.youtube.com/watch?v=gNv81_4X0uU&list=PL6BsET-8jgYULSxiV2garZ0FxbnXR08MP&index=1>
- **教材**(小节级):
  - Ch. 1.1 Getting Started — `pages/11-getting-started.html`(1.1.1 / 1.1.3 / 1.1.4 / 1.1.5)
  - Ch. 1.2 Elements of Programming — `pages/12-elements-of-programming.html`(1.2.1-1.2.6)
  - Ch. 1.3 Defining New Functions — `pages/13-defining-new-functions.html`(1.3.1-1.3.4)
- **字幕**: 字幕 Lecture 02(Lec01 无独立字幕,内容落在 2.1) — 2.1 Welcome / 2.2 Names & Assignment / 2.3 Environment Diagrams / 2.4 Function Definition / 2.5 Print & None
- **代码**: `assets/slides/02.py`
- **配套**: Disc00 / Lab00(Due Wed 9/4)、HW01(Due Mon 9/9;独立闭环)

***

## 基调:Three Means(教材 1.2 开篇)

教材 1.2 第一段才是真正的定调处。原文:

> "A programming language is more than just a means for instructing a computer to perform tasks. The language also serves as a **framework within which we organize our ideas** about computational processes... When we describe a language, we should pay particular attention to the means that the language provides for combining simple ideas to form more complex ideas. **Every powerful language has three such mechanisms**:
> - **primitive expressions and statements**, which represent the simplest building blocks
> - **means of combination**, by which compound elements are built from simpler ones
> - **means of abstraction**, by which compound elements can be named and manipulated as units"

注意教材用的三个词:**framework**(框架)、**organize our ideas**(组织思想)、**Every powerful language**(所有强大语言通用)。这是纲领性陈述,不是局部说明。

教材同时预告全书核心张力——"two kinds of elements: functions and data. (**Soon we will discover that they are really not so distinct.**)"——这条"function vs data 终将统一"的主线,从基调段就埋下种子。

> **关于 1.1.4 的定位**:教材 1.1.4 "First Example" 自己点明这是 **"sneak preview"**(预览),原文:"In the next section, we will **start from scratch** and build up the language piece by piece." 1.1.4 里那句 "computer programs consist of instructions to either ① Compute some value ② Carry out some action" 只是镶嵌在 Shakespeare 示例里的随手说明,不是教材的纲领性陈述。真正的二元划分要到 1.2.5 讲嵌套求值规则时才落地(见下文"汇合点"一节)。

***

## 第一轴:Primitive(原始构建块)

### 1.2.1 表达式的 primitive:数字字面量

教材 1.2.1 开宗明义:primitive expression 的第一种是**数字字面量** `42`。求值规则:值就是自身。

教材 1.2.1 还提到中缀表达式 `1/2 + 1/4 + 1/8`,但说"Rather than attempt to enumerate them all immediately, we will introduce new expression forms as we go"——不穷举,逐步引入。

### 1.2.4 的 primitive 之二:名字

名字是第二类 primitive(教材 1.2.4)。求值规则:去 environment 查 binding。

**注意**:名字求值的结果不一定是 data——`add` 求值得到的是 function。**这是 function vs data 张力的第一颗种子,在 primitive 层就埋下了。** 后续 `f = max`、`max = 5`、def 与 assignment 同构,都在展开这条统一主线。

***

## 第二轴:Combination(组合)

### Call expression(教材 1.2.2)

最重要的组合形式是 **call expression** —— `operator(operand1, operand2, ...)`。

**求值规则是 recursive**:

1. 先 evaluate operator 和所有 operand subexpressions
2. 再 apply function to arguments

每个 operand 自己可能又是 call expression,继续套同样规则。终止于 leaf(数字/名字)。

教材给出 expression tree 的可视化——自底向上求值,叶子先算,根最后算。

### Call expression 优于中缀数学记法的三点(教材 1.2.2)

**困惑**:call expression 的三个优点为什么只和数学记法(infix notation)比?感觉怪怪的。

**解答**:教材的对比对象确实就是数学记法。因为教材的读者刚从数学过来,1.2.2 要解决的核心疑问是"为什么要换一种记法"——回答这个疑问,自然要拿读者熟悉的数学记法做参照。三个优势:

1. **任意参数数量** `max(1,-2,3,-4)`,且无歧义(函数名总在参数之前)
2. **嵌套结构完全显式** `max(min(1,-2), min(pow(3,5),-4))`,嵌套深度无原则性限制
3. 数学记法形式繁多(上标、分数线、根号),call expression 把一切统一为 **named function**(`any operator can be expressed as a function with a name`)

### 叶子判定口诀

表达式树的叶子有两类:数字字面量**和名字(标识符)**。`add`、`pow` 这类函数名也是叶子,它绑定的值是函数对象。

判定口诀:**看见 `xxx(...)` 是内部节点,其余都是叶子。**

- `add(1,10)` —— 带括号,调用表达式,内部节点,继续递归拆解
- `add` —— 仅仅是名字,叶子,去环境查表
- `10` —— 数字字面量,叶子,值是自身

### Combination 的矛盾

拼完就没了——`max(2,3)` 算完得到 3,这个 3 没有名字,下次要用还得重算。**combination 是临时的,无法保存复用。**

→ 教材 1.2.4 转向 abstraction 侧。

***

## 第三轴:Abstraction(抽象)

### 第一层:assignment(教材 1.2.4,limited)

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

教材原文:"Changing the value of one name does not affect other names."——改 radius 不会影响 area,要更新 area 只能再写一条 assignment。概括:assignment 绑的是**值**,不是**关系**。

**多赋值** `x, y = 3, 4.5`:all right evaluated before any left bound → 一行交换 `y, x = x, y`。

教材自己点明:assignment 是 "**simplest** means of abstraction"。教材 1.2.4 只陈述现象,到此为止——def 并非为解决它而生(教材 1.3 的真实动机见下)。

### 第二层:def(教材 1.3,powerful)

**教材 1.3 引入 def 的真实动机**:给**计算过程**命名(square 抽象的是"任意数平方"这个模式)。教材 1.3 原文:"Function definitions are a much more powerful abstraction technique"——powerful 在于绑的是**整个运算流程**,不是某个值。def 本身不含数据,调用时提供实参,过程才执行。

```python
def square(x):
    return mul(x, x)
```

**def 与 assignment 本质同构**——教材 1.3 原文:"Both def statements and assignment statements bind names to values"。区别在 value 的粒度:assignment 绑**数据值**(314 这个数),def 绑**计算流程**(`mul(x, x)` 这套运算,藏起来等数据配合)。

def 执行时(教材 1.3 原文 "return expression is **not evaluated right away**; it is **stored** as part of the newly defined function"):

1. 创建新 function object(签名 = def 到冒号那行)
2. 把 return expression **藏起来**(squirrel)作为 function 的一部分
3. bind 函数名到 function object

调用 user-defined function 时(教材 1.3.2):

1. 新建 **local frame**
2. 形参 bind 到实参值
3. 在新 environment 里 execute 函数体

**为什么必须新建 local frame(嵌套调用不互相踩的硬需求)**:

```python
def outer(x):
    return add(inner(100), x)   # 算完 inner 之后,还要用 x=1

def inner(x):
    return mul(x, 2)

outer(1)
```

outer(1) 执行到一半,x=1 还没死;inner(100) 也要一个叫 x 的名字。如果共用 global frame,x 绑 1 还是 100?**所以不能共用一张绑定表——每次调用各建一个独立 frame。**

注意:**"屏蔽"不是"覆盖"**。inner(100) 算完后,inner 的 local frame 直接**销毁**,outer 的 x=1 从头到尾没被动过——它一直待在 outer 自己的帧里。不是"改了再改回来",而是两个 x 从出生就住在不同的帧里。global frame 唯一,local frame 每次调用临时出生、用完销毁。

### 组合 ≠ 抽象(关键判据)

**组合是必要条件,不是充分条件。** 抽象一定依赖组合(没有基础砖块谈不上抽象),但组合本身不等于抽象。

判据一句话:**修改内部实现,外部调用代码要不要改?**
- 要改 = 没抽象(只是组合,变量全局可见)
- 不改 = 有抽象(组合 + 边界 + 隐藏细节)

```python
# 只有组合,没有抽象
a = 2
b = 3
c = a * b
print(c + 10)   # 所有变量暴露,没有边界

# 组合 + 抽象
def calc(x, y):
    c = x * y       # 内部临时变量外面访问不到
    return c + 10    # 外部只需 calc(2, 3),不用看内部怎么算
```

内部代码一模一样的组合逻辑,但多了边界,抽象才诞生。反过来:如果外部代码到处直接读写 `node.next`,代码绑定死存储结构,即使用了 class 也没有数据抽象。

***

## 枢纽:Environment(三机制在这里交汇)

Environment 不是独立一段,它是**被迫四次引出**的概念,连接 primitive、combination、abstraction 三轴:

| 出处    | 为什么被迫定义                | 定义                                                              |
| ----- | ---------------------- | --------------------------------------------------------------- |
| 1.2.4 | assignment 需要存 binding | interpreter 的"记忆",记录 names/values/bindings                      |
| 1.2.5 | 名字求值时要去哪查              | 给符号赋予含义(脱离环境,`add(x,1)` 毫无意义)                                   |
| 1.3.1 | 形参名和全局名冲突怎么解决          | **sequence of frames**;每 frame 含 bindings;有 single global frame |
| 1.3.2 | def 调用内部用什么环境          | 新建 local frame + 形参绑实参 + 在新 environment execute 函数体             |

**三轴在 environment 交汇**:

- Primitive 侧要 environment —— 名字求值靠它给含义(1.2.5)
- Combination 侧用 environment —— call expression 求值时查名字 binding(1.2.2、1.2.5)
- Abstraction 侧改 environment —— assignment 和 def 都改 binding(1.2.4、1.3.2)

### Name lookup rule(教材 1.3.1)

从当前帧开始,依次向前查找,找到第一个 binding 就用。

→ 解决 `def square(square): return mul(square, square)` + `square(2)` 为什么不出错:调用时新建 local frame,形参 square 绑到实参 2,查找 square 时 local 帧先找到 2,根本不看 global 帧的 square 函数。这就是 **local name 屏蔽 global**。

### Intrinsic name vs bound name(教材 1.3.1)

function 有 intrinsic name(函数自身名)和 bound name(帧里绑的名)。`f = max` 后 f 和 max 是两个 bound name 指向同一函数;**intrinsic name 不参与求值**。

→ 解决 `f = max; max = 5; f(2,3,4)` 为什么还能调用原 max:max 这个 bound name 被重绑到 5,但 f 还指着原来的 function object;intrinsic name 是函数自身的,查找时不用它。

***

## 汇合点:print 与 expression/statement 的二分落地

### 1.2.5 才是 expression vs statement 的真正定义处

教材 1.1.4 在 sneak preview 里提过 "Compute some value / Carry out some action",但真正把这条边界说硬是在 1.2.5。原文:

> "statements are not evaluated but **executed**; they do not produce a value but instead make some change."

- **Expression(表达式)**:被 *evaluate*(求值),产出 value
- **Statement(语句)**:被 *execute*(执行),产生 change(改环境、改流程)

`x = 3` 不走 evaluate 规则,它走 execute 规则,目的是 bind name to value,不产出 value。

### print:两条线在这里跨界(教材 1.2.6)

教材 1.2.6 引入 print,**这是两条线的汇合点** —— 一个 expression 调用(call expression),却产生了 statement 的效果(side effect)。

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

教材点明:"Pure functions can be composed more reliably into compound call expressions"——**这是后面所有 function design 的根**。print 不可靠嵌套,正因为它跨界:作为 expression 被求值,却产生 statement 的 side effect。

***

## 收束:Three Means 是骨架,some of 预留口子

教材 1.2 开篇给全书骨架(three means),1.3 结尾用 **"some of"** 预留口子:

> "We have identified in Python **some of** the elements that must appear in any powerful programming language"

### 叙事顺序 ≠ 集合包含

教材 1.2 → 1.3 的叙述顺序是:primitive → combination → assignment(limited)→ def(powerful)。这不是定义四种互相独立的能力,而是"由简到繁逐步增强语言"的教学叙事。

**集合包含关系**(工程使用视角):def 内部天然使用前面全部能力(原始值、嵌套调用、名字绑定)。反过来不成立——存在只拥有前三者、缺失 def 的简易语言(计算器、老式电子表格),但不存在拥有 def 却缺少前三者的通用语言。

### "some of" 是阶段小结,不是全集

作者加 some 已经预留口子:这只是"到当前进度为止,不可或缺的基础构件",不是编程语言完备要素全集。后续会引入:

- 布尔值、字符串、序列(新增 primitive)
- if/while/for(新增 combination means:分支/迭代)
- 对象、消息传递(新增 abstraction means)
- 高阶函数、闭包(function 作为 data 流动,展开"not so distinct"主线)

### 终极预告呼应开篇

教材 1.1.4 结尾:"functions are objects, objects are functions, and interpreters are instances of both"——呼应 1.2 开篇的 "Soon we will discover that they are really not so distinct"。整本书都在论证 function、data、object、interpreter 的最终统一。本讲已经埋下种子:

- `f = max` —— function 像 data 一样是 value,可绑名字
- `max = 5` 后 max 不能调用 —— 名字绑 data 或 function 本质都是 binding
- def 与 assignment 同构 —— 都 bind name to value

后续 Lec04 HOF(function 当参数/返回值,function 作为 data 流动)、Lec05 environment 深入、Lec29-32 interpreter,都在展开这条统一主线。

***

## 核心洞察(结论式)

> 来源:`MY/思维片段/Week1&2/`。闭包/一等公民/语言流派对比属 Lec04-05,不收进本讲。

**1. 基调在 1.2 开篇,不在 1.1.4**

1.1.4 是 sneak preview(教材原文自认),1.2 开篇的 three means 才是 framework。expression vs statement 的二分是 1.2.5 才落地的局部定义,不是全书纲领。

**2. 组合 ≠ 抽象**

组合是拼装;抽象是拼装后**加边界、隐藏细节**。判据:**修改内部实现,外部调用代码要不要改**——要改=没抽象,不改=有抽象。组合是抽象的必要条件,不是充分条件。

**3. 叶子 ≠ 只是数字**

叶子有两类:数字字面量**和名字(标识符)**。`add`、`pow` 这类函数名也是叶子,它绑定的值是函数对象。口诀:看见 `xxx(...)` 是内部节点,其余都是叶子。

**4. 环境的根本作用是赋予含义**

环境给标识符赋予含义(脱离环境,`add(x,1)` 毫无意义);每次调用新建独立帧带来的**隔离只是衍生能力**,不是环境的定义。

**5. "some of" 是阶段小结,不是全集;叙事顺序 ≠ 集合包含**

教材 1.3 结尾 "We have identified in Python **some of** the elements" 预留了口子。叙事顺序(由简到繁逐步增强)和集合包含(后者包含前者)是两个维度,不冲突。

**6. Python 是"通用通道"流派**

Python/JS/Scheme 用同一套语法通道承载 data 和 function 两种语义;C/Java 选隔离通道(编译期检查、啰嗦但安全)。这是语言设计的主动取舍,对接 Lec04 HOF 和 Lec29 Scheme。

**7. def 是"小的 global 域",独立 frame 是硬需求**

函数调用和 global 环境本质同类:都是 frame,都是 name→value 绑定表。函数内部的绑定(形参、中间计算)不该外泄,且嵌套调用时各自要用同名——**不能共用一张绑定表,否则互相踩**,所以每次调用各建独立 frame。

精确点:**"屏蔽"≠"覆盖"**——inner 算完后它的帧直接销毁,outer 的 x 从头到尾没被动过,两个 x 从出生就住在不同的帧里。

**8. def 抽象的是计算过程,assignment 绑定的是数据值**

def 绑的是运算流程(如 `mul(x, x)`),本身无数据,调用时提供实参配合才执行;assignment 绑的是具体的值。这就是 def 比 assignment "more powerful" 的实质:**绑定的粒度不同(过程 vs 值)**。

***

## Java 桥梁

| Python                                    | Java                                                   | 桥梁说明                                                                    |
| ----------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------- |
| `radius=10; area=pi*r*r` 不同步              | `int r=10; int area=r*r;`                              | Java 一模一样,绑值不绑关系。但 Java 没画 environment diagram                          |
| `def square(x): return mul(x,x)` 函数体不立即执行 | `int square(int x){return x*x;}` 方法体也不立即执行             | 都是"定义时藏起来,调用时才算"。Python 用 environment diagram 把 local frame 可视化,Java 没有 |
| Function 是 first-class(`f=max`)           | Java 8 前方法必须依附 class;Lambda 是 FunctionalInterface 的语法糖 | Python/Scheme 通用通道流派,Java 隔离通道流派。这解释 Java 写 HOF 要包接口                    |
| `print` non-pure(返回 None + 副作用)           | `System.out.println` 返回 void + 副作用                     | 都是非纯函数。Java 显式 `void`,Python 用 `None` 隐式                                |
| Environment(帧 + binding)                  | JVM stack frame + 局部变量表                                | 本质同源,都是 name→value 绑定表。Python 把 environment 作为一等概念教,Java 藏在字节码层         |

***

## 纠错与突破

**误区1:基调在 1.1.4 的 compute/carry** —— 1.1.4 自己说是 sneak preview,1.2 开篇的 three means 才是 framework。expression vs statement 的二分在 1.2.5 才落地。

**误区2:组合 = 抽象** —— 组合只是拼装,抽象是拼装后加边界隐藏细节。判据:改内部实现,外部要不要改。组合是抽象的必要条件,不是充分条件。

**误区3:叶子 = 数字** —— 叶子有两类:数字字面量和名字。函数名也是叶子。

**误区4:教材在穷举语言要素** —— 原文 "some of" 表明这是阶段性小结,不是全集。叙事顺序 ≠ 集合包含。

**误区5:def 为解决 radius/area 不同步而生** —— 教材 1.2.4 只陈述现象,未定性为痛点;1.3 引入 def 的动机是给计算过程命名。把两者串成因果链是过度解读。

**突破** —— "修改内部实现,外部要不要改"这条抽象判据比教材表述更直白,直接对接 Lec04 function design 和后续设计模式的"依赖倒置"。

***

## 相关概念

- \[\[Lec03 - Control]] — 新的 combination means(if/while:分支/迭代),扩展本讲的 expression combination
- \[\[Lec04 - Higher-Order Functions]] — function 作为 combination 的对象,正是 function vs data 张力的展开
- \[\[Lec05 - Environments]] — environment 枢纽的深入,多帧环境模型
- \[\[HW01 - Functions, Control]] — 配套作业,独立闭环
- \[\[Lab00 - Getting Started]]、\[\[Disc00 - Getting Started]] — 配套,独立闭环
- \[\[Common\_Pitfalls]] — 本讲通用坑:组合≠抽象、leaf≠数字、assignment 绑值不绑关系、def 不立即执行

***

## 复习卡

### R1 三机制是什么

<details><summary>primitive / combination / abstraction 分别指什么?教材为什么说 function & data "not so distinct"?</summary>
primitive = 最简构建块(数字、名字、内置函数);combination = 从简单构建复合(call expression、嵌套、import);abstraction = 给复合元素命名作为单元(assignment、def)。"not so distinct" 因为 function 也能绑名字(f=max)、当 value 流动,def 和 assignment 本质同构(都 bind name to value)。
</details>

### R2 call expression 求值规则

<details><summary>evaluate `sub(pow(2, add(1,10)), pow(2,5))` 的步骤?叶子节点有哪些?为什么是 recursive?</summary>
先 evaluate operator 和所有 operands,再 apply。每个 operand 自己可能是 call expression,继续套同样规则,所以 recursive。leaf(数字/名字)终止递归:数字值是自身,名字去 environment 查 binding。自底向上算 expression tree。

叶子节点:sub、pow、2、add、1、10、5(注意 sub/pow/add 都是名字,属于叶子,去环境查表得到函数)。判定口诀:看见 xxx(...) 是内部节点,其余都是叶子。
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

### R6 基调在哪里

<details><summary>教材的基调陈述在 1.1.4 还是 1.2 开篇?1.1.4 的定位是什么?expression vs statement 的二分在哪落地?</summary>
基调在 1.2 开篇的 three means(primitive/combination/abstraction)。1.1.4 自己说是 "sneak preview"(预览),原文 "In the next section, we will start from scratch"。1.1.4 提到 compute/carry 但只是随手说明。expression vs statement 的二分在 1.2.5 才真正落地("statements are not evaluated but executed")。
</details>

### R7 组合 vs 抽象判据

<details><summary>一段代码拼在一起但变量全局可见,算抽象吗?判断标准?组合和抽象的因果关系?</summary>
不算,只是 combination。判据:修改内部实现,外部调用代码要不要改?要改=没抽象,不改=有抽象。Abstraction = Combination + 边界 + 隐藏细节。组合是抽象的必要条件(抽象一定依赖组合),不是充分条件(组合本身不产生抽象)。
</details>

### R8 pure vs non-pure

<details><summary>`print(print(1), print(2))` 输出?为什么最后 None 不显示?为什么这是两条线的汇合点?</summary>
依次输出 `1` `2` `None None`。求值规则没变(先算 operator 和 operands 再 apply),但 operand 求值时 side effect 发生:print(1) 副作用显示 1 + 返回 None;print(2) 副作用显示 2 + 返回 None;外层 print(None,None) 副作用显示 "None None" + 返回 None。最后那个 None 不显示因为 interpreter 默认不显示 None。汇合点:print 作为 expression 被求值,却产生 statement 的 side effect,两条线在这里跨界。
</details>

### R9 environment 四次递进定义

<details><summary>environment 在 1.2.4 / 1.2.5 / 1.3.1 / 1.3.2 分别被定义成什么?为什么说它是三轴的交汇?</summary>
1.2.4:interpreter 的"记忆"(names/values/bindings 的存储);1.2.5:给符号赋予含义(脱离环境 add(x,1) 毫无意义);1.3.1:sequence of frames(每 frame 含 bindings,有 single global frame);1.3.2:调用 user-defined function = 新建 local frame + 形参绑实参 + 在新 environment execute 函数体。交汇:primitive 侧要 environment(名字求值靠它给含义),combination 侧用 environment(call expression 求值时查名字),abstraction 侧改 environment(assignment 和 def 都改 binding)。
</details>

### R10 assignment vs def

<details><summary>两者都属于 abstraction means,本质区别?def 的 "powerful" 在哪?</summary>
都 bind name to value,本质同构(教材 1.3 原文)。区别在 value 的粒度:assignment 绑**数据值**(一个具体的数);def 绑**计算过程**(无数据,调用时提供实参配合才执行)。教材 1.3 引入 def 的动机是给计算过程命名(square 抽象"任意数平方"这个模式),不是解决 radius/area 不同步——教材并未建立这条因果链。
</details>

### R11 intrinsic name vs bound name

<details><summary>`f = max; max = 5; f(2,3,4)` 为什么还能调用原 max?</summary>
function 有 intrinsic name(函数自身名)和 bound name(帧里绑的名)。f 和 max 是两个 bound name 指向同一 function object。max = 5 后 max 这个 bound name 重绑到 5,但 f 还指着原来的 function object。intrinsic name 不参与求值(查找时用 bound name),所以 f(2,3,4) 还能调用原 max,max(1,2) 报错 "int object is not callable"。
</details>

### R12 为什么必须新建 local frame

<details><summary>outer(1) 执行到一半(还需要 x=1),此时开始算 inner(100)(也需要叫 x)。如果共用 global frame 会怎样?"屏蔽"和"覆盖"的区别是什么?</summary>
共用一张绑定表则互相踩:x 绑 1 还是 100 没法同时满足,outer 算第二个 operand 时拿到的是被 inner 覆盖后的值。所以每次调用各建独立 frame。"屏蔽"不是"覆盖":inner 算完后它的 local frame 直接销毁,outer 的 x=1 从头到尾没被动过——两个 x 从出生就住在不同的帧里。global frame 唯一,local frame 临时出生、用完销毁。
</details>

### R13 叙事顺序 vs 集合包含

<details><summary>教材 1.2→1.3 的叙述顺序是 primitive→combination→assignment→def。这代表四种互相独立的能力吗?"some of" 什么意思?</summary>
不是。这是"由简到繁逐步增强语言"的教学叙事,不是集合划分。集合包含视角:def 内部天然使用前面全部能力(原始值、嵌套调用、名字绑定)。反过来不成立——存在只拥有前三者、缺失 def 的简易语言(计算器、老式电子表格),但不存在拥有 def 却缺少前三者的通用语言。"some of" 表明这是阶段性小结(到当前进度为止不可或缺的基础构件),不是编程语言完备要素全集。后续会引入 boolean、sequence、object、control flow 等更多 primitive/combination/abstraction。
</details>
