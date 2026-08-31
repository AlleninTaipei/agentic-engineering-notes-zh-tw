# 從 AI-Assisted 到 AI-Native: AWS Frontier Development 的五項團隊習慣

- 影片: [From AI-Assisted to AI-Native: Building a Frontier Development Team - Clare Liguori, AWS](https://www.youtube.com/watch?v=pqlWNihgdjI)
- 頻道: AI Engineer
- 講者: Clare Liguori, AWS Senior Principal Engineer
- 發布日期: 2026-08-28
- 片長: 20:57
- Video ID: `pqlWNihgdjI`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

AWS 在 Amazon 內部觀察不同工程團隊採用 coding agents 的結果, 發現是否使用同一套工具並不是生產力差距的主要解釋。約 50 個具有一般資歷分布、在既有系統上工作的團隊中, 半數團隊的 production deployment velocity 提升不到 3 倍, 另一半的中位數則達 4.5 倍, 個別團隊超過 10 倍。約 90% 的受觀察團隊都使用 Kiro, 差異主要出現在團隊是否刻意改變工作方式。

講者 Clare Liguori 將這種工作型態稱為 frontier development。Frontier developers 很少親自輸入程式碼, 讓 agent 長時間獨立執行, 並同時運行多個工作流。他們不是靠更頻繁地與 agent 對話取得槓桿, 而是先準備 context、改善 codebase 與工具, 明確寫下 intent, 再提供快速且確定性的驗證迴路。

AWS 從多個內部案例整理出五項習慣:

1. 投資並持續修剪 agent context。
2. 先減速改善環境, 才能在後續加速。
3. 餵給 agent 足夠資訊與驗證方式, 不要全程 babysit。
4. 先在文件中釐清 intent, 再產生程式碼。
5. Shift testing left, 建立快速的本機 feedback loop。

## Frontier development 的行為定義

[00:00](https://www.youtube.com/watch?v=pqlWNihgdjI&t=0s)

講者回顧 coding assistance 從 inline completion、chat、vibe coding 到 frontier development 的演進。依她個人的經驗, 前幾個階段只帶來約 10% 至 20% 的主觀生產力提升。Frontier development 則開始在 Amazon 內部 pilot 中出現數倍的 production delivery 改善。

她不用特定工具或職稱定義 frontier developer, 而是觀察三種行為:

| 行為 | 表現 |
| --- | --- |
| Hands-off coding | 工程師只親自撰寫約 1% 至 2% 的產出程式碼, 其餘交由 agents 完成 |
| Infrequent interaction | 目標是讓 agent 在沒有人工介入下持續執行, 最長可達數小時 |
| Minimize idle time | 同時運行多個 agents, 分別處理 backlog 中不同任務 |

這個定義強調工作系統的改變, 而不是單次 prompt 的品質。若工程師仍在每次生成後等待、閱讀並繼續對話, 人類注意力仍限制所有工作的 throughput, 難以利用平行執行。

## 三組內部案例與證據邊界

### Bedrock Mantle: 極高槓桿, 但團隊不具代表性

[02:34](https://www.youtube.com/watch?v=pqlWNihgdjI&t=154s)

Amazon Bedrock 的 Mantle 團隊需要建立新的 inference data plane, 包括遷移客戶與模型。原始估算為 30 人、18 個月, 最後由 6 人使用 Kiro 在 76 天內完成。團隊依 commits 等資料估計改善幅度最高可達約 20 倍。

講者立即指出重要限制: 這 6 人包含公司內頂尖工程師與兩位 Distinguished Engineers, 並具備 distributed systems、LLM 與相關架構的專業知識。這個案例證明極高槓桿可能發生, 卻不能回答一般團隊能否重現。

### Prime Video sprint: 可重現部分成果, 但環境經過高度準備

[04:51](https://www.youtube.com/watch?v=pqlWNihgdjI&t=291s)

Prime Video 安排 6 位工程師進行 10 天的 Kiro sprint。團隊根據 sprint 進展與過去 commit history, 將專案交付估算從 90 週降至 24 週。

這次參與者不是 Mantle 的同一組頂尖工程師, 但條件仍偏離日常工作:

- 工程師沒有 on-call 任務。
- 會議及其他干擾很少。
- 一位資深工程師事前花了 3 週, 將工作拆成小型、範圍清楚且具有詳細需求的任務。

因此, 這個 sprint 顯示 prepared environment 的效果, 但不能直接當成一般團隊長期 productivity 的估計。

### Amazon Stores pilot: 工具相近, 工作方法造成分化

[05:57](https://www.youtube.com/watch?v=pqlWNihgdjI&t=357s)

Amazon Stores 進行較接近日常工作的 pilot。約 50 個團隊具有一般的 early-career、mid-career 與 senior engineer 分布, 並在 brownfield systems 和既有 codebases 上工作。觀察時間涵蓋前一年的大部分期間。

這次不以 commits 作為主要 productivity metric, 而是測量 deployment velocity to production, 也就是變更送達客戶的速度。結果出現明顯分化:

- 約半數團隊提升不到 3 倍。
- 另一半的中位數約為 4.5 倍。
- 個別案例超過 10 倍。
- 約 90% 的團隊使用 Kiro, 也可能搭配其他內部工具。

訪談顯示, 高改善團隊會刻意改造日常工作流程, 低改善團隊則傾向把 agent 疊加在原有流程之上。這是影片的核心觀察: 工具使用率相近, 不代表團隊能取得相同槓桿。

## 五項日常習慣

### 1. 投資並修剪 agent context

[08:14](https://www.youtube.com/watch?v=pqlWNihgdjI&t=494s)

工程知識原本存在人的腦中, 並透過 Slack、onboarding、mentoring、code review、stand-up 與 sprint planning 傳遞。Agent 無法自然取得這些隱性知識, 團隊必須把它們寫入 skill files、steering files 或其他可存取的 context。

一個可持續的更新規則是:

```text
Agent 產生錯誤或不符合團隊做法的結果
  -> 找出它缺少的知識或限制
  -> 將可泛化的內容寫入 context
  -> 在後續任務中驗證是否改善
```

Context 不能只增不減。講者以 Sonnet 3.7 到 Opus 4.5 的模型變化為例, 說明舊模型需要的大量 `do not` 規則, 新模型可能已不再需要。過時 workaround 留在檔案中會占用 context, 甚至限制較新模型的能力。因此每次模型升級後都應重新問: 這條 steering instruction 還有存在必要嗎?

### 2. Slow down to speed up

[10:31](https://www.youtube.com/watch?v=pqlWNihgdjI&t=631s)

幾乎所有受訪團隊在刻意轉換工作方式的初期都回報 productivity 下降。Brownfield codebase 不會因安裝 agent 就立即變得適合自主工作, 團隊需要先進行真正的工程投資:

- 建立 agent 所需的 context。
- 改善既有工具的錯誤訊息, 讓模型知道失敗原因。
- 建立新工具或 MCP servers, 補足 agent 的操作能力。
- 重整 codebase, 讓 agent 更容易定位與修改內容。
- 在部分情況下, 重新評估程式語言或型別系統。

講者觀察到某些團隊從 Python 或 JavaScript 移向 TypeScript, 也提到 Rust 在 Amazon 內部受歡迎, 原因包括 compiler 能提供具體錯誤訊息, 讓 agent 更容易自我修正。她沒有主張所有團隊都必須改用這些語言, 而是把可測試性、型別與錯誤品質視為 agent-friendly environment 的設計因素。

### 3. Feed agents, do not babysit them

[12:50](https://www.youtube.com/watch?v=pqlWNihgdjI&t=770s)

持續與 agent 進行短週期來回對話, 會讓工程師留在每一步 execution loop 中。等待 30 秒至 1 分鐘、檢查輸出、補一句指示再繼續, 很難與多個 agents 平行工作。

較高槓桿的方法是預先提供:

- 需要完成的工作與範圍。
- 判斷完成的 quality bar。
- 編譯、執行與測試方法。
- Coverage 或其他驗證要求。
- 失敗時可用來診斷及修正的 feedback。

Agent 應先自行執行、驗證與修正, 到達預設品質條件或真正需要人類判斷時才返回。若相同要求會重複出現, 應放入 steering file, 而不是每次手動提示。

### 4. 在文件中先把 intent 說清楚

[13:58](https://www.youtube.com/watch?v=pqlWNihgdjI&t=838s)

常見的 vibe coding 流程是先給高階 prompt, 讓 agent 修改大量檔案, 然後再透過多輪對話修正需求與技術方向。講者認為, 當根本問題是 intent 錯誤時, 對分散於 codebase 的變更反覆修改效率很差。

對模糊或複雜功能, Amazon 工程師會先建立 specification。模型可以協助產生初稿, 但人與模型應先在單一文件中討論需求、行為與技術設計。文件比跨檔案 code changes 更容易比較、評論與修正。Intent 穩定後, 才讓 agent 執行大規模實作。

### 5. Shift testing left

[15:09](https://www.youtube.com/watch?v=pqlWNihgdjI&t=909s)

Agent 能長時間獨立工作的前提不是不犯錯, 而是能快速取得錯誤訊號並自我修正。團隊因此補強:

- Linters。
- Unit tests。
- Integration tests。
- Performance tests。
- Security tests。

一項具體做法是為外部服務建立能完全在本機執行、回傳 deterministic responses 的 mocks。若 agent 不必等待遠端服務、建立完整雲端環境或處理不穩定依賴, feedback loop 會更短, 相同時間內可進行更多次修正。

這些不是 AI 時代才出現的新工程原則。變化在於 agents 能反覆消費這些 feedback, 因此改善測試與工具的投資回報可能顯著提高。

## 團隊與組織的新成本

### Burnout、FOMO 與 cognitive load

[16:15](https://www.youtube.com/watch?v=pqlWNihgdjI&t=975s)

講者提醒, 採用五項習慣不代表工程組織會自動進入理想狀態。她觀察到一些工程師為了讓 agent 隔夜運行, 深夜反覆嘗試 prompt。多個 agents 並行也會增加 terminal tabs 之間的 context switching。

Review AI output 對部分人可能比親自撰寫更耗費認知。Senior engineers 已累積大量 review 經驗, early-career engineers 則可能尚未形成這項能力。若組織只擴張生成量, 卻沒有同時發展 review、整合及判斷能力, 新的 throughput 可能轉化成注意力壓力。

### 組織必須容許初期減速

[17:23](https://www.youtube.com/watch?v=pqlWNihgdjI&t=1043s)

領導者不能一方面要求團隊投入數月改善 codebase、context 與 feedback loops, 另一方面又因為「已經有 AI」而要求每月立即交付更多功能。Frontier development 的轉型成本需要明確預算。

講者也不建議在尚未找出有效 practices 時一次擴及整個大型組織。Amazon 先從 pathfinder team、結構化 sprint 與 50-team pilot 累積經驗, 2026 年的挑戰才是將做法擴展到下一批約 2,000 個團隊。過早全面 rollout 會讓大量團隊同時摸索, 卻沒有可用的組織 context 與成功模式。

### Decision speed 成為新瓶頸

[19:41](https://www.youtube.com/watch?v=pqlWNihgdjI&t=1181s)

當原本需要 9 至 12 個月的產品可能只需 1 至 2 個月完成程式碼, 過去相對不明顯的決策及 launch review 時間便成為主要延遲。如果決定是否開發產品需要 2 個月, launch approval 又需要 2 個月, 這 4 個月可能已超過實作時間。

因此 frontier teams 可能花更多時間做決策, 而不是寫程式。組織需要加快容易逆轉的決定, 並重新檢查 review processes 是否仍與新的實作速度相稱。這不等於取消高風險審查, 而是讓決策成本與可逆性、blast radius 及合規需求匹配。

## 可採用的轉型順序

以下是根據影片內容整理的實務順序:

1. 先用一至數個代表性團隊進行 pilot, 不直接要求全組織轉型。
2. 以 deployment velocity、lead time 或交付給使用者的結果衡量, 不只計算 commits 或生成程式碼量。
3. 記錄 agent 經常缺少的 context, 同時建立定期刪除過時規則的機制。
4. 改善 compiler、linters、tests、error messages 與本機 deterministic mocks。
5. 對複雜工作先確認 specification, 再讓 agent 長時間執行。
6. 定義完成條件與 self-validation loop, 讓人類從 execution loop 移到 decision checkpoints。
7. 為 review cognitive load、parallel-agent 上限與非工作時間建立團隊規則。
8. 取得穩定結果後再擴張, 並重新設計組織決策及 launch review 流程。

## 證據與限制

影片資料來自 Amazon 內部案例與 pilot, 講者直接參與 Amazon 的 agentic AI 工作, 但沒有提供完整研究報告、原始資料、統計方法或控制組。`4.5x`、`10x` 與 `20x` 使用的評估方式並不完全相同: Mantle 與 Prime Video 案例參考 commits 和專案估算, 50-team pilot 則使用 deployment velocity to production。這些數字不能直接互相比較, 也不能視為採用五項習慣後的保證成效。

50-team pilot 的訪談顯示工作方式與成果相關, 但影片不足以排除團隊能力、任務組合、codebase quality、管理支持或其他因素。Kiro 由 AWS 開發, 講者也主要參與該產品, 因此產品案例具有第一手價值, 同時存在商業推廣誘因。

部分產品名稱、模型版本、團隊名稱與數字來自英文自動字幕, 可能有辨識誤差。本文已使用影片 metadata 與上下文修正能可靠辨識的名稱, 未能取得的內部資料則不做額外推測。
