# Request Design

> Status: Final Design  
> Version: V1  
> 本文件由 Repository 既有 Request 與相關 Workflow Discussion 收斂整理，作為後續 Coding 的正式依據。

---

## 1. 文件目的

本文件定義 IT Automation Platform V1 的 Request 定位、資料內容、狀態、處理模式、生命週期與責任邊界。

Request 的目的，是將不同來源的申請轉換為 Workflow 可統一處理的標準工作資料。

平台設計原則：

- Automation First
- Rule First
- AI Optional
- Request 與來源解耦
- Request 與 Workflow 保持低耦合
- 不提前設計尚未發生的需求

---

## 2. Request 定位

Request 是 IT Automation Platform 唯一的工作請求資料，也是 Workflow 的唯一工作來源。

所有外部來源都必須先經由 Collector 轉換為標準化 Request，再交由 Workflow 處理。

```text
External Source
    ↓
Collector
    ↓
Request
    ↓
Dispatcher
    ↓
Worker
```

Request 僅描述工作內容，不包含任何執行邏輯。

---

## 3. 核心原則

### 3.1 一個來源申請對應一個 Request

V1 原則：

- 一封 Email 對應一個 Request
- 一個 Request 對應一個申請事項
- 一個 Request 對應一個 Workflow
- 不將一封 Email 拆成多個 Request

原因是不同申請通常具有不同的主管、審核流程與負責人。

### 3.2 Request 不依賴來源

Request 可以由不同 Collector 建立，例如：

- Email
- Web
- API
- 其他未來來源

Workflow 不依賴 Source 判斷執行流程。

Workflow 僅依據：

- RequestType
- Content

執行對應工作。

### 3.3 Request 不包含執行邏輯

Request 不負責：

- 執行 Workflow
- 呼叫外部 API
- 執行 Database 操作流程
- 發送 Notification
- 保存 Mail Archive

Request 只保存 Workflow 執行所需的標準化資料與執行結果。

---

## 4. Collector 與 Request 的責任邊界

Collector 負責：

- 收集外部來源資料
- 解析內容
- 驗證資料
- 加工資料
- 判斷 RequestType
- 決定 ProcessingMode
- 產生 Content
- 建立 Request

Collector 不執行業務流程。

Rule、Parser 或 AI 都可以參與資料解析，但 AI 不是平台核心。只要能產生符合規格的標準化 Request，即可作為 Collector 的一部分。

Request 本身是標準化工作資料，不負責執行上述處理。

---

## 5. Request 欄位

| 欄位 | 說明 |
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

本文件只定義欄位用途。

欄位的實體 Database 型別、長度、Null 規則與索引，應以 `docs/05-Database-Design.md` 為準。

---

## 6. 欄位定義

### 6.1 RequestId

Request 的唯一識別碼。

用於：

- 識別每一筆 Request
- Workflow 執行追蹤
- Log 關聯
- MIS 問題查詢

RequestId 的實體資料型別不在本文件定義。

### 6.2 Source

Source 記錄 Request 的來源系統。

用途：

- MIS 回查原始資料
- 問題追蹤
- 區分 Request 建立來源

可能來源包括：

- Email
- Web
- API
- 其他 Collector

Workflow 不得依賴 Source 決定執行流程。

### 6.3 Subject

Subject 定義為 Request 主旨，不限定為 Email Subject。

依來源可對應不同內容，例如：

- Email：Mail 主旨
- Web：申請名稱
- API：Request 標題
- 其他來源：對應的主旨或標題

Subject 主要提供 MIS 辨識與查詢 Request。

### 6.4 RequestType

RequestType 表示 Request 的工作類型，並用於決定對應 Worker。

範例：

- VPN
- Meeting
- LargeFile

V1 的 RequestType 由程式定義。

RequestType 不由 Workflow 根據 Source 再次判斷；Collector 建立 Request 前必須完成 RequestType 判斷。

### 6.5 Content

Content 是 Workflow 的輸入資料。

規則：

