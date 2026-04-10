# 健康风险画像评估系统

> 你的身体，正在悄悄告诉你一些事。

一个开源的健康风险画像评估系统，基于 15 个健康行为维度进行科学化分析和匹配。帮助用户了解自己的健康风险类型，获得个性化的改善建议。

## 在线体验

👉 [点击开始评估](https://jonk-wu.github.io/HBTI/)

## 主要特性

- 🧠 **25 种标准画像** — 全面覆盖健康风险类型（+ 2 种特殊隐藏/兜底画像）
- 📊 **15 个评估维度** — 饮食、运动、睡眠、心理、风险五大健康模型
- 🎯 **曼哈顿距离匹配** — 基于 15 维向量空间的科学化匹配算法
- 🍺 **隐藏彩蛋机制** — 吸烟门控触发"老烟枪"隐藏画像
- 📱 **移动端优先** — 响应式设计，完美适配各种屏幕
- 🔧 **高度可定制** — 数据与代码完全分离（JSON 驱动），无需改代码即可创建你自己的评估系统

## 项目结构

```
├── data/                    # 评估系统数据（修改这里来定制）
│   ├── questions.json       # 30+2道问卷题目和选项
│   ├── dimensions.json      # 15个维度定义和三级标准
│   ├── types.json           # 25个标准画像和2个特殊画像
│   └── config.json          # 评分参数、显示文案和门控配置
├── src/                     # 源代码
│   ├── engine.js            # 核心评分算法（纯函数，支持矩阵运算）
│   ├── quiz.js              # 答题流程的状态管理
│   ├── result.js            # 结果页面渲染和分享生成
│   ├── chart.js             # 雷达图绘制（Canvas API）
│   ├── utils.js             # 工具函数库
│   ├── main.js              # 应用入口和事件绑定
│   └── style.css            # 样式（使用CSS变量支持主题定制）
├── docs/
│   └── analysis.md          # 完整的数据分析报告（维度分析、画像聚类等）
└── index.html               # 主页面

## 快速开始

```bash
# 克隆项目
git clone https://github.com/jonk-wu/HBTI.git
cd SBTI

# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build
```

## 定制你自己的评估系统

所有评估内容都存储在 `data/` 目录中 JSON 文件里，修改这些文件即可定制，**无需改动任何代码**。

### 修改题目

编辑 `data/questions.json`，每道题的结构示例：

```json
{
  "id": "q1",
  "dim": "D1",
  "text": "您平时有什么饮食习惯？",
  "options": [
    { "label": "基本不吃甜食", "value": 1 },
    { "label": "偶尔吃甜食", "value": 2 },
    { "label": "每天都要吃甜食或喝奶茶", "value": 3 }
  ]
}
```

- `dim` 指定该题属于哪个维度（D1-D3, E1-E3, S1-S3, M1-M3, R1-R3）
- `value` 分值：1 = 低风险（健康），2 = 中风险，3 = 高风险
- **重要**：每个维度需要恰好 2 道题
- 可选：某些题目需要反向计分，将 value 顺序改为 3,2,1

### 添加新的健康风险画像

编辑 `data/types.json` 的 `standard` 数组，添加新画像：

```json
{
  "code": "PROFILE",
  "pattern": "HHH-HHL-MHH-HHL-MMH",
  "cn": "健身房幽灵",
  "intro": "一句话简介描述这个画像。",
  "risks": [
    "高风险特征 1",
    "高风险特征 2",
    "高风险特征 3"
  ],
  "suggestions": [
    "改善建议 1",
    "改善建议 2",
    "改善建议 3"
  ],
  "desc": "详细的健康建议文本（建议 200-400 字）..."
}
```

**关键说明**：
- `pattern` 是 15 个字母的 L/M/H 组合（按维度顺序：D1-D3, E1-E3, S1-S3, M1-M3, R1-R3），用 `-` 分隔每个模型
- 设计新画像时检查与现有 25 个画像的曼哈顿距离，**最小距离≥3** 为佳
- 详见 [docs/analysis.md](./docs/analysis.md) 中的"6. 定制指南"章节

### 调整评分参数

编辑 `data/config.json` 中的 `scoring` 字段：

```json
{
  "scoring": {
    "levelThresholds": {
      "L": [2, 3],  // 原始分2-3映射为低风险
      "M": [4, 4],  // 原始分4映射为中风险
      "H": [5, 6]   // 原始分5-6映射为高风险
    },
    "levelNumeric": {"L": 1, "M": 2, "H": 3},
    "maxDistance": 30,           // 理论最大曼哈顿距离（15维×2）
    "fallbackThreshold": 60      // 兜底触发阈值（相似度<60%）
  }
}
```

参数说明：
- `levelThresholds`：控制原始分多少分落入L/M/H三级
- `fallbackThreshold`：用户向量与所有标准画像匹配度都低于此阈值时，触发"均衡健康人"兜底

### 自定义吸烟门控

编辑 `data/config.json` 的 `smokeGate` 字段来配置隐藏画像触发条件：

```json
{
  "smokeGate": {
    "questionId": "smoke_gate_q1",
    "triggerValue": 3,        // 选择"吸烟"时回答的value
    "smokerTriggerValue": 2   // smoke_gate_q2选择"每天一包以上"时的value
  }
}
```

### 修改显示文案

编辑 `data/config.json` 的 `display` 字段：

```json
{
  "display": {
    "title": "健康风险画像评估",
    "subtitle": "你的身体，正在悄悄告诉你一些事。",
    "author": "健康科普页面配套测试",
    "funNote": "本测试仅供健康教育参考，不构成医疗建议。"
  }
}
```

### 修改样式主题

编辑 `src/style.css` 顶部的 CSS 变量来快速改变配色：

```css
:root {
  --bg: #f0f4f1;              /* 背景色 */
  --accent: #4c6752;          /* 强调色 */
  --text-primary: #1a1a1a;    /* 主文字 */
  --text-secondary: #666;     /* 辅助文字 */
  --border: #e0e0e0;          /* 边框色 */
  /* ... 更多变量 */
}
```

## 核心算法

### 评分流程

1. **维度求和**：用户答完 30 题后，每个维度将 2 题分值相加（原始分 2-6）
2. **等级映射**：原始分转化为三级风险等级
   - L（低）：2-3 分
   - M（中）：4 分  
   - H（高）：5-6 分
3. **向量化**：L=1, M=2, H=3，生成**15维数值向量**
4. **距离计算**：计算用户向量与 25 个标准画像的**曼哈顿距离**
5. **多条件排序**：
   - 优先级1：距离最小 (distance ASC)
   - 优先级2：精准命中维度最多 (exact_count DESC)
   - 优先级3：相似度最高 (similarity DESC)
6. **特殊规则覆盖**：
   - 如果吸烟门控触发 → 返回 **SMOKER（老烟枪）**
   - 如果最佳相似度 < 60% → 返回 **BALANCED（均衡健康人）** 兜底
   - 否则返回排序第一的标准画像

### 关键指标

- **相似度公式**：`similarity = max(0, round((1 - distance/30) * 100))`
- **理论距离范围**：0 ~ 30（15维 × 每维最大差2）
- **实际观察值**：最小距离 1（JOKER vs WRECK），最大距离 24（GRIND vs WRECK）

### 详细技术文档

详见 [docs/analysis.md](./docs/analysis.md)，包含：
- 完整的数据结构说明
- 25 个标准画像的详细分析
- 画像空间的聚类研究
- 维度分布和区分力统计
- 完整的定制指南

## 部署

### GitHub Pages（推荐）

Fork 本项目后，在 Settings → Pages → Source 选择 GitHub Actions 即可自动部署。（base 路径已在 `vite.config.js` 配置）

### Vercel / Netlify

直接连接 GitHub 仓库，Vite 项目自动识别，零配置部署。

### 本地测试

```bash
npm run dev
# 访问 http://localhost:5173
```

### 构建生产版本

```bash
npm run build
# 输出到 dist/，可部署到任何静态服务器
```

## 技术栈

- [Vite](https://vitejs.dev/) — 现代前端构建工具
- 原生 JavaScript（ES Module）— 零依赖
- Canvas 2D API — 雷达图数据可视化
- CSS Custom Properties — 主题系统

## 文件大小

- 打包后总体积 < 100KB（包括所有数据）
- 非常适合快速加载和移动端体验

## License

[MIT](LICENSE)
"# HBTI" 
