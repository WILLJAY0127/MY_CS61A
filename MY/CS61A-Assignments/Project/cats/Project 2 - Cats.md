---
type: project
id: "02"
title: Computer Aided Typing Software
status: in_progress
source: SOURCE/CS61A-Website-Archive/proj/cats/index.html
solution: none
code: MY/CS61A-Assignments/Project/cats/cats.py
lectures: Lec09, Lec10
due: 2024-10-08
---

# Project 2: Computer Aided Typing Software

- **题目来源**：`SOURCE/CS61A-Website-Archive/proj/cats/index.html`
- **Starter 基线**：`SOURCE/CS61A-Website-Archive/proj/cats/cats.zip`
- **我的代码**：`MY/CS61A-Assignments/Project/cats/cats.py`
- **关联讲次**：Lec09 · Tree Recursion（Q6/Q7 编辑距离）、Lec10 · Sequences（Phase 1 字符串与列表）
- **Due**：2024-10-08（Checkpoint 2024-10-03）

---

## 总起

**一句话定位**：实现一个打字速度测试器 —— 先做「选段落 + 算准确率 + 算 WPM」，再写两个自动纠错的 diff 函数（递归 + 剪枝），最后做多人对战的进度同步。

### 三个阶段

| 阶段 | 题 | 你在搭什么 | 核心技能 |
|---|---|---|---|
| Phase 1: Typing | Problem 1–4 | 单机打字测试：按条件选段落、判断段落是否含关键词、算准确率、算 WPM | 列表遍历、字符串处理、高阶函数当参数 |
| Phase 2: Autocorrect | Problem 5–7 | 自动纠错：在词表里找最接近的词，并实现两种 diff 函数 | `min`/`max` 的 `key` 参数、递归 + limit 剪枝 |
| Phase 3: Multiplayer | Problem 8–10 | 多人对战：上传进度、算每个玩家的每词耗时、算每词谁最快 | 字典、`zip`、多列表对齐 |
| Phase 4: Efficiency（EC） | Problem EC | 用记忆化装饰器加速 diff 函数 | 装饰器、闭包、缓存 |

### 题目地图

| 题号 | 名称 | 分值 | 类型 |
|---|---|---|---|
| Problem 1 | `pick` | 1 | 编程 |
| Problem 2 | `about` | 1 | 编程（闭包） |
| Problem 3 | `accuracy` | 2 | 编程 |
| Problem 4 | `wpm` | 1 | 编程 |
| Problem 5 | `autocorrect` | 2 | 编程（高阶函数） |
| Problem 6 | `furry_fixes` | 3 | 编程（递归 + 剪枝） |
| Problem 7 | `minimum_mewtations` | 3 | 编程（树递归 + 剪枝） |
| Problem 8 | `report_progress` | 2 | 编程（字典） |
| Problem 9 | `time_per_word` | 1 | 编程 |
| Problem 10 | `fastest_words` | 3 | 编程 |
| Problem EC | `memo_diff` | 0 | 选做（装饰器） |

**对应讲义**：Lec09 · Tree Recursion，Lec10 · Sequences

### 依赖关系（必须按顺序做）

| 题 | 依赖 | 说明 |
|---|---|---|
| Problem 1 `pick` | 无 | 独立 |
| Problem 2 `about` | 无 | 返回的函数要喂给 `pick` 的测试 |
| Problem 3 `accuracy` | 无 | 独立 |
| Problem 4 `wpm` | 无 | 独立 |
| Problem 5 `autocorrect` | 无 | diff 函数是参数，先按「黑盒」写 |
| Problem 6 `furry_fixes` | 无 | 写完回 Problem 5 验证 |
| Problem 7 `minimum_mewtations` | 思路复用 Problem 6 | 多出「增/删」两条递归 |
| Problem 8 `report_progress` | 无 | 独立 |
| Problem 9 `time_per_word` | 无 | 输出结构喂给 Problem 10 |
| Problem 10 `fastest_words` | **强依赖 Problem 9** | 输入就是 Problem 9 的返回字典 |
| Problem EC `memo_diff` | 依赖 Problem 6/7 | `memo` 作为范例在同一个文件里 |

### 关键数据结构

- `words_and_times = {'words': [...], 'times': [[...], ...]}`：`times[i][j]` = 玩家 `i` 打第 `j` 个词花的时间
- `timestamps_per_player`：**累计**时间戳，每人长度 = 词数 + 1（第一个是开始时间）→ 换成 `times` 后每人少一项
- diff 函数契约：`diff_function(typed, source, limit) -> number`；返回值 **> limit 时具体是多少不重要**，只表示「差太远」
- `upload(dict)`：入参固定为 `{'id': ..., 'progress': ...}` 两项字典

### 函数名词典（字面含义 → 实际功能）

| 函数 | 字面含义 | 实际功能 |
|---|---|---|
| `pick` | 挑 | 取第 `k` 个满足条件的段落 |
| `about` | 关于 | 返回「段落是否含关键词」的判定函数（闭包） |
| `accuracy` | 准确率 | 同位置完全相同的词数 ÷ `typed` 词数 |
| `wpm` | 每分钟词数 | `(字符数 / 5) / (秒数 / 60)` |
| `autocorrect` | 自动纠正 | 词表里找最像的词，超 `limit` 就原样返回 |
| `furry_fixes` | 毛茸茸修复（project 梗） | 逐位替换计数 + 长度差 |
| `minimum_mewtations` | 最小喵化（mutation 的猫化拼写） | 增/删/改三种操作的编辑距离 |
| `report_progress` | 报告进度 | 上传进度并返回「第一个错词之前」的正确率 |
| `time_per_word` | 每词用时 | 把累计时间戳换成相邻差 |
| `fastest_words` | 最快词 | 每个词归给耗时最短的玩家 |
| `memo_diff` | 记忆化 diff | 按 `(typed, source)` 缓存 `(value, limit)` |