- Collector 完成解析、驗證與加工後產生 Content
- Content 使用 JSON
- 不同 RequestType 可以有不同的 Content 結構
- Workflow 依 RequestType 解析 Content
- Workflow 不修改 Content
- 新增 RequestType 時，不因專屬欄位而持續修改 Request 主資料結構
- V1 不建立 Content Version

範例：

```json
{
  "username": "demo",
  "group": "SSLVPN"
}
```

各 RequestType 的 Content 必須包含該 Workflow 執行所需的完整資料。

個別 RequestType 的 Content Schema 不在本文件中定義，應於該 Workflow 的 Design 中確認。

### 6.6 ProcessingMode

ProcessingMode 用於區分 Request 是否由系統自動執行。

| ProcessingMode | 說明 |
|---|---|
| Automatic | 系統自動執行 |
| Manual | 人工處理 |

`Manual` 是 ProcessingMode，不是 Status。

當 Collector 因資料不足、無法判斷或多重需求而無法建立可自動執行的 Request 時，應使用 Manual ProcessingMode，交由 MIS 人工處理。

Dispatcher 不自動派送 Manual Request。

### 6.7 Status

Status 是 Request 與 Workflow 共用的工作狀態。

V1 不建立獨立的 Workflow Status。

| Status | 說明 |
|---|---|
| New | 等待處理 |
| Processing | 執行中 |
| Completed | 已完成 |
| Failed | 執行失敗 |
| Cancelled | 已取消 |

`Manual` 不屬於 Status。

### 6.8 Remark

Remark 提供一般使用者閱讀。

內容必須：

- 避免技術細節
- 使用可理解的結果說明
- 不包含 Exception StackTrace

範例：

- 檔案已成功上傳。
- VPN 帳號已建立。
- 會議已建立完成。
- 您的申請已完成。
- 系統忙碌，請稍後再試。

### 6.9 AdminRemark

AdminRemark 提供 MIS 或系統管理員進行維運與診斷。

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

AdminRemark：

- 不提供一般使用者閱讀
- 不存放完整 StackTrace

完整 Exception 與 StackTrace 僅寫入 Log。

### 6.10 CreatedBy / CreatedAt

CreatedBy 與 CreatedAt 記錄 Request 的建立者與建立時間。

建立後不再變更。

用途：

- MIS 查詢
- Audit
- 問題追蹤

### 6.11 UpdatedBy / UpdatedAt

UpdatedBy 與 UpdatedAt 記錄 Request 最後一次異動者與異動時間。

每次 Request 狀態或結果異動時都必須更新。

---

## 7. Request 建立規則

Collector 建立 Request 前，必須完成：

1. 收集來源資料
2. 解析來源內容
3. 驗證必要資料
4. 加工 Workflow 所需資料
5. 判斷 RequestType
6. 決定 ProcessingMode
7. 產生完整 Content
8. 建立 Request

Automatic Request 必須具備可由對應 Worker 執行的完整 Content。

無法自動執行的 Request 使用 Manual ProcessingMode，交由 MIS 人工處理。

---

## 8. Automatic Request 生命週期

Automatic Request 的 Workflow 主要狀態流程：

```text
New
    ↓ Dispatcher
Processing
    ↓ Worker
Completed / Failed
```

### New

表示 Request 等待 Dispatcher 派工。

### Processing

Dispatcher 取得 Request 後，先更新：

```text
Status = Processing
```

再將完整 Request 傳入對應 Worker。

### Completed

Worker 正常完成 Request 時更新：

- Status = Completed
- Remark
- AdminRemark
- UpdatedBy
- UpdatedAt

### Failed

Worker 無法完成 Request 時更新：

- Status = Failed
- Remark
- AdminRemark
- UpdatedBy
- UpdatedAt

V1 不區分 Business Failure 與 System Exception，只要 Worker 無法完成 Request，Status 均為 Failed。

需要人工確認、人工補件或人工操作，但原本屬於 Automatic Request 且 Worker 無法完成時，也使用 Failed，不新增 Manual Status。

### Cancelled

Cancelled 表示 Request 已取消。

Cancelled Request 不進入自動 Workflow 執行流程。

取消的操作時機與權限尚未在 Repository 中完成進一步設計，本文件不自行新增規格。

