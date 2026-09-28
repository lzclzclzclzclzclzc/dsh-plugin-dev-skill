# 来源、版本与维护

核对日期：2026-09-28。官方仓库：<https://github.com/deepseek-ai/deepseek-harness>。

本次核对官方 `master` 提交 `21638c56315ae6a2b552d6091945d3144c9af32e`（提交时间 2026-09-27）。以下固定链接使本次依据可复核；将来开发应匹配用户的安装版本，不将此快照视为永久最新版本。

## 官方阅读索引

| 任务 | 固定版本来源 |
| --- | --- |
| 第一个插件、入口和清理 | [basic/index.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/basic/index.md) |
| 最小工具示例 | [basic/tool.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/basic/tool.md) |
| Config | [basic/config.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/basic/config.md) |
| Bundle 与安装 | [basic/publish.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/basic/publish.md) |
| Profile、路径和命令行为 | [CLI reference](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/apps/cli/reference/README.md) |
| 工具完整合同 | [adding-a-tool.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/cookbook/adding-a-tool.md) |
| DSL 的真实类型与实现 | [tools/src/schema.ts](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/packages/core/tools/src/schema.ts) |
| 工具策略、PTC 和限制 | [tools/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/packages/core/tools/README.md) |
| 生命周期 | [framework/index.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/framework/index.md) |
| Service 与 inject | [framework/service.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/framework/service.md) |
| 事件模式 | [framework/events.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/framework/events.md) |
| 能力分层 | [practice/index.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/practice/index.md) |
| LLM 适配器 | [practice/llm-adapter.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/user/develop/practice/llm-adapter.md) |
| 插件形态与扩展点 | [extension-cookbook.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/cookbook/extension-cookbook.md) |
| 设置表单、Client 打包规则 | [adding-a-settings-card.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/docs/cookbook/adding-a-settings-card.md) |
| Client 加载器与 factory 格式 | [client/modules/README.md](https://github.com/deepseek-ai/deepseek-harness/blob/21638c56315ae6a2b552d6091945d3144c9af32e/packages/client/modules/README.md) |

官方文档大多有同目录 `.zh.md` 中文版。需要当前文档时把固定 commit 换为与目标版本匹配的 tag/commit；只有明确针对最新版开发时才追踪 `master`。

## 避免版本误导

- 本快照入门教程仍要求 overlay 使用绝对路径，而 CLI reference 已描述相对路径按 patch 位置解析。本 skill 用绝对路径作为稳妥示例，并要求按目标 CLI 确认相对路径行为。
- 部分教程可能仍含旧会话来源字面量。会话注入和格式迁移必须查目标版本类型与对应实现；不能因为代码出现在官方教程中就跳过类型检查。
- Web Client、presets、宿主共享依赖解析更新较快。当前接口没有出现在参考中时，读取官方源码，不把其他 agent 或社区模板的 API 拼接进去。
- 未联网时可用本 skill 完成基于快照的初稿；明确标记尚未验证的版本兼容性。

## 社区 skill 检索结果

本次对以下三个仓库的入口 skill 和许可做了评估，没有将其代码或整套流程直接复制安装：

| 候选 | 评估与本 skill 的选择 |
| --- | --- |
| [dsh-io/dsh-plugin-skill](https://github.com/dsh-io/dsh-plugin-skill)（MIT） | 工具入门简洁，但要求导出 name 匹配 patch id，且局部文字把输出 schema 说成 object；官方示例允许不同名称和 scalar 输出，因此重新依据官方合同编写。 |
| [stepupgaming/deepseek-plugin-skill](https://github.com/stepupgaming/deepseek-plugin-skill)（MIT） | 包含 vendor 文档和更多排错经验，但固定在早期提交、禁止查阅更新文档，并带有特定场景的统一限制；本 skill 采用目标版本校验，不继承这些限制。 |
| [omdsh-dev/dsh-plugin-skills](https://github.com/omdsh-dev/dsh-plugin-skills)（MIT） | `dsh-write-plugin` 主要面向官方 monorepo 包开发，包含全仓库结构与检查约定；独立插件不应默认套用。 |

这些是检索候选，不是 DeepSeek 官方发布的 Codex skill。检索结果不能证明不存在其他可用实现。

## 许可

官方源码与文档采用 MIT。此 skill 中依据官方示例改写的代码与说明保留其许可，见 [LICENSE](../LICENSE)。社区候选仅作为比较来源，没有复制其实现。
