# 安装指南（主流程）

把 kickstart.claude 的模块引入**当前项目**。本文档写给 Claude Code 执行。

当用户明确要求引入时，直接执行文中命令，无需再次确认。但以下情况**必须先问用户**：覆盖已存在的文件、删除任何东西、模块安装失败需要换方案。

所有命令都在 bash 中运行（Windows 下 Claude Code 使用 Git Bash）。本文档中的相对链接，以本文档所在目录为基准解析。部分命令首次运行要下载东西，超时时间至少给 3 分钟。

## 前置条件

- Node.js（`npx`、`npm` 可用）
- Claude Code CLI（`claude` 可用）

各模块需要的 CLI（`openspec`、`codegraph`、`agent-browser`），在对应文档的"前置条件"一节里检查和安装。

## Step 1：确定项目根目录

```bash
git rev-parse --show-toplevel 2>/dev/null || pwd
```

后续所有路径都以项目根目录为准，所有命令也在根目录执行。如果当前目录不是 git 仓库，告诉用户：不提交到 git，就无法随仓库共享。然后继续执行。

## Step 2：盘点现状

接手已有项目时，这一步决定后面哪些要装、哪些跳过。

```bash
ls -1 .claude/skills 2>/dev/null           # 已有的项目级 skills
cat .mcp.json 2>/dev/null                  # 已有的 MCP 服务器
cat .claude/settings.json 2>/dev/null      # 权限、hook
ls -d openspec .codegraph skills-lock.json .caveman.json agent-browser.json CLAUDE.md .claude/CLAUDE.md 2>/dev/null
```

如果存在 `skills-lock.json`，但其中记录的 skill 在 `.claude/skills/` 下缺失，先恢复它们。下面这行命令按来源分组，列出锁文件里的 skill：

```bash
node -e "const s=require('./skills-lock.json').skills;const g={};for(const[k,v]of Object.entries(s))(g[v.source]=g[v.source]||[]).push(k);for(const[src,ns]of Object.entries(g))console.log(src+': '+ns.join(' '))"
```

对每个来源，只把**缺失的** skill 补装回来：

```bash
npx -y skills@latest add <来源> --agent claude-code --skill <缺失1> --skill <缺失2> -y --copy
```

> **不要用 `npx skills experimental_install`**：它会忽略 `--agent` 参数，始终把 skill 恢复到 `.agents/skills/`，而 Claude Code 读不到那里。
> 也**不要**把已存在的 skill 写进 `--skill`：`add` 会直接覆盖目录，原作者的本地修改会丢失。

用几行话向用户汇报盘点结果。

## Step 3：选择模块

| 模块 | 类型 | 安装文档 |
|---|---|---|
| exa | MCP：联网搜索 | [installation-mcp.md](installation-mcp.md) |
| context7 | MCP：库和框架文档 | [installation-mcp.md](installation-mcp.md) |
| searchcode | MCP：公开仓库代码搜索 | [installation-mcp.md](installation-mcp.md) |
| caveman | 插件：压缩回复，省 token | [installation-caveman.md](installation-caveman.md) |
| **openspec** | **核心：开发主流程，始终安装** | [installation-openspec.md](installation-openspec.md) |
| codegraph | 代码知识图谱 | [installation-codegraph.md](installation-codegraph.md) |
| agent-browser | skill：浏览器自动化 | [installation-agent-browser.md](installation-agent-browser.md) |

**openspec 是核心模块，始终安装**，不出现在选项里。本项目以 OpenSpec 为开发主流程，其他工具都挂在它的各个阶段上。只有用户明确说"不要 OpenSpec"时才跳过，并提醒用户：这样 CLAUDE.md 里就没有开发主流程了。

其余模块：

- 用户已指定模块（例如 `/rsh-kickstart exa codegraph`）：装这些，再加上 openspec，不再询问
- 用户说"全部"或"默认"：全部安装
- 用户没指定：用多选问题让用户选择，**全部预先勾选**。Step 2 已确认装好的模块不必再列。可以按项目类型给建议，例如纯后端项目不需要 agent-browser

## Step 4：逐个安装并应用默认配置

按顺序读取每个选中模块的安装文档，并照做。**openspec 放在最后装**：它的 `config.yaml` 要按实际装成功的模块来裁剪。每个模块分两步：

1. **安装**：按文档的"安装"一节执行
2. **默认配置**：按文档的"默认配置"一节，写配置文件、添加放行规则

