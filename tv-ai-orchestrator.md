# `@quantdo/tv-ai-orchestrator`

[npm 包](https://www.npmjs.com/package/@quantdo/tv-ai-orchestrator) · [提问与反馈](https://github.com/gitlwz/quantdo-tradingview-mcp/issues)

Orchestrator 负责把一次图表分析组织成受限的步骤，并跟踪本次创建的对象。它不连接 MCP/WebSocket，不读取页面 DOM，也不直接实现动画；页面根据事件绘制进度、图例、镜头和强调效果。

## 接入方式

```bash
npm install @quantdo/tv-ai-orchestrator
```

```ts
import { createAnalysisOrchestrator } from "@quantdo/tv-ai-orchestrator";

const orchestrator = createAnalysisOrchestrator({
  tools: core, // { execute(name, args): Promise<unknown> }
  onEvent: (event) => renderAnalysis(event.status, event.type),
});

const running = await orchestrator.execute("present_analysis", plan);
const finalStatus = await orchestrator.execute("await_analysis", {
  analysisId: running.analysisId,
  timeoutMs: 50_000,
});
```

`present_analysis` 可能先返回 `running`，不代表已完成；应等待最终状态或查询 `get_analysis_status`。`analysisId` 关联本次创建的图形与指标，`clear_analysis` 只清理该批次对象。

## 步骤、模板和播放

允许的步骤包括定位时间范围/实体、创建图形或指标、展示结论、有界等待，以及强调、恢复、逐段显示已创建对象。单次分析最多 30 步，等待和步骤间隔最长 3 秒；不接受任意工具名或脚本。每个绘图步骤需要语义 `label`，状态中的 `annotations` 和 `entities` 可供页面生成图例。

内置趋势、震荡、突破、量价四类模板；可先预览参数，再播放。播放支持暂停、继续、单步、跳过、调速、重播、取消及清理。一键速览只标记当前可见 K 线中的**已观察高低点**，不能将它们当作已确认支撑/阻力或投资建议。

分析计划的时间和价格锚点必须来自用户或可核对的数据规则。镜头缓动、低动效模式、手动接管和图形标签避让由页面负责；动画完成不等于数据校验通过。当前分析状态仅在实例内存中，页面刷新后不自动恢复。
