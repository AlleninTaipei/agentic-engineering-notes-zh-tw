# Agentic Search 與 Context Engineering: 如何設計 Agent 的搜尋工具組

- 影片: [Agentic Search for Context Engineering: Leonie Monigatti, Elastic](https://www.youtube.com/watch?v=ynJyIKwjonM)
- 頻道: AI Engineer
- 講者: Leonie Monigatti, Elastic
- 發布日期: 2026-05-08
- 片長: 1:03:12
- Video ID: `ynJyIKwjonM`
- 內容依據: YouTube 英文原始自動字幕 (`en`)

## 摘要

Context engineering 不只是整理已取得的內容, 更重要的問題是 agent 如何從檔案、skills、database、web 與 memory 中找到應放進 context window 的資訊。講者將這個選擇與取得過程稱為 agentic search, 並主張搜尋工具是 context sources 與 context window 之間容易被低估的核心元件。

影片透過 conference schedule 資料展示三種搜尋方式:

1. 專用 semantic search tool, 容易使用但能力範圍窄。
2. 通用 database query tool, 能處理 filter、aggregation 與精確查詢, 但要求模型理解 query language。
3. Shell tool 搭配 `grep` 或 semantic-search CLI, 介面通用且能漸進擴充, 但需要 sandbox、清楚指示與成本控制。

實務上不應尋找單一萬用工具。比較可靠的設計是同時提供低使用門檻的專用工具與能力上限較高的通用工具, 再根據 execution traces 找出反覆出現的搜尋行為, 將它們逐步產品化為專用介面。

## Context Engineering 的核心其實是搜尋

[00:51](https://www.youtube.com/watch?v=ynJyIKwjonM&t=51s)

Context engineering 要從所有可能的來源中, 決定哪些內容應進入有限的 context window。常見討論集中在 context curation, 但 curation 之前仍需要一個搜尋層完成:

```text
context sources
      ↓
search tools / retrieval strategy
      ↓
selected context
      ↓
LLM context window
```

講者用「context engineering 約有 80% 是 agentic search」表達個人觀點。這不是經過實驗驗證的比例, 而是強調搜尋層的重要性。

## 從固定 RAG 到 Agentic Search

[02:21](https://www.youtube.com/watch?v=ynJyIKwjonM&t=141s)

傳統 RAG 使用固定 retrieval pipeline。系統通常直接把 user message 當成查詢, 先取得一批 chunks, 再將問題與結果一起送進模型。這種方式有兩個主要限制:

- 問題不需要外部資訊時, 系統仍會 retrieval, 不相關內容可能干擾模型。
- 複雜問題需要 multi-hop retrieval 時, 單次查詢無法根據第一批結果產生下一個問題。

Agentic RAG 將 retrieval 包成工具。Agent 可以判斷是否需要搜尋、選擇查詢內容、檢查結果是否充分, 再決定是否改寫查詢或繼續搜尋。

Context engineering 又比單一 database 的 agentic RAG 更廣。可能的 context sources 包含:

| 來源 | 常見搜尋介面 |
| --- | --- |
| Local files | file search、`grep`、shell commands |
| Agent skills | skill loader、filesystem |
| Database | semantic search、SQL 或其他 query language |
| Web | web search、HTTP client、CLI |
| Long-term memory | memory retrieval tool |

## Shell Tool 為何具有吸引力

[06:30](https://www.youtube.com/watch?v=ynJyIKwjonM&t=390s)

不同框架將 shell access 稱為 shell、Bash 或 exec tool。它讓 agent 能執行 terminal commands, 因而可以用相同介面操作多種 context sources:

- 以 `ls`、`find` 或 `grep` 探索 local files。
- 呼叫 database CLI。
- 撰寫短 script 連接資料來源。
- 使用 `curl` 存取 HTTP API。
- 安裝或呼叫專用 search CLI。

通用介面能降低預先設計工具的需求, 但不代表 shell 可以取代所有 search techniques。Keyword search、dense embeddings、sparse embeddings、multi-vector retrieval 與不同 indexing strategies 各有適用資料、準確度及 latency 條件。

此外, shell access 允許 agent 修改或刪除檔案及執行任意程式。講者明確建議放在 sandbox 中, 並指出 demo 使用的 LangChain shell tool 預設沒有完整 safeguards。

## Agentic Search 的三種常見失敗

[08:50](https://www.youtube.com/watch?v=ynJyIKwjonM&t=530s)

Elastic 協助內外部團隊建立 database agents 時, 觀察到搜尋流程可能在多個位置失敗。影片聚焦三種情況:

1. Agent 認為 parametric knowledge 已足夠, 完全不呼叫 retrieval tool。
2. Agent 有多個工具時選錯來源, 例如使用 web search 而不是內部 database。
3. Agent 選對工具, 卻無法產生正確的 filters、query syntax 或其他 parameters。

這些錯誤不能只靠增加模型能力解決。工具的描述、參數複雜度、錯誤回饋與可用文件都會影響行為。

## Tool Description 是 Routing Interface

[10:41](https://www.youtube.com/watch?v=ynJyIKwjonM&t=641s)

Tool description 不只是說明文字, 它同時影響模型何時選擇工具。若 tool calling 不穩定, 可以逐步補充:

- Core purpose: 工具解決什麼問題。
- Trigger conditions: 何時應使用。
- Exclusions: 哪些問題不應使用。
- Relationships: 是否必須先載入 skill、取得確認或呼叫另一個工具。
- Parameter semantics: 欄位格式、允許值與限制。

當 description 已充分但 routing 仍失敗時, 可以在 system prompt 中再次強化規則。不過, 將每個錯誤都修補進 system prompt 會持續增加 context 成本, 並使規則難以維護。

參數越接近簡單識別碼或自然語言 query, 模型越容易正確產生。讓模型從頭撰寫 SQL 或 ES|QL 則提供更多能力, 同時提高 syntax、schema 與語意錯誤的機率。

## Demo 1: Semantic Search 的低門檻與窄能力

[13:53](https://www.youtube.com/watch?v=ynJyIKwjonM&t=833s)

第一個 demo 將 AI Engineer conference sessions 切成 chunks 並存入 Elasticsearch。`title` 與 `description` 被放入 embedding field, 日期、時間、房間及講者則作為 metadata。

LangChain tool 接受自然語言 query, 執行 similarity search, 固定回傳 top 3。對「哪些 sessions 討論 AI systems 的 regulatory constraints」這類語意問題, agent 能搜尋、改寫 query 並找出相關場次。

但工具能力被固定為 vector similarity 與 top 3。當問題要求尋找包含特定縮寫的 session 時, semantic search 回傳 Gemma models、harness engineering 等表面相近但實際無關的結果, 沒有找到真正包含該縮寫的場次。

這個失敗顯示, semantic search 並非 keyword search、filter 或 aggregation 的替代品。Demo 成功只代表預先挑選的語意問題可被處理, 不代表工具適用所有搜尋需求。

## Demo 2: 通用 Database Query 與自我修正

[23:26](https://www.youtube.com/watch?v=ynJyIKwjonM&t=1406s)

第二個 demo 將專用 semantic search tool 改成通用 ES|QL execution tool。Agent 可以自行產生完整 query, 因而使用 keyword matching、filters、sorting、aggregation 與其他 database capabilities。

Tool implementation 使用 `try/except` 捕捉 query error, 再將錯誤訊息回傳 agent。這讓模型可以修改 query, 而不是讓整個 agent loop 因一次 syntax error 終止。

實際 demo 中, agent 沿用了 SQL 的 `%` wildcard, 但 ES|QL 需要 `*`, 因此第一次查詢沒有正確結果。這不是 retrieval engine 的問題, 而是模型缺少 query-language knowledge。

## 用 Agent Skill 提供按需文件

[28:28](https://www.youtube.com/watch?v=ynJyIKwjonM&t=1708s)

若每遇到一個 query error 就把修正規則加入 system prompt, 最後可能把大部分 query-language documentation 都常駐在 context 中。影片改用 skill loading pattern:

1. System prompt 只保留 skill 的名稱、用途與位置。
2. Agent 要使用 ES|QL tool 時, 先讀取相關 skill。
3. Skill 提供 syntax、wildcards、filters、aggregation 與 examples。
4. Agent 根據文件產生或修正 query。

加入 skill 後, agent 能用正確 wildcard 找到縮寫, 也能建立包含日期 filter 與統計計算的查詢。讓 database 執行 counting 或 aggregation, 也避免要求 LLM 自己計數大量 records, 並減少傳入 context window 的原始資料。

這裡的 skill 不是唯一修正方式。單一規則可以直接加入 prompt 或由程式正規化, 但當知識量持續增加時, 按需載入的文件更容易維護。

## Demo 3: 使用 Shell 搜尋 Local Files

[34:42](https://www.youtube.com/watch?v=ynJyIKwjonM&t=2082s)

第三個 demo 把每場 conference session 存成獨立 local file, 再讓 agent 透過 shell 探索目錄。對精確縮寫查詢, agent 先列出 folder structure, 接著使用 `grep`, 最後讀取命中的完整檔案。

對「handling regulatory constraints」這種語意問題, `grep` 沒有 semantic understanding。Agent 嘗試自行展開 `regulation`、`compliance`、`GDPR`、`governance` 等同義詞, 最後找到結果。

這種做法在小型資料集上可能成功, 但存在明顯限制:

- 需要多次 tool calls, latency 與 token cost 較高。
- 同義詞列舉不可能完整。
- 資料量增加後, 大量掃描可能變慢。
- 每輪結果都可能占用 context window。

Agent 看起來在做 semantic search, 實際上是反覆產生 keyword variants。兩者的 recall 與擴展性不能視為等價。

## 用 CLI 擴充 Shell Tool

[41:26](https://www.youtube.com/watch?v=ynJyIKwjonM&t=2486s)

Shell 的另一項價值是可以呼叫既有 CLI, 不必為每個 backend 建立新的 agent protocol。影片安裝 Jina CLI, 並在 system prompt 說明:

- Exact matching 使用 `grep`。
- Semantic 或 fuzzy search 使用 `jina grep`。
- 提供 CLI syntax 與 examples。

Agent 隨後能在第一次 semantic query 找到 regulatory constraints 相關 session。這顯示通用 shell 介面可以承載專用 retrieval capability, 但模型仍需要知道 CLI 存在、適用時機與正確用法。

## 搜尋工具組的設計原則

[44:42](https://www.youtube.com/watch?v=ynJyIKwjonM&t=2682s)

講者借用 UX 中 low floor 與 high ceiling 的概念描述搜尋工具組:

| 工具類型 | 優點 | 代價 |
| --- | --- | --- |
| 專用工具 | 參數簡單、錯誤少、tool calls 少、小模型也可能使用 | 只能處理預期行為, 新需求容易超出能力 |
| 通用工具 | 能處理複雜及未預期查詢, 可利用 backend 完整能力 | 需要較強模型、更多文件與迭代, 錯誤率及 latency 較高 |

可靠的工具組應同時具有:

- Low floor: 常見操作有簡單、穩定且高效率的專用介面。
- High ceiling: 罕見或複雜問題仍可透過 shell 或 general-purpose query tool 解決。

若尚不了解使用者的 query patterns, 可以先提供通用工具並完整記錄 traces。當某類問題反覆需要 4 至 5 次 tool calls, 或 agent 經常重複相同 query construction, 就表示值得建立 purpose-built tool。

```text
先提供 general-purpose tool
          ↓
記錄 tool calls、errors 與 retries
          ↓
辨認重複且高成本的 query patterns
          ↓
建立 specialized interface
          ↓
持續以 traces 驗證是否降低錯誤與成本
```

## Q&A 中的重要補充

### 模型能力與工具複雜度

[49:30](https://www.youtube.com/watch?v=ynJyIKwjonM&t=2970s)

講者表示, Elastic 的內部測試觀察到較強模型能大幅降低 parameter errors, 但她沒有記得確切數字。這項說法只能視為未量化的實務觀察。即使模型較強, 通用工具仍不會因此消除所有錯誤。

### 固定 RAG 不需要全面改成 Agentic RAG

對 latency 敏感的簡單查詢, 固定 RAG 仍可能有效。講者沒有提出如何可靠地在 fixed RAG 與 agentic RAG 之間 routing, 並明確表示當下沒有成熟答案。因此影片不能作為全面遷移到 agentic retrieval 的依據。

### Hybrid Tools 可能提高準確度

講者引用一篇外部實驗, 表示同時擁有 Bash 與 database tool 的 agent, 在特定 analytical queries 中先用 database 搜尋, 再用 shell 驗證結果, 準確度高於只使用單一工具的版本。影片沒有提供原始數據或完整實驗設定, 實作前應回到原始研究驗證。

### Irrelevant Results 仍有長期成本

模型通常能從單次 top-k results 排除不相關項目, 但這些結果若長期留在 conversation context, 仍可能增加干擾。Threshold、reranking、結果壓縮與 context lifecycle 應依實際任務評估, 不能只依賴模型事後判斷。

### Skill 的 Progressive Disclosure

[01:00:30](https://www.youtube.com/watch?v=ynJyIKwjonM&t=3630s)

Q&A 討論長時間 session 載入多個 skills 後如何控制 context。回答者建議只預先暴露 skill name、description 與檔案位置, 使用時才載入完整內容, 任務推進後再透過 context management 或 compaction 移出。完整資料仍留在 filesystem, 需要時可重新搜尋。

## 實作檢查表

- 每個 context source 是否有清楚的搜尋與授權介面?
- Tool description 是否說明 purpose、trigger、exclusion 與其他工具的關係?
- Tool parameters 是否超出目前模型穩定產生的複雜度?
- Query errors 是否回到 agent, 讓它能安全自我修正?
- 精確字詞、semantic similarity、filters 與 aggregation 是否使用合適的 backend capability?
- 常見查詢是否有低成本的 specialized tool?
- 未預期查詢是否保留 general-purpose fallback?
- Shell 是否在 sandbox 中執行, 並限制 filesystem、network、credentials 與 destructive commands?
- 是否記錄每次 tool selection、query、error、retry、latency 與結果品質?
- 是否根據 traces 將高頻且高成本行為提升為 purpose-built interface?
- Retrieval results 是否經 threshold、reranking 或壓縮, 避免無關內容長期污染 context?
- Skills 與大型文件是否按需載入, 而不是全部常駐 system prompt?

## 核心結論

Agentic search 的主要挑戰不是把搜尋 API 包成 tool, 而是讓 agent 在正確時機選擇正確來源, 產生有效 query, 判斷結果是否充分, 並在失敗時修正。

專用工具提供低錯誤率與低 latency, 通用工具提供解決未知問題的能力。兩者不是互斥選擇。比較務實的演進路徑是從可觀察的通用介面開始, 以真實 traces 找出瓶頸, 再建立專用工具降低成本與不確定性。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理, 專有名詞可能有辨識誤差。影片中的程式碼、投影片與 repository 沒有在本筆記中逐行驗證。

Workshop 使用 Elasticsearch、ES|QL、LangChain、OpenAI models 與 Jina CLI 作為示範。相關 API、模型名稱及工具行為可能隨版本改變, 實作時應查閱目前的官方文件。

講者提供的內部測試觀察沒有附數字與 evaluation protocol。Q&A 提及的外部 hybrid-tool benchmark 也未在影片內展示原始資料。因此這些內容適合形成待驗證假說, 不宜直接視為一般化結論。
