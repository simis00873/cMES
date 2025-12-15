# Engineering Scenarios（BDD/SDD 草案）

> 模組 `Engineering` · 來源 `src/Simis.Core/MES/Engineering`

## 主要欄位

- （本文件為 Scenario / 邊界定義，無固定資料欄位）

## 使用方式與約定

### 目的
- 作為 BDD：可直接轉成 `.feature`（Gherkin）並連到自動化測試/驗收。
- 作為 SDD：補齊「系統預期行為、例外邊界、狀態/版本策略、跨模組契約」。

### 名詞（建議統一）
- 製程群組：`ProcessGroup`
- 製程：`ProcessVariant`
- 途程：`ProcessRouting`
- BOM：`Bom`
- BOR：`Bor`
- 製程參數模板：`ProcessParameter`
- 產品製程參數：`ProductProcessParameter`
- 配方：`ProductionRecipe`
- 技術文件：`TechnicalDocument`

### 共用測試資料（建議）
以下為 scenario 中常用的假資料命名（可依你們的資料字典調整）：
- 料品（Material）：`P-100`（產品）、`RM-001`（原料）
- 單位（Unit）：`PCS`、`KG`
- 製程群組：`PG-SMT`（SMT）、`PG-PACK`（包裝）
- 製程：`SMT`、`AOI`、`PACK`
- 技術文件：`SOP-SMT-001`（SOP）、`SIP-AOI-001`（SIP）
- 日期：以 `2025-01-01` 為範例

## Feature：ProcessGroup / ProcessVariant（製程字典）

> 對應文件：`docs/mes/Engineering/ProcessGroup.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/ProcessGroup/ProcessGroupService.cs`

```gherkin
Feature: ProcessGroup 維護

  @engineering @processgroup
  Scenario: ENG-PG-001 建立製程群組（含多個製程）
    Given 使用者輸入 ProcessGroup Code="PG-SMT", Name="SMT群組", IsActive=true
    And 製程清單包含:
      | Code | Name | IsInspectionExempt | IsSchedulingIncluded | IsActive |
      | SMT  | SMT  | false              | true                 | true     |
      | AOI  | AOI  | true               | true                 | true     |
    When 呼叫 ProcessGroupService.Create
    Then 應成功建立 ProcessGroup 與 ProcessVariants
    And ProcessGroup.IsInspectionExempt 應為 false（只要任一製程非免驗）

  @engineering @processgroup
  Scenario: ENG-PG-002 製程群組編號重複
    Given 系統已存在 ProcessGroup Code="PG-SMT"
    When 再次以 Code="PG-SMT" 建立
    Then 應回傳錯誤 "編號重複"

  @engineering @processgroup
  Scenario: ENG-PG-003 同群組下製程名稱重複
    Given 使用者輸入同一群組內兩個 ProcessVariant Name 都為 "SMT"
    When 呼叫 ProcessGroupService.Create
    Then 應回傳錯誤 "生產製程名稱重複"
```

### 邊界與備註（SDD）
- 目前程式只檢核 `ProcessGroup.Code` 唯一、以及「同群組內 `ProcessVariant.Name` 唯一」；是否需要 `ProcessVariant.Code` 全域唯一需在 SDD 明確定義。
- `IsInspectionExempt` 由群組下製程推導（只要任一製程 `IsInspectionExempt=false`，群組即為 false）。

## Feature：ProcessRouting（產品途程）

> 對應文件：`docs/mes/Engineering/ProcessRouting.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/ProcessRoutings/ProcessRoutingService.cs`

```gherkin
Feature: ProcessRouting 維護

  @engineering @routing
  Scenario: ENG-ROUT-001 建立途程（適用多料品 + 有序明細）
    Given 使用者輸入 ProcessRouting Code="R-P100-01", IsActive=true, ActiveDate="2025-01-01"
    And 適用產品包含 MaterialIds=[P-100]
    And 明細包含:
      | Stage | ProcessVariant | Type |
      | 10    | SMT            | 生產 |
      | 20    | AOI            | 檢驗 |
      | 30    | PACK           | 生產 |
    When 呼叫 ProcessRoutingService.Create
    Then 應成功建立途程與明細

  @engineering @routing
  Scenario: ENG-ROUT-002 明細類型非法
    Given 明細 Type="返工"
    When 呼叫 ProcessRoutingService.Create
    Then 應回傳錯誤 "明細類型需為[生產]或[檢驗]"

  @engineering @routing
  Scenario: ENG-ROUT-003 Stage 不可為負值
    Given 明細 Stage=-1
    When 呼叫 ProcessRoutingService.Create
    Then 應回傳錯誤 "階段不可為負值"
```

