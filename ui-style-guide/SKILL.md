---
name: ui-style-guide
description: Apple Liquid Glass（暗色）界面风格规范。用于 E:\webwork\vibe（Three.js 手势粒子 + 控制台）与 E:\webwork\web_work（课堂作业索引站）的 UI 设计/实现与审查。当用户要求实现、修改、审查暗色玻璃拟态 / Apple 风 UI，或提及液态玻璃、折射亮环、镜面高光、iOS 蓝强调色、连续圆角、8pt 栅格时触发。
---

# 界面风格规范 · Apple Liquid Glass（暗色）

> 适用范围：`E:\webwork\vibe`（Three.js 手势粒子 + 控制台）与 `E:\webwork\web_work`（课堂作业索引站）。
> 视觉目标：**iOS 26 Liquid Glass**。玻璃是「材质」不是「色块」——层次靠**折射、镜面高光、发丝边、中性阴影**拉开，而不是靠发光和彩色渐变。
> 本文件替代旧版「暗色科技风 · 玻璃拟态」规范（蓝紫青渐变 + 彩色发光），两者是相反方向。

---

## 0. 设计系统覆盖声明（Brand Override）

本规范以 **Apple Human Interface Guidelines / Liquid Glass** 为真实设计系统约束，因此**显式覆盖** `ui-design` skill 中的以下通用禁令：

| 通用禁令 | 本项目处理 | 理由 |
|---|---|---|
| 禁止 `system-ui` / `-apple-system` 字体 | **允许且强制** | SF Pro 是 Apple 设计系统的本体，无法替代为第三方字体 |
| 禁止紫/靛/紫罗兰渐变 | **禁止保留** | 苹果风只保留单一系统蓝 `#0A84FF`，旧规范的粉紫 `#ff9ad8` 全面移除 |
| 禁止居中对称布局 | 分场景 | Apple 官网式 hero 允许居中；控制台/列表沿用 Apple 左对齐 + 8pt 栅格 |
| 禁止 emoji 图标 | **保留禁令** | 统一使用 Lucide / Heroicons（SF Symbols 的 Web 近似） |

覆盖范围仅限上述四项，其余禁令（无 emoji 图标、明确美学方向、先出设计规范后写代码）依然生效。

---

## 1. 设计调性（Design Tone）

**Apple HIG 三原则：**

- **Clarity 清晰**：文字在任何材质上都保持高对比；层级靠字号/字重/灰度，不靠颜色数量。
- **Deference 遵从**：界面退让于内容。装饰（阴影、边框、高光）只服务于层级，不喧宾夺主。
- **Depth 纵深**：玻璃材质堆叠出真实物理层次——越靠前的材质越亮、模糊越强、阴影越深。

**关键词**：Liquid Glass（液态玻璃）、Specular highlight（镜面高光）、Refraction edge（折射亮环）、Hairline（发丝边）、Continuous corner（连续圆角）、Neutral shadow（中性阴影）、Spring motion（弹簧动效）。

**反模式（立即停止）**：
- ❌ 彩色 `box-shadow` 发光（如 `rgba(80,140,255,.65)`）
- ❌ 彩虹渐变标题 / clip-text 渐变文字
- ❌ 三团偏心彩色光晕（蓝 + 紫 + 青）
- ❌ 全大写 + 字距 `.14em` 的科技徽章
- ❌ 纯黑实体面板（无模糊、无材质）
- ❌ 生硬线性过渡（`linear` 或无缓动）

---

## 2. 背景：壁纸层（Wallpaper Layer）

> Liquid Glass 的折射必须建立在**有内容的背景**上。纯黑底上的玻璃是死的。这是与旧规范最本质的差别。

| 层 | 规则 |
|---|---|
| 基色 | `#000000`（Apple 深色系统背景就是纯黑） |
| 壁纸渐变 | 2–3 团**极低饱和**深色氛围，明度 ≤ `L 22%`，例：`#16233a`、`#1d1a2e`、`#101c1e` |
| 尺度 | 大半径 `radial-gradient(120% 90% at 20% 0%)`，`background-attachment: fixed` |
| 颗粒 | 叠加 2–4% 不透明度噪点，消除渐变色带（banding） |
| 禁止 | 高饱和色块、青/粉/紫亮色光晕、多色同时出现（最多 2 个低饱和色相） |

