## [MES-APS-DC-0001] 工作中心日曆（Calendar）建模與維護

### Goal / KPI
- 能定義工廠／工作中心工作時段與例外（停機／假日）。
- 可用時段查詢效能：**< 50ms/次**（以 1 個工作中心、1 週窗口為基準）。

### Data Objects
| Entity | Fields |
|---|---|
| **WorkCenter** | `Id: long`, `Code: string(16)`, `Name: string`, `PlantId: long` |
| **Calendar** | `Id: long`, `Name: string`, `TimeZone: string` |
| **CalendarRule** | `Id: long`, `CalendarId: long`, `Type: enum(Shift, Holiday, Exception)`, `ByDay: string` (e.g., `"MO,TU,WE,TH,FR"`), `StartTime: time`, `EndTime: time` |
| **WcCalendarMap** | `WorkCenterId: long`, `CalendarId: long` — **UQ**(`WorkCenterId`) |

### API Sketch
```http
POST /api/aps/calendars
POST /api/aps/work-centers/{id}/calendar/{calendarId}
GET  /api/aps/work-centers/{id}/availability?from=YYYY-MM-DDTHH:mm&to=YYYY-MM-DDTHH:mm
```

### Acceptance (UAT)
- 建立一個 **8×5** 班（週一至週五 08:00–17:00，含 1 小時休息或以規則表示）。
- 設定 **週六為假日**（Holiday 例外規則）。
- 查詢「下週可用時段」返回 **5 天 × 8 小時**的連續可用區間集合。

### NFR
- 多時區、夏令時間（DST）正確處理（以 `Calendar.TimeZone` 為準）。
- 審計與版本管理（Calendar／Rule 需版本化與追溯）。
- 權限控制：僅**排程工程師**可編輯。

---
