---
name: rsh-kickstart
description: 用 kickstart.claude 给当前项目建立以 OpenSpec 为主的开发流程，并引入配套工具：MCP（exa 联网搜索、context7 文档、searchcode 代码搜索）、caveman、CodeGraph、agent-browser。同时应用默认配置，把开发流程和使用方法写进 CLAUDE.md，全部为项目级配置并随仓库共享。当用户新建项目或刚接手/clone 一个项目，说"初始化项目""给项目装上我常用的工具/skills/MCP""配置 Claude Code 环境""kickstart""rsh-kickstart"时使用。
argument-hint: "[模块名...] 例如：exa codegraph；留空则走完整流程"
---

# rsh-kickstart

kickstart.claude 仓库位于 `F:/code/skills`。如果仓库搬了位置，修改这一行。

读取 `F:/code/skills/docs/installation.md`，并严格按其中的步骤执行。

- OpenSpec 是核心模块，始终安装，除非用户明确说不要
- 带参数（如 `/rsh-kickstart exa codegraph`）：Step 3 装这些模块，再加上 OpenSpec，不再询问。参数无法对应任何模块时，列出可用模块让用户选
- 不带参数：执行完整流程，Step 3 让用户选择，全部预先勾选
- 只盘点不安装（如"看看这个项目装了哪些"）：执行 Step 1、Step 2 后停止
