---
version: 1.0
name: Bubble-light-console-extensions
description: >
  Bubble-light 的「中后台 / 管理控制台」扩展层。DESIGN-bubble-light.md 定义了
  颜色、字体、圆角、间距的 C 端静态合同；DESIGN-bubble-light-extensions.md 定义了
  C 端网站的材质 / 光影 / 动效配方；本文档记录把这套语言落到 B 端管理后台
  （首个实现：Ouro Pass admin console, S0023）时沉淀的全部设计决策——
  B 端合法偏离清单、控件密度、组件交互规范（列表 / 行动作 / 选择器 / 外壳）、
  以及 Web3 钱包选择器这类领域组件的解剖。三份文档合起来 =
  一套同一品牌下 C 端网站 + B 端控制台的完整设计方案。

# ============ 控制台 token（覆盖/补充基础文档同名项） ============

console-radius:            # B 端圆角标尺（覆盖基础文档的 6/8/12/16/24）
  xs: 2px                  # 复选框、小标记
  sm: 3px                  # 导航项、菜单项、tab、badge
  md: 4px                  # 按钮、输入框、下拉、tooltip
  lg: 6px                  # 卡片、表格、抽屉、toast
  xl: 8px                  # 对话框、登录焦点卡
  pill: 9999px             # 仅头像、状态圆点；badge 不再用 pill

console-metrics:
  control-height: 36px     # 按钮/输入/下拉统一高度（C 端合同为 44px，B 端合法偏离）
  control-height-sm: 32px  # 表格行内按钮、分页器
  icon-action: 32px        # 行动作图标按钮（icon-sm）
  wallet-row: 48px         # 钱包选择器行（领域组件，见 §10）
  sidebar: 15rem
  sidebar-rail: 4.25rem
  topbar: 56px
  page-padding: 24px

console-colors:            # 语义色的 B 端适配（正文可读版，底色仍用品牌原值做 soft tint）
  destructive-text: "#c94f1c"   # brand-orange 加深至 4.5:1
  success-text: "#5a7a1e"       # brand-grass 加深
  warning-text: "#8a6414"       # brand-lemon 加深
  info-text: "#2f6fc4"          # brand-sea 加深
  link: "#2f6fc4"
  focus-ring: "#488be1"         # brand-sea，2px + 2px offset

console-materials:         # 材质在后台的映射（基础决策树见 C 端 extensions §3）
  paper: "内容卡：#fff + shadow-card，无边框"
  flat: "表格 / 输入框 / 列表：hairline 边框，无阴影"
  glass-allowed: "仅两处：顶栏（canvas@60% blur20 saturate180）+ 登录焦点卡"
  scrim: "rgba(23,32,38,.45)"   # 对话框/抽屉遮罩，ink 色系非纯黑

console-motion:            # 沿用 C 端 ease-brand；后台手势更克制
  hover-control: "translateY(-1px) + 阴影升档（仅按钮）"
  hover-row: "背景染色 surface/40，无位移"        # 行悬停 = 染色
  hover-menu-item: "背景染色 surface"
  press: "scale(.97)"
  duration: "150ms（染色）/ 200ms（控件）"
  aura: "仅登录页；业务页面零常驻动画"
---

# Bubble-light 控制台扩展层：B 端密度、组件交互与领域组件

> **与姊妹文档的关系**：`DESIGN-bubble-light.md` 是颜色/字体/圆角/间距的静态
> 合同；`DESIGN-bubble-light-extensions.md` 是 C 端网站的材质/光影/动效配方；
> **本文档是 B 端控制台的实现合同**。凡本文档未覆盖处，按「先 C 端 extensions、
> 再基础合同」的顺序回退。首个参照实现：Ouro Pass admin（S0023，含 21 项
> 用户评审驱动的迭代 + 一轮三模型交叉代码评审）。

## 0. B 端合法偏离清单（The Divergence Contract）

C 端合同为触屏和情感化表达而设；控制台为键鼠密集操作而设。下面是**唯二**
获准偏离基础合同的维度，其余一切（色板、字体、间距基数、阴影公式、
材质纪律）与 C 端一致：

