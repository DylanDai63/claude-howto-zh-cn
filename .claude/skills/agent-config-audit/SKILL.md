---
name: agent-config-audit
version: 1.0.0
description: 审查一个 Claude Code 项目的配置质量并给改进建议。Use when asked to "review my agent config", "check my claude code setup", "audit my .claude", "agent config audit", "帮我审查配置", or similar Chinese requests. 覆盖 slash commands、CLAUDE.md/memory、skills、subagents、MCP、hooks、plugins、checkpoints、高级设置、CLI 等 10 个维度的写法与实践质量。
---

# 项目配置审计（Agent Config Audit）

这是一个交互式审查 skill，用来给一个 Claude Code 项目的配置与使用质量做「体检」。它覆盖 10 个维度：slash commands、CLAUDE.md/memory、skills、subagents、MCP、hooks、plugins、checkpoints、高级设置、CLI 用法。

评分标准存放在 `references/checklist.md`，每条标注 `[配置]`（能对文件直接判定）或 `[实践]`（使用习惯，作自查建议）。

## 使用说明

### Step 1: 探测审查范围

用 **Glob / Read**（不要用 shell，保证 Windows 兼容）枚举目标项目里**真实生效**的配置构件：

- 根 `CLAUDE.md`、`.claude/CLAUDE.md`、`CLAUDE.local.md`（个人项目记忆）
- `.claude/commands/*.md`（自定义命令）
- `.claude/skills/*/SKILL.md`（skills）
- `.claude/agents/*.md`（subagents）
- `.claude/settings.json` 与 `.claude/settings.local.json`（`hooks`、permission mode、env 等）
- `.mcp.json` 或 settings 里的 `mcpServers`
- `.claude-plugin/plugin.json`（若这是个 plugin）

判定纪律（很重要）：

1. **只审生效配置**。教学/示例文件（README 里的样例、`*-examples.json`、教程目录下的演示文件）只标注「这是示意，不是生效配置」，**不扣分**。
2. **缺失的构件标「未使用」**，不扣分（例如项目本来就不用 MCP 属正常）。
3. 若无法确定这是用户自己的项目还是教程仓库本身，先说明你扫描到了什么，再继续。

---

### Step 2: 读取评分标准

读取本 skill 目录下的 `references/checklist.md`，拿到 10 个模块的检查点。按 Step 1 探测到的构件，只需评估相关模块。

---

### Step 3: 自动评估 [配置] 类检查点

对每个存在的构件，逐条比对 checklist 里的 `[配置]` 项，给出判定：

- ✅ 通过
- ⚠️ 可改进
- ❌ 问题

每条判定附一句话具体说明（指出是哪个文件的哪一点）。对「未使用」的模块，直接标注、跳过打分。

---

### Step 4: [实践] 类自查

checklist 里的 `[实践]` 项无法从静态文件客观判定，因此不要伪装成评分。改为：

1. 把与本项目相关的实践检查点，列成一份「实践自查清单」（每条前面标 `[实践]`）。
2. 可选：用 **AskUserQuestion** 做一轮快速自评（最多 1 轮、每题 3-4 选项），例如：
   - `你做高风险改动（改 auth / 批量替换）前，会先确认有 rewind 回退点吗？`
   - `写自动化脚本时，你用 --permission-mode dontAsk 还是 --dangerously-skip-permissions？`
   根据回答把对应实践项标为「已养成 / 建议加强」。
3. 若用户不想互动，就直接把实践清单作为建议输出，让用户自己对照。

---

### Step 5: 输出体检报告

按下面的模板输出：

```markdown
## Claude Code 项目体检报告

### 扫描到的构件

[列出发现的生效构件；标注哪些「未使用」、哪些是「教学示例（不计分）」]

### 配置类评分

#### 01 slash-commands ｜ [已评估 / 未使用]

| 检查点 | 判定 | 说明 |
|--------|------|------|
| ... | ✅/⚠️/❌ | ... |

（对每个「已评估」的模块给一张小结表；未使用的模块只写一行「未使用」）

### 实践自查清单

- [实践] ... — [已养成 / 建议加强]
- ...

### 总体评级

**[健康 / 良好 / 需改进]** — [一句话总结，指出最突出的问题或亮点]

### Top 优先修复项（按影响排序）

1. **[问题标题]** — [具体怎么改] → 参考教程 03-skills 一节
2. ...
（给 3-5 条；每条附具体改法 + 指向对应模块，用「参考教程 <模块名> 一节」这种可移植写法，不要写相对路径链接——审查对象可能不是本教程仓库）
```

---

## 输出要求

- 全程使用中文
- 保留关键英文术语，例如 skills、MCP、CLI、hooks、subagents、frontmatter
- **缺失的构件与教学示例文件不扣分**——只标注，不算问题
- `[实践]` 项标清楚、不伪装成客观评分
- 每条建议必须可执行，并指向对应模块（写「参考教程 04-subagents 一节」这种可移植写法，**不要用相对路径链接**——审查对象通常不是本教程仓库，相对链接会断）
- 不空泛鼓励，直接给出可改的点
- 报告聚焦「最该先改的几件事」，不要把所有检查点都平铺成流水账
