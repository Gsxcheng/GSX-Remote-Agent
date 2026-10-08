# GSX Remote Agent MVP Roadmap

状态：Draft v0.1

## Phase 0 — Architecture Baseline

目标：只做设计和工程骨架决策，不写业务功能。

交付：

- [x] 项目定位
- [x] 核心架构
- [x] Transport / Capability 分层
- [x] GUI 技术路线
- [x] 权限与审计基线
- [x] Browser 策略
- [ ] 许可证与第三方依赖清单
- [ ] ADR 拆分与冻结
- [ ] MVP Tool Contract 定稿

退出条件：架构验收标准全部通过。

---

## Phase 1 — Desktop Shell + Agent Core Skeleton

目标：先把“一个真正能常驻、能看到状态、能暂停”的桌面 Agent 跑起来。

交付：

- Tauri 2 + React GUI
- System Tray
- Rust Agent Core
- Local IPC
- SQLite Audit Store
- Capability Registry
- Policy Engine skeleton
- Kill Switch
- 开机启动（可选）

验收：

1. GUI 可启动/退出；
2. 托盘可 Pause / Resume；
3. Core 与 UI 进程通信正常；
4. 本地日志可查看；
5. 无 MCP 时 Agent 也能独立启动。

---

## Phase 2 — MCP Gateway + First Tool Call

目标：打通最小 AI → Agent 闭环。

第一批 Tools：

- device.health
- device.info
- filesystem.list_directory
- filesystem.read_file

交付：

- MCP TypeScript Gateway
- stdio transport
- Streamable HTTP transport
- Gateway → Core IPC
- schema validation
- session identity
- audit log

验收：

从本地 MCP Host 发起 `device.health`，结果经过 Gateway/Core/Audit 后返回。

---

## Phase 3 — Files / Shell / Long Tasks

目标：达到 Desktop Commander 类基础生产力。

能力：

- read/write/move/search files
- start process
- background process
- stdout/stderr incremental read
- stdin
- list/kill process
- timeout/cancel

安全：

- 路径 Scope
- 命令风险规则
- Ask/Allow/Deny
- GUI Approval

验收：

AI 可以受控地完成：读取项目 → 修改文件 → 启动命令 → 观察输出 → 终止任务。

---

## Phase 4 — Visible Browser Agent

目标：实现“用户能看见 AI 在浏览器里操作”。

能力：

- Managed Browser Profile
- visible Chromium/Chrome/Edge
- open_url
- inspect_page
- click/type/select
- upload/download
- screenshot
- page status

GUI：

- Browser Online
- 当前 URL
- Profile
- 打开/关闭浏览器
- 当前截图

验收：

用户手动登录一次网站后，下一次启动仍保留会话，AI 能在可见浏览器中继续操作。

---

## Phase 5 — Windows GUI Capability

目标：控制非浏览器 Windows 程序。

优先顺序：

1. UI Automation
2. Win32 Window API
3. Screenshot/Vision
4. Mouse/Keyboard fallback

能力：

- list_windows
- focus_window
- inspect_controls
- click control
- set text
- keyboard input
- capture window

验收：

至少可稳定控制 2~3 个常见 Windows 应用完成结构化操作。

---

## Phase 6 — Remote Access Adapters

目标：网页 AI 可以安全连接本机。

### Adapter A — OpenAI Secure MCP Tunnel

- detect tunnel-client
- configure/run sidecar
- health state
- GUI status
- restart/recovery

### Adapter B — Generic Remote MCP

保留标准 Streamable HTTP 接口，为 Cloudflare Tunnel、自建 Relay 或其他平台预留。

验收：

ChatGPT Web 可以通过远程 MCP 路径调用本机 `device.health` 与至少一个低风险能力。

---

## Phase 7 — Productization

目标：从开发工具变成真正“好用”的桌面产品。

内容：

- Installer
- Auto update
- First-run wizard
- Permission presets
- Diagnostics
- Support bundle
- Crash recovery
- Browser worker recovery
- MCP/tunnel recovery
- Settings import/export

首次安装期望体验：

```text
下载安装
 → 打开 GSX Remote Agent
 → 选择连接 ChatGPT / Local MCP
 → 选择允许的能力
 → 浏览器首次登录
 → Ready
```

不要求用户手改 JSON 或常驻 CMD。

---

## Phase 8 — Optional Future Work

只有验证真实需求后再进入：

- 多设备
- Headless Linux server agent
- 自建 Relay
- Device pairing account system
- Mobile monitor
- Team policy
- Full remote screen stream
- Plugin marketplace
- Capability SDK

---

## MVP Definition of Done

MVP 完成不是“Tool 能跑”。必须同时满足：

- ChatGPT/一个远程 MCP Host 能调用；
- Windows 有 GUI；
- 有托盘；
- 有 Pause/Kill；
- 有权限确认；
- 有 Audit；
- Browser 是可见的；
- 登录态可持久化；
- Shell 长任务可管理；
- Agent 重启后基本配置保留；
- 用户不需要一直开 CMD 窗口。