| 维度 | C 端合同 | B 端取值 | 理由 |
|---|---|---|---|
| 控件高度 | 44px | **36px**（表格内 32px） | 信息密度；键鼠精度不需要 44px 触摸目标 |
| 圆角标尺 | 6/8/12/16/24 | **2/3/4/6/8**（Cloudflare 档） | 密集界面里大圆角显「泡」，利落方角承载专业感 |

**不偏离**的显性重申：canvas #F9F8F6 地板、ink 主色、冷紫灰 surface、
hairline 描边、Inter（自托管，勿只写 font-family）、双层 ink 蓝黑阴影
（ambient alpha ≤ 0.12）、玻璃纪律（≤2 处）、brand-sea 焦点环与选区。

## 1. Token 适配

- **语义色正文可读化**：brand-orange/grass/lemon/sea 直接做正文色对比度不足，
  B 端派生一组加深值（见 frontmatter `console-colors`），**底色 tint 仍用品牌
  原值的 soft 版**——例如 destructive 按钮底 #c94f1c 文字白、soft 徽章底
  #fbe9df 文字 #c94f1c。
- **enable/disable 语义色**：enable 动作绿染（success），disable 中性 ink。
- **深色主题**：基础合同未定义 dark。实现层存在一份从 surface-dark #172026
  派生的**临时**暗色（去饱和强调色），标注为契约外债务；正式暗色需另行推导，
  不能反色。
- **保留位 token**：`--incana`、`--success-accent` 等在实现中零引用，作为
  合同保留位存在；新项目可裁剪。

## 2. 材质映射（后台版）

沿用 C 端 extensions §3 的「border 和 shadow 二选一」决策树，后台的对号入座：

| 容器 | 材质 | 配方 |
|---|---|---|
| 内容卡 / 统计卡 | Paper | 白底 + shadow-card，无边框，rounded-lg |
| 表格 / 输入框 / 列表 | Flat | hairline 边框，无阴影；表头带 surface/50 |
| 顶栏 | Glass 导航档 | canvas@60% + blur20 + saturate180 + 极淡 ink 发丝线 |
| 登录焦点卡 | Glass 内容档 | 白@66% + blur24 + 白描边 + 顶缘内高光（全站唯二玻璃） |
| 对话框 / 抽屉 | Paper 浮层 | 白底 + glass 档阴影，无边框；遮罩 ink@45% |
| 下拉 / tooltip | Paper 小浮层 | 白底 + card-hover 阴影；tooltip 反色（ink 底白字） |

侧边栏**直接坐在 canvas 上**（无卡片感），与内容区共用地板，右缘 hairline-soft。

## 3. 按钮体系

| 变体 | 用途 | 规则 |
|---|---|---|
| primary（ink 底白字） | 视图内唯一主动作 | **一个视图最多一个**；hover 深化 + 抬起 1px |
| secondary（surface 底） | 主力工作按钮 | 无阴影，染色 hover |
| outline（白底 hairline） | 次级动作、选项行 | soft 阴影 + 抬起 |
| ghost | 工具性动作 | 仅染色 |
| destructive（orange 底） | 危险主动作 | 永不出现在表格行内 |
| link | 行文内跳转 | 下划线，无按钮形态 |

- **loading 契约**：所有异步按钮用内置 `loading` 态（spinner + 禁用 +
  **保留原文案**防宽度跳动）；禁止 "Working…" 一类临时文案。
- 手势：hover = 抬起（仅按钮），press = scale(.97)，全站统一。

## 4. 列表页范式（旗舰模式）

结构自上而下：**PageHeader**（标题 + 描述 + 右侧主动作）→ **ListToolbar**
（左搜索 / 右列表级动作）→ **Flat Table** → **分页页脚**。

- **分页**：可无限增长的列表必须分页；有界小列表（如签名密钥）豁免。
  页脚三件套：`1–20 of 134` 范围文本 · 每页行数（10/20/50）· 翻页器。
