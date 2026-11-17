# Steam 游戏数据分析 · 项目工作报告

2026-11-18 · 徐希（Xi Xu）

## 项目概述

本项目用 Python 分析了约 12.4 万款 Steam 游戏，回答一个问题：哪些因素和游戏的口碑（好评率）有关？目前完成了两个阶段，最主要的结论是：控制其他因素后，$40 以上的游戏好评率仍比免费游戏低约 13 个百分点。

| 项目 | 内容 |
| --- | --- |
| 数据 | [FronkonGames Steam Games Dataset](https://huggingface.co/datasets/FronkonGames/steam-games-dataset)（MIT 协议），124,146 款游戏、41 列，数据截至 2026 年 9 月 |
| 工具 | Python、pandas、matplotlib、statsmodels、Jupyter |
| 代码仓库 | [github.com/XiXu0827/steam-games-analysis](https://github.com/XiXu0827/steam-games-analysis) |
| 产出 | 2 个 notebook、8 张图表、英文 README |
| 状态 | 第一、二阶段完成；最终总结进行中 |

## 工作内容

两个阶段都在 2026 年 11 月 18 日完成，代码、图表和本报告（HANDOFF.md）已全部上传 GitHub。

| 阶段 | 工作内容 | 产出 |
| --- | --- | --- |
| 第一阶段：数据清洗与探索性分析 | 去重、解析发行日期、拆分“预估拥有人数”区间、计算好评率；只保留评价数 ≥ 10 的游戏做好评率分析；分析发行趋势、类型分布、好评率分布、价格区间 | `steam_analysis_phase1.ipynb`、4 张图 |
| 第二阶段：好评率影响因素 | 按类型、评价数、发行年份做单因素分析；建立多元线性回归（稳健标准误 HC1），控制评价数、类型和年份后检验价格的影响 | `steam_analysis_phase2.ipynb`、4 张图 |
| 文档与发布 | 编写英文 README（含全部发现和图表）、`requirements.txt`、`.gitignore`；建立公开 GitHub 仓库 | GitHub 仓库 |

## 当前状态与文件位置

本地项目文件夹：`C:\Users\w2752\PycharmProjects\steam-analysis`（Windows）。GitHub 仓库：`XiXu0827/steam-games-analysis`（公开，分支 `main`）。本地和 GitHub 上的文件一致。Claude GitHub App 已安装在这个仓库上（仅此一个仓库），Claude 可以直接提交；其他工具用 git 推送也可以。

| 文件 | 说明 | GitHub 上 |
| --- | --- | --- |
| `data/games.parquet` | 原始数据，约 192 MB（HF 仓库里的 `data/train-00000-of-00001.parquet`） | 不上传（已写入 `.gitignore`） |
| `steam_analysis_phase1.ipynb` | 第一阶段 notebook，已运行，含输出 | 已上传 |
| `steam_analysis_phase2.ipynb` | 第二阶段 notebook，已运行，含输出 | 已上传 |
| `figures/01_…08_*.png` | 两个阶段共 8 张图 | 已上传 |
| `README.md` | 英文项目说明，含两个阶段的发现 | 已上传 |
| `requirements.txt` | 依赖列表，含 `statsmodels` | 已上传 |
| `.gitignore` | 忽略 `data/`、`.ipynb_checkpoints/`、`__pycache__/` | 已上传 |
| `HANDOFF.md` | 本报告的 Markdown 版 | 已上传 |
| `games.parquet.tmp` | 多余的数据副本，可删除 | — |

**运行环境：** Windows，已装 Python。需要 `pandas`、`pyarrow`、`matplotlib`、`statsmodels`、`jupyter`。notebook 用相对路径（`data/games.parquet`、`figures/`），需要在项目根目录下启动。

**简历：** 中文简历做在 Claude 的设计画布「徐希 简历设计」里（单栏 A4，已很满，再加内容需要删减其他条目）。项目经历第一条当前写的是：

> **Steam 游戏数据分析（进行中）**｜个人项目 · github.com/XiXu0827/steam-games-analysis
>
> - 使用 Python（pandas、matplotlib）对约 12.4 万款 Steam 游戏数据进行清洗与探索性分析：解析发行日期、拆分拥有人数区间、计算好评率，并以评价数 ≥ 10 过滤噪声样本。
> - 使用 statsmodels 建立多元线性回归，控制评价数、游戏类型与发行年份后，发现 $40 以上游戏好评率仍比免费游戏低约 13 个百分点；评价数每增加 10 倍，好评率平均高约 5 个百分点。

## 数据与方法要点

接手时最容易出错的是数据格式和已定的分析口径，继续分析时请沿用下面这些规则，结果才能和前两个阶段对得上。

**用到的列和格式**

- `release_date`：文字，如 `"Aug 1, 2023"`，用 `pd.to_datetime(..., format="mixed")` 可全部解析；范围 1997-06-30 至 2026-09-08。
- `estimated_owners`：区间文字，如 `"20000 - 50000"`，拆成 `owners_low` / `owners_high`。
- `price`：美元，0 表示免费（免费游戏占 21.3%）。
- `positive` / `negative`：好评数 / 差评数。
- `genres`：字符串数组，一款游戏可属于多个类型，分析时用 `explode`。
- `tags`：这个版本里基本是空数组，**不能用**。

**清洗规则**

1. 删除 1 条名字为空的记录；`appID` 无重复。
2. `total_reviews = positive + negative`；`positive_rate = positive / total_reviews`，没有评价的游戏记为缺失（不是 0）。
3. 凡是分析好评率，只用**评价数 ≥ 10** 的游戏（56,657 款）；第二阶段另加**发行年份 2008–2025**（56,473 款）。
4. 2026 年数据不完整，按年统计时只到 2025 年。
5. 类型分析去掉 `Free To Play` 和 `Early Access`（它们是销售方式，不是玩法类型）。
6. 价格区间：`bins = [-0.01, 0, 4.99, 9.99, 19.99, 39.99, inf]`，标签 Free / <$5 / $5–10 / $10–20 / $20–40 / $40+。

**回归模型（第二阶段）**

```
smf.ols("positive_rate ~ C(price_band, Treatment('Free')) + log_reviews + C(year) + g_Indie + g_Casual + g_Action + g_Adventure + g_Simulation + g_Strategy + g_RPG + g_Sports + g_Racing + g_Massively_Multiplayer", data=rated).fit(cov_type="HC1")
```

`log_reviews = log10(total_reviews)`；`g_*` 是 0/1 类型变量。n = 56,473，R² = 0.157。

**风格约定**

- 图表文字用英文（避免中文字体问题，也方便放 GitHub），主色 `#1F5FA8`，强调色 `#C2410C`，保存为 `figures/NN_name.png`（dpi 150）。
- notebook 代码用英文，说明和注释用中文；每个结论的数字都由代码打印出来。
- README 用英文，是 GitHub 首页。

## 关键结果

下表数字都来自 notebook 的实际输出，简历和 README 引用的是同一组数字。

| 指标 | 数值 | 来源 |
| --- | --- | --- |
| 清洗后游戏数 | 124,145 | 第一阶段 |
| 没有任何评价的游戏 | 33.2% | 第一阶段 |
| 评价数 ≥ 10 的游戏 | 56,657（45.6%） | 第一阶段 |
| 好评率中位数（评价数 ≥ 10） | 81.2%；≥ 80% 的占 53.5%，< 50% 的占 8.4% | 第一阶段 |
| 每年新发行游戏 | 2015 年 2,515 款 → 2025 年 24,973 款（约 10 倍）；2020–2025 年发行的占 74% | 第一阶段 |
| 带 Indie 标签的游戏 | 66% | 第一阶段 |
| 各价位好评率中位数 | Free 78% · <$5 81% · $5–10 84% · $10–20 83% · $20–40 81% · $40+ 69% | 第一阶段 |
| 各类型好评率中位数 | Casual / Indie 82%（最高），Massively Multiplayer 67%（最低） | 第二阶段 |
| 评价数 10k+ 的游戏 | 好评率中位数 89% | 第二阶段 |
| 按发行年份 | 2014–2018 年最低（73–75%），2025 年 90% | 第二阶段 |
| 回归：$40+ vs 免费 | −13.3 个百分点（95% CI −16.0 至 −10.6）；原始差距 −9.3 | 第二阶段 |
| 回归：$20–40 vs 免费 | −3.7（−4.8 至 −2.6） | 第二阶段 |
| 回归：$20 以下各价位 vs 免费 | <$5 +0.1 · $5–10 +1.2 · $10–20 −0.1 | 第二阶段 |
| 回归：评价数每增加 10 倍 | +5.1 个百分点 | 第二阶段 |
| 回归：类型系数 | MMO −11.6 · Simulation −6.3 · Indie +2.2 · Casual +1.8 | 第二阶段 |

## 待办与下一步

- [ ] 以后每次更新 notebook、图表或 README，都同步提交到 GitHub，并更新本报告和 `HANDOFF.md`。
- [ ] 删除两个多余的数据副本：项目文件夹里的 `games.parquet.tmp`，以及“下载”里的 `6a5383ed-4071-43af-9135-70654937ee65.tmp`。
- [ ] 第三阶段（收尾）：做稳健性检验，比如把评价数门槛换成 50 重新跑回归；可以补充游戏时长（`average_playtime_forever`）、平台（`windows` / `mac` / `linux`）、`dlc_count` 等还没用过的变量。注意 `metacritic_score` 为 0 表示缺失。
- [ ] 写最终总结，然后去掉 README 和简历里的“进行中”。
- [ ] 下一个项目（已规划）：剧本杀 AI 推理助手，大模型 API + RAG 检索 + Streamlit 界面，对应 AI 开发岗位。

## 交给其他模型时的说明

新对话开头先贴上这段话，再贴上本报告全文。如果对方能直接读项目文件夹，就让它读文件夹里的 `HANDOFF.md`（就是本文件）。

```
我在做一个用于求职的数据分析作品集项目：用 Python 分析 Steam 游戏数据，研究哪些因素和游戏好评率有关。
前两个阶段已经完成，下面是工作报告。请先读完报告再开始。

要求：
1. 项目文件在 C:\Users\w2752\PycharmProjects\steam-analysis，代码仓库是 github.com/XiXu0827/steam-games-analysis。
2. 继续分析时沿用报告里“数据与方法要点”的清洗规则和风格约定，新内容放在 steam_analysis_phase3.ipynb，图表编号从 09 开始。
3. 任何写进 README 或简历的数字，都必须是 notebook 实际运行出来的结果。
4. 用中文解释，代码用英文。我喜欢先理解每一步的概念再往下做，面试时需要能讲清楚每个决定。

第一件事：[写下你想做的下一步，比如“开始第三阶段的稳健性检验”]
```
