# 何時應自建 Agent Harness: Distribution、Evals 與 Trace-Driven Improvement

- 影片: [When to Build Your Own Agent Harness | Harrison Chase, LangChain](https://www.youtube.com/watch?v=HI2q3ci3Iuc)
- 頻道: Sequoia Capital
- 講者: Harrison Chase, LangChain co-founder and CEO
- 發布日期: 2026-08-13
- 片長: 23:57
- Video ID: `HI2q3ci3Iuc`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

Harrison Chase 將 agent 拆成 model、context 與 harness 三部分。Harness 的核心工作不是單純反覆呼叫模型, 而是在每一步選擇 context、執行工具、接收 observations, 再把新的資訊送回模型。

是否自建 harness 不是二元選擇, 而是一條連續光譜:

```text
off-the-shelf harness
        ↓
hooks / middleware customization
        ↓
domain-specific gates and checks
        ↓
bespoke cognitive architecture
```

講者的主要判斷方式是 task distribution。若任務接近模型與既有 harness 的訓練分布, 現成方案通常有較快的 time to value。當任務愈偏離訓練分布, 或組織需要更強的 predictability、control 與 domain-specific behavior, 自訂程度才應逐步增加。

無論使用哪一層, 都需要以 private evals 定義「對組織而言什麼叫好」, 並保存 traces、feedback 與 institutional context。這些資料形成持續改善 model、context 與 harness 的共同基礎。

## Agent 由 Model、Context 與 Harness 組成

[00:58](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=58s)

LangChain 將 agent 分成三個相互依賴的部分:

| 元件 | 主要責任 | 所謂 ownership |
| --- | --- | --- |
| Model | 理解、推理與產生 actions | 能切換 models, 降低 lock-in, 選擇適合任務的能力 |
| Context | Memory、semantic knowledge、歷史對話與動態 observations | 保存、治理並決定哪些資訊可供 agent 使用 |
| Harness | 組織 loop、context injection、tool execution 與 control flow | 能觀察、修改與評估 orchestration behavior |

Harness 的核心任務是「在正確時間把正確 context 帶給模型」。Agent 與外部系統互動後會產生新資訊, harness 再決定如何將這些 observations 帶回下一次 model call。

## 最小 Agent Loop 與自訂位置

[02:12](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=132s)

多數 agents 的最小結構都可簡化成:

```text
request
  ↓
LLM generation
  ↓
需要工具? ── 否 ──→ final response
  │
  是
  ↓
execute tool
  ↓
observation 回到 context
  └────────────────────→ 下一次 model call
```

不同 harness 的差異通常不在這個 loop 是否存在, 而是在 loop 的不同位置加入哪些行為。講者以 LangChain 的 middleware 說明常見插入點:

- Agent invocation 之前。
- 每次 model call 之前或周圍。
- Tool call 之前、之後或周圍。
- Context 累積超過限制時。

這類 middleware 與 coding agents 中的 hooks 或 plugins 概念相近。

## Middleware 如何形成進階 Harness

[03:25](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=205s)

簡單 loop 不需要被完整重寫, 就能加入許多進階能力:

| 能力 | 可介入的位置 |
| --- | --- |
| Summarization | Model call 前檢查 context 長度並摘要 |
| Context offloading | Tool call 後將大型結果存到外部儲存, context 只保留引用 |
| Sandbox | Tool execution layer |
| Filesystem | 作為工具與持久 context source |
| Sub-agents | 將特定任務委派為另一段 loop |
| Memory | Model call 前 retrieval, 或執行後更新 |
| Domain checks | Tool 或 response 前後加入驗證與 gates |

講者以 LangChain 的 Deep Agents 為例。它建立在相同的基本 loop 上, 再透過 middleware 加入 filesystem、skills、sub-agents、memory 與 summarization。

這是一手產品設計說明, 但同時帶有 LangChain 產品定位。它能支持「middleware 是 LangChain 的擴充方式」, 不能單獨證明這是所有 harness 的最佳架構。

## Cognitive Architecture 仍有適用場景

[05:47](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=347s)

2023 至 2024 年間, 模型尚不容易只靠通用 loop 完成複雜任務, 因此常見做法是建立明確的 cognitive architecture。例如 deep research workflow 會先拆 sub-questions, 平行搜尋後再整合; code review bot 則可能強制依固定階段檢查。

模型與通用 harness 進步後, 許多能力可以改用 loop 加 middleware 表達。不過, 以下情況仍可能需要更明確的流程:

- Domain task 需要固定分析順序。
- 某些步驟必須經過 gates 或 verifier。
- 結果必須具有較高 predictability。
- 組織不接受 agent 自由決定完整路徑。
- 法規、金融或其他高風險場景需要清楚 control points。

講者建議先從 general harness 開始, 快速取得可運作版本。當 use case 與 failure patterns 逐漸清楚後, 再加入 gates、checks 或更專用的 cognitive architecture。

## 自建 Harness 的 Distribution 判斷

[07:03](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=423s)

現成 harness 通常與特定 models 一起演進。例如 Claude Code 與 Claude Agent SDK 配合 Anthropic models, Codex 配合 OpenAI models。模型可能已經透過訓練或 reinforcement learning 熟悉其中的 tool schemas、file editing 方式與 interaction patterns。

講者提出以下判斷:

```text
任務愈接近模型與 harness 的訓練分布
  → off-the-shelf harness 通常愈有優勢

任務愈偏離訓練分布
  → 愈需要 domain-specific customization
```

### Legal AI 的混合案例

[08:24](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=504s)

Legal AI 作為整體任務可能超出通用 coding agent 的主要訓練分布, 因此需要自訂 harness。但其中的 file editing 已是模型反覆受訓的能力。

因此, custom harness 不代表每個 primitive 都要自建。較好的組合可能是:

1. 對 legal workflow、permissions 與 validation 使用自訂 orchestration。
2. 對 file editing 等模型熟悉的操作, 保留該模型原本適應的 tool implementation。
3. 切換 model 時, 同時選擇相應的 tool profile。

講者表示 Deep Agents 使用 model profiles, 依模型切換 file-editing implementations。這項設計假設不同模型已適應不同工具介面, 實際效益仍應用 task-specific eval 驗證。

## Evals 定義組織自己的 Good

[09:39](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=579s)

Model、context 與 harness 都能改變 agent behavior。若沒有組織自己的 benchmark, 團隊無法判斷一次修改是改善、退步, 還是只改變成本與 latency。

講者主張 mission-critical agent 應建立 private evals, 因為通用 benchmark 無法完整表達內部資料、流程、risk tolerance 與品質標準。Evals 可以用於:

- 捕捉 regressions。
- 比較 models、harnesses 與 reasoning effort。
- 驗證 middleware 或 prompt change。
- 在既定 benchmark 上持續 hill climbing。
- 同時比較 accuracy、latency、token usage 與 cost。

## Harbor Task 的基本結構

[11:04](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=664s)

影片以開源 eval runner Harbor 說明 task-based benchmark。每個 task 是一個可在隔離環境執行與評分的工作單元, 核心包含:

| 元件 | 用途 |
| --- | --- |
| Environment | 以 Dockerfile 等方式定義 agent 執行環境 |
| Instruction | 提供給 agent 的任務 prompt |
| Test / verifier | 執行 tests、程式、LLM judge 或 agent judge |
| Reference solution | 用來 sanity check task 是否可解 |

多個 tasks 組成 dataset, 再對不同 model 與 harness combinations 執行。Stateful 或 long-running tasks 通常各自在 sandbox 中運作, 也方便平行化。

Benchmark 不應只輸出單一 accuracy。影片展示的比較還包含 latency 與 tokens, 因為較高成功率如果需要顯著更多時間或成本, 不一定符合產品需求。

## Observability 要回答 Context 如何形成

[13:20](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=800s)

講者把 agent failure 粗分為兩個原因:

1. Model capability 不足。
2. Model 收到的 context 不足或錯誤。

依他的經驗, 第二種情況更常見。這是講者的實務判斷, 影片沒有提供量化比例。

Agent observability 不應只保留 final response, 還需要讓團隊追蹤:

- 每次 model call 實際收到的 context。
- Context 經過哪些 retrieval、summarization 或 offloading。
- Agent 執行過哪些 steps 與 tools。
- 每個 observation 如何影響後續 trajectory。
- Failure 發生前有哪些 errors、retries 或錯誤假設。

一般使用者介面可以折疊細節, 方便閱讀 trajectory。工程除錯介面則必須能展開完整 trace, 檢查每次 call 的 inputs 與 outputs。

## 從 Traces 建立 Data Flywheel

[14:34](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=874s)

影片將持續改善流程整理為:

```text
build agent
    ↓
run in real workflows
    ↓
collect traces and feedback
    ↓
curate recurring issues
    ↓
run controlled experiments
    ↓
update model, context, or harness
    └──────────────────────────→ repeat
```

最大困難之一是取得可用 feedback。使用者很少主動點選 thumbs up 或 thumbs down, 因此產品 UX 應讓行為自然產生 signals。例如使用者接受、修改、重試、撤銷或接手 agent output, 都可能比明確評分更接近實際品質。

系統也可以執行 online evaluators, 對 traces 產生 synthetic feedback。Evaluator 可以是 deterministic code、off-the-shelf model、custom prompted model 或 fine-tuned small model。若對每條 production trace 都使用昂貴 frontier model 當 judge, 成本可能無法接受。

經過整理的 trace data 可以用於三個方向:

- Harness engineering: 修改 middleware、checks、tools 或 control flow。
- Fine-tuning: 更新 model behavior。
- Memory and context: 改善 retrieval、instructions 與長期知識。

## LangSmith Engine Demo

[16:51](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=1011s)

LangSmith Engine 是一個在背景分析 LangSmith traces 的 agent。Demo 中, Engine 使用 LangSmith CLI、prompt 與 sub-agents 搜尋 traces, 將重複問題整理為 issue board, 並連回 supporting traces。Issue 還能提出 prompt、context 或 harness code 的修改建議。

這個 demo 支持以下能力已被產品化:

- Agent 能查詢並分群 traces。
- Issue 可以附帶來源證據。
- 建議可能同時影響 instructions、context 與 code。

但影片沒有展示 Engine 建議的 precision、recall、誤報率, 也沒有呈現修改被採用後的 production improvement。因此它是工作流程展示, 不是自動化改善效果的完整證明。

## Dogfooding 與 Codexification

[19:23](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=1163s)

Q&A 中, 講者表示 LangChain 讓 Engine 分析自己的 traces, 並透過 Slack 回報。團隊也建立 Harbor 格式的 `issue bench`, 持續比較不同 models 與 harnesses。

一次比較中, 團隊觀察到 Codex 會積極撰寫許多小 scripts 分析 traces, 且在 benchmark 上表現良好。團隊隨後進行所謂的「codexification」sprint, 把這項行為加入 Engine harness。

這是 data flywheel 的具體例子:

1. 在相同 benchmark 比較多個 agents。
2. 檢查高表現 agent 的 trajectory。
3. 找出可能造成差異的策略。
4. 將策略移植到自家 harness。
5. 再用 benchmark 驗證。

影片沒有公開 improvement 數字或完整實驗條件, 因此只能確認團隊採用這套程序, 不能量化其效果。

## Harness 會收斂還是分化

[20:44](https://www.youtube.com/watch?v=HI2q3ci3Iuc&t=1244s)

講者沒有斷言單一答案。他觀察到 general-purpose harness 已能處理許多 basic tasks, 因此適合作為起點。但隨任務偏離通用分布, 或 predictability 與 control 要求提高, harness 會逐漸分化。

金融服務客戶是其中一個例子。部分客戶認為 general-purpose agent 自主性過高, 更偏好明確 cognitive architecture 以控制執行路徑。

同時, model labs 可能都讓 coding harness 變得更強, 形成某種收斂。但不同 labs 已採用不同 file-editing patterns。未來若某個 model lab 深入 bio、legal 或其他領域, model 與 harness 也可能共同往不同 domain specialization 發展。

## 決策矩陣

以下是依影片內容整理的實務判斷, 不是講者逐字提供的表格:

| 條件 | 建議起點 |
| --- | --- |
| 通用任務、尚未確定 product fit | 使用 off-the-shelf harness |
| 任務大致通用, 但需要自訂 context、tools 或 summaries | 使用 hooks 或 middleware |
| 已知 failure patterns, 需要固定 checks 與 verifier | 加入 domain-specific gates |
| 高風險流程要求 predictability 與控制 | 建立較明確的 cognitive architecture |
| 整體 domain 特殊, 但局部操作屬於模型熟悉範圍 | 自訂 orchestration, 保留 model-native primitives |
| 無法從直覺判斷哪種設計更好 | 建立 private evals, 比較 accuracy、latency 與 cost |

## 實作檢查表

- 是否將 model、context 與 harness 的責任分開觀察?
- 現有任務與 model 或 off-the-shelf harness 的訓練分布有多接近?
- 是否能先以 middleware 修改 loop, 而不必完整重建 harness?
- 哪些 operations 已是特定 model 熟悉的 primitives, 應避免任意替換?
- 高風險步驟是否需要 deterministic gates、permissions 或 verifier?
- Private eval tasks 是否反映組織自己的資料、流程與 failure costs?
- 每個 task 是否有隔離 environment、instruction 與可重現 verifier?
- 比較 harnesses 時是否同時記錄 accuracy、latency、tokens 與 cost?
- Trace 是否能還原每次 model call 的 context 與完整 tool trajectory?
- Feedback 是否能從接受、修改、重試或撤銷等產品行為取得?
- Online evaluator 的成本與誤報率是否經過評估?
- 每項 harness change 是否回到相同 benchmark 驗證?

## 核心結論

自建 harness 的理由不應只是想取得更多控制, 而應來自可觀察的 distribution gap、failure pattern 或風險要求。通用 harness 適合快速開始, middleware 適合局部調整, cognitive architecture 則適合需要明確流程與高度 predictability 的場景。

真正讓組織累積 agent 能力的資產不只是 harness code, 還包括 private evals、完整 traces、feedback 與 institutional context。只有保留這些資料, 團隊才能判斷應修改 model、context 還是 orchestration, 並驗證每次改動是否真的改善結果。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。影片發布於 2026 年 8 月, 提及的 LangChain、Deep Agents、LangSmith Engine、Harbor、Codex、Claude Code 與 Claude Agent SDK 均可能快速演進, 實作前應查閱目前官方文件。

Harrison Chase 是 LangChain 的 co-founder and CEO, 對 LangChain 產品設計具有第一手責任, 同時也有推廣自家工具的利益。影片對 middleware、LangSmith 與 Engine 的描述可作為第一方產品行為說明, 但跨產品優劣仍需獨立 benchmark。

影片展示 architecture、workflow 與產品 demo, 沒有公開 Engine 的正式 evaluation、production success rate、misclassification rate 或成本改善數字。講者對 context failure、distribution 與 harness convergence 的部分判斷屬於實務觀察及產業推論, 不宜視為已證實定律。
