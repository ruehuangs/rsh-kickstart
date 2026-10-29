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

只写装了的那几个。三者的分工：**context7 查文档，searchcode 看真实代码，exa 查网页上的最新信息**。同一个问题按这个顺序考虑：文档能回答的先查 context7；文档不够、要看别人怎么写，用 searchcode；与库无关、或者要最新动态，用 exa。

工具名已用 MCP 的 `tools/list` 核实过。

```markdown
### context7（库和框架文档）—— 涉及第三方库时首选
- **什么时候用**：写或改依赖某个库、框架的代码；确认 API 签名、参数、配置项；判断某个能力在当前版本是否存在；升级依赖前查版本差异
- **怎么用**：先 `resolve-library-id` 把库名解析成 ID，再用 `query-docs` 带上具体主题（如 "routing"、"middleware"）查询，不要笼统地查整个库
- **版本**：以项目的依赖文件（package.json、go.mod、pyproject.toml 等）为准，查对应版本的文档
- **OpenSpec 阶段**：design 核对能力是否存在、API 是否可用；apply 照着文档实现
- **不要**凭记忆写 API；不要用 WebSearch 去搜官方文档

### searchcode（公开仓库代码搜索）—— 需要参考真实代码时首选
- **什么时候用**：文档不够，想看某个库或 API 在真实项目里怎么用；研究某个开源依赖的内部实现、排查它的行为；比较几种开源实现
- **怎么用**：
  - 不熟悉的仓库，先 `code_analyze` 了解概况，或用 `code_file_tree` 看目录结构
  - `code_search` 搜代码（支持正则、布尔查询）；`code_find_usages` 找某个库在仓库里被哪些文件使用
  - 定位后用 `code_get_file` 读具体函数或文件；一次读多个文件用 `code_get_files`
- **OpenSpec 阶段**：explore、design，寻找参考实现，比较方案
- **不要** clone 整个仓库来看几个文件；不要用 WebFetch 抓 GitHub 网页

### exa（网页搜索）—— 需要最新信息时首选
- **什么时候用**：训练数据之后的信息，如最新版本号、发布说明、安全公告、新闻；报错信息的社区讨论；第三方服务、产品的现状和价格；context7 里没有收录的资料
- **怎么用**：先 `web_search_exa` 搜索，搜索词用英文效果更好；摘要不够时，用 `web_fetch_exa` 读取全文
- **OpenSpec 阶段**：explore、design，调研方案和业界做法
- 写进提案、设计的关键事实，要注明来源链接
- **不要**用内置的 WebSearch / WebFetch，除非 exa 不可用或调用失败
```

## 卸载

```bash
claude mcp remove exa --scope project
```

然后从 `.claude/settings.json` 中删掉对应的放行规则。
