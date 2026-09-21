# 色裂 · ChromaSplit

> 《色裂》——一款由 AI 生成的霓虹风小游戏：切换准星颜色点掉同色方块即可消除，点错颜色会把方块一分为二，方块持续加速下落，撑得越久越好。
> ChromaSplit — an AI-generated neon arcade game: cycle your crosshair through four neon colors to erase matching blocks, but every wrong click splits a block in two as they keep accelerating downward; survive as long as you can.

一个单文件、零依赖的霓虹风网页街机小游戏。规则只有一条对立关系：用准星选中颜色点击方块，同色即消除，异色则把方块一分为二。方块持续加速下落，生成频率随存活时间上升，撑得越久越好。

## 运行

双击 `index.html` 即可，无需构建、无需服务器、无第三方库，也没有任何网络请求。建议使用较新版本的 Chrome / Edge / Firefox / Safari。

## 操作

| 操作 | 效果 |
| --- | --- |
| 鼠标滚轮 | 切换准星颜色：红 → 青 → 黄 → 紫（循环） |
| 鼠标左键 | 在准星位置发射冲击波，消除半径内的同色方块 |
| 点击异色方块 | 该方块分裂成两个小方块，连击清零 |
| 空格 | 反转（2 格充能，消耗 1 格；充能不足 1 格时无法使用） |
| 右上角 `?` | 暂停并查看规则；点按钮或遮罩空白处继续（开始界面与死亡界面同样可点击） |
| 死亡界面：`重新开始` / 空格 | 原局重置后立即重开（不再回到开始界面） |

## 玩法规则

### 生成

- 方块在画面**顶部随机 x 位置**出现，初始 x、y 轴速度均为 0。
- 生成**只受存活时间影响**：间隔从 900ms 线性缩短到 450ms，第 45 秒达到最快并保持。场上现有方块数量不影响生成频率（没有数量负反馈，也没有数量上限）。

### 下落与加速度

- 全局恒定加速度 `0.03 px/帧²`（以 60fps 为基准），方向始终向下：每下落 1 秒，y 轴速度增加 `1.8 px/帧`。
- x 轴速度**不受加速度影响**，一旦确定，大小与方向就保持不变（左右边界除外）。
- 方块碰到画面左右边缘会反弹：只翻转水平方向，速度大小不变，保证方块始终留在可点击范围内。

### 消除与分裂

- **同色点击**：消除所有落在冲击波半径内的同色方块，每次消除 `Combo + 1`。
- 冲击波半径 = `60 + Combo × 4` px，上限 `350` px。
- **异色点击**：不消除，改为分裂——边长变为原来的 `1/1.8`，生成左右两个小方块：
  - y 轴速度**完全继承**原方块，并继续受同一个加速度影响；
  - x 轴获得随机水平分速度，取值 `1.5 ~ 3.5 px/帧`，左块向左、右块向右，两块**数值相同**；
  - 分裂后连击清零。
- 只要本次点击命中了任意一个异色方块，Combo 立即归零。

### 反转（空格，2 格充能）

- 触发瞬间：场上所有方块**位置不变**、**x 轴分速度不变**，仅把 y 轴速度按原数值翻转方向、一律改为向上。
- 反转**不改变加速度**：重力依旧向下，被翻转的方块先减速、到顶点后再落回。
- 底部中央技能条共 2 格充能，每次按空格消耗 **50%（1 格）**；不足 1 格时无法使用。
- 游戏进行中技能条**持续线性恢复**：每分钟恢复 1 格（= 总进度的 50%），恢复期间物理、生成与计时照常进行；查看规则暂停时技能条不恢复。
- 技能条颜色：未充能部分填充为灰色，未充满的格子（含空格）边框为白色，正在充能的格子填充为白色；充满一格后填充与边框一起转为青色。
- 若在恢复过程中再次使用技能，已恢复的不足 1 格的进度**不会被清空**，而是平移到更低的格位继续恢复。
- 每次充满一格时，该格四周会播放一圈**白色长条光晕**（0.5 秒，如水波扩散）；消耗时有一格 0.2 秒的熄灭过渡。

### 界面提示

- 右上角 `?` 在开始界面、游戏进行中与死亡界面都可点击查看规则：游戏中点击会暂停，开始界面与死亡界面查看规则不影响计时。
- **前两次**死亡界面的右上角会出现一行「点击？查看规则」提示，并用一条弯曲箭头指向 `?` 按钮（箭头尾部连接文字末尾、箭头头部指向问号）；第三次及以后不再出现。

### 结束条件

任意方块的底边越过画布底部即游戏结束，结算显示存活时间与最高连击。左上角 HUD 实时显示 Combo、最高连击与存活秒数。

## 参数速查

