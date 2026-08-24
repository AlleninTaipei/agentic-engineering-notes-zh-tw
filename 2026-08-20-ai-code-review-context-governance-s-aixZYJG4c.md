# AI 生成程式碼的信任問題: Code Review、Context 與治理層

> [!info] 來源
> - 影片: [The Last Human Code Review: Building Trust in AI-Generated Code — Itamar Friedman, Qodo](https://www.youtube.com/watch?v=s-aixZYJG4c)
> - 頻道: AI Engineer
> - 發布日期: 2026-08-20
> - 片長: 18:54
> - Video ID: `s-aixZYJG4c`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

當 AI 讓寫程式變快, 瓶頸會移到驗證與治理。要逐步減少人工 code review, 不能只換用更強模型, 而要把架構、標準、服務契約、事故經驗與審查決策轉成可追蹤的 context, 同時為人類與 agent 提供可稽核介面。

## Code review 為什麼存在

講者把 code review 的價值分為兩類 ([00:00](https://www.youtube.com/watch?v=s-aixZYJG4c&t=0s)):

1. 品質與安全驗證: 確認程式碼安全、可維護、符合架構與團隊慣例。
2. 對齊與學習: 讓資深工程師在上線前傳遞判斷、修正方向並教導團隊。

若要自動化 pull request review, 新流程仍必須覆蓋這兩類工作。它不一定要在傳統 PR comment 中完成, 但品質關卡與知識傳承不能直接消失。

## 兩種風險哲學

面對 AI 生成程式碼, 影片描述工程團隊光譜的兩端 ([03:38](https://www.youtube.com/watch?v=s-aixZYJG4c&t=218s)):

| 取向 | 核心信念 | 主要風險 |
| --- | --- | --- |
| 每行都需信任 | 人類必須詳細審查, 防止缺陷進入正式環境 | Review throughput 成為瓶頸 |
| 快速發布、快速修復 | Velocity 優先, 問題上線後迅速處理 | 事故成本、使用者傷害與回滾壓力 |

大多數團隊位於中間。適合的位置取決於系統風險、回復能力、法規、資料敏感度與故障成本。自動化策略應由這個風險立場推導, 不能只以程式碼產量決定。

## 講者主張: 模型不是唯一瓶頸, context 才是

影片主張, 現有模型若得到正確 context, 已能對程式變更做出相當好的推理 ([05:19](https://www.youtube.com/watch?v=s-aixZYJG4c&t=319s))。缺少 context 時, 即使模型能力強, 也容易產生泛化建議, 例如一律詢問是否考慮 error handling, 卻不知道該項問題在目前服務中是否關鍵。

這是一項講者觀察, 不是「模型能力已完全解決」的普遍證明。Code review 的準確度仍受模型、工具、repository 規模、程式語言、測試、攻擊面與評估方法影響。

## Context 為何四散且不一致

團隊可能同時維護 `AGENTS.md`、`CLAUDE.md`、skills、MCP server、RAG 資料與各工具自己的規則 ([07:03](https://www.youtube.com/watch?v=s-aixZYJG4c&t=423s))。常見問題包括:

- 不同 agent 讀取不同 instruction file。
- 組織、子團隊與 repository 採用不同結構。
- Coding agent 與 reviewing agent 使用不同 context。
- IDE、CLI 與背景 workflow 的設定不一致。
- MCP 或檢索來源改版後, 缺少 versioning 與回歸 eval。

結果是相同變更可能因入口不同得到不同審查結果, 團隊也很難回答某次 review 到底使用了哪些規則。

## 關鍵知識存在人的腦中與聊天紀錄裡

正式文件只保存部分工程知識。大量判斷仍存在資深工程師的經驗、Slack 或 Teams 討論、事故處理與歷史 PR 中 ([08:48](https://www.youtube.com/watch?v=s-aixZYJG4c&t=528s))。

這些 tribal knowledge 可能包含:

- 某種實作曾造成正式環境事故。
- 某個 microservice contract 不能隨意更改。
- 某段程式看似可重構, 實際上受舊客戶相容性限制。
- 哪些告警可以接受, 哪些必須阻擋上線。
- 團隊在過去 review 中反覆接受或拒絕哪些模式。

若這些資訊沒有被整理, AI reviewer 只能看見 diff 與一般程式知識, 無法重現團隊真正的工程判斷。

## Codify knowledge 的目標

講者認為, 自動審查需要 context lake 或 context engine, 將規則、標準與經驗整理成可查詢資產 ([09:42](https://www.youtube.com/watch?v=s-aixZYJG4c&t=582s))。

這套知識必須同時服務兩種對象:

- 人類需要易讀說明、來源、規則連結與稽核能力。
- Agent 需要結構化、可定位且能直接用於修正工作的 context。

只為模型建立冗長機器指令, 人類難以維護與信任。只建立傳統 wiki, agent 又可能無法在正確時機取得適用規則。因此重點是同一份治理知識具有雙重介面。

## Human interface 與 agent interface

影片以 Qodo 畫面示範雙重介面 ([10:34](https://www.youtube.com/watch?v=s-aixZYJG4c&t=634s))。

對人類 reviewer, 系統顯示:

- 本次 review 使用哪些規則。
- 哪些規則被違反。
- 每個判斷連回哪一份標準。
- 人類可以檢查、接受或反駁什麼。

對後續 agent, 系統則提供機器可操作的 review 結果, 例如已發現的問題與另一個修正 PR, 讓 agent 能採用修正而不必從頭重新分析。

這個設計的信任來源不是「AI 說它是對的」, 而是人類與 agent 都能追查所用規則、判斷與修正。

## 以人類 comment 減少作為 readiness signal

講者提出一個漸進判斷方式: 當 context 與 review 自動化運作後, 觀察人類在 PR 中還需要補充多少 comment ([12:19](https://www.youtube.com/watch?v=s-aixZYJG4c&t=739s))。

如果經過大量 PR, 人類幾乎不再指出新問題, 才表示某類變更可能接近自動化門檻。這比直接宣布「從明天開始不再人工 review」更能累積證據。

不過 comment 數量不能單獨代表品質。人類可能因警報疲勞而停止評論, 或 reviewer 根本沒有充分檢查。較完整的 readiness 指標還應包括 escaped defects、revert、incident、false positive、false negative 與規則覆蓋率。

## 真正的 context 是 software graph

規則與 skills 只是較簡單的 context。更深層的工程知識存在於整個軟體系統的關係中 ([13:11](https://www.youtube.com/watch?v=s-aixZYJG4c&t=791s)):

- Repository 與 service 的依賴。
- Microservice 之間的 contract。
- Node 與 edge 對應的設計理由。
- 歷史 P0 incident 與 root cause。
- 修復事故時的工程討論。
- 多個同時進行 PR 之間的衝突。

若只審查單一 PR diff, 可能看不見 service A 的 contract 變更會破壞 service B, 也看不見兩個各自正確的 PR 合併後會產生衝突。

因此影片預測, code governance 將從 review 單一 PR, 轉為 review 整個 software graph。

## 自動 approve 與 block 應逐步加入

當 system architecture、contracts、歷史事故與規則都能被定位到 software graph 的節點和邊, 團隊才開始具備自動核准或阻擋的基礎 ([14:05](https://www.youtube.com/watch?v=s-aixZYJG4c&t=845s))。

推薦的漸進流程是:

1. Shadow mode: AI 產生 review, 不影響合併。
2. Advisory mode: 顯示問題與規則, 由人類決定。
3. Narrow blocking: 只阻擋少數高信心、高風險規則。
4. Narrow auto-approval: 只自動核准低風險且已有充分歷史證據的變更。
5. 持續擴張: 依 false positive、escaped defect 與事故結果調整規則。

影片明確表示這不會立即完成。Approval 與 blocking rule 必須逐步累積。

## 從靜態規則到即時自我學習 context

影片提出的 context engine 不只是 instruction file 集合, 還要持續從實際結果學習:

- 哪些 review comment 被接受或拒絕。
- 哪些規則常被觸發, 是否真正有用。
- 哪些 production incident 揭露了新限制。
- 哪些服務契約或架構關係改變。
- 哪些 PR 同時碰觸相同元件並可能衝突。

每條規則需要 analytics, 例如使用次數、命中率、接受率與版本。否則 knowledge base 會逐漸堆積過時或互相衝突的規則。

## 「Artificial wisdom」是願景, 不是既成能力

講者將模型掌握組織經驗與 judgement 的方向稱為從 artificial intelligence 走向 artificial wisdom。其意義是把工程師知道「什麼好、什麼壞」的經驗轉成 agent 可用的判斷依據。

這是一個產品願景。經驗被寫入 context 並不保證它正確, 模型也不會因此自動取得人類同等 judgement。仍需要擁有者、證據、版本、衝突處理與淘汰機制。

## Qodo 產品主張與演講觀點的區分

演講者是 Qodo 的 CEO 與共同創辦人, 影片多次以 Qodo 作為解法示範。以下內容應視為公司願景或產品主張:

- Qodo 能整理規則並提供人類與 agent 介面。
- Qodo 能建立跨 repository 的軟體關係圖。
- 公司希望協助團隊將 code review 自動化。
- 2027 年達到零 outage、零正式環境 critical/high bug 的願景。

影片沒有提供足以獨立驗證這些產品效果的實驗設計、資料集或比較結果。特別是「零事故、零重大缺陷」屬於願景目標, 不應理解為可保證的工程結果。

## 可落地的治理資料模型

以下是依影片內容整理的編輯性實作框架:

| 資產 | 至少應記錄 |
| --- | --- |
| Rule | 說明、適用範圍、擁有者、來源、版本、嚴重度 |
| Contract | 生產者、消費者、schema、相容性政策、測試 |
| Incident lesson | 根因、受影響元件、預防規則、證據連結 |
| Review decision | 建議、接受或拒絕、理由、最終結果 |
| Graph edge | 元件關係、契約、歷史變更與相關事故 |
| Automation policy | Advisory、block 或 approve, 信心門檻與回退方式 |

這些欄位不是影片逐字提出的 schema, 而是將其治理觀點轉成較容易實作的結構。

## 實務檢查表

- Code review 的品質驗證與知識傳承是否都有替代流程?
- Coding agent 與 reviewing agent 是否使用同一版本的必要規則?
- 每項 AI 建議能否追查使用過的 context 與規則?
- 架構、service contract 與歷史事故是否可被 review 系統取得?
- 是否評估跨 repository 與多個並行 PR 的影響?
- 規則是否有擁有者、版本、適用範圍與淘汰程序?
- 是否量測 false positive、false negative 與 escaped defect?
- 自動 blocking 是否先限定於高信心、高影響問題?
- 自動 approval 是否有低風險範圍、回退與人工抽查?
- 人類能否理解、覆寫並修正 agent 的判斷?

## 時效性與限制

影片發布於 2026 年 8 月, 反映當時 AI coding 與 code review 工具的狀態。提到的 instruction file、agent、MCP、模型 benchmark 與 Qodo 功能可能持續更新。

本筆記以英文原始自動字幕與 YouTube 章節整理。部分產品名稱與口語內容可能有字幕辨識誤差。文中的產業趨勢、模型能力與自動化時程多屬講者觀點, 並非所有團隊都適用。

## 結語

如果 AI 產生程式碼的速度已超過人類 review 能力, 單純增加 reviewer 或接受更多未審查變更都不是完整答案。較穩健的方向是先把組織判斷轉成可治理的 context, 建立 software graph 與可追蹤規則, 再用真實 PR 與事故資料逐步證明哪些 review 可以安全自動化。
