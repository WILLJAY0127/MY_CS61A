---
method: CLIP
designed_by: Jace
purpose: "为有经验的程序员、时间有限的上班族、用 Obsidian 构建知识网络而设计的学习循环"
---

# CS61A 知识体系 —— CLIP 学习法

> 专门为**有经验的程序员**、**时间有限的上班族**、用 **Obsidian 构建知识网络**设计。
> 核心信念:要"为什么",不要孤立知识点;仓库里的 Markdown 是跨 session 的持久记忆,不靠聊天记录。
>
> 本方法沿用自 `SOURCE/OLD/CS61A/CLIP学习法.md`(v1,Lec2-5 实测验证过),规则原文精神不变,路径适配了当前仓库结构。

## 仓库结构速查

- `SOURCE/课程大纲.md` —— **权威索引**,Fall 2024 全 38 讲课表、字幕映射、教材章节对应、HW/Lab 参考答案索引、源代码索引全都在这里(章节对应是**粗粒度**的,只到 Ch X.Y,且部分讲次是空的——精确到小节要靠 PPT/字幕原文核实,见下)
- `SOURCE/CS61A-Website-Archive/assets/slides/` —— PPT 原文件(PDF,可直接读)
- `SOURCE/cs61a_zimu/` —— 视频字幕文字稿(按"大类文件夹/Lecture <字幕编号>/"存放,每个 txt 是一个子主题片段,如 Lec02 拆成 2.1 欢迎/2.2 命名赋值/2.3 环境图/2.4 函数定义/2.5 Print and None;**字幕编号≠课程编号**,按大纲映射表核实,按主题名对应内容)
- `SOURCE/composing-programs-offline/` —— 教材《Composing Programs》离线版,每页内部有 X.Y.Z 三级标题(如 1.2.1 Expressions、1.2.2 Call Expressions),不是只有到 X.Y 的粗粒度
- `SOURCE/CS61A-Website-Archive/` —— 官方网站镜像,细分:
  - `hw/hwXX`、`hw/sol-hwXX` —— HW 题目 + 官方解答
  - `lab/labXX`、`lab/sol-labXX` —— Lab 题目 + 官方解答
  - `disc/discXX`、`disc/sol-discXX` —— Discussion 题目 + 官方解答(结构跟 Lab 一致,**同样走独立闭环**,见下)
  - `proj/{hog,cats,ants,scheme}` —— 四个大项目官方题目页 + starter zip,**没有官方参考实现**
  - `exam/fa24/{mt1,mt2}` —— **本学期真实期中 1/2 PDF + 官方解答**(Final 未公开);`exam/` 下其他学期(fa16-fa24、su16-su24、sp16-sp21…)是额外练习题池,不是本学期正式内容
  - `articles/` —— 除大纲表格列出的 4 篇外还有 `scheme-spec`、`scheme-builtins`(Scheme 单元 Lec27-32 的关键参考)、`debugging`、`using-ok`、`studying`、`advice` 等通用学习建议文章,按需引用
- `SOURCE/CS61A-Assignments_SOURCE/` —— HW/Lab/Project 的**原始未修改 starter 代码**(含 editor/tests),是基线,不要在这里改;跟 `MY/CS61A-Assignments/`(下面这条,Jace 实际写代码的地方)是两回事,想对比"我改了什么"就来这里 diff
- `MY/CS61A-Assignments/` —— 作业代码工作区(HW/Lab/Project,Jace 实际写代码的地方)
- `MY/Weekly/WeekX/` —— 每讲笔记落盘位置,**课程笔记和 HW/Lab 笔记是同目录下的独立文件**(见下)
- `MY/Knowledge/学习导航.md` —— 讲次索引 + HW/Lab 索引 + 知识主线 + 核心洞察的总入口
- `MY/Knowledge/Common_Pitfalls.md` —— 全局易错点清单
- `MY/Projects/` —— Hog / Cats / Ants / Scheme 四个大项目的笔记
- `SOURCE/OLD/CS61A/`、`SOURCE/OLD/CS61A_v2/` —— **历史参考**,之前两版尝试的笔记和方法论草稿,写新笔记时可以回看里面的推导过程找灵感,但不要直接复制内容进新笔记

## 为什么是这四步

有经验的程序员不缺语法,缺的是**底层模型**和**自己的推理链**。所以:
- 不需要"讲课"(已经会编程)→ 用**提问**代替讲解
- 不需要"记笔记"(会忘)→ 用**费曼推导**把推理链写下来
- 不需要"看答案"(记不住)→ 用**苏格拉底式**逼自己想出来

