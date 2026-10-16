# FGO 羁绊计算器 / FGO Bond Calculator

A fully offline, single-file web app that computes the optimal 5-servant team
for maximizing bond-point gain per round in *Fate/Grand Order* (国服 / CN server).
毕竟是个vibe coding的产物，但后续应该会更新和维护的（根本没人会看到这东西吧？）
简述:该程序的作用就是在已知你的BOX的前提下（这需要抓包的配合）自动计算出羁绊最大化的队伍（怎么过图先别管，先贪再说）
目前的使用环境是今年年底的空想树战，所以所有冠位都算了额外礼装框，后面跟进会去掉
> 📌 **2026-10-15 更正**：上面这句已不适用于当前代码。现在的默认值是
> **「仅 1 个冠位给免费框」**（机制参数页的「冠位额外礼装框」= `仅 1 个冠位（实际）`）；
> 想让每个冠位都给，把该项改成 `每个冠位都给` 即可。
预计后面可能会出现的功能：限定职介的组队（方便冠位战） 优化冠位从者的计算 给出特定羁绊值的刷法（114514和1314520)
> 中文完整技术文档与验证记录见 [DELIVERY.md](./DELIVERY.md)。

> ⚠️ **2026-10-15 重要修正**：早期版本有两个会给出**错误结论**的 bug，已修复并推送。
> 如果你在 2026-10-15 之前克隆过本仓库，请务必 `git pull`。
> 详见下方 [Fixed bugs](#fixed-bugs-2026-10-15)。

## What it does

Given your servant box, craft-essence (礼装) pool, and game parameters, the
calculator finds the team composition that maximizes per-round bond gain,
accounting for:

- Front-row ×1.2 multiplicative bonus
- 15-bond team bonus and the Mashu (玛修) bonus
- Craft-essence self / support rates and Cost constraints
- "Crown" (冠位) free-frame logic

Six tabs:

1. **自动最优 (Auto Optimal)** — exhaustive search over the candidate pool, top-N teams with per-servant bonus breakdown.
2. **手动配队 (Manual)** — pick 5 servants, see the live breakdown.
3. **我的 BOX (My Box)** — 275 servants, edit bond / crown state; per-servant **屏蔽 (exclude)** and **候选池 (candidate-pool)** columns; import a new packet capture.
4. **礼装 (Craft Essences)** — 49 essences, toggle and tune self / support / Cost; one-click refresh from Mooncell.
5. **机制参数 (Parameters)** — base bond, Cost cap, 15-bond / Mashu / front-row bonuses.
6. **说明与依据 (Sources)** — data provenance for every rule.

## How to run

No build, no server, no dependencies. **Double-click `bond_calculator.html`**
(or open it in any modern browser). Everything — your box data, the essence
pool, the servant static table, and the full solver — is embedded in the file.

To refresh your box from a new capture, drop the Reqable `ac.php` export onto
the "我的 BOX" tab (it auto-decodes and rebuilds), or run the dev pipeline's
`python update_box.py`.

## Exclusion list (屏蔽名单)

Some of your servants should never enter the candidate pool — you are saving them
for another team, they are already bonded out, or you simply do not want to bring
them. The **我的 BOX** tab has a per-servant **屏蔽** column (plus a *只看已屏蔽*
filter and *清空屏蔽名单* button), and a bulk helper that adds everyone at
bond ≥ N in one click.

- The list is **manual by design**. Bond caps are affected by 梦火 (Dreamfire) —
  a non-crowned servant can be pushed past bond 10 — so the app cannot reliably
  infer who is "already full". You decide; the calculator just honours it.
- **Automatic Optimal** only. The **手动配队 (Manual)** tab is untouched: it is
  your hand-picked team, so an excluded servant is still selectable there.
- Excluded servants are removed *before* the crown-group / Pareto-front quota is
  applied, so they do not silently eat one of the N candidate slots.
- The list is saved with the rest of your state (and in the JSON export), and is
  pruned automatically when you import a new box.

## Candidate pool (候选池) — and why it is adjustable

The exhaustive search cannot run over all 275 servants (that is ~12.6 **billion**
5-servant combinations), so the app first builds a **candidate pool** by a
three-part heuristic:

1. **Must-include** — already-crowned / bond-15 / Mashu.
2. **Top N per crown group** (default 3), ranked by matched-essence count.
3. **Top N of the Pareto front** among ordinary servants (default 22), where
   "front" means *no other servant is both cheaper and covers at least as many
   essences*.

**This heuristic can miss the true optimum.** Measured against full enumeration
within a single class:

| Class | pool / total | pool best | full best | gap |
|---|---|---|---|---|
| Lancer | 14/34 | 58,056 | 64,797 | **−11.61%** |
| Rider | 8/30 | 62,623 | 66,537 | −6.25% |
| Caster | 11/36 | 62,840 | 64,362 | −2.42% |
| Assassin | 12/29 | 66,754 | 68,059 | −1.95% |
| Archer | 13/26 | 58,274 | 59,361 | −1.87% |
| Saber | 12/35 | 64,362 | 65,232 | −1.35% |
| Avenger | 7/11 | 56,969 | 57,578 | −1.07% |
| Berserker | 13/35 | 60,666 | 61,101 | −0.72% |

8 of 11 classes were affected. So the pool is **fully adjustable**:

- **候选池 column** in the **我的 BOX** tab — a three-state dropdown per servant:
  *自动 (auto)* / *强制拉进 (force-in)* / *强制踢出 (force-out)*.
  Force-in bypasses every automatic cut-off; force-out is like excluding, but
  scoped to "not this time" rather than a permanent block.
- **全量模式 (full mode)** — skip the heuristic entirely and put every
  non-excluded servant in the pool. Guarantees no missed optimum. Only usable
  when the resulting combination count is manageable (e.g. a single class), and
  the built-in guard refuses to run past 3.5 M combinations instead of hanging
  the browser.
- Precedence: **class filter > exclusion list > force-out > auto heuristic > force-in**.
- Both lists persist with your saved state and are pruned on a new box import.

## Fixed bugs (2026-10-15)

Two bugs in earlier revisions produced **wrong answers**, not crashes:

1. **Essence applicability was entirely misaligned.** Each servant's
   "which essences apply" bitmask had been generated against an older **46-essence**
   table, while the pool had grown to **49** — three essences were *inserted* at
   indices 9–11, shifting every index ≥ 9 by three. Consequences: 迦勒底午餐时光
   (lunch) was read as an FSN-only essence and matched **0 servants**; the seven
   英灵逢魔 essences were read as unconditional ones and matched **all 275**; and
   the three newest essences could never activate. Now the bitmask is always
   recomputed from the servant trait table and the current essence table
   (13,475 bits audited: 275/275 rows wrong → **0 wrong**).
2. **The support servant was counted as a bond recipient.** A friend support
   gains bond on *their* account, not yours — so it must not appear in your
   total. It had been added as a sixth beneficiary (and inflated every essence's
   coverage count by one). The **essence on the support still applies** to your
   five servants, which is a separate, genuine effect and is retained.

## Features added (2026-10-15)

- **冠位从者至少上场一位 (at least one crowned servant)** — hard constraint in
  the solver. "Crownable" (Lv100 + 3× skill 10) counts; you need not have crowned
  them in-game yet.
- **尽量用满 Cost (use as much Cost as possible)** — a *tie-breaker*, not a
  sacrifice: among teams with equal bond gain, the one using more Cost is shown
  first. Verified over 19 equal-score groups (strictly descending Cost when on;
  17/20 groups unordered when off).
- **好友冠位礼装框数 (friend crown essence frames)** — default **2**
  (1 regular support frame + 1 crown bonus frame).
- **无重复模式 (no-repeat mode)** — output **K** teams (default 3) in which most
  servants appear at most once across all teams; a few servants you mark
  **重复豁免 (repeat-exempt)** may appear in every team. See below.

## No-repeat mode (无重复模式)

When you plan to farm one node with **K** alternating teams (so the same servant is
never (or rarely) double-booked), turn on **无重复模式** in the **自动最优** tab
and set **K** (输出几支互不重复的队伍, default 3, range 1–20). Instead of one
optimal team, the solver returns K teams that cover most of your box without
overlap.

- **How it works — greedy, one team at a time.** Each round it runs the normal
  exhaustive search over the *current* candidate pool, takes the single best team,
  then **removes every non-exempt servant** in that team from the pool and searches
  again. Enumeration cost shrinks each round (Σ C(n−5k, 5) ≪ one-shot C(n, 5)),
  so it is fast even on the full 275-servant box.
- **重复豁免 (repeat-exempt) column** in the **我的 BOX** tab — a per-servant
  checkbox. An exempt servant (e.g. a Mashu battery, or a servant you want on every
  team) stays in the candidate pool and may appear in all K teams. **Mashu (玛修)
  is always exempt** because she is on by default every team.
- **It is an approximation, not a globally optimal K-team set.** Each round picks
  the local optimum, so the K-th team may be slightly worse than a globally
  co-ordinated plan. For "rotate teams to farm one node" this is more than enough,
  and the result page says so. If you need a single best team, leave the mode off.
- Honours the same constraints as normal mode: Cost cap, crown / Cost tie-breakers,
  候选池 force-in/out, and the 屏蔽 list all still apply.
- Verified by an automated headless-Chrome probe: with no exemptions, the K teams
  share only Mashu; marking a servant exempt makes it repeat across all K teams;
  zero JS errors on the result page.

## Craft-essence data (礼装数据)

The 49 bond-boosting craft essences are baked into the file. To pick up new
ones without waiting for a rebuild, open the **礼装** tab and click
**联网更新礼装（Mooncell）**: the page pulls the complete essence table from
[Mooncell](https://fgo.wiki) (MediaWiki API — anonymous CORS is enabled, so a
`file://` page can fetch it directly), diffs it against the local table, and
stores only the missing entries in `localStorage`.

- **Existing entries are never modified.** Condition parsing is best-effort;
  an unrecognised condition makes the button *skip* that essence and say so,
  rather than guessing and corrupting numbers that were already correct.
- Bond-boosting essences store the **max-limit-break** value (e.g. 4% → 20%).
- The trait ids used by essence conditions (`100` Saber, `101` Lancer,
  `102` Archer, `103` Rider, `104` Caster, `105` Assassin, `106` Berserker,
  `302` Neutral, …) were derived by set arithmetic over the servant static
  table and cross-checked, not guessed.

## Highlights

- **Single-file, fully offline** — ~270 KB HTML embedding a 275-servant box,
  49 craft essences, a 474-servant static table, and the complete solver.
- **Verified algorithm** — validated 60/60 against a Python brute-force oracle;
  regression-checked against prior conclusions; real-capture import is idempotent
  (275 servants, 0 field differences).
- **Zero-allocation solver** — typical case ~0.9 s, worst case ~7.7 s.

## Development

This artifact was produced via AI pair-programming (**WorkBuddy** + **DeepSeek
Harness**). See [DELIVERY.md](./DELIVERY.md) for the build / test pipeline, the
bugs that were found and fixed, and the full verification record.

## License

MIT — free to use and modify.