- **客户端分页语义**（列表以全量数组到达时）：搜索或改页容量 → 回第 1 页；
  结果集收缩 → 页码钳位**并落回 state**（防回涨跳页）；`setPage` 下界钳 1。
  已知边界：后端本身限流的列表（如审计 200 条），客户端分页触不到更老记录，
  需在文案或后续服务端分页中言明。
- **搜索**：前置放大镜 + 有值时清除钮；搜索函数按 latest-ref 持有
  （调用方可放心传内联 lambda）。
- **状态**：加载 = 表格骨架屏；错误 = 描边警示条；空 = 虚线 EmptyState；
  搜索无命中 = 表内整行 "No X match “query”"。
- 数字列一律 `tabular-nums`。

## 5. 行动作规范（Row Actions）

调研基线：Carbon data-table/overflow-menu、NN/G contextual menus、
RainbowKit 等（评审记录见实现仓 code_review/）。

1. **≤2 个动作 → 图标按钮**（32px ghost，tooltip + sr-only 文案），危险动作
   红染；**≥3 个 → 常显 kebab**（不做 hover 才出现——触屏不可发现），
   破坏性动作分隔线隔开置底。
2. **高频动作可钉出**：`pinned` 的动作以图标按钮列在 kebab 左侧
   （例：Edit/Send 外露，Duplicate/Delete 收纳）。
3. **动作列**：`w-px + nowrap` 收缩到内容宽、右对齐、表头 sr-only——
   横向占位最小化，永不换行。
4. **图标注册表**：操作类型 → 图标是全局唯一契约（edit=Pencil、delete=Trash2、
   toggle 拆 enable=Power/disable=PowerOff…）。**图标表达「点了会发生什么」，
   不是「现在是什么状态」**。feature 代码只传语义 kind，禁止直接引图标。
5. **危险动作必须二次确认**：普通危险 → ConfirmDialog（标题 + 后果 + 红色
   确认钮）；凭证级操作（吊销、密钥）→ step-up 钱包重签。菜单项触发的对话框
   **提升到页面级受控**（menu item onSelect → state → 受控 dialog），不在行内
   挂 trigger 式对话框。
6. 整行点击开抽屉的表格（详情型）不设动作列，行尾 ChevronRight 指示。

## 6. 表单与选择器

- **单选**：Radix Select，options 驱动的一行式 API；触发器与输入框同形；
  纸面弹层 + 选中打勾；**选项值不得为空串**——"None" 用哨兵值映射；
  超长实体 id 在 label 层中段截断（`pool:95e81a69…79d458.state`），
  value 与提交数据保留全量。
- **多选**：MultiSelect 下拉（摘要触发器：占位 / 单项名 / "N selected"；
  自绘勾选项 + Select all；勾选保持展开）。行内裸 checkbox 清单仅限
  ≤3 项且需要常显对比的场景。
- **复选框**：自绘（ink 填充 + SVG 勾），原生语义保留。
- 校验错误显示在 Field 下方；label 走加字距小型大写。

## 7. 外壳（App Shell）

- **视口锁定**：md+ 布局 100dvh + 主内容区内滚——侧边栏高度永不随内容增长。
- **侧边栏**：canvas 地板；分组 caption-uppercase 标签；激活项 =
  surface-card 底 tab（category-tab 范式）；owner 级目的地带锁标；
  **可折叠成图标轨**（4.25rem，localStorage 记忆，折叠后分组标签变细分隔线、
  项带 tooltip）；底部仅留折叠开关。
- **顶栏**：玻璃导航档，sticky；左侧面包屑（分组 / 页名）；右上角
  **账号菜单**（角色头像 + 下拉：身份、登出）——身份与退出住右上，
  不住侧边栏底部。
- 移动端：侧边栏转覆盖式抽屉（ink 遮罩），布局恢复整页滚动。

## 8. 反馈层

- **对话框**：paper 浮层 + ink 遮罩；同时支持 trigger 驱动与受控
  （open/onOpenChange）两种模式——菜单场景必须受控。
