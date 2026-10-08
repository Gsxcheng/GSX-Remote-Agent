# GSX Remote Agent 架构设计

状态：Draft v0.1  
阶段：Architecture Baseline  
目标平台：Windows 优先，后续扩展 Linux/macOS

## 1. 设计目标

GSX Remote Agent 的核心目标不是“让 AI 获得无限制电脑权限”，而是提供一个：

- 标准化：通过 MCP 暴露能力；
- 易用：有桌面 GUI、托盘、状态与日志；
- 可见：用户能知道 AI 正在做什么；
- 可控：权限、目录、程序、域名等可配置；
- 可中断：用户可以立即暂停/终止；
- 可审计：每次调用记录来源、参数摘要、结果与风险等级；
- 可替换：不绑定单一 AI 平台、远程中继或浏览器实现；
- Local-first：敏感能力尽可能在本机执行。

## 2. 核心架构决策

### ADR-01：MCP 是协议边界，不是业务核心

AI Host 只通过标准 MCP 与 GSX Remote Agent 通信。

Agent Core 不感知 ChatGPT、Claude、Cursor 等具体产品，仅处理统一的 capability request。

这样可以避免客户端能力变化导致核心重写。

### ADR-02：传输层插件化

Transport 与 Capability 分离。

支持：

1. Local stdio / Streamable HTTP；
2. OpenAI Secure MCP Tunnel；
3. 未来自建 Relay；
4. 其他兼容 MCP Remote Connector 的传输方式。

OpenAI Tunnel 是推荐适配器之一，但不是唯一入口。

### ADR-03：Remote Desktop Commander 作为 Prior Art，而非硬依赖

Remote Desktop Commander Hosted Service 的服务端实现不是开源组件，因此不作为本项目的基础设施依赖。

重点研究：

- DesktopCommanderMCP 的文件、Shell、进程与编辑工具设计；
- Remote Desktop Commander 的设备在线、配对、远程调用和 UX；
- 审计、超时、长任务、设备管理等成熟经验。

需要复用源码时，只从明确开源且许可证兼容的仓库引入。

### ADR-04：GUI 与 Agent Core 分层

桌面 GUI 负责：

- 状态展示；
- 配置；
- 权限确认；
- 任务查看；
- 日志与审计；
- 浏览器与设备状态；
- 暂停 / 恢复 / Kill Switch。

Agent Core 负责所有真实执行。

UI 关闭时，可按用户配置决定 Agent 是否继续以托盘/后台模式运行。

### ADR-05：Capability-first，而非 Remote-Desktop-first

第一版不实现高带宽“屏幕视频流 + 远程鼠标”式 RDP。

优先提供结构化能力：

- 文件
- Shell
- 进程
- 浏览器 DOM/CDP
- 截图
- Windows UI Automation
- 鼠标/键盘兜底

结构化能力优先于纯视觉点击，只有无法结构化控制时才降级到 GUI 输入。

## 3. 总体组件

```text
┌────────────────────────────────────────────────────────────┐
│ AI Hosts                                                   │
│ ChatGPT / Claude / Cursor / VS Code / Codex / Others      │
└──────────────────────────┬─────────────────────────────────┘
                           │ MCP
                           ▼
┌────────────────────────────────────────────────────────────┐
│ MCP Gateway                                                │
│ - tool/resource exposure                                   │
│ - request validation                                       │
│ - transport-independent interface                          │
└──────────────────────────┬─────────────────────────────────┘
                           │ Local RPC / IPC
                           ▼
┌────────────────────────────────────────────────────────────┐
│ GSX Agent Core                                             │
│                                                            │
│  ┌──────────────────┐  ┌──────────────────┐               │
│  │ Capability       │  │ Permission &     │               │
│  │ Registry         │  │ Policy Engine    │               │
│  └──────────────────┘  └──────────────────┘               │
│                                                            │
│  ┌──────────────────┐  ┌──────────────────┐               │
│  │ Task / Process   │  │ Audit / Event    │               │
│  │ Manager          │  │ Store            │               │
│  └──────────────────┘  └──────────────────┘               │
└──────┬───────────┬────────────┬──────────────┬──────────────┘
       │           │            │              │
       ▼           ▼            ▼              ▼
   Files/Shell   Browser     Windows GUI    Screenshot
                 Worker       Provider

┌────────────────────────────────────────────────────────────┐
│ Desktop App                                                │
│ Tauri 2 + React                                            │
│ Dashboard / Permission Prompt / Logs / Browser / Settings │
│ System Tray / Pause / Kill Switch                          │
└────────────────────────────────────────────────────────────┘

Transport adapters:
- local stdio
- local HTTP
- OpenAI Secure MCP Tunnel
- future relay
```

## 4. 推荐技术栈

### Desktop Shell

- Tauri 2
- React
- TypeScript
- Vite

理由：

