# agent-config-audit 评分标准（Checklist）

> 这是 `/agent-config-audit` skill 的评分数据源，覆盖 Claude Code 配置的 10 个维度。
> 每条前缀标签：
> - `[配置]` = 能对静态文件/settings 直接判定（✅通过 / ⚠️可改进 / ❌问题）。
> - `[实践]` = 偏使用习惯，无法从静态文件判定，作为自查建议呈现，不打客观分。
>
> 判定原则：只审**生效配置**（真实的 `.claude/` 内容、`.mcp.json`、`.claude-plugin/plugin.json`）；
> 教学/示例文件（README 里的样例、`*-examples.json`）只标注、不扣分；不存在的构件标「未使用」、不扣分。

---

## 01 slash-commands（自定义命令）

对应模块：01 slash-commands（自定义命令）
生效构件：`.claude/commands/*.md`

1. `[配置]` 自定义命令是否都位于 `.claude/commands/` 且扩展名为 `.md`？（放别处或非 .md → ❌ 不生效）
2. `[配置]` `.claude/commands/` 里是否混入了 `README.md` 或其他纯说明文档？（该目录下每个 `.md`（frontmatter 可选）都会变成一个命令，`README.md` 会生成一个无意义的 `/README` 命令、其正文被当作 prompt → ❌）
3. `[配置]` 每个命令文件顶部是否为合法 YAML frontmatter，且 key（`description`、`allowed-tools` 等）保持英文未中文化？（key 被翻或解析失败 → ❌）
4. `[配置]` `allowed-tools` 是否保留原文且遵循最小权限、带作用域（如 `Bash(git add:*)`）？裸 `Bash(*)` → ⚠️；被翻译/删除 → ❌。
5. `[配置]` 正文代码块里的命令名 / 可执行命令（如 `/optimize`、`git ...`）是否保持英文？（被翻译则用户复制即失败 → ❌）
6. `[配置]` 文件名（即触发名）是否 kebab-case，并与文档里宣称的 `/xxx` 一致？（不一致 → ⚠️）
7. `[实践]` 某个命令若跨项目复用、依赖脚本/模板、或希望被自动调用，是否更该升级为 skill？（重命令仍留在 commands/ → ⚠️，建议迁 03-skills）
8. `[实践]` 是否还在依赖已移除或过时的命令写法？三类各有不同修法：① `/pr-comments`（已移除 → 改为直接让 Claude 查看 PR 评论）；② `/vim`（已移除 → 编辑器模式改到 `/config` 里设置）；③ 把 `/think` 当 slash command（它从来不是命令 → 想提高思考强度就在 prompt 里写 `ultrathink`，或用 `/effort` 调档）。老截图/老博客里出现这些即 ⚠️，按对应替换处理。

---

## 02 memory / CLAUDE.md（记忆）

对应模块：02 memory / CLAUDE.md（记忆）
生效构件：根 `CLAUDE.md`、`.claude/CLAUDE.md`、`CLAUDE.local.md`、`~/.claude/CLAUDE.md`

1. `[配置]` CLAUDE.md 是否以具体可执行的规则为主（如提交前跑测试、默认工具、别动的文件），而非一段「项目是什么」的泛泛介绍？
2. `[配置]` 内容是否长度克制、聚焦长期稳定规则，没有塞入频繁变化的实时数据或每次不一定相关的大段背景？
3. `[配置]` 项目级规则（`./CLAUDE.md`）与个人偏好（`~/.claude/CLAUDE.md`）是否分开，没有混在一起？
4. `[配置]` 若用了 `CLAUDE.local.md` 存个人项目记忆，它是否已加入 `.gitignore`？
5. `[配置]` memory 里描述的目录结构/路径/规范是否与当前实际仓库一致（无过期规则、提到的路径真实存在）？
6. `[配置]` 是否把「流程模板/工作流细节」这类更该做成 skill 的内容错误写进了 CLAUDE.md？
7. `[配置]` 是否还在用已停用的 `# ...` inline memory 前缀写法？（应改用 `/memory` 或自然语言）
8. `[实践]` 若项目依赖特定工具（uv/npm/pnpm/bun）、特定 shell（Windows 路径）、镜像源/代理，这些环境前提是否写进了 memory？

---

## 03 skills（技能）

