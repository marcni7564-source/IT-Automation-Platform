# 2026-07-31-Workflow-Common-Capability-Log

> Status: Discussion Date: 2026-07-31

本文件為 Discussion，記錄今日 Workflow Common Capability
的討論內容，不代表最終 Design。

------------------------------------------------------------------------

# Exception Handling

## Discussion

### Failure Status

目前不區分 Business Failure 與 System Exception，只要 Worker 無法完成
Request：

    Status = Failed

### Worker Responsibility

-   更新 Status
-   更新 Remark
-   更新 AdminRemark
-   寫入 Log

Remark 提供使用者閱讀；AdminRemark 提供 MIS 診斷，不存放 StackTrace。

Exception 與 StackTrace 僅寫入 Log。

### Exception Flow

    Worker
     ↓
    更新 Request
     ↓
    寫入 Log
     ↓
    Throw Exception
     ↓
    Dispatcher Catch
     ↓
    繼續下一筆 Request

### Current Discussion Summary

-   Worker 更新 Request
-   Worker 寫 Log
-   Dispatcher Catch Exception，避免流程中斷

------------------------------------------------------------------------

# Dependency

## Discussion

### Dispatcher → Worker

Dispatcher 傳入完整 Request。

### Repository

所有 Database 存取皆透過 Repository。

    Worker
     ↓
    Repository
     ↓
    Database

Request 使用 RequestRepository，其餘 Entity 使用對應 Repository。

### Service

Worker 不直接呼叫外部 API。

    VpnWorker
     ↓
    VpnService
     ↓
    HttpClient
     ↓
    FortiGate API

### Logger

Worker 透過 Logger 寫 Log。

### Restriction

允許：

-   Worker → Repository
-   Worker → Service
-   Worker → Logger

不允許：

-   Worker → Worker
-   Worker → Dispatcher
-   Worker → Direct SQL
-   Worker → Direct External API
-   Service → Service

------------------------------------------------------------------------

# Naming Convention

## Discussion

### Language

程式碼、Class、API、Database 使用英文。

### Worker

    {RequestType}Worker

### Service

Workflow Service：

    {RequestType}Service

Infrastructure Service 不受限制，例如：

-   EmailService
-   SharePointService

### Repository

    {Entity}Repository

### Database

Table：單數名詞

Column：PascalCase

### Method

使用動詞開頭：

-   Create()
-   Update()
-   Delete()
-   GetById()

### Boolean

使用：

-   Is
-   Has
-   Can

### Interface

    I{Name}

### Current Discussion Summary

-   Worker：{RequestType}Worker
-   Service：{RequestType}Service
-   Repository：{Entity}Repository
-   採統一英文命名
