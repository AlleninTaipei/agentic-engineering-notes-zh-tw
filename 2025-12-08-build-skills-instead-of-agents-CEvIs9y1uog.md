# 不要重建 Agent, 改為建立 Skills

> 影片: [Don't Build Agents, Build Skills Instead - Barry Zhang & Mahesh Murag, Anthropic](https://www.youtube.com/watch?v=CEvIs9y1uog)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=CEvIs9y1uog  
> 發布日期: 2025-12-08  
> 片長: 16:22  
> Video ID: `CEvIs9y1uog`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Anthropic 的 Barry Zhang 與 Mahesh Murag 認為, agent 架構正逐漸收斂為通用模型, agent loop, 可寫入檔案與執行程式碼的 runtime。不同領域不再需要各自重建一套 agent scaffolding, 真正缺少的是能穩定執行專業工作的領域知識。

Agent skills 就是用資料夾封裝的程序知識。資料夾可以包含 `SKILL.md`, scripts, reference files, executables 與 assets。Agent 只在任務需要時載入相關 skill, 因此可以同時持有大量能力, 又不必把所有指令永久塞入 context window。

演講主張 MCP 與 skills 是互補關係: MCP 連接外部資料和工具, skills 則告訴 agent 如何運用它們完成工作。

## 為何不再為每個領域建立獨立 Agent

早期的假設是, 不同領域的 agent 會需要完全不同的工具與 scaffolding, 因此每個 use case 都要有專用 agent。講者認為, 隨著模型與 runtime 發展, 底層 agent 比原先想像得更通用。[00:35](https://www.youtube.com/watch?v=CEvIs9y1uog&t=35s)

這個轉變來自三項生態成熟度:

- MCP 成為 agent 連接外部系統的標準化介面.
- Claude Code 證明 coding agent 可以處理廣泛任務.
- Claude Agent SDK 提供可直接投入生產使用的 agent 基礎.

講者將新架構描述為模型與 runtime environment 的緊密結合。Agent 能讀寫檔案, 執行 shell 和程式碼, 因此不必為每個工作預先設計大量專用工具。

## Code 是數位世界的通用介面

演講提出 "code is all we need", 但其含義不是所有工作都只需要產生原始碼。更精確地說, code 可以成為 agent 操作數位世界的通用介面。[01:25](https://www.youtube.com/watch?v=CEvIs9y1uog&t=85s)

以產生財務報告為例, 通用 agent 可以:

1. 呼叫 API 取得資料並執行研究.
2. 在檔案系統中整理資料.
3. 使用 Python 分析資料.
4. 綜合洞察並輸出成所需檔案格式.

這些步驟都能透過程式碼和檔案系統完成。核心 scaffolding 因此可以縮減為 Bash 與 filesystem 等基礎能力。

## 真正缺少的是領域專業

高智力不等於專業知識。講者用報稅作類比: 與其讓一位數學天才從第一原理重新推導 2025 年稅法, 多數人會選擇熟悉規則與流程的稅務專家。[02:15](https://www.youtube.com/watch?v=CEvIs9y1uog&t=135s)

現有 agent 能力強, 但常有以下缺口:

- 重要的任務 context 沒有在一開始提供.
- 使用者必須投入大量提示與指導才能得到穩定結果.
- 專家的隱性流程難以完整傳入模型.
- Agent 不會自然把每次回饋保存成下次可重用的能力.

因此, 系統需要的不是另一個專用 agent 外殼, 而是可傳遞, 可修改且可重用的程序知識。

## Skill 的定義: 一個有組織的資料夾

講者將 agent skill 定義為 "organized collections of files that package composable procedural knowledge for agents"。也就是以有組織的檔案集合, 封裝可組合的程序知識。[03:03](https://www.youtube.com/watch?v=CEvIs9y1uog&t=183s)

最簡單的 skill 可以只有一份 `SKILL.md`, 較完整的 skill 則可包含:

```text
skill-name/
├── SKILL.md
├── scripts/
├── references/
├── assets/
└── executables-or-other-files/
```

這個格式刻意保持簡單, 讓人類和 agent 只要有一台電腦就能建立及使用。它也能沿用成熟的檔案工具:

- 使用 Git 做版本管理.
- 儲存在 Google Drive 等檔案服務.
- 壓縮成 zip 後分享給團隊.
- 由 agent 直接讀取, 修改或產生內容.

## 為何 scripts 也是 Skill 的重要部分

傳統 agent tools 可能有模糊的說明, 而且一旦工具行為不符合需求, 模型通常不能修改工具本身。工具定義與結果還可能長期占用 context。[03:45](https://www.youtube.com/watch?v=CEvIs9y1uog&t=225s)

相較之下, code 具有幾個優點:

- 可從內容理解用途, 具有一定程度的 self-documenting 特性.
- Agent 能直接修改與改善.
- 可以保留在檔案系統, 需要時才載入 context.
- 能將重複生成的步驟固定成一致流程.

演講中的例子是投影片樣式。團隊發現 Claude 一再撰寫相同 Python script 來套用樣式, 於是要求它把 script 存入 skill, 供未來的自己直接執行。這提高一致性, 也減少重複生成與推理成本。[04:19](https://www.youtube.com/watch?v=CEvIs9y1uog&t=259s)

## Progressive disclosure 保護 Context Window

如果 agent 同時擁有數百或數千個 skills, 不可能將所有完整內容都放入 context。Skills 因此採用 progressive disclosure, 也就是漸進式揭露。[04:45](https://www.youtube.com/watch?v=CEvIs9y1uog&t=285s)

載入流程可以理解為三層:

1. Discovery: runtime 先只向模型顯示簡短 metadata, 讓它知道有哪些 skills.
2. Instruction loading: 任務符合時, agent 讀取該 skill 的完整 `SKILL.md`.
3. Resource loading: Agent 再依 `SKILL.md` 指引, 按需讀取 scripts, references 或 assets.

這種設計同時達到兩個目標:

- Composability: Agent 可以持有大量 skills 並組合使用.
- Context efficiency: 與當前任務無關的內容不占用 context window.

## 三類 Skill 生態

講者表示, skills 發布五週後已形成數千個 skills 的生態, 並觀察到三種主要類型。[05:22](https://www.youtube.com/watch?v=CEvIs9y1uog&t=322s)

### Foundational skills

這類 skills 為 agent 增加原本不穩定或不存在的通用及領域能力。演講舉例:

- Anthropic 的 document skills, 用於建立和編輯專業 Office 文件.
- Cadence 的科學研究 skills, 用於 EHR 資料分析與常見 Python 生物資訊函式庫.

### Partner product skills

合作夥伴用 skills 教 Claude 更有效地操作自己的產品:

- Browserbase 為開源 browser automation 工具 Stagehand 建立 skill.
- Notion 建立 skills, 幫助 Claude 理解 workspace 並進行深入研究.

### Enterprise and team skills

企業將內部慣例, 專用軟體使用方式與工程規範封裝成 skills。講者提到大型企業和 developer productivity 團隊利用它們服務數千至數萬名開發者, 傳遞 code style 與內部最佳實務。[06:37](https://www.youtube.com/watch?v=CEvIs9y1uog&t=397s)

Skills 的重要特性是建立者不必是軟體工程師。財務, 招募, 會計與法務人員都能把自己的工作方式傳給通用 agent。

## Skills 正在從 Prompt 變成 Software

最基本的 skill 仍可以只是一份含 prompt 和簡單指令的 `SKILL.md`, 但複雜 skills 已開始封裝 software, binaries, scripts, assets 與其他檔案。[07:28](https://www.youtube.com/watch?v=CEvIs9y1uog&t=448s)

講者預測, 現在可能只需數分鐘或數小時建立的 skill, 未來可能像一般軟體一樣需要數週或數月開發及維護。

這也帶來軟體工程問題:

- 如何測試 skill 是否在正確任務觸發.
- 如何評估 agent 載入 skill 後的輸出品質.
- 如何追蹤 skill 版本與 agent 行為變化.
- 如何宣告其他 skills, MCP servers, packages 與 runtime dependencies.
- 如何確保同一 skill 在不同環境中仍有可預測行為.

因此, "skill 只是資料夾" 描述的是封裝介面簡單, 不代表內容不需要工程治理。

## MCP 提供連接, Skills 提供專業

Skills 並不是用來取代 MCP。演講明確區分兩者角色。[08:10](https://www.youtube.com/watch?v=CEvIs9y1uog&t=490s)

| 元件 | 主要責任 | 典型內容 |
| --- | --- | --- |
| MCP server | 連接外部世界 | Tools, APIs, external data |
| Skill | 封裝執行工作的專業方法 | Instructions, scripts, references, assets |
| Agent runtime | 管理模型與執行環境 | Agent loop, context, filesystem, code execution |

複雜 skill 可以協調多個 MCP tools, 把零散操作組成完整 workflow。例如, MCP 讓 agent 取得企業資料, skill 則定義要依什麼程序分析, 驗證與輸出。

## 通用 Agent 的收斂架構

講者觀察到 general agent 架構正收斂為四個部分。[09:04](https://www.youtube.com/watch?v=CEvIs9y1uog&t=544s)

```text
Model
  ↕
Agent loop and context management
  ↕
Runtime: filesystem and code execution
  ├── MCP servers: 外部工具與資料
  └── Skills library: 按需載入的程序知識
```

在這個架構下, 讓 agent 進入新領域, 可能只需配置合適的 MCP servers 與 skills, 不必重新建立整套 agent。

演講舉例, Anthropic 在發布 skills 後推出金融服務與生命科學方案, 各自搭配一組 MCP servers 和 skills, 讓同一個 Claude 更適合該領域的專業工作。

## Skills 作為企業的可演進知識庫

講者對企業應用的長期願景, 是由人與 agent 共同維護一個持續演進的能力知識庫。[11:27](https://www.youtube.com/watch?v=CEvIs9y1uog&t=687s)

Skills 保存的不是所有企業資訊, 而是執行特定任務所需的 procedural knowledge。當使用者提供回饋或新的制度知識時, agent 可以更新 skill, 使整個團隊後續使用的 agent 一起進步。

這可能形成累積效果:

1. 專家把工作程序編入 skill.
2. Agent 在真實任務中使用.
3. 使用者對結果提出回饋.
4. Skill 被修正, 測試與版本化.
5. 新進人員一開始就取得團隊累積的方法.

Skills 的分享還可以跨越組織。就像外部社群建立的 MCP server 能增加 agent 的連接能力, 社群建立的 skill 也能增加可靠的專業行為。

## 走向可轉移的持續學習

講者將 skills 視為 continuous learning 的具體步驟。標準格式提供一項保證: Claude 現在寫下的程序知識, 未來版本仍能有效使用。[12:43](https://www.youtube.com/watch?v=CEvIs9y1uog&t=763s)

Skills 讓 memory 變得更具體, 但它不是完整記憶系統。它主要捕捉可在特定任務中重用的程序知識, 不負責保存所有對話和資訊。

由於 skills 可由 agent 建立和修改, Claude 可以:

- 從反覆執行的工作中抽取新能力.
- 隨需求改變既有流程.
- 淘汰已經過時的 skills.
- 透過 in-context learning 立即使用更新內容, 不必每次重新訓練模型.

講者期望 Claude 與使用者合作到第 30 天時, 能比第 1 天更了解工作方式。當時 Claude 已可透過 skill creator skill 建立新的 skills。

## Processor, OS 與 Application 的類比

演講最後用傳統運算堆疊說明 agent 生態。[14:25](https://www.youtube.com/watch?v=CEvIs9y1uog&t=865s)

| 傳統運算 | Agent 生態 | 功能 |
| --- | --- | --- |
| Processor | Foundation model | 提供通用運算與智能潛力 |
| Operating system | Agent runtime | 協調 context, processes, resources 與資料 |
| Applications | Skills | 編碼領域專業與建立者的獨特觀點 |

少數公司可能建造模型與 agent runtime, 但大量開發者, 專家與組織可以建立 skills。講者認為, 最具創造力和具體價值的競爭會發生在這個應用層。

## 編輯整理: 何時建立 Agent, 何時建立 Skill

以下判斷表是根據演講論點整理, 不是講者逐字提供的規則。

| 需求 | 較適合的做法 |
| --- | --- |
| 缺少基本 agent loop, context 管理或 runtime | 建立或採用 agent platform |
| 缺少外部系統連接能力 | 建立 MCP server 或 API tool |
| 同一 agent 不知道如何完成特定專業流程 | 建立 skill |
| 重複產生相同 script | 將 script 保存進 skill |
| 團隊需要共享內部工作規範 | 建立企業或團隊 skill |
| 需要完全不同的安全或執行邊界 | 可能仍需獨立 agent 或 runtime |

影片標題的 "Don't Build Agents" 是方向性主張, 不是禁止所有 agent 開發。若底層 agent platform 不存在, 或任務需要特殊的安全, 合規與 runtime 隔離, 仍可能需要建立專用 agent。講者反對的是為每個領域重複建造相似 scaffolding。

## 建立可維護 Skill 的實務原則

1. 讓 description 清楚說明 skill 的適用時機, 以提升觸發準確度.
2. 把核心流程與資源導覽放在 `SKILL.md`, 大型細節移至 references.
3. 將重複且可確定執行的步驟寫成 scripts.
4. 只在需要時載入 references 和 assets, 保護 context window.
5. 使用 Git 追蹤 skill 與 agent 行為的變化.
6. 建立代表性任務, 測試觸發條件與最終輸出品質.
7. 明確宣告 MCP, package, runtime 與其他 skill dependencies.
8. 讓領域專家參與建立和驗收, 不把程序知識完全交由工程師猜測.
9. 從 agent 回饋中改善 skill, 但在分享前仍需人工審查.

## 重點回顧

- 通用模型加上 agent runtime 已能處理廣泛任務, 不必為每個領域重建完整 agent.
- Code 與 filesystem 是 agent 操作數位世界的通用介面, 領域專業才是主要缺口.
- Skill 是封裝程序知識的資料夾, 可包含 instructions, scripts, references 與 assets.
- Progressive disclosure 讓大量 skills 可以組合, 又不會同時占滿 context window.
- MCP 負責連接外部世界, skills 負責提供運用工具與資料的方法.
- Skills 越來越像軟體, 因此需要測試, eval, 版本管理與依賴宣告.
- Skills 可成為企業由人與 agent 共同維護的程序知識庫, 支援可轉移的持續學習.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 自動字幕多次將 Claude, Claude Code 與 Anthropic 辨識為相似讀音的其他單字, 本筆記僅在上下文足以確認時修正.
- YouTube metadata 的標題破折號出現編碼異常, 本筆記將其正規化為 `Don't Build Agents, Build Skills Instead - Barry Zhang & Mahesh Murag, Anthropic`.
- 影片沒有章節, 時間連結是依演講內容轉折人工選取.
- 影片提到的 skills 數量, 發布週數和產品狀態反映錄影當時情況, 可能已在影片發布後改變.
- "Code is all we need" 與 "stop rebuilding agents" 是講者的架構主張, 並不表示所有領域都不需要專用安全設計, 評估或 runtime.