- 系统托盘能力成熟；
- Windows 安装包和常驻资源占用优于重型 Chromium Shell；
- Rust 后端适合做系统级权限、进程、IPC 与 Windows API；
- React 适合快速构建控制台 GUI。

### Agent Core

优先 Rust：

- 进程生命周期；
- Windows API；
- 文件系统；
- 权限与策略；
- 本地 IPC；
- 日志；
- Sidecar 管理。

核心原则：**所有高权限执行最终必须经过 Agent Core。**

### MCP Gateway

MVP 推荐使用官方 TypeScript MCP SDK v2 作为 sidecar/gateway。

原因：

- 当前 TypeScript SDK 是 Tier-1 stable；
- 新协议支持最完整；
- MCP 协议迭代与 OS 能力实现解耦；
- Agent Core 不需要自己维护协议兼容细节。

后续如果 Rust MCP SDK 稳定度满足要求，可评估合并进 Rust Core，减少 sidecar。

### Browser Worker

- Playwright
- Chromium/Chrome/Edge CDP
- 可见浏览器窗口
- 持久化受管 Profile

MVP 默认使用 **GSX Managed Browser Profile**：用户登录一次后长期保留 Cookie/Session。

不要在 MVP 阶段承诺无条件接管用户任意已经打开的默认 Chrome Profile；浏览器安全策略、远程调试限制和 Profile 锁会造成不稳定。

后续再增加：

- Attach existing browser
- Browser extension bridge
- Chrome DevTools Protocol attach

### Windows GUI Provider

能力顺序：

1. Windows UI Automation / Accessibility；
2. Win32 Window APIs；
3. Screenshot + visual target；
4. Mouse / keyboard input fallback。

不要默认以坐标点击作为第一方案。

## 5. Capability Model

所有能力通过统一接口注册。

建议基础结构：

```text
Capability
├── id
├── name
├── version
├── risk_level
├── required_scopes
├── input_schema
├── execute()
└── audit_metadata()
```

### P0 Capability

#### filesystem

- list_directory
- read_file
- write_file
- move_file
- search_files
- file_info

#### shell

- start_process
- send_input
- read_output
- list_processes
- kill_process

#### screenshot

- capture_screen
- capture_window
- list_windows

#### browser

- browser_status
- open_url
- inspect_page
- click
- type
- select
- upload_file
- download_file
- screenshot_page

### P1 Capability

#### windows_gui

- list_apps
- focus_window
- inspect_controls
- invoke_control
- set_text
- click_screen
- type_keys

#### clipboard

- read_clipboard
- write_clipboard

#### device

- device_info
- health
- agent_status

## 6. 权限模型

不采用单一的“AI 已授权 / 未授权”。

至少分以下 Scope：

```text
files.read
files.write
shell.read
shell.exec
process.manage
browser.read
browser.write
screen.capture
gui.inspect
gui.input
clipboard.read
clipboard.write
system.admin
```

每个 Scope 支持：

- Allow
- Ask
- Deny

还应支持更细粒度规则：

- 文件路径白名单/黑名单；
- Shell 命令规则；
- 可执行程序白名单；
- 浏览器域名规则；
- 是否允许下载/上传；
- 是否允许 Admin；
- 是否允许后台执行。

### 强制确认操作

无论默认策略如何，以下类型默认需要单独确认：

- 删除大量文件；
- 修改系统安全设置；
- 安装/卸载软件；
- 提权；
- 密钥/凭据操作；
- 金融/支付/购买；
- 发布/提交不可逆外部操作；
- 关机/重启；
- 大规模批量操作。

## 7. Risk Engine

每个 Tool Call 进入 Core 后：

```text
Request
 → Schema Validate
 → Identify Caller/Session
 → Resolve Capability
 → Calculate Risk
 → Policy Evaluate
 → [Allow | Ask | Deny]
 → Execute
 → Audit
 → Return Result
```

建议风险等级：

- R0：纯读取、状态查询
- R1：低风险本地变更
- R2：可恢复系统/浏览器操作
- R3：高影响或外部副作用
- R4：管理员/安全敏感/不可逆

## 8. GUI 信息架构

### Dashboard

显示：

- Agent Online/Offline
- MCP 连接状态
- 当前 Transport
- Browser 状态
- 当前任务
- 最近调用
- 风险提示

### Devices

MVP 只有 Local Device。

未来支持：

- 多设备
- 云服务器
- Device Alias
- Device Health

### Capabilities

用户可查看和修改每个 Capability 的权限。

### Browser

- 浏览器启动/停止
- 当前 Profile
- 当前页面
- 页面截图
- 打开浏览器
- 登录状态提示

### Activity / Audit

展示：

- 时间
- AI Host
- Tool
- 参数摘要
- 风险等级
- 用户确认状态
- 成功/失败
- 执行耗时

### Settings

