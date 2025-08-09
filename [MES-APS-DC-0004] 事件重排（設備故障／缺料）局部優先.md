## [MES-APS-DC-0004] 事件重排（設備故障／缺料）局部優先

### Goal / KPI
- 故障／缺料事件觸發**局部重排**（受影響 WC ±N 小時窗口）。
- 完成於 **< 15s**。

### Data Objects
| Entity | Fields |
|---|---|
| **ApsEvent** | `Id: long`, `Type: enum(Downtime, MaterialShortage, PriorityChange)`, `WorkCenterId: long?`, `AffectsFrom: datetime`, `AffectsTo: datetime`, `Payload: json` |

### API Sketch
```http
POST /api/aps/events                       ; 由 MES/SCM 投遞事件
POST /api/aps/schedule/replan
Body: { "scenarioId": 101, "eventId": 9001, "windowHours": 4 }
```

### Acceptance (UAT)
- 模擬 `WC-01` 故障 2 小時 → 受影響作業**右移**；跨 WC 的前後作業**保持順序**；未受影響者**不變**。

### NFR
- 事件**去重**與**節流**（Debounce／Throttling）。
- 審計可**回放**（Event Sourcing 友善）。

---
