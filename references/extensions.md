# 配置、服务、事件、LLM 与 Web 扩展

按需读取本文件相关部分，再查目标版本接口。官方文件定位见 [sources.md](sources.md)；不要把这里的示例当作完整服务 API 清单。

## Config 与服务生命周期

```ts
import type { Context } from '@deepseek-ai/cordis'
import Schema from '@deepseek-ai/schemastery'

export interface Config {
  greeting: string
  timeoutMs: number
}

export const Config: Schema<Config> = Schema.object({
  greeting: Schema.string().default('Hello'),
  timeoutMs: Schema.number().min(1).default(10000),
})

export function apply(ctx: Context, config: Config) {
  // Register the requested capability using validated config.
}
```

Config 不能仅导出普通对象。会随部署变化的 endpoint、超时等写成字段；凭据使用目标版本已有的凭据机制，不写进可发布包。

普通配置变化可导致实例替换；声明为 volatile 的配置是另一条路径，使用当前 `Volatile<T>`、`.volatile()`、`.get()`、`loader/volatile-update` 合同。设置表单功能先阅读官方 `adding-a-settings-card.md`，不要把热更新都实现成整个插件重启。

提供命名能力可继承 `Service`，在构造器 `super(ctx, 'serviceName')`，并通过 TypeScript declaration merging 扩展 Cordis `Context`。消费方声明 `inject`。接口、实现、消费者需要独立演化时再拆包，简单功能保持单包即可。

`PENDING` 表示依赖未就绪；`LOADING` 表示正在执行加载；`FAILED` 表示加载抛错。required service 消失会卸载消费者，恢复后重新加载。`ctx.plugin()` 创建随父级释放的子作用域。

`ctx.effect()` 的清理调用按逆注册顺序开始，但多个异步 disposer 并发执行，无串行完成保证。串行依赖的清理放在同一个 async disposer 内。HMR 是否发生取决于组合中是否加载相应能力。

## 事件与策略

从相应包导入事件类型，并按目标版本接口签名写监听器。自定义事件通过 Cordis `Events` declaration merging 声明。

| 模式 | 语义 |
| --- | --- |
| `emit` | 同步广播，忽略返回值 |
| `bail` | 按顺序取第一个非 null/false/undefined 的结果 |
| `serial` | 顺序等待异步 listener，同样可以短路 |
| `waterfall` | 通过 `next()` 委托后续处理，省略调用表示主动截断 |

`tools/pre-execute` 可实现工具策略，`tools/result` 适合观察结果。按目的选接口，不把审计 listener 写成更改执行权限的 handler。

Cordis 事件与持久 Session 事件是不同系统。`turn/end`、`tool/call`、`tool/result` 等记录通过 `session/event` 观察并检查 `event.type`；不要凭记录名注册同名 Cordis 事件。

会话注入、预设、持久消息来源类型变化较快。涉及 `agent.inject()` 或 `source.kind` 时直接核对目标版本类型与迁移规则；不要照抄旧教程中的来源字面量。注入上下文与唤醒/发送用户输入的语义也要分别核实。

## LLM 适配器

先读官方 `docs/user/develop/practice/llm-adapter.md` 及 `@deepseek-ai/dsh-llm` 的目标版本类型，对照一个现有适配器。

- 通常继承 `LlmAdapter`，实现 `stream(options: GenerateOptions): AsyncIterable<StreamChunk>`，通过 `ctx.llm.registerAdapter(config.providers, adapter)` 注册 provider routes。
- 声明 `inject = ['llm']`；provider 标识与 model 标识按各自语义传递，不把所有 model 注册成 adapter。
- 每个 `block-start` 对应一个 `block-end`；保持块 index，正确累积工具调用参数；usage 在最终 finish 之前发出。
- 根据当前类型处理系统提示、消息、工具、结构化输出与推理参数。无法支持的明确请求返回稳定 `LlmError`，不静默丢字段。
- 网络调用传递 `options.signal`；按官方要求合并 `attributionHeaders()`。身份/能力查询与 `resolveModel` 也应支持取消。
- 有需要再实现 `listModels()` 与 `resolveModel()`；reasoning id 是适配器能力元数据，不发明跨 provider 通用枚举。
- 验证文本流、工具调用流、错误、取消及用户实际所需的模型特性。真实 API 测试用已有凭据与获准额度；未运行时明确说明。

## Web Client 与设置页

官方 `packages/client/modules/README.md` 和 `docs/cookbook/adding-a-settings-card.md` 是优先入口；查阅目标页面所属 Client 包，避免使用别的版本的 slot 名。

当前结构中，Web 半边需要同时声明 `dsh.client` 与 `exports['./client']`，文件清单中包含构建后的 Client 产物。Client 模块附着在使用**包裸名**的 Loader 行上；仅从包子路径挂载的行不会承载该包的浏览器半边。

`./client` 不是任意浏览器 ESM 文件：当前加载器要求 lazy-CJS factory 注册格式。官方 `packages/client/tsdown.client.ts` 的 `clientBundle` 属于仓库构建辅助，独立插件不能假装它是已发布 npm API；读取构建契约后在独立包中实现相同输出。

Client 代码仅导入浏览器可用 API，不把 Host 实现或 Node 模块打入浏览器包。当前模块系统用 `dsh.client.external` 声明平台基线以外的精确模块请求，并检查供应方与同步循环；类型导入不产生运行时请求。它不同于 Cordis 服务注入，不能仅照旧示例填写 `inject` 就认为模块依赖完整。注册所需 slots，并依据目标版本判断是否支持及需要 `immediately` 等选项。

工具专用 Web 卡片使用 `tool.call.toolview` keyed slot，基于持久事件、参数、content 和 `result.meta` 派生展示；Host 的 `presentCall`/`presentResult` 不会自动生成 Web 专用卡片。对旧日志/非法值保留通用展示，不让回放崩溃。

设置页优先使用插件 Config 的官方表单机制。需要 live 设置时验证保存后的文件、下次操作读取值、实例身份是否应保持，以及重启恢复。无效设置不应写入或改变运行值。

Web 验证至少覆盖目标 profile 启动、Client 产物加载、页面刷新、相关交互和禁用插件后移除其组件；涉及持久显示时覆盖回放。
