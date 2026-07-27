# Request Design

## 定位

Request 負責：

- 收集來源資訊
- 判斷 RequestType
- 產生 Workflow 所需 Content
- 驗證是否可執行

## Status

- New：可交由 Workflow 執行
- Manual：資訊不足、無法判斷或多重需求

## RequestType

V1 由程式定義。

## Content

每個 Workflow 定義自己的 Schema，
Request Rule 必須依照該 Schema 產生 Content。

## Rule

Rule 負責：

1. 判斷 RequestType
2. 擷取資料
3. 將人工判斷轉為規則
4. 產生完整 Content

## 多重需求

V1 原則：

- 一個 Request = 一個業務需求
- 多重需求 → Manual

原因：

目前 Email 已有權責核准流程，因此通常不會出現多重需求。
