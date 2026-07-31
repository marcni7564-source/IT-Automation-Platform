# Database Design

> Status: Final Design  
> Version: V1  
> 本文件定義目前 Repository 已確認的 Database 邏輯設計。未經 Discussion 決議的實體型別、長度、索引與資料保留規則不在本文件中自行新增。

---

## 1. Database 原則

- 所有 Database 存取都必須透過 Repository
- Worker 不可直接執行 SQL
- Dispatcher 使用 RequestRepository 查詢與更新 Request
- Request 是平台唯一的工作來源
- 新增 RequestType 時，不因專屬欄位而持續修改 Request 主表
- RequestType 專屬資料統一存放於 Content
- Table 使用單數英文名詞
- Column 使用 PascalCase

---

## 2. Request Table

Table 名稱：

```text
Request
```

### 2.1 欄位

| 欄位 | 用途 |
|---|---|
| RequestId | Request 唯一識別碼 |
| Source | Request 來源系統 |
| Subject | Request 主旨 |
| RequestType | 工作類型 |
| Content | Workflow 輸入資料 |
| ProcessingMode | Automatic 或 Manual |
| Status | Workflow 共用工作狀態 |
| Remark | 提供一般使用者閱讀的備註 |
| AdminRemark | 提供 MIS 維運與診斷的管理備註 |
| CreatedBy | 建立者 |
| CreatedAt | 建立時間 |
| UpdatedBy | 最後更新者 |
| UpdatedAt | 最後更新時間 |

---

## 3. 欄位定義

### 3.1 RequestId

Request 的唯一識別碼。

Repository 尚未決定 RequestId 的實體資料型別，本文件不指定 int、bigint 或 GUID。

### 3.2 Source

記錄 Request 的來源系統，用於 MIS 回查原始資料與問題追蹤。

可能來源包括：

- Email
- Web
- API
- 其他 Collector

Workflow 不得依賴 Source 判斷執行流程。

### 3.3 Subject

Subject 是 Request 主旨，不限定為 Email Subject。

依來源可對應：

- Email：Mail 主旨
- Web：申請名稱
- API：Request 標題
- 其他來源：對應主旨或標題

### 3.4 RequestType

RequestType 用於決定交由哪一個 Worker 處理。

範例：

- VPN
- Meeting
- LargeFile

V1 的 RequestType 由程式定義。

### 3.5 Content

Content 是 Workflow 的輸入資料。

規則：

- Collector 解析、驗證並加工來源資料後產生 Content
- 不同 RequestType 使用各自的資料結構
- Workflow 依 RequestType 解析 Content
- Workflow 不修改 Content
- Content 使用 JSON 表達
- V1 不建立 Content Version

範例：

```json
{
  "username": "demo",
  "group": "SSLVPN"
}
```

Repository 尚未決定 Content 對應的實體 Database 型別與長度，本文件不自行指定。

### 3.6 ProcessingMode

| 值 | 說明 |
|---|---|
| Automatic | 系統自動執行 |
| Manual | 人工處理 |

Manual 是 ProcessingMode，不是 Status。

### 3.7 Status

| 值 | 說明 |
|---|---|
| New | 等待處理 |
| Processing | 執行中 |
| Completed | 已完成 |
| Failed | 執行失敗 |
| Cancelled | 已取消 |

Status 是 Request 與 Workflow 共用的工作狀態，不建立獨立 Workflow Status。

### 3.8 Remark

Remark 提供一般使用者閱讀。

內容應避免技術細節。

範例：

- 檔案已成功上傳。
- VPN 帳號已建立。
- 會議已建立完成。
- 您的申請已完成。
- 系統忙碌，請稍後再試。

Remark 不得包含 Exception StackTrace。

### 3.9 AdminRemark

AdminRemark 提供 MIS 或系統管理員查看。

可記錄：

- API 回傳訊息摘要
- Exception 摘要
- 系統診斷資訊
- 其他維運資訊

範例：

- SharePoint API Timeout
- SMTP Send Failed
- User Not Found
- SQL Timeout
- Return Code = -61

AdminRemark 不提供一般使用者閱讀，也不得存放 StackTrace。

### 3.10 CreatedBy / CreatedAt

- Request 建立時寫入
- 建立後不再變更
- 用於 MIS 與 Audit 追蹤

### 3.11 UpdatedBy / UpdatedAt

- Request 每次異動時更新
- Dispatcher 更新 Processing 時必須更新
- Worker 更新 Completed 或 Failed 與 Remark、AdminRemark 時必須更新
- MIS 人工異動時必須更新

---

## 4. Request 建立規則

Collector 建立 Request 時：

- 一封 Email 建立一筆 Request
- 一個 Request 只代表一個申請事項
- 一個 Request 對應一個 Workflow
- Request 不包含執行邏輯
- Content 必須先完成解析、驗證與加工
- ProcessingMode 決定自動或人工處理
- Automatic Request 的初始 Status 為 New
- Manual Request 保留給 MIS 人工處理

多重需求、資料不足或無法判斷的來源資料，由 Collector 建立為 Manual ProcessingMode，不交由 Dispatcher 自動執行。

---

## 5. 欄位更新責任

| 元件 | 可更新內容 |
|---|---|
| Collector | 建立完整 Request 初始資料 |
| Dispatcher | Status = Processing、UpdatedBy、UpdatedAt |
| Worker | Status、Remark、AdminRemark、UpdatedBy、UpdatedAt |
| MIS | 人工處理所需的 Request 欄位 |

Workflow 不得修改 Content。

---

## 6. Log 與 Request 的資料分層

| 資料 | 保存內容 |
|---|---|
| Status | Workflow 執行狀態 |
| Remark | 一般使用者可閱讀的結果 |
| AdminRemark | MIS 診斷摘要 |
| Log | 完整技術執行過程、Exception、StackTrace |

目前 Repository 未正式決定 Log 使用 Database Table 或檔案保存，也未決定保存天數，因此本文件不建立 Log Table。

---

## 7. Repository

命名規則：

```text
{Entity}Repository
```

Request 使用：

```text
RequestRepository
```

規則：

- Dispatcher 透過 RequestRepository 查詢與更新 Request
- Worker 透過 RequestRepository 更新 Request 執行結果
- 其他 Entity 使用各自對應 Repository
- Worker 與 Dispatcher 不直接執行 SQL

---

## 8. 尚未定義的實體設計

以下內容尚未在 Repository Discussion 中完成決議，因此不屬於本版正式規格：

- RequestId 資料型別
- 各欄位實體資料型別
- 字串長度
- Null / Not Null
- Default Value
- Primary Key 實作
- Index
- Concurrency Control
- RowVersion
- Foreign Key
- Check Constraint
- Log Table
- Configuration Table
- 資料保留與清除規則
- Migration 或 SQL Script 管理方式

上述項目需另行 Discussion，確認後再更新本文件。

---

## 9. Design 來源

本文件收斂自：

- `docs/Discussion/2026-07-28-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Request-Decision-Log.md`
- `docs/Discussion/2026-07-31-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-31-Workflow-Common-Capability-Log.md`
