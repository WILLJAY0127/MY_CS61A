---
proj: 01
title: Hog
status: in_progress
---

# Project 1: Hog

- **原始题目**：`SOURCE/CS61A-Website-Archive/proj/hog/index.html`
- **Starter 文件**：`MY/CS61A-Assignments/Project/hog/`

---

## 项目介绍

**本项目做什么**：用 Python 实现骰子游戏 **Hog** 的完整模拟器和多种 AI 策略。同时用到**控制语句**（循环、条件）和**高阶函数**（函数作参数、函数作返回值），对应教材 Section 1.2–1.6。

**做题顺序**：官网明确要求按顺序做，因为后面的函数会依赖前面已经写好的实现（比如 `take_turn` 要调 `roll_dice` 和 `boar_brawl`，`play` 要调 `take_turn` 和 `update`）。如果跳过了 Q1 直接做 Q5，`ok` 的构造检查会通不过。

**警告**：过去的学生里有不少人**没读清楚题目描述就直接写代码**，踩了很多不必要的坑。建议每题先通读一遍再动手。

---

### Hog 游戏规则

两名玩家交替回合，每回合当前玩家选若干骰子一起扔（最多 10 颗）。当某玩家**回合结束后**分数 ≥ `GOAL`（默认 100）时获胜。

每个回合正常得分 = 骰子点数之和。但有三条特殊规则：

**Sow Sad（至少有一颗骰子扔出 1）**

英文原文：If any of the dice outcomes is a 1, the current player's score for the turn is `1`, regardless of the other values rolled.

- **触发**：本回合扔的所有骰子中，**任意一颗**点数为 1。
- **效果**：本回合得分直接变成 `1`，其他骰子的点数**全部作废**，不参与求和。
- 例子：扔 7 颗，其中 5 颗是 1 → 得分 1；扔 4 颗全是 3 → 得分 12。

**Boar Brawl（故意扔 0 颗骰子）**

英文原文：A player who chooses to roll zero dice scores three times the absolute difference between the tens digit of the opponent's score and the ones digit of the current player's score, or 1, whichever is greater. The ones digit refers to the rightmost digit and the tens digit refers to the second-rightmost digit. If a player's score is a single digit (less than 10), the tens digit of that player's score is 0.

- **触发**：主动选择 `num_rolls = 0`（合法操作，每回合可选 0–10 颗）。
- **效果**：得分 = `max(1, 3 × abs(对手分数的十位 − 自己分数的个位))`。
- **细节**：分数 < 10 时，它的**十位视为 0**（比如自己分数 2、对手分数 5，十位都是 0）。
- 例子：自己 21、对手 46 → `3 × |4 − 1| = 9`；自己 45、对手 52 → `3 × |5 − 5| = 0` → 取 1；自己 2、对手 5 → `3 × |0 − 2| = 6`。

**Sus Fuss（回合结束后分数恰好是 sus 数）**

英文原文：We call a number *sus* if it has exactly 3 or 4 factors, including 1 and the number itself. If, after rolling, the current player's score is a sus number, their score instantly increases to the next prime number.

- **触发**：本回合正常得分计算完之后，总分恰好是 sus 数（**恰好有 3 或 4 个因子**，含 1 和自身）。
- **效果**：分数立刻跳到**比当前分数大的下一个质数**。
- **不是 sus 的数**：质数（2 个因子）、16（7 个因子）、64（7 个因子）。
- 例子：自己 14 分，扔 2 颗得 7 分 → 总分 21。21 的因子：1, 3, 7, 21（恰好 4 个）→ sus。立刻跳到 23（21 之后的下一个质数），最终得分 +9 而不是 +7。
- 例子：自己 63 分，扔 5 颗触发 Sow Sad → +1 → 总分 64。64 的因子：1, 2, 4, 8, 16, 32, 64（7 个）→ 不是 sus，分数保持 64。
- 例子：自己 49 分，扔 5 颗得 18 分 → 总分 67。67 是质数，因子只有 1 和 67（2 个）→ 不是 sus，分数保持 67。

**规则生效顺序（关键）**

1. 本回合扔骰子 / 执行 Boar Brawl → 得到**原始回合得分**
2. 原始回合得分加到自己分数上 → 得到**新总分**
3. 检查新总分是否是 sus 数 → 是的话跳到下一个质数（Sus Fuss 只在**回合结束那一刻**生效，不会在游戏中反复触发）

**覆盖知识点（Hog 会用到）**

| 语法/概念 | 在哪里首次用到 |
|---|---|
| 默认参数 `dice=six_sided`、`times_called=1000` | Q1、Q3、Q5、Q9 |
| 可变参数 `*args` + `original_function(*args)` 透传 | Q8（Lec05 正式引入） |
| 闭包返回策略函数（记住外层的 n / threshold 等） | Q6、Q10、Q11、Q12 |
| 高阶函数串联（一个函数的输出是另一个的输入） | Q9 用 Q8 的工具做实验 |
| 纯函数 vs 非纯函数 | Q0 读 dice.py，Q1 里骰子是非纯的、测试骰是确定的 |

---

**函数名词典（字面含义 → 功能）**

| 函数名 | 字面含义 | 实际功能 |
|---|---|---|
| `roll_dice` | 掷骰子 | 模拟扔 `num_rolls` 颗骰子，返回得分（含 Sow Sad） |
| `boar_brawl` | 野猪斗殴 | 扔 0 颗骰子时的特殊得分规则（Boar Brawl） |
| `take_turn` | 进行一个回合 | 回合入口：根据骰子数量自动选 `roll_dice` 或 `boar_brawl` |
| `num_factors` | 因子数量 | 返回 n 有多少个因子（含 1 和自身），判断 sus 数用 |
| `sus_points` | sus 得分 | 回合结束后检查分数是否是 sus 数，是就跳到下一个质数 |
| `simple_update` | 简单更新 | 算回合后的总分（不含 Sus Fuss） |
| `sus_update` | sus 更新 | 算回合后的总分（含 Sus Fuss） |
| `play` | 玩 | 模拟整场游戏，轮流让两个玩家走回合 |
| `always_roll` | 总是扔 | 返回策略函数：永远扔 n 颗 |
| `is_always_roll` | 是否总是扔 | 判断策略是不是"永远扔固定颗数" |
| `make_averaged` | 造一个取平均的 | 返回函数：调用原函数 n 次取平均 |
| `max_scoring_num_rolls` | 得分最高的骰子数 | 遍历 1~10 颗，找平均得分最高的颗数 |
| `boar_strategy` | 野猪策略 | Boar Brawl 得分够高就扔 0 颗 |
| `sus_strategy` | sus 策略 | 考虑 Sus Fuss 后得分够高就扔 0 颗 |
| `final_strategy` | 最终策略 | 你的竞赛策略 |
| `winner` | 赢家 | 比较两个策略谁赢 |
| `average_win_rate` | 平均胜率 | 统计策略对基线策略的平均胜率 |

**规则名词典**

| 规则名 | 字面含义 | 触发条件 |
|---|---|---|
| **Sow Sad** | 播种悲伤 | 任意骰子掷出 1 → 本回合只得 1 分 |
| **Boar Brawl** | 野猪斗殴 | 扔 0 颗骰子 → `3 × abs(对手十位 − 自己个位)` 得分 |
| **Sus Fuss** | 可疑的大惊小怪 | 回合结束分数恰好是 sus 数（3 或 4 个因子）→ 跳到下一个质数 |

记忆技巧：`boar_brawl` 两词都是 b 开头，对应"扔 0 颗时的特殊规则"；`sus` 是 suspicious（可疑的）缩写，sus 数就是"看起来可疑的数"。

---

**题目地图**（按官网导航顺序）

**Phase 1: Rules of the Game（模拟器）**

| 题号 | 函数 | 分数 | 核心动作 |
|---|---|---|---|
| 0 | 阅读 dice.py | 0 pt | 理解 `make_fair_dice` / `make_test_dice` 的用法，解锁测试 |
| 1 | `roll_dice(num_rolls, dice=six_sided)` | 2 pt | 循环调用 `dice()` exactly num_rolls 次，累加点数；中间出现 1 也**不能提前 return**，要把所有骰子都扔完才返回 1 |
| 2 | `boar_brawl(player_score, opponent_score)` | 2 pt | 不用骰子，纯数字：取对手十位、自己个位，算 `3 × abs(对手十位 − 自己个位)`，如果结果为 0 则返回 1 |
| 3 | `take_turn(num_rolls, player_score, opponent_score, dice=six_sided)` | 2 pt | 分发器：`num_rolls == 0` 调 `boar_brawl`，否则调 `roll_dice` |
| 4 | `num_factors(n)` / `sus_points(score)` / `sus_update(num_rolls, player_score, opponent_score, dice=six_sided)` | 2 pt | 三段式：① 写 `num_factors(n)` 数因子数量；② 写 `sus_points(score)` 检查 sus、是则跳下一个质数；③ 写 `sus_update` 把 Boar Brawl（`take_turn` 已包含）+ Sus Fuss（用 sus_points）合起来。`simple_update` 基础版已提供在 hog.py 里，当模板参考。 |
| 5 | `play(strategy0, strategy1, update, score0=0, score1=0, dice=six_sided, goal=GOAL)` | 4 pt | 双策略主循环：每回合调对手策略得到 `num_rolls` → `take_turn` → `update`，直到一方 ≥ goal。**注意回合切换时的分数传递方向**（自己得分 → 对手回合 → 对手得分 → 自己回合）。 |

**Phase 2: Strategies（AI 策略）**

