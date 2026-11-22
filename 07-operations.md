# 07 · 日常运维

## 启动与停止

- 启动：双击 `启动.cmd`（或 `启动.vbs` 无控制台）
- 停止：关主窗口即可，主进程会软停引擎
- 异常：`外壳/engine-recovery.cjs` 会尝试拉起；连续失败看 `外壳/server.log`

## 端口与地址

| 服务 | 地址 | 说明 |
|---|---|---|
| Web GUI | `http://127.0.0.1:<GUI_PORT>` | 本机实装值，随配置变 |
| 引擎 RPC | 由 token 文件发现 | `外壳/scripts/workbench-client.mjs` 里有 `engineCookie/engineRpc` |
| 模型网关 | `http://127.0.0.1:<GATEWAY_PORT>` | 模型代理 |
| <发布台> | <SITE_PORT> | 内容发布 |

访问本机服务遇到 502：系统代理把 `127.0.0.1` 也转出去了。先 `curl --noproxy '*' http://127.0.0.1:<GATEWAY_PORT>/v1/models` 直连验证。

## 日志在哪

| 日志 | 位置 |
|---|---|
| 外壳/引擎 | `外壳/server.log`、`server.log.prev` |
| 定时任务/热载 | `外壳/scripts/热载界面.log`、`重启工作台.log` |
| 记忆审计 | `场景记忆/_共享记忆/审计/` |
| 会话本体 | `引擎数据/.dsh/storages/`（sqlite / projcache） |

## 上下文失控怎么办

症状：界面百分比不动、回复空、日志里 `413 egress window fuse`。

1. `工具/会话体检.mjs` 看 `contextBreakdown` 真实负载；
2. 超窗口就 `工具/换会话.mjs` 换新会话（换前把结论写进记忆）；
3. 别信界面上的 `contextPressure.pressureTokens`，它停在最后一次成功请求。

## 备份

- 改 `外壳/`、`工具/` 前：`外壳/data/backups/<主题>-<时间>-<文件名>`
- 改 profile 前：备份 `profiles/web/package.json` 与 `pnpm-lock.yaml`
- `引擎/` 整目录禁写（升级覆盖也是禁止理由）

## 场景/面板不生效

1. 场景 JSON 是否被解析（`node -e "JSON.parse(...)"`）；
2. `scene-tab-order.json` 里有没有这个 id；
3. 重启外壳（标签只在启动时读一次）；
4. 面板 `sessionId` 是否为空；
5. `sync-scene-index.mjs` 是否只改了旧行、没加新行。

## 定时任务与重启

- 定时任务写在 `schedule_*`，重启外壳不会丢；
- 重启会中断进行中的会话轮次，长任务先等完或写好断点；
- 重启后必须验：MCP 工具是否注册、面板是否渲染、env 是否读到（如 `GHIDRA_INSTALL_DIR`/`JAVA_HOME`）。
