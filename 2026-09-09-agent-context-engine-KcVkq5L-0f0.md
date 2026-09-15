# Agent 缺少組織脈絡: Context Engine 如何補上搜尋與理解之間的落差

> - 影片: [Your agents lack context: Here's how to fix "You're absolutely right!" - Brandon Waselnuk, Unblocked](https://www.youtube.com/watch?v=KcVkq5L-0f0)
> - 頻道: [AI Engineer](https://www.youtube.com/channel/UCLKPca3kwwd-B59HNr-_lvA).
> - 講者: Brandon Waselnuk, Unblocked. 公開說明標示其工作領域為 developer relations.
> - 發布日期: 2026-09-09. 此為 YouTube 上傳日期, 不代表演講日期.
> - 片長: 14:09.
> - Video ID: `KcVkq5L-0f0`.
> - 內容依據: 完整英文自動字幕 `en-orig`, 搭配影片公開 metadata 與章節. 無創作者字幕.
> - 來源性質: Context engine 供應商的架構與工具分享. 客戶成效與測試數字為講者自述, 未另行驗證.

## 核心觀點

Brandon 提出的目標是讓 AI 生成的程式碼, 像是在團隊待了多年的同事所寫. 資深同事知道的不只有程式碼, 還包括 PR 被退回的原因, 歷史事故, 部署慣例與近期決策. 每次新開的 agent session 都需要取得這些背景.

影片主張, 只提供文件或搜尋入口仍有缺口. Agent 可能找到過期答案, 忽略新決策, 或取得與使用者工作無關的資料. Context engine 的責任是跨來源整理脈絡, 處理衝突與權限, 再將任務所需資訊送進工作流程.

## 1. 自主程度越高, 錯誤 context 的成本越大

[00:00](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=0s) / [01:50](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=110s)

在 tab completion 階段, 工程師能立即用自己的背景知識判斷是否接受建議. 當工作逐漸交給平行或背景 agents, 人不再逐步補足脈絡, 最初的誤解便可能持續擴散.

| 工作方式 | 缺少脈絡造成的成本 |
| --- | --- |
| 即時補全 | 人辨識錯誤並拒絕建議 |
| Agent 執行任務 | 反覆糾正與重做, 浪費搜尋 tokens |
| 平行 agents | 產出增加, 人承擔更多審查負擔, 即 review tax |
| 背景 agents | 遇到問題時若無法取得可靠答案, 就難以持續完成工作 |

講者尤其關注既有 brownfield codebase. 程式碼通過編譯, 不代表符合公司的上線方式. 他舉例, 忽略 rollout 程序或 feature flag 操作, 仍可能造成 production 事故. Code review agent 也需要業務邏輯與操作脈絡, 才能辨識這類問題.

## 2. 兩種有幫助, 但容易遇到瓶頸的做法

[04:20](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=260s)

### 人工整理 context 的維護問題

將專案知識寫成 Markdown, 讓 agent 用 grep 查詢, 確實能改善表現. 講者稱之為 curated context trap, 因為團隊接著必須處理:

- 分發: 如何讓其他成員取得這些檔案.
- 過期: 文件如何跟上系統與決策變化.
- 維護責任: 誰有足夠知識與判斷力, 為整個組織持續整理內容.

這段批評針對規模化後的維護瓶頸. 字幕並未否認 Markdown context 的初期效益.

### MCP 提供存取, 仍需要檢索判斷

MCP 讓 agent 能從其他系統取資料, 但 server 與 tool description 會影響它是否呼叫工具. 即使有呼叫, agent 也可能找到第一個看似合理的答案後便停止搜尋. 講者稱這種情況為 satisfaction of search bias.

影片例子是, Agent 讀到舊架構紀錄便開始實作, 卻漏掉昨晚 Slack 上要求改採另一種方案的討論. 問題不只是能否讀取資料, 還包括是否找齊相關資訊與處理矛盾. 這是講者用來說明失敗模式的例子, 並非附有 trace 的事故紀錄.

## 3. Context engine 的六項能力

[05:52](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=352s) / [07:26](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=446s)

講者描述的流程是, 從工程團隊使用的多個來源匯入資料, 經過 context engine 處理, 再依接收者與工作流程輸出資訊. 人類可能透過 Slack 提問, Agent 則需要節省 tokens 的機器用回覆.

| 能力 | 影片提出的要求 |
| --- | --- |
| Unified system context | 跨來源掌握系統脈絡, 讓 agent 有機會發現使用者原本不知道的相關資訊 |
| Targeted retrieval | 若已提供文件連結, 能快速取回內容; 需要深入探索時才進行較長的研究 |
| Conflict resolution | 遇到舊架構文件與新討論互相矛盾時, 判斷應採用哪項資訊 |
| Personalized relevance | 理解提問者是誰, 所屬團隊與目前工作, 以縮小檢索範圍 |
| Token optimization | 回覆足以支援任務, 同時避免不必要的格式與內容占滿 context window |
| Permission enforcement | 尊重來源權限, 避免在答案中洩漏提問者無權得知的專案資訊 |

個人化的線索包括 Git commits, 工作涉及的 repository, 以及誰審查這些變更. 這些關係能作為尋找其他相關資料的起點.

衝突處理則沒有在演講中展開成可實作的演算法. 講者指出需要辨識舊文件與近期決策的差異, 但沒有交代權威性排序或無法判定時的處理方式. 權限部分也主要停留在 OAuth, scopes 與不洩漏資訊的要求.

## 4. 同一 prompt 的測試, 能支持什麼結論

[09:16](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=556s)

講者表示, 團隊使用相同 prompt 與相同模型, 比較有無 context engine 的執行結果:

| 指標 | 未使用 context engine | 使用 context engine |
| --- | ---: | ---: |
| Token 用量 | 約 2,100 萬 | 約 1,080 萬 |
| 完成時間 | 作為比較基準 | 約縮短 2 小時 |

字幕中講者先說出另一個數字, 隨即更正為 10.8 million. 此處採用更正後的數值, 也與影片公開說明一致.

講者的解釋是, 提前提供脈絡能減少每次 session 重新探索的搜尋成本, 也減少重做. 他另提到更快的 triage 與更好的答案品質.

影片未交代完整任務, 模型版本, 重複次數, token 計算範圍或品質評分方法. 因此這是供應商展示的個別比較, 不能推定任何任務都會節省約一半 tokens. 是否包含 context engine 自身的處理成本, 字幕也未說明.

## 5. 三個可進一步探索的開源工具

[10:11](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=611s)

以下連結來自影片公開說明. 功能依講者介紹整理, 本次未下載或測試 repositories.

### Engineering Social Graph

[unblocked/engineering-social-graph](https://github.com/unblocked/engineering-social-graph)

透過確定性程式處理 GitHub 資料, 分析成員在哪些地方提交程式碼, 誰審查變更, 並整理專家關係圖. 講者表示, 可選擇加入模型 API key, 協助標記團隊分組.

它提供的線索是人與程式碼的關係, 可用來聚焦 context engine 的檢索範圍.

### Repo Rules Agent

[unblocked/repo-rules-agent](https://github.com/unblocked/repo-rules-agent)

尋找 repository 內不同位置的 rules files, 彙整規則與嚴重程度資訊, 並指出重複或其他問題. 整理後的 index 可供後續檢索與去重使用.

這個工具回應前述人工整理 context 的維護問題. 影片沒有證明它能自動解決所有規則衝突.

### Document Query Engine

[unblocked/document-query-engine](https://github.com/unblocked/document-query-engine)

講者介紹一份從頭建立 relational context engine 的 workshop, 以六個堆疊 PR 引導學習. 其重點是讓 agent 探索 schema, 再形成查詢以取得關聯資料.

影片提問例子, 保留英文:

> What are the open PRs that I worked on in the last week with authentication?

中文意譯: 找出我上週參與, 與 authentication 有關且仍開啟的 PR.

這個問題同時包含人物, 時間, PR 狀態與主題條件. 講者認為 RAG 本身不足以回答, 還需要查詢能力. 他肯定 RAG 的用途, 並主張加入能處理關係與條件的工具.

## 6. 應用範圍超過程式碼生成

[12:25](https://www.youtube.com/watch?v=KcVkq5L-0f0&t=745s)

講者提到, 同一套 context engine 也可支援 customer success 處理 tickets, 或讓業務在客戶現場取得內部資訊. 這些是產品使用情境與客戶成效自述, 演講沒有提供可核對的案例資料或量化方法.

結尾將重點放在模型周圍的組織脈絡: 即使模型持續變強, 仍需要知道當前公司如何工作, 才能有效完成任務.

## 編輯整理: 可用來檢查現有系統的問題

以下是依演講整理的檢查問題, 不是影片提供的正式評測標準.

1. Agent 出錯時, 是缺少來源, 未呼叫工具, 還是找到第一個答案就停止?
2. 舊文件與近期決策互相衝突時, 系統如何呈現依據與不確定性?
3. 已知文件連結的查詢, 是否能快速完成, 不必重新搜尋全部資料?
4. 人物與團隊關係能否改善檢索範圍, 同時維持原有權限?
5. 評估 context engine 時, 是否同時量測完整成本, 完成時間與答案品質?

## 來源與限制

- 筆記依完整英文自動字幕轉述, 已移除重複字幕, 寒暄與攤位宣傳. 時間連結採影片官方章節起點.
- 自動字幕對部分人名與開場模型名稱辨識不清. 這些枝節未納入筆記, 也未據此推定產品發布資訊.
- 未逐幀檢視投影片. 工具名稱與 repository 對應以公開說明補足, 不推測畫面中未被口述的功能.
- 效能比較與客戶成果出自產品供應商, 缺少完整實驗條件與獨立驗證. 六項能力是架構要求, 不代表影片已展示所有實作細節.
- 本次以已取得的英文原語字幕整理. 中文字幕請求遇到 HTTP 429, 不影響英文字幕的完整讀取.
