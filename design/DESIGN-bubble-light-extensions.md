---
version: 1.0
name: Bubble-light-extensions
description: >
  Bubble-light 的「材质 / 光影 / 动效」扩展层。DESIGN-bubble-light.md 定义了
  颜色、字体、圆角、间距等静态 token；本文档记录网站实现时在其之上叠加的
  设计决策 —— 类 Apple 毛玻璃、双层轻阴影、边框决策规则、氛围光背景、
  微交互与装饰语言 —— 并整理为可直接复用于任何 C 端网站的配方。
  两份文档合起来 = 一套完整的 C 端网站设计方案。

# ============ 扩展 token（与 DESIGN-bubble-light.md 的 token 并用） ============

materials:
  glass-nav:                       # 顶部导航毛玻璃（最透，站在氛围光之上）
    background: "rgba(249,248,246,.6)"        # canvas @ 60%
    backdropFilter: "blur(20px) saturate(180%)"
    borderBottom: "1px solid rgba(23,32,38,.06)"
  glass-card:                      # 内容级毛玻璃卡（hero 数据卡）
    background: "rgba(255,255,255,.66)"       # 纯白 @ 66%
    backdropFilter: "blur(24px) saturate(160%)"
    border: "1px solid rgba(255,255,255,.6)"  # 白色描边，不是灰 hairline
    rounded: 20px
    shadow: "0 1px 2px rgba(23,32,38,.05), 0 18px 48px rgba(23,32,38,.12), inset 0 1px 0 rgba(255,255,255,.7)"
  glass-pill:                      # 悬浮小组件（提示条、浮动标签）
    background: "rgba(255,255,255,.8)"
    backdropFilter: "blur(10px)"
    shadow: "{shadows.soft}"
  glass-sublayer-head: "rgba(231,232,244,.6)"   # 玻璃卡内的表头带（surface-card @ 60%）
  glass-sublayer-zebra: "rgba(235,235,244,.55)" # 玻璃卡内的斑马纹（surface-soft @ 55%）
  glass-inner-hairline: "rgba(23,32,38,.06)"    # 玻璃内部分隔线

shadows:
  # 双层结构：贴地 contact 影 + 大半径 ambient 环境影；色相统一用 ink 蓝黑 rgba(23,32,38,x)
  soft: "0 1px 2px rgba(23,32,38,.05), 0 4px 12px rgba(23,32,38,.06)"          # 小控件
  soft-hover: "0 2px 4px rgba(23,32,38,.05), 0 8px 20px rgba(23,32,38,.09)"
  card: "0 2px 6px rgba(23,32,38,.04), 0 12px 32px rgba(23,32,38,.07)"         # 白色内容卡
  card-hover: "0 4px 12px rgba(23,32,38,.05), 0 24px 56px rgba(23,32,38,.11)"
  glass: "0 1px 2px rgba(23,32,38,.05), 0 18px 48px rgba(23,32,38,.12)"        # 毛玻璃卡（最高浮起）
  btn-primary: "0 2px 10px rgba(23,32,38,.18)"
  btn-primary-hover: "0 5px 18px rgba(23,32,38,.24)"
  btn-ghost: "0 1px 2px rgba(23,32,38,.05), 0 4px 14px rgba(23,32,38,.07)"
  btn-ghost-hover: "0 3px 6px rgba(23,32,38,.05), 0 8px 22px rgba(23,32,38,.10)"
  # 彩色表面投同色系阴影（不是灰影）：
  btn-oncolor: "0 2px 8px rgba(4,42,46,.24)"            # 深 teal 卡上的白按钮
  btn-oncolor-hover: "0 5px 16px rgba(4,42,46,.30)"
  tier-incana: "0 18px 44px rgba(19,88,93,.26)"         # incana 深色卡自身的 teal 影

highlights:
  # 顶缘内高光：模拟「光从上方来」，卡片上边缘 1px 亮线
  on-glass: "inset 0 1px 0 rgba(255,255,255,.7)"
  on-light-band: "inset 0 1px 0 rgba(255,255,255,.5)"
  on-dark-card: "inset 0 1px 0 rgba(255,255,255,.14)"

