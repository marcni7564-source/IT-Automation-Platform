> 本文件討論 Workflow 平台中，所有 Worker 是否需要具備共同能力。

> 本次僅整理討論方向，不代表最終設計。
> 待各項能力逐步確認後，再整理為正式 Design 文件。

---

# 一、討論目的

Request 與 Workflow V1 已完成。

接下來開始思考：

是否存在一套所有 Workflow 都共用的能力？

例如：

- Large File
- Meeting
- VPN
- Active Directory
- FortiGate
- Webex

未來皆可能共用相同設計。

因此希望先整理 Workflow Common Capability。

---

# 二、目前想到的能力

目前討論方向：

- Configuration
- Logging
- Worker Input
- Worker Output
- Single Responsibility

後續仍可持續增加。

---

# 三、Configuration

## 討論

Workflow 是否應將可調整參數集中管理？

例如：

- Dispatcher PollingInterval
- Worker MaxParallelCount
- LogLevel
- Feature Enable / Disable

目前傾向：

- 系統可調整參數集中管理。
- 啟動必要設定仍使用 appsettings.json。

資料表設計待 Database Design 再討論。

---

# 四、Logging

## 討論

Workflow 是否建立共用 Logger？

目前討論方向：

- 每天一個 Log 檔。
- 所有 Worker 共用 Logger。
- 預設記錄：
  - DEBUG
  - INFO
  - WARN
  - ERROR

Log Level 可由系統設定調整。

每筆 Log 建議至少包含：

- Timestamp
- Level
- RequestId
- WorkerName
- Message
- Exception（有才記錄）

Log 保存方式與保存天數待後續討論。

---

# 五、Worker Input

## 討論

Dispatcher 是否需要建立 Execution Context？

目前討論傾向：

Dispatcher 直接將完整 Request 傳入 Worker。

Worker 自行解析：

Request.Content

取得執行所需參數。

是否建立額外 Context，待後續評估。

---

# 六、Worker Output

## 討論

Worker 執行完成後，

目前討論方向：

直接更新：

- Request.Status
- Request.Remark

Notification V1 不納入流程。

因此 Workflow 至 Worker 更新完成即結束。

---

# 七、Single Responsibility

## 討論

每個 Worker 是否應只專注完成一項工作？

例如：

LargeFileWorker

負責：

- 建立分享連結

不負責：

- Email
- Dashboard
- Notification
- 其他 Workflow

是否將「單一責任」列為所有 Worker 共通原則，待後續討論。

---

# 八、待討論

目前尚未討論：

- Result Model
- Cancellation
- Progress
- Retry
- Event
- Plugin Architecture
- Workflow Version
- Worker Base Class
- 共用 Library

後續依需要逐步補充。
