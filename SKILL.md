---
name: html-framework-builder
description: 将框架文档/知识体系/制度文件转化为带侧边导航的交互式 HTML 框架浏览器。支持封面页、侧栏导航、面包屑、挑战卡片、维度卡片、理念卡片、手风琴策略、文化宣言等14种组件。输入结构化内容描述，输出单个自包含 HTML 文件。适用于国企党建、绩效管理、企业文化、制度框架等场景。
license: MIT
version: 1.0.0
---

# HTML Framework Builder

将用户的框架性材料（文字大纲/结构化描述/制度文件要点）转化为交互式 HTML 框架文档——带封面页、侧边导航、多板块内容的探索式布局。

## 核心理念

- **探索式布局，非幻灯片**：左侧固定导航栏，用户可以任意跳转板块；不是线性翻页
- **内容语义驱动组件选择**：根据每块内容的性质（概览/挑战/维度/理念/策略/文化）自动匹配最佳 UI 组件
- **品牌色自动推算**：输入一个主色，自动生成 5 个色阶变体
- **零外部依赖**：单 HTML 文件，离线可用（Google Fonts 为增强项，可降级）
- **中文优先**：Noto Sans SC / Noto Serif SC 字体栈，serif 标题 + sans-serif 正文

---

## ⚠️ 关键规则

1. **必须先确认再生成**——绝对不能跳过 Step 0 的用户确认环节
2. **不能编造内容**——用户没有给的数据，用占位符 `[待补充]` 标注
3. **所有中文文本使用全角标点**——特别是引号必须用 `""`
4. **品牌色不能使用用户品牌之外的色系**——如"党建红"只能用红色系，不能用蓝色
5. **生成后必须质量自检**——逐项检查 Step 5 的清单

---

## Workflow

### Step 0：理解材料并确认参数（不生成代码）

从用户提供的材料中提取以下信息，并向用户确认（每次问 2-3 个问题）：

#### 0.1 全局信息（必确认）

| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `brand_name` | string | 品牌/机构名（如"美丽中国支教项目"） | — |
| `brand_name_en` | string | 英文品牌名（如"Teach For China"） | — |
| `title` | string | 框架主标题（支持 `<br>` 换行） | — |
| `subtitle` | string | 封面副标题/核心理念一句话 | — |
| `motto` | string[] | 封面座右铭，2-3个词组 | — |
| `brand_color` | string | 品牌主色 hex（如 `#E6001E`） | `#C41E3A`（中国红） |
| `enter_text` | string | 封面进入按钮文字 | `"了解框架"` |
| `footer_text` | string | 版权文字 | `"© {brand_name}"` |
| `footer_info` | string | 页脚详细信息 | — |

#### 0.2 板块结构（必须逐板块确认）

对每个板块，确认：

| 参数 | 类型 | 说明 |
|------|------|------|
| `id` | string | 唯一标识（如 `overview`, `context`） |
| `label` | string | 英文标签（如 `"Chapter 01"`） |
| `nav_label` | string | 侧栏导航中文名（如 `"框架概览"`） |
| `title` | string | 板块中文标题 |
| `subtitle` | string | 板块副标题/引言 |
| `content_type` | enum | 见下方映射表 |

#### 0.3 content_type 映射表（6 种预设 + 1 种灵活）

| content_type | 自动使用的组件 | 典型场景 | 需补充的内容参数 |
|---|---|---|---|
| `overview` | intro-block + card-grid-2(challenge-card) + 通栏卡片 | 框架总览/目录页 | `intro_text`, `cards[]` |
| `challenges` | card-grid-2(challenge-card + data-num) + question-block*N + transition-text | 问题分析/挑战列举 | `cards[]`, `questions[]`, `transition_text` |
| `dimensions` | dimension-card*N（含 ability-list） | 多维度能力/成果展示 | `dimensions[]` |
| `beliefs` | belief-cards grid（2×2） | 并列理念/原则/价值观 | `beliefs[]` |
| `strategies` | strategy-group 手风琴*N | 分组可折叠策略/措施 | `groups[]` |
| `culture` | culture-block + closing-text + page-footer | 文化宣言/组织土壤/结尾 | `words[]`, `culture_desc`, `closing_text` |
| `custom` | 用户自行指定组件列表 | 灵活组合 | 手动描述 |

