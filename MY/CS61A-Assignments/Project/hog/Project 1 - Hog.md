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

**Ok 验证**：`python3 ok -q 05 -u` → `python3 ok -q 05`

**逐步思路**

1. 用 `while True` 无限循环（因为不知道要几回合），每轮先判断 `score0 >= goal or score1 >= goal`，是的话 return 赢方。
2. **策略接收的两个参数是「自己的分数」和「对手的分数」**，不是 `(score0, score1)`。所以调用时：
   - player0 的回合：`num_rolls = strategy0(score0, score1)`
   - player1 的回合：`num_rolls = strategy1(score1, score0)`
   - 注意第二个参数是**对手**的分数，顺序反了就把策略搞反！
3. 调 update 得分：`score0 = update(num_rolls, score0, score1, dice)`
4. 交换回合（用一个 player 变量或交替调 strategy）。

**关联知识点**

- 双策略主循环是**函数式游戏引擎**的雏形——strategy 函数决定动作、update 函数决定得分规则、play 函数负责循环调度。这是把"规则"和"策略"分离的基本架构，比写死在 play 里强得多。
- 可变默认参数陷阱：`score0=0, score1=0` 是整数、不可变，所以不会有经典 Python mutable default argument bug（如果是 `[]` 就会出问题），这里安全。

**易错点**

- **strategy 参数顺序反了**：player1 调用 `strategy1(score0, score1)` 传反了变成自己分数作第二个参数、对手分数作第一个——这样策略就会把自己分数当成对手分数来判断。
- **忘记先判断输赢**：`while True` 里要先检查 ≥ goal 再进入策略/更新步骤，否则会多走一个回合。
- **player 切换混乱**：推荐写法是用 `current_player = 0`，`(score0, score1) = (score1, score0)` 交换元组的写法，然后 `current_player = 1 - current_player`；或者把 score0/score1 放进列表里，用 index 切换。

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

**Ok 验证**：`python3 ok -q 06 -u` → `python3 ok -q 06`

**逐步思路**

HW02 Q3 `make_repeater` 同根——返回闭包记住外层的 `n`，内层函数忽略传入的 score 和 opponent_score：

```python
def always_roll(n):
    def strategy(score, opponent_score):
        return n
    return strategy
```

`always_roll(6)` 等价于 `always_roll_5`（但传的是 6）。**return strategy 别加括号**。

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

**Ok 验证**：`python3 ok -q 07 -u` → `python3 ok -q 07`

**逐步思路**

直接双层 for 循环扫所有组合（100×100 = 10000 次，计算机瞬间跑完）：

```python
def is_always_roll(strategy, goal=GOAL):
    first = strategy(0, 0)
    for score in range(goal):
        for opponent_score in range(goal):
            if strategy(score, opponent_score) != first:
                return False
    return True
```

**关联知识点**

- 穷举法（brute force）：当组合空间很小（goal 是常数、100² 不过万级）时，直接双层循环就好，不需要优化。CS61A 里你会发现很多题目都是"先简单暴力写一遍能过就行"。
- strategy 是函数对象——可以被当作参数传给另一个函数（is_always_roll 的第一个参数），这是高阶函数的又一用法。

**易错点**

- **循环写成 `range(goal+1)`**：题目说 score 是 0 到 goal-1（游戏没结束的状态），googol 数应该是 goal² 不是 (goal+1)²。
- **忘了提前记录 first**：如果每次都单独跟第一个比要先存下来；或者看当前是否等于前一个（更脆弱，因为有"第一次"的问题）。

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

**Ok 验证**：`python3 ok -q 08 -u` → `python3 ok -q 08`

**逐步思路**

```python
def make_averaged(original_function, times_called=1000):
    def averaged(*args):
        total = 0
        for _ in range(times_called):
            total += original_function(*args)
        return total / times_called
    return averaged
```

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

**Ok 验证**：`python3 ok -q 09 -u` → `python3 ok -q 09`

**逐步思路**

