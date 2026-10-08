# GSX Remote Agent 产品方向

状态：Target Baseline v0.2  
参考形态：ZeroTier Central + 轻量设备 Agent

## 1. 产品定义

GSX Remote Agent 采用 **轻 Agent + Web Control Plane** 的产品形态。

目标不是在每台电脑上做一个复杂桌面控制台，而是：

- 每台 Windows / Linux 设备只安装一个轻量 Agent；
- Agent 主动出站连接控制平面，不要求用户开放本机入站端口；
- 设备、连接、权限、日志、任务和策略统一在 Web Console 管理；
- AI Host 通过标准 MCP 调用设备能力；
- 实际执行仍发生在用户自己的电脑或服务器；
- 高风险操作受策略、确认和审计约束；
- 用户可以随时暂停或断开 Agent。

## 2. 产品原则

### 2.1 Web Console First

复杂管理能力全部优先进入 Web：

- Devices
- Connections
- Policies
- Activity
- Settings

本机 Agent 不复制这些管理页面。

### 2.2 Lightweight Agent

本机 Agent 只承担：

- 设备身份与绑定；
- 与 Control Plane 的持久连接；
- MCP / Capability 请求执行；
- Browser Bridge；
- 本地 Policy Enforcer；
- Pause / Resume / Reconnect；
- 自动更新和基础诊断。

### 2.3 Local Execution

文件、Shell、浏览器和 Windows GUI 能力默认在设备本地执行。Control Plane 不替用户执行本机操作，只负责路由、策略、授权和审计。

### 2.4 Capability First

第一版优先结构化能力，而不是传统远程桌面：

1. Files
2. Shell / Process
3. Browser
4. Screenshot
5. Windows GUI
6. Docker / Git / SSH 等扩展

### 2.5 Visible / Controllable / Auditable

任何 AI 操作都应满足：

- 可看到设备当前状态；
- 可配置允许范围；
- 高风险动作可要求确认；
- 可立即暂停；
- 有完整 Activity / Audit 记录。

## 3. 产品信息架构

```text
GSX Web Console
├── Devices
│   ├── Device List
│   └── Device Detail
│       ├── Overview
│       ├── Capabilities
│       ├── Browser
│       ├── Activity
│       └── Settings
├── Connections
├── Policies
├── Activity
└── Settings
```

## 4. 本机 Agent 信息架构

Agent UI 保持极简：

```text
GSX Remote Agent

PC-01                         ● Connected
Windows 11 Pro

Connection      Secure MCP / Connected
Browser Bridge  Ready
Service         Running
Last Seen       just now

[ Open Web Console ]

[ Pause Agent ] [ Reconnect ] [ Exit ]
```

## 5. 非目标

当前不做：

- 大而全的桌面 Dashboard；
- 高带宽远程视频桌面；
- 自研聊天客户端；
- 自研模型规划器；
- 为了炫酷而增加非必要可视化；
- 将 OpenAI、Remote Desktop Commander 或任一厂商变成不可替换硬依赖。

## 6. 目标体验

用户侧最终体验应接近：

```text
下载安装 Agent
→ 登录 / 绑定设备
→ Agent 自动保持在线
→ 在 Web Console 管理设备和权限
→ ChatGPT / Claude 通过 MCP 调用
→ Agent 本地执行
→ Web Console 查看结果和审计
```

一句话目标：**像 ZeroTier 一样安装和管理设备，像 MCP 一样把设备能力提供给 AI。**
