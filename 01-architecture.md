# 01 · 总体架构

## 六层

```
┌──────────────────────────────────────────────┐
│ L1 运行时   Electron 43 + Node 24            │  运行时/Electron/electron.exe
├──────────────────────────────────────────────┤
│ L2 外壳     主进程 main.js + 渲染层 renderer │  外壳/
├──────────────────────────────────────────────┤
│ L3 内核      @deepseek-ai/dsh + 插件 bundle  │  引擎/desktop-runtime/resources/kernel
├──────────────────────────────────────────────┤
│ L4 配置      profiles/web + AGENTS.md        │  引擎数据/.dsh/
├──────────────────────────────────────────────┤
│ L5 记忆      场景私有记忆 + 共享记忆         │  引擎数据/.dsh/场景记忆/
├──────────────────────────────────────────────┤
│ L6 产物      各场景工作区                     │  工作区/
└──────────────────────────────────────────────┘
```

## 目录地图（本机实装）

| 路径 | 作用 | 能不能手改 |
|---|---|---|
| `启动.cmd` / `启动.vbs` | 双击启动，拉起 Electron 外壳 | 可 |
| `运行时/Electron/` | Electron 二进制 | 不动 |
| `外壳/main.js` | 主进程：窗口、webview、IPC、引擎拉起与恢复 | 改前备份 |
| `外壳/renderer/` | 渲染层：场景标签、面板渲染器、webview 管理 | 改前备份 |
| `外壳/scenarios/*.json` | 场景定义（面板布局 + 会话绑定） | 可 |
| `外壳/data/scene-tab-order.json` | 标签顺序（主进程唯一权威） | 可 |
| `外壳/data/backups/` | 改内核/外壳前的备份 | 只增 |
| `外壳/scripts/*.mjs` | 外壳侧工具（建会话、刷新面板、同步索引） | 可 |
| `引擎/desktop-runtime/` | Electron 运行时 + 内核 node_modules | 禁写 |
| `引擎/engine-runtime.cjs` | 引擎进程入口 | 改前备份 |
| `引擎数据/.dsh/` | DSH_HOME：AGENTS.md、profiles、skills、storages、场景记忆 | 部分可 |
| `引擎数据/.dsh/AGENTS.md` | 全局指令，每轮注入所有会话 | 可 |
| `引擎数据/.dsh/profiles/web/` | 插件档位（package.json + cordis.patch.yml） | 可 |
| `引擎数据/.dsh/场景记忆/` | 记忆体系根 | 只能由记忆脚本写 |
| `工具/` | 跨场景脚本（派活、记忆、会话） | 可 |
| `工作区/` | 各场景产物 | 可 |
| `说明/` | 人读文档 | 可 |

## 进程模型

1. `启动.vbs` → `electron.exe 外壳`
2. 主进程 `外壳/main.js` 拉起引擎：`engine-runtime.cjs` → `dsh --profile web`
3. 引擎监听本机端口，写 token 到 `引擎数据/`（`engine-cookie`），外壳拿 token 走 RPC
4. 外壳用 `<webview>` 承载 Harness 界面（每个场景面板一个 webview，按 `partition` 隔离）
5. 关窗时先软停引擎；异常退出由 `engine-recovery.cjs` 兜底重启

关键 RPC：`session/create`、`session/prompt`、`session/state`、`session/modelCatalog`、`workspace/create`、`session/projections`。

## 数据流

```
用户在面板输入 → webview → 引擎 session/prompt → 模型
                                    ↓ 工具调用
                         插件（MCP / 内置工具）
                                    ↓
                        工作区文件 / 场景记忆 / 外部服务
```

## 为什么不是「一个 Electron 应用写完」

- 模型/工具/插件演进快，放进内核可用插件热更新，不用重打外壳；
- 外壳只管像素，崩溃不丢会话；
- 记忆与产物分开，换机器只搬 `.dsh` 与 `工作区`。
