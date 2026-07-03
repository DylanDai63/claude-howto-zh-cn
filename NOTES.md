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
## 04-subagents 子代理
## 05-mcp 外部集成
## 06-hooks 钩子
## 07-plugins 插件
## 08-checkpoints 检查点
## 09-advanced-features 高级特性
## 10-cli 命令行
