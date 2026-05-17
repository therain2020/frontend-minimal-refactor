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

## Round 4: 管理端页面

### 4.1 设计令牌迁移策略

**问题**: 旧 SCSS 变量系统与 DESIGN.md 令牌系统并存，旧系统是实际生效的。

**策略：保留旧变量名，值指向新令牌。** 不逐文件替换变量名，而是在 theme 文件中将旧变量重定向到新令牌。旧标记不改，自动获得新外观。零破坏性迁移。

**迁移顺序**:
1. 确认设计令牌文件已定义（亮色/暗色两套）
2. Theme 文件一次改写，旧变量值全部指向新令牌
3. Element Plus 变量映射到设计令牌
4. 全局 SCSS mixin 颜色值同步更新
5. 入口 HTML 修复字体加载 + 暗色模式初始化（需同时设置属性选择器和 class，部分 UI 库依赖 class）
6. 持久化 key 对齐（HTML 初始化脚本与 store 使用相同 key）

### 4.2 组件重构模式

**排版就是组件** — 纸墨风不需要装饰性 UI。标题/作者/标签的对齐本身就是设计。

**删除清单：**
- emoji（模板中的和 JS 中 hardcoded 的 emoji 映射表）
- 旧系统专有色渐变 → 全部替换为设计令牌强调色
- 玻璃态效果（backdrop-filter、旧系统半透明背景） → 设计令牌纸色容器
- SCSS mixin 引用 → 直接使用 CSS 设计令牌
- JS 中的硬编码颜色 → 移除或改用 CSS 变量引用

**列表呈现：表格优先于卡牌网格。** 管理端列表数据天然适合表格——更紧凑、更可比较、更可扫读。表头使用最小字号 + 增大字间距 + 最浅中性色。行 hover 次浅中性色。

**统一交互元素：**
- 操作按钮：固定尺寸 SVG 图标，默认中性灰，hover 后变色
- 状态标签：药丸形，根据状态着不同 tint 色（同一色相，低/中透明度区分 bg/border/text）
- 对话框：统一结构 `overlay > dialog > head(title + SVG close) + body + foot(cancel + confirm)`

### 4.3 角色隔离

不同角色不应看到其他角色的功能入口。导航栏根据用户角色切换内容。

- 角色判断优先于 URL 路径匹配（高权限用户始终看管理导航）
- 导航栏仅放高频入口，其余入口放入首页快速操作区
- 每个路由必须有入口点（导航栏或快速操作，不能遗漏任何功能）

### 4.4 仪表盘设计原则

- **KPI 展示**: 最大字号呈现数字，等宽数字字体特性，仅用强调色标记异常值
- **快速操作**: sticky 定位的侧边栏，纯文字列表，hover 时箭头滑入
- **活动记录**: 时间线视觉（左侧竖线引导 + 圆点节点），非简单列表
- **规则/配置管理**: 行内编辑 + CSS toggle switch 替代文字按钮

### 4.5 数据可视化设计

**图表是手段，数据洞察是目的。**

- KPI: 大数字直接呈现，不包裹卡片
- 趋势: 单线图优于多线图，减少视觉噪音，让趋势本身说话
- 分布: 水平条图优于饼图（标签可读、数值可比）
- 排行: 排版化有序列表优于柱状图（编号 + 名称 + 数值，前三高亮）
- 组织: 按业务模块分组而非按图表类型堆砌
- 暗色适配: Canvas 渲染的图表库不接受 CSS 变量字符串。在计算属性中根据主题状态返回实际 hex 色值

### 4.6 组件依赖简化

- 页面尽量自包含，减少子组件依赖（内联表单替代通用表单对话框组件）
- 图表子组件剥离外层卡片包装，由父页面统一提供卡片容器
- 共享组件保持极度简洁，不接受装饰性参数
- 功能合并后被替代的旧子组件可以删除

### 4.7 浮层交互组件

浮层小组件（聊天球、快捷对话框等）适用同样原则：
- 图标用 SVG 替代 emoji
- 配色继承设计令牌强调色
- 容器用纸色卡片替代玻璃态
- 消息气泡：己方用强调色，对方用中性纸色

## 常见陷阱

