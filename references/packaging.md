# Bundle、profile 与开发加载

以目标版本官方 `docs/user/develop/basic/publish.md` 和 `apps/cli/reference/README.md` 为依据；固定链接见 [sources.md](sources.md)。

## 明确名称各自的含义

| 字段 | 含义与关联 |
| --- | --- |
| `package.json.name` | npm 包名，如 `dsh-greeting` |
| patch 行 `name` | Node 可解析的包名/导出子路径，或本地入口路径；包根入口可写 `dsh-greeting` |
| patch 行 `id` | 定位和覆盖该配置行，如 `greeting-main`；后续 override 针对它 |
| 模块 `export const name` | Cordis 插件名称，如 `greeting-plugin` |
| `defineTool({ name })` | 模型工具调用名，如 `greet_person` |

这些标识可以不同。确保模块定位符实际解析到需要的入口，行 id 在目标结构中可明确定位，工具名在其注册层不冲突。

## 独立包形状

不依赖任何宿主服务的可加载 JavaScript bundle 示例：

```text
dsh-greeting/
  package.json
  cordis.patch.yml
  index.js
```

`package.json`：

```json
{
  "name": "dsh-greeting",
  "version": "0.1.0",
  "type": "module",
  "main": "./index.js",
  "exports": { ".": "./index.js" },
  "files": ["index.js", "cordis.patch.yml"],
  "dsh": { "bundle": { "patch": "./cordis.patch.yml" } }
}
```

`index.js`：

```js
export const name = 'greeting-plugin'

export function apply() {
  console.log('[greeting-plugin] loaded')
}
```

`cordis.patch.yml`：

```yaml
- insert:
    - id: greeting-main
      name: dsh-greeting
```

此例只证明加载；需要模型工具时替换入口并添加相应依赖，不把日志插件当作已实现的业务功能。

## TypeScript 与依赖

- TypeScript 独立包通常用 `src/index.ts` 编译到 `lib/index.js`，把 `main`/`exports`/`types` 指向实际产物，将 `lib` 和 patch 纳入 `files`。采用项目现有构建工具；无需复制官方 monorepo 的全部配置。
- 必须与宿主共享实例的 DSH 包，应在 `peerDependencies` 和 `devDependencies` 中声明兼容范围，开发副本用于类型检查/测试；第三方依赖与无状态工具依用途放 `dependencies`。不要猜版本或统一使用 `*`，也不要默认添加社区 skill 的旧依赖覆盖规则。
- 链接开发下实际模块解析还受 Node 祖先查找顺序和较近物理安装影响。peer 声明不是版本校验器，也不能保证任何位置都绕过物理副本。出现重复服务身份时检查实际导入路径。
- 发包前运行已有构建和 `npm pack --dry-run`（或对应包管理器清单检查），确认入口、声明文件、patch、Client 资源确实包含在包内。
- 没有 `dsh.bundle` 的包可以被安装为普通依赖，但不会激活插件配置层。

## overlay 调试

本地 overlay 可将 patch 的 `name` 改为实际绝对入口路径。Windows 使用单引号 YAML 字符串和正斜杠，避免转义问题：

```yaml
- insert:
    - id: greeting-dev
      name: 'C:/work/dsh-greeting/lib/index.js'
```

已有 CLI：

```sh
dsh web --patch ./dev.cordis.yml --dump-config
dsh web --patch ./dev.cordis.yml
```

官方源码项目在完成其构建后使用 `pnpm dsh web --patch ./dev.cordis.yml`。不要假定安装版 CLI 可以直接执行任意 TypeScript 源文件。

绝对路径是跨版本稳妥的本地示例。早期入门文档要求绝对路径，而本次核对的 CLI reference 说明插入行的相对模块路径按 patch 文件旁解析；不要把前者提升为所有版本的永久限制，遇到差异用目标版本 CLI/实现确认。

## 持久安装与 profile

从插件目录执行以下命令时，将 `demo` 换为用户的目标 profile：

```sh
dsh plugin --profile demo add .
dsh --profile demo --dump-config
```

`add .` 的本地路径相对于调用目录。安装成功应同时检查：profile 依赖存在，包的 bundle 在 `dsh.profile.bundles` 中，dump 中出现目标配置行。由 CLI 管理 profile manifest，不直接生成或覆盖用户的整个 manifest。

`dsh web` 是启动 `web` profile 的简写；安装到 `demo` 并不影响 Web。任意新自定义 profile 的插件管理初始化通常只带 base，并不会自动包含 Web UI。若确需隔离的 Web 开发 profile，可在**目标目录尚不存在**时用当前 CLI 支持的：

```sh
dsh --profile greeting-dev --from-default-profile web --dump-config
dsh plugin --profile greeting-dev add .
dsh --profile greeting-dev
```

先核对目标 CLI 的 flag 支持。创建成功后不要重复 `--from-default-profile`；初始化后的失败重试也应省略它。Desktop 由其应用运行环境管理，不把它当普通 CLI Web profile 启动。

普通 `--dump-config` 会初始化缺失 profile，不能称为完全无写入的读取；`--dump-config-schema` 还可能导入模块，不当作无执行静态检查。

配置覆盖顺序：bundle 列表顺序 → profile 自身 patch → `$DSH_HOME/cordis.patch.yml` → CLI `--patch` 顺序。后层覆盖前层；命中行的 `config` 被整体替换，非递归合并。HMR 未启用时要重启才应用改动。

## 分发与移除

- Git 安装通常拉取源码，单有 `build` script 不会自动产生输出。需要自足的 `prepare` 或已构建产物；构建不能依赖未发布的兄弟仓库。
- pnpm 新版本会限制依赖构建脚本。需要时依据实际诊断，仅授权明确的包键并固定可信提交；不要全局关闭构建限制。
- npm 发布包或 `pnpm pack` tarball 可预先包含构建产物，避免安装时构建。
- 先准备并验证包；`publish`、推送、用户 profile 安装等操作按当前用户授权范围执行。
- `dsh plugin --profile demo remove dsh-greeting` 同时移除依赖与 bundle 层，不保证删除插件自行写入的配置、预设或数据。涉及此类状态时验证卸载后的原会话/宿主可继续使用。
