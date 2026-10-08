# GSX Remote Agent

> 让 ChatGPT、Claude、Cursor 等支持 MCP 的 AI，以可见、可控、可审计的方式操作你的 Windows 电脑或云服务器。

当前阶段：**架构设计 / MVP 基线冻结前**。

## 项目目标

GSX Remote Agent 不是另一个聊天客户端，也不是从零自建一套专有远程中继。

它定位为一个 **Local-first AI Computer Agent**：

- 提供桌面 GUI 与系统托盘，普通用户安装后即可使用；
- 通过标准 MCP 向 AI 暴露本机/服务器能力；
- 文件、Shell、浏览器、截图、Windows GUI 等能力插件化；
- 所有高风险操作经过统一权限策略与审计；
- 传输层可替换：本地 MCP、OpenAI Secure MCP Tunnel、未来自建 Relay 均为适配器，而非核心依赖；
- 优先复用成熟开源项目与官方协议，不重复造基础设施。

## 架构原则

```text
AI Host (ChatGPT / Claude / Cursor / ...)
                │
                │ MCP
                ▼
        ┌──────────────────┐
        │   MCP Gateway    │
        └────────┬─────────┘
                 │ Local RPC
                 ▼
┌──────────────────────────────────────┐
│           GSX Agent Core             │
│                                      │
│  Capability Registry                 │
│  Permission / Policy Engine          │
│  Audit Log                           │
│  Session / Task Manager              │
└──────┬────────┬────────┬─────────────┘
       │        │        │
       ▼        ▼        ▼
   Files/Shell Browser  Windows GUI ...
               │
         Playwright/CDP

Desktop GUI: Tauri 2 + React
Transport adapters: Local / OpenAI Tunnel / Future Relay
```

## MVP 范围

第一版只验证一个闭环：

```text
ChatGPT / MCP Host
      → MCP Gateway
      → Permission Engine
      → Capability
      → Windows / Browser
      → Result + Audit Log
```

P0/P1 能力：

- 文件读取/写入
- PowerShell / CMD
- 进程与长任务管理
- 截图
- 可视化浏览器控制（Playwright/CDP，持久化浏览器 Profile）
- Git / Docker 可通过受控 Shell 使用
- GUI 中查看连接状态、能力权限、当前任务与日志
- 托盘暂停 / 恢复 AI 控制
- 高风险操作确认

暂不作为 MVP 必做：

- 自建 Hosted Relay
- 全量远程桌面视频流
- 多租户 SaaS
- 自研模型或 Agent Planner
- 完整跨平台 GUI 自动化

## 文档

- [架构设计](docs/ARCHITECTURE.md)
- [MVP 路线图](docs/ROADMAP.md)

## Prior Art / Upstream

当前重点研究与复用：

- OpenAI `tunnel-client` — Secure MCP Tunnel，作为可选远程传输适配器
- `wonderwhy-er/DesktopCommanderMCP` — 文件、Shell、进程、审计等能力设计参考
- Remote Desktop Commander — 设备配对、在线状态、远程使用体验参考；Hosted Service 不作为开源核心依赖
- Model Context Protocol official SDK
- Tauri 2 — Desktop GUI / Tray
- Playwright — 浏览器能力

## License

尚未冻结。正式引入第三方代码前单独完成依赖与许可证审查。