```css
.wallpaper{
  position:fixed; inset:0; z-index:-1; pointer-events:none;
  background:
    radial-gradient(120% 90% at 18% -10%, #16233a 0%, transparent 58%),
    radial-gradient(100% 80% at 92% 18%,  #1d1a2e 0%, transparent 62%),
    radial-gradient(120% 100% at 50% 112%, #101c1e 0%, transparent 60%),
    #000;
}
.wallpaper::after{                 /* 颗粒 */
  content:""; position:absolute; inset:0; opacity:.035;
  background-image:url("data:image/svg+xml;utf8,\
<svg xmlns='http://www.w3.org/2000/svg' width='160' height='160'>\
<filter id='n'><feTurbulence type='fractalNoise' baseFrequency='.8' numOctaves='3'/></filter>\
<rect width='160' height='160' filter='url(%23n)'/></svg>");
}
```

---

## 3. 色彩系统（Color）

### 3.1 材质填充（Materials）

Apple 深色材质在 Web 上的近似：**白透明叠层 + backdrop-filter**，四档厚度。

| 材质 | 填充 | backdrop-filter | 用途 |
|---|---|---|---|
| Ultra Thin | `rgba(255,255,255,.04)` | `blur(20px) saturate(160%)` | 大卡内层、列表分组条 |
| Thin | `rgba(255,255,255,.06)` | `blur(24px) saturate(170%)` | 卡片 |
| **Regular（默认）** | `rgba(255,255,255,.09)` | `blur(30px) saturate(180%)` | 面板、弹层 |
| Thick | `rgba(255,255,255,.13)` | `blur(40px) saturate(190%)` | 导航栏、Tab Bar、Sheet |
| Bar / Chrome | `rgba(22,22,24,.72)` | `blur(30px) saturate(180%)` | 顶栏、底栏（需更强遮挡时） |

> 所有材质统一叠加顶部镜面渐变：`linear-gradient(180deg, rgba(255,255,255,.10), rgba(255,255,255,.035))`，让上沿更亮、下沿更沉。

### 3.2 系统灰阶（深色）

| Token | 值 | 用途 |
|---|---|---|
| `--bg` | `#000000` | 页面底 |
| `--bg-2` | `#1C1C1E` | 分组底、输入框填充 |
| `--bg-3` | `#2C2C2E` | 三级容器、按压态 |
| `--fill` | `rgba(120,120,128,.36)` | 默认填充（chip、轨道底） |
| `--fill-2` | `rgba(120,120,128,.24)` | 次级填充 |
| `--separator` | `rgba(84,84,88,.65)` | 发丝分隔线 |

### 3.3 文字层级

| Token | 值 | 对应 iOS |
|---|---|---|
| `--ink` | `rgba(255,255,255,.92)` | label |
| `--ink-2` | `rgba(235,235,245,.60)` | secondaryLabel |
| `--ink-3` | `rgba(235,235,245,.30)` | tertiaryLabel |
| `--ink-4` | `rgba(235,235,245,.18)` | quaternaryLabel |
| 反色 | `#FFFFFF` on `#0A84FF` | 强调按钮文字 |

### 3.4 强调色与语义色（Apple System Colors · Dark）

**主强调色只有一个：iOS 蓝。**

| 语义 | 值 | 用途 |
|---|---|---|
| Accent / Blue | `#0A84FF`（按压 `#0A7BEA`） | 主按钮、链接、选中态、激活滑块 |
| Green | `#30D158` | 成功、开关开启 |
| Red | `#FF453A` | 录制中、危险、删除 |
| Orange | `#FF9F0A` | 警告 |
| Yellow | `#FFD60A` | 提示 |
| Teal | `#40C8E0` | 次要数据高亮（vibe 状态点可用） |

**规则**：一屏之内强调色只出现 1–2 处（主行动点 + 当前选中）。状态指示点用「单色 + 极轻中性阴影」，**不用彩色外发光**。

---

## 4. 玻璃材质规范（核心）

### 4.1 三段式构成

