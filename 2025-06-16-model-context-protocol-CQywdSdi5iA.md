# Anthropic MCP 共同建立者談協定起源、開源決策與下一步

- 影片: [The Model Context Protocol (MCP)](https://www.youtube.com/watch?v=CQywdSdi5iA)
- 頻道: Anthropic
- 主持人: Alex Albert
- 來賓: Theo Chu、David Soria Parra
- 發布日期: 2025-06-16
- 片長: 19:35
- Video ID: `CQywdSdi5iA`
- 內容依據: YouTube 英文創作者字幕 (`en`)

## 摘要

這支影片由 Anthropic MCP 團隊說明 Model Context Protocol 的設計目的、內部起源、開源理由、2025 年 6 月的採用狀態及近期 roadmap。它不是完整規格教學, 但提供協定建立者對問題定義與設計取捨的第一手說明。

MCP 要標準化的不是一般 API, 而是 AI application 如何把外部能力與資料交給 model。Server 當時主要公開三種 primitives: tools、resources 與 prompts。這層共同介面讓 server builder 不必為每個 AI application 重做 integration, client 也能接入多種資料來源與動作。

MCP 最初源於 Anthropic 工程師在 Claude Desktop、IDE 與內部資料之間反覆複製貼上的摩擦。2024 年 9 月左右的內部 hackathon 成為早期驗證: 參與者自發將 Slack、3D printer 等專案做成 MCP servers。團隊於 2024 年 11 月發布並開源 MCP, 希望降低 integration builders 對單一 AI product 的綁定風險, 讓產品競爭回到 model intelligence 與 workflow。

截至錄製時, 團隊描述生態系已從 local developer tools 走向 remote MCP 與 cloud-hosted servers。未來重點包括 security primitives、registry API、long-running tasks 與 elicitation。這些是當時的 roadmap, 不應直接視為目前已完成的功能。

## MCP 標準化的是 Model 所需介面

[00:35](https://www.youtube.com/watch?v=CQywdSdi5iA&t=35s)

David Soria Parra 將 MCP 簡化為一種把 workflow 和 context 接入 AI application 的方法。它可以傳遞可執行動作、原始資料或其他 model 所需的內容。

MCP 和直接呼叫 API 的差異在於抽象層級。Model 不直接理解任意 API contract, 而是接收 prompts、tools 與 context。MCP 將不同 API 或內部資料來源轉成 AI application 與 model 可使用的共同形式。

影片列出 server 可公開的三種核心 primitives:

- Tools: Model 可呼叫的動作。
- Resources: Files、data 或其他原始 context, 可交給 RAG pipeline 或直接使用。
- Prompts: 由使用者觸發並可編輯的 prompt templates, 在 application 中常呈現為 slash commands。

可以把責任邊界理解為:

```text
external API or internal data
              ↓
          MCP server
  tools + resources + prompts
              ↓
        MCP-aware client
              ↓
             model
```

影片是概念介紹, 沒有涵蓋 transports、message lifecycle、capability negotiation、authorization 或完整 client/server architecture。實作時仍需以對應版本的正式 specification 為準。

## 起點是跨工具 Copy and Paste

[03:13](https://www.youtube.com/watch?v=CQywdSdi5iA&t=193s)

David 表示, MCP 最初源於自己開發 Anthropic 內部工具時的挫折。他需要在 Claude Desktop、IDE 與工作資料之間來回複製內容, 因此開始思考如何讓兩個 applications 直接交換自己在意的 context。

他向另一位共同建立者 Justin Spahr-Summers 說明構想後, 兩人一起擴充設計並整合進 Claude Desktop。這個起源顯示 MCP 最初處理的是具體 workflow friction, 而不是先從「建立通用網路協定」的抽象目標出發。

2024 年 9 月左右, Anthropic 舉辦內部 hackathon。團隊沒有要求參與者使用 MCP, 但許多人自發把想法做成 MCP servers, 範圍從 Slack integration 到控制 3D printer。

這次 hackathon 提供兩項早期訊號:

1. Client 一旦支援 MCP, server builder 只需專注 integration 的一側。
2. 開發者能在數分鐘內讓 Claude 獲得新能力, 形成直接而明顯的 feedback loop。

這是 Anthropic 內部的定性觀察, 不是 controlled developer study。影片沒有提供參與人數、完成率、開發時間分布或與其他 integration methods 的比較。

## 2024 年 11 月發布後的 Adoption Flywheel

[06:22](https://www.youtube.com/watch?v=CQywdSdi5iA&t=382s)

Anthropic 在 2024 年 11 月, 約美國 Thanksgiving 前後發布 MCP。團隊表示初期反應較慢, 許多外部與內部使用者都先問「MCP 是什麼」。實際試用後, 採用才逐漸增加。

團隊觀察到的採用順序是:

```text
IDEs and early clients adopt MCP
              ↓
more servers become useful across clients
              ↓
model providers adopt the protocol
              ↓
server providers gain stronger incentive to build
```

Alex Albert 在訪談中稱 MCP 已成為 integration protocol 的 industry standard。這是 Anthropic 主持人的判斷, 並非獨立市場調查。影片沒有定義 market share、活躍 client 數量或相容性程度, 因此不宜把這句話當成精確的產業統計。

## 為什麼必須是 Open Standard

[08:12](https://www.youtube.com/watch?v=CQywdSdi5iA&t=492s)

團隊認為, closed integration ecosystem 會迫使 server builders 判斷每個 AI application 是否長期存在, 以及值得投資哪一套專用整合。Open standard 降低這種 product-specific risk, 也讓同一個 server 有機會服務多個 clients。

Anthropic 在影片中提出的價值判斷是: AI application 的主要差異不應是擁有多少獨占 integrations, 而應是 model intelligence 與建立在 model 上的 workflow。因此, integration layer 適合成為共同基礎設施。

Open source 也允許社群直接修正 servers、documentation 與 protocol implementation。Theo Chu 舉例, 訪談拍攝當天就有人提交 pull request 更新過期的文件圖片。

這個決策帶來的工程含義是:

- Server builders 可降低對單一 client vendor 的依賴。
- Clients 可共享 server ecosystem, 但仍能在 model 與 UX 上競爭。
- Protocol 與文件可以接受跨公司的需求與修正。
- 相容性、安全性和治理問題也會成為共同責任, 不會因 open source 自動解決。

## 從 Local MCP 走向 Remote MCP

[09:55](https://www.youtube.com/watch?v=CQywdSdi5iA&t=595s)

Theo 表示, MCP 早期主要服務 developers, client 與 server 都在本機執行。到 2025 年 6 月, 生態系開始轉向 cloud-hosted remote MCP servers。使用者可以把提供 MCP endpoint 的網站接入日常 Claude workflow, 不再需要每個 integration 都在本機啟動 process。

團隊在錄影時提到 10,000-plus server builders, 但沒有說明計算口徑、活躍程度或資料來源。這個數字只能視為 Anthropic 對當時 ecosystem 規模的概略描述。

Remote MCP 擴大了使用情境, 同時引入 local prototype 不一定需要處理的 production concerns:

- Identity 與 authorization。
- Credential storage 與 rotation。
- User consent 與 delegated permissions。
- Tenant isolation。
- Network reliability、timeouts 與 retries。
- Tool output 的 provenance 與 audit trail。
- Malicious or compromised servers。

Theo 表示團隊正和企業及領域專家討論 identity、authorization 和 specification evolution。影片沒有提供完整 security architecture, 因此不能從「支援 remote MCP」推論某個 server 已適合 production。

## 不要從複雜 Server 開始

[13:23](https://www.youtube.com/watch?v=CQywdSdi5iA&t=803s)

團隊建議新手先使用一個既有 server, 觀察它在 Claude AI 或 Claude Desktop 中的 interaction pattern, 再建立自己的 MCP server。

第一個實作可只涵蓋最小 primitives:

1. 建立只回傳 `Hello world` 的單一 tool。
2. 分別嘗試一個 resource 與 prompt。
3. 在 local environment 驗證 client 如何列出和呼叫它們。
4. 閱讀品質良好的現有 servers, 再逐步修改。

David 也建議先讓 Claude Code 協助建立 local server。Alex 描述自己把 MCP 官方文件交給 Claude Code 後, 工具自行取得內容並產生 server。這是個人 demo 經驗, 不代表 generated implementation 已經通過 security、error handling 或 production readiness 審查。

## Physical Tools 顯示 Protocol 的通用性

[14:57](https://www.youtube.com/watch?v=CQywdSdi5iA&t=897s)

David 特別喜歡連接 physical world 的 MCP servers, 例如控制 synthesizer、door 與 3D printer。Blender integration 則讓 Claude 透過 tools 寫入 Blender scripts, 動態建立 scene。

這些案例證明相同 interface 可以包裝非常不同的能力, 但也揭示 tool use 的風險會隨 side effect 增加:

```text
read-only data lookup
        ↓ lower consequence
modify digital artifact
        ↓
send external communication
        ↓
control a physical device
        ↓ higher consequence
```

Protocol interoperability 不等於行為安全。涉及 physical devices 或外部副作用時, client 仍需要權限限制、參數驗證、human confirmation、audit logging、rate limits 與 emergency stop。

以上安全控制是依案例做的編者整理, 並非影片展示的 MCP built-in guarantees。

## Claude 4 與更長時間的 Agent Tasks

[16:23](https://www.youtube.com/watch?v=CQywdSdi5iA&t=983s)

Theo 認為, 更高能力且能處理較長任務的 models, 會增加 MCP 中 statefulness 和 sampling 等早期 primitives 的使用。David 補充, model 也可能更擅長從更多 attached servers 中選擇正確工具。

可掛載多少 servers 沒有固定答案。主要限制不是 server 數量本身, 而是 tools 是否語意重疊。三個 issue-tracker servers 若提供名稱和用途相似的 tools, model 容易混淆。彼此用途明顯不同的 tools 則較容易共存。

這裡可推導出 server design 的實作原則:

- Tool name 要穩定且能區分類似能力。
- Description 應清楚說明適用情境與不適用情境。
- Input schema 應減少模糊參數。
- 相似 integrations 應揭露 system 或 tenant boundary。
- 用實際 model evals 測量 selection accuracy, 不只人工閱讀 schema。
- 不因 context window 變大就無限制暴露所有 tools。

這些是依訪談問題整理的工程建議。影片沒有提供 Claude 4 的 tool-selection benchmark 或可支援 server 上限。

## Roadmap: Discovery、Long-Running Tasks 與 Elicitation

[18:18](https://www.youtube.com/watch?v=CQywdSdi5iA&t=1098s)

團隊表示 protocol 已發布並取得採用, 下一階段會改善 examples、documentation 與 security primitives。影片特別提到三項 agent-oriented 工作:

### Registry API

Registry API 的目標是讓 model 搜尋額外 servers, 再按需求將能力帶入當前工作。Client 不必事先把所有 integrations 固定放進 context。

```text
agent encounters a missing capability
              ↓
searches a server registry
              ↓
evaluates and connects a suitable server
              ↓
uses the new capability in its task loop
```

Dynamic discovery 同時會帶來 trust、ranking、publisher identity、version compatibility 與 supply-chain security 問題。影片只描述方向, 沒有展開治理機制。

### Long-Running Tasks

團隊希望 MCP 更容易承載長時間工作。這通常需要清楚的 task identity、progress reporting、cancellation、reconnection、partial results 和 failure recovery。影片沒有說明當時預定的具體 protocol shape。

### Elicitation

Elicitation 讓 server 在缺少必要資料時反向向使用者詢問。它使 server 不必猜測關鍵輸入, 但 client 需要呈現問題並控制哪些資訊可回傳。

這三項在影片中都是 2025 年 6 月的 upcoming work。閱讀本筆記時, 應以目前 MCP specification 和 SDK release notes 確認它們是否已發布、改名或更改設計。

## Production 實作檢查表

### Server Contract

- 每個 tool 是否只有一個清楚責任?
- Tool name 與 description 能否和相似 servers 區分?
- Input/output schema 是否有明確 constraints?
- Read operation 與 mutating operation 是否分開?
- Resources 是否標示來源、更新時間與 sensitivity?
- Prompts 是否允許使用者在送出前查看與編輯?

### Security 與 Operations

- Local 與 remote deployment 的 trust boundary 是否分開建模?
- Authentication、authorization 與 user consent 是否各自處理?
- Credentials 是否避免進入 model context 和 logs?
- 高風險 actions 是否要求 confirmation 或 policy approval?
- 是否保留 tool call、result、actor 與 server version 的 audit trail?
- Timeout、retry、idempotency、cancellation 與 partial failure 是否定義?
- Third-party server 是否有 publisher verification 和 revocation path?

### Model Interaction

- 是否以真實 tasks 評估 tool selection, 而非只確認 server 能連線?
- 多個 tools 重疊時, selection error rate 是否可接受?
- Tool result 是否可能包含 prompt injection 或不可信 instructions?
- Client 是否只暴露當前任務必要的能力?
- Model 選錯 tool 時, 是否有低成本 recovery path?

## 核心結論

這場對談最值得保留的是 MCP 建立者對協定責任邊界的說明。MCP 不取代 API, 而是在外部 systems 與 AI applications 之間建立共同的 model-facing interface。

它的早期成功來自三個條件:

1. 解決了跨 Claude Desktop、IDE 與資料來源反覆複製貼上的具體痛點。
2. Client 和 server 透過標準介面解耦, 降低每組產品都要重新整合的成本。
3. Open standard 降低 builders 對單一 vendor 的投資風險, 促成 clients 與 servers 彼此增加價值。

影片也顯示協定普及後的新瓶頸。Remote deployment 需要 identity 和 authorization, 大量 tools 需要更好的描述與選擇 evals, dynamic discovery 需要 registry trust, long-running agents 需要 lifecycle controls。MCP 提供 interoperable plumbing, 但 production reliability 仍取決於 client、server、model 與營運治理的共同設計。

## 時效性與限制

本筆記依 Anthropic 官方 YouTube 頻道提供的英文創作者字幕與九個正式 chapters 整理。字幕內容可清楚辨識主要術語, 但影片沒有附逐項引用的 specification 版本或外部統計來源。

Theo Chu 與 David Soria Parra 是 Anthropic MCP 團隊成員, David 在影片中自述為共同建立者。因此, 關於內部起源、hackathon、launch 與設計動機屬於直接參與者的一手說明。關於 adoption、10,000-plus builders、industry-standard status 與未來影響則主要是 Anthropic 團隊自述, 缺少獨立數據或比較研究。

影片發布於 2025-06-16。Remote MCP、Claude integrations、Claude 4 能力、registry API、security primitives、long-running tasks 與 elicitation 都可能在後續版本改變。實作時應重新查閱目前 MCP specification、SDK documentation 與所用 client 的安全模型。