| 常量 | 值 | 含义 |
| --- | --- | --- |
| `DIFFICULTY_CONFIG.maxSpawnInterval` | 900 ms | 开局生成间隔 |
| `DIFFICULTY_CONFIG.minSpawnInterval` | 450 ms | 最快生成间隔（45 秒后达到） |
| `DIFFICULTY_CONFIG.rampUpDuration` | 45 s | 难度爬坡总时长，之后封顶 |
| `BASE_ACCEL` | 0.03 px/帧² | 全局下落加速度（= 1.8 ÷ 60） |
| `SPLIT_VX_MIN` / `SPLIT_VX_MAX` | 1.5 / 3.5 px/帧 | 分裂小方块水平分速度范围 |
| `BLOCK_BASE_SIZE` | 40 px | 方块初始边长 |
| `BLAST_BASE_RADIUS` | 60 px | 冲击波基础半径 |
| `BLAST_RADIUS_PER_COMBO` | 4 px | 每点 Combo 增加的半径 |
| `BLAST_MAX_RADIUS` | 350 px | 冲击波半径上限 |
| `REVERSAL_MAX` | 2 | 反转技能条总格数（= 2 次反转） |
| `SKILL_BAR.chargeDuration` | 60000 ms | 线性充满一格的时长（= 总进度的 50% / 分钟） |
| `SKILL_BAR.rippleDuration` | 500 ms | 充满一格时白色水波光晕时长 |

四种颜色（即准星切换顺序）：`#FF0055` 红 · `#00FFCC` 青 · `#FFDD00` 黄 · `#AA66FF` 紫。

## 技术说明

- **单文件**：`index.html` 内含全部 HTML / CSS / JavaScript，无构建步骤、无依赖、无网络请求。
- **渲染**：Canvas 2D 绘制画面与特效，DOM 负责 HUD、开始 / 结束面板与玩法说明遮罩。
- **主循环**：`requestAnimationFrame`，按 `dt` 做帧率归一（`step = dt / (1000 / 60)`），60Hz 与 144Hz 屏幕下速度一致；单帧步长上限 50ms，避免切后台回来跳帧。
- **计时**：存活时间 = 当前时间 − 开局时间 − 累计暂停时长，暂停期间物理、生成与计时全部冻结。
- **特效**：冲击波为扩散光环（约 170ms 展开、520ms 淡出），分裂时原地炸开 6~10 个同色粒子（存活 500ms）。
- **技能条**：反转改为连续充能进度（0~2 格），每帧按 `dt` 线性累加，充满整格时触发 0.5s 水波光晕。

## 目录结构

```
ChromaSplit/
└── index.html    # 完整游戏（HTML + CSS + JS 单文件）
```

## English

> ChromaSplit — an AI-generated neon arcade game: cycle your crosshair through four neon colors to erase matching blocks, but every wrong click splits a block in two as they keep accelerating downward; survive as long as you can.

ChromaSplit is a single-file, dependency-free neon arcade game that runs in the browser. Just open `index.html` — no build step, no server, no third-party libraries, no network requests.

### Rules at a glance

- Blocks appear at rest at random positions along the top edge and accelerate downward at a constant `0.03 px/frame²` (about `1.8 px/frame` after one second). Horizontal speed is never affected by that acceleration.
- The spawn rate depends only on survival time: the interval shortens from 900 ms to 450 ms, peaking at 45 seconds.
- Clicking a block in your crosshair color erases it; every clear raises your combo, and the shockwave radius grows as `60 + combo × 4` px (capped at 350 px).
- Clicking the wrong color splits the block in two: each half inherits the falling speed exactly and flies off with mirrored random horizontal velocities (1.5–3.5 px/frame, equal magnitude, left and right). Your combo resets.
- Blocks bounce off the left and right edges — direction flips, speed is preserved.
- Space triggers Reverse (2 charges): every block on screen instantly flips to upward motion at the same speed while gravity keeps pulling down, so they slow, hover, and fall back. Positions and horizontal velocities are left untouched. Each press spends 50% of the charge bar, which refills linearly during play (one charge per minute) — an unrecovered cell keeps its grey fill and a white border, the filling part is drawn in white, and a fully charged cell turns cyan (fill and border). Using it mid-refill keeps the fractional progress and shifts it to the lower slot; completing a cell plays a 0.5 s white ripple around it.
- The run ends the moment any block touches the bottom of the screen.

### Controls

| Input | Effect |
| --- | --- |
| Mouse wheel | Cycle crosshair color: red → cyan → yellow → purple |
| Left click | Fire a shockwave that erases matching blocks inside the radius |
| Space | Reverse (2 charges, refilling over time) |
| `?` button (top right) | Pause and show the rules — also available on the start and game-over screens |
| `重新开始` / Space after death | Reset and restart immediately |

On the first two game-over screens a hint line "点击？查看规则" appears in the top-right corner with a curved arrow pointing at the `?` button.

### Tech

Single `index.html`, Canvas 2D plus DOM rendering, `requestAnimationFrame` loop with delta-time normalization, so physics runs at the same speed on any refresh rate.
