# Web4ParticleInteraction

[English](README-en.md)

一个高性能、零依赖的 **ASCII 流体交互背景库**。用户鼠标移动时注入流体，产生可视化的"墨水"流动效果，支持自定义形状、参数微调、内容避让，可轻松集成到任何网页。

## ✨ 特性

- **流体模拟**：基于 Jos Stam stable-fluids 算法，在字符网格分辨率上直接运行，保证形状清晰
- **交互光标**：鼠标跟随笔刷（默认猫头形状），移动时平流成拖尾，停止时自动归位并轻微浮动
- **内容避让**：自动检测页面文字、按钮、图片等元素，不在其上方渲染特效，保持可读性
- **响应式**：自动适配窗口缩放、滚动、字体加载，支持触摸设备
- **无障碍**：尊重 `prefers-reduced-motion` 系统设置，自动降级为静态纹理
- **零依赖**：纯 JavaScript + Canvas 2D，无任何外部库
- **高性能**：模拟 45Hz、渲染 60fps，实测 1280×820 分辨率 ~60FPS 稳定帧率
- **可定制**：换形状（编辑点阵）、改参数（调整流体手感、颜色、范围）、换颜色（CSS 变量）

## 🚀 快速开始

### 1. 复制脚本

将 `js/background.js` 拷贝到你的项目中。

### 2. 修改 HTML

在 `<body>` 标签最开头加两个容器：

```html
<body>
  <div id="bg-gradient" aria-hidden="true"></div>
  <canvas id="bg-canvas" aria-hidden="true"></canvas>
  <!-- 你的页面内容... -->
</body>
```

### 3. 添加 CSS

在你的样式文件（或 `<head>` 的 `<style>` 块）中添加：

```css
:root {
  /* 渐变光晕颜色 */
  --bg-grad-1: #c7d0ff;
  --bg-grad-2: #d9ccf2;
  --bg-grad-3: #cfe0f5;
  /* 背景底色 */
  --bg-grad-base: #e3e7ea;
  /* 字符颜色（RGB 三元组） */
  --bg-glyph: 70, 84, 150;
  /* 可选：等宽字体 */
  --font-mono: 'Courier New', monospace;
}

#bg-gradient {
  position: fixed;
  inset: 0;
  z-index: -2;
  pointer-events: none;
  background:
    radial-gradient(60% 60% at 78% 18%, var(--bg-grad-1) 0%, transparent 62%),
    radial-gradient(55% 55% at 18% 30%, var(--bg-grad-2) 0%, transparent 60%),
    radial-gradient(70% 70% at 50% 92%, var(--bg-grad-3) 0%, transparent 64%),
    var(--bg-grad-base);
  opacity: 0.55;
}

#bg-canvas {
  position: fixed;
  inset: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  pointer-events: none;
}
```

**重要**：两个背景层必须使用 `position: fixed`、负 `z-index`、`pointer-events: none`，确保总在内容之下且不阻止用户交互。

### 4. 引入脚本

在 `</body>` 前加：

```html
<script src="path/to/js/background.js"></script>
```

完成！刷新页面，移动鼠标就能看到 ASCII 流体特效。

## ⚙️ 参数调整

编辑 `js/background.js` 顶部的常量来微调效果。

### 外观

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `CELL_PX` | `8` | 字符网格大小（像素）；越小越细密 |
| `RAMP` | `[' ','·','_','-','>','○']` | 亮度阶梯：从暗到亮对应的字符 |
| `CONTRAST` | `1.9` | 密度→亮度对比度；越大越明显 |
| `GAMMA` | `0.6` | 亮度曲线；越小暗部越容易显现 |
| `THRESHOLD` | `0.12` | 低于此值的格子不画（保持留白） |

### 透明度

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `ALPHA_MAX` | `0.45` | 整体不透明度上限；越小越淡 |
| `ALPHA_BASE` | `0.05` | 最暗字符的透明度下限 |
| `ALPHA_GAIN` | `0.40` | 靠近墨水时变亮的幅度 |

