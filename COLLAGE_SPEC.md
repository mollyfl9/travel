# 拼图方案

## 目标
旅行后，用户选几张照片，一键拼成一张，导出到相册，用于发朋友圈。

## 两条路径

### 路径 A：固定模板拼图（Phase 3，默认）
- App 端用 Skia 确定性渲染
- 离线、免费、像素精确、可控
- 模板：1 / 2 / 3 / 4 / 6 / 9 图 / 杂志风 / 胶片风 / 拍立得

### 路径 B：信封拼贴（P1 增值，可选）
- 基于 Codex Skill travel-envelope 的风格规则
- 后端复刻 prompt，调图像生成 API
- 效果最接近原 Skill，但有成本、联网、延迟、结果不稳定

---

## 路径 A：固定模板拼图

### 布局引擎
输入 N 张图，输出矩形布局数组。

```ts
type Slot = { x: number; y: number; w: number; h: number }
type Layout = { id: string; slots: Slot[]; aspectRatio: number }
```

### 智能裁剪
- 按图片主体（人脸 / 中心 / 显著区域）做 focus point 裁剪
- 人脸检测用 MLKit，无脸时用显著性区域

### 去重
- 感知哈希（pHash）
- 相似度阈值 0.9 以上视为重复

### 选图打分
- 清晰度：拉普拉斯方差
- 曝光：直方图分布
- 人脸：含人脸加分
- 地标：含地标加分

### 顺序
按拍摄时间或地理位置聚类。

### 模板清单（初版）
| ID | 名称 | 槽位数 | 比例 |
|---|---|---|---|
| grid-1 | 单图 | 1 | 3:4 |
| grid-2 | 双拼 | 2 | 3:4 |
| grid-3 | 三拼 | 3 | 3:4 |
| grid-4 | 四宫格 | 4 | 1:1 |
| grid-6 | 六宫格 | 6 | 3:4 |
| grid-9 | 九宫格 | 9 | 1:1 |
| magazine-1 | 杂志风 | 3-5 | 3:4 |
| film-strip | 胶片条 | 3-5 | 3:4 |
| polaroid | 拍立得 | 4 | 3:4 |

### 可调参数
间距 gap、圆角 cornerRadius、背景色 background、边框 border。

### 导出
- 1080×1920 或 3:4 高分辨率
- 保存到相册
- 可选 App 水印（可关）

### 数据结构
见 DATA_SCHEMA.md 的 CollageLayout 和 Collage。

---

## 路径 B：信封拼贴（基于 travel-envelope）

### 来源
GitHub Skill HwawH-J-Y/travel-envelope。
它是 Codex Skill，依赖 image_gen，不能直接跑在 App 里。
所以：抽取其风格规则，作为后端 AI 合成的 prompt 骨架。

### 风格规则（从 Skill 抽取）
1. 选一张风景原图作为全幅背景
2. 从背景色提取淡色低饱和纸张色
3. 信封宽度占画布 58–68%，保证风景可见
4. 信封是悬浮拼贴层，不落地、无接地阴影
5. 紧凑不对称簇状布局，元素间真实遮挡
6. 三连图 / 人像序列保持原顺序做胶片条
7. 不要求有人。无人像时，地标 / 建筑 / 食物 / 交通 / 票根做偏心焦点
8. 前景元素 < 5 且提供目的地时，自动补 1–3 个目的地小装饰
9. 标题克制，不用编造日期、票根信息、个人记忆

### 后端实现
POST /api/collage/envelope

请求：
```json
{
  "photos": ["url1", "url2"],
  "destination": "London",
  "title": "London",
  "backgroundPhotoIndex": 0
}
```

响应：
```json
{ "imageUrl": "...", "expiresAt": "..." }
```

实现：
- 后端组装 prompt（上述 9 条规则 + 用户照片）
- 调图像生成 API（gpt-image-1 / Gemini / 即梦）
- 返回图片 URL，App 下载后存本地

### 约束
- 成本：每张约 $0.02–0.19，用户主动触发
- 延迟：10–60s，显示进度
- 不稳定：同一输入可能结果不同
- 合规：AI 生成内容标注；用户照片上传需授权

### UI
- 入口在拼图页，标记「AI 信封拼贴（Beta）」
- 提示：会上传照片到服务器，可能产生费用，结果不保证一致
- 用户确认后才执行

---

## 不做
- 不做自动视频
- 不做 AI 文案
- 不做实时协作拼图