aura:
  # 氛围光：多枚 radial-gradient 品牌色光斑 + 大半径 blur + 底部渐隐 mask
  hero:
    background: |
      radial-gradient(620px 460px at 70% 32%, rgba(85,202,232,.38), transparent 70%),
      radial-gradient(540px 400px at 26% 14%, rgba(189,145,201,.20), transparent 70%),
      radial-gradient(480px 360px at 54% 80%, rgba(142,197,209,.30), transparent 72%),
      radial-gradient(340px 280px at 88% 70%, rgba(238,201,99,.16), transparent 70%)
    blur: 48px            # 移动端降到 36px
    inset: "-130px 0 -40px"   # 顶部上溢 130px，垫到毛玻璃导航背后
    mask: "linear-gradient(180deg, #000 62%, transparent 100%)"
  page:                   # 内页轻量版：同构，alpha 整体 × ~0.8，高度 640px
    blur: 48px
    mask: "linear-gradient(180deg, #000 55%, transparent 100%)"
  grain:                  # SVG 噪点覆盖层，防止大面积 blur 出现色带
    svg: "feTurbulence fractalNoise baseFrequency=0.8 numOctaves=2"
    opacity: 0.06

motion:
  ease-brand: "cubic-bezier(.22, .9, .35, 1)"   # 快出发、缓落地
  duration-color: 180ms        # 纯颜色变化（nav 链接、行 hover）
  duration-control: 200ms      # 按钮、小控件
  duration-card: 300ms         # 卡片抬升
  duration-reveal: 600ms       # 滚动入场
  hover-lift-control: "translateY(-1px)"
  hover-lift-card: "translateY(-4px) scale(1.01)"
  press: "scale(.97)"
  reveal-from: "opacity 0 / translateY(18px)"
  reveal-stagger: 70ms
  pulse-dot: "scale 1→1.5, opacity 1→.55, 1.6s ease-in-out infinite"
  aura-drift: "translate3d(±5~6%, ∓3~4%, 0) scale(1~1.12), 10s ease-in-out infinite alternate"

focus:
  outline: "3px solid {colors.brand-sea}"
  offset: 3px
  radius: 12px
selection:
  background: "{colors.brand-sea}"
  color: "#fff"
---

# Bubble-light 扩展层：材质、光影与动效

> **状态：非规范示例（NON-NORMATIVE）** —— 规范入口是 `DESIGN-bubble-system.md`。
> 本文件记录 bubble-studio.xyz 实现的**精确配方**，即 system 第 2 层各参数区间内的
> **一组合法取值**，不是唯一解。公式与允许区间以 system 为准；生成新页面前先执行
> system 第 3 层的骰子程序，再回本文件按需取材。

> **与基础文档的关系**：`DESIGN-bubble-light.md` 是**颜色 / 字体 / 圆角 / 间距**的静态合同；
> 本文档是实现层沉淀出来的**材质（毛玻璃）、深度（阴影与边框）、氛围（背景光）、动效（微交互）**合同。
> 基础文档说"不靠重阴影做深度、不记录 hover"——本层是对它的**修订与补全**：阴影可以有，
> 但必须轻得像空气；hover 有统一语言，在此处一次性定义。

## 0. 一句话哲学：用「光」做深度，不用「重量」

整套视觉的深度模型是**一盏来自上方的柔光**：

1. 卡片浮起 → 底下有影子（双层、极低透明度、蓝黑色调，永不发灰发脏）；
2. 卡片顶缘 → 有 1px 内高光（`inset 0 1px 0 white`），像被光打亮的上边缘；
3. 玻璃 → 让背后的彩色氛围光透进来（`saturate()` 提饱和，防止透过来的颜色变灰）；
4. 越重要 / 越浮起的元素，ambient 影越大越远，但 alpha 始终 ≤ 0.12。

所有阴影、玻璃、高光都服务于这一个光源假设，所以页面上不会出现方向矛盾的深度线索。

---

## 1. 材质系统：三种表面 + 一种玻璃

页面上的任何容器都属于四种材质之一，不混用：