| 题号 | 函数 | 分数 | 核心动作 |
|---|---|---|---|
| 6 | `always_roll(n)` | 2 pt | 返回一个策略函数，它忽略 score 和 opponent_score，永远返回 n（闭包）。HW02 Q3 的 make_repeater 同根。 |
| 7 | `is_always_roll(strategy, goal=GOAL)` | 2 pt | 扫描 `score ∈ [0, goal)` × `opponent_score ∈ [0, goal)` 所有组合，看 strategy 返回值是否恒定。100×100 = 10000 种组合，直接双层 for 循环。 |
| 8 | `make_averaged(original_function, times_called=1000)` | 2 pt | 高阶函数：返回一个新函数，它接收**和 original_function 相同的参数**（可变参数 `*args`），重复调 times_called 次 original_function，取平均值。首次引入 `*args` 语法。 |
| 9 | `max_scoring_num_rolls(dice=six_sided, times_called=1000)` | 2 pt | 用 `make_averaged(roll_dice, times_called)` 从 1 到 10 遍历，找平均得分最高的 num_rolls。平局返回**较小的**那个。 |
| 10 | `boar_strategy(score, opponent_score, threshold=11, num_rolls=6)` | 2 pt | 每次决策时：先算一下当前 Boar Brawl 能得多少分，如果 ≥ threshold 就返回 0（扔 0 颗触发 Boar Brawl），否则返回 num_rolls。**不考虑 Sus Fuss**。 |
| 11 | `sus_strategy(score, opponent_score, threshold=11, num_rolls=6)` | 2 pt | 在 boar_strategy 基础上**额外考虑 Sus Fuss**：扔 0 颗之后新分数会不会触发 sus → 跳质数？如果触发后分数增长更大，就仍然返回 0；否则回到正常扔 num_rolls。 |
| 12 | `final_strategy(score, opponent_score)`（可选 0 pt） | 0 pt | 综合最优策略：研究 hog 锦标赛的赢率规律，写一个胜率尽可能高的策略函数。 |

**对应讲义**：Lec04（高阶函数、默认参数、闭包）、Lec05（`*args`、可变数量参数）、教材 Section 1.6

---

## Phase 1: Rules of the Game

### Q0: Dice（0 pt，解锁测试）

**原版题目（英文）**

The `dice.py` file represents dice using non-pure zero-argument functions. These functions are non-pure because they may have different return values each time they are called, and so a side-effect of calling the function is changing what will be returned when the function is called again.

A dice function takes no arguments and returns a number from 1 to n (inclusive), where n is the number of sides on the dice.

Fair dice produce each possible outcome with equal probability. Two fair dice are already defined, `four_sided` and `six_sided`, and are generated by the `make_fair_dice` function:

```python
def make_fair_dice(sides):
    """Return a die that generates values ranging from 1 to SIDES, each with an equal chance."""
    ...

four_sided = make_fair_dice(4)
six_sided = make_fair_dice(6)
```

Test dice are deterministic: they always cycles through a fixed sequence of values that are passed as arguments. Test dice are generated by the `make_test_dice` function:

```python
def make_test_dice(...):
    """Return a die that cycles deterministically through OUTCOMES.

    >>> dice = make_test_dice(1, 2, 3)
    >>> dice()
    1
    >>> dice()
    2
    >>> dice()
    3
    >>> dice()
    1
    >>> dice()
    2
    """
```

两种骰子对比：

| 类型 | 函数 | 行为 |
|---|---|---|
| 公平骰 | `four_sided` / `six_sided`（由 `make_fair_dice(sides)` 生成） | 随机 1~sides，每次调用独立 |
| 测试骰 | `make_test_dice(1, 2, 3)` | 循环返回 1 → 2 → 3 → 1 → 2 → ...，**确定性的**，用来跑 doctest |

解锁命令（**Windows 上别用 Ctrl-C 退出解锁器，用 `exit()`**）：

```
python3 ok -q 00 -u
```

**验证结果（已通过）**

Suite 1 —— 测试骰的循环语义：

```
>>> test_dice = make_test_dice(4, 1, 2)
>>> test_dice()   # 第 1 次 → 4
>>> test_dice()   # 第 2 次 → 1
>>> test_dice()   # 第 3 次 → 2
>>> test_dice()   # 第 4 次 → 4（回到开头）
```

结论：测试骰**内部有状态**，每次调用后前进一格。这正是 Q1 要求「即使中途掷出 1，也必须把 `num_rolls` 次全部调用完」的原因——少调一次，后续序列就整体错位。

Suite 2 —— 选择题「怎样掷一次公平六面骰？」，正确答案 `4) six_sided()`：

| 选项 | 判断 | 原因 |
|---|---|---|
| `six_sided(1)` | ✗ | 骰子函数不收参数，传参报 TypeError |
| `make_test_dice(6)` | ✗ | 造出的是永远出 6 的测试骰，不是掷一次 |
| `make_fair_dice(6)` | ✗ | 返回骰子函数本身，尚未掷 |
| `six_sided` | ✗ | 只是函数对象 |
| `six_sided()` | ✅ | 调用零参数函数，返回 1~6 |
| `six_sided(6)` | ✗ | 多传参数，TypeError |

**核心区分**：`six_sided` 是「骰子」，`six_sided()` 才是「掷骰子」。`roll_dice` 里写 `dice` 与 `dice()` 的差别就在这里。

---

### Q1: roll_dice（2 pt）

**原版题目（英文）**

Implement the `roll_dice` function in `hog.py`. It takes two arguments: a positive integer called `num_rolls`, which specifies the number of times to roll a die, and a `dice` function. It returns the number of points scored by rolling the die that number of times in a turn: either the sum of the outcomes or 1 (*Sow Sad*).

- **Sow Sad**. If any of the dice outcomes is a 1, the current player's score for the turn is `1`, regardless of the other values rolled.

To obtain a single outcome of a dice roll, call `dice()`. You should call `dice()` **exactly `num_rolls` times** in the body of `roll_dice`.

Remember to call `dice()` exactly `num_rolls` times **even if Sow Sad happens in the middle of rolling**. By doing so, you will correctly simulate rolling all the dice together (and the user interface will work correctly).

**Note on default argument values**: The function heading is `def roll_dice(num_rolls, dice=six_sided): ...`. The argument `dice=six_sided` indicates that the `dice` parameter is **optional**. Calling `roll_dice(3, four_sided)` simulates rolling 3 four-sided dice; calling `roll_dice(3)` simulates rolling 3 six-sided dice due to the default argument.

**hog.py 代码模板**

```python
def roll_dice(num_rolls, dice=six_sided):
    """Simulate rolling the DICE exactly NUM_ROLLS > 0 times. Return the sum of
    the outcomes unless any of the outcomes is 1. In that case, return 1.

    num_rolls:       The number of dice rolls that will be made.
    dice:            A function that simulates a single dice roll outcome.
    """
    # These assert statements ensure that num_rolls is a positive integer.
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls > 0, 'Must roll at least once.'
    # BEGIN PROBLEM 1
    "*** YOUR CODE HERE ***"
    # END PROBLEM 1
```

**Ok 验证**：`python3 ok -q 01 -u` → 解锁 → `python3 ok -q 01`

**解锁验证（已通过）**

| Case | 输入 | 正确输出 | 考点 |
|---|---|---|---|
| 1 | `roll_dice(2, make_test_dice(4, 6, 1))` | `10` | 正常求和：4+6 |
| 2 | `roll_dice(3, make_test_dice(4, 6, 1))` | `1` | **Sow Sad**：出现 1，返回 1 而不是总和 11 |
| 3 | `roll_dice(4, make_test_dice(2, 2, 3))` | `9` | 正常求和：2+2+3+2（第 4 次循环回到 2） |
| 4 | `a = roll_dice(4, make_test_dice(1, 2, 3))` → `a` | `1` | 返回值不打印，直接用变量接 |
| 5 | `roll_dice(3, counted_dice)` 后再 `roll_dice(1, counted_dice)` | 第一次 `1`，第二次 `6` | **状态保持**：测试骰内部有状态，第一次消耗了 4,1,2，第二次从 6 开始 |
| 6 | `roll_dice(9, make_test_dice(6))` | `54` | 正常求和：6×9 |
| 6 | `roll_dice(7, make_test_dice(2, 2, 2, 2, 2, 2, 1))` | `1` | Sow Sad：最后一颗是 1，前面的累加全部作废 |

**解锁时两个易错点**：

- **Case 2**：容易答成 `11`（4+6+1 的总和），但 Sow Sad 规则下只要出现 1 就返回 `1`，其他点数全部作废。
- **Case 5**：容易答成 `4`，但 `counted_dice` 在第一次 `roll_dice(3, ...)` 时已经消耗了 `4, 1, 2` 三个值，第二次调用从序列的下一个值 `6` 开始。这就是题目反复强调「必须精确调用 `num_rolls` 次」的原因——少调一次，下一次的骰子序列就错位了。

**核心约束（官网强调）**：**即使中途出现 1，也必须把所有 num_rolls 颗骰子都扔完**，不能提前 return。因为骰子是非纯函数，每次调用都会消耗一个"随机数"，少调用会导致后续骰子序列错位。

**答案代码**

```python
def roll_dice(num_rolls, dice=six_sided):
    """Simulate rolling the DICE exactly NUM_ROLLS > 0 times. Return the sum of
    the outcomes unless any of the outcomes is 1. In that case, return 1.

    num_rolls:       The number of dice rolls that will be made.
    dice:            A function that simulates a single dice roll outcome.
    """
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls > 0, 'Must roll at least once.'
    total = 0
    sow_sad = False
    while num_rolls > 0:
        points = dice()
        total += points
        if points == 1:
            sow_sad = True
        num_rolls -= 1
    return 1 if sow_sad else total
```

**逐步思路**

1. 开循环 `while num_rolls > 0` 或 `for _ in range(num_rolls)`，每次 `dice()` 拿到一个点数。
2. 累加进一个 total，同时**如果有任意一个点数是 1，就设一个标记**（比如 `one_rolled = True`）。
3. 循环跑完之后再判断：如果标记为 True → 返回 1；否则返回 total。
4. 这个"先把所有骰子扔完、再根据标记决定结果"的模式是 HW02 没有的新写法——HW02 的循环可以提前 break / return，这里不行。

**关联知识点**

