# [SYS-xxx] <System 名稱>（例：MES）
## 系統目標（OKR）
- O：在 90 天內交付可用的 NPI→BOM/BOP→工單→報工 閉環
- KR：上線 3 家中小型工廠；報工 < 30s；缺料預警 > 95%

## 目標市場
- 產業/規模/地域/合規：<從 Portfolio Segments 關聯帶入>

## 目標客戶（ICP）
- 主要 Persona：生管/線長/製程工程師/品保/維護/財務
- 導入條件：IT 1 人即可維運；能接受雲端或私有

## 目標用法（主路徑）
- Hero Flows（3–5 條）：例：① 工單釋放→派工→報工→繳庫；② ECN 發佈→影響報告→鎖版
- 部署模式：SaaS｜On-Prem｜Edge
- 整合面：OpenAPI、事件（Outbox）、OPC-UA（SCADA）

## 邊界 & 非目標
- 不做：APS 最佳化排程（先對接第三方）、財會總帳（由 ERP）

## 關聯
- Portfolio：<PF-xxx>
- Modules：<清單>
- 路線圖：Alpha→Beta→GA Gate（準入條件）