### 硬性约束清单（官方 Important 汇总，写之前先扫一遍）

- **Problem 2**：`subject` 必须全小写（官方 `assert`）；比较忽略大小写与标点，且只认整词（"dogs" 不匹配 "dog"）
- **Problem 3**：分母是 `typed` 的词数；`typed` 比 `source` 多出来的词一律算错
- **Problem 5**：差值并列时取词表里**最靠前**的那个
- **Problem 6**：禁用 `while` / `for` / 列表推导式，只能递归
- **Problem 7**：**禁止辅助函数**；测试前删掉 `assert False, 'Remove this line'`；超过 `limit` 立刻停止递归（有按调用次数计的测试）
- **Problem 10**：不能修改传入的 `times` 列表；玩家数不固定，不能写死两人
- **Problem EC**：cache 里存的值必须用**元组**（字典键必须不可变）

### 本地怎么跑

```bash
python3 ok -q 01          # 单题测试
python3 ok --score        # 看总分
python3 cats.py -t cats   # 命令行测打字速度
python3 cats_gui.py       # 网页版界面
python3 score.py          # Final Diff 成功率（选做）
```

### 为什么 Cats 是第二个项目

- Hog 练的是「控制流 + 高阶函数 + 闭包」；Cats 把同样的工具用在**字符串和列表**上，并第一次引入「递归 + limit 剪枝」
- 三个 Phase 对应三种数据结构：Phase 1 字符串/列表 → Phase 2 递归 → Phase 3 字典
- 题目之间有真实依赖链（Problem 9 → 10），和 Hog 一样「跳过就报错」

---

## Introduction

> **Important submission note:** For full credit:
>
> - Submit with Phases 1 and 2 complete by **Thursday 10/03**, worth 1 pt.
> - Submit with all phases complete by **Tuesday 10/08**.
>
> Try to attempt the problems in order, as some later problems will depend on earlier problems in their implementation and therefore also when running `ok` tests.
>
> The entire project can be completed with a partner. You can get 1 bonus point by submitting the entire project by **Monday 10/07**.

> **中文**：Phase 1+2 在 10/03 前提交记 1 分；全部在 10/08 前提交；10/07 前提交有 1 分加分。题目必须按顺序做，后面的题依赖前面的实现。