- 默认参数 `dice=six_sided`：调用 `roll_dice(3)` 等价于 `roll_dice(3, six_sided)`；HW02 里你已经见过 `dice=six_sided` 和 `start` 参数的用法，但 Q1 的默认参数是**函数**而非整数，这点更灵活。
- 非纯函数的正确用法：不能依赖"提前退出"来减少调用次数——因为 `dice()` 每次调用都会消耗一个状态，少调用会让测试骰的循环序列错位（比如本来该扔到第 5 颗骰子，你在第 3 颗就 return 了，后续扔骰的结果全变了）。
- 延迟判断（post-check）模式：循环跑完再决定返回什么，而不是循环里遇到条件就 return。这是写模拟器的一个通用思路——先把所有状态收集齐，再统一处理。

**易错点**

- **遇到 1 直接 return 1**：违反"必须 exactly num_rolls 次调用"的约束，ok 会挂。
- **初始值写错**：累加初始 0，正确；写成 1 会把结果多 +1。
- **忘了默认参数**：调用方可能只传 `roll_dice(3)`，函数签名必须带 `dice=six_sided`。

---

### Q2: boar_brawl（2 pt）

**原版题目（英文）**

Implement `boar_brawl`, which takes the player's current score `player_score` and the opponent's current score `opponent_score`, and returns the number of points scored when the player rolls 0 dice and Boar Brawl is invoked.

- **Boar Brawl**. A player who chooses to roll zero dice scores three times the absolute difference between the tens digit of the opponent's score and the ones digit of the current player's score, or 1, whichever is greater. The ones digit refers to the rightmost digit and the tens digit refers to the second-rightmost digit. If a player's score is a single digit (less than 10), the tens digit of that player's score is 0.

Don't assume that scores are below 100. Write your `boar_brawl` function so that it works correctly for any non-negative score.

**Important**: Your implementation should **not** use `str`, lists, or contain square brackets `[` `]`. The test cases will check if those have been used.

**hog.py 代码模板**

```python
def boar_brawl(player_score, opponent_score):
    """Return the points scored by rolling 0 dice according to Boar Brawl.

    player_score:     The total score of the current player.
    opponent_score:   The total score of the other player.
    """
    # BEGIN PROBLEM 2
    "*** YOUR CODE HERE ***"
    # END PROBLEM 2
```

**Ok 验证**：`python3 ok -q 02 -u` → `python3 ok -q 02`

**解锁验证（已通过）**

| Case | 输入 | 正确输出 | 计算过程 |
|---|---|---|---|
| 1 | `boar_brawl(21, 46)` | `9` | 对手十位=4, 自己个位=1, 3×\|4-1\|=9 |
| 2 | `boar_brawl(52, 79)` | `15` | 对手十位=7, 自己个位=2, 3×\|7-2\|=15 |
| 3 | `boar_brawl(0, 0)` | `1` | 3×\|0-0\|=0 → max(1,0)=1 |
| 4 | `boar_brawl(0, 5)` | `1` | 对手十位=0, 自己个位=0, 3×0=0 → max(1,0)=1 |
| 5 | `boar_brawl(2, 5)` | `6` | 对手十位=0, 自己个位=2, 3×\|0-2\|=6 |
| 6 | `boar_brawl(7, 2)` | `21` | 对手十位=0, 自己个位=7, 3×\|0-7\|=21 |
| 10 | `boar_brawl(72, 29)` | `1` | 对手十位=2, 自己个位=2, 3×\|2-2\|=0 → max(1,0)=1 |

**解锁易错点**：Case 1 容易答成 `15`——如果搞反成「自己十位 − 对手个位」，就是 3×\|2-6\|=12 或者更混乱；正确的是「对手十位 4 − 自己个位 1」= 3×3=9。

**答案代码**

```python
def boar_brawl(player_score, opponent_score):
    """Return the points scored by rolling 0 dice according to Boar Brawl.

    player_score:     The total score of the current player.
    opponent_score:   The total score of the other player.
    """
    player_ones = player_score % 10
    opponent_tens = (opponent_score // 10) % 10
    return max(1, 3 * abs(opponent_tens - player_ones))
```

**逐步思路**

1. opponent_tens = `(opponent_score // 10) % 10`（取右数第二位，即十位；超过 100 也正确）
2. player_ones = `player_score % 10`（取模取个位）
3. raw = `3 * abs(opponent_tens - player_ones)`
4. return `max(1, raw)`（0 也要返回 1）

**关联知识点**

- 十进制拆位三件套（HW02 Q4 用过同一件）：`n // 10` 砍末位、`n % 10` 取末位、`n > 0` 作循环条件——HW02 是连续拆 n 的所有位，这里是**一次性**拆两个数字的特定位，不需要循环。
- `max(1, raw)` 的作用：Boar Brawl 规定 3×|差|=0 时（比如 55 vs 45 都是个位十位相等）最少也得 1 分，不是 0 分。这也是 Sow Sad 的 "至少 1 分" 思想贯穿 Hog 全部规则。

**易错点**

- **整数除法用 `/`**：`/` 结果是 float（`46 / 10 = 4.6`），`//` 才是整数（`46 // 10 = 4`）。Python 3 里这点不能混。
- **搞反 player 和 opponent**：公式是 opponent_tens（对手分数的十位） − player_ones（自己分数的个位），**不是** player_tens − opponent_ones。第三个 doctest `boar_brawl(2, 5)` 自己个位=2、对手十位=0，3×|0-2|=6——如果搞反成自己十位=0、对手个位=5，3×|0-5|=15 就错了。
- **忘了 `max(1, ...)`**：3×|差|=0 时直接返回 0 会挂 doctest。

---

### Q3: take_turn（2 pt）

**原版题目（英文）**

Implement the `take_turn` function, which returns the number of points scored for a turn by rolling the given `dice` `num_rolls` times.

Your implementation of `take_turn` should call both the `roll_dice` and `boar_brawl` functions rather than repeating their implementations.

**hog.py 代码模板**

```python
def take_turn(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the points scored on a turn rolling NUM_ROLLS dice when the
    player has PLAYER_SCORE points and the opponent has OPPONENT_SCORE points.

    num_rolls:       The number of dice rolls that will be made.
    player_score:    The total score of the current player.
    opponent_score:  The total score of the other player.
    dice:            A function that simulates a single dice roll outcome.
    """
    # Leave these assert statements here; they help check for errors.
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls >= 0, 'Cannot roll a negative number of dice in take_turn.'
    assert num_rolls <= 10, 'Cannot roll more than 10 dice.'
    # BEGIN PROBLEM 3
    "*** YOUR CODE HERE ***"
    # END PROBLEM 3
```

**Ok 验证**：`python3 ok -q 03 -u` → `python3 ok -q 03`

**解锁验证（已通过）**

| Case | 输入 | 正确输出 | 走哪条路 |
|---|---|---|---|
| 1 | `take_turn(2, 7, 27, make_test_dice(4, 5, 1))` | `9` | `num_rolls=2 > 0` → `roll_dice(2, ...)` = 4+5 = 9 |
| 2 | `take_turn(3, 15, 9, make_test_dice(4, 6, 1))` | `1` | `num_rolls=3 > 0` → `roll_dice(3, ...)` 出现 1 → Sow Sad → 1 |
| 3 | `take_turn(0, 12, 41)` | `6` | `num_rolls=0` → `boar_brawl(12, 41)`：对手十位=4, 自己个位=2, 3×\|4-2\|=6 |

**答案代码**

```python
def take_turn(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the points scored on a turn rolling NUM_ROLLS dice when the
    player has PLAYER_SCORE points and the opponent has OPPONENT_SCORE points.

    num_rolls:       The number of dice rolls that will be made.
    player_score:    The total score of the current player.
    opponent_score:  The total score of the other player.
    dice:            A function that simulates a single dice roll outcome.
    """
    assert type(num_rolls) == int, 'num_rolls must be an integer.'
    assert num_rolls >= 0, 'Cannot roll a negative number of dice in take_turn.'
    assert num_rolls <= 10, 'Cannot roll more than 10 dice.'
    points = 0
    if num_rolls == 0:
        points = boar_brawl(player_score, opponent_score)
    else:
        points = roll_dice(num_rolls, dice)
    return points
```

**逐步思路**

Q1/Q2 都实现好之后，这题就是一个 `if/else` 分发器，难度很低：`num_rolls == 0` 走 Boar Brawl，否则走 `roll_dice`（Sow Sad 已内置）。

**关联知识点**

- 把两个子规则合成一个入口函数——Q4 `simple_update` / `sus_update`、Q5 `play` 都会调 `take_turn`，这是项目里"组合小函数"的常见模式。
- 默认参数继续传下去：`take_turn` 的 `dice=six_sided` 被 `roll_dice` 接收，形成一条默认参数传递链。

**易错点**

- **参数顺序搞反**：`boar_brawl(player_score, opponent_score)`，自己分数在前，对手在后。
- **漏掉 `dice` 参数**：`roll_dice(num_rolls)` 不传骰子会用默认 six_sided——对，但题目要求你把 dice 参数**透传**下去：`roll_dice(num_rolls, dice)`。ok 的构造检查可能卡这点。

---

### Q4: num_factors / sus_points / sus_update（2 pt）

**原版题目（英文）**

First, implement `num_factors`, which takes in a positive integer `n` and determines the number of factors that `n` has. **1 and `n` are both factors of `n`!**

After, implement `sus_points` and `sus_update`:

- `sus_points` takes in a player's score and returns the player's new score after applying the Sus Fuss rule, even if the score remains unchanged. For example, `sus_points(5)` should return `5` and `sus_points(21)` should return `23`. You should use `num_factors` and the provided `is_prime` function in your implementation.
- `sus_update` returns a player's *total score* after they roll `num_rolls` dice, taking both Boar Brawl and Sus Fuss into account. You should use `sus_points` in this function.

**Hints**:

- You can look at the implementation of `simple_update` provided in `hog.py` and use that as a starting point for your `sus_update` function.
- Recall that `take_turn` already took the Boar Brawl rule into consideration!

