# REVIEW — 远程 agent 回看 prompt

## 调用方式

由 `/schedule` 在指定日期 cron-fire 一个新会话。新会话**没有任何对话上下文**，只有：

- 调用参数：`<project-root>` `<decision-id>`
- 文件系统访问

所以这个 prompt 必须是 self-contained。

## Step 1：定位决策对象

```bash
DECISION_DIR="<project-root>/.claude/decisions/<decision-id>"
```

如果不存在，检查全局：

```bash
DECISION_DIR="$HOME/.claude/decisions/<decision-id>"
```

仍不存在 → 这个 routine 已失效（用户可能删了决策目录），输出错误日志，不创建任何文件，结束。

## Step 2：读决策对象（精简后 2 文件）

```
Read decision.md
Read council-log.md
Read outcome.md (如存在)
```

## Step 3：输出 review-pending.md

写入 `$DECISION_DIR/review-pending.md`：

```markdown
# 第 N 次回看 — 远程 agent 初步分析

**生成时间**: <ISO 时间戳>
**回看时点**: <预定的 review_dates 中哪一天>
**决策状态**: <当前 status>

## 假设状态盘点

[逐条列出 assumptions.md 中每个假设]
- A1 <内容>: 状态待你更新（agent 无法替你验证产品形态对齐这种事）
- A2 <内容>: agent 提议的检查方式 = ...
- ...

## Kill criteria 状态盘点

[逐条列出 kill-criteria.md 中每条]
- K1 <内容>: agent 无法直接判断，请回答 yes/no/未知
- ...

## Agent 的判断

如果有任何**可独立判断**的项（比如：你的代码库里 commits、你的 outcome.md 已经写了的内容、文件系统状态），agent 在这里给出判断。

如果完全依赖人类输入（绝大多数假设和 kill criteria 都是这样），明确说"agent 无法判断，等待用户回答"。

**严禁**幻觉判断：不要假设 benchmark 跑过了、不要假设合伙人已对齐。只判断你能从文件系统直接读出来的事实。

## 给用户的问题

[列出用户必须回答的具体问题，每条 yes/no 或具体数据]

回答方式：直接编辑此文件，回答完后告诉 Claude "审完了"，Claude 会把回答整合到 council-log.md 并更新 decision.md 状态。

## 状态机建议

基于以上盘点，agent 建议 decision.md status 改为：
- `reviewing` — 正在等用户填写本文件
- 或保持当前状态如果用户尚未回答
```

## Step 4：不要做的事

- 不要修改 decision.md / assumptions.md / kill-criteria.md（写权限交给用户主对话）
- 不要发起新一轮智囊团（没有对话上下文，新视角必然是泛泛建议）
- 不要 PushNotification 干扰用户（让 surface 在用户主动靠近话题时发生）

## Step 5：可选 — push notification

如果用户在 `~/.claude/council-config.yml` 里启用了 push（默认关闭），发一条简短通知：

```
📋 决策回看就绪：<title>。下次进入 oral-trainer 项目时会主动 surface。
```

否则静默落盘，等用户主动靠近话题。