| 材质 | 配方 | 用途 |
|---|---|---|
| **Solid 哑光色块** | 8 色品牌色实心填充，无影或同色影 | feature card、序号 tile、图标底 |
| **Paper 纸面** | `#fff` 填充 + `{shadows.card}`，**无边框** | 内容卡、列表卡、metric 卡 |
| **Flat 平面** | `canvas` / `surface-card` 填充 + hairline 边框，**无影** | 表格、行列表、输入框 |
| **Glass 玻璃** | 半透明底 + `backdrop-filter` + 白描边 + 内高光 | 导航、hero 数据卡、悬浮小组件 |

### 1.1 毛玻璃（类 Apple frosted glass）——两档配方

**导航档（最透、最轻）** —— sticky 顶栏：

```css
.nav {
  position: sticky; top: 0; z-index: 50;
  background: rgba(249,248,246,.6);              /* canvas 色 @ 60%，不是白色 */
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  backdrop-filter: blur(20px) saturate(180%);
  border-bottom: 1px solid rgba(23,32,38,.06);   /* 极淡 ink 发丝线收边 */
}
```

**内容档（更实、可承载数据）** —— hero 玻璃卡：

```css
.glass-card {
  background: rgba(255,255,255,.66);             /* 纯白 @ 66% */
  -webkit-backdrop-filter: blur(24px) saturate(160%);
  backdrop-filter: blur(24px) saturate(160%);
  border: 1px solid rgba(255,255,255,.6);        /* 白描边模拟玻璃切边，勿用灰 hairline */
  border-radius: 20px;
  overflow: hidden;                               /* 内部子层贴边裁切 */
  box-shadow:
    0 1px 2px rgba(23,32,38,.05),                /* contact */
    0 18px 48px rgba(23,32,38,.12),              /* ambient（全站最大浮起） */
    inset 0 1px 0 rgba(255,255,255,.7);          /* 顶缘内高光 */
}
```

**玻璃的四条纪律：**

1. **玻璃背后必须有内容可透**。毛玻璃只有压在彩色氛围光（见 §3）或页面内容上才成立，
   压在素色 canvas 上会退化成一块灰板。hero aura 特意向上溢出 130px（`inset: -130px 0 -40px`），
   就是为了给 sticky 导航垫一层可以被 blur 的颜色。
2. **一定加 `saturate(160%~180%)`**。blur 会稀释背后颜色的饱和度，saturate 把它拉回来——
   这是"苹果味"和"廉价磨砂"的分水岭。
3. **玻璃用白描边，不用灰 hairline**；灰线会把玻璃变成"贴图"，白线才像切边反光。
4. **玻璃内部再分层用半透明色**，让透光连续：表头带 `rgba(231,232,244,.6)`、
   斑马纹 `rgba(235,235,244,.55)`、内部分隔线 `rgba(23,32,38,.06)`——全部带透明度，
   不要在玻璃内部使用实心色块（页脚 CTA 这类刻意压重的实心带除外）。

**悬浮小组件档**：`rgba(255,255,255,.8)` + `blur(10px)` + `{shadows.soft}`，
用于浮动提示条、角标之类的轻量元素。

### 1.2 玻璃卡的「窗口感」

hero 数据卡在玻璃之上叠了 macOS 窗口隐喻：左上三颗 9px 圆点（hairline 色）、
标题行、右侧外链、斑马纹数据行、底部实心 `primary` 确认条。
这个"实时窗口"让数据类信息天然带来可信感，是 C 端展示实时数据 / 价格 / 状态的通用配方。

---

## 2. 阴影系统：双层、蓝黑、低 alpha

### 2.1 公式

每个阴影都是**两层**（这是"Apple 式"的关键，单层阴影要么糊要么脏）：

```
box-shadow:
  0 {1~4px} {2~12px} rgba(23,32,38, .04~.05),    /* contact：贴地小影，定住轮廓 */
  0 {8~24px} {20~56px} rgba(23,32,38, .06~.12);  /* ambient：大半径环境影，营造浮起 */
```

三条铁律：

1. **色相**：阴影色永远是 ink 蓝黑 `rgba(23,32,38,x)`（即 `--dark` #172026），
   不用纯黑 `rgba(0,0,0,x)` —— 纯黑影在暖 canvas 上会发灰发脏。
