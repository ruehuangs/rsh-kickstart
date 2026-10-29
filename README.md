# rsh-kickstart

一个注释详尽、按需取用的 [Claude Code](https://code.claude.com/docs) **项目级**配置起点：新建或接手项目时，一条指令把你日常用的 MCP 和 skills 引入项目，并随仓库共享给团队。

> 灵感来源：[kickstart.pi](https://github.com/orionpax1997/kickstart.pi)，它又受 [kickstart.nvim](https://github.com/nvim-lua/kickstart.nvim) 启发。

---

## 目录

- [设计理念](#设计理念)
- [快速开始](#快速开始)
- [项目结构](#项目结构)
- [模块](#模块)
- [接手已有项目](#接手已有项目)
- [队友 clone 后要做什么](#队友-clone-后要做什么)
- [如何让它为你所用](#如何让它为你所用)

---

## 设计理念

装在 `~/.claude/` 下的东西只存在于你这台电脑。换台机器、队友 clone 仓库，全都没有。

**以 OpenSpec 为开发主流程**：每个项目都会装上 [OpenSpec](https://github.com/Fission-AI/OpenSpec)，按"探索 → 提案 → 确认 → 实现 → 验证 → 归档"推进开发。其他工具都挂在这条流程的各个阶段上：提案时用 CodeGraph 查影响范围，设计时用 context7 核对 API，验证时用 agent-browser 在浏览器里检查。

**rsh-kickstart 只做项目级安装**：

- 所有东西写进项目目录（`.mcp.json`、`.claude/`、`openspec/`、`CLAUDE.md`），提交到 git 后，团队共享同一套环境
- 不只是装上：顺带写好**默认配置**，并把**使用方法**写进 `CLAUDE.md`，Claude 装完就知道怎么用
- 每个模块一份短小的安装文档，写清楚**装了什么、写到哪、为什么这样装**
- 安装文档写给 Claude 读：让 Claude 照文档执行，而不是你手敲命令
- 接手已有项目时**只补不覆盖**：已存在的配置会先盘点、合并，冲突交给你决定
- 所有命令都在 Windows + Git Bash 下实测过，踩过的坑写在了各模块文档里

---

## 快速开始

**方式一：斜杠命令**（推荐）

先把 skill 装到全局，让任何项目里都能用 `/rsh-kickstart`：

```bash
npx skills add ruehuangs/rsh-kickstart -g -a claude-code -y
```

以后更新也是同一条命令。之后在任意项目里：

```
/rsh-kickstart                    # 完整流程：盘点现状 → 选择模块 → 安装和默认配置 → 把使用方法写进 CLAUDE.md
/rsh-kickstart exa codegraph      # 装指定模块（OpenSpec 始终会装）
```

也可以直接说"给这个项目装上我常用的工具"，Claude 会自动触发。

**方式二：粘贴指令**（无需任何准备）

在项目目录下启动 Claude Code，粘贴：

```
阅读安装指南并照做：https://raw.githubusercontent.com/ruehuangs/rsh-kickstart/main/skills/rsh-kickstart/references/installation.md
```

---

## 项目结构

```
rsh-kickstart/
├── README.md
└── skills/rsh-kickstart/                  ← npx skills add 安装的就是这个目录
    ├── SKILL.md                           ← /rsh-kickstart 入口，本身不会被装进项目
    └── references/
        ├── installation.md                ← 主流程：盘点 → 选模块 → 安装 → CLAUDE.md → 汇报
        ├── installation-mcp.md            ← exa / context7 / searchcode
        ├── installation-caveman.md        ← 压缩回复，省 token
        ├── installation-openspec.md       ← 核心：OpenSpec 开发主流程
        ├── installation-codegraph.md      ← 代码知识图谱
        └── installation-agent-browser.md  ← 浏览器自动化
```

---

## 模块

| 模块 | 作用 | 写进项目的文件 | 本机需要的 CLI |
|---|---|---|---|
| [exa](skills/rsh-kickstart/references/installation-mcp.md) | 联网搜索 | `.mcp.json` | — |
| [context7](skills/rsh-kickstart/references/installation-mcp.md) | 库和框架的最新文档 | `.mcp.json` | — |
| [searchcode](skills/rsh-kickstart/references/installation-mcp.md) | 公开仓库代码搜索 | `.mcp.json` | — |
| [caveman](skills/rsh-kickstart/references/installation-caveman.md) | 压缩回复约 65–75%，省 token | `.claude/settings.json`（插件）、`.caveman.json` | — |
| **[OpenSpec](skills/rsh-kickstart/references/installation-openspec.md)（核心，始终安装）** | 开发主流程：先写提案和规范，确认后实现，验证后归档 | `openspec/`、`.claude/skills/openspec-*`、`.claude/commands/opsx/` | `openspec` ≥ 1.15 |
| [CodeGraph](skills/rsh-kickstart/references/installation-codegraph.md) | 预索引的代码知识图谱 | `.mcp.json`、`.claude/settings.json`、`.claude/CLAUDE.md` | `codegraph` |
| [agent-browser](skills/rsh-kickstart/references/installation-agent-browser.md) | 通过 CDP 控制浏览器 | `.claude/skills/agent-browser`、`skills-lock.json`、`agent-browser.json` | `agent-browser` |

### 默认配置

安装时会顺带应用这些默认配置。想换成别的值，改对应模块文档里的"默认配置"一节：

| 模块 | 默认配置 | 配置写在 |
|---|---|---|
| caveman | 每个会话自动开启标准压缩（`defaultMode: caveman`） | `.caveman.json` |
| OpenSpec | 开启全部 12 个工作流（含 `verify`）；产物用中文书写；在规则里把其他工具挂到各个阶段（提案要列影响范围、任务要写验证方式、归档前要先 verify 等） | `openspec/config.yaml`、本机 OpenSpec 全局配置 |
| agent-browser | 无头运行；截图存到 `.agent-browser/screenshots/`（已加入 `.gitignore`） | `agent-browser.json` |
| 全部 | 预先放行：`mcp__exa__*`、`mcp__context7__*`、`mcp__searchcode__*`、`mcp__codegraph__*`、`Bash(openspec *)`、`Bash(codegraph *)`、`Bash(agent-browser *)` | `.claude/settings.json` |

### 使用方法

项目根目录的 `CLAUDE.md` 会写入两段：

1. **`## 开发流程：以 OpenSpec 为主`**，放在最前面：规定哪些改动必须走 OpenSpec、哪些可以直接改，以及完整的流程：

   ```
   /opsx:explore  →  /opsx:propose  →  用户确认  →  /opsx:apply  →  /opsx:verify  →  /opsx:archive
   （需求不清时）     （生成全部产物）   （确认前不写代码） （按 tasks 实现）  （核对规范）     （合并进规范）
   ```

2. **`## 工具使用方法`**：每个工具在 OpenSpec 的哪个阶段用、怎么用

具体内容见各模块文档里的"开发流程""使用方法"两节，可以按你的习惯修改。

---

## 接手已有项目

`/rsh-kickstart` 会先盘点再动手：

1. 项目已有 `skills-lock.json` → 只补装缺失的 skills，已存在的不动
2. 列出已有的 MCP、skills、`openspec/`、`.codegraph/`
3. 只补缺失的模块。例如已有 `openspec/` 时执行 `openspec update` 而不是重新 init；配置齐全但缺少 `.codegraph/` 时，只建索引
4. `.mcp.json`、`settings.json`、`CLAUDE.md` 一律合并或追加，不整份替换

---

## 队友 clone 后要做什么

MCP 和 skill 文件都随仓库走，但 CLI 要每个人在自己的电脑上装：

```bash
npm install -g @colbymchenry/codegraph      # 用了 CodeGraph 时必装，否则每次发消息 hook 都会报错
codegraph init                              # 建本机索引
npm install -g @fission-ai/openspec@latest  # 必装，版本 ≥ 1.15
openspec config set profile custom          # 开启全部 12 个工作流，否则执行 openspec update 时会删掉一半
openspec config set workflows '["propose","explore","new","continue","ff","apply","verify","update","sync","archive","bulk-archive","onboard"]'
npm install -g agent-browser && agent-browser install   # 用了 agent-browser 时
```

首次启动 Claude Code 时，会询问是否信任 `.mcp.json` 里的服务器，选择信任即可。

---

## 如何让它为你所用

1. **添加新模块**：复制一份现有的 `skills/rsh-kickstart/references/installation-*.md` 改写，再在 [`installation.md`](skills/rsh-kickstart/references/installation.md) Step 3 的表格和本 README 的模块表里登记。查找可用的 skill：`npx skills find <关键词>`，或浏览 [skills.sh](https://skills.sh)
2. **删掉用不到的模块**：删除对应文档，同时删掉两张表里的那一行

它是起点，不是框架。
