# Agentic Engineering: 與 AI 一起工作, 不只是使用 AI

> 來源: [Agentic Engineering: Working With AI, Not Just Using It — Brendan O'Leary](https://www.youtube.com/watch?v=BEKc4P87XKo)
> 頻道: AI Engineer
> 發布日期: 2026-04-07
> 片長: 27:03
> Video ID: `BEKc4P87XKo`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

Kilo Code 的 Brendan O'Leary 將 agentic engineering 定義為「能清楚說明如何與 AI 分工」的工程實務。重點不是模型能多快產生程式碼, 而是工程師能否判斷哪些工作可以交付、哪些判斷必須保留, 並為 agent 提供正確 context、流程與驗證方式。

演講提出一套實用工作法:

1. 把 coding agent 當成反應快、知識廣、願意重做, 但缺乏商業與架構判斷的 junior developer。
2. 每個 session 只處理一項任務, 主動管理 context, 偏離方向時重新開始。
3. 採用 research -> plan -> implement 流程, 先理解系統, 再寫明可驗證計畫, 最後才改程式碼。
4. 用 modes、`AGENTS.md`、skills 和 permissions 定義 agent 的角色、專案知識與自主範圍。
5. 將 agent 的工作隔離後以 Git diff 和 pull request 方式審查, 不盲目接受輸出。

AI 不能取代工程思考。它會放大已完成的思考, 也會放大缺乏思考造成的錯誤。

## 從使用工具到與 Collaborator 工作

[00:00](https://www.youtube.com/watch?v=BEKc4P87XKo&t=0s)

Brendan 先提出一個問題: 如果被問到「你如何在工作中使用 AI」, 能否說出真實 workflow, 而不只回答「寫程式變快」?

真正需要描述的是:

- 哪些任務交給 AI。
- 哪些工作由人類保留。
- 如何決定兩者邊界。
- 如何提供 context。
- 如何驗證輸出。
- 發現方向錯誤時如何恢復。

多數團隊早已有人使用 AI, 因此問題不再是「要不要採用」, 而是是否只把它當 autocomplete, 或已建立可重複、可說明的協作方法。

### Coding AI 的三個階段

[01:23](https://www.youtube.com/watch?v=BEKc4P87XKo&t=83s)

| 時期 | 主要能力 | 工作模式 |
| --- | --- | --- |
| 2020 年代初 | 完成一行或部分 function signature | Autocomplete |
| 約 2022 年 | 根據描述產生整個 function | Chat-based code generation |
| 約 2025 年起 | 分解任務、找檔案、修改、執行測試並提交 PR | Agentic execution |

當模型只能建議 code 時, 它比較像加速工具。當模型能接受任務並在 repository 中採取多步驟行動時, 工作模式開始接近 collaborator。

Brendan 引用 Flask 創作者 Armin Ronacher 的觀點, 將轉變描述為: 我們不再只是使用 machines, 而開始與它們工作。

## 把 Agent 當成高速但缺乏判斷的 Junior Developer

[03:08](https://www.youtube.com/watch?v=BEKc4P87XKo&t=188s)

Brendan 建議使用以下 mental model:

> Agent 是精力充沛、非常樂意工作、讀過大量資料, 但常自信地犯錯的 junior developer。

它的優勢包括:

- 執行速度快。
- 不容易疲倦。
- 不會因被要求重寫而產生 ego。
- 熟悉大量 languages、frameworks 和 patterns。

它缺少的則是:

- 對 business context 的理解。
- 對團隊歷史決策的掌握。
- 對特定架構取捨理由的判斷。
- 辨識「技術上正確, 情境上錯誤」的能力。

因此 agentic engineering 不是接受所有建議, 而是 directing the work。工程師要知道可委派的部分, 並保留需要 judgment 的責任。

## Context Engineering 是第一核心能力

[05:01](https://www.youtube.com/watch?v=BEKc4P87XKo&t=301s)

Brendan 引用 Andrej Karpathy 對 context engineering 的描述: 這是一門把下一步所需資訊放進 context window 的藝術與科學。

### Context 有成本

每次 interaction 都可能重新傳入 chat history 與其他 input tokens。Context 愈長, 花費與 latency 通常愈高。

MCP servers 也會載入 tool descriptions。長期啟用大量與目前任務無關的 MCP, 會持續消耗 token budget。

### 更多 Context 不一定更聰明

演講指出, context 使用量超過約一半後, 品質可能開始降低。這是講者在實務上的概括, 不是對所有模型與任務都固定成立的硬性界線。

長 context 可能造成:

- 關鍵資訊 lost in the middle。
- 新舊指令互相矛盾。
- 已過期的假設持續影響輸出。
- Model 無法判斷哪些內容仍有效。

### Bad Context 會污染後續工作

[06:38](https://www.youtube.com/watch?v=BEKc4P87XKo&t=398s)

典型問題包括:

- 把兩個沒有實際關聯的任務塞進同一 session。
- Code comments 或過去指示已經過時。
- Agent 走錯方向後, 人類只在同一長對話中不斷更正。

最後一種情況尤其危險。即使人類明確表示先前方向錯誤, 原有錯誤模式仍留在 context 中, 之後可能再次出現。與其繼續累積否定指令, 更穩健的方式通常是整理正確狀態並開始新 session。

## 管理 Context 的四種操作

[07:32](https://www.youtube.com/watch?v=BEKc4P87XKo&t=452s)

### 1. Persist

將長期有用資訊放在 context window 外, 需要時再載入。例如:

- Scratch pads。
- Memory files。
- `AGENTS.md`。
- Research notes 與 plan files。

### 2. Select

只選擇目前步驟需要的資訊。可以明確引用相關 files、commits 或 terminal output, 並關閉與任務無關的 MCP servers。

### 3. Summarize and Compress

完成一段 debugging 或研究後, 把過程壓縮成已確認的問題、證據、決策和下一步, 不必保留所有探索細節。

### 4. Isolate

用不同 sessions 或 parallel agents 分隔獨立任務, 避免 context 相互污染。每個 agent 只承擔一項清楚工作。

這些操作與管理 junior engineer 十分類似: 提供正確 spec、說明哪些細節是 placeholder、限制工作範圍, 並在適當時機重新整理任務。

## Comic Sans Prototype: 錯誤 Context 是管理責任

[08:56](https://www.youtube.com/watch?v=BEKc4P87XKo&t=536s)

Brendan 分享早期擔任 engineering manager 的經驗。他用 Balsamiq 為 iPad 病史填寫功能建立 wireframe, 其中使用 Comic Sans 和滑稽圖示作為 placeholder, 再把設計交給 interns 實作。

幾週後收到的 prototype 真的使用 Comic Sans 和 placeholder icons。問題不在 interns, 而在 manager 沒有說明:

- Wireframe 哪些部分代表需求。
- 哪些視覺元素只是 placeholder。
- 真正要解決的 user problem。
- 最終成果的品質與設計標準。

相同原則適用於 agent。Agent 忠實模仿錯誤 spec 時, 不能把結果完全歸咎於模型。提供 context 與判斷資訊是委派者的責任。

## Session 使用習慣

[10:31](https://www.youtube.com/watch?v=BEKc4P87XKo&t=631s)

Brendan 將 context management 濃縮成三個習慣:

1. One task per session。
2. 留意 context meter。
3. 感覺方向偏離時, 相信判斷並開始新 session。

切換前可請目前 agent 摘要進度, 因為模型通常擅長為另一個模型撰寫 prompt。人類應先審查摘要, 確認它和自己的理解一致, 再用這份精簡 context 啟動新 agent。

## Research -> Plan -> Implement

[11:29](https://www.youtube.com/watch?v=BEKc4P87XKo&t=689s)

初次使用 coding agent 時, 常見做法是直接要求「幫我實作功能 X 和 Y」。模型很擅長快速輸出大量程式碼, 但如果沒有先理解系統, 就容易把錯誤假設轉成更多錯誤 code。

Brendan 推薦一個簡單的三階段 loop:

```text
Research
  -> Plan
  -> Implement
  -> Review / Feedback
  -> 必要時返回前一階段
```

他引用 Dex Horthy 的說法: 一條錯誤的 research 路徑, 可能變成數百行錯誤 code。

## Research: 先理解系統, 禁止過早修改

[13:11](https://www.youtube.com/watch?v=BEKc4P87XKo&t=791s)

Research 階段應使用只能詢問與閱讀、不能修改檔案的模式。Kilo 將其稱為 Ask mode。

這個階段要回答:

- 系統目前如何運作?
- 哪些 files 和 modules 真的相關?
- 現有 codebase 採用哪些 patterns?
- 新需求和既有功能有何相同與不同?
- Data 如何流過系統?
- 變更會影響哪些上下游?
- 有哪些 edge cases?

AI 擅長協助 brainstorm 潛在情況, 但 research 結果仍需輸出成 document, 由人類確認是否符合自己對問題的理解。

Research 的完成條件不是「模型已讀過很多檔案」, 而是人類與 agent 對系統現況和問題形成一致、可檢查的描述。

## Plan: 寫下 Scope、步驟與驗證

[14:37](https://www.youtube.com/watch?v=BEKc4P87XKo&t=877s)

人類核准 research 後, 才建立 implementation plan。Plan 應包含:

- 要建立或修改哪些 files。
- 每一步具體做什麼。
- 哪些內容 in scope。
- 哪些內容明確 out of scope。
- 測試需要新增或修改什麼。
- 要執行哪些 commands 驗證結果。
- 變更如何影響整體系統。

Code snippets 不一定是必要內容。Plan 的價值是讓下一階段不必重新做高層判斷。

Research 和 planning 做得足夠清楚後, implementation 甚至可能交給更小、更快或更便宜的模型。昂貴 reasoning 應集中在理解和決策, 而不是無差別地用於每一行 code generation。

## Implement: 用乾淨 Context 執行 Plan

[15:55](https://www.youtube.com/watch?v=BEKc4P87XKo&t=955s)

Implementation 應啟動新 session, 只提供核准後的 plan 和執行所需 context。這樣能:

- 降低 context 大小。
- 避免 research 中的死路影響執行。
- 逐步審查每項 change。
- 清楚比較實作是否偏離 plan。

Brendan 建議頻繁 commit。Git 可以作為人類與 agent 之間的第一層 pull request review:

1. Agent 完成一個小步驟。
2. 人類查看 diff。
3. 執行測試與驗證。
4. 接受後 commit。
5. 再進入下一步。

真正送給同事的 PR 之前, 先在本機用相同心態審查 agent 的工作。

## 人類時間的最高槓桿在 Research 與 Planning

[16:45](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1005s)

到了 implementation 階段, 困難的系統思考應該已經完成。人類最有價值的投入不是陪模型看每行 code 飛過, 而是在前面確認問題、架構、scope 和驗證標準。

演講引用 Dex Horthy 的另一個觀點:

> AI 無法取代 thinking。它只會放大你已做的思考, 或放大你沒有思考造成的缺口。

如果 research 和 plan 錯誤, 更快的 implementation 只會更快累積錯誤。

## 用 Modes、Rules 與 Skills 配置 Agent

[17:25](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1045s)

### Modes

Modes 定義 agent 當下角色與能力:

| Mode | 目的 | 建議權限 |
| --- | --- | --- |
| Ask | Research 與理解問題 | 讀取、搜尋, 不寫檔 |
| Architect | 建立 plan 與設計 | 讀取, 可寫 plan document |
| Code | 執行已核准 plan | 修改檔案、執行測試 |

角色與 permissions 配合, 可以避免 research 階段因模型太積極而提前修改系統。

### Workspace Rules

Repository 或個人環境應保存長期規則, 例如 conventions、build commands、testing requirements 和提交前檢查。Agent 只有在規則寫下且載入 context 後, 才能可靠遵循。

### Parallel Agents 與 Worktrees

團隊可以決定是否平行執行多個 agents, 並用 Git worktrees 隔離各自修改。工作完成後先在本機合併與審查, 再建立正式 PR。

### Auto-approval

應明確調整 agent 可自主進行的 action:

- 是否能讀取 workspace 內檔案。
- 是否能讀取 workspace 外資料。
- 是否能執行 tests 或 commands。
- 哪些 tools 必須人工核准。

權限不必永久固定。使用者可先採取保守設定, 隨著對模型和 workflow 的理解逐步調整。

## `AGENTS.md` 與 Skills 的分工

[19:19](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1159s)

Brendan 將 agent configuration 分成 modes、`AGENTS.md` 和 skills 三類。

### `AGENTS.md`: Always-on Project Context

`AGENTS.md` 正逐漸成為 agents 使用的 repository README, 保存每次工作都應知道的最小資訊:

- 專案 conventions。
- Build 與 test commands。
- Testing requirements。
- 提交前必須完成的 checks。
- 重要結構與限制。

因為它常被載入 context, 內容應保持 minimal, 避免把所有文件複製進去。

### Skills: On-demand Workflow Playbooks

Skills 用於特定、可重複的工作流程, 例如:

- 使用 Remotion 製作 motion graphics。
- 編譯每日、每週或每月 changelog。
- 執行某個團隊專屬的 release 流程。

它們通常按需載入。`AGENTS.md` 說明「在這個專案工作時始終要知道什麼」, skill 則說明「執行這類任務時應遵循什麼程序」。

## 加速日常互動的操作方式

[20:39](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1239s)

演講列出幾項 agent 工具常見的 power-user patterns:

- 以 `@` mention 加入 files、commits 或 terminal output。
- 用 slash commands 開新 task 或壓縮 context。
- 在 IDE 選取特定 code 後直接加入 agent context。
- 保留 autocomplete 處理小型、即時編輯。
- 從 CLI、mobile、cloud agent 或 Slack 委派工作。

這些介面不改變核心原則。無論入口在哪裡, 都需要隔離任務、控制 context、定義 permissions 並審查結果。

## MCP: 提供能力, 也消耗 Context

[22:10](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1330s)

MCP 擴展模型可使用的 tools。例如:

- GitHub MCP 可讀取 pull requests、comments 和 issues。
- Context7 可取得較新的 framework documentation。
- Database MCP 可查詢特定資料來源。

每個 MCP server 都需要把工具描述加入 system prompt 或其他 context。啟用與任務無關的 MCP 會造成兩種成本:

1. 浪費 input tokens。
2. 提供無關能力, 使 agent 誤判工作範圍。

例如純 frontend 任務不需要 Postgres MCP。關閉它不只節省 tokens, 也明確告訴模型目前不應碰 database。

## 將內部 Platform APIs 提供給 Agent

[23:41](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1421s)

對企業內部 APIs, Brendan 建議依複雜度與更新頻率選擇呈現方式:

| 情況 | 建議方式 |
| --- | --- |
| 已有 OpenAPI / Swagger spec | 直接讓 agent 使用現有 spec |
| 文件相對穩定 | 轉成 Markdown 放在 repository 中 |
| 文件頻繁更新 | 提供 reference URL, 工作時取得最新內容 |
| 多步驟、跨系統 workflow | 考慮建立專用 MCP server |

不應因 MCP 流行就預設所有整合都要建立 MCP server。簡單、穩定的 Markdown 或既有 API spec 可能更便宜也更容易維護。

## 隔離並像 Pull Request 一樣審查 Agent Work

[24:36](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1476s)

無論使用哪一種 agent, 都應把人類工作和 agent work 隔離。Review 心態應與審查 junior engineer 的 PR 相同:

- 修改是否符合研究結果和 plan?
- 是否遵守 repository conventions?
- 是否加入正確 tests?
- 是否意外擴大 scope?
- 是否留下難以維護的 shortcut?
- 是否有充分證據證明功能正確?

Agent 產生 code 的速度不應降低 review 標準。相反地, 更高產量需要更明確的驗證與 change boundaries。

## 建立個人 Agentic Engineering 方法

[25:46](https://www.youtube.com/watch?v=BEKc4P87XKo&t=1546s)

Brendan 最後建議選擇一套工具並累積大量實作經驗。Agentic engineering 同時是 art 和 science, 使用者需要透過 reps 建立直覺:

- 哪些任務可以信任模型完成。
- 哪些工作需要更明確 spec。
- 何時 context 已被污染。
- 何時要切換模型、mode 或 session。
- 哪些 action 可以自動核准。

Research -> plan -> implement 是容易開始的基礎 loop。熟悉後再逐步加入 parallel agents、worktrees、skills、MCP 或其他進階能力。

## 實作檢查表

### 開始任務前

- 這個 session 是否只有一項主要任務?
- 需要載入哪些 files、rules 和 tools?
- 哪些 MCP servers 與此任務無關, 可以關閉?
- Agent 現在應該 research、plan 還是 implement?

### Research

- 是否禁止 agent 過早寫 code?
- 是否確認現有 data flow、patterns 和相關 files?
- 是否列出 edge cases 與未知項目?
- 人類是否閱讀並核准 research document?

### Plan

- 是否列明 in scope 與 out of scope?
- 每一步是否對應具體 files 或 modules?
- 是否包含 tests、commands 和完成條件?
- 是否足以讓另一個乾淨 session 直接執行?

### Implement

- 是否使用新 session 和精簡 context?
- 是否逐步檢查 diff 並頻繁 commit?
- Agent 是否只執行 plan, 沒有重新發明 scope?
- 是否像正式 PR 一樣驗證結果?

### 長期維護

- `AGENTS.md` 是否精簡且正確?
- 重複 workflow 是否應封裝成 skill?
- Permissions 與 auto-approval 是否符合風險?
- 過時的 memory、rules 和 docs 是否定期清理?

## 核心結論

Agentic engineering 的真正轉變, 不是讓 AI 代替工程師寫更多 code, 而是讓工程師從逐行生產者轉為研究者、規劃者、context curator 與 reviewer。

高品質協作依賴清楚分工: 人類負責 business context、architecture、scope 和 judgment; agent 負責快速探索、重複執行與具體實作。Research 和 planning 愈紮實, implementation 才愈可能帶來真正的時間收益。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕整理, 並非逐字稿。影片未提供公開章節, 本文段落和時間戳依演講內容轉折編排。口語贅詞、重複字幕、現場互動及產品宣傳已刪除。

影片包含講者對 Kilo Code features 與 workflow 的介紹, 本文將可泛用的工程原則與特定產品範例分開表述。關於 context 使用量超過約 50% 後品質降低的說法, 應視為講者的實務經驗與簡化界線, 不宜套用為所有模型的固定規則。工具功能、`AGENTS.md` 慣例與 MCP 生態可能在影片發布後改變, 實際使用時應查閱當前文件。
