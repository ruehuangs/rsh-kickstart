# 安装 caveman（插件）

> [caveman](https://github.com/juliusbrussee/caveman)：极简回复模式，在保留全部技术信息的前提下，把 Claude 的回复压缩约 65–75%，节省 token。代码、命令、报错原文不会被压缩。

## 为什么装插件，而不是只装 skill

只装 skill 时没有 hook，caveman 只能每个会话手动开启，项目里设的默认模式也不会生效。装成插件后，它自带的 SessionStart hook 会在每次会话开始时读取项目里的 `.caveman.json`，自动进入设定的模式，队友那边也一样生效。这一点已实测确认。

## 前置条件

本机有 Node.js。hook 用 node 运行；没有 node 时，hook 会静默跳过，不会报错。

## 安装

在项目根目录执行：

```bash
claude plugin marketplace add JuliusBrussee/caveman --scope project
claude plugin install caveman@caveman --scope project
```

第一条命令把 caveman 的市场声明进项目的 `extraKnownMarketplaces`。缺了它，队友的 Claude Code 不知道去哪里下载这个插件。首次拉取较慢，超时时间至少给 3 分钟。

## 默认配置

在项目根目录写入 `.caveman.json`：

```json
{ "defaultMode": "caveman" }
```

可选的值：

| 值 | 效果 |
|---|---|
| `caveman` | **默认**。每个会话自动开启标准压缩 |
| `ultracave` | 更激进的压缩 |
| `manual` | 默认关闭，用户说 `/caveman` 时才开启 |
| `off` | 完全不启用 |

生效的优先级：环境变量 `CAVEMAN_DEFAULT_MODE` > 项目的 `.caveman.json` > 个人配置。所以队友想在自己电脑上关掉，设置 `CAVEMAN_DEFAULT_MODE=off` 即可，不用改仓库。

`claude -p` 这类非交互会话会自动以 `manual` 启动，不会影响脚本和 CI 的输出解析。

## 写入了什么

全部需要提交：

- `.claude/settings.json`：`extraKnownMarketplaces.caveman`，以及 `enabledPlugins["caveman@caveman"]: true`
- `.caveman.json`：默认模式

## 已存在时

- 插件已启用：跳过安装
- 插件标记为 `false`：说明有人特意关掉了它，先问用户
- `.caveman.json` 已存在：保留原值，不要改
- 项目里已经用 `npx skills` 装过 `.claude/skills/caveman/`：保留它，告诉用户插件里已经包含同样的 skill，可以删掉这份重复的

## 使用方法（写进 CLAUDE.md）

```markdown
### caveman（压缩回复）
- 本项目默认开启 caveman 模式（见 `.caveman.json`）：回复要简短，去掉客套和铺垫，技术细节、代码、命令、报错原文保持完整
- 用户说 "stop caveman" 或 "normal mode" 时恢复正常回复；说 `/caveman` 或 "caveman mode" 时重新开启
- **OpenSpec 的产物（proposal、specs、design、tasks）一律用正常、完整的语言书写，不压缩**：它们是团队共享的规范，要经得起别人反复阅读
- 写代码注释、commit message、PR 描述、给他人看的文档时，也用正常语言
```

## 卸载

```bash
claude plugin uninstall caveman@caveman --scope project
```

然后删除 `.caveman.json`。