一张 Liquid Glass 表面 = **折射亮环 + 镜面高光 + 中性投影**，缺一不可。

```css
.glass{
  position:relative;
  border-radius:var(--r-card);

  /* ① 材质本体 */
  background:
    linear-gradient(180deg, rgba(255,255,255,.10), rgba(255,255,255,.035)) ,
    var(--glass-fill);                       /* 见 §3.1 */
  backdrop-filter:blur(30px) saturate(180%);
  -webkit-backdrop-filter:blur(30px) saturate(180%);
  border:.5px solid rgba(255,255,255,.14);

  /* ② 发丝内高光：上沿亮、下沿暗（模拟厚度） */
  box-shadow:
    inset 0  .5px 0 rgba(255,255,255,.26),
    inset 0 -.5px 0 rgba(255,255,255,.06),
    0 10px 30px -10px rgba(0,0,0,.55),       /* ③ 中性投影，无彩色 */
    0  2px  8px  -4px rgba(0,0,0,.35);
}

/* 折射亮环：边缘一圈由亮到暗的渐变描边 */
.glass::before{
  content:""; position:absolute; inset:0; padding:.5px; border-radius:inherit;
  pointer-events:none;
  background:linear-gradient(180deg,
    rgba(255,255,255,.38), rgba(255,255,255,.08) 38%,
    rgba(255,255,255,.02) 68%, rgba(255,255,255,.20));
  -webkit-mask:linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
          mask:linear-gradient(#000 0 0) content-box, linear-gradient(#000 0 0);
  -webkit-mask-composite:xor; mask-composite:exclude;
}

/* 镜面高光：顶部弧形反光 */
.glass::after{
  content:""; position:absolute; inset:0; border-radius:inherit; pointer-events:none;
  background:radial-gradient(140% 70% at 50% -32%, rgba(255,255,255,.20), transparent 55%);
}
```

### 4.2 连续圆角（Squircle）

Apple 的圆角是超椭圆（连续曲率），不是圆弧。CSS 原生 `border-radius` 会有"四角发尖"的廉价感。

```css
:root{ --r-card:16px; }
@supports (corner-shape: superellipse(2)){
  .glass,.card,.btn,.sheet{ corner-shape:superellipse(2); }   /* 渐进增强 */
}
```
不支持时自动降级为 `border-radius`，可接受；对 hero / 大弹层等固定尺寸元素，可用 SVG 超椭圆 `mask` 精确还原。

### 4.3 降级与兼容

```css
@supports not ((backdrop-filter:blur(1px)) or (-webkit-backdrop-filter:blur(1px))){
  .glass{ background:rgba(28,28,30,.92); }   /* 无模糊时提高不透明度保证文字对比 */
}
```

---

## 5. 排版（Typography）

**字体栈（SF Pro 优先，中文回退苹方）：**

```css
--font: -apple-system, BlinkMacSystemFont, "SF Pro Text", "SF Pro Display",
        "Helvetica Neue", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif;
--mono: ui-monospace, "SF Mono", "JetBrains Mono", Menlo, Consolas, monospace;
```

**iOS 字号阶梯**（Apple 用「语义样式」而非裸字号）：

| 样式 | 字号 / 字重 | 字距 | 用途 |
|---|---|---|---|
| Large Title | 34px / 700 | `-.022em` | 页面主标题 |
| Title 1 | 28px / 700 | `-.018em` | 分区标题 |
| Title 2 | 22px / 700 | `-.016em` | 卡片组标题 |
| Title 3 | 20px / 600 | `-.014em` | 面板标题 |
| Headline | 17px / 600 | `-.012em` | 强调正文、按钮 |
| Body | 17px / 400 | `-.008em` | 正文 |
| Callout | 16px / 400 | `-.006em` | 次级正文 |
| Subhead | 15px / 400 | `-.006em` | 说明、副标题 |
| Footnote | 13px / 400 | `0` | 注释、元信息 |
| Caption | 12px / 400 | `0` | 最小标注 |

**规则**：
- 字号越大字距越紧（负字距），这是 Apple 排版的辨识度来源。
- **禁止渐变文字**，标题用纯白或 `--ink`。
- **禁止全大写宽字距徽章**；改用 13px Footnote + `--ink-2` + 胶囊浅底。
- 等宽字体仅用于数值、路径、代码（vibe 控制台的帧率/坐标）。

