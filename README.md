# AI Life Scheduler · AI 自动排程系统

> 一个用 **Notion + Claude + Google 日历** 搭建的个人日程自动化系统：我只负责录入任务和打勾，排程、顺延、同步全部自动完成。
>
> A personal scheduling system built with **Notion + Claude + Google Calendar**. I only add tasks and tick them off; scheduling, rollovers and calendar sync run automatically every night.

[中文](#中文) · [English](#english)

---

## 中文

### 背景

2026 年秋天，我同时在推进五条线：律所实习、秋招（投递、笔试、面试）、毕业论文定稿、法语学习（目标 DELF B1），以及把作息从凌晨入睡调整到 21:00 入睡。

手动排日程的问题很快暴露出来：

- 临时插入的面试、测评会打乱整天的计划，重排很耗时间；
- 没完成的事容易被遗忘，或者在多个表格里状态不一致；
- 计划总是排得过满，实际执行率低，晚上越拖越晚。

### 方案

我把排程规则写成一套明确的「规则说明」，交给 AI 每晚自动执行：

```
任务收集（Notion） ──┐
法语进度表 ──────────┤
求职投递表 ──────────┼──▶ 每晚 19:00 自动排程（Claude） ──▶ 生活安排（Notion） ──▶ Google 日历 ──▶ iPhone
作息与专注规则 ──────┘              ▲                                   │
                                    └──────── 我只在这里打勾「完成」 ◀──┘
```

完整流程图见 [docs/architecture.md](docs/architecture.md)。

### 关键设计

| 设计              | 解决的问题                                                           |
| --------------- | --------------------------------------------------------------- |
| **单点打勾，自动回写**   | 只在「生活安排」里勾完成，系统自动同步到法语进度表、投递表和任务库，避免多处状态不一致                     |
| **没完成自动顺延**     | 未勾选的事项在第二天重新排入，旧条目自动清理，日历上不留过期事项                                |
| **依赖任务连带顺延**    | 论文的「润色 → 调整引注 → 调整格式 → 发给导师」有先后顺序，前一步延期，后续步骤整体后移，并检查最终截止日是否还赶得上 |
| **截止日倒排 + 优先级** | 按截止时间由近到远、优先级由高到低排入空档                                           |
| **每日上限，防止过载**   | 根据真实专注数据设定上限（工作日下班后 ≤ 2.5 小时，周末 ≤ 5 小时），排不下就提醒，而不是压缩睡眠          |
| **作息渐进调整**      | 每 2 天把入睡和起床时间提前 15 分钟；连续 3 天没做到就暂停在当前档位                         |
| **睡前固定留白**      | 睡前 1 小时洗漱放松、法语后 30 分钟留白，不排任何任务                                  |

数据库字段设计见 [docs/schema.md](docs/schema.md)，完整排程规则见 [prompts/daily-scheduler.md](prompts/daily-scheduler.md)。

### 用真实数据迭代

上线几天后，我导出了自己的专注计时记录做分析（[analysis/](analysis/)），发现原有假设和实际情况差距很大：

- 单次专注的中位数只有 **47 分钟**，原规则却允许 2 小时的任务块；
- 早上几乎没有专注记录，原规则却在上班前排了论文；
- 专注集中在 14:00–18:00 和晚上，最后一段经常拖到午夜后结束。

据此我把任务块改为 45–60 分钟（最长 90 分钟），取消上班前的专注任务，并加入每日上限。完整的迭代记录见 [docs/iterations.md](docs/iterations.md)。

### 效果

- 每天只需要做两件事：往任务库里录入任务、做完后打勾；
- 临时插入面试或笔试时，说一句话就能重排当天的日程；
- 求职投递每天固定处理 2 家，不再积压；
- 作息开始按计划逐步提前。

### 可迁移到法务工作

这套方法本质上是**把规则写清楚，让工具稳定执行**，可以直接迁移到法务场景：

- **合同审核排期**：按业务截止日倒排审核、修订、签署各环节，前一环节延误时自动后移后续环节；
- **合规事项跟踪**：备案、年报、个人信息保护影响评估等周期性事项自动提醒，未完成的自动顺延并预警；
- **诉讼与仲裁期限管理**：举证、答辩、上诉期限倒排，超出容量时提前预警，而不是临近截止才发现。

### 技术栈

- **Notion**：任务库、日程库、规则页（规则写在页面上，修改后下次排程自动生效）
- **Claude**：每晚定时运行，读取规则和数据，完成同步、顺延、排程
- **Google Calendar**：接收排程结果，同步到 iPhone
- **Python**：分析专注计时数据

### 仓库结构

```
├── README.md
├── docs/
│   ├── architecture.md     系统流程图
│   ├── schema.md           数据库字段设计
│   └── iterations.md       迭代记录
├── prompts/
│   └── daily-scheduler.md  每晚排程规则（脱敏）
├── analysis/
│   ├── focus_analysis.py   专注数据分析脚本
│   ├── sample_data.csv     示例数据（合成）
│   └── findings.md         分析结论
└── screenshots/            界面截图
```

> 隐私说明：本仓库已去除所有真实链接、账号、公司名称和个人记录，`sample_data.csv` 为合成数据。

---

## English

### Background

In autumn 2026 I was juggling five workstreams at once: a law-firm internship, campus recruitment (applications, online tests, interviews), finalising my master's thesis, learning French (target: DELF B1), and shifting my sleep from after midnight to a 9 pm bedtime.

Planning by hand broke down quickly: last-minute interviews derailed whole days, unfinished tasks slipped through or drifted out of sync across several trackers, and plans were consistently over-packed, so I kept working later into the night.

### Approach

I wrote my scheduling logic down as an explicit rulebook and let an AI agent run it every evening. I add tasks to a Notion inbox and tick items off in one place; everything else is automatic. See [docs/architecture.md](docs/architecture.md) for the full flow.

### Key design decisions

- **Single point of truth for completion** – ticking an item in the schedule writes back to the French tracker, the job-application tracker and the task inbox.
- **Automatic rollover** – unfinished items move to the next day and stale entries are cleaned up, so the calendar never shows outdated items.
- **Dependency-aware rescheduling** – sequential tasks (e.g. thesis: polish → citations → formatting → send to supervisor) shift together, with a check against the final deadline.
- **Deadline-first prioritisation** within available time slots.
- **Daily capacity caps based on real data** (≤ 2.5 h after work on weekdays, ≤ 5 h on weekends); overflow triggers a warning instead of cutting into sleep.
- **Gradual sleep shift** – bedtime and wake time move 15 minutes earlier every two days, pausing if three days in a row are missed.
- **Protected buffers** – one hour before bed and 30 minutes after French study are never scheduled.

### Data-driven iteration

After a few days I analysed my own focus-timer exports ([analysis/](analysis/)). Median focus session: **47 minutes**; almost no focused work happened in the morning; most deep work fell between 2–6 pm and in the evening. I cut task blocks to 45–60 minutes (max 90), stopped scheduling deep work before my commute, and added daily caps. See [docs/iterations.md](docs/iterations.md).

### Transferable to legal work

The core skill is **turning rules into a reliable process**. The same pattern applies to in-house legal operations: contract-review pipelines with dependency-aware deadlines, recurring compliance obligations (filings, annual reports, privacy impact assessments) with automatic escalation, and litigation/arbitration deadline management with early capacity warnings.

### Stack

Notion (task and schedule databases, rule page) · Claude (nightly scheduled agent) · Google Calendar (sync to iPhone) · Python (focus-data analysis)

> Privacy note: all real links, account identifiers, company names and personal records have been removed. `sample_data.csv` is synthetic.