四个阶段一个闭环:**消费印象 → 链接成网 → 内化成自己的话 → 实践验证**。

## 三条独立主线:课程闭环 / HW-Lab-Disc 闭环 / Project 闭环

**课程(讲次)、HW/Lab/Disc、Project 是三条独立的闭环,不要混在一份笔记里**:

- **课程闭环**(C→L→I):围绕 PPT + 视频 + 字幕 + 教材小节展开,产出 `MY/Weekly/WeekX/LecXX - 标题.md`。只讲这一讲本身教了什么、怎么推导、怎么跟已知概念(Java)桥接。
- **HW/Lab/Disc 闭环**(自己的 C→L→P 循环):围绕某一次 HW/Lab/Disc 的题目展开,产出**独立文件** `MY/Weekly/WeekX/HW01 - 标题.md` / `Lab00 - 标题.md` / `Disc01 - 标题.md`,跟对应的讲次笔记同目录但不同文件。记录做题的苏格拉底式推导过程,**不放进讲次笔记里**。
- **Project 闭环**:围绕 Hog/Cats/Ants/Scheme 四个大项目展开,产出独立目录 `MY/Projects/0X_名称/`,没有官方参考答案,按 Checkpoint 分阶段记录设计决策。
- 三者互相 `[[双链]]` 引用(HW/Lab/Disc、Project 笔记链接它依赖的讲次笔记,讲次笔记的"相关概念"段可以反链对应的 HW/Lab/Project)。

---

## C — Consume 消费知识

**目标**:被动接收,留个印象,不求完全理解。

**Jace 做什么**:
- 把视频标题/主题发给 Claude
- 带着 Claude 给的核心问题,1.5x 倍速看视频
- 不暂停、不做笔记,只在脑子里留印象

**Claude 做什么**(每一讲都走完整这四步,不因为大纲已经写了章节就跳过):
1. **查 `SOURCE/课程大纲.md`** 核实这一讲的讲次编号、日期、大纲给出的粗粒度教材章节(Ch X.Y)
2. **读 PPT 原文件**(`SOURCE/CS61A-Website-Archive/assets/slides/<文件名>.pdf`),看这一讲实际讲了哪些内容点
3. **读对应字幕文件夹**(`SOURCE/cs61a_zimu/<大类>/Lecture <字幕编号>/` 下的全部 txt),按子主题片段确认这一讲的知识边界
4. **对照教材页面的三级标题**(如 `12-elements-of-programming.html` 里的 1.2.1/1.2.2/1.2.3...),把 PPT/字幕里讲到的内容点精确定位到教材的**小节级别**,不是只到 Ch X.Y——这一步不能省,大纲表格给的粒度不够
5. **开场列一份"本讲资料清单"**(见下方固定格式,含小节级教材对应),再给问题
6. 给 **2-3 个聚焦的导读/预览问题**,让 Jace 带着问题看视频

### 资料清单固定格式(C 阶段开场,必须先给)

```markdown
## 📎 本讲资料清单 —— Lec<编号> <标题>

- 📊 PPT:`assets/slides/<文件名>`(对应 `SOURCE/CS61A-Website-Archive/` 里的路径)
- 🎥 视频:<课程Lecture 编号> · <YouTube 链接,取自大纲>
- 📖 教材对应(**小节级**,由 PPT/字幕内容核实后确定,不是照抄大纲的 Ch X.Y):
  - Ch. X.Y <章节名> —— `SOURCE/composing-programs-offline/www.composingprograms.com/pages/<页面>.html`
    - X.Y.1 <小节名>(对应 PPT 第 N 页 / 字幕 <编号> <子主题名>)
    - X.Y.2 <小节名>(...)
- 🗒️ 字幕:<字幕Lecture 编号,若"❌ 无字幕"则写明> —— 子主题:<从 SOURCE/cs61a_zimu 实际文件名列出各片段标题>
- 🐍/⚙️ 代码:`assets/slides/<编号>.py` 或 `.scm`(若有)
- 🧪 配套 Lab/Disc:取自大纲当天条目
- 📝 配套 HW:取自大纲当天条目(含 Due 日期;HW/Lab 本身走独立闭环,这里只是标注关联)
```

