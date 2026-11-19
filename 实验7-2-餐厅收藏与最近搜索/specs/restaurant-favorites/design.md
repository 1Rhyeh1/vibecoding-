# 实现方案设计书：餐厅收藏与最近搜索

| 项目 | 内容 |
| --- | --- |
| 设计编号 | DESIGN-2026-0716-01 |
| 对应规格 | SPEC-2026-0716-01 v1.1 |
| 技术形态 | 单文件静态 Web 应用（HTML + CSS + ES6） |
| 归档路径 | `specs/restaurant-favorites/design.md`（实验过程中由码道写入 `.codeartsdoer/specs/restaurant-favorites/design.md`） |

---

## 一、设计概述

在实验 7-1 的校园美食地图助手基础上，以**最小侵入**方式新增“餐厅收藏”和“最近搜索”，并补充规范约束。整体仍是零依赖、双击即可运行的单文件应用，便于交付与维护。

设计原则：

1. **数据与视图分离**：收藏、历史搜索、餐厅结果统一放入 `state`，交互只改状态，渲染函数统一输出。
2. **安全前置**：写入 `localStorage` 前过滤，渲染 DOM 时转义。
3. **失败可降级**：无 Key、接口异常、地图加载失败均不影响核心功能。
4. **约束可验证**：每条规格约束对应可检查的函数或测试项。

---

## 二、技术选型

| 层次 | 选型 | 说明 |
| --- | --- | --- |
| 视图 | 原生 HTML + CSS | 无构建依赖，单文件交付 |
| 逻辑 | 原生 ES6 | `const`/`let`、箭头函数、模板字符串 |
| 持久化 | `localStorage` | 收藏与最近搜索本地保存 |
| 地图与定位 | 高德 JS API 2.0 | 地图底图、`AMap.CitySearch` + `AMap.Geolocation`（`citylimit`）定位、`AMap.Geocoder` 逆地理 |
| 数据 | 高德 Web 服务 REST API | POI 搜索、步行路线 |
| 兜底 | 内置 Mock 数据 | 12 条演示餐厅 |

---

## 三、模块划分

```
index.html
├── 常量层
│   ├── STORAGE            存储键名
│   ├── MAX_FAVORITES      收藏上限 20
│   ├── MAX_RECENT_SEARCHES 历史上限 8
│   └── MOCK_RESTAURANTS   演示数据
├── 工具层
│   ├── escapeHtml         HTML 转义
│   ├── stripDangerous     XSS 过滤
│   ├── haversine          距离计算
│   ├── formatDistance     距离格式化
│   └── fetchJson          带超时请求
├── 规则层
│   ├── Validator          ID / 关键词 / 收藏项校验
│   └── XSSFilter          存储前过滤
├── 数据层
│   ├── readFavorites / writeFavorites
│   └── readRecentSearches / writeRecentSearches
├── 状态层
│   └── state              结果集、筛选、选中、收藏、地图实例
├── 视图层
│   ├── renderList         餐厅列表
│   ├── renderFavorites    收藏面板
│   ├── renderRecentSearches 最近搜索
│   ├── renderMarkers      地图标记
│   ├── openInfoWindow     信息窗体
│   └── showRouteCard      路线卡片
├── 服务层
│   ├── locate             CitySearch + Geolocation（citylimit）
│   ├── reverseGeocode     AMap.Geocoder 逆地理
│   ├── searchRemote       POI 搜索
│   ├── searchMock         演示搜索
│   ├── planRoute          步行路线
│   └── ensureMap          地图初始化
└── 交互层
    └── bindEvents         统一事件绑定
```

---

## 四、数据模型

### 4.1 Restaurant（搜索结果）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 餐厅唯一标识，非空 |
| `name` | string | 名称 |
| `cuisine` | string | 口味分类（川菜/粤菜/日料/西餐/小吃/奶茶） |
| `cost` | number | 人均价格（元） |
| `rating` | number | 评分（0-5） |
| `lng` / `lat` | number | 经纬度 |
| `address` | string | 地址 |
| `distance` | number | 距用户距离（米），运行时计算 |
| `reason` | string | 推荐理由，运行时生成 |

### 4.2 FavoriteItem（收藏项）

`Restaurant` 的安全子集，只保留持久化必要字段：

```json
{
  "id": "m01",
  "name": "川渝人家川菜馆",
  "cuisine": "川菜",
  "address": "学院路 32 号",
  "cost": 28,
  "rating": 4.6,
  "lng": 116.3621,
  "lat": 39.9845
}
```

不存储 `distance` 与 `reason`：距离随定位变化，推荐理由可重新生成，避免脏数据。

### 4.3 存储结构

| 键名 | 类型 | 上限 | 说明 |
| --- | --- | --- | --- |
| `cfm_favorites` | `Array<FavoriteItem>` | 20 | 收藏列表，新收藏在前 |
| `cfm_recent_searches` | `Array<string>` | 8 | 关键词列表，新的在前 |
| `cfm_rest_key` | string | - | Web 服务 Key |
| `cfm_js_key` | string | - | JS API Key |
| `cfm_js_secret` | string | - | 安全密钥 |

