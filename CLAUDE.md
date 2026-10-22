# CLAUDE.md — 涂鸦街区 (Doodle District)

> 给 AI 看的项目地图（项目专属信息）。**开工前先读 `AI-WORKFLOW.md`（通用协作协议），再读这里，最后只读我指名的那一个文件。**

## 一句话

第一人称生存射击。画面是蓝色圆珠笔画在横格笔记本纸上的风格——**所有东西都用代码生成**：纸、墨线、排线、敌人、音效、8-bit 主题曲，一个资产文件都没有。单机打波次，或最多 10 人 P2P 联机（WebRTC，无服务器）。

## 怎么跑

```bash
python3 serve.py 8910     # 然后开 http://127.0.0.1:8910
docker compose up -d      # 或 nginx 静态托管，127.0.0.1:8901
```

## 硬性约束（违反即打回）

1. **纯静态站，零构建**。ES modules + `index.html` 里的 importmap。不引入 webpack/vite/npm 包。
2. **零美术/音频资产**。禁止新增图片、模型、贴图、音频文件。一切程序化。
3. **依赖只走 `vendor/`**（three.js、peerjs 已本地化），禁止任何 CDN 外链。
4. **UI 文案中文，代码注释英文**。
5. **不碰 `vendor/`**。
6. 一次只改一件事，改完说清改了什么。

## 模块地图

| 文件 | 量级 | 职责 | 什么时候改它 |
|---|---|---|---|
| `src/main.js` | 800+ | 引导、游戏状态机、波次、FFA 计分、大厅、网络消息分发、主循环 `step()` | 加玩法规则、加模式、改流程 |
| `src/enemies.js` | 800 | 敌人程序化建模 + AI + 弹幕 + 3 个 Boss | 加/改敌人、改 AI |
| `src/level.js` | 450 | 两张地图的程序化搭建、注册碰撞体、可破坏道具、出生点 | 加地图、改地形、加物件 |
| `src/player.js` | 440 | 本地玩家：移动、二段跳/墙跳/滑铲/冲刺、抓钩、生命、相机手感 | 调手感、加移动能力 |
| `src/weapons.js` | 360 | 步枪/霰弹/狙击/武士刀：视图模型、开火、弹道、弹反 | 加/调武器 |
| `src/render.js` | 260 | three.js 墨水材质 + 后处理（描边/排线/纸纹/横格线） | 改画风 |
| `src/effects.js` | 310 | 命中火花、血、爆炸、曳光、贴花 | 加特效 |
| `src/audio.js` | 320 | WebAudio 合成音效 + 程序化芯片音乐 | 加/改声音 |
| `src/net.js` | 220 | PeerJS：大厅码、快速匹配、主机迁移、消息中继 | 改联机 |
| `src/physics.js` | 160 | AABB 世界、空间哈希、扫掠碰撞+台阶、射线、视线遮挡 | 改碰撞 |
| `src/players.js` | 150 | 远端玩家插值、命中盒、`encodeLocal` | 改同步/命中判定 |
| `src/hud.js` | 120 | DOM HUD、菜单、计分板、按键提示 | 改界面 |
| `src/input.js` | 130 | 键鼠 + 手柄、指针锁定、按键重绑定 | 改操作 |
| `src/nav.js` | 110 | 导航网格（敌人寻路） | 改寻路 |
| `src/util.js` | 60 | 数学、`Spring`、`Cooldown`、四元数工具 | 通用工具 |

## 架构要点

**单一 `ctx` 上下文**——`main.js` 里创建 `{scene, camera, world, level, nav, input, hud, effects, audio, renderer}`，并把跨模块回调挂上去（`ctx.hitPlayer`、`ctx.canHurt`、`ctx.playersInArc`、`ctx.raycastPlayers`）。各模块只依赖 `ctx`，**不互相 import**，因此没有循环依赖。加跨模块能力时：往 `ctx` 上挂，不要新增 import。

**权威模型**：主机记分，每人跑自己的身体。命中方自己判定伤害，再发 `pdmg` / `parry` 给对方。没有服务器，没有状态回滚。

