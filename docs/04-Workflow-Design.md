# Workflow Design

> Status: Final Design  
> Version: V1  
> 本文件定義 Workflow V1 的正式執行流程與責任分工。

---

## 1. Workflow 定位

Workflow 負責執行 Request，不負責產生 Request。

Workflow 的唯一工作來源是 Request。

```text
Collector
    ↓
Request
    ↓
Dispatcher
    ↓
Worker
    ↓
Status / Remark / AdminRemark
```

V1 在 Worker 完成 Request 更新後結束，不執行自動 Notification。

---

## 2. 執行條件

Dispatcher 僅自動處理同時符合下列條件的 Request：

```text
ProcessingMode = Automatic
Status = New
```

`ProcessingMode = Manual` 的 Request 不由 Dispatcher 自動派工，交由 MIS 人工處理。

`Status = Cancelled` 的 Request 不得進入 Workflow 執行流程。

---

## 3. Dispatcher

### 3.1 查詢週期

Dispatcher 依固定週期查詢可執行 Request。

規則：

- 查詢週期由設定控制
- 預設每 5 分鐘執行一次
- 每次可以取得多筆 Request
- SQL 查詢集中於 Dispatcher
- Dispatcher 透過 RequestRepository 存取 Database

### 3.2 派工流程

```text
查詢 Automatic + New Request
    ↓
更新 Status = Processing
    ↓
依 RequestType 選擇 Worker
    ↓
將完整 Request 傳入 Worker
    ↓
Worker 執行
```

Dispatcher 必須先將 Request 更新為 Processing，再交由 Worker 執行。

### 3.3 RequestType 分派

V1 使用 `switch` 依 RequestType 選擇 Worker。

V1 不使用：

- Dictionary 分派
- Plugin
- Registry

採用 switch 的原因：

- V1 RequestType 數量少
- 可讀性高
- 重構成本低

### 3.4 Dispatcher 責任

Dispatcher 負責：

- 查詢可執行 Request
- 更新 Processing
- 指派 Worker
- 傳入完整 Request
- 控制派工流程
- Catch Worker Exception
- 單筆 Request 失敗後繼續處理其他 Request

Dispatcher 不負責：

- 執行 RequestType 業務邏輯
- 直接呼叫外部 API
- 發送通知
- 更新 Completed 或 Failed 的業務結果

---

## 4. Worker

### 4.1 Worker 輸入

Dispatcher 將完整 Request 傳入 Worker。

Worker 從 Request 取得：

- RequestId
- RequestType
- Content
- 其他執行與追蹤所需欄位

Worker 自行解析 Content，取得該 RequestType 所需的執行參數。

V1 不建立額外 Execution Context。

### 4.2 執行原則

- 一個 Worker 對應一個 RequestType
- 每個 Worker Instance 一次只處理一筆 Request
- Worker 不主動尋找工作
- Worker 不修改 Request.Content
- Worker 不呼叫其他 Worker
- Worker 不呼叫 Dispatcher
- Worker 不直接執行 SQL
- Worker 不直接呼叫外部 API

### 4.3 平行執行

Dispatcher 一次可以派送多筆 Request。

每個 Worker Instance 一次僅處理一筆 Request，但多個 Worker Instance 可以同時執行，因此 V1 支援平行處理。

Repository 目前未定義 MaxParallelCount 的正式數值，本文件不自行指定。

---

## 5. Worker 成功流程

```text
Worker 執行業務流程
    ↓
更新 Status = Completed
    ↓
更新 Remark
    ↓
更新 AdminRemark
    ↓
更新 UpdatedBy / UpdatedAt
    ↓
寫入 Log
    ↓
結束
```

成功時：

- Status 設為 Completed
- Remark 提供一般使用者可閱讀的結果
- AdminRemark 提供 MIS 所需的執行資訊
- Log 記錄完整技術執行過程

---

## 6. Worker 失敗流程

V1 不區分 Business Failure 與 System Exception。

只要 Worker 無法完成 Request：

```text
Status = Failed
```

失敗流程：

```text
Worker 發生失敗
    ↓
更新 Status = Failed
    ↓
更新 Remark
    ↓
更新 AdminRemark
    ↓
寫入 Log
    ↓
Throw Exception
    ↓
Dispatcher Catch
    ↓
繼續處理其他 Request
```

責任分工：

### Worker

- 更新 Status
- 更新 Remark
- 更新 AdminRemark
- 更新 UpdatedBy / UpdatedAt
- 寫入 Log
- 將 Exception 向上拋出

### Dispatcher

- Catch Exception
- 避免單筆失敗中斷整體派工流程
- 繼續處理其他 Request

資訊分層：

- Remark：一般使用者閱讀，不包含技術細節
- AdminRemark：MIS 診斷摘要，不包含 StackTrace
- Log：完整 Exception 與 StackTrace

---

## 7. Request Status

V1 使用以下 Status：

| Status | 說明 |
|---|---|
| New | 等待處理 |
| Processing | 執行中 |
| Completed | 已完成 |
| Failed | 執行失敗 |
| Cancelled | 已取消 |

Workflow 的主要自動狀態流程：

```text
New
    ↓ Dispatcher
Processing
    ↓ Worker
Completed / Failed
```

Cancelled 不進入自動派工流程。

V1 不建立 Manual Status。人工處理與自動處理由 ProcessingMode 區分。

---

## 8. 人工介入

需要人工確認、人工補件或人工操作時：

- 不新增 Manual Status
- 若自動執行後無法完成，Status 設為 Failed
- Remark 記錄提供使用者閱讀的原因
- AdminRemark 記錄 MIS 診斷摘要
- MIS 依實際狀況人工處理

V1 對於停電、主機故障、作業系統異常或程式被強制結束造成的 Processing 異常，不實作自動復原，由 MIS 人工判斷。

---

## 9. Notification

Notification 保留於平台架構，但 V1 不實作。

Workflow 完成後：

```text
Status
    ↓
Remark / AdminRemark
    ↓
MIS 人工監測
```

需要通知使用者時，由 MIS 人工處理。

---

## 10. V1 不實作項目

- Retry
- Timeout Recovery
- Heartbeat
- Processing 自動復原
- Plugin
- Registry
- Dictionary 分派
- 多 Step Workflow
- Distributed Worker
- Result Model
- Event
- Progress
- Cancellation
- Worker Base Class
- Workflow Version
- 自動 Notification

---

## 11. Design 來源

本文件收斂自：

- `docs/Discussion/2026-07-29-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Notification-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Common-Capability-Log.md`
- `docs/Discussion/2026-07-31-Workflow-Common-Capability-Log.md`
- `docs/Discussion/2026-07-29-Request-Decision-Log.md`
- `docs/Discussion/2026-07-31-Request-Decision-Log.md`