#### 0.4 确认对话示例

```
我已分析您的材料，框架结构如下：

📋 全局：品牌色 #C41E3A | 标题「XXX管理办法框架」
📂 板块：
  1. overview → 框架概览（4 张概览卡片）
  2. challenges → 现状与挑战（4 个挑战 + 2 个追问）
  3. dimensions → 考核维度（3 个维度，各含 3-4 项能力指标）
  4. strategies → 实施策略（4 组手风琴，共 10 条策略）
  5. culture → 组织保障（3 个关键词 + 文化宣言 + 结尾）

请确认：
① 品牌色是否准确？
② 板块结构是否需要调整？
③ 如果有任何板块内容暂缺，我将标注 [待补充]。
```

---

### Step 1：内容类型 → 组件选择

根据用户在 Step 0 确认的每个板块的 `content_type`，按以下决策表自动拼接组件：

```
section.content_type === 'overview'
  → intro-block(text)
  → card-grid-2( cards.map→challenge-card(clickable=true) )
  → 底部通栏 challenge-card(clickable=true)   // 如有

section.content_type === 'challenges'
  → card-grid-2( cards.map→challenge-card(data-num) )
  → question-block*N
  → transition-text

section.content_type === 'dimensions'
  → dimension-card*N
    每个 dimension-card 内含 ability-list(2列)

section.content_type === 'beliefs'
  → belief-cards( belief-card*N )
    2×2 网格，每个 belief-card 含独立配色

section.content_type === 'strategies'
  → strategy-group*N
    第一个 strategy-group 默认展开(open)

section.content_type === 'culture'
  → culture-block(words, description)
  → closing-text
  → page-footer(brand, info)
```

---

### Step 2：品牌色自动推算

从用户输入的 `brand_color` 自动生成 5 个色阶：

```
输入: brand_color = "#C41E3A"

推算:
--color-brand        = brand_color              // #C41E3A 主色
--color-brand-dark   = darken(brand_color, 10%) // #9A182D 深色
--color-brand-light  = lighten(brand_color, 15%) // #E84D65 亮色
--color-brand-bg     = brand_color + "1A"       // 主色 10% 透明度背景
--color-brand-subtle = brand_color + "0D"       // 主色 5% 透明度背景
```

**辅助色预设（4 个固定色系，用于维度卡片的 dim 系统）**：

| 辅助色名 | color | color-light | dim 映射 |
|----------|-------|-------------|----------|
| green | `#2D8B4E` | `#E8F5ED` | `dim-others` |
| gold | `#C49A2A` | `#FFF8E7` | `dim-world` |
| blue | `#2563EB` | `#EFF6FF` | `dim-strategies` |
| orange | `#D97706` | `#FFF7ED` | 备用 |

---

### Step 3：生成 HTML

按照以下固定结构生成单个 HTML 文件：

```
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{title}</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+SC:wght@300;400;500;700&family=Noto+Serif+SC:wght@400;600;700;900&display=swap" rel="stylesheet">
  <style>
    /* ========== CSS 变量区 ========== */
    /* ========== 全局样式 ========== */
    /* ========== 封面页 ========== */
    /* ========== 主内容框架 ========== */
    /* ========== 侧边导航 ========== */
    /* ========== 内容区域 ========== */
    /* ========== 板块通用 ========== */
    /* ========== 引言块 ========== */
    /* ========== 卡片网格 ========== */
    /* ========== 挑战卡片 ========== */
    /* ========== 追问区 ========== */
    /* ========== 维度卡片 ========== */
    /* ========== 理念卡片 ========== */
    /* ========== 策略手风琴 ========== */
    /* ========== 组织土壤/文化块 ========== */
    /* ========== 页脚 ========== */
    /* ========== 移动端菜单 ========== */
    /* ========== 响应式 ========== */
  </style>
</head>
<body>

  <!-- ===== 封面页 ===== -->
  <div id="cover" onclick="enterSite()">...</div>

  <!-- ===== 主内容 ===== -->
  <div id="app">
    <!-- 侧边导航 -->
    <nav class="sidebar" id="sidebar">...</nav>
    <!-- 内容区域 -->
    <div class="content-area">
      <header class="content-header">...</header>
      <!-- 板块列表 -->
      <div class="section active" id="sec-{id}">...</div>
      <!-- ...更多板块... -->
    </div>
  </div>

  <!-- 移动端菜单按钮 -->
  <button class="mobile-menu-btn" id="mobileMenuBtn" onclick="toggleMobileMenu()">...</button>
  <!-- 移动端遮罩 -->
  <div class="sidebar-overlay" id="sidebarOverlay" onclick="toggleMobileMenu()"></div>

  <script>
    // ===== 封面进入/返回 =====
    // ===== 板块切换 =====
    // ===== 手风琴 =====
    // ===== 移动端菜单 =====
    // ===== 键盘导航 =====
  </script>
</body>
</html>
```

