# 03 · 场景系统

## 一个场景是什么

一份 JSON 描述「页面上摆几块面板、每块面板干什么、绑定哪个会话」。外壳读 JSON 渲染，内核只管会话本身。

```
外壳/scenarios/<id>.json       场景定义
外壳/scenarios/<id>-context.md 给人和 AI 看的定位卡
外壳/data/scene-tab-order.json 标签顺序（唯一权威）
外壳/scenarios/_场景身份索引.md 场景/面板/会话/记忆 对照表
```

## 面板类型

| type | 用途 | 关键字段 |
|---|---|---|
| `harness` | 内嵌 DSH 会话 | `sessionId`、`sessionTitle`、`pinTarget` |
| `view` | 文件/网页预览 | `filesPanel`、`homeFolders`、`quickLinks` |
| `file` | 文件浏览器 | `root`、`mode`(`local`)、`viewPanel`、`localShortcuts` |
| `embed` | 内嵌网页 | `url` |
| `neuron` | 记忆神经元 3D 视图 | 只读渲染 `场景记忆/_共享记忆` |
| `review-doc` / `review-files` | 复盘专用（文稿 + 文件对照） | `root` |
| `pc-drives` / `pc-toplarge` / `pc-junk` / `pc-ledger` | 本地巡检场景的磁盘巡检面板 | 由主进程只读采样 |

## 布局

`layout: "grid"` + `cols`，每块面板给 `col/row/w/h`。`keepAlive: true` 表示切走不销毁 webview（会话不断）。

## 会话绑定

- 面板的 `sessionId` 必须指向真实会话；空串会导致面板空白。
- `pinTarget` 是会话标题白名单，防止串台。
- 建会话：

```bash
node 外壳/scripts/建会话.mjs "<场景名>" --cwd "<工作区路径>"
```

该脚本会：注册/复用工作区 → `session/create` → `session/rename` → 把同名场景里所有 `harness` 面板的 `sessionId` 改掉 → 同步身份索引。

## 标签顺序

`外壳/data/scene-tab-order.json`：

```json
{ "version": 1, "savedAt": "...", "ids": ["harness", "command", "game-studio", "..."] }
```

主进程 `listScenarios()` 按它排序；**启动时读一次**，新建场景或改顺序都要重启外壳。

## 场景身份索引

`_场景身份索引.md` 是一张表，一行一个面板：

| 场景 id | 面板 id | 面板标题 | 会话标题 | sessionId | 定位卡 | 主技能 |

用途：

- 派活脚本按 `--场景` 查会话（多面板场景用 `--面板` 区分）；
- 记忆脚本据此判断「这个场景属于哪个会话」；
- `外壳/scripts/sync-scene-index.mjs` 只更新已有行的 3–5 列，**不会自己加行**——新场景要手写一行。

## 跨场景协作

- 派活：`node 工具/派活.mjs --场景 <名> [--面板 <id>] --文件 <工单.md>`
- 通告：`场景记忆/_共享记忆/_跨场景通知.md`
- 派单账本：`场景记忆/_共享记忆/派单台账.jsonl`

原则：场景之间不直接改对方记忆；发现错误只提醒，由所属场景自己改。
