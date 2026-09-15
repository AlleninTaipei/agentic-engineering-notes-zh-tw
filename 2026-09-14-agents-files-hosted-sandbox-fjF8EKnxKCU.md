# 從 Python Loop 到檔案式 Agent: Skills, CLI 與託管 Sandbox 的責任分工

> - 影片: [Agents Without Code: Skills, YAML, and Filesystems Replaced Python - Philipp Schmid, Google DeepMind](https://www.youtube.com/watch?v=fjF8EKnxKCU).
> - 頻道: [AI Engineer](https://www.youtube.com/channel/UCLKPca3kwwd-B59HNr-_lvA).
> - 講者: Philipp Schmid, Google DeepMind, 身分依影片標題.
> - 發布日期: 2026-09-14. 此為上傳日期; 示範中講者提到當天是 7 月 2 日, 本文不推定完整錄影日期.
> - 片長: 18:27.
> - Video ID: `fjF8EKnxKCU`.
> - 內容依據: 完整 YouTube 原語英文自動字幕 `en-orig`, 搭配公開 metadata 與章節. 無創作者字幕.
> - 來源性質: 供應商第一方產品架構介紹與現場示範. 未逐幀檢視程式碼畫面, 未執行範例或驗證現行 API.

## 核心觀點

Philipp 用同一個 GitHub PR review agent 示範三種實作. 第一版自行管理 Python loop 與工具, 第二版將重複工作交給 agent framework, 第三版使用遠端 agent 與託管 sandbox, 以指引檔案和 CLI 提供能力.

隨著示範推進, 應用端需要維護的 orchestration code 減少. 開發者仍需定義領域規則, 提供工具與執行環境設定, 並負責 evals 和結果驗證. 影片的重點是改變責任分工, 而非整套系統不再需要程式碼.

## 1. 用 steps 表達 agent 的工作過程

[00:00](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=0s)

講者以 Gemini Interactions API 作為示範介面, 描述其可呼叫模型或 agent, 並支援伺服器端狀態與背景執行.

他指出, 單純交替的 user/model 訊息不容易清楚表達 agent 的中間工作. 示範中的介面以 steps 呈現不同類型的事件:

```text
使用者輸入
  -> reasoning
  -> function call
  -> function result
  -> 後續處理與回覆
```

這樣能將環境回傳資料標記為工具結果, 不必借用 user role 表達. 此處是演講對介面設計的說明, 不提供可直接複製的 API schema.

## 2. 三版 Agent, 哪些責任逐步移出應用程式

[02:45](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=165s) / [05:21](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=321s) / [07:52](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=472s)

| 面向 | 手寫 Python loop | Agent framework | 遠端 Agent 與 sandbox |
| --- | --- | --- | --- |
| 執行迴圈 | 應用自行處理 | Framework 處理 | 託管 agent 處理 |
| 工具選派與結果回傳 | 自行解析 function call 並執行 | Framework 處理對應流程 | 平台與 sandbox 共同執行 |
| 工具定義 | 手寫 JSON schema 與 Python 函式 | 從函式 signature 產生 schema, 仍需實作工具 | 示範改用既有 Bash, 檔案系統與 GitHub CLI |
| 狀態 | 自行管理 | 仍需安排應用與執行環境 | 伺服器管理 session 與 context compaction |
| 開發者重點 | 迴圈, 工具, 指引與 hosting | 工具實作, 指引與 hosting | 指引, skills, 環境設定與 evals |

### 第一版: 明確定義每個動作

程式呼叫模型, 判斷輸出是文字或 function call, 再執行對應 Python 函式. 結果或錯誤要放回後續輸入. System instruction 另存檔案, 工具則有描述與 JSON schema, 底層呼叫 GitHub API.

Agent 能取得 PR 資料與 diff, 但功能受限於提供的工具. 講者以天氣問題測試任務外能力, 它表示無法完成.

### 第二版: Framework 移除重複程式

示範改用 ADK. 原先的 agent loop 檔案消失, framework 接手迴圈, function calling, retries 與 error handling. 工具 schema 可由 Python 函式 signature 產生.

不過, 開發者仍要寫工具函式並安排執行環境. 問到 San Francisco 天氣時, Agent 仍因沒有相關工具而無法回答. Framework 減少了重複實作, 並未自動增加外部資訊能力.

### 第三版: 以指引檔案搭配通用工具

講者介紹 Gemini API 上的 Antigravity remote agent. 他特別區分, 與 IDE 使用相同 harness 不代表完全相同的 agent; 指引與工具可能不同. 演講中的 API 版本被描述為 general-purpose agent, 並具備 Google Search.

PR review 範例不再保留原先的 source directory. 主要工作設定變成 `AGENTS.md` 與一個 Bash 安裝腳本. 指引告訴 agent 可以使用 GitHub CLI, Bash 與檔案系統; 腳本檢查 CLI 是否存在, 缺少時在首次執行安裝.

這裡仍有 API 呼叫與輸入介面程式, 也依賴 CLI 和平台實作. 消失的是這個範例中自行撰寫的 PR 工具與 agent orchestration 部分.

## 3. Sandbox, 來源檔案與憑證代理

[08:42](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=522s)

演講中的 environment 設定為 agent 提供隔離的遠端 Linux sandbox, 可執行命令並儲存檔案. 來源可包括 GitHub repository, GCS bucket 或 inline files.

### 憑證在網路出口注入

講者描述 sandbox 外部有 network proxy. Agent 對外發送請求時, 代理依設定注入憑證, 因而不必讓 agent 直接讀取 token 本身.

```text
Agent 在 sandbox 使用工具
  -> 發出對外請求
  -> Network proxy 依設定注入憑證
  -> 存取外部服務
```

示範為 GitHub API 與 github.com 配置憑證, 以支援不同 GitHub 操作. 另允許一般網路存取, 但不為其他網站提供這些憑證.

講者也提到可限制網域, 並描述預設開放網路. 字幕對「留空」設定的說法不足以還原精確欄位語意, 本文不推定空清單, 省略欄位或萬用字元的實際行為. 這些都應以使用時的官方 API 文件核對.

憑證不暴露給模型, 是這項設計宣稱的邊界. 演講沒有展開重新導向處理, 權限撤銷, 憑證匹配規則或攻擊測試, 不能僅據此認定所有憑證風險都已排除.

### 可重用的 Agent 設定

講者另介紹 agents API, 可用自訂 ID 保存 system instruction, base agent 與 environment 設定. 後續透過該 ID 呼叫, 重用既有配置. 本文保留概念, 不假設 API 名稱或可用性至今不變.

## 4. 同樣的天氣問題, 第三版為何能回答

[12:09](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=729s)

第三版先探索 sandbox, 發現 GitHub CLI 尚未安裝後進行安裝, 再使用 CLI 處理 PR. 開發者不必為讀取 PR, diff 與檔案各寫一個專用函式.

講者再次提問 San Francisco 天氣時, Agent 使用 Google Search 並回覆. 這展示了通用工具可支援原先 PR review 範圍以外的工作.

編輯辨析: 三版提供的能力並不相同. 第三版新增可用的搜尋工具, 因此這個展示不能單獨證明「改成 Markdown」使模型更聰明, 或 framework 無法做到相同工作. 影片也沒有提供三版 PR review 品質, 延遲或成本的對照評測.

示範使用 streaming 呈現中間事件, 並以 previous interaction ID 延續多輪互動. Backend 啟動 sandbox, 載入指引與 skills, 再處理模型呼叫與工具結果的往返.

## 5. 平台接手執行機制, 團隊保留領域責任

[13:50](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=830s)

| 平台依演講描述接手 | 開發團隊仍需負責 |
| --- | --- |
| Agent loop 與工具選派 | 領域指引, 規則與預期行為 |
| Conversation 與 session state | 在 `AGENTS.md` 表達工作要求 |
| Context window 管理與 compaction | 透過 skills 提供能力與必要 context |
| 遠端隔離執行環境 | 工具與環境配置, evals 與結果驗證 |

### 擴充安全掃描的例子

講者以 PR security scanning 說明擴充方式. 先前的做法需要撰寫 Python 工具, 定義 schema 並加入工具集合. 檔案式做法則可以提供 skill 指引, 說明該用哪個 CLI, 並將工具放入環境.

減少的是新增能力時的包裝與接線工作. 掃描工具本身仍需存在, 指引也需讓 agent 知道何時使用及如何判讀結果. 演講沒有實際展示安全掃描的結果或準確率.

### 以檔案外部化 context

講者也描述將偏好, 規則與筆記寫入檔案, 供後續工作使用. 長任務中出現新的工作方向時, 可先將交接資訊存檔, 稍後再處理.

這段說明的是工作模式. 影片未詳述跨 session 檔案持久化, 版本管理或錯誤記憶的清理政策, 不宜推定 sandbox 中所有檔案都會自動永久保存.

## 6. Harness 複雜度是需要重新檢查的訊號

[16:22](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=982s) / [17:11](https://www.youtube.com/watch?v=fjF8EKnxKCU&t=1031s)

講者提出一項經驗法則: 當模型變強, harness 卻持續變得更複雜時, 團隊應檢查是否過度設計. 他鼓勵以通用工具讓模型探索解法, 將重心放在領域指引, workflows, evals 與結果驗證.

演講引用 Cursor 將約 12,000 行 TypeScript orchestration 改成約 200 行指引檔案的案例, 並提到其他團隊頻繁重構 harness 或減少工具數量. 這些是講者引用的外部案例, 公開說明未附相應原始資料, 本次未另外核實. 不將它們視為本場示範的測量結果.

編輯辨析: 程式碼減少可作為維護方式改變的線索, 不能單獨代表可靠性提升. 本場缺少平台依賴, 費用, 可攜性與除錯能力的完整比較. 是否移除特定 orchestration, 仍需由任務結果與評測支持.

## 編輯整理: 可用於評估遷移的問題

以下為依影片整理的檢查問題, 非講者提供的完整遷移程序.

1. 現有程式哪些只是重複的 loop 與工具包裝, 哪些包含不可省略的業務規則?
2. 專用工具是否能由成熟 CLI 與清楚指引取代, 取代後的行為如何驗證?
3. Sandbox 的檔案生命週期, 網路範圍與憑證注入是否符合任務需求?
4. 三種方案是否在相同工具能力與相同任務下比較品質, 延遲與完整成本?
5. 誰維護 skills 與 evals, 如何發現模型更新或 compaction 後的退步?

## 來源與限制

- 本文轉述完整英文自動字幕, 移除重複片段與現場等待. 章節時間連結依影片公開 metadata.
- 字幕中 tools routing, evals 與檔案名稱的明顯誤辨依上下文整理. 標題雖提到 YAML, 口述未提供足夠 YAML 內容, 因此未重建設定檔.
- 未逐幀核對操作畫面, 也未取得可重現的示範 repository. 文中流程是口述整理, 不代表已執行或測試的程式.
- API 功能, 網路預設與產品可用性均為演講當時描述, 不作為現行操作文件. 本次未查證影片以外的產品資訊.
- PR review 與天氣示範呈現了功能差異, 未提供正式品質評測. Harness 簡化的外部案例也未另行核實.