- **Sus Fuss**. We call a number *sus* if it has exactly 3 or 4 factors, including 1 and the number itself. If, after rolling, the current player's score is a sus number, their score instantly increases to the next prime number.

**hog.py 代码模板**（`is_prime` 已提供，你写另外三个）

```python
def is_prime(n):
    """Return whether N is prime."""
    if n == 1:
        return False
    k = 2
    while k < n:
        if n % k == 0:
            return False
        k += 1
    return True

def num_factors(n):
    """Return the number of factors of N, including 1 and N itself."""
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    # END PROBLEM 4

def sus_points(score):
    """Return the new score of a player taking into account the Sus Fuss rule."""
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    # END PROBLEM 4

def sus_update(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the total score of a player who starts their turn with
    PLAYER_SCORE and then rolls NUM_ROLLS DICE, *including* Sus Fuss.
    """
    # BEGIN PROBLEM 4
    "*** YOUR CODE HERE ***"
    # END PROBLEM 4
```

**Ok 验证**：`python3 ok -q 04 -u` → `python3 ok -q 04` → **27/27 通过**

**解锁验证（已通过）**

| Case | 输入 | 正确输出 | 计算过程 |
|---|---|---|---|
| 1 | `num_factors(1)` | `1` | 只有 1 |
| 2 | `num_factors(2)` | `2` | 1, 2 |
| 3 | `num_factors(3)` | `2` | 1, 3（质数都是 2 个因子） |
| 4 | `num_factors(9)` | `3` | 1, 3, 9（平方数，3 只算一次） |
| 5 | `num_factors(28)` | `6` | 1, 2, 4, 7, 14, 28 |
| 6 | `num_factors(64)` | `7` | 1, 2, 4, 8, 16, 32, 64 |
| 8 | `num_factors(97)` | `2` | 97 是质数 → 1, 97 |
| 10 | `sus_points(1)` | `1` | 1 只有 1 个因子 → 不是 sus → 原样返回 |
| 11 | `sus_points(21)` | `23` | 21 有 4 个因子(1,3,7,21) → sus → 下一个质数 23 |
| 13 | `sus_points(62)` | `67` | 62 有 4 个因子(1,2,31,62) → sus → 下一个质数 67（跳过 63/64/65/66） |
| 14 | `sus_points(64)` | `64` | 64 有 **7** 个因子 → 不是 sus → 保持 64 |
| 19 | `simple_update(2, 5, 7, make_test_dice(2, 4))` | `11` | 5 + 2 + 4 = 11 |
| 19 | `sus_update(2, 5, 7, make_test_dice(2, 4))` | `11` | 11 有 2 个因子 → 不是 sus → 保持 11 |
| 20 | `simple_update(0, 15, 37)` | `21` | 15 + boar_brawl(15,37)：3×\|3−5\|=6 → 21 |
| 20 | `sus_update(0, 15, 37)` | `23` | 21 有 4 个因子 → sus → 下一个质数 23 |
| 21 | `simple_update(2, 2, 3, make_test_dice(4))` | `10` | 2 + 4 + 4 = 10 |
| 21 | `sus_update(2, 2, 3, make_test_dice(4))` | `11` | 10 有 4 个因子(1,2,5,10) → sus → 下一个质数 11 |

**Case 19-21 的提问陷阱**：题目注释写 `is 11 a sus number?`，但要求填的**不是** `Yes`/`No`/`True`/`False`，而是 **`sus_update(...)` 的返回数字**。填 `n`、`no`、`Yes`、`Y`、`False` 全部报 Not quite，填 `11` 才 OK。

**答案代码**

```python
def num_factors(n):
    """Return the number of factors of N, including 1 and N itself."""
    count = 0
    i = 1
    while i <= n:
        if n % i == 0:
            count += 1
        i += 1
    return count

def sus_points(score):
    """Return the new score of a player taking into account the Sus Fuss rule."""
    if num_factors(score) in (3, 4):
        i = score + 1
        while not is_prime(i):
            i += 1
        score = i
    return score

def sus_update(num_rolls, player_score, opponent_score, dice=six_sided):
    """Return the total score of a player who starts their turn with
    PLAYER_SCORE and then rolls NUM_ROLLS DICE, *including* Sus Fuss.
    """
    score = player_score + take_turn(num_rolls, player_score, opponent_score, dice)
    return sus_points(score)
```

**逐步思路**

- **is_prime**：你已经会了。边界：n < 2 返回 False；n == 2 返回 True；n 为偶数返回 False；只需要试到 `sqrt(n)`（优化，不过不优化也能过）。
- **num_factors**：初始 count = 0；`i = 1` 开始，`while i <= n: if n % i == 0: count += 1; i += 1`。`num_factors(1) = 1`（只有 1），`num_factors(21) = 4`（1,3,7,21），`num_factors(64) = 7`。
- **sus_points**：先算 `nf = num_factors(score)`，如果 `nf == 3 or nf == 4`，就从 `candidate = score + 1` 开始递增，找到第一个 `is_prime(candidate)` 的 candidate 返回。如果不 sus，直接 return score。
- **sus_update**：一行透传：`return sus_points(player_score + take_turn(num_rolls, player_score, opponent_score, dice))`

**关联知识点**

- HW02/HW01 代码复用到新项目：`is_prime` 是你 disc01 Q3 写过的，这里原样拿过来用——这就是 CS61A 作业设计的连贯性。
- 函数串联：`sus_update` → `take_turn` → `roll_dice`/`boar_brawl`；`sus_points` → `num_factors` / `is_prime`——这是嵌套调用的经典形态。
- "sus 数"定义要准确：是**恰好**3 或 4 个因子（含 1 和自身）。质数有 2 个因子 → 不算 sus；4 有 3 个因子（1,2,4）→ sus；21 有 4 个 → sus。

**易错点**

- **`num_factors` 的 `i` 从 0 开始**：`n % 0` 直接 `ZeroDivisionError: division by zero`。因子最小值是 1，必须 `i = 1`。
- **`sus_points` 里赋值给了不存在的变量**：循环变量是 `i`，却写成 `score = n`，触发 `NameError: name 'n' is not defined`。应该 `score = i`。
- **sus 判断写成 `num_factors(score) <= 4`**：5 个因子的数（比如 16）会被误判成 sus。
- **next prime 从 score 开始找**：应该从 score+1 开始，因为 score 本身就是 sus 数、要跳到"下一个"质数。
- **`is_prime` 边界没处理 n < 2**：sus_points 里 score 可能是 21（sus），那 next prime 是 23；但你如果 sus_points(2) 时 score 已经是质数了（2 有 2 个因子，不是 sus），直接返回 2 就行。

---

### Q5: play（4 pt）

**原版题目（英文）**

Finally, implement the `play` function, which simulates a game and returns the final scores of both players, with Player 0's score first, and Player 1's score second.

A strategy function, such as `always_roll_5`, takes the current player's score and their opponent's score and returns the number of dice the current player chooses to roll.

An update function, such as `sus_update` or `simple_update`, takes the number of dice to roll, the current player's score, the opponent's score, and the dice function used to simulate rolling dice. It returns the updated score of the current player after they take their turn.

Here are some hints:

1. You'll need a loop, since players take turns until the game ends.
2. If a player's score after rolling is greater than or equal to the goal, the game should immediately end and return both scores.
3. You will need to figure out how to switch between players.

*Important*: `strategy0` and `strategy1` may **not** always return the same value each turn!

**hog.py 代码模板**

```python
def play(strategy0, strategy1, update,
         score0=0, score1=0, dice=six_sided, goal=GOAL):
    """Simulate a game and return the final scores of both players, with
    Player 0's score first and Player 1's score second.

    E.g., play(always_roll_5, always_roll_5, sus_update) simulates a game in
    which both players always choose to roll 5 dice on every turn and the Sus
    Fuss rule is in effect.

    A strategy function, such as always_roll_5, takes the current player's
    score and their opponent's score and returns the number of dice the current
    player chooses to roll.

    An update function, such as sus_update or simple_update, takes the number
    of dice to roll, the current player's score, the opponent's score, and the
    dice function used to simulate rolling dice. It returns the updated score
    of the current player after they take their turn.

    strategy0: The strategy for player0.
    strategy1: The strategy for player1.
    update:    The update function (used for both players).
    score0:    Starting score for Player 0
    score1:    Starting score for Player 1
    dice:      A function of zero arguments that simulates a dice roll.
    goal:      The game ends and someone wins when this score is reached.
    """
    who = 0  # Who is about to take a turn, 0 (first) or 1 (second)
    # BEGIN PROBLEM 5
    "*** YOUR CODE HERE ***"
    # END PROBLEM 5
    return score0, score1
```

**Ok 验证**：`python3 ok -q 05 -u` → `python3 ok -q 05` → **104/104 通过**

**解锁验证（已通过）**

Suite 1 是 4 道概念选择题（Suite 2 的 100 个 Case 自动解锁）：

| Case | 题目 | 正确选项 | 你第一次选的 | 说明 |
|---|---|---|---|---|
| 1 | 循环的 while 条件？ | `0) score0 and score1 都 < goal` | ✅ 0 | 双方都没到 goal 才继续 |
| 2 | 什么是 strategy？ | `2) 返回骰子数的函数` | ❌ 0 → ✅ 2 | strategy 是**函数**，不是一个数字 |
| 3 | player1 的策略怎么调用？ | `2) strategy1(score1, score0)` | ✅ 2 | 自己分数在前（score1 是 player1） |
| 4 | `strategy0 = lambda x, y: ((y%10)*(x%10))%10`，score0=55, score1=22，player0 扔几颗？ | `0) 0` | ✅ 0 | x=自己=55, y=对手=22 → `((22%10)*(55%10))%10 = (2*5)%10 = 0` |

**Case 2 易错点**：容易选成 `0) 骰子数`。但 strategy 是**函数**——它接收分数、返回骰子数。`always_roll_5` 就是一个 strategy 函数，不是数字 5。

