# `@quantdo/tv-ai-bridge`

[npm 包](https://www.npmjs.com/package/@quantdo/tv-ai-bridge) · [提问与反馈](https://github.com/gitlwz/quantdo-tradingview-mcp/issues)

Bridge 是运行在浏览器页面的 WebSocket 客户端：注册页面、在断线后重连、接收外部工具调用、交给页面注入的执行器，并将对应结果或错误返回。它不创建 Core、不操作 TradingView Widget，也不是 MCP HTTP 服务端。

## 接入方式

```bash
npm install @quantdo/tv-ai-bridge
```

```ts
import { createTvAiBridge } from "@quantdo/tv-ai-bridge";

const bridge = createTvAiBridge({
  url: websocketUrl,
  tools: executor, // { execute(name, args): Promise<unknown> }
  title: () => document.title,
  pageUrl: () => location.href,
  tabId: () => currentTabCode,
  onTabConflict: () => createAndPersistAnotherTabCode(),
  onStatus: (status) => renderConnectionState(status),
});
bridge.connect();
```

`executor` 应由页面显式提供，可把原子工具交给 Core，把分析工具交给可选的 Orchestrator。页面负责显示连接状态和当前标签页标识；Bridge 不自行生成授权 UI。

## 生命周期与边界

- `connect()` 建立连接并在需要时重连；`disconnect()` 主动停止连接和重连。
- `setTabId(code)` 用于页面重置标识后立即更新注册值。
- `onStatus` 报告连接、执行和错误状态，具体显示由页面实现。
- `reportAnalysisEvent(event)` 可报告异步分析步骤与失败，供应用诊断。
- 默认定期检查图表上下文；内容不变时不重复上报。

内部 `sessionId` 只用于传输关联，不应展示给 AI。当前本地使用的四位 `tabCode` 用于定位页面，不是生产授权；远程部署需要另行实现身份认证、权限和安全传输。
