### Goal / KPI
- 換線時間依「前後產品／特徵」決定，查詢 **< 100ms**。
- 具備預設回退規則。

### Data Objects
| Entity | Fields |
|---|---|
| **ItemFamily** | `Id: long`, `Code: string`, `Name: string` |
| **SetupMatrix** | `Id: long`, `WorkCenterId: long`, `FromFamilyId: long`, `ToFamilyId: long`, `SetupMinutes: int` |

> 規則：若無精確對映 → 先落到 **Family 預設**，再落到 **WorkCenter 預設**（多層回退）。

### API Sketch
```http
PUT /api/aps/work-centers/{id}/setup-matrix         ; 批次上傳（CSV/JSON），含校驗
GET /api/aps/work-centers/{id}/setup-time?fromFamily=...&toFamily=...
```

### Acceptance (UAT)
- A → B 需 **12** 分；B → A 需 **8** 分；未定義則回傳預設 **5** 分。

### NFR
- 支援萬級資料量；上傳需校驗**重複**與**循環**（避免自循環或圖中負回圈）。

---
