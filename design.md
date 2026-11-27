# design.md — 组件库设计规范（AI 生成页面的权威源）

> 版本 v1.6.7（2026-11-27）｜ 配套实现：《组件库设计规范.html》（同目录）｜ 复用模式：**快照模式**

本文件与《组件库设计规范.html》是同一份规范的两个视图：本文件供 AI 阅读与检索，组件库 HTML 是视觉实现与人工验收场。**两者冲突时以组件库 HTML 的 `[DESIGN-TOKENS]` / `[COMPONENTS-CSS]` 标记区实现为准，并把差异视为需要修复同步的 BUG。**

---

## 0. 给 AI 的使用契约（生成页面前必读）

### 0.1 复用模式：快照

生成的页面是**自包含单文件 HTML**，通过把组件库样式「快照」进页面的方式复用：

1. 打开《组件库设计规范.html》源码；
2. 在 `<style>` 内定位四个单行标记注释，复制**两个标记行之间的内容**（不含标记行）：
   - `/* [DESIGN-TOKENS-START] */` 与 `/* [DESIGN-TOKENS-END] */` 之间 → 快照区①（设计变量）
   - `/* [COMPONENTS-CSS-START] */` 与 `/* [COMPONENTS-CSS-END] */` 之间 → 快照区②（组件样式）
3. 在 body 末尾 `<script>` 内定位：
   - `/* [COMPONENTS-JS-START] */` 与 `/* [COMPONENTS-JS-END] */` 之间 → 快照区③（交互脚本）
4. 三区原样嵌入生成页面（顺序：①② 进 `<style>`，③ 进 body 尾部 `<script>`），**不得删改其中的类定义与变量名**。

注意：`[DEMO-STYLES-START]` ~ `[DEMO-STYLES-END]` 是组件库页面自己的展示样式，**不进快照**，禁止复制；`[GALLERY-CSS-START]` ~ `[GALLERY-CSS-END]` 与 `[GALLERY-JS-START]` ~ `[GALLERY-JS-END]` 是画廊层（侧栏组件搜索、hash 深链），仅组件库页面自用，同样**不进快照**，禁止复制。

### 0.2 生成流程（五步，两段式按需读取）

**第一段·搭框架（只读契约层，不碰组件库 HTML）**

1. **读契约**：§0.3 内容纪律 + §2 token + §3 页面骨架，一次读入——这是搭框架的全部输入；
2. **搭框架**：选定 §3 骨架或页面级模板，列出页面有哪些区块、每个区块承载什么信息；

**第二段·按框架索引与填充（索引与实现分离，按需读取）**

3. **按框架索引组件**：逐区块查 §4 注册表，只取用到的组件的类名 API——注册表是索引不是实现；禁止因为「组件库里有」而引入框架用不到的组件；
4. **按锚点抄实现**：只到组件库 HTML 抄**用到的** `#panel-{组件id}` 面板的「结构模板」代码块（如按钮在 `#panel-btn`），**禁止整读组件库 HTML**（60 个面板的演示 DOM 会污染上下文、诱发拖拽无关组件）；
5. **组装 + 自检**：组件类填充框架，按 §0.3 过文案，逐条钩 §0.4 清单。

### 0.3 内容纪律（生成的是产品页，不是设计方案说明页）

唯一判据：**删掉这段文字，用户还能不能理解当前信息、作出判断、完成操作？** 能就删；不能就先改 UI，改不动才补文案。

**只保留三类文案**：

| 类型 | 典型内容 | 缺了会怎样 |
|---|---|---|
| 理解所必需 | 字段标签、单位、统计口径、时间范围 | 误解数据含义 |
| 判断所必需 | 状态标签、风险提示、操作后果、校验错误 | 做错决定 |
| 操作所必需 | 按钮文案、空态引导动作、结果反馈 | 不知下一步做什么 |

**禁止四类文案**（出现即视为缺陷）：

1. 复述标题的导语/副标题——「在这里管理你的全部订单」；
2. 功能卖点——「高效管理」「一站式操作」「支持 XX 能力」；
3. 设计说明与交互提示——「核心功能」「设计亮点」「点击可查看详情」「支持拖拽排序」；
4. 对控件本身的解释——按钮旁边再写一句这个按钮是干什么的。

**不直观先改设计，不加说明**。替代顺序：信息层级 → 分组 → 列名/字段标签 → 默认值 → 占位符 → 即时校验 → 状态色 → Tooltip / 示例 / 展开区。降级到文字时，每区块常驻辅助说明 ≤1 句，且不得与标题、字段标签、按钮文案重复。

**空态与异常态**只写「发生了什么 + 下一步做什么」，不解释原因和原理。

**设计理由与实现备注**写在交付说明里，不得进入页面 DOM（含 HTML 注释与隐藏元素）。

**适用范围**：本节约束生成的产品页；《组件库设计规范.html》是规范视图，其面板内的页面说明与组件 API 块属验收文档，不受此限。

**给外部 AI 工具的前置提示词**（对方读不到本文件时，整段粘贴）：

```
请按真实上线产品的标准设计原型，不要把原型做成设计方案说明页。
1. 页面只保留用户理解信息、作出判断和完成操作所必需的文案。删除面向评审者的设计解释、功能介绍、装饰性标签，以及重复表达页面标题、字段或按钮含义的说明。
2. 优先通过信息层级、布局分组、控件选择、视觉状态和交互反馈，让界面本身清晰易懂。遇到不直观的地方，先优化设计，不要直接补一段说明文字。
3. 必要的字段标签、输入格式、校验提示、操作结果和风险提醒必须保留。次要帮助信息可按需放在 Tooltip、示例或展开区域中，不要全部常驻页面。
4. 不要在界面中加入「高效管理」「一站式操作」「支持 XX 能力」等无助于当前任务的宣传性文案，也不要添加「核心功能」「交互说明」「设计亮点」等讲解性内容。
5. 输出前逐项检查：这段文字被删除后，是否影响用户理解或操作？如果不影响，就删除；如果影响，先判断能否通过更好的 UI 和交互消除对它的依赖。
6. 设计理由与实现备注如需提供，请放在原型之外，不要展示在产品界面中。
```

### 0.4 生成自检清单（交付前逐条打钩）

- [ ] `<style>` 首段来自组件库 `[DESIGN-TOKENS]` + `[COMPONENTS-CSS]`，未删改类定义
- [ ] `[COMPONENTS-JS]` 已嵌入 body 尾部，且未新增内联 onclick
- [ ] 颜色只出现 `var(--...)`；透明色用 `color-mix(in srgb, var(--...) N%, transparent)`（唯一例外：深色面上的白色文字 `#fff`）
- [ ] 圆角、间距、阴影、字号、字重全部使用对应 token
- [ ] 所有 class 仅来自 §4 注册表，或以 `.biz-` 开头的本页业务类
- [ ] 业务样式未覆盖/复用组件类名（.biz-* 与组件类零交集）
- [ ] 表格状态列用 `.tag` 语义色，操作列用 `.btn link`
- [ ] ≥4 列宽数据表外包 `.table-wrap`（v1.6.6：横向滚动只发生在表格上，卡片标题与说明不跟着位移）
- [ ] 文件零外部依赖（无 CDN、无外链字体图片），双击可开
- [ ] 随机删掉页面上半数说明性文字后，核心任务仍可完成；删了就走不下去的，回去改 UI 而不是补文案
- [ ] 页面内不存在「功能卖点 / 设计说明 / 交互提示 / 控件的自我解释」四类文案
- [ ] 每个区块常驻辅助说明 ≤1 句，且不与页面标题、字段标签、按钮文案重复
- [ ] 未整读组件库 HTML：仅按 `#panel-{id}` 锚点抄取用到的结构模板
- [ ] 切换型按钮（非 toggle 的纯提交/取消按钮不要求）带 `aria-pressed="true|false"` 反映 on/off 状态；普通 `<button>` 不需 aria-pressed
- [ ] 浮层（Modal / Drawer / Popover / Dropdown / Popconfirm）支持 Esc 关闭，关闭后焦点返回触发器；模态期间背景 inert、Tab 不逃逸
- [ ] 下拉类浮层（Dropdown / Cascader / Select 自定义弹层）支持 ↑/↓ 方向键在可选项间循环，Home/End 跳首尾；与 shadcn ui 基于 Radix UI 的键盘行为对齐（WAI-ARIA Authoring Practices）
- [ ] §4 注册表每条组件条目都带 `variants:` 与 `a11y:` 两个元数据子段（v1.4.0 起强制、v1.6.4 已全量回填 55/55）；新增组件入库必须同时写入，维度取值须与 `[COMPONENTS-CSS]` 实际支持的修饰类一致，禁止凭空声明
- [ ] 响应式断点块位于 `[COMPONENTS-CSS]` **区末**，其后不得再出现同优先级基础类定义（v1.6.5 修复的顺序陷阱）
- [ ] 隐藏元素用 `hidden` 属性即可（快照区② 已内置 `[hidden]{display:none!important}`），勿用行内 `display` 或另造隐藏类覆盖
- [ ] 未使用组件库画廊层的 `u-*` / `data-bg` / `data-fg` 演示工具类（仅存在于 `[GALLERY-CSS]`，不随快照带走；生成页只用 §4 注册表类与 `.biz-*`）
- [ ] 响应式：≤1100px 栅格降为 2 列、≤900px 内容单栏全宽且**仅 ≥4 列宽数据表**可横向滚动（2~3 列表格须自适应折行）、`.descriptions` 降 2 列、≤600px 栅格与 `.descriptions` 降 1 列（断点基线已内置在 `[COMPONENTS-CSS]` 末尾，随快照带走，勿删改）；页面骨架若需窄屏抽屉导航，参照组件库 `[GALLERY-CSS]` / `[GALLERY-JS]` 实现（画廊层不进快照）

### 0.5 信息流向（维护者规则，AI 不反向执行）

```
design.md + 组件库HTML（源） ──生成──> 业务页面（产物，永不反向修改源）
```

业务页面中的新模式：先以 `.biz-*` 实现于单页 → 复用出现第 2 次时按 §5 准入流程提炼入组件库，不直接改页面复用。

---

## 1. 设计原则

1. **克制**：一屏一个 primary 主按钮；色彩只用于表达状态与层级，不做装饰；
2. **token 化**：一切视觉常量（颜色/字号/圆角/间距/阴影）来自 `:root` 变量，出现第二处相同硬编码值即为 BUG；
3. **组件即真身**：演示页展示的效果与实际 CSS 完全一致，「看即所得」；
4. **语义优先**：类名表达用途与状态（`.tag.green` = 成功），禁止用视觉命名（禁止 `.red-tag2`）；
5. **单文件自包含**：产出页面双击即开，零外部依赖。

### 1.1 风格基线：Semi 现代 SaaS

**风格定位**：企业级中后台 / 现代 SaaS（唯一基准 **Semi Design**）。

**风格由三层共同决定**（影响从大到小）：

| 层 | 决定什么 | sloped 对应 |
|---|---|---|
| 基础规范 Tokens | 调性基调（一眼像不像） | §2 色板 / 圆角 / 阴影 / 字号 / 间距 |
| 基础组件 | 观感与手感（用起来像不像） | §4 组件形态、尺寸档位、状态表现 |
| 页面骨架与密度 | 整页气质（整页像不像） | §3 骨架、留白比例、分块方式 |

**风格特征（新增组件与生成页面必须保持）**：

1. 白底为主 + 极浅灰分块，靠 1px 细边框（`--border-2`）分隔，**阴影极轻**（仅浮层用 `--shadow-2` 及以上）
2. 主色单一克制（`--primary`），色彩只服务于主操作与状态，大面积留白
3. 圆角按 Semi 归组（见 variables.scss）：按钮/标签/输入框/文本域 3px（`--radius-sm`，Semi border-radius-small）；菜单类浮层（dropdown/cascader） 6px（`--radius-md`，border-radius-medium）；卡片 12px（`--radius-lg`，border-radius-large）；少用大圆角
4. 中等密度：控件高 32px（大 40 / 小 28），行列紧凑、信息密度高
5. 字号以 12/14/16 三档为主，字重 400/500/600，避免 700 泛用
6. 状态用浅底标签（`*-light` 底 + 语义色字），不用高饱和实心块铺面
7. 聚焦态用柔光环（`--focus-ring`，3px 低透明主色），不做粗描边
8. 微交互克制：约 150ms 过渡，hover 仅轻微加深/变边色

**反面清单（禁止）**：重投影、大圆角胶囊化、饱和渐变、大面积彩色块、玻璃拟态、拟物、过度动画。

**判定口诀**：一眼是「干净、轻盈、信息清晰的企业后台」，而不是「营销落地页或玩具应用」。

---

## 2. Design Tokens（变量全集）

### 2.1 颜色

