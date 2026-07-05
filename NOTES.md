# 我的 Claude Code 学习笔记

> 由 Claude 边学边自动维护；我随时翻看复习。

## 01-slash-commands 斜杠命令

### 自定义命令 = Markdown 文件，位置决定是否生效
问：01 教程里那些 optimize.md、pr.md 是什么？能直接用吗？
答：每个 .md 文件就是一个自定义 slash command（pr.md → /pr），但躺在教程目录里只是教材，必须复制到 .claude/commands/ 才会被 Claude Code 发现并生效。对比：仓库的 .claude/skills/ 里的 self-assessment 能直接用，就是因为它在 .claude/ 下。内置命令（/help、/model）是 Claude Code 自带的，与此不同。（2026-07-03）

### 亲手验证：自定义命令的生命周期
问：安装前后 /optimize 的行为有什么区别？
答：安装前输入 /optimize 报 Unknown command；把 .md 复制到 .claude/commands/ 后，再输 /optimize，它的 Markdown 正文就被展开为给 Claude 的指令并执行。命令本质 = 提示词模板文件。注意：cp *.md 会把 README.md 也装成无意义的 /README 命令，要删掉。（2026-07-03）

### /context 看懂上下文占用
问：/context 面板里各项是什么意思？
答：它显示当前会话上下文窗口的用量构成：System prompt（Claude Code 出厂指令）+ System tools（工具定义）+ Memory files（加载的 CLAUDE.md，底部会列出具体文件）+ Skills（可用 skills 的简介清单，调用时才加载全文）+ Messages（对话本身，通常是大头）。快满时用 /compact 压缩或 /clear 重开。（2026-07-03）

### 上下文窗口 ≠ 用量额度
问：/context 能告诉我会话还能用多久吗？
答：不完全能。/context 显示的是"工作记忆空间"，满了会自动压缩（auto-compact），会话可以继续，只是早期细节变模糊。真正决定"还能用多久"的是订阅用量限额，用 /usage 查看——两者是不同的东西。（2026-07-03）
## 02-memory 记忆

### repo 文件是跨设备记忆，会话是本机的
问：两台电脑（Ubuntu + Win）学同一个 repo，进度能否用 git 同步？
答：能。NOTES.md、CLAUDE.md（含学习进度）、.claude/skills/ 都随 git 同步；但对话历史、Claude 持久记忆、settings.local.json 是本机的不会同步。习惯：学前 pull、学后 commit+push。（2026-07-03）
## 03-skills 技能