In this project, you will write a program that measures typing speed. Additionally, you will implement typing autocorrect, which is a feature that attempts to correct the spelling of a word after a user types it. This project is inspired by [typeracer](https://play.typeracer.com/).

> **中文**：本项目写一个打字速度测试程序，并实现「用户敲完一个词后自动纠正拼写」的功能。

---

## Final Product

Our staff solution to the project can be interacted with at [cats.cs61a.org](https://cats.cs61a.org). Feel free to try it out now. When you finish the project, you'll have implemented a significant part of this yourself, including the multiplayer mode!

> **中文**：官方成品在 cats.cs61a.org，做完项目后你实现的部分包括多人对战模式。

---

## Download Starter Files

You can download all of the project code as a [zip archive](cats.zip). This project includes several files, *but your changes will be made only to* `cats.py`. Here are the files included in the archive:

- `cats.py`: The typing test logic.
- `utils.py`: Utility functions for interacting with files and strings.
- `ucb.py`: Utility functions for CS 61A projects.
- `data/sample_paragraphs.txt`: Text samples to be typed.
- `data/common_words.txt`: Common English words in order of frequency.
- `data/words.txt`: Many more English words in order of frequency.
- `data/final_diff_words.txt`: Even more English words!
- `data/testcases.out`: Test cases for the optional Final Diff extension.
- `cats_gui.py`: A web server for the web-based graphical user interface (GUI).
- `gui_files`: A directory of files needed for the graphical user interface (GUI).
- `multiplayer`: A directory of files needed to support multiplayer mode.
- `favicons`: A directory of icons.
- `images`: A directory of images.
- `ok`, `cats.ok`, `tests`: Testing files.
- `score.py`: Part of the optional Final Diff extension.

> **中文**：整个项目只改 `cats.py` 一个文件，其余都是工具与数据。

---

## Logistics

The project is worth 20 points. 19 points are for correctness and 1 point is for submitting Phases 1 & 2 by the checkpoint date.

You will turn in the following files: `cats.py`.

You do not need to modify or turn in any other files to complete the project. To submit the project, **submit the required files to the appropriate Gradescope assignment.**

You may not use artificial intelligence tools to help you with this project or reference solutions found on the internet.

For the functions that we ask you to complete, there may be some initial code that we provide. If you would rather not use that code, feel free to delete it and start from scratch. You may also add new function definitions as you see fit.

**However, please do not modify any other functions or edit any files not listed above**. Doing so may result in your code failing our autograder tests. Also, please do not change any function signatures (names, argument order, or number of arguments).

> **中文**：总分 20（19 正确性 + 1 Checkpoint 准时）。只交 `cats.py`；不要改其他文件、不要改函数签名；不借助 AI 或网上答案。

We have provided an **autograder** called `ok` to help you with testing your code and tracking your progress. The first time you run the autograder, you will be asked to **log in with your Ok account using your web browser**. Each time you run `ok`, it will back up your work and progress on our servers.

> **中文**：`ok` 首次运行要网页登录 Ok 账号；每次运行都会把进度备份到服务器。

---

## Getting Started Videos

> 官方在此提供 YouTube 播放列表（需 berkeley.edu 账号）：`https://youtu.be/playlist?list=PLx38hZJ5RLZfQfRYgUehyb_HNRv_YCVEs`

---

## Phase 1: Typing

> **Reminder**: Throughout the project, we will only be making changes to functions in `cats.py`.

> **中文**：整个项目只修改 `cats.py` 里的函数。

### Problem 1 (1 pt)

**原题（English）**

Implement `pick`. This function selects which paragraph the user will type for the typing test. It takes three parameters:

- `paragraphs`: a list of potential paragraphs (strings)
- `select`: a function that evaluates a paragraph and returns `True` if it meets certain criteria, and `False` otherwise
- `k`: a non-negative integer representing the index of the desired paragraph among those that meet the criteria

The `pick` function returns the `k`th paragraph that satisfies the `select` function. If no such paragraph exists (because `k` is greater than or equal to the number of qualifying paragraphs), then `pick` returns an empty string.

> **中文**：`pick(paragraphs, select, k)` 返回第 `k` 个满足 `select` 的段落（索引从 0 开始）；符合条件的段落不足 `k+1` 个时返回空字符串。

**Starter**（原样摘录）

```python
def pick(paragraphs, select, k):
    """Return the Kth paragraph from PARAGRAPHS for which the SELECT returns True.
    If there are fewer than K such paragraphs, return an empty string.

    Arguments:
        paragraphs: a list of strings representing paragraphs
        select: a function that returns True for paragraphs that meet its criteria
        k: an integer

    >>> ps = ['hi', 'how are you', 'fine']
    >>> s = lambda p: len(p) <= 4
    >>> pick(ps, s, 0)
    'hi'
    >>> pick(ps, s, 1)
    'fine'
    >>> pick(ps, s, 2)
    ''
    """
    "*** YOUR CODE HERE ***"
```

**提示（Hint）**

Don't worry about the specific implementation of the `select` function. Just assume it takes a paragraph as input and returns `True` or `False`. **Reminder**: Indexing starts at 0. If `k` is 0, we want to pick the *first* qualifying paragraph.

> **中文**：不用管 `select` 怎么实现，把它当成「输入段落、输出 True/False」的黑盒；`k = 0` 表示第一个符合条件的段落。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 01
```

### Problem 2 (1 pt)

**原题（English）**

Implement the `about` function, which takes a list of `subject` words. It returns a function that, when given a paragraph, checks whether the paragraph contains any of the words from the subject list. The returned function will return `True` if any of the words in the subject list are found in the paragraph and `False` otherwise.

Once `about` is implemented, we can use the function it returns as the `select` argument in `pick`. This is useful because it allows us to filter paragraphs based on whether they contain any words from the subject list provided to the `about` function. This functionality will be useful as we continue to develop our typing test.

To ensure accurate comparisons, you will need to:

1. Ignore case (treat uppercase and lowercase letters as equivalent).
2. Ignore punctuation in the paragraph.
3. Only check for exact matches of the words in the `subject` list, not substrings. For example, instances of "dogs" in `paragraph` should not match "dog" in `subject`.

> **中文**：`about(subject)` 返回一个判断函数：段落里只要出现 `subject` 中任意一个词就返回 True。比较时必须忽略大小写、忽略标点、只认整词匹配（"dogs" 不算 "dog"）。

**Starter**（原样摘录）

```python
def about(subject):
    """Return a function that takes in a paragraph and returns whether
    that paragraph contains one of the words in SUBJECT.

    Arguments:
        subject: a list of words related to a subject

    >>> about_dogs = about(['dog', 'dogs', 'pup', 'puppy'])
    >>> pick(['Cute Dog!', 'That is a cat.', 'Nice pup!'], about_dogs, 0)
    'Cute Dog!'
    >>> pick(['Cute Dog!', 'That is a cat.', 'Nice pup.'], about_dogs, 1)
    'Nice pup.'
    """
    assert all([lower(x) == x for x in subject]), "subjects should be lowercase."
    "*** YOUR CODE HERE ***"
```

**提示（Hint）**

Use the `split`, `lower`, and `remove_punctuation` functions in `utils.py`.

> **中文**：用 `utils.py` 里的 `split`、`lower`、`remove_punctuation` 处理段落和单词。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 02
```

### Problem 3 (2 pts)

**原题（English）**

Implement `accuracy`, which takes both a `typed` paragraph and a `source` paragraph. It returns the percentage of words in `typed` that exactly match the corresponding words in `source`. Case and punctuation must match as well. "Corresponding" here means that two words must occur at the same indices in `typed` and `source`; the first words of both must match, the second words of both must match, and so on.

A *word* in this context is any sequence of characters separated from other words by whitespace. Therefore, treat sequences like "dog;" as a single word.

If `typed` is longer than `source`, then the extra words in `typed` that have no corresponding word in `source` are all *incorrect*.

If both `typed` and `source` are empty, then the accuracy is 100.0. If `typed` is empty but `source` is not empty, then the accuracy is zero. If `typed` is not empty but `source` is empty, then the accuracy is zero.

> **中文**：`accuracy(typed, source)` 返回 `typed` 中与 `source` **同位置**完全相同的词所占百分比（大小写和标点都必须一致）。分母是 `typed` 的词数；都为空返回 100.0，`typed` 非空而 `source` 为空返回 0.0。

**Starter**（原样摘录）

```python
def accuracy(typed, source):
    """Return the accuracy (percentage of words typed correctly) of TYPED
    compared to the corresponding words in SOURCE.

    Arguments:
        typed: a string that may contain typos
        source: a model string without errors

    >>> accuracy('Cute Dog!', 'Cute Dog.')
    50.0
    >>> accuracy('A Cute Dog!', 'Cute Dog.')
    0.0
    >>> accuracy('cute Dog.', 'Cute Dog.')
    50.0
    >>> accuracy('Cute Dog. I say!', 'Cute Dog.')
    50.0
    >>> accuracy('Cute', 'Cute Dog.')
    100.0
    >>> accuracy('', 'Cute Dog.')
    0.0
    >>> accuracy('', '')
    100.0
    """
    typed_words = split(typed)
    source_words = split(source)
    "*** YOUR CODE HERE ***"
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 03
```

### Problem 4 (1 pt)

**原题（English）**

Implement `wpm`, which computes the *words per minute*, a measure of typing speed, given a string `typed` and the amount of `elapsed` time in **seconds.** Despite its name, *words per minute* is not based on the number of words typed, but instead the number of groups of 5 characters, so that a typing test is not biased by the length of words. The formula for *words per minute* is the ratio of the number of characters (including spaces) typed divided by 5 (a typical word length) to the elapsed time in **minutes.**

For example, the string `"I am glad!"` contains ten characters (not including the quotation marks). The words per minute calculation uses 2 as the number of words typed (because 10 / 5 = 2). If someone typed this string in 30 seconds (half a minute), their speed would be 4 words per minute.

> **中文**：`wpm(typed, elapsed)` 按「每 5 个字符算一个词」计算：`(字符数 / 5) / (秒数 / 60)`。`"I am glad!"` 共 10 字符，30 秒打完 → 4 wpm。

**Starter**（原样摘录）

```python
def wpm(typed, elapsed):
    """Return the words-per-minute (WPM) of the TYPED string.

    Arguments:
        typed: an entered string
        elapsed: an amount of time in seconds

    >>> wpm('hello friend hello buddy hello', 15)
    24.0
    >>> wpm('0123456789',60)
    2.0
    """
    assert elapsed > 0, "Elapsed time must be positive"
    "*** YOUR CODE HERE ***"
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 04
```

After that you can test your typing speed on the command line, or try the GUI:

```bash
python3 cats.py -t cats kittens
python3 cats_gui.py
```

> **中文**：`python3 cats.py -t cats kittens` 会在「猫」主题段落上测打字速度；`python3 cats_gui.py` 打开网页版界面。

---

## Phase 2: Autocorrect

In the web-based GUI, there is an "Enable Auto-Correct" option, but right now it doesn't do anything. Let's implement automatic typo correction. Whenever the user presses the space bar, if the last word they typed doesn't match a word in the dictionary but is close to one, then that similar word will be substituted for what they typed.

> **中文**：GUI 里的「Enable Auto-Correct」现在没实现。用户按下空格时，如果刚敲的词不在词典里但很接近某个词，就用那个词替换。

### Problem 5 (2 pts)

**原题（English）**

Implement `autocorrect`, which takes a `typed_word`, a `word_list`, a `diff_function`, and a `limit`. The goal of `autocorrect` is to return the word in `word_list` that is closest to the provided `typed_word`, as determined by `diff_function`.

Specifically, `autocorrect` does the following:

- If the `typed_word` is contained inside the `word_list`, `autocorrect` returns that word.
- Otherwise, `autocorrect` returns the word from `word_list` that has the lowest difference from the provided `typed_word`. This difference is the number returned by the `diff_function`.
- However, if the lowest difference between `typed_word` and any of the words in `word_list` is greater than `limit`, then `typed_word` is returned instead. In other words, `limit` sets a maximum threshold on how severe a typo can be for it to still be corrected.

Assume that `typed_word` and all elements of `word_list` are lowercase and have no punctuation.

> **Important**: If multiple strings in `word_list` are tied for the lowest difference from `typed_word`, `autocorrect` should return the string that appears earliest (with the smallest index) in `word_list`.

A diff function takes in three arguments. The first is the `typed_word`, the second is the source word (in this case, a word from `word_list`), and the third argument is the `limit`. The output of the diff function, which is a number, represents the amount of difference between the two strings.

Here is an example of a diff function that computes the minimum of `1 + limit` and the difference in length between the two input strings:

```python
>>> def length_diff(w1, w2, limit):
...     return min(limit + 1, abs(len(w2) - len(w1)))
>>> length_diff('mellow', 'cello', 10)
1
>>> length_diff('hippo', 'hippopotamus', 5)
6
```

> **中文**：`autocorrect(typed_word, word_list, diff_function, limit)`：如果 `typed_word` 本来就在词表里，原样返回；否则返回 `diff_function` 差值最小的词；若最小差值仍大于 `limit`，返回 `typed_word` 本身。差值并列时取词表里靠前的那个。

**Starter**（原样摘录）

```python
def autocorrect(typed_word, word_list, diff_function, limit):
    """Returns the element of WORD_LIST that has the smallest difference
    from TYPED_WORD based on DIFF_FUNCTION. If multiple words are tied for the smallest difference,
    return the one that appears closest to the front of WORD_LIST. If the
    difference is greater than LIMIT, return TYPED_WORD instead.

    Arguments:
        typed_word: a string representing a word that may contain typos
        word_list: a list of strings representing source words
        diff_function: a function quantifying the difference between two words
        limit: a number

    >>> ten_diff = lambda w1, w2, limit: 10 # Always returns 10
    >>> autocorrect("hwllo", ["butter", "hello", "potato"], ten_diff, 20)
    'butter'
    >>> first_diff = lambda w1, w2, limit: (1 if w1[0] != w2[0] else 0) # Checks for matching first char
    >>> autocorrect("tosting", ["testing", "asking", "fasting"], first_diff, 10)
    'testing'
    """
    "*** YOUR CODE HERE ***"
```

**提示（Hint）**

Try using `max` or `min` with the optional `key` argument (which takes in a one-argument function). For example, `max([-7, 2, -1], key=abs)` would return `-7` since `abs(-7)` is greater than `abs(2)` and `abs(-1)`.

> **中文**：试试 `min`/`max` 的 `key` 参数（接受一个单参数函数）。`max([-7, 2, -1], key=abs)` 返回 `-7`。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 05
```

### Problem 6 (3 pts)

**原题（English）**

Implement `furry_fixes`, a diff function that could be passed into the `diff_function` parameter in `autocorrect`. This function takes in two strings and returns the minimum number of characters that must be changed in the `typed` word in order to transform it into the `source` word. If the strings are not of equal length, the difference in lengths is added to the total difference count.

Here are some examples:

```python
>>> big_limit = 10
>>> furry_fixes("nice", "rice", big_limit)    # Substitute: n -> r
1
>>> furry_fixes("range", "rungs", big_limit)  # Substitute: a -> u, e -> s
2
>>> furry_fixes("pill", "pillage", big_limit) # Don't substitute anything, length difference of 3.
3
>>> furry_fixes("roses", "arose", big_limit)  # Substitute: r -> a, o -> r, s -> o, e -> s, s -> e
5
>>> furry_fixes("rose", "hello", big_limit)   # Substitute: r->h, o->e, s->l, e->l, length difference of 1.
5
```

> **Important**: You may not use `while`, `for`, or list comprehensions in your implementation. Use recursion.

If the number of characters that must change is greater than `limit`, then `furry_fixes` should return any number larger than `limit` and should minimize the amount of computation needed to do so.

Why is there a limit? From Problem 5, we know that `autocorrect` will reject any `source` word whose difference with the `typed` word is greater than `limit`. It doesn't matter if the difference is greater than `limit` by 1 or by 100; autocorrect will reject it just the same. Therefore, as soon as we know the difference is above `limit`, it makes sense to stop making recursive calls, saving time, even if the returned difference won't be exactly correct.

These two calls to `furry_fixes` should take about the same amount of time to evaluate:

```python
>>> limit = 4
>>> furry_fixes("roses", "arose", limit) > limit
True
>>> furry_fixes("rosesabcdefghijklm", "arosenopqrstuvwxyz", limit) > limit
True
```

To ensure that you are correctly saving time by stopping the recursion after `limit` is reached, there is an autograder test that measures the performance of your solution based on the number of function calls that it makes. If you fail this test, consider adding a base case related to the `limit`.

> **中文**：`furry_fixes(typed, source, limit)` 逐字符比较，不同就 +1；长度差直接加进总数。禁用 `while`/`for`/列表推导式，必须递归。一旦差值超过 `limit` 就可以提前返回「比 limit 大的任意数」——`autocorrect` 只关心「超没超」，不关心超多少，所以剪枝能省下大量调用。

**Starter**（原样摘录）

```python
def furry_fixes(typed, source, limit):
    """A diff function for autocorrect that determines how many letters
    in TYPED need to be substituted to create SOURCE, then adds the difference in
    their lengths and returns the result.

    Arguments:
        typed: a starting word
        source: a string representing a desired goal word
        limit: a number representing an upper bound on the number of chars that must change

    >>> big_limit = 10
    >>> furry_fixes("nice", "rice", big_limit)    # Substitute: n -> r
    1
    >>> furry_fixes("range", "rungs", big_limit)  # Substitute: a -> u, e -> s
    2
    >>> furry_fixes("pill", "pillage", big_limit) # Don't substitute anything, length difference of 3.
    3
    >>> furry_fixes("roses", "arose", big_limit)  # Substitute: r -> a, o -> r, s -> o, e -> s, s -> e
    5
    >>> furry_fixes("rose", "hello", big_limit)   # Substitute: r->h, o->e, s->l, e->l, length difference of 1.
    5
    """
    assert False, 'Remove this line'
```

**提示（Hint）**

You will need more than one base case to solve this problem.

> **中文**：这题需要不止一个 base case。

**String Slicing**

A string is a sequence of characters. A slice of a string is another string that contains some of the characters of the original. Here are some examples:

```python
>>> a = 'strap'
>>> a[0]
's'
>>> a[1:]
'trap'
>>> a[2:]
'rap'
>>> a[1:][1:]
'rap'
```

> **中文**：字符串切片：`a[0]` 取首字符，`a[1:]` 去掉首字符，`a[1:][1:]` 等价于 `a[2:]`。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 06
```

### Problem 7 (3 pts)

**原题（English）**

Implement `minimum_mewtations`, a more advanced diff function that can be used in `autocorrect`, which returns the *minimum* number of edit operations needed to transform the `typed` word into the `source` word.

There are three kinds of edit operations, with some examples:

1. Add a letter to `typed`. Adding `"k"` to `"itten"` gives us `"kitten"`.
2. Remove a letter from `typed`. Removing `"s"` from `"scat"` givs us `"cat"`.
3. Substitute a letter in `typed` for another. Substituting `"z"` with `"j"` in `"zaguar"` gives us `"jaguar"`.

Each edit operation contributes 1 to the difference between two words.

```python
>>> big_limit = 10
>>> minimum_mewtations("cats", "scat", big_limit)       # cats -> scats -> scat
2
>>> minimum_mewtations("purng", "purring", big_limit)   # purng -> purrng -> purring
2
>>> minimum_mewtations("ckiteus", "kittens", big_limit) # ckiteus -> kiteus -> kitteus -> kittens
3
```

We have provided a template of an implementation in `cats.py`. You may modify the template however you want or delete it entirely.

If the number of edits required is greater than `limit`, then `minimum_mewtations` should return **any number** larger than `limit` (such as `limit + 1`) and should stop making recursive calls once the limit is reached to save time.

These two calls to `minimum_mewtations` should take about the same amount of time to evaluate:

```python
>>> limit = 2
>>> minimum_mewtations("ckiteus", "kittens", limit) > limit
True
>>> minimum_mewtations("ckiteusabcdefghijklm", "kittensnopqrstuvwxyz", limit) > limit
True
```

To ensure that your code stops making recursive calls after the `limit` is reached, there is an autograder test that measures the performance of your solution based on the number of function calls that it makes.

> **Important**: You *should not* use any helper functions in your implementation of `minimum_mewtations`. Otherwise the autograder test might fail.

> **Important**: Remember to remove the following line of code when you are ready to test your implementation: `assert False, 'Remove this line'`

> **中文**：`minimum_mewtations` 求「编辑距离」：允许增、删、改三种操作，每种记 1 分，返回把 `typed` 变成 `source` 的最少操作数。超过 `limit` 就返回任意大于 `limit` 的数并停止递归。**不能用辅助函数**，测之前要删掉 `assert False` 那行。

**Starter**（原样摘录）

```python
def minimum_mewtations(typed, source, limit):
    """A diff function for autocorrect that computes the edit distance from TYPED to SOURCE.
    This function takes in a string TYPED, a string SOURCE, and a number LIMIT.

    Arguments:
        typed: a starting word
        source: a string representing a desired goal word
        limit: a number representing an upper bound on the number of edits

    >>> big_limit = 10
    >>> minimum_mewtations("cats", "scat", big_limit)       # cats -> scats -> scat
    2
    >>> minimum_mewtations("purng", "purring", big_limit)   # purng -> purrng -> purring
    2
    >>> minimum_mewtations("ckiteus", "kittens", big_limit) # ckiteus -> kiteus -> kitteus -> kittens
    3
    """
    assert False, 'Remove this line'
    if ___________: # Base cases should go here, you may add more base cases as needed.
        # BEGIN
        "*** YOUR CODE HERE ***"
        # END
    # Recursive cases should go below here
    if ___________: # Feel free to remove or add additional cases
        # BEGIN
        "*** YOUR CODE HERE ***"
        # END
    else:
        add = ... # Fill in these lines
        remove = ...
        substitute = ...
        # BEGIN
        "*** YOUR CODE HERE ***"
        # END


# Ignore the line below
minimum_mewtations = count(minimum_mewtations)
```

**提示（Hint）**

One of the recursive calls in `minimum_mewtations` will be similar to `furry_fixes`. However, because `minimum_mewtations` considers *specific* types of edits (add, remove, substitute), there will need to be additional recursive calls to handle each of these cases.

> **中文**：其中一个递归调用和 `furry_fixes` 很像；但因为这题把「增、删、改」当作三种具体操作，所以要为每种情况各写一个递归调用。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 07
```

### (Optional) Extension: Final Diff (0 pts)

**原题（English）**

You may optionally design your own diff function called `final_diff`. Here are some ideas for making even more accurate corrections:

- Take into account which additions and deletions are more likely than others. For example, it's much more likely that you'll accidentally leave out a letter if it appears twice in a row.
- Treat two adjacent letters that have swapped positions as one change, not two.
- Try to incorporate common misspellings.
- Letters near to each other on the keyboard are more commonly substituted.

You can also set the limit you'd like your diff function to use by changing the value of the variable `FINAL_DIFF_LIMIT` in `cats.py`.

You can check your `final_diff`'s success rate on a provided dataset of common misspellings by running:

```bash
python3 score.py
```

If you don't know where to start, try copy-pasting your code for `furry_fixes` and `minimum_mewtations` into `final_diff` and scoring them. Looking at the typos they fixed (and didn't fix) might give you some ideas!

> **中文**：选做：自己设计 `final_diff`，可考虑「连写字母漏一个」「相邻字母互换算一次」「键盘就近替换」等启发式；改 `FINAL_DIFF_LIMIT` 调阈值，用 `python3 score.py` 看纠错成功率。

**答案**

（做完再补）

**验证**

```bash
python3 score.py
```

---

## Checkpoint Submission

Check to make sure that you completed all the problems in Phase 1 and Phase 2:

```bash
python3 ok --score
```

Then, submit `cats.py` to the **Cats Checkpoint** assignment on **Gradescope** before the checkpoint deadline.

When you run `ok` commands, you'll still see that some tests are locked because you haven't completed the whole project yet. You'll get full credit for the checkpoint if you complete all the problems up to this point.

> **中文**：Checkpoint 前跑 `ok --score` 确认 Phase 1+2 全过，然后交 `cats.py`；此时后面的测试仍锁着是正常的。

---

## Phase 3: Multiplayer

Typing is more fun with friends! You'll now implement multiplayer functionality, so that when you run `cats_gui.py` on your computer, it connects to the course server at [cats.cs61a.org](https://cats.cs61a.org) and looks for someone else to race against.

To race against a friend, 5 different programs will be running: your GUI; your `cats_gui.py`; your opponent's `cats_gui.py`; your opponent's GUI; and the CS 61A multiplayer server, which matches players together and passes messages around.

> **中文**：多人模式下同时跑 5 个程序：双方 GUI、双方 `cats_gui.py`、课程服务器。你的 `cats_gui.py` 负责算进度并上传。

### Problem 8 (2 pts)

**原题（English）**

Implement `report_progress`, which is called every time the user finishes typing a word. It takes a list of the words `typed`, a list of the words in the `source`, the user's `user_id`, and a `upload` function that is used to upload a progress report to the multiplayer server. There will never be more words in `typed` than in `source`.

Your progress is a ratio of the words in the `source` that you have typed correctly, up to the first incorrect word, divided by the number of `source` words. For example, this example has a progress of `0.25`:

```python
report_progress(["Hello", "ths", "is"], ["Hello", "this", "is", "wrong"], ...)
```

Your `report_progress` function should do two things: upload a message to the multiplayer server and return the progress of the player with `user_id`.

To upload a message to the multiplayer server, call the `upload` function on a two-item dictionary containing the keys `'id'` and `'progress'`. The function should then return the player's progress, which is the ratio of words you computed.

> **中文**：`report_progress(typed, source, user_id, upload)`：从第一个词开始逐词比对，遇到第一个错的就停；进度 = 正确词数 / `source` 词数。先调用 `upload({'id': user_id, 'progress': ...})`，再返回这个进度。

**Starter**（原样摘录）

```python
def report_progress(typed, source, user_id, upload):
    """Upload a report of your id and progress so far to the multiplayer server.
    Returns the progress so far.

    Arguments:
        typed: a list of the words typed so far
        source: a list of the words in the typing source
        user_id: a number representing the id of the current user
        upload: a function used to upload progress to the multiplayer server

    >>> print_progress = lambda d: print('ID:', d['id'], 'Progress:', d['progress'])
    >>> # The above function displays progress in the format ID: __, Progress: __
    >>> print_progress({'id': 1, 'progress': 0.6})
    ID: 1 Progress: 0.6
    >>> typed = ['how', 'are', 'you']
    >>> source = ['how', 'are', 'you', 'doing', 'today']
    >>> report_progress(typed, source, 2, print_progress)
    ID: 2 Progress: 0.6
    0.6
    >>> report_progress(['how', 'aree'], source, 3, print_progress)
    ID: 3 Progress: 0.2
    0.2
    """
    "*** YOUR CODE HERE ***"
```

**提示（Hint）**

See the dictionary below for an example of a potential input to the `upload` function. This dictionary represents a player with `user_id` 4 and `progress` 0.6.

`{'id': 4, 'progress': 0.6}`

> **中文**：`upload` 的入参长这样：`{'id': 4, 'progress': 0.6}`。

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 08
```

### Problem 9 (1 pt)

**原题（English）**

Implement `time_per_word`, which takes in two arguments:

1. `words`: a list of words that players are typing.
2. `timestamps_per_player`: a list of lists where each inner list contains the timestamps indicating when each player finished typing each word in `words`.

The function should return two values:

- The list of words that the players are typing.
- A list of lists `times` that stores the durations it took each player to type each word. Specifically, the value at `times[i][j]` should indicate how long it took player `i` to type the word at `words[j]`.

Timestamps found in the parameter `timestamps_per_player` are cumulative and always increasing, while the values in `times` are **differences between consecutive timestamps for each player**.

Here's an example: If `timestamps_per_player = [[1, 3, 5], [2, 5, 6]]`, then `times` would be `[[2, 2], [3, 1]]`.

This is because the first player finished typing each word at timestamps `1`, `3`, and `5`, while the second player finished typing each word at timestamps `2`, `5`, and `6`. So the differences in timestamps are `(3-1)`, `(5-3)` for the first player and `(5-2)`, `(6-5)` for the second player. The first value of each list within `timestamps_per_player` represents the initial starting time for each player.

> **中文**：`time_per_word` 返回 `{'words': words, 'times': times}`，其中 `times[i][j]` 是玩家 `i` 打第 `j` 个词花的时间。时间戳是累计值，每个玩家列表的第一个是起始时间，所以 `times` 里是**相邻时间戳之差**（每人少一项）。

**Starter**（原样摘录）

```python
def time_per_word(words, timestamps_per_player):
    """Return a dictionary {'words': words, 'times': times} where times
    is a list of lists that stores the durations it took each player to type
    each word in words.

    Arguments:
        words: a list of words, in the order they are typed.
        timestamps_per_player: A list of lists of timestamps including the time
                          the player started typing, followed by the time the
                          player finished typing each word.

    >>> p = [[75, 81, 84, 90, 92], [19, 29, 35, 36, 38]]
    >>> result = time_per_word(['collar', 'plush', 'blush', 'repute'], p)
    >>> result['words']
    ['collar', 'plush', 'blush', 'repute']
    >>> result['times']
    [[6, 3, 6, 2], [10, 6, 1, 2]]
    """
    tpp = timestamps_per_player  # A shorter name (for convenience)
    times = []  # You may remove this line
    return {'words': words, 'times': times}
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 09
```

### Problem 10 (3 pts)

**原题（English）**

Implement `fastest_words`, which returns which words each player typed fastest. This function is called once all players have finished typing. It takes in a dictionary returned by `time_per_word`.

The `fastest_words` function returns a list of lists of words, one list for each player. The list for each player contains the words they typed faster than all the other players. In the case of a tie, the player with the smallest index is considered to be the one who typed it the fastest.

For example, consider two players who typed `Just have fun`. Player 0 typed `'fun'` the fastest (3 seconds), Player 1 typed `'Just'` the fastest (4 seconds), and they tied on the word `'have'` (both took 1 second). In this case, Player 0 is considered the fastest for `'have'` because their index is smaller.

```python
>>> player_0 = [5, 1, 3]
>>> player_1 = [4, 1, 6]
>>> fastest_words({'words': ['Just', 'have', 'fun'], 'times': [player_0, player_1]})
[['have', 'fun'], ['Just']]
```

Use the helper function `get_time` (provided) to get an individual time from `times`. It provides helpful error messages when you try to access a time that doesn't exist.

```python
def get_time(times, player_num, word_index):
    """Return the time it took player_num to type the word at word_index,
    given a list of lists of times returned by time_per_word."""
```

> **Important**: Make sure your implementation does not mutate the given player input lists. For the example above, calling `fastest_words` on `[player_0, player_1]` should **not** mutate `player_0` or `player_1`.

There might not always be two players, so generalize this function in a way that will allow it to handle an indeterminate number of players.

> **中文**：`fastest_words` 为每个玩家返回「他打字最快的那批词」。同一个词并列最快时，索引小的玩家算赢。玩家数不固定，不能写死两个；**不能修改输入的 `times` 列表**；取时间用官方给的 `get_time`。

**Starter**（原样摘录）

```python
def fastest_words(words_and_times):
    """Return a list of lists indicating which words each player typed fastests.

    Arguments:
        words_and_times: a dictionary {'words': words, 'times': times} where
        words is a list of the words typed and times is a list of lists of times
        spent by each player typing each word.

    >>> p0 = [5, 1, 3]
    >>> p1 = [4, 1, 6]
    >>> fastest_words({'words': ['Just', 'have', 'fun'], 'times': [p0, p1]})
    [['have', 'fun'], ['Just']]
    >>> p0  # input lists should not be mutated
    [5, 1, 3]
    >>> p1
    [4, 1, 6]
    """
    check_words_and_times(words_and_times)  # verify that the input is properly formed
    words, times = words_and_times['words'], words_and_times['times']
    player_indices = range(len(times))  # contains an *index* for each player
    word_indices = range(len(words))    # contains an *index* for each word
    "*** YOUR CODE HERE ***"
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q 10
```

Congratulations! Now you can play against other students in the course. Set `enable_multiplayer` to `True` near the bottom of `cats.py` and type swiftly!

```bash
python3 cats_gui.py
```

> **中文**：把 `cats.py` 底部的 `enable_multiplayer` 改成 `True`，就能联机对战。

---

## Project Submission

Run `ok` on all problems to make sure all tests are unlocked and pass:

```bash
python3 ok
```

You can also check your score on each part of the project:

```bash
python3 ok --score
```

Once you are satisfied, submit this assignment by uploading `cats.py` to the **Cats** assignment on **Gradescope**.

> **中文**：全部做完后 `python3 ok` 确认无锁测试全过，`ok --score` 看分，然后到 Gradescope 交 `cats.py`。

---

## Phase 4: Efficiency (Extra Challenge)

> **Note**: This problem is **optional** and **will not worth any points**. It is meant to be a extra challenge for those who are interested in improving the efficiency of their code. **Only attempt this problem if you have completed all other problems in the project.**

> **中文**：纯选做，0 分，只在其他题全做完后再碰。

### (Optional) Problem EC (0 pt)

**原题（English）**

In this problem, you will implement memoization decorators that will increase the efficiency of our program by "remembering" the results of particularly intensive operations.

We will be working with two memoization decorators. `memo` is a general all-purpose decorator that memoizes the function it annotates. If `memo` encounters an input it has not seen, it will store the calculated result into its `cache`. If `memo` receives an input it has already seen, it will take the stored value in the `cache` and returns it directly without doing any extra computation. We have provided you with the full implementation of `memo`.

Your task is to implement `memo_diff`. `memo_diff` is a higher-order function that takes in a `diff_function` and returns another diff function called `memoized` that, like all diff functions, takes in `typed`, `source`, and `limit`. `memoized` should do the following:

- When `memoized` sees a (`typed`, `source`) pair for the first time, it should calculate the difference using `diff_function` and cache that value along with the `limit` used as a (`value`, `limit`) tuple pair.
- If `memoized` encounters the (`typed`, `source`) pair again, it should return the memoized `value` if the provided `limit` is less than or equal to the cached limit. Otherwise, the difference should be recalculated, recached, and returned.

> **Important:** When implementing this function, make sure you store pairs of values in the cache with a tuple, **not** a list. In dictionaries, keys must be *immutable* (that's why using a tuple is fine, but using a list is not).

> **中文**：`memo_diff(diff_function)` 返回一个带缓存的 diff 函数：第一次遇到 `(typed, source)` 时算差值，并把 `(value, limit)` 存进 cache；再遇到同一对词时，若新 `limit` ≤ 缓存里的 limit，直接返回缓存值，否则重算并覆盖缓存。字典的键必须不可变，所以缓存值用元组不能用列表。

**Decorators（下拉内容）**

A Python decorator allows you to modify a pre-existing function without changing the function's structure. Specifically, a decorator function is a higher-order function that:

- Takes the original function as an input
- Returns a new function with modified functionality
- This new function **must** contain the same arguments as the original function

```python
>>> def do_twice(original_function):
...     def repeat(x):
...             original_function(x)
...             original_function(x)
...     return repeat
```

> **中文**：装饰器是「输入原函数、返回改造后的新函数」的高阶函数，新函数参数必须与原函数一致。

**Memoization（下拉内容）**

Notice that the diff functions we wrote in the previous questions are very inefficient: you will likely find that the computer will make the same recursive call multiple times. Our goal is to have our program store past results of evaluated recursive calls so that we can reuse them if the same recursive call comes up in the future. This is the concept of **memoization**: store the results of expensive computations in a cache, and retrieve information from the cache in the case we execute a repeated action.

> **中文**：记忆化 = 把重复出现的子问题结果存进 cache，下次直接取，避免重复计算（`fib` 的树图就是典型例子）。

**Starter**（原样摘录）

```python
def memo(f):
    """A general memoization decorator."""
    cache = {}

    def memoized(*args):
        immutable_args = deep_convert_to_tuple(args)  # convert *args into a tuple representation
        if immutable_args not in cache:
            result = f(*immutable_args)
            cache[immutable_args] = result
            return result
        return cache[immutable_args]

    return memoized


def memo_diff(diff_function):
    """A memoization function."""
    cache = {}

    def memoized(typed, source, limit):
        # BEGIN PROBLEM EC
        "*** YOUR CODE HERE ***"
        # END PROBLEM EC

    return memoized
```

**答案**

（做完再补）

**验证**

```bash
python3 ok -q EC
```

---

## 验收

| 题号 | 状态 |
|---|---|
| Problem 1 `pick` | 未做 |
| Problem 2 `about` | 未做 |
| Problem 3 `accuracy` | 未做 |
| Problem 4 `wpm` | 未做 |
| Problem 5 `autocorrect` | 未做 |
| Problem 6 `furry_fixes` | 未做 |
| Problem 7 `minimum_mewtations` | 未做 |
| Problem 8 `report_progress` | 未做 |
| Problem 9 `time_per_word` | 未做 |
| Problem 10 `fastest_words` | 未做 |
| Problem EC `memo_diff` | 未做（选做） |

---

## 本次踩的坑

- （待补）

---

## 相关概念

- [[Homework 3 - Recursion, Tree Recursion]] — Problem 6/7 的 diff 函数是「一次调用分多支」的树递归，与 `count_dollars` 同骨架
- [[Lab 3 - Recursion, Python Lists]] — Phase 1 用到的 `split`、切片、列表遍历来自这里
- [[Discussion 4 - Tree Recursion]] — Problem 7 的三种操作对应三支递归，分支数由「操作种类」这个维度决定
- [[Project 1 - Hog]] — 同为 CS61A 项目：Hog 练高阶函数与模拟，Cats 练字符串/递归/字典