| Token | 值 | 用途 | 使用规则 |
|---|---|---|---|
| `--primary` | #0064FA | 主按钮、链接、选中态、聚焦边框 | 每屏主操作最多 1 个 |
| `--primary-hover` | #0062D6 | 主色悬停 | 仅 hover 态 |
| `--primary-active` | #0050B3 | 主色按下 | 仅 active 态 |
| `--primary-disabled` | #98CDFD | 主色禁用 | 禁用态自动 |
| `--primary-light` | #EAF5FF | 次要按钮底、信息标签底 | 浅底配 `--primary` 文字 |
| `--primary-light-hover` | #C7E7FE | 浅主色底悬停 | 仅 hover 态 |
| `--success` | #3BB346 | 成功状态 | 配 `--success-light` 底 |
| `--success-light` | #ECF7EC | 成功标签底 | — |
| `--warning` | #FC8800 | 警告状态 | 配 `--warning-light` 底 |
| `--warning-light` | #FFF7E8 | 警告标签底 | — |
| `--danger` | #F93920 | 危险/错误/删除 | 配 `--danger-light` 底；删除类操作需二次确认 |
| `--danger-hover` | #D52515 | 危险悬停 | 仅 danger 按钮 hover 态 |
| `--danger-active` | #B2140C | 危险按下 | 仅 danger 按钮 active 态（Semi red-7 实测值，按下瞬间反馈色，现版仅登记 token，接入按钮按下态待后续版本） |
| `--danger-light` | #FDF0EC | 危险标签底 | — |
| `--rating` | #FAC800 | 评分星选中色（Semi rating-star 实测值） | 仅评分组件使用 |
| `--purple` | #722ED1 | 扩展语义色：标签 `.tag.purple` 文字 | 配 `--purple-light` 底，仅中性分类扩展用 |
| `--purple-light` | #F5E8FF | 标签 `.tag.purple` 浅底 | 仅标签 purple 变体 |
| `--cyan` | #13C2C2 | 扩展语义色：标签 `.tag.cyan` 文字 | 配 `--cyan-light` 底，仅中性分类扩展用 |
| `--cyan-light` | #E6FFFB | 标签 `.tag.cyan` 浅底 | 仅标签 cyan 变体 |
| `--magenta` | #EB2F96 | 扩展语义色：标签 `.tag.magenta` 文字 | 配 `--magenta-light` 底，仅中性分类扩展用 |
| `--magenta-light` | #FFF0F6 | 标签 `.tag.magenta` 浅底 | 仅标签 magenta 变体 |
| `--mask` | rgba(22,22,26,.6) | 浮层遮罩（Semi modal mask 实测值） | modal/drawer 共用 |
| `--bg-toast` | rgba(28,31,35,.88) | 深色提示底：全局提示 | 深底配 #fff 白字 |
| `--text-1` | rgba(28,31,35,1) | 主文字（标题、正文重点） | — |
| `--text-2` | rgba(28,31,35,.82) | 次级文字（表头、label） | — |
| `--text-3` | rgba(28,31,35,.62) | 辅助文字（占位、说明） | — |
| `--border-1` | rgba(28,31,35,.08) | 常规边框：输入框、单选/开关轨道、强调分割线 | — |
| `--border-2` | rgba(28,31,35,.05) | 弱边框：分割线、卡片描边、表头底线 | 优先于 border-1 使用 |
| `--fill-1` | rgba(46,50,56,.09) | 填充底：图标底座、表头底、禁用底 | — |
| `--fill-hover` | rgba(46,50,56,.13) | 表格行悬停底 | — |
| `--fill-0` | rgba(46,50,56,.05) | 最浅填充底（Semi fill-0）：滑动条轨道、评分未选中星 | — |
| `--bg-page` | #F7F8FA | 页面背景 | — |
| `--bg-card` | #FFFFFF | 卡片/容器背景 | — |
| `--code-bg` / `--code-text` | #0A1626 / #C6D6F0 | 代码块底/文字 | 深底配 `--overlay-fill` 控件 |

### 2.2 侧边导航深色区（左导航布局时整组使用）

| Token | 值 | 用途 |
|---|---|---|
| `--nav-bg-1` / `--nav-bg-2` | #0A1626 / #16243A | 侧栏渐变起止 |
| `--nav-text` | #C9D6F3 | 侧栏默认文字 |
| `--nav-text-dim` | #6B7F9E | 分组标题 |
| `--nav-text-sub` | #7F93B8 | 品牌副标题 |
| `--nav-hover-bg` | rgba(255,255,255,.06) | 悬停底 |
| `--nav-active-bg` | rgba(0,100,250,.18) | 选中底 |
| `--nav-divider` | rgba(255,255,255,.08) | 深色面分隔线（品牌区下边框） |
| `--overlay-fill` | rgba(255,255,255,.1) | 深色面控件底（代码块复制钮等） |
| `--overlay-fill-hover` | rgba(255,255,255,.2) | 深色面控件 hover 底与边框 |

### 2.3 圆角 / 间距 / 字体 / 阴影

| 类别 | Token |
|---|---|
| 圆角 | `--radius-sm`(3px 小标签) · `--radius-md`(6px 按钮/输入框) · `--radius-lg`(12px 卡片/代码块) · `--radius-pill`(9999px 胶囊) |
| 间距 | `--sp-1`(4) `--sp-2`(8) `--sp-3`(12) `--sp-4`(16) `--sp-5`(20) `--sp-6`(24) `--sp-8`(32) `--sp-10`(40)，单位 px |
| 字号 | `--fs-caption`(12) `--fs-body`(14) `--fs-title`(16) `--fs-h2`(20) `--fs-h1`(24) `--fs-display`(32) |
| 字重 | `--fw-regular`(400) `--fw-medium`(500) `--fw-semibold`(600) `--fw-bold`(700) |
| 行高 | `--line-height-heading`(1.4) `--line-height-body`(1.6) |
| 字体族 | `--font-family-base`（界面） · `--font-family-code`（代码/类名） |
| 阴影 | `--shadow-1`~`--shadow-4`(由弱到强) · `--shadow-card`(白底卡片默认) |
| 焦点 | `--focus-ring`(0 0 0 3px rgba(0,100,250,.15)) · `--focus-ring-danger`(0 0 0 3px rgba(249,57,32,.12) 校验失败) · `--halo`(0 0 0 5px rgba(0,100,250,.12) 步骤条激活点光晕) |

---

## 3. 页面骨架（外层布局模板）

### 3.1 左导航 + 右内容（组件库同款，管理台类页面标配）

```html
<div class="app">
  <nav class="sidebar">
    <div class="sidebar-brand">
      <div class="logo">系统名</div>
      <div class="sub">副标题</div>
    </div>
    <div class="nav-group">分组一</div>
    <button class="nav-item active" data-target="home">首页</button>
    <button class="nav-item" data-target="list">列表</button>
  </nav>
  <main class="main">
    <section class="content-panel active" id="panel-home">
      <h1 class="page-title">首页</h1>
      <div class="block">…内容…</div>
    </section>
    <section class="content-panel" id="panel-list">…</section>
  </main>
</div>
```

约定：`.nav-item` 的 `data-target="X"` 对应面板 `id="panel-X"`，切换交互由 `[COMPONENTS-JS]` 自动接管，**不要另写导航 JS**。

`.page-desc` 非必填：仅当标题承载不了必要口径时才写一行（如「数据截至 11-20 03:00」「含已归档项目」）；复述标题的导语一律不加，见 §0.3。

### 3.2 表单页（区块直排）

```html
<main class="main">
  <h1 class="page-title">新建项目</h1>
  <div class="block">
    <div class="block-title">基本信息</div>
    <div class="col" style="gap:var(--sp-4);max-width:420px">
      <div class="form-item">
        <label class="form-label required">项目名称</label>
        <input type="text" class="input" placeholder="请输入项目名称">
      </div>
      <div class="form-item">
        <label class="form-label required">所属产品线</label>
        <select class="select"><option>Web 端</option><option>移动端</option></select>
      </div>
      <div class="form-item">
        <label class="form-label">项目描述</label>
        <textarea class="textarea" placeholder="选填"></textarea>
      </div>
    </div>
    <div class="row" style="margin-top:var(--sp-6)">
      <button class="btn md primary">提交</button>
      <button class="btn md default">取消</button>
    </div>
  </div>
</main>
```

必填由标签后的 `*` 表达，不再额外说明；确需提示时，在提交按钮旁写一行「带 * 为必填项」，**不放标题下方**。

### 3.3 列表页（工具栏 + 表格 + 空状态）

列表页 / 高级表格 / 高级表单 / 详情页 / 仪表盘 / 审批流 六个整页模板已产品化，落组件库「业务页面」分组；**生成整页时直接抄取对应面板的「结构模板」**，不要零散拼组件再自造布局。各模板的关键交互（排序/批量/搜索筛选/空态/范围切换/审批追加/表单校验）均已内置在 [COMPONENTS-JS]，随快照自动带走。

```html
<main class="main">
  <h1 class="page-title">项目管理</h1>
  <div class="block">
    <div class="row" style="justify-content:space-between;margin-bottom:var(--sp-4)">
      <div class="input-wrap">
        <span class="prefix">🔍</span>
        <input type="text" class="input" placeholder="搜索项目">
      </div>
      <button class="btn md primary">新建项目</button>
    </div>
    <!-- 有数据：<table class="table">（模板见组件库 #panel-table） -->
    <!-- 无数据：<div class="empty-state">（模板见 #panel-empty） -->
  </div>
</main>
```

### 3.4 页面级模板（v1.0.0 收录）

列表页 / 高级表格 / 高级表单 / 详情页 / 仪表盘 / 审批流 六个**整页模板**已产品化，落组件库「业务页面」分组；生成整页时直接抄取对应面板的「结构模板」，不要零散拼组件再自造布局。各模板的关键交互（排序 / 批量操作 / 搜索筛选 / 空态切换 / 图表范围切换 / 审批追加记录 / 表单校验）均已内置在 [COMPONENTS-JS]，随快照自动带走。模板由 §3.1～§3.3 的骨架叠加 §5 业务组件组合而成。

### 3.5 响应式基线（v1.6.5 建立，v1.6.6 调整表格滚动）

断点**必须置于**组件库 `[COMPONENTS-CSS]` **区末**（v1.6.5 修复：此前断点块被挤到 `.grid-cols-2/3/4` 基础定义之前，同优先级下被后者覆盖，≤1100px / ≤600px 的降列自 v1.6.2 起从未生效），**随快照自动带走**；生成页面不得删改，也不要另写一套覆盖。硬判据：断点块之后不得再出现任何同优先级的基础类定义。

| 断点 | 生效范围 | 行为 |
|---|---|---|
| ≥1201px | 桌面 | 双栏骨架，侧栏 252px（≤1400px 收至 220px）；视觉基线不变 |
| ≤1100px | 窄桌面 / 右侧预览面板 | `.grid-cols-3/4` 降为 2 列 |
| ≤900px | 平板 / 窄面板 | `.main` 与 `.block` 内边距收敛；**仅 ≥4 列的真实数据表**保留 560px 最小宽、由 `.table-wrap` 横向滚动承接（2~3 列「类 API 表」保持自适应，长类名允许折行）；`.descriptions` 降为 2 列；`.page-title` 降至 `--fs-h2`；`.form-item.inline` 改上下排布 |
| ≤600px | 手机 | 内边距再收敛、`.block` 圆角降一级；所有栅格与 `.descriptions` 降为 1 列；`.row` 间距收紧 |

约定与边界：

- **表格最小宽不得一刀切**：全库 66 张表中仅 3 张是 ≥4 列数据表（组件库「表格 / 高级表格 / 列表页」演示），其余 63 张为 2~3 列「类 API 与结构模板」表，窄屏应自然折行而非强制横向滚动；否则每个 `.block` 都变成 `overflow-x/y:auto` 的双向滚动容器，既增加滚动负担又会裁剪下拉浮层。实现用 `.table:has(thead th:nth-child(4))` 精确限定，滚动由 `.table-wrap` 承接（v1.6.6：撤掉 `.block{overflow-x:auto}`，表格横滚时卡片标题与说明不再跟着位移）。
- `.descriptions` 默认 3 列，窄屏降列由断点接管；`.descriptions-item` 为 grid 项（`min-width:0` 防长文本撑列）。若调用方内联覆盖 `grid-template-columns`，内联优先级高于断点，需自行保证窄屏可用。
- 上述断点只处理**内容层**（间距 / 栅格 / 表格 / 标题 / 表单），不改 `.app` / `.sidebar` 的显示方式——生成页的左导航骨架在窄屏仍按 §3.1 双栏呈现，避免快照在缺少触发节点时把导航藏死。
- 组件库页面自身有 68 个导航项，窄屏改为「顶部工具条 + 抽屉侧栏」，实现位于 `[GALLERY-CSS]` / `[GALLERY-JS]`（画廊层，**不进快照**）：≤900px 出现汉堡按钮；抽屉打开时遮罩 + `body.nav-open` 锁滚动；支持遮罩点击 / Esc / 选中导航项后自动收起；视口回到 >900px 自动复位。桌面端内容区另加 1180px 阅读宽度上限、切面板回到顶部（v1.6.6），均为画廊层行为，生成页不带。「类 API 与结构模板」块改用原生 `<details>` **默认折叠**（v1.6.7，62 处），展开/收起文案、悬停与焦点环由 `[GALLERY-CSS]` 提供，演示区优先。面板底部由 `[GALLERY-JS]` 按侧栏顺序生成「同组组件」索引与上一页/下一页；短面板用 `.content-panel.active { min-height:100% }` 撑满内容区、页脚沉底，翻页卡 340px、居中显示「分组 · n / N」。默认落点为「总览」面板（规模 / 怎么用 / 分组索引 / 基础变量 / 版本，v1.6.7）；桌面端 H1 滚出视口后（`scrollTop > 120`）出现 48px 固定上下文条——组件名 + 分组序号 + 回到顶部，右侧按主区滚动条宽度让位，≤900px 隐藏（由 `gallery-bar` 承担同一职责）；窄屏顶栏显示「分组 / 组件名」两级，面板 H1 改用 clip 视觉隐藏（保留标题语义），分组名与标题同前缀时自动省略，避免同一句话出现两次。画廊层另内置 `u-*` 演示工具类与 `data-bg` / `data-fg` token 映射（v1.6.7 内联样式清零的产物），仅服务组件库自身版面，**不进快照、禁止复制到生成页**。
- 若生成的管理台页面也需要抽屉导航，按上条自造 `.biz-*` 触发节点与样式，**不要修改快照区内的 `.sidebar` 定义**。
- **`[hidden]` 属性优先于组件 `display`**：快照区② 顶部已内置 `[hidden] { display: none !important }`（v1.6.5）。此前 `.row` / `.col` / `.flex` / `.grid` / `.space` 的 `display` 会盖过 UA 默认 `[hidden]{display:none}`，导致 `hidden` 属性失效；生成页隐藏元素直接用 `hidden` 属性即可，不必再自造 `.biz-hidden` 类。

