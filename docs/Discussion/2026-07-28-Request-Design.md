# 2026-07-28 Request Design

## 今日目標

-   確認 Request 模組定位
-   確認 Request 欄位
-   確認 Request Status
-   明確劃分 Request 與 Workflow 的責任

------------------------------------------------------------------------

# 今日結論

## 平台流程

``` text
Request
    ↓
Workflow
    ↓
Notification
```

------------------------------------------------------------------------

## Request 責任

Request 負責：

-   收集來源資料
-   判斷 RequestType
-   Rule / AI / Parser 整理 Content
-   驗證資料完整性
-   建立標準化 Request

Request 不負責：

-   Workflow 執行
-   Notification
-   Mail Archive

------------------------------------------------------------------------

## Request Status

第一階段僅保留：

``` text
New
Manual
```

New： - 可交由 Workflow

Manual： - 人工介入 - Rule/AI 判斷失敗 - 資料不足 - 多個申請

Workflow Status 留待 Workflow 模組設計。

------------------------------------------------------------------------

## Request 欄位

  欄位            用途
  --------------- --------------------
  RequestId       流水號
  Source          MIS 找原始資料來源
  Subject         Mail 主旨
  RequestType     Workflow 類型
  Content         整理完成內容
  RequestStatus   New / Manual
  CreatedBy       建立者
  CreatedAt       建立時間
  UpdatedBy       最後修改者
  UpdatedAt       最後修改時間

------------------------------------------------------------------------

## Source 定義

Source 的目的不是 Workflow 判斷。

主要用途：

-   MIS 人工處理
-   問題追查
-   找到原始資料來源

例如：

``` text
EmailOutlook
EmailExchange
Web
API
```

------------------------------------------------------------------------

## Subject

保存原始 Mail 主旨。

目前不保存 MessageId / SourceId。

理由：

目前可透過：

-   Source
-   Subject
-   CreatedAt

由 MIS 人工回查原始郵件。

------------------------------------------------------------------------

## Content

Content 定義：

> Workflow Input

無論來源為 Rule、AI 或人工整理， Workflow 都只接收 Content。

------------------------------------------------------------------------

## Created / Updated

Created：

-   建立後不可修改

Updated：

-   每次異動更新

建立時：

``` text
CreatedBy = system
CreatedAt = 建立時間
UpdatedBy = system
UpdatedAt = 建立時間
```

之後修改：

``` text
只更新

UpdatedBy
UpdatedAt
```

------------------------------------------------------------------------

# 尚未決定

留待 Workflow 討論：

-   Workflow 如何取得 Request
-   WorkflowInstance
-   Retry
-   執行狀態
-   UpdatedAt 查詢方式
-   是否需要 RequestVersion

------------------------------------------------------------------------

# Future Backlog

## 2026-07-28

### ADR

保留設計決策與原因。

暫不建立。

------------------------------------------------------------------------

### Rule 分類

-   Business Rule
-   Security Guard
-   System Rule

------------------------------------------------------------------------

### Project Governance

-   Git Flow
-   Coding Guideline
-   Document Standard
-   ADR

------------------------------------------------------------------------

### Idea Log

每日累積想法。

第一階段完成後重新評估。

------------------------------------------------------------------------

# 下次建議討論

1.  Workflow 模組

-   Workflow 定位
-   Workflow Status
-   Workflow 與 Request 邊界

2.  Database

-   Request Table
-   Workflow Table

3.  Notification

-   成功通知
-   失敗通知
