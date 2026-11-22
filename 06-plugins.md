# 06 · 插件档位（profiles/web）

DSH 的「档位」= 一组按顺序叠加的插件 bundle。本机 `profiles/web/package.json` 的 `dsh.profile.bundles`：

```json
["@deepseek-ai/dsh-base","@deepseek-ai/dsh-web-app","@zseven-w/dsh-game-studio","dshmarket",
 "dsh-purge","dsh-our-free-model","dsh-openai-oauth","dsh-media-desk",
 "@local/context-meter-fallback","@feng0531/dsh-code-review","@linyanzhi/dsh-mesh",
 "@liustack/modlens","@liustack/modsearch","@michengai/dsh-skills-manager",
 "billion-context","dsh-agy-link","dsh-find-plugins","dsh-image-gen","dsh-ops",
 "dsh-plugin-openmotion-local"]
```

## 分工

| 插件 | 作用 | 注意 |
|---|---|---|
| `dsh-base` / `dsh-web-app` | 内核与 Web 界面底座 | 必装 |
| `dsh-openai-oauth` | ChatGPT 账号直连（`openai-codex` 提供方） | 模型表来自本地 `codex.exe` 的 `model/list`，不是 OpenAI 返回的；工具面一变整线程重建 |
| `dsh-our-free-model` | 免费模型通道 | 配额与可用性看服务端 |
| `billion-context` | 上下文压缩/折叠 | 压缩会重写历史，重要结论先落盘 |
| `dsh-ops` | 文件/命令工具面（`ops_*`） | 未加载的名字调不到，先 `ops_load_sdk` |
| `dshmedia-desk` | 自媒体工作台 | 见 `dsh-media-desk` 目录 |
| `@feng0531/dsh-code-review` | 代码审阅 | `review:srcmap` 依赖它 |
| `dsh-image-gen` | 文生图/改图 | 需要 key 或订阅通道 |
| `@liustack/modsearch` / `modlens` | 联网搜索 / 看图 | 本机服务被系统代理拦时先 `--noproxy` |
| `@linyanzhi/dsh-mesh` | 多机/多会话协作 | 端口冲突先看它 |
| `dshmarket` | 插件市场 | 装插件后可热载 |
| `dsh-purge` | 清理/清洗配置 | 会改配置，先备份 |
| `@michengai/dsh-skills-manager` | 技能管理 | 技能目录 `.dsh/skills` |
| `@zseven-w/dsh-game-studio` | 小游戏面板插件 | 需要 Chrome；发布需 `gh` |
| `dsh-plugin-openmotion-local` | 视频工程（MCP） | 重启后才注册 MCP 工具 |
| `@local/context-meter-fallback` | 上下文指示兜底 | 只读数字，不执行命令 |
| `dsh-agy-link` / `dsh-find-plugins` | 外部 agent 桥 / 插件检索 | 按需 |

## 装插件的正确步骤

1. `node <kernel>/lib/bin.js plugin --profile web add <包名>`
2. 检查 `package.json` 的 `dependencies` **和** `dsh.profile.bundles` 都出现；
3. `--dump-config | grep <包名>` 确认被组合进 tree；
4. 重启外壳，验证工具/面板真的出现。

> 只进 `dependencies` 没进 `bundles` = 装了但不启用，这是最常见的「装了没反应」。

## 插件与工具面的关系

- **工具面变化会重建 Codex 线程**：同一会话里临时改工具白名单（如 `ops_discover_tools`），会把挂起调用作废，表现为 `Tool "X" is unavailable`。
- 子代理别选 `openai-codex/*`，走无状态的 `模型网关-responses/codex/*`，否则容易整线程重建。

## 卸载/回滚

- 卸载同样用 `plugin remove`，不要手删 node_modules；
- 更新插件可能整档重装依赖，**不要并发更新**；
- 取消 ≠ 回滚：取消只是停止，已写入的依赖不会自动还原，需备份 `package.json` + `pnpm-lock.yaml`。