#### 3.1 CSS 变量区模板

```css
:root {
  /* === 品牌色系 === */
  --color-brand: {brand_color};
  --color-brand-dark: {brand_color_dark};
  --color-brand-light: {brand_color_light};
  --color-brand-bg: {brand_color_bg};
  --color-brand-subtle: {brand_color_subtle};

  /* === 中性色 === */
  --color-white: #FFFFFF;
  --color-bg: #FAFAFA;
  --color-bg-alt: #F5F5F5;
  --color-text: #1A1A1A;
  --color-text-secondary: #4A4A4A;
  --color-text-muted: #8C8C8C;
  --color-border: #E5E5E5;
  --color-border-light: #F0F0F0;

  /* === 辅助色 === */
  --color-green: #2D8B4E;
  --color-green-light: #E8F5ED;
  --color-gold: #C49A2A;
  --color-gold-light: #FFF8E7;
  --color-blue: #2563EB;
  --color-blue-light: #EFF6FF;
  --color-orange: #D97706;
  --color-orange-light: #FFF7ED;

  /* === 排版 === */
  --font-heading: 'Noto Serif SC', 'Source Han Serif SC', 'Songti SC', 'SimSun', serif;
  --font-body: 'Noto Sans SC', 'Source Han Sans SC', 'PingFang SC', 'Microsoft YaHei', sans-serif;
  --text-xs: 0.75rem;
  --text-sm: 0.875rem;
  --text-base: 1rem;
  --text-lg: 1.125rem;
  --text-xl: 1.25rem;
  --text-2xl: 1.5rem;
  --text-3xl: 1.875rem;
  --text-4xl: 2.25rem;
  --text-5xl: 3rem;
  --leading-tight: 1.25;
  --leading-normal: 1.7;
  --leading-relaxed: 1.85;

  /* === 间距 === */
  --space-1: 0.25rem;  --space-2: 0.5rem;
  --space-3: 0.75rem;  --space-4: 1rem;
  --space-5: 1.25rem;  --space-6: 1.5rem;
  --space-8: 2rem;     --space-10: 2.5rem;
  --space-12: 3rem;    --space-16: 4rem;
  --space-20: 5rem;    --space-24: 6rem;

  /* === 圆角 === */
  --radius-sm: 4px;  --radius-md: 8px;  --radius-lg: 12px;

  /* === 阴影 === */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.06);
  --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
  --shadow-lg: 0 8px 24px rgba(0,0,0,0.1);

  /* === 布局 === */
  --nav-width: 260px;
  --nav-height: 64px;
}
```

#### 3.2 各组件 HTML 模板

##### 封面页

```html
<div id="cover" onclick="enterSite()">
  <div class="cover-decoration">
    <div class="cover-circle"></div>
    <div class="cover-circle"></div>
    <div class="cover-circle"></div>
  </div>
  <div class="cover-brand">{brand_name}</div>
  <h1 class="cover-title">{title_line1}<br>{title_line2}</h1>
  <p class="cover-subtitle">{subtitle}</p>
  <div class="cover-motto">
    <span>{motto[0]}</span><span>·</span>
    <span>{motto[1]}</span><span>·</span>
    <span>{motto[2]}</span>
  </div>
  <button class="cover-enter" onclick="event.stopPropagation(); enterSite()">
    {enter_text}
    <svg width="16" height="16" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><path d="M3 8h10M9 4l4 4-4 4"/></svg>
  </button>
  <div class="cover-footer">{footer_text}</div>
</div>
```