---

## 6. 形状与间距（Shape & Spacing）

| 维度 | 取值 |
|---|---|
| **间距栅格** | **8pt**：4 / 8 / 12 / 16 / 24 / 32 / 48 |
| 圆角 · 按钮 | 8–12px（胶囊按钮 = `height/2`） |
| 圆角 · 卡片 | 14–18px |
| 圆角 · 面板/弹层 | 22–28px |
| 圆角 · 图标容器 | 8–10px（或圆形） |
| 控件高度 | 输入框/按钮 36–44px，iOS 触控目标 **≥ 44×44** |
| 分隔线 | **0.5px** 发丝线，与文字左对齐内缩（Apple list inset） |
| 容器宽 | `min(1140px, 92vw)`（桌面站点沿用） |
| 网格 | `repeat(auto-fill, minmax(272px, 1fr))`，gap **16px**（8 的倍数） |
| 卡片内边距 | 16–20px |

---

## 7. 组件规范（Components）

### 7.1 卡片 `.card`
玻璃 Regular 材质 + 发丝边 + 16px 圆角 + 内边距 18px。
**hover**：`transform: scale(1.015)` + 顶部高光位移 + 阴影加深 —— **不是** `translateY(-5px)` + 彩色发光。
**press**：`scale(.985)`。

### 7.2 按钮（三档，Apple 官方分层）

| 类型 | 样式 |
|---|---|
| **Filled** | 实底 `#0A84FF`，白字 17px/600，圆角 12px，无阴影；按压 `brightness(.92)` |
| **Tinted** | `rgba(10,132,255,.18)` 填充 + `blur(20px) saturate(180%)`，`#0A84FF` 文字 |
| **Glass / Plain** | 玻璃材质（发丝边 + 镜面高光），`--ink` 文字；Plain 无底无框，仅强调色文字 |

胶囊按钮（Segmented / Toolbar）用 `border-radius: 999px`，高度 32–36px。

### 7.3 分段控件 `.segmented`
胶囊玻璃底（`--fill`）+ 内部滑块 `background:#0A84FF`（或玻璃白 `.14`）+ 滑块切换用 spring 曲线；每项 44px 高、等宽。

### 7.4 开关 `.switch`
51×31px，轨道 `rgba(120,120,128,.36)` → 开启 `#30D158`；thumb 28px 纯白 + `0 1px 3px rgba(0,0,0,.3)`；切换 thumb 位移 `.25s` spring。

### 7.5 滑块 `.slider`
轨道 4px 胶囊（`--fill` 底 + `#0A84FF` 进度）；thumb 28px 白色圆 + 轻阴影 + `inset 0 .5px 0 #fff`。**无外发光。**

### 7.6 列表 `.list`（Inset Grouped）
分组为圆角玻璃卡（14px），组内行间用 0.5px 发丝线内缩分隔，行高 44px，左文右值（值用 `--ink-2`）。这是 Apple 设置页的标志性形态，**优先于散落的独立卡片**。

### 7.7 标签 `.chip`（替代旧 `.badge`）
13px Footnote、`--ink-2`、填充 `--fill-2`、胶囊圆角、无字距、无描边。

### 7.8 导航栏 / Tab Bar
Thick 或 Bar 材质 + 底部 0.5px 发丝线（无边框时靠材质边界区分）；高度 44 / 49px；需处理 `env(safe-area-inset-bottom)`。

### 7.9 弹层 `.sheet`
22–28px 圆角（上方两角）、Thick 材质、顶部中央 36×5px 抓取器 `--ink-3`；从底部上推 `.5s` `--ease-sheet`。

### 7.10 图标
统一 **Lucide**（或 Heroicons）单一库，`stroke-width: 1.75`，尺寸 17px（正文级）/ 22px（标题级）。对齐 SF Symbols 的语义：线宽一致、圆头端点、无填充混用。

---

## 8. 动效（Motion）