---

## 4. 基础组件注册表

> 「模板」列 = 组件库 HTML 中该组件「结构模板」代码块所在面板锚点，生成时前往抄取。布局与通用类在组件库中无独立演示页，模板以本节为准。

### 4.0 分类与新增归属判据

| 分类 | 导航分组 | 现有组件 |
|---|---|---|
| 基础规范 | 基础规范 | 颜色 Colors、字号 Type Scale、圆角 Radius、阴影 Shadow、边框与分割线 Borders、间距刻度 Spacing（规范展示页，非组件，见 §2） |
| 通用组件 | 通用组件 | 按钮、标签、徽标、开关、图标、排版文本、文本收起 |
| 布局 | 布局 | 分割线、栅格、弹性布局、间距 Space |
| 表单组件 | 表单组件 | 输入框、选择器、单选复选、表单、文本域、数字输入、日期选择、上传、滑动条、级联、穿梭框、评分、时间选择、自动完成、树选择 |
| 导航 | 导航 | 步骤条、菜单、面包屑、标签页、分页、下拉菜单 |
| 数据展示 | 数据展示 | 卡片、表格、空状态、描述列表、时间线、头像、统计数值、列表、折叠面板、树、文字提示、气泡卡片、分段控制器 |
| 反馈 | 反馈 | 警告提示、进度条、加载、骨架屏、结果页、全局提示、通知、对话框、抽屉、气泡确认 |
| 业务页面 | 业务页面 | 高级表格 Advanced Table、高级表单 Advanced Form、列表页 List Page、详情页 Detail Page、仪表盘 Dashboard、审批流 Approval Flow（页面级模板，见 §5） |
| 布局与通用 | 无独立导航项（工具类） | §4.1 各布局类（.app / .sidebar / .row / .col / .hint / .icon / .code-wrap） |

**新增判据**：先判断组件领域归属 → 落入对应分组（或预留分组）；仍无法归类的复用价值高的模式，先以 `.biz-*` 落地，达标后按 §5 定组入库。**每个组件入库必须四联同步**：① 导航加项 + 面板（演示/类 API/结构模板）② `[COMPONENTS-CSS]` 加 token 化 CSS ③ 本注册表加条目 ④ changelog 记一行。

**每条组件条目须附两个元数据子段**（借鉴 shadcn/ui 文档三段式 + CVA 变体矩阵，v1.4.0 起强制，老组件逐批回填不强求）：

- **variants**：CVA 风格的「维度 × 类型 × 状态」矩阵，写明每个维度的可选值与互斥约束。例如按钮 Button：`size ∈ {sm, md, lg} × type ∈ {primary, default, secondary, danger, outline, outline.danger, dashed, link} × state ∈ {default, hover, active, disabled, loading}`；表格状态列固定走 `.tag` 语义色。该矩阵**登记在本注册表各条目的「关键禁忌」列末尾**作为入库与验收判据（v1.6.4 起 §4.2~§4.7 全 55 条补齐）；组件库面板的「类 API」块保留类名与结构模板说明，二者不重复。
- **a11y**：该组件必须满足的键盘可达 + ARIA 规范清单，逐条可勾验。至少覆盖（源自 Radix UI / shadcn/ui 默认集）：原生语义标签优先、`aria-pressed` / `aria-expanded` / `aria-current` 等状态属性按需反映、方向键遍历可选项集合、Esc 关闭 + 焦点返回、focus-visible 柔光环。

### 4.1 布局与通用（无独立演示页，直接按下表使用）

| 组件 | 类名 API | 说明与禁忌 |
|---|---|---|
| 应用骨架 | `.app` `.sidebar` `.main` | 固定组合，见 §3.1；不要在 sidebar 外用 nav 系列类 |
| 导航项 | `.nav-item[data-target]` + `.nav-group` `.sidebar-brand` | data-target 与面板 id 严格对应 |
| 内容面板 | `.content-panel` + `.active` | 同屏仅一个 active；id 必须为 `panel-{target}` |
| 页面标题 | `.page-title`，`.page-desc` 选填 | 一页一个 `.page-title`；`.page-desc` 仅在标题无法承载口径/范围时用一行，禁复述标题 |
| 内容区块 | `.block` + `.block-title` `.block-desc` `.block-label` | 承载一组信息的白底卡片容器；`.block-desc` 每区块 ≤1 句；`.block-label` 为区块内小节分隔（上距前文 20px）——区块含 ≥3 组异质范畴的演示时必须用其分节，同质样例列表（如字号刻度、间距刻度）不需分节；`.block-title` 文案必须覆盖其下全部内容范畴 |
| 行排布 | `.row` | flex 横排、居中、自动换行，gap 12px；需要其他间距用内联 `style="gap:var(--sp-*)"` |
| 列排布 | `.col` | flex 纵排，gap 16px |
| 辅助文字 | `.hint` | 12px 次要色说明文字；只写用户需要的口径/后果，不写功能讲解 |
| 图标底座 | `.icon` | 30px 浅灰底圆角块，放 emoji/字符；尺寸可内联覆盖 |
| 代码块 | `.code-wrap > .code-copy-btn + .code` | 展示 HTML 模板/命令时使用；内嵌代码须转义 `&lt;` `&gt;` |

### 4.2 通用组件

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 按钮 Button | `.btn` + 尺寸 `sm/md/lg` + 类型 `primary/default/secondary/danger/outline/outline.danger/dashed/link` + 修饰 `block/group/icon-only/loading` | #panel-btn | primary 每屏 1 个；danger 需二次确认；表格行内用 `sm link`；outline 为主色描边（Semi 同义），危险描边走 `.outline.danger`；`.icon-only` 须 aria-label；`.loading` 由调用方切换类、pointer-events 自动禁用。variants: size ∈ {sm,md,lg} × type ∈ {primary,default,secondary,danger,outline,outline.danger,dashed,link} × state ∈ {default,hover,active,disabled,loading} × modifier ∈ {block,group,icon-only}；a11y: icon-only 须 aria-label，focus-visible 柔光环 |
| 标签 Tag | `.tag` + `blue/green/orange/red/gray/purple/cyan/magenta`（可叠加 `solid/bordered/closable/checkable`） | #panel-tag | 颜色=状态语义，禁止自造组合；中性分类用 gray；closable 的 × 点击关闭所在标签；checkable 点击切换 `.checked`。variants: color ∈ {blue,green,orange,red,gray,purple,cyan,magenta} × style ∈ {default,solid,bordered} × mode ∈ {static,closable,checkable}；a11y: closable 的 × 须 aria-label，checkable 须 role=button + tabindex + Enter/Space 切换 |
| 徽标 Badge | `.badge-wrap > .icon + .badge-count` / `.badge-dot` / `.badge-status(.success/.warning/.danger/.info)` | #panel-badge | 必须包在 .badge-wrap 内；数字超 99 显示 `99+`；dot 为 8px 红点，status 为语义色圆点。variants: kind ∈ {count,dot,status} × status-color ∈ {success,warning,danger,info}；a11y: 纯视觉装饰，父级须有 aria-label 描述含义 |
| 开关 Switch | `.switch`(`.on`) + 尺寸 `.sm/.lg`；禁用 `:disabled` | #panel-switch | 交互已全局内置，禁止加内联 onclick；必须 `type="button"`；sm 36×20、lg 52×28；禁用 opacity .5。variants: size ∈ {sm,md,lg} × state ∈ {on,off,disabled}；a11y: 须 aria-label 播报开/关，role=switch，键盘可达 |
| 图标 Icon | `.icon-svg`（`.sm/.lg/.xl/.spin`）；SVG `24` 视口 + `stroke="currentColor"` | #panel-icon | 只用线性图标；颜色随父级，非必要不单独设色；spin 复用 spin-rotate 动画。variants: size ∈ {sm,md,lg,xl} × mode ∈ {static,spin}；a11y: 纯装饰图标须 aria-hidden，装饰性 SVG 不入 Tab 序 |
| 排版文本 Typography | `.text-display/h1/h2/title/body/caption` + `.text-secondary/tertiary/strong/link/ellipsis/code/mark/delete/sup/sub` | #panel-text | 正文优先用这些类，禁止随手 inline 字号；`text-ellipsis` 需配宽度；code 用等宽字体浅底，mark 用 warning-light 底，delete 删除线，sup/sub 上下标。variants: role ∈ {display,h1,h2,title,body,caption} × modifier ∈ {secondary,tertiary,strong,link,ellipsis,code,mark,delete,sup,sub}；a11y: 纯样式无语义，链接须可键盘聚焦 |
| 文本收起 EllipsisExpand | `.ellipsis-expand` + `.ellipsis-expand-content` + `.ellipsis-expand-btn`（`.show` 溢出显示 / `.expanded` 展开态）；`.suffix` 后缀式 | #panel-ellipsis | 多行省略（line-clamp），行数经 `--ellipsis-lines` 覆盖（默认 3）；交互已内置，禁止加内联 onclick；未溢出按钮自动隐藏（JS 切 `.show`，不写行内 display）；`.suffix` 时按钮跟紧靠省略号、不另起一行，文案经 `data-more`/`data-less` 覆盖。variants: mode ∈ {default, suffix} × state ∈ {collapsed, expanded} × lines ∈ {1,2,3…（--ellipsis-lines 覆盖，默认 3）}；a11y: 展开按钮用原生 button，aria-expanded 由 JS 同步，未溢出时按钮自动隐藏，focus-visible 柔光环 |

