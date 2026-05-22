# 红泥数智宣传片 · RedMud Intro Video

基于 HyperFrames 的自动化视频渲染项目，输出红泥数智官方宣传片（5m 47s，347s）。

## 环境要求

- **Node.js** ≥ 18
- **HyperFrames CLI**: `npx hyperframes@0.6.4`
- **macOS**（渲染使用系统浏览器）

## 项目结构

```
redmud-intro-v2/
├── index.html                  # 入口文件，定义场景时长和 root timeline
├── compositions/
│   └── scenes.html             # 所有场景（CSS + GSAP 动画）
├── assets/
│   ├── audio/
│   │   └── new-radio.mp3       # 音频（347.3s）
│   └── images/
│       ├── logo.png             # Logo
│       ├── ewm.jpg              # 公众号二维码
│       ├── my body.png          # 人物全身照（用于 S1/S10 右侧）
│       └── my-id.jpg             # 头像（预留）
├── package.json
└── README.md
```

## 渲染流程

```bash
# 安装依赖
npm install

# 语法检查（推荐每次渲染前运行）
npm run check

# 渲染视频
npx hyperframes@0.6.4 render . renders/redmud-intro-v{N}.mp4 --fps 30

# 上传到服务器
scp renders/redmud-intro-v{N}.mp4 ubuntu@43.153.213.134:/tmp/
ssh ubuntu@43.153.213.134 "sudo mv /tmp/redmud-intro-v{N}.mp4 /var/www/wisdomdance/"
```

## 场景时间轴（v30）

| 场景 | 时间范围 | 时长 | 说明 |
|------|---------|------|------|
| S1   | 0–17s   | 17s  | 开场，左文字 + 右人物照 |
| S2   | 17–45s  | 28s  | 公司介绍 |
| S3   | 45–76s  | 31s  | 数据资产入表 |
| S4   | 76–102s | 26s  | 7大Agent（逐个打光 79.5s 起）|
| S5   | 102–115s| 13s  | 核心能力 |
| S6   | 115–159s| 44s  | 服务体系 |
| S7   | 159–190s| 31s  | 行业覆盖 |
| S8   | 190–230s| 40s  | 服务3+4 详解 |
| S9   | 230–295s| 65s  | 7个Agent 详解（逐个打光 233.5s 起）|
| S10  | 295–347s| 52s  | 核心服务 + 联系方式 |
| S11  | 347s+   | 定格 | Logo + 二维码并排 |

**总时长**: 347s（5m 47s），音频 `new-radio.mp3` 347.3s。

## 三区域布局模板

所有场景共享三区域布局：

```
┌──────────────────────────────────────┬───────────────┐
│                                      │               │
│           上部内容区（980px）          │   右侧 50%    │
│                                      │               │
├──────────────────────────────────────┴───────────────┤
│              底部金句区（100px）                      │
└─────────────────────────────────────────────────────┘
```

- **S1 / S10**：左半区文字（居中）+ 右半区照片（`object-fit: contain`）
- **S2–S9**：左侧内容 50% + 右侧内容 50%
- **底部金句**：金色 `#E8C97A`，无头像，GSAP fade-in

## CSS / GSAP 规范

### ID 属性规范（强制）

所有需要 GSAP 动画的元素**必须使用显式 `id` 属性**，禁止依赖 class 精细化控制。

```html
<!-- ✓ 正确 -->
<div id="s3-headline">数据资产入表</div>
<span id="s3-quote-txt">赋能企业数据资本化</span>

<!-- ✗ 错误：class 无法作为 GSAP 目标 -->
<div class="headline">数据资产入表</div>
```

```javascript
// ✓ 正确：直接用 id 选择器
gsap.fromTo('#s3-headline', { opacity: 0, y: 20 }, { opacity: 1, y: 0, duration: 1 });

// ✗ 错误：class 选择器
gsap.fromTo('.headline', { opacity: 0 }, { opacity: 1, duration: 1 });
```

### CSS 初始状态

所有 GSAP 动画目标必须在 CSS 中设置 `opacity: 0`：

```css
#s3-headline { opacity: 0; }
#s3-quote-txt { opacity: 0; }
```

### GSAP 动画模式

**GSAP `tl.set` + `tl.to`**（推荐，headless seek 兼容）：

```javascript
const tl = gsap.timeline({ scrollTrigger: { trigger: '#s3', start: 'top top' } });
tl.set('#s3-headline', { opacity: 0, y: 20 })
  .set('#s3-quote-txt', { opacity: 0 })
  .to('#s3-headline', { opacity: 1, y: 0, duration: 1 }, 0)
  .to('#s3-quote-txt', { opacity: 1, duration: 1 }, 0.3);
```

**不要使用 `tl.from({opacity:0})`**，在 headless seek 模式下不工作。

### Agent 打光动画

S4 / S9 场景中，7个 Agent 逐个高亮：

```javascript
// 在 s4-spotlight label 处执行（79.5s / 233.5s）
for (let i = 1; i <= 7; i++) {
  tl.to(`#s4-agent-${i}`, { className: 'agent agent-highlight', duration: 0.1 }, `spotlight+=${(i-1)*1.6}`)
    .to(`#s4-agent-${i}`, { className: 'agent', duration: 0.1 }, `spotlight+=${(i-1)*1.6+1.4}`);
}
```

```css
.agent-highlight {
  border-color: #a78bfa;
  box-shadow: 0 0 18px rgba(167, 139, 250, 0.4);
}
```

## 场景时长调整

修改 `index.html` 中的 `data-duration` 和 `rootTl` 总时长，以及 `scenes.html` 中对应场景的 CSS `@keyframes` 参数。

```bash
# 示例：S8 停留时间 +10s，S9 -10s
# 1. 修改 scenes.html CSS: animation-duration
# 2. 修改 scenes.html GSAP: 所有时间 +10s / -10s
# 3. 修改 index.html: rootTl duration
```

## 环境变量 / 凭证

| 内容 | 值 |
|------|-----|
| 服务器 IP | 43.153.213.134 |
| 服务器路径 | `/var/www/wisdomdance/` |
| 在线地址 | `https://wisdomdance.cn/redmud-intro-v{N}.mp4` |

## 版本记录

| 版本 | 日期 | 主要改动 |
|------|------|---------|
| v30 | 2026-05-22 | 所有 GSAP 目标改用显式 id 选择器 |
| v29 | 2026-05-22 | S8/S9 时长调整（+10s/-10s）|
| v28 | 2026-05-22 | S11 结尾定格画面（logo+二维码并排）|
| v27 | 2026-05-22 | 音频完整播放（321s → 347s）|
| v26 | 2026-05-22 | Agent 逐个打光动画 |
| v25 | 2026-05-22 | 放大 S4/S9 agent 图标，移除 avatar |
| v24 | 2026-05-22 | `object-fit: contain` 保持照片比例 |
| v22 | 2026-05-22 | 三区域布局，左右 50%/50% |
| v20 | 2026-05-22 | 完整重写，347s 总时长 |
| v19 | 2026-05-22 | 修复 `tl.from({opacity:0})` 黑屏问题 |

## 常见问题

**Q: 渲染出现黑屏？**
A: 检查是否所有 GSAP 动画目标都有 `opacity: 0` CSS 初始值，且 GSAP 使用 `tl.set` + `tl.to` 而非 `tl.from`。

**Q: S10 照片拉伸变形？**
A: 确认 CSS 使用 `object-fit: contain`，不要用 `cover` 或 `fill`。

**Q: 某个场景内容不显示？**
A: 检查该场景所有元素的 `id` 是否与 GSAP 动画中的选择器完全匹配，包括大小写。