### 邊界與備註（SDD）
- `ProcessRouting` 目前同一筆可涵蓋多產品，且缺少版本/期間/優先序，消費端容易出現「同產品多筆時挑第一筆」；建議在 SDD 定義唯一選用規則。
- `ProcessRouting` 既有 `Links`（圖狀流程欄位），但 Service/消費端仍以 `Stage` 作線性流程；需在 SDD 決定採線性或圖狀，避免雙軌。

## Feature：BOM（產品結構）

> 對應文件：`docs/mes/Engineering/Bom.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/BOMs/BomService.cs`

```gherkin
Feature: BOM 維護

  @engineering @bom
  Scenario: ENG-BOM-001 建立 BOM（Code 唯一 + UniqueKey 唯一）
    Given 已存在產品料品 Material Code="P-100"
    And 使用者輸入 Bom Code="BOM-0001", Category="製造", Version=1.0
    When 呼叫 BomService.Create
    Then 應成功建立 BOM
    And UniqueKey 應為 "P-100-1.0-製造"

  @engineering @bom
  Scenario: ENG-BOM-002 BOM 編號重複
    Given 系統已存在 Bom Code="BOM-0001"
    When 再次建立 Bom Code="BOM-0001"
    Then 應回傳錯誤 "Bom編號重複"

  @engineering @bom
  Scenario: ENG-BOM-003 同產品+版本+類別重複（UniqueKey）
    Given 系統已存在 UniqueKey="P-100-1.0-製造"
    When 再次建立同組合 BOM
    Then 應回傳錯誤 "Bom已存在 P-100-1.0-製造"

  @engineering @bom
  Scenario: ENG-BOM-004 BOM 類別非法
    Given Category="其他"
    When 呼叫 BomService.Create
    Then 應回傳錯誤 "Bom類別錯誤"

  @engineering @bom
  Scenario: ENG-BOM-005 新增/更新 BOM 明細（Level 由 Parent 推導）
    Given 已存在 Bom Id=1
    And 使用者輸入 BomLine Material="RM-001", Unit="KG", Quantity=2
    When 呼叫 BomService.UpdateDetail
    Then 應成功寫入 BomLine
    And 若 ParentId 為空，Level 應為 1

  @engineering @bom
  Scenario: ENG-BOM-006 明細用量不可為負值
    Given Quantity=-1
    When 呼叫 BomService.UpdateDetail
    Then 應回傳錯誤 "用量不可為負值"

  @engineering @bom
  Scenario: ENG-BOM-007 指定 ParentId 但父層不存在
    Given ParentId=999 且該父層不存在於此 BOM
    When 呼叫 BomService.UpdateDetail
    Then 應回傳錯誤 "父層不存在"

  @engineering @bom
  Scenario: ENG-BOM-008 刪除 BOM 明細（含子階遞迴刪除）
    Given BomLine A 有子階 BomLine B, C
    When 呼叫 BomService.DeleteLine 刪除 A
    Then A, B, C 都應被刪除
```

### 邊界與備註（SDD）
- `Version` 四捨五入精度在 Create/Update 不一致（Create: 1 位；Update: 2 位），可能造成 UniqueKey 與版本語意漂移；建議在 SDD 統一版本策略。
- `BomLine` 同時用 `ParentId` 與 `Level` 表達階層，且目前不允許改 Parent（以避免 Level 問題）；若階層變更是需求，需補齊重算與規則。

## Feature：BOR（資源途程 / 用時用量）

> 對應文件：`docs/mes/Engineering/Bor.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/BORs/BorService.cs`

```gherkin
Feature: BOR 維護

  @engineering @bor
  Scenario: ENG-BOR-001 建立 BOR（產品 + 版本 + 生效期間）
    Given 已存在產品料品 ProductId=P-100
    And 使用者輸入 Version="A01", EffectiveDate="2025-01-01", ExpiredDate="2025-12-31"
    And 初始 Lines 包含:
      | Sequence | ResourceId | UsageTime | UsageUnitId | Quantity |
      | 10       | 1001       | 5.0       | MIN         | 1.0      |
    When 呼叫 BorService.Create
    Then 應成功建立 BOR
    And State 預設為 "草稿"

  @engineering @bor
  Scenario: ENG-BOR-002 版本不可為空
    Given Version="" 或只含空白
    When 呼叫 BorService.Create
    Then 應回傳錯誤 "版本號不可為空"

  @engineering @bor
  Scenario: ENG-BOR-003 失效日期不可早於生效日期
    Given ExpiredDate < EffectiveDate
    When 呼叫 BorService.Create
    Then 應回傳錯誤 "失效日期不可早於生效日期"

  @engineering @bor
  Scenario: ENG-BOR-004 明細順序必須大於 0
    Given Line.Sequence=0
    When 呼叫 BorService.Create
    Then 應回傳錯誤 "明細順序 (Sequence) 必須大於 0"
```

