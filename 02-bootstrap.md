# 02 · 从零搭建

目标：一台干净的 Windows，从零到「双击启动出现场景标签 + 能和 AI 对话 + 记忆能落盘」。

## 0. 前置

| 依赖 | 版本 | 装哪 |
|---|---|---|
| Node.js | 24.x（本机 v24.22.0） | `<NODE_HOME>/`，加入 PATH |
| Git | 任意 | 用于插件与仓库 |
| PowerShell | 7.x 优先 | 5.1 也能跑，注意 `-UseBasicParsing` |
| 磁盘 | ≥ 20 GB | 工作区与模型缓存会涨得很快 |

## 1. 目录骨架

```
<WORKBENCH>/
├─ 启动.cmd / 启动.vbs
├─ 外壳/            Electron 应用（main.js + renderer/ + scenarios/ + scripts/）
├─ 引擎/            desktop-runtime（Electron + 内核 node_modules）、engine-runtime.cjs
├─ 引擎数据/        DSH_HOME → .dsh/（AGENTS.md、profiles、skills、storages、场景记忆）
├─ 工具/            跨场景脚本
├─ 工作区/          各场景产物
├─ 说明/            文档
└─ 运行时/Electron/ Electron 二进制
```

## 2. 装 Electron 运行时与内核

```bash
# 运行时（固定版本，别自动升级）
npm i -g electron@43.0.0 --prefix "<NODE_HOME>/"
# 内核（装进引擎目录，供 profile 解析）
cd "<WORKBENCH>/引擎/desktop-runtime/resources/kernel"
npm init -y
npm i @deepseek-ai/dsh@0.2.2-rc.3
```

验收：`node "<kernel>/node_modules/@deepseek-ai/dsh/lib/bin.js" --version`。

## 3. 建外壳

`外壳/package.json` 最小形态：

```json
{
  "name": "dsh-workbench",
  "version": "0.2.2-rc.3",
  "main": "main.js",
  "scripts": { "start": "electron ." },
  "devDependencies": { "electron": "43.4.0" }
}
```

主进程要做四件事：

1. `BrowserWindow`（`webviewTag: true`）+ 启动页
2. 拉起引擎：`spawn(process.execPath, [engineRuntime])`，等 token 文件出现再放行
3. 注册 IPC：`listScenarios`、`harness:url`、`file:*`、`app:*`
4. 关窗时软停引擎；异常退出走 recovery

## 4. 装插件档位

```bash
DSH_HOME="<WORKBENCH>/引擎数据/.dsh" node "<kernel>/lib/bin.js" plugin --profile web add dsh-ops
```

最少要有的四类：

| 类别 | 例子 | 没有会怎样 |
|---|---|---|
| 模型接入 | `dsh-openai-oauth`、`dsh-our-free-model` | 没有可选模型 |
| 上下文 | `billion-context` | 长对话必炸 |
| 工具面 | `dsh-ops`、`@liustack/modsearch` | 只能空谈，动不了文件 |
| 市场/管理 | `dshmarket`、`dsh-purge` | 后续插件不好装 |

装完检查：`node bin.js --profile web --dump-config | grep -i <插件名>`。

## 5. 建场景

每个场景一个 JSON，最小可用：

```json
{
  "id": "command",
  "name": "demo",
  "layout": "grid",
  "theme": "dark",
  "cols": 4,
  "keepAlive": true,
  "panels": [
    { "id": "cmd-chat", "title": "AI·demo", "type": "harness",
      "col": 1, "row": 1, "w": 4, "h": 5,
      "sessionTitle": "demo", "sessionId": "session-xxx", "pinTarget": ["demo"] }
  ]
}
```

再建会话并绑定：

```bash
node 外壳/scripts/建会话.mjs "demo" --cwd "<WORKBENCH>/工作区/demo"
```

## 6. 让记忆能落盘

```bash
node 工具/场景记忆.mjs init  --场景 demo     # 生成骨架 + 身份卡 + 场景画像
node 工具/场景记忆.mjs stamp --场景 demo     # 场景 JSON 改完后重盖指纹
node 工具/场景记忆.mjs check --场景 demo     # 体检（§〇 ≤4KB、核心职责 ≥3 条等）
node 工具/场景记忆.mjs 交接   --场景 demo     # 归档旧版 + 重建索引 + 审计
```

## 7. 验收清单

- [ ] 双击 `启动.cmd` 出现主窗口与场景标签
- [ ] 标签顺序与 `外壳/data/scene-tab-order.json` 一致
- [ ] 点场景能进面板，AI 会话能回话
- [ ] `session/modelCatalog` 能列出至少一个可用模型
- [ ] 让 AI 用工具写一个文件，产物落在 `工作区/<场景>/` 下
- [ ] `场景记忆.mjs check` 全绿
- [ ] 关掉再开，会话与场景绑定还在

## 8. 常见第一次失败

| 现象 | 原因 |
|---|---|
| 白屏 | 引擎没就绪就加载 webview；等 token 再放行 |
| 场景标签少一个 | 标签顺序只在启动时读一次，新建场景要重启 |
| 会话面板空白 | `sessionId` 是空串或未绑定；跑一遍 `建会话.mjs` |
| 插件装了没生效 | 只写进 `dependencies`，没进 `dsh.profile.bundles` |
