# Request Decision Log

**日期**：2026-07-29

> 本文件記錄今日 Request 模組已確認且 V1 將實作的設計內容。

------------------------------------------------------------------------

# 一、Request 定位

Request 為 IT Automation Platform 唯一的工作請求資料。

所有外部來源（Email、Web、API...）皆需先轉換為 Request，再交由 Workflow
處理。

``` text
Collector
    ↓
Request
    ↓
Workflow
    ↓
Notification
```

------------------------------------------------------------------------

# 二、Request 欄位

  欄位             說明
  ---------------- -------------------------------------------
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

------------------------------------------------------------------------

# 三、Status

  Status       說明
  ------------ ----------
  New          等待處理
  Processing   執行中
  Completed    已完成
  Failed       執行失敗
  Cancelled    已取消

------------------------------------------------------------------------

# 四、ProcessingMode

  Mode        說明
  ----------- --------------
  Automatic   系統自動執行
  Manual      人工處理

------------------------------------------------------------------------

# 五、設計決策

-   一封 Email 對應一個 Request。
-   Request 僅描述工作內容，不包含任何執行邏輯。
-   RequestType 用於決定交由哪一個 Worker 處理。
-   ProcessingMode 用於決定自動或人工處理。
-   Status 為整個 Workflow 共用的工作狀態。
-   Notification 與 Workflow 分離。
-   Remark 為備註欄位。
-   Request 為平台唯一工作來源（Queue Source）。

------------------------------------------------------------------------

# 六、Content

不同 RequestType 的專屬資料統一存放於 Content。

例如：

``` json
{
  "username": "demo",
  "group": "SSLVPN"
}
```

避免因新增 RequestType 而持續修改 Request 主表。

------------------------------------------------------------------------

# 七、目前結論

Request V1 設計已完成。

目前不新增其他欄位，也不預留未來需求；待未來有新需求時，再透過 Decision
Log 討論後調整。