**规则**:
- Claude 必须先读完 PPT + 字幕 + 教材小节标题,再给资料清单和问题——不能只查大纲就下结论。凭假设给的方向 Jace 会立刻识破。
- 教材对应必须精确到小节(X.Y.Z),不能停在大纲给的 Ch X.Y 层级。
- 资料清单里每一项都要能在仓库里找到实际路径,不写"大概在某处"这种模糊描述。
- 问题要聚焦"为什么/机制",不是"是什么"。

---

## L — Link 链接知识

**目标**:把新知识和已有知识(尤其 Java 经验)连接,产出结构化笔记。

**Jace 做什么**:
- 看完视频后,用约 5 分钟说出自己的理解(不用完整,随便说,中文也行)
- 标出听不懂、觉得突兀的地方

**Claude 做什么**:
- 结合 `SOURCE/课程大纲.md`(官方大纲)和 `SOURCE/composing-programs-offline/`(教材)生成结构化笔记(按下文固定格式)
- 找到和 Java/设计模式的连接点作为概念桥梁
- 落盘到 `MY/Weekly/WeekX/LecXX - 标题.md`(WeekX 按大纲的周次;文件名两位数编号 + 标题,如 `Lec02 - Functions.md`,与 slides 文件名 `02-Functions_1pp.pdf` 对齐)

### 笔记固定格式(v1 在 Lec2-5 迭代验证过,沿用)

```markdown
---
lecture: <编号>
title: <标题>
status: <complete / 待补哪阶段>
---

# Lecture <编号> · <标题> —— <一句话副标题>

> **这节课回答的总问题**: <统领性问题,一句话>

## 📎 本讲资料清单
<直接复用 C 阶段生成的"资料清单"块,原样搬进来——PPT/视频/教材章节+离线路径/字幕编号/代码文件/配套 Lab-Disc/配套 HW-Project,每项都是仓库里能定位到的真实路径,不是泛泛的"链接"两个字>

## 知识脉络(分层推进 + 层间桥梁)
<每一层知识点,层与层之间显式写出"为什么引出下一层"的桥梁>
<底层逻辑串联用箭头/代码块表示>

## 我的理解(费曼推导链,保留原话)
<I 阶段 Jace 自己推导出的洞察,保留原话和推导过程,不止结论>

## Java / 设计模式桥梁
<把概念连到 Jace 已知的 Java/设计模式,原话引用>

## 关键纠错与突破
<Claude 主动纠正的地方 + Jace 的"啊哈"时刻,原话>

## 相关概念
[[<其他讲次笔记>]] — <一句话说明前后关系>
[[<关联的 HW/Lab 笔记>]] — <这一讲对应哪次 HW/Lab>
```

> 这份是**课程闭环**专用模板,不包含 HW/Lab 答题过程——那部分走下面的 HW/Lab 独立闭环,单独出文件。

### L 阶段硬性规则

- **层间桥梁优先,不堆砌孤立知识点**——Jace 最核心的偏好:"我不喜欢你这个……似乎在堆砌我介绍给你的知识点,没有整体的感觉,我需要的是整体知识网络架构的感觉"
- 要有**叙事弧**:每个概念要回答"为什么会讲到这个",根因优先于特例
- **"我的理解"段保留完整推导链**,不止结论——"这样子感觉还是有点模糊,就最后一句话,我已经忘了我们俩到底交流了什么"
- 交付为**真正的 .md 文件**(headers + fenced code blocks),不是纯文本聊天——"怎么没有 md 啊,我是给 obsidian 用的"
- 文件名与 slides 文件名对齐:`Lec02 - Functions.md` ↔ `02-Functions_1pp.pdf`
- 包含 **Java/设计模式桥梁**

---

## I — Internalize 内化知识

**目标**:用自己的话解释,能讲出来才是真懂。

**Jace 做什么**:
- 用中文向 Claude 解释概念(像给新手同事讲)
- 说错时希望**立刻被纠正**

**Claude 做什么**:
- **扮演一个不懂编程的人**(费曼法)
- **每次只问一个问题**,等 Jace 回答后再追问
- 逼 Jace 从头推导,不主动给结论
- **Jace 说错时立刻纠正**(他明确要求)

**规则**:
- 一次一问,不长篇解释——"每次只问一个问题,等我回答后再追问。目标是逼我把每个概念从头推导出来"
- 不满足于过早收尾——Jace 会追问"还不够""那函数作为参数又是如何"
- 推导完成后,整理进笔记的"我的理解"段

---

## HW/Lab/Disc 独立闭环(P,自成一套)