2. **alpha 上限 0.12**（按钮 primary 例外，允许到 0.24，因为面积小）。
   阴影要读作"空气"，一旦能被明确看见就是失败。
3. **hover 时两层同时长大**：contact 抬高 ~2 倍，ambient 半径 +50%、alpha +~0.04，
   配合 `translateY` 完成"抬起"错觉。

### 2.2 阴影 token 分档

| token | 值 | 用在 |
|---|---|---|
| `soft` | `0 1px 2px .05 / 0 4px 12px .06` | 语言切换器、汉堡按钮等小控件 |
| `card` | `0 2px 6px .04 / 0 12px 32px .07` | 白色内容卡默认态 |
| `card-hover` | `0 4px 12px .05 / 0 24px 56px .11` | 卡片 hover |
| `glass` | `0 1px 2px .05 / 0 18px 48px .12` | 毛玻璃卡（全站最高层级） |
| `btn-primary` | `0 2px 10px .18` → hover `0 5px 18px .24` | 黑色主按钮（单层即可，面积小） |
| `btn-ghost` | `0 1px 2px .05 / 0 4px 14px .07` | 白底幽灵按钮 |

### 2.3 彩色表面投「同色影」

彩色卡如果投灰影，边缘会显脏。规则：**深色 / 彩色表面的阴影取自身色相的深色版**：

```css
/* incana 深 teal 会员卡：teal 色系阴影 + 暗面内高光 */
.tier { box-shadow: 0 18px 44px rgba(19,88,93,.26), inset 0 1px 0 rgba(255,255,255,.14); }

/* 深色卡上的白色按钮：影子也偏 teal 黑 */
.btn-oncolor { box-shadow: 0 2px 8px rgba(4,42,46,.24); }
```

效果像"光透过有色玻璃落下"，彩色卡与暖 canvas 之间不再有灰边。

### 2.4 顶缘内高光

浮起的容器统一加 `inset 0 1px 0 rgba(255,255,255,α)`：
玻璃卡 α=.7、浅色大 band α=.5、深色卡 α=.14。
一行代码把"光从上面来"写进每个容器，深度语言全站自洽。

---

## 3. 边框决策树：border 和 shadow 只选一个

基础文档只有 hairline 一种深度手段；实现层把它扩成一棵决策树——**同一容器永远不同时
用灰边框和阴影**（玻璃的白描边除外，它是材质不是分界）：

```
这个容器要「浮起来」吗？
├─ 要，且是白卡        → shadow-card，无边框
├─ 要，且是玻璃        → 白描边 rgba(255,255,255,.6) + shadow-glass
├─ 要，且是彩色/深色卡  → 同色系阴影（§2.3），无边框
└─ 不要（平铺融入页面） → 1px hairline 边框，无阴影
                          （表格、行列表、chip--blank、输入框）
```

**行分隔线**用伪元素做「内缩线」，不做通栏 `border-bottom`：

```css
.row::after {
  content: ""; position: absolute;
  left: 24px; right: 24px; bottom: 0; height: 1px;   /* 左右各内缩一个 padding */
  background: rgba(23,32,38,.05);
}
.row:last-child::after { display: none; }
```

内缩 + 5% 透明度的分隔线让列表读起来是"一张卡"，而非"一摞格子"。

---

## 4. 氛围背景层：aura + grain（毛玻璃成立的前提）

### 4.1 品牌光斑 aura

hero 背后铺 4 枚品牌色 radial-gradient 光斑，大 blur，底部 mask 渐隐融回 canvas：

```css
.hero-aura {
  position: absolute; inset: -130px 0 -40px;   /* 上溢 130px，垫到玻璃导航背后 */
  pointer-events: none; z-index: 0;
  background:
    radial-gradient(620px 460px at 70% 32%, rgba(85,202,232,.38), transparent 70%),  /* sky 主光 */
    radial-gradient(540px 400px at 26% 14%, rgba(189,145,201,.20), transparent 70%), /* lavender */
    radial-gradient(480px 360px at 54% 80%, rgba(142,197,209,.30), transparent 72%), /* mint */
    radial-gradient(340px 280px at 88% 70%, rgba(238,201,99,.16), transparent 70%);  /* lemon 点缀 */
  filter: blur(48px);                          /* 移动端降到 36px（省性能） */
  mask-image: linear-gradient(180deg, #000 62%, transparent 100%);
  animation: aura-drift 10s ease-in-out infinite alternate;   /* 缓慢漂移，似呼吸 */
}
@keyframes aura-drift {
  0%   { transform: translate3d(0,0,0) scale(1); }
  50%  { transform: translate3d(6%,-4%,0) scale(1.12); }
  100% { transform: translate3d(-5%,3%,0) scale(1.04); }
}
```