### 4.3 表单组件

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 输入框 Input | `.input`(`.lg`)；前缀组合 `.input-wrap > .prefix + .input` | #panel-input | 必填项 label 前加红 `*`；前缀时不要手写 padding-left。variants: size ∈ {md(32), lg} × state ∈ {default, hover, focus, disabled, invalid} × addon ∈ {none, prefix(.input-wrap)}；a11y: 原生 input 键盘可达，label 须 for/id 关联，invalid 必须同排配 .form-error 文案，focus-visible 柔光环 |
| 选择器 Select | `.select` | #panel-select | 选项 ≥5 用 select，<5 用单选。variants: state ∈ {default, focus, disabled, invalid} × options ∈ {<5 用单选, ≥5 用 select}；a11y: 原生 select 键盘可达，勿用自定义浮层替代轻量方案，label 须关联 |
| 单选复选 Radio | `.radio` `.checkbox`（label 包 input） | #panel-radio | 单选组 name 相同。variants: kind ∈ {radio, checkbox} × state ∈ {unchecked, checked, disabled}；a11y: 用 label 包裹原生 input 保证键盘可达，单选组 name 必须相同，成组字段用 fieldset/legend 语义 |
| 表单 Form | `.form-item`(`.inline`) + `.form-label`(`.required`) + `.form-help`/`.form-error`；控件 `.invalid` | #panel-form | 字段一律用 .form-item 包裹；错误态 .invalid + .form-error；`.form-item.inline` 字段自动撑满剩余宽度（v1.6.7）。variants: layout ∈ {vertical, inline} × state ∈ {default, required, help, invalid} × label ∈ {with, without}；a11y: label 与控件 for/id 关联，错误文案须可读且与 .invalid 同时出现，提交校验由 [data-form-validate] 内置 |
| 文本域 Textarea | `.textarea` | #panel-textarea | 多行输入用它，不用 input 撞高。variants: state ∈ {default, hover, focus, disabled, invalid} × rows ∈ {业务自定}；a11y: 原生 textarea 键盘可达，label 须关联，勿用单行 input 撑多行高度 |
| 数字输入 InputNumber | `.number`(data-step/min/max) + `.number-btn`(data-op) + `.input` | #panel-number | 步进交互已内置，勿另写。variants: state ∈ {default, focus-within, at-min(减号禁用), at-max(加号禁用)} × step ∈ {data-step}；a11y: 步进按钮须 type=button 且带 aria-label，输入框原生键盘可达，边界禁用由 JS 联动 |
| 日期选择 DatePicker | `.date-input`（input[type=date]） | #panel-date | 范围用两个 .date-input + 「至」。variants: mode ∈ {single, range(两个控件 + 「至」)} × state ∈ {default, hover, focus, invalid}；a11y: 原生 input[type=date] 键盘可达，范围模式两个控件各配 label |
| 上传 Upload | `.upload` + `.upload-list` > `.upload-item` | #panel-upload | 真实上传由业务 JS 接入，此组件为展示态。variants: state ∈ {empty(dropzone), hover, with-list} × item ∈ {file, removing}；a11y: 触发区须键盘可达（button 或 role=button），移除按钮带 aria-label；真实上传由业务 JS 接入，本组件仅展示态 |
| 滑动条 Slider | `.slider`（input[type=range]）；带值 `.slider-wrap > .slider + .slider-value` | #panel-slider | 拖动同步交互内置（进度走 `--val` 变量），勿另写 JS；禁用用原生 disabled。variants: state ∈ {default, hover, active, focus-visible, disabled} × display ∈ {bare, with-value(.slider-wrap)} × value ∈ {--val 驱动}；a11y: 原生 range 支持 ←→/Home/End 键盘调节，focus-visible 柔光环，禁用用原生 disabled |
| 级联选择 Cascader | `.cascader` + `.cascader-trigger` + `.cascader-value(.placeholder)` + `.cascader-panel > .cascader-menu[data-level] > li > .cascader-option(.has-child/.active)` | #panel-cascader | 子级选项必须带 `data-parent` 关联父级；开合、逐级展开与路径回填交互内置。variants: state ∈ {closed, open, placeholder, filled} × option ∈ {leaf, has-child, active} × level ∈ {1,2,3…（data-level）}；a11y: 触发器 button 键盘可达、focus-visible 柔光环，↑↓/Home/End 遍历与 Esc 收起按 §0.4，子级须 data-parent 关联 |
| 穿梭框 Transfer | `.transfer > .transfer-panel×2 + .transfer-actions > .transfer-btn[data-op]`；条目 `.checkbox.transfer-item`；计数 `.transfer-count` | #panel-transfer | 面板 DOM 顺序即源/目标；穿梭移动与按钮禁用联动交互内置。variants: panel ∈ {source, target} × item ∈ {unchecked, checked} × action ∈ {enabled, disabled(无勾选)}；a11y: 条目用原生 checkbox 键盘可达，方向按钮带 aria-label 且 disabled 与勾选联动 |
| 评分 Rate | `.rate[data-value] > .rate-star[data-star]`；禁用 `.rate.disabled` | #panel-rate | 星色固定走 `--rating` 禁止改色；点选与 hover 预览交互内置，禁止加内联 onclick。variants: state ∈ {default, hover-preview, active, disabled} × value ∈ {0..5（data-value）}；a11y: 星按钮须 type=button + aria-label，disabled 时禁点选，focus-visible 柔光环，星色固定走 --rating |
| 时间选择 TimePicker | `.time-input`（input[type=time]） | #panel-time | 原生时间输入美化，轻量方案；:hover/:focus/:disabled 内置；浮层面板版留待真实需求。variants: 单一尺寸 32px；a11y: 原生 input 键盘可达 |
| 自动完成 AutoComplete | `.autocomplete` + `.autocomplete-input` + `.autocomplete-list > .autocomplete-item[data-value]` | #panel-autocomplete | 聚焦/输入展开、↑↓ 遍历、Enter/点击回填、ESC/外点收起交互内置；data-value 为回填值。variants: 单一尺寸；a11y: ↑↓/Enter/ESC 键盘遍历，aria-activedescendant 由 active 类反映 |
| 树选择 TreeSelect | `.treeselect` + `.treeselect-trigger` + `.treeselect-value(.placeholder)` + `.treeselect-panel > .tree` | #panel-treeselect | 复用 .tree 结构；触发器开合、节点选中回填、外点/ESC 收起内置；开合不阻止冒泡以复用 tree-toggle。variants: 单一尺寸；a11y: 触发器 button 键盘可达，Esc 关闭 |

### 4.4 数据展示

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 卡片 Card | `.card > .card-title + .card-body` | #panel-card | 需浮起感叠加 `--shadow-2`，不用 --shadow-3+。variants: elevation ∈ {default(--shadow-card), raised(--shadow-2)} × content ∈ {title+body, body-only}；a11y: 纯容器无语义，标题用 heading 元素，勿叠加 --shadow-3+ |
| 表格 Table | `.table-wrap > .table > thead/th + tbody/td` | #panel-table | 状态列嵌 .tag，操作列放 .btn sm link；宽表外包 `.table-wrap`（v1.6.6：滚动不带动卡片标题）；>6 列考虑拆分。variants: column ∈ {≤6, >6 建议拆分} × sort ∈ {none, ascending, descending} × row ∈ {default, hover} × cell ∈ {text, .tag 状态, .btn sm link 操作}；a11y: 表头用 th 语义，排序列 aria-sort 由 JS 同步，状态列颜色须配文字 |
| 空状态 Empty | `.empty-state` | #panel-empty | 图标 40px + 一句主文案 + 引导按钮。variants: content ∈ {icon+文案+引导按钮, icon+文案}；a11y: 装饰图标 aria-hidden，引导按钮键盘可达，主文案一句说清后果 |
| 描述列表 Descriptions | `.descriptions`（grid 3 列）+ `.descriptions-item > .descriptions-label + .descriptions-value` | #panel-descriptions | 列数经内联 grid-template-columns 覆盖；value 可嵌 .tag。variants: cols ∈ {3(默认), 2, 1（≤900/≤600px 断点降级）} × value ∈ {text, .tag}；a11y: label/value 成对，建议 dl/dt/dd 语义；.descriptions-item 须 min-width:0 防窄屏撑列 |
| 时间线 Timeline | `.timeline > .timeline-item(.primary/.success/.danger)` + `.timeline-node(.timeline-dot + .timeline-tail)` + `.timeline-content` | #panel-timeline | 末项尾线自动隐藏；圆点语义色仅表状态。variants: status ∈ {default, primary, success, danger} × position ∈ {middle, last(尾线自动隐藏)}；a11y: 建议 ol/li 语义，圆点语义色仅表状态、必须配文字说明 |
| 头像 Avatar | `.avatar`（尺寸 `.sm/.md/.lg`，语义 `.success/.warning/.danger`）+ `img`；组 `.avatar-group` | #panel-avatar | 文字头像取单字；组内叠放自动负边距。variants: size ∈ {sm, md, lg} × tone ∈ {default, success, warning, danger} × content ∈ {text(单字), image} × layout ∈ {single, group}；a11y: img 须 alt，文字头像带 aria-label，组内叠放为纯视觉 |
| 统计数值 Statistic | `.statistic > .statistic-title + .statistic-value`（`.statistic-suffix` `.statistic-trend.up/.down`） | #panel-statistic | 大数字固定 20px/600；升绿降红不得反用；trend 必须带对比口径（较昨日/较上周等），不带口径的百分比禁止上屏。variants: trend ∈ {none, up, down} × suffix ∈ {none, unit}；a11y: 升绿降红不得反用，trend 必须带「较昨日/较上周/较上月」对比口径，纯展示无需焦点 |
| 列表 List | `.list > .list-item > 头像/图标 + .list-item-main(.title/.desc) + .list-item-actions` | #panel-list | 操作区放 .btn sm；末条分隔线自动隐藏。variants: layout ∈ {avatar+main+actions, icon+main, main-only} × position ∈ {middle, last(分隔线自动隐藏)}；a11y: 操作区用 .btn sm 保证键盘可达，装饰性头像 aria-hidden |
| 折叠面板 Collapse | `.collapse > .collapse-item(.open) > .collapse-header(+.collapse-arrow) + .collapse-body` | #panel-collapse | 开合交互内置；头部固定 36px。variants: state ∈ {collapsed, open} × count ∈ {1..n}；a11y: 头部用原生 button 键盘可达，aria-expanded 反映开合（§4.0） |
| 树 Tree | `.tree > .tree-node(.open) + .tree-children`（兄弟结构）；`.tree-toggle(.empty)` + `.tree-label(.active)` | #panel-tree | node 与 children 为兄弟结构；叶子节点 toggle 加 .empty；展开/选中交互内置。variants: node ∈ {branch, leaf(.empty)} × state ∈ {collapsed, open} × label ∈ {default, active}；a11y: toggle 由 JS 补 role=button + tabindex=0 保证键盘可达，↑↓ 遍历按 §4.0，node 与 children 为兄弟结构 |
| 文字提示 Tooltip | 触发 `[data-tooltip="文案"]`；浮层 `.tooltip-bubble`（JS 动态生成） | #panel-tooltip | 深底白字，文案要短；浮层勿手写，边界自动换向。variants: trigger ∈ {hover, focus} × placement ∈ {边界自动换向} × text ∈ {短文案}；a11y: 触发元素须键盘可聚焦才能触发，浮层由 JS 动态生成勿手写，文案要短 |
| 气泡卡片 Popover | `.popover` + 触发 `[data-popover]` + `.popover-card > .popover-title + .popover-content` | #panel-popover | 点击触发、外点/ESC 收起均内置；内容比 Tooltip 丰富时才用。variants: state ∈ {closed, open} × content ∈ {title+content, content-only}；a11y: 点击触发、外点/Esc 收起内置，关闭后焦点返回触发器（§0.4），内容比 Tooltip 丰富时才用 |
| 分段控制器 Segmented | `.segmented > .segmented-item(.active)` | #panel-segmented | 横向单选；点击切换 active、aria-pressed 同步内置。variants: 单一尺寸；a11y: button 键盘可达，aria-pressed 反映选中 |

### 4.5 导航

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 步骤条 Steps | `.steps` + `.steps-item.done/.active` + `.steps-node(.steps-dot + .steps-line)` + `.steps-label` | #panel-steps | 流程 ≤5 步；done 的 dot 放对勾 SVG，active/未完成放序号；末项不写 `.steps-line`。variants: state ∈ {done, active, pending} × position ∈ {first, middle, last(不写 .steps-line)} × count ∈ {≤5}；a11y: 建议 ol/li 语义，当前步 aria-current=step（§4.0），done 的 dot 放对勾 SVG |
| 菜单 Menu | `.menu &gt; .menu-item(.active/.disabled)` + `.menu-group-title` | #panel-menu | 选中切换交互内置；一级平铺，不做嵌套子菜单。variants: state ∈ {default, active, disabled} × grouping ∈ {flat, grouped}；a11y: 选中项 aria-current，disabled 项不可聚焦，一级平铺不做嵌套子菜单 |
| 面包屑 Breadcrumb | `.breadcrumb &gt; .breadcrumb-item`（中间项包 a，末项纯文本） | #panel-breadcrumb | 分隔符由 CSS 自动渲染，禁止手写「/」；末项为当前页不加链接。variants: item ∈ {link(中间项), current(末级纯文本)} × length ∈ {short, long(自动换行)}；a11y: 用 nav + ol 语义，分隔符由 CSS 生成不读作内容，末项当前页不加链接 |
| 标签页 Tabs | `.tabs &gt; .tabs-nav &gt; .tabs-tab[data-tab](.active)` + `.tabs-panel[data-tab-panel](.active)` | #panel-tabs | data-tab 与 data-tab-panel 严格对应；切换交互内置，勿另写 JS。variants: state ∈ {active, inactive} × binding ∈ {data-tab ↔ data-tab-panel 严格对应}；a11y: tab 用原生 button，建议 role=tablist/tab + aria-selected（§4.0），方向键切换，切换交互内置勿另写 |
| 分页 Pagination | `.pagination &gt; .page-btn[data-page](.active)`（prev/next）+ `.page-ellipsis` + `.page-total` | #panel-pagination | 翻页与首末页禁用联动交互内置；页码按钮只放数字与 prev/next。variants: item ∈ {prev, page, ellipsis, next} × state ∈ {default, active, disabled(首末页)}；a11y: 页码 button 键盘可达，当前页 aria-current=page，省略号 aria-hidden |
| 下拉菜单 Dropdown | `.dropdown` + 触发 `[data-dropdown]` + `.dropdown-menu &gt; li &gt; .dropdown-item(.danger)` + `.dropdown-divider` | #panel-dropdown | 开合/选中收起/外点/ESC 均内置；浮层勿再包容器；danger 项仅用于删除类操作。variants: state ∈ {closed, open} × item ∈ {default, danger, divider}；a11y: ↑↓ 循环 + Home/End 跳首尾（§0.4 已强制），Esc/外点收起，danger 仅用于删除类操作，浮层勿再包容器 |

