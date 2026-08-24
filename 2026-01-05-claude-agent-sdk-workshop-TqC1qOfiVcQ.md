# Claude Agent SDK 完整工作坊

> 影片: [Claude Agent SDK Full Workshop - Thariq Shihipar, Anthropic](https://www.youtube.com/watch?v=TqC1qOfiVcQ)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=TqC1qOfiVcQ  
> 發布日期: 2026-01-05  
> 片長: 1:52:25  
> Video ID: `TqC1qOfiVcQ`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Anthropic 的 Thariq Shihipar 透過觀念講解, 現場問答和 live coding, 示範如何用 Claude Agent SDK 設計 agent。他的核心主張是, agent 不應只被動接收一大包 context, 而要獲得足夠的工具, 自行搜尋資訊, 採取行動, 驗證結果並修正錯誤。

Claude Agent SDK 建立在 Claude Code 的 agent harness 上, 封裝 tools, prompts, filesystem, Bash, skills, subagents, compaction, hooks, memory 和 web search 等能力。講者尤其強調 Bash 與 filesystem, 因為它們讓模型能組合既有軟體, 保存中間結果, 產生 scripts, 檢查工作並把非 coding 任務轉換成可執行流程。

工作坊的重要結論不是 "多給模型一個 Bash 就完成了", 而是 agent 設計需要同時處理 context gathering, action, verification, permissions, sandboxing, state reversibility 與 production hosting。

## 從單次 LLM 功能走向 Agent

講者將 AI 應用大致分為三個階段。[00:00:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=0s)

| 形態 | 控制方式 | 例子 |
| --- | --- | --- |
| Single LLM feature | 單次輸入與輸出 | 將文字分類到固定類別 |
| Workflow | 開發者明確規定步驟, 輸入和輸出 | 標記郵件, 根據程式碼產生下一個編輯建議 |
| Agent | 模型自行建立 context, 選擇路徑與執行動作 | Claude Code 根據自然語言要求操作程式庫 |

Agent 的差異在於自主決策範圍。使用者描述目標後, agent 可以選擇多種工具與執行路徑, 並持續工作 10 至 30 分鐘或更久。模型不再只是 workflow 中的一個固定節點, 而是負責決定中間步驟。

## Claude Agent SDK 封裝了什麼

Claude Agent SDK 建立在 Claude Code 上。Anthropic 內部建立不同 agents 時, 發現團隊反覆重建相同元件, 因此將成熟的 harness 封裝為 SDK。[00:05:15](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=315s)

這個 harness 包含:

- Tools 與 tool execution loop.
- 核心 prompts 與狀態轉換指令.
- Bash 與 filesystem.
- Skills 與程序知識.
- Subagents 與平行工作.
- Web search 與 research.
- Context compaction.
- Hooks 與 deterministic checks.
- Memory 與持久資訊.
- 從 Claude Code 大規模使用中累積的 tool-error handling 經驗.

因此, 使用 SDK 的價值不只是少寫一個 API loop, 而是直接取得經過實際產品驗證的 agent 基礎設施。

講者列出的應用包括 software reliability, security, incident triage, bug finding, site and dashboard builders, Office 文件, legal, finance 與 healthcare agents。

## Anthropic 的 Agent 設計取向

工作坊將 Anthropic 的取向歸納為幾個強烈但非唯一的架構選擇:

1. 使用 Unix primitives, 特別是 Bash 與 filesystem.
2. 讓 agent 主動建立自己的 context.
3. 對非 coding 任務也使用 code generation.
4. 每個 agent 在本機或獨立 container 中運作.
5. 用簡單且通用的工具取代持續增加的專用 tools.

這是一套 opinionated stack。講者沒有聲稱所有 agent 都只能如此設計, 而是解釋 Anthropic 為何將這些選擇放進 Agent SDK。

## Bash 為何是強大的通用工具

講者將 Bash 稱為最強大的 agent tool, 甚至形容它是早期的 code mode 或 programmatic tool use。[00:42:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=2520s)

Bash 讓 agent 可以:

- 將 tool results 保存成檔案.
- 使用檔案作為 memory 或 scratchpad.
- 動態產生並執行 scripts.
- 用 pipe 組合多個程式.
- 使用 `grep`, `awk`, `tail`, `jq` 等 Unix 工具.
- 呼叫 `ffmpeg`, LibreOffice, package managers 與既有 CLI.
- 透過 lint, test 和實際執行檢查自己的工作.

若每遇到一個新需求就建立專用 search, lint 或 execution tool, tool 數量會持續膨脹。Bash 讓模型直接使用既有工具。例如, Claude 可以自行發現專案的 package manager, 執行 `npm run lint`, 或在缺少 linter 時提議安裝 ESLint。

### 非 Coding 任務也能使用 Code Generation

講者以郵件 agent 計算一週叫車支出為例。只提供 inbox search tool 時, 模型可能一次收到上百封 Uber 與 Lyft 郵件, 然後只能在 context 中逐一推理。

若有 Bash, agent 可以:

1. 執行 Gmail 搜尋 script.
2. 把結果保存成檔案.
3. 用 `grep` 擷取金額.
4. 用 script 加總.
5. 保留行號與來源, 回頭核對每筆數值.

影片也提到影片會議搜尋。Agent 可以用 `ffmpeg` 切割影片, 再使用 `jq` 等工具分析結構化結果。Code generation 在這裡是組合 API 和處理資料的手段, 不代表最終產品一定是 coding 工具。

## Workflow 與 Agent 可以共用 SDK

Agent SDK 不只適合開放式 agent。Anthropic 也用它建立 GitHub 與 Slack automations。[00:22:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=1320s)

例如 issue triage 表面上像固定 workflow, 但中間可能需要:

- Clone 程式庫.
- 搜尋相關程式碼.
- 啟動 Docker container.
- 重現問題並執行測試.
- 最後產生結構化分類結果.

外部輸入與輸出可以保持確定, 中間過程則交由 agent 自由規劃。因此 workflow 和 agent 不是互斥類型, 而是自主程度不同的設計選擇。

## 安全模型: Swiss Cheese Defense

授予 Bash 權限會同時增加能力與風險。Anthropic 採用多層防禦, 講者稱為 Swiss cheese defense。每一層都可能有缺口, 但多層疊加後降低完整攻擊路徑成立的機率。[00:29:30](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=1770s)

### 第一層: Model alignment

模型本身接受安全與 alignment 訓練, 降低主動執行危險行為或 reward hacking 的可能性。

### 第二層: Harness permissions and parsing

Harness 透過 permission rules, prompts 與 Bash AST parser 分析命令意圖。這比單純比對命令文字可靠, 也避免每個應用團隊自行重建複雜 parser。

### 第三層: Sandboxing

即使 agent 被惡意控制, sandbox 仍限制它能存取的 filesystem 和 network。生產環境應使用隔離 container, 避免直接提供個人電腦, 廣泛 secrets 或不受限制的網路。

安全設計的實務原則包括:

- Read-only 與 read-write 權限分離.
- 網路預設受限, 只開放必要目的地.
- Agent 使用短期且 scope 受限的 API keys.
- 必要時透過 proxy 注入 credentials, 降低模型直接看見 secrets 的機會.
- 後端仍執行 RBAC, 不把授權完全交由 prompt.
- 不把 YOLO mode 視為 production default.

## Tool, Script, Skill 與直接 Bash 的選擇

工作坊沒有給出所有情境通用的唯一規則, 而是把它視為 system design 問題。[00:50:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=3000s)

| 方法 | 適合情況 | 代價 |
| --- | --- | --- |
| Structured tool | 需要原子操作, 嚴格 schema 與權限保證 | 彈性較低, 需預先設計用途 |
| Script or CLI | 流程可重用, 需要 `--help` 與漸進式探索 | 需要維護程式與 runtime dependencies |
| Skill | 有大量可重複的專業指令和相關資源 | 需要 filesystem, Bash 與觸發設計 |
| Direct Bash or code generation | 查詢與轉換方式高度動態 | 權限, 驗證和沙箱治理較困難 |

講者以資料庫為例。若資料高度敏感, 只能接受有限輸入並遮蔽其他內容, 應使用受限 tool。若需要撰寫動態 SQL, Bash 或 code generation 更適合, 因為模型能保存 query, 執行, 讀取 error, 修改後重試。

自訂 CLI 應提供清楚的 `--help`, 讓 agent 先發現工具, 再按需取得 subcommands 和參數。這也是 progressive disclosure 的一種形式。

## Skills 與 File System Context

講者把 skills 視為 filesystem context engineering 的一種抽象。Skill 適合封裝需要豐富專業知識的重複任務, 例如 Office 文件或 front-end design。[00:46:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=2760s)

`CLAUDE.md` 通常承載目前專案普遍適用的指令, skill 則只在特定任務需要時載入。由於 skills 和相關 scripts 保留在 filesystem, 它們不必永久占用 context。

講者也提醒, skills 和 Claude Code 都是快速演進的新概念, 最佳實務仍在形成。他個人會大約每六個月重新檢視 agent code, 因為模型能力改變後, 先前必要的 scaffolding 可能已變成負擔。

他的創業建議是, 既然 AI 使程式開發速度提高, 團隊也應願意更快淘汰過時程式碼。新創公司的優勢是能立刻利用當前能力, 不必等待大型組織的長週期決策。

## Agent 的核心循環

工作坊以三個動作描述 agent 工作方式:

```text
Think or gather context
        ↓
Act with tools
        ↓
Observe and verify
        ↓
Use feedback to continue or finish
```

Agent 不一定要在每輪嚴格執行固定步驟。System prompt 可以要求它先讀檔案, 執行 lint 與檢查結果, 但應保留判斷空間。例如只讀問題不需要執行寫入後驗證。

模型的優勢是能讀取 errors 和 feedback 後調整路徑。Harness 的工作是提供可行工具, 明確限制和高品質回饋, 而不是以硬編碼流程取代所有推理。

## Context Engineering: 讓 Agent 自己找資料

講者不建議開發者每次猜測 agent 需要哪些資訊, 再一次性塞入 prompt。他更傾向提供搜尋與執行工具, 讓 agent 根據任務逐步建立 context。[00:58:15](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=3495s)

他的類比是, 如果一個人被關在房間裡完成任務, 他會希望獲得電腦和搜尋能力, 而不是只收到另一個人猜測可能有用的一疊文件。

這不表示完全不提供初始資訊。開發者仍需設計:

- Agent 一開始能看到什麼.
- 它有哪些 context gathering tools.
- 如何縮小搜尋範圍.
- 中間結果存放在哪裡.
- 何時使用 subagent, compact 或重新開始 context.

長 tool output 適合先保存至 filesystem, tool call 只回傳檔案路徑。Agent 之後可以用 `grep` 或其他工具搜尋, 也能重新檢查原始結果。

## Spreadsheet Agent 的設計練習

工作坊以 spreadsheet agent 討論 gathering context, action 和 verification。[00:58:15](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=3495s)

### 搜尋介面

試算表具有二維結構, 單純搜尋 "revenue" 可能無法同時確定年份和正確欄列。講者和現場提出數種介面:

- 讀取 headers 和 sheet names.
- 使用 cell ranges, 例如 `B3:B5`.
- 將 CSV 匯入 SQLite 後使用 SQL.
- 直接查詢 XLSX 背後的 XML.
- 使用 `grep`, `awk` 或 spreadsheet API.
- 先由另一個 agent 加上 KPI 與欄位 metadata.

設計原則是把問題轉換成模型熟悉的表示方式。若模型非常熟悉 SQL, 將資料轉成 SQL-queryable format 可能比發明專用 search language 更可靠。

### 採取行動

寫入可以使用與讀取相似的介面, 例如 range update, SQL mutation 或 XML editing。讀寫介面一致時, agent 較容易從觀察結果形成下一步動作。

### 大型資料

面對數百萬列或大量欄位, 不應把完整試算表載入 context。Agent 應像人類一樣先看少量 headers 和 rows, 再搜尋, 導航其他 sheets, 建立 scratchpad 並保存 references。

資料規模增加仍會降低準確度。良好工具能改善問題, 但不能消除大型搜尋空間的基本難度。

## Verification 應分布在整個循環

講者認為最佳 verification 優先使用 deterministic rules, 並在所有可行位置執行, 而不只在最後檢查一次。[01:12:45](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=4365s)

可用方法包括:

- Lint, compile 和 unit tests.
- Schema 與 null checks.
- Query size, row count 或 mutation limits.
- 在 tool call 前後驗證 permissions.
- 寫入檔案前確認 agent 已讀取該檔案.
- 最終輸出前執行完整驗收.

Claude Code 的例子是, 若 agent 嘗試寫入尚未讀取的檔案, harness 會回傳錯誤並要求先讀檔。這是 deterministic feedback, 模型可以理解錯誤後繼續工作。

當規則無法完整評估品質時, 可以使用新的 subagent 做 adversarial review。驗證 subagent 最好使用乾淨 context, 避免受到主 agent 原始推理污染。但 model-based verification 仍應排在可確定規則之後。

## 可逆性是選擇 Agent 任務的重要尺度

Agent 特別適合狀態容易還原的工作。程式碼有 Git history, 可以 checkpoint, diff 和 rollback。相較之下, computer use 操作外部網站時, 錯誤動作可能使狀態機變得更複雜。[01:15:30](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=4530s)

例如誤把 Pepsi 加入購物車後, 並非只按返回就能恢復, 還需要找到購物車並移除項目。若操作已送出訂單, 回復成本更高。

Agent 系統應盡可能加入:

- Atomic operations.
- Checkpoints 和 snapshots.
- Undo and redo.
- Transaction boundaries.
- Dry-run mode.
- 高風險動作前的人類核准.

能否將任務轉換成可逆 state machine, 是判斷 agent 適用性的重要問題。

## Subagents 用於隔離 Context 與平行工作

Subagent 適合執行需要大量中間工作, 但主 agent 只需要最終結果的任務。例如 spreadsheet agent 可讓不同 subagents 分別搜尋或摘要各 sheets, 再把精簡結果交回主 agent。[01:21:20](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=4880s)

常見用途包括:

- Search and research.
- 各資料分區的平行分析.
- Adversarial verification.
- 產生多個候選方案.
- 保護主 agent context 不受大量中間資訊污染.

平行 subagents 共享 Bash 和 filesystem 時會產生 race conditions。Agent SDK 的價值之一, 是封裝這些底層協調問題, 讓產品團隊專注在該派哪些 subagents, 而不是自行重建 process management。

## Context 不應無限制累積

講者本人使用 Claude Code 時很少讓 session 經歷多次 compact。他傾向頻繁清除 context, 因為程式工作的狀態已保存在 files 和 Git diff 中。新的 session 可以重新讀取目前狀態, 不必保留整段聊天歷史。[01:18:30](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=4710s)

非技術使用者不會理解 context window, 因此產品需要設計相應 UX:

- 在新任務開始時自動 compact 或 reset.
- 將長期偏好保存至 memory 或 files.
- 從工作成品本身重建狀態.
- 使用 subagents 隔離中間資訊.
- 避免 tool output 直接長期占用主 context.

重點不是追求填滿最大 context window, 而是讓目前 window 保持與任務相關。

## 用 Claude Code 快速原型化 Agent

講者建議先在 Claude Code 中原型化, 再將成功模式移到 Agent SDK。[01:35:10](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=5710s)

建議流程是:

1. 把目標 API, scripts, libraries 與資料放進可操作的 filesystem.
2. 建立清楚的 `CLAUDE.md`, 說明 agent 身分, 目標與可用資源.
3. 直接要求 Claude Code 完成代表性任務.
4. 觀察它如何搜尋, 產生 script, 執行和修正.
5. 找出成功軌跡中的必要 instructions 和 helper scripts.
6. 將它們整理成簡單的 Agent SDK application.
7. 加入 allowed tools, permissions, hooks, tests 與 sandbox.

講者強調 "simple is not the same as easy"。最終 agent 程式可能只有數十行, 但找出模型真正需要的介面, context 和驗證方式仍需要多次實驗。

## Pokémon Agent Live Demo

Live coding 使用 PokéAPI 與 Smogon 資料建立 Pokémon agent。這個領域包含大量 Pokémon, moves, types, counters 和 teammates, 適合比較固定 tools 與 code generation。[01:21:20](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=4880s)

講者先要求 Claude Code 根據 PokéAPI 建立 TypeScript library, 再提供 `CLAUDE.md` 說明 API 和 scripts directory。Agent 根據查詢產生 TypeScript script, 呼叫 API, 執行程式並分析結果。

對照版本則使用一般 Messages API, 逐一定義 `get_pokemon`, `get_species`, `get_ability`, `get_type` 和 `get_move` 等 tools。固定 tools 能完成查詢, 但 API 範圍擴大時, 很難預先為所有組合建立工具。

在查詢第二世代水系 Pokémon 和以 Venusaur 為核心的隊伍時, code-generation 版本可以搜尋大量資料, 保存中間結果, 找出 teammates 與 counters, 再產生分析 script。Demo 也出現模型直接依既有知識回答, 沒有完全遵循預期資料路徑的情況, 顯示 live agent 仍需要 hooks 或驗證規則確保它真的讀取指定資料。

## Hooks: 注入確定性與即時 Context

Hooks 是 Agent SDK 中的事件介面, 可在特定時點執行 deterministic logic。[01:41:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=6060s)

用途包括:

- 驗證 spreadsheet 每次修改後仍符合規則.
- 在每次 tool call 後加入使用者同時做出的變更.
- 若 agent 沒有執行必要 script, 回傳 feedback 要求重試.
- 在允許寫入或外部操作前執行 policy checks.
- 追蹤工具呼叫和輸出, 便於 monitor 與除錯.

Hooks 不一定取代模型推理。它們適合處理能寫成明確規則, 或必須即時注入的狀態變化。

## 從本機原型到 Production

Agent SDK application 可以有兩種部署方向。[01:38:00](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=5880s)

### Local application

像 Claude Code 一樣安裝在使用者電腦, 直接使用本機 filesystem 與 runtime。這減少伺服器端 container 成本, 但需要謹慎處理本機權限與資料。

### Hosted sandbox

每位使用者啟動隔離 sandbox, 由 Cloudflare, Modal, AWS, DigitalOcean 或其他 provider 管理。應用程式透過服務與 sandbox 溝通。

若 agent 需要動態 UI, 可以在 sandbox 中啟動 dev server 並暴露受控 port。Agent 修改前端程式後, 使用者介面即時更新。講者指出許多 site builders 採用相似模式。

## 成本與商業模式

講者指出, 當時 agent 的運作成本仍高, 因為需要使用較強模型並進行多輪操作。產品應優先解決客戶願意高價支付的困難問題。[01:43:30](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=6210s)

定價可採 subscription, usage-based 或混合模式。Claude Code 的例子是 subscription 內含 rate limits, 超過後採 usage-based pricing。

Monetization 應在早期納入 agent 設計, 因為 context, tool calls, subagents 和重試策略都會直接影響單次任務成本, 上線後再逆向壓低成本可能很困難。

## 超大型程式庫的策略

針對 5,000 萬行以上的 codebase, 講者承認 `grep` 會面臨限制。Semantic search 需要索引, 可能更脆弱, 而且模型未必像使用 `grep` 一樣熟悉自訂查詢介面。[01:50:31](https://www.youtube.com/watch?v=TqC1qOfiVcQ&t=6631s)

他觀察到大型 codebase 客戶常使用:

- 高品質且分層的 `CLAUDE.md`.
- 從正確子目錄啟動 agent.
- 清楚的 verification steps.
- Hooks 與 navigation links.
- 將任務範圍限制在相關模組.

這不表示 semantic indexing 沒有價值, 而是它本身有維護和模型適配成本。講者的通則是, AI 相關自訂基礎設施可能在幾個月內因模型或產品演進而需要重做。

## 編輯整理: Agent 設計檢查表

以下清單根據工作坊內容整理, 不是影片中的逐字模板。

### 任務適配

- 任務是否需要自由規劃, 或確定 workflow 已足夠?
- 輸出是否可驗證?
- 錯誤是否可逆?
- 使用 agent 的額外成本是否由問題價值支持?

### Context

- Agent 起始時真正需要哪些最小資訊?
- 它能否自行搜尋其餘資訊?
- 長 tool output 是否保存至 files?
- 是否需要 skills, memory, compaction 或 subagents?

### Tools and actions

- 哪些操作需要嚴格 structured tool?
- 哪些操作適合 script, CLI 或 direct Bash?
- CLI 是否提供 `--help` 和清楚 errors?
- 讀寫工具是否共用一致的資料表示?

### Verification

- 能否使用 schema, lint, tests 和 deterministic rules?
- 驗證是否出現在中間步驟與最終結果?
- 是否需要乾淨 context 的 adversarial subagent?
- Agent 收到錯誤後能否明確知道如何修正?

### Safety and production

- 是否使用隔離 container 或本機受限環境?
- Filesystem 與 network 是否採最小權限?
- API keys 是否短期, scoped 並受後端 RBAC 保護?
- 是否有 checkpoint, rollback 和 human approval?
- 是否監控 tool calls, costs, loops 和 stuck states?

## 重點回顧

- Agent 的核心是自行建立 context, 選擇路徑, 採取行動並根據結果修正.
- Claude Agent SDK 封裝 Claude Code 已驗證的 harness, 讓團隊專注在領域問題.
- Bash 與 filesystem 提供通用, 可組合且可檢查的執行介面.
- Structured tools 仍適合需要嚴格權限和 schema 的原子操作.
- Context engineering 應讓 agent 按需取得資料, 而不是無限制填滿 window.
- Verification 越 deterministic 越好, 並應分布在 agent loop 各階段.
- Reversibility, checkpoints 和 sandboxing 是安全自主操作的前提.
- Subagents 能平行工作並保護主 context, 但共享 runtime 時需處理競爭條件.
- 最快的原型方式是先讓 Claude Code 解決真實任務, 再提煉成 Agent SDK application.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 自動字幕多次誤辨 Claude, Claude Code, Anthropic, Git, Sentry, Smogon 與 PokéAPI 等名稱, 本筆記只在上下文足以確認時修正.
- YouTube metadata 標題中的分隔符出現編碼異常, 本筆記將其正規化為 `Claude Agent SDK Full Workshop - Thariq Shihipar, Anthropic`.
- 影片包含大量即興問答與 live coding. 本筆記保留設計推理和結果, 省略操作等待, 重複口語與未完成片段.
- 影片中的產品 API, model capabilities, Agent SDK, Claude Code, skills, hooks, sandboxing 和 pricing 描述反映錄影當時狀態, 可能已在發布後變更.
- 講者多次強調 agent engineering 仍偏向經驗與實驗, 許多 tool, skill, memory 和多 agent 最佳實務尚未定型.
