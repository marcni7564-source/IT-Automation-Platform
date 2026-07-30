# Workflow Decision Log

**建立日期：** 2026-07-29

> 本文件為 Workflow 討論文件，會隨著相關討論持續更新，直到開始 Coding 前，再整合至正式 Design 文件。

---

# 一、Workflow 目標

Workflow 負責執行 Request，不負責產生 Request。

目的：

- 將不同類型的 Request 自動化執行。
- 各模組職責單純。
- V1 以簡單、可維護為原則。

---

# 二、系統流程

```text
Collector
    │
    ▼
Request
    │
    ▼
Dispatcher
    │
    ▼
Worker
    │
    ▼
Notification
```

## 決議

- Request 為 Workflow 唯一工作來源。
- Dispatcher 負責分派。
- Worker 負責執行。
- Notification 為獨立模組。

---

# 三、Dispatcher

## 討論

曾討論由 Worker 自行查詢 Request。

考量查詢會分散在各 Worker，因此改由 Dispatcher 統一管理。

## 決議

Dispatcher 負責：

- 查詢可執行 Request
- 指派 Worker
- 控制整體工作流程

Dispatcher 不負責：

- 執行商業邏輯
- 發送通知

---

# 四、Worker

## 決議

Worker 專責執行工作。

原則：

- 一次只處理一筆 Request。
- 不主動尋找工作。
- 完成後等待 Dispatcher 再次分派。

---

# 五、SQL 查詢策略

## 討論

討論過：

1. Worker 自行查詢 SQL。
2. Dispatcher 集中查詢 SQL。

## 決議

採 Dispatcher 集中查詢。

原因：

- SQL 查詢集中。
- Worker 保持單純。
- 後續維護容易。

---

# 六、Request

Workflow 與 Request 相依。

Workflow 依 Request 狀態執行。

Request 欄位若有新增或調整，需同步更新本文件相關內容。

詳細 Request 設計請參考 Request Decision Log。

---

# 七、Request Status

（2026-07-29 補充）

因 Workflow 已依 Request Status 控制流程，因此 Request 新增 Status 欄位。

Workflow 僅依 Status 控制流程，不自行定義 Workflow Status。

Status 詳細定義與流程，請參考 Request Decision Log。

---

# 八、RequestType 分派

## 討論

目前討論：

- switch
- Dictionary
- Plugin

## 決議

V1 採 switch。

原因：

- RequestType 數量少。
- 可讀性高。
- 重構成本低。

---

# 九、V1 原則

目前確認不納入：

- Retry
- Plugin
- Registry
- Dictionary 分派
- 多 Step Workflow
- Distributed Worker

原則：

有需求再設計，不提前設計。

---

# 十、目前共識

Workflow 目前以簡單架構完成 V1：

- Request 為唯一工作來源。
- Dispatcher 統一管理。
- Worker 專責執行。
- Notification 保持獨立。
- Request 與 Workflow 保持低耦合。
- Workflow 相關討論持續更新於本文件，Coding 前再整合至 Design 文件。
