# 08 · 坑与决策

## 上下文与压缩

1. **界面百分比会骗人**：413 之后不再更新，停在最后一次成功请求。判断标准是 `contextBreakdown` 或 `换会话.mjs` 的压力读数。
2. **压缩会重写历史**：重要结论先落盘（§五 / 项目记忆），否则压缩后就只剩摘要。
3. **换会话不是重启**：换会话只换 session，场景与面板不动。

## 派活

4. **`accepted:true` 不是交付**：只是引擎收下 prompt。验收看 `asOfSeq` 增长 + 台账回执 + 回读产物。
5. **目标会话可以拒答**：模型层会按策略拒绝某类任务，派单方管不到；被拒就自己干。
6. **必须用 `派活.mjs`**：`发消息.mjs` 只入库不唤醒，任务不会开始跑。

## 场景与面板

7. **标签顺序只在启动时读一次**：新建场景不重启就不出现，别怀疑 JSON 写错了。
8. **`sync-scene-index.mjs` 不加新行**：它只改已有行的 3–5 列；新场景要手写索引行。
9. **面板空白先查 sessionId**：空串/错 id 都表现为白屏。

## 插件

10. **装插件要同时进 `dependencies` 和 `dsh.profile.bundles`**，否则装了不启用。
11. **工具面变化会重建 Codex 线程**：临时改工具白名单会让挂起调用作废。
12. **OAuth 模型表来自本地 codex**：改 `model_catalog_json` 能加条目，但服务端不认就是不认（例如 ChatGPT 账号直连拒绝 `gpt-6.2-*`，返回 400 `model_not_found`）。

## 记忆

13. **§〇 必须 ≤4KB**：它是新会话的第一屏，超了等于没人读。
14. **身份卡/场景画像禁手改**：改场景 JSON 后跑 `stamp` 重新生成。
15. **§五 只追加**：结论要推翻就再写一条「更正」，不要改历史。

## 进程与环境

16. **`引擎/` 全目录禁写**：包括 node_modules；升级覆盖也不能作为理由。
17. **改源码前备份**：`外壳/data/backups/`，文件名带日期与主题。
18. **系统代理会拦本机请求**：502 先 `--noproxy '*' ` 直连验证。
19. **PowerShell 可能是 5.1**：用 PS 特有能力前先打印 `$PSVersionTable.PSVersion`。
20. **杀进程要校验归属**：按端口查 PID + 程序路径，不按进程名批量杀。

## 逆向/分析类

21. **客户端锁 ≠ 服务端锁**：Electron 应用可以关 fuse、解 asar、改 hook，界面全开；但只要会员校验在服务端（403 `MEMBERSHIP_EXPIRED`），真能力拿不到。
22. **Electron fuse 字节**：哨兵后 +32 版本、+33 长度、+34 起 8 字节；`AsarIntegrity` 与 `OnlyLoadAppFromAsar` 是改包的前提。
23. **无头验证优先 CDP**：`--remote-debugging-port` + `Runtime.evaluate`/`Page.captureScreenshot` 比 PowerShell 截屏可靠（GDI+ 在无交互桌面会失败）。
