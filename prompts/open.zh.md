# OPEN — 新决策

Mission：替用户**检索领域共识** + **援引真实来源**。负责任 = 具体出处 + 三档置信度（见 SKILL.md）。

每个 Step 受 SKILL.md 三条 meta 规则约束（字字千钧 / 引用 freshness / Step 2 default-confirm 不假问）。**Step 9.5 self-check 强制 enforce**。

---

## Step 0 — 读个人 playbook（含用户决策模式）

```
Read ~/.claude/projects/<slug>/memory/code_playbook.md
```

playbook 合二为一：

- **Section A 用户决策模式**：必读 — 帮你识别 user 的盲区 / 偏好视角 / 不接受的建议类型
- **Section B 决策技术 playbook**：检查**之前是否处理过同 problem class** — 如有，引用之前 pattern + outcome 作为 prior

如果 playbook 不存在：跳过，本次结束后必须创建（Step 11）。

## Step 0.5 — 模式分发（inline 是默认）

默认 council 走 **inline 模式**：1 句 canonical 援引 + 1 句陷阱 + 继续。Heavy 模式（Step 1-11）需显式触发。

3 个判断走哪个模式：

1. **外部 blast**：决策影响他人 / 公开承诺 / 用户已感知吗？
2. **生产数据**：决策接触已经在 production 的数据吗？
3. **累积锁定**：≥3 个未来决策会建立在它之上吗？

任一"是" → heavy 模式（继续 Step 1-11）。全部"否" → inline 模式（直接走下面 inline 输出，不进 Step 1-11）。

用户显式说"正式召开 council" / "formal council" / "open council" → 不管别的，走 heavy 模式。

Step 0 playbook 显示同 problem class 上次 outcome 失败 → heavy 模式（值得重新走流程）。

### Inline 模式输出

```
[inline canonical check]
Choice: <1 句>
Canonical: <pattern + 🟢/🟡/🔴 + URL（如适用）>
Trap: <1 句>
Going with: <选哪个 + 1 句理由>
```

3-5 行。不创建 decision 对象。不写 playbook。只出现在主对话。

### v1 → v2 变更

- 删除"回退 < 1 天"赌注测试 — vibe coding 时代代码回退时间不再有意义（AI 几小时重写）
- 删除"是不是简单 bug"过滤 — 即使简单 bug 也受益于 inline canonical 查询
- 从"early-exit gate"改名为"模式分发" — inline 是默认模式，不是 abort

## Step 1 — 识别问题阶段

判断用户处于哪个阶段（明确告诉用户判断结果）：

- **A. 不知道有什么选项**："我想做 X，怎么搞？"
  → 主要任务：**展示 canonical 选项地图**（Step 3 是重头）
- **B. 在已知选项里选**："A vs B vs C 怎么选？"
  → 主要任务：**trade-off + 红队**（Step 5 / 6 是重头）
- **C. 已经在做但要 review**："我实现成 X，对吗？"
  → 主要任务：**对照 canonical + 找已知 trap**（Step 3 + Step 5 重头）

不同阶段后续 Step 权重不同。

## Step 2 — Problem class 抽象（default-confirm，不强制 STOP）

把用户的具体问题抽象到**问题域**。抽象错 = 后面全错。

例：
- "口语训练器要不要拆 ASR / 评估 / 对话" → "low-latency multimodal pipeline architecture under small-team constraint"
- "cache 总是 OOM" → "in-process cache eviction policy for read-heavy workload"
- "微服务还是 monolith" → "service decomposition under <N>-engineer team"
- "agent memory 怎么存" → "long-term episodic memory for autonomous agents"

### Default-confirm 模式

用 default-confirm 减少打断（不强制 STOP）：

> "我把这问题抽象成：**[一句话 problem class]**。
> 关键约束：[1-3 个，e.g. solo dev / 10 路由 / latency < 200ms]。
> 没异议直接走，**有异议立刻打断我说"不是，应该是 X"**。"

然后**立即进 Step 3**，不等用户确认。用户打断 → 在 Step 3 检索任何内容**之前**回到 Step 2，吸收修正。

### 仍强制真 STOP 的 3 种情况