对应模块：03 skills（技能）
生效构件：`.claude/skills/*/SKILL.md`（及其 `templates/`、`scripts/`、`references/`）

1. `[配置]` 每个 skill 的 `description` 是否写清「什么时候调用它」（触发场景/触发词），而不只描述功能？
2. `[配置]` 最关键的使用场景是否放在 `description` 前半句？（description 预算收紧、会被裁短）
3. `[配置]` frontmatter key（`name`/`description`/`effort`/`shell`/`paths`/`allowed-tools`/`disallowed-tools`/`reloadSkills`/`${CLAUDE_EFFORT}` 等）与 skill 名本身是否保持原样、未中文化？（翻译会解析失败）
4. `[配置]` 每个 skill 是否单一职责，没写成「大杂烩」？
5. `[配置]` SKILL.md 是否结构化（步骤/清单/模板引用），而非一大段散文？
6. `[配置]` skill 依赖的脚本/模板/参考是否就地放在 skill 目录内，而非散落各处？
7. `[配置]` skill 名称是否避免遮蔽内置 bundled skills（如把自定义 skill 命名成 `code-review` 会遮蔽内置 `/code-review`）？
8. `[配置]` 若 skill 只应在特定路径触发，是否用了 frontmatter `paths` 字段避免误触发？（团队级加分项）

---

## 04 subagents（子代理）

对应模块：04 subagents（子代理）
生效构件：`.claude/agents/*.md`

1. `[配置]` 每个 subagent 的 `description` 是否具体到能让 Claude 判断「何时委派给它」，而非模糊空泛？
2. `[配置]` `tools` 字段是否遵循最小必要原则？（审查型 agent 不应给 `Write`；给太多失去隔离价值，给太少做不了事）
3. `[配置]` frontmatter key（`name`/`description`/`tools`/`model`/`effort`/`permissionMode`/`skills`/`mcpServers`）是否未中文化？
4. `[配置]` 每个 subagent 是否对应一个边界清晰、可独立带回结果的子任务，而非高度耦合、需反复共享细节的任务？
5. `[配置]` 是否存在把过小/过简单任务也拆成 subagent 的过度设计？（⚠️）
6. `[配置]` 文件是否为「YAML frontmatter + Markdown system prompt 正文」结构，正文写清了角色定位与执行流程？
7. `[配置]` 若 agent 通过 `claude --agent <name>` 作为主线程启动，`mcpServers`/`permissionMode`/`tools`/`disallowedTools` 是否被有意识地正确配置（此时真正生效）？
8. `[实践]` 若 agent 依赖 Git/Python/Node/数据库 CLI 或 shell，是否在正文里写清依赖与（Windows）兼容性？

---

## 05 mcp（外部集成）

对应模块：05 mcp（外部集成）
生效构件：`.mcp.json`、settings 里的 `mcpServers`

1. `[配置]` `mcpServers`/`command`/`args`/`env`/`alwaysLoad`、server 名（如 `github`）、环境变量名（如 `GITHUB_TOKEN`）是否全部保持原样、未翻译？（翻译直接导致 MCP 无法加载）
2. `[配置]` 密钥是否通过 `env`/环境变量注入（`${GITHUB_TOKEN}`），而非硬编码进 JSON？
3. `[配置]` `alwaysLoad: true` 是否仅用于极高频、几乎每次都用的工具？（常驻工具占上下文预算，装多了挤掉更相关的动态工具 → 滥用则 ⚠️）
4. `[实践]` 是否先接一个核心 server（GitHub/filesystem）跑通再扩展，而非一上来接很多服务？
5. `[实践]` 需认证的 server 是否确实完成登录（`claude mcp login <name>` 或 `/mcp` 检查），避免「配置有了、实际没登录」的静默失效？

---

## 06 hooks（钩子）

对应模块：06 hooks（钩子）
生效构件：`.claude/settings.json` / `settings.local.json` 里的 `hooks`（及被调用的脚本）

