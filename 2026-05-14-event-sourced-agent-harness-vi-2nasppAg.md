# 用 Event Sourcing 建立可除錯、可組合的 Agent Harness

- 影片: [Make your own event-sourced agent harness using stream processors - Jonas Templestein, Iterate](https://www.youtube.com/watch?v=vi-2nasppAg)
- 頻道: AI Engineer
- 講者: Jonas Templestein 與 Misha, Iterate
- 發布日期: 2026-05-14
- 片長: 01:04:26
- Video ID: `vi-2nasppAg`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

這場 workshop 示範一種 event-sourced agent harness。核心設計只有三個部分:

1. Append-only event stream, 保存 agent 發生過的一切。
2. 同步且無副作用的 reducer, 從事件重建目前 state。
3. `afterAppend` hook, 只在追上最新事件後執行 LLM request、tool call 或新增事件等 side effects。

所有輸入、streaming chunks、model outputs、tool calls、errors、排程、pause 與 circuit breaker 都表示成事件。Agent 重啟時可以重播 event log 恢復 state, 卻不會重新呼叫 LLM 或再次執行工具。這種分離讓 trajectory、錯誤與控制決策都留在同一條可檢查的紀錄中。

另一項主張是 composability。不同作者的 processors 可以在不同 server、語言或 runtime 中訂閱同一條 stream, 各自衍生 state 或追加新事件。甚至可以把 processor source code 放進 `dynamic worker configured` event, 讓原本只有資料的 stream 動態取得 agent 行為。

這是一個臨時完成的 proof of concept。現場 SDK 與 live coding 多次失敗, 服務沒有 authentication, distributed processors 也可能產生 race conditions 或無限事件迴圈。因此影片最有價值的部分是 architecture experiment 與失敗邊界, 不是可直接部署的 production framework。

## 為什麼採用 Event Sourcing

[00:01:33](https://www.youtube.com/watch?v=vi-2nasppAg&t=93s)

講者認為許多 agent harness 已經接近 event-driven system, 卻仍有部分 side effects 只存在於外部 traces、runtime memory 或特殊 callback 中。結果是 agent 執行失敗後, 使用者能看到部分對話紀錄, 卻不能由單一資料來源回答:

- 哪個事件觸發了 LLM request?
- Model streaming 過程產生了什麼?
- 哪個 tool call 已實際執行?
- Error 或 retry 為何發生?
- Stream 何時被 pause 或 circuit break?
- 重啟後 state 如何恢復?

Event-sourced harness 將所有可觀察變化寫進 append-only log。State 不是另一份必須同步維護的資料, 而是 event history 的 projection。

```text
Event 1 -> Event 2 -> Event 3 -> ... -> Event N
                  |
                  v
          reduce(events) -> current state
```

這讓 debug 不必跨對話、application logs、tool traces 與資料庫猜測事件順序。只要 event schema 與 ordering 足夠可靠, 就能重建 agent 看見過的狀態。

## 設計目標

[00:02:24](https://www.youtube.com/watch?v=vi-2nasppAg&t=144s)

講者希望 architecture 具備以下性質:

| 目標 | 意義 |
| --- | --- |
| Debuggable | Agent 的輸入、輸出、錯誤與控制事件都留在同一條 log |
| Extensible | 人類或 agent 本身可以加入新的 processor |
| Composable | 不同 processors 可使用共同事件協定協作 |
| Edge-native | Agent 建立後即可透過 URL 與 HTTP 接收外部事件 |
| Distributed | Processor 可位於不同 server、語言及 runtime |

公開 routability 可簡化 Slack webhook、web form 或其他服務的輸入, 不必為每種來源另建 connector abstraction。但公開端點也直接帶來 authentication、authorization、rate limiting 與 secret handling 問題。影片中的 demo 沒有處理這些安全需求。

Distributed processors 允許某個 Rust plugin 與另一個 TypeScript processor 操作同一條 stream, 同時也引入 concurrency、ordering、duplicate delivery、race conditions 與 processors 互相觸發形成 infinite loop 的風險。

## Event Stream API

[00:05:40](https://www.youtube.com/watch?v=vi-2nasppAg&t=340s)

Workshop 使用臨時建立的 `events.iterate.com` 服務。每個 agent 使用類似 filesystem 的 hierarchical path, 對任一路徑追加第一個事件時便隱式建立 stream。

一個事件至少需要 `type`, 通常還有 `payload`。Server 接收後補上 envelope metadata:

```text
type
payload
stream path
offset
created_at
```

`offset` 是 server serialization point 產生的遞增整數, 可用來排序及追蹤 consumer 已讀位置。講者偏好用可連到文件的 URL 作為 event type, 但這不是必要條件。

### 讀取與持續訂閱

[00:08:38](https://www.youtube.com/watch?v=vi-2nasppAg&t=518s)

Client 可以透過 HTTP 取得既有 events, 並使用 Server-Sent Events 持續接收新增內容。`live=true` 讓連線在 catch-up 後保持開啟。

```text
GET stream
  -> 回傳既有 events
  -> 到達最新 offset
  -> 保持 SSE connection
  -> 持續傳送新 events
```

除了 client pull, stream 也能設定 push subscription, 在事件追加時通知 webhook 或其他 server。Subscription 可以傳送完整或經過篩選的事件流。

### Errors 也是 Events

[00:10:48](https://www.youtube.com/watch?v=vi-2nasppAg&t=648s)

如果 client 提交的物件不符合 event schema, 例如缺少 `type`, server 不只回傳瞬時錯誤, 還會在 stream 中追加 invalid-event 類型的 event。這使 validation failure 也成為 trajectory 的一部分。

不過並非所有錯誤都能繼續 append。例如 paused stream 拒絕新事件時, 若每次拒絕又新增 error event, 反而可能維持原本想阻止的無限迴圈。

### Idempotency、Pause 與 Circuit Breaker

[00:11:57](https://www.youtube.com/watch?v=vi-2nasppAg&t=717s)

Webhook 或 distributed consumer 可能重送相同 request, 因此 API 支援 idempotency key, 避免同一 logical event 被重複追加。

Pause 與 resume 本身也由事件表示。Demo 中, 當 stream 在一秒內出現超過約 100 個事件時, circuit breaker 會追加 pause event, 後續輸入被拒絕, 直到有人 resume。這是針對 processors 互相觸發、快速形成 event storm 的基本保護。

### Scheduling 與 Heartbeats

[00:13:49](https://www.youtube.com/watch?v=vi-2nasppAg&t=829s)

Client 可以追加 schedule event, 要求系統在指定時間或固定間隔加入另一個 event。例如每 5 秒產生 heartbeat, 或在 10 分鐘後喚醒 agent。排程的建立與取消都保留在 log, 因此能檢查某次 wake-up 的來源。

## Stream Processor 的三部分

[00:30:06](https://www.youtube.com/watch?v=vi-2nasppAg&t=1806s)

Agent behavior 被包成 stream processor。概念性介面如下:

```ts
type Processor<State, Event> = {
  initialState: State;
  reduce: (state: State, event: Event) => State;
  afterAppend: (state: State, event: Event) => Promise<void>;
};
```

這段介面是根據影片概念重寫的說明, 不是 workshop repository 的逐字程式碼。

### Reducer 必須同步且沒有 Side Effects

Reducer 接收前一個 state 與下一個 event, 回傳新的 state。它只做 deterministic computation, 不呼叫 LLM、不執行工具, 也不寫入外部服務。

```text
state0 + event1 -> state1
state1 + event2 -> state2
state2 + event3 -> state3
```

同一組 events 應能重建相同 state。UI 也可以是一個 reducer: 它從原始事件選擇重要項目, 投影成較容易閱讀的 feed items, 而不必改變原始 log。

### Side Effects 集中在 afterAppend

[00:31:36](https://www.youtube.com/watch?v=vi-2nasppAg&t=1896s)

LLM request、tool execution、HTTP request 或追加新事件都屬於 side effects, 放在 `afterAppend`。分離的原因是 processor 重新啟動時必須先讀完歷史 events 才能恢復 state, 但不能對每個歷史事件重新執行外部動作。

```text
Processor restart
  -> replay event 1..N through reducer
  -> rebuild state at offset N
  -> do not rerun historical side effects
  -> begin afterAppend only for newly appended events
```

例如 agent 已完成 100 次事件互動後 restart, catch-up 過程不應重新送出之前的 LLM requests 或 tools。抵達 stream tip 後, processor 再根據最新 state 決定下一個動作。

這個規則是架構最重要的 correctness boundary。若 reducer 偷做 side effect, 或 consumer 無法可靠區分 replay 與 live delivery, restart 就可能造成重複付款、重複外部訊息或重複修改資料。

## 將 Agent Loop 表示成 Events

[00:36:21](https://www.youtube.com/watch?v=vi-2nasppAg&t=2181s)

基本 agent loop 可以表示為:

```text
User message appended
  -> reducer 更新 conversation state
  -> afterAppend 發出 LLM request
  -> streaming chunks 逐一追加為 events
  -> reducer 投影 assistant response
  -> model requested tool call
  -> afterAppend 執行 tool
  -> tool result appended
  -> reducer 更新 state
  -> afterAppend 決定是否繼續 LLM loop
```

Streaming deltas、完整 output items、tool arguments、tool results 與 errors 都不需要特殊的旁路 API, 可共用 event abstraction。不同 UI 或 plugins 可以各自選擇消費原始 deltas、materialized messages 或更高階 projection。

講者認為, agent harness 常見的 callback、hook 與 trace types 最後都能收斂成 event types 加 processors。這個簡化具有吸引力, 但也把複雜度轉移到 event schema evolution、ordering、idempotency 與 processor coordination。

## Processor Composability

[00:47:34](https://www.youtube.com/watch?v=vi-2nasppAg&t=2854s)

所有 processors 不必運行在同一 process。它們只要能取得 events、維護 offset 並追加新 events, 就能共同擴展 agent:

```text
Shared event stream
  -> Core agent processor
  -> UI projection processor
  -> Circuit breaker processor
  -> Scheduling processor
  -> Safety or policy processor
  -> Domain-specific plugin
```

Reducer 是同步函式, 因此一個 processor 也可以 import 另一個 processor 的 reducer, 在其 state projection 上建立更高階 abstraction。這種組合不必共享完整 runtime 或 programming language, 但必須共享 event contracts。

Circuit breaker 是影片中的 production-like example。Reducer 保留最近事件的 timestamps; `afterAppend` 發現短時間事件數超過門檻後, 追加 pause event。Web UI 也使用相同思想, 從 events 投影成 feed items。

## Dynamic Workers: 將 Processor 當成 Event 部署

[00:50:35](https://www.youtube.com/watch?v=vi-2nasppAg&t=3035s)

Workshop 最特別的 demo 是 `dynamic worker configured` event。其 payload 包含一段 JavaScript string, 其中定義 reducer 與 `afterAppend` hook。事件追加後, backend 在 Cloudflare dynamic worker 中執行 processor。

講者先展示簡單的 ping-pong processor:

```text
Append dynamic worker source code
  -> stream gains processor behavior
Append ping
  -> processor appends pong
```

同一概念也能部署約數十行的基礎 AI agent。若需要 dependencies, 可以先把 package 與 source bundle 成單一 payload。API secrets 不應直接放進公開 event log; demo 將 OpenAI API key 留在 stream 外, 執行 fetch 時再由環境替換 header。

這讓「部署 agent」變成追加 configuration event, 也讓 agent 有可能透過產生新 processor source code 修改自己的功能。但它同時等同執行動態遠端程式碼, production implementation 必須處理 sandbox、code signing、版本、rollback、resource quotas、dependency integrity 與秘密管理。

## Before Hooks 與 Eventual Consistency

[00:58:45](https://www.youtube.com/watch?v=vi-2nasppAg&t=3525s)

講者對一般 third-party `before` hooks 持保留態度。同步 hook 若阻塞 agent loop, 可能增加 latency、破壞 context caching, 也讓某個外部 processor 故障時拖垮整個 agent。

他偏好的模式是 eventual consistency。例如 agent 在發出下一次 LLM request 前, 最多等待約 200 ms, 讓 safety checker 或 retrieval processor 有機會追加 context。時間到後即使外部 processor 沒有回應, agent 仍繼續執行。

```text
LLM request planned
  -> publish pending-request event
  -> wait up to 200 ms for optional context or warning
  -> incorporate events that arrived in time
  -> continue even if optional processor is unavailable
```

這個設計適合非關鍵 enrichment, 例如 optional RAG context。若 safety check 的目標是阻止不可逆或高風險操作, fail-open 行為可能不可接受, 應改用可信的同步 enforcement point。影片的內建 pause enforcement 也顯示, 某些規則必須在 event append 前由核心系統執行, 不能完全交給 eventual third-party processor。

## Workshop 的失敗本身揭露了什麼

這場 workshop 在開場便說明 SDK 剛於現場前一分鐘推送, 服務也只花幾天製作。Live coding 後來因 SDK 或範例無法正常執行而中止, 講者明確承認 demo 失敗, 改以已能運作的 stream、circuit breaker 與 dynamic worker 說明概念。

這個失敗揭露幾項現實:

- Architecture idea 與可供他人使用的 SDK 之間仍有很大距離。
- AI 協助臨時 refactor 可能在 workshop 前破壞原本能執行的範例。
- Dynamic worker 的最吸睛路徑能 demo, 不代表完整開發體驗已成熟。
- 沒有 authentication、tenant isolation 與完整 quota 的公開 stream 只能視為短期實驗環境。

講者承諾另行錄製完整示範, 但本筆記只記錄這支影片中實際展示與說明的內容, 不假定後續材料已完成。

## Production 化前的檢查清單

以下為根據影片架構整理的工程檢查項目:

- Event schema 是否有版本與相容策略?
- Offset、ordering 與 concurrent appends 的語意是否明確?
- Consumer 是否能正確區分 catch-up replay 與 live delivery?
- Side effects 是否具有 idempotency key 及重試政策?
- Processor failure 是否形成 error event, 並避免 error loop?
- Circuit breaker 的門檻、pause owner 與 resume 流程是否可稽核?
- Scheduling 是否支援取消、重複排程與 missed delivery?
- Dynamic code 是否經過 sandbox、簽章、版本控制及資源限制?
- Secrets 是否永遠位於 event payload 之外?
- Stream 是否具有 authentication、authorization 與 tenant isolation?
- 哪些 enrichment 可 fail open, 哪些 safety rule 必須 fail closed?
- 是否能重播 production event sequence 到隔離環境進行 debug?

## 證據與限制

講者直接設計並現場操作 prototype, 因此對 event API、stream processors 與 dynamic workers 的行為具有第一手性。影片也誠實呈現 SDK 未成熟與 live coding 失敗, 提供罕見的 failure evidence。

然而, 這套服務不是 production system。影片沒有提供負載測試、delivery guarantees、資料持久性 SLA、multi-region consistency、security model、成本、正式使用者或 incident history。Dynamic worker、200 ms 等數字只是 demo 設計, 不是經驗驗證出的通用門檻。

影片使用英文原始自動字幕。部分 API 名稱與即興程式碼可能有辨識誤差; 本文只保留上下文可確認的設計, 並將概念性程式碼明確標為編輯整理。