配方要点：**一枚主光（最高 alpha，放在视觉焦点侧）+ 一枚次光 + 一两枚低 alpha 点缀色**；
alpha 全部 ≤ 0.4；用 mask 渐隐而不是让 gradient 自己收尾（更干净）。
内页用同构的轻量版（alpha ×0.8，固定 640px 高，`z-index:-1`），保持全站"从光里开始"的一致开场。

### 4.2 噪点 grain 防色带

大面积 blur 渐变在低色深屏幕上会出现 banding。上面盖一层 6% 透明度的 SVG 分形噪点即可：

```css
.hero-grain {
  position: absolute; inset: -130px 0 0; pointer-events: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='160' height='160'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.8' numOctaves='2' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='160' height='160' filter='url(%23n)' opacity='0.06'/%3E%3C/svg%3E");
  mask-image: linear-gradient(180deg, #000 55%, transparent 100%);  /* 与 aura 同步渐隐 */
}
```

顺带给画面一点"纸感"，中和数码渐变的塑料味。

### 4.3 进阶选项：WebGL orb

`final/orb-demo.html` 保留了一个浅色主题适配的 WebGL 泡泡（OGL 实现，移植自 vue-bits Orb）：
着色器接收页面底色，按背景亮度自动切换明暗渲染分支；hue 参数对齐 lavender/sky。
适合品牌 hero 需要"活物"时替换静态 aura，务必保留 `prefers-reduced-motion` 降级为静帧。

---

## 5. 动效语言：一条曲线、四档时长、三种手势

### 5.1 全站只有一条缓动曲线

```css
--ease: cubic-bezier(.22, .9, .35, 1);   /* 快出发、缓落地，轻盈不弹跳 */
```

时长四档，按"变化的物理量"选：颜色 180ms · 控件 200ms · 卡片 300ms · 入场 600ms。

### 5.2 三种交互手势（全站统一，不再逐组件发明）

| 手势 | 配方 | 适用 |
|---|---|---|
| **悬停 = 抬起** | 控件 `translateY(-1px)`；卡片 `translateY(-4px) scale(1.01)`；阴影同步升档（§2.1） | 按钮、卡片 |
| **按下 = 收缩** | `active: scale(.97)` | 所有可点元素 |
| **行悬停 = 染色** | 仅背景变 `hairline-soft`，不位移不加影 | 列表行、表格行 |

位移量刻意小（1px / 4px）——抬升感主要由阴影完成，位移只是佐证。

### 5.3 滚动入场 Reveal

```css
.js .reveal      { opacity: 0; transform: translateY(18px);
                   transition: opacity .6s var(--ease), transform .6s var(--ease); }
.js .reveal.in   { opacity: 1; transform: none; }
```

工程约定（都是踩过坑的）：

- **JS-gated**：隐藏态挂在 `.js .reveal` 下，`<html>` 有 JS 才加 `.js` 类 ——
  无 JS / 爬虫 / RSS 场景内容永远可见，SEO 不受损；
- IntersectionObserver：`threshold: 0.12`，`rootMargin: '0px 0px -8% 0px'`（提前一点触发），
  进场后立即 `unobserve`；
