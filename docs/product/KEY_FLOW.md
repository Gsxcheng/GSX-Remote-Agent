# GSX Remote Agent 关键执行流程

状态：Target Flow v0.2

## 1. 设备首次接入

```mermaid
sequenceDiagram
    participant U as User
    participant A as GSX Agent
    participant C as GSX Control Plane
    participant W as Web Console

    U->>A: 安装并启动 Agent
    A->>C: 创建设备注册请求
    C-->>A: 返回绑定码 / 登录链接
    U->>W: 登录并确认绑定
    W->>C: 授权设备
    C-->>A: 下发 Device Identity / Policy
    A->>C: 建立持久出站连接
    C-->>W: Device = Online
```

目标：用户不手改 JSON、不开放公网端口、不常驻 CMD。

## 2. AI 调用设备能力

```mermaid
flowchart TD
    A[AI Host 发起 MCP Tool Call] --> B[MCP Gateway 校验 Schema / Session]
    B --> C[选择目标 Device]
    C --> D[Server Policy Engine]
    D --> E{是否需要用户确认?}

    E -- 否 --> G[路由到在线 Agent]
    E -- 是 --> F[Web / Agent Approval]
    F --> H{用户允许?}
    H -- 否 --> X[拒绝请求 + 写审计]
    H -- 是 --> G

    G --> I[Agent Local Policy Check]
    I --> J{本地允许?}
    J -- 否 --> X
    J -- 是 --> K[Capability Router]
    K --> L[执行 Files / Shell / Browser / GUI]
    L --> M[结果标准化]
    M --> N[Audit / Activity]
    N --> O[结果回传 AI Host]
```

## 3. 浏览器任务流程

```mermaid
flowchart TD
    A[browser.open_url / inspect / click / type] --> B[Browser Capability]
    B --> C{Browser Worker 是否在线?}
    C -- 否 --> D[启动 Managed Browser Profile]
    C -- 是 --> E[复用现有 Profile]
    D --> E
    E --> F[Playwright / CDP]
    F --> G{结构化定位可用?}
    G -- 是 --> H[DOM / Accessibility / Locator 执行]
    G -- 否 --> I[Screenshot / Vision fallback]
    H --> J[返回页面状态]
    I --> J
    J --> K[Activity / Screenshot Evidence]
```

浏览器执行优先级：

```text
DOM / Accessibility
→ Playwright Locator
→ CDP
→ Screenshot / Vision
→ Mouse / Keyboard fallback
```

## 4. 高风险操作确认

典型高风险：删除文件、安装软件、系统配置修改、发送/提交、敏感目录写入等。

```mermaid
sequenceDiagram
    participant AI as AI Host
    participant C as Control Plane
    participant U as User
    participant A as Agent

    AI->>C: 请求高风险 Tool Call
    C->>C: 风险分类 + Policy 匹配
    C-->>U: 展示操作、目标、风险、参数摘要
    U-->>C: Allow once / Always ask / Deny
    alt Allow once
        C->>A: Signed execution request
        A->>A: Local policy check
        A-->>C: Result
        C-->>AI: Result
    else Deny
        C-->>AI: Permission denied
    end
    C->>C: 写入 Audit Log
```

## 5. 用户随时干预

```text
Pause Agent
→ Agent 停止接收新任务
→ 当前可取消任务进入 Cancel
→ Control Plane 标记 Paused
→ AI 新请求返回 Device Paused
```

Kill Switch 必须优先于任何 AI 请求。

## 6. Agent 断线恢复

```mermaid
flowchart LR
    A[Connection Lost] --> B[Local Backoff]
    B --> C[Reconnect]
    C --> D{Identity 有效?}
    D -- 是 --> E[拉取最新 Policy]
    D -- 否 --> F[重新认证 / 绑定]
    E --> G[恢复 Online]
    F --> G
```

目标：断网、休眠、进程崩溃后能够自动恢复，不依赖用户打开终端重启。
