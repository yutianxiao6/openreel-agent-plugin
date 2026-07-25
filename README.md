# OpenReel Agent Plugin

[English](./README.en.md) · 简体中文

让外部智能体直接连接并操作正在运行的 [OpenReel Studio](https://github.com/yutianxiao6/openreel-studio)：项目、节点、依赖线和资产库使用直接工具，复杂或低频画布能力通过 `search → describe → execute` 按需加载。

仓库包含通用的本地 stdio MCP 工具桥，以及供 Codex 完整安装的插件清单和操作 Skill。Codex 可以从 marketplace 一次安装；Claude Code、Cursor、VS Code/Copilot、Gemini CLI、Windsurf 等支持 stdio MCP 的客户端可以直接配置同一个工具桥。OpenReel 主程序和内置 Agent 保持独立运行。

## Codex 完整安装

先把本仓库添加为 Codex marketplace，再安装插件：

```bash
codex plugin marketplace add https://github.com/yutianxiao6/openreel-agent-plugin.git
codex plugin add openreel-studio@openreel-agent
```

安装或更新后，新建一个 Codex 会话即可加载最新版工具和 Skill。

## 其他智能体

其他客户端克隆本仓库后，把
`plugins/openreel-studio/scripts/openreel-mcp.mjs --stdio` 配置为本地 MCP
服务。它们会获得同一组 OpenReel 工具，但不会自动安装 Codex 的
`.codex-plugin` 清单和 Skill。配置示例与支持边界见
[插件文档](plugins/openreel-studio/README.md)。

## 使用

启动 OpenReel Studio，然后在智能体中明确要求连接，例如：

> 连接本机 OpenReel，列出项目，切换到“产品演示”，然后概览画布。

桌面版和本地源码版通过受验证的本机端口自动发现。Docker 或远程部署首次连接时设置地址及可选认证；验证成功后保存到当前用户的私有配置文件，后续新会话直接复用。项目可以在同一智能体会话中显式切换，新建项目会自动成为当前操作目标。完整调用流程、安全边界和项目选择规则见[插件文档](plugins/openreel-studio/README.md)。

## 仓库结构

```text
.agents/plugins/marketplace.json       Agent 插件 marketplace 清单
plugins/openreel-studio/               OpenReel Agent 插件本体
  .codex-plugin/plugin.json            Codex 安装清单
  .mcp.json                            MCP 服务入口
  scripts/openreel-mcp.mjs             OpenReel 控制桥
  skills/openreel-director/SKILL.md    Codex 操作规范
```

公开名称是 **OpenReel Agent Plugin**。`.codex-plugin` 是 Codex 要求的安装
格式，不代表底层 MCP 工具只支持 Codex。

## 本地验证

```bash
node --check plugins/openreel-studio/scripts/openreel-mcp.mjs
node --test plugins/openreel-studio/tests/*.test.mjs
node plugins/openreel-studio/scripts/openreel-mcp.mjs --check
```

## License

MIT