### 4.6 布局

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 分割线 Divider | `hr.divider` / `.divider-text > span` / `.divider-vertical` | #panel-divider | 区块分隔用水平线，行内元素间用竖线；不要用 border 手搓。variants: orientation ∈ {horizontal, vertical} × content ∈ {plain, with-text}；a11y: 水平线用 hr 原生分隔语义，装饰性竖线/带文字分隔 aria-hidden，勿用 border 手搓 |
| 栅格 Grid | `.grid` + `.grid-cols-2 / .grid-cols-3 / .grid-cols-4` | #panel-grid | 等分列布局；列数经修饰类指定，间距默认 12px。variants: cols ∈ {2,3,4}；a11y: 纯布局无语义 |
| 弹性布局 Flex | `.flex` + `.flex-row/.flex-col` + `.flex-center/.flex-between/.flex-around/.flex-start/.flex-end` + `.flex-wrap/.flex-1` | #panel-flex | 比 .row 更灵活；方向与对齐可任意组合。variants: direction ∈ {row,col} × justify ∈ {center,between,around,start,end}；a11y: 纯布局无语义 |
| 间距 Space | `.space` + `.space-x/.space-y` + `.space-sm/.space-md/.space-lg` | #panel-space | 子元素间距容器；横向/纵向 + 三档尺寸。variants: dir ∈ {x,y} × size ∈ {sm,md,lg}；a11y: 纯布局无语义 |

### 4.7 反馈

| 组件 | 类名 API | 模板 | 关键禁忌 |
|---|---|---|---|
| 警告提示 Alert | `.alert` + 变体 `.success/.warning/.danger` + `.alert-icon/.alert-body/.alert-title/.alert-content` + `.alert-close` | #panel-alert | 色彩即语义，禁止自造配色；关闭交互内置。variants: tone ∈ {info(默认), success, warning, danger} × closable ∈ {static, closable}；a11y: 色彩即语义且必须配文字，关闭按钮带 aria-label，重要提示按需 role=alert/status |
| 进度条 Progress | 线形 `.progress[data-percent]` + `.progress-track > .progress-inner` + `.progress-percent`；环形 `.progress-circle[data-percent]`（SVG 双 circle） | #panel-progress | 进度由 data-percent 驱动，JS 初始化，勿手写 width/dashoffset。variants: shape ∈ {line, circle} × tone ∈ {default, success, warning(线形), danger} × value ∈ {0..100（data-percent）}；a11y: 进度值须文字可读（.progress-percent），环形建议 role=progressbar，勿手写 width/dashoffset |
| 加载 Spin | `.spin`（尺寸 `.sm/.lg`）+ `.spin-wrap` | #panel-spin | 旋转由 CSS keyframes 驱动，勿另写 JS。variants: size ∈ {sm, md(默认 20px), lg} × mode ∈ {bare, with-text(.spin-wrap)}；a11y: 装饰性 aria-hidden，配「加载中」文字说明，prefers-reduced-motion 已内置降级 |
| 骨架屏 Skeleton | `.skeleton-line`（宽度 `.w-sm/.w-md`）`.skeleton-avatar` `.skeleton-rect` `.skeleton-row` | #panel-skeleton | 仅作加载占位，完成后替换为真实内容。variants: shape ∈ {line, avatar, rect, row} × line-width ∈ {default, w-sm, w-md}；a11y: 容器 aria-busy=true、内部 aria-hidden，加载完成必须替换为真实内容 |
| 结果页 Result | `.result`（`.success/.danger`）+ `.result-icon/.result-title/.result-desc/.result-actions` | #panel-result | 图标必须语义浅底圆；主操作每屏 1 个。variants: tone ∈ {success, danger} × action ∈ {0..1（主操作每屏 1 个）}；a11y: 图标为语义浅底圆且须配文字，结果文案说清后果，主操作键盘可达 |
| 全局提示 Message | 触发 `[data-toast="类型::文案"]`；挂载 `.message-host > .message-item` | #panel-message | 深底白字，文案要短；2.5s 自动消失（JS 内置），禁止手动搭挂载点另写逻辑。variants: tone ∈ {success, info, warning, danger} × state ∈ {enter, visible, out(2.5s 自动)}；a11y: 深底白字文案要短，role=status，禁止手动搭挂载点另写逻辑 |
| 通知 Notification | 触发 `[data-notice="类型::标题::内容"]`；`.notice-host > .notice-item > .notice-icon + .notice-body + .notice-close` | #panel-notification | 右上角常驻，重要信息才用。variants: tone ∈ {info(默认), success, warning, danger} × state ∈ {enter, visible, out}；a11y: 右上角常驻仅重要信息才用，关闭按钮带 aria-label，勿与 Message 叠加同屏 |
| 对话框 Modal | `[data-modal-open="id"]` + `.modal-mask[data-mask="id"]` + `.modal#id` + `.modal-header/.modal-title/.modal-body/.modal-footer` + `[data-modal-close]` | #panel-modal | 删除类操作用 danger 主按钮并二次确认；ESC/遮罩/按钮关闭均内置；开合期间 `.app` 与 `.content-panel` 自动 inert（Tab 不逃逸），滚动锁按宽度补偿滚动条。variants: state ∈ {closed, open} × action ∈ {default, danger(删除类二次确认)} × close ∈ {Esc, 遮罩, 按钮}；a11y: aria-modal + aria-labelledby，开合期间 .app 与 .content-panel 自动 inert（Tab 不逃逸），关闭后焦点返回触发器 |
| 抽屉 Drawer | `[data-drawer-open="id"]` + `.drawer-mask[data-mask="id"]` + `.drawer#id` + `.drawer-header/.drawer-body/.drawer-footer` + `[data-drawer-close]` | #panel-drawer | 大体量内容用抽屉不用对话框；ESC/遮罩/按钮关闭均内置；开合期间背景 inert，与 Modal 同锁机制。variants: state ∈ {closed, open} × close ∈ {Esc, 遮罩, 按钮} × content ∈ {大体量详情/复杂表单/列表筛选}；a11y: 与 Modal 同锁机制（背景 inert + 滚动锁），关闭后焦点返回触发器 |
| 气泡确认 Popconfirm | `.popconfirm` + 触发 `[data-popconfirm]` + `.popconfirm-bubble > .popconfirm-title + .popconfirm-actions` + `[data-popconfirm-close]` | #panel-popconfirm | 轻量删除确认用它，重量级流程用 Modal；外点收起内置。variants: state ∈ {closed, open} × action ∈ {confirm, cancel}；a11y: 轻量删除确认用它，外点收起内置，确认/取消按钮键盘可达，重量级流程改用 Modal |

---

## 5. 业务组件（积累区）

当页面中出现第 2 次复用的模式，按以下流程提炼入库，未入库前一律以 `.biz-*` 类在单页内实现。

**准入条件（全部满足）**：
1. 在 ≥2 个场景出现真实复用（不是预期会复用）；
2. 已去除业务耦合：内容通过结构替换（文案/图标/数据不同），样式不变；
3. 命名 `.biz-` 前缀 + 语义名称（如 `.biz-user-card`），不与基础组件类重叠。

**入库流程**：
1. 组件库 HTML：新增导航项 + 面板（演示/类 API/结构模板三块齐全）；
2. 本文件 §5 登记「组件名 / 用途 / 类 API / 模板锚点 / 禁忌」；
3. 组件库 changelog 记一行。

**登记格式**（入库时照此填写）：

| 组件 | 用途 | 类 API | 模板 | 禁忌 |
|---|---|---|---|---|
| 页面工具栏 | 列表/仪表盘页顶部操作区 | `.page-toolbar`（含 `.spacer` 占位） | #panel-list-page | 主操作每屏仅 1 个；工具项左、主操作右 |
| 批量操作条 | 表格多选后的操作提示 | `.bulk-bar`（`.hidden` 隐藏态；计数 `.bulk-count`） | #panel-table-advanced | 显隐与计数由 JS 驱动，勿手写 |
| 排序表头 | 数据列点击排序 | `th[data-sort]`（aria-sort 状态） | #panel-table-advanced | 数值列 td 必须带 `data-value`，否则按文本排序 |
| 纯 CSS 条形图 | 仪表盘趋势占位 | `.biz-chart-bars > .biz-chart-bar`（高度 `--h`）+ `.biz-chart-labels` | #panel-dashboard | 仅趋势示意；精确图表接入业务图表库 |
| 表格查询联动 | 搜索/筛选/空态切换 | `[data-table-search]` `[data-table-filter]` `[data-search-empty]`（状态列 `data-filter-value`） | #panel-list-page | 全部由 JS 联动，勿另写过滤逻辑 |
| 审批操作 | 通过/驳回结果落账 | `[data-approve]` `[data-reject]` + `.approval-log` + `.approval-comment` | #panel-approval | 当前记录原位转换为结果（JS 内置）；操作后按钮自动禁用（一次性演示）；意见取 textarea 值 |
| 表单提交校验 | 必填检查与错误提示 | `[data-form-validate]`（空 → `.invalid` + `.form-error`） | #panel-form-advanced | 校验仅覆盖同 block 内 required 字段 |
| 页面级模板 ×6 | 整页结构 | 高级表格 Advanced Table / 高级表单 Advanced Form / 列表页 List Page / 详情页 Detail Page / 仪表盘 Dashboard / 审批流 Approval Flow | #panel-* | 整页抄取，勿零散拼装 |

> v1.0.0 起，上述条目随 B7 批次计划性入库（由《组件补全计划》授权，跳过 ≥2 次复用计数）；后续新增业务组件仍须满足上述准入条件。

**状态约定**：组件可选状态 `active`（默认）/ `deprecated`（废弃：注册表标注「已废弃」+替代组件，组件库面板标注，存量页面可继续用，新页面禁用）。

---

## 6. 变更记录（Changelog）

