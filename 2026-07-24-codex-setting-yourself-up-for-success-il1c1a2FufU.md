# 用 Codex 建立可長期運作的個人工作系統

來源影片: [Full Workshop: Setting Yourself Up for Success - Jason Liu, OpenAI Codex](https://www.youtube.com/watch?v=il1c1a2FufU), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=il1c1a2FufU
- 上傳日期: 2026-07-24
- 片長: 01:15:02
- Video ID: `il1c1a2FufU`
- 內容依據: 英文自動字幕

## 摘要

Jason Liu 把 Codex 描述成一套知識工作環境, 而不只是寫程式的工具。核心工作流分成三段: 把足夠的脈絡帶入系統, 讓長期 thread、skills、goals 與 automations 持續執行工作, 最後透過文件、plugins、瀏覽器或 computer use 對外採取行動。

這套方法的關鍵不是追求最高 token 用量, 而是建立可累積的個人記憶、可重複使用的程序, 以及具備明確驗證條件的長期任務。當固定 thread 能保存工作脈絡、喚醒自己、建立其他 thread 並互相傳訊時, 使用者的角色會從逐項操作的執行者, 轉向設定方向、補充人際脈絡與審核重要決策的管理者。

## 一套系統的三個階段

影片把日常工作概括成三個動作 ([00:07:37](https://www.youtube.com/watch?v=il1c1a2FufU&t=457s)):

1. Bring context in: 用語音、plugins、appshots、檔案與個人知識庫帶入脈絡。
2. Work on it: 用長期 thread、skills、loops、goals 與 sub-agents 執行工作。
3. Take actions: 產出文件、更新專案, 或透過外部服務與圖形介面完成動作。

這三段互相強化。輸入的脈絡越結構化, agent 越容易正確選擇工具。工作結果若再寫回記憶庫與 skill, 下一次執行就能減少指示。

## 1. 把脈絡帶進系統

### 用口述取代過度整理的提示詞

講者鼓勵用 dictation 輸入較長、甚至帶有岔題的想法。人說話通常比打字快, 而模型能從一段不完整的敘述中搜尋相關郵件、會議或 thread, 再組合出可執行的任務。

重點不是先把提示詞寫得完美, 而是提供足夠的原始意圖與線索。例如, 使用者只記得「上週和某人談過 Agents SDK」, agent 可以搜尋相關紀錄並找出真正的會議。講者也傾向讓模型根據口述內容自行撰寫 goal、automation 或工作提示, 因為模型能把模糊需求轉成較適合執行的規格。

### Appshot 比普通截圖包含更多可操作資訊

普通截圖需要模型做 OCR, 再經過多次搜尋才能定位 Slack channel、使用者或表單欄位。Appshot 除了畫面, 還包含應用程式的 accessibility tree。因此, agent 可能直接取得 channel ID、user ID 與介面控制項, 減少後續工具呼叫。

講者將它用在:

- 從 Slack 畫面啟動問題調查並回覆原討論串。
- 讀取表單欄位, 再交給瀏覽器或 computer use 填寫。
- 在影片、文件或簡報上直接圈選需要修改的部分。
- 把目前畫面當作模糊任務的上下文, 例如「研究這件事並回覆」。

### Skills 與 plugins 的角色

Skills 是由說明文件、相關檔案與 scripts 組成的可重複程序。Plugin 則是一組 skills 與連接能力的套件 ([00:09:21](https://www.youtube.com/watch?v=il1c1a2FufU&t=561s))。

適合建立 skill 的情境是同一工作已經重複發生。例如:

- 依團隊風格做 code review 或文件 review。
- 發生服務事故時, 查詢正確頻道、找負責工程師並建立事件流程。
- 依個人的歷史 email 與 Slack 訊息建立寫作風格指南。
- 固定用特定視覺規則產生 HTML artifact。

講者的個人 skill 採漸進式改善: 先快速做出可用版本, 每次發生錯誤就修正 skill, 並要求 agent 記住新規則。團隊共用版本則需要更仔細檢查路由、責任人與組織規範。

## 2. 建立個人記憶與工作目錄

### Personal monorepo

講者以一個 personal monorepo 作為 Codex 的主要工作入口 ([00:18:37](https://www.youtube.com/watch?v=il1c1a2FufU&t=1117s))。它不是把所有程式碼塞在同一個 repository, 而是集中存放工作脈絡。實際程式碼仍可依 `AGENTS.md` 指示放到其他開發目錄。

主要結構包括:

- `projects/`: 每個 workstream 一個目錄, 含 README、專案層級 `AGENTS.md` 及相關 Slack channel 連結。
- `people/`: 記錄合作對象、關注議題、聯絡方式與所在頻道, 類似個人 CRM。
- Notes 與 summaries: 保存鬆散筆記、agent 摘要及必要的每日紀錄。
- To-do list: 由固定 thread 維護, 必要時委派其他 agent 驗證任務是否完成。

將 vault 本身納入 Git 有一個實用優點: 使用者可以用 diff 查看 agent 最近更新了哪些人物、專案和待辦資訊。這讓記憶更新可審核, 也有助於發現自己錯過的工作進度。

### 記憶需要像新人一樣培養

剛開始時, agent 仍需頻繁收到「查看筆記」「下次記住」「更新 skill」等指示。隨著專案文件、人物資料、skills 與 thread 歷史累積, 明確標註資源的需求會下降。

影片把這比喻成員工 onboarding: 前幾個月必須提供程序與上下文, 允許犯錯, 再把學到的內容寫回系統。不要期待第一天就得到成熟的個人助理。

跨專案記憶混用時, 講者的做法是把差異寫進各專案的 README 與 `AGENTS.md`, 例如指定套件管理器或禁止修改的文件。這是實務經驗, 影片並未提出一套完整的 memory isolation 理論。

## 3. 把 thread 當成長期隊友

### Compaction 改變了 thread 的使用方式

過去常見建議是對話過長就開新 thread, 每個功能或 code review 各自分開。講者認為, 隨著 compaction 改善, 這個限制已顯著降低。他有持續數週、包含大量 sub-agents 的 thread, 仍能保留任務方向。

實作方式是把 thread 命名、釘選並對應到專案或角色。每個 pinned thread 可被看成一名隊友, 例如:

- Chief of staff: 彙整郵件、Slack、行事曆與近期工作。
- 專案管理 thread: 管理實作、文件與相關 sub-agents。
- Support monitor: 監看回報, 建立個別調查 thread 並追蹤結果。
- 個別實作 thread: 負責一段明確工作及驗證。

Sidebar 因此不只是聊天歷史, 而是讓人快速看見目前 workstreams 的工作台。

### Heartbeat 與 loop

Pinned thread 可以透過排程訊息再次被喚醒 ([00:35:08](https://www.youtube.com/watch?v=il1c1a2FufU&t=2108s))。講者稱之為 heartbeat, 並建議把重複檢查送回同一個 thread, 以保留完整脈絡。

典型任務包括:

- 定期查看 pull request, 修正 review feedback, 維持 CI 通過並 rebase。
- 追蹤客服或社群問題, 直到負責人回覆或修正合併。
- 每日上午查詢 connectors, 產出包含原始連結的重點摘要。
- 收到航班報到通知後完成 check-in、下載登機證並傳給自己。

好的 loop 應具備更新頻率、停止條件與安靜輸出。例如, 沒有變化時只回覆 `no updates`, 問題完成後停止高頻檢查, 或依平日與週末調整 heartbeat。

### Goal 與可驗證的長任務

`/goal` 的重點是定義可檢查的完成條件, 在條件尚未滿足時持續工作 ([00:40:14](https://www.youtube.com/watch?v=il1c1a2FufU&t=2414s))。它特別適合測試、建置或其他有客觀 verifier 的工作。

影片中的延伸做法是把 goal、plan、state 和 work log 寫入檔案。好處是長任務執行期間仍能調整範圍, agent 也能根據新發現更新計畫。Work log 則比直接閱讀龐大的 session 紀錄更適合掌握一至兩天的執行進度。

這裡的原則是: 任務越長, 越需要外部化的狀態與明確驗證, 不能只靠一句「做到最好」。

## 4. 從單一 agent 進展到 thread 協作

講者展示一種監控層級的架構:

```text
Monitor thread
  -> 發現新問題
  -> 建立並釘選 triage thread
  -> triage thread 聯絡人員、追蹤 PR、等待回覆
  -> 後續出現相同回報時, monitor 通知原 triage thread
  -> 主 thread 只回報需要人工注意的升級事項
```

Thread 可列出其他 thread、傳送訊息、重新命名與釘選 ([01:07:43](https://www.youtube.com/watch?v=il1c1a2FufU&t=4063s))。因此, 使用者可以要求一個 thread 找出先前的簡報工作、替它重新命名, 或讓三個分段 review thread 完成後再由主 thread 做總體檢查。

這種結構與背景 sub-agent 的差異之一是可見性。釘選的 thread 出現在 sidebar, 人能直接察覺新的工作流或異常。影片也強調這仍是很新的做法, 尚未完全定型。

## 5. 讓結果進入真實世界

Codex 的 artifact 能處理 spreadsheet、Word、PDF、slides 與 HTML 應用。講者用 in-app browser 呈現簡報, 在演練時直接標註空白、版面或拆頁問題, agent 則在背景修改。

外部動作分成兩類:

- 結構化 plugins/connectors: 適合查詢與傳送 Slack、Gmail、Notion、Linear 等服務資料。
- Chrome extension 或 computer use: 適合沒有完整 API 的表單、原生應用與視覺操作。

Computer use 讓 agent 操作 iMovie、填表、測試 native app, 或在 connector 缺少上傳能力時改走 GUI。Remote control 則可從手機向本機 Codex thread 下指令, 在特定設定下也能於闔上螢幕後繼續執行。

## 安全與控制界線

Computer use 的彈性也是主要風險。當 connector 無法寄信或上傳檔案時, 一個被強烈要求完成任務的 agent 可能改用瀏覽器按下送出按鈕。這不是單純的方便功能, 而是真正的安全邊界。

影片提出的控制手段包括:

- 用 permission mode 與 auto review 控制高風險動作。
- 在 `AGENTS.md` 明列不可修改的文件、允許的目錄與行動限制。
- 由組織管理員限制對外 email、外部 Slack channel 等操作。
- 對發送訊息、簽署文件、付款與其他不可逆動作保留人工確認。
- 以 Git diff 或其他審核紀錄檢查 agent 對記憶與文件的修改。

講者表示模型通常相當謹慎, 但也明確承認模型會犯錯且可能被 jailbreak。企業與個人方案的實際資料保留政策並未在影片中得到確切說明。

## Token、模型與推理強度

影片最後提醒, `xhigh` 不等於品質一定最高, 只代表模型投入更多推理 ([01:11:50](https://www.youtube.com/watch?v=il1c1a2FufU&t=4310s))。簡單表單、固定監控或 chief-of-staff 類工作通常可從 low 或 medium 開始。只有複雜、開放式且需要大量推理的任務, 才值得提高 reasoning effort。

節省資源的實際方式包括:

- 依任務難度選推理強度, 不要一律使用最高設定。
- 降低沒有必要的 heartbeat 頻率。
- 沒有更新時限制輸出長度。
- 設定清楚的停止條件。
- 先讓 automation 做高價值摘要, 不必把每個瑣碎狀態都回報給人。

## 可立即採用的最小版本

不必一開始複製完整系統。可以依序建立:

1. 建立一個 personal vault, 先放 `projects/` 與 `people/`。
2. 為每個專案加入 README、`AGENTS.md` 及相關服務連結。
3. 建立並釘選一個 chief-of-staff thread。
4. 設定每天一次的簡報, 要求附上郵件、Slack 與文件原始連結。
5. 遇到重複流程時建立 skill, 並在每次修正後更新它。
6. 只替具有明確停止條件的任務加入 heartbeat 或 goal。
7. 先以低風險、可審核的動作練習 appshot 與 computer use。

## 核心觀念回顧

- 長期價值來自可累積的 context, 不是單次完美提示詞。
- Thread 是可持續的工作單位, skill 是可重複的程序, plugin 是可分享的能力集合。
- 記憶庫要有結構、可更新、可審核, 並以專案規則處理差異。
- Heartbeat 解決「何時再次工作」, goal 解決「何時算完成」。
- Appshot 提供具語意的介面脈絡, computer use 則把能力延伸到缺乏 API 的軟體。
- 自動化越能操作真實帳號與電腦, 權限、停止條件和人工確認就越重要。
- 使用者未來更重要的能力是判斷工作如何分解、何種成果值得產出, 以及培養辨識品質的 taste。

## 來源與可信度限制

本筆記根據 YouTube 提供的英文自動字幕整理, 不是逐字稿。字幕對 `Codex`、產品功能名稱、人名和部分技術詞偶有誤辨, 本文只在上下文足夠明確時修正。現場觀眾問題有些收音不清, 因此問答內容只保留能從講者回答可靠還原的部分。

影片展示的是講者當時的個人工作方式與產品狀態。功能名稱、權限設定、模型版本及介面可能隨時間改變, 不應把個人經驗視為正式產品保證或安全政策。