### 邊界與備註（SDD）
- 「同產品+版本」有效期間不可重疊的檢核目前被註解；若 BOR 會被排程/工時計算消費，建議優先落地（含 DB 索引/唯一性策略）。
- Create 對 Lines 有完整檢核，但 Update 對 Lines 缺少同等檢核；若 Update 允許改明細，建議補齊一致性。

## Feature：ProcessParameter（製程參數模板）

> 對應文件：`docs/mes/Engineering/ProcessParameter.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/ProcessParameters/ProcessParameterService.cs`

```gherkin
Feature: ProcessParameter 維護

  @engineering @processparameter
  Scenario: ENG-PP-001 建立製程參數（明細去重、上下限與異常處理）
    Given 使用者輸入 ProcessParameter Code="PP-SMT-01", ProcessVariantId=SMT
    And 明細包含:
      | GroupName | Name   | Category | DefaultValue | MinValue | MaxValue | ExceptionHandlings |
      | 溫度      | 預熱溫度 | 定量     | 120          | 100      | 140      | 重工|再次確認      |
    When 呼叫 ProcessParameterService.Create
    Then 應成功建立 ProcessParameter 與明細

  @engineering @processparameter
  Scenario: ENG-PP-002 同組別同名稱重複
    Given 明細中出現重複 (GroupName, Name)
    When 呼叫 ProcessParameterService.Create
    Then 應回傳錯誤 "{參數組別、參數名稱}重複"

  @engineering @processparameter
  Scenario: ENG-PP-003 類型只允許 定性/定量
    Given Category="布林"
    When 呼叫 ProcessParameterService.Create
    Then 應回傳錯誤 "類型錯誤"

  @engineering @processparameter
  Scenario: ENG-PP-004 下限不得大於上限
    Given MinValue=10, MaxValue=5
    When 呼叫 ProcessParameterService.Create
    Then 應回傳錯誤 "下限不能大於上限"
```

### 邊界與備註（SDD）
- 異常處理值域目前以常數字串控制；若後續要國際化或擴充類型，建議收斂為 enum/值物件並集中管理。

## Feature：ProductProcessParameter（產品×製程×參數落地）

> 對應文件：`docs/mes/Engineering/ProductProcessParameter.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/ProductProcessParameters/ProductProcessParameterService.cs`

```gherkin
Feature: ProductProcessParameter 維護

  @engineering @ppp
  Scenario: ENG-PPP-001 建立產品製程參數（含技術文件限制）
    Given 使用者輸入 Code="PPP-P100-01", Version=1.0
    And 適用料品包含 [P-100]
    And 明細包含 ProcessVariantId=SMT, ProcessParameterId=PP-SMT-01
    And 明細項目包含 TechnicalDocumentId 指向「共用」且製程匹配的技術文件
    When 呼叫 ProductProcessParameterService.Create
    Then 應成功建立 ProductProcessParameter

  @engineering @ppp
  Scenario: ENG-PPP-002 適用產品清單不可為空
    Given MaterialIdList 為空
    When 呼叫 ProductProcessParameterService.Create
    Then 應回傳錯誤 "是用產品清單不可為空"

  @engineering @ppp
  Scenario: ENG-PPP-003 製程與製程參數不匹配
    Given ProcessParameter.ProcessVariantId != 明細 ProcessVariantId
    When 呼叫 ProductProcessParameterService.Create
    Then 應回傳錯誤 "製程與製程參數不匹配"

  @engineering @ppp
  Scenario: ENG-PPP-004 不允許包含非共用技術文件
    Given TechnicalDocument.MaterialId 有值（非共用）
    When 呼叫 ProductProcessParameterService.Create
    Then 應回傳錯誤 "包含非共用[測試手法(技術文件)]"
```

### 邊界與備註（SDD）
- `ProcessParameterDetail` 與 `ProductProcessParameterDetailItem` 欄位高度重疊；若要避免資料雙寫，需在 SDD 決定「快照」或「引用+覆蓋」策略。

## Feature：ProductionRecipe（配方）

> 對應文件：`docs/mes/Engineering/ProductionRecipe.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/ProductionRecipes/ProductionRecipeService.cs`

