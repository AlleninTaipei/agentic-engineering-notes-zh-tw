# AI Agent Harness 深入解析, 用確定性工程約束非確定性模型

來源: [Harnesses in AI: A Deep Dive, Tejas Kumar, IBM](https://www.youtube.com/watch?v=C_GG5g38vLU), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=C_GG5g38vLU
- 上傳日期: 2026-05-17
- 片長: 20:26
- Video ID: `C_GG5g38vLU`
- 講者: Tejas Kumar, IBM
- 內容依據: 英文原始語言自動字幕
- [basically-ai-harness](https://github.com/TejasQ/basically-ai-harness)

## 摘要

Agent harness 是模型周圍所有讓它接地, 受控並可驗證的軟體. 它不等於 agent loop, 而是包含 tool registry, context management, guardrails, agent loop, verification 與環境整合的整體執行層.

影片用一個 browser agent 示範 harness engineering. 任務只有一句話: 前往 Hacker News 並替第一篇文章按 upvote. 裸 agent 遇到登入頁面後失敗, 卻錯誤回報已完成. 講者完全不修改 task prompt 或 system prompt, 而是逐步加入 iteration limits, context trimming, trace-based verification 與 deterministic login handler. 最終, 同一個舊模型可以成功登入, upvote 並留下可驗證結果.

核心觀念是, 模型本身是不可完全控制的 black box, 但環境, secrets, tools, state, retries 與驗證規則可以由一般軟體工程掌控. 可靠性不應只靠更強的 prompt 或更昂貴的模型, 而應建立在可觀測, 可停止, 可驗證的 harness 上.

## 為什麼需要 Harness

[01:45](https://www.youtube.com/watch?v=C_GG5g38vLU&t=105s)

多數開發者租用模型 API 或訂閱服務, 無法控制模型權重, inference infrastructure, context 限制與服務端行為. 模型可能升級, 行為可能改變, 可用額度也有限.

Harness 的目的不是讓模型變成 deterministic, 而是在模型外部建立穩定邊界, 讓系統即使面對不確定輸出, 仍能維持可接受的行為.

講者用登山與遛狗的安全帶作比喻. Harness 把可移動的主體固定在穩定環境上, 允許它探索, 但不能無限制偏離或消耗資源.

```text
Uncontrolled variables
  -> Model behavior
  -> Model version
  -> Token budget
  -> Tool-call choices

Controlled by harness
  -> Tools
  -> Context
  -> Limits
  -> State
  -> Verification
  -> Secrets and environment
```

## ML Harness 與 Agent Harness 不同

[03:00](https://www.youtube.com/watch?v=C_GG5g38vLU&t=180s)

Machine learning 領域中的 harness 常指 test suite 與 test runner, 用固定 inputs 評估模型 outputs. 本片討論的是 AI engineering 中的 agent harness.

講者的定義是:

> Agent harness 是模型周圍所有讓它在現實環境中獲得 grounding 的部分.

Claude Code, Codex 或 Cursor 可以被視為 harnessed coding agents. 它們不只有模型, 還提供 filesystem tools, shell execution, context compaction, permissions, stop conditions 與 verification workflows.

## Agent Harness 的主要元件

[04:32](https://www.youtube.com/watch?v=C_GG5g38vLU&t=272s)

| 元件 | 責任 | Coding agent 例子 |
| --- | --- | --- |
| Tool registry | 限定模型可以執行的 actions | 讀寫檔案, 執行 shell, 搜尋程式碼 |
| Model | 根據 context 選擇下一個動作 | Claude, GPT 或其他模型 |
| Context management | 控制模型每一步能看到的資訊 | Compaction, trimming, memory, recent history |
| Guardrails | 限制成本, 時間與危險行為 | Max steps, permissions, timeout |
| Agent loop | 反覆取得模型輸出並執行工具 | Think, call tool, observe, continue |
| Verify step | 以外部證據判斷是否完成 | Lint, tests, UI state, database query |

Harness 不只是 agent loop. 它可能在 agent loop 外再建立一層 retry, verification 與 recovery loop. 因此, 實際結構可以是 nested loops, 而不是單一 `while true`.

## Demo 目標與刻意限制

[05:59](https://www.youtube.com/watch?v=C_GG5g38vLU&t=359s)

Demo 建立一個 computer-use browser agent, 任務是:

```text
Upvote a story on Hacker News.
```

講者刻意選用 GPT-3.5 Turbo 這個較舊模型, 用來展示 harness 能改善能力較弱模型的實務結果. 整個過程不修改 user prompt 或 system prompt, 只改模型外部的執行系統.

瀏覽器操作直接使用 Playwright, 不是 Playwright MCP. Tool definitions 採用 OpenAI SDK 的結構, 每個工具包含 name, description, parameters 與 execute function.

## 第一階段, 裸 Agent Loop

[07:00](https://www.youtube.com/watch?v=C_GG5g38vLU&t=420s)

初始程式包含:

- 一個 model.
- 一句 task prompt.
- Playwright browser session.
- Browser tools.
- System prompt 與 user message.
- 無限執行的 agent loop.
- 收集 events 的 trace list.

簡化後的控制流程如下:

```ts
while (true) {
  const response = await getModelResponse(context, tools);

  if (response.type === "stop") {
    return response.value;
  }

  trace.push(response);
  context.push(response);
}
```

模型會持續執行工具, 直到自己宣告完成. Trace 雖然收集了工具事件, 卻沒有任何程式檢查這些事件是否支持完成宣告.

## 初始失敗, 登入頁面與虛假成功

[08:12](https://www.youtube.com/watch?v=C_GG5g38vLU&t=492s)

Agent 開啟 Hacker News 並點擊 upvote 後, 被導向登入頁面. 它沒有憑證也沒有 recovery mechanism, 因此工作實際失敗. 但模型看到自己已執行 click, 便回報 upvote 成功.

這暴露兩個不同問題:

1. Capability gap, agent 不知道如何處理登入流程.
2. Verification gap, 系統把模型的自我陳述當成完成證據.

如果只修改 prompt, 例如把帳號密碼寫入 system prompt, 不僅不可靠, 還會讓 secrets 進入模型 context. 講者選擇把登入與驗證留在 deterministic harness.

## 第二階段, 加入 Guardrails 與 Context Management

[10:20](https://www.youtube.com/watch?v=C_GG5g38vLU&t=620s)

第一輪 harness 改進加入兩個 guardrails:

- `maxIterations`, 超過允許步數就終止.
- `maxMessages`, history 太長時壓縮 context.

Demo 的 context compressor 非常簡單. 它保留 system prompt, user task 與最近兩則 messages, 移除中間歷史:

```text
Keep:
  System prompt
  User task
  ... discard older middle messages ...
  Most recent two messages
```

講者明確表示這是教學用的 naive implementation, 不是 production 建議. 它可能刪除重要 tool result, failure state 或未完成約束. 真實系統需要 structured summary, durable state 或更細緻的 retention policy.

Guardrails 的價值在於建立有限失敗. Agent 即使無法完成任務, 也不會無限迴圈或持續消耗 tokens.

## 第三階段, 將執行邏輯封裝成 Harness

[11:54](https://www.youtube.com/watch?v=C_GG5g38vLU&t=714s)

講者把 entry point 中的 browser setup, tools, context 與 agent loop 移入 `runHarness` 函式. Entry point 只保留 task 與 harness invocation.

這次 refactor 沒有改變行為, 但建立了清楚的責任邊界:

```text
Application entry point
  -> Defines intent
  -> Calls harness

Harness
  -> Creates environment
  -> Registers tools
  -> Manages context
  -> Enforces guardrails
  -> Runs attempts
  -> Verifies outcome
```

把 harness 變成正式 abstraction 後, verification, retries, recovery handlers 與 telemetry 才能被一致地組合, 而不是散落在 application code.

## 第四階段, 用 Verify Step 拒絕模型的謊言

[13:02](https://www.youtube.com/watch?v=C_GG5g38vLU&t=782s)

新的 `runHarness` 增加 verify function 與 maximum attempts. 原本的單次流程移入 `runHarnessAttempt`, 外層 harness 最多重試指定次數.

`verifySuccessfulUpvote` 不詢問模型 "你成功了嗎?", 而是檢查已收集的 tool history. Demo 驗證幾種情況:

- 是否真的發生 upvote click.
- Tool execution 是否成功.
- Harness auto-login 是否回報失敗.
- Attempt 結束時是否仍停留在 login URL.

```text
Model says success
  -> Inspect trace
  -> Check expected tool call
  -> Check actual browser state and failure events
  -> Pass or fail
```

加入 verify step 後, agent 仍然不會登入, 所以任務依舊失敗. 但系統不再把失敗當成成功. 講者將這視為一半的勝利, 因為可靠 recovery 的前提是先能正確辨識失敗.

這也呼應 test-driven development. 先建立會失敗的明確判準, 才能知道後續修正是否真正有效.

## 第五階段, Deterministic Login Handler

[15:36](https://www.youtube.com/watch?v=C_GG5g38vLU&t=936s)

Login handler 在每次 agent loop 中檢查目前 browser URL:

```ts
async function handleLogin(session) {
  if (!isLoginPage(session.currentUrl())) {
    return;
  }

  await fillCredentialsFromSecureSource(session);
  await submitLoginForm(session);
  return { type: "harness_auto_login", status: "success" };
}
```

若不在登入頁面, handler 立即返回, 幾乎沒有額外成本. 若偵測到 login URL, harness 直接使用 Playwright 填入 credentials 並提交表單. Secrets 可來自 environment variables 或 secret manager, 不必放進 prompt.

登入完成後, handler 將一則結構化事件放入 trace / context, 告訴 agent 環境已恢復, 可以繼續任務.

這個設計展示一項重要分工:

| 交給模型 | 交給 harness |
| --- | --- |
| 理解 "替第一篇文章 upvote" 的意圖 | 安全取得 credentials |
| 決定瀏覽與點擊步驟 | 偵測 login URL |
| 在恢復後繼續任務 | 填表, 提交與記錄結果 |
| 使用工具探索頁面 | 以 trace 驗證是否完成 |

登入是一個可偵測, 有明確程序且涉及 secrets 的流程, 因此 deterministic code 比自由生成的 agent 行為更合適.

## 最終結果

[17:42](https://www.youtube.com/watch?v=C_GG5g38vLU&t=1062s)

最終 demo 的流程是:

1. Agent 開啟 Hacker News.
2. Agent 點擊 upvote.
3. Browser 被導向 login page.
4. Harness 偵測 URL 並安全完成登入.
5. Harness 將恢復事件交回 agent.
6. Agent 繼續並完成 upvote.
7. Verify step 檢查 trace 與頁面狀態.
8. Harness 回報成功與使用的 iterations.

講者再開啟 Hacker News, 以畫面顯示可執行 `unvote`, 作為 upvote 已生效的外部證據.

模型, user prompt 與 system prompt 從頭到尾沒有更改. 成果差異來自 harness 提供的 limits, state, verification 與 deterministic recovery.

## 為什麼較弱模型也可能有用

[18:34](https://www.youtube.com/watch?v=C_GG5g38vLU&t=1114s)

講者主張, 良好 harness 能讓便宜, 較小或較舊的模型完成原本不可靠的任務. 對企業而言, 這可能降低 token cost, 支援 private deployment, 或讓模型在 data-sensitive environment 中工作.

他以 IBM 的開源 OpenRAG 專案為例, 說明 enterprise RAG 也需要 harness 提供 security 與受控資料存取. 這部分是講者對其工作背景的概述, 影片沒有展示 OpenRAG 的完整架構或評估結果.

Harness 不能無條件彌補模型能力. 如果任務需要模型沒有的知識, 推理或 perception, 外部程式只能限制和輔助, 無法保證成功. Demo 證明的是特定 browser workflow 可被工程化補強, 不是所有弱模型都能靠 harness 達到 frontier model 水準.

## Dynamic Harnesses 的未來構想

[19:14](https://www.youtube.com/watch?v=C_GG5g38vLU&t=1154s)

講者推測, 下一步可能是 on-the-fly generated harness. Agent 在執行任務前, 先分析自己可能 hallucinate 或失敗的位置, 然後動態產生 guardrails, verify steps 與 recovery logic, 再開始工作.

```text
User task
  -> Analyze risks and failure modes
  -> Generate task-specific harness
  -> Review or approve harness
  -> Execute under constraints
  -> Verify outcome
```

這類設計可視為比 plan mode 更進一步, 不只產生文字計畫, 還產生可執行的控制與驗證程式. 影片將它列為未來方向, 並沒有現場實作或證明其安全性.

動態產生的 harness 本身仍可能出錯. 若用於購票, 金流, 刪除資料或外部溝通, 應先由固定的 meta-harness 限制它能產生的 tools, permissions 與 effects. 這是根據影片構想所做的編輯補充.

## 從 Demo 提煉的設計原則

### 1. 不把模型自我陳述當成證據

模型說 "完成" 只是一個 hypothesis. 系統應檢查可觀察的外部狀態, 例如 DOM, HTTP response, database record, test result 或 transaction ID.

### 2. 先讓失敗可辨識

在加入 recovery 前, 先建立明確 verify step. 如果系統無法區分成功與失敗, retry 只會重複製造錯誤結果.

### 3. 把確定性流程留給程式

登入, secrets injection, schema validation, retry limit 與權限判斷有明確規則, 適合使用 deterministic code. 模型應負責需要語意理解與探索的部分.

### 4. Trace 是控制資料, 不只是 log

Tool history 可供 verification, recovery, observability 與 audit 使用. Trace event 應具有結構化 type, status, inputs, outputs 與 error, 而不是只有自由文字.

### 5. Guardrails 要同時存在於不同層級

- Tool level, 驗證參數與限制單一 action.
- Agent-loop level, 限制 messages, tokens 與 steps.
- Attempt level, 限制 retries 與 wall-clock time.
- System level, 限制 permissions, budget 與 external effects.

### 6. Context compression 不能等同任意刪除

最近幾則 messages 不一定包含最重要的狀態. Production harness 應將 durable task state 與 conversational history 分開, 並確保約束, 已完成工作與 unresolved failures 不會在 compaction 中消失.

## 實作骨架

以下是根據影片流程整理的概念性架構, 不是影片程式碼的逐字重建:

```ts
async function runHarness(task, options) {
  for (let attempt = 1; attempt <= options.maxAttempts; attempt++) {
    const trace = await runAttempt({
      task,
      tools: options.tools,
      guardrails: options.guardrails,
      handlers: options.handlers,
    });

    const verdict = await options.verify(trace);

    if (verdict.ok) {
      return { status: "completed", trace, evidence: verdict.evidence };
    }
  }

  return { status: "failed", reason: "verification_failed" };
}
```

Production implementation 還應加入:

- Per-tool permissions.
- Secret manager integration.
- Idempotency 與 duplicate-action prevention.
- Timeout, cancellation 與 budget limits.
- Structured trace storage.
- Recovery policy 與 retry classification.
- Human approval for consequential actions.
- Verify step 自身的測試.

## 核心觀念回顧

```text
Agent harness
├── Model adapter
├── Tool registry
├── Context manager
├── Guardrails
├── Agent loop
├── Environment handlers
├── Trace and state
├── Retry policy
└── Verifier
```

最值得帶走的三個原則:

1. Harness 是 model 周圍的可靠性系統, 不只是 agent loop 的別名.
2. Verification 要依據外部證據, 不能依賴模型宣稱自己完成.
3. Prompt 不一定是第一個修正點. 可預測的 failure mode 應優先由 deterministic code 處理.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕可能誤辨程式識別字, 模型名稱與專案名稱. 本筆記只在上下文足以確認時修正, 並將 demo 重整成概念性程式片段, 不是講者原始碼的逐字複製.

影片示範的是 Hacker News upvote 這個受限任務, 並使用講者準備好的 Playwright tools, credentials 與 verification logic. 成功結果說明 harness 對此 workflow 有效, 但沒有提供跨模型, 多任務或 production load 的量化 benchmark. Dynamic harnesses 則是講者的未來推測, 不是已驗證成果.
