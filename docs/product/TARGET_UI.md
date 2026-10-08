# GSX Remote Agent 目标界面

状态：Target UI v0.2  
设计参考：ZeroTier Central 的“轻 Agent + Web Console”产品形态。

## 1. 设计结论

复杂管理全部进入 Web Console；本机 Agent 只保留最小状态和基础控制。

设计原则：

- 少卡片、少装饰、少渐变；
- 表格 / 列表优先；
- 信息密度高但层级清晰；
- 蓝灰中性色，状态色仅用于在线 / 警告 / 错误；
- 设备是一级对象；
- 用户始终知道“哪台设备在线、允许什么、刚执行了什么”。

## 2. Web Console 一级导航

```text
GSX
├── Devices
├── Connections
├── Policies
├── Activity
└── Settings
```

## 3. Devices 列表页

目标：像 ZeroTier Central 一样，一眼查看所有设备。

建议字段：

| 字段 | 含义 |
|---|---|
| Name | 设备名称 |
| Status | Online / Paused / Offline |
| OS | Windows / Linux |
| Agent | Agent 版本 |
| Connection | Direct / Tunnel / Relay |
| Capabilities | Files / Shell / Browser / GUI |
| Last Seen | 最近在线时间 |
| Actions | 打开详情 / Pause / Remove |

交互：

- 行点击进入 Device Detail；
- 支持搜索、状态过滤；
- `+ Add Device` 进入一次性绑定流程。

## 4. Device Detail

一级 Tab：

```text
Overview | Capabilities | Browser | Activity | Settings
```

### Overview

展示：Device Name、Online State、OS / Hostname、Agent Version、Device ID、Current Connection、Last Seen、Browser Bridge State、当前运行任务。

### Capabilities

权限统一三态：

```text
Allow / Ask / Deny
```

可进一步限制 Path Scope、Program Scope、Domain Scope、Command Scope、Risk Level。

### Browser

展示 Browser Worker、Managed Profile、当前 URL、登录状态、打开 / 重启浏览器、最近操作和必要截图。

### Activity

```text
Time | Source | Tool | Target | Decision | Result | Duration
```

点击记录查看参数摘要、授权决策和结果摘要。

## 5. Policies

以规则表格为主，不做复杂可视化编排器。

| Scope | Capability | Target | Action |
|---|---|---|---|
| PC-01 | filesystem.read | D:/Projects/** | Allow |
| PC-01 | filesystem.delete | * | Ask |
| PC-01 | shell.exec | powershell.exe | Ask |
| PC-01 | browser.* | github.com | Allow |
| * | windows_gui.* | * | Ask |

## 6. 本机 Agent UI

本机窗口目标极简，只显示连接状态、设备信息、浏览器桥、服务状态，以及 Open Web Console / Pause / Reconnect / Exit。

高级设置仅保留开机启动、自动更新、Web Console URL、本地诊断、重置设备绑定。

## 7. Tray

```text
GSX Remote Agent
● Connected

Open Web Console
Pause Agent
Reconnect
Diagnostics
Exit
```

## 8. 目标参考图

### Web Console

![Web Console target](../../assets/target-ui/web-console-product-reference.svg)

### Local Agent

![Minimal Agent target](../../assets/target-ui/minimal-agent-client.svg)

这些 SVG 是**目标导向参考**，不是像素级最终规范。实现时优先遵守本文的信息架构和简洁原则。
