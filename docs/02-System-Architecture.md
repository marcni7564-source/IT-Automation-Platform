# System Architecture

> Status: Final Design  
> Version: V1  
> 本文件由 Repository 既有 Discussion 收斂整理，作為後續 Coding 的正式依據。

---

## 1. 文件目的

本文件定義 IT Automation Platform V1 的系統模組、責任邊界、資料流向與依賴方向。

平台目標不是建立 AI 平台，而是將 IT 部門具固定 SOP、固定判斷規則且高度重複的工作，逐步轉換為可重複執行的自動化流程。

設計原則：

- Automation First
- Rule First
- AI Optional
- Workflow 可逐步擴充
- V1 優先維持簡單與可維護
- 不提前設計尚未發生的需求

---

## 2. 整體流程

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
    ↓
Status / Remark / AdminRemark
    ↓
MIS Monitoring
```

Notification 保留為整體架構中的獨立模組，但 V1 不實作自動通知。

V1 的 Workflow 在 Worker 完成 Request 狀態與結果更新後即結束，由 MIS 人工監測執行結果，並在需要時人工通知使用者。

---

## 3. 核心模組

### 3.1 Collector

Collector 負責將不同外部來源轉換為標準化 Request。

外部來源可包含：

- Email
- Web
- API
- 其他未來來源

Collector 負責：

- 收集來源資料
- 解析內容
- 驗證資料
- 加工資料
- 判斷 RequestType
- 決定 ProcessingMode
- 產生符合對應 Workflow 所需格式的 Content
- 建立 Request

Collector 不負責：

- 執行 Workflow
- 執行外部系統業務操作
- 發送 Notification

AI 不是平台核心，只是未來可使用的 Parser 之一。Rule、API、Web Form 或 AI 只要能產生標準化 Request，即可接入平台。

---

### 3.2 Request

Request 是平台唯一的工作來源，也是 Workflow 的 Queue Source。

所有外部來源都必須先轉換為 Request，再交由 Workflow 處理。

核心原則：

- 一封 Email 對應一個 Request
- 一個 Request 對應一個申請事項
- 一個 Request 對應一個 Workflow
- Request 僅描述工作內容，不包含執行邏輯
- Workflow 不依賴 Request 的來源
- Workflow 只依 RequestType 與 Content 執行
- Workflow 不修改 Content
- 不建立獨立 Workflow Status，統一使用 Request.Status

Request 的完整資料模型定義於：

`docs/05-Database-Design.md`

---

### 3.3 Dispatcher

Dispatcher 是 Workflow 的統一派工入口。

Dispatcher 負責：

- 依固定週期查詢可自動執行的 New Request
- 將 Request 更新為 Processing
- 依 RequestType 選擇對應 Worker
- 將完整 Request 傳入 Worker
- 控制整體派工流程
- Catch Worker 向上拋出的 Exception
- 單筆 Worker 失敗時繼續處理其他 Request

Dispatcher 不負責：

- 執行各 RequestType 的業務邏輯
- 直接呼叫外部 API
- 發送 Notification
- 代替 Worker 判斷執行結果

Dispatcher 的詳細流程定義於：

`docs/04-Workflow-Design.md`

---

### 3.4 Worker

Worker 負責執行單一 RequestType 的自動化工作。

核心原則：

- 每個 Worker Instance 一次只處理一筆 Request
- Worker 不主動查詢 Request
- Worker 接收 Dispatcher 傳入的完整 Request
- Worker 解析 Request.Content 取得執行參數
- Worker 依 RequestType 專責完成一項工作
- 多個 Worker Instance 可以同時執行
- Worker 完成後自行更新 Request 執行結果

Worker 更新：

- Status
- Remark
- AdminRemark
- UpdatedBy
- UpdatedAt

Worker 不允許：

- 呼叫其他 Worker
- 呼叫 Dispatcher
- 直接執行 SQL
- 直接呼叫外部 API

---

### 3.5 Service

Service 封裝特定 Workflow 或 Infrastructure 的外部操作。

Workflow Service 命名：

```text
{RequestType}Service
```

範例：

```text
VpnWorker
    ↓
VpnService
    ↓
HttpClient
    ↓
FortiGate API
```

Infrastructure Service 可依其用途命名，例如：

- EmailService
- SharePointService

Service 不允許直接依賴其他 Service。

---

### 3.6 Repository

所有 Database 存取都必須透過 Repository。

```text
Worker
    ↓
Repository
    ↓
Database
```

規則：

- Request 使用 RequestRepository
- 其他 Entity 使用各自對應的 Repository
- Worker 不可直接執行 SQL
- Dispatcher 查詢與更新 Request 時使用 RequestRepository

---

### 3.7 Logger

所有 Worker 共用 Logger 記錄執行過程。

Log 與 Request 欄位的責任分層：

- Status：Workflow 執行狀態
- Remark：提供一般使用者閱讀
- AdminRemark：提供 MIS 維運與診斷
- Log：完整技術資訊、Exception 與 StackTrace

Exception 與 StackTrace 不得寫入 Remark 或 AdminRemark。

Logger 的實際儲存技術、保存天數與檔案管理方式尚未在 Repository 中完成正式設計，本文件不自行新增規格。

---

### 3.8 Notification

Notification 與 Workflow 保持分離。

V1：

- 保留 Notification 模組的架構位置
- 不實作自動寄送
- MIS 依 Status、Remark 與 AdminRemark 人工確認
- 需要通知使用者時由 MIS 人工處理

Notification 的觸發時機、Template、收件者、Retry 與 Notification Log 留待後續版本另行討論。

---

## 4. 模組依賴方向

允許：

```text
Collector → RequestRepository
Dispatcher → RequestRepository
Dispatcher → Worker
Worker → Repository
Worker → Service
Worker → Logger
Service → External API / Infrastructure
Repository → Database
```

不允許：

```text
Worker → Worker
Worker → Dispatcher
Worker → Direct SQL
Worker → Direct External API
Service → Service
```

---

## 5. V1 範圍

V1 納入：

- 標準化 Request
- Collector 建立 Request
- Dispatcher 週期查詢與派工
- RequestType 分派
- Worker 執行
- Worker 平行 Instance
- Status、Remark、AdminRemark 更新
- Logging
- MIS 人工監測

V1 不納入：

- 自動 Notification
- Retry
- Timeout Recovery
- Heartbeat
- Processing 自動復原
- Plugin
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

未納入項目待實際需求出現後，再經 Discussion → Design 流程決定。

---

## 6. Design 來源

本文件收斂自：

- `docs/01-Why-IT-Automation-Platform.md`
- `docs/Discussion/06-Decisions-2026-07-28.md`
- `docs/Discussion/2026-07-28-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Request-Decision-Log.md`
- `docs/Discussion/2026-07-29-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Decision-Log.md`
- `docs/Discussion/2026-07-30-Notification-Decision-Log.md`
- `docs/Discussion/2026-07-30-Workflow-Common-Capability-Log.md`
- `docs/Discussion/2026-07-31-Request-Decision-Log.md`
- `docs/Discussion/2026-07-31-Workflow-Common-Capability-Log.md`
