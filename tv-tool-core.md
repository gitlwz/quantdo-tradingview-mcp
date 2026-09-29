# `@quantdo/tv-tool-core`

[npm 包](https://www.npmjs.com/package/@quantdo/tv-tool-core) · [提问与反馈](https://github.com/gitlwz/quantdo-tradingview-mcp/issues)

Core 把 AI 可用的图表操作限制为有明确输入和结果的语义工具。它负责工具定义、JSON Schema、参数校验、统一错误和 Chart Adapter 接口；不会自行寻找 Widget、获取行情或连接 MCP/WebSocket。

## 接入方式

```bash
npm install @quantdo/tv-tool-core
```

页面持有图表实例，并显式创建 Adapter：

```ts
import { createTradingViewChartAdapter, createTvToolCore } from "@quantdo/tv-tool-core";

const adapter = createTradingViewChartAdapter(widget, {
  title: document.title,
  getVisibleBars: () => readVisibleBars(),
});
const core = createTvToolCore(adapter);
const context = await core.execute("get_chart_context");
const available = await core.execute("get_capabilities");
```

`widget` 和 `readVisibleBars` 由使用方提供。自研图表可以实现 `TvChartAdapter`，不必依赖 TradingView-compatible Widget。`core.listTools()` 可查看当前工具定义和输入 Schema；运行时是否真正支持某能力，应以 `get_capabilities` 为准。

## 工具范围

- 图表与数据：`get_chart_context`、`get_capabilities`、`get_visible_bars`、`get_chart_snapshot`、`set_visible_range`。
- 合约：宿主注入搜索和解析能力后，可用 `search_symbols`、`set_chart_symbol`。
- 绘图：`activate_drawing_tool`，以及图形的创建、查询、更新、删除。
- 指标：查询宿主指标目录，以及已注册指标的创建、查询、修改、删除。

绘图点位使用 `{ time, price }`：`time` 是 Unix 秒，`price` 是有限正数，不传屏幕像素。创建图形后返回稳定 `id`、最终点位或明确错误。模糊“画线”应激活交互工具；数据派生的线应先读取 K 线、按明确规则计算锚点。

`get_chart_snapshot` 返回短时效 `snapshotId`。依据快照执行写操作时，应同时传 `expectedSymbol`、`expectedResolution` 和适用时的 `expectedSnapshotId`；数据或可见范围变化、快照过期后应重新取数并计算点位。Core 不执行 Pine、任意 JavaScript 或 DOM 操作。`price_label` 在当前 TradingView-compatible 实现只保证原生价格显示，自定义语义文字应由页面图例承载。