| 版本 | 日期 | 变更 | 影响面 |
|---|---|---|---|
| v0.1.0 | 2026-11-18 | 初始化：Arco 风格 token 体系 + 10 个基础组件 + 三类页面骨架；确立快照复用模式与 AI 契约 | — |
| v0.1.1 | 2026-11-18 | 导航分组重构为职责清晰的 4 组（通用组件/表单组件/数据展示/基础规范）；§4 固化新增组件归属判据与四联同步规则；修正组件库侧栏分组顺序 | 导航分组与注册表 §4.0；组件类名/API 不变，存量页面无需改动 |
| v0.1.2 | 2026-11-18 | 新增「导航」分组 + 首个组件「步骤条 Steps」（垂直，三态 done/active/默认） | 新增类 .steps*；存量页面无影响 |
| v0.1.3 | 2026-11-18 | 新增《组件补全计划.md》：对照 Ant Design v6.7.1 与 Semi 80+ 组件完成缺口分析，制定 7 批入库计划 | 纯规划文档，不影响现有组件与存量页面 |
| v0.1.4 | 2026-11-18 | 新增 §1.1 风格基线（Arco/Semi 现代 SaaS）：三层决定因素 + 8 条风格特征 + 反面清单 | 新增风格硬约束；新增组件与生成页面须遵循，现有组件已符合 |
| v0.2.0 | 2026-11-18 | B1：新增「布局」分组 + 分割线 Divider；通用组件补 图标 Icon、排版文本 Typography（均含 演示/类 API/结构模板） | 新增类 .divider* / .text-* / .icon-svg*；存量页面无影响 |
| v0.3.0 | 2026-11-18 | B2：表单组件补齐 5 个——表单 Form、文本域 Textarea、数字输入 InputNumber、日期选择 DatePicker、上传 Upload；§3.2 表单骨架改用 .form-item | 新增类 .form-* / .textarea / .number* / .date-input / .upload*；存量页面无影响 |
| v0.4.0 | 2026-11-18 | **基准对齐 Semi**：token 全量切到 Semi 真实值（品牌蓝 #0064FA、半透明边框/文字 rgba(28,31,35,α)、圆角 3/6/12、极轻阴影），风格基线改为唯一基准 Semi | 系统性视觉变更（变量名不变、值变），存量组件与页面重新生成即生效 |
| v0.5.0 | 2026-11-19 | 通用组件新增「文本收起 EllipsisExpand」：多行省略（line-clamp，行数经 `--ellipsis-lines` 覆盖默认 3）+ 展开/收起切换，交互内置、未溢出自动隐藏 | 新增类 .ellipsis-expand*；存量页面无影响 |
| v0.5.1 | 2026-11-19 | EllipsisExpand 新增后缀式变体 `.suffix`：按钮内联跟在省略号后、不另起一行；按钮文案经 `data-more`/`data-less` 覆盖 | 新增选择器 .ellipsis-expand.suffix 及 ::before；存量页面无影响 |
| v0.5.2 | 2026-11-19 | 修复 EllipsisExpand 两处缺陷：① `.suffix` 按钮缺少定位包含块，absolute 按钮跑到页面右下角不可见，补 `.ellipsis-expand.suffix { position:relative }`；② 溢出检测 `scrollHeight` 对比法在 Safari/Firefox 的 line-clamp 下恒判未溢出（按钮不显示），改为「临时解除 clamp 量自然高度」对比，并补窗口 resize 重测（100ms 节流） | 快照区② 补 1 条 CSS、快照区③ JS 变更；存量页面重新生成即修复 |
| v0.5.3 | 2026-11-19 | 后缀式 `.suffix` 调整：去掉按钮自带的 ::before 省略号，改为跟随文本原生截断省略号（按钮紧跟省略号右侧、不另起一行）；`.suffix` 按钮不再自带「…」前缀 | 快照区② 后缀 CSS 变更；存量页面重新生成即更新 |
| v0.5.4 | 2026-11-19 | 文档状态同步（无组件/样式变更）：① §4.0 分类表补齐此前漏更的 9 个组件（通用 +文本收起，表单 +表单/文本域/数字输入/日期选择/上传，§4.2/§4.3 明细注册表本就完整）；② README 版本行与组件数追平至 20 组件；③《组件补全计划》现状盘点更新、B1/B2 标记完成、B3 起版本顺延（v0.6.0～v1.0.0） | 纯文档修订，无 CSS/类名/交互变更；存量页面无影响 |
| v0.6.0 | 2026-11-19 | B3：表单组件新增 4 件——滑动条 Slider（原生 range 定制，进度经 `--val` 变量渲染、随拖动同步）、级联选择 Cascader（多列菜单逐级展开、`data-parent` 关联父子、叶子点击回填路径、外点收起）、穿梭框 Transfer（双列勾选穿梭、计数与方向按钮禁用联动）、评分 Rate（hover 预览点亮、值存 `data-value`）；token 新增 `--fill-0` rgba(46,50,56,.05) 与 `--rating` #FAC800（均 Semi 官网实测值）；视觉参数对齐 Semi 官网实测：轨道 4px/handle 24px、选项行 36px/菜单列宽 150px、面板 260px/操作钮 32px、星 24px | 新增类 .slider* / .cascader* / .transfer* / .rate*；新增 2 个 token；[COMPONENTS-JS] 扩展 4 组件交互（快照自动带走）；存量页面无影响 |
| v0.6.1 | 2026-11-19 | 修复 §2.1/§2.3 与组件库 HTML 实现不一致的色值（以 HTML [DESIGN-TOKENS] 为准的同步 BUG）：--success #00A03D→#3BB346、--success-light #E7F6E5→#ECF7EC、--warning #FF8A00→#FC8800、--danger #F93D1E→#F93920、--danger-hover #D42F14→#D52515、--focus-ring 透明度 .12→.15；补录 HTML 已有但文档缺失的 3 个 token：--primary-light-hover、--success-hover、--warning-hover | 纯文档修正，HTML 实现零改动；存量页面无影响 |
| v0.7.0 | 2026-11-19 | B4：新建「反馈」分组 + 10 组件——警告提示 Alert（四变体浅底语义色、可关闭）、进度条 Progress（线形 data-percent + 环形 72px SVG）、加载 Spin（圆弧旋转三档）、骨架屏 Skeleton（灰块脉动）、结果页 Result（成功/失败终态）、全局提示 Message（深底白字 toast，`data-toast` 触发 2.5s 自动消失）、通知 Notification（右上角 352px 卡，`data-notice` 触发）、对话框 Modal（448px/圆角 12px/遮罩+ESC+外点三重关闭）、抽屉 Drawer（右滑 448px）、气泡确认 Popconfirm（280px 向上气泡）；token 新增 `--mask` rgba(22,22,26,.6) 与 `--bg-toast` rgba(28,31,35,.88)；视觉参数对齐 Semi 官网实测（Modal/Sidesheet 448px、Notification 352px、Popconfirm 280px、遮罩 rgba(22,22,26,.6)）；[COMPONENTS-JS] 扩展浮层交互（开合/ESC/外点/toast 自动消失，快照自动带走） | 新增类 .alert* / .progress* / .spin* / .skeleton* / .result* / .message-* / .notice-* / .modal* / .drawer* / .popconfirm*；新增 2 个 token；新建导航分组「反馈」；存量页面无影响 |
| v0.8.0 | 2026-11-19 | B5：导航分组补齐 5 件——菜单 Menu（垂直、项高 40px、选中浅主色底+主色字、分组标题、禁用态）、面包屑 Breadcrumb（12px、CSS 自动「/」分隔、末项 600 字重主文字色）、标签页 Tabs（52px 标签头、active 主色+底部 2px 指示线、data-tab/data-tab-panel 声明式切换）、分页 Pagination（32px 页码钮、圆角 3px、active 浅主色底、prev/next 禁用联动）、下拉菜单 Dropdown（与 Cascader 同构浮层：菜单 padding 4px 0、行 36px、圆角 6px；danger 语义红项 + 分隔线）；全部视觉参数对齐 Semi 官网实测，零新增 token；[COMPONENTS-JS] 扩展 tabs 切换、menu 选中、pagination 翻页、dropdown 开合/收起/ESC（快照自动带走） | 新增类 .menu* / .breadcrumb* / .tabs* / .pagination 与 .page-* / .dropdown*；无新增 token；存量页面无影响 |
| v0.9.0 | 2026-11-19 | B6：数据展示分组补齐 9 件——描述列表 Descriptions（grid 3 列、label 次级色 value 主文字色、列数可覆盖）、时间线 Timeline（行高 64px、圆点语义色变体、尾线 1px）、头像 Avatar（圆形四档 24/32/40/64、语义浅底、图片填充、叠放组）、统计数值 Statistic（20px/600 大数字、单位与升绿降红趋势）、列表 List（item 45px、头像+主内容+操作区、1px 分隔线）、折叠面板 Collapse（36px 头部、开合内置）、树 Tree（节点 28px、兄弟结构展开、引导线缩进、单选选中）、文字提示 Tooltip（任意元素 data-tooltip 触发、深底白字、fixed 定位边界自动换向）、气泡卡片 Popover（240px 白底卡、标题+内容、外点/ESC 收起）；视觉参数对齐 Semi 官网实测（Descriptions 14px 键值、Timeline 尾线 1px、List padding 12/24、Collapse header 36px、Tree 节点 28px），零新增 token；[COMPONENTS-JS] 扩展 collapse/tree/popover 交互与 tooltip 动态浮层（快照自动带走） | 新增类 .descriptions* / .timeline* / .avatar* / .statistic* / .list-* / .collapse* / .tree-* / .tooltip-bubble / .popover*；无新增 token；存量页面无影响 |
| v1.0.0 | 2026-11-20 | B7 收官：新建「业务页面」分组 + 6 个页面级模板——高级表格（表头排序 aria-sort + 全选/行勾选/批量删除联动）、高级表单（分区块 + required 校验 + 折叠高级设置）、列表页（工具栏搜索/状态筛选 + 表格 + 空态切换 + 分页）、详情页（面包屑+标题行+描述列表+Tabs 分区）、仪表盘（4 指标卡 + 纯 CSS 条形图（--h 驱动、周/月范围切换）+ 待办列表）、审批流（步骤进度 + 审批记录时间线 + 通过/驳回原位更新记录与 toast）；§3 新增 3.4 页面级模板约定；§5 业务组件层首次登记 8 条（工具栏/批量条/排序表头/条形图/查询联动/审批操作/表单校验/整页模板）；新增类 .page-toolbar / .bulk-bar / th[data-sort] / .biz-chart-*；零新增 token；[COMPONENTS-JS] 扩展排序/全选/批量/搜索筛选/图表切换/审批/表单校验七组交互（快照自动带走） | 新建导航分组「业务页面」；新增 6 面板与 4 组业务类；无 token 变更；存量页面无影响；《组件补全计划》七批全部完成，核心高频覆盖目标达成，项目进入 v1.x 维护期 |
| v1.1.0 | 2026-11-20 | 新增 §0.3 内容纪律：生成页面的文案准入判据（三类保留 / 四类禁止 / 先改 UI 的替代顺序 / 空态与交付说明归属），并附可整段粘贴给外部 AI 工具的前置提示词；原 §0.3 自检清单顺延为 §0.4 并新增 3 条内容检查（半数字文案删除实验、四类禁用文案、区块辅助说明 ≤1 句、与标题/字段/按钮零重复），原 §0.4 信息流向顺延为 §0.5；§0.2 生成流程由四步改为五步；§3.1～§3.3 骨架删除示例导语（「页面说明一句话」「全部项目数据」）并明确 `.page-desc` 选填口径；§4.1 登记 `.page-desc`/`.block-desc`/`.hint` 的内容限制；快照区② 新增 1 条规则 `.page-title:not(:has(+ .page-desc))`（省略导语时标题独担 24px 段间距，实测无导语 6px→24px、有导语维持 6/24px 不变） | 契约层 + 1 条快照 CSS；无 token / 类名 / 交互改动；含导语的存量页面视觉零变化，省略导语的新页面需带新快照 |
| v1.2.0 | 2026-11-20 | **硬编码清零**：快照区② 除深色面白字 #fff 外不再出现任何裸色值——复制钮「已复制」态的 Tailwind 绿 rgba(34,197,94) 改为 color-mix(var(--success))、深色面白叠层/分隔线 token 化（新增 --nav-divider / --overlay-fill / --overlay-fill-hover）、校验失败焦点环 token 化（新增 --focus-ring-danger）、步骤条激活点光晕 token 化（新增 --halo）、复制钮文字色由外来色 #e2e8f0 统一为 --code-text；删除 3 个零引用 token（--success-hover / --warning-hover / --radius-xl 同值冗余）；补录文档缺失的 --code-bg / --code-text；#panel-colors 教学示例同步；侧栏版本号对齐。**新增画廊层**（不进快照）：[GALLERY-CSS] / [GALLERY-JS] 两个标记区——侧栏组件搜索（中文名 / 英文名 / 组件 id 过滤 + 无匹配空态）与 hash 深链（#panel-{id} 直达面板、点击导航同步地址栏）；§0.4 颜色例外条款收窄为「白字 #fff + color-mix 透明色」 | 新增 5 个 token、删除 3 个 token、新增 2 个画廊层标记区；类名 API 零变化；存量页面不受影响（旧快照自带旧 token 且无引用），重新生成即获得新色值体系；搜索与深链为画廊专用功能，生成页面不带 |
| v1.2.1 | 2026-11-20 | **§0.2 生成流程改两段式**：第一段只读契约层（§0.3 + §2 + §3）搭框架，第二段按框架逐区块查 §4 索引、按 `#panel-{id}` 锚点抄实现，明文**禁止整读组件库 HTML**、禁止引入框架用不到的组件；§0.4 新增第 12 条检查项。**浏览体验三修**（快照区② / 画廊层）：hash 深链激活后 `scrollIntoView` 滚入 active 导航项（修复 60 项长列表下深链无位置感）；新增 `@media (hover: none)` 让触屏设备复制按钮常显（原 hover 才可见，触屏不可用）；`@media (max-width: 720px)` 内 `.block` 补 `overflow-x: auto`（窄屏表格不再撑破卡片，代价：窄屏下 block 内浮层可能被裁剪，管理台以桌面为主可接受） | §0.2 流程重排 + 1 条 CSS 媒体查询 + 1 处窄屏规则 + 画廊层 1 行；类名 API 与 token 零变化；存量页面不受影响，重新生成获得触屏与窄屏兜底 |
| v1.2.2 | 2026-11-20 | 品牌残留清理：侧栏 logo「Arco UI」→「sloped UI」、favicon 字母 A→S、页面 title 加项目名「sloped · 组件库设计规范」；§1.1 删除与「唯一基准」自相矛盾的「参考 Arco 补充」表述（changelog 历史行保留原始事实不动） | 纯品牌文案与文档表述，无 CSS / 类名 / 交互 / token 变化；生成页面不受影响（§3.1 骨架 logo 位本就是「系统名」占位） |
| v1.3.0 | 2026-11-23 | **交互与无障碍打磨（无新增类 / token）**：① 统一焦点环——快照区②新增一条 `:focus-visible` 柔光环规则覆盖 `button/input/select/textarea/a` 及 `.nav-item/.cascader-trigger/.rate-star`，归并原 slider/cascader/rate 三处分散规则为专注特例，键盘 Tab 时键盘用户可见焦点、鼠标点击不显示（与 Semi 一致）；② 亮色意图——head 加 `<meta name="color-scheme" content="light">` + `:root { color-scheme: light }`，原生 select 下拉弹层与滚动条不再跟随暗色系统风格冲突；③ 动效降级——快照区②新增 `@media (prefers-reduced-motion: reduce)` 把过渡缩到 .01ms、动画迭代改为 1 次，前庭敏感用户友好；④ 浮层背景锁定——Modal/Drawer 开合期间对 `.app` 与 `.content-panel` 设 `inert`（Tab 键不逃逸到背后内容），并改用计数式滚动锁（`documentElement.paddingRight` 补偿滚动条宽度，关闭后清零）。补 nav-item `aria-current="page"` 同步。 | 新增 meta / CSS 规则 / JS 逻辑，零类名 / token / API 破坏；存量页面重新生成即生效（含焦点环、inert、动效降级、滚动条补偿） |
| v1.3.1 | 2026-11-23 | **失真修复与流程补齐（无规范变更）**：① #panel-colors「完整 Token 定义」教学镜像补齐缺失的 10 个 token（--fill-0 / --rating / --mask / --bg-toast / --focus-ring-danger / --halo / --code-bg / --code-text 及历史遗漏的 --font-family-base / --font-family-code）与 v1.3.0 的 `color-scheme`，消除「与 [DESIGN-TOKENS] 一致」标题下的镜像漂移；② references/ 5 张线上截图补齐同名 `.md` 描述并登记 `_index.md` 索引（违反 README 自定流程的洞收口：删除确认/切换确认标 `已入库`，列表页/表单页/配置弹窗三张标 `待提炼`）；③ 服务管理.html 补 favicon（字母 S data URI）并同步 color-scheme meta | 纯演示内容、描述文件与 meta 修复；零快照区类定义/交互变更；存量页面无影响 |
| v1.3.2 | 2026-11-23 | **侧栏密度档位**：快照区②新增 `@media screen and (max-width: 1400px) { .sidebar { width: 220px; } }`，1280–1400px 级笔记本整窗（含常见 1280/1366 与 Intel Mac CSS 宽）下收窄侧栏 32px 释放主区余量；该条置于 720px 小屏档之前，保证 ≤720px 时后写规则覆盖、维持 200px。纯响应式密度决策，主区 padding 不变 | 新增 1 条 CSS 媒体查询，零类名 / token / 交互变更；存量页面重新生成即获新断点（若自写侧栏宽度或依赖 252px 计算的布局需自行评估，§3.1 骨架不受影响） |
| v1.4.0 | 2026-11-24 | **Semi token 校准 + shadcn/ui 方法吸收**：① §2.1 颜色表补录 `--danger-active: #B2140C`（Semi red-7 实测值：按下瞬间反馈色，原先只有 hover 无 active），HTML [DESIGN-TOKENS] 区与教学镜像同步；本版仅登记 token，接入按钮 `:active` 态留待后续版本；② §0.4 自检清单加 3 条 a11y 项——切换型按钮 `aria-pressed`、浮层 Esc 关闭+焦点返回、下拉类 ↑↓/Home/End 遍历（源自 shadcn/ui 基于 Radix UI 的 WAI-ARIA 默认集）；③ §4.0 后追加「variants + a11y 子段」强制字段说明，CVA 风格变体矩阵 + 组件级无障碍清单，新组件入库必带、老组件逐批回填 | 新增 1 个 token、零类名 / 现有交互变更；存量页面不受影响（无引用 `--danger-active` 的场景），重新生成即获新 token 与新自检项 |
| v1.4.1 | 2026-11-24 | **通用组件视觉校准（向 Semi 靠拢）**：快照区② 修正圆角归组 + 按钮字重 + switch 轨道——① `.btn` / `.tag` / `.input` / `.select` / `.textarea` 的 `border-radius` 由 `--radius-md`(6px) 改 `--radius-sm`(3px)，对齐 Semi variables.scss「small=3px 用于 button/tag/tabs 等大多数表单控件」的官方归组（原误归 medium=6px，导致按钮/标签比 Semi 偏圆润一档）；② `.btn` 字重由 `--fw-semibold`(600) 改 `--fw-medium`(500)，对齐 Semi 按钮 medium 字重（原偏重）；③ `.switch` 关态轨道背景由 `--border-1`(.08) 改 `--fill-1`(.09)，对齐 Semi fill 阶梯（原偏淡，关态存在感弱）；④ §1.1 风格基线第 3 条同步改写圆角归组口径。零新增/删除 token、零类名变更、零交互变更，存量页重新生成即生效 | 6 处 CSS 值调整（圆角 ×5、字重 ×1、switch 轨道底 ×1），纯值变更；无 DOM 结构改动；存量页重新生成即获更贴 Semi 的方角细字开关观感 |
| v1.4.2 | 2026-11-24 | **outline 按钮语义修正**：`.btn.outline` 由 danger 描边（红字红框）改为主色描边（主色字主色框），对齐 Semi Design outline 变体默认语义；新增 `.btn.outline.danger` 修饰符保留危险描边变体（红字红框+hover danger-light 底）；#panel-btn 演示 HTML 增加「描边按钮」与「危险描边」两个并列按钮；页面说明「字重 600、圆角 6px」更新为「字重 500、圆角 3px」（v1.4.1 已修但页面文案漏同步）；§4.2 Button 行 + variants 示例同步补 `outline.danger` | 类名 API 扩展：新增 `.outline.danger`；`.outline` 语义变更为主色描边（旧用法传入 danger 视觉的存量页面需改加 `.danger` 修饰符或改为 `.btn.danger`）。属破坏性变更：`.outline` 视觉由红→蓝 |
| v1.4.3 | 2026-11-24 | **axe-core 无障碍审计修复**：快照区③ 初始化段加 1 行——为所有 `pre.code` 代码块自动设置 `tabindex="0"`，使键盘用户可 Tab 进入并滚动查看长代码，解决 axe-core `scrollable-region-focusable` serious 级违规（55 处）。全页面 WCAG 2.x AA 级审计通过：0 violations / 19 passes | 新增 1 行 JS init，零 CSS / 类名 / token 变更；存量页重新生成即获键盘可达代码块 |
| v1.5.0 | 2026-11-25 | **基础组件增量丰富（7 件，P2 暂缓清单第一档提级入库）**：① 布局组 +3——栅格 Grid（.grid + .grid-cols-2/3/4，纯 CSS）、弹性布局 Flex（.flex + 方向/对齐/撑满修饰，纯 CSS）、间距 Space（.space + .space-x/y + 三档，纯 CSS）；② 表单组 +3——时间选择 TimePicker（原生 input[type=time] 美化，三态，零 JS）、自动完成 AutoComplete（输入联想浮层，↑↓/Enter/ESC/外点内置）、树选择 TreeSelect（trigger + 复用 .tree 浮层，开合/选中回填内置）；③ 数据展示组 +1——分段控制器 Segmented（横向单选，aria-pressed 同步，点击切换内置）；均带 variants + a11y 子段；零新增 token；[COMPONENTS-JS] 扩展 segmented/autocomplete/treeselect 三组交互（快照自动带走） | 新增类 .grid*/.flex*/.space*/.time-input/.autocomplete*/.treeselect*/.segmented*；无 token 变更；存量页面无影响；《组件补全计划》P2 第一档 7 件提级入库，补齐核心高频之外的中后台实用件 |
| v1.6.0 | 2026-11-25 | **通用组件增强（汲取 shadcn/ui + Ant Design 灵感，补状态维度）**：① 按钮 Button +4 修饰——.block 撑满、.btn-group 按钮组（合并圆角）、.icon-only 图标按钮（宽=高=尺寸，须 aria-label）、.loading 加载态（::before 旋转 spinner + pointer-events 禁用）；② 标签 Tag +3——.closable 可关闭（× 点击隐藏所在标签）、.checkable 可选中（点击切换 .checked，role=button + Enter/Space）、多色 purple/cyan/magenta；③ 徽标 Badge +2——.badge-dot 红点、.badge-status 状态点（success/warning/danger/info）；④ 排版文本 Typography +5——.text-code 行内代码、.text-mark 高亮、.text-delete 删除线、.text-sup/.text-sub 上下标；⑤ 开关 Switch +2——.sm(36×20)/.lg(52×28) 尺寸、:disabled 禁用视觉；⑥ 图标 Icon +1——.icon-svg.spin 旋转动画；token 新增 --purple/--cyan/--magenta 及 light 变体（Semi 实测值）；[COMPONENTS-JS] 扩展 tag closable/checkable 交互（快照自动带走） | 新增类 .btn.block/.btn-group/.btn.icon-only/.btn.loading/.tag.closable/.tag-close/.tag.checkable/.tag.purple/.tag.cyan/.tag.magenta/.badge-dot/.badge-status/.text-code/.text-mark/.text-delete/.text-sup/.text-sub/.switch.sm/.switch.lg/.icon-svg.spin；新增 6 个 token（--purple/--purple-light/--cyan/--cyan-light/--magenta/--magenta-light）；存量页面无影响（新类无引用即不生效） |
| v1.6.0 修复 | 2026-11-25 | **边框与分割线规范面板展示增强**：原面板仅两条 1px 线（.08 vs .05 白底上几乎无视觉差异，演示无效）且缺应用场景与类 API 块；补「应用场景 In Context」区——输入框（--border-1 常规边框）、卡片描边（--border-2 弱边框）、表头底线（--border-2）三个真实组件上下文，让 token 差异在真实用法中可见；补「类 API 与结构模板」块（token 说明表 + 代码模板）；纯面板 HTML 展示层修复，零 CSS/token/快照区 JS 变更，存量页面无影响 | 仅 #panel-border 面板 HTML 展示内容增强；无类名/token/快照区变更；存量页面无影响 |
| v1.6.0 修复2 | 2026-11-26 | **内容区间距优化**：原 `.page-title` margin-bottom 6px 偏紧（标题与描述视觉粘连）、`.block` margin-bottom var(--sp-5)(20px) 小于内部 padding var(--sp-6)(24px)（卡片间分隔缝窄、区块边界感不足）；调整为 `.page-title` margin-bottom var(--sp-2)(8px)、`.block` margin-bottom var(--sp-6)(24px)，使卡片间间距 ≥ 内部 padding，层级更清晰；属快照区②安全变更，存量页面重新生成即获新间距 | 快照区② 2 处 margin 值调整（.page-title 6→8px、.block 20→24px）；零类名/token/交互变更；存量页面重新生成即获更舒适间距 |
| v1.6.0 修复3 | 2026-11-26 | **区块标题层级修复**：原 `.block-title` margin-bottom var(--sp-1)(4px) 过紧，区块标题与下方内容视觉粘连、层级不分（标题和内容混在一起）；调整为 `.block-title` margin-bottom var(--sp-3)(12px)，标题与内容拉开呼吸空间，区块标题作为统领层独立可辨；属快照区②安全变更，存量页面重新生成即获新层级 | 快照区② 1 处 margin 值调整（.block-title 4→12px）；零类名/token/交互变更；存量页面重新生成即获清晰层级 |
| v1.6.1 | 2026-11-26 | **规范页产品化优化（PM + UI 设计师评审轮）**：① 演示内容去技术化——Card 演示改为真实业务样例（项目卡 + 状态标签 + 形态标注），不再把组件规格参数塞进演示卡正文；类 API 拆为 .card/.card-title/.card-body 三行独立说明；结构模板补强调态示例；② 实现备注归位——dashboard 演示区「纯 CSS 无图表库」「--h 驱动」等面向开发者的备注从可见页面移入类 API 表格；审批流 hint 删去「（演示交互）」元信息；③ 趋势口径补齐——仪表盘/统计数值面板的全部 statistic-trend 补「较昨日/较上周/较上月」对比口径，转化率涨幅补 pp 单位；「待处理工单↓5」语义错配改为「新增工单」；④ 导航去冲突——基础规范「字体排版 Typography」改名「字体刻度 Typography」（与通用组件「排版文本」区分）；搜索占位符「搜索组件」改「搜索面板」；黑话清理（「走 token」→「统一用 token」）；⑤ 滚动条细化——侧栏/主区原生粗滚动条定制为 6/8px 细滚动条（深浅面分别配色）；⑥ 侧栏分组层级强化——.nav-group 颜色 dim→sub、组间距 sp-4/sp-1→sp-6/sp-2；⑦ 色板增强——swatch 色块与信息区加分隔线，点击复制 HEX（画廊层，不进快照）；⑧ 图表柱条 hover 加数值 title（快照区③，随范围切换同步） | 快照区② +2 处 CSS（滚动条新增、nav-group 层级）、快照区③ 图表 title 同步；面板演示 HTML 若干文案修正；画廊层新增 swatch 复制；零类名/token 变更；存量页面重新生成即获滚动条与图表提示 |
| v1.6.1 修复 | 2026-11-26 | **信息层级全量审计（67 面板结构扫描）**：脚本审计 page-title/page-desc/block/block-title/block-label/hint 层级骨架，结果——结构完整性 100%（全部面板有导语、全部 block 有标题、类 API 块均居末），实际缺陷 4 处并全修复：① 按钮面板「类型与尺寸」标题失实且 7 组异质演示混排同层级（v1.6.0 增量直接 append 所致）——标题改「类型 / 尺寸 / 扩展形态」，补 block-label 三分节（类型/尺寸与禁用/扩展形态）；② 标签面板「状态 States」标题不覆盖内容（含样式叠加/交互/扩展色）——改「形态与变体」+ 四分节（语义色/样式叠加/可交互/扩展色）；③ 徽标面板数字角标与红点状态点两组无分节——补「数字角标」「红点与状态点」两 block-label；④ 开关面板尺寸与禁用混排——补「尺寸（默认/sm/lg）」「禁用态」两分节；另修字体刻度面板 type-meta 元信息列未对齐（token 名 min-width 100px，演示层 CSS）；依据 §4.1 block-label 分节职责 + §0.3 标题准确性精神，并把该判据沉淀入 §4.1 内容区块条目（≥3 组异质演示须分节、同质刻度列表不需、标题须覆盖内容范畴），对后续新组件入库形成硬约束 | 4 个面板演示 HTML 分节 + 1 处演示层 CSS；§4.1 新增分节判据；零快照区② / token / 交互变更；存量页面无影响 |
| v1.6.1 修复2 | 2026-11-26 | **全量静态审计 + 四联同步补漏（4 类，含 2 次历史复发）**：① §2.1 颜色表补录 v1.6.0 遗漏的 6 个扩展语义色 token（--purple/--purple-light/--cyan/--cyan-light/--magenta/--magenta-light；HTML [DESIGN-TOKENS] 已定义且被 .tag.purple/.cyan/.magenta 引用，§4.2 variants 已列，唯文档 §2 零登记）；② #panel-colors「完整 Token 定义」教学镜像补同 6 个定义（73→79，消除与 [DESIGN-TOKENS] 的镜像漂移，属 v1.3.1 同类问题复发）；③ §4.0 分类表按 HTML 侧栏导航重排并补齐——新增「基础规范」行、表单组 +时间选择/自动完成/树选择、数据展示组 +分段控制器、布局组 +栅格/弹性布局/间距（属 v0.5.4 同类问题复发；明细表 §4.3/§4.4/§4.6 与 HTML 导航本就齐全）；④ 快照区② 收敛 2 处 token 外裸值——.badge-count `font-size:11px`→`var(--fs-caption)`、.text-mark `border-radius:2px`→`var(--radius-sm)`，对齐 §0.4「字号/圆角全量 token」与 §1.1 字号/圆角刻度；[DEMO-STYLES] 演示层 2 处 11px 标注字属规范视图豁免（§0.3），不动 | 纯文档同步 + 快照区② 2 处值收敛；零 token 增删、零类名 / API / 交互变更；存量页面重新生成即获 12px 徽标角标字与 3px 高亮圆角；HTML 侧栏版本号维持 v1.6.1 |
| v1.6.2 | 2026-11-26 | **响应式自适应补齐（原页面仅 2 条断点把侧栏 252→220→200px，从不收起，窄视口内容区被挤至百余像素）**：① `[COMPONENTS-CSS]` 末尾重写断点基线并随快照带走——≤1100px 栅格 3/4 列降 2 列、≤900px `.main`/`.block` 内边距收敛 + `.table` 最小宽 560px 由 `.block` 横向滚动承接 + `.page-title` 降 `--fs-h2` + `.form-item.inline` 上下排布、≤600px 内边距/圆角再收敛且栅格全降 1 列；② `[GALLERY-CSS]` 新增组件库专用「顶部工具条 + 抽屉侧栏」——≤900px 汉堡按钮唤出 272px 抽屉、遮罩 + `body.nav-open` 锁滚动、演示区固定宽度（.swatch/.type-meta/.radius-card/.shadow-box/.sp-label）改弹性、≤600px 单列；③ `[GALLERY-JS]` 新增抽屉交互（toggle / 遮罩 / Esc / 选中项自动收起 / 断点复位 / 标题同步）；④ body 新增 `#gallery-bar` `#gallery-mask` 两节点与 `#gallery-sidebar` id，均为画廊层不进快照；⑤ design.md 新增 §3.5 响应式基线、§0.4 自检清单补响应式条目；⑥ 版本四处（README / design.md 头部 / HTML 侧栏 / 工具条）同步 v1.6.2 | 桌面 ≥1201px 视觉零变化；快照区② 断点随页面带走，存量页面重新生成即获内容层自适应；生成页骨架 `.app`/`.sidebar` 显示方式未改（不破坏既有页面）；零 token / 类名 / API / 组件交互变更 |
| v1.6.3 | 2026-11-26 | **菜单对应内容层专项审查（67 面板逐项 + 浏览器窄屏实测）**：① 修 v1.6.2 响应式残留——≤900px `.table { min-width:560px }` 一刀切套到 63 张 2~3 列「类 API 表」，实测 375px 下 62/67 面板的 `.block` 全部溢出约 249px 并变成 `overflow-x/y:auto` 双向滚动容器（还会裁剪下拉浮层），改为 `.table:has(thead th:nth-child(4))` 仅对 3 张 ≥4 列真实数据表生效，并加 `.table code { overflow-wrap:anywhere }` 消除长类名撑列；② `.descriptions` 补窄屏降列（≤900px 2 列 / ≤600px 1 列，原 375px 下仍 3 列、每列仅 88px）；③ 补 `.descriptions-item { min-width:0 }`——§4.4 与类 API 表声明该类名，但 CSS 零定义，现声明成立且防 grid 长文本撑列；④ 清掉 10 处演示 `href="#"`（含结构模板 2 处）——点击会滚到页面顶部，且结构模板被复制进生成页后同样带此副作用；保留 `<a>` 元素与 hover 样式并补 `cursor:pointer` 维持可点击暗示；⑤ 实测**排除一项误报**：初判「面包屑末级当前项高亮失效」，浏览器 computed style 复核确认两组末级均为纯文本无 `<a>`、主色 / 600 正常，「角色」是普通层级项、次级色 / 400 本就正确，故撤回该条修复、不新增无用 CSS；⑥ §3.5 断点表与约定、§0.4 自检条目同步收紧 | 快照区② +1 条组件 CSS（.descriptions-item）、断点规则改写、`.breadcrumb-item a` 补 cursor；面板演示 HTML 10 处去 href；纯内容层修复，零 token / 类名 / API / 组件交互变更；桌面 ≥1201px 视觉零变化（仅面包屑末级与 `href` 语义修正）；窄屏需横向滚动面板 62 → 3（均为真实宽表） |
| v1.6.4 | 2026-11-26 | **组件层打磨第一批：变体规格全量回填 + 边界态演示**（对应「下一步选 A：打磨组件」）：① 依据 §4.0「每条组件条目须附 variants + a11y，v1.4.0 起强制、老组件逐批回填」，全量补齐 **42 条**欠账，`variants:` 13 → **55**、`a11y:` 0 缺口 → **55**，§4.2~§4.7 组件条目 **55/55** 全覆盖；每个维度取值均从 `[COMPONENTS-CSS]` 真实选择器反推（如 `.input.lg` / `.invalid` / `.avatar.sm/.success` / `.progress-circle.success` / `.menu-item.disabled` / `.page-btn.active`），a11y 只写已由 JS 同步的状态（aria-checked / aria-pressed / aria-expanded / aria-sort / aria-current / inert）或 §4.0 的规范约束，不虚报「已内置」；② 按矩阵给 **input / select / textarea / date** 四个面板补 `.invalid` 校验失败态演示（危险色描边 + 焦点环 + `.form-error` 文案 + `aria-invalid`），date 另补 disabled，并同步 4 张类 API 表各 +1 行，形成「矩阵 → 演示 → 类 API」三处一致；③ 修 §4.0 措辞歧义——原写「矩阵放组件库面板类 API 块」，实际登记在 §4 明细表「关键禁忌」列，现明确二者分工不重复；④ §0.4 自检清单补「§4 条目须带 variants + a11y 且维度须与 CSS 一致」硬判据 | 纯文档 §4 追加 42 行元数据 + 4 面板演示 HTML + 4 张类 API 表各 1 行；**零 token / 类名 / CSS / JS / 交互变更**，零新增裸值；桌面与窄屏视觉除新增错误态样例外零变化；版本四处同步 v1.6.4。剩余 51 个组件的边界态演示按同一矩阵分批推进 |