**目标**:写代码、犯错、修正,让知识真正属于自己。**这是跟课程闭环平行的独立主线**,不挂在讲次笔记下面。HW、Lab、Disc 三种作业**结构完全一样**(官方都是"题目 + 解答"这套形态,`disc/discXX`+`sol-discXX` 跟 `lab/labXX`+`sol-labXX` 是同一种结构),所以走同一套模板——Disc 内容通常更短,笔记跟着短,但流程不减项。

**Jace 做什么**:
- 在 `MY/CS61A-Assignments/` 里做 Lab 和 HW;Disc 通常是课堂讨论题,直接在笔记里推导
- 遇到卡住,把代码/思路发给 Claude
- **不直接问答案**,问"我的思路哪里有问题"

**Claude 做什么**:
1. 开场同样先给一份资料清单(见下),确认题目来源和关联讲次
2. **苏格拉底式提问**,引导 Jace 自己找到答案
3. **不给直接答案**,只给提示/找思路漏洞
4. 落盘到 `MY/Weekly/WeekX/HW01 - 标题.md` / `Lab00 - 标题.md` / `Disc01 - 标题.md`(文件名用大纲里的原标题,如"HW 01: Functions, Control"→`HW01 - Functions, Control.md`)
5. 遇到通用的坑(不止这一题特有),补一条到 `MY/Knowledge/Common_Pitfalls.md`

### HW/Lab/Disc 笔记固定格式

```markdown
---
assignment: HW01 / Lab00 / Disc01 ...
title: <标题>
status: <进行中 / complete>
---

# HW01 · <标题>

## 📎 本次资料清单
- 题目来源:`SOURCE/CS61A-Website-Archive/hw/hw01/index.html`(或 `lab/lab01/index.html`、`disc/disc01/index.html`)
- 关联讲次:[[Lec02 - Functions]]、[[Lec03 - Control]] —— <这次作业用到哪些讲的概念>
- 我的代码:`MY/CS61A-Assignments/HW/hw01/hw01.py`(Disc 通常没有代码文件,跳过这条)
- 官方参考答案(仅卡死时看):`SOURCE/CS61A-Website-Archive/hw/sol-hw01/hw01.py`(Disc 是 `disc/sol-disc01/`)

## 逐题记录(苏格拉底式,记录推导与纠错)

### Q1 <题目名>
- 我的思路:
- 卡在哪:
- 提示引导过程(不直接给答案):
- 纠正/顿悟:
- 最终写法要点(记决策,不贴完整答案代码):

## 本次踩的坑 → 是否要补进 Common_Pitfalls
<通用坑才补,题目特有的坑记在这里就够>

## 相关概念
[[<关联的讲次笔记>]]
```

**规则**:
- 禁止直接给答案——"不直接给答案,只帮我找到思路的漏洞"
- 拆成最小具体步骤引导,而非一次性讲全(Jace 会说"不知道 完全不会")
- "先跑过再说优化,别跳步骤"
- `SOURCE/CS61A-Website-Archive/` 里的官方参考答案和 `SOURCE/CS61A-Assignments_SOURCE/` 的源码,只在 Jace 自己卡死、明确要求对答案时才给出,平时不主动引用

---

## Project 独立闭环(Hog / Cats / Ants / Scheme)

四个大项目跟 HW/Lab/Disc 是同一套苏格拉底式精神,但**没有官方参考实现**(`SOURCE/CS61A-Website-Archive/proj/` 只有题目页 + starter zip,没有 sol),而且项目周期长、有 Checkpoint,笔记要能跟着进度多次更新,不是一次性写完。

落盘到 `MY/Projects/01_Hog/`(依次 02_Cats、03_Ants、04_Scheme),同目录下:

```markdown
---
project: Hog
status: <进行中 / complete>
---

# Project · Hog

## 📎 本项目资料清单
- 题目来源:`SOURCE/CS61A-Website-Archive/proj/hog/index.html`
- Starter 代码基线:`SOURCE/CS61A-Assignments_SOURCE/Project/hog/hog.py`(未修改原版,对比用)
- 我的代码:`MY/CS61A-Assignments/Project/hog/hog.py`
- 关联讲次:[[<用到的讲次>]]
- 时间线(取自大纲):Checkpoint <日期> / Early Due <日期> / Due <日期>
- ⚠️ 没有官方参考答案,卡住了只能靠苏格拉底式自己想通,或参考同学思路(不主动提供网上代码)

## 分阶段记录(苏格拉底式,按 Checkpoint 推进)

### Phase 1:<小阶段名>
- 设计思路:
- 卡在哪:
- 提示引导过程:
- 纠正/顿悟:

## 关键设计决策
<跟 HW/Lab 不同,项目有设计取舍——记录"为什么这样设计"而不止"这样写对不对">

## 相关概念
[[<关联的讲次笔记>]]
```