##### 侧边导航

```html
<nav class="sidebar" id="sidebar">
  <div class="sidebar-brand">
    <div class="sidebar-brand-title">{title_short}</div>
    <div class="sidebar-brand-sub">{brand_name_en}</div>
  </div>
  <div class="sidebar-nav">
    {#each sections}
    <div class="nav-item {#if first}active{/if}" data-section="{id}" onclick="switchSection('{id}')">
      <span class="nav-icon"><svg viewBox="0 0 20 20">{icon_svg}</svg></span>
      {nav_label}
    </div>
    {/each}
  </div>
  <div class="sidebar-footer">
    {brand_name}<br>{brand_name_en}
  </div>
</nav>
```

##### 板块容器

```html
<div class="section {#if first}active{/if}" id="sec-{id}">
  <div class="section-label">{label}</div>
  <h1 class="section-title">{title}</h1>
  <p class="section-subtitle">{subtitle}</p>
  {children_components}
</div>
```

##### 引言块

```html
<div class="intro-block">{text}</div>
```

##### 挑战卡片

```html
<div class="challenge-card"{#if data-num} data-num="{data-num}"{/if}{#if clickable} style="cursor:pointer" onclick="switchSection('{click_section}')"{/if}>
  <div class="challenge-card-header">
    <div class="challenge-icon" style="background:{icon_bg|var(--color-brand-bg)}">
      <svg viewBox="0 0 22 22" style="stroke:{icon_stroke|var(--color-brand)}">{icon_svg}</svg>
    </div>
    <div>
      <h3>{title}</h3>
      <span class="challenge-sub" style="color:{sub_color|var(--color-brand)}">{sub}</span>
    </div>
  </div>
  <p>{description}</p>
</div>
```

##### 卡片网格

```html
<div class="{grid_class}">
  {cards_html}
</div>
<!-- grid_class: card-grid-2 (2列), card-grid-4 (2列紧凑), belief-cards (2列理念) -->
```

##### 追问块

```html
<div class="question-block">
  <div class="question-label">{label}</div>
  <p class="question-text">{text}</p>
</div>
```

##### 过渡文字

```html
<div class="transition-text">{text}</div>
```

##### 维度卡片

```html
<div class="dimension-card dim-{dim}">
  <div class="dimension-header">
    <div class="dimension-icon">
      <svg viewBox="0 0 26 26">{icon_svg}</svg>
    </div>
    <div>
      <div class="dimension-title">{title} <span class="dimension-title-sub">{title_sub}</span></div>
    </div>
  </div>
  <p class="dimension-desc">{description}</p>
  <div class="ability-list">
    {#each abilities}
    <div class="ability-item">
      <span class="ability-dot"></span>
      <div>
        <div class="ability-name">{name}</div>
        <div class="ability-desc">{desc}</div>
      </div>
    </div>
    {/each}
  </div>
</div>
```

##### 理念卡片

```html
<div class="belief-cards">
  {#each beliefs}
  <div class="belief-card">
    <div class="belief-icon" style="background:{icon_bg|var(--color-brand-bg)}">
      <svg viewBox="0 0 24 24" style="stroke:{icon_stroke|var(--color-brand)}">{icon_svg}</svg>
    </div>
    <h3>{title}</h3>
    <p>{description}</p>
  </div>
  {/each}
</div>
```

##### 策略手风琴

```html
<div class="strategy-group {#if default_open}open{/if}">
  <div class="strategy-header" onclick="toggleAccordion(this)">
    <div class="strategy-header-left">
      <span class="strategy-num">{num}</span>
      <span class="strategy-title">{title}</span>
    </div>
    <span class="strategy-arrow">
      <svg viewBox="0 0 20 20"><path d="M5 8l5 5 5-5"/></svg>
    </span>
  </div>
  <div class="strategy-body">
    <div class="strategy-list">
      {#each strategies}
      <div class="strategy-item">
        <h4>{title}</h4>
        <p>{description}</p>
      </div>
      {/each}
    </div>
  </div>
</div>
```

