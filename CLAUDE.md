# CLAUDE.md

这个文件给 Claude Code 提供“如何在这个仓库里协作”的工作说明。

## 项目定位

`claude-howto-zh-cn` 是一个 **documentation-as-code** 仓库。

- 主体产物是 Markdown 教程与示例，不是可执行应用
- 核心目标是把上游 `luongnv89/claude-howto` 做成更适合中国小白的中文主线指南
- 兼容性优先级很高：命令名、frontmatter、JSON/YAML key、环境变量、路径约定都不能被本地化改坏

## 你最常用的命令

### 本地化校验

```bash
uv run python scripts/validate_localization.py
```

检查重点包括：

- frontmatter 是否可解析
- 关键保留词是否被误翻
- Markdown 链接是否有效
- JSON / YAML / shell 脚本是否还能正常解析

### 测试脚本

```bash
uv run pytest scripts/tests/ -q
```

### EPUB 构建

```bash
uv run scripts/build_epub.py
```

### Python 质量检查

```bash
uv run ruff check scripts/
uv run ruff format scripts/
```

## 仓库结构

```text
01-slash-commands/      slash commands 教程与模板
02-memory/              CLAUDE.md / memory 教程与模板
03-skills/              skills 教程与示例
04-subagents/           subagents 教程与示例
05-mcp/                 MCP 教程与配置示例
06-hooks/               hooks 教程与脚本示例
07-plugins/             plugins 教程与完整样例
08-checkpoints/         checkpoints / rewind 教程
09-advanced-features/   高级能力说明
10-cli/                 CLI 指南
.claude/skills/         仓库内置的教学 skills
scripts/                校验、构建、测试脚本
```

## 修改文档时的核心原则

1. 先保兼容，再谈翻译  
   任何会被 Claude Code 直接读取或执行的标识，优先保留原样。

2. 先讲用途，再讲命令  
   中文读者更容易先理解“这是什么 / 什么时候用 / 为什么有价值”，再接受 CLI、配置和脚本细节。

3. 不要把正文写成翻译腔  
   优先自然中文表达；必要时保留 `skills`、`CLI`、`hooks`、`MCP`、`subagents` 这类英文术语。

4. 高风险文件少改、谨慎改  
   `.sh`、`.py`、`.json`、`.yml` 默认只同步必要的兼容性变化；注释能不动就不动。

## 修改示例文件时要特别注意

以下内容默认不要翻译：

- 文件名、目录名
- slash command 名称
- skill / subagent / plugin 名称
- YAML frontmatter key
- JSON / YAML key
- CLI flags
- 环境变量名
- MCP server 名
- 代码块里的可执行命令

## 推荐提交流程

1. 改文档或示例
2. 跑 `uv run python scripts/validate_localization.py`
3. 跑 `uv run pytest scripts/tests/ -q`
4. 如果改了 EPUB 构建，再跑 `uv run scripts/build_epub.py`
5. 在 `README.md`、`UPSTREAM.md`、`CHANGELOG.md` 里记录最近同步内容

## 提交信息风格

优先使用 conventional commits，例如：

- `docs(readme): sync upstream April 2026 updates`
- `feat(subagents): add performance-optimizer example`
- `fix(hooks): switch shell hooks to stdin JSON protocol`
- `refactor(epub): polish zh-cn cover and reading experience`

## 学习笔记助手模式

我正在用这个仓库学习 Claude Code。请在我们对话时充当我的笔记助手：

- 维护根目录下的 NOTES.md 作为我的个人学习笔记本。
- 边学边追加：每当①我的问题被解答、②我表现出困惑或混淆、③讲到一个重点概念时，就立刻把它以精简条目追加进 NOTES.md，不要等会话结束才写。
- 条目格式：`### 标题`，下面写一句问题/困惑 + 简洁的解答或澄清 + 日期。
- 只记重点，不要把闲聊和往返细节都记进去。
- 追加前可快速扫一眼 NOTES.md 避免重复，但不要每轮都全量读它。
- 每学完一个模块，就更新下面「学习进度」小节。

### 学习进度（每学完一块就更新这里）
- 当前阶段：Level 1 · 里程碑 1A
- 已完成：/self-assessment 自测（2026-07-03，Level 1 Beginner，1/8 项，起点定位与预期一致）
- 正在学：01-slash-commands（自读中），02-memory 排在其后
- 待解决的疑问：（暂无）
- 下一步：读完 01 后动手写一个自定义命令（如 /sync-progress），再回到 02-memory