### 光标笔刷形状

编辑 `BRUSH_SHAPE` 数组（`#` 表示有墨，空格表示空）即可换成任意形状：

```javascript
const BRUSH_SHAPE = [
  '##   ##',     // 两个尖耳朵
  '### ###',
  '#######',
  '# ### #',     // 眼睛
  ' ##### ',     // 嘴
];
// 或试试爱心：
// const BRUSH_SHAPE = [
//   ' ##### ',
//   '#######',
//   '#######',
//   ' ##### ',
//   '  ###  ',
// ];
```

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `BRUSH_SHAPE` | 猫头点阵 | 光标笔刷的形状 |
| `BRUSH_SCALE` | `2` | 每个点阵像素占几个字符格；越大形状越大 |
| `STAMP_VALUE` | `0.9` | 笔刷的密度强度 |

### 流体手感

这些参数控制墨水的流动、消散、浮动：

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `flow` | `0.7` | 平流强度：单位速度下墨水移动的格数；越大越容易散开 |
| `iter` | `4` | 压力求解迭代次数；越大 swirl 越平滑但越耗性能 |
| `densDecay` | `0.85` | 拖尾消散速度；越小拖尾越短 |
| `velDecay` | `0.55` | 速度阻尼；越小停得越快，同时决定 `idleAmp` 的稳态幅度（≈`idleAmp/(1-velDecay)`）；调大会让浮动越来越夸张甚至发散 |
| `force` | `0.6` | 鼠标速度→注入速度的缩放；越大流动越剧烈 |
| `velStop` | `0` | 速度低于此值直接归零；保持 `0` 让 `velDecay` 自然阻尼，避免和 `idleAmp` 冲突 |
| `idleAmp` | `0.07` | 静止时持续注入的微弱"浮动力"；让猫猫不会完全冻死，轻微呼吸感 |
| `idleFreq` | `0.0015` | 浮动力的摆动速度；越大浮动越快越明显 |

### 环境层

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `AMBIENT_AMP` | `0.08` | 静止时的微弱底纹强度（使用多频率 sine 噪声）；设 `0` 则完全空白 |

### 避让内容 / 范围控制

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `SAFE_SELECTOR` | 多个选择器 | 这些元素（标题、段落、按钮、图片…）上方不画字符 |
| `SAFE_PAD` | `6` | 内容框的硬外扩边距（px） |
| `SAFE_FEATHER` | `16` | 遮罩边缘的柔和羽化带宽度（px） |
| `SAFE_REBUILD_MS` | `140` | 遮罩重算频率（毫秒） |
| `CUTOFF_SELECTOR` | `'#about'` | 特效只显示到这个元素底部；改成不存在的选择器可铺满整页 |
| `CUTOFF_FEATHER` | `70` | 截止线上方的淡出带宽度（px） |

**示例**：不让特效显示在某些元素上，改 `SAFE_SELECTOR`：

```javascript
const SAFE_SELECTOR = 'h1, h2, .logo, .nav-bar, .floating-widget';
```

**示例**：让特效铺满整个视口：

```javascript
const CUTOFF_SELECTOR = '#__nonexistent__';
```

## 🎨 颜色主题

所有颜色都由 CSS 变量驱动，支持亮/暗主题自动切换：

```css
/* 浅色主题 */
:root {
  --bg-grad-1: #c7d0ff;     /* 紫蓝 */
  --bg-grad-2: #d9ccf2;     /* 淡紫 */
  --bg-grad-3: #cfe0f5;     /* 蓝白 */
  --bg-grad-base: #e3e7ea;  /* 底色 */
  --bg-glyph: 70, 84, 150;  /* 字符颜色 RGB */
}

/* 暗色主题 */
@media (prefers-color-scheme: dark) {
  :root {
    --bg-grad-1: #2b343b;
    --bg-grad-2: #313442;
    --bg-grad-3: #283640;
    --bg-grad-base: #1a1f24;
    --bg-glyph: 141, 163, 153;
  }
}
```

