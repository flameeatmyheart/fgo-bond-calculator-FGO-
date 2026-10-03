# FGO 羁绊计算器 / FGO Bond Calculator

A fully offline, single-file web app that computes the optimal 5-servant team
for maximizing bond-point gain per round in *Fate/Grand Order* (国服 / CN server).
毕竟是个vibe coding的产物，但后续应该会更新和维护的（根本没人会看到这东西吧？）
简述:该程序的作用就是在已知你的BOX的前提下（这需要抓包的配合）自动计算出羁绊最大化的队伍（怎么过图先别管，先贪再说）
目前的使用环境是今年年底的空想树战，所以所有冠位都算了额外礼装框，后面跟进会去掉
预计后面可能会出现的功能：限定职介的组队（方便冠位战） 优化冠位从者的计算 给出特定羁绊值的刷法（114514和1314520)
> 中文完整技术文档与验证记录见 [DELIVERY.md](./DELIVERY.md)。

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
3. **我的 BOX (My Box)** — 275 servants, edit bond / crown state; import a new packet capture.
4. **礼装 (Craft Essences)** — 46 essences, toggle and tune self / support / Cost.
5. **机制参数 (Parameters)** — base bond, Cost cap, 15-bond / Mashu / front-row bonuses.
6. **说明与依据 (Sources)** — data provenance for every rule.

## How to run

No build, no server, no dependencies. **Double-click `bond_calculator.html`**
(or open it in any modern browser). Everything — your box data, the essence
pool, the servant static table, and the full solver — is embedded in the file.

To refresh your box from a new capture, drop the Reqable `ac.php` export onto
the "我的 BOX" tab (it auto-decodes and rebuilds), or run the dev pipeline's
`python update_box.py`.

## Highlights

- **Single-file, fully offline** — ~190 KB HTML embedding a 275-servant box,
  46 craft essences, a 474-servant static table, and the complete solver.
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