- **`<script setup>` 函数位置**: 必须在 setup 块内定义，模板和 style 之间不是合法 JS 位置
- **TransitionGroup + Fragment**: Vue 允许多根元素，但 Transition/TransitionGroup 要求子组件输出单一根节点。多根组件（如含 Teleport）需外层包裹
- **HTTP 响应解包**: 项目级拦截器可能已解包顶层，不要二次解包
- **Store 方法名不匹配**: 调用名与 store 导出名不一致时，结果是 undefined 被 try-catch 吞掉返回空数据，页面静默显示空状态。先 grep store 的 `return { ... }` 再写调用
- **ComputedRef 嵌套不解包**: computed ref 放在普通对象属性中，模板访问拿到的是 ComputedRef 对象而非 `.value`。`{ books: computedRef }` → 模板中需先 `.value` 取值再赋值
- **overflow:hidden 破坏弹出层**: 为圆角裁剪加的 `overflow: hidden` 会同时裁剪所有绝对定位子元素（预览条、下拉菜单）。去掉即可，padding 自然保持圆角内容不溢出
- **import 移除但模板仍引用**: 运行时报 ReferenceError 而非编译错误，容易被构建通过掩盖
- **编译器保留字**: `<script setup>` 顶层作用域中某些短变量名可能与 Vue 编译宏冲突

## 功能完整性检查清单

重构后逐项验证：
- [ ] 所有路由都有入口点（导航栏或首页快速操作）
- [ ] 所有 CRUD 操作可用（增删改查 + 导入导出）
- [ ] 所有 API 调用参数正确（分页参数名、响应解包层级）
- [ ] 主题切换无闪烁（初始化脚本与 store 行为一致）
- [ ] 暗色模式下所有背景色来自设计令牌，无硬编码白色
- [ ] 所有 emoji 已消除（模板内联 + JS 配置常量）
- [ ] 分页组件在各页面间视觉一致且居中
- [ ] hover 交互用 v-if 驱动而非 CSS opacity
- [ ] v-if 条件使用白名单（`=== 'OPEN'`）而非黑名单（`!== 'CLOSED'`）
- [ ] 构建通过 + 浏览器实测功能

## Round 5: 用户端页面 — 补充模式

以下模式在用户端深度重构中反复出现，是对 Round 4 管理端模式的补充。

### 列表卡片的 Accent Strip 模式

用户端列表（藏书、搜索结果、预约、收藏、相似推荐、活动记录）统一使用左侧 accent 竖线卡片。与 Round 4 的表格优先策略互补——用户端适合卡片，管理端适合表格。

结构: 垂直列表 → 每项 `flex row` → 左侧竖线 + 右侧内容区。竖线常态中性色，hover 时变为强调色并加宽、卡片微抬。

### Emoji 系统性消除

三种来源三种策略。不可交换使用——配置常量的文本缩写不适合模板内联，反之亦然。

| 来源 | 策略 | 示例 |
|------|------|------|
| JS 配置常量映射表 | 双字母文本缩写 | `{ LOGIN: 'L', AI_CHAT: 'AI' }` |
| 模板装饰性 emoji | 直接删除 | `🎯 标题` → `标题` |
| 模板功能性 emoji | 语义等价的文字 | `✏️ 编辑` → `编辑` |

底线: 不引入新 emoji 替换旧 emoji。替换物只能是文字或纯 CSS 形状。

### 隐藏 UI 的空间代价

`opacity: 0` 的元素不可见但仍占据完整布局空间——多个按钮叠加形成大段不明空白。用 `v-if` + JS 状态（`hoveredId`）+ `mouseenter/mouseleave` 从 DOM 中完全移除。仅当需要过渡动画时保留 CSS 方案。

### 用户反馈信号

特定措辞反复指向同类根因，值得编码为诊断捷径：

| 用户措辞 | 优先排查 |
|---------|---------|
| "不喜欢" / "不好看" | 布局模式选择（card / timeline / list），不是颜色或间距 |
| "彻底重构" | 从 DOM 结构到交互全量替换，不做局部修补 |
| "不搭" / "完全不搭" | 新老组件混用 或 全局样式污染 |
| "没数据" / "读不到" | Store 方法名正确性 → axios 拦截器解包层级 → ComputedRef 是否解包 |
| "没有区别" | 缺少状态差异化视觉线索（选中/未选中、已置顶/未置顶） |

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
