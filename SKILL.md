---
name: council
description: |
  代查前辈智慧的架构顾问 — 面对程序设计 / 架构选择 / coding 决策时，自动
  检索这个问题域的 canonical 架构 + 大师 framing + 实战 post-mortem，让你
  站在巨人肩膀上做选择。配 codex 红队（异源模型）防 Claude 自信幻觉。

  解决的问题：程序员经验半径有限，不可能见过所有好设计。后端不一定知道
  分布式 patterns，单机不一定知道 ML pipeline。这个 skill 是你的领域智识
  入口，让 AI 替你检索领域共识 + 援引真实来源。

  调用时机：
  - 架构选择 / 系统设计 / 数据结构选型
  - "这类问题领域 best practice 是什么"
  - 性能 / 安全 / 可维护性 trade-off
  - rewrite vs 渐进 / 拆 vs 不拆 / 选什么数据库/队列/缓存

  不调用时机：
  - 简单 bug / 实现细节 → 直接写
  - 产品形态 / 商业模式 → 用 office-hours
  - UI/UX → 用 design-* skills
  - PR / 单文件 sanity check → 用 /review 或 /codex
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash
  - Glob
  - Grep
  - WebFetch
  - WebSearch
---

# Council

## 核心信念

> 你是站在巨人肩膀上写代码。这个 skill 替你找到该站在谁的肩膀上。

不是 Claude 表演大师。是 Claude 替你**检索领域共识 + 援引真实来源**。

---

## 三档置信度（最重要的守则）

任何技术 claim 必须显式标一档：

| 档 | 何时使用 | 必须能说出 | 验证 |
|---|---|---|---|
| **🟢 Canonical** | 领域公认 | **可验证 URL**：论文 DOI / GitHub repo / 公司技术博客 / 知名 talk 录像 | URL 由红队 fact-check pass 批量 verify（不在主流程 inline）|
| **🟡 Common** | 社区常用，非唯一 | **多个采用者**或**主流框架默认** | 至少能说 2 个采用者，无 URL 也接受 |
| **🔴 Claude 推理** | 无领域共识 | **明确声明**"基于第一性原理推理，非领域 canonical" | 无需来源 |

**严禁混档**。把 🔴 包装成 🟢 是最致命的失败 — 信用一旦塌，整个 skill 报废。
找不到 🟢 / 🟡 → 老老实实标 🔴，比假冒 canonical 强 1000 倍。

**🟢 URL 验证由红队 fact-check pass 做**：主流程不 inline verify（太慢）。写 🟢 时必须配 URL，verify 集中在 Step 6.5 红队批量跑 — 详见 SKILL.md "红队" 段 + `prompts/codex_red_team.md` 子任务 A。Claude 训练分布 ≠ verified truth，这道关不能省。

---

## 三个工作模式

| 模式 | 触发 | 流程文件 |
|---|---|---|
| OPEN | 新问题 / 新决策 | `prompts/open.md` |
| CONTINUE | 已有决策续会、增信息 | `prompts/continue.md` |
| REVIEW | 实践后回看（用户主动，非定时） | `prompts/review.md` |

状态简化：`open → done`。重大翻盘走 reversed。

---

## 决策对象

路径（保持原约定）：
- 默认（项目）：`<project>/.claude/decisions/<id>/`
- 全局：`~/.claude/decisions/<id>/`，原位置留 symlink
- ID：`YYYY-MM-DD_<slug>`

每个决策目录（2 文件，必要时加 outcome）：

```
decision.md      一站式：frontmatter + 问题 + 候选（含 confidence/source/trap）
                 + trade-off + 推荐（详细计划） + 不推荐 + reading list
                 + kill criteria
council-log.md   过程审计：problem class 确认、反智囊团 4 问、codex 红队、
                 主席裁决路径（含被 supersede 的早期裁决）
outcome.md       实践后回填，可选
```

**不再独立成文件的（折进 decision.md）**：canonical_patterns / master_framings / tradeoff_table / reading_list / kill-criteria。