虽然 default 是 confirm，下列情况仍要求真停下：
1. 问题里有**多个独立子问题**，无法一个 problem class 概括 → 停下问"先解决哪个"
2. 抽象出来明显**和用户表述偏差大**（用户说要做 X 但抽象出来是 Y）→ 停下确认
3. Step 0 playbook 显示同 problem class 上次 outcome 是失败 → 停下确认"这次和上次本质问题是否一样"

其余 default-confirm 继续走。**不能假问** — 写 "对吗？" 又自顾自往下走 = 失职；要么用 default-confirm 明说"没异议直接走"，要么真停下等。

## Step 3 — Canonical Patterns 检索

对该 problem class，列出方案。每个方案三段式：

```markdown
### [pattern 名]

**置信度**：🟢 Canonical / 🟡 Common / 🔴 Claude 推理

**来源 URL**（🟢 必填可验证 URL，🟡 尽量填）：
- 例：`https://www.usenix.org/legacy/publications/library/proceedings/fast03/tech/full_papers/megiddo/megiddo.pdf` (ARC FAST 2003)
- 例：`https://github.com/ben-manes/caffeine`

**核心 idea**：一句话

**典型采用者**：[实际用了它的系统 / 公司]

**适用场景**：什么情况下是首选

**已知 trap**：什么时候反咬 / 常见误用
```

### URL 处理（本步不 verify）

- 🟢 必须配 URL，但 **verify 在 Step 6.5 红队 fact-check pass 批量做** — 本步不卡延迟
- 🟢 配不上 URL → 直接退档 🟡 或 🔴
- URL 找不到但记得 pattern 名 → 可先 WebSearch 该 pattern + 关键词，从结果里取真 URL（verify 仍由红队做）

**最低产出**：🟢 至少 1 个（带 URL，verify 由红队做）；🟡 1-3 个；🔴 视情况。

如果**找不到任何 🟢 verified / 🟡** → 明确告诉用户：

> "我对这个 problem class 不知道存在可验证的领域共识。下面全部是 🔴 第一性原理推理，建议单独调 /codex 拿 OpenAI 视角对照。"

诚实 > 假冒 canonical。

## Step 4 — 大师 Framing（top 2-3 pattern）

为最有希望的 pattern，找**最有助于理解的大师 framing**：

```markdown
### 理解 [pattern] 的最佳视角

**人物**：Hickey / Pike / Carmack / antirez / Linus / Knuth / Dijkstra / ...

**他的 framing**：[一段话，他怎么看这类问题]

**来源**：[具体的 talk / book / commit msg / 邮件列表 / blog]
- 例：`Hickey "Simple Made Easy" (Strange Loop 2011)`
- 例：`antirez blog: "The Bug, the Idea, the Cycle" (2013)`

