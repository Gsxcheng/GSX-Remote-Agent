# GSX Remote Agent

> 让 ChatGPT、Claude、Cursor 等支持 MCP 的 AI，以可见、可控、可审计的方式操作你的 Windows 电脑或云服务器。

当前阶段：**架构设计 / 目标形态冻结中**。

## 产品方向

GSX Remote Agent 采用 **ZeroTier 风格的轻 Agent + Web Control Plane**：

- 每台设备只安装一个轻量 Agent；
- Agent 主动出站连接，不要求用户开放本机入站端口；
- 设备、连接、权限、日志、任务统一在 Web Console 管理；
- AI Host 通过标准 MCP 调用设备能力；
- 文件、Shell、Browser、Screenshot、Windows GUI 等能力在本机执行；
- 高风险动作受 Policy / Approval / Audit 约束；
- 用户可随时 Pause / Disconnect Agent；
- Transport 可替换，不锁定 OpenAI 或任一 Hosted Relay。

一句话目标：

> **像 ZeroTier 一样安装和管理设备，像 MCP 一样把设备能力提供给 AI。**

## 目标架构

```text
ChatGPT / Claude / Cursor / Codex
                │
               MCP
                ▼
┌──────────────────────────────────┐
│       GSX Control Plane          │
│  Device / Policy / Audit / MCP   │
│          + Web Console           │
└────────────────┬─────────────────┘
                 │ Secure outbound channel
        ┌────────┴────────┐
        ▼                 ▼
┌───────────────┐  ┌───────────────┐
│ Windows Agent │  │ Linux Agent   │
│ Files         │  │ Files         │
│ Shell         │  │ Shell         │
│ Browser       │  │ Process       │
│ Windows GUI   │  │ Docker / Git  │
└───────────────┘  └───────────────┘
```

本机 Agent 不做复杂 Dashboard，只保留：

- Connected / Offline 状态；
- Device ID / Agent Version；
- Browser Bridge / Service 状态；
- Open Web Console；
- Pause / Reconnect / Exit；
- 基础诊断和自动更新。

## 第一阶段目标闭环

```text
Windows 安装 Agent
→ Web Console 显示 Device Online
→ 配置 Capability Policy
→ ChatGPT 通过 MCP 调 device.health
→ Control Plane 路由到 Agent
→ Agent 本地执行
→ 结果返回 ChatGPT
→ Activity / Audit 可查看全过程
```

第一个里程碑不是“窗口画出来”，而是上面这条链路完整跑通。

## MVP 能力顺序

1. Device / Connection
2. MCP Gateway
3. Files / Shell / Process
4. Browser Bridge（Playwright / CDP + Managed Profile）
5. Policy / Approval / Audit
6. Screenshot
7. Windows GUI
8. Installer / Auto Update / Recovery

暂不作为 MVP 必做：

- 自建完整 Hosted Relay；
- 高带宽远程桌面视频流；
- 多租户 SaaS；
- 自研模型或 Agent Planner；
- 复杂桌面端控制台。

## 设计文档

### 目标导向

- [产品方向](docs/product/PRODUCT_DIRECTION.md)
- [目标 UI](docs/product/TARGET_UI.md)
- [关键执行流程](docs/product/KEY_FLOW.md)
- [实现方向](docs/product/IMPLEMENTATION_DIRECTION.md)

### 架构

- [系统架构总览](docs/architecture/SYSTEM_OVERVIEW.md)
- [架构设计](docs/ARCHITECTURE.md)
- [MVP 路线图](docs/ROADMAP.md)

### ADR

- [ADR-001：Web Console First + Lightweight Agent](docs/adr/ADR-001-web-console-first.md)

## Target References

目标参考图存放在：

```text
assets/target-ui/
```

其中包括：

- Web Console / Control Plane 产品形态参考；
- 极简本机 Agent 客户端参考。

这些图片用于约束产品方向，不作为像素级最终 UI 规范。

## Prior Art / Upstream

当前重点研究与复用：

- ZeroTier — 轻 Agent + Central Web 管理产品形态参考；
- OpenAI `tunnel-client` — Secure MCP Tunnel，可选远程传输适配器；
- `wonderwhy-er/DesktopCommanderMCP` — Files / Shell / Process / 长任务能力设计参考；
- Remote Desktop Commander — 设备配对、在线状态、远程使用体验参考，Hosted Service 不作为开源核心依赖；
- Model Context Protocol official SDK；
- Tauri 2 — 轻量 Windows Agent Shell / Tray；
- Playwright — Browser Capability。

## License

尚未冻结。正式引入第三方代码前单独完成依赖与许可证审查。