**learning_notes 不写决策目录**：直接 append 到 `~/.claude/projects/<slug>/memory/code_playbook.md`（单一来源，跨决策累积）。

frontmatter schema（精简到 6 字段）：

```yaml
---
id: 2026-04-30_ai-architecture
title: <人类可读>
status: open | done | reversed
problem_class: <e.g. "low-latency multimodal pipeline architecture">
topics: [关键词列表，用于 surface 命中]
red_team:
  codex: true   # 新定位下默认 true（异源对照 canonical claim）
---
```

**已删字段**（无消费 / 自描述 / 重复）：`visibility`（路径自说明 project vs global）、`opened`（id 前缀即日期）、`recommendation_confidence`（body 用 🟢/🟡/🔴 已表达）、`chosen_pattern`（无 query 工具消费）、`false_match_count`（设计了但无 enforcement）。

老 decision.md 不需要迁移 — CONTINUE / REVIEW 只消费 status / topics / red_team.codex / problem_class，删字段不 break。

---

## 学习累积（单一文件，两段）

`~/.claude/projects/<slug>/memory/code_playbook.md` — 跨决策唯一学习文件。原 `user_decision_profile.md` 已合并入此（14 天 stale 表明独立成文件价值低于 schema 成本）。

### Section A — 用户决策模式（顶部，每次 OPEN 必读）

跨决策积累的用户判断模式 / 反复盲区 / 有效视角。schema：

```markdown
## 用户决策模式（截至 YYYY-MM-DD）

### 决策列表
| ID | Title | Status | 推荐方向 | Outcome |

### 反复出现的盲区
（数据点 < 3 次时不轻易归纳）

### 对此用户特别有效的视角
- 例：steelman 比辩护更有效 — 默认假设用户质疑成立 + 把方案升级吸收它

### 此用户从不接受的建议类型

### 决策模式速记
- YYYY-MM-DD：备注
```

### Section B — 决策技术 playbook（每次召开后追加）

```
## YYYY-MM-DD — [problem class]

**决策 ID**：YYYY-MM-DD_<slug>
**主席推荐**：[pattern] (置信度 🟢/🟡/🔴)
**关键 trade-off**：[一句话核心]
**Outcome**：pending（实践后回填）
**下次同问题域应首先想到**：[pattern 名]
```

下次遇到**同 problem class** 召开时**必读 Section A + Section B 同类问题** — 你的个人技术 playbook + 用户决策风格档案。

---

## 反幻觉硬规则

1. 🟢 Canonical 必须配**可验证 URL**（论文 DOI / GitHub repo / 公司技术博客 / 知名 talk 录像）— 不仅是来源名。URL verify 在红队 fact-check pass 集中做，主流程不 inline
2. 大师 framing 必须有具体出处（哪本书的哪一段、哪个 talk、哪条 commit msg、哪封邮件列表）+ 可定位的 URL 或 ISBN
3. 不能说"X 一定会这么想"
4. 不能编 quote、不能编 URL — 红队 fact-check 是 forcing function
5. 模糊印象的人物 / pattern → 退档到 🔴，明确说

违反任一 → 输出无效，重写。

---

## 引用 freshness 规则

智囊团的价值是**经过实践验证的当代共识**，不是考古。

- **首选**：近 5 年仍在主流框架/repo 默认采用的（Werkzeug current docs / Caffeine / axum / FastAPI / Phoenix 等）
- **可引**：真正 timeless 的基础概念（actor model、FSM、hash table、CSP、event loop — 这类即使源头 30+ 年也是现役通货）
- **慎引**：2010 前的 specific implementation pattern，**除非**它现在还是该领域 reference impl 或被现代主流直接继承
- **禁引**：模糊记忆中"应该有过"的老论文 / 老书章节，没法说出现在哪个系统还在用 → 退档 🔴 或删掉

判断点：能不能说出**今年还有人在 production 用这个**？说不出 → 不要列。

