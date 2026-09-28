---
name: dsh-plugin-dev
description: "开发、修改、调试和打包 DeepSeek Harness（DSH）插件。Use for DSH plugins, Cordis apply(ctx)/inject, defineTool, cordis.patch.yml bundles, LLM adapters, hooks, services, and DSH Web client extensions. 不用于普通 DeepSeek API 应用、模型训练或 Codex 自身插件开发。"
---

# DeepSeek Harness 插件开发

帮助用户交付能在其目标 DSH 版本与 profile 中加载的插件。默认用用户的语言交流；代码遵循目标项目的约定。

## 先确认开发边界与版本

1. 查看工作区说明、现有插件结构、`package.json` 和锁文件，区分独立插件与官方 monorepo 内的包。普通第三方插件默认独立开发；不要为它添加官方仓库专用的包布局、全仓库检查或文档要求。
2. 读取已有 CLI 的 `dsh --version`、`dsh --help`，或官方源码项目的版本与启动脚本；辨别目标是 Web、Desktop 还是自定义 profile。不要仅因目录叫 `dsh` 就认定它是官方源码。
3. API 以目标版本的 TypeScript 类型、实现和匹配版本官方文档为准。此 skill 核对于 **2026-09-28**，官方提交 **21638c56315ae6a2b552d6091945d3144c9af32e**；这不是所有用户环境的版本。涉及新接口、旧插件迁移或文档冲突时，按 [sources.md](references/sources.md) 查证，不凭记忆补接口。
4. 未安装 DSH 时仍可编写和静态检查插件。说明运行验证所缺前提；不要把静态检查说成已成功加载，也不要仅为了读取文档安装整套 Harness。

## 按能力选择扩展点

| 用户需要 | 实现路径 | 按需读取 |
| --- | --- | --- |
| 给模型增加一个可调用能力 | 函数插件，`ctx.tools.register(defineTool(...))` | [tools.md](references/tools.md) |
| 配置、监听事件、提供可替换服务 | `Config`、`ctx.on`、`Service` | [extensions.md](references/extensions.md) |
| 接入模型服务商 | `LlmAdapter`，注册到 `ctx.llm` | [extensions.md](references/extensions.md) 的 LLM 部分及官方接口 |
| Web 页面、设置页、工具卡片 | Host 与 Client 分离 | [extensions.md](references/extensions.md) 的 Web 部分及官方 Client 文档 |
| 发布可安装包、排查安装后不生效 | `dsh.bundle` + patch + 明确 profile | [packaging.md](references/packaging.md) |

能力可以组合。仅配置既有插件时，可以做配置 bundle，不必为了形式增加空的工具或服务。已有公开扩展点足够时，不改 Harness 核心循环。

## 实现约束

- 使用 `@deepseek-ai/cordis` 的上下文。函数模块导出 `apply(ctx, config?)`；需要的服务通过 `inject` 声明。仅可选的服务在使用处 `ctx.get()` 并处理缺失。
- `package.json.name` 是包名；patch 的 `name` 是模块定位符；patch 的 `id` 是配置行标识；模块导出的 `name` 是插件名称；tool 的 `name` 是模型调用名。它们用途不同，**不要求全部同名**。具体解析关系见打包参考。
- 给用户可调的选项导出同名 `Config` 类型和 Schemastery schema；验证与默认值写进 schema。工具的参数 DSL 与 Schemastery 配置 schema 是两套接口，不可混用。
- 工具执行返回 `output.schema` 对应的规范 JSON 值；`output.render` 才返回模型内容块。不要套用其他框架的 `run()`、MCP 返回信封或标准 JSON Schema 参数根结构。
- `ctx` 注册的监听器、工具和适配器跟随插件释放。自建连接或定时器用 `ctx.effect()` 返回 disposer；有顺序要求的异步清理放在一个 disposer 中依次等待。
- I/O 支持取消，初始化保持有界；缺失服务时检查组合配置，不用无限重试掩盖依赖错误。
- 使用项目已有的权限、文件系统、进程和凭据扩展点；不要为让插件运行而禁用宿主策略。开发授权不自动包含发布、生产安装或开放网络服务。

## 开发与交付

1. 先完成最小闭环：入口加载 → 注册能力 → 一次可观察调用 → 卸载清理。复用已有构建与测试工具。
2. 本地 overlay 调试优先使用明确的绝对路径；源码启动与已安装 CLI 的命令不要混用。需要持久安装时按打包参考准备 bundle，再操作用户指定的 profile。
3. 运行与改动相关的检查：构建/类型检查、关键输入输出和失败路径；涉及网络时验证取消；涉及注册时验证卸载或 HMR 不残留旧注册。
4. bundle 检查发布文件清单、入口、patch、依赖和目标 profile 的 `--dump-config`。配置合成成功不能替代真正启动与调用验证。Web 功能还要确认页面刷新、对应组件与数据回放。
5. 改动 profile、预设或持久数据时验证移除插件后宿主仍可使用；不要把包移除等同于自写文件已清理。
6. 用简短结果说明交付文件、兼容版本/目标 profile、实际通过的验证、未完成的运行验证及原因。给出准确的本地使用命令。发布仅在用户已授权时执行。

## 排错入口

| 现象 | 优先核查 |
| --- | --- |
| 已安装却没有插件 | `dsh.bundle.patch`、已发布 patch、bundle 列表、是否启动同一 profile |
| `PENDING` | 所需服务是否在当前上下文可见；不要只看包是否安装 |
| `LOADING` 不结束或 `FAILED` | `apply` 的 I/O/异常、完整启动诊断；区分挂起与失败 |
| 插件正常但模型找不到工具 | 注册名、scope/agent preset、工具权限与过滤、Native/PTC 模式 |
| 工具返回失败 | 参数 DSL、规范返回值、输出 schema、renderer、取消与底层错误 |
| 配置改了没生效 | profile/patch 覆盖顺序、整行 config 替换、HMR 是否启用 |
| 链接包存在重复服务/类型身份 | 宿主共享依赖的 peer/dev 声明、实际解析路径和兼容版本 |
| Host 正常而 Web UI 缺失 | `dsh.client`、根模块行、`./client` 产物格式、Client 注入与 slot |

官方来源、版本冲突处理及社区 skill 评估见 [sources.md](references/sources.md)。