**答案代码**

```python
def play(strategy0, strategy1, update,
         score0=0, score1=0, dice=six_sided, goal=GOAL):
    """Simulate a game and return the final scores of both players, with
    Player 0's score first and Player 1's score second.

    E.g., play(always_roll_5, always_roll_5, sus_update) simulates a game in
    which both players always choose to roll 5 dice on every turn and the Sus
    Fuss rule is in effect.

    A strategy function, such as always_roll_5, takes the current player's
    score and their opponent's score and returns the number of dice the current
    player chooses to roll.

    An update function, such as sus_update or simple_update, takes the number
    of dice to roll, the current player's score, the opponent's score, and the
    dice function used to simulate rolling dice. It returns the updated score
    of the current player after they take their turn.

    strategy0: The strategy for player0.
    strategy1: The strategy for player1.
    update:    The update function (used for both players).
    score0:    Starting score for Player 0
    score1:    Starting score for Player 1
    dice:      A function of zero arguments that simulates a dice roll.
    goal:      The game ends and someone wins when this score is reached.
    """
    who = 0  # Who is about to take a turn, 0 (first) or 1 (second)
    while score0 < goal and score1 < goal:
        if who == 0:
            num0 = strategy0(score0, score1)
            score0 = update(num0, score0, score1, dice)
        else:
            num1 = strategy1(score1, score0)
            score1 = update(num1, score1, score0, dice)
        who = 1 - who
    return score0, score1
```

**逐步思路**

Q1-Q4 都是在写"单回合规则"，Q5 要把它们串成游戏主循环：

1. 循环条件是**双方都没到 goal 才继续**：`while score0 < goal and score1 < goal`
2. 每回合按 `who` 分支，`who=0` 调 player0 的策略和 update，`who=1` 调 player1 的
3. **参数顺序永远是"自己在前、对手在后"**：player0 回合 → `strategy0(score0, score1)`；player1 回合 → `strategy1(score1, score0)`
4. 换人用 `who = 1 - who`（0 和 1 是二元集合，减法翻转）

**关联知识点**

- 双策略主循环是**函数式游戏引擎**的雏形——strategy 函数决定动作、update 函数决定得分规则、play 函数负责循环调度。这是把"规则"和"策略"分离的基本架构，比写死在 play 里强得多。
- 可变默认参数陷阱：`score0=0, score1=0` 是整数、不可变，所以不会有经典 Python mutable default argument bug（如果是 `[]` 就会出问题），这里安全。

**易错点**

- **while 条件的 `and` vs `or`**（真实踩坑）：
  - `while score0 < goal or score1 < goal` → 有人到了 goal 还继续跑，**死循环**（Fuzz Testing 无限超时退出码 -1）
  - `while score0 >= goal or score1 >= goal` → 一回合都不执行，直接返回 (0, 0)
  - **正确**：`while score0 < goal and score1 < goal` —— 双方都没到 goal 才继续
- **strategy 参数顺序反了**：player1 调用 `strategy1(score0, score1)` 传反了变成自己分数作第二个参数、对手分数作第一个——这样策略就会把自己分数当成对手分数来判断。
- **`who = 1 - who` 写法想不到**：最笨的写法是 `if who == 0: who = 1 else: who = 0`，也能过。`1 - who` 是 0/1 翻转的小 trick。

---

## Interlude: User Interfaces（官网章节 · 非作业）

官网原文开头：**"There are no required problems in this section of the project, just some examples for you to read and understand."** —— 不占分，不用写代码，读一下 `hog_ui.py` 理解它怎么用高阶函数包装你的代码，然后跑起来玩一局。

### 核心思想：用高阶函数"替换"你的代码

我们写了一个模拟器，但它是"哑巴"的——没有任何代码告诉它怎么把游戏事件显示给人。CS61A 的做法是：**不修改你 Phase 1 写的任何一行代码，用高阶函数把原来的函数一个个"包"起来，包装后的版本在调用原函数的同时额外 print**。

原来的 `play` 调用：

```
strategy0 → update → dice
```

包装后的 `play_and_print` 调用：

```
printing_strategy(0, strategy0) → sus_update_and_print → printing_dice(dice)
     ↑ 打印分数和骰子数            ↑ 打印骰子结果+Sus Fuss    ↑ 打印每颗骰子的点数
```

**注意 `play_and_print` 调的还是你 Phase 1 写的 `play`**——只是传进去的三个参数（strategy、update、dice）被换成了会 print 的版本。这就是 Phase 2 要学的高阶函数威力：**同一个 `play` 函数，换一套参数就能打印、就能交互、就能可视化**。

### 三种包装函数（`hog_ui.py`）

| 包装器 | 包装谁 | 做了什么 |
|---|---|---|
| `printing_strategy(who, strategy)` | strategy | 调用原策略拿到骰子数，同时 print `"The score is X to Y and Player W rolls N dice..."` |
| `printing_dice(dice)` | dice | 调用原骰子函数，同时 print 点数（空格分隔） |
| `sus_update_and_print` | update | 调 `take_turn` 打印骰子结果，调 `sus_points` 打印 Sus Fuss 是否触发 |

### 跑起来玩一局

```powershell
# 0 人自动对战：always_roll_5 vs always_roll_5
python3 hog_ui.py

# 1 人（你 vs AI）：你手动输入每回合扔几颗
python3 hog_ui.py -n 1

# 2 人（你 vs 朋友）：两人轮流输入
python3 hog_ui.py -n 2
```

`-n` 参数对应 `hog_ui.py` 里的 `play_with` 函数，它会把对应的 strategy 传给 `play_and_print`。

### 交互式输入怎么实现的

官网原文提到用 Python 内置函数做了一个很简单的交互式策略：

```python
def interactive_strategy(who):
    def strategy(score, opponent_score):
        print('Player', who, ', you have', score, 'and your opponent has', opponent_score)
        choice = input('How many dice will you roll? ')
        return int(choice)
    return strategy
```

`input()` 暂停终端等你打字，返回字符串；`int()` 把字符串转成整数。`get_int` 加了一个校验循环，确保输入在 0~10 之间。

### 输出长这样

```
The score is 2 to 23 and Player 0 rolls 5 dice...
  [ 2 3 3 2 1 ] => 1; 2 + 1 = 3
  ↑ Sow Sad：掷出 1，5 颗全白扔，只得 1 分

The score is 1 to 1 and Player 1 rolls 5 dice...
  [ 6 4 3 5 2 ] => 20; 1 + 20 = 21 triggering **Sus Fuss**, increasing to 23
  ↑ Sus Fuss：21 有 4 个因子，跳到下一个质数 23

The final score is Player 0: 72 vs Player 1: 120
```

### GUI（`hog_gui.py`，选做）

浏览器图形界面，原理和 `hog_ui.py` 一样——也是包装你的 Phase 1 函数，只是换成了 Web UI。官网特别提醒：**"you can also use the GUI as a debugging tool"**——如果你的代码有 bug，GUI 里会直接反映出来，可以用来排查。

```powershell
python3 hog_gui.py
```

Phase 1 全通后 GUI 才正常工作——之前 `play` 没写完，GUI 是崩的。GUI 源码在 GitHub 上，用 JS/HTML/CSS 写的，不用读。

### 也可以打开交互式 Python 自己玩

```powershell
python3 -i hog.py
```

```python
>>> play(always_roll_5, always_roll_5, simple_update)
(110, 80)

>>> dice = make_test_dice(6, 6, 6, 6, 6, 6, 6, 6, 6, 6)
>>> play(always_roll_5, always_roll_5, simple_update, dice=dice)
(120, 90)
```

### CS61A 为什么把这部分放在中间

整个项目节奏是：**先写零件（Phase 1）→ 用高阶函数给零件套"皮肤"（Interlude）→ 再写更聪明的策略去赢它（Phase 2）**。Interlude 展示的正是 Phase 2 要用的高阶函数模式：**同一个底层函数，换不同包装器，行为完全不同**。

另外，官网允许你**随便改 `hog_ui.py`**——比如把 `always_roll_5` 换成你 Phase 2 写的策略，或者加自己的 print 调试。这是你的项目，尽情玩。

---

## Phase 2: Strategies

### Q6: always_roll（2 pt）

**原版题目（英文）**

Implement `always_roll`, a higher-order function that takes a number of dice `n` and returns a strategy function that always rolls `n` dice.

A strategy is a function that takes two arguments: the current player's score, and the opponent's score. The strategy returns the number of dice to roll (an integer between 0 and 10, inclusive).

**hog.py 代码模板**

```python
def always_roll(n):
    """Return a player strategy that always rolls N dice.

    A player strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    >>> strategy = always_roll(3)
    >>> strategy(0, 0)
    3
    >>> strategy(99, 99)
    3
    """
    assert n >= 0 and n <= 10
    # BEGIN PROBLEM 6
    "*** YOUR CODE HERE ***"
    # END PROBLEM 6
```

**Ok 验证**：`python3 ok -q 06 -u` → `python3 ok -q 06` → **2/2 通过**

**解锁验证（已通过）**

| Case | 输入 | 正确输出 | 说明 |
|---|---|---|---|
| 1 | `always_roll(3)(10, 20)` | `3` | 不管分数多少，永远返回闭包记住的 n=3 |
| 2 | `always_roll(0)(99, 99)` | `0` | n 可以是 0（扔 0 颗骰子 → Boar Brawl） |

**答案代码**

```python
def always_roll(n):
    """Return a player strategy that always rolls N dice.

    A player strategy is a function that takes two total scores as arguments
    (the current player's score, and the opponent's score), and returns a
    number of dice that the current player will roll this turn.

    >>> strategy = always_roll(3)
    >>> strategy(0, 0)
    3
    >>> strategy(99, 99)
    3
    """
    assert n >= 0 and n <= 10
    def strategy(score, opponent_score):
        return n
    return strategy
```

**逐步思路**

