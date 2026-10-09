# 数据模型

存储：本地 SQLite。无云同步。所有表带 createdAt / updatedAt。

## Trip
```ts
Trip {
  id: string
  title: string
  destination: string
  destinationCountryCode?: string   // ISO 3166-1 alpha-2
  destinationCity?: string
  origin?: string
  startDate: string
  endDate: string
  travelers: number
  budget?: number
  pace: 'relaxed' | 'moderate' | 'packed'
  styles: string[]
  preferences: string[]
  avoid: string[]
  visitedBefore: string[]
  notes?: string
  coverPhotoId?: string
  status: 'planning' | 'ongoing' | 'completed'
}
```

## Hotel
```ts
Hotel {
  id: string
  tripId: string
  name: string
  address: string
  lat: number
  lng: number
  checkIn?: string
  checkOut?: string
  note?: string
}
```

## AttractionCandidate
```ts
AttractionCandidate {
  id: string
  tripId: string
  name: string
  lat: number
  lng: number
  address?: string
  sourceUrls: string[]
  heatScore: number
  rating?: number
  reviewCount?: number
  tags: string[]
  suggestedDurationMin: number
  ticketPrice?: number
  openHours?: string
  suitableFor: string[]
  notSuitableFor: string[]
  warnings: string[]
  aiScore: number
  decision: 'undecided' | 'keep' | 'exclude' | 'maybe'
}
```

## DayPlan
```ts
DayPlan {
  id: string
  tripId: string
  date: string
  index: number
}
```

## Activity
```ts
Activity {
  id: string
  dayId: string
  order: number
  time?: string
  type: 'attraction' | 'food' | 'shopping' | 'rest' | 'transport' | 'other'
  name: string
  address?: string
  lat?: number
  lng?: number
  durationMin?: number
  cost?: number
  notes?: string
  locked: boolean
  nearestStation?: { name: string; exitNo?: string; walkMin: number }
}
```

## TransportLeg
```ts
TransportLeg {
  id: string
  tripId: string
  fromType: 'hotel' | 'activity'
  fromId: string
  toType: 'hotel' | 'activity'
  toId: string
  mode: 'transit' | 'walk' | 'taxi' | 'drive'
  durationMin: number
  distanceMeters: number
  fare?: number
  polyline?: string
  steps: TransportStep[]
  source: 'amap' | 'mapbox' | 'google' | 'mock'
  fetchedAt: string
  confidence: 'high' | 'medium' | 'low'
}
```

## TransportStep
```ts
TransportStep {
  type: 'walk' | 'bus' | 'subway' | 'train' | 'transfer'
  lineName?: string
  lineColor?: string
  fromStop?: string
  toStop?: string
  exitNo?: string
  exitHint?: string
  stopCount?: number
  durationMin: number
  instruction: string
}
```

## Photo
```ts
Photo {
  id: string
  tripId?: string
  activityId?: string
  localPath: string
  thumbnailPath?: string
  takenAt: string
  lat?: number
  lng?: number
  filterId?: string
  filterParams?: Record<string, number>
  scene: 'landscape' | 'food' | 'shopping' | 'portrait' | 'night' | 'street' | 'other'
  width: number
  height: number
  orientation: 'portrait' | 'landscape' | 'square'
  isFavorite: boolean
  imported: boolean
}
```

## Filter
```ts
Filter {
  id: string
  name: string
  scene: string[]
  type: 'lut' | 'param'
  lutAsset?: string
  params?: Record<string, number>
  intensity: number
}
```

## Album
```ts
Album {
  id: string
  type: 'trip' | 'country' | 'city' | 'date' | 'scene' | 'smart'
  name: string
  coverPhotoId?: string
  photoCount: number
  filter?: Record<string, unknown>
}
```

## Visit
```ts
Visit {
  id: string
  tripId?: string
  countryCode: string
  regionName?: string
  cityName?: string
  startDate: string
  endDate: string
}
```

## Country / Region
```ts
Country {
  code: string
  name: string
  nameLocal: string
  continent: string
  visited: boolean
  tripCount: number
  photoCount: number
  firstVisitAt?: string
  lastVisitAt?: string
}

Region {
  id: string
  countryCode: string
  name: string
  level: 'province' | 'state' | 'city'
  visited: boolean
  tripCount: number
  photoCount: number
}
```

## Recap
```ts
Recap {
  id: string
  tripId: string
  title: string
  coverPhotoId?: string
  chapters: RecapChapter[]
  stats: {
    days: number
    distanceKm: number
    spotCount: number
    photoCount: number
    cityCount: number
  }
  theme: 'film' | 'magazine' | 'minimal'
}
RecapChapter {
  dayIndex: number
  date: string
  title: string
  photoIds: string[]
  activityNames: string[]
}
```

## Collage
```ts
CollageLayout {
  id: string
  name: string
  slots: { x: number; y: number; w: number; h: number }[]
  aspectRatio: number
}
Collage {
  id: string
  tripId?: string
  photoIds: string[]
  layoutId: string
  styleId: string
  background: string
  gap: number
  cornerRadius: number
  border: number
  exportedPath?: string
  exportedAt?: string
}
```

## 索引建议
- Photo(tripId), Photo(activityId), Photo(scene), Photo(takenAt)
- Activity(dayId)
- TransportLeg(fromId, toId, mode) 唯一索引，用于缓存
- Visit(countryCode), Visit(cityName)