- 同组元素 **70ms 阶梯 stagger**，超过 4 个从头循环（`i % 4 * 70`）；
- `prefers-reduced-motion: reduce` 时全部动画一刀切关掉且内容直接可见：

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: .001ms !important;
    transition-duration: .001ms !important;
    scroll-behavior: auto !important;
  }
  .js .reveal { opacity: 1; transform: none; }
}
```

### 5.4 常驻微动效（吝啬地用）

- **pulse-dot**：9px 圆点 `scale 1→1.5 / opacity 1→.55`，1.6s 循环——全页唯一的"活着"信号，
  只给真正实时的数据（live 状态）用；
- **aura-drift**：背景光 10s 呼吸（§4.1）。
- 除这两处外，静止的页面不该有任何东西在动。

---

## 6. 装饰语言：泡泡宇宙

给"干净但冷"的系统注入 C 端亲和力的三件套：

1. **溢出气泡**：大色块 band 的角落放 1–2 枚超出边界的半透明圆
   （浅色 band 用 `rgba(canvas,.6)`，深色 band 用 `rgba(255,255,255,.07)`），
   `overflow:hidden` 裁切出月牙形。成本一个 `<span>`，立刻打破大色块的呆板。
2. **跨界吉祥物**：CSS-only 泡泡（sky 圆身 + 背景色高光点 + lavender 卫星圆）骑在
   section 分界线上（`top:-26px`），可复用组件化（`PaopaoMascot`，高光色跟随所处表面）。
   吉祥物骑线 = 打破带状布局的横向切割感。
3. **跨带 CTA**：底部 CTA 卡用 `margin-bottom:-72px` 压进 footer，两个 band 被它缝合，
   页面结尾不再是"三条平行横带"。

节奏底盘：canvas 与 `surface-card` 两种底色交替成 band；圆角按嵌套层级递减
——band/玻璃 24/20px → 卡 16px → 按钮/tile 12px → chip pill——内层永远比外层小一档。

---

## 7. 细节合同（C 端网站的免检清单）

- **focus-visible**：`outline: 3px solid {brand-sea}; outline-offset: 3px`——键盘态比 hover 态更醒目；
- **::selection**：`brand-sea` 底白字，选中文本也在品牌里;
- **skip-link**：屏外定位，`:focus` 时回到 16px 处；
- **数字一律 `font-variant-numeric: tabular-nums`**（`.tnum`）：价格、统计、日期不跳动；
- **标题 `text-balance`** + `clamp()` 流式字号（如 `clamp(38px, 5vw, 60px)`）；
- **CJK 排版**：拉丁 display 的负字距在 CJK 下归零（`:lang(zh) h1..h3 { letter-spacing: normal }`），
  CJK 字体走 `unicode-range` 按需加载（详见基础文档）；同一标题拉丁文案比 CJK 长 ~40%，
  字号档位按 locale 微调（拉丁 clamp 上限调低 + 负字距）；
- **移动端性能**：blur 降档（48→36px）、sticky 元素在窄屏改 static；
- 触控目标 ≥ 42×42px（汉堡按钮 42px，按钮 44px+）。

---

## 8. 复用手册：移植到任何 C 端网站的步骤

1. **先落基础层**（DESIGN-bubble-light.md）：换上你的 canvas 色 + 中性色 + 品牌色盘，
   保持"暖中性底 + 少量冷中性面 + 饱和色块"的结构；
2. **定光源**：把 `rgba(23,32,38,x)` 换成你的 ink 深色的 RGB，生成 §2.2 的阴影分档——
   全站阴影只从这套 token 里取；
3. **铺氛围**：从品牌色盘挑 1 主 + 1 次 + 2 点缀，套 §4.1 的 aura 配方（记得上溢到导航背后 + grain）；
4. **上玻璃**：导航用 60% canvas 档，hero 焦点卡用 66% 白档——只这两处，玻璃多了就贱；
5. **接动效**：拷贝 `--ease`、四档时长、三种手势和 Reveal（含 reduced-motion / no-JS 降级）；
6. **点装饰**：溢出气泡、跨线元素、跨带 CTA，按品牌吉祥物 / 图形语言替换造型；
7. **走查 §7 清单**。

### Tailwind 接线参考

```ts
// tailwind.config.ts — 摘自本站实现
extend: {
  transitionTimingFunction: { brand: 'cubic-bezier(.22,.9,.35,1)' },
  boxShadow: {
    soft:  '0 1px 2px rgba(23,32,38,.05), 0 4px 12px rgba(23,32,38,.06)',
    'soft-hover': '0 2px 4px rgba(23,32,38,.05), 0 8px 20px rgba(23,32,38,.09)',
    card:  '0 2px 6px rgba(23,32,38,.04), 0 12px 32px rgba(23,32,38,.07)',
    'card-hover': '0 4px 12px rgba(23,32,38,.05), 0 24px 56px rgba(23,32,38,.11)'
  },
  keyframes: {
    'pulse-dot': { '0%,100%': { transform: 'scale(1)', opacity: '1' },
                   '50%': { transform: 'scale(1.5)', opacity: '.55' } },
    'aura-drift': { '0%': { transform: 'translate3d(0,0,0) scale(1)' },
                    '50%': { transform: 'translate3d(6%,-4%,0) scale(1.12)' },
                    '100%': { transform: 'translate3d(-5%,3%,0) scale(1.04)' } }
  },
  animation: {
    'pulse-dot': 'pulse-dot 1.6s ease-in-out infinite',
    'aura-drift': 'aura-drift 10s ease-in-out infinite alternate'
  }
}
```

```html
<!-- 组件配方速查（Tailwind 任意值写法，摘自实现） -->
<!-- 毛玻璃导航 -->
<nav class="sticky top-0 z-50 bg-[rgba(249,248,246,.6)] backdrop-blur-[20px]
            backdrop-saturate-[1.8] border-b border-[rgba(23,32,38,.06)]">

