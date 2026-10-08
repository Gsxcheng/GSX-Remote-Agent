# GSX Remote Agent MVP Roadmap

状态：Draft v0.2

## Phase 0 — Architecture Baseline

目标：冻结产品形态、系统边界和首个闭环，不急于写业务功能。

交付：

- [x] 项目定位
- [x] Web Console First + Lightweight Agent
- [x] Control Plane / Agent 职责划分
- [x] MCP / Device RPC 分层
- [x] Capability / Policy / Audit 基线
- [x] Browser 策略
- [x] 目标 UI 方向
- [x] 端到端流程图
- [ ] 许可证与第三方依赖清单
- [ ] MCP Tool Contract 定稿
- [ ] Device RPC Message Contract 定稿

退出条件：架构基线通过评审。

---

## Phase 1 — Lightweight Agent Skeleton

目标：Windows 上先有一个真正可长期运行的轻 Agent。

交付：

- Rust Agent Core
- Tauri 2 极简窗口
- System Tray
- Device Identity
- Connection Manager skeleton
- Local Policy Enforcer skeleton
- Capability Registry skeleton
- Task Manager skeleton
- Pause / Resume / Exit
- Auto Start
- Local diagnostics

验收：

1. Agent 可安装、启动、退出；
2. Tray 可 Pause / Resume；
3. Device Identity 可持久化；
4. 无 Control Plane 时 Agent 不崩溃；
5. 不依赖常驻 CMD。

---

## Phase 2 — Control Plane + Device Online

目标：打通设备注册和在线状态。

交付：

- Device Registry
- Device Binding
- Agent outbound HTTPS/WSS connection
- Heartbeat / Presence
- Policy sync skeleton
- Device detail API

验收：

```text
安装 Agent
→ 绑定设备
→ Agent 主动连接
→ Control Plane 显示 Online
→ Pause 后状态同步为 Paused
```

---

## Phase 3 — Web Console MVP

目标：建立真正的主要产品界面。

第一批页面：

- Devices List
- Device Detail / Overview
- Capabilities
- Activity
- Settings

要求：

- 参考 ZeroTier Central；
- 表格 / 列表优先；
- 不做首页大屏；
- 不堆装饰性卡片。

验收：

用户可以只通过网页查看设备在线状态、版本、能力和最近 Activity。

---

## Phase 4 — MCP Gateway + First Tool Call

目标：打通 AI → Control Plane → Agent → Result 闭环。

第一批 MCP Tools：

```text
device.list
device.health
device.info
```

交付：

- MCP Gateway
- session identity
- device target selection
- request router
- execution.request/result
- audit record

验收：

> ChatGPT / 本地 MCP Host 调 `device.health` → 目标 Agent 返回结果 → Activity 中可查看完整记录。

这是第一个真正的系统里程碑。

---

## Phase 5 — Files / Shell / Long Tasks

目标：获得基础生产力能力。

Files：

- list
- read
- write
- move
- search
- info

Shell / Process：

- exec
- start long task
- read output
- send stdin
- list process
- cancel / kill

安全：

- Path Scope
- Program Scope
- Allow / Ask / Deny
- timeout / cancel
- Activity / Audit

验收：

AI 可以受控完成：读取项目 → 修改文件 → 执行命令 → 查看输出 → 停止任务。

---

## Phase 6 — Visible Browser Capability

目标：让 AI 在用户可见、可复用登录态的浏览器中操作网页。

能力：

- Managed Browser Profile
- visible Chrome / Edge / Chromium
- open
- inspect
- click
- type
- select
- upload / download
- screenshot
- current state

执行优先级：

```text
DOM / Accessibility
→ Playwright Locator
→ CDP
→ Screenshot / Vision
→ Mouse / Keyboard
```

验收：

用户手动登录一次网站后，Agent 重启仍能复用会话，AI 可以在可见浏览器里继续操作。

---

## Phase 7 — Policy / Approval / Audit

目标：让远程 AI 操作真正可控。

交付：

- Policy UI
- Path / Program / Domain scope
- Risk classification
- Allow / Ask / Deny
- Web approval
- Agent local recheck
- Activity detail
- Audit retention

验收：

高风险操作必须可以被 Web Console 拦截并由用户 Allow Once / Deny。

---

## Phase 8 — Windows GUI Capability

目标：控制非浏览器 Windows 应用。

优先顺序：

1. UI Automation / Accessibility
2. Win32 Window API
3. Screenshot / Vision
4. Mouse / Keyboard fallback

能力：

- list windows
- focus window
- inspect controls
- click control
- set text
- keyboard input
- capture window

验收：

至少稳定控制 2~3 个常见 Windows 应用完成结构化任务。

---

## Phase 9 — Productization

目标：从开发 POC 变成普通用户能安装的产品。

内容：

- Windows Installer
- Auto Update
- First-run binding flow
- Crash recovery
- Agent reconnect
- Browser worker recovery
- Diagnostics / Support bundle
- Settings import/export
- Uninstall / device revoke

期望体验：

```text
下载安装
→ 登录 / 绑定设备
→ Agent 自动在线
→ 打开 Web Console
→ AI 调用
```

不要求用户：

- 手改 JSON；
- 开放公网端口；
- 常驻终端；
- 理解 MCP 底层细节。

---

## Phase 10 — Optional Future Work

只有真实需求验证后再进入：

- Linux headless agent
- 多设备高级策略
- Self-hosted Control Plane
- 自建 Relay
- Team / RBAC
- Mobile monitoring
- Full remote screen stream
- Capability SDK / marketplace

---

## MVP Definition of Done

MVP 不是“几个 Tool 能跑”，而是：

- Windows 可一键安装 Agent；
- Agent 能自动保持在线；
- Web Console 能看到 Device；
- AI Host 能通过 MCP 指定设备调用；
- Files / Shell / Browser 至少形成完整工作流；
- 有 Allow / Ask / Deny；
- 有 Pause / Kill；
- 有 Audit / Activity；
- Browser 可见且登录态持久化；
- Agent 重启可恢复配置；
- 用户不需要一直开 CMD。