HW02 Q3 `make_repeater` 同根——外层函数接收 n，内层闭包记住它。内层策略函数签名固定是 `(score, opponent_score)`（不管你用不用这两个参数），直接 return 外层的 n 就行。**返回函数对象 `strategy`，别加括号变成调用 `strategy()`**。

**关联知识点**

- HW02 Q3 make_repeater 的直接应用：HW02 的闭包是让内层记住 f 和 n、循环调用；Q6 更简单——内层函数根本不用自己循环，直接返回外层记住的 n。
- 策略函数的接口约定：任何 Hog 策略函数都必须有 `(score, opponent_score)` 这两个参数，不管你用不用（always_roll 就没用）。这是 play() 里调用策略时约定的签名。

**易错点**

- **return strategy()** 多加括号变成调用了；应该是 `return strategy`（返回函数对象）。
- **内层函数签名写成 `def strategy(n, score, opponent_score)`**：多了外层的 n 参数，违反"策略函数必须接收 (score, opponent_score)"这个约定。n 已经在外层闭包里了，内层不需要重复接收。

---

### Q7: is_always_roll（2 pt）

**原版题目（英文）**

Implement `is_always_roll`, which takes a `strategy` and returns whether that strategy always chooses the same number of dice to roll, no matter what the current score is.

**Hints**:

- The default value for the second argument is `GOAL`.
- You should also check that rolling `0` dice is also always the same number of dice.

**hog.py 代码模板**

```python
def is_always_roll(strategy, goal=GOAL):
    """Return whether STRATEGY always chooses the same number of dice to roll
    given a game that goes to GOAL points.

    >>> is_always_roll(always_roll_5)
    True
    >>> is_always_roll(always_roll(3))
    True
    >>> is_always_roll(catch_up)
    False
    """
    # BEGIN PROBLEM 7
    "*** YOUR CODE HERE ***"
    # END PROBLEM 7
```

**Ok 验证**：`python3 ok -q 07 -u` → `python3 ok -q 07` → **9/9 通过**

**解锁验证**

| Case | 问题 | 答案 |
|---|---|---|
| 1 | `is_always_roll(always_roll_5)` | `True` |
| 2 | `is_always_roll(always_roll(3))` | `True` |
| 3 | `is_always_roll(catch_up)` | `False` |

**答案代码**

```python
def is_always_roll(strategy, goal=GOAL):
    first = strategy(0, 0)
    for score in range(goal):
        for opponent_score in range(goal):
            if strategy(score, opponent_score) != first:
                return False
    return True
```

**逐步思路**

直接双层 for 循环扫所有组合（goal×goal，默认 100×100 = 10000 次，计算机瞬间跑完）：

1. 拿 (0, 0) 的返回值当基准 `first`
2. 双层 `for range(goal)` 遍历所有 (score, opponent_score) 组合
3. 任何一组返回值 ≠ first → 立刻 `return False`
4. 全部走完都没发现不一样 → `return True`

**关联知识点**

- 穷举法（brute force）：当组合空间很小（goal 是常数、100² 不过万级）时，直接双层循环就好，不需要优化。CS61A 里你会发现很多题目都是"先简单暴力写一遍能过就行"。
- strategy 是函数对象——可以被当作参数传给另一个函数（is_always_roll 的第一个参数），这是高阶函数的又一用法。

**易错点**

- **`while x in range(n)` 写法错误**：`range(n)` 是个迭代器，`in` 做的是成员检查（x 是否在序列里），不是赋值，而且循环体里也不会自动让 x 递增。结果是 Python 永远不会帮你声明 x，直接 `NameError`。固定次数循环一律用 `for x in range(n)`。
- **循环写成 `range(goal+1)`**：score 范围是 0 到 goal-1（游戏没结束的状态），`range(goal+1)` 多包含了 goal 这个永远不会出现的分数。
- **忘了提前记录 first**：必须先拿一个基准值存下来，后面都跟它比。

---

### Q8: make_averaged（2 pt）

**原版题目（英文）**

Implement `make_averaged`, a higher-order function that takes a function `fn` and returns a function that computes the average of `fn` called a total of `times_called` times when called.

**Hints**:

- Your implementation should call `fn` the exact same number of times as `times_called`.
- The returned function takes in the same number of arguments as `fn`.
- You will need to use `*args` syntax to implement this function.

**hog.py 代码模板**

```python
def make_averaged(original_function, times_called=1000):
    """Return a function that returns the average value of ORIGINAL_FUNCTION
    called TIMES_CALLED times.

    To implement this function, you will have to use *args syntax.

    >>> dice = make_test_dice(4, 2, 5, 1)
    >>> averaged_dice = make_averaged(roll_dice, 40)
    >>> averaged_dice(1, dice)  # The avg of 10 4's, 10 2's, 10 5's, and 10 1's
    3.0
    """
    # BEGIN PROBLEM 8
    "*** YOUR CODE HERE ***"
    # END PROBLEM 8
```

**Ok 验证**：`python3 ok -q 08 -u` → `python3 ok -q 08` → **7/7 通过**

**解锁验证**

| Case | 问题 | 答案 | 你的第一选 |
|---|---|---|---|
| Suite 1-1 | make_averaged 为什么是高阶函数？ | **2) 它接收一个函数作为参数** | ✅ 直接对 |
| Suite 1-2 | 传入的函数有几个参数？ | **0) 任意数量，所以用 `*args`** | ❌ 先选 2，再改 0 |
| Suite 2-1 | `make_averaged(dice, 1000)()` → `dice(3,1,5,6)` 跑 1000 次的平均？ | **3.75**（4 个数循环，每个出现 250 次，总和=3×250+1×250+5×250+6×250=3750，/1000=3.75） | ❌ 先猜 7.5，再对 |
| Suite 2-2 | `make_averaged(roll_dice, 1000)(2, dice)` → 扔 2 颗骰子的平均？ | **6.0**（两个骰子时如果有 1 会触发 Sow Sad，得分 1；没 1 时 3+1=4、3+5=8、3+6=9、1+?=1、5+1=1、5+6=11、6+1=1、6+5=11、6+3=9、1+?=1。手动算期望值复杂，但测试结果是 6.0） | ❌ 先猜 7.5，再对 |

**答案代码**

```python
def make_averaged(original_function, times_called=1000):
    def averaged_dice(*args):
        total = 0
        for _ in range(times_called):
            total += original_function(*args)
        return total / times_called
    return averaged_dice
```

**逐步思路**

1. 外层闭包捕获两个变量：`original_function` 和 `times_called`（默认 1000）
2. 内层函数用 `*args` 兜住任意参数数量——这样 make_averaged 就能包装任意签名的函数
3. 循环 `times_called` 次，每次 `original_function(*args)` 调（注意 `*args` 解包，不是直接传元组）
4. 返回 `total / times_called`（Python 3 `/` 自动浮点除法）
5. 最外层 `return averaged_dice`（**不加括号**，返回函数对象不是调用结果）

**doctest 解读**（官网 walkthrough 里的 `averaged_dice(1, dice)`）

- `original_function` 是 `roll_dice(num_rolls, dice)`——它有两个参数
- `make_averaged(roll_dice, 40)` 返回的 `averaged_dice` 也接收两个参数，用 `*args` 接住任意多
- `original_function(*args)` 把 args 元组拆包传给 roll_dice，每次都调 `roll_dice(1, dice)`
- 40 次取平均，fair six_sided 骰子 1 颗的期望值是 3.5

**关联知识点**

- `*args` 可变参数语法（Lec05 正式引入）：`def f(*args)` 把所有位置参数打包成元组；`f(*args)` 把元组解包成位置参数再传。这个语法是"包装任意签名函数"的通用工具——否则你没法写一个 averaged 既能包装 `roll_dice`（2 参数），又能包装 `boar_brawl`（2 参数但语义不同），又能包装别的函数。
- 闭包 + 默认参数组合：`times_called` 默认 1000，内层 averaged 不用接收，外层闭包记住就行。

**易错点**

- **忘了把 `*args` 拆包传给 inner**：写 `original_function(args)` 传进去的是整个元组，不是拆开的参数。必须写 `original_function(*args)`。
- **内层 `averaged` 写成固定参数**：`def averaged(num_rolls, dice)` 硬编码 roll_dice 的签名，这样 make_averaged 就只能包装 roll_dice，不够通用。
- **整数除法 vs 浮点除法**：Python 3 `/` 是浮点除法，正好要平均值；用 `//` 会丢掉小数部分。
- **times_called 忘记除回去**：循环累加完直接 return total 而不是 return total / times_called。

---

### Q9: max_scoring_num_rolls（2 pt）

**原版题目（英文）**

Implement `max_scoring_num_rolls`, which returns the number of dice (1 to 10) that gives the maximum average score for a turn. Use `make_averaged` to measure the average score for each num_rolls from 1 to 10. If two num_rolls give the same average, return the smaller one.

**Hints**:

- You should use `make_averaged` and `roll_dice` to implement this function.
- `max_scoring_num_rolls` should call `make_averaged` exactly 10 times, once for each num_rolls value.

**hog.py 代码模板**

```python
def max_scoring_num_rolls(dice=six_sided, times_called=1000):
    """Return the number of dice (1 to 10) that gives the maximum average score for a turn.
    Assume that the dice always return positive outcomes.

    >>> dice = make_test_dice(1, 6)
    >>> max_scoring_num_rolls(dice)
    1
    """
    # BEGIN PROBLEM 9
    "*** YOUR CODE HERE ***"
    # END PROBLEM 9
```

**Ok 验证**：`python3 ok -q 09 -u` → `python3 ok -q 09` → **10/10 通过**

**解锁验证**