<!-- 玻璃数据卡 -->
<div class="bg-[rgba(255,255,255,.66)] backdrop-blur-[24px] backdrop-saturate-[1.6]
            border border-white/60 rounded-[20px] overflow-hidden
            shadow-[0_1px_2px_rgba(23,32,38,.05),0_18px_48px_rgba(23,32,38,.12),inset_0_1px_0_rgba(255,255,255,.7)]">

<!-- 白色内容卡（无边框，纯影） -->
<article class="bg-white rounded-lg p-7 shadow-card transition-all duration-300 ease-brand
                hover:-translate-y-1 hover:scale-[1.01] hover:shadow-card-hover">

<!-- 主按钮 -->
<a class="bg-primary text-on-dark rounded-md shadow-[0_2px_10px_rgba(23,32,38,.18)]
          transition-all duration-200 ease-brand active:scale-[.97]
          hover:bg-primary-active hover:-translate-y-px hover:shadow-[0_5px_18px_rgba(23,32,38,.24)]">
```

---

## 9. Do / Don't

### Do

- 阴影永远双层、永远 ink 蓝黑色调、ambient alpha ≤ 0.12；
- 彩色 / 深色卡投同色系阴影；
- 毛玻璃必配 `saturate(160%+)`、白描边、顶缘内高光，且背后必须有氛围光或内容；
- aura 上溢到 sticky 导航背后，玻璃才有东西可磨；
- 悬停 = 抬起（位移 + 影），按下 = `scale(.97)`，全站统一；
- Reveal 隐藏态 JS-gated，`prefers-reduced-motion` 全量降级；
- 数字用 tabular-nums，键盘焦点环比 hover 更醒目。

### Don't

- 不给同一容器同时上灰边框和阴影（二选一，见 §3 决策树）；
- 不用纯黑阴影（`rgba(0,0,0,x)` 在暖底上发脏）；
- 不给毛玻璃用灰 hairline 描边；
- 不超过两处玻璃档位（导航 + 一处焦点卡）——玻璃泛滥立刻显廉价；
- 不让静止页面有超过一处常驻动画（live 圆点之外别加）;
- 不在玻璃内部用不透明大色块（会掐断透光连续性；刻意压重的锚定条除外）；
- 不把 hover 位移做大（>4px 就从"轻盈"变"果冻"）。

## 已知边界

- 本层与基础文档"无重阴影"条款的关系：**修订**——阴影存在但以"空气感"为验收标准
  （大半径、低 alpha、蓝黑色调）；
- `backdrop-filter` 在旧版 Firefox / 部分 WebView 不可用：玻璃底色的 60~66% 透明度
  本身可读，属可接受降级（无需 JS 探测）；
- 玻璃卡叠玻璃卡（双层 backdrop-filter 嵌套）未定义，避免使用；
- 深色模式尚未定义——aura、阴影、玻璃三套 token 均需另行推导，不能直接反色。