##### 文化宣言块

```html
<div class="culture-block">
  <div class="culture-words">
    {#each words}<span class="culture-word">{word}</span>{/each}
  </div>
  <p>{description}</p>
</div>
```

##### 结语

```html
<p class="closing-text">{text}</p>
```

##### 页脚

```html
<div class="page-footer">
  <div class="footer-brand">{brand}</div>
  <div class="footer-info">{info}</div>
</div>
```

#### 3.3 JS 交互逻辑（固定模板，参数化 sections 数组）

```javascript
// ===== 封面进入 =====
function enterSite() {
  const cover = document.getElementById('cover');
  const app = document.getElementById('app');
  cover.classList.add('leaving');
  setTimeout(() => {
    cover.classList.add('hidden');
    app.classList.add('visible');
  }, 600);
}

// ===== 返回封面 =====
function backToCover() {
  const cover = document.getElementById('cover');
  const app = document.getElementById('app');
  cover.classList.remove('hidden', 'leaving');
  app.classList.remove('visible');
}

// ===== 板块切换 =====
const sectionNames = {
  // 由 sections 数组自动生成
  // "{id}": "{nav_label}"
};
const sectionOrder = [/* 由 sections 数组自动生成: id列表 */];

function switchSection(id) {
  document.querySelectorAll('.nav-item').forEach(item => {
    item.classList.toggle('active', item.dataset.section === id);
  });
  document.querySelectorAll('.section').forEach(sec => {
    sec.classList.remove('active');
  });
  const target = document.getElementById('sec-' + id);
  if (target) target.classList.add('active');
  document.getElementById('breadcrumb-current').textContent = sectionNames[id] || id;
  // 移动端关闭菜单
  const sidebar = document.getElementById('sidebar');
  const overlay = document.getElementById('sidebarOverlay');
  if (sidebar.classList.contains('mobile-open')) {
    sidebar.classList.remove('mobile-open');
    overlay.classList.remove('visible');
  }
  window.scrollTo(0, 0);
}

// ===== 手风琴 =====
function toggleAccordion(header) {
  header.parentElement.classList.toggle('open');
}

// ===== 移动端菜单 =====
function toggleMobileMenu() {
  const sidebar = document.getElementById('sidebar');
  const overlay = document.getElementById('sidebarOverlay');
  sidebar.classList.toggle('mobile-open');
  overlay.classList.toggle('visible');
}

// ===== 键盘导航 =====
document.addEventListener('keydown', (e) => {
  const currentIdx = sectionOrder.findIndex(s => {
    const el = document.getElementById('sec-' + s);
    return el && el.classList.contains('active');
  });
  if (e.key === 'ArrowDown' || e.key === 'ArrowRight') {
    e.preventDefault();
    const next = sectionOrder[(currentIdx + 1) % sectionOrder.length];
    switchSection(next);
  } else if (e.key === 'ArrowUp' || e.key === 'ArrowLeft') {
    e.preventDefault();
    const prev = sectionOrder[(currentIdx - 1 + sectionOrder.length) % sectionOrder.length];
    switchSection(prev);
  }
});
```

#### 3.4 完整 CSS（标准输出，始终包含）

生成时必须包含以下完整 CSS 内容（从复刻模板提取并简化）：

**注意**：以下 CSS 片段为核心结构，完整 CSS 应包含所有 14 种组件的样式定义。生成时使用 `/workspace/teaching-as-leadership-2.0.html` 中的完整 CSS 作为基础，替换品牌色变量即可。

关键 CSS 要点：
- `#cover` 固定定位全屏，背景 `var(--color-brand)`，进入动画 0.6s 淡出+微放大
- `.sidebar` 固定左侧 260px，白色背景，右边框
- `.section` 默认 `display:none`，`.active` 后 `display:block`，fadeIn 动画
- 响应式断点 768px：侧边栏 `translateX(-100%)` 隐藏，内容区 `margin-left:0`
- 手风琴采用 `max-height` 过渡动画

---

### Step 4：SVG 图标系统

侧边导航使用 6 个内置 SVG 图标（根据板块内容语义自动匹配）：

