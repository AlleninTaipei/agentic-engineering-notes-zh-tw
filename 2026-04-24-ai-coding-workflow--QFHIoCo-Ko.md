# AI Coding 完整工作流: 從需求對齊到代理實作與 QA

> 來源: [Full Walkthrough: Workflow for AI Coding — Matt Pocock](https://www.youtube.com/watch?v=-QFHIoCo-Ko)  
> 頻道: AI Engineer  
> 發布日期: 2026-04-24  
> 影片長度: 1 小時 36 分 30 秒  
> Video ID: `-QFHIoCo-Ko`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 核心摘要

Matt Pocock 的核心論點是: AI 帶來新的開發能力, 但沒有淘汰軟體工程基本功. 小任務、明確介面、良好測試、快速回饋、需求對齊與人工審查, 對 AI 代理同樣重要.

他示範的完整流程是:

1. 研究現有程式碼並製作原型.
2. 透過密集問答消除需求中的模糊與隱性假設.
3. 將共識整理成描述目的地的 PRD.
4. 把 PRD 拆成有依賴關係的小型 issues.
5. 讓 AI 代理在短而乾淨的 context 中逐項實作.
6. 由人進行 QA 與 code review, 再把問題送回 issue backlog.
7. 完成後部署、監控, 並維持適合 AI 工作的程式架構.

這套方法不是「規格直接編譯成程式碼」. 人仍需掌握系統形狀、模組邊界、驗收標準與產品品味.

## 1. AI 工程的兩個限制

[00:00:14](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=14s)

### Smart zone 與 dumb zone

講者把 LLM 的 context 分成兩個區域:

- `smart zone`: 對話剛開始、上下文乾淨時, 模型較能正確探索、推理與實作.
- `dumb zone`: token 持續累積後, 注意力關係變得更吃力, 判斷品質開始下降.

他的實務估計是, 即使模型宣稱有很大的 context window, 適合高品質 coding 的區域仍可能只有前約 100K tokens. 大 context 對資料檢索有幫助, 不代表整段都同樣適合複雜程式設計.

因此, 大型任務不應長期塞在同一段對話中. 應把工作拆成小單位, 讓每一次代理執行都盡量發生在 smart zone.

### 每次 session 都像重新失憶

新的代理 session 通常會經歷:

1. 載入 system prompt 與固定指令.
2. 探索 codebase.
3. 實作.
4. 執行測試與其他 feedback loops.

清除 context 後, 代理會回到固定的初始狀態. 講者偏好主動清除並重建必要背景, 而不是一再 compact 舊對話. 原因是 compact 會留下經過摘要的歷史沉積, 初始狀態則較穩定、可預測.

固定載入的指令也應保持精簡. 若 system prompt 一開始就佔用大量 token, 代理會更快進入品質較差的區域.

## 2. 研究與原型

[00:04:20](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=260s)

工作坊以課程管理平台的遊戲化功能為例. 客戶 brief 只說留存率不好, 希望加入 gamification. 這類需求無法直接安全地轉成程式碼, 因為獎勵條件、歷史資料、UI 位置與濫用風險都尚未決定.

AI 的第一項工作是探索 repository, 理解現有模組、資料結構和可行的修改位置. 探索可交給 isolated sub-agent, 再把精簡結果送回主要代理. 這樣能利用另一個 context 做大量閱讀, 又不讓主 session 被探索細節塞滿.

研究或原型的目的不是立即產出最終實作, 而是降低未知數, 找出接下來必須由人決定的問題.

## 3. Grill session: 先取得共同理解

[00:12:45](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=765s)

講者幾乎會用 `grill me` skill 開始每一項重要工作. 其做法是要求代理逐一訪談使用者, 沿著決策樹處理依賴問題, 並為每個問題提供建議答案.

示範中的問題包括:

- 哪些行為可以獲得點數?
- 是否應替既有完成紀錄補發點數?
- 等級曲線如何設計?
- streak 是否另外計分?
- gamification UI 應出現在哪裡?

這一步要建立的是 Frederick P. Brooks 所說的 `design concept`, 也就是參與者共享的設計概念. 目標不是急著拿到一份 plan, 而是讓人與代理對問題、限制和選擇處於同一條理解線上.

Grill session 是 human-in-the-loop 工作, 不適合完全自動化. 若問題涉及產品或領域知識, 可以讓產品負責人、domain expert、開發者與 AI 一起回答. 會議轉錄也可以成為輸入, 再由代理追問尚未說清楚的假設.

## 4. PRD: 描述目的地

[00:22:10](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=1330s)

完成對齊後, 對話中已累積許多有價值的決策. PRD 的作用是把這些共識壓縮成可供後續使用的「目的地文件」.

示範的 PRD 包含:

- problem statement.
- proposed solution.
- user stories 與 definition of done.
- implementation decisions.
- testing decisions.
- 預計新增或修改的模組.

講者特別要求代理提出 module map, 因為這不是忽略程式碼的 specs-to-code 流程. 從規劃階段開始, 人就必須考慮系統會被切成哪些模組、介面放在哪裡, 以及測試邊界如何形成.

他的個人做法是不花大量時間逐字優化 PRD. 前面的 grill session 已完成核心對齊, 而 LLM 通常擅長摘要. 他認為工程時間應更多投入真正能觀察成品品質的 QA. 這是講者的工作偏好, 並非所有團隊都必須省略 PRD review.

## 5. Issues: 描述旅程

[00:35:50](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=2150s)

PRD 描述最終狀態, issues 則描述抵達目的地的旅程. 講者把功能拆成 Kanban board 上的小型 tickets, 並標示彼此的 blocking relationships.

理想的 issue 應具備:

- 足夠小, 能在一次乾淨的 context 中完成.
- 有明確驗收條件與 feedback loops.
- 完成後可以獨立檢查.
- 依賴關係清楚, 可判斷哪些工作能平行執行.

這比單一線性的 multi-phase plan 更有彈性. 當 QA 發現新問題時, 可以新增 issue 並建立阻擋關係, 不必把所有工作硬塞進一場持續膨脹的代理對話.

## 6. 使用 AI 代理實作

[00:48:15](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=2895s)

每個可執行 issue 都交給新的代理 session. 代理先探索完成該 issue 所需的部分, 做出一個小變更, 執行驗證, 然後提交結果. 整體形式類似結構化的 Ralph loop: 重複選取下一個小步驟, 直到抵達 PRD 所描述的終點.

講者區分兩類工作:

- Human-in-the-loop: 需求對齊、重要設計決策、QA 與品味判斷.
- AFK: 邊界清楚、可透過測試驗證的實作工作.

當 issue 之間沒有依賴時, 可以讓多個代理在隔離的 worktree 或 sandbox 中平行處理. 講者展示的 Sandcastle 流程包含 planner、implementer、reviewer 與 merger:

1. Planner 從 backlog 選出可平行的 issues.
2. 每個 implementer 在獨立環境中工作.
3. Reviewer 檢查產生的 commits.
4. Merger 合併 branches 並處理型別、測試或整合問題.

在 coding standards 的提供方式上, 講者建議 implementer 可按需 `pull` skills 或規範, reviewer 則應被明確 `push` 完整標準, 讓審查時有清楚的比較基準.

## 7. 人工 QA 與回饋迴圈

[01:05:30](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=3930s)

AI 完成 issue 後, 先執行自動 feedback loops, 例如單元測試與 type checking. 講者把回饋品質視為 AI 實作品質的上限: 若 codebase 沒有可靠測試或可快速執行的驗證, 代理等於閉著眼睛寫程式.

人工 review 建議先看測試是否驗證了正確行為, 再看實作是否合理. 接著實際操作產品進行 QA. 示範中, 自動測試通過後, 手動操作仍發現缺少資料表造成的 SQLite 錯誤, 說明測試通過不等於功能真的可用.

QA 的作用不只是找 bug, 也是人把產品判斷、偏好與品味重新施加到系統上的位置. 發現的問題應回到 Kanban board 成為新 issue, 然後再進入實作與驗證循環.

## 8. 部署、監控與完成條件

[01:18:45](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=4725s)

當程式碼與產品行為都通過審查後, 才進入團隊 review、合併、部署與監控. 這套流程的完成條件不是「代理停止輸出」, 而是:

- PRD 的使用者結果已達成.
- 自動 feedback loops 通過.
- 人工 QA 已驗證實際行為.
- review 中發現的 issues 已處理.
- 變更能安全整合並在部署後觀察.

## 9. 為 AI 設計容易理解的 codebase

[01:28:10](https://www.youtube.com/watch?v=-QFHIoCo-Ko&t=5290s)

講者引用 John Ousterhout 的 deep modules 概念, 對比兩種架構:

| 架構 | 特徵 | 對 AI 的影響 |
| --- | --- | --- |
| Shallow modules | 許多小檔案、小函式與交錯依賴 | 難以追蹤完整流程, 測試邊界模糊 |
| Deep modules | 小而穩定的公開介面, 內部封裝較多功能 | 容易形成明確測試邊界, 代理較能理解與修改 |

設計 deep module 時, 人應掌握模組的目的、介面、依賴與外部行為, 內部實作則可較放心地委派給 AI. 講者把這種模組視為 `gray box`: 不必記住每行內部程式碼, 但仍理解整體結構與契約.

這也有助於解決 AI 開發中的認知流失. 開發速度提高後, 人容易逐漸不熟悉自己的 codebase. 掌握少數重要模組及其介面, 能在委派實作的同時保留系統層級理解.

## 10. 文件腐化與暫時性規劃資料

講者通常不會把已完成的 PRD 永久留在主要 codebase 中. 原因是需求、檔名、架構與實作日後可能改變, 舊文件卻會被代理當成最新事實, 形成 `doc rot`.

他的替代做法是將 PRD 與 implementation issues 保存在 GitHub, 完成後關閉. 這樣仍保有歷史紀錄, 同時有明確的「已結束」狀態. 他也說明資料庫 migrations 不完全等同於一般規劃文件, 因為 migration 是較具決定性、可執行的變更紀錄.

## 工作流速查

```text
模糊需求
  -> 探索 codebase / 原型
  -> Grill session 建立共同理解
  -> PRD 描述目的地
  -> Issues + dependencies 描述旅程
  -> 小型代理 session 實作
  -> Tests / types / lint 等自動回饋
  -> 人工 QA 與 code review
  -> 新問題回到 backlog
  -> 團隊 review / 合併 / 部署 / 監控
```

## 實務重點

- 把 context 視為有限的推理資源, 不要只看模型宣稱的最大長度.
- 將固定指令保持精簡, 其他知識透過 skills 按需載入.
- 先消除模糊需求, 再產出 PRD.
- PRD 是目的地, issues 是可執行旅程.
- 讓每個 issue 足夠小、可測試、可獨立 review.
- 平行化只適合依賴關係允許的工作.
- 自動測試是必要 feedback loop, 但不能取代人工 QA.
- 人負責系統形狀、介面、產品品味與最後判斷.
- Deep modules 與清楚的測試邊界能同時幫助人與 AI.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 自動字幕包含少量專有名詞、人名與工具名稱的辨識不確定性. 本文只保留能由上下文可靠確認的名稱與概念.
- 影片是現場工作坊, 包含觀眾問答、休息與即時操作. 本文刪除與教學無關的寒暄、設備狀況及重複片段.
- 章節時間取自影片公開 metadata. 中文標題、表格、流程圖式文字與「實務重點」是編輯整理, 不是講者投影片原文.
- 本文為學習摘要, 不取代影片中的完整示範、畫面操作與問答脈絡.