**应用到当前问题**：他的 framing 怎么帮你看清当前选择
```

**人物只能选你能说出具体公开材料的**。模糊印象 → 跳过此 pattern 的 master framing 部分，**不强行加**。

## Step 5 — Trade-off 表

各 canonical pattern 横向对比：

| Pattern | 适用场景 | 不适用 | 实现复杂度 | 已知 trap | 演化负债 |
|---|---|---|---|---|---|

填**实数据**，不填 vague adjective（"很好" / "稍差" 不算）。

## Step 6 — 反智囊团（固定四问）

每问必须有具体回答（"无明显问题"也明确说）：

1. 我推荐的 canonical 是不是过时的（领域近 5 年有没有 newer consensus）？
2. 我有没有把"我熟悉的"误认为"canonical"？
3. 用户当前资源 / 团队规模 / 时间约束下，canonical 是不是反而不适用？
4. 用户场景是否足够特殊，所有 canonical 都不适用，应该逆主流？

## Step 6.5 — 红队（事实校验 + Codex 异源对照）

执行 `prompts/codex_red_team.md` 全流程（两子任务）。

### 子任务 A — 事实校验（默认强制跑，不可关）

- grep 本次 Step 3-5 全部 🟢 标记
- 提取每个 🟢 的 URL
- 批量 WebFetch verify（status 200 才放行）
- 4xx / timeout / dead → 退档 🟡 或删，council-log.md 标 "fact-check failed: <url>"

主流程不 inline verify 是为了流畅；事实校验集中在这步做 forcing function。

### 子任务 B — Codex 异源对照（开关控制）

读 `decision.md` frontmatter 的 `red_team.codex`：
- **未设 / false**：跳过（SKILL.md 默认 true，应少见）
- **true**：调 codex 攻击主推荐 / 漏列 / canonical claim（详见 codex_red_team.md Section B）

**严禁伪造**：开关为 false 时不要让 Claude 自己模拟 codex 输出。要么调真模型，要么不出现。

## Step 7 — 主席裁决（详细可执行计划，不是概括）

裁决不是"我推荐方案 A 因为它好"。是**给用户能直接照着走的详细计划**。

### 共通必须有（任一决策类型）

1. **置信度 emoji**：🟢 / 🟡 / 🔴。🔴 明说"基于第一性原理，不是领域 canonical"。
2. **不平均主义**：明确推荐 + 明确不推荐，各 1 句具体理由。"两边都有道理"是失职。
3. **失败信号**：observable 事实告诉你这计划错了。不是"如果性能不好"这种废话。

### 决策类型识别 → 选模板

- **实现决策**（写代码 / 重构 / pipeline / handler / state machine）→ **模板 A**
- **选型决策**（数据库 / 队列 / 缓存 / 工具链 / 框架 / 架构 pattern 二选一）→ **模板 B**

#### 模板 A — 实现决策

加入：
- **文件级**：哪几个文件、各 N 行预算（现 X 行 → 目标 Y 行）
- **签名级**：关键函数 / 类型 / handler 签名（伪代码或 type hint）
- **数据结构级**：核心 dict / list / dataclass 长什么样
- **关键代码骨架**：5-15 行核心 dispatch / loop / 装饰器（不是完整实现）
- **第一个 commit**：先动哪个文件、改哪几行、用什么 case 验证

示例（HTTP 路由重构）：

> 🟡 推荐：HTTP 走 `dict[(method, path), Callable]` 路由表 + handler 签名 `(request: dict) → (status: int, body: dict)` + 中间件 `for fn in middlewares: request = fn(request)`。
>
> **文件**：`http_router.py` < 180 行（现 ~280）；`ws_router.py` < 200 行（现 ~280）。
> **核心数据结构**：
> ```python
> ROUTES: dict[tuple[str, str], Handler] = {
>     ("POST", "/api/echo"): handle_echo,
> }
> MIDDLEWARES: list[Callable[[dict], dict]] = [auth, parse_json]
> ```
> **dispatch 骨架**：
> ```python
> def dispatch(scope, body):
>     req = {"path": scope["path"], "method": scope["method"], "body": body}
>     for mw in MIDDLEWARES: req = mw(req)
>     handler = ROUTES.get((req["method"], req["path"]))
>     if not handler: return 404, {}
>     return handler(req)
> ```
> **第一个 commit**：先把 `http_router.py` 重写成上面骨架，跑现有 6 个 endpoint e2e 测试通过 → commit。
> **失败信号**：`/agent/{id}` 路径参数让 `dict` 查表失效 → 退回到正则或 trie。
> **不推荐**：完整 sans-IO — 10 路由不值；onion middleware 框架 — 3 个 cross-cutting 直接 for 循环更清楚。

#### 模板 B — 选型决策

**不强行写代码骨架**。加入：

- **Why this not that**：对每个备选 1-3 句直接说为什么不选 — 引证据 / 数据 / 约束 / canonical pattern
- **First validation commit**：第一个能验证"这选择对"的具体动作（observable，1-2 周内）
- **Migration path**（如果 migrate）：从现状 → 目标的最小步骤序列
- **Lock-in 评估**：选 Y 后回退到 X 的工作量

示例（PostgreSQL vs SQLite）：

> 🟢 选 SQLite。
> **Why not Postgres**：当前 < 100 req/s 写入、单机部署、无分布式需求 — Postgres 的 MVCC / replication / 连接池全用不上，cost without benefit。
> **First validation commit**：现有 schema 跑 SQLite + load test 100 req/s × 24h，看 WAL fsync p99。
> **Lock-in**：迁回 Postgres ≈ 1 周（schema 大部分兼容，改 connection string + 少数 SQLite-specific 用法）。
> **Fail signal**：load test p99 > 200ms 或 db lock contention > 5% 请求 → 立刻迁 Postgres。
> **不推荐 DuckDB**：分析型而非 OLTP，写入特征不匹配。

### 语言风格（共通）

- **大白话**：术语第一次出现时一句话讲清。"sans-IO" → "handler 不直接读 socket，输入输出走参数返回值"。
- **保守式推销**：事实而不是夸张。
  - ✅ "现 561 行 → 目标 < 400 行；新概念只有路由表 + handler dict 两个。"
  - ❌ "革命性精简" / "可能会更好维护"
- **不画饼**：不说"未来扩展性好"、"可以无缝接入 X" — 猜测不是事实。

### 反例（被用户标过的废话）

> "推荐 H1 + H2 lite + H3 lite，因为它们在简洁性和功能性之间取得平衡。下一步：开始实现。"

任何决策类型都不能这么写。失职。

## Step 8 — Reading List

至少 3 条**用户真该读的**材料：
- canonical pattern 的原始论文 / 书章节
- 1-2 个实战 post-mortem
- 一段大师 talk（如适用）

**不要列 hallucinated URL**。如果你不确定 URL，写：

> 搜：`["Caffeine cache design" by Ben Manes]`

让用户自己 google，比给假链接好。

## Step 9 — Learning Notes

为用户提取**通用化教训**（不局限于本次具体问题）：

- 你下次遇到同 problem class 时该首先想到什么 pattern
- 这次 council 暴露了你之前的什么认知盲区
- 这个领域的"必须知道但你之前不知道"清单

这部分**追加到 code_playbook.md**（不是覆盖）。这是 skill 真正的复利来源。

## Step 9.5 — 产出前 self-check（强制 forcing function）

Step 7-9 全部写完后、Step 10 写文件前，**逐条 self-grade** checklist。任一不通过 → 重写那部分，再 self-check，直到全过。

### 字字千钧（8 条）

- 每段问"删了决策会变吗"，无变化的都删了？
- 没有 framing 句（"以下分三块"、"先说结论"、"综上"、"针对你这次"）？
- 没有对称 padding（候选只 1 句话能说就 1 句，没凑废话）？
- 没有 hedging 套话（"取决于"、"两边都有道理"、"可能"）？
- 每个 canonical pattern 描述 ≤ 3 行？
- trade-off 单元格全是决策性事实（数字 / 是否 / 一句概括），没"性能较好"这种废话？
- 主席裁决 = 1 段 + 1 置信度 + 1 具体下一步，没"权衡之下"开头？
- 没无中生有找事改（review 类问题真有问题才动）？

### 反幻觉（6 条）

- 每个 🟢 都有 URL（verify 由 Step 6.5 红队 fact-check 做，本项只检查 URL 字段填了）？
- 大师 framing 都有具体出处 + URL/ISBN？
- 没说"X 一定会这么想"（X = 任何真实人物）？
- 没编 quote / 没编 URL？
- 模糊印象的人物 / pattern 都退档 🔴 了？
- 没把 🔴 包装成 🟢？

### 引用 freshness（2 条）

- 引用都是近 5 年 production 现役 或 真正 timeless 概念？
- 模糊老论文 / 说不出"今年还在 production 用"的都已退档或删？

### 流程（2 条）

- STOP 点是真停（Step 2 problem class 用户确认了 / default-confirm 时用户没异议）？
- 主席裁决不是平均主义（明确推荐 + 明确不推荐 + 各带具体理由）？

**self-check 不通过的具体案例**：

- 发现 Step 7 主席裁决开头是"权衡之下推荐 A 因为它好" → 重写为"🟡 推荐 [pattern] + [文件级 / 数据结构 / 骨架]"
- 发现 🟢 ARC paper 没填 URL → 现在补 URL；verify 由 Step 6.5 红队 fact-check 做
- 发现 trade-off 表有 "性能较好" 一格 → 改成 "FAST 2003 benchmark 显示 hit rate +X%" 或删掉那一列
- 发现 Step 8 reading list 列了一个记不清的老论文 URL → 删，或先 WebSearch 找真 URL

通过全部 → 进 Step 10。不通过 → 重写直到全过。

## Step 10 — 创建决策对象

询问用户：
1. 决策标题（默认从 problem class 生成 slug）
2. visibility：project（默认）or global
3. red_team.codex（建议默认 true，除非赌注极小 — 明确告诉用户成本约 $0.01-0.05）
4. 关键 topics（你提议一份，用户改）
5. review point：实践后回看（不是 7 天定时）— 询问用户预计什么时候能拿到实践数据

### 写入：2 个文件（agent-first 设计）

```
<project-or-global>/.claude/decisions/YYYY-MM-DD_<slug>/
  decision.md      ~50-80 行：frontmatter + 问题 + 推荐 + 不推荐 + kill criteria
  council-log.md   仅当有审计价值：红队 4 问实质 + codex distill + 主席路径