### skill 是文件夹，放对层级就能用；通用的放个人级免得到处复制
问：skill 写好放 .claude 就能用吗？想攒一套文件夹，新项目粘过去用？
答：①skill 不是单个 md，而是一个**文件夹**，里面至少有 `SKILL.md`（带 frontmatter：name + description，description 决定何时被触发），放好后 Claude Code 自动发现，无需注册。②两个存放层级：项目级 `<项目>/.claude/skills/` 只在该项目生效、能随 git 跟团队走；个人级 `~/.claude/skills/`（`C:\Users\16009\.claude\skills\`）你所有项目自动可用。③想「写一套到处用」最省事是放**个人级**，放一次处处可用，不用每次复制粘贴；复制到每个项目也行但要维护多份。经验：只给自己用的通用 skill → 个人级；跟仓库/团队一起走的 → 项目级并提交 git。进阶：想把「一组 skill+命令+配置」打包分发是 plugins 的活（07-plugins）。（2026-07-05）

### 攒很多个人 skill 会拖慢新对话吗？——几乎不会，关键在 description
问：把很多 skill 放个人级 ~/.claude/skills/，会让新对话记忆混乱/变慢吗？
答：①先纠正——skill 不是记忆（记忆是 CLAUDE.md/memory），两者不干扰，不存在搞混记忆。②效率影响靠「渐进加载」机制：每开新会话只加载每个 skill 的名字+一句 description（=/context 里 Skills 栏），正文 SKILL.md 只在被判定相关时才临时读入。所以代价=N 个 skill 固定多占 N 行简介。几个到一二十个几乎无感；攒到几十上百才开始可观，且 description 含糊/重叠时会被误触发。③结论：真正影响效率的不是数量而是 description 写得准不准——写清「何时用/何时不用」就不会乱触发。放心攒，但要：description 写利落、定期清理不用的、`/context` 随时看占用。（2026-07-05）

### 去哪逛社区 skill、怎么装、注意什么
问：skill 有社区吗？想逛逛热门的。
答：有，主要靠 GitHub 的「awesome list」人工精选清单，Anthropic 暂无官方付费商店。推荐入口：hesreallyhim/awesome-claude-code（最被认可、门槛严）、ComposioHQ/awesome-claude-skills 与 travisvn/awesome-claude-skills（按领域分类找 skill）、rohitg00/awesome-claude-code-toolkit（打包工具箱）。安装官方推荐用**插件市场**：会话里 `/plugin marketplace add <user>/<repo>` → `/plugin` 勾选安装（这正是 07-plugins 的内容）。判断口碑看 GitHub 页面的 star 数+近期更新，别信聚合页夸大的安装量数字。⚠️安全：装第三方 skill = 让别人的指令/脚本在你机器上跑，装前扫一眼 SKILL.md 和附带脚本有无可疑命令；装完可用本仓库的 agent-review skill 体检。（2026-07-05）

### 亲手做了一个覆盖全教程的审查 skill：/agent-review
问：想要一个"常驻审查员"，我新开 agent 项目时能持续 review 我的配置写得怎么样，有现成模板吗？
答：不抄外部 boilerplate，而是**从本仓库 10 个模块提炼评分标准**做了一个 `/agent-review` skill（`.claude/skills/agent-review/`：`SKILL.md` 编排流程 + `references/checklist.md` 存评分标准 + `README.md` 介绍）。关键设计三点：①检查点分两类——`[配置]`类能对文件直接打分（✅/⚠️/❌），`[实践]`类是使用习惯只做自查建议、不伪装成客观评分；②**只审生效配置**，教学示例文件（如 config-examples.json）和缺失构件都不扣分，避免制造假问题；③每条建议回链对应 lesson，边审边复习。跑通验证时它真抓到一个 live 问题：`.claude/commands/README.md` 被注册成了幽灵 `/README` 命令。收获：一个"审查员"本质就是把最佳实践写成**可判定 checklist** + 编排读取和比对的流程。（2026-07-05）

### 把 skill 做成"所有项目常态化"：装个人级 + 同步坑
问：agent-review 是通用的吗？想让它以后开发任何新项目都能 review 我。
答：内容通用（读任意项目的 .claude/，无写死本仓库的东西），但要真"常态化"需两步：①**改可移植回链**——原来让它回链 `04-subagents/README.md` 这种相对路径，只在本教程仓库指得到，别的项目里是死链；改成"参考教程 <模块名> 一节"的纯文字写法。②**装到个人级** `~/.claude/skills/agent-review/`（`C:\Users\16009\.claude\skills\`），这样任何项目 `/agent-review` 都在（当前会话用 `/reload-skills` 立即刷出）。同步坑：个人级不随 git 走 → 现在有两份，**仓库那份是"主"（有 git 记录、跨机同步）、个人级那份是"跑"（实际生效）；改主→拷到跑**；换到 Ubuntu 机器要 pull 后再拷一次到 ~/.claude/skills/。（2026-07-05）

### 让 skill 审自己（dogfood）：跨项目用的 skill 措辞不能绑死母仓库
问：变成个人级后，让 agent-review 先审自己写得够不够好；既然要通用，本仓库的 lesson 链接失效要不要改？
答：做法叫 **dogfood**——用它自己的 03-skills 标准审它自己。派了个独立 Explore agent 挑刺，结论：skill 质量达标（03-skills 八条基本全过、命名不遮蔽内置），唯一质量瑕疵是 **description 触发词排太靠后**（03-skills 说关键场景要放前半句，否则预算裁剪先丢它）→ 把 `Use when…` 提前、10 个维度枚举挪句尾。真正要修的是一批"只对本仓库成立"的措辞：checklist 里 10 处 `对应 lesson：NN-xxx/README.md`（指向母仓库的相对路径，别的项目里读不通）→ 改纯主题名 `对应模块：NN topic`；3 处"本教程"→"10 个维度"；2 处写死 `config-examples.json`→"教程/文档里的示例配置"；README 里"回链仓库内 lesson"（还和 SKILL.md"别用相对链接"自相矛盾）→"指回模块名"。校验器（validate_localization）确认这些 inline-code 路径不被当链接校验，安全，唯一红线是**每个 Markdown 标题至少留一个中文字**。收获：①好 skill 要经得起自己的标准审；②要跨项目复用的 skill，任何绑死母仓库语境的文字都是隐性 bug。（2026-07-05）

### 删回引≠变通用：没了兜底，每条判据必须自己说准说全
问：变通用不是删掉本仓库措辞就完事——以前有回引可点进教程看全文，现在没回引了，每条检查点在对应位置都提炼准确了吗？
答：这个质疑对，且暴露了真问题。派 3 个 agent 把 10 个模块检查点逐条对教程原文核"准确性+自足性"，真查出几条本就不够准的（不是措辞、是内容 bug）：①09 把 `checkpoints.autoCheckpoint`（已过时的**真实**开关，应判 ⚠️）和 `permissions.mode:unrestricted`/`extendedThinking` 这类**虚构 key**（真实里根本不存在，判 ❌）并列，还和 08 第 1 条自相矛盾；②06 matcher 漏了逗号列表 `"Write,Edit"`（v2.1.191+ 才生效的版本坑）和 `""`；③07"plugin name 要和 marketplace entry 对应"已被 v2.1.195+ 放宽、会误报；④01 过时命令的补救办法张冠李戴（`/pr-comments`、`/vim` 各有不同修法）。教训：当一条引用被删、内容失去"点进去看全文"的兜底时，**该位置必须自己把判据说全说准**，否则简略+无回引=隐性错误。做参考类内容(checklist/文档)时，"自足性"和"准确性"是一对必须一起满足的质量线。（2026-07-05）
## 04-subagents 子代理
## 05-mcp 外部集成

### MCP 是什么
问：mcp 是啥？
答：MCP = Model Context Protocol（模型上下文协议），一个开放标准，像"USB 统一插头"一样让 Claude Code 用统一方式连接外部世界（数据库、GitHub、Google Drive、浏览器、内部 API 等）。三角色：Host/Client 是 Claude Code 自己（发起调用）、Server 是被连接的工具（提供能力）、传输走 stdio（本地进程）或 HTTP/SSE（远程）。价值：不用为每个工具重写对接，配好 server 后可直接让 Claude 拿真实数据而非瞎编。（2026-07-05）

### 能把 MCP "抄进"我的 subagent 里吗
问：MCP 能被 agent 内化吗？即抄它的代码放进我的 agent？
答：认知纠偏——Claude Code 的 subagent 不是能塞代码的程序容器，它只是一个 Markdown 文件（提示词 + 允许用的工具清单），本身不执行代码、只会调用工具，所以没有"把 MCP 代码粘进 agent 内部"这回事。MCP server 底层的业务逻辑（如调 GitHub API、跑 SQL）确实可以抄出来自用。想要那个能力有三条路：①保留 MCP server 让 agent 调用（即插即用、能跟上游更新）；②写成脚本用 Bash 跑（逻辑简单、只自己用）；③封装成 Skill（想可复用、自然语言触发）。要点：MCP 的价值恰恰是"不用抄"，自己抄反而丢了标准化与自动更新。（2026-07-05）
### MCP 配置文件放哪、怎么用
问：mcp 文件具体怎么用？在 .claude/ 里放个文件吗？
答：容易混淆点——MCP 配置**不放 .claude/**，而是放**项目根目录的 `.mcp.json`**（对比：命令放 .claude/commands/*.md，skill 放 .claude/skills/）。文件结构：顶层固定 key `mcpServers` → 里面每个 server 一个自起的名字（如 `github`）→ `command`+`args` 说明怎么启动它（如用 npx 跑官方包）→ `env` 传环境变量（如 GITHUB_TOKEN）。这些 JSON key 和 server 名不能翻译/改名，否则加载失败。两种装法：①命令行自动加 `claude mcp add --transport http|stdio <名> ...`（新手推荐）；②直接抄模板 `cp 05-mcp/github-mcp.json .mcp.json` 并先 export 好 token。装完在会话里输 `/mcp` 排错（看连上没、几个工具、要不要登录）。Windows 原生跑 npx server 常需 `cmd /c` 风格。（2026-07-05）

### skill 能"授予"MCP 吗？+ MCP 配置放哪档
问：mcp 相当于给 agent 一个接口、怎么用看任务对吧？我让一个 skill 允许它调用 mcp？放项目哪个地址合适？
答：①对——MCP 给 Claude 一组 tools，但**哪个任务调哪个工具是模型自己判断**，你只把连接配好，不硬编码任务→工具。②纠错：**MCP 访问权不是 skill 授予的**，server 一配好就是**会话级可用**、随时能调，不需要 skill"开门"。skill 与 MCP 只有两种关系：a) 正文里写指令**引导** Claude 用某个 MCP 工具（纯提示词）；b) frontmatter 的 `allowed-tools`/`disallowed-tools`**收窄**权限（不是授予），MCP 工具名写成 `mcp__<server>__<工具>` 如 `mcp__github__*`。真正"给某个 agent 挂指定 MCP server"的是 **subagent** 的 `mcpServers` 字段——skill 没这个字段。③放哪按"给谁用"分三档：项目根 `.mcp.json`=团队共享（可提交 git，最常用）；`claude mcp add` 默认 local scope=只你、只本项目；用户级=你所有项目。铁律：token 永远走环境变量 `${GITHUB_TOKEN}`，不写进会提交的文件。（2026-07-05）
## 06-hooks 钩子
## 07-plugins 插件
## 08-checkpoints 检查点
## 09-advanced-features 高级特性
## 10-cli 命令行