**渲染管线**：场景 → G-buffer（明暗 + ink id + 视空间法线 + float 深度）→ 单个 post pass 做逆深度 Laplacian 描边、跟随表面的排线、纸纹、横格线与红色边距。**抖动必须是静态噪声**（用世界坐标/顶点做种子），用 `Math.random()` 会逐帧闪烁。

**物理**：`World` 是 AABB 世界，`cell=8` 空间哈希。`addBox(min, max, data)` → `finalize()`。角色用 `makeBody()` 扫掠移动 + 台阶（stepHeight 0.55）。`data.noGrapple` / `data.noShoot` 是碰撞体上的标记位。

**网络**：`PREFIX` 本地 `doodledev-`、线上 `doodledistrict-`，5 位大厅码。消息类型：`welcome` / `refused` / `stay` / `lobby` / `score` / `pdmg` / `parry` / `backtolobby`。非主机发消息带 `relay` 由主机转发。主机掉线时最早加入的人按 generation 接管。

**状态机**：`game.state` ∈ `start` / `play` / `dying` / `dead` / `pause` / `lobby` / `over`；另有 `screen` 表示当前菜单页。

**存档**：localStorage `doodle_map` / `doodle_best` / `doodle_music` / `doodle_checkpoint` / `doodle_name` / `doodle_sens` / `doodle_invert`。

**调试**：`window.__game` 暴露了 `ctx, game, player, enemies, nav, world, level, hud, effects, input, net, remote, lobby, scores`，控制台直接调。

**地图**：`level.js` 导出 `LEVELS`（`{key, name, blurb}`）和 `buildLevel(scene, world, key, {arena})`。`arena: true` 是联机的开阔版。换图走 `main.js` 的 `setLevel()`，它会销毁旧网格、`world.clear()`、重建 nav。

## 主循环顺序（`step()`）

`input` → 玩家更新 → 敌人 AI → 弹丸 → 波次/计分 → 网络 → 特效 → `level.animated` → `audio.setListener` → HUD 全量刷新 → `R.render()`。时间是 `dt` 缩放过的（有 hitstop 慢放）。

## 代码风格（照抄现有代码）

- 2 空格缩进，分号，ES modules。
- 文件第一行写一句英文注释说明这个文件干什么。
- 常量集中在文件顶部，一行一组：`const G = 26, WALK = 6.6, SPRINT = 10.6;`
- 模块级临时向量 `_v, _v2, _v3`，热路径里**不要** `new THREE.Vector3()`。
- 密集风格：相关赋值用分号串在同一行，短函数一行写完。
- 注释解释"为什么"，不是"做了什么"。
- 数值提常量 → 方便后面只改数字调平衡。

## 常见任务 → 改哪里

| 我想… | 去哪 |
|---|---|
| 调武器伤害/射速/后坐力 | `weapons.js` 顶部 `GUNS` 表 + 各类 `Gun` 子类 |
| 加一把枪 | `weapons.js` 加 `class X extends Gun`，`player.js` 构造函数里塞进 `this.weapons` |
| 加一种敌人 | `enemies.js`：`build*` 建模 + 在 `EnemyManager` 注册表里加一项，AI 走 `nav.js` |
| 加一张地图 | `level.js`：`buildX()` + 加进 `LEVELS`；音乐在 `audio.js` 加 tune |
| 改移动手感 | `player.js` 顶部常量组 + `update()` |
| 改画风 | `render.js` 的 `postFrag`（不要逐个改材质） |
| 改 UI 文案/菜单 | `hud.js` 的 HTML 模板 + `style.css` |
| 加音效 | `audio.js` 加合成方法（禁止音频文件） |
| 改联机逻辑 | `net.js` 消息表 + `main.js` 里对应的 handler |

## 协作方式

通用流程见 **[AI-WORKFLOW.md](./AI-WORKFLOW.md)**（四段式需求模板、里程碑切法、修 bug 流程、提交规范、反模式表）。
那份文件是跨项目通用的；这一份只描述本项目。

本项目的两条补充：

1. **指名文件**，不要让我搜全库。"改 `src/weapons.js` 的 `GUNS` 表"。
2. 一条 commit = 一个玩家能感知的行为变化，一句话英文描述：

```
Grapple spam costs breath, falls cost a kill, faster grenades, 15-minute clock
Press any button to respawn; only a short spawn shield
Kick idle players; a lobby with nobody active shuts down
```