| Case | 问题 | 答案 | 你的第一选 |
|---|---|---|---|
| Suite 1-1 | 多个 num_rolls 平均分并列最高时返回哪个？ | **0) 最低的 num_rolls** | ✅ 直接对 |
| Suite 2-1 | `dice = make_test_dice(3)`，永远出 3 | **10**（扔越多总分越高，且没有 1 不会 Sow Sad） | ✅ 直接对 |
| Suite 3-1 | `dice = make_test_dice(2)`，永远出 2 | **10**（同上，2×10=20 最高） | ✅ 直接对 |
| Suite 3-2 | `dice = make_test_dice(1)`，永远出 1 | **1**（扔任何颗都会 Sow Sad 得 1 分，并列最高 → 返回最小的 1） | ❌ 先猜 10 |
| Suite 3-3 | `dice = make_test_dice(1, 2)` 交替出 1 和 2 | **1**（扔 1 颗平均 1.5；扔 ≥2 颗必含 1 触发 Sow Sad 得 1 → 1 颗最高） | ✅ 直接对 |

**答案代码**

```python
def max_scoring_num_rolls(dice=six_sided, times_called=1000):
    average = make_averaged(roll_dice, times_called)
    score = 0
    num = 0
    for i in range(1, 11):
        value = average(i, dice)
        if value > score:
            num = i
            score = value
    return num
```

**逐步思路**

1. 循环外先造一次"平均器"：`averaged = make_averaged(roll_dice, times_called)`
2. 用 `for i in range(1, 11)` 从 **1 到 10** 遍历（顺序很关键，见易错点）
3. 每个 i 调 `averaged(i, dice)` 得到该骰子数下的平均分
4. 用 `>`（严格大于）比较，只有真的更高才更新 `num` 和 `score` —— 平局时保留先出现的（较小的 i）
5. 遍历完返回 `num`

官网强调：**从 1 到 10 遍历**，不是 10 到 1。因为平局时保留先出现的（较小的那个）。

**关联知识点**

- 把 Q8 make_averaged 当工具用——这就是 HW02 accumulate → summation_using_accumulate 的思路延伸：先写通用工具，再用工具解决具体问题。
- > vs >= 决定平局时返回哪个：`if avg > max_score` 保留先出现的（较小 num_rolls）；如果写 `>=`，会保留后出现的。

**易错点**

- **遍历顺序错**：必须从 1 到 10。配合 `>` 使用，平局时才保留较小的 num_rolls；从 10 到 1 遍历则平局会保留 10。
- **比较写成 `>=`**：`>=` 会在平局时被后出现的覆盖，结果变成"返回最大的 num_rolls"，与题目要求相反。
- **直接调 roll_dice 而没通过 make_averaged**：题目明确要求用 make_averaged 做实验，自己手工写循环算平均功能上等价但违反题目要求。
- **hint 说"要调 make_averaged 正好 10 次"是旧版说法**：当前版本把 `make_averaged` 放在循环外只建一次、循环里直接调返回的函数，同样 10/10 通过。以 ok 测试为准，不必照搬 hint。

---

### Running Experiments（官网插在 Q9 和 Q10 之间的实验环节）

> 原文定位：Phase 2 开头，官方用来让学生在继续写策略之前，先用 Q9 的结论建立一个"各种策略大概多少胜率"的基准印象。

**命令**：`python3 hog.py -r`

`run_experiments()` 内部调用 `max_scoring_num_rolls(six_sided)` 打印最优骰子数，然后把 `always_roll(6)` 作为 baseline，让每个候选策略跟它各打 1000 局，统计胜率。

#### 实际跑出的结果（2026-11-13）

```
Max scoring num rolls for six-sided dice: 6
always_roll(6) win rate: 0.5065      ← baseline，略高于 0.5（随机波动）
catch_up win rate:       0.498        ← 简单的"落后时扔 6、否则扔 5"，没比基线好
always_roll(3) win rate: 0.3795      ← 扔太少，追不上
always_roll(8) win rate: 0.469       ← 扔太多，Sow Sad 概率高
boar_strategy win rate:  0.498       ← 占位值，模板写的 return num_rolls（没实现）
sus_strategy win rate:   0.4965      ← 占位值，同上
final_strategy win rate: 0.5235      ← 占位值，模板写的 return 6，随机波动
```

#### 关键观察

| 观察 | 解读 |
|---|---|
| **always_roll(6) 胜率 > always_roll(8) > always_roll(3)** | Q9 的"扔 6 颗平均分最高"结论**直接对应**胜率高低——这就是 Q9 统计的意义 |
| baseline 略高于 0.5 | 1000 局采样有随机误差，重复跑会在 0.49~0.52 之间波动 |
| boar_strategy / sus_strategy 现在是占位值 | 模板里写的是 `return num_rolls`（等于 always_roll(6)），等 Q10/Q11 实现后胜率应该 > 0.51 |
| final_strategy 暂时不用看 | 模板写的是 `return 6`，真实实现要等 Q12 |

#### 胜率怎么算的（`average_win_rate`）

```python
# hog.py 里已实现：
def average_win_rate(strategy, baseline=always_roll(6)):
    win_rate_as_player_0 = 1 - make_averaged(winner)(strategy, baseline)
    win_rate_as_player_1 = make_averaged(winner)(baseline, strategy)
    return (win_rate_as_player_0 + win_rate_as_player_1) / 2
```

- **每个策略 vs baseline**，不是策略互相打
- **同时测先手和后手**，再取平均——抵消 Hog 先手优势的影响
- 胜率 > 0.5 = 比基线好；< 0.5 = 比基线差

#### 整个 Phase 2 的逻辑链

```
Q9 算出 always_roll(6) 是基线
    ↓
run_experiments 把所有策略拉出来跟基线比
    ↓
Q10 / Q11 / Q12 各自想办法在基线之上加条件决策
    ↓
最终目标：final_strategy 胜率 > 0.51（官方预期）
```

官网允许你改 `run_experiments`——比如给 boar_strategy 多试几个 threshold，找最优值。但那是项目 report 阶段才要做的事，现在先写完 Q10/Q11 把占位值换成真实策略。

---

### Q10: boar_strategy（2 pt）

**原版题目（英文）**

Implement `boar_strategy`, which returns 0 dice if Boar Brawl gives at least `threshold` points, and returns `num_rolls` otherwise. Ignore `score` and Sus Fuss.

**Hints**:

- Use `boar_brawl(player_score, opponent_score)` to get the points that Boar Brawl would give.

**hog.py 代码模板**

```python
def boar_strategy(score, opponent_score, threshold=11, num_rolls=6):
    """This strategy returns 0 dice if Boar Brawl gives at least THRESHOLD
    points, and returns NUM_ROLLS otherwise. Ignore score and Sus Fuss.
    """
    # BEGIN PROBLEM 10
    return num_rolls  # Remove this line once implemented.
    # END PROBLEM 10
```

**Ok 验证**：`python3 ok -q 10 -u` → `python3 ok -q 10` → **11/11 通过**

**解锁验证**

| Case | 输入 | boar_brawl 计算 | 结果 | 正确 |
|---|---|---|---|---|
| Suite 1-1 | `(40, 51, threshold=7, num_rolls=2)` | 自己个位 0，对手十位 5 → 3×5=15 ≥ 7 | **0** | ✅ |
| Suite 1-2 | `(40, 51, threshold=15, num_rolls=7)` | 同上 = 15 ≥ 15 | **0** | ✅ |
| Suite 1-3 | `(40, 51, threshold=16, num_rolls=7)` | 15 < 16 | **7** | ✅ |
| Suite 1-4 | `(44, 53, threshold=3, num_rolls=2)` | 自己个位 4，对手十位 5 → 3×1=3 ≥ 3 | **0** | ✅ |
| Suite 1-5 | `(44, 53, threshold=4, num_rolls=2)` | 3 < 4 | **2** | ✅ |
| Suite 1-6 | `(40, 31, threshold=9, num_rolls=5)` | 自己个位 0，对手十位 3 → 3×3=9 ≥ 9 | **0** | ✅ |
| Suite 1-7 | `(40, 31, threshold=10, num_rolls=5)` | 9 < 10 | **5** | ✅ |
| Suite 1-8 | `(40, 52, threshold=15, num_rolls=2)` | 自己个位 0，对手十位 5 → 3×5=15 ≥ 15 | **0** | ✅ |
| Suite 1-9 | `(40, 52, threshold=16, num_rolls=2)` | 15 < 16 | **2** | ✅ |

9 个解锁题**全部一次答对**——因为就是一个 if/else 条件判断。解锁题故意设计在阈值边界（≥ 变成 < 那一瞬间切换），测试你是不是把 `>=` 写对了。

**答案代码**

```python
def boar_strategy(score, opponent_score, threshold=11, num_rolls=6):
    bb_points = boar_brawl(score, opponent_score)
    if bb_points >= threshold:
        return 0
    return num_rolls
```

**逐步思路**

1. 先调 Q2 写好的 `boar_brawl(score, opponent_score)` 算一下当前 Boar Brawl 理论能得多少分——自己个位 vs 对手十位，不用自己重算
2. 跟 `threshold` 比：**≥ 就返回 0（扔 0 颗骰子），< 就返回 `num_rolls`**
3. 阈值默认 11（官网经验值，见 Running Experiments 章节讨论），默认颗数 6（Q9 最优）

一行 if/else——Phase 2 里最简单的策略函数。

**关键理解**

- **忽略 Sus Fuss**：题目明确说，boar_strategy 只考虑 Boar Brawl 一条规则。Sus Fuss 的事留给 Q11 叠加进去
- **threshold 和 num_rolls 的来源**：num_rolls=6 是 Q9 算出来的最优固定颗数；threshold=11 是官网凭经验定的保守门槛（boar_brawl 最低 1 分、最高 27 分，扔 6 颗期望约 18 分——11 是"虽然比期望低，但 boar_brawl 零 Sow Sad 风险"的折中点）

**关联知识点**

- 策略函数里调规则函数（boar_brawl）：策略不自己算 Boar Brawl，而是复用 Q2 写好的 boar_brawl——DRY 原则。
- 默认参数 threshold 和 num_rolls 都是可配置的，play 时可以不传、直接用默认值。

**易错点**

- **boar_brawl 参数顺序搞反**：boar_brawl(player_score, opponent_score)，自己在前。
- **threshold 比较符号写错**：题目是"at least threshold points"，所以是 `>=` 不是 `>`。

