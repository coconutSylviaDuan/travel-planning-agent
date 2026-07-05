# Trip Map Builder — 本地项目使用说明

> 基于 [hiyeshu/trip-map-builder](https://github.com/hiyeshu/trip-map-builder) (MIT License)

## 项目是什么

一套面向 AI Agent 的**旅行行程规划工作流技能**，将零散的出行信息（截图、心愿单、文字描述）转化为一个可部署的**交互式行程地图页面**。

核心理念：**行程是出发前的参考坐标，不是必须逐小时执行的脚本。**

## 三阶段流水线

```
Phase 1: 规划行程          Phase 2: 调研             Phase 3: 生成地图
─────────────           ─────────────           ─────────────
提取硬约束               大众点评 → 口味/排队        复制模板
按区域分组               小红书   → 氛围/体验        填入数据
主动删高风险点           两段式：粗筛→精读           生成单文件 HTML
填充餐饮                 压缩为决策有用的一句话       可选：部署到 Vercel
```

## 项目结构

```
trip-map-builder/
├── CLAUDE.md                    # 项目地图 — 目录职责说明
├── SKILL.md                     # 技能入口 — 触发条件 + 三阶段流程定义
├── README.md                    # 项目说明
├── demo/
│   └── index.html               # ← 演示页面（上海4日游）
├── assets/
│   └── template.html            # 可复用 HTML 地图模板（Apple 设计系统）
└── references/
    ├── CLAUDE.md                # references 局部地图
    ├── trip-planning.md         # 行程规划方法论（选点原则、选餐原则、四拍交互）
    ├── dianping-research.md     # 大众点评调研工作流（OpenCLI adapter）
    └── xhs-research.md          # 小红书调研工作流（OpenCLI + Chrome CDP）
```

## 共享记忆系统

路径：`~/.trip-map-builder/MEMORY.md`（已初始化）

存储内容：
- 旅行者默认偏好（出发城市、节奏、餐饮偏好、预算习惯、支付/导航偏好）
- 历史行程索引
- 可复用的偏好规则
- 未解决事项

不存储：原始截图、证件信息、订单号、完整聊天记录

## 如何使用

### 方式一：作为 AI Agent 技能使用

将项目目录放入 AI 工具的 skills 目录，Agent 读取 `SKILL.md` 后自动按三阶段流程执行：

1. 告诉 Agent 你的出行信息（日期、航班、酒店、想去的地方）
2. Agent 提取约束、规划行程、调研餐厅
3. Agent 基于模板生成 `index.html`
4. Agent 更新共享记忆

触发词：`做个行程` / `行程规划` / `行程地图` / `plan my trip`

### 方式二：手动使用模板

1. 复制 `assets/template.html` → 你的新文件
2. 修改 `HOTEL` 对象（酒店名称和坐标）
3. 修改 `DAYS` 数组（每天的行程数据）
4. 修改 `overviewContent()` 函数（总览页内容）
5. 直接在浏览器打开即可

### 地点数据结构

```javascript
{
  name: '地点名称',        // 必填
  lat: 31.2304,           // 必填：纬度
  lng: 121.4737,          // 必填：经度
  type: 'food',           // 必填：food | spot | drink | hotel | transport
  time: '18:30',          // 必填：时间，'—' 表示备选
  desc: '简短描述',        // 必填
  budget: '¥200/人',      // 可选：预算
  detail: '详细信息',      // 可选：\n 换行
  pay: {                  // 可选：支付方式
    cash: 1,              //   1=支持  0.5=可能支持  省略=不显示
    card: 1,
    alipay: 1,
    wechat: 1
  },
  xhs: 'https://...',     // 可选：小红书链接
  xhsKeyword: '搜索词',    // 可选：小红书搜索词
  dianping: 'https://...', // 可选：大众点评链接
  dianpingKeyword: '搜索词',// 可选：大众点评搜索词
  reserve: 'https://...',  // 可选：预约链接
  gmap: '搜索查询词'       // 可选：Google Maps 搜索词
}
```

## 依赖工具

| 工具 | 用途 | 是否必须 | 安装 |
|------|------|----------|------|
| 浏览器 | 查看地图页面 | ✅ 必须 | 已有 |
| Leaflet.js | 地图渲染 | ✅ 必须 | CDN 自动加载 |
| OpenCLI | 大众点评 + 小红书调研 | ❌ 可选 | `npm install -g @jackwener/opencli` |
| Chrome | 远程调试（小红书调研） | ❌ 可选 | 已有 |
| gh CLI | GitHub 部署 | ❌ 可选 | 按需安装 |

> 不安装 OpenCLI 也能使用：手动调研后填入模板数据即可生成地图页面。

## 演示页面

`demo/index.html` 是一个完整的上海4日游演示，包含：
- 4天行程 × 18个地点
- 总览页（行程概览 + 支付提醒 + 天气提醒）
- 按天切换的交互式地图
- 时间轴卡片（可折叠详情）
- 导航三选一（Apple Maps / Google Maps / 高德地图）
- 大众点评 / 小红书 App 深链
- 支付方式标签
- 地理定位

直接用浏览器打开 `demo/index.html` 即可预览。

## 自定义设计系统

默认使用 Apple 设计系统。修改 `template.html` 中 `:root` 的 CSS 变量即可切换风格：

```css
:root {
  --accent: #0071e3;        /* 主题色 */
  --bg-primary: #ffffff;     /* 主背景 */
  --bg-secondary: #f5f5f7;   /* 次背景 */
  --text-primary: #1d1d1f;   /* 主文字色 */
  /* ... 更多变量 */
}
```

可选设计系统：[awesome-design-md](https://github.com/VoltAgent/awesome-design-md)（Apple / Vercel / Linear / Stripe / Notion / Airbnb 等 60+ 品牌风格）

## 部署到线上（可选）

```bash
cd trip-map-builder
git init && git add . && git commit -m "trip map"
# GitHub 创建仓库后
git remote add origin <your-repo-url>
git push -u origin main
# 在 vercel.com/new 导入仓库 — 推送即自动部署
```