- **抽屉**：右侧滑出，详情用 **DetailList**（hairline 行分隔 +
  加字距大写小标签 + ink 值）。
- **toast**：右下角 paper 卡，语义色标题；5s 自清。
- **tooltip**：ink 底白字小气泡，200ms 延迟；自带 Provider
  （隔离环境可独立工作）；图标按钮必须配 tooltip + sr-only。
- **Tabs**：分段控件（surface 轨道 + 浮起白 pill），完整 WAI-ARIA 契约——
  roving tabindex、方向键/Home/End、tab↔panel id 关联。声明 ARIA 语义
  就必须履行其键盘契约，否则宁可不声明。

## 9. 可访问性底线

- 焦点环：2px brand-sea + 2px offset，全组件一致；组件自带 ring 时
  `outline-none`，避免双指示。
- 图标动作：tooltip + sr-only 双通道；kebab 触发器有 aria-label。
- reduced-motion 总闸：一刀切关动画（含登录 aura）。
- 表格动作列表头 sr-only；分页控件带 aria-label。

## 10. 领域组件：Web3 钱包选择器

行业基线：RainbowKit / ConnectKit / Reown AppKit。**它是「选择身份」的菜单，
不是动作按钮**——尺寸偏离按钮体系是刻意的：

- 48px 行高、28px 圆角图标（无图标用首字母色块兜底）、名称居左加粗省略；
- **右侧必须锚定内容**：常态 `Installed` 灰字（CIP-30 可发现即已安装），
  连接中换 spinner（React 侧）或 `Connecting…`（原生 JS 侧，**失败必须复位**）；
- hover 用菜单行染色，不用按钮抬起；
- **探测逻辑三铁律**（S0023 B1 + 评审 P1 的教训）：① 别名注入按 name+icon
  去重、canonical key 优先；② 集合比较用 key 签名而非数量；③ 轮询跑满
  探测窗口（首个钱包出现不停止——Lace 类晚注入）。**该逻辑存在于 React SPA 与
  Go 内嵌 JS 两份实现，任何修复必须双侧同步**——这是本项目吃过的亏。

## 11. C 端授权页（/bind、/connect 类）

成员/用户可见的授权页**回归 C 端合同**：44px+ 触摸目标、C 端大圆角、
aura + 玻璃焦点卡配方（见 C 端 extensions）；结构上采 Apple 授权页版式
（品牌标识 → 一句话标题 → 选项列表 → 极简 fine print，能删则删）。
多页共享的样式抽成模板 partial，单一来源。渠道/身份行前置类型图标
（静态可信 SVG，注册表守卫 hasOwnProperty，禁止插值 API 数据）。

## 12. Do / Don't

### Do
- 一个视图一个 primary；异步按钮走 loading 契约。
- 可增长列表必分页；页码钳位落 state。
- ≥3 行动作收 kebab、危险置底分隔；≤2 用图标按钮 + tooltip。
- 图标 = 动作方向；同类操作全局同图标（注册表收口）。
- 危险操作二次确认；凭证操作 step-up。
- 声明 ARIA 语义就实现其键盘契约。
- 共享逻辑（如钱包探测）改一处必须同步所有实现副本。

### Don't
- 不在后台用 C 端的 44px/大圆角（除 C 端授权页）；不加第三个偏离维度。
- 不在表格行内放实心按钮；不做 hover 才出现的行动作。
- 不用 "Working…" 类临时按钮文案。
- 玻璃不超过两处（顶栏 + 登录卡）；业务页面零常驻动画。
- 不让 `Select` 出现空串 option value。
- 不把 API 数据插进 innerHTML 或未白名单的 className。

## 已知边界

- 深色主题是契约外临时派生，正式版需单独推导；
- 客户端分页依赖「全量数组到达」前提，后端限流列表存在可达性边界；
- a11y 为约定式覆盖（tooltip/sr-only/键盘），未跑自动化 axe 审计；
- 36px 密度与 2-8px 圆角为当前两项合法偏离，新增偏离需回到本文档修订。