- 开机启动
- 后台运行
- MCP Gateway
- Tunnel Adapter
- Browser Profile
- Logging
- Update

### System Tray

至少：

- 状态
- Open Dashboard
- Pause AI Control
- Resume
- Browser Status
- Exit

## 9. Kill Switch

Kill Switch 是 MVP 必须能力，不是增强项。

支持：

- GUI 一键 Pause；
- Tray Pause；
- 可配置全局快捷键；
- 停止新 Tool Call；
- 可选终止当前执行；
- 浏览器 Worker / Shell Task 可单独 Kill。

## 10. Transport Layer

### Local MCP

用于：

- Claude Desktop
- Cursor
- Codex CLI
- VS Code
- 本地调试

### OpenAI Secure MCP Tunnel

作为 OpenAI 生态 Remote Adapter：

```text
ChatGPT
  → OpenAI hosted tunnel endpoint
  → tunnel-client
  → local MCP Gateway
  → GSX Agent Core
```

GSX GUI 可以负责：

- 检测 tunnel-client；
- 启停 sidecar；
- 健康检查；
- 展示 tunnel 状态；
- 引导配置。

但不把 OpenAI tunnel identity、控制平面和实现写死到 Core。

### Future Relay

只有出现以下需求再实现：

- 非 OpenAI 网页 AI 需要同样的远程接入；
- 多设备聚合；
- 自托管；
- 无官方 Tunnel 可用；
- 跨平台账户/设备管理。

## 11. Browser Strategy

为了“好用 + 有 GUI”，浏览器是第一等能力，不是普通插件。

优先策略：

```text
DOM / Accessibility Tree
        ↓
Playwright Locator
        ↓
CDP
        ↓
Screenshot + Vision
        ↓
OS mouse/keyboard fallback
```

浏览器必须默认可见，用户可以在前台观察 AI 操作。

MVP 浏览器使用独立持久化 Profile：

```text
%APPDATA%/GSX-Remote-Agent/browser-profiles/default
```

用户第一次手动登录后，后续 Agent 复用该 Profile。

## 12. Process / Long Task Model

不能把 Shell 设计成一次 request = 一次阻塞命令。

Task Manager 需要：

- task_id
- pid
- state
- started_at
- stdout cursor
- stderr cursor
- timeout
- cancel
- send_stdin

用于：

- dev server
- SSH
- REPL
- 编译
- Docker logs
- 安装流程

## 13. Audit Log

默认使用本地 SQLite。

建议表：

- sessions
- tool_calls
- approvals
- tasks
- events
- settings_history

日志不默认记录完整 secret。

敏感字段应：

- redact
- hash
- metadata only

## 14. 项目目录建议

```text
GSX-Remote-Agent/
├── apps/
│   ├── desktop/              # Tauri 2 + React
│   └── mcp-gateway/          # Official MCP TS SDK
├── crates/
│   ├── agent-core/
│   ├── capability-core/
│   ├── policy-engine/
│   ├── audit-store/
│   ├── task-manager/
│   ├── win-provider/
│   └── ipc/
├── workers/
│   └── browser-worker/       # Playwright
├── packages/
│   ├── protocol/             # shared schemas/types
│   └── ui-types/
├── docs/
│   ├── ARCHITECTURE.md
│   ├── ROADMAP.md
│   └── adr/
├── tests/
│   ├── integration/
│   └── e2e/
└── README.md
```

## 15. MVP 数据流示例

用户：

> 打开浏览器进入 GitHub，把这个仓库 README 打开。

执行链：

```text
ChatGPT
 → browser.open_url
 → MCP Gateway
 → Agent Core
 → Policy: browser.write = Allow
 → Browser Worker
 → Playwright open_url
 → Audit Store
 → result
```

用户：

> 删除 C:\Users\xxx\Downloads 下所有 zip。

执行链：

```text
ChatGPT
 → filesystem.delete
 → Agent Core
 → Risk R3
 → Policy = Ask
 → GUI Approval Dialog
 → User Approve / Deny
 → Execute
 → Audit
```

## 16. 当前不做的事情

为避免第一版失控，以下内容先明确排除：

- 自建云端 Relay；
- 完整 Team / Tenant / Billing；
- LLM 推理层；
- Agent Planner；
- RDP/VNC 替代品；
- 手机客户端；
- macOS/Linux GUI Automation；
- 高权限无人值守 Admin Agent。

## 17. 架构验收标准

进入编码阶段前，至少确认：

1. MCP Gateway 可以被替换，不影响 Agent Core；
2. Capability 不直接访问 GUI 状态；
3. 所有执行经过 Policy Engine；
4. 所有 Tool Call 可进入 Audit；
5. Browser Worker 可独立崩溃/重启；
6. Tunnel 不属于 Agent Core 必选依赖；
7. GUI 能暂停全部 AI 控制；
8. Windows 本地闭环优先于云端功能。
