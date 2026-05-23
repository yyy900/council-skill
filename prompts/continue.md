# CONTINUE — 续会已有决策 prompt

## 触发条件

- 用户消息或当前文件路径命中某个 status 为 open / reviewing 的 decision.md 的 topics
- 或用户明确说"继续看 X 决策"、"再开一次智囊团"

## 严禁

- 严禁从零重开会议、忽略已有 council-log.md
- 严禁产生与已有结论矛盾但不说明的新建议
- 严禁默默修改 decision.md 状态

## Step 1：读决策对象（精简后只 2 文件）

```
Read decision.md
Read council-log.md
Read outcome.md (如存在)
Read review-pending.md (如存在 — 远程 agent 写的)
```

进入工作记忆。引用时**显式说明出处**（"council-log.md 第一次召开 codex 红队第 4 视角说过 ..."）。

### Outcome write-back forcing function（强制检查）

读完后立即对 outcome 状态做判断（用 Bash `stat` 看文件 mtime）：

| 状态 | 触发条件 | CONTINUE 主回复必须做 |
|---|---|---|
| ✅ outcome 新鲜 | outcome.md 存在且 mtime < 90 天 | 引用 outcome 数据 |
| ⏰ outcome 缺失（短） | status=done 且 outcome.md 不存在，decision.md mtime < 30 天 | 温和提醒："上次决策跑完了吗？要不要补一下 outcome？" |
| ⚠️ outcome 缺失（中） | status=done 且 outcome.md 不存在，decision.md mtime 30-90 天 | 明确提示："outcome 没回填，数据是不是已经出了但忘了写？" |
| 🚨 outcome 缺失（长） | status=done 且 outcome.md 不存在，decision.md mtime > 90 天 | CONTINUE 主回复**开头**加 `🚨 outcome 90+ 天未回填，决策状态可能已经过时` |
| ⏰ outcome 老化 | outcome.md 存在但 mtime > 180 天 | 提示"回看一次，验证 outcome 还成立" |

forcing function 的本质：把 outcome write-back 从"用户主动"变成"系统主动催"。code_playbook 复利的前提是 outcome 真的被回填。

## Step 2：判断本次召开类型

询问或自己判断：

- **追问已有结论**：用户对某个视角想细聊 → 不开新会，针对性回答
- **新信息需更新假设**：用户说"我跟合伙人聊了"、"benchmark 跑出来了" → 更新 assumptions.md 状态
- **触发 kill criteria**：检查用户描述是否命中任何 K1-K6 → 如命中，进入 reversed 状态机，重开智囊团
- **真实新视角加入**：用户说"我没想到 X" → 在 council-log.md 追加新一节，标注日期

## Step 3：更新文件，不重写

所有修改都是**追加**或**显式更新**：

- council-log.md：追加 `## YYYY-MM-DD 第 N 次召开` 段
- assumptions.md：更新假设状态（待验证 → 已验证 / 已推翻），加注 "更新于 YYYY-MM-DD"
- decision.md frontmatter：必要时更新 status

## Step 3.5：Codex 红队（条件触发）

如果 `decision.md` frontmatter `red_team.codex == true` **且** 满足以下任一：

- 这次续会触发了 kill criteria（K1-K6 任一）
- 关键假设有重大状态变化（A1-A3 任一从"待验证"变"已推翻"）
- 主席裁决方向被修改

→ 执行 `prompts/codex_red_team.md`，输出追加到 council-log.md。

不满足触发条件不要重跑 codex（避免无意义 cost）。

## Step 4：用户档案同步（写入 code_playbook Section A）

`user_decision_profile.md` 已合并入 `code_playbook.md` Section A。

如果本次续会发现用户的**新偏好 / 决策模式 / 反复盲区** → 追加到 `~/.claude/projects/<slug>/memory/code_playbook.md` 的 Section A（schema 见 SKILL.md "学习累积"）。

什么都顺利、没新东西 → 不写。

## Step 5：surface 给用户的初始消息格式

```
📋 命中 decision: <title>
   状态: <status>
   上次召开: <date>
   未验证假设: <count>
   是否相关？(y / 不相关 / 暂不看)
```

用户回"不相关"则记录到 decision.md frontmatter 的 `false_match_count`，连续 3 次以上则降低这个 decision 的 topics 权重（移除最易误触的关键词）。
