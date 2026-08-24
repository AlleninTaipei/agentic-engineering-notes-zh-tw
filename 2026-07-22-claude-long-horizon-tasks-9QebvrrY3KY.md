# Claude 長時程任務: 非同步 Agent、Verifier 與自我學習記憶

> [!info] 來源
> - 影片: [Claude for Long-Horizon Tasks — Lance Martin, Anthropic](https://www.youtube.com/watch?v=9QebvrrY3KY)
> - 頻道: AI Engineer
> - 發布日期: 2026-07-22
> - 片長: 25:18
> - Video ID: `9QebvrrY3KY`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

當模型能自主工作數小時後, agent 產品不能只把聊天介面搬到背景執行。它需要持久且可回查的 session、與執行環境解耦的 harness、獨立 verifier、可修正的長期記憶, 以及面向整個組織的多人協作與主動通知能力。

## 模型能力改變產品介面

講者 Lance Martin 把 Claude 比喻成光源, 產品則是讓能力穿透出來的窗戶。當模型能力改變, 合適的產品介面也會跟著改變。

他以 task horizon, 即模型能自主完成工作的時間長度, 說明三個階段 ([01:00](https://www.youtube.com/watch?v=9QebvrrY3KY&t=60s)):

| 大致能力階段 | 合適的產品介面 | 人類參與方式 |
| --- | --- | --- |
| 約 10 至 20 分鐘 | Autocomplete、chat | 人類持續在迴路中引導 |
| 約 1 小時 | 本機同步 coding agent | Agent 可連續工作, 人類仍容易即時介入 |
| 更長時程 | 非同步 agent | Agent 在背景執行, 完成或需要協助時再通知人類 |

如果模型只能自主工作很短時間, 非同步體驗反而不理想。它可能剛離開使用者視線就撞上錯誤, 很快又要求人工介入。較長的 task horizon 才讓 async agent 真正有價值。

## API surface 的三個層次

影片描述 Anthropic agent 介面的演進:

1. Messages API: 提供 prompt-response 基礎, 團隊自行建立並部署 harness。
2. Agent SDK: 以程式方式呼叫 Claude Code 類型的 harness, 省去自行實作 loop。
3. Managed Agents: 同時包裝 harness 與受管理的部署基礎設施。

後續四個主題不只適用於特定產品, 也可作為長時程非同步 agent 的一般架構原則。

## 主題一: Brain 與 hands 解耦

Brain 指模型與 harness, hands 指實際執行工作的 sandbox 或 container。早期將兩者放在同一個 container, 會帶來兩類風險 ([03:19](https://www.youtube.com/watch?v=9QebvrrY3KY&t=199s)):

- Harness 或 container 故障時, 整個 session 可能一起遺失。
- Agent 長時間自主執行時, 把憑證直接放在相同環境會擴大安全風險。

影片提出的架構是:

```text
stateless harness (brain)
          ↓
append-only session event log
          ↓
一個或多個 sandbox/container (hands)

credentials → 獨立 vault, 不放入 sandbox
```

Session 是 append-only event log ([04:30](https://www.youtube.com/watch?v=9QebvrrY3KY&t=270s))。Harness 本身可以無狀態化, sandbox 也只是可替換的執行資源。因此無論 harness 或 sandbox 故障, 工作紀錄仍留在 session 中, 新 process 可以從中恢復。

更進一步, 一個 harness 可以管理多個 hands, 把工作分派給不同 container。這使平行執行與隔離成為架構層能力, 而非全塞在同一 process 中。

## Session 是外部 context object

傳統 compaction 會選擇要保留的資訊, 其餘上下文可能永久消失。影片把持久 session 視為模型外部的 context object:

- 原始 session event log 保持不被破壞, 只允許追加。
- 模型眼前的 context window 是從 session 選出的工作切片。
- 被 compaction 移出的內容仍存在, 模型之後可以重新查找 ([05:57](https://www.youtube.com/watch?v=9QebvrrY3KY&t=357s))。

這裡的關鍵不是讓 context window 無限大, 而是把「目前放進模型的內容」與「可長期回取的完整紀錄」分離。

## 主題二: 使用獨立 verifier

若同一個 context 既負責產出工作, 又負責評分自己的成果, 模型可能合理化既有答案, 或受前面累積資訊干擾。影片建議把驗證放進獨立 context ([06:20](https://www.youtube.com/watch?v=9QebvrrY3KY&t=380s))。

```text
Build agent 執行
        ↓
Verifier agent 依 goal/rubric 檢查
        ↓
未達標 → 回饋後繼續執行
達標   → 結束
```

Verifier context 可以只放目標、rubric、候選成果與必要證據, 專門調整成批判性檢查模式。影片將 Claude Code 的 goals 與 Managed Agents 的 outcomes 視為相似 primitive, 都是在建立可衡量的終止狀態。

### Parameter Golf 案例

講者用 Parameter Golf benchmark 展示 verifier loop ([08:24](https://www.youtube.com/watch?v=9QebvrrY3KY&t=504s))。任務要求模型在限制時間與硬體條件下訓練小型模型, 並符合 benchmark 的實驗規格。

他透過 outcomes 要求 agent 完成固定數量的迭代並符合實驗條件。環境與 verifier 持續提供回饋, 讓模型自行修正。講者的結論是, 高能力模型配合軟體 loop 與可量測驗證, 是長時間非同步工作的有效通用模式。

這個案例的核心不是把人工完全移除, 而是把原本由人類反覆給予的 steering signal 編碼進環境與 verifier。

## 主題三: In-band memory 與 dreaming

影片借用人類記憶作類比 ([10:12](https://www.youtube.com/watch?v=9QebvrrY3KY&t=612s)):

- 白天的海馬迴快速記錄近期經驗, 類似 agent 執行中的短期記憶寫入。
- 睡眠期間把重要內容整合進長期記憶, 類似離線 dreaming。

這只是解釋系統設計的類比, 不是主張模型以人腦相同機制運作。

### In-band memory

In-band memory 是 agent 在執行任務過程中自行寫入記憶 ([11:08](https://www.youtube.com/watch?v=9QebvrrY3KY&t=668s))。實作可以很簡單, 例如提供能讀寫 memory directory 的工具。

影片用 Claude 玩 Pokémon 與 Continual Learning Bench 說明, 講者觀察到較新的高能力模型更擅長:

- 判斷哪些資訊值得留下。
- 從單次事實提煉可供未來使用的抽象規則。
- 撰寫策略性記憶, 而不只是當下的操作筆記。

不同容量模型的主要差異之一, 在於能否完成這個 distillation step。

### 為什麼仍需要 dreaming

執行中的記憶可能錯誤, 或只對當前任務局部最佳。Dreaming 是離線、out-of-band 的整理程序 ([13:19](https://www.youtube.com/watch?v=9QebvrrY3KY&t=799s)):

1. 讀取現有 memory store。
2. 回顧多次過去 session 或 execution trace。
3. 比對結果, 找出矛盾、錯誤與可泛化經驗。
4. 修正並重新整理長期記憶。

### Pokémon 錯誤記憶案例

Agent 寫入一條錯誤的遊戲位置記憶, 因而反覆錯判所在位置並掉入陷阱 ([14:20](https://www.youtube.com/watch?v=9QebvrrY3KY&t=860s))。講者表示, 使用原始 memory store 的五次測試全都重現此問題。加入 dreaming 後, 系統能從過去 traces 找到並修正錯誤, 之後順利前進。

這個案例揭示一項重要風險: 持久記憶會放大錯誤。一條不正確但被反覆讀取的記憶, 可能比沒有記憶更糟。因此 memory 系統需要校正、版本與評估機制。

## 主題四: 從單人 harness 到組織級 harness

傳統 coding agent 多半是 single-player。它在個人電腦上執行, 使用個人的 context、設定與憑證。影片以 Claude Tag 為例, 說明 org-level harness 的不同之處 ([16:52](https://www.youtube.com/watch?v=9QebvrrY3KY&t=1012s)):

- 組織內所有成員都能存取同一套 harness。
- Agent 有獨立身分, 不綁定特定使用者。
- 可使用組織層級 context 與授權。
- 多人可共同引導同一套能力。
- 新成員第一天就能使用已建好的 connectors 與工作流程。

它能協助檢查他人是否做過相同實驗、減少重複研究、搜尋內部知識, 並讓能力不再取決於個人是否花了數週調校自己的 agent。

### Reactive 走向 proactive

本機同步 agent 通常等待使用者下指令。擁有組織 context 的 async agent 則可能主動監控並通知使用者值得注意的事情 ([18:26](https://www.youtube.com/watch?v=9QebvrrY3KY&t=1106s))。

這種 proactivity 也引入新的產品與治理問題, 例如通知條件、權限邊界、錯誤警報、敏感資訊揭露與不同使用者同時 steering 時的衝突處理。

## Q&A: 長時程能力不只取決於模型

有人詢問為何 frontier model 在長時程 benchmark 上與其他模型拉開差距。講者沒有斷言單一原因, 而是指出可靠的長時程 agent 需要多項能力共同成立:

- 模型本身的推理與操作能力。
- 能表達使用者偏好及求助時機的 memory。
- 對 prompt injection 的抵抗與其他安全措施。
- Brain/hands 解耦及失敗恢復架構。
- 適合長時間工作的 agent product 與 infrastructure。

因此 task horizon 不是單看模型規格就能推導的產品能力。

## Q&A: 記憶儲存不必限定為檔案系統

對 file system 或 database 哪個更適合記憶, 講者認為底層形式不是關鍵 ([22:10](https://www.youtube.com/watch?v=9QebvrrY3KY&t=1330s))。較重要的是提供簡單、通用且可程式操作的 primitive, 讓模型自行管理記憶。

他不建議預先規定過度細緻的 memory schema, 例如先決定模型只能儲存哪些記憶類型。理由是模型能力提高後, 可能比開發者更能判斷如何組織自己的工作記憶。

這不代表產品不需要資料治理。可讓模型自由組織內容, 同時仍由系統強制執行權限、保留期限、來源追蹤、刪除與稽核政策。

## 實作上的整體架構

```text
使用者或組織目標
        ↓
Build agent / stateless harness
        ↓
持久 append-only session log ←→ 可回取的 context
        ↓
一個或多個隔離 sandbox
        ↓
成果 → 獨立 verifier → 未達標則回到 loop

執行期間: agent 寫入 in-band memory
離線期間: dreaming 回顧 traces 並修正長期 memory
```

## 實務檢查表

- Async agent 的任務是否真的長到值得離開同步互動介面?
- Harness 或 sandbox 中斷後, session 能否完整恢復?
- 完整 event log 是否獨立於目前 context window 持久保存?
- 憑證是否存放在獨立 vault, 而非交給模型或 sandbox?
- 是否以獨立 context 的 verifier 檢查成果?
- Goal 或 rubric 是否可量測, 並設定迴圈上限?
- In-band memory 是否保留來源與寫入時間?
- Dreaming 是否能找出錯誤記憶, 且有 eval 證明它值得額外運算?
- 組織級 context 是否依使用者身分與用途實施權限控制?
- Proactive 通知是否有明確門檻、降噪與人工介入方式?

## 時效性與限制

影片發布於 2026 年 7 月, 內容包含當時 Anthropic 的產品名稱、模型世代、研究觀察與未來方向。Managed Agents、Claude Tag、goals、outcomes 等功能的介面與供應狀態可能改變, 實作前應查閱當前官方文件。

本筆記依英文原始自動字幕整理。影片沒有提供 YouTube 章節, 因此本文結構與段落名稱是依講者的四個主題及內容轉折編輯而成。Benchmark 數字、模型比較及 Pokémon 重複測試均為講者在影片中的報告, 不是本筆記進行的獨立驗證。

## 結語

長時程 agent 的核心問題不是如何塞進更多 context, 而是如何讓工作在 context 之外仍可持久、回查、驗證與修正。Append-only session 保存事實來源, verifier 提供獨立完成標準, memory 保留跨任務經驗, dreaming 則負責修正長期累積的偏差。當這些能力進一步共享到組織層級, agent 的使用方式也會從個人、同步、被動, 逐漸轉向多人、非同步與主動。
