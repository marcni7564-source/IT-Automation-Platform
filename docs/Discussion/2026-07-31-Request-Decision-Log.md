## Decision

Request 資料表新增：

- Remark（備註）
- AdminRemark（管理備註）

取代原本僅有一個 Remark 欄位。

---

## Reason

原本僅有 Remark 欄位時，

需同時承擔：

- 使用者閱讀
- MIS 除錯

兩種用途。

隨著 Workflow 增加，

容易造成：

- 使用者看到過多技術訊息。
- MIS 無法留下完整診斷資訊。

因此決定拆分為兩個欄位。

---

## Decision Detail

### Remark（備註）

用途：

提供一般使用者閱讀。

內容應避免技術細節。

例如：

- 檔案已成功上傳。
- VPN 帳號已建立。
- 會議已建立完成。
- 您的申請已完成。
- 系統忙碌，請稍後再試。

---

### AdminRemark（管理備註）

用途：

提供 MIS / 系統管理員查看。

可記錄：

- API 回傳訊息
- Exception 摘要
- 系統診斷資訊
- 其他維運相關資訊

例如：

- SharePoint API Timeout
- SMTP Send Failed
- User Not Found
- SQL Timeout
- Return Code = -61

AdminRemark 不提供一般使用者閱讀。

---

## Responsibility

Worker 完成後應更新：

- Status（狀態）
- Remark（備註）
- AdminRemark（管理備註）

Logger（日誌）仍負責完整執行過程與 Exception StackTrace。

---

## Result

Request 的資訊分層如下：

- Status（狀態）：Workflow 執行狀態。
- Remark（備註）：提供使用者閱讀。
- AdminRemark（管理備註）：提供 MIS 維運與診斷。
- Log（日誌）：完整技術細節與執行紀

# Request Decision Log

**日期**：2026-07-29

> 本文件記錄今日 Request 模組已確認且 V1 將實作的設計內容。

---

# 一、Request 定位

Request 為 IT Automation Platform 唯一的工作請求資料。

所有外部來源（Email、Web、API...）皆需先轉換為 Request，再交由 Workflow
處理。

```text
Collector
    ↓
Request
    ↓
Workflow
    ↓
Notification
```

---

# 二、Request 欄位

  欄位             說明

---

  RequestId        唯一識別碼
  Source           Request 來源
  Subject          主旨
  RequestType      工作類型（例如：VPN、Meeting、LargeFile）
  Content          Request 內容
  ProcessingMode   Automatic / Manual
  Status           工作狀態
  Remark           備註
  CreatedBy        建立者
  CreatedAt        建立時間
  UpdatedBy        更新者
  UpdatedAt        更新時間

---

# 三、Status

  Status       說明

---

  New          等待處理
  Processing   執行中
  Completed    已完成
  Failed       執行失敗
  Cancelled    已取消

---

# 四、ProcessingMode

  Mode        說明

---

  Automatic   系統自動執行
  Manual      人工處理

---

# 五、設計決策

- 一封 Email 對應一個 Request。
- Request 僅描述工作內容，不包含任何執行邏輯。
- RequestType 用於決定交由哪一個 Worker 處理。
- ProcessingMode 用於決定自動或人工處理。
- Status 為整個 Workflow 共用的工作狀態。
- Notification 與 Workflow 分離。
- Remark 為備註欄位。
- Request 為平台唯一工作來源（Queue Source）。

---

# 六、Content

不同 RequestType 的專屬資料統一存放於 Content。

例如：

```json
{
  "username": "demo",
  "group": "SSLVPN"
}
```

避免因新增 RequestType 而持續修改 Request 主表。

---

# 七、目前結論

Request V1 設計已完成。

目前不新增其他欄位，也不預留未來需求；待未來有新需求時，再透過 Decision
Log 討論後調整。
