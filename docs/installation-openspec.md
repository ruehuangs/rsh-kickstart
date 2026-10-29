# 安装 OpenSpec（核心模块）

> [OpenSpec](https://github.com/Fission-AI/OpenSpec)：spec 驱动开发。先写变更提案和规范，人确认后再让 Claude 实现，验证后归档。
>
> **kickstart.claude 以 OpenSpec 为开发主流程**：这个模块始终安装，其他工具都挂在 OpenSpec 的各个阶段上配合使用。

## 前置条件

### 1. openspec CLI ≥ 1.15

初始化要用它，装好的 skill 运行时也会调用它。下文的 `--language` 参数、`instructions` 等命令都需要 1.15 及以上版本：

```bash
openspec --version
npm install -g @fission-ai/openspec@latest   # 未安装，或版本低于 1.15 时执行
```

### 2. 全局工作流设为全部 12 个

OpenSpec 按**全局** profile 生成项目里的 skill 和命令。默认只开 6 个，缺少 `verify`（归档前核对实现）、`ff`、`new`、`continue`、`bulk-archive`、`onboard`。

更要命的是：在项目里执行 `openspec update` 时，**全局没开的工作流会被删除**（已实测）。所以本机必须开全：

```bash
openspec config list   # 查看 workflows 一项
```

缺少任何一个时，告诉用户"要把本机 OpenSpec 全局工作流设为全部 12 个"，然后执行：

```bash
openspec config set profile custom
openspec config set workflows '["propose","explore","new","continue","ff","apply","verify","update","sync","archive","bulk-archive","onboard"]'
```

这只是多开几个工作流，不会影响其他项目里已有的文件。想恢复时执行 `openspec config profile core`。

## 安装

在项目根目录执行：

```bash
openspec init --tools claude --language zh-CN --no-animation
```

| 参数 | 作用 |
|---|---|
| `--tools claude` | 只为 Claude Code 生成文件。不加会进入交互式选择 |
| `--language zh-CN` | 让生成的提案、规范用中文书写（结构性标题和 SHALL/MUST 关键词仍保留英文） |
| `--no-animation` | 不播放欢迎动画，适合非交互执行 |

执行完后，检查 `.claude/commands/opsx/` 下是否有 12 个文件。如果不够，说明全局工作流没设好，回到前置条件第 2 步。

## 默认配置

### 1. `openspec/config.yaml`

**用下面的内容覆盖** `init` 刚生成的 `config.yaml`（仅限本次新建的情况）。OpenSpec 会把 `context` 和 `rules` 注入到每个产物的生成指令里，把 `operations` 注入到 apply、archive 的指导里（已实测）。这样，其他工具就成了 OpenSpec 流程的一部分：

```yaml
schema: spec-driven

context: |
  Language: zh-CN
  All artifacts must be written in zh-CN.
  Keep OpenSpec structural headings and SHALL/MUST keywords in English.
  本项目以 OpenSpec 为开发主流程，openspec/specs/ 是系统行为的唯一依据。
  产物要写得完整清晰，不使用 caveman 等压缩写法。
  可用工具：CodeGraph（代码图谱）、context7（库文档）、exa（联网搜索）、searchcode（公开代码搜索）、agent-browser（浏览器验证）。

rules:
  proposal:
    - 写明本次变更不做什么（范围之外）
    - 用 CodeGraph 查清受影响的模块和调用方，列在影响部分
  specs:
    - 每条需求至少包含一个可验证的 Scenario（WHEN/THEN）
  design:
    - 涉及第三方库或框架时，先用 context7 核对项目所用版本的 API
    - 有多个可选方案时，列出取舍和选择理由；需要参考业界做法时，用 exa 或 searchcode 查
  tasks:
    - 每个任务足够小，并写明验证方式（测试命令或检查步骤）
    - 涉及界面的任务，验证步骤要包含用 agent-browser 打开页面截图、检查控制台报错

operations:
  apply:
    guidance:
      - 修改函数或接口前，先用 CodeGraph 查调用方，确保改动覆盖全部调用方
      - 每完成一个任务，运行它的验证，并在 tasks.md 中勾选
      - 发现提案或设计不成立时，先停下来用 /opsx:update 更新产物，不要偏离规范硬写
  archive:
    guidance:
      - 归档前先运行 /opsx:verify，确认实现与规范、任务一致
```

**按本次实际安装的模块裁剪**：没装的工具，把提到它的那一行删掉，同时改掉 `可用工具：` 那一行，只列装了的。比如没装 caveman，就删掉"不使用 caveman 等压缩写法"这一行。注意：`context: |` 里的内容是纯文本，不能用 `#` 写注释。

### 2. 放行规则

加法见[主流程 Step 4](installation.md#添加放行规则)：

```
Bash(openspec *)
```

openspec 的 skill 会频繁调用 `openspec list`、`openspec status`、`openspec instructions` 等命令，放行后不用一次次确认。

## 写入了什么

全部需要提交：

- `openspec/`：规范（`specs/`）、进行中的变更（`changes/`）、配置（`config.yaml`）
- `.claude/skills/openspec-*/`：12 个 skill
- `.claude/commands/opsx/`：12 个斜杠命令
- `.claude/settings.json`：放行规则

## 已存在时

项目里已有 `openspec/` 目录：

- **不要重新 init**，改为在完成前置条件第 2 步之后执行 `openspec update`，刷新指令文件
- 如果 `.claude/skills/` 下没有 `openspec-*`（原作者用的是别的 AI 工具），执行 `openspec init --tools claude --no-animation`。它只补 Claude Code 需要的文件，不动已有的 `openspec/` 内容
- **已有的 `config.yaml` 不要覆盖**：原作者的 `context` 和 `rules` 是项目知识。把上面模板里它缺少的 `rules`、`operations` 条目展示给用户，由用户决定要不要补

## 开发流程（写进 CLAUDE.md）

这一段放在 `CLAUDE.md` 的**最前面**，作为本项目的开发主流程（见[主流程 Step 5](installation.md#step-5把开发流程和使用方法写进项目-claudemd)）：

```markdown
## 开发流程：以 OpenSpec 为主

本项目的开发以 OpenSpec 为主线。`openspec/specs/` 是系统行为的唯一依据，代码必须与规范一致。

### 开始任何开发任务前
先执行 `openspec list`，看有没有进行中的变更。和当前任务相关的，接着那个变更做，不要另起一个。

### 必须走 OpenSpec 的
- 新功能、新能力
- 改变已有行为：接口、数据结构、交互、配置项
- 破坏性变更、架构调整、跨多个模块的改动
- 拿不准时，走 OpenSpec

### 可以直接改的
- 修 bug（恢复规范里已经写明的行为）
- 错别字、格式、注释
- 不改变行为的依赖升级，为已有行为补测试

直接改完后，如果发现规范里没有写到这个行为，提议发起一个变更，把它补进规范。

### 标准流程
1. **探索**（需求不清时）：`/opsx:explore`，只讨论和调研，不写代码
2. **提案**：`/opsx:propose "描述"`，一次生成 proposal、specs、design、tasks。想逐个产物推进，用 `/opsx:new` 加 `/opsx:continue`；需求已经很明确，用 `/opsx:ff` 快速生成全部产物
3. **确认**：把提案交给用户确认。**用户确认前，不写实现代码**
4. **实现**：`/opsx:apply`，按 tasks.md 逐项完成并勾选。实现中发现提案不成立时，停下来用 `/opsx:update` 修改产物，再继续
5. **验证**：`/opsx:verify`，核对实现是否与规范、任务一致
6. **归档**：`/opsx:archive`，把变更合并进 `openspec/specs/`。多个变更一起归档，用 `/opsx:bulk-archive`

### 其他
- 不想归档、只想先把规范同步进主规范时，用 `/opsx:sync`
- 刚接手项目、不熟悉这套流程时，可以用 `/opsx:onboard` 带着走一遍
- 提案、规范等产物用中文书写，要完整清晰
```

## 卸载

删除 `.claude/skills/openspec-*`、`.claude/commands/opsx/`，从 `.claude/settings.json` 中删掉放行规则，再删除 `CLAUDE.md` 里的"开发流程"一节。`openspec/` 里是项目的规范文档，删除前先问用户。