| Token | 值 | 用途 |
|---|---|---|
| `--ease-sheet` | `cubic-bezier(.32,.72,0,1)` | Sheet / 大转场（Apple 官方曲线） |
| `--ease-std` | `cubic-bezier(.4,0,.2,1)` | 通用进出 |
| `--ease-out` | `cubic-bezier(0,0,.2,1)` | 入场、展开 |
| `--ease-in` | `cubic-bezier(.4,0,1,1)` | 退场、收起 |

- **时长**：微交互 `.15–.25s`；控件状态 `.25s`；转场 `.35–.5s`；Sheet `.5s`。
- **hover**：`scale(1.015)` + 高光位移（用背景 `background-position` 或 `::after` 的 `opacity`），`.25s var(--ease-out)`。
- **press**：`scale(.97)` + `brightness(.94)`，`.12s`。
- **状态点**：不再做 `opacity 1→.35` 呼吸闪烁；改为静态单色点，仅在活跃时用 **1.6s spring 缩放** `scale(1)→scale(1.08)` 的极轻脉动。
- **入场**：整屏 `opacity` 淡出 `.6s var(--ease-out)`（保留旧规范做法，但去掉 `visibility` 硬切）。
- **必须支持** `prefers-reduced-motion: reduce` → 关闭位移与缩放，仅保留 opacity。

---

## 9. 无障碍（Accessibility）

- 正文对比度 ≥ **4.5:1**，大字 ≥ 3:1；玻璃上的文字不得因材质过透而失效（必要时提高材质不透明度）。
- 触控目标 ≥ **44×44px**。
- 分隔线、边框不得作为唯一的信息载体（需配合间距或文字层级）。
- 焦点可见：`:focus-visible` 用 `2px #0A84FF` 外环 + 2px offset。

---

## 10. 响应式（Responsive）

| 断点 | 行为 |
|---|---|
| ≥ 1024px | 容器 `min(1140px,92vw)`，网格 ≥ 3 列 |
| 744–1023px | 网格 2 列，侧栏收起（vibe 控制台转为底部 Sheet） |
| < 744px | 单列；hero 内边距降至 24px；大标题降至 28px |
| 通用 | 容器 `92vw`；`padding-bottom: env(safe-area-inset-bottom)` |

vibe 控制台原 `≤720px` 断点、web_work 原 `≤560px` 断点统一改为 **744px / 560px** 两级。

---

## 11. 统一 Token（`:root`，两项目共用）

```css
:root{
  /* -------- 背景 -------- */
  --bg:#000;
  --bg-2:#1C1C1E;
  --bg-3:#2C2C2E;
  --fill:rgba(120,120,128,.36);
  --fill-2:rgba(120,120,128,.24);
  --separator:rgba(84,84,88,.65);

  /* -------- 文字 -------- */
  --ink:rgba(255,255,255,.92);
  --ink-2:rgba(235,235,245,.60);
  --ink-3:rgba(235,235,245,.30);
  --ink-4:rgba(235,235,245,.18);

  /* -------- 强调 / 语义 -------- */
  --accent:#0A84FF;
  --accent-press:#0A7BEA;
  --green:#30D158; --red:#FF453A; --orange:#FF9F0A; --yellow:#FFD60A; --teal:#40C8E0;

  /* -------- 玻璃材质 -------- */
  --glass-fill:rgba(255,255,255,.09);              /* Regular */
  --glass-sheen:linear-gradient(180deg,rgba(255,255,255,.10),rgba(255,255,255,.035));
  --glass-blur:blur(30px) saturate(180%);
  --hairline:rgba(255,255,255,.14);
  --specular:rgba(255,255,255,.20);
  --shadow:0 10px 30px -10px rgba(0,0,0,.55), 0 2px 8px -4px rgba(0,0,0,.35);

  /* -------- 形状 -------- */
  --r-btn:12px; --r-card:16px; --r-panel:24px; --r-sheet:28px; --r-cap:999px;

  /* -------- 字体 -------- */
  --font:-apple-system,BlinkMacSystemFont,"SF Pro Text","SF Pro Display",
         "Helvetica Neue","PingFang SC","Hiragino Sans GB","Microsoft YaHei",sans-serif;
  --mono:ui-monospace,"SF Mono","JetBrains Mono",Menlo,Consolas,monospace;

  /* -------- 动效 -------- */
  --ease-sheet:cubic-bezier(.32,.72,0,1);
  --ease-std:cubic-bezier(.4,0,.2,1);
  --ease-out:cubic-bezier(0,0,.2,1);
  --ease-in:cubic-bezier(.4,0,1,1);
}
```

