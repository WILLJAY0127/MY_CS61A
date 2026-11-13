# Q12 final_strategy 设计文档

> 目标：在 Hog 项目 Q12 实现一个尽可能强的 `final_strategy(score, opp_score)`，对抗基线 `always_roll(6)` 胜率越高越好。

---

## 一、规则整理（FA24 原版）

GOAL = 100；两名玩家轮流行动，先在回合结束总分 ≥100 获胜。

| 规则 | 触条件 | 效果 |
|---|---|---|
| **Sow Sad** | 本次掷的任意一颗骰子出现 1 | 本轮得分直接 = 1，忽略其他骰子点数 |
| **Boar Brawl** | 本轮掷 0 颗骰子 | gain = max(3 × \|opponent 十位 - 我方 个位\|, 1)，分数 < 10 时十位视为 0 |
| **Sus Fuss** | 回合结算后新总分恰好有 3 或 4 个正因数 | 总分自动提升到下一个质数 |

Sus Fuss 例子：4 (1,2,4) 3 因子 → sus → 跳到 5；6 (1,2,3,6) 4 因子 → sus → 跳到 7；5 只有 2 因子，不是 sus。

`final_strategy(score, opp_score)` → 返回 0~10 的整数（本轮掷骰数量）。

---

## 二、优化思考框架（5 个维度，按优先级排序）

思考顺序 = 代码里判断分支的排列顺序。

### 1. 必胜优先（最高优先级）

**先评估 roll 0 的最终分数（必须算 Sus Fuss），如果拿 Boar Brawl 之后已经 ≥ 100，直接 return 0。**

原因：0 骰子无 Sow Sad 风险，稳定拿到收益，能赢就绝不赌骰子。

### 2. 临近终点风险控制

**剩余分数少 → 用极少骰子（1/2 颗），压低掷出 1 的概率，保守求稳。**

离目标分数很近时，Sow Sad 的代价极大：比如还差 3 分，扔 6 颗骰子一旦出 1，本轮只拿 1 分，浪费回合。

### 3. 评估 Boar Brawl 的真实价值（Sus Fuss 是隐藏加成）

| 做法 | 说明 |
|---|---|
| ❌ 错误 | 直接拿原始 `bb_gain` 对比 |
| ✅ 正确 | 算 `有效收益 = sus 结算后的总分 - 当前 score` |

有效收益高 → 比扔骰子的数学期望高 → 选 0；
有效收益低（经常等于 1）→ 不要选 0。

### 4. 分差博弈：领先保守、落后激进

| 状态 | 策略 |
|---|---|
| 大幅领先 | 少量骰子，降低翻车概率，稳住优势 |
| 大幅落后 | 更多骰子，承担 Sow Sad 风险搏高分 |
| 胶着状态 | 均衡默认骰子数 |

### 5. Sus Fuss 的主动利用（进阶）

Sus 不是只给 roll 0 用的：

- 掷骰子拿到某分值后，如果新总分刚好是 sus，可以白嫖到下一个质数，提升有效得分
- 反面：刻意避开"变成 sus 但下一个质数只涨一点点"的鸡肋情况（收益很低不值得）

**约束**：骰子只能选 0~10，没有上限。

---

## 三、Debug 版完整代码

每一步打印：原始 bb 收益、roll 0 之后原始总分、是否 sus、sus 修正后的总分、有效收益、还差多少分、最终返回几颗骰子。

> **注意**：这是独立测试版本，自己实现了 `boar_brawl`、`is_sus`、`next_prime`——如果要放进 hog.py，请删掉这些重复函数，直接用 hog.py 里的。