```python
def max_scoring_num_rolls(dice=six_sided, times_called=1000):
    averaged = make_averaged(roll_dice, times_called)
    max_score = 0
    best_num = 1
    for num_rolls in range(1, 11):
        avg = averaged(num_rolls, dice)
        if avg > max_score:       # > 而非 >=，平局时保留旧的（小的那个）
            max_score = avg
            best_num = num_rolls
    return best_num
```

官网强调：**从 1 到 10 遍历**，不是 10 到 1。因为平局时保留先出现的（较小的那个）。

**关联知识点**

- 把 Q8 make_averaged 当工具用——这就是 HW02 accumulate → summation_using_accumulate 的思路延伸：先写通用工具，再用工具解决具体问题。
- > vs >= 决定平局时返回哪个：`if avg > max_score` 保留先出现的（较小 num_rolls）；如果写 `>=`，会保留后出现的。

**易错点**

- **遍历顺序错**：从 10 到 1 时 `>=` 保留大的、`>` 还是保留 1（因为 10 先被 max_score 覆盖）——必须严格从 1 到 10。
- **直接调 roll_dice 而没通过 make_averaged**：题目明确要求用 make_averaged 做实验，你自己手工调 times_called 次也算平均——功能上等价但违反题目要求。

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

**Ok 验证**：`python3 ok -q 10 -u` → `python3 ok -q 10`

**逐步思路**

```python
def boar_strategy(score, opponent_score, threshold=11, num_rolls=6):
    if boar_brawl(score, opponent_score) >= threshold:
        return 0
    return num_rolls
```

一行 if/else——难度很低。

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

**Ok 验证**：`python3 ok -q 11 -u` → `python3 ok -q 11`

**逐步思路**

```python
def sus_strategy(score, opponent_score, threshold=11, num_rolls=6):
    # 先算 boar_brawl 理论能得多少分
    boar = boar_brawl(score, opponent_score)
    if boar >= threshold:
        return 0
    return num_rolls
```

等等——官网说"额外考虑 Sus Fuss"，实际 sus_strategy 的判断逻辑是**扔 0 颗之后分数会不会因为 sus 规则跳质数、跳了之后增长更大**。完整写法：

```python
def sus_strategy(score, opponent_score, threshold=11, num_rolls=6):
    boar = boar_brawl(score, opponent_score)
    # 如果 boar ≥ threshold，直接返回 0（和 boar_strategy 一样）
    # 如果 boar 不够 threshold，但加上 sus 后足够了呢？
    # 先算扔 0 颗后的分数（boar + score）
    next_score = sus_points(score + boar)
    # sus 触发的话 next_score > score + boar，增长 = next_score - score
    growth = next_score - score
    if growth >= threshold:
        return 0
    return num_rolls
```

**关联知识点**

- 策略链：sus_strategy 本质是 boar_strategy 的增强版，先算 boar_brawl、再 sus_points 包装、再看总增长。
- sus_points 是 Q4 写好的——再次复用小函数。

**易错点**

- **sus_points 里 score 本身就是质数的情况**：比如 score=100（质数？不是，100 不是 sus 数）——sus_points(100) = 100，growth = boar，符合预期。但如果 score 是某个质数且 boar + score 恰好是 sus 数，那 sus_points 会跳；如果 boar + score 不是 sus 数，sus_points 原样返回。
- **没考虑 sus 跳后增长更大的情况**：直接照搬 boar_strategy 会漏掉"分数本来不 sus，加 boar 后变 sus、跳了质数后增长够 threshold"这一种判断。

---

### Q12: final_strategy（0 pt，可选）

**原版题目（英文）**

Write a `final_strategy(score, opponent_score)` that performs as well as possible against `always_roll(6)`. This is a competition problem—you're encouraged to be creative!

**Hints**:

- You should use `boar_brawl`, `sus_points`, and any other functions you've implemented to help you write your strategy.
- Think about when rolling 0 dice would be better than rolling `num_rolls` dice.
- You might want to experiment with different values of `num_rolls` using `max_scoring_num_rolls` and `run_experiments`.

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
