# Anthropic 如何使用 Claude Code: 大規模 Agentic Software Engineering

> 來源: [How Anthropic uses Claude Code: Agentic Software Engineering at Scale - Daisy Hollman](https://www.youtube.com/watch?v=shZgedW15vg)  
> 頻道: NDC Conferences  
> 講者: Daisy Hollman, Anthropic  
> 發布日期: 2026-08-11  
> 影片長度: 1 小時 25 秒  
> Video ID: `shZgedW15vg`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 核心摘要

Daisy Hollman 將 `agentic programming` 與完整的 `agentic software engineering` 區分開來. Coding agent 如果只有 repository 和 shell, 可以很好地完成小型或 zero-to-one 專案, 卻仍看不到專業軟體工程師每天使用的大量資訊:

- Team chat 與 ticket history.
- CI results.
- Production dashboards.
- Design docs 與 institutional knowledge.
- 內部 APIs、術語與團隊慣例.
- 其他 agents 正在進行的工作.

要讓 agent 不只寫程式, 而是參與完整軟體工程, 必須解決三件事:

1. Access: 讓 agent 能接觸完成工作所需的系統.
2. Knowledge: 把組織知識在正確時刻帶入.
3. Tooling: 建立緊密 feedback loops, 讓 agent 能快速發現與修正錯誤.

所有客製化最終都受 context window 限制. 因此, 真正能在大型 monorepo 或數千個 repositories 中擴展的方法, 必須接近 zero overhead: 不相關時不要占用 context, 只有命中條件時才注入必要資訊.

## 1. 從 Chatbot 到 Agent

[00:04:19](https://www.youtube.com/watch?v=shZgedW15vg&t=259s)

傳統 chatbot 的互動是人與模型輪流說話. Tool calling 出現後, 模型可以要求電腦執行操作, 取得結果後再決定下一步. 當 tool calls 從一次擴展為多次、連續且具有一定自主性時, 人們開始稱它為 agent.

Coding agents 的工具通常就是程式設計師熟悉的能力:

- Shell commands.
- File reads 與 edits.
- Compilation 與 tests.
- CI 相關操作.
- 執行程式並觀察結果.

講者刻意說「programmer tools」而不是「software-engineering tools」. 因為寫 code 只是工程工作的一部分, 後面仍有知識、協作、production context 與組織流程需要補足.

## 2. Tool Calling 其實非常樸素

Tool call 的核心是結構化文字. Harness 告訴模型可以輸出某種 JSON schema, 模型產生 JSON 後, 系統執行對應操作, 再把結果放回 context.

影片以 Claude Code 的 edit tool 為例. 模型提供:

- File path.
- Old string.
- New string.
- 是否預期多個 matches.

這本質上是嚴格的 find-and-replace. Old string 必須逐位元組匹配; 若出現多次且未事先聲明, 操作可能失敗. 工具很原始, 但新模型已能可靠產生非常長、巢狀且精確的 JSON edits.

這帶出兩個觀察:

- Agent 能力不只來自工具複雜度, 也來自模型能否精確使用簡單工具.
- 目前工具仍遠未達到人類 IDE 的即時回饋與語意操作能力, 其中存在很大的 developer-tooling 空間.

## 3. Agent Task Horizon 正在延長

[00:10:34](https://www.youtube.com/watch?v=shZgedW15vg&t=634s)

講者引用 METR 的研究趨勢, 說明模型能以約 50% 成功率完成的任務時長曾呈現每約四個月翻倍的趨勢. 她也明確指出, 長任務到底等同多少人類工時越來越難測量, 圖表後段的不確定性很大.

她另引用 Mozilla Foundation 的案例, 表示在某個月份使用新模型修復的 security vulnerabilities 超過前 15 個月總和. 這些資料用來支持一個假設: agents 已能完成真實、具價值的維護工作, 而不只是產生 prototypes.

影片接著邀請觀眾思考一個假設情境: 如果 agents 最終能比你寫出更好的 code, software engineering、harness design 和 context engineering 應該如何改變?

這是講者用來推進論證的假設, 並非影片證明的必然結果.

## 4. 為什麼一般智能仍需要客製化

[00:15:56](https://www.youtube.com/watch?v=shZgedW15vg&t=956s)

講者的核心句是:

> If Claude can't do everything you can do, it can't do your job with you.

中文意譯: 如果 Claude 無法做你能做的所有事情, 它就無法真正和你一起完成你的工作.

即使模型能力很強, 它仍無法從訓練權重知道:

- 某家公司獨有的 code conventions.
- 團隊兩季前做過但未公開的實驗.
- 上週才改變的流程.
- Internal APIs 與 vocabulary.
- 某個設計決策背後的歷史原因.

因此客製化的目的不是補償模型不夠聰明, 而是縮小「模型預設知道的內容」與「團隊實際知道的內容」之間的差距.

### 一日 Terminal 實驗

講者建議嘗試一整天不離開 terminal 工作. 每當你必須開啟 Slack、CI dashboard、internal docs 或其他 UI, 就代表 agent 目前也缺少那一部分 access.

如果人需要複製 CI error 貼回 prompt, 表示 harness 尚未讓 agent 自行取得 CI 結果. 如果人必須閱讀 chat thread 後再轉述, 表示資訊仍由人充當 transport layer.

## 5. In-context Learning 就是控制文字

大部分組織客製化發生在 text space, 而不是修改 model weights. 講者用 `in-context learning` 稱呼把必要文字放進 context, 並戲稱其實就是 text files.

這對軟體工程師很有利:

- 不必理解模型權重或訓練數學.
- 可以用既有的 version control 管理規則.
- 可以 review、測試、更新和回復 prompts.
- 可以依任務選擇載入哪些資訊.

輸出雖不完全 deterministic, 但團隊仍可透過控制輸入文字與 feedback loops, 系統性提高結果品質.

## 6. Agent 的「紅色波浪線」

[00:23:25](https://www.youtube.com/watch?v=shZgedW15vg&t=1405s)

人類使用 IDE 時, 錯誤會在輸入當下以 red squiggle 出現. Agent 通常只收到 raw text, 而且可能等到最後 compilation 才發現前面的錯誤, 浪費 context 和 tokens.

Claude Code 的 post-tool-use hook 可以在 tool 執行後附加額外回饋. 例如 edit 完成時立即執行:

- Type checking.
- Linting.
- 針對修改檔案的專案規則.
- 與 `CLAUDE.md` 或團隊標準相關的檢查.

若此時就把具體錯誤與修復提示放進 tool result, 模型能在思路仍新鮮時修正.

講者的原則是:

> The fastest way to make an agent better at your codebase isn't necessarily a smarter model. It's a tighter feedback loop.

中文意譯: 讓 agent 更適合你的 codebase, 最快的方法未必是換更聰明的模型, 而是縮短回饋迴圈.

### 能隨智能擴展的工具

有些工具靠禁止操作來補償使用者能力不足, 當使用者變成熟後便成為限制. 另一類工具只提供提醒和即時資訊, 能同時幫助 junior、senior 和更強的模型.

講者建議優先建造第二類工具: 提供高品質 signal, 讓 agent 自行判斷何時是錯誤、何時是合理例外.

## 7. Context Window 是基本容器

[00:26:44](https://www.youtube.com/watch?v=shZgedW15vg&t=1604s)

模型能力迅速提升, 但 context window 沒有以相同速度增長. 任務時間跨度越長, 越需要仔細選擇放入哪些資訊.

Context 中可能包括:

- System prompt.
- Tool definitions.
- `CLAUDE.md`.
- Skills 與 subagent descriptions.
- 已讀取的 files.
- Conversation history.
- Tool calls 與 results.

所有預先載入的客製化都會減少真正執行任務的空間. 大型 codebase 不可能整份直接塞入 context, 文件、規則與工具描述也同樣不能無限制載入.

### Zero-overhead 原則

講者借用 C++ 的 `don't pay for what you don't use`: 不相關的知識不應占用 tokens. 有效的客製化必須在任務需要時才展開, 否則 context 很快會被靜態規則耗盡.

## 8. KV Cache 對 Context Engineering 的限制

[00:31:24](https://www.youtube.com/watch?v=shZgedW15vg&t=1884s)

Next-token prediction 如果能重用相同前綴的 KV cache, 成本會低很多. 若每一步都在前段 context 中插入、移除或重排規則, 後續 tokens 的 cache 可能失效, 造成顯著額外成本.

這表示 context management 不能直接照搬一般 LRU cache:

- 不能每次只留下「目前最相關」的規則, 隨意交換前綴內容.
- 需要兼顧 relevance、context 空間與 prefix stability.
- Dynamic injection 應盡量發生在不破壞大量既有 prefix cache 的位置.

影片提到早期 Cursor rules 曾採較動態的替換方式, 後來發現成本很高. 這項敘述是講者的技術回顧, 本文未以外部資料驗證其實作細節.

## 9. Plugin Primitives 的擴展性

講者以大型 monorepo 或數千個 repositories 為尺度, 比較 MCP、skills、subagents 與 hooks. 關鍵問題不是「能不能用」, 而是「有十萬個時是否仍能用」.

| Primitive | 主要用途 | Context 成本 | 擴展限制 |
| --- | --- | --- | --- |
| MCP | 提供結構化外部 tools | Tool name、description、schema 可能常駐 | 工具太多會擠滿 system prompt |
| Skill | 延遲載入 instructions 與 resources | Description 常駐, body 按需載入 | 大量 descriptions 仍會累積 |
| Subagent | 在獨立 context 中執行工作 | Description 常駐, 結果摘要回主 context | 大量 descriptions 仍占空間 |
| Hook | 事件發生時執行 script 或 agent | 未命中時幾乎不注入 context | 需正確設計快速退出與觸發條件 |

## 10. MCP: 結構化工具, 但不是所有內部整合的首選

[00:35:17](https://www.youtube.com/watch?v=shZgedW15vg&t=2117s)

MCP 以 JSON-based protocol 暴露工具, 對 client 來說具有 transport-agnostic、server-side authentication 等優點. 若要讓各種 chatbots、agents 或第三方 clients 使用公開整合, MCP 很適合.

但企業內部開發環境通常不是「任何 client、任何環境」. 如果公司已有成熟 CLI, 講者認為建立一個教 agent 如何使用 CLI 的 skill, 往往比重新建立 MCP server 更直接.

### 為何大量 MCP servers 不會自然擴展

每個 tool 需要 name、description 和 schema. 假設有 20 個 MCP servers, 每個提供 15 個 tools, system prompt 便可能被工具定義占據相當比例.

### Tool Search

[00:37:20](https://www.youtube.com/watch?v=shZgedW15vg&t=2240s)

Tool search 只先載入 tool names, 再讓 Claude 搜尋完整工具定義. 這比一開始載入所有 schemas 更省 context, 但仍有取捨:

- Name 太短或含糊, agent 不知道何時應搜尋.
- Name 越描述性, 常駐成本越高.
- 工具總數極大時, names 本身仍會占用大量 context.

因此 tool search 改善了擴展性, 但沒有創造免費的無限工具空間.

## 11. Skills: Lazy System Prompts

[00:38:48](https://www.youtube.com/watch?v=shZgedW15vg&t=2328s)

Skill 是包含 `SKILL.md` 與可選 resources 的資料夾. Description 告訴模型何時該載入, 完整 body 則在需要時才展開. 講者稱它為 `lazy system prompt`.

這種設計符合 pay-per-use:

- 不需要時不支付完整 instructions 的 context 成本.
- 需要時可讀取完整流程和 supporting resources.
- 簡單資料夾形式便於 version control 與分享.

但每個 skill description 仍會載入. 在數十萬個 skills 的極端規模下, descriptions 也會填滿 context. 影片提到 skill hierarchy 與 skill search 是正在探索的方向, 並指出 skill search 比 tool search 更難, 因為模型比較容易知道「我沒有某個工具」, 卻不容易知道「我缺少某項知識」.

## 12. Skills 與 Subagents 的差別

兩者都有一段常駐 description, 差別在完整工作發生的位置:

- Skill: Body 載入目前的 context, 屬於 in-context expansion.
- Subagent: Prompt 和工作在另一個 context 中執行, 主 agent 只取得摘要.

Subagent 適合隔離大量探索或平行工作, 能避免主 context 被所有細節占滿. 但若企業定義成千上萬個 subagents, descriptions 同樣會造成靜態成本.

## 13. Hooks: 最接近 Zero-overhead 的 Primitive

[00:44:31](https://www.youtube.com/watch?v=shZgedW15vg&t=2671s)

Hook 在特定事件發生時執行, 例如:

- Tool use 後.
- User 提交 prompt 時.
- Model 停止時.
- Context compact 時.

Hook 可以先在 context 外執行 script, 判斷事件是否相關. 若不相關便立即退出, 不把任何新內容放進 context. 只有命中條件時才進行處理並回傳資訊.

例如 JavaScript lint hook 遇到 Rust 專案時只需快速退出. 它消耗少量 CPU, 卻不必為十個不相關 JavaScript skills 支付常駐 token 成本.

Hooks 也是前面「red squiggles for agents」最適合的實作位置. 必要時 hook 也能呼叫另一個 agent, 但講者提醒這可能消耗大量 tokens, 應謹慎使用.

## 14. 為何 Plugin 不提供無條件 `CLAUDE.md`

講者常被問到為何 plugin 沒有直接注入 `CLAUDE.md` 的抽象. 原因是這違反 zero-overhead 原則: 每個 plugin 都在所有 sessions 開始時無條件放入大量文字, 幾個 plugins 就可能耗掉可觀 context.

問題在於對 plugin author 來說, 新增一份說明檔很便宜; 對每位使用者的每次 session 來說, 卻是永久成本.

若真的需要, 可以使用 session-start hook 注入, 但這會使「每次都付出成本」變得明顯, 促使作者更謹慎評估.

## 15. Memory 不等於 Context Engineering

Agent memory 本質上通常也是模型寫入並在稍後讀取的 text files. 但講者主張應將兩者概念分開:

- Context engineering: 人或系統設計的、可持續且可重用的 context 機制.
- Memory: 模型自行整理與選擇的紀錄.

即使 memory 與 skill 形式相似, 它們的來源、可靠性與治理方式不同. 超大型軟體工程環境必須明確區分哪些規則是團隊認可的 durable knowledge, 哪些只是 agent 自己留下的暫時記憶.

## 16. 多 Session 平行工作

[00:50:02](https://www.youtube.com/watch?v=shZgedW15vg&t=3002s)

單一 agent 的速度再高, 下一步規模化仍要靠 parallelism. 講者認為 2026 年的重要問題是: 如何讓開發者有足夠 cognitive space, 在多個 sessions 之間快速切換.

### Git Worktrees

每個 session 使用不同 worktree, 可以避免 agents 同時修改同一 working directory. Worktree 類似讓每個 agent 擁有獨立的 developer workspace.

### Rename 與 Color

替 session 命名並配置顏色, 可以降低切換時的認知負擔. 顏色提供快速的視覺提示, 幫助人回想該 session 正在做什麼.

### Persistent Agent Identities

講者的個人設置包含長期存在的 worktrees, 每個都有固定名稱和 agent. Agent 負責維護自己的 branch、scratch files 與 upstream main tracking.

這種設計讓她以 tech lead 的方式理解工作: 不同「成員」在一天中回報進度, 而不是一堆沒有身份的臨時 terminals.

## 17. Agent Teams

[00:53:39](https://www.youtube.com/watch?v=shZgedW15vg&t=3219s)

Agent teams 的基本 primitive 是讓 sessions 互相傳訊. 可以採用:

- Leader 將工作委派給 teammates.
- 多個 peer agents 自行協作.
- Agent 發現重要資訊後選擇分享給其他 sessions.

跨 agent 溝通也是 access 問題. 如果人能看到另一個 agent 的結果, 當前 agent 也應在授權下能取得它. 影片提到跨機器 session 通訊是產品發展方向, 但具體可用狀態可能已隨版本改變.

## 18. `/loop` 與長時間等待

[00:55:10](https://www.youtube.com/watch?v=shZgedW15vg&t=3310s)

影片中的 `/loop` 讓模型安排週期性訊息, 類似 cron. 它可以每隔一段時間喚醒 session, 檢查 CI 或其他延遲結果, 完成後再自行關閉.

這解決一種常見 failure mode: agent 已提交工作, 但在外部結果回來前 session 停止, 必須由人稍後輸入 `continue`. 若 agent 能安排自己的下一次喚醒, 人便不必充當排程器.

週期性 loop 仍會消耗 tokens, 必須有明確停止條件, 避免無限輪詢.

## 19. Auto Mode 與安全審查

[00:56:20](https://www.youtube.com/watch?v=shZgedW15vg&t=3380s)

Auto mode 是大量並行 sessions 的重要基礎. 它減少人一直按 Enter 批准操作的需要, 並使用多層 models/classifiers 檢查命令是否危險、是否得到 prompt 授權.

講者將它與無限制的 YOLO mode 區分, 但也指出 auto mode 會增加約 10% 到 40% 的模型成本, 視模型而定.

即使有 safety classifier, auto mode 也不應被理解為零風險. Repository、secrets、外部帳號和可執行命令的權限邊界仍需由團隊設計.

## 20. 人類 Attention 是最小的 Box

[00:58:06](https://www.youtube.com/watch?v=shZgedW15vg&t=3486s)

講者總結, 2025 年的重點是把更多資訊放進模型, 補償 agent 能力不足. 隨模型變強, 2026 年的瓶頸逐漸轉為如何把 agent 的狀態與結果有效呈現給人.

> Your attention is the smallest box in the system.

中文意譯: 整個系統中最小的容器, 是人的注意力.

因此下一代工具應優化:

- 如何一眼看到所有 active sessions.
- 如何判斷哪個 agent 需要人類輸入.
- 如何以極短摘要呈現剛完成的工作.
- 如何在 sessions 間快速跳轉.
- 如何從手機或其他裝置遠端管理 agents.

影片展示的 fleet view 會把多個 Claude Code sessions 放在同一畫面, 以輕量 classifier 提供狀態摘要. Remote control 則讓講者能從手機操作 cloud server 上的 persistent agents.

## 四種 Primitive 快速選擇

| 需求 | 優先考慮 |
| --- | --- |
| 提供跨 clients 的標準化外部工具 | MCP |
| 說明如何執行一項可重複工作 | Skill |
| 隔離大量探索或平行任務 | Subagent |
| 在明確事件發生時提供即時回饋 | Hook |
| 公司已有成熟 CLI | Skill + CLI, 未必需要另建 MCP |
| 只有特定語言或檔案類型才需規則 | 快速退出的 Hook |
| 規則每次 session 都不可缺少 | 謹慎使用啟動注入, 並衡量永久成本 |

## 可立即採用的 Context Engineering 原則

- 盤點一天工作中離開 terminal 的所有原因, 找出 agent 缺少的 access.
- 不要把整套企業知識無條件塞進 system prompt.
- Skill body 按需載入, description 保持具體而精簡.
- MCP tool 太多時使用搜尋或縮小連接範圍.
- 用 post-tool-use hooks 提供即時 type/lint feedback.
- Hook 先判斷 relevance, 不相關時快速退出.
- 將 team-approved knowledge 與 model-curated memory 分開治理.
- 大型任務用 subagent 隔離 context, 主 agent 只接收摘要.
- 多 session 使用獨立 worktrees、名稱和顏色.
- 長時間 loop 要有停止條件與成本限制.
- 優化人看到 agent 狀態的介面, 因為 human attention 才是最終瓶頸.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 影片沒有公開章節. 本文的章節與時間標記依內容轉折建立.
- 自動字幕將 `Anthropic`、`Claude Code`、`Codex`、模型版本與部分工具名稱辨識成近音詞. 本文只在上下文可可靠確認時修正.
- Agent task horizon、Mozilla security fixes、context window 歷史、KV-cache 成本差異與 auto-mode 額外成本等數據來自講者現場陳述, 本文未用外部來源獨立驗證.
- Tool search、skill hierarchy/search、跨機器 agent messaging、fleet view 與 remote control 等產品能力可能隨 Claude Code 版本迅速變動. 本文記錄發布當時的演講內容, 不等同目前官方規格.
- 講者是 Claude Code 團隊成員, 對工具設計有第一手經驗, 同時也可能帶有產品團隊視角.
- 中文標題、比較表、選擇指南與實務原則是編輯整理. 本文不是逐字稿, 也不取代完整演講或最新官方文件.
