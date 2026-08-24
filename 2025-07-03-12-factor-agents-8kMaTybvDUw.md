# 12-Factor Agents: 建立可靠 LLM 應用的軟體工程 Patterns

> 來源: [12-Factor Agents: Patterns of reliable LLM applications — Dex Horthy, HumanLayer](https://www.youtube.com/watch?v=8kMaTybvDUw)
> 頻道: AI Engineer
> 發布日期: 2025-07-03
> 片長: 17:05
> Video ID: `8kMaTybvDUw`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)
> [12-Factor Agents](https://github.com/humanlayer/12-factor-agents)

## 摘要

HumanLayer 的 Dex Horthy 從超過一百次與 founders、builders 和 engineers 的訪談中, 整理出可靠 LLM 應用反覆出現的工程 patterns。他將這些 patterns 稱為 12-Factor Agents, 呼應早期 12-Factor App 對 cloud-native application 的整理。

演講的核心主張是: Production agents 多半沒有想像中那麼「agentic」。可靠系統通常仍以傳統軟體為主, 只在適合語意判斷的局部加入短小 LLM loops。

建立可靠 agent 的重點包括:

- 把 LLM 視為 `tokens in -> tokens out` 的 stateless function。
- 把 tool use 視為 structured output 加上 deterministic code。
- 自己掌控 prompts、context、state 和 control flow。
- 將長流程拆成 deterministic DAG 與 small, focused agents。
- 把 human interaction 設計成正式工具與流程節點。
- 讓 frameworks 處理周邊基礎設施, 而不是隱藏最重要的 AI 行為。

## 為什麼 Agent 常停在 70% 至 80%

[00:00](https://www.youtube.com/watch?v=8kMaTybvDUw&t=0s)

許多 agent 專案有相似發展路徑:

1. 團隊使用現成 library 或 framework 快速建立 prototype。
2. 系統做到約 70% 至 80%, 足以展示價值並取得資源。
3. 剩餘錯誤開始需要精細調整 prompt、tools 和 context。
4. 開發者卻發現關鍵邏輯藏在 framework 多層 call stack 之下。
5. 為了理解 prompt 如何組合、tools 如何傳入, 最後只好重寫核心。

問題不一定是 framework 品質不佳, 而是可靠性提高後, 團隊需要直接控制原先被 abstraction 隱藏的細節。

### 並非所有工作都需要 Agent

Dex 曾嘗試建立 DevOps agent, 讓模型閱讀 Makefile 並依序執行 build commands。模型反覆弄錯順序, 於是他持續擴充 prompt, 最後把每一步的正確順序都明確寫出來。

此時系統已不再需要自主判斷。相同流程用約 90 秒就能寫成 Bash script, 執行速度和可靠性也更高。

判斷原則是:

```text
如果正確步驟已能完整、穩定地列舉
  -> 優先使用 deterministic code

如果需要根據自然語言、模糊狀態或人類意圖判斷下一步
  -> 考慮在局部加入 LLM
```

## 12-Factor Agents 的定位

[02:16](https://www.youtube.com/watch?v=8kMaTybvDUw&t=136s)

Dex 訪談超過一百名實作者後發現, 成功的 production agents 通常具備兩項特徵:

- 系統大部分仍是一般 software。
- 團隊逐步把小型、模組化 patterns 加入既有 codebase, 而不是進行 greenfield rewrite。

這些方法不要求深厚 AI 研究背景, 更接近成熟的 software engineering。12-Factor Agents 也不是 anti-framework 宣言, 而可視為對 framework 的 feature wish list: 工具應該幫助開發者快速移動, 同時保留建立高可靠系統所需的控制權。

影片因時間限制沒有按照正式順序逐一解釋全部 factors。以下依講者在演講中呈現的組合與關係整理, 只有講者明確說出的 factor numbers 才保留編號。

## Factor 1: Natural Language to Structured Output

[04:05](https://www.youtube.com/watch?v=8kMaTybvDUw&t=245s)

LLM 最實用的能力之一, 是把自然語言轉成符合 schema 的 JSON。Agent loop、tool magic 或複雜 framework 並非必要前提。

```text
使用者自然語言
  -> LLM 解讀意圖
  -> Structured JSON
  -> Deterministic application code
```

JSON 之後要做什麼, 應由一般程式決定。這讓 LLM 專注處理語意模糊性, 並把真正的 side effects 留給可測試、可觀測的 code。

## Factor 4: Tools 只是 Structured Outputs

[04:34](https://www.youtube.com/watch?v=8kMaTybvDUw&t=274s)

Dex 借用「Go To Statement Considered Harmful」的說法, 挑釁地提出「tool use is harmful」。他不是反對 agent 存取外部世界, 而是反對把 tool use 描述成神祕、特殊的機制。

實際流程只是:

1. LLM 輸出 JSON。
2. Deterministic code 解析資料。
3. `switch` 或其他 control flow 選擇 handler。
4. Handler 執行 API、資料庫或檔案操作。
5. 必要時把結果重新放入 context。

把 tools 去神祕化後, 開發者可以使用既有軟體工程方法處理 validation、authorization、retry、logging 和 testing。

## Factor 8: Own Your Control Flow

[05:32](https://www.youtube.com/watch?v=8kMaTybvDUw&t=332s)

軟體本來就是 graph。`if`、`switch` 和 function calls 都能形成 directed graph; Airflow、Prefect 等 orchestrators 則讓 DAG 更明確。

理想化 agent 省去預先定義 DAG 的工作。開發者只提供目標, LLM 在 loop 中自行決定下一步:

```text
事件或使用者訊息
  -> Prompt
  -> LLM 選擇 action
  -> 執行並把結果加入 context
  -> 再次詢問 LLM
  -> 直到模型宣告完成
```

這種 naive loop 在短任務中可能有效, 但長 workflow 容易失敗。主要原因之一是 context window 不斷膨脹。即使 API 接受數百萬 tokens, 更短、更密集、更一致的 context 往往仍能產生更可靠結果。

### Agent Loop 的四個基本部件

Dex 將 agent 拆成:

| 元件 | 責任 |
| --- | --- |
| Prompt | 指示模型如何選擇下一步 |
| Switch statement | 將 structured output 對應到 deterministic action |
| Context builder | 決定模型知道哪些歷史、結果和狀態 |
| Loop / exit logic | 決定何時繼續、暫停、切換或結束 |

當開發者擁有 control flow, 就能加入 break、switch、summary、LLM-as-judge 或其他針對任務的可靠性機制。

## 統一 Execution State 與 Business State

[07:15](https://www.youtube.com/watch?v=8kMaTybvDUw&t=435s)

Agent 同時存在兩種 state:

| State 類型 | 範例 |
| --- | --- |
| Execution state | Current step、next step、retry count、執行狀態 |
| Business state | 對話訊息、向使用者展示的資料、等待核准的 action |

可靠系統應像一般 API 一樣支援 launch、pause 和 resume。Dex 描述的 long-running tool 流程如下:

1. Request 進入 REST API 或 MCP server。
2. 系統載入 agent state 並建立 context。
3. Agent 呼叫 long-running tool。
4. Workflow 暫停, context 與 state 序列化至 database。
5. 背景工作完成後, 以 state ID 回呼。
6. 系統從 database 恢復狀態並附加結果。
7. Agent loop 從中斷位置繼續。

模型不必知道程序曾被暫停。因為 application 擁有 context 和 state, 非同步執行可以被包裝成連續的 agent experience。

## Factor 2: Own Your Prompts

[08:45](https://www.youtube.com/watch?v=8kMaTybvDUw&t=525s)

Prompt abstraction 可以快速產出高品質起點, 但當團隊需要跨越特定 reliability bar, 往往必須精確控制 prompt 中的 tokens。

Dex 的推理是:

```text
Model 與 sampling 設定固定時
  -> Input tokens 決定可用資訊與行為偏好
  -> Output tokens 決定 agent 的下一步
```

因此開發者應能:

- 查看完整 prompt。
- 修改每一部分內容。
- 嘗試不同順序、格式和措辭。
- 以 evaluations 比較結果。
- 避免 framework 在不可見位置注入內容。

演講沒有宣稱存在唯一正確 prompt。重點是保留足夠 knobs, 讓團隊能透過實驗找出適合自己任務的版本。

## Own Your Context Window

[09:44](https://www.youtube.com/watch?v=8kMaTybvDUw&t=584s)

Context engineering 涵蓋 prompt、memory、RAG、history 與 tool results。對模型而言, 這些最後都只是 input tokens。

開發者不必被標準 chat messages format 限制。若當前任務只是要求模型選擇下一步, 可以將事件狀態和歷史整理成一個密集的 user message, 也可以使用其他結構。重要的是資訊的 clarity 與 density。

Context builder 應回答:

- 目前目標是什麼?
- 已經完成哪些事?
- 有哪些仍有效的事實與限制?
- 哪些錯誤已不再重要?
- 模型現在真正需要什麼資訊才能做下一個決定?

## Compact Errors, 不要盲目累積失敗內容

[10:43](https://www.youtube.com/watch?v=8kMaTybvDUw&t=643s)

模型呼叫錯誤 API、參數不合法或服務故障時, 最簡單的做法是把 tool call 與完整 error 丟回 context, 再讓模型重試。這很容易造成模型反覆嘗試、context 失焦或陷入 loop。

更穩健的策略是:

- 不把完整 stack trace 無限附加到 context。
- 只保留模型能採取行動的錯誤資訊。
- 成功取得有效 tool call 後, 清除已解決的 pending errors。
- 將多次相似失敗摘要為一個限制或學習結果。
- 控制 retry 次數與退出條件。

這項 pattern 必須與「own your context window」一起使用。讓模型看到錯誤不是目的, 目的是提供足夠資訊讓下一次決策變好。

## Contact Humans with Tools

[11:31](https://www.youtube.com/watch?v=8kMaTybvDUw&t=691s)

許多 agent 在每一輪都面臨二選一: 呼叫 tool, 或向人類傳送一般訊息。Dex 建議把 human interaction 也設計成正式 tools, 讓 agent 明確表達 intent。

Human tools 可以區分:

- 任務已完成。
- 需要使用者澄清。
- 需要主管或專家判斷。
- 需要核准某個 action。
- 需要使用者提供缺少的資料。

這樣做能讓自然語言模型在產生第一個 token 時, 就在熟悉的語意空間中選擇溝通目的, 並讓 application 將不同 intent 對應到明確 workflow。

## Trigger from Anywhere, Meet Users Where They Are

[12:15](https://www.youtube.com/watch?v=8kMaTybvDUw&t=735s)

使用者不希望為每一個 agent 開啟不同的 ChatGPT-style tab。Agent 應該能從既有工作介面被觸發並回覆, 例如:

- Email。
- Slack。
- Discord。
- SMS。
- 既有 application events。

Channel 只是 input/output adapter。Business state、control flow 和 agent logic 應獨立於特定聊天 UI。

## Small, Focused Agents

[12:42](https://www.youtube.com/watch?v=8kMaTybvDUw&t=762s)

長而開放的 autonomous loop 容易因 context 膨脹和責任模糊而失敗。Dex 觀察到更可靠的做法是 micro agents:

- 整體保持 mostly deterministic DAG。
- 只在需要語意判斷的位置插入小型 agent loop。
- 每個 loop 約執行 3 至 10 個 steps。
- Agent 擁有清楚、狹窄的責任。
- 完成判斷後立即回到 deterministic code。

### HumanLayer 的部署案例

HumanLayer 的 deployment pipeline 大部分是 deterministic CI/CD。當 GitHub PR 已合併, development tests 通過後, 才把下一步交給模型。

範例流程:

```text
PR merged + development tests passed
  -> Deployment agent 提議部署 frontend
  -> Human: 先部署 backend
  -> Agent 將自然語言轉為下一步 JSON
  -> Backend deployment
  -> Frontend deployment
  -> 回到 deterministic end-to-end tests
  -> 若失敗, 交給另一個 focused rollback agent
```

LLM 的工作不是接管整套 CI/CD, 而是處理部署順序、自然語言核准與局部調整。真正的 build、deploy 和 test 仍由 deterministic systems 執行。

這種架構即使總系統擁有大量 tools 和完整長流程, 每個 agent context 仍可維持可管理大小。

## 模型變強後, Deterministic 與 Agentic 的邊界會移動

[14:22](https://www.youtube.com/watch?v=8kMaTybvDUw&t=862s)

系統可以從 mostly deterministic workflow 開始, 在局部加入 LLM。隨模型可靠性提高, agent 能承擔的範圍可以逐步擴大, 最終可能接管完整 API endpoint 或 pipeline。

但模型變強不會消除可靠性工程。Dex 引用 NotebookLM 團隊的類似看法: 找到位於模型可靠能力邊界的工作, 再透過 context、control flow 與 evaluation 讓它穩定成功, 就能建立比單純呼叫模型更有價值的產品。

因此邊界會移動, 基本能力仍然重要:

- 知道模型目前能穩定處理多大範圍。
- 能把任務縮小到可驗證單位。
- 能量測失敗並逐步擴大責任。
- 不因 context window 容量增加就停止整理 context。

## Agents 應該是 Stateless Reducers

[15:07](https://www.youtube.com/watch?v=8kMaTybvDUw&t=907s)

Dex 主張 agent 本身應保持 stateless, 由 application 掌控所有持久 state。有人指出多步驟轉換更接近 transducer 而非 reducer, 但工程含義相同:

```text
Current application state + new event
  -> Build context
  -> LLM selects action
  -> Deterministic code applies transition
  -> New application state
```

State 不應只存在 framework 內部、不可序列化的 agent object 中。Application 擁有 state 後, 才能可靠地 pause、resume、replay、inspect 和 migrate workflow。

## Framework 應處理什麼

[15:35](https://www.youtube.com/watch?v=8kMaTybvDUw&t=935s)

Dex 區分兩種工具哲學:

| 封裝式 framework | Scaffold / library approach |
| --- | --- |
| 將核心 loop 包在 abstraction 內 | 產生可直接擁有與修改的 code |
| 初期容易上手 | 初期需要理解更多細節 |
| 深度客製時可能受限 | 保留 prompts、state 和 control flow 的所有權 |

他將理想工具比作 shadcn: 不是把 internal implementation 永久藏在 wrapper 裡, 而是 scaffold 出 code, 讓團隊接手並負責維護。

Framework 不應試圖消除所有「困難的 AI 部分」。更有價值的工具是處理 database、queues、callbacks、observability 等重要但非差異化的周邊工作, 讓團隊把時間投入:

- Prompt quality。
- Context construction。
- Control flow。
- Evaluations。
- Human collaboration。

## 十二項 Patterns 的整體關係

影片沒有按正式編號完整列舉清單, 但所說明的 patterns 可以整理為以下系統:

| Pattern | 解決的問題 |
| --- | --- |
| Natural language to structured output | 將模糊人類意圖轉為可執行資料 |
| Own your prompts | 保留可靠性調整與 evaluation 的控制權 |
| Own your context window | 控制模型每輪實際看到的 tokens |
| Tools are structured outputs | 用一般 code 執行和驗證 side effects |
| Unify execution and business state | 讓 orchestration 與產品狀態一致 |
| Launch, pause and resume with APIs | 支援 long-running 與 asynchronous work |
| Contact humans with tools | 將澄清、核准與 escalation 納入流程 |
| Own your control flow | 能加入 break、retry、summary 與自訂退出條件 |
| Compact errors | 避免失敗內容污染 context 並造成 spin loop |
| Small, focused agents | 限制責任、steps 和 context 大小 |
| Trigger from anywhere | 把 agent 放入使用者既有工作環境 |
| Stateless reducer / transducer | 讓 application 擁有、持久化和恢復 state |

## 實作檢查表

### 選擇是否使用 Agent

- 正確步驟是否已能完整寫成 script?
- 問題是否真的需要語意理解或動態決策?
- LLM 的局部判斷能否嵌入 deterministic workflow?
- 失敗時是否有明確 fallback 和退出條件?

### Context 與 Prompt

- 能否查看送入模型的每一個 token?
- 是否移除已過期、重複或矛盾資訊?
- Error 是否已摘要為可行動資訊?
- Prompt 與 context variants 是否經 evaluation 比較?

### State 與 Control Flow

- State 是否由 application 擁有並可序列化?
- Workflow 是否能 pause、resume 和 replay?
- Long-running tools 是否使用 state ID 安全回呼?
- Human approval 是否為正式流程節點?
- Agent loop 是否具有 steps、time 或 cost limits?

### Agent Scope

- 每個 agent 是否只有清楚、聚焦的責任?
- 能否把 loop 限制在約 3 至 10 steps?
- 完成後是否立即返回 deterministic code?
- 模型能力提升時, 是否以評估結果逐步擴大範圍?

## 核心結論

可靠 agent 並不是把更多 autonomy 交給一個巨大 loop。更常見的成功路徑是從成熟 software architecture 出發, 只把自然語言理解和模糊決策交給 LLM。

對開發者而言, 最重要的所有權是 prompts、context、state 和 control flow。擁有這四者, 才能觀察模型失敗、建立 evaluations、加入人類協作, 並隨模型能力演進逐步調整 deterministic 與 agentic 的邊界。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕整理, 並非逐字稿。影片未提供公開章節, 本文段落和時間戳依演講內容轉折編排。口語贅詞、重複字幕、現場互動及產品宣傳已刪除。

本文的十二項 patterns 依講者在影片中的說明重新組織。影片為縮短版演講, 未按原始專案的正式順序完整介紹每個 factor, 因此除講者明確指出的 Factor 1、2、4、8 外, 本筆記沒有自行補上其他編號。專案內容可能在影片發布後更新, 應以 12-Factor Agents repository 的當前版本為準。