1. `[配置]` hook 脚本是否从 `stdin` 读 JSON 输入、用 `file_path`/`command`/`user_prompt` 等字段取值，而非「第一个位置参数即文件路径」的过时写法？
2. `[配置]` 协议字段/事件名（`hooks`/`matcher`/`type`/`command`/`timeout`/`PreToolUse` 等）与实际命令行片段是否保持原样、未中文化？
3. `[配置]` 逻辑是否绑定到语义正确的事件？（如「整场会话结束记录一次」应用 `SessionEnd` 而非每轮触发的 `Stop`）
4. `[配置]` `matcher` 是否精确匹配目标工具？合法形式：精确名 `"Write"`、正则多选 `"Edit|Write"`、逗号列表 `"Write,Edit"`（`v2.1.191+` 起才按「任一命中」生效，旧版会静默不触发）、全匹配 `"*"` 或 `""`、MCP 工具模式 `"mcp__memory__.*"`。若还需按工具参数收窄，应在**具体 handler 上**（与 `type`/`command` 同级，而非写在 `matcher` 上）加 `if`，用 permission-rule 语法 `ToolName(pattern)`，如 `Edit(src/**)`、`Read(.env)`、`Bash(git push *)`。
5. `[配置]` hook 是否「高频、确定、低风险、轻量」，没有变成过重过慢的复杂系统？
6. `[配置]` 简单 shell 检查是否遵循退出码契约（成功 0 / 失败非 0），需控制行为时返回被认可的 stdout JSON（`allow`/`deny`/`ask`/`updatedInput`/`additionalContext`）？
7. `[配置]` 若 hook 需与用户交互，`read` 是否显式从 `/dev/tty` 读取？（stdin 已被 JSON payload 占用，这是常见坑）
8. `[实践]` 若 hook 调用 python/node/uv/npm/pytest，是否确认了本机路径与（Windows Git Bash）兼容性？（「配置看起来对、运行却没效果」多源于此）

---

## 07 plugins（插件）

对应模块：07 plugins（插件）
生效构件：`.claude-plugin/plugin.json` 及打包目录

1. `[配置]` 是否存在 `.claude-plugin/plugin.json` 且路径正确？（缺失或放错目录 → ❌）
2. `[配置]` manifest 的 `name`/`version`/`description` 是否齐全、JSON 合法、`version` 用 semver？
3. `[配置]` `name`/`version`/`description`/`author`/`license` 这些 key 与 plugin `name` 本身是否保持英文未中文化？（`plugin.json` 的 `name` 与 marketplace entry name 即便不一致，enable/disable 也能正确处理（`v2.1.195+`），故重点查是否被中文化改名，而非强制两者字面相同）
4. `[配置]` `commands/`、`agents/`、`skills/`、`hooks/`、`.mcp.json`、`scripts/`、`templates/` 是否各归其位、没有散落在非约定目录？
5. `[配置]` 若 plugin 带外部集成（GitHub/K8s/webhook），README 是否写清外部依赖、必需 token/env、所需 CLI、平台（Windows/WSL）、权限？
6. `[实践]` 是否存在过早打包？（plugin 内实际只有单个能力、工作流尚未稳定 → ⚠️，README「先别急着做 plugin」）
7. `[配置]` 若面向团队分发，settings 是否关注 marketplace 治理（`strictKnownMarketplaces`/`blockedMarketplaces`/`hostPattern`/`pathPattern`）？
8. `[配置]` 若用了 `monitors`，每项是否有 `command` + 合法 `trigger`（`session_start`/`skill_invoke`）？

---

## 08 checkpoints / rewind（检查点）

对应模块：08 checkpoints / rewind（检查点）
生效构件：`.claude/settings.json` 里的 retention 设置（其余为使用习惯）

1. `[配置]` 是否还在配置已过时的 `autoCheckpoint`（`checkpoints.autoCheckpoint: true`）？（checkpoints 默认开启，无需此项 → ⚠️ 旧说法）
2. `[配置]` 若想控制历史保留周期，是否用 `cleanupPeriodDays`（现统一影响 checkpoints、`~/.claude/tasks/`、`shell-snapshots/`、`backups/`），而非找「开关」？
3. `[实践]` 做高风险改动（UI/API/auth/权限变更、大批量文档或本地化替换）前，是否确认有回退点（改坏就 `/rewind`）？
4. `[实践]` 是否理解 rewind 各选项的区别（`Restore code and conversation` / `Restore conversation` / `Restore code` / `Summarize from here` / `Never mind`），而不总是「代码+对话一起退」？
5. `[实践]` 上下文过长时，是否用 `Summarize from here` 压缩，而非硬扛？
6. `[实践]` 是否存在两种误解——①以为必须先手动「存档」才能回退（其实默认自动）；②以为 rewind/summarize 一定会改磁盘文件（`Summarize from here` 只压缩上下文，不动磁盘代码）？

