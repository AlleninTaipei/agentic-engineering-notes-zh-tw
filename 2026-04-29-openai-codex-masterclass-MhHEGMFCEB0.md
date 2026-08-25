# OpenAI Codex Masterclass: 從程式碼代理到軟體工程系統

> 影片: [OpenAI Codex Masterclass — Vaibhav Srivastav & Katia Gil Guzman](https://www.youtube.com/watch?v=MhHEGMFCEB0), AI Engineer<br>
> 頻道: AI Engineer<br>
> 上傳日期: 2026-04-29<br>
> 片長: 01:01:58<br>
> Canonical URL: https://www.youtube.com/watch?v=MhHEGMFCEB0<br>
> Video ID: `MhHEGMFCEB0`<br>
> 內容依據: 英文原始自動字幕

## 摘要

這場工作坊把 Codex 定位為軟體工程代理, 而不只是產生程式碼的聊天工具. 它能探索程式碼庫, 執行指令與測試, 進行程式碼審查, 使用外部服務, 並透過多個介面承接完整工程工作.

講者展示三種擴充工作方式. 第一種是外掛, 將技能, 應用程式連線與 MCP 伺服器包成可重用流程. 第二種是自動化, 讓代理按排程在背景執行工作. 第三種是子代理, 將複雜任務拆成可平行處理的獨立工作, 再由主代理彙整結果.

影片後段聚焦控制與安全. 包括依工作性質設定子代理的模型, 推理強度, 沙箱權限與工具, 使用 Guardian approvals 判斷高權限操作, 以 hooks 在工作階段事件發生時執行固定程序, 以及使用 Codex Security 掃描並修補漏洞.

## 1. Codex 是軟體工程代理

[00:01:48](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=108s)

講者強調, Codex 不只負責寫程式. 它的工作範圍包括:

- 執行命令與測試.
- 探索及理解既有程式碼庫.
- 修改程式並驗證結果.
- 透過 IDE, CLI, Codex app, Slack 或 GitHub 接受任務.
- 連接 Figma, Linear, Notion 等既有工具.

其整體能力可理解為三層:

1. 基礎模型負責推理與生成.
2. 統一的 agent harness 負責工具執行, 環境設定, 行為管理與安全控制.
3. App, IDE extension, CLI 與其他整合介面提供不同工作入口.

這個觀點很重要. 模型品質只是代理表現的一部分, 工具環境, 權限控制與工作介面同樣會影響最終結果.

## 2. 模型與執行速度

[00:02:18](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=138s)

影片依當時產品狀態提到 GPT-5.2 Codex, GPT-5.3 Codex, GPT-5.3 Codex Spark, GPT-5.4, 以及較小型的 mini 與 nano 模型. 講者將大型模型定位於複雜或長時間任務, 小型模型則適合短任務與子代理.

[00:04:11](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=251s)

除了提高模型能力, 團隊也改善推理結果傳送速度. 影片提到 WebSocket 連線與 Fast mode, 用意是降低等待時間, 讓互動式代理更適合日常工程流程.

注意: 型號, 速度倍率與供應狀態屬於影片發布時的產品資訊, 可能隨後續版本改變.

## 3. Codex app, 專案與 worktree

[00:07:04](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=424s)

Codex app 將多個專案與多條工作執行緒集中在同一介面. 每個專案內可利用 Git worktree 隔離功能開發, 錯誤修正或問答任務, 讓多項工作同時進行而不互相覆蓋.

這種安排降低了頻繁切換上下文的成本. 它也讓代理產生的變更留在獨立工作樹, 便於比較, 審查與選擇性整合.

影片也提到原生 Windows 支援與 Windows 沙箱. 這同樣是發布當時的產品狀態, 應以實際安裝版本為準.

## 4. 外掛的組成與用途

[00:12:28](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=748s)

外掛將多種能力包成一個可安裝單位:

| 元件 | 角色 | 適合用途 |
| --- | --- | --- |
| Skill | 特定流程的可重用指示, 可附腳本與資源 | 將反覆執行的方法標準化 |
| App | 對其他服務的連線 | 存取 Google Drive, Slack, Gmail 等服務 |
| MCP server | 對外部系統暴露工具與資料 | 擴充代理可呼叫的專業能力 |
| Plugin | 將上述元件組合成完整工作流程 | 一次安裝並降低手動設定成本 |

如果某個流程反覆出現, 可以建立 skill, 也可以請 Codex 協助產生初始版本. 當流程同時需要指示, 外部資料和工具時, 再將它們包成 plugin.

## 5. 視覺開發示範

[00:15:13](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=913s)

講者介紹兩項適合 Web app 與遊戲開發的能力:

- Playwright Interactive: 讓 Codex 開啟應用程式, 操作頁面, 擷取畫面並檢查實際結果.
- Image generation: 為應用程式或遊戲產生 sprite 與其他視覺素材.

[00:16:26](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=986s)

遊戲示範只給出高階需求, 要求建立由磚塊平台構成的平台遊戲. Codex 使用 Game Studio plugin, 產生角色與遊戲素材, 再以 Playwright 檢查成品. 完整執行約需一小時, 因此現場同時啟動新任務並展示前一天完成的版本.

這個案例顯示, 視覺任務不能只檢查程式是否成功執行. 代理還需要看見頁面, 操作互動元件並反覆比較畫面.

## 6. 使用外部資料更新程式庫內容

[00:17:23](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=1043s)

另一個示範將 Codex 連接 Google Drive. Codex 先在開發者網站的程式碼庫中找出活動資料來源, 讀取 YAML 檔, 再把 57 筆活動寫入試算表.

這類工作不必侷限於寫程式. 只要代理能讀取程式庫和適當的外部資料, 就能執行資料盤點, 比對與更新等耗時工作.

## 7. 自動化與背景工作

[00:08:37](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=517s)

Automation 是按排程在背景執行的代理工作, 類似 cron job. 使用者指定任務指示, 執行頻率, 所屬專案和所需連線, 之後由 Codex 定期執行.

[00:18:52](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=1132s)

影片展示兩個日常案例:

- 每天上午檢查 Slack, 找出待回覆或時效敏感的訊息, 並依主題摘要前一天的重要內容.
- 檢查 Gmail, 辨識需要回覆, 具時效性或可能不可信的郵件.

現場以自然語言要求建立一個尋找 Slack 中 Codex 使用案例的 automation. 自動建立介面一度沒有依預期運作, 講者因此示範手動設定. 這段失敗值得保留, 因為代理介面仍需準備可控的替代流程.

## 8. 程式碼審查是代理擴張後的品質關卡

[00:27:14](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=1634s)

當代理同時處理更多專案與功能時, 人類很難逐行審查所有輸出. 講者主張至少先讓另一個獨立代理進行第一輪審查.

Codex code review 可在不同介面啟動:

- GitHub pull request 可設定自動審查.
- Codex CLI 或 app 可使用 `/review`.
- 審查目標可以是分支, 特定基準分支或尚未提交的變更.

審查程序會啟動新的 Codex 執行緒, 使用專門的 review system prompt. 它不只查看 diff, 還會讀取程式庫脈絡, 因此可能找出變更對其他模組造成的二階影響. 結果以 P0, P1, P2 等優先級呈現, 之後可交由另一輪工作修正.

影片聲稱 OpenAI 內部所有 pull request 都預設經過 Codex code review. 這是講者於影片中的陳述, 本筆記未另行查證.

## 9. 子代理: 拆解, 平行與彙整

[00:32:39](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=1959s)

子代理把一個主要任務分解為獨立且可平行執行的小任務. 每個代理在自己的工作範圍內探索或執行, 完成後把結果交回主代理彙整.

適合平行化的工作通常具備以下特徵:

- 子任務之間依賴很少.
- 每個子任務有清楚的輸入與完成條件.
- 可由主代理比較, 去重並整合各項結果.
- 平行處理能實際縮短總等待時間.

[00:36:18](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=2178s)

示範要求 Codex 以 20 個子代理審查一個包含 45 份 persona 設定檔的程式庫. Codex 判斷任務複雜, 自動進入 plan mode, 建立工作清單並分割審查範圍. 現場環境的並行上限是 6 條代理執行緒, 因此代理分批執行.

主代理會為各子代理指定:

- 要審查的確切檔案.
- 專用角色與任務說明.
- `README`, `CONTRIBUTING` 或 skill 等程式庫規範.
- 最後需要回報的發現.

完成後, 主代理收集所有結果, 找出例如權限過大或沙箱設定不符等問題. 同樣的方法可用於多角度安全分析, 架構方案比較或大型程式庫探索.

## 10. 自訂子代理的人格, 權限與工具

[00:44:52](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=2692s)

影片介紹一般用途, worker 與 explorer 等內建角色, 也鼓勵依團隊需求建立自訂 persona. 每個子代理可分別指定:

- 使用的模型.
- 推理強度.
- 沙箱模式.
- 專用指示.
- MCP server 與 skill.

權限應按照任務最小化. 例如 reviewer, explorer 或弱點分析代理通常只需讀取權限. 需要實際修改程式或撰寫文件的 worker 才授予寫入權限. 這可降低審查者意外改變受審內容的風險.

示範中的 PR explorer 使用快速模型與唯讀沙箱, 指示它追蹤執行路徑, 保持探索模式, 不提出修正. 另一個 docs researcher 則連接文件 MCP server, 讓它根據官方 API reference 與指南回答問題.

講者也建議讓 Codex 分析過往 session, 從重複工作中推薦可建立的自動化或子代理. 影片 Q&A 補充, 本機 session 儲存在 `.codex` 目錄下的 session 資料中.

## 11. Guardian approvals

[00:49:29](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=2969s)

Guardian approvals 是影片當時的實驗功能, 目的在減少兩種極端狀態:

- 完全放開權限的 YOLO mode, 風險過高.
- 每次高權限操作都要求人類確認, 容易產生 approval fatigue.

當 Codex 要刪除目錄, 啟動伺服器或把檔案暴露到網路等高權限工作時, Guardian 會啟動另一個子代理評估請求. 若依規則判斷可安全執行, 它可以代為核准. 需要人類判斷時才中斷工作並提出確認.

這不等於移除安全邊界. 核心做法是用受限制的判斷層過濾低風險請求, 同時保留重大操作的人類決策權.

## 12. Hooks 與長時間任務

影片接著介紹 hooks, 這也是當時的實驗功能. Hooks 讓使用者在特定生命週期事件上執行固定程式. 講者提到的事件包括:

- Session start: 工作階段啟動或恢復時執行, 例如拉取最新 GitHub 內容.
- Tool use: 每次工具使用前後執行, 例如記錄研究代理做過的操作.
- Stop: 一輪工作結束時執行.

講者的長時間任務範例在 stop hook 呼叫 Python 腳本, 要求 Codex 再進行一輪檢查, 執行一個有效的驗證命令, 然後回報結果. 這能延長自主工作時間, 但也必須設置明確終止條件, 避免無限循環或無效消耗.

## 13. 個人化, 安全與其他代理整合

Codex 可設定較友善或務實等 personality, 也能加入自訂指示, 例如要求回答時附上來源. Personality 影響溝通風格, 專案規範與安全政策仍應由明確指示和權限配置控制.

[00:56:11](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=3371s)

影片將 Codex Security 描述為能逐一檢查 GitHub commit, 建立弱點報告並產生修補變更的能力. 另外展示的 Claude Code plugin 則允許使用者在其他代理工作流程內呼叫 Codex 進行程式碼審查, 對抗式檢查或救援既有變更.

## 14. Q&A: 雲端任務與本機技能

[00:57:10](https://www.youtube.com/watch?v=MhHEGMFCEB0&t=3430s)

Q&A 提到, Codex app 可把任務交給 cloud agent, 也能使用 best-of-N 方式在雲端平行執行多次, 再選擇最佳結果.

對於雲端工作是否能使用本機 skill, 講者解釋了信任邊界. 程式庫內已提交的指示或資源仍可能被讀取, 但本機 skill 可以攜帶 Python 腳本或可執行內容, 雲端沙箱無法直接判定它是否可信, 因此不能視為與本機環境完全等價. 影片也透露團隊正考慮可信 MCP server, CLI 與 SSH agent 等後續能力, 但沒有承諾確切時程.

## 實務工作流程

綜合影片示範, 可以把 Codex 導入流程整理為以下步驟:

1. 先讓主代理讀取程式庫規範, 建立明確計畫與完成條件.
2. 把互相獨立的探索, 測試或審查工作交給子代理.
3. 為每個角色選擇合適的模型與最低必要權限.
4. 需要外部資料時使用 app 或 MCP, 重複流程則封裝為 skill 或 plugin.
5. 以 Playwright, 測試指令或其他可觀察證據驗證輸出.
6. 由獨立 review agent 進行第一輪審查, 再由人類處理高風險判斷.
7. 只有穩定且可驗證的週期性工作才改成 automation.
8. 為 hooks 與長時間任務設定停止條件, 成本上限和錯誤處理.

## 核心觀念

這場工作坊的主線不是讓單一模型一次完成所有工作, 而是建立一個可管理的工程系統. 模型負責推理, harness 管理工具與環境, 外掛連接工作所需的能力, 子代理提供平行化, 審查代理與權限機制則形成品質及安全邊界.

最有效的代理化流程通常同時具備三項特徵: 任務可以拆解, 結果可以驗證, 權限可以限制. 缺少任何一項, 提高自主程度都可能只是更快地產生難以審查的結果.

## 來源與限制

- 本筆記依英文原始自動字幕整理, 不是逐字稿. 已移除口語贅詞, 重複片段與現場操作等待時間.
- 自動字幕可能誤辨講者姓名, 產品名稱與技術詞彙. 標題中的講者姓名及章節時間以影片公開中繼資料為準.
- 影片沒有提供可由工具辨識的正式 chapter 欄位. 時間段主要依影片說明中的 timestamps 與字幕內容整理.
- 產品型號, 功能狀態, 效能倍率, 使用者數和內部採用情況屬於影片發布時的講者陳述. 本筆記未以外部資料逐項驗證, 不應視為目前仍有效的產品規格或承諾.
