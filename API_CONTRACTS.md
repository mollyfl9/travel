# 后端接口契约

后端：Cloudflare Workers。仅做代理，不存用户数据。
所有接口返回 JSON。AI 输出必须符合 JSON Schema。

## 通用
- 认证：可选，用 App 生成的匿名 device token
- 错误格式：
```json
{ "error": { "code": "string", "message": "string", "retryable": true } }
```
- 超时：AI 30s，搜索 10s，地图 10s
- 重试：指数退避，最多 3 次

## 1. 发现景点
POST /api/attractions/discover

请求：
```json
{
  "destination": "东京",
  "countryCode": "JP",
  "days": 5,
  "styles": ["food", "shopping"],
  "preferences": ["动漫", "美食"],
  "avoid": ["爬山"],
  "constraints": {
    "travelers": 2,
    "pace": "relaxed",
    "budget": 8000,
    "physical": ["怕晒"],
    "diet": ["不吃辣"]
  }
}
```

响应：
```json
{
  "candidates": [
    {
      "name": "浅草寺",
      "lat": 35.7148,
      "lng": 139.7967,
      "address": "东京都台东区浅草2-3-1",
      "sourceUrls": ["https://..."],
      "heatScore": 92,
      "rating": 4.5,
      "reviewCount": 128000,
      "tags": ["寺庙", "人文", "地标"],
      "suggestedDurationMin": 90,
      "ticketPrice": 0,
      "openHours": "06:00-17:00",
      "suitableFor": ["人文", "摄影"],
      "notSuitableFor": ["怕人多"],
      "warnings": ["高峰排队久"],
      "aiScore": 88
    }
  ]
}
```

实现：搜索 API + 地图 POI + AI 结构化总结。

## 2. 决策辅助
POST /api/attractions/evaluate

请求：
```json
{
  "destination": "东京",
  "userProfile": { "physical": ["恐高"], "pace": "relaxed", "budget": 8000 },
  "attractions": [{ "name": "东京塔", "tags": ["地标", "观景"] }]
}
```

响应：
```json
{
  "evaluations": [
    {
      "name": "东京塔",
      "verdict": "not_recommended",
      "reason": "观景台高度较高，用户标注恐高",
      "alternatives": ["东京晴空塔裙楼商场", "浅草文化观光中心展望台"]
    }
  ]
}
```

## 3. 生成路线
POST /api/itinerary/generate

请求：
```json
{
  "destination": "东京",
  "startDate": "2026-04-01",
  "endDate": "2026-04-05",
  "pace": "relaxed",
  "hotel": { "lat": 35.68, "lng": 139.76 },
  "attractions": [{ "id": "a1", "name": "浅草寺", "lat": 35.7148, "lng": 139.7967, "suggestedDurationMin": 90 }],
  "transportPreference": ["walk", "transit"]
}
```

响应：
```json
{
  "days": [
    {
      "date": "2026-04-01",
      "activities": [
        {
          "order": 1,
          "type": "attraction",
          "name": "浅草寺",
          "lat": 35.7148,
          "lng": 139.7967,
          "time": "09:00",
          "durationMin": 90
        }
      ]
    }
  ]
}
```

约束：地理聚类、营业时间、用餐时间、节奏匹配。

## 4. 交通方案
POST /api/transport/route

请求：
```json
{
  "from": { "lat": 35.68, "lng": 139.76, "type": "hotel" },
  "to": { "lat": 35.7148, "lng": 139.7967, "type": "activity" },
  "mode": "transit",
  "region": "JP",
  "departAt": "2026-04-01T09:00:00+09:00"
}
```

响应：
```json
{
  "leg": {
    "mode": "transit",
    "durationMin": 32,
    "distanceMeters": 5400,
    "fare": 210,
    "polyline": "...",
    "steps": [
      { "type": "walk", "durationMin": 5, "instruction": "步行 350 米至浅草站 3 号口进站", "exitNo": "3" },
      { "type": "subway", "lineName": "地铁银座线", "lineColor": "#FF9500", "fromStop": "浅草", "toStop": "上野", "stopCount": 3, "durationMin": 7, "instruction": "乘坐银座线 3 站至浅草站" },
      { "type": "walk", "durationMin": 4, "exitNo": "2", "instruction": "从 2 号口出站，步行 200 米" }
    ],
    "source": "amap",
    "confidence": "high"
  }
}
```

出口号补全逻辑：
1. 调地图路径规划得到路线
2. 对到达站坐标 100m 内 POI 搜索「地铁出口」/「X号线 X站 出口」
3. 匹配最近出口 → exitNo
4. 匹配不到 → exitNo 为空，exitHint = "按站内指示出站"，confidence = "low"
5. 前端不因出口缺失报错

缓存：fromId + toId + mode 为键，本地 SQLite 缓存，TTL 24h。

## 5. 沿途推荐
POST /api/enroute/recommend

请求：
```json
{
  "polyline": "...",
  "bufferMeters": 500,
  "types": ["food", "cafe", "convenience", "pharmacy", "shopping"],
  "openAt": "2026-04-01T12:00:00+09:00",
  "limit": 10
}
```

响应：
```json
{
  "recommendations": [
    {
      "type": "food",
      "name": "一兰拉面 浅草店",
      "rating": 4.3,
      "distanceMeters": 180,
      "priceLevel": 2,
      "openNow": true,
      "lat": 35.7135,
      "lng": 139.7960,
      "reason": "顺路，评分高，营业中"
    }
  ]
}
```

## 6. 酒店搜索
POST /api/hotel/search

请求：
```json
{ "query": "东京 浅草", "region": "JP" }
```

响应：
```json
{
  "results": [
    { "name": "浅草豪景酒店", "address": "...", "lat": 35.71, "lng": 139.79 }
  ]
}
```

## 7. 地理编码 / 反查
POST /api/geo/geocode
POST /api/geo/reverse

用于：
- 目的地 → 国家码 + 城市
- 照片 EXIF GPS → 地点
- 老照片归类

## 8. AI 提示词要求
- 所有 AI 调用要求返回 JSON
- 使用 function calling / JSON mode
- 输出必须通过 JSON Schema 校验
- 校验失败 → 重试一次 → 仍失败 → 返回降级结果
- AI 生成内容在 UI 标注「AI 生成」

## 9. 密钥
后端环境变量：
```
OPENAI_API_KEY
DEEPSEEK_API_KEY
AMAP_KEY
MAPBOX_TOKEN
SEARCH_API_KEY
```
前端禁止出现任何密钥。