---

## 9. Request 更新責任

### Collector

建立 Request 初始資料，包括：

- Source
- Subject
- RequestType
- Content
- ProcessingMode
- Status
- CreatedBy
- CreatedAt
- UpdatedBy
- UpdatedAt

### Dispatcher

Dispatcher 負責：

- 查詢可自動執行的 New Request
- 將 Request 更新為 Processing
- 更新 UpdatedBy
- 更新 UpdatedAt
- 將完整 Request 傳入 Worker

Dispatcher 不更新 Completed 或 Failed 的業務結果。

### Worker

Worker 完成或失敗後負責更新：

- Status
- Remark
- AdminRemark
- UpdatedBy
- UpdatedAt

Worker 不修改：

- Source
- Subject
- RequestType
- Content
- ProcessingMode
- CreatedBy
- CreatedAt

### MIS

Manual Request 與需要人工介入的 Request，由 MIS 依實際作業處理。

MIS 的操作介面、權限與人工狀態轉換流程尚未在 Repository 中完成正式設計。

---

## 10. Request 與 Workflow 的責任邊界

Request：

- 保存標準化工作資料
- 保存 ProcessingMode
- 保存 Workflow 共用 Status
- 保存執行結果摘要
- 作為 Workflow 唯一工作來源

Dispatcher：

- 查詢可執行 Request
- 更新 Processing
- 依 RequestType 分派 Worker

Worker：

- 解析 Content
- 執行自動化工作
- 更新 Status、Remark 與 AdminRemark

Workflow 不修改 Request.Content。

---

## 11. Request 與 Notification

Notification 與 Workflow 分離。

V1：

- Notification 模組保留於整體架構
- 不實作自動 Notification
- Worker 更新 Request 結果後，Workflow 即結束
- MIS 依 Status、Remark 與 AdminRemark 人工確認
- 需要通知使用者時，由 MIS 人工處理

因此 V1 Automatic Request 的自動生命週期終點為：

```text
Completed / Failed
```

Notification 的觸發方式、收件者、Template、Retry 與 Notification Log 留待後續版本另行 Discussion。

---

## 12. 資訊分層

Request 執行資訊分為四個層級：

| 資訊 | 用途 |
|---|---|
| Status | Workflow 執行狀態 |
| Remark | 一般使用者可閱讀的結果 |
| AdminRemark | MIS 維運與診斷摘要 |
| Log | 完整執行過程、Exception 與 StackTrace |

規則：

- Remark 不包含技術細節
- AdminRemark 不包含 StackTrace
- Exception 與 StackTrace 僅寫入 Log

---

## 13. V1 不納入項目

Request V1 不提前實作：

- 一封 Email 拆成多個 Request
- 一個 Request 對應多個 Workflow
- Content Version
- WorkflowInstance
- 獨立 Workflow Status
- Manual Status
- Retry
- Timeout Recovery
- Heartbeat
- Processing 自動復原
- 多 Step Workflow
- Distributed Worker
- 自動 Notification

上述項目待實際需求出現後，再經 Discussion → Design 流程確認。

---

## 14. 尚未定義的項目

以下內容尚未在 Repository Discussion 中完成正式決議，因此不屬於本版規格：

- RequestId 實體資料型別
- 各欄位 Database 型別與長度
- Null / Not Null
- Index
- Concurrency Control
- Cancelled 的操作時機與權限
- Manual Request 的人工操作介面
- RequestType 的完整清單
- 各 RequestType 的 Content Schema
- Mail Archive
- 原始來源資料保存方式
- 資料保留與清除方式

上述項目需另行 Discussion，確認後再更新正式 Design。

---

## 15. Design 來源

本文件收斂自：

- `ReadMe-For-AI.md`
- `docs/01-Why-IT-Automation-Platform.md`
- `docs/Discussion/06-Decisions-2026-07-28.md`
- `docs/Discussion/2026-07-28-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Request-Decision-Log.md`
- `docs/Discussion/2026-07-31-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Notification-Decision-Log.md`
- `docs/Discussion/2026-07-31-Workflow-Common-Capability-Log.md`
