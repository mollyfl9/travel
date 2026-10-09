# 滤镜方案

> 状态：滤镜 GitHub skill 尚未提供 [待确认]。本文件先按 LUT + 参数滤镜方案写，收到 skill 后替换实现细节。

## 目标
- 实时预览（相机取景阶段就带滤镜）
- 拍照后出成片，可调强度
- 按场景分类：风景 / 美食 / 购物 / 人像 / 夜景 / 街拍

## 技术方案
- VisionCamera 取景 → 帧数据 → Skia shader 实时渲染
- 滤镜分两类：
  - 参数滤镜：亮度、对比度、饱和度、色温、锐化、暗角、色调分离
  - LUT 滤镜：512×512 PNG，shader 查表
- 实时预览用参数滤镜（性能）
- 拍照后应用 LUT 出成片（效果）

## 数据结构
见 DATA_SCHEMA.md 的 Filter 和 Photo.filterParams。

## 滤镜清单（初版）
| ID | 名称 | 场景 | 类型 |
|---|---|---|---|
| landscape-vivid | 风景·鲜明 | landscape | param |
| landscape-film | 风景·胶片 | landscape | lut |
| food-warm | 美食·暖调 | food | param |
| food-clean | 美食·清透 | food | lut |
| shopping-bright | 逛街·明亮 | shopping | param |
| portrait-soft | 人像·柔光 | portrait | lut |
| night-neon | 夜景·霓虹 | night | lut |
| street-mono | 街拍·黑白 | street | param |

[待确认] 收到 skill 后替换为实际滤镜集。

## 强度控制
- 所有滤镜 0–100 可调
- 默认 80
- 实时预览和成片强度一致

## 性能要求
- 实时预览 ≥ 24fps（中端机）
- 帧处理耗时 ≤ 16ms
- 降级：低端机自动关闭实时预览，改为拍后应用

## UI
- 相机页底部横向滚滤镜缩略图
- 选中高亮，显示强度滑杆
- 顶部场景切换（风景 / 美食 / 购物 / 人像 / 夜景 / 街拍）
- 拍后进入快速编辑：强度 / 裁剪 / 旋转 / 对比原图

## 集成 skill 的接口约定
收到滤镜 skill 后，按以下方式接入：
1. 若提供 LUT 资源 → 放入 src/assets/luts/，注册到 Filter 表
2. 若提供 shader 代码 → 放入 src/features/camera/shaders/
3. 若提供滤镜库 → 在 src/features/camera/filters/ 封装适配层
4. 统一暴露接口：
```ts
type FilterEngine = {
  list(): Filter[]
  preview(filterId: string, intensity: number): void
  capture(filterId: string, intensity: number): Promise<string> // 返回文件路径
}
```
