---

## [MES-APS-DC-0005] 排程版控與差異報告（Scenario → Baseline → Release）

### Goal / KPI
- 支援 Scenario 派生、Baseline 鎖版、Release 下發。
- 差異報告列出工單**起迄／工作中心**變動與 `MinutesDelta`。

### Data Objects
| Entity | Fields |
|---|---|
| **ScenarioDelta** | `Id: long`, `ScenarioId: long`, `WorkOrderId: long`, `Field: enum(Start, End, WorkCenter)`, `OldValue: string`, `NewValue: string`, `MinutesDelta: int` |

### API Sketch
```http
POST /api/aps/scenarios/{id}/baseline
POST /api/aps/scenarios/{id}/release
GET  /api/aps/scenarios/{id}/delta?compareTo=baseline
```

### Acceptance (UAT)
- 比較 Draft 與 Baseline → 列出 **3 筆差異**。
- 發布（Release）後**不可修改**（需另開新 Scenario）。

### NFR
- 權限：發布需**主管核可**。
- **回滾機制**與通知（Email／Line／SignalR）。
