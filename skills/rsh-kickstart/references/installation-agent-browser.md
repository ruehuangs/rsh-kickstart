# 安装 agent-browser

> [agent-browser](https://github.com/vercel-labs/agent-browser)：通过 CDP（Chrome DevTools 协议）让 Claude 控制浏览器，用来打开页面、点击、填表、截图、读取控制台和网络请求。比传统的无头浏览器方案更省 token。

## 前置条件

需要 `agent-browser` CLI 和它自带的 Chromium：

```bash
agent-browser --version || { npm install -g agent-browser && agent-browser install; }
```

`agent-browser install` 会下载 Chromium，只需执行一次。

## 安装

在项目根目录执行：

```bash
npx -y skills@latest add vercel-labs/agent-browser --skill agent-browser --agent claude-code -y --copy
```

| 参数 | 作用 |
|---|---|
| `--agent claude-code` | 写到 `.claude/skills/`。不加的话，会给所有已知 agent 各装一份 |
| `--copy` | 复制文件，而不是创建符号链接。默认行为是把文件放进 `.agents/skills/` 再链接过来，但符号链接提交到 git 后，在 Windows 和别人的电脑上经常失效 |
| `-y` | 跳过交互确认 |

## 默认配置

1. 在项目根目录写入 `agent-browser.json`：

   ```json
   {
     "$schema": "https://agent-browser.dev/schema.json",
     "headed": false,
     "screenshotDir": ".agent-browser/screenshots"
   }
   ```

   - `headed: false`：无头运行，不弹出浏览器窗口。想看到操作过程时，单次命令加 `--headed` 即可
   - `screenshotDir`：截图统一保存到这个目录。它已经在[主流程 Step 6](installation.md#step-6处理-gitignore) 里加入 `.gitignore`
   - 键名就是命令行参数的驼峰写法（`--screenshot-dir` → `screenshotDir`）。加上 `$schema` 后，编辑器里有自动补全
   - 生效的优先级：命令行参数 > `AGENT_BROWSER_*` 环境变量 > 项目的 `agent-browser.json` > 个人的 `~/.agent-browser/config.json`

   **不要**往这个文件里写代理地址、本机路径、账号凭证：它会被提交。这类配置放到个人的 `~/.agent-browser/config.json` 里。

2. **放行规则**（加法见[主流程 Step 4](installation.md#添加放行规则)）：

   ```
   Bash(agent-browser *)
   ```

## 写入了什么

需要提交：

- `.claude/skills/agent-browser/`
- `skills-lock.json`
- `agent-browser.json`
- `.claude/settings.json`：放行规则

不提交：`.agent-browser/`（截图等运行产物）

skill 只教 Claude 怎么用；真正干活的是本机的 `agent-browser` CLI。所以队友 clone 后也要完成上面的"前置条件"。

## 已存在时

- `.claude/skills/agent-browser/` 已存在：跳过。用户要求更新时，先用 `git diff` 确认目录里没有本地修改，再重新执行安装命令（`add` 会不加提示地覆盖整个目录）
- `agent-browser.json` 已存在：只补缺少的键，已有的值不改

> 不要用 `npx skills update`：它在 Windows 上退出时会触发 Node 断言错误（`Assertion failed ... async.c`），退出码不可靠。

## 使用方法（写进 CLAUDE.md）

```markdown
### agent-browser（浏览器自动化）
- OpenSpec 阶段：**apply、verify**。完成涉及界面的任务后，按 tasks.md 里写的验证步骤打开页面实际检查；`/opsx:verify` 时，逐条对照 specs 里的 Scenario 在浏览器里验证
- 检查内容：截图看效果，查控制台报错（`console`、`errors`）和失败的网络请求（`requests`）
- 操作页面前先拿快照（`snapshot`），用快照里的元素引用（如 `@e3`）点击、填写，不要猜 CSS 选择器
- 截图保存在 `.agent-browser/screenshots/`，引用截图时给出路径
- 默认无头运行；用户想看操作过程时加 `--headed`
- 用完执行 `agent-browser close` 关闭浏览器
```

## 卸载

```bash
npx -y skills@latest remove agent-browser -y   # Windows 上结束时的 Assertion failed 可忽略
```

然后删除 `agent-browser.json`、`.agent-browser/`，并从 `.claude/settings.json` 中删掉放行规则。
