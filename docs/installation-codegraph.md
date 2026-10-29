# 安装 CodeGraph

> [CodeGraph](https://github.com/colbymchenry/codegraph)：把代码库预先索引成知识图谱。Claude 查一次就能拿到调用链、影响范围、相关符号和源码，而不用逐个文件 grep、读文件。

## 前置条件

需要 `codegraph` CLI：

```bash
codegraph --version || npm install -g @colbymchenry/codegraph
```

## 安装

在项目根目录依次执行，两步都不能少：

```bash
codegraph install -t claude -l local -y   # 把 MCP、权限、hook、说明写进项目
codegraph init                            # 建立本项目的索引
```

- `-t claude`：目标是 Claude Code
- `-l local`：写到项目里。**不能省略**：只加 `-y` 时，默认会写到全局 `~/.claude.json`
- 不执行 `init`，MCP 工具就没有索引可查。索引建好后会自动增量同步

## 默认配置

`codegraph install` 已经自动完成了大部分配置：

- 放行 `mcp__codegraph__*`
- 添加一个 `UserPromptSubmit` hook（`codegraph prompt-hook`），在每次提问时提示 Claude 优先使用图谱

再补一条放行规则（加法见[主流程 Step 4](installation.md#添加放行规则)），让 Claude 可以直接运行 `codegraph sync`、`codegraph status` 等命令：

```
Bash(codegraph *)
```

## 写入了什么

| 文件 | 内容 | 提交？ |
|---|---|---|
| `.mcp.json` | `codegraph` MCP 服务器（`codegraph serve --mcp`） | 是 |
| `.claude/settings.json` | 放行规则和 `UserPromptSubmit` hook | 是 |
| `.claude/CLAUDE.md` | CodeGraph 自带的工具说明，包在 `<!-- CODEGRAPH_START -->` 标记里 | 是 |
| `.codegraph/` | 索引数据库，每台机器自己生成 | 否（目录里自带 `.gitignore`） |

已有的 `.mcp.json`、`settings.json`、`.claude/CLAUDE.md` 都是**合并或追加**，不会被覆盖（已实测）。

## ⚠️ 队友也必须安装 codegraph

hook 和 MCP 都会调用本机的 `codegraph` 命令。队友如果没装，**每次发消息都会看到 hook 报错**。汇报时一定要提醒：队友 clone 后需要执行

```bash
npm install -g @colbymchenry/codegraph
codegraph init
```

## 已存在时

- `.mcp.json` 已有 `codegraph`，`.codegraph/` 也存在：跳过 install，只补放行规则
- 配置都在，但 `.codegraph/` 不存在（刚 clone 下来的项目就是这种情况）：只执行 `codegraph init`

## 使用方法（写进 CLAUDE.md）

`.claude/CLAUDE.md` 里已经有 CodeGraph 自带的工具说明。根目录的 `CLAUDE.md` 再写一段场景说明：

```markdown
### CodeGraph（代码知识图谱）
- OpenSpec 阶段：**explore、propose**。理解现有实现，查清受影响的模块和调用方，写进提案的影响部分
- OpenSpec 阶段：**apply**。修改函数或接口之前，先查调用方和被调用方，改动要覆盖所有调用方
- 理解代码、定位实现、追调用链时，先用 `codegraph_explore`，不要一上来就 grep 或逐个读文件
- 结果和预期明显不符（比如刚加的符号查不到）时，执行 `codegraph sync` 刷新索引后再查
```

## 卸载

删除 `.mcp.json` 里的 `codegraph` 条目，删除 `settings.json` 里对应的放行规则和 hook，删除 `.claude/CLAUDE.md` 中 `CODEGRAPH_START` 到 `CODEGRAPH_END` 之间的内容，最后执行 `codegraph uninit`（会删除 `.codegraph/`）。
