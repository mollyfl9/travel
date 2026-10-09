# 任务清单

按 Phase 拆分，每项是可运行的增量。
每个任务完成后：本地跑通 → 提交 → 再进下一个。
先写接口和数据结构，再写 UI。

---

## Phase 0：项目初始化

- [ ] 0.1 初始化 React Native + Expo Dev Client 项目
- [ ] 0.2 接入 TypeScript、ESLint、Prettier
- [ ] 0.3 目录结构：src/features、src/shared、src/db、src/api
- [ ] 0.4 接入 expo-sqlite，建空库 + 迁移框架
- [ ] 0.5 接入 React Navigation（Tab + Stack）
- [ ] 0.6 主题系统：src/shared/theme（按 UI_SPEC）
- [ ] 0.7 通用组件：Card、Button、Tag、Badge、Skeleton、EmptyState
- [ ] 0.8 初始化 Cloudflare Workers 项目，写 /health
- [ ] 0.9 .env.example + 前端读环境变量
- [ ] 0.10 CI：lint + typecheck

---

## Phase 1：旅行规划

### 1.1 新建旅行
- [ ] Trip 表 + DAO
- [ ] 新建旅行页：目的地、日期、人数、预算、节奏
- [ ] 风格选择卡片
- [ ] 限制条件表单
- [ ] 必去 / 避雷 / 已去过
- [ ] 目的地 geocoding → 国家码 + 城市
- [ ] 保存草稿 + 完成创建

### 1.2 景点发现
- [ ] POST /api/attractions/discover 后端
- [ ] 搜索 API + 地图 POI 接入
- [ ] AI 结构化总结 → JSON Schema 校验
- [ ] 无 API Key 时 mock 数据
- [ ] AttractionCandidate 表 + DAO
- [ ] 景点发现页：卡片瀑布流
- [ ] 景点卡片完整字段
- [ ] 标签筛选

### 1.3 筛选与决策
- [ ] 收藏 / 排除 / 待定 交互
- [ ] 筛选决策页：左右滑
- [ ] POST /api/attractions/evaluate 后端
- [ ] 决策辅助卡
- [ ] 批量操作

### 1.4 路线生成
- [ ] POST /api/itinerary/generate 后端
- [ ] 地理聚类算法
- [ ] 营业时间 / 用餐时间 / 节奏约束
- [ ] DayPlan + Activity 表 + DAO
- [ ] 路线详情页：上地图下时间轴
- [ ] 地图与时间轴联动
- [ ] Activity 编辑

### 1.5 酒店
- [ ] Hotel 表 + DAO
- [ ] POST /api/hotel/search 后端
- [ ] 酒店录入页
- [ ] 地图显示酒店

### 1.6 交通指引
- [ ] POST /api/transport/route 后端
- [ ] 高德 / Mapbox 路径规划接入
- [ ] 出口号补全逻辑
- [ ] TransportLeg + TransportStep 表 + DAO
- [ ] 缓存逻辑
- [ ] 交通卡片组件
- [ ] 交通卡片插入时间轴
- [ ] 展开分段步骤
- [ ] 导航 deep link
- [ ] 未安装地图回退网页

### 1.7 沿途推荐（P1，可延后）
- [ ] POST /api/enroute/recommend 后端
- [ ] 路径缓冲区 POI 搜索
- [ ] 排序逻辑
- [ ] 沿途推荐列表

### 1.8 首页与我的旅行
- [ ] 旅行卡片流
- [ ] 状态筛选
- [ ] 删除 / 复制旅行

---

## Phase 2：记录

### 2.1 拍照
- [ ] VisionCamera 接入
- [ ] 相机权限
- [ ] 相机页全屏取景
- [ ] 顶部场景切换
- [ ] 拍照保存本地
- [ ] 从 Activity 进入相机
- [ ] 照片自动挂到 Activity

### 2.2 实时滤镜
- [ ] Skia shader 实时渲染管线
- [ ] 参数滤镜实现
- [ ] LUT 滤镜实现
- [ ] 滤镜清单 [待确认 skill]
- [ ] 底部滤镜横滑
- [ ] 强度滑杆
- [ ] 性能降级
- [ ] 拍后快速编辑

### 2.3 相册
- [ ] Photo 表 + DAO
- [ ] 相册首页：分类入口
- [ ] Album 表 + 自动生成逻辑
- [ ] 相册详情：瀑布流
- [ ] 照片详情
- [ ] 多选 + 批量操作
- [ ] 收藏

### 2.4 老照片导入
- [ ] 相册导入入口
- [ ] EXIF 读取
- [ ] 归类降级链
- [ ] 导入进度与结果

### 2.5 足迹地图
- [ ] Visit 表 + DAO
- [ ] Country / Region 表 + 初始化数据
- [ ] 全球 / 全国自动判定逻辑
- [ ] 世界地图 GeoJSON + Skia 绘制
- [ ] 中国地图 GeoJSON + 省市下钻
- [ ] 已访问区域着色
- [ ] 足迹首页
- [ ] 国家详情
- [ ] 城市详情

---

## Phase 3：回顾与分享

### 3.1 回顾场域
- [ ] Recap 表 + 数据聚合
- [ ] 回顾页三段式布局
- [ ] 无生成按钮
- [ ] 自动选图逻辑
- [ ] 去重

### 3.2 朋友圈拼图
- [ ] CollageLayout 表 + 模板数据
- [ ] Skia 布局引擎
- [ ] 智能裁剪
- [ ] 模板选择页
- [ ] 选图 + 排序 + 替换
- [ ] 微调参数
- [ ] 导出到相册
- [ ] 水印开关

### 3.3 信封拼贴（可选）
- [ ] POST /api/collage/envelope 后端
- [ ] 组装 prompt
- [ ] 接入图像生成 API
- [ ] 前端入口 + 用户确认
- [ ] 进度显示
- [ ] 结果保存本地
- [ ] 失败重试 + 降级

---

## 通用任务

- [ ] 错误处理统一
- [ ] 骨架屏组件
- [ ] 空状态组件
- [ ] 隐私政策页
- [ ] 权限说明文案
- [ ] 首次启动引导
- [ ] 设置页
- [ ] 数据导出 / 清空
