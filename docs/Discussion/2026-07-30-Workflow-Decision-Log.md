
# 2026-07-30 Workflow Decision Log

> 延續 2026-07-29 Workflow Decision Log，本次討論主要確認 Workflow V1 的實作細節，並釐清 Dispatcher、Worker 的責任範圍與例外處理方式。

---

# 一、討論目的

前一天已完成 Workflow 架構設計。

今日希望確認：

- Dispatcher 實際如何派工。
- Worker 如何更新 Request 狀態。
- Processing 狀態是否需要 Timeout Recovery。
- Worker 是否允許平行執行。
- V1 是否需要 Retry、Heartbeat 等機制。

---

# 二、Dispatcher 執行方式

## 討論

昨天雖已決定 Dispatcher 負責派工，但尚未決定實際執行流程。

討論重點：

- 一次抓一筆 Request？
- 還是一次抓多筆 New Request？
- 查詢頻率如何控制？
- Request 何時更新為 Processing？

## 決議

Dispatcher：

- 依固定週期查詢 New Request。
- 查詢週期由參數檔控制。
- 預設每 5 分鐘執行一次。
- 每次可取得多筆 New Request。
- Dispatcher 更新 Status = Processing 後，再分派 Worker。

---

# 三、Worker 責任

## 討論

曾討論：

Worker 是否只負責執行？

還是：

Dispatcher 統一更新 Success / Failed？

討論後認為：

真正知道執行結果的是 Worker。

如果 Dispatcher 回寫狀態，反而需要依賴 Worker 回傳更多資訊。

## 決議

Worker：

- 一次只處理一筆 Request。
- Dispatcher 指派後開始執行。
- 不主動查詢 Request。
- 依 RequestType 執行對應流程。
- 使用 Request Content 作為輸入參數。
- 執行完成後自行更新：

  - Status
  - Remark

---

# 四、Status 更新方式

## 討論

Completed：

正常完成。

Failed：

程式執行失敗。

另外討論：

若需要人工介入，例如：

- 人工確認
- 人工補件
- 人工操作

是否應建立 Manual Status？

討論後認為：

若建立 Manual，未來統計容易混亂。

真正角度應該是：

程式沒有完成。

因此仍屬 Failed。

Remark 紀錄人工處理原因即可。

## 決議

Status：

- Completed
- Failed

Remark：

- Exception
- 人工處理原因
- 其他說明

---

# 五、Exception 處理

## 討論

討論：

是否需要：

- Retry
- Timeout Recovery
- Processing 自動復原
- Heartbeat

考量：

程式 Exception：

可由 try-catch 處理。

不可預期事件：

例如：

- 停電
- 主機故障
- 作業系統異常
- 程式被強制結束

這些事件即使增加 Recovery，仍可能需要人工確認。

V1 若加入上述機制，將增加大量複雜度。

## 決議

程式 Exception：

- try-catch
- Failed
- Remark

重大異常：

交由 MIS 人工判斷。

V1 不實作：

- Retry
- Timeout Recovery
- Heartbeat
- Processing 自動復原

保留 V2 再評估。

---

# 六、Worker 平行執行

## 討論

曾討論：

Worker 是否只能依序執行？

若 New 很多：

是否允許平行處理？

## 決議

Dispatcher：

一次可派送多筆 Request。

Worker：

每個 Worker Instance 一次僅處理一筆 Request。

多個 Worker Instance 可同時執行。

因此 Workflow 可支援平行處理。

---

# 七、今日結論

Workflow V1：

Dispatcher

↓

查詢 New Request

↓

更新 Processing

↓

依 RequestType 指派 Worker

↓

Worker 執行

↓

Worker 更新 Status、Remark

↓

Completed / Failed

---

# 待討論

目前無。

Workflow V1 實作方向已確認。
