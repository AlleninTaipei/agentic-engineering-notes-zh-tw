# Agent Context Management: 從截斷、記憶到 Sub-Agent 隔離

- 影片: [How we solved Context Management in Agents - Sally-Ann Delucia](https://www.youtube.com/watch?v=esY99nYXxR4)
- 頻道: AI Engineer
- 講者: Sally-Ann Delucia, Arize Head of Product、Alyx core contributor
- 發布日期: 2026-05-10
- 片長: 16:16
- Video ID: `esY99nYXxR4`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

這場演講整理 Arize 團隊近一年建置 AI agent Alyx 時的 context management 經驗。Alyx 需要分析 observability platform 中的 traces 與 spans, 但 agent 本身執行時又會產生更多 trace data。資料成長會使 context 超限, 失敗與重試再繼續增加資料, 形成惡性循環。

團隊先後嘗試三種策略:

1. 只保留 context 開頭, 結果 agent 無法理解 follow-up。
2. 用 LLM 摘要完整 context, 但無法穩定控制哪些資訊被保留。
3. 保留 head、tail 與最新結果, 將中段移入可回取的 memory store。這是 Alyx 當時實際採用且已穩定運作數月的策略。

當單一 agent 即使經過壓縮仍承載太多 traces、queries 與中間推理時, 團隊再將 data-intensive work 交給 sub-agents, 主對話只接收精簡結果。此外, 他們以 long-session eval 重播前十輪, 測試第十一輪, 讓只在長對話後段出現的遺忘問題可以重現。

這些做法是 Alyx 特定 workload 的 production 經驗, 不是不同 agent 架構的受控比較。講者也明確承認 long-term memory、principled context budget、context quality metrics 與超大單次 input 仍未解決。

## Context Engineering 不只是塞滿 Window

[01:29](https://www.youtube.com/watch?v=esY99nYXxR4&t=89s)

講者將 context engineering 定義為策略性選擇 model 看見什麼, 而不只是讓 tokens 保持在上限以下。她的原則是:

```text
Agent 應記得完成任務所需的資訊, 並忘掉不再需要的資訊。
```

這個選擇同時是 engineering、product 與 UX 問題。Context 不正確會直接造成錯誤回答, 即使 request 沒有超出 provider 的 token limit, 使用者體驗仍會失敗。

Alyx 的 workload 讓這個問題特別明顯。它需要讀取包含 user input、prompts、metadata 與 tool activity 的 traces, 有時還要跨多個 traces 尋找模式。Context volume 因而隨分析範圍快速增長。

## 惡性循環: Agent 分析自己產生的資料

[04:06](https://www.youtube.com/watch?v=esY99nYXxR4&t=246s)

Arize 團隊用 Alyx 協助建置 Alyx, 因而遇到遞迴式資料成長:

```text
Alyx 分析 traces / spans
          ↓
執行產生更多 span data
          ↓
context 達到上限
          ↓
執行失敗並重試
          ↓
新增更多 trace data
```

受限於資料量的系統, 正在分析的正是造成限制的資料。這使 context management 成為必要能力, 而不是事後的 token optimization。

## 嘗試一: 只保留 Context 開頭

[05:16](https://www.youtube.com/watch?v=esY99nYXxR4&t=316s)

第一版直接保留開頭 100 characters, 捨棄其餘內容。簡單任務起初看似可用, 但後續問題很快失效。例如使用者先問最常見的 inputs, 再追問其中一個 input, Alyx 已無法理解指涉對象。

這個案例說明 token 數量不是唯一變數。若截斷移除了最近對話、tool result 或 referent, context 即使很小且合法, reasoning chain 仍會斷裂。

影片中的 `100 characters` 是團隊早期實驗值, 不應被當成通用設定。

## 嘗試二: 讓 LLM 摘要全部 Context

[06:14](https://www.youtube.com/watch?v=esY99nYXxR4&t=374s)

第二版讓 LLM 將長 context 摘要成較短內容。團隊認為結果過於不一致, 因為系統把「什麼重要」完全交給一次摘要決定, 缺少可控的保留規則。

這項觀察不代表 summarization 一律無效。影片沒有說明 summary prompt、model、schema、更新頻率或 evaluation dataset, 因此能支持的較窄結論是: 在 Alyx 當時的 trace-analysis workflow 中, 未加強約束的摘要不足以可靠保存後續分析所需的細節。

## 採用方案: Smart Truncation 加可回取 Memory

[06:46](https://www.youtube.com/watch?v=esY99nYXxR4&t=406s)

團隊最後採用 head/tail preservation 與 external memory 的組合:

- 保留內容開頭 100 characters。
- 保留內容結尾 100 characters。
- 移除中段, 但將它存入 memory store。
- Duplicate messages 與長 tool calls 只保留較新的結果。
- 不重設 system prompt。
- Agent 可依 ID、conversation position 與 preview 回取被移出的內容。

```text
active context
  ├─ system prompt
  ├─ head
  ├─ recent / latest results
  └─ tail

external memory
  └─ truncated middle, addressable and retrievable
```

這項設計將兩個責任分開:

- Context 決定 model 此刻直接看見什麼。
- Memory 決定哪些移出的資料仍可存活並再次取得。

它比永久丟棄多一條 recovery path, 又比每次摘要保留更多原始資訊。講者表示這個方法在 Alyx 中運作良好, 幾個月未需調整, 但團隊已開始重新檢視策略。

## 用 Long-Session Eval 找出延遲失敗

[08:02](https://www.youtube.com/watch?v=esY99nYXxR4&t=482s)

實際使用者常在同一 chat 中跨頁面完成多段 workflow, 不會主動重新開啟 session。短對話測試因此可能通過, 遺忘或錯誤卻在很晚的輪次才出現。

Arize 的測試方法是載入前十輪 conversation, 再測試第十一輪。這讓 long-session bug 不必等到 user report 才被發現。

實作時可將這個方法擴展為多種位置與壓力條件:

| 測試類型 | 要驗證的問題 |
| --- | --- |
| Early reference, late recall | 早期需求是否在後段仍可取得? |
| Tool-result follow-up | Agent 是否能找回先前工具輸出? |
| Duplicate tool calls | 去重是否誤刪不同版本的關鍵結果? |
| Cross-page workflow | 使用者切換工作頁面後, task state 是否仍一致? |
| Near-limit input | 單次大型輸入是否在進入 agent loop 前就失敗? |

後四項是依影片問題延伸的測試建議, 不是講者展示的完整 eval suite。

## 將重型資料移到 Sub-Agent

[09:23](https://www.youtube.com/watch?v=esY99nYXxR4&t=563s)

Alyx 的 search task 可能涉及數百個 spans、多輪 queries、大量資料與中間推理。團隊發現這些內容不必全部留在主對話中, 因而將 data-intensive operations 交給 sub-agents:

```text
main agent
  chat history + light context
          ↓ delegate
sub-agent
  heavy data + search + intermediate reasoning
          ↓ compact result
main agent
  continue user conversation
```

主 agent 保留使用者互動與協調責任, sub-agent 在獨立 context 中完成狹窄任務, 最後只回傳結果。必要時兩者仍可從 memory store 回取資訊。

這個 pattern 的價值不只是增加平行工作者, 而是建立 context boundary。若 sub-agent 回傳完整 raw data 與所有 reasoning, 主 context 仍會再次膨脹。實作時需要明確定義 delegation input、result schema、provenance reference 與最大回傳量。

## 尚未解決的問題

[11:19](https://www.youtube.com/watch?v=esY99nYXxR4&t=679s)

講者列出當時仍在處理的限制:

- 巨大的 prompt 或單次 input 仍可能直接觸及 provider limit。
- 使用者從少於十輪延伸到二十輪以上, 現行 memory store 不等於真正的 long-term memory。
- 新 chat 無法自然引用過去與 Alyx 討論的 issue。
- Context selection 仍採 first 100、last 100 等 heuristics。
- 團隊尚無 principled context budget。
- Context quality 尚無清楚的專用 metrics, 主要透過 task evals 間接判斷。
- Sub-agent decomposition 仍是處理超大 context 時反覆採用的方向, 不是問題已完全消失。

講者也提到 Claude Code 公開內容中採用相似的 truncation 與 compression 策略, 但影片沒有提供版本、程式碼位置或逐項技術證據。本筆記不把這段敘述當成 Claude Code 現行實作規格。

## Q&A: Cache 尚不是目前優先項

[14:58](https://www.youtube.com/watch?v=esY99nYXxR4&t=898s)

觀眾詢問 context management 是否會刻意避免 invalidating cache。講者回答, 團隊目前主要投入 long-term memory, 還未深入處理 cache optimization。

被移出的內容保存在 database, 並以 IDs、conversation position 與 preview 供 Alyx 的工具查找。講者預期未來需要更精細的設計, 但現況可用, 因而先處理使用者抱怨較多的 long-term memory。

這呈現一個實務排序原則: 不一定先優化最底層的理論成本, 而是先處理已由 usage 與 complaints 顯示的產品失敗。

## 實作檢查表

- Context policy 是否明確區分 active context、retrievable memory 與 long-term memory?
- 每種 message、tool result 與 artifact 的保留理由是什麼?
- Truncation 是否保留 head、recent state 與重要 tail?
- 被移出的資料是否可透過穩定 ID 回取?
- Agent 是否看得到足以判斷是否回取的 preview 與位置資訊?
- Duplicate removal 是否能區分真正重複與版本更新?
- Summary 是否有固定 schema、重要欄位與 information-loss eval?
- 是否測試第十輪、第二十輪或接近 context limit 時的 follow-up?
- Eval 是否包含早期資訊在晚期被引用的案例?
- 大型單次 input 是否在進入 agent loop 前先做 admission control?
- Heavy search 或 analysis 是否能隔離到 sub-agent context?
- Sub-agent 是否只回傳 compact result 與 provenance references?
- Main agent 是否可能把完整 heavy context 再次拉回?
- 是否記錄每輪 input tokens、retrieval 次數、truncation event 與 failure point?
- Context quality 是否以 task success 衡量, 而不只看 token utilization?
- 新 session 要如何取得過去 issue, 又如何避免錯誤記憶永久累積?

## 核心結論

Context management 不是在超限時刪掉一些 tokens, 而是設計資訊的生命週期:

```text
直接可見 → 暫時移出但可回取 → 跨 session 保存或淘汰
```

Alyx 的經驗顯示, 過度截斷會切斷 follow-up reasoning, 無約束摘要可能丟失不可預測的資訊。Head/tail preservation 加可回取 memory 為兩者之間提供了可逆的取捨。當任務資料本身過重時, sub-agent 則把中間工作限制在獨立 context, 避免污染主對話。

最值得移植的不是 `100 characters` 這個數字, 而是三個工程原則:

1. 將 context selection 與 memory retention 分開設計。
2. 用 long-session eval 重現延遲出現的遺忘問題。
3. 以 sub-agent 作為 context isolation boundary, 並限制回傳內容。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。字幕將產品名稱 Alyx 多次辨識為 Alex, 本筆記依影片 metadata 與講者介紹統一為 Alyx。未檢查投影片畫面, 因此未補寫字幕沒有明確說明的 UI、code 或數值。

講者是 Arize Head of Product、Alyx core contributor, 並表示自己實際參與 agent engineering。內容屬於對 production system 的第一手經驗, 同時包含 Arize 與 Alyx 的產品介紹。

影片提供具體 failure modes、策略演進與現行限制, 但沒有公開 eval dataset、success rate、token reduction、latency、retrieval accuracy、memory store schema 或與其他策略的受控比較。`100 characters`、十輪加第十一輪測試、使用者對話增至二十輪以上等數字均為講者描述, 不應直接套用到不同 workload。
