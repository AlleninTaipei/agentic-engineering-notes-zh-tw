# Anthropic Applied AI: Agentic Surfaces 的演進與 Managed Agents 架構

> [!info] 來源
> - 影片: [Anthropic's Applied AI team on the Evolution of Agentic Surfaces](https://www.youtube.com/watch?v=K0X9QDRkIdg)
> - 頻道: AI Engineer
> - 發布日期: 2026-08-11
> - 片長: 31:23
> - Video ID: `K0X9QDRkIdg`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

隨著模型從回答問題進展到承擔完整成果, agent 開發的瓶頸逐漸由模型能力轉向 harness 與生產基礎設施。Anthropic 在影片中提出 managed agents, 以 agent、environment、session 三種核心資源, 加上「腦與手分離」架構, 統一處理執行隔離、可靠性、憑證、可觀測性與長期上下文。

## 從問題、任務到成果

講者將 agent 任務的演進分為三個層次 ([01:52](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=112s)):

1. 問題: 模型回答單次、相對簡單的問題。
2. 任務: 使用者把一段可執行工作委派給模型。
3. 成果: agent 持續採取行動, 直到完成使用者定義的結果。

任務越長、工具越多、狀態越複雜, 開發者需要管理的便不再只是 prompt 與回覆, 還包括執行環境、上下文、失敗恢復和安全邊界。

## 三代 agentic surface

### Messages API

最早的介面可概括為 tokens in、tokens out。當模型需要自行查資料、呼叫工具並管理逐漸增長的上下文時, 開發團隊會在 API 外自行實作 agentic loop ([02:45](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=165s))。

一個手工 loop 通常負責:

- 呼叫模型。
- 執行模型要求的工具。
- 將工具結果送回模型。
- 維護對話、狀態與上下文。
- 判斷何時結束或重試。

### Claude Agent SDK

Agent SDK 把 Claude Code 使用的 harness 包裝起來, 提供內建 agentic loop、檔案系統存取、工具與 sandbox 能力 ([04:29](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=269s))。它減少了 loop 的重複實作, 但團隊仍需自行處理部署、擴展、憑證與部分 session 基礎設施。

### Claude managed agents

Managed agents 再往上承擔生產環境基礎設施 ([05:20](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=320s))。影片將責任邊界描述為:

| 開發團隊負責 | Managed agents 負責 |
| --- | --- |
| 產品體驗 | Agent loop 與 harness |
| 任務定義 | 執行 sandbox |
| 業務上下文與領域知識 | Hosting 與 scaling |
| System prompt、skills、tools | Session、可觀測性與憑證基礎設施 |

這個劃分的目標不是讓開發者放棄控制, 而是讓團隊把時間放在能區分產品的領域邏輯。

## 生產級 agent 的六個基礎設施問題

影片列出六個由原型走向正式產品時無法迴避的問題:

1. Hosting 與 scaling: agent 在哪裡執行, process 存活多久, 高負載下如何擴展?
2. Session management: 歷史紀錄與進度存放在哪裡, 如何同時執行大量 agent?
3. File system: 模型如何建立、讀取與編輯檔案?
4. Execution isolation: 模型產生的程式在哪裡執行, 如何限制破壞範圍?
5. Credentials: agent 如何存取敏感系統, 又不直接看見 token?
6. Observability: 如何理解模型、工具與 orchestration 實際發生了什麼?

這些問題彼此相依。若 session 不持久, process 掛掉就無法恢復。若工具與憑證沒有隔離, 增加工具能力也會同步擴大安全風險。

## Harness 的假設會隨模型進步而過期

Harness 通常會編碼「模型目前做不到什麼」的假設, 例如何時壓縮上下文、何時重設、如何糾正特定行為。問題是模型更新後, 原本的補丁可能從必要措施變成負擔 ([07:02](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=422s))。

影片以「context anxiety」為例。講者表示, Sonnet 4.5 接近 context window 上限時會過早收尾, 因此 harness 加入 context reset。到了 Opus 4.5, 這項行為消失, 舊 reset 反而增加延遲, 並可能錯誤捨棄 cache ([07:51](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=471s))。

核心教訓是:

> 當模型前進而 harness 沒有同步演進, harness 會開始限制甚至降低 agent 表現。

因此架構要讓元件可替換、可個別更新, 並定期重新驗證針對舊模型加入的 workaround。

## 長時間執行需要哪些能力

長時間、非同步 agent 可能持續數小時或數天, 並同時處理多條工作流。影片列出幾項關鍵需求 ([10:25](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=625s)):

- Context engineering: 控制長期累積的資訊, 避免 context rot。
- 安全 sandbox: 允許 agent 採取行動, 但限制其影響範圍。
- 可靠性: 面對 process 或工具失敗仍能恢復。
- 平行化: 將複雜問題拆成可同時進行的部分。

## 核心設計: 把 brain 與 hands 分開

影片把 agent loop 與模型推理稱為 brain, 把工具執行環境稱為 hands。早期設計將兩者放在同一 container 中, 工具呼叫雖直接, 卻有兩個明顯問題:

- 模型必須等待 container 完成啟動, 才能開始推理。
- 任一元件故障, 整個 agent 都會一起中斷。

Managed agents 因此把兩者解耦 ([11:16](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=676s))。Brain 可先開始推理, 等確實需要工具時才即時建立 sandbox。Sandbox 故障時可重建後重試, brain 故障時則可從持久 session log 恢復。

這個分離也使 hands 可以部署在客戶自己的環境, 而 brain 維持在受管理的 agent loop 中。

## 三個核心 primitive

Managed agents 以三種資源形成架構 ([12:59](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=779s)):

| Primitive | 內容 | 主要用途 |
| --- | --- | --- |
| Agent | 模型、prompt、tools、skills | 定義 agent 是誰、能做什麼 |
| Environment | agent 的 container 與網路、執行限制 | 定義工作在哪裡發生及其邊界 |
| Session | Agent 與 Environment 的一次持久互動 | 保存事件、進度與恢復依據 |

同一 environment definition 可建立多個隔離的 container instance, 讓多個 session 同時執行而不共用實際執行空間。

## Session 的可靠性與四種狀態

Session 是雲端持久資源, 記錄 agent 的所有互動。影片定義四種狀態 ([13:51](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=831s)):

- `idle`: 等待使用者輸入。
- `running`: 正在執行。
- `rescheduling`: 遇到錯誤, 準備重試或重新安排。
- `terminated`: 發生無法恢復的狀況, 已終止。

明確狀態機能把「卡住」拆成可觀察、可操作的狀態, 並為重試策略提供界線。

## Session log 不等於目前的 context window

傳統 harness 容易把 session history 與模型眼前的 context window 視為同一件事。一旦內容被壓縮或捨棄, 模型就無法再取回。

Managed agents 把所有事件寫入持久 session log, 目前的 context 只是從 log 選取的一個工作切片 ([15:32](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=932s))。若模型之後需要先前被移出的資訊, harness 可以重新讀取對應片段。

這個區分可以寫成:

```text
session log = 持久、完整的事件來源
context window = 目前推理所需的暫時工作集合
```

它同時支援失敗恢復、可觀測性與後續記憶整理。

## SRE agent 示範

示範場景是某服務的 P99 latency 突然升至基準值十倍, 團隊建立 SRE Investigator agent 尋找根因 ([17:14](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1034s))。

建置流程如下:

1. 定義 agent: 指定模型、system prompt、Bash、Grep、blob 等工具, 並透過 MCP 連接監控 dashboard。
2. 定義 environment: 使用雲端 sandbox, 將允許連線的 host 限制為指定 MCP server。
3. 提供證據: 上傳 application logs 與所需 skills。
4. 建立 session: 組合 agent、environment 與 log resource, 啟動調查。

Agent 先搜尋應用程式 log, 再經 MCP 取得 metrics 與近期 deployment, 找到事件起點、檢查 code diff, 最後綜合證據提出 root cause。Console 中的 observability trace 則顯示每個模型訊息、工具呼叫、結果與 session 事件 ([20:38](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1238s))。

這個示範的重點不是 SRE prompt 本身, 而是相同定義能為大量使用者建立彼此隔離的 session, 且每次執行都有可追蹤紀錄。

## 四項實務教訓

### 1. 不讓 agent 看見憑證

影片介紹 vault 機制, 讓安全憑證只在工具執行時解密, 模型本身不會看到 token ([22:25](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1345s))。

這項原則比「在 prompt 中要求模型不要讀 `.env`」更可靠。安全邊界應由架構與執行層強制實施, 不能只依賴模型遵循文字要求。

### 2. 解耦能降低首 token 延遲

Brain 與 hands 放在同一 container 時, 模型必須等待 container setup。解耦後, 推理與環境準備可平行進行, 不需要工具的任務甚至可以略過 container ([23:15](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1395s))。

影片報告的內部測試結果為:

- P50 use case 的 time to first token 快 60%。
- P95 use case 的 time to first token 改善超過 90%。

這些數字是影片中的產品測試主張, 字幕未提供測試樣本、環境或完整方法, 不宜直接外推至其他系統。

### 3. Session log 同時支援觀測與記憶

Session log 逐項記錄 user message、model response、tool execution 與 result ([24:58](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1498s))。呈現在 UI 中時是 observability trace, 作為後續處理輸入時則能協助建立跨 session 的 memory。

### 4. Tool execution 應能留在私有環境

Self-hosted sandbox 允許企業在自己的 VPC 與安全政策下執行工具。MCP tunnel 則讓私有網路內的 MCP server 只需向外連線至 agent loop, 不必直接公開在網際網路上 ([25:47](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1547s))。

## Dreaming: 從 session log 整理長期記憶

影片將 dreaming 描述為週期性批次處理 ([27:28](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1648s)):

1. 收集過去一段時間的 session transcripts。
2. 與目前 memory state 一起交給模型分析。
3. 擷取新洞見並重新組織內容。
4. 更新 memory, 供下一次 session 使用。

其目標是讓 agent 從歷史執行中改善, 並逐漸形成更高層次的組織知識, 例如團隊 runbook。這仍需要治理機制, 包括來源追蹤、錯誤資訊修正、隱私範圍與記憶淘汰政策。

## Outcomes: 讓 agent 朝成功條件持續迭代

Outcomes 讓開發者定義成功 rubric 與 failure cases, 並啟動獨立 grader agent 評估主要 agent 是否完成任務 ([29:08](https://www.youtube.com/watch?v=K0X9QDRkIdg&t=1748s))。

```text
主要 agent 執行任務
        ↓
grader 依 rubric 評估
        ↓
達標 → 完成
未達標 → 繼續嘗試
```

這把停止條件從「模型說完成了」改為「評分器認為符合成功規格」。不過 grader 本身仍是模型, rubric 的完整性、評分偏差、成本與最大重試次數都必須由開發者設計。

## 可帶回實作的架構檢查表

- Agent loop 是否與工具執行環境解耦?
- Agent、environment 與 session 是否是可獨立管理的資源?
- Session 是否有明確、可觀察的狀態機?
- 完整 session log 是否獨立於目前 context window 持久保存?
- Sandbox 故障與 agent loop 故障是否能分別恢復?
- 憑證是否只在工具執行層注入, 而不進入模型上下文?
- 網路是否採 allowlist 或相等強度的限制?
- Tool execution 是否能部署至企業控制的環境?
- Harness 中針對舊模型的 workaround 是否有定期移除機制?
- 成功條件是否明確, grader 是否有重試上限與人工接管路徑?
- 從 session log 產生的記憶是否能追溯、修正與刪除?

## 時效性與限制

影片發布於 2026 年 8 月, 介紹當時 Anthropic managed agents 的架構與產品方向。功能名稱、可用區域、API、模型版本、定價與實際供應狀態可能變動, 導入前應查閱當前官方文件。

本筆記以英文自動字幕與 YouTube 章節為依據。字幕中的個別產品名稱和模型版本可能辨識不準, 因此未對不確定的示範模型版本作額外推斷。延遲改善數字與產品能力是講者在影片中的陳述, 並非本筆記的獨立測試結果。

## 結語

影片最重要的架構觀點是, 模型只是 agent 的 brain, 真正可上線的系統還需要受控的 hands、持久 session 與可演進的 harness。當模型快速增強時, 過度固定的 orchestration 反而會成為限制。較穩健的方向是保持元件鬆耦合, 將安全與可靠性落實在基礎設施, 並讓產品團隊專注於任務、工具與領域知識。