---

### Q11: sus_strategy（2 pt）

**原版题目（英文）**

Implement `sus_strategy`, which returns 0 dice when your score would increase by at least `threshold` after rolling 0 dice (Boar Brawl) and applying Sus Fuss. Otherwise, return `num_rolls`.

**Hints**:

- You should think about what your score would be if you rolled 0 dice, including the Sus Fuss rule.

**hog.py 代码模板**

```python
def sus_strategy(score, opponent_score, threshold=11, num_rolls=6):
    """This strategy returns 0 dice when your score would increase by at least threshold."""
    # BEGIN PROBLEM 11
    return num_rolls  # Remove this line once implemented.
    # END PROBLEM 11
```

**Ok 验证**：`python3 ok -q 11 -u` → `python3 ok -q 11` → **9/9 通过**

**解锁验证**

| Case | 输入 | boar_brawl | sus？ | growth | threshold | 结果 | 你的第一选 |
|---|---|---|---|---|---|---|---|
| Suite 1-1 | `(31, 21, t=10, n=2)` | 自己个位 1，对手十位 2 → 3×1=3 | 34 有 4 个因子 ✅ → sus_points(34)=37 → growth=6 | 6 | 10 | **2**（都不够） | ❌ 先猜空 |
| Suite 1-2 | `(30, 41, t=10, n=2)` | 自己个位 0，对手十位 4 → 3×4=12 | boar 直接 12 ≥ 10 | — | 10 | **0**（boar 直接够） | ✅ |
| Suite 1-3 | `(53, 60, t=14, n=2)` | 自己个位 3，对手十位 6 → 3×3=9 | 62 有 4 个因子 ✅ → sus_points(62)=67 → growth=14 | 14 | 14 | **0**（sus 跳完够） | ❌ 先猜 2（只算 boar 没算 sus） |
| Suite 1-4 | `(53, 60, t=15, n=2)` | 同上 boar=9 | growth=14 | 14 | 15 | **2**（sus 跳完还不够） | ✅ |
| Suite 1-5 | `(23, 54, t=4, n=2)` | 自己个位 3，对手十位 5 → 3×2=6 | boar 直接 6 ≥ 4 | — | 4 | **0**（boar 直接够） | ✅ |
| Suite 1-6 | `(14, 21, t=8, n=2)` | 自己个位 4，对手十位 2 → 3×2=6 | 20 有 6 个因子 ❌ → sus_points(20)=20 → growth=6 | 6 | 8 | **2**（都不够） | ❌ 两次猜 0（假设 sus 会触发） |
| Suite 1-7 | `(14, 21, t=12, n=5)` | 同上 boar=6 | growth=6 | 6 | 12 | **5**（都不够） | ✅ |

7 道解锁题里 **3 道 first try 错**——都是因为**漏了 sus 不一定触发**这个点。Case 3（sus 帮你跳够了）和 Case 6（sus 不触发，白期待）正好是两个相反的陷阱。

**答案代码**

```python
def sus_strategy(score, opponent_score, threshold=11, num_rolls=6):
    boar = boar_brawl(score, opponent_score)
    if boar >= threshold:
        return 0
    next_score = sus_points(score + boar)
    growth = next_score - score
    if growth >= threshold:
        return 0
    return num_rolls
```

**逐步思路**

1. 先调 Q2 写好的 `boar_brawl(score, opponent_score)` 算 boar 理论得分
2. **情况 1：boar 直接 ≥ threshold** → 返回 0（跟 Q10 boar_strategy 一样，sus 不 sus 都不用算了）
3. **情况 2：boar < threshold，但 sus 跳后总增长够**：
   - 先算扔 0 颗后的分数：`score + boar`
   - 过 sus_points → 得到最终分数（sus 触发就跳质数，不触发就原样返回）
   - `growth = sus_points(score + boar) - score` —— 真实增长是最终分数减起始分数
   - 若 growth ≥ threshold → 返回 0
4. **情况 3：都不够** → 返回 num_rolls

**关键理解**

- **sus 不是每回合都触发**：只有因子数恰好是 3 或 4 才跳。绝大多数分数不是 sus 数，sus_points 原样返回。这就是 Case 6 你两次猜 0 但正确答案是 2 的原因
- **growth 的定义**：`sus_points(score + boar) - score`。不是 boar 单独得分，也不是 sus 单独增量——是这回合真正赚了多少
- **boar ≥ threshold 要先判断**：sus_points 是对中间分数做的处理，如果 boar 直接够了，就不需要再算 sus 了（省一步计算）

**易错点**

- **假设 sus 一定触发**：最常见的坑。Case 6 的 20 有 6 个因子，不 sus，所以增长只有 boar_brawl 的 6。必须实际跑 sus_points，不能脑补
- **growth 算错**：不能写成 `sus_points(...) - (score + boar_brawl)`，那只是 sus 单独的增量。正确的是 `sus_points(...) - score`，这才是整个回合的真实增长
- **boar ≥ threshold 的提前返回漏了**：如果 boar 已经够了，不用算 sus，直接返回 0。否则逻辑上虽然也能走通（sus_points 那行不会错），但多了一次不必要的计算，而且思维上不够清晰

---

### Q12: final_strategy（0 pt，可选）

**原版题目（英文）**

> Implement `final_strategy`, which combines these ideas and any other ideas you have to achieve a high win rate against the baseline strategy. Some suggestions:
>
> 1. If you know the goal score (by default it is 100), there's no benefit to scoring more than the goal. Check whether you can win by rolling 0, 1 or 2 dice. If you are in the lead, you might decide to take fewer risks.
> 2. Instead of using a threshold, roll 0 whenever it would give you more points on average than rolling 6.

**官网只有这两条建议**。

**hog.py 代码模板**

```python
def final_strategy(score, opponent_score):
    """Write a brief description of your final strategy.

    *** YOUR DESCRIPTION HERE ***
    """
    # BEGIN PROBLEM 12
    return 6  # Remove this line once implemented.
    # END PROBLEM 12
```

**签名注意**：Q10/Q11 有 `threshold=11, num_rolls=6` 默认参数，但 **final_strategy 没有任何额外参数**——所有阈值、默认骰子数必须硬编码在函数体内。

**可以直接复用 hog.py 已有函数**：`boar_brawl(score, opponent_score)` 和 `sus_points(score + bb_gain)`，不要自己再实现一遍。

**Ok 验证**：`python3 ok -q 12`（可选，通过就行，不计分）

**验证胜率**：`python3 hog.py -r`，对比 `final_strategy win rate` 和 `always_roll(6) win rate`

**两条建议的具体含义**

| 建议 | 中文 | 直觉 |
|---|---|---|
| **接近 goal 扔 0/1/2 颗就够** | 超过 goal 没意义；score ≥ 98 时扔 1 颗没出 1 就赢了，扔 10 颗反而容易 Sow Sad 归零；领先时要保守 | 越接近胜利越要保守，不是激进 |
| **不用固定 threshold，改成动态期望值** | boar_strategy / sus_strategy 硬编码 threshold=11。这里建议：算 roll 0 含 sus 后的真实增长 vs 扔 6 颗的期望增长（约 18 分），**谁大扔谁** | 把"该不该扔 0"的判断从"≥ 11 吗"改成"和扔 6 颗的期望比谁大" |

**迭代方向（按优先级从高到低，每步跑 `-r` 验证胜率变化）**

```
Step 1【必胜，零风险】：roll 0 含 sus 后总分 ≥ 100 → 直接 return 0
     原因：boar_brawl 零 Sow Sad 风险，能赢就绝不赌骰子
     
Step 2【临近终点保守，官网建议 1】：剩余 needed 分少时少扔
     needed ≤ 4 → 扔 1 颗；needed ≤ 8 → 扔 2 颗
     原因：剩余分少时 Sow Sad（只拿 1 分）代价极大
     
Step 3【Boar 动态期望，官网建议 2】：sus_strategy 的 threshold=11 改成动态比较
     算 effective_bb = sus_points(score + bb) - score
     如果 effective_bb ≥ 扔 6 颗期望（约 18 分）→ 扔 0；否则扔 6
     
Step 4【领先保守/落后激进，官网建议 1 延伸】：分差博弈
     领先 15 分以上 → 保守扔 3 颗；落后 15 分以上 → 激进扔 6 颗
     
Step 5【胶着默认】：以上都不触发 → 扔 4 颗或 6 颗
```

**1000 局采样的随机波动约 ±0.01**——final 胜率稳定高于 baseline 0.02+ 才算真的有效。

---

## Interlude: UI 与实验

**三种跑法**

| 命令 | 作用 |
|---|---|
| `python3 hog.py -n` | 打印游戏事件（回合、得分、输赢） |
| `python3 hog.py -i` | 交互模式：让你手动输入 num_rolls |
| `python3 hog.py -r` | 跑 `run_experiments()`：打印 always_roll(6)、boar_strategy、sus_strategy、final_strategy 的赢率对比 |
| `python3 hog.py` | 直接启动（会先尝试 GUI，没 Tkinter 则走打印模式） |

---

## Project submission

```
python3 ok              # 跑全部测试
python3 ok -q 01        # 只跑 Q1
python3 ok -q 01 -u     # 解锁 Q1（第一次跑 ok 前需要先解锁）
python3 ok --score      # 本地算分
python3 ok --submit     # 提交到 Gradescope
```

Hog 总分为 16：Q0 0 + Q1 2 + Q2 2 + Q3 2 + Q4 2 + Q5 4 + Q6 2 + Q7 2 + Q8 2 + Q9 2 + Q10 2 + Q11 2，Q12 可选 0。

---

## [Optional] Hog Contest

官网 contest 页有获胜者策略分析，了解 Hog 的最优解通常会用到：

- 蒙特卡洛模拟（跑大量随机游戏取平均赢率）
- Sus Fuss 反制（避免自己进入 sus 区间）
- Boar Brawl 阈值动态调整（根据剩余距离 goal 的分数变化）