---

## 09 advanced-features（高级设置）

对应模块：09 advanced-features（高级设置）
生效构件：`.claude/settings.json`、环境变量（注意：教程/文档里的示例配置多是概念示意 schema，非真实 settings）

1. `[配置]` permission mode 是否与风险匹配？（日常项目默认挂 `bypassPermissions`/`auto` → ❌；非交互脚本用 `dontAsk`、只读分析用 `plan` → ✅）
2. `[配置]` 若启用 Auto Mode，是否把破坏性命令写进 `autoMode.hard_deny` 黑名单（如 `rm -rf /`、`git push --force`）？（区别于软判定 `soft_deny`）
3. `[配置]` 含生产凭证/公司设备上跑自动化时，是否设了 `sandbox.credentials`（阻止读取 secret env/凭证文件）？
4. `[配置]` `fallbackModel`（最多 3 个）/ `worktree.baseRef`（默认 `"fresh"`）/ 受管 `disableRemoteControl` 等是否配置合理、放对层级（受管开关不该放个人本地 settings）？
5. `[配置]` 环境里是否堆了看不懂用途的实验变量（`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`、`DISABLE_AUTOUPDATER`+`FORCE_AUTOUPDATE_PLUGINS`、`ENABLE_PROMPT_CACHING_1H` 等应明确需要才开）？
6. `[配置]` 是否把教程/文档里用于概念示意的**虚构 key**（`permissions.mode: unrestricted/confirm`、`extendedThinking`、`planning.autoEnter`）当真实 settings 复制进了 `.claude/settings.json`？这些 key 在真实 Claude Code 里并不存在（真实权限枚举是 `default/acceptEdits/plan/dontAsk/bypassPermissions/auto`，不是 `unrestricted`/`confirm`；思考强度由 `/effort` 或 prompt 里的 `ultrathink` 控制，没有 `extendedThinking`；也没有 `planning.autoEnter`）→ 复制即 ❌。（注：`checkpoints.autoCheckpoint` **不属此列**——它是已过时的真实开关，归 08 第 1 条 ⚠️，勿与虚构 key 混为一谈）
7. `[实践]` 是否把全自动（Auto Mode/`--dangerously-skip-permissions`）当新手默认？（应只在可丢弃 sandbox/临时实验仓使用；复杂任务先 planning，再逐步放权）
8. `[实践]` planning mode 的输出是否含分阶段步骤、预计改动文件、风险点、验证方式、需用户确认处，而非空话？

---

## 10 cli（命令行）

对应模块：10 cli（命令行）
主要为使用习惯，少量涉及脚本/settings

1. `[实践]` 交互 vs print mode 是否选对？（连续对话用 `claude`；一次性任务/管道/CI 用 `claude -p`。把 `-p` 当聊天来回用 → ⚠️）
2. `[配置]` 自动化/CI 是否用 `--permission-mode dontAsk` 而非 `--dangerously-skip-permissions`？（脚本里 skip-permissions → ❌）
3. `[实践]` 无头/自动化任务是否用 `--tools` 缩小工具面（如 `--tools "Read,Grep,Glob"`）？
4. `[实践]` 要接 `jq`/Python/CI 时是否用 `--output-format json`（必要时 `--json-schema`）？（下游解析 JSON 却用默认文本 → ⚠️）
5. `[实践]` 长期多任务是否给 session 命名（`-n/--name`、`/rename`、`claude -r "name"`）？
6. `[配置]` 文档/脚本里的 CLI flags/子命令（`claude -p`、`--model`、`--permission-mode`、`claude mcp`、输出格式名 `json`）是否保持英文未中文化？（被翻译则复制即失败 → ❌）
7. `[配置]` 涉及 CLI 行为的 settings key（`respondToBashCommands`、`language`、`wheelScrollAccelerationEnabled`、`footerLinksRegexes` 等）是否保持英文、值类型正确？
8. `[实践]` 是否还在传播「`--continue`/`--resume` 恢复时会悄悄降级 permission-mode」的旧说法？（`v2.1.132+` 起已真正生效）