| 图标 | 语义 | SVG viewBox |
|------|------|-------------|
| overview | 文档/概览 | `<rect x="3" y="3" width="14" height="14" rx="2"/><line x1="7" y1="7" x2="13" y2="7"/><line x1="7" y1="10" x2="13" y2="10"/><line x1="7" y1="13" x2="10" y2="13"/>` |
| clock | 时间/背景 | `<circle cx="10" cy="10" r="7"/><path d="M10 6v4l3 2"/>` |
| star | 成果/目标 | `<path d="M10 3l2 4h4.5l-3.5 3 1.5 4.5L10 12l-4.5 2.5L7 10 3.5 7H8z"/>` |
| heart | 理念/信念 | `<path d="M10 17s-7-4.35-7-8.5C3 5.91 5.24 4 8 4c1.5 0 2 .5 2 .5S10.5 4 12 4c2.76 0 5 1.91 5 4.5 0 4.15-7 8.5-7 8.5z"/>` |
| trend | 策略/行动 | `<path d="M4 14l4-4 3 3 5-5"/><path d="M14 5h3v3"/>` |
| person | 组织/文化 | `<path d="M3 10c0-3.87 3.13-7 7-7s7 3.13 7 7"/><path d="M7 14c0-1.66 1.34-3 3-3s3 1.34 3 3"/><circle cx="10" cy="17" r="1"/>` |

卡片内 SVG 图标根据内容语义现场设计（16-26px viewBox，1.5px stroke-width）。

---

### Step 5：质量自检（必须执行，不可跳过）

生成后逐项检查：

```
□ 封面页
  □ 品牌色背景正确
  □ 标题、副标题、座右铭完整
  □ 点击"了解框架"按钮可进入主内容
  □ 点击封面任意区域可进入主内容

□ 侧边导航
  □ 所有板块显示正确
  □ 点击导航项可切换板块
  □ 当前激活项有红色左边框高亮
  □ 面包屑文字同步更新

□ 内容板块
  □ 所有板块可正常切换
  □ 概览卡片（如有）点击可跳转到对应板块
  □ 手风琴（如有）展开/收起正常，箭头旋转
  □ 所有中文文本使用全角标点
  □ 无 [待补充] 外的空白内容

□ 返回封面
  □ "返回封面"按钮可用
  □ 返回后再次进入，初始板块为框架概览

□ 键盘导航
  □ ← → 键可切换板块
  □ Escape/特殊键不冲突

□ 响应式（缩放浏览器窗口到 < 768px）
  □ 侧边栏隐藏，右下角出现浮动菜单按钮
  □ 点击菜单按钮可打开/关闭侧边栏
  □ 卡片网格变为单列

□ 其他
  □ 浏览器控制台无 JS 报错
  □ 中文显示正常，无乱码
  □ 所有 SVG 图标可见
```

**如果任何一项不通过，必须修复后重新检查。**

---

### Step 6：预览

生成完成后：
1. 保存到 `/workspace/{title-slug}.html`
2. 使用 preview skill 启动预览

---

## 颜色推算参考

从 `brand_color` 推算深/亮色的辅助函数逻辑：

```
深色 (dark)：RGB 各通道 × 0.78
亮色 (light)：RGB 各通道 × 1.15（上限 255）
背景色 (bg)：brand_color + "1A"（hex 透明度）
微妙色 (subtle)：brand_color + "0D"
```

示例：
- `#C41E3A` → dark=`#99172D` light=`#E62344` bg=`#C41E3A1A` subtle=`#C41E3A0D`
- `#E6001E` → dark=`#B30017` light=`#FF1A38` bg=`#E6001E1A` subtle=`#E6001E0D`

---

## 注意事项

1. **不要凭空创造数据**——用户没说某板块有几条能力指标，就先写 3-4 条模板再让用户补充
2. **中文标点**——所有引号用 `""`（全角），不用 `""`（半角）；书名号用 `《》`
3. **板块数量建议**——最佳体验 3-7 个板块，少于 2 个不需要侧边栏，多于 8 个考虑合并
4. **移动端体验**——移动端侧边栏默认隐藏，通过右下角浮动按钮打开
5. **字体降级**——Google Fonts 加载失败时自动降级到系统中文字体