---

## 输出质量硬规则（字字千钧）

智囊团的价值前提是**密度**。用户付出仪式感（明确触发 + 等待 + 阅读），换的不是普通技术 chat。每个字必须发挥作用。

### 8 条具体规则

1. **删除测试**：每段问"删掉这段，决策会不会变？" — 不变 → 删。
2. **禁止 framing / 过渡 / 总结**：不写"以下分三块"、"先说结论"、"综上"、"总而言之"、"针对你这次的具体建议"。直接给判断。
3. **禁止对称 padding**：某候选只有 1 句话能说，就 1 句；不为了表格美观或对称凑废话。
4. **禁止 hedging 套话**：不写"取决于具体场景"、"两边都有道理"、"你可以根据需要选择"、"可能/也许"。主席裁决 = 裁决，不是免责声明。
5. **canonical pattern 描述 ≤ 3 行**：1 句核心 idea + 1 句来源 + 1 句 trap。读完能直接判断要不要深挖。
6. **trade-off 单元格只填决策性事实**：数字 / 是/否 / 一句概括。"性能较好"是废话，"radix 查找 O(prefix)、~10 路由意义不大"是事实。
7. **主席裁决 = 1 段 + 1 置信度 + 1 具体下一步**。不要"权衡之下"开头的废话段。
8. **不无中生有找事改**：reviewing 候选 / 文件 / 配置时，先按"删了/改了能解决什么真问题"判断 — 解决不了真问题 → 不动，"无 actionable 改动"是合法输出。不为产出而产出。

### 反例（实测被用户标过废话的）

- "针对你这次重写的具体建议" 这种 framing 标题 → 直接说建议
- "三个观点指向同一个重构" 之后又重述一遍重构 → 已经说完了，删
- 解释 H1 时把 Werkzeug 怎么演化讲一遍 → 用户问 canonical 不是听历史

违反任一 → 输出无效，重写。

---

## 红队（事实校验 + Codex 异源对照）

执行 `prompts/codex_red_team.md`（包含两子任务，一道 prompt 跑完）。

### 子任务 A — 事实校验（Claude 自己跑，必跑）

抽出本次产出所有 🟢 + URL → 批量 WebFetch → 200 才放行；4xx / timeout / dead → 退档 🟡 或删，council-log.md 标 "fact-check failed: <url>"。

**为什么集中在红队做**：主流程每个 🟢 inline verify 太慢。事实校验是给来源兜底，不是 Claude 实时检索新信息，集中跑一次 forcing function 够了。

### 子任务 B — Codex 异源对照（开关控制，`red_team.codex == true`）

- Claude 标 🟢 的，codex 没听过 → 大概率 confabulate
- Claude 漏掉的明显 canonical 选项（codex 知道但 Claude 没列）
- 不同模型训练分布的盲区互补

**默认 true**，除非赌注极小（赌注小时事实校验仍跑，Codex 跳过）。

---

## Topic-relevant Surface

用户第一条消息后扫两个位置的 decision.md，对 status 是 open 的：
- grep 用户消息和当前文件路径是否命中 topics
- 命中 → surface 提示
- 不命中 → 静默

**严禁**进入项目自动列、超期强提醒。

---

## 边界 — 这个 skill 不做什么

- 产品形态 / 商业模式 → `/office-hours`
- UI/UX 视觉 → `/design-*` skills
- 实现细节 / 单文件审查 → `/codex consult`
- PR diff 级 review → `/review` 或 `/codex review`
- 安全审计 → `/security-review` 或 `/cso`

如果用户的问题 ≥ 50% 落在以上其中之一，**主动建议分流**而不是硬接。

---

## 命令

- `/decisions list` — 列当前项目所有 decision
- `/decisions list global` — 列全局
- `/decisions show <id>` — 完整展示
- `/decisions promote <id>` — 项目 → 全局（生成 symlink）
- `/decisions review <id>` — 手动触发回看
