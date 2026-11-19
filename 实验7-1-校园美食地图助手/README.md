# 校园美食地图助手（实验 7-1 · Vibe Coding）

用自然语言驱动开发，三轮对话完成一个可运行的校园美食地图应用：定位、POI 搜索、地图打点、价格/口味筛选、智能推荐理由、步行路线规划。

> 本目录对应《实验 7-1：Vibe Coding 实战：多轮对话打造校园美食推荐小助手》。

---

## 一、快速开始

### 方式 A：直接双击（最简单）

双击 `index.html`，浏览器会打开应用。首次进入会弹出 Key 配置窗口，可先点 **「先用演示数据」** 体验全部功能。

### 方式 B：本地静态服务器（推荐）

```bash
# 在项目目录下任选其一
python -m http.server 8080
# 或
npx serve .
```

然后浏览器访问 <http://localhost:8080>。

---

## 二、配置高德地图 Key

应用不内置任何 Key，所有 Key 只保存在**本机浏览器 localStorage**，不会进入代码仓库。

到 [高德开放平台控制台](https://console.amap.com/dev/key/app) 创建应用后，按用途申请两类 Key：

| Key 类型 | 平台 | 用途 | 是否必填 |
| --- | --- | --- | --- |
| Web 服务 | Web 服务 | POI 搜索、步行路线 | 必填（不填则只能用演示数据） |
| Web 端（JS API） | Web 端（JS API） | 地图底图、校园定位（`AMap.Geolocation`） | 定位必填；不填则地图待激活且定位降级 |

JS API 2.0 还需要在控制台为 Key 配置**安全密钥 securityJsCode**，在应用弹窗中一并填入。

填写位置：右上角 **「配置 Key」** → 保存并刷新。

---

## 三、功能清单

| 模块 | 能力 | 实现要点 |
| --- | --- | --- |
| 定位 | 校园网下 `AMap.CitySearch` 限定城市 + `AMap.Geolocation` 浏览器定位 + `AMap.Geocoder` 逆地理，显示详细地址 | `AMap.Geolocation`（`citylimit: true`）→ `AMap.Geocoder` |
| 搜索 | 关键词搜索周边餐厅，展示名称、距离、人均、评分 | `/v3/place/text`，`extensions=all` 获取 `biz_ext` |
| 地图 | 高德地图底图、红色自定义标记、信息窗体 | JS API 2.0 `AMap.Marker` + `AMap.InfoWindow` |
| 联动 | 点列表定位标记，点标记高亮列表 | 共享 `selectedId` 状态 |
| 排序 | 距离最近 / 评分最高 | 前端排序，评分相同按距离兜底 |
| 筛选 | 价格三档（<30 / 30-80 / >80）、六类口味 | 前端筛选 |
| 推荐理由 | 结合价格、距离、评分、口味自动生成一句推荐 | `buildReason()` |
| 路线规划 | “去这里”按钮，步行距离/时间/分步指引 | `/v3/direction/walking`，Polyline 绘制 |
| 降级 | 接口失败或无 Key 时使用内置 Mock 数据 | 12 条真实感演示数据 |

---

## 四、目录结构

```
实验7-1-校园美食地图助手/
├── index.html                      # 单文件应用（HTML + CSS + JS）
├── README.md                       # 本文件
├── 实验报告.md                      # 实验目的、步骤、测试与总结
└── skill/
    └── food-recommender/
        └── SKILL.md                # 任务二固化的美食推荐 SKILL
```

---

## 五、对应实验步骤

- **任务一**：新建项目 `campus-food-map-assistant`、申请高德 Key、配置 `mcp_settings.json` 中的高德 MCP 服务。
- **任务二**：创建 `food-recommender` SKILL，把距离优先、价格分档、口味分类、评分参考、输出格式固化为规则（见 `skill/food-recommender/SKILL.md`）。
- **任务三**：三轮对话交付
  1. 第一轮：基础框架（定位 + POI 搜索 + 地图展示 + 排序联动）
  2. 第二轮：路线规划 + 智能推荐 + 价格/口味筛选
  3. 第三轮：修复「距离显示 0m」与「地图缩放卡顿」

---

## 六、关键实现说明

### 1. 距离为什么不再显示 0m

优先读取高德返回的 `poi.distance`；当接口未返回或为 0 时，用 **Haversine 公式** 基于用户坐标与餐厅坐标自行计算，避免依赖单一字段。超过 1000 米自动换算为公里。

### 2. 地图性能优化

地图初始化参数：

```js
new AMap.Map('map', {
  zooms: [12, 18],              // 限制缩放范围
  enableHighResolution: false,  // 降低分辨率，减少绘制压力
  fadeEffect: false,            // 关闭动画特效
  viewMode: '2D'
});
```

标记采用**分批渲染**（每帧 8 个，`requestAnimationFrame`），避免一次性创建大量 DOM 标记造成掉帧。

### 3. MCP 与 REST 的关系

MCP 工具本质是对高德服务的封装。HTML 页面运行在浏览器中，无法直接调用 MCP 工具，因此：

- **搜索与路线**：用 `fetch` 直接请求 REST 接口（`/v3/place/text`、`/v3/direction/walking`），与 MCP 调用的是同一套服务。
- **定位与地图**：使用高德 JS API 2.0 的 `AMap.CitySearch`（校园网 IP 识别城市）+ `AMap.Geolocation`（`citylimit: true` 限定城市）+ `AMap.Geocoder`（逆地理编码），避免校园网 IP 定位漂移。

---

## 七、常见问题

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| 搜索一直返回演示数据 | 未填 Web 服务 Key，或 Key 未通过个人开发者认证 | 配置 Key；确认控制台配额与认证状态 |
| 地图显示“待激活” | 未填 JS API Key | 填写 Web 端（JS API）Key |
| 地图报 `INVALID_USER_DOMAIN` / 权限错误 | JS API Key 未配安全密钥或域名白名单 | 填 `securityJsCode`；本地调试可加 `localhost` |
| 浏览器控制台 CORS 报错 | 用了不支持跨域的方式调用 | 本实现使用高德官方 REST 接口，默认支持跨域；检查网络与 Key 类型 |
| 定位失败 / 落到默认位置 | 未配 JS API Key、未允许定位权限、非 localhost/HTTPS 打开、安全密钥缺失 | 配置 JS API Key 与安全密钥；用 localhost/HTTPS 打开并允许定位；失败时自动使用演示位置 |

---

## 八、说明

- 本项目为教学实验作品，演示数据仅用于功能展示。
- 代码不包含任何真实 API Key、账号或密钥。
- 高德地图相关服务遵循高德开放平台的服务条款。