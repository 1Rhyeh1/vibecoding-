# 校园美食地图助手 · Vibe Coding 与 SDD 实验合集

本仓库包含两个递进的课程实验，围绕同一个“校园美食地图助手”展开：

| 实验 | 主题 | 开发方式 | 交付物 |
| --- | --- | --- | --- |
| [实验 7-1](./实验7-1-校园美食地图助手/) | Vibe Coding 多轮对话开发 | 自然语言驱动 + MCP + SKILL | 可运行的美食地图应用 |
| [实验 7-2](./实验7-2-餐厅收藏与最近搜索/) | SDD 规范驱动开发 | 规格书 → 方案 → 任务 → 验收 → 归档 | 完整应用 + 规格书 + 报告 |

两个实验都是**纯静态单文件应用**，无需安装依赖，双击 `index.html` 即可运行。没有高德 Key 时会自动使用内置演示数据，全部功能仍可体验。

---

## 一、目录结构

```
vibecoding作业/
├── README.md                              # 本文件
├── 实验7-1-校园美食地图助手/
│   ├── index.html
│   ├── README.md
│   ├── 实验报告.md
│   └── skill/food-recommender/SKILL.md
└── 实验7-2-餐厅收藏与最近搜索/
    ├── index.html
    ├── README.md
    ├── 实验报告.md
    └── specs/restaurant-favorites/
        ├── spec.md
        ├── design.md
        └── tasks.md
```

---

## 二、快速开始

```bash
# 任选一个实验目录，启动本地静态服务
cd 实验7-1-校园美食地图助手
python -m http.server 8080
```

浏览器打开 <http://localhost:8080>。

也可以直接双击对应目录下的 `index.html`。

首次打开会提示配置高德 Key，可先点 **「先用演示数据」**。

---

## 三、高德 Key 说明

仓库**不包含任何真实 Key**。Key 只保存在浏览器 `localStorage`，不会进入代码。

申请地址：<https://console.amap.com/dev/key/app>

| Key 类型 | 用途 | 是否必填 |
| --- | --- | --- |
| Web 服务 | POI 搜索、步行路线 | 是（否则使用演示数据） |
| Web 端（JS API） | 地图底图、校园定位（`AMap.Geolocation`） | 定位必填 |
| 安全密钥 securityJsCode | JS API 2.0 配套 | 配置 JS API 时必填 |

---

## 四、功能对照

| 功能 | 7-1 | 7-2 |
| --- | --- | --- |
| 校园网定位 + 详细地址（`AMap.Geolocation`） | ✅ | ✅ |
| 关键词搜索与 Mock 降级 | ✅ | ✅ |
| 地图标记与信息窗 | ✅ | ✅ |
| 列表 / 地图联动 | ✅ | ✅ |
| 距离 / 评分排序 | ✅ | ✅ |
| 价格 / 口味筛选 | ✅ | ✅ |
| 智能推荐理由 | ✅ | ✅ |
| 步行路线规划 | ✅ | ✅ |
| 餐厅收藏（20 家上限） | — | ✅ |
| 最近搜索（8 条） | — | ✅ |
| 校验 / XSS / 代码规范约束 | — | ✅ |
| 规格书 / 方案 / 任务归档 | — | ✅ |

---

## 五、技术栈

- 原生 HTML + CSS + ES6，无框架、无构建步骤
- 高德地图 JS API 2.0（底图、标记、`AMap.Geolocation` 校园定位）
- 高德 Web 服务 REST API（POI 搜索、步行路线）
- `localStorage`（收藏、最近搜索、Key 本地保存）
- 响应式布局，桌面左右分栏 / 移动端上下布局

---

## 六、上传 GitHub 前的检查

- [x] 不含任何 API Key、账号、密码
- [x] 不含 `node_modules`、构建产物
- [x] 每个实验可独立运行
- [x] 每个实验均有 README 与实验报告
- [x] 7-2 的规格书已随代码归档

推荐操作：

```bash
git init
git add .
git commit -m "feat: 完成实验7-1 Vibe Coding 与实验7-2 SDD 规范驱动开发"
git branch -M main
git remote add origin <你的仓库地址>
git push -u origin main
```

---

## 七、实验文档

- [实验 7-1 报告](./实验7-1-校园美食地图助手/实验报告.md)
- [实验 7-2 报告](./实验7-2-餐厅收藏与最近搜索/实验报告.md)
- [美食推荐 SKILL](./实验7-1-校园美食地图助手/skill/food-recommender/SKILL.md)
- [收藏功能规格书](./实验7-2-餐厅收藏与最近搜索/specs/restaurant-favorites/spec.md)
- [收藏功能实现方案](./实验7-2-餐厅收藏与最近搜索/specs/restaurant-favorites/design.md)
- [收藏功能任务规划](./实验7-2-餐厅收藏与最近搜索/specs/restaurant-favorites/tasks.md)

---

## 八、免责说明

本项目为教学实验作品，演示数据仅用于功能展示。高德地图相关服务遵循高德开放平台服务条款。