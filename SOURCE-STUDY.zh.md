# AgentScope Java 源码跟读

上游基线是 `v2.0.4` 的提交 `0b6743b263ba8e39eb417e35048026f1e0196548`。本分支在上游基础上添加中文注释和跟读路线。[官方发布](https://github.com/agentscope-ai/agentscope-java/releases/tag/v2.0.4)

从 [Maven 模块索引](SOURCE-MODULES.zh.md) 查看所有构建入口。建议先顺着下面三条路径读，每条路径只关注一类状态变化。

| 路线 | 源码入口 | 观察内容 |
| --- | --- | --- |
| Core 的 ReAct 循环 | [ReActAgentNewLoopE2ETest](agentscope-core/src/test/java/io/agentscope/core/agent/ReActAgentNewLoopE2ETest.java) 的 `twoToolReactLoopProducesOrderedEventsAndFinalText` → [ReActAgent](agentscope-core/src/main/java/io/agentscope/core/ReActAgent.java) 的 `beforeAgentExecution`、`reasoning`、`executeToolCalls` | 本次调用使用哪个会话状态，模型输入有哪些消息与工具，工具结果怎样与调用配对，最终产生哪些事件 |
| Harness 会话 | [HarnessAgent.prepareRun](agentscope-harness/src/main/java/io/agentscope/harness/agent/HarnessAgent.java) → [ReActAgent.prepareRun](agentscope-core/src/main/java/io/agentscope/core/ReActAgent.java) | 对照传入的消息与运行上下文，再看状态定位和准备运行的分支 |
| Web 请求与事件 | [AG-UI 示例配置](agentscope-examples/agui/src/main/java/io/agentscope/examples/agui/config/AgentConfiguration.java) → [AguiWebFluxHandler.handle](agentscope-extensions/agentscope-spring-boot-starters/agentscope-agui-spring-boot-starter/src/main/java/io/agentscope/spring/boot/agui/webflux/AguiWebFluxHandler.java) → [AguiAgentAdapter.run](agentscope-extensions/agentscope-extensions-protocol/agentscope-extensions-agui/src/main/java/io/agentscope/core/agui/adapter/AguiAgentAdapter.java) | 跟踪请求体、会话和运行标识、框架事件映射，以及返回的 SSE |

## 第一次调试

1. 在 IDEA 中打开本仓库根 `pom.xml`，设 Project SDK、Maven Importer 和 Runner 为 JDK 17，并使用本机 Maven 3.9.14。
2. 在 Core 的 `ReActAgentNewLoopE2ETest` 中找到 `twoToolReactLoopProducesOrderedEventsAndFinalText`。在测试调用 `streamEvents` 前下断点，然后用 IDEA 的 **Debug** 执行该方法。该测试提供固定的模型响应，无需模型 API Key。
3. 进入 `beforeAgentExecution`，比较 `ctx` 与 `scope.state`。在 `reasoning` 中观察 `iter`、`modelInput` 和 `tools`。Reactor 异步回调可能切换线程，分别设置方法体内的断点便于跟踪。
4. 在 `executeToolCalls` 查看每个 `ToolUseBlock` 的 ID 和名称。继续到 `dispatchToolCalls`，查看正常工具请求如何交给 `Toolkit.callTools`，再回到测试核对工具事件和最终文本。
5. 运行同一测试类中的 `toolReturningErrorBlockEmitsErrorResultEndState`，对照正常路径的工具事件与结果状态。

当前分支先注释上述 ReAct 入口。Workspace、Skill、压缩、恢复及沙箱等专题会随实验追加。工作区外层项目的 README 记录构建和调试实测状态。
