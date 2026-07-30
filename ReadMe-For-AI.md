

**本文件會隨專案開發階段持續調整。**

不同階段（Discussion、Design、Coding、Maintenance），
AI 應遵循當前版本所定義的工作流程。


## 文件目的

本文件說明 AI 如何閱讀、理解與參與本專案。

本文件不定義系統功能，而是定義 AI 的工作流程。

---

# Repository 文件分類

## docs/*.md

用途：

正式設計文件（Final Design）。

特性：

- 已完成討論。
- 已確認最終決策。
- 作為 Coding 的主要依據。
- 不保留討論過程。

---

## docs/Discussion/*.md

用途：

專案討論脈絡（Project Context）。

特性：

- 記錄討論內容。
- 記錄決策原因。
- 可持續新增。
- 可持續修正。
- 不代表最終設計。

Discussion 的目的，是累積知識，而不是直接作為 Coding 規格。

---

# Discussion 檔案命名

格式：

yyyy-MM-dd-{Topic}-Log.md

例如：

2026-07-30-Workflow-Log.md

2026-08-05-Workflow-Log.md

---

# AI 閱讀流程

第一次閱讀專案時：

ReadMe-For-AI.md

↓

01-Why-IT-Automation-Platform.md

↓

與本次主題相關的 docs/*.md

↓

與本次主題相關的 docs/Discussion/*.md

↓

開始討論

AI 不需要閱讀所有文件。

僅閱讀與目前主題相關的文件即可。

---

# Discussion 與 Design 關係

Discussion

↓

持續討論

↓

累積專案知識

↓

整理

↓

Design

↓

Coding

Design 並非同步撰寫。

Design 應於相關 Discussion 完整後，再整理產生。

一份 Design 可整理多份 Discussion。

---

# AI 討論原則

AI 回答前：

優先閱讀相關文件。

若文件已有定義：

依文件回答。

若文件沒有定義：

可以提出建議。

但必須明確標示：

「以下為建議」。

不得將建議視為既定規格。

---

# AI Coding 原則

Coding 前：

應優先依據 docs/*.md。

Discussion 僅提供設計脈絡。

若 Design 尚未完成：

應持續討論。

而非直接開始 Coding。

---

# AI 不應做的事情

不得：

- 自行新增規格。
- 自行修改專案方向。
- 自行假設需求。
- 自行套用一般最佳實務。

AI 可以提出建議。

但最終決策由使用者確認。

---

# 專案流程

Discussion

↓

Discussion MD

↓

Design MD

↓

Coding

↓

Testing

# 目前 Discussion

- docs/Discussion/06-Decisions-2026-07-28.md
- docs/Discussion/2026-07-28-Request-Decision-Log.md
- docs/Discussion/2026-07-29-Request-Decision-Log.md
- docs/Discussion/2026-07-29-Workflow-Decision-Log.md
