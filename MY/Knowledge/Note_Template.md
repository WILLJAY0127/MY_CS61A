# 讲次笔记模板(可复用)

> **用途**:CS61A 每讲课程笔记的标题结构模板。忠于教材骨架,网络感优先,精简非知识内容。
> **来源**:Lec01&02 验证版,2026-09-07 定稿。
> **关联**:CLAUDE.md 的「L — Link 链接知识」段;实际写笔记时套这个结构。

## 半结构化规则

- **每讲必有**:Resources / Skeleton / Evolution Path / My Understanding / Java Bridge / Corrections & Breakthroughs / Related Concepts / Review Cards
- **按内容定**:Tension(0-2 条)、Pivot(0-1 个枢纽概念)
- **英文术语为主**:标题英文,术语英文(primitive/combination/abstraction/call expression/assignment/environment/frame/binding/pure/non-pure/side effect 等),正文中文
- **精简**:去掉非知识元内容(复习卡用法说明、冗余过渡、客套话)。只放知识结构 + Jace 推导 + 桥梁 + 卡片本身

## 标题结构

```markdown
---
lecture: XX
title: <标题>
status: <complete / 待补哪阶段>
---

# Lecture XX · <标题> —— <一句话主线/张力>

> **Core Question**: <统领性问题,一句话,问"为什么/机制"不是"是什么">

## 📎 Resources
- **PPT**: `assets/slides/<文件名>.pdf`
- **Video**: <Lec 编号> · <YouTube 链接,取自大纲>
- **Textbook**(小节级,由 PPT/字幕内容核实,不是照抄大纲 Ch X.Y):
  - Ch. X.Y <章节名> — `pages/<页面>.html`(X.Y.1 / X.Y.2 / ...)
- **Subtitles**: 字幕 Lecture <编号> — 子主题:<从 SOURCE/cs61a_zimu 实际文件名列出>
- **Code**: `assets/slides/<编号>.py` 或 `.scm`(若有)
- **Lab/Disc**: 取自大纲当天条目
- **HW**: 取自大纲当天条目(含 Due 日期;独立闭环,只标关联)

## Skeleton: <本讲整体网络骨架>
<一张表或结构图,展示本讲核心结构。常见形式:
- 2×N 网格(如 Three Means × Two Elements)
- 或核心概念关系图
- 一眼能看全本讲知识网络>

## Evolution Path(每层 Limitation 引出下层)
### 1. <层名>
- 内容
- **Limitation**: <局限> → 引出下一层
### 2. ...

## Tension: <贯穿张力线>(0-2 条,按内容)
<本讲或跨讲的二元张力,如 Function vs Data / Statement vs Expression / Syntax vs Semantics>
<教材原文佐证 + 证据链 + 后续讲次预告>

## Pivot: <枢纽概念>(0-1 个,按内容)
<串起本讲多个概念的核心枢纽,如 Environment。讲清它在教材里被几次递进定义>

## My Understanding(Jace 推导链,保留原话)
<来源:`MY/思维片段/` 下 Jace 自己推导的笔记>
<每条:原话 + 推导链(直觉→修正→结论)>
<只收与本讲直接相关的,跨讲的留到对应讲次>

## Java Bridge
<表格:Python 概念 | Java 对应 | 桥梁说明>
<绑到 Jace 已知的 Java/设计模式>

## Corrections & Breakthroughs
<Claude 主动纠正的误区 + Jace 的啊哈时刻,注明来源思维片段>
<Breakthrough:Jace 自己推导出、比教材讲得更准的洞察>

## Related Concepts
[[<其他讲次笔记>]] — <一句话前后关系>
[[<关联 HW/Lab/Project 笔记>]] — <关联说明>

## Review Cards
### R1 <问题摘要>
<details><summary><问题,问机制不问是什么></summary>
<答案要点,精简,术语英文>
</details>
<每个 Skeleton 网格点、Evolution 每层、Tension、Pivot 都至少一张卡>
```

## 硬性规则(沿用 CLAUDE.md)

- **层间桥梁优先,不堆砌孤立知识点** — 每个概念要回答"为什么会讲到这个"
- **"My Understanding"段保留完整推导链**,不止结论
- **交付为真正的 .md 文件**(headers + fenced code blocks + Obsidian 双链)
- **文件名与 slides 文件名对齐**:`Lec02 - Functions.md` ↔ `02-Functions_1pp.pdf`
- **包含 Java/设计模式桥梁**
- **忠于教材**:先读 PPT + 字幕 + 教材小节正文(不是只读标题),再写笔记——教材正文必读,标题不够
