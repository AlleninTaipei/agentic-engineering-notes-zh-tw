# 為 Agent 演進 Claude API

> 影片: [Katelyn Lesse - Evolving Claude APIs for Agents, Anthropic](https://www.youtube.com/watch?v=aqW68Is_Kj4)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=aqW68Is_Kj4  
> 發布日期: 2025-12-04  
> 片長: 13:24  
> Video ID: `aqW68Is_Kj4`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Anthropic Claude Developer Platform 團隊負責人 Katelyn Lesse 以 Claude Code 為例, 說明 Anthropic 如何演進 API, 讓開發者能把模型能力轉換成高效能 agent 系統。

整套平台策略可分為三層:

1. Harness capabilities: 將模型新能力公開為可調整的 API 功能.
2. Manage context: 動態決定哪些資訊應進入, 留在或移出 context window.
3. Give Claude a computer: 提供安全沙箱與執行基礎設施, 讓模型能自主寫程式並執行結果.

演講的核心不是單一 API 功能, 而是 agent 效能取決於模型, context 與執行環境三者的共同設計。

## 第一層: 把模型能力變成可控制的 API

Anthropic 將服務對象描述為持續 "raising the ceiling of intelligence" 的開發者, 也就是希望取得模型最佳表現, 並建立高效能 agent 系統的人。[00:42](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=42s)

研究團隊訓練出新能力後, 平台團隊需要提供相應的 API primitive, 讓開發者能依產品需求控制這些能力, 而不是只能接受固定行為。

### Extended thinking 與推理預算

模型處理複雜問題的表現, 通常會隨可用推理時間增加。API 因此讓開發者決定任務需要較長推理或快速回答, 並以 token budget 控制模型可投入的思考量。[02:02](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=122s)

以 Claude Code 為例:

- 複雜除錯可配置較多推理資源.
- 簡單問題可選擇快速回答, 降低延遲與成本.

這代表 agent 不應讓每個步驟都使用相同推理強度。較好的設計是依任務難度, 風險與驗證成本動態分配預算。

### Tool use

模型除了生成文字, 也能可靠地判斷何時呼叫工具並提供參數。平台同時提供內建工具, 例如 web search, 以及自訂工具介面。[02:47](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=167s)

自訂工具的基本定義包含:

- Name: 工具的穩定識別名稱.
- Description: 工具用途與適用時機.
- Input schema: 模型呼叫工具時必須產生的結構化參數.

Claude Code 會透過工具讀取檔案, 搜尋檔案, 寫入內容與重新執行測試。工具介面把模型的推理轉換成可觀察, 可驗證的外部操作。

## 第二層: 管理 context window

演講將 context management 視為提高模型表現最重要的工作之一。模型需要在正確時間看到正確資訊, 但 coding agent 可能同時面對技術設計, 完整程式庫, 使用者指令與大量 tool calls, 不可能永遠把所有內容放進 context。[03:31](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=211s)

平台以 MCP, memory 與 context editing 分別處理資訊的取得, 外部保存和移除。

### MCP: 連接外部系統

Model Context Protocol 提供標準化方法, 讓 agent 與外部系統互動。以 Claude Code 而言, GitHub 或 Sentry 等服務可能包含目前 context 之外的重要資訊和工具。[04:16](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=256s)

MCP 的功能不只是增加 context。它讓 agent 能在需要時主動取得資料或執行操作, 避免使用者每次都將所有資訊手動貼入 prompt。

### Memory: 把可重用資訊留在 window 外

Memory tool 讓 Claude 將資訊保存在 context window 外, 並在真正需要時取回。演講中的第一版實作是由客戶端控制的檔案系統, 因此資料仍由開發者掌握。[05:02](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=302s)

適合保存的內容包括:

- 程式庫慣例與架構模式.
- Git 工作流程偏好.
- 長期有效, 但不必出現在每輪對話中的專案資訊.

Memory 解決的是資訊可能稍後有用, 但目前不值得占用 context 的問題。

### Context editing: 清除現在不再相關的內容

Context editing 用來移除已不相關的資訊。第一版主要清除舊的 tool results, 因為工具輸出可能很大, 而早期呼叫結果未必有助於後續回答。[05:50](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=350s)

Claude Code 在一個工作階段中可能呼叫數百次工具。若每次讀取的檔案與命令輸出都永久保留, context 很快會被歷史資訊占滿。

演講指出, Anthropic 在內部評估中結合 memory tool 與 context editing 後, benchmark 表現提升 39%。這是 Anthropic 自有 eval 的結果, 字幕未提供測試集名稱, 基準值或完整實驗設計, 因此不宜外推為所有 agent 工作負載都會得到相同幅度。[06:28](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=388s)

### 大 context window 仍需要管理

部分模型提供 100 萬 token context window, 但更大的容量不能取代內容管理。平台的方向是同時提供更大 window 與編輯工具, 並讓 Claude 理解剩餘空間, 根據容量調整行為。[06:52](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=412s)

可以將這三項機制整理如下:

| 機制 | 主要問題 | 資訊流向 |
| --- | --- | --- |
| MCP | 外部系統有需要的資料或能力 | 從外部系統按需取得 |
| Memory | 資訊稍後可能有用, 目前不需占用 window | 暫存至 window 外, 日後取回 |
| Context editing | 歷史內容已失去價值 | 從 window 移除 |

## 第三層: 給 Claude 一台電腦

演講的第三個方向是讓模型在安全環境中自主工作。Katelyn 的論點是, 若 Claude 可以寫程式, 又能執行同一份程式, 它就能反覆觀察結果並完成更專業的輸出。[07:24](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=444s)

這使 agent harness 的重點從堆疊大量固定 scaffolding, 轉為提供足夠的執行空間與安全邊界。

### Claude Code Web 與 Mobile 的基礎設施問題

本機 Claude Code 可以直接把使用者的電腦當作執行環境。Web 或 mobile session 則不同, 使用者可能啟動工作後離開, agent 仍需在遠端持續執行。[08:06](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=486s)

平台必須解決三類問題:

1. Secure environment: 未經使用者逐行核准的程式碼必須隔離執行.
2. Container orchestration: 大量同時執行的 session 需要可擴展的容器管理.
3. Session persistence: 使用者離開後工作仍持續, 返回時可以看到完整結果.

### Code execution tool

Code execution tool 讓 Claude 在由平台管理的安全沙箱中撰寫並執行程式碼。容器與安全隔離由平台處理, 開發者不必自行建造完整的執行基礎設施。[09:12](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=552s)

模型因而可以形成閉環:

1. 產生程式碼.
2. 在沙箱內執行.
3. 觀察輸出或錯誤.
4. 修改程式碼.
5. 重複執行直到符合目標.

這種自主性仍應受到沙箱, 權限, 資源限制與驗收條件約束。演講所說的 "let it do its thing" 是在安全環境中的自主執行, 不是取消所有控制。

## Agent skills: 為模型提供領域專業

只有電腦和工具還不夠。Agent 也需要知道組織希望它如何完成某類工作。Agent skills 因此把 scripts, instructions 與 resources 組成資料夾, 讓 Claude 根據使用者要求和 skill description, 判斷何時載入並執行。[09:55](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=595s)

演講對 MCP 與 skills 的區分是:

| 元件 | 提供什麼 | 回答的問題 |
| --- | --- | --- |
| MCP | 工具與外部 context | Agent 能存取什麼? |
| Skills | 指令, 腳本, 資源與領域做法 | Agent 應如何善用這些能力? |
| Code execution | 安全的程式執行環境 | Agent 在哪裡執行與驗證工作? |

三者結合後, MCP 提供存取能力, skill 提供專業方法, code execution 則提供實際運作空間。

演講以 landing page 為例。當使用者要求推出新功能的頁面時, Claude 可以辨識這是 web design 任務, 載入組織的 web design skill, 並遵循既有設計系統與版型。[10:42](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=642s)

## 平台未來方向

Katelyn 將未來工作重新歸納為三條路線。[11:34](https://www.youtube.com/watch?v=aqW68Is_Kj4&t=694s)

### 持續公開新的模型能力

研究團隊提升 Claude 後, API 需要同步提供可用介面, 讓開發者能在產品中控制並運用新能力。

### 讓模型更主動管理 context

未來工具會讓 Claude 更自主地決定:

- 哪些資訊現在應載入 context.
- 哪些資訊值得保存供日後使用.
- 哪些歷史內容應從 context 清除.

### 深化 agent infrastructure

若要讓模型擁有可持續工作的電腦, 平台需要持續改善 container orchestration, secure environments 與 sandboxing。這些不是附屬功能, 而是自主 agent 能否安全擴展的基礎。

## 編輯整理: 可落地的 Agent 架構

以下是根據演講內容整理的實作順序, 不是講者逐字提出的固定框架。

1. 依任務難度配置 thinking budget, 不要讓所有步驟使用相同推理成本.
2. 將外部動作定義為結構化 tools, 提供清楚名稱, 描述與 input schema.
3. 透過 MCP 按需取得外部資料, 避免把所有資訊預先塞入 prompt.
4. 把穩定偏好和專案慣例保存至 memory, 需要時再取回.
5. 定期移除過期 tool results, 避免 context 被歷史輸出污染.
6. 在隔離沙箱內執行模型產生的程式碼, 限制權限與資源.
7. 將組織方法封裝成 skills, 讓模型不只取得工具, 也知道正確工作方式.
8. 使用測試, schema, policy 與人工驗收檢查最終結果.

## 重點回顧

- Agent 效能不只由模型大小決定, 也取決於 API 控制面, context 品質與執行環境.
- Extended thinking 讓產品能依任務動態交換品質, 延遲與成本.
- MCP, memory 和 context editing 分別負責外部取得, 長期保存與主動清除資訊.
- 大 context window 不是保留所有歷史內容的理由, 有效管理仍能顯著影響表現.
- Code execution 使模型形成寫程式, 執行, 觀察與修正的閉環.
- Skills 提供領域方法, MCP 提供工具與資料, 沙箱提供安全執行空間.
- Agent autonomy 的前提是 orchestration, persistence, isolation 與 verification.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 自動字幕可能誤辨講者姓名, 產品名稱與技術名詞.
- YouTube metadata 中的標題破折號出現編碼異常, 本筆記將其正規化為 `Katelyn Lesse - Evolving Claude APIs for Agents, Anthropic`.
- 字幕將 Claude 多次辨識為相似讀音的其他單字, 本筆記只在上下文足以確認時修正.
- 39% 表現提升是演講所述的 Anthropic 內部 eval 結果. 影片未提供足以重現實驗的 benchmark 細節.
- API, model context size, memory, context editing, code execution 與 agent skills 的產品行為可能在影片發布後變更. 本筆記描述的是演講當時的內容.
