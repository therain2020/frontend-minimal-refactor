# Frontend Minimal Refactor Methodology

从 Vue 3 + Element Plus + SCSS 项目迁移到纸墨质感极简设计系统的系统性方法论。

## 核心原则

1. **设计先行** — 先有 DESIGN.md，再有代码。所有视觉决策回归设计令牌。
2. **令牌驱动** — CSS 变量是单一真相源。组件不硬编码颜色/间距/字体。
3. **先删后建** — 审计现有组件，删除被框架覆盖的冗余，再重写保留的。
4. **逐页迭代** — 不改一个全局开关然后祈祷。每个页面独立设计判决。
5. **设计审查闭环** — 每完成一组，浏览器看效果。不行继续改。不依赖 HMR 的"应该行了"。

## 工作流

```
/design-consultation → DESIGN.md (设计令牌: 配色/字体/间距/圆角/阴影/动效)
         ↓
/frontend-design → 组件层重构 (分轮次: 通用组件先, 页面后)
         ↓                     ↓
    design-tokens.css     每轮: 浏览器验证 → 调整 → commit → 下一轮
         ↓
/design-review → 视觉 QA (一致性/间距/暗色/对比度)
         ↓
/qa-only → 全量回归
```

## 设计令牌结构 (design-tokens.css)

```css
:root {
  /* 纸墨中性灰阶 — 11 级 */
  --color-neutral-50: #FEFCF8;   /* 页面背景 */
  --color-neutral-100: #F6F3EF;  /* 容器/hover */
  --color-neutral-200: #EDE9E4;  /* 边框/分割线 */
  /* ... 50 到 950 */

  /* 暖橙强调 — 唯一打破灰阶的颜色 */
  --color-accent: #C2640D;
  --color-accent-soft: #EBB17B;

  /* 间距 — 4px 基准, 10 级 */
  --space-1: 4px; --space-2: 8px; --space-3: 12px;
  --space-4: 16px; --space-5: 24px; --space-6: 32px;
  --space-8: 48px; --space-10: 64px; --space-12: 80px; --space-16: 128px;

  /* 字体 — clamp 响应式 */
  --font-size-h1: clamp(2.5rem, 6vw, 4rem);
  --font-size-body: 1rem;
  --font-size-small: 0.875rem;
  --font-size-caption: 0.75rem;

  /* 圆角 — 6 级 */
  --radius-xs: 2px; --radius-sm: 4px; --radius-md: 8px;
  --radius-lg: 16px; --radius-xl: 24px; --radius-full: 9999px;

  /* 阴影 — 5 级纸质层级 (微妙 rgba) */
  --shadow-1: 0 1px 2px rgba(0,0,0,0.03), 0 1px 3px rgba(0,0,0,0.05);
  /* ... --shadow-5: 模态框/最高层 */

  /* 动效 */
  --transition-fast: 150ms ease;
  --transition-base: 250ms ease;
}

[data-theme="dark"] {
  --color-neutral-50: #1A1816;
  --color-accent: #F0A94B;
  /* 完整反转中性灰阶, 强调色调亮 */
}
```

## 组件重构轮次

### Round 1: 通用组件 (最高杠杆)

先审计，后删除。40 个组件中，12 个被 Element Plus 完全覆盖或功能重复。

**删除清单:**

| 删除 | 替代 |
|------|------|
| `ui/Button, Card, Input, Modal, SearchBox` | `el-button, el-card, el-input, el-dialog` |
| `common/CustomTable, CustomTableWithPagination, Pagination` | `el-table + el-pagination` |
| `common/Message, ToastMessage` | `ElMessage` (element-plus) |
| `AnimatedBackground` ×2 | 纸墨风不需要动画背景 |
| `ThemeToggle` | 内联到 NavigationBar |

**重写:** NavigationBar, ThemedPageContainer, App.vue (footer + shell)

**影响:** 44 文件，+400/-3500 行。净删 3100 行死代码。

### Round 2: 业务组件 (逐组件打磨)