```gherkin
Feature: ProductionRecipe 維護

  @engineering @recipe
  Scenario: ENG-RECIPE-001 建立配方（含步序、比例與規格）
    Given 使用者輸入 Code="RCP-P100-01", Version=1.0
    And 適用料品包含 [P-100]
    And Steps 包含 StepNumber=10，Details 含 RM-001 Ratio=60、RM-002 Ratio=40
    And Specs 包含 "黏度"=1.234 (Unit=...)
    When 呼叫 ProductionRecipeService.Create
    Then 應成功建立 ProductionRecipe

  @engineering @recipe
  Scenario: ENG-RECIPE-002 比例不可為負值
    Given 任一 Detail.Ratio < 0
    When 呼叫 ProductionRecipeService.Create
    Then 應回傳錯誤 "比例不可為負值"

  @engineering @recipe
  Scenario: ENG-RECIPE-003 產品製程參數三條件需同填或同不填
    Given ProductProcessParameterCode 有值
    And ProcessVariantId 或 ProcessParameterId 其中一個為空
    When 呼叫 ProductionRecipeService.Create
    Then 應回傳錯誤 "[產品製程參數編號]、[製程]、[製程參數]只允許皆填或皆不填"
```

### 邊界與備註（SDD）
- 目前未強制「比例總和=100」；若業務需要，需明確寫入規格並加上驗證。
- `ProductionRecipeStep` 的明細集合在程式為 public field，需確認 EF mapping 與 Include 行為一致性（避免執行期落差）。

## Feature：TechnicalDocument（技術文件與審核/生效）

> 對應文件：`docs/mes/Engineering/TechnicalDocument.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/TechnicalDocuments/TechnicalDocumentService.cs`

```gherkin
Feature: TechnicalDocument 生命週期

  @engineering @techdoc
  Scenario: ENG-TD-001 建立技術文件（草稿）
    Given 使用者輸入 Code="SOP-SMT-001", Type="SOP", Version=1.0
    And EffectiveDate="2025-01-01", ExpirationDate="9999-12-31"
    When 呼叫 TechnicalDocumentService.Create
    Then ReviewStatus 應為 "草稿"
    And ValidityStatus 應為 "修訂中"

  @engineering @techdoc
  Scenario: ENG-TD-002 送審（草稿 → 送審）
    Given 技術文件 ReviewStatus="草稿"
    When 呼叫 TechnicalDocumentService.SubmitForReview
    Then ReviewStatus 應為 "送審"

  @engineering @techdoc
  Scenario: ENG-TD-003 核可（送審 → 核可）
    Given 技術文件 ReviewStatus="送審"
    When 呼叫 TechnicalDocumentService.Review
    Then ReviewStatus 應為 "核可"
    And ValidityStatus 應依生效/失效日期與今天計算

  @engineering @techdoc
  Scenario: ENG-TD-004 非草稿不可修改/刪除
    Given 技術文件 ReviewStatus != "草稿"
    When 呼叫 TechnicalDocumentService.Update 或 Delete
    Then 應回傳錯誤 "已送審，無法修改" 或 "已送審，無法刪除"
```

### 邊界與備註（SDD）
- `CancelReview`（退審）會檢查是否已被 QC 的檢驗項目使用；若已使用則拒絕退審（避免追溯斷裂）。
- 有效性狀態計算目前存在分支邏輯風險（例如 today < EffectiveDate 時可能落入例外）；若要做排程/稼動的「依日期取生效版本」，建議先修正並補測試。

## Feature：FeedingMap（投料/上料位對應，待落地）

> 對應文件：`docs/mes/Engineering/FeedingMap.md`  
> 主要程式：`src/Simis.Core/MES/Engineering/FeedingMap/FeedingMap.cs`

```gherkin
Feature: FeedingMap（待落地）

  @engineering @feedingmap @future
  Scenario: ENG-FM-001 建立投料對照（WorkCenter + 產品 + 製程 + 版本）
    Given WorkCenterId=LINE1, ProductId=P-100, ProcessVariantId=SMT, Version="V1"
    And Points 包含多個投料點（料品、位置、是否必投）
    When 建立 FeedingMap
    Then 應能用 (WorkCenterId, ProductId, ProcessVariantId, Version) 唯一識別

  @engineering @feedingmap @future
  Scenario: ENG-FM-002 PositionCode 逐步淘汰
    Given 舊資料仍有 PositionCode
    When 新版維護改用 PositionId(Location)
    Then 系統仍可讀舊資料，但寫入只存 PositionId
```

## 跨模組邊界（Engineering ↔ PMC/QC）

- Engineering 提供「可被選用的標準」：Routing/BOM/BOR/參數/配方/文件；PMC/QC 不應自行用「挑第一筆」決策。
- 建議為「標準選用」定義可測試的 Policy（依產品、日期、版本/核可狀態），並把 scenario 轉成跨模組的 BDD（例如：PMC 匯入工單時如何選 Routing、如何驗證 WorkCenter 能力）。

