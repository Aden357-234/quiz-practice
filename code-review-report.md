# 代码审查报告

**审查时间**：2026-05-24
**审查范围**：`index.html` 全部代码（HTML + CSS + JS，约 1856 行）
**问题统计**：严重 0 个 | 高危 4 个 | 中等 7 个 | 低危 5 个 | 建议 6 个

---

## 🔴 严重问题 (Critical)

无。

---

## 🟠 高危问题 (High)

### 问题 1：考试倒计时依赖 setInterval，切后台导致计时不准

- **位置**：`startTimer()` 函数
- **描述**：考试模式使用 `setInterval(() => { state.timeLeft-- }, 1000)` 倒计时。浏览器在页面不可见时会大幅降低 `setInterval` 的触发频率，导致实际 15 分钟的考试可能被延长，削弱考试限时的意义。
- **修复**：✅ 已修复 — 改用 `Date.now()` 绝对时间戳 + 250ms 轮询

### 问题 2：全局数据未做空值检查

- **位置**：`startQuiz()`、`startLearn()`、`startWrongPractice()`
- **描述**：直接访问 `window.QUESTIONS_1_3` / `window.QUESTIONS_4_6`，无防御性检查。JS 文件加载失败时页面白屏。
- **修复**：✅ 已修复 — 新增 `getQuestionsByGrade()` 函数，加载失败弹 toast 提示

### 问题 3：showToast 快速连续调用定时器冲突

- **位置**：`showToast()` 函数
- **描述**：2 秒内再次调用时旧定时器未清除，可能把新 toast 提前隐藏。
- **修复**：✅ 已修复 — 全局 `toastTimer`，新 toast 清除旧定时器

### 问题 4：escapeHtml 频繁创建临时 DOM 元素

- **位置**：`escapeHtml()` 函数
- **描述**：每次调用创建 `div` 元素，学习模式渲染 150 题时创建约 900 个临时 DOM。
- **修复**：✅ 已修复 — 改用纯字符串替换，避免 DOM 创建

---

## 🟡 中等问题 (Medium)

### 问题 1：selectByType 函数复杂度较高

- **描述**：约 40 行，多轮条件分支和 while 循环补足，可读性差。极端情况下 while 循环可能不终止。
- **建议**：抽取"按权重分配整数"为独立函数，添加守卫 `if (count > allQuestions.length) count = allQuestions.length`

### 问题 2：renderQuestion 函数过于庞大

- **描述**：约 120 行，同时负责渲染标签、文本、选项、反馈区、确认按钮、导航按钮、跳转网格、进度条和题号。
- **建议**：拆分为 `renderQuestionBody(q)`、`renderOptions(q)`、`updateNavButtons(q)`

### 问题 3：CSS 存在冗余样式

- **描述**：`.question-card` 和 `.card` 样式重复；`.options-list` 中 `clear: both` 在 flexbox 中无效；`.quiz-nav .btn` 重复 `min-width`
- **建议**：抽取共用样式，移除无效属性

### 问题 4：jumpToQuestion 在即时模式下丢弃用户作答

- **描述**：跳转时 `state.answers = {}` 清空所有选中状态，但 `state.instantDone` 保留，导致已答对题目无法交互。
- **建议**：不清空 answers，改为在 renderQuestion 中根据 instantDone 恢复完整反馈状态

### 问题 5：模态弹窗缺少焦点管理

- **描述**：弹窗打开后焦点仍在底层元素，Tab 键可在弹窗背后导航，Escape 关闭后焦点未回归。
- **建议**：添加 `role="dialog"`、`aria-modal="true"`，管理焦点

### 问题 6：HTML 注释页面编号不一致

- **描述**：注释标记 `Page 4: Learn`，但 `showPage` 数组顺序为 `[home, quiz, summary, learn]`
- **建议**：统一编号

### 问题 7：学习模式过滤后编号混乱

- **描述**：过滤"多选题"时显示 `#95` 这样的全局编号，用户困惑
- **建议**：过滤模式使用 `i + 1`，全部模式使用全局编号

---

## 🟢 低危问题 (Low)

1. 部分 CSS 使用内联 `style` 属性，应抽取为 CSS 类
2. `learnQuestions` 和 `learnFilter` 是全局变量，应移入 `state` 对象
3. 键盘快捷键 1-4 可能误触，未处理小键盘 Numpad 键
4. 响应式断点仅 480px，缺少 481-768px 平板适配
5. `updateStartBtn` 函数名与实际行为不符

---

## 🔵 优化建议 (Suggestion)

1. 外部题库 JS 使用 `defer` 加载
2. emoji 图标添加 `aria-hidden="true"` 和 `aria-label`
3. 使用 CSS 变量管理主题色（`#667eea` / `#764ba2` 硬编码 15+ 处）
4. SVG 圆周长提取为常量
5. `scrollTo` 行为统一（`instant` vs `smooth`）
6. 错题数据添加版本号或时间戳

---

## ✅ 亮点

- XSS 防护到位（`textContent` + `escapeHtml`）
- localStorage 操作全量 try-catch 保护
- `selectByType` 按题型比例抽题设计合理
- 状态管理集中在单一 `state` 对象
- UI 细节周到：毛玻璃弹窗、错题自动跳转、考试最后 60 秒红色警示、题目卡片固定 min-height 防抖动
- 键盘导航支持：← → 翻题、1-4 选选项、Escape 关弹窗