```python
import math

def is_sus(n):
    """判断数字 n 是否是 sus：恰好 3 或 4 个因数"""
    if n < 2:
        return False
    factor_count = 0
    for i in range(1, int(math.isqrt(n)) + 1):
        if n % i == 0:
            other = n // i
            if i == other:
                factor_count += 1
            else:
                factor_count += 2
    return factor_count == 3 or factor_count == 4

def next_prime(n):
    """返回大于 n 的最小质数"""
    def is_prime(x):
        if x < 2:
            return False
        for i in range(2, int(math.isqrt(x)) + 1):
            if x % i == 0:
                return False
        return True
    candidate = n + 1
    while not is_prime(candidate):
        candidate += 1
    return candidate

def boar_brawl(my_score, opp_score):
    my_ones = my_score % 10
    opp_tens = opp_score // 10
    return max(3 * abs(opp_tens - my_ones), 1)

def final_strategy(score, opp_score):
    GOAL = 100
    bb_gain = boar_brawl(score, opp_score)
    new_bb_raw = score + bb_gain

    # Sus Fuss 结算
    new_bb = new_bb_raw
    if is_sus(new_bb_raw):
        new_bb = next_prime(new_bb_raw)
    effective_bb = new_bb - score

    needed = GOAL - score

    # Debug 输出
    print(f"[DEBUG] my:{score:3d}, opp:{opp_score:3d} | bb_gain={bb_gain}, raw+bb={new_bb_raw}, after_sus={new_bb}, eff_bb={effective_bb}, need={needed}")

    # 1. roll 0 直接获胜
    if new_bb >= GOAL:
        print("  -> DECISION: roll 0, win immediately")
        return 0

    # 2. 临近终点，保守
    if needed <= 4:
        print("  -> DECISION: close to goal, roll 1")
        return 1
    elif needed <= 8:
        print("  -> DECISION: close to goal, roll 2")
        return 2

    # 3. Boar Brawl 有效收益很高，直接选 0
    if effective_bb >= 6:
        print("  -> DECISION: good boar brawl, roll 0")
        return 0

    # 4. 分差博弈
    if score > opp_score + 15:
        print("  -> DECISION: big lead, conservative roll 3")
        return 3
    if opp_score > score + 15:
        print("  -> DECISION: falling behind, aggressive roll 6")
        return 6

    # 5. 胶着，默认均衡
    print("  -> DECISION: even game, default roll 4")
    return 4
```

---

## 四、自测命令

```bash
# 自动对局测试，看胜率
python3 hog.py -r

# ok 检查 Q12（可选，不计分）
python3 ok -q 12

# 交互式跑多局，看到每回合策略的实际行为
python3 hog.py -n
```

---

## 五、调优方向（逐个修改阈值观察胜率变化）

| 阈值 | 当前值 | 可调范围 | 调整思路 |
|---|---|---|---|
| `effective_bb >= 6` | 6 | 5 或 7 | 改成 5 让 roll 0 更容易触发；改成 7 更保守 |
| 分差阈值 `+15` | 15 | 12 / 18 | 小一点更容易触发领先保守；大一点等领先更多才触发 |
| 临近终点 `needed <= 4` | 4 | 3 / 5 | 决定什么时候切换到极保守的 1 颗骰子 |
| 临近终点 `needed <= 8` | 8 | 6 / 10 | 决定什么时候切换到 2 颗骰子 |
| 胶着默认值 | 4 | 3 / 5 | 胶着状态的基准骰子数 |

进阶扩展：

- 给掷 k 颗骰子做期望估算，对比 Boar Brawl 的有效期望，而不是写死判断
- 终极版本：预计算所有 (score, opp_score) 状态的最优动作（动态规划），胜率上限最高，但代码量大很多

---

## 六、Clean 提交版（去 debug、用 hog.py 已有函数）

放进 hog.py 时，**删掉自己实现的 `boar_brawl`、`is_sus`、`next_prime`**，直接用 hog.py 里已有的 `boar_brawl(score, opp_score)` 和 `sus_points(score + bb_gain)`：

```python
def final_strategy(score, opponent_score):
    GOAL = 100
    bb_gain = boar_brawl(score, opponent_score)
    new_bb = sus_points(score + bb_gain)
    effective_bb = new_bb - score
    needed = GOAL - score

    # 1. roll 0 直接获胜
    if new_bb >= GOAL:
        return 0

    # 2. 临近终点，保守
    if needed <= 4:
        return 1
    elif needed <= 8:
        return 2

    # 3. Boar Brawl 有效收益很高
    if effective_bb >= 6:
        return 0

    # 4. 分差博弈
    if score > opponent_score + 15:
        return 3
    if opponent_score > score + 15:
        return 6

    # 5. 胶着默认
    return 4
```
