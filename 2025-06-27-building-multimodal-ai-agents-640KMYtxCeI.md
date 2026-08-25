# 從零建立 Multimodal AI Agent

來源: [Building Multimodal AI Agents From Scratch, Apoorva Joshi, MongoDB](https://www.youtube.com/watch?v=640KMYtxCeI), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=640KMYtxCeI
- 上傳日期: 2025-06-27
- 片長: 36:58
- Video ID: `640KMYtxCeI`
- 講者: Apoorva Joshi, MongoDB
- 內容依據: 英文原始語言自動字幕

## 摘要

這場 workshop 從 agent 與 multimodality 的基本概念開始, 設計一個能回答 mixed-media documents 問題, 也能直接分析 chart 或 diagram 的 multimodal agent. 系統使用 multimodal embedding model 將 PDF pages 的完整畫面轉成 vectors, 透過 MongoDB Vector Search 找到相關頁面, 再把原始 page images 交給 Gemini 類 multimodal LLM reasoning.

影片最重要的設計選擇是, 不先把 page 中的 text, image 與 table 全部拆開. 每個 PDF page 先轉成 screenshot, 使用同一個 vision-language-model-based encoder 建立 embedding. 這可以保留頁面內文字與視覺元素的相對關係, 並簡化 extraction, summarization 與 embedding pipeline.

這種 page-as-image 方法也有明確限制. Page boundary 仍會失去跨頁 context, screenshot retrieval 可能降低文字精確搜尋能力, 高解析圖片增加 storage 與 inference cost, time series 或弱對齊 modalities 也不適合硬塞進同一 vector space. Production architecture 通常需要 metadata filters, neighboring-page expansion, hybrid retrieval 與 modality-specific tools.

## 什麼時候需要 Agent

[01:40](https://www.youtube.com/watch?v=640KMYtxCeI&t=100s)

講者將 AI agent 定義為一個使用 LLM 進行 reasoning, 建立 plan, 並透過 tools 執行與迭代的系統.

她比較三種 LLM application paradigms:

| 形式 | 能力 | 適用範圍 | 主要限制 |
| --- | --- | --- | --- |
| Simple prompting | 使用 model parametric knowledge 回答 | 一次性, 通用問答 | 無法取得模型外知識, personalization 有限 |
| RAG | 從 external data source 取得 context | Domain QA, grounded response | 流程通常預先定義, 不一定處理複雜多步任務 |
| Agent | 自行選擇 steps 與 tools, 根據結果迭代 | 不確定, 多步, adaptive tasks | Cost, latency 與 nondeterminism 更高 |

影片提醒, 不應因 agents 流行就把每個問題都改成 agent. 適合 agent 的訊號包括:

- 任務複雜且需要多步處理.
- Workflow 難以事先完整預測.
- 可以容忍較高 latency.
- 可以接受 non-deterministic behavior.
- 需要長期 personalization 或 adaptive behavior.

如果步驟固定且可用一般程式表達, deterministic pipeline 往往更便宜, 更快且更容易測試.

## Agent 的四個元件

[05:04](https://www.youtube.com/watch?v=640KMYtxCeI&t=304s)

### Perception

Agent 從 environment 接收資訊. Input 可以是 user message, email, Slack event, image, voice 或 video. 本 workshop 使用 text 與 image.

### Planning and Reasoning

LLM 根據 query, available tools 與 memory 決定下一步. 它可以只產生一次 plan, 也可以依 tool outcomes 反覆修正.

### Tools

Tools 是 agent 對外界採取 action 的 interfaces, 例如 APIs, vector stores, databases 或 specialized models. LLM 通常只選擇 tool 並產生 arguments, 真正執行 function 的仍是 application code.

### Memory

Memory 保存 conversation 與 past interactions. Workshop 實作 short-term conversation memory, 用 session ID 區分不同 conversations.

```text
Perception -> LLM plans -> Select tool -> Application executes
     ^                                         |
     |                                         v
  Memory <- Store interaction <- Observe tool result
```

## Planning Without Feedback

[06:43](https://www.youtube.com/watch?v=640KMYtxCeI&t=403s)

最簡單的 planning 是讓 LLM 根據初始理解產生 plan, 之後不依 tool result 或 reasoning trace 更新. 影片以 chain-of-thought 作為例子, 可用 zero-shot instruction 要求逐步思考, 或用 few-shot examples 示範推理方式.

講者是在解釋設計 pattern, 不是要求 application 暴露 private reasoning traces. Production system 應保存可觀察的 plan, actions, tool results 與 decision summary, 不必依賴完整隱藏思維內容. 這是編輯補充.

## Planning With Feedback, ReAct

[07:57](https://www.youtube.com/watch?v=640KMYtxCeI&t=477s)

ReAct 代表 reasoning and acting. Agent 交替進行:

```text
Reason about current state
  -> Choose action
  -> Execute tool
  -> Observe result
  -> Update plan
  -> Repeat or answer
```

每個 observation 都可能讓 agent 修改原本策略. 當 LLM 判斷已有足夠 evidence, 才離開 loop 並產生 final answer.

這個 loop 必須由 harness 限制 max steps, timeout, tool permissions 與 termination conditions, 否則錯誤 plan 可能造成無限循環或不必要成本.

## Tool Calling 的責任邊界

[09:03](https://www.youtube.com/watch?v=640KMYtxCeI&t=543s)

Tool 通常包含兩部分:

1. 真正執行工作的 function.
2. 提供給模型的 schema.

Schema 至少描述 tool name, purpose, parameters, types 與 parameter descriptions. 模型根據 schema 判斷是否呼叫, 並產生 structured arguments.

```json
{
  "name": "search_documents",
  "description": "Retrieve relevant document pages for a question.",
  "parameters": {
    "query": {
      "type": "string",
      "description": "The user question used for retrieval."
    }
  }
}
```

這是依影片概念整理的範例, 不是 workshop 原始 schema. Application 必須驗證 arguments, 執行 function, 捕捉 errors, 再把 result 交回 LLM. 模型本身不直接執行 Python function.

## Short-Term 與 Long-Term Memory

[10:36](https://www.youtube.com/watch?v=640KMYtxCeI&t=636s)

影片使用兩類簡化分類:

- Short-term memory, 保存同一 conversation 中的 messages 與 state.
- Long-term memory, 跨 conversations 保存與更新 information, 支援長期 personalization.

Workshop 只實作 short-term memory. 這個範圍足以支援 follow-up questions, pronoun references 與同一 session 內的多輪討論, 但不包含 preference learning, cross-session profile 或 durable episodic memory.

## Simple Tool-Calling Agent Flow

[11:49](https://www.youtube.com/watch?v=640KMYtxCeI&t=709s)

影片先用 weather query 建立完整直覺:

1. User query 進入 agent.
2. Agent 將 query, tools 與 memory 提供給 LLM.
3. LLM 選擇 weather API 並抽取 city argument.
4. Application code 執行 API.
5. Tool result 回到 LLM.
6. LLM 判斷是否還需要其他 tools.
7. 資訊足夠後生成 natural-language answer.

Multimodal agent 延續相同 control flow, 差別是 perception, retrieval results 與 LLM inputs 可以包含 images.

## Multimodality 是什麼

[13:10](https://www.youtube.com/watch?v=640KMYtxCeI&t=790s)

Multimodality 指 model 能處理, 理解或生成多種 data types, 例如 text, image, audio 與 video. Workshop 聚焦 text 和 images.

Mixed-media data 常見於:

- Research papers.
- Financial and organizational reports.
- Charts and tables.
- Technical diagrams.
- Healthcare documents.
- Scanned or visually structured PDFs.

這些文件的意義常存在於 layout 和 modalities 的關係. 若只抽出文字, 可能失去 legend 與 chart 的對應, table cells 的位置, 或 caption 指向哪個 figure.

## 兩類 Multimodal Models

[14:04](https://www.youtube.com/watch?v=640KMYtxCeI&t=844s)

### Multimodal Embedding Model

接收 text, image 或其他 modalities, 產生可供 retrieval 使用的 vector representation. 目標是讓語意相關的跨模態 items 在 vector space 中接近.

### Multimodal LLM

接收 text 與 images 等 inputs, 理解 retrieved visual context, 並產生 answer 或 tool decision. 一個系統可以使用 embedding model 做 retrieval, 再由 multimodal LLM 做 reasoning.

```text
Multimodal embedding model
  -> Find relevant pages

Multimodal LLM
  -> Interpret pages and answer
```

將 multimodal LLM 配上可搜尋 multimodal data 的 tools 與 agent loop, 就形成 workshop 中的 multimodal agent.

## Agent 的兩個目標

[15:06](https://www.youtube.com/watch?v=640KMYtxCeI&t=906s)

Workshop 設定兩項能力:

1. 回答大型 document corpus 中的問題.
2. 對使用者直接提供的 chart 或 diagram 進行說明與分析.

第二種任務可能不需要 retrieval. 如果 image 已隨 query 提供, LLM 可以直接分析. 第一種任務則要先搜尋 mixed-media corpus.

## 傳統 Mixed-Media Processing Pipeline

[16:12](https://www.youtube.com/watch?v=640KMYtxCeI&t=972s)

一種常見方法是先對 document layout 做 element extraction:

```text
Document
  -> Detect text, images and tables
  -> Chunk text
  -> Summarize images and tables into text
  -> Embed all text and summaries
  -> Retrieve
```

另一種方法保留 images 和 tables, 使用 multimodal embedding model 分別建立 embeddings:

```text
Document
  -> Detect text, images and tables
  -> Chunk text
  -> Embed each modality
  -> Retrieve across modalities
```

這兩種 pipeline 都需要 layout detection, extraction, chunking 與多個 model calls. 第一種還會在 image-to-text summary 中遺失 visual details.

## Chunking 與 Modality Gap

[18:27](https://www.youtube.com/watch?v=640KMYtxCeI&t=1107s)

### Chunk Boundary Context Loss

固定 text chunks 可能把相關 paragraph, caption, table 或 figure 切開. Parent-document retrieval 與 metadata pre-filtering 可以補回部分 context, 但同時增加 pipeline complexity.

### CLIP-Style Modality Gap

影片說明, 早期常見 multimodal embeddings 採用 text encoder 與 image encoder 的雙塔架構. 即使共同訓練, 不同 modalities 仍可能形成分離 clusters, 造成不相關 text 彼此比相關 text-image pair 更接近的情況.

這是影片對 modality gap 的高層解釋. 實際表現取決於 model architecture, training objective 與 benchmark, 不能推論所有 dual-encoder models 都不適用.

## VLM-Based Unified Encoder

[19:48](https://www.youtube.com/watch?v=640KMYtxCeI&t=1188s)

影片提出使用 vision-language-model-based embedding architecture. Text 與 visual features 經由更統一的 encoder 表示, 讓跨模態 context 能在同一 representation 中保留.

Workshop 因此採取 page-as-image 方法:

```text
PDF page
  -> Render complete page as image
  -> Multimodal embedding
  -> Store vector + page metadata
```

不論 page 是純文字, 純圖片, table 或混合 layout, 都使用同一條 embedding path. 這顯著減少 element extraction 與 summarization steps.

## Page-as-Image Data Preparation

[21:58](https://www.youtube.com/watch?v=640KMYtxCeI&t=1318s)

Workshop pipeline:

1. 將每份 PDF 逐頁 render 成 screenshots.
2. 將 screenshots 存在 local disk, production 可改用 S3 或其他 blob storage.
3. 記錄 image path / object URI 與 page metadata.
4. 使用 Voyage AI 的 multimodal embedding model 建立 page embedding.
5. 將 embedding 與 metadata 存入 MongoDB vector collection.

Vector database 不保存 raw screenshot, 只保存 reference 與 embedding. Retrieval 後, agent 再依 URI 從 blob storage 載入真正 image.

概念性 document schema:

```json
{
  "document_id": "report-2025",
  "page_number": 12,
  "image_uri": "s3://bucket/report-2025/page-0012.png",
  "embedding": [0.12, -0.04, 0.31],
  "metadata": {
    "title": "Annual Report",
    "section": "Financial Results"
  }
}
```

這是依影片架構整理的 schema, 不是 workshop repository 的逐字內容.

## 為什麼不用完整 PDF 直接進 Context

[24:03](https://www.youtube.com/watch?v=640KMYtxCeI&t=1443s)

現場問答提出, 為何不把 full PDF 直接交給 multimodal LLM. 講者的回答是, large context window 不代表應把整份文件塞入. Model 仍可能出現 lost-in-the-middle, 且每次處理完整 PDF 的 cost 和 latency 更高.

Retrieval 先縮小候選 pages, 再把少量高相關 visuals 交給 LLM, 可以控制 context budget. 不過 retrieval miss 會讓 LLM 永遠看不到正確 page, 所以必須評估 end-to-end recall.

## 跨頁 Context 與 Overlap

[23:06](https://www.youtube.com/watch?v=640KMYtxCeI&t=1386s)

一頁仍是一種 chunk boundary. Paragraph, table 或 argument 可能跨頁. 現場討論提出幾種補救方式:

- Render overlapping page regions.
- 保存 page number 與 document structure metadata.
- 命中一頁後擴展 previous / next pages.
- 使用 parent-document retrieval.
- 先用 metadata filter 限定 document 或 section.

```text
Retrieve page N
  -> Expand to N-1, N, N+1
  -> Rerank with original query
  -> Fit selected pages into context budget
```

固定擴展前後兩頁會增加 token 和 image cost. Production system 可依 score, section boundary 與 document type 動態決定 neighborhood.

## 不同 Modalities 不一定適合同一 Vector Space

[26:58](https://www.youtube.com/watch?v=640KMYtxCeI&t=1618s)

現場問到 time-series data 這類與 text / image 弱對齊的 modality. 講者建議使用不同 retrieval strategy, 不要強迫所有 data 進同一 embedding space.

例如:

| Data type | 較合適的 retrieval |
| --- | --- |
| Page image + text layout | Multimodal vector search |
| Exact identifier | Structured lookup |
| Time series | Time range, numeric filters, feature query |
| Tables | SQL / structured query 或 table-aware retrieval |
| Entity relations | Graph query |
| Keyword-heavy legal terms | Full-text or hybrid search |

Agent 可以擁有多個 modality-specific tools, 由 router 或 LLM 依 query 選擇. Multimodal 不等於 single-retriever-for-everything.

## Multimodal Agent Workflow

[29:02](https://www.youtube.com/watch?v=640KMYtxCeI&t=1742s)

完整 query flow 如下:

```text
User query + optional image + session ID
  -> Load conversation history
  -> Multimodal LLM decides whether retrieval is needed
      -> No: analyze supplied image directly
      -> Yes: call vector_search(query)
          -> Embed query
          -> Retrieve page metadata and image URIs
          -> Load screenshots from blob storage
  -> Send query + history + retrieved images to multimodal LLM
  -> Generate answer
  -> Store user query and response in session history
```

LLM 每次可能收到 original query, chat history 與 images. 若需要 retrieval, tool result 先提供 image references, application 再載入 binary images 並轉成 model API 需要的格式.

## Vector Search Tool

[29:46](https://www.youtube.com/watch?v=640KMYtxCeI&t=1786s)

Agent 在 workshop 中只有一個主要 tool, vector search. Tool 的責任應保持清楚:

```text
Input:
  Natural-language query

Process:
  Create query embedding
  Search vector index
  Apply metadata filters if present

Output:
  Ranked page IDs
  Scores
  Image URIs
  Document and page metadata
```

Tool 不應直接假設 top-k 全都正確. Orchestrator 可設定 minimum score, deduplicate pages, expand neighbors, rerank, 並限制總 image / token budget.

## Session-Based Short-Term Memory

[31:33](https://www.youtube.com/watch?v=640KMYtxCeI&t=1893s)

每個 user query 帶有 session ID. Agent 使用 session ID 查詢 database, 取得該 conversation 的 previous turns. LLM 回答後, 再把 current query 與 response 寫回同一 session.

```text
session_id
  -> Load ordered messages
  -> Add to model context
  -> Execute tool loop
  -> Store user message
  -> Store assistant response
```

最低限度保存 user query 與 assistant response. 更完整的 observability 也可以記錄:

- Tool calls and arguments.
- Retrieval results and scores.
- Selected image URIs.
- Tool errors.
- Model and prompt versions.
- Final citations.

若保存 reasoning traces, 應考量 model provider policy, privacy 與是否真的有必要. 更安全的做法是記錄 concise decision summary 和 observable actions.

## Lab 的實際範圍

[33:50](https://www.youtube.com/watch?v=640KMYtxCeI&t=2030s)

講者最後引導現場參與者前往 GitHub repository, 提供兩個 notebooks:

- `lab.ipynb`, 依 inline reference documentation 自行填寫程式碼.
- `solutions.ipynb`, 已填入完整 code, 可逐段執行與閱讀 comments.

YouTube 錄影在 hands-on portion 開始時結束, 字幕沒有包含後續 45 分鐘的逐步 coding 與執行結果. 因此本筆記只重建影片中明確說明的 architecture 與 data flow, 不推測 repository 中未展示的 API calls 或 credentials setup.

## Production 強化清單

### Data ingestion

- 保留 source document, page number, section 與 image URI.
- 選擇適合 OCR 與 visual detail 的 render resolution.
- 對跨頁內容使用 neighborhood expansion 或 overlap.
- 為 text-heavy pages 同時保存 extracted text, 支援 hybrid search.
- 版本化 embeddings 與 source documents.

### Retrieval

- 建立 text-only, image-only 與 mixed-page eval sets.
- 分別量測 page recall, document recall 與 final-answer quality.
- 使用 metadata pre-filtering 限定 tenant, document type 與 date.
- 對候選 pages rerank 並去重.
- 對 time series, tables 或 exact lookup 使用專門 tools.

### Agent loop

- 限制 max tool calls, timeout 與 image context budget.
- 驗證 tool arguments 和 image URIs.
- Tool failure 時回傳結構化 error, 不讓模型猜測內容.
- 簡單 direct-image query 不必強制 retrieval.
- 最終回答附 page-level citations 或 evidence.

### Memory

- 以 session ID 做 tenant-safe partitioning.
- 對 history 設定 retention 與 compaction policy.
- 不把所有舊 images 每輪重新傳入.
- 保存 durable task state, 不只 raw transcript.
- 敏感文件與 conversation memory 使用 access control 和 encryption.

### Cost and latency

- 預先計算 corpus embeddings.
- Cache frequently retrieved pages.
- 限制 screenshot resolution 與 top-k.
- 比較 page-as-image 與 hybrid text / visual pipeline.
- 量測 retrieval, blob loading 與 multimodal inference 各階段 latency.

## 核心觀念回顧

```text
Multimodal RAG agent
├── Ingestion
│   ├── PDF pages -> screenshots
│   ├── Blob storage
│   └── Multimodal embeddings + metadata
├── Retrieval
│   ├── Query embedding
│   ├── Vector and metadata search
│   ├── Neighbor expansion
│   └── Image loading
├── Reasoning
│   ├── Multimodal LLM
│   ├── Tool decision
│   └── ReAct loop
└── Memory
    ├── Session ID
    ├── Conversation history
    └── Persist current turn
```

最值得帶走的三個原則:

1. Multimodal retrieval 與 multimodal reasoning 是兩個不同職責, 可以使用不同 models.
2. Page-as-image 簡化 mixed-media ingestion, 但仍需要跨頁 context, hybrid retrieval 與 storage cost 的補強.
3. Agent 只適合需要動態 tool choice 與 multi-step reasoning 的部分, 固定 ingestion 和 function execution 應保持 deterministic.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理. 影片沒有 YouTube chapters, 章節與時間戳由內容結構編排. 自動字幕可能誤辨講者姓名, model 名稱與 API terms. 本筆記只在上下文足以確認時修正.

Workshop 使用當時的 Gemini multimodal model, Voyage AI multimodal embeddings 與 MongoDB Vector Search. 具體 model versions, experimental endpoints, SDK 與產品功能可能已改變. 影片帶有 MongoDB workshop 與產品示範背景, 沒有比較其他 vector databases 或完整 benchmark.

錄影內容停在 hands-on lab 開始, 未包含 notebooks 的完整實作過程. 本筆記中的 JSON, pseudocode 與 production checklist 是依影片架構整理的編輯說明, 不是 workshop repository 原始碼的逐字複製.
