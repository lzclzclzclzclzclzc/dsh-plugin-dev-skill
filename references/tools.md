# 工具插件：API 与验证

对应官方 `docs/user/develop/basic/tool.md`、`docs/cookbook/adding-a-tool.md` 和 `packages/core/tools/src/schema.ts`；固定版本链接见 [sources.md](sources.md)。

## 最小工具

以下是 TypeScript 入口示例；安装包需要编译产物与 bundle，参见 [packaging.md](packaging.md)。

```ts
import type { Context } from '@deepseek-ai/cordis'
import { defineTool } from '@deepseek-ai/dsh-tools'

export const name = 'greeting-plugin'
export const inject = ['tools']

export function apply(ctx: Context) {
  ctx.tools.register(defineTool({
    name: 'greet_person',
    description: 'Return a greeting for the supplied person.',
    parameters: {
      person: { type: 'string', required: true, description: 'Person to greet' },
    },
    output: {
      schema: { type: 'string' },
      render: (_args, value) => [{ type: 'text', text: value }],
    },
    async execute(args) {
      return `Hello, ${args.person}!`
    },
  }))
}
```

## 参数 DSL 与规范输出

- `parameters` 是属性名到 schema 的映射，本身不是 `{ type: 'object', properties: ... }`。其隐式根对象开放；不要在这个映射上添加 JSON Schema 根关键字。
- 必填项在对应属性上写 `required: true`；可选项省略该字段。不要写 `required: false` 或标准 JSON Schema 的 `required: ['field']`。
- 显式对象节点，包括嵌套参数和对象输出，都必须声明 `additionalProperties: true` 或 `false`。
- 支持的 DSL 字段以目标版本 `ValueSchemaSpec` 为准。核对版本支持 scalar、array、object、`json`、`oneOf` 等；不要把任意 JSON Schema 关键字直接塞进 DSL。
- DSL 的 `default` 是注释，不负责给缺失工具参数赋值。需要默认值时在执行逻辑中明确处理。这不同于插件 Schemastery `Config` 的默认值。
- `oneOf` 要求恰好匹配一个分支；区分联合优先采用互斥的 `const` 判别字段。
- `output.schema` 可以是标量、数组、对象或 null，不限于对象。`execute(args, exec)` 返回与它一致的可无损 JSON 化值，不返回 `{ content: [...] }`、`ToolResult` 或类实例。

对象输出示例：

```ts
output: {
  schema: {
    type: 'object',
    properties: {
      found: { type: 'boolean', required: true },
      count: { type: 'integer', required: true },
    },
    additionalProperties: false,
  },
  render: (_args, value) => [{ type: 'text', text: `Found ${value.count} items.` }],
},
```

上例是 definition 内的片段，不能单独当入口。领域约束如非空字符串、正数和跨字段约束，应在 DSL 无法表达时补检查。

## 生命周期、取消与并发

- 注册时借用 readonly definition；注册后不要修改 schema 或更换回调。用释放旧注册并重新注册的方式替换。
- `args` 是已验证、分离且冻结的输入；不要原地改写。`exec` 提供执行身份、调用 agent 与必需的 `signal`。
- 前台 I/O 转发 `exec.signal`，例如 `fetch(url, { signal: exec.signal })`；失败和取消应结束实际工作，而不只是结束等待。
- `timeoutMs` 是声明的协作超时预算；核对版本的 registry 本身不施加 deadline，需要相应 timeout policy wrapper。不要声称设置该字段就强制终止底层工作。
- 仅在实际允许调用重叠时声明 `isConcurrencySafe(args)`。写入共享状态、串行外设或同一资源时按真实约束判断。
- 基础设施错误抛出；成功取得但领域结果不理想的情形用规范值表达，例如进程的非零退出状态。
- 不注册保留名 `run_code`。重复工具名与可见 scope 要检查，不能靠重新注册覆盖。

## PTC、策略与展示

已注册且可见的工具在 PTC 模式中获得生成的调用接口，例如 JavaScript runtime 中的 `await tools.greet_person({ person: 'Ada' })`；它返回规范值，仍经过工具执行策略。仅 PTC 模式下未必允许同名 Native 直调，不能据此认定注册失败。

事件用途：`tools/pre-execute` 处理 allow/deny/ask；`ctx.tools.guard()` 提供后续监听器不可撤销的拒绝；`tools/execute` 包装调用；`tools/post-execute` 变换结果；`tools/result` 观察最终结果。采用前先读取当前接口与事件模式。只替换展示文本不会隐藏规范值。

`output.render` 面向模型。`presentationMeta` 可携带有界且可回放的 JSON 展示数据。Host 的 `presentCall` / `presentResult` 应是纯函数，不读文件、时间或可变会话状态；**当前 Web Client 不使用这两个 Host presenter**，其专用卡片需 Client keyed slot，见 [extensions.md](extensions.md)。

长任务只有在需求确实要求后台生命周期时才接入 `ctx.jobs`。检查目标版本的 job spec、owner、控制工具与清理语义；已发布 job 的取消归属与前台 `exec.signal` 不同。不要简单用脱离管理的 Promise 代替。

## 有效验证

验证正常输入、缺少必填项、业务边界、底层失败，以及规范输出能否通过 schema 和渲染。取消要确认底层活动停止；插件释放后工具应不可调用。项目使用 PTC 时，同时检查结构化返回值。工具可见性应在目标 agent/preset 下验证，不能只检查全局 registry。