## 📐 技术细节

### 算法

基于 Jos Stam 的经典稳定流体求解器（Stable Fluids），采用半拉格朗日平流 + 高斯-赛德尔压力投影：

1. **速度注入**：鼠标移动时，在笔刷范围内按 `force` 缩放注入速度
2. **压力投影**：确保速度场无散度（保证流体不膨胀）
3. **平流**：沿速度场反向追踪，计算新密度（墨水）位置
4. **衰减**：每步对密度和速度应用 `densDecay` / `velDecay` 指数衰减
5. **渲染**：将密度映射到 RAMP 字符，并计算对应的透明度

### 性能优化

- 直接在字符网格分辨率运行（1280×820@8px ≈ 160×103 格），避免上采样模糊
- 模拟固定 45Hz，渲染 60fps（解耦 sim 和 render）
- 遮罩检测节流 140ms，只在必要时（scroll/resize/load/font-ready）重算
- 单 pass 渲染，无多重缓冲

### 浏览器兼容性

需要支持：
- Canvas 2D Context
- `requestAnimationFrame`
- CSS custom properties（CSS variables）
- `matchMedia` 用于响应式检测

**最低要求**：现代浏览器（Chrome 49+, Firefox 31+, Safari 9.1+, Edge 15+）

## 📝 集成建议

### 1. 调整安全区

根据你页面的结构调整 `SAFE_SELECTOR`：

```javascript
const SAFE_SELECTOR = 'h1,h2,h3,h4,h5,h6,p,li,a,button,img,.header,.nav';
```

### 2. 调整特效范围

改 `CUTOFF_SELECTOR` 决定特效在哪里停止：

```javascript
const CUTOFF_SELECTOR = '.footer'; // 只显示到 footer 前
// 或
const CUTOFF_SELECTOR = '#__none__'; // 铺满整页
```

### 3. 颜色匹配

把 CSS 变量改成你网站的配色，或从现有 CSS 变量继承：

```css
:root {
  --bg-grad-base: var(--primary-bg);
  --bg-glyph: var(--primary-text-rgb);
}
```

### 4. 禁用/启用

如果特定页面不需要特效，可以在 `<head>` 检查条件：

```html
<script>
  if (location.pathname === '/admin') {
    // Don't load the background effect
    document.addEventListener('DOMContentLoaded', () => {
      const canvas = document.getElementById('bg-canvas');
      if (canvas) canvas.parentNode.removeChild(canvas);
    });
  }
</script>
<script src="js/background.js"></script>
```

## 📦 文件结构

```
Web4ParticleInteraction/
├── README.md                   # 这份文件
├── background-effect.md        # 详细参数文档
└── js/
    └── background.js           # 核心库（495 行，纯 JS）
```

## 🔧 常见问题

**Q：特效遮挡了我的文字**  
A：调整 `SAFE_SELECTOR` 或提高 `SAFE_PAD` / `SAFE_FEATHER` 值。

**Q：想换一个不同的形状**  
A：编辑 `BRUSH_SHAPE` 数组。每行是一个字符串，`#` 表示墨水，空格表示空。

**Q：流动太慢/太快**  
A：调大/调小 `velDecay`（越小越快停）、`force`（越大越敏感）。

**Q：浮动太明显，让猫猫看起来抖**  
A：减小 `idleAmp` 或调整 `velDecay`。

**Q：想让特效更淡**  
A：减小 `ALPHA_MAX` / `ALPHA_GAIN`，或增加 `THRESHOLD`。

**Q：特效在弱性能设备上卡顿**  
A：减小 `CELL_PX`（网格变粗，但性能更好）或降低 `iter`（减少 swirl 平滑度）。

## 📄 许可证

MIT

## 👤 作者

原始设计思路来自 OpenAI Codex 官方演示页面；本库从头实现，额外添加了流体浮动、内容避让、参数微调等扩展。