| v1.6.5 | 2026-11-26 | **端到端验证暴露的设计源缺陷回修（3 修 + 1 误报更正，按 §0.5 不反向改产物）**：① **P0** 响应式断点块从 `.grid-cols-*` 基础定义**之前**移到 `[COMPONENTS-CSS]` **区末**——同优先级下原顺序让基础定义覆盖断点，§3.5 声明的「≤1100 降 2 列 / ≤600 降 1 列」自 v1.6.2 起从未生效且随快照传导到所有生成页；浏览器探针实测 500px 下 `.grid-cols-4/3/2` 由 4/3/2 列全部修正为 1 列，1440px 桌面 4/3/2/3 列零变化。② **P1** `[COMPONENTS-JS]` 的 `openOverlay` 记录触发器、`closeOverlay` 在 `unlockBackground()` 后恢复焦点，兑现 §4.7 与 §0.4「关闭后焦点返回触发器」的 a11y 声明（原为文档虚报）。③ **P2** 快照区② 顶部补 `[hidden] { display: none !important }`，修正 `.row/.col/.flex/.grid/.space` 的 `display` 盖过 UA `[hidden]` 规则。④ **P1 误报更正**：`.modal` 的 `visibility` 过渡经 500/1440px 双视口 + 20ms 步长采样复核，仅 t=0 同步读取为 hidden、13ms 起恒 visible，属 CSS `visibility` 过渡在 t=0 取起始值的规范行为，人眼无感；当时「空白白板」为 headless 同步截图假象，**源侧 CSS 零改动**，产物 `.biz-modal-wide *` 兜底保留但不作为源缺陷。§3.5 补断点须在区末的硬判据、§0.4 补 2 条自检、版本四处同步 | 快照区② 断点块位置 + 1 条全局规则、快照区③ 焦点恢复逻辑；零 token / 类名 / 组件 API / 视觉变更；桌面 ≥1201px 视觉零变化；**存量页面按惯例重新生成即获修复**，`pages/新增用户限流配置-弹窗.html` 为 v1.6.4 快照、不反向修改 |
| v1.6.6 | 2026-11-27 | **视觉 / 排版 / 信息层级第一批（8 项，零组件 API 变更）**：① `[GALLERY-JS]` 切面板重置滚动（实测旧 scrollTop 1200 → 新面板被夹到 449，落在页底；现归零）；② `.content-panel` 加 1180px 阅读上限（1920 下 `.block` 原 1577px、行长过百字）；③ 色板由 `flex + 固定 148px` 改 `grid auto-fill/minmax(160px,1fr)`，各行共用列轨道（原右缘 1124/1288/632/960 四种；160px 下限保证 `--primary-disabled` 等长 token 名单行不折行）；④ 新增 `.table-wrap` 承接宽表横滚，撤掉 `.block{overflow-x:auto}`（500px 下原整卡 461/584 滚动，卡片标题被一起推走），66 张表已包裹；⑤ `.swatch .chip` 补 inset 1px 描边，`--warning-light` 等浅色 token 不再与白卡同色；⑥ `.page-desc` 改 `--text-2`（4.41 → 8.76:1）、搜索占位改 token 混合色（3.42 → 5.9:1）；⑦ 复制钮补 `:focus-within` / `:focus-visible` 显形，键盘焦点不再落到透明按钮上；⑧ `.block` 内边距 28→24、`.main` 底部 60→40，回到 4px 刻度 | 快照区②：新增 `.table-wrap`、`.block`/`.main` 内边距、`.page-desc` 色值、复制钮焦点态、≤900 断点移除 `.block` 横滚；画廊层：1180 上限、色板栅格、滚动重置；桌面 ≥1201px 视觉除上述 8 处外零变化；存量页面重新生成即获新规则 |
| v1.6.7 | 2026-11-27 | **视觉 / 信息层级第二批（Batch B）+ 命名统一（Batch C 部分）**：① 62 个「类 API 与结构模板」块改原生 `<details>` 默认折叠，折叠态 74px（原 502–608px），演示区回到首屏；② 页尾新增「同组组件」索引 + 上一页/下一页，短面板由 `.content-panel.active { min-height:100% }` 撑满、页脚沉底（Modal 尾部空白 469 → 158px）；③ 桌面 H1 滚出后出现 48px 上下文条（组件名 + 分组 n/N + 回到顶部），右侧按滚动条宽度让位，≤900 隐藏；④ 新增「总览」默认落点（规模 / 怎么用 / 分组索引 / 基础变量 / 版本），面板数 67 → 68；⑤ 移动端顶栏改「分组 / 组件名」两级，H1 用 clip 视觉隐藏保留语义，同名前缀自动省略；⑥ 命名统一：68 个面板 H1 与侧栏逐字一致——颜色规范→颜色 Colors、字体刻度 Typography→字号 Type Scale（补 data-keywords）、圆角/阴影/边框补英文、间距 Spacing→间距刻度 Spacing、单选复选 Radio、6 个页面模板补英文；Alert 区块「变体」→「形态」；⑦ 表单演示改「垂直布局 2 列 + 水平布局」两段，去掉 280/300px 内联宽度与 inline item 混排，`.form-item.inline` 字段 flex 撑满（快照区②）；⑧ **全库内联样式清零**：257 处 DOM 内联样式改为 94 个 `u-*` 演示工具类 + 28 条 `data-bg` / `data-fg` token 映射（置于 `[GALLERY-CSS]` 末尾），60 处 API 表头 `width:34%` 收敛为一条 `.api-block .table th:first-child` 规则，5 处 `display:none` 改 `hidden` 属性并同步级联 / 表单校验 / 搜索空态三处 JS；省略号测高改 `.is-measuring` 类、按钮显隐改 `.show` 类、环形进度改 SVG `stroke-dasharray` 呈现属性，均不再写行内 style；运行时仅剩 7 处（`--val` ×3、进度宽度 ×3、上下文条 right ×1）属数据 / 环境值 | 快照区②：`.form-item.inline .input/.select { flex:1; min-width:0 }`，生成页水平表单不再需要 `style="flex:1"`；画廊层：API 折叠 / 页尾导航 / 上下文条 / 总览 / 移动顶栏 / 命名；68 面板 × 500–1920 零溢出零错误；零 token 增删；桌面 ≥1201px 除上述外视觉零变化；存量页面重新生成即获 inline 撑满，其余为画廊层行为 |
> 维护提醒：改 token 值（var 引用不变）为安全变更，存量页面下次重新生成时自然更新；新增类为安全变更；**删除或改名组件类为破坏性变更**，必须在本表登记并标注影响，同时将原类标记 deprecated 至少一个版本期。

> AI生成