书卡 (RecommendationCard) 经历了 6 次迭代——这个过程揭示了方法论的真正核心：

| v | 方案 | 问题 | 教训 |
|---|------|------|------|
| 1 | 封面图+文字竖排 | 太像产品卡片，与纸墨气质冲突 | 不要用"web 组件"思维设计——想"如果这是一张纸" |
| 2 | 书脊色带+标题+作者 | max-width:520px 左边内容右边空白 | CSS 不生效时先用浏览器检查，可能是另一个组件在渲染 |
| 3 | ds-web-3 风格横向纯文字 | 风格照搬，缺乏独立设计判断 | DESIGN.md 是基准，不是模具——每个组件可以有自己的诠释 |
| 4 | 纸白卡片+shadow+暖橙hover线 | 标题和标签挤在左侧，右侧留白 | 排版问题用 space-between 解决，不要用 max-width |
| 5 | title+tag 同行 space-between | 右边空旷 | max-width:520px 还在——卡片被限宽，不是布局问题 |
| 6 | 去掉 max-width | 卡片填满父容器，tag 自然贴右 | 浏览器验证 > 代码审查——看代码永远看不出 CSS 问题 |

**核心教训:**

1. **CSS 问题用浏览器验证，不用代码审查** — 代码里看了 5 遍以为是布局问题，实际是 max-width 限宽
2. **每个组件独立设计，不照搬模板** — ds-web-3 是灵感源，不是模具
3. **排版就是组件** — 纸墨风不需要装饰性 UI 元素。标题、作者、标签的对齐方式本身就是设计
4. **HMR 不可靠** — 如果反复刷新效果不变，先确认渲染的是不是你以为的组件 (用 `browse js` 检查 className)

### Round 3: 页面级组件

**页脚:** 12 行 HTML + 130 行 SCSS → 1 行 `<p>` + 7 行 CSS 变量。纸墨风的页脚只需要一行字。不需要链接、社交图标、网格布局。极简不是"少放点东西"而是"每个存在的东西都必须有存在的理由"。

**导航栏:** 内联主题切换、用户头像、通知红点、移动端汉堡菜单——全部在一个组件内，不依赖外部 ThemeToggle。

## 关键设计决策

### 为什么不直接用 Element Plus CSS 变量映射？

60% 的工作量可以通过一次性覆盖 `--el-color-primary` 等变量完成。但这会产生"颜色对但气质不对"的页面——所有组件都是同一个模板。

毕设项目需要**每个页面的独立设计品质**，而非流水线产品。代价是更多的手工调整，收益是每个页面有自己的纸墨表情。

### 字体选择：为什么是 Inter？

用户从 ds-web-3.html 明确选择了 Inter——清晰、中英文混排兼容、字重覆盖正好 400-500。当 `/frontend-design` 技能建议避免 Inter 时，用户的 DESIGN.md 优先于技能的字体偏好。

## 文件结构

```
project/
├── DESIGN.md                    # 设计系统
├── frontend/src/
│   └── styles/
│       └── design-tokens.css    # CSS 变量实现
│   └── components/
│       └── common/
│           ├── NavigationBar.vue
│           └── ThemedPageContainer.vue
│       └── recommendation/
│           ├── RecommendationCard.vue
│           └── RecommendationCarousel.vue
│   └── views/
│       └── home/
│           ├── Home.vue
│           └── components/
│               ├── WelcomeSection.vue
│               └── RecommendationSection.vue
└── App.vue                      # Footer + shell
```

## 参考实现

所有变更在 `dev_010_重构前端页面：极简风`，仓库 `github.com/therain2020/AI_libary`:

```
98913ed4 refactor: consolidate shared components
8a71d936 refactor: minimal paper-ink footer + App shell
c1ec11af refactor: paper-ink recommendation section + carousel + card
f867ac0d refactor: book card v3 — horizontal text-only
76186d8e refactor: book card v4 — paper slip with warm accent edge
eee9548a fix: book card — title+tag on same row, space-between
```
