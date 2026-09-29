# QuantDo TradingView MCP

这份文档记录项目的整体思路、三个 npm 包如何配合，以及使用中常见问题的讨论入口。希望 AI 不只是描述图表，而是能够在用户指定的图表上读取数据、画线、添加指标，并把分析过程清楚地展示出来。

## 三个 npm 包

| 包 | 负责什么 | 何时使用 |
|---|---|---|
| [`@quantdo/tv-tool-core`](https://www.npmjs.com/package/@quantdo/tv-tool-core) | 定义受限的图表工具、校验参数、查询能力，并通过 Chart Adapter 操作图表 | [阅读 Core 文档](tv-tool-core.md) |
| [`@quantdo/tv-ai-bridge`](https://www.npmjs.com/package/@quantdo/tv-ai-bridge) | 在浏览器页面建立 WebSocket 连接，把外部工具调用交给页面注入的执行器 | [阅读 Bridge 文档](tv-ai-bridge.md) |
| [`@quantdo/tv-ai-orchestrator`](https://www.npmjs.com/package/@quantdo/tv-ai-orchestrator) | 执行受限的分析步骤、模板和播放控制，跟踪并清理本次分析创建的对象 | [阅读 Orchestrator 文档](tv-ai-orchestrator.md) |

三个包可以分别使用。它们不包含行情源，也不会自动创建图表页面或 MCP 服务。Core 和 Orchestrator 不依赖 TradingView 页面实现：当前可以使用 TradingView-compatible Chart Adapter，未来也可以接入自研图表。Bridge 不负责分析逻辑，只负责页面连接与调用传递。

安装示例：

```bash
npm install @quantdo/tv-tool-core @quantdo/tv-ai-bridge @quantdo/tv-ai-orchestrator
```

不需要分析播放器时，可以省略 `tv-ai-orchestrator`。目前三个包只提供 ESM 入口；详细 API 说明随 npm 包提供，这里保留常用接入说明。

## 整体调用思路

```text
用户提出图表任务
  → AI 客户端选择语义工具
  → 应用的 MCP 接入层将调用送到指定页面
  → tv-ai-bridge 将调用交给页面执行器
      ├─ tv-tool-core → Chart Adapter → 当前图表
      └─ tv-ai-orchestrator → 分析步骤 → tv-tool-core
  → 图表结果返回 AI；分析进度和动画由页面展示
```

页面持有图表实例，并显式注入 Chart Adapter 和工具执行器。Core 负责“图表实际发生了什么”，Orchestrator 负责“按什么顺序讲解”，Bridge 负责“外部调用怎样到达页面”。动画、图例、面板以及用户手动接管由页面实现，不能把视觉效果当成数据正确的证明。

外部 AI 必须明确指定目标页面，不应自动选择最近打开的标签页，也不应得到其他页面的会话列表。当前本地方案用页面展示的 `tabCode` 路由；这只是开发阶段的页面标识，不是可用于公网服务的身份认证。

## 典型任务怎么处理

| 用户说法 | 合理的处理方式 |
|---|---|
| “画一条趋势线，我自己选点” | 激活交互式画线工具，让用户在图表上落点 |
| “连接昨日低点和今日高点” | 先确认合约与周期，读取 K 线，按明确区间计算时间/价格锚点，再创建图形 |
| “分析当前图表” | 读取当前可见数据，用保守的观察结果启动分析展示；观察到的高低点不等于已确认支撑/阻力 |
| “展示支撑、阻力和突破路径” | 在用户给出价位或可复核规则后，逐步创建图形、关联图例与结论 |
| “清理刚才的分析” | 只清理该次分析创建的对象，不影响用户原有图形和指标 |

图表工具使用 Unix 秒、价格、合约和周期等语义参数，不接收浏览器像素坐标，也不执行任意 JavaScript 或 Pine。数据派生的写入应核对当前合约、周期和短时效快照；数据变化或快照过期时重新取数、重算锚点，不能让 AI 猜测。

## 当前能力与边界

- Core 支持读取图表上下文与可见 K 线、合约搜索/精确切换、绘图和宿主已注册指标的生命周期；实际可用能力以宿主运行时返回为准。
- Orchestrator 提供趋势、震荡、突破和量价模板，以及暂停、单步、跳过、调速、重播、取消和按分析批次清理。
- 专业标注、镜头转场和指标组合仍在持续优化；不同图表宿主的细节能力需要分别验证，不能把某个页面的效果直接当作所有宿主都支持。
- 当前本地页面访问码不能替代正式认证、权限控制、审计、限流和 TLS/WSS；不要把本地接入方案直接部署到公网。

## 提问和反馈

欢迎在 [GitHub Issues](https://github.com/gitlwz/quantdo-tradingview-mcp/issues) 提问、报告问题或讨论希望增加的图表能力。请写明所用 npm 包及版本、期望效果、实际效果和尽可能简短的复现步骤。分享截图或日志前，请移除访问码、账户信息、Cookie、认证 Header 和行情密钥。