---

## 五、数据流设计

### 5.1 校园网定位流

```
页面启动
  → ensureMap 动态加载 JS API
  → AMap.CitySearch.getLocalCity 获取校园网城市
  → AMap.Geolocation({ city, citylimit: true }).getCurrentPosition
  → 成功：position + formattedAddress
  → 地址为空：AMap.Geocoder 逆地理编码
  → state.userLoc / state.city / 地址栏
  → 地图 setZoomAndCenter
  → 失败：降级演示位置 + toast 提示
```

### 5.2 搜索流

```
用户输入关键词
  → sanitizeKeyword 过滤
  → Validator.isValidKeyword 校验
  → 有 REST Key ? searchRemote : searchMock
  → 归一化 + 计算距离 + 生成推荐理由
  → state.restaurants
  → applyFilters（价格/口味/排序）
  → state.filtered
  → renderList + renderMarkers + renderFavorites
  → addRecentSearch（记录历史）
```

### 5.3 收藏流

```
点击收藏图标
  → Validator.isValidRestaurantId
  → 已收藏 ? remove : add
  → add 前：上限校验（20）→ XSSFilter.sanitizeRestaurant
  → writeFavorites(localStorage)
  → renderFavorites + renderList
  → toast 反馈
```

### 5.4 收藏定位流

```
点击收藏项
  → 在当前结果中查找同 ID 餐厅
  → 找到：复用，实时刷新距离
  → 未找到：用收藏数据构造临时对象
  → 地图 setZoomAndCenter + openInfoWindow
  → 新增高亮标记 favoriteMarker
```

### 5.5 最近搜索流

```
搜索完成
  → 关键词去重
  → 新关键词置顶
  → 截断至 8 条
  → writeRecentSearches
  → renderRecentSearches
```

---

## 六、关键设计决策

### 6.1 为什么第二轮才加规范约束

先让功能跑通，再统一加约束，避免“功能未定型就过度设计”。第二轮走完整的 SDD 四阶段，对已有代码做**增强而非推倒重写**：

- 需求规格：追加非功能约束章节
- 实现方案：新增 Validator / XSSFilter 模块
- 编码规划：插入重构类任务，排在校验与渲染改造之前
- 任务执行：逐项改造并回归

### 6.2 XSS 为什么做两层

| 层次 | 位置 | 作用 |
| --- | --- | --- |
| 存储过滤 | 写入 `localStorage` 前 | 防止恶意内容持久化，污染后续会话 |
| 渲染转义 | 插入 DOM 前 | 防止被篡改的本地数据或接口数据直接执行 |

只做一层都不够：只存不转义，接口数据仍有风险；只转义不过滤，恶意脚本会长期驻留本地。

### 6.3 校验为什么集中在 Validator

分散的 `if (!id)` 容易漏。集中到一个模块后：

- 规格约束“ID 必须为字符串且非空”只在一处实现
- `console.warn` 格式统一，便于排查
- 后续新增字段校验只需扩展模块

### 6.4 收藏为什么存坐标而不存距离

距离依赖用户当前位置，定位变化后旧距离立即失效。存坐标、展示时实时计算，保证列表与地图数字一致。

### 6.5 为什么保留演示数据

教学演示场景常缺少 Key、网络受限或高德配额用尽。Mock 数据保证任何环境下都能完整展示收藏、最近搜索、筛选、路线等功能。

---

## 七、性能与兼容设计

| 项 | 方案 |
| --- | --- |
| 地图标记 | 分批渲染，每帧 8 个，`requestAnimationFrame` |
| 地图缩放 | `zooms: [12, 18]` |
| 地图清晰度 | `enableHighResolution: false` |
| 地图动画 | `fadeEffect: false` |
| 列表滚动 | 容器 `overflow-y: auto` + 细滚动条样式 |
| 移动端 | `@media (max-width: 940px)` 切换上下布局 |
| 请求超时 | `AbortController` + 9 秒超时 |
| 空状态 | 列表、收藏均有空状态提示 |

---

## 八、任务规模

| 轮次 | 主任务 | 子任务 | 说明 |
| --- | --- | --- | --- |
| 第一轮（v1.0） | 10 | 36 | 收藏 + 最近搜索功能实现 |
| 第二轮（v1.1） | 11 | 24 | 规范约束增强与重构 |

详细拆解见 `tasks.md`。

---

## 九、风险与应对

| 风险 | 影响 | 应对 |
| --- | --- | --- |
| localStorage 被禁用或写满 | 收藏/历史丢失 | 写入 `try/catch`，失败时仅提示，不影响主流程 |
| 本地数据被手动篡改 | 渲染异常或 XSS | 读取时结构校验 + 渲染转义 |
| 高德 Key 失效 | 搜索/路线不可用 | 自动降级 Mock 数据并提示 |
| JS API 未配安全密钥 | 地图不加载 | 地图区域显示待激活，保留搜索与收藏功能 |
| POI 字段缺失 | 价格/评分异常 | 归一化时给默认值 |
| 收藏上限误判 | 无法继续收藏 | 每次写入前按实际数组长度校验 |