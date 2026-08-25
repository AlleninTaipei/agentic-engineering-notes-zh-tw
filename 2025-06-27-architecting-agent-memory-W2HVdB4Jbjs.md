# 設計 Agent Memory, 原則、Patterns 與實務方法

來源: [Architecting Agent Memory: Principles, Patterns, and Best Practices, Richmond Alake, MongoDB](https://www.youtube.com/watch?v=W2HVdB4Jbjs), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=W2HVdB4Jbjs
- 上傳日期: 2025-06-27
- 片長: 17:36
- Video ID: `W2HVdB4Jbjs`
- 講者: Richmond Alake, MongoDB
- 內容依據: 英文原始語言自動字幕

## 摘要

Agent memory 的目的不是把所有歷史資料塞入 context window, 而是讓系統能累積 state, 從過往 interaction 與 execution 中選出相關資訊, 並用它改善下一步行為. 講者將有效 agent 的目標歸納為 believable, reliable 與 capable, 並認為 memory 是支援 reflection, personalization, recovery 與 autonomy 的關鍵基礎.

影片借用 human memory 的 short-term, working, semantic, episodic 與 procedural 等概念, 再對應到 agent 中的 conversation, persona, entity, workflow, tool 與 long-term memory. 這些名稱是設計隱喻, 能幫助團隊拆分責任, 但不代表 AI 系統與人腦使用相同機制.

工程上, memory management 是一個完整 lifecycle: generation, storage, retrieval, integration, updating 與 forgetting. Retrieval 最重要, 因為 context window 再大也不應無限制填入資料. 系統需要以 recency, relevance, association 與任務需求選出少量記憶, 再結構化地放進當前 context.

## 從 Stateless Application 到 Stateful Agent

[00:00](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=0s)

早期 LLM application 多半是 stateless chatbot. 每次 request 主要依賴當前 prompt, 缺乏持久化 history, identity 與 execution experience. RAG 隨後讓 chatbot 可以取得 domain-specific knowledge, 但 retrieval knowledge 不等於完整 agent memory.

隨著 models 具備 reasoning 與 tool use, 系統開始形成不同程度的 agentic behavior. 講者不把 agent 定義成單一門檻, 而視為 spectrum:

```text
Minimal agent
  -> LLM in a loop
  -> Tool-using agent
  -> Multi-step agent
  -> Multi-agent system
  -> Highly autonomous agent
```

無論 autonomy 程度如何, agent 都需要某種 short-term 或 long-term memory, 才能把 observation, action 與 outcome 連接起來.

## Agent 的基本構成

[02:18](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=138s)

影片將 AI agent 描述為一個 computational entity, 具有:

- Perception, 了解 environment 的目前狀態.
- Cognition, 由 LLM 或其他推理元件決定下一步.
- Action, 透過 tools 改變 environment.
- Memory, 保存短期 state 與長期 experience.

```text
Environment
  -> Perception
  -> Cognition
  -> Action
  -> Outcome
  -> Memory
  -> Next cognition step
```

Memory 使 agent 不必每一步都從零開始. 它可以記住使用者偏好, 已完成工作, tool failures, environment changes 與成功策略.

## 為什麼 Agent 需要 Memory

[03:21](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=201s)

講者希望 memory 支援四種能力:

### Reflective

Agent 能回顧過去 action 與 outcome, 找出有效或失敗的策略.

### Interactive

對話不只是獨立 responses, 而能延續使用者的目標, 偏好與先前決策.

### Proactive and reactive

Agent 能根據累積 state 主動提醒, 也能在新 observation 出現時採取符合歷史脈絡的反應.

### Autonomous

長流程中, agent 能保存進度, 避免重複工作, 並在中斷後恢復.

Memory 本身不保證上述能力. 如果 retrieval 取回錯誤或過時資訊, memory 也可能讓 agent 穩定地犯錯. 所以 memory quality 與 lifecycle governance 和容量同樣重要.

## Human Memory 是設計隱喻

[04:22](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=262s)

影片列出人類常見的記憶分類:

- Short-term memory.
- Working memory.
- Semantic memory.
- Episodic memory.
- Procedural memory.

例如, procedural memory 可用來類比熟練的動作與技能, episodic memory 則可類比特定事件與經驗. Agent architecture 可以借用這些名稱, 將不同資料的用途, retention 與 retrieval policy 分開.

但這只是一種 conceptual framework. 生物記憶分散於複雜神經系統, Agent memory 則由 databases, files, embeddings, summaries 與 application logic 實作. 兩者不能做一對一的機制對應.

## Agent Memory 的定義

[05:32](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=332s)

講者將 agent memory 描述為一組讓 state 持久化的機制. 系統把 data 轉成可供未來使用的 memory, 並讓它影響下一次 execution.

這個定義包含三個層次:

```text
Raw event
  -> Memory formation
  -> Stored representation
  -> Retrieval and selection
  -> Context integration
  -> Changed agent behavior
```

只有保存 log 還不算完成 memory system. 若資料永遠不會被正確找回, 或取回後不會改變 decision, 它只是 archive.

## Memory Management Lifecycle

[06:20](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=380s)

影片列出 memory management 的主要 components:

| 階段 | 核心問題 |
| --- | --- |
| Generation | 哪些 event 應被轉成 memory? |
| Storage | 以什麼 schema, location 與 precision 保存? |
| Retrieval | 當前任務需要哪些 memories? |
| Integration | 如何放入 context 或 control flow? |
| Updating | 新資訊如何修正舊記憶? |
| Deletion / forgetting | 何時降低權重, 封存或移除? |

講者特別指出 forgetting mechanism. 人類不會像刪除 database row 一樣精確刪掉記憶, 但 agent system 可以模擬遺忘, 例如降低 retrieval score, 將資料移入 cold storage, 設定 expiration 或在矛盾資訊中優先採用較新且可靠的來源.

Production system 仍需要真正 deletion, 尤其涉及 privacy, retention policy 或使用者刪除要求時. "Forgetting" 不能取代合規刪除. 這是根據影片概念所做的編輯補充.

## Large Context 不等於 Memory Strategy

[06:03](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=363s)

Context window 擴大, 不代表應把所有資料一次放入. 過多 context 會增加成本, latency, distraction 與 conflicting evidence. Memory manager 的工作是選擇相關資訊, 並以適合模型理解的結構整合.

```text
Persistent store != Context window

Persistent store
  -> Search and filtering
  -> Ranking
  -> Compression
  -> Context assembly
  -> LLM
```

這也說明 memory 與 prompt engineering 的關係. Prompt 不需要重複攜帶所有歷史, 但 harness 必須知道要檢索什麼, 如何排序, 以及哪些 constraints 永遠不可遺失.

## Retrieval 是 Memory 的核心

[07:10](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=430s)

影片以 RAG 說明 retrieval. 基本 RAG 在 generation 前搜尋外部 knowledge, agentic RAG 則把 retrieval 暴露為 tool, 讓 agent 判斷何時需要資料.

講者強調, retrieval 不只等於 vector search. 不同 memory types 可能需要:

- Semantic vector search.
- Full-text search.
- Metadata filters.
- Graph traversal.
- Time and recency queries.
- Exact key lookup.
- Geospatial or structured queries.
- Hybrid ranking.

例如, 尋找與目前問題語意相近的事件適合 vector search, 取得特定 conversation ID 應使用 exact query, 找出最近一次失敗則適合 timestamp filter. 全部用 embeddings 可能犧牲 precision 與可解釋性.

## Memory Components

[08:33](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=513s)

影片展示多種可組合的 memory components. 每一種都應有自己的 schema, retrieval condition 與 update policy.

### Persona Memory

保存 agent 的角色, 語氣, boundaries, relationship style 或對特定使用者的互動偏好. 它的目標是讓體驗更一致與 believable.

Persona 不應變成未經同意的心理側寫. 使用者偏好應可查看, 修正與刪除, 並區分使用者明確提供的資料與模型推測. 這是編輯補充.

### Toolbox Memory

將 tools 的 names, descriptions 與 JSON schemas 存在外部 registry, 不必把所有工具定義一次放入 context. 執行前先依 task 找出最相關的工具集合, 再提供給模型.

```text
Task
  -> Retrieve relevant tool schemas
  -> Load small tool subset
  -> LLM selects tool
  -> Execute under permissions
```

這能降低 tool schema context load, 支援更大的工具生態. 但錯誤 retrieval 可能讓模型看不到必要工具, 所以需要 invocation evals 與 fallback.

影片提到 OpenAI 對 context 中工具數量的建議範圍, 但該數字具有版本與產品時效性, 本筆記不將它視為固定規則.

### Conversation Memory

保存 messages, timestamps, conversation ID 與其他 retrieval signals. 它支援 multi-turn continuity, 但不應永久保存所有 raw dialogue 並每次全部載入.

可將 conversation data 分層:

- Recent raw turns, 保留精確局部脈絡.
- Structured state, 保存 goals, decisions 與 unresolved questions.
- Long-term summary, 濃縮較舊內容.
- Archived transcript, 供 audit 或按需查找.

### Workflow Memory

保存 agent execution steps, tools, outcomes, errors 與 recovery. 任務失敗不是只有 exception, 也是可供下一次決策使用的 experience.

例如, 某一 tool sequence 在特定前置條件下失敗, 下一次 execution 可以先取回該 episode, 避免走相同路徑或改用其他策略.

### Episodic Memory

表示特定時間發生的事件, 包含 situation, action, outcome 與可能的 lesson. Episodic memory 適合支援 case-based reasoning 與 reflection.

### Entity Memory

保存人物, 公司, 專案或其他 entities 的穩定 attributes 與關係. 它需要 entity resolution, 否則同一實體可能被重複建立, 或不同實體被錯誤合併.

### Agent Registry

保存 agent identity, capabilities, tools, persona 與 routing metadata. Multi-agent system 可以根據 registry 找到適合的 agent, 而不必把所有角色定義載入同一 context.

## Memory Signals

[10:44](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=644s)

講者在 conversation memory schema 中示範 recency, recall 與 associated conversation ID 等 signals. 一個可用的 retrieval score 可以綜合:

```text
score = semantic relevance
      + recency
      + importance
      + entity or task association
      + source confidence
      - staleness
      - contradiction penalty
```

這是依影片概念整理的公式, 不是講者提供的固定演算法. 權重應由 domain evals 校準.

Memory signal 的目的不是讓每個 event 永久維持相同重要性. 系統可以隨時間衰減一般事件, 保留高價值決策, 並在多次成功 retrieval 後提升某類記憶的 utility estimate.

## Memory Modes

影片以 short-term 與 long-term 作為高層分類, 並提到不同 operation strategies. 可將常見 modes 整理為:

| Mode | 儲存內容 | 使用時機 |
| --- | --- | --- |
| Working memory | 當前任務 state, recent observations | 每個 execution step |
| Short-term memory | 最近 conversation 與 session events | 同一 session 或短期恢復 |
| Long-term memory | 穩定 facts, preferences, episodes | 跨 session personalized behavior |
| Dynamic memory | 依 relevance 即時組合多種來源 | 每次 task-specific context assembly |

這些 modes 不一定需要不同 databases. 重點是 retention, ranking, mutability 與 access policy 不同.

## Database 作為 Memory Provider

[07:45](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=465s)

講者以 MongoDB 說明 memory provider. Document model 可保存不同 memory schemas, 並透過 structured query, text, vector, graph 或其他 retrieval 方法取得資料.

在通用架構中, memory provider 應提供:

- Flexible but governed schemas.
- Transactional updates where needed.
- Metadata and provenance.
- Multiple retrieval methods.
- Indexing and ranking.
- Access control and tenant isolation.
- Retention, deletion and audit.
- Scale and availability.

選擇 MongoDB, relational database, vector database, graph database 或組合式架構, 應取決於 memory types 與 query patterns. 影片有明確 MongoDB 產品立場, 但通用原則是不應先選單一 retrieval 技術, 再把所有記憶問題強迫轉成它能處理的形狀.

## Memoriz 實驗性 Library

[09:02](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=542s)

講者介紹自己開發的實驗性, 教育用途 memory library, 字幕名稱近似 `Memoriz`. 它整理 persona, toolbox, conversation, workflow, episodic, long-term, entity 與 agent registry 等 design patterns, 並示範如何用 MongoDB document 表示.

講者明確稱它為 experimental and educational. 因此這些 schemas 適合用來理解 memory decomposition, 不應在未評估 consistency, security, migration 與 operational behavior 前直接視為 production standard.

自動字幕對 library 拼法不一致, 本筆記保留近似名稱並標示不確定性.

## Failure 是可保存的 Experience

[11:27](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=687s)

Workflow 執行到某一步失敗時, 可保存:

- Goal 與環境條件.
- 執行過的 steps.
- Tool inputs and outputs.
- Error type.
- Recovery attempts.
- Final outcome.
- 經驗適用範圍.

下一次執行相似任務時, agent 可以先 retrieval 相關 failure episode, 再選擇其他 path.

但 raw failure log 不等於可靠 lesson. 如果根因尚未確認, 將模型的推測直接存為長期規則可能污染 memory. 建議區分:

```text
Observed fact
  -> Tool returned status X

Hypothesis
  -> Agent thinks cause was Y

Verified lesson
  -> Human or test confirmed fix Z
```

這項分層是根據影片 workflow memory 概念所做的編輯補充.

## 沒有單一 Memory 解法

[12:41](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=761s)

影片提到當時的 memory-management tools, 並強調 agent memory 沒有唯一解法. 不同 use cases 可能更重視:

- Personalized conversation.
- Durable workflow state.
- Long-term factual knowledge.
- Tool discovery.
- Multi-agent coordination.
- Compliance and audit.
- Low-latency online retrieval.

因此, memory platform 應允許組織建立自訂 lifecycle 與 policies, 而不是假設一套 summary-plus-vector-search 能解決所有問題.

## Embeddings, Rerankers 與 Integrated Retrieval

[13:18](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=798s)

後半段介紹 MongoDB 收購 Voyage AI, 並談到 embeddings 與 rerankers 整合進 database / retrieval stack 的方向. 這部分主要是產品介紹.

通用技術角色可以分成:

```text
Candidate retrieval
  -> Vector, text, metadata, graph
  -> Broad set of possible memories

Reranking
  -> More expensive relevance model
  -> Smaller ordered set

Context assembly
  -> Deduplicate, compress, add provenance
  -> Fit token and policy budget
```

Embeddings 與 rerankers 可以改善 retrieval relevance, 但無法單獨消除 hallucination. Hallucination 還涉及 model behavior, insufficient evidence, conflicting sources, prompt / harness design 與 verification. 因此, 影片中 "reduce hallucination" 應理解為降低某些 retrieval-related failure, 不是完整保證.

產品整合狀態與功能可能在影片發布後改變, 實際使用時應查閱最新官方文件.

## 從 Neuroscience 取得靈感

[15:17](https://www.youtube.com/watch?v=W2HVdB4Jbjs&t=917s)

講者最後引用 Hubel 與 Wiesel 對視覺皮質的研究, 說明 neuroscience 曾啟發 computer vision 中的 hierarchical representations. 他藉此主張, nature 是現有 intelligence architecture 的重要靈感來源, agent memory 也可以從 cognitive science 與 neuroscience 借用問題分類.

影片字幕對研究者姓名與 Nobel Prize 年代有誤辨或口語誤差, 本筆記不保留不可靠的日期. 更重要的限制是, 從生物系統取得靈感不等於直接複製. 工程設計仍要接受 latency, cost, privacy, consistency 與 evals 的檢驗.

## Production Memory Architecture

以下是依影片原則整理的概念架構:

```text
Events
  -> Conversation messages
  -> Tool calls
  -> Workflow outcomes
  -> User feedback
  -> Environment changes

Memory formation
  -> Validate source
  -> Classify memory type
  -> Extract structured fields
  -> Attach provenance and confidence

Persistent memory provider
  -> Working state
  -> Conversation
  -> Entities
  -> Episodes
  -> Procedures
  -> Tool and agent registries

Retrieval
  -> Exact filters
  -> Text / vector / graph search
  -> Recency and importance
  -> Reranking

Context integration
  -> Deduplicate
  -> Resolve conflicts
  -> Compress
  -> Enforce token and policy budget

Lifecycle
  -> Update
  -> Decay
  -> Archive
  -> Forget
  -> Delete when required
```

## Memory Design Checklist

### Purpose

- 這項 memory 要改善哪個可觀察 behavior?
- 它是 task state, user preference, factual knowledge, episode 還是 procedure?
- 沒有這項 memory 時, agent 具體會如何失敗?

### Formation

- 哪些 events 值得保存?
- Fact, inference 與 hypothesis 是否分開?
- 是否保存 source, timestamp, confidence 與 tenant?
- 是否需要 human confirmation 才能進入 long-term memory?

### Storage

- Schema 是否符合 retrieval query?
- 是否需要 structured, vector, graph 或 hybrid index?
- Raw event, summary 與 verified lesson 是否分層?
- Encryption, access control 與 regional policy 是否完整?

### Retrieval

- Trigger 是每次自動 retrieval, 還是 agent tool call?
- Ranking 使用哪些 signals?
- 如何避免 stale, duplicated 或 contradictory memories?
- Retrieval miss 時的 fallback 是什麼?
- 是否能顯示 provenance?

### Integration

- Memory 以 raw text, structured block 或 tool result 進入 context?
- Token budget 如何分配?
- 哪些 instructions 永遠不能被 memory 覆蓋?
- Retrieved memory 是否會直接觸發外部 action, 或仍需 verification?

### Lifecycle

- 何時更新或合併 memory?
- 哪些資訊會 decay 或 expire?
- 使用者如何查看, 修正與刪除?
- 法規要求的 deletion 是否可跨 primary store, cache 與 backup 執行?
- 如何測量 memory 對結果的真實貢獻?

## 核心觀念回顧

```text
Agent memory system
├── Memory components
│   ├── Persona
│   ├── Conversation
│   ├── Entity
│   ├── Workflow and episodic
│   ├── Toolbox
│   └── Agent registry
├── Memory modes
│   ├── Working
│   ├── Short-term
│   ├── Long-term
│   └── Dynamic
└── Lifecycle
    ├── Generate
    ├── Store
    ├── Retrieve
    ├── Integrate
    ├── Update
    └── Forget or delete
```

最值得帶走的三個原則:

1. Memory 不是無限 context, 而是持久化, retrieval 與 context assembly 的完整系統.
2. 不同 memory types 需要不同 schemas, retrieval methods 與 retention policies.
3. 保存 experience 前要區分 observation, hypothesis 與 verified lesson, 否則 memory 會累積並放大錯誤.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理. 影片沒有 YouTube chapters, 章節與時間戳由內容結構編排. 自動字幕可能誤辨研究者, 產品, library 與技術名稱. 本筆記只在上下文足以確認時修正, 並對 Memoriz 名稱與歷史日期保留不確定性.

演講由 MongoDB developer advocate 主講, 後半包含 MongoDB, Voyage AI 與相關產品定位. 影片展示的是高層 architecture patterns 與 document examples, 沒有提供完整 schema, security design, benchmark 或跨 memory platforms 的比較. Human memory 與 neuroscience 內容是設計類比, 不應視為 agent memory 的生物學等價模型.