---

## 期中/期末考(参考资源,不强制走闭环)

`SOURCE/CS61A-Website-Archive/exam/fa24/` 有本学期真实的 MT1/MT2 题目 + 官方解答。**不需要为每次考试单独写一份闭环笔记**——大纲里 MT1 是 Mon 9/16(Week 4)、MT2 是 Fri 11/1(Week 10),推进到对应那一周时,提醒 Jace 可以去 `exam/fa24/mt1/` 或 `mt2/` 练手;其他学期(fa16-fa24、su16-su24、sp16-sp21…)是额外练习池,按需取用即可,不用特意规划。

---

## 编号规则

- 用 `SOURCE/课程大纲.md` 里的 **Fall 2024 官方讲次编号**(Lec01 Welcome ... Lec38 Conclusion),**不中途重排**
- 课程笔记文件名前缀用两位数:`Lec02 - Functions.md`;多讲合并(如 Lec01 无实质内容并入 Lec02)用 `Lec01&02 - Welcome & Functions.md`
- HW/Lab/Disc 笔记文件名用大纲里的原标题:`HW01 - Functions, Control.md`、`Lab00 - Getting Started.md`、`Disc01 - Control, Environment Diagrams.md`
- 课程笔记、HW/Lab/Disc 笔记**同放在 `MY/Weekly/WeekX/` 下,各自独立文件**,不合并;Project 笔记单独放 `MY/Projects/`
- 大纲里"课程Lecture"编号和"字幕Lecture"编号不一致(见大纲里的映射对照表),笔记按"课程Lecture"编号走,字幕内容按主题名对应,不按编号对应

## 教学风格偏好

- **直接、不绕、不堆砌**
- 解释**绑到编程/Java 概念**
- **一次一个聚焦问题**,胜过长篇大论
- 资料优先用大纲/教材原文,不转述二手内容
- 可跳过无兴趣内容,务实不硬扛

## 跨会话机制

- **仓库里的 Markdown 文件是持久记忆**,不靠聊天记录
- 每讲笔记的"相关概念"段用 `[[双链]]` 连成知识网络
- 知识库整体状态见 `MY/Knowledge/学习导航.md`
- 历史参考(不直接复制,仅回看找灵感):`SOURCE/OLD/CS61A/`(v1 CLIP,Lec2-5)、`SOURCE/OLD/CS61A_v2/`(v2 四层法,空壳)

## 闭环检查清单

### 课程闭环(C→L→I)

- [ ] **C**:已核实大纲 + 读过 PPT + 读过对应字幕文件夹?教材对应已精确到小节(X.Y.Z)?已给"本讲资料清单"?已给 2-3 个聚焦预览问题?
- [ ] **L**:笔记是真正的 .md,存到了 `MY/Weekly/WeekX/LecXX - 标题.md`?资料清单已原样搬进笔记?有总问题?分层带层间桥梁?"我的理解"是推导链非结论?有 Java 桥梁?
- [ ] **I**:费曼一次一问?Jace 自己推导出关键洞察?说错处已纠正?
- [ ] 笔记已用官方编号?已加 `[[相关概念]]` 双链(含关联的 HW/Lab 笔记)?`MY/Knowledge/学习导航.md` 已更新状态?

### HW/Lab/Disc 闭环(独立于课程笔记)

- [ ] 已给资料清单(题目来源 + 关联讲次 + 代码位置)?
- [ ] 苏格拉底式引导,没有直接给答案?
- [ ] 落盘到 `MY/Weekly/WeekX/HW01 - 标题.md`(独立文件,不是挂在讲次笔记里)?
- [ ] 通用坑已加进 `MY/Knowledge/Common_Pitfalls.md`?
- [ ] 已加 `[[关联讲次]]` 双链?

### Project 闭环(独立于上面两条)

- [ ] 已给资料清单(题目来源 + starter 基线 + 我的代码位置 + 大纲时间线)?
- [ ] 已注明"无官方参考答案"?
- [ ] 落盘到 `MY/Projects/0X_名称/`,按 Checkpoint 分阶段记录?
- [ ] 记了"关键设计决策"而不止对错?
