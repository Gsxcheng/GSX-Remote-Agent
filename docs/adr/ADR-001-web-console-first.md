# ADR-001: Web Console First + Lightweight Agent

- Status: Accepted
- Date: 2026-10-08

## Context

早期方案倾向在 Windows 本地客户端承载大量设备状态、权限、任务、日志和浏览器管理页面。该形态虽然直观，但会导致：

- 桌面客户端越来越重；
- 多设备管理体验差；
- Windows / Linux / 云服务器难统一；
- UI 与本地执行强耦合；
- 后续远程管理和账号体系需要重复建设。

ZeroTier 的产品形态验证了“设备运行轻 Agent，集中在 Web 管理”的可用性和扩展性。

## Decision

GSX Remote Agent 采用：

> **Lightweight Device Agent + Web Control Plane / Web Console**

本地 Agent 只保留：

- 在线状态；
- 设备身份；
- 连接管理；
- Capability 执行；
- Browser Bridge；
- Local Policy Enforcer；
- Pause / Resume / Reconnect / Exit；
- 基础诊断和自动更新。

Web Console 承担：

- Devices；
- Connections；
- Policies；
- Activity / Audit；
- Browser 状态；
- 设备配置；
- 高风险审批。

## Consequences

### Positive

- 一套 Web UI 管多台电脑和服务器；
- Agent 更轻、更稳定；
- Linux server 可无 GUI 运行；
- AI Host、传输层和本机执行更容易解耦；
- 后续可扩展 SaaS / self-hosted Control Plane。

### Negative

- 需要更早实现 Control Plane；
- 设备离线时 Web Console 的部分操作不可用；
- 账号、设备身份、连接协议需要从 MVP 初期设计；
- 高风险操作不能只依赖云端，仍要在 Agent 做本地校验。

## Rejected Alternative

### Full Desktop Dashboard

不采用“大而全桌面端”作为主要控制界面。

原因：它适合单机工具，不适合本项目后续多设备、服务器和远程 AI 接入目标。

## Guardrail

未来任何新增管理功能，默认先问：

> 这是设备本地必须完成的执行职责，还是可以进入 Web Control Plane 的管理职责？

若属于管理职责，默认放 Web Console，避免 Agent 再次膨胀。
