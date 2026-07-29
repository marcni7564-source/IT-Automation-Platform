
# 2026-07-28 Request Design

## 今日目標

- 確認 Request 模組定位
- 確認 Request 欄位
- 確認 Request Status
- 明確劃分 Request 與 Workflow 的責任

---

# 今日結論

## 平台流程

```text
Request
    ↓
Workflow
    ↓
Notification
```

---

## Request 責任

Request 負責：

- 收集來源資料
- 判斷 RequestType
- 透過 Rule / AI / Parser 整理 Content
- 驗證資料完整性
- 建立標準化 Request

Request 不負責：

- Workflow 執行
- Notification
- Content 格式設計
- Mail Archive

---

## Request Status

第一階段僅保留：

```text
New
Manual
```

- New：可交由 Workflow 執行
- Manual：需人工介入（Rule 判斷失敗、資料不足、多個申請...）

---

## Request 欄位

|欄位|用途|主要使用者|原因|
|---|---|---|---|
|RequestId|Request 唯一識別碼|System|識別每一筆 Request|
|Source|記錄 Request 來源系統|MIS|方便人工回查原始資料來源|
|Subject|Request 主旨|MIS|方便辨識 Request，不限定 Mail，可依來源存放對應主旨（Mail 主旨、Web 申請名稱…）|
|RequestType|決定要執行哪個 Workflow|Workflow|Workflow 不需判斷來源，只依 RequestType 執行|
|Content|Workflow 所需資料|Workflow|Workflow 僅處理 Request 整理完成的資料|
|RequestStatus|Request 目前狀態|Workflow、MIS|判斷是否可交由 Workflow 執行或需人工介入|
|CreatedBy|建立者|MIS、Audit|保留建立紀錄|
|CreatedAt|建立時間|MIS、Audit|保留建立時間|
|UpdatedBy|最後修改者|MIS、Audit|保留最後修改人|
|UpdatedAt|最後修改時間|MIS、Audit|保留最後修改時間|

---

## Subject

Subject 定義為 **Request 主旨**。

依不同來源可對應不同內容，例如：

- Email：Mail 主旨
- Web：申請名稱
- API：Request 標題
- 其他來源：對應的主旨或標題

因此不限定為 Mail Subject。

---

## Source

Source 記錄 Request 的來源系統。

用途：

- MIS 人工查找原始資料
- 問題追查

Workflow 不依賴 Source 判斷。

---

## Content

Content 定義：

> Workflow 的輸入資料（Workflow Input）

目前僅確認：

- Request 負責產生 Content
- Workflow 使用 Content

Content 的格式與 Schema 留待 Workflow 設計時再決定。

---

## Created / Updated

Created：

- 建立後不再變更

Updated：

- 每次異動皆更新

---

# 尚未決定

- Content 格式（JSON / Schema）
- Workflow 如何取得 Request
- Workflow Status
- Retry 機制
- WorkflowInstance

---

# Future Backlog

- ADR
- Rule 分類
- Git Flow
- Coding Guideline
- Document Standard

---

# 下次建議討論

1. Workflow 設計
2. Database 設計
3. Notification 設計