```

`outcome.md` **不预创**，实践后再写。
`learning_notes` **不写决策目录**，直接 append 到 `code_playbook.md`（Step 11）。

### decision.md 模板（严格遵守）

```markdown
---
id: YYYY-MM-DD_<slug>
title: <人类可读>
status: open
problem_class: <一句话>
topics: [关键词]
red_team:
  codex: true | false
---

## 问题

- 场景：<1-2 行>
- 关键约束：<3-5 个 bullet，每条一行>

## 推荐

🟢/🟡/🔴 [pattern 名]

**文件**：<具体 path + LOC 预算>
**核心数据结构**：
\`\`\`
<5-10 行类型/数据结构>
\`\`\`
**dispatch / 入口骨架**：
\`\`\`
<5-15 行核心代码骨架>
\`\`\`
**第一步**：<具体到 commit 级>
**失败信号**：<observable，不是"如果不好">

（可选）权威 reference：
- <1-2 个验证过的 URL>

## 不推荐

- <pattern X>：<1 行理由 — 含 codex 红队下来的 nuance，如 "Claude 误标 canonical，实际不是 ASGI 共识">
- <pattern Y>：<1 行>

## Kill criteria

- <observable 条件 1>
- <observable 条件 2>
```

### council-log.md 模板（仅当有内容时写）

