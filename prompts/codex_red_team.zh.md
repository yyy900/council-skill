# 红队 — 事实校验 + Codex 异源对照（两子任务一道 prompt）

## 为什么集中在这里

主流程每个 🟢 inline WebFetch verify 太慢。事实校验（给 🟢 URL 兜底）+ 异源对照（给主推荐兜底）都是**给 Claude 单模型自信度兜底**，集中跑一次足够，不混进主流程打断节奏。

---

## Section A — 事实校验（Claude 自己跑，必跑，不可关）

### Step A1：抽取本次所有 🟢 URL

grep council 当前在写的产出（Step 3 canonical 段 + Step 4 大师 framing 段 + Step 8 reading list），列出：

```
🟢 ARC: https://www.usenix.org/legacy/publications/library/proceedings/fast03/tech/full_papers/megiddo/megiddo.pdf
🟢 Caffeine: https://github.com/ben-manes/caffeine
🟢 Werkzeug Routing: https://werkzeug.palletsprojects.com/en/latest/routing/
...
```

### Step A2：批量 WebFetch verify

对每个 URL 调一次 WebFetch（同 session 同 URL 15 min cache 自动复用，不重复付）。

只看 status：
- **200** → 放行，保留 🟢
- **4xx / 5xx / timeout / dead** → 退档 🟡（或直接删此条），council-log.md 标 "fact-check failed: <url>"

不深读 content（除非你想，cost 上不划算）— 看 status 200 足以堵住"Claude 编 URL"这一类。

### Step A3：写入 council-log.md

```markdown
### ✅ 红队 子任务 A — 事实校验

**verify 日期**：YYYY-MM-DD
**verify 总数**：N 个 🟢 URL
**通过**：M 个
**退档**：K 个

| URL | status | 处理 |
|---|---|---|
| https://... | 200 | 🟢 保留 |
| https://... | 404 | 🟡 退档 + 移除来源 |
```

### Step A4：影响 Step 7 主席裁决

如果有 🟢 因 fact-check 退档：
- 主推荐的支撑 source 受影响 → 主席必须**显式说**：原推荐基于 [X canonical]，但 X 的 URL fact-check 未通过，因此推荐降级为 🟡 / 或推荐改方向
- 不能装作没看见

---

## Section B — Codex 异源对照（`red_team.codex == true` 才跑）

### 为什么这个子任务单独存在

council 默认是 Claude 一个人演多角色。所有视角共享同一个模型偏置 — "反智囊团"用左手反右手，找不到真盲区。

Codex 是 OpenAI 模型，**真 diversity 来源**。开启后它不读 council-log 的全部讨论（避免被诱导），只看问题陈述 + 主席裁决 + 关键假设，独立质疑。

### 触发条件

`decision.md` frontmatter：

```yaml
red_team:
  codex: true
```

为 `false` 或缺失则跳过 Section B（**Section A 仍跑**）。

### 何时不要开 Section B

- 决策赌注小（< 1 周时间投入）→ 浪费 cost
- 你已经强烈倾向某个方向 → codex 反驳无法改变你 → 浪费
- 假设全部待验证（benchmark 没跑、合伙人没聊）→ 现在质疑早，先去拿数据

### 何时该开 Section B

- 不可逆决策（一旦走了改回成本巨大）
- 你和合伙人内部已强共识 → 容易 group-think，需要外部视角
- 智囊团内部高度一致 → 可疑，需要异源质疑
- 涉及大资源投入（钱 / 时间 / 团队精力）

### Step B1：组装传给 codex 的最小输入

不要把整个 council-log 喂给它。只传：

```
问题陈述：<decision.md 的"问题陈述"段>
当前推荐：<council-log.md 的主席裁决推荐方向，1-2 句>
关键假设（前 3）：<assumptions 的前 3 个>
关键 kill criteria（前 3）：<kill-criteria 的前 3 条>
用户 identity / 背景：<1 行，比如 "AI agent 架构师，2 人团队"，用于让 codex 找 identity bias>
```

理由：少喂上下文 → codex 不被诱导 → 质疑更独立。

### Step B2：调 /codex consult

```
你是独立红队，不参与之前的讨论。我有一个决策推荐如下：

[Step B1 的输入]

任务：

1. 找出推荐里最危险的隐藏假设（不是已列出的，是没人问的那个）
2. 这个推荐方向最可能因为什么失败？给出 1-3 个具体场景
3. 如果你完全反对这个方向，你会建议什么？为什么？
4. 决策者的 identity / 背景是否在偏置这个推荐？怎么偏？
5. 12 个月后这个推荐最可能在什么条件下变成负债？
6. Claude 标 🟢 Canonical 的来源你听过吗？漏列了哪些明显 canonical？

要求：尖锐、具体、不要客套话、不要"两边都有道理"的平衡叙述。
你的工作是找漏洞，不是给安慰。
```

### Step B3：把 codex 输出追加到 council-log.md

```markdown
### 🔴 红队 子任务 B — Codex 异源对照（真 diversity 来源）

**调用模型**：codex (OpenAI) via /codex consult
**调用日期**：YYYY-MM-DD
**只看了**：问题陈述 + 主席裁决 + 前 3 假设 + 前 3 kill criteria（未读 council-log 完整讨论）

[codex 原文输出，六段对应六个问题]

---

#### Claude 对 codex 质疑的回应

[针对 codex 每条质疑，明确：接受 / 部分接受 / 反驳。带理由。]

#### 主席二次裁决（如有改动）

[如果 codex 质疑成立到需要修改推荐方向，写新的裁决。否则明确说"维持原裁决"+ 为什么。]
```

### Step B4：影响假设和 kill criteria

如果 codex 暴露了未列出的隐藏假设 → 加到 assumptions，标 "由 Codex 红队发现"
如果 codex 给出新的失败场景 → 加到 kill-criteria

---

## 严禁

- 严禁跳过 Section A — 事实校验是无条件必跑（开关只控 Section B）
- 严禁把 codex 输出当圣旨（它也是一个模型，也会错）
- 严禁不让 Claude 对 codex 质疑做回应（要有 Claude 的二次判断）
- 严禁默默修改主席裁决（要么显式说"维持"，要么显式说"改了，因为 X"）
- 严禁在 codex 关闭时**伪造** codex 输出（如果开关 false，跳过 Section B，不要让 Claude 自己演 codex）
- 严禁假装 fact-check 通过 — 写入 council-log.md 的 verify 表必须是 WebFetch 真跑过的结果，不能脑补

## 成本提示

- **Section A**：每个 URL 一次 WebFetch（同 session 同 URL 有 15 min cache 自动省）。典型 5-10 个 🟢 → 5-10 秒延迟，无 LLM cost
- **Section B**：每次 codex 调用 ~ $0.01-0.05（GPT 模型）+ 几秒延迟。建议在重大决策**第一次召开** + **重大假设变更后续会**时各开一次，不要每次续会都开
