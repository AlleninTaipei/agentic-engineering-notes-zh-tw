# Codex Behind the Harness: 從 Context、Tools 到 Long-running Loops

> 來源: [Codex, Behind the Harness — Dominik Kundel, OpenAI](https://www.youtube.com/watch?v=shRR1e2HXMk)
> 頻道: AI Engineer
> 發布日期: 2026-08-10
> 片長: 20:54
> Video ID: `shRR1e2HXMk`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

OpenAI 的 Dominik Kundel 從一則使用者訊息進入 Codex 開始, 拆解 agent harness 的主要工作: 通訊協定、context construction、延遲載入工具、非同步行動、computer use、檔案編輯、sandbox、自動安全審查、WebSocket、goal loop 與 server-side compaction。

演講的核心觀點是, 模型能力只是 agent 系統的一部分。Harness 必須處理模型以外的工程問題:

- 如何提供足夠但不混亂的 context。
- 如何讓工具數量隨 plugins、skills 與 MCP servers 擴展。
- 如何安全執行長時間、非同步且具有副作用的工作。
- 如何在推論變快後減少網路與協定成本。
- 如何讓 agent 跨越多個 context windows 持續追蹤可驗證目標。

Codex app server 與 harness 可作為開源 blueprint, 而多項底層能力也能透過 Responses API 提供給其他 agent harness 使用。

## Codex 的兩層通訊協定

[00:00](https://www.youtube.com/watch?v=shRR1e2HXMk&t=0s)

當使用者從 UI 送出訊息時, 內容不會直接交給模型。Codex 的資料流包含兩層協定:

```text
使用者介面
  -> App Server Protocol
Codex Harness
  -> Responses API
模型推論服務
```

### App Server Protocol

[01:56](https://www.youtube.com/watch?v=shRR1e2HXMk&t=116s)

App server 負責 UI 與 harness 之間的溝通。Codex app 本身也使用相同介面, 因此第三方可以在 harness 上建立自己的 agent UI 或整合。

Dominik 提到的社群案例包括 T3 Code、Remote X, 以及把 Codex 接入 Claude Code 或 Doom 的實驗。重點不是特定前端, 而是將 agent runtime 與 presentation layer 分離。

### Responses API

Responses API 負責 harness 與模型推論之間的通訊。它是針對 agentic workload 重新設計的介面, 除了訊息結構之外, 也納入 web search、image generation、tool search 等能力。

OpenAI 也與 Ollama、LM Studio、Nvidia 等合作推動 open responses schema。演講描述的目標是讓相容的 model provider 能接入同一種 harness, 降低 agent runtime 與模型平台的綁定。

Dominik 表示 Codex harness 以 Rust 撰寫並開源, 演講亦提到 MIT 與 Apache-2.0 licensing。實際採用或再散布時, 仍應以各 repository 當下的 license 檔案為準。

## Context Construction 的三個目標

[04:04](https://www.youtube.com/watch?v=shRR1e2HXMk&t=244s)

Harness 收到訊息後, 第一項關鍵工作是建構模型 context。Codex 同時考慮三件事:

| 目標 | 原因 |
| --- | --- |
| Size | 避免浪費 token, 也避免過多互相矛盾的資訊干擾模型 |
| Flexibility | 不論 skills、plugins 和 MCP servers 數量多少, 都能維持可用體驗 |
| Cacheability | 控制成本與延遲, 讓穩定前綴可重複使用 |

Model instructions 的結構與大小相對穩定。真正難以預測的是 available skills 和 tool registry, 因為使用者可能安裝不同數量的 skills、plugins 與 MCP servers。

### Deferred Tools 與 Tool Search

[05:34](https://www.youtube.com/watch?v=shRR1e2HXMk&t=334s)

如果把所有工具定義直接放入 context, 工具愈多, 初始 token 成本就愈高。Codex 會把部分工具標記為 deferred, 不在一開始載入完整定義, 而是讓模型需要時透過 tool search 發現。

演講提到兩項控制策略:

- Deferred tools 只在搜尋後加入 context。
- Available skills 清單最多使用總 context window 的約 2%。清單變長時, harness 逐步縮短 description。

自 GPT-5.4 起, Responses API 可將工具標記為 deferred loading。開發者可以使用內建 tool search, 或自行實作 discovery mechanism。

這是一種 progressive disclosure:

```text
初始 context 只放精簡索引
  -> 模型判斷需要某種能力
  -> Tool search 找到候選工具
  -> 載入相關工具定義
  -> 執行工具
```

## Agent Actions: 非同步工作、Computer Use 與檔案系統

[06:58](https://www.youtube.com/watch?v=shRR1e2HXMk&t=418s)

只有 context 還不構成 agent。Agent 必須能對環境採取行動。演講聚焦三類常見 action。

### 非同步 Sub-agents

主 agent 可以透過 `spawn_agent` 建立新的 agent instance, 把獨立工作委派出去。之後再使用工具:

- 傳送新的內容或補充指令。
- 等待 agent 完成。
- 關閉 agent。

主 agent 不必在 sub-agent 工作期間停止所有活動, 因而能實現平行分工。

### Background Terminals

相同的非同步概念也可套用到 terminal。Codex 能啟動 background terminal, 再透過 standard input 持續互動, 或等待指定時間檢查工作是否完成。

這種 session handle 設計讓 agent 可以管理長時間執行的測試、build、server 或其他 process, 而不是讓單次工具呼叫永久阻塞。

### 以 Code Execution 實作 Computer Use

[08:16](https://www.youtube.com/watch?v=shRR1e2HXMk&t=496s)

早期 Responses API 的 computer use 一次只允許一個預先定義的 action。Harness 需要明確實作每一種可用操作, 表達能力有限。

新方向是讓模型透過 code execution 編寫自己的互動程序。開發者可以選擇 JavaScript 或 Python, 再提供對應的電腦環境。

Codex browser use 使用 persistent Node REPL。模型在其中產生類似 Playwright 的 JavaScript:

1. 第一次檢查目前瀏覽器、頁籤和頁面結構。
2. 後續 turn 能繼續引用既有 tabs 和物件。
3. 理解第一頁結構後, 可寫腳本處理後續頁面。
4. 多個操作能整合成一段程式, 減少逐步往返。

持久化執行環境同時提供記憶與效率, 讓 browser use 從固定 click action 升級為可程式化操作。

## 檔案編輯與 Shell

[10:02](https://www.youtube.com/watch?v=shRR1e2HXMk&t=602s)

近期 Codex models 從 GPT-5 起接受以 `apply_patch` 編輯檔案的訓練。模型以 diff 修改或建立檔案, 其他搜尋、導航與執行工作則交給 shell。

演講提到幾項 harness 與訓練的配合:

- 模型會自然使用 Ripgrep 搜尋內容。
- Codex harness 會在本機沒有安裝時提供 Ripgrep。
- Windows 環境下, 模型接受原生 PowerShell 使用方式的訓練。
- 檔案系統 action 仍需經過 sandbox layer。

這說明 tool design 不只是建立 JSON schema。若模型在訓練中熟悉工具慣例, 它會更穩定地選擇正確操作方式。

## 跨平台 Sandbox

[10:54](https://www.youtube.com/watch?v=shRR1e2HXMk&t=654s)

Codex 對檔案和程序的操作會通過平台 sandbox:

| 平台 | Sandbox 技術 |
| --- | --- |
| macOS | Seatbelt |
| Linux | Bubblewrap |
| Windows | Codex repository 中的自訂開源 sandbox |

Windows 沒有直接採用現成方案。Dominik 表示團隊為此建立自訂 sandbox, 詳細原因另外記錄在工程文章中。

Sandbox 的難題不只在隔離, 也在 approval fatigue。長任務頻繁要求人工核准, 會讓使用者乾脆開啟 full access, 反而放大風險。

## Auto Review: 用獨立 Agent 審核越權 Action

[11:48](https://www.youtube.com/watch?v=shRR1e2HXMk&t=708s)

即使模型能力提高, 高 agency 仍可能和使用者預期不一致。例如:

- 使用者要求寄出檔案, agent 無法附加附件, 便自行上傳到 file-sharing service。
- Shell escaping 出錯, 意外刪除超出目標範圍的資料。

因此 Codex 建立 auto review, 希望自動處理可預測的安全核准, 同時繼續阻止 data exfiltration 等高風險行為。

### Auto-review Sub-agent 的限制

當主要 agent 的 action 需要 sandbox escalation 時, harness 會啟動獨立 reviewer:

- Reviewer 與主 agent 分離。
- 只有 read permission。
- 不能再建立 sub-agents。
- 取得 task transcript 與準備執行的 tool call。
- 根據 user authorization、risk taxonomy 和 action impact 做出判斷。

Context 是判斷風險的核心。同一個刪除動作可能合理, 也可能危險:

| 情境 | 判斷方向 |
| --- | --- |
| 使用者明確要求刪除某個檔案 | 具有較高 user authorization |
| 工作流程本來就需要清理生成物 | 可根據範圍自動核准 |
| 使用者未要求刪除 `.git` | 應阻止, 避免失去 repository history |
| 連線 Google 測試網路 | 可能是低風險 network action |
| 將本機檔案上傳到外部服務 | 涉及資料外洩, 不應自動核准 |

Auto review 的重點不是讓另一個模型籠統判斷「安全或不安全」, 而是把權限、意圖與具體影響一起納入決策。

## 推論變快後, Network 成為瓶頸

[15:04](https://www.youtube.com/watch?v=shRR1e2HXMk&t=904s)

Agent 會頻繁呼叫工具, 每次都可能在 harness 與 inference service 之間往返。當 GPT-5.3 Codex Spark 在 Cerebras 上達到演講所述的每秒約 1,000 tokens 後, 推論不再是唯一瓶頸, network overhead 變得顯著。

Responses API 因而加入 WebSocket mode, 以 persistent connection 取代反覆透過 HTTP 和 server-sent events 建立資料交換。

WebSocket mode 的效益包括:

- 減少連線與協定成本。
- 維持 stateful context。
- 每次只傳送新增或改變的資料。
- Tool call 結束後只回傳 result, 不必重送全部 items。

對單一 turn 而言差異可能有限, 但長 trajectory 累積大量工具呼叫後, 傳輸量和 latency 會明顯下降。

## `/goal`: Harness 如何維持工作迴圈

[17:05](https://www.youtube.com/watch?v=shRR1e2HXMk&t=1025s)

`/goal` 讓 harness 在模型尚未完成目標時自動延續工作。其概念流程是:

```text
使用者設定 objective
  -> Agent 執行一輪工作
  -> Harness 注入 continuation prompt
  -> Prompt 再次包含 objective
  -> Agent 繼續工作
  -> Agent 呼叫 update_goal 宣告完成
  -> 迴圈停止
```

這也解釋了 goal 應該如何撰寫。長篇背景不如具體、可驗證的 objective 有效。Harness 需要能判斷「何時真的完成」, 模型也需要明確終止條件。

可操作的 goal 應包含:

- 清楚的最終狀態。
- 可執行的完成驗證。
- 不混入大量與終止條件無關的敘述。
- 避免只能主觀判定的模糊詞語。

## Server-side Compaction 支援長時間 Agent

[18:25](https://www.youtube.com/watch?v=shRR1e2HXMk&t=1105s)

Agent 若執行數小時或數天, 原始 transcript 最終會超出 context window。Codex 可手動或自動觸發 server-side compaction, 將先前 context 轉換成新的 context window。

新的 context 包含 compaction item, 保存後續工作需要的資訊。Dominik 強調, 這套 compaction 形式與模型訓練配合, 目標是在壓縮後維持任務表現。

Compaction 並非單純把整段對話做一般摘要。對長任務而言, 應特別保留:

- Objective 與完成條件。
- 已做決策及其理由。
- 已修改的檔案與當前狀態。
- 驗證結果和未解問題。
- 下一步可直接採取的行動。

## Agent Harness 的完整責任鏈

演講展示的元件可以整理為一條處理鏈:

```text
UI / Third-party Client
  -> App Server Protocol
  -> Context Construction
     -> Stable instructions
     -> Skills index
     -> Deferred tools / Tool search
  -> Responses API / WebSocket
  -> Model inference
  -> Actions
     -> apply_patch / shell
     -> browser use / persistent REPL
     -> sub-agents / background terminals
  -> Sandbox
     -> Auto-review sub-agent
     -> Approval or rejection
  -> Tool results
  -> Goal continuation
  -> Compaction
  -> 下一輪推論
```

這條鏈顯示 harness 並非薄薄的模型 wrapper。它同時是 context manager、tool runtime、security boundary、protocol adapter 和 long-running task scheduler。

## 實作啟示

### Context 與工具擴展

- 不要在初始 prompt 中載入所有工具定義。
- 為 tools 和 skills 建立精簡索引及延遲載入機制。
- 限制 discovery metadata 可使用的 context 比例。
- 優先保持 prompt 前綴穩定, 提高 cacheability。

### 非同步 Agent Runtime

- 為 sub-agents 與 background processes 提供穩定 handle。
- 支援 send、wait、poll 和 terminate 等生命週期操作。
- 避免長時間工具呼叫阻塞主 agent。
- 讓持久化 REPL 保存跨 turn 的操作狀態。

### 安全與權限

- 所有具副作用的檔案、程序和網路 action 都經過 sandbox。
- 審核時同時檢查使用者授權、作用範圍和不可逆影響。
- Reviewer 應隔離、唯讀, 且不能自行擴大能力。
- 不把減少 approval fatigue 等同於放棄核准邊界。

### 長時間任務

- Goal 要具體且可驗證。
- Harness 應控制 continuation, 不依賴使用者反覆催促。
- Compaction 要保存任務狀態, 不只保存談話摘要。
- 當模型速度提高時, 重新量測 network、tools 和 storage latency。

## 核心結論

Codex harness 的設計說明, 建立有效 agent 的工作重心正從「寫一個 model loop」轉向管理整個執行環境。模型負責推理, harness 則決定模型能看到什麼、能做什麼、如何安全行動, 以及任務能否跨越長時間與多個 context windows 延續。

對自建 agent 的團隊而言, 不必複製整套 Codex。更實際的做法是逐一採用這些結構性原則: progressive tool discovery、持久化 execution sessions、context-aware review、stateful transport、可驗證 goal 與 model-aware compaction。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕整理, 並非逐字稿。影片未提供公開章節, 本文段落與時間戳依演講內容轉折編排。口語贅詞、重複字幕、現場互動及非必要的 demo 操作已刪除。

演講描述的是當時的 Codex harness 與 Responses API 狀態。講者也明確提醒, 模型發布時 API 與 harness 可能快速變動。本文中的版本、模型能力、授權、協定與實作細節, 應以使用時的官方 repository 和 API 文件為準。自動字幕可能誤辨第三方專案名稱, 不確定處未延伸推論。
