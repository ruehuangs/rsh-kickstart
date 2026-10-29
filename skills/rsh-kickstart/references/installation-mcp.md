# 安装 MCP：context7 / searchcode / exa

> 三个远程 MCP 服务器，补齐 Claude 的外部信息来源：
>
> - **[exa](https://exa.ai)**：联网搜索（实时信息、新闻、事实）
> - **[context7](https://context7.com)**：查询库和框架的最新文档，避免依赖过时的训练数据
> - **[searchcode](https://searchcode.com)**：搜索和分析公开 git 仓库里的代码

三个都是 HTTP 服务，不需要在本机装任何东西，不配 API key 也能用（有频率限制）。

## 安装

在项目根目录执行，只装用户选中的那几个：

```bash
claude mcp add --scope project --transport http exa https://mcp.exa.ai/mcp
claude mcp add --scope project --transport http context7 https://mcp.context7.com/mcp
claude mcp add --scope project --transport http searchcode https://api.searchcode.com/v1/mcp
```

`--scope project` 写入项目根目录的 `.mcp.json`。默认的 `local` 级别只对你自己生效，也不会进仓库。

## 默认配置

**放行规则**：只加装了的那几个，加法见[主流程 Step 4](installation.md#添加放行规则)。

```
mcp__exa__*
mcp__context7__*
mcp__searchcode__*
```

这三个服务都只读取外部信息，不会改动本地文件，所以放行是安全的。放行后，Claude 调用它们时不再弹确认框。

## 写入了什么

全部需要提交：

- `.mcp.json`：

  ```json
  {
    "mcpServers": {
      "exa":        { "type": "http", "url": "https://mcp.exa.ai/mcp" },
      "context7":   { "type": "http", "url": "https://mcp.context7.com/mcp" },
      "searchcode": { "type": "http", "url": "https://api.searchcode.com/v1/mcp" }
    }
  }
  ```

- `.claude/settings.json`：上面的放行规则

队友第一次在项目里启动 Claude Code 时，会被询问是否信任 `.mcp.json` 里的服务器。

## API Key

**绝不能把 key 明文写进 `.mcp.json`**，因为这个文件会被提交。确实需要 key 时，用 `${环境变量}` 引用，让每个人在自己的电脑上设置。例如 context7：

```json
"context7": {
  "type": "http",
  "url": "https://mcp.context7.com/mcp",
  "headers": { "CONTEXT7_API_KEY": "${CONTEXT7_API_KEY}" }
}
```

## 已存在时

`.mcp.json` 里已有同名服务器：跳过 `claude mcp add`，但仍然要补上放行规则。`claude mcp add` 只会追加新条目，其他服务器原样保留。

## 使用方法（写进 CLAUDE.md）

只写装了的那几个：

```markdown
### exa（联网搜索）
- OpenSpec 阶段：**explore、design**。调研方案、了解业界做法、确认第三方服务的现状
- 需要训练数据之后的信息时用：最新版本号、发布说明、新闻、报错信息的社区讨论
- 搜索词用英文效果更好；写进提案或设计的关键事实，要注明来源链接

### context7（库和框架文档）
- OpenSpec 阶段：**design、apply**。设计时核对所用版本有没有这个能力；实现时照着文档写，不要凭记忆写 API
- 先 resolve 出库的 ID，再带上具体主题（如 "routing"、"hooks"）查文档，不要整库拉取
- 版本以项目的依赖文件（package.json、go.mod 等）为准

### searchcode（公开仓库代码搜索）
- OpenSpec 阶段：**explore、design**。想看某个库、某个 API 在真实项目里怎么用时，搜索公开仓库，找参考实现
- 分析某个开源依赖的源码行为时，用它直接读远程仓库，不必 clone
```

## 卸载

```bash
claude mcp remove exa --scope project
```

然后从 `.claude/settings.json` 中删掉对应的放行规则。