每个文档都有"已存在时"一节，接手项目时以它为准。已有的配置文件只补缺少的键，不改动原作者设置过的值。

某个模块失败时，记录原因后继续装下一个，不要中断整个流程。

### 添加放行规则

各模块"默认配置"里列出的放行规则，统一用下面这条命令合并进 `.claude/settings.json` 的 `permissions.allow`。它会保留文件里的其他内容，已有的规则不会重复添加，文件不存在时会自动创建：

```bash
node -e "const fs=require('fs'),f='.claude/settings.json';fs.mkdirSync('.claude',{recursive:true});const s=fs.existsSync(f)?JSON.parse(fs.readFileSync(f,'utf8')):{};const p=s.permissions=s.permissions||{};const a=p.allow=p.allow||[];for(const r of process.argv.slice(1))if(!a.includes(r))a.push(r);fs.writeFileSync(f,JSON.stringify(s,null,2)+'\n')" "<规则1>" "<规则2>"
```

为了避免反复改文件，可以先汇总所有选中模块的规则，最后执行一次。如果某条规则已经出现在 `permissions.deny` 或 `permissions.ask` 里，说明有人特意限制过它：跳过这条，并告诉用户。

## Step 5：把开发流程和使用方法写进项目 CLAUDE.md

把下面两段写进项目根目录的 `CLAUDE.md`。第一段规定开发主流程，第二段说明各个工具在流程的哪个阶段用、怎么用：

```markdown
<installation-openspec.md 中"开发流程（写进 CLAUDE.md）"一节的内容>

## 工具使用方法

以下工具服务于上面的 OpenSpec 流程。每个阶段开始前，先判断是否有合适的工具，有就优先使用，不要退回到效果更差的默认做法。

<其余各模块文档"使用方法（写进 CLAUDE.md）"一节中的内容，每个模块一个 ### 小节>
```

规则：

- `CLAUDE.md` 不存在时：创建，并按上面的顺序写入
- `CLAUDE.md` 已存在时：不改动原有内容。"开发流程"一节插在第一个标题（通常是 `# 项目名`）及其简介之后、其余内容之前，因为它是这个项目的主规则；"工具使用方法"追加到末尾
- 已经有"开发流程"或"工具使用方法"段落时，只补充缺少的内容，不要重复整段
- 原有内容与 OpenSpec 流程冲突时（例如写了"直接改代码，不用写文档"），不要擅自改动，把冲突指给用户，由用户决定
- 只写本项目实际装好的模块

## Step 6：处理 .gitignore

下面这些是个人或运行时产生的文件，不应提交。把缺少的条目追加到 `.gitignore`（文件不存在时创建）：

```
.claude/settings.local.json
.agent-browser/
```

`.agent-browser/` 只在装了 agent-browser 时需要加。

**不要**忽略 `.claude/`、`.mcp.json`、`openspec/`、`skills-lock.json`、`.caveman.json`、`agent-browser.json`。它们要提交，团队才能共享。（`.codegraph/` 自带 `.gitignore`，不用处理。）

## Step 7：汇报

向用户汇报以下内容：

1. 本次装了哪些模块、跳过了哪些（以及原因）、哪些失败了。如果修改了本机的 OpenSpec 全局工作流设置，单独说明
2. 新增或修改了哪些文件（`git status --short` 的结果）
3. **需要重启 Claude Code 会话**，新的 MCP、skill 和命令才会生效。重启后，Claude Code 会询问是否信任 `.mcp.json` 里的服务器
4. 应用了哪些默认配置（配置文件、放行规则），以及想修改时改哪个文件
5. 提醒提交这些改动，例如 `git add .claude .mcp.json openspec skills-lock.json .caveman.json agent-browser.json .gitignore CLAUDE.md`（只列出实际存在的文件）
6. **队友 clone 后需要做什么**：把本次涉及的 CLI 安装命令列出来（`codegraph` 必须装，否则 hook 每次都会报错；用到 openspec、agent-browser 时，对应的 CLI 也要装），再加上 `codegraph init`，以及 OpenSpec 全局工作流的设置命令（见 installation-openspec.md 前置条件第 2 步），否则队友执行 `openspec update` 会删掉一半工作流
7. 下一步怎么开始：在新会话里执行 `/opsx:propose "第一个需求"`，或者需求还不清楚时先 `/opsx:explore`；不熟悉流程时用 `/opsx:onboard`
