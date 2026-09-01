# AI Research OS: 將萬筆 Notes 轉成可查詢、可演進的研究記憶

- 影片: [Turn 10,994 Notes Into Memory - Paul Iusztin, Decoding AI & Louis-François Bouchard, Towards AI](https://www.youtube.com/watch?v=ZRM_TfEZcIo)
- 頻道: AI Engineer
- 講者: Paul Iusztin, Decoding AI; Louis-François Bouchard, Towards AI
- 發布日期: 2026-06-26
- 片長: 39:32
- Video ID: `ZRM_TfEZcIo`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

影片展示一套以 plain files、agent skills 與 deep research algorithm 建立的 AI Research OS。它不是把整個 second brain 一次塞進 context window, 而是先從 Obsidian、Readwise、NotebookLM、GitHub、web links 等來源找出與專案相關的內容, 建立一份 scoped research wiki。

核心資料結構分成三層:

```text
raw/          immutable source snapshots
index.yaml    metadata、summary 與 file references
wiki/         LLM 產生的 concepts、entities、comparisons 與 notes
```

Agent 查詢時先讀取小型 index, 再按需讀 source wiki page, 只有資訊仍不足時才開啟完整 raw source。這種 progressive retrieval 以 filesystem references 控制 token 使用, 不依賴 vector database 或 knowledge graph。

影片的價值在於解釋系統為何從單一 research file 演進到 raw/index/wiki, 並展示 personal research、GitHub repository comparison 與 custom URLs 三種 ingestion。講者也明確承認系統仍缺乏完善的 source provenance、staleness ranking、memory compaction、connectors 與 end-user UX。

## 原始問題: 收集很多, 工作時卻找不到

[00:00](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=0s)

Paul 表示自己的 Obsidian 有超過 5,000 notes, Readwise 另有約 5,000 notes, 還有散落於 Notion 與 Google Drive 的資料, 每月增加約 250 files。影片標題使用 10,994 notes, 但沒有展示逐項盤點方法。

資料量本身不是成果。當他開始文章、project、codebase 或 feature 時, 真正需要的是從龐大 vault 中找出與當前任務相關的 high-signal notes。

他們描述的失敗模式包括:

- Reading list 成為很少重訪的 graveyard。
- 忘記曾儲存哪些 posts、articles、videos 或 repositories。
- 找有用筆記需要大量時間。
- 新 session 必須重新提供相同 links 與 context。
- 既有研究無法自然影響下一個 project。
- 個人觀點、價值與過去作品無法進入通用研究工具。

因此, Research OS 位於 agent harness 與 second brain 之間, 負責選擇、壓縮與組織當前專案需要的研究材料。

## 先判斷問題是否值得進入 Research OS

[03:31](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=211s)

講者沒有主張所有問題都要進入自建系統。建議依任務深度與重用需求選擇工具:

| 任務 | 較合適的起點 |
| --- | --- |
| 快速事實或一次性問題 | Google 或一般 chat assistant |
| 小型 repo change、單篇文章 | Codex、Claude Code 等現成 agent |
| 需要消化固定資料集 | NotebookLM 類工具 |
| Product-scale retrieval | Vector database、RAG infrastructure |
| 長期、個人化、可重用且 agent-native 的研究 | File-based Research OS |

NotebookLM 的優點是能消化與重訪來源, 但講者認為它的資料 ownership、personalization、agent integration 與 coding workflow 不符合自己的需求。

Vector database 與 RAG 適合 large-scale products, 但需要額外 infrastructure, 且不容易由人直接檢查及修改。對個人日常研究, 講者選擇更簡單、可瀏覽的 files。

這是針對講者 workflow 的設計取捨, 不是 vector retrieval 普遍無用的結論。

## 將分散來源搬到 Local Files

[09:16](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=556s)

Louis-François 原本把 skills、meeting recaps、Apple Notes 與 browser bookmarks 分散在不同服務。後來將資料集中到 Obsidian, 原因不是 Obsidian 特有的 AI 能力, 而是它提供跨裝置 UI, 同時讓資料保留為 local files。

他使用 Codex 建立自動化, 將 Granola meeting notes 等內容同步進 vault。Research OS repository 再透過 skills 與 plugins 讀取:

- Obsidian。
- Readwise。
- NotebookLM。
- GitHub repositories。
- YouTube videos。
- Web URLs 與 documents。

講者強調 repository 應由使用者依自身來源調整, 不是固定 connector 清單。

## 系統演進 V1: 單一 Research File

[12:48](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=768s)

第一版用於建立 agent engineering course lessons。輸入是一個 topic 與人工挑選的 golden links, 輸出是一份 static `research.md`。

Deep research algorithm 大致如下:

1. Scrape golden links, 作為初始 framing context。
2. Orchestrator 根據 topic 與 seed context 產生多個 questions。
3. Sub-agents 各自使用 Gemini grounded in Google 搜尋。
4. 每個 agent 回傳 links 與 executive summaries。
5. Orchestrator 壓縮並聚合結果, 避免 context 爆增。
6. 重複多個 query rounds。
7. 依 source 與 topic 的相關性 ranking。
8. 只 full-scrape top-k sources, 其餘保留 summaries。
9. 編譯成單一 `research.md`。

影片描述的完整設定為三 rounds、每 round 六 queries, 最終可能取得約 40 至 50 links。這些是 algorithm configuration 與講者經驗值, 不是 retrieval quality benchmark。

團隊使用此流程快速建立 35 lessons。影片沒有提供與人工研究的品質比較, 但這個版本足以暴露兩項限制:

- Golden links 必須人工尋找。
- Static research file 無法有效支援後續問題與增量更新。

## 系統演進 V2: 讓 Second Brain 成為 Seed Context

[13:28](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=808s)

第二版保留相同 query、ranking 與 summarization loop, 但將搜尋來源從 public web 擴展為:

```text
personal sources + public web
```

Topic 本身成為主要輸入。系統先查詢 second brain, 找到使用者曾保存、已初步篩選的材料, 再將它們當成 organic golden links。這讓研究結果更容易反映個人的過去閱讀、作品與觀點。

問題仍然存在: 最終輸出是一份 static file。若資料過時、讀者想追問, 或新來源出現, 系統可能需要重新執行昂貴的 deep research loop。

## 系統演進 V3: 加入 Living Wiki

[18:28](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1108s)

第三版不再把所有內容壓成單一檔案, 而是保存個別 sources, 建立 index, 再產生 wiki derivatives:

```text
sources
   ↓
deep research / ingestion
   ↓
raw source files
   ↓
index.yaml
   ↓
wiki pages
  ├─ sources
  ├─ concepts
  ├─ entities
  ├─ comparisons
  ├─ notes
  └─ open questions
```

Wiki 可增量更新。新來源、deep research round 或使用者問題都可能建立新的 concept、comparison 或 note, 不必每次重建全部研究。

## File-Based Index, 不先引入 Database

[20:13](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1213s)

講者刻意不在個人 Research OS 的第一版使用 vector database、knowledge graph、semantic index 或全文搜尋服務。系統以 `index.yaml` 作為 entry point, 每筆 source 保存:

- Raw file reference。
- Origin。
- Title 與 authors。
- Publication date。
- Compact summary。
- 對應 wiki pages。

Index 同時指向 raw sources 與 derived wiki pages。優點是人類與 agent 都能直接查看、diff、編輯及 version control。代價是資料量與查詢模式成長後, YAML scan、keyword routing 與 agent reasoning 可能成為 latency、recall 及一致性瓶頸。

影片沒有提供 file count 擴展測試, 因此「不需要 database」只適用於此 prototype 與目前使用規模。

## Raw、Index 與 Wiki 的責任邊界

[21:26](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1286s)

| Layer | 內容 | 修改規則 |
| --- | --- | --- |
| Raw | Article、paper、video transcript、repository analysis 等來源快照 | Immutable, agent 不應改寫 |
| Index | Metadata、summary、references 與 navigation | 可重建或增量更新 |
| Wiki | LLM 合成的 concepts、entities、comparisons、notes 與 unanswered questions | Derived content, 可持續演進 |

Raw layer 是 provenance anchor。Wiki 只是一種解讀與導航, 不應取代來源本身。若 agent summary 或 comparison 有問題, 使用者仍可回到 raw file 查核。

## 三段式 Query Strategy

[22:49](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1369s)

Agent 不直接掃描所有 raw files, 而是逐層增加閱讀成本:

```text
1. index.yaml
   metadata + compact summaries
        ↓ 資訊不足
2. source wiki page / derivatives
   expanded summaries + concepts + comparisons
        ↓ 仍不足
3. raw source
   complete article, transcript, code or document
```

這個 hierarchy 同時提供 progressive disclosure 與 token control。Index 適合 routing, wiki 適合回答已整理的問題, raw source 則保留細節與驗證能力。

影片沒有量測不同層級的 cache hit rate、token savings 或回答準確度。實作時應記錄 agent 在哪一層找到答案, 以及未讀 raw source 是否增加錯誤。

## Wiki 會因問題留下 Trace

每次 query 都寫入 log, 並可能建立新的 concept、comparison 或 note。Knowledge base 因而不只在 ingest 時更新, 也能反映使用者持續提出的問題。

這項能力同時有累積錯誤的風險。若 derived page 來源不足或推論錯誤, 後續 agent 可能把它當成既有知識再次引用。需要保存:

- Page creation query。
- Supporting source references。
- Agent、model 與 skill version。
- Last validated time。
- Superseded 或 disputed state。

以上 metadata 是依影片 architecture 延伸的編者建議, 影片沒有展示完整 schema。

## Global Second Brain 與 Project Wiki 分離

[25:01](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1501s)

Paul 使用 Tiago Forte 的 PARA method, 將 second brain 分為 Projects、Areas、Resources 與 Archive。個人 vault 是 global snapshot, LLM 不修改手寫 notes。

開始新 project 時, 系統才透過 deep research 選取相關 sources, 建立 scoped wiki:

```text
global second brain, immutable
          ↓ research and selection
project-specific raw snapshot
          ↓
project index + evolving wiki
          ↓
article, video, slides, course or codebase work
```

這個 boundary 避免一個 agent 直接重寫整個 personal knowledge base, 也限制每個 project 的 context scope。Project wiki 是可演進的工作記憶, global vault 則保留原始個人資料。

## Demo 1: Agent Engineering Research

[27:04](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1624s)

第一個 demo 以講者自己的 agent engineering notes 與預選 references 為 seed, 呼叫 research skill 建立 wiki。Skill 會詢問研究深度:

| Mode | 影片中的設定 |
| --- | --- |
| Light | 一 round, 三 queries |
| Fast | 兩 rounds, 每 round 三 queries |
| Deep | 更多 questions 與 rounds, 具體值未在字幕中完整說明 |

講者表示一次執行約需 10 至 20 分鐘, 且消耗大量 tokens, 因此多數情況使用 light 或 fast。

輸出包含 raw files、index 與 wiki。Wiki 自動建立 agent loop、context compaction、sandboxing 等 concepts, 以及 Agentic RAG vs. filesystem、compaction vs. recursive language models 等 comparisons。

影片展示生成結果, 沒有逐項查核每個 concept 或 comparison 的正確性。

## Demo 2: 比較 GitHub Harness Repositories

[31:58](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=1918s)

第二個 demo 不執行 public-web deep research, 而是 clone 三個 open-source harness repositories, 再依指定主題分析:

- General architecture。
- Agent and sub-agent architecture。
- Memory system。
- Permission flow。

系統先建立各 repository 的個別 notes, 再產生 cross-repository comparisons 與共同 concepts。這種流程適合形成 architecture map, 但 repository 會快速更新, snapshot commit、branch 與 ingestion time 必須保留, 否則比較結果難以重現。

影片字幕對部分 repository 名稱辨識不穩定。本筆記不嘗試在未檢查畫面的情況下修正完整名稱。

## Demo 3: Ingest Custom URLs

[34:25](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=2065s)

第三個 demo 只 ingest 三個 URLs, 不需要設定 Obsidian、Readwise 或 NotebookLM。使用者安裝 plugin 後, 透過 Git 與 `curl` 等基礎工具即可建立 raw/index/wiki。

Wiki 建立後, 使用者可以繼續提問。例如 demo 詢問 remote sandboxing 如何接入 harness。Agent 會查詢現有 wiki, 回答問題, 並可能將新的 comparison、concept 或 entity 寫回 knowledge base。

## 講者承認的未完成部分

[36:50](https://www.youtube.com/watch?v=ZRM_TfEZcIo&t=2210s)

講者明確將此系統定位為 builder workflow 與教學 repository, 不是 polished end-user product。已知限制包括:

- 缺少 Google Drive、Notion、Slack 等 connectors。
- 不容易判斷 source 是否 stale、weak 或 authoritative。
- 需要更強的 linting。
- Memory compaction 仍不成熟。
- Source provenance 與 ranking 需要改善。
- 主要從 Claude Code 或 Codex terminal workflow 操作, UX 尚未產品化。

影片最後介紹付費 Agent Engineering course。課程內容與時數屬於產品推廣, 不是本筆記保留 Research OS 架構的證據。

## 實作檢查表

- 問題是否真的需要長期重用, 還是一般 search 或 chat 已足夠?
- Global notes 是否保持 immutable, project wiki 是否獨立?
- Raw snapshot 是否記錄 source URL、日期、repository commit 與取得時間?
- Index 是否足夠小, 能作為 agent routing context?
- Summary 與 wiki page 是否都連回 raw source?
- Query 是否先走 index、再走 wiki、最後才讀 raw?
- 是否記錄每層 retrieval 的命中率、tokens 與 latency?
- Deep research depth 是否依任務價值與成本調整?
- Ranking 是否同時考慮 relevance、authority、freshness 與 source diversity?
- Derived pages 是否保存 creation query 與 supporting sources?
- 新問題寫回 wiki 前是否需要 confidence threshold 或 review?
- Stale、contradictory 與 superseded content 如何標記?
- Ingestion、index update 與 wiki generation 是否具 idempotency?
- Connector credentials 與 personal notes 是否限制在必要環境?
- System 是否能從 raw layer 完整重建 index 與 wiki?

## 核心結論

大量 notes 不會自動形成 memory。真正的工程問題是如何從 global archive 中選出與當前 project 有關的 sources, 以低成本建立可追溯、可追問且能增量演進的工作記憶。

這套 Research OS 的核心不是 Obsidian 或某個特定 model, 而是三個 boundary:

1. Global personal notes 保持 immutable。
2. Project research 以 raw snapshots 保存 provenance。
3. Index 與 wiki 作為可重建、可演進的 derived views。

File-based architecture 提供可檢查性與低 setup cost, 但不是免除 retrieval engineering。當資料規模、協作者或 query complexity 增長後, source ranking、staleness、contradiction handling、security 與 performance 仍需要正式設計。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。影片字幕對人名、repository 名稱與部分產品名稱有辨識誤差, 本筆記只修正能由影片 metadata 或上下文可靠確認的項目。

講者是 AI education company 的創辦人與課程作者, 並實際使用此 Research OS 進行影片、文章與課程研究。這提供第一手 prototype 經驗, 同時也存在推廣 repository 與付費課程的利益。

影片展示 architecture、generated files 與三種 demos, 但沒有公開 retrieval accuracy、source ranking evaluation、token cost、latency distribution、wiki hallucination rate、長期維護資料或與 NotebookLM、RAG systems 的受控比較。10,994 notes、每月 250 files、35 lessons 與 10 至 20 分鐘執行時間均為講者陳述, 未在影片中獨立驗證。
