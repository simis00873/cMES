## [MES-APS-DC-0003] 有限產能排程 v1（EDD + CR 混合）

### Goal / KPI
- 到期日優先（**EDD**），同到期以 **Critical Ratio** 排序。
- 支援「一次排單廠」涵蓋**多工作中心**。
- 效能：**< 60s 完成 1k 作業**。

### Data Objects
| Entity | Fields |
|---|---|
| **WorkOrder** | `Id: long`, `Code: string`, `ItemId: long`, `Qty: decimal`, `DueOn: datetime`, `Priority: int` |
| **Operation** | `Id: long`, `WorkOrderId: long`, `Seq: int`, `WorkCenterId: long`, `StdMinutes: int`, `BatchSize: int?` |
| **ScheduleJob** | `Id: long`, `OperationId: long`, `StartOn: datetime`, `EndOn: datetime`, `WorkCenterId: long`, `ScenarioId: long` |
| **Scenario** | `Id: long`, `Code: string`, `Status: enum(Draft,Baseline,Released)`, `CreatedOn: datetime`, `ParentId: long?` |

### API Sketch
```http
POST /api/aps/schedule/run
Body: { "scenarioCode":"...", "rule":"EDD_CR", "horizonDays":7, "plantId":123, "includeMaterials":false }

GET  /api/aps/schedule/{scenarioId}/gantt
POST /api/aps/scenarios/{id}/baseline
POST /api/aps/scenarios/{id}/release
```

### Acceptance (UAT)
- 3 張工單（不同 `DueOn`）× 2 個工作中心 → 生成**不重疊**的 Gantt。
- 瓶頸工作中心（WC）作業連續擺放（最小換線、最大稼動）。

### NFR
- 可重入；**Idempotent**（同一輸入重跑結果一致）。
- 附大量測試資料種子（Seed）以便壓測與回歸。

---
