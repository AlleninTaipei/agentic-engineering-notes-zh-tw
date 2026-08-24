# Claude Code 如何運作: Coding Agent 的簡單迴圈、Tools 與 Context Engineering

> 來源: [How Claude Code Works - Jared Zoneraich, PromptLayer](https://www.youtube.com/watch?v=RFKCzGlAU6Q)
> 頻道: AI Engineer
> 發布日期: 2025-12-26
> 片長: 1:05:42
> Video ID: `RFKCzGlAU6Q`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

PromptLayer 創辦人 Jared Zoneraich 從外部研究與使用經驗拆解 Claude Code, 並和 Codex、Amp、Cursor 等 coding agents 比較。這不是 Anthropic 官方架構說明, 而是講者對公開行為、system prompt、工具介面和產品設計的分析。

演講將 coding agents 近期突破歸納成兩點:

1. 模型在 tool calling、instruction following 與錯誤恢復方面變得更強。
2. Harness 不再用大量 scaffolding 彌補模型缺陷, 而採用簡單 loop、少量通用 tools 與積極 context 管理。

Claude Code 的核心可概括為:

```text
Project instructions
  + User request
  + Small set of tools
  -> Model chooses tool call
  -> Harness executes tool
  -> Tool result returns to context
  -> Repeat until model stops calling tools
```

真正困難的不是寫出 `while` loop, 而是選擇工具、限制權限、管理 context、設計 feedback loops, 以及建立可重複的 evaluations。

## Coding Agents 為什麼突然變得可用

[00:00:00](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=0s)

Coding assistance 大致經歷以下變化:

| 階段 | 使用方式 |
| --- | --- |
| ChatGPT copy/paste | 人類在聊天視窗與 editor 間搬運 code |
| Inline editing | 在 IDE 內用指令改寫局部 code |
| IDE assistant | Agent 可搜尋 repository 並多輪修改 |
| Headless coding agent | 人類委派任務, agent 自行使用 terminal、修改 files、執行 tests |

PromptLayer 的團隊甚至建立一項內部規則: 如果工作能在一小時內用 Claude Code 完成, 就直接完成, 不再花時間排入正式 prioritization。講者將這視為小團隊處理大量產品 edge cases 的方式。

早期 autonomous coding agents 常失敗, 不一定因為 orchestration 不夠複雜, 而是模型尚未具備穩定工具使用與自我修正能力。當模型能力提高後, 相同的簡單 architecture 才開始產生可靠結果。

## 核心哲學: Give It Tools and Get Out of the Way

[00:07:54](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=474s)

Jared 認為 Claude Code 的設計哲學是「提供工具, 然後讓開」。開發者容易為每種 hallucination 或 failure path 加入新 prompt、新 branch 與新 workflow, 最終形成脆弱 scaffolding。

更適合 frontier models 的策略是:

- 保持 harness 簡單。
- 提供清楚、通用的 tools。
- 讓模型探索 repository 和執行環境。
- 容許模型觀察錯誤後自行修正。
- 不為今天的短期模型缺陷建立永久架構。

講者將原則概括為「less scaffolding, more model」。這不代表完全放棄 deterministic controls, 而是避免在人類無法預先列舉路徑的 general-purpose task 上硬寫複雜 DAG。

### Python Zen 與 Agent Architecture

Jared 引用 Python Zen 的幾項原則:

- Simple is better than complex。
- Complex is better than complicated。
- Flat is better than nested。

同樣的 engineering taste 適用於 agent harness。過度 nested 的 prompts 與 branching workflows 會增加難以觀察的狀態, 而單一 loop 和工具介面比較容易理解與替換。

## Project Constitution: `CLAUDE.md` 與 `AGENTS.md`

[00:10:40](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=640s)

Claude Code 使用 `CLAUDE.md` 保存 repository instructions, Codex 等其他 agents 常使用 `AGENTS.md`。這類 Markdown file 是最簡單的 project-specific adaptation layer。

與其建立複雜 repository indexing 或自動推斷所有 convention 的系統, 使用者和 agent 可以直接維護:

- Build 與 test commands。
- Repository structure。
- Coding conventions。
- Architectural constraints。
- 完成任務前的 checks。
- 不應修改的區域。

它們本質上是 prompt / context engineering。優勢是透明、版本化、可由團隊和 agent 共同修改。

## Master Loop 只有幾行概念

[00:11:25](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=685s)

講者將 Claude Code、Codex、Cursor Agent 和 Amp 的核心簡化為同一種 loop:

```text
while model response contains tool calls:
    execute requested tools
    append tool results to context
    ask model again

return final response to user
```

模型決定:

- 是否需要繼續。
- 下一步使用哪個 tool。
- 失敗後如何修正。
- 何時任務已完成。

這種 loop 過去難以運作, 因為模型無法可靠遵循 tool schema 或維持多步驟目標。近期模型的 instruction following 使許多原本需要 code enforcement 的行為可以先透過 prompt 建立。

## Claude Code 的核心 Tools

[00:12:11](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=731s)

演講整理的 tool set 包括 Read、Grep、Glob、Edit、Bash、Web Search、Web Fetch、Todo 和 Task。實際工具可能隨版本變動。

### Read

雖然 shell `cat` 也能讀檔, 專用 Read tool 可以限制回傳 token 數量, 避免大型 file 一次填滿 context。

### Grep 與 Glob

Claude Code 以傳統搜尋方式探索 repository, 而非所有情況都依賴 embeddings 或 vector database。這和人類在 terminal 使用 `grep`、`find`、Ripgrep 的方式接近。

講者並非主張 RAG 完全無用, 而是指出 general-purpose coding agent 能以文字搜尋取得很高的彈性, 並減少 indexing system 的複雜度。

### Edit

模型通常使用 diff 修改 code, 而非完整重寫 file。Diff 的好處包括:

- 輸出 tokens 更少。
- 修改範圍更容易 review。
- 降低重寫未相關內容時產生錯誤的機率。
- 更符合開發者理解 change 的方式。

### Web Search 與 Web Fetch

搜尋或擷取網頁可交給較快、較便宜的 model tier, 避免 master agent 消耗所有能力處理資料擷取。

### Todo

將長任務整理成可見步驟, 協助模型維持方向, 也讓使用者看到進度。

### Task / Sub-agent

把大型研究或讀檔工作委派給獨立 context, 再只把摘要結果帶回 main agent。其主要價值是 context isolation, 不只是 parallelism。

## Bash 是 Universal Adapter

[00:15:52](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=952s)

Jared 認為如果只能保留一個 coding agent tool, Bash 最接近完整解法。它能:

- 搜尋與讀寫 files。
- 建立暫時 Python scripts。
- 執行 tests、linters 和 builds。
- 啟動 local environment。
- 使用既有 CLI tools。
- 探索錯誤並反覆嘗試。

Bash 有另一項重要優勢: training data 豐富。模型看過大量 shell commands、error messages 和 debugging sessions, 因而比冷門、專用的抽象工具更熟悉。

專用 tools 仍有價值。例如 Read 可以控制 tokens, Grep 可以配合 sandbox 和輸出限制。設計重點是讓 tool 反映人類實際操作, 不要為模型發明缺乏 training examples 的特殊介面。

System prompt 也會加入細節規則, 例如先 Read 再 Edit、獨立操作可平行執行、含空白的 path 需要 quoting。這些通常是 dogfooding 後發現的高頻 failure patterns。

## Todo Lists: Structured but Not Structurally Enforced

[00:17:57](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1077s)

Claude Code 的 todo list 要求模型:

- 一次只維持一個 in-progress task。
- 完成後更新狀態。
- 遇到 block 或 error 時不要任意跳過。
- 把大型工作拆成較小 instructions。

講者認為有趣之處在於, 這些行為主要由 system prompt 與 tool description 引導, 並非全部透過 deterministic state machine 強制。

Todo item 可以包含 ID、title、status 和 evidence 等結構。結構提供可引用的穩定 state, 但模型仍保留調整步驟的彈性。

Todo lists 的價值包括:

- 強迫 agent 在動手前形成 plan。
- Crash 後能恢復工作。
- 使用者能觀察進度。
- 使用者可在執行中 steer。

UX 不會直接提高模型 intelligence, 但會提高人類監督與修正 agent 的能力。

## Todo 與複雜 DAG 的差異

[00:19:25](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1165s)

Todo list 描述預期工作, 卻不把每條 transition 寫死。複雜 DAG 則預先定義可走的 branches。

| Todo-guided loop | Deterministic DAG |
| --- | --- |
| 模型可依新資訊調整步驟 | 執行順序由 code 定義 |
| 適合開放、general-purpose tasks | 適合固定 deliverable 與已知程序 |
| 依賴模型 instruction following | 依賴 workflow engine |
| 彈性高, evaluation 較困難 | 可預測, 容易做 step-level testing |

影片問答補充, 不應全面淘汰 DAG。Travel itinerary 的 research 階段可能需要自由探索, 但最終 output formatting 可交給固定 DAG 或 deterministic tool。General-purpose coding 比較適合 simple loop; 固定業務流程則應混合兩種方法。

## Async Buffer 與 Context Compaction

[00:21:03](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1263s)

Terminal 和 tools 可能持續輸出大量資料。Harness 不應把所有 stdout、intermediate events 和 UI updates 立即塞入 model context。

演講提及 async buffer, 用來分離 I/O 與 reasoning。目的是:

- 保持 UI responsive。
- 控制何時將 tool output 交給模型。
- 合併或截斷重複 terminal output。
- 避免非必要資訊佔滿 context。

當 context 接近容量時, harness 需要 compaction 或 summarization。長期目標不是保留完整 transcript, 而是保存後續工作需要的 decisions、files、errors、progress 和 next steps。

## 信任模型, 但依靠 Sandbox 限制 Side Effects

[00:23:24](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1404s)

「依賴模型」不代表給予無限制系統權限。模型應能自由探索解法, 但 file system、network 和 commands 必須在明確 security boundary 中執行。

Sandbox 的功能是:

- 限制可讀寫 paths。
- 控制 network access。
- 阻止危險 commands 或要求 approval。
- 讓 agent 能多次試錯, 卻不輕易破壞 host environment。

模型能力愈強, agent 能採取的 action 愈多, sandbox 反而更加重要。Simple architecture 的 deterministic 部分應集中在 permission boundary、tool execution 和 audit trail。

## Sub-agents 的主要價值是 Context Isolation

[00:27:23](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1643s)

Sub-agent 可以處理 repository research、large files 或相對獨立的問題, 再把 compact result 回傳 main agent。

適合委派的工作具有:

- 清楚問題與 output contract。
- 不需要 main agent 全部 conversation history。
- 會產生大量 intermediate tokens。
- 可以獨立驗證或摘要。

不適合的工作則是高度依賴主線細微決策、需要頻繁同步的 task。過多 sub-agents 可能造成 coordination overhead, 並把錯誤摘要帶回主線。

## System Prompt 是高頻失敗的修正層

[00:31:55](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=1915s)

System prompt 不只描述人格, 更像 operating policy:

- 何時讀檔、搜尋或修改。
- 如何使用 todo list。
- 何時平行執行 tools。
- 遇到錯誤如何繼續。
- 如何處理 path、tests 和 user instructions。

產品團隊可以透過 dogfooding 找到重複錯誤, 再加入精簡指示。但每條規則都會消耗 context 並可能產生衝突, 因此不能把所有個案無限加入 master prompt。

## Skills: On-demand Context 與可重用 Workflow

[00:33:35](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2015s)

Skills 讓 agent 按任務載入特定知識和流程, 避免所有內容永久存在 system prompt。講者舉出的使用方式包括:

- 文件更新時載入 product 與 writing style。
- 編輯 Microsoft Office files。
- 執行特定 library 或工具的標準 workflow。
- 將既有專案操作方式重新封裝成 skill。

Skills 的優勢是 progressive disclosure。Agent 初始只知道 skill 名稱與 description, 需要時才讀取詳細 instructions 和 resources。

### Skills 的可靠性挑戰

[00:36:05](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2165s)

講者也分享 Claude 有時忽略 skills, 或未在適當時機選中它們。這反映兩種取捨:

- 將內容放在 global prompt, 可靠看到但 context 膨脹。
- 將內容放在 skill, 節省 context 但需要模型正確 discover 和 invoke。

改善方向包括:

- 寫清楚 skill description 與 trigger conditions。
- 在 user request 明確指定 skill。
- 對 critical workflow 不完全依賴自動 discovery。
- 追蹤 skill 是否被載入及結果是否改善。

## 「AI Therapist」問題: 不同工作需要不同 Harness

[00:39:21](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2361s)

Coding agent 成功後, 開發者容易把同一架構複製到所有 agent use cases。但 shell-heavy、tool-driven loop 適合 coding, 不一定適合長期心理支持、客服或固定文件生產。

這類比較凸顯 architecture 應跟隨 task:

- Coding 需要 repository exploration、tests 和 file edits。
- 固定 business workflow 可能需要 DAG 與明確 transition。
- 高風險人類互動需要 policy、memory boundary 和 escalation。
- 格式嚴格的 output 可由可獨立測試的 workflow tool 產生。

「相信模型」是 default heuristic, 不是拒絕所有 task-specific engineering。

## Codex、Amp 與 Cursor 的不同觀點

[00:42:14](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2534s)

### Codex

講者將 Codex 視為另一種 model-optimized coding agent。其 open-source harness 讓開發者能檢查 architecture, 也可能預示 model provider 直接發布與 agent endpoint 配套的方向。

### Amp

Amp 的產品取向包括:

- 不強調讓使用者選擇具體模型。
- 由產品依 workload 切換 model tier。
- 以 fast、smart、Oracle 等能力語意呈現選擇。
- 著重 agent-friendly environment 和 feedback loop。

Amp 的 handoff concept 是建立新 thread 並只帶入下一階段所需資訊, 而不是等待原 context 完整 compact。這類 fresh-context strategy 可能與 compaction 互補。

### Cursor

[00:45:03](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2703s)

Cursor 以 UI-first workflow 和速度見長。自家 Composer model 來自產品使用資料與特定 coding workload 的 optimization, 顯示 fine-tuning / distillation 仍可形成產品差異。

Cursor 同時讓使用者選擇 frontier models, 因為 fast implementation model 和強 planning model 可能適合不同工作。這和完全隱藏 model selection 的產品哲學不同。

這些 agents 沒有唯一最佳 architecture。它們分別在 transparency、speed、model choice、environment design 與 context strategy 上做出不同取捨。

## Context Management: Compact 與 Handoff

[00:47:07](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2827s)

長 session 最終都會遇到 context limit。兩種常見策略是:

| Strategy | 做法 | 風險 |
| --- | --- | --- |
| Compact | 摘要目前 thread 後繼續 | 摘要成本高, 可能遺漏狀態 |
| Handoff | 開新 thread, 只交付下一階段所需資訊 | 需要定義清楚交接 contract |

Handoff 類似軟體團隊交班: 不複製全部歷史, 而是提供 objective、current state、decisions、artifacts 和 next actions。Fresh context 可降低舊錯誤與無關探索造成的干擾。

## 如何 Evaluating Coding Agents

[00:48:42](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=2922s)

Public benchmarks 常被用於 marketing, 而且難以代表特定 repository 和 workflow。Jared 將 evaluation 分成三層。

### End-to-end Test

提供真實 task, 檢查 agent 是否完成。例如要求 headless Claude Code 搜尋指定 model provider 的最新、最大模型並回傳名稱。

優點是接近 user outcome。缺點是難以定位中間哪一步失敗, 且結果可能具有 nondeterminism。

### Point-in-time Test

保存半完成 trajectory 的 context, 檢查模型在該狀態是否選擇預期 tool call 或 action。

這能針對 planning decision 建立 regression test, 但 snapshot 會和特定 prompt、tool schema 或 model version 耦合。

### Backtest

先收集 production traces, 之後以新 model、prompt 或 harness 重跑歷史 tasks。這通常是最容易開始、最接近真實分布的方法。

## Agent Smell: 用行為指標做 Sanity Check

單看任務是否完成可能不足。Jared 提出「agent smell」概念, 觀察:

- Tool call 次數。
- Retry 次數。
- 完成時間。
- Token 與成本。
- 是否反覆搜尋相同內容。
- 是否在簡單 task 上產生過長 trajectory。

這些 metrics 不直接證明品質, 但能找出模型或 harness 更新後的異常。例如成功率不變, tool calls 卻增加數倍, 可能代表 agent 正在以低效率方式勉強完成工作。

## Rigorous Tools: 將 Determinism 移到可測試邊界

[00:52:01](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=3121s)

Simple master loop 可以保持彈性, 對格式和品質要求明確的部分則封裝成可獨立測試的 tool。

講者以特定風格 email 或 SEO article 為例。若 output 必須具有固定 sections、writing voice、links 和 conclusion, 可以建立內部 workflow:

```text
Generate draft
  -> Check required sections
  -> LLM-as-judge evaluates style
  -> Revise missing or weak parts
  -> Repeat until conditions pass
```

這類 tool 比讓 general agent 自由探索更容易測試。Tool 可視為 function, 有明確 input、output 與 evaluation suite。

架構因此形成兩層:

- 外層 agent loop 負責選擇下一步與調度。
- 內層 rigorous tools 負責高規格、可重複驗證的工作。

## Headless Coding Agent 作為更高階 Primitive

[00:55:11](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=3311s)

Headless Claude Code SDK 可以成為 pipeline 的一個 step。Jared 舉例, 用 GitHub Action 定期:

1. Pull 多個 repositories。
2. 檢查最近 commits。
3. 讀取 `CLAUDE.md` 判斷文件是否需要更新。
4. 修改 documentation。
5. 建立 PR, 交由人類 review。

這可能提高 agent 開發的 abstraction level。開發者不再為每項工作自行重建 search、edit、shell 和 retry loop, 而直接委派給成熟 coding agent harness。

代價是 control 與 observability 降低。對固定、重要 task, 仍可能需要直接呼叫 model 或建立更靠近 metal 的 workflow。

## 簡單 Loop 與 DAG 如何混合

[00:57:25](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=3445s)

問答中, 聽眾詢問不使用 DAG 時如何強制順序。Jared 的回答是依 task 區分:

- General-purpose coding 沒有固定解法路徑, 應更依賴模型與 simple loop。
- Travel research 會因城市不同而改變, 不宜用固定 DAG。
- Travel itinerary 的最終 output 有固定結構, 可以用 DAG 或 rigorous tool。
- 客服蒐集 name、email 等固定流程, 可保留 sequence enforcement。

最佳架構通常是 mix and match。模型負責開放探索, deterministic workflow 負責不可缺少的順序、格式和 safety constraints。

## 未來會只呼叫 Agent SDK 嗎

[01:00:15](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=3615s)

問答討論未來是否不再直接呼叫 LLM API, 而全面使用 headless coding agent 或 agentic endpoint。

支持理由:

- Agent SDK 已包含 loop、tools、context 和 retry。
- 開發速度更快。
- 可以自動跟上 frontier harness 改善。
- Reasoning model 本身也可理解成 provider-side 多步 inference loop。

保留直接 model calls 的理由:

- 特定 task 需要更細控制。
- 成本、latency 和 output schema 更容易預測。
- 更接近 model 的介面更容易做 narrow evaluation。
- General-purpose harness 可能帶入不需要的能力。

Jared 認為兩種路徑都可能存在。許多 builders 可能只使用 agentic endpoints, 但高控制需求仍會保留較低層介面。

## Spec-driven 與 Test-driven Development

[01:02:18](https://www.youtube.com/watch?v=RFKCzGlAU6Q&t=3738s)

對 coding agents 而言, tests 提供可執行 feedback loop, 因此通常能提高自主工作品質。Spec 和 plan 則降低 agent 對需求的猜測。

講者沒有宣稱所有 task 都必須採用 TDD。簡單 edit 可以省略正式 planning; 複雜工作則更適合:

- 先建立或確認 spec。
- 定義 tests 和 success criteria。
- 讓 agent 反覆執行直到通過。
- 保留人類 review。

基本原則是回到團隊認可的良好 engineering practices, 不因 coding agent 出現就放棄測試與設計。

## 五項核心 Takeaways

### 1. Trust the Model

在開放、general-purpose task 中, 優先讓模型探索, 不急著加入複雜 branches。

### 2. Simple Design Wins

使用小型 loop、平坦 architecture 和透明 files, 避免過度 scaffolding。

### 3. Bash Is All You Need

少量通用、模型熟悉的 tools, 通常勝過大量特製 tools。

### 4. Context Management Matters

Compaction、handoff、sub-agents、bounded tool output 和 skills 都在解決相同問題: 保留高密度、當下相關的 context。

### 5. Different Perspectives Matter

Claude Code、Codex、Amp 和 Cursor 代表不同合理取捨。未來甚至可能由多個 coding agents 互相 review、討論或分工, 而不是由單一產品壟斷所有任務。

## 實作檢查表

### Harness

- 能否用單一清楚 loop 說明核心 control flow?
- 是否只提供少量、通用且可測試的 tools?
- Side effects 是否受 sandbox、permissions 和 audit 控制?
- 固定流程是否被封裝成 rigorous tool 或 DAG?

### Context

- Read、Grep 和 terminal output 是否有 token bounds?
- Large research 是否使用 sub-agent 或 handoff 隔離?
- Compaction 是否保存 decisions、artifacts 和 next steps?
- Skills 是否有明確 descriptions 和 trigger conditions?

### Evaluation

- 是否保存真實 historical traces 以便 backtest?
- 是否有 end-to-end success criteria?
- 關鍵 planning decisions 是否可做 point-in-time regression?
- 是否監控 tool calls、retries、latency 和 cost 等 agent smell?

### Coding Workflow

- Repository 是否有簡潔、最新的 project instructions?
- Agent 是否先讀取再修改?
- 是否使用 diffs 並執行 tests?
- Headless automation 是否只建立 PR, 而非未經審查直接合併?

## 核心結論

Claude Code 的價值不在複雜 orchestration, 而在模型能力、簡單 loop、通用 tools、context 管理和安全執行環境的組合。

對 agent builders 而言, 最重要的判斷不是「是否使用 DAG」或「是否完全信任模型」, 而是把彈性與 determinism 放在正確層級。未知路徑交給模型探索, 已知規則交給可測試 code, 高風險 action 交給 sandbox 和 approval, 長期品質則交給 production traces 與 evaluations。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕與公開章節整理, 並非逐字稿。內容已移除口語贅詞、重複字幕、現場互動與產品宣傳, 以繁體中文重新組織。

這場演講未獲 Anthropic 官方背書。關於 Claude Code internals、system prompt、tool schema 與代號的描述, 來自講者的外部研究、公開資訊和產品觀察, 不應視為官方架構保證。Claude Code、Codex、Amp、Cursor 及相關模型更新快速, 工具數量、prompt、context 策略與產品能力可能已改變。部分第三方專案名稱可能被自動字幕誤辨, 本文只在上下文足夠明確時修正。