如果红队全 OK + codex 同意 + 一次定稿 → council-log 5 行 stub：

```markdown
problem class（用户确认）：<>
红队 4 问：无新发现
codex 红队：同意，sources verified
裁决：一次定稿
```

如果有实质内容（红队找出问题 / codex 改了裁决 / 多轮迭代）→ 详细写：

```markdown
## problem class（用户确认）

<>

## 反智囊团 4 问

只列**有实质发现**的问。无发现的问跳过，不写"无问题"。

## codex 红队

- **disagreed**：<codex 反驳 Claude 的 canonical claim>
- **added**：<Claude 漏掉的 canonical 选项>
- **verified**：<sources 已验证存在的 URL>

（不要 verbatim 全部 codex 输出 — distill。）

## 主席路径

仅当 codex 后修订过裁决，写早期裁决 + 当前裁决 + 修订理由。
否则跳过这段。
```

输出 `/schedule` 命令让用户启用 cron 回看（**不替用户跑** — 远程 agent 是 cost 行为）。

## Step 11 — 更新 code_playbook（两段都可能要写）

**playbook 不存在则创建**（含 Section A + Section B 框架，见 SKILL.md "学习累积"）。

### Section B 必追加

```markdown
## YYYY-MM-DD — [problem class]

**决策 ID**：YYYY-MM-DD_<slug>
**主席推荐**：[pattern] (置信度 🟢/🟡/🔴)
**关键 trade-off**：[一句话核心]
**Outcome**：pending（实践后回填）
**下次同问题域应首先想到**：[pattern 名]
```

### Section A 视情况更新

本次召开发现用户的**新偏好 / 决策模式 / 反复盲区** → 更新 Section A：

- 新决策加到决策列表
- 新有效视角（如 steelman pattern）→ "对此用户特别有效的视角"
- 用户明确反对的方向 → "此用户从不接受的建议类型"
- 决策模式速记 → 加日期 + 1 行备注

什么都顺利、没新东西 → 不写。

---

## 严禁

所有"严禁"项已合入 Step 9.5 self-check checklist + SKILL.md 反幻觉硬规则。产出前对照 checklist 逐条 self-grade，违反任一重写。

唯一未在 self-check 覆盖：**越界给产品 / 商业 / UI 判断** — 主动建议分流到 office-hours / design-* / etc（见 SKILL.md "边界"段）。