---

## 12. 迁移映射（旧规范 → Liquid Glass）

| 项目 | 旧值 | 新值 |
|---|---|---|
| 主背景 | `#04050a` / `#070912` | `#000` + 低饱和壁纸层（§2） |
| 彩色光晕 | 蓝 `.20` / 紫 `.18` / 青 `.14` | 删除；改为低饱和深灰蓝 `#16233a` 等，最多 2 个色相 |
| 玻璃填充 | `rgba(255,255,255,.12 ~ .045)` 斜向渐变 | `--glass-fill` + `--glass-sheen` 垂直渐变 |
| 模糊 | `blur(14~24px)` | `blur(30px) saturate(180%)`（Thick 40px） |
| 边框 | `1px solid rgba(255,255,255,.12~.16)` | `.5px solid var(--hairline)` + 折射亮环 `::before` |
| 阴影 | `0 22px 54px -20px rgba(0,0,0,.8)` + **彩色发光** | `--shadow`（纯中性）+ 上下发丝内高光 |
| 强调色 | 三色蓝紫青渐变 + `#ff9ad8` | 单一 `#0A84FF`；语义色仅用于状态 |
| 渐变标题 | `linear-gradient(100deg,#fff,#8fc4ff,#c9b3ff,#8ff0e0)` clip-text | 纯色 `--ink`，34px/700，字距 `-.022em` |
| 字体 | `Outfit` / `Segoe UI` / `JetBrains Mono` | SF Pro 栈（§5）；mono 仅用于数值 |
| 徽章 | 13px 大写 + `.14em` 字距 + 白描边 | `.chip`：13px Footnote、`--ink-2`、`--fill-2`、胶囊、无字距 |
| hover | `translateY(-5px)` + 蓝紫发光 | `scale(1.015)` + 高光位移 + 中性阴影加深 |
| 缓动 | `cubic-bezier(.22,1,.36,1)` | `--ease-sheet` / `--ease-out` |
| 状态点 | `#ff6b6b` + `0 0 8px` 发光 + 呼吸闪烁 | 静态单色点，活跃时 1.6s spring 极轻缩放 |
| 圆角 | 卡片 16–20 / 容器 20–30 | 按钮 8–12 / 卡片 14–18 / 面板 22–28，启用 `corner-shape: superellipse(2)` |
| 间距 | 18px gap、20–22px padding | 8pt 栅格：gap 16px、padding 16–20px |
| 分隔线 | 未定义 | 0.5px `--separator`，内缩对齐 |
| 断点 | 720px / 560px | 1024 / 744 / 560 |

---

## 13. 自审清单（交付前逐条过）

- [ ] 页面内彩色发光 `box-shadow` 数量为 **0**
- [ ] 渐变文字数量为 **0**；标题为纯色
- [ ] 强调色（`#0A84FF`）单一，一屏出现 ≤ 2 处
- [ ] 玻璃元素同时具备：模糊 + 折射亮环 + 发丝内高光 + 中性阴影
- [ ] 所有圆角走 token，且大容器尝试了 `corner-shape: superellipse(2)`
- [ ] 字体栈以 `-apple-system` / SF Pro 开头，中文回退 `PingFang SC`
- [ ] 间距全部是 8 的倍数
- [ ] 图标来自单一图标库，`stroke-width` 统一，无 emoji
- [ ] 动效使用 §8 四条曲线之一，且支持 `prefers-reduced-motion`
- [ ] 正文对比度 ≥ 4.5:1，触控目标 ≥ 44px

---

## 14. 一句话总纲

**纯黑壁纸底 + 低饱和氛围层 + 液态玻璃材质（大模糊 + 折射亮环 + 镜面高光 + 发丝边）+ 单一 iOS 蓝强调 + 连续圆角 + 8pt 栅格 + 弹簧动效** —— 层次来自光与材质，不来自颜色与发光。
