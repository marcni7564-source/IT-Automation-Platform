
> 本文件記錄 Notification 模組於 V1 階段的討論結果。
>
> Notification 完整設計將於未來需要時再進行，本文件僅記錄目前已確認的決策。

---

# 一、討論目的

Workflow V1 已完成 Dispatcher 與 Worker 的討論。

接下來討論：

- V1 是否需要自動寄送通知？
- Notification 是否納入第一階段開發？

---

# 二、目前架構

目前 Workflow 流程：

```text
Collector
    ↓
Request
    ↓
Dispatcher
    ↓
Worker
```

Worker 完成後，原規劃可交由 Notification 模組通知申請人。

---

# 三、討論

自動通知雖可降低人工作業，

但 Workflow V1 尚屬第一版。

目前仍需要確認：

- Request 是否建立正確。
- Dispatcher 是否正常派工。
- Worker 是否執行正確。
- Status 是否更新正確。
- Remark 是否足以描述執行結果。

若此時直接自動寄信，

一旦流程或程式發生錯誤，

可能造成錯誤通知寄送給使用者。

因此目前希望：

先驗證 Workflow，

再驗證 Notification。

---

# 四、決議

Notification 模組保留於整體架構中，

但 V1 不實作。

Workflow 執行完成後：

```text
Collector
    ↓
Request
    ↓
Dispatcher
    ↓
Worker
    ↓
Status
    ↓
Remark
    ↓
MIS 人工監測
```

MIS 依據：

- Status
- Remark

人工確認 Workflow 執行結果。

需要通知使用者時，

由 MIS 人工處理。

---

# 五、原因

採此方式原因：

- 避免錯誤通知寄送。
- 優先驗證 Workflow。
- 降低第一版複雜度。
- 發生異常時容易人工介入。
- 待 Workflow 穩定後，再討論 Notification。

---

# 六、V2 再討論

Notification 未來將討論：

- 觸發時機。
- Email Template。
- 收件者來源。
- 成功通知。
- 失敗通知。
- Retry。
- Notification Log。
- 與 Workflow 的責任分工。

---

# 七、今日結論

Workflow V1 到 Worker 完成即結束。

Notification 不納入 V1 實作。

待 Workflow 經實際驗證穩定後，再開始 Notification 設計。
