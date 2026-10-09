# Travel App（项目代号待定）

个人旅行规划 + 记录 + 回顾 App。

## 一句话
输入目的地和偏好，发现热门景点、辅助决策、生成路线与交通指引；旅行中拍照加滤镜；旅行后按国家/地区归类照片，一键回顾。

## 技术栈
- App：React Native (Expo Dev Client)
- 渲染：@shopify/react-native-skia（滤镜、拼图、足迹地图绘制）
- 相机：react-native-vision-camera
- 本地存储：SQLite (expo-sqlite)
- 图片：本地文件系统，无云同步
- 地图：高德 SDK（国内）+ Mapbox（全球），按目的地自动切换
- 后端：Cloudflare Workers（仅做 API 代理，不存数据）
- AI：OpenAI / DeepSeek，走代理

## 目录约定
- src/features/ 按功能模块组织
- src/shared/ 通用组件与工具
- src/db/ SQLite schema 与迁移
- src/api/ 后端代理客户端
- backend/ Cloudflare Workers

## 开发阶段
- Phase 1：旅行规划（景点发现 + 筛选 + 路线 + 酒店 + 交通）
- Phase 2：记录（拍照 + 滤镜 + 相册 + 足迹地图 + 老照片导入）
- Phase 3：回顾与分享（回顾场域 + 朋友圈拼图）

## 关键文档
- PRD.md 产品需求
- USER_FLOW.md 用户流程
- DATA_SCHEMA.md 数据模型
- API_CONTRACTS.md 后端接口
- UI_SPEC.md 视觉规范
- FILTER_SPEC.md 滤镜方案
- COLLAGE_SPEC.md 拼图方案（含 travel-envelope 规则）
- TASKS.md 任务清单
- ACCEPTANCE.md 验收标准
- .env.example 环境变量

## 密钥
所有第三方密钥只放后端。前端用 .env 占位，禁止提交真实密钥。
