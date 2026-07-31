# Coding Conventions

> Status: Final Design  
> Version: V1  
> 本文件定義 IT Automation Platform 的命名、責任邊界、依賴與 Exception Handling 規範。

---

## 1. 語言規範

以下內容統一使用英文：

- 程式碼
- Class
- Interface
- Method
- Property
- API
- Database Table
- Database Column

文件說明與註解可依專案需求使用繁體中文。

---

## 2. Class 命名

### 2.1 Worker

格式：

```text
{RequestType}Worker
```

範例：

```text
LargeFileWorker
VpnWorker
MeetingWorker
```

Worker 必須與 RequestType 對應，並專責執行該 RequestType 的工作。

### 2.2 Workflow Service

格式：

```text
{RequestType}Service
```

範例：

```text
LargeFileService
VpnService
MeetingService
```

Workflow Service 封裝該 RequestType 所需的外部操作。

### 2.3 Infrastructure Service

Infrastructure Service 不受 `{RequestType}Service` 限制，依實際用途命名。

範例：

```text
EmailService
SharePointService
```

### 2.4 Repository

格式：

```text
{Entity}Repository
```

範例：

```text
RequestRepository
UserRepository
```

### 2.5 Interface

格式：

```text
I{Name}
```

範例：

```text
IRequestRepository
IVpnService
ILogger
```

---

## 3. Method 命名

Method 使用動詞開頭。

範例：

```text
Create()
Update()
Delete()
GetById()
```

Method 名稱必須描述實際動作，不使用無法判斷用途的模糊名稱。

Repository 目前尚未決定非同步 Method 是否統一加上 `Async`，本文件不自行新增規則。

---

## 4. Boolean 命名

Boolean 使用下列前綴：

```text
Is
Has
Can
```

範例：

```text
IsEnabled
HasPermission
CanExecute
```

---

## 5. Database 命名

### Table

使用單數英文名詞。

範例：

```text
Request
User
```

### Column

使用 PascalCase。

範例：

```text
RequestId
RequestType
CreatedAt
AdminRemark
```

Database Key、Index、Constraint 的命名規則尚未完成 Discussion，本文件不自行新增。

---

## 6. Worker 責任

Worker 負責：

- 接收 Dispatcher 傳入的完整 Request
- 解析 Request.Content
- 執行對應 RequestType 的業務流程
- 透過 Repository 存取 Database
- 透過 Service 呼叫外部系統
- 透過 Logger 寫入 Log
- 更新 Status
- 更新 Remark
- 更新 AdminRemark
- 更新 UpdatedBy 與 UpdatedAt

Worker 不負責：

- 主動查詢 Request
- 呼叫其他 Worker
- 呼叫 Dispatcher
- 直接執行 SQL
- 直接呼叫外部 API
- 發送 V1 Notification
- 修改 Request.Content

---

## 7. Dependency 規範

### 7.1 允許依賴

```text
Dispatcher → RequestRepository
Dispatcher → Worker
Worker → Repository
Worker → Service
Worker → Logger
Repository → Database
Service → External API / Infrastructure
```

### 7.2 禁止依賴

```text
Worker → Worker
Worker → Dispatcher
Worker → Direct SQL
Worker → Direct External API
Service → Service
```

所有 Database 存取都必須透過 Repository。

所有 Worker 對外部 API 的操作都必須透過 Service。

---

## 8. RequestType 分派

V1 的 Dispatcher 使用 `switch` 依 RequestType 選擇 Worker。

V1 不使用：

- Dictionary 分派
- Plugin
- Registry

不得在未經 Discussion → Design 確認前自行替換分派方式。

---

## 9. Exception Handling

### 9.1 Failure 判定

V1 不區分 Business Failure 與 System Exception。

只要 Worker 無法完成 Request：

```text
Status = Failed
```

### 9.2 Worker Exception 責任

Worker 發生失敗時依序：

```text
更新 Request
    ↓
寫入 Log
    ↓
Throw Exception
```

Worker 必須更新：

- Status = Failed
- Remark
- AdminRemark
- UpdatedBy
- UpdatedAt

### 9.3 Dispatcher Exception 責任

Dispatcher 必須 Catch Worker 向上拋出的 Exception。

目的：

- 避免單筆 Request 失敗造成整體 Dispatcher 中斷
- 繼續處理其他 Request

### 9.4 訊息分層

#### Remark

- 提供一般使用者閱讀
- 避免技術細節
- 不得包含 Exception
- 不得包含 StackTrace

#### AdminRemark

- 提供 MIS 診斷
- 可包含 API 回傳摘要
- 可包含 Exception 摘要
- 不得包含 StackTrace

#### Log

- 記錄完整執行過程
- 記錄 Exception
- 記錄 StackTrace

---

## 10. Logging

Worker 透過共用 Logger 寫入 Log。

每筆 Log 的討論方向至少包含：

- Timestamp
- Level
- RequestId
- WorkerName
- Message
- Exception（有才記錄）

Log Level 討論方向：

- DEBUG
- INFO
- WARN
- ERROR

Logger 的具體實作套件、輸出格式、檔案路徑、保存天數與輪替方式尚未完成正式決議，因此 Coding 時不得自行視為既定規格。

---

## 11. Configuration

已確認的可設定項目：

- Dispatcher PollingInterval
- 預設值為 5 分鐘

Repository 曾討論集中管理其他系統參數，但 Configuration Table、MaxParallelCount、LogLevel 與 Feature Enable / Disable 的正式儲存方式尚未定案，本文件不自行新增。

---

## 12. V1 禁止提前實作項目

除非先完成新的 Discussion → Design，V1 不得提前實作：

- Retry
- Timeout Recovery
- Heartbeat
- Processing 自動復原
- Plugin Architecture
- Registry
- Dictionary 分派
- 多 Step Workflow
- Distributed Worker
- Workflow Version
- Content Version
- Worker Base Class
- Result Model
- Event
- Progress
- Cancellation
- 自動 Notification

---

## 13. Design 來源

本文件收斂自：

- `docs/Discussion/2026-07-29-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Common-Capability-Log.md`
- `docs/Discussion/2026-07-31-Workflow-Common-Capability-Log.md`
- `docs/Discussion/2026-07-31-Request-Decision-Log.md`
