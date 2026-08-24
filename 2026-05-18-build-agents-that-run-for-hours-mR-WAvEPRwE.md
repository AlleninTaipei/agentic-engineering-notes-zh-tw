# 建立能連續執行數小時的 Agent

來源: [Anthropic Workshop: Build Agents That Run for Hours, Ash Prabaker & Andrew Wilson](https://www.youtube.com/watch?v=mR-WAvEPRwE), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=mR-WAvEPRwE
- 上傳日期: 2026-05-18
- 片長: 01:15:40
- Video ID: `mR-WAvEPRwE`
- 內容依據: 英文原始語言自動字幕

## 摘要

這場工作坊討論如何讓 agent 從只能工作數十分鐘, 進展到能連續執行數小時甚至數天. 單靠更大的 context window 並不夠, 因為長時間任務還要面對規劃失準, context rot, 半成品被誤判為完成, 以及模型對自己產出的偏袒.

影片提出的核心架構是將工作拆成 planner, generator 與 evaluator 三種角色. Planner 定義高層產品方向, generator 與 evaluator 先協商可測試的完成契約, generator 再實作, evaluator 則以獨立 context 實際操作產品並提出批評. 這個對抗式分工比要求同一個 agent 自我檢查更可靠.

更重要的結論是, harness 沒有一個永久正確的版本. 模型能力持續進步, 原本必要的 fresh context, 細粒度 sprint 或高頻 evaluator 可能變成負擔. 開發者必須讀 trace, 了解當代模型的尖峰能力與弱點, 再決定該保留或刪除哪些 scaffold.

## 長時間 Agent 的三個主要難題

[00:02:29](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=149s)

### Context

Context window 有限. 新 session 帶來近似失憶的問題, 長 session 則可能產生 context rot, 使模型愈到後段愈不連貫. 講者也提到 context anxiety, 模型在接近容量上限時可能急著收尾, 留下未完成工作.

### Planning

模型可能嘗試一次完成過大的任務, 只實作半個功能就停止, 或在 context 用盡時留下半成品. 長時間任務需要可持續的規劃與狀態, 不能只依靠最初的一次性計畫.

### Judgment

最不直觀的難題是模型不擅長判斷自己的輸出. 它可能看到按鈕已出現在畫面上, 就宣告功能完成, 即使按鈕沒有後端行為. 這也是 generator 與 evaluator 必須分離的主要原因.

## 模型與 Harness 共同演進

[00:04:14](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=254s)

改善長時間 agent 有兩條路:

1. 將能力訓練進模型權重, 強化規劃, context 管理與工具選擇.
2. 改善模型周圍的 harness, 包含 agent loop, 工具, MCP, sub-agent, skills, 權限與持久化狀態.

講者的歷史回顧顯示, Claude 的模型版本與 Claude Code harness 一直共同演進:

| 時期 | 模型或工具進展 | 對長時間工作的意義 |
| --- | --- | --- |
| Sonnet 3.5 前後 | Artifacts, Computer Use, MCP | 模型開始能查看成果, 操作介面與使用外部工具 |
| Sonnet 3.7 | Claude Code research preview | 以實際開發行為反饋模型與 harness 設計 |
| Opus 4 / Sonnet 4 | Claude Code GA, SDK | 更能管理 context 並完成任務 |
| Sonnet 4.5 | context awareness, checkpoints, Agent SDK | 能追蹤 token 使用量, 回復先前狀態, 並將 coding harness 泛化到其他工作 |
| Opus 4.5 | 更好的規劃, 經濟的 sub-agent | 可由較強模型規劃, 再由較便宜模型執行 |
| Skills 與程式化工具呼叫 | progressive disclosure, 減少工具輸出進入主 context | 提高 context 使用效率 |
| Opus 4.6 / Sonnet 4.6 | Agent Teams, server-side compaction, 1M context | 更長的連續工作, agent 之間可直接協調 |

講者引用的衡量方式是, 在最小 scaffold 下完成 50% 任務的可持續時間. 簡報中的數字從約 1 小時提高至約 12 小時. 這是特定測量設定下的結果, 不代表所有實務任務都能可靠執行相同時數.

## Ralph Loop 的價值與限制

[00:07:55](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=475s)

Ralph loop 的基本形式是反覆把工作交給 Claude Code, 直到完成所有任務. 完整做法不只是重複相同 prompt, 而是先規劃功能, 每回合選取一項工作, 並以新的 context 執行.

它的重要精神是可預測地失敗, 優於不可預測地成功. 另一種 Claude Code plugin 實作則在單一 session 中使用 stop hook 攔截結束, 只要尚未滿足 safe word 或最大迭代次數, 就讓模型繼續工作. 這種版本依賴 compaction, 因此不完全等同於每回合建立 fresh context 的原始模式.

Ralph loop 的限制是缺少真正獨立的反方. 固定的 `plan.md` 能維持方向, 卻沒有人在實作前質疑範圍與驗證方法, 也容易在同一條錯誤路徑上持續修補.

## 第一代長時間 Harness

[00:12:05](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=725s)

早期做法由 initializer agent 接收模糊需求, 再建立持久化產物:

- `featurelist.json`, 記錄功能與測試狀態. 講者觀察到模型較不容易任意覆寫 JSON.
- progress file, 保存跨 session 的進度.
- Git repository 與 init script, 讓每次 session 能快速恢復環境.
- 每項功能是否通過測試的旗標.

執行迴圈會在 fresh context 中重新了解工作目錄與進度, 跑 smoke test, 選一個尚未通過的功能, 完成實作與瀏覽器測試, 通過後提交 Git commit 並更新狀態. 若仍有未完成項目, 就在下一個 fresh context 重複流程.

這個架構把良好的預先規劃, 持久化狀態, 獨立 session 與 verification loop 結合起來. 後續模型更強後, 其中一些步驟可以簡化, 但這些原則仍是後來架構的基礎.

## 對抗式 Generator-Evaluator

[00:17:28](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1048s)

新架構借用 generative adversarial network 的直覺:

```text
Planner -> 高層產品規格
             |
             v
Generator <-> Evaluator
  實作        協商契約, 實際操作, 評分與批評
```

Generator 與 evaluator 擁有不同的 context window, system prompt 與職責. Evaluator 不只閱讀 diff, 還會使用 Playwright 開啟實際頁面, 點擊操作, 觀察畫面並把問題交回 generator.

Evaluator 仍是 LLM, 也可能偏愛 LLM 產出的內容. 但講者認為, 將獨立 critic 調整得嚴格, 比要求 builder 對自己的作品保持嚴厲更容易. 人類也常能辨認一道菜或一幅畫的缺點, 卻未必能親自做得更好. Harness 正是利用批評能力與生成能力之間的落差.

## 主觀品質也能建立 Rubric

[00:21:30](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1290s)

功能正確不是完整產品的唯一標準. 前端還涉及外觀, 使用感受, 原創性與品味. 講者將評分拆成四項:

- Design
- Originality
- Craft
- Functionality

權重可依模型弱點調整. 當模型已很擅長基本功能時, 可以提高 design 與 originality 的權重, 避免紫色漸層或同質化的 "AI slop" 視覺. 再以少量參考案例校準 evaluator, 讓其判斷逐漸接近團隊的品味.

對抗式 evaluator 的另一項優點是敢於推翻方案. 如果 generator 多輪修改後仍無法提高 originality 分數, evaluator 可以要求全部重做. 單一 agent 或單純循環通常較容易繼續修補原有方案, 因為它對自己的既有成果有依附.

## Planner 應保持高層

[00:23:44](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1424s)

Planner 把一行需求轉成高層產品規格與一系列 sprint, 但刻意不預先決定所有技術細節. 過度具體的初始計畫如果出錯, 錯誤會沿著每個 sprint 放大.

這個結構接近一個小型產品團隊:

- Planner 類似 PM, 定義產品邊界與方向.
- Generator 類似實作者, 選擇技術方法並完成工作.
- Evaluator 類似 QA 或 critic, 驗證是否真正達成目標.

角色分離的目的不只是模仿組織圖, 而是讓每一種責任擁有乾淨, 專注的 context.

## 先協商 "完成" 的定義

[00:25:04](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1504s)

Generator 在寫程式前, 先與 evaluator 透過磁碟上的檔案協商契約. Generator 提議要建立哪些功能及如何驗證, evaluator 則檢查:

- 範圍是否過大.
- 測試是否太弱.
- 是否遺漏邊界條件.
- 每項結果是否可觀察與可重現.

雙方同意後才開始實作. Evaluator 最後依雙方同意的 contract 評分, 而不是直接用 planner 的模糊規格評分. 這一步把 user story 轉成具體, 可測試的 assertions, 同時避免 planner 過度指定實作細節.

## Retro Forge 案例

[00:26:00](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1560s)

講者用相同模型與相同的一行 prompt, "build a retro game maker", 比較單一 agent 與新 harness.

單一 agent 產生的介面表面完整, 有 sprite editor, palette, frame timeline 與 preview, 但真正進入 play mode 後, 方向鍵與空白鍵沒有反應. 它理解外觀, 卻沒有建立足夠的實際測試來確認遊戲能玩.

Planner-generator-evaluator harness 約執行 6 小時, 花費約 200 美元, 建立名為 Retro Forge 的產品. 它加入完整色盤, 尺度一致的 sprite preview, AI level assistant, 可運作的物理迴圈, 碰撞與 debug HUD. Evaluator 實際啟動遊戲並操作, 因此能抓到僅靠 unit test 不易發現的 route ordering 與鍵盤刪除邏輯問題.

這是示範當時的特定成本與結果, 不是效率或價格保證. 案例的意義是, 同一模型因 scaffolding 不同, 可以從 "看起來完成" 進展到 "能被實際操作與驗證".

## 契約必須夠具體

[00:31:28](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=1888s)

Retro Forge 的 evaluator 使用了 27 項 contract criteria. 講者的經驗是:

```text
模糊標準 -> 模糊批評 -> generator 不知道該改什麼
具體標準 -> 可定位問題 -> generator 能採取明確修正
```

Claude 預設並不是嚴格的 QA agent. 它可能找到 bug, 卻以 "之後再修" 結案. 團隊必須閱讀 trace, 找出模型判斷與人類判斷分歧的位置, 再調整 prompt. 這種除錯方式類似閱讀 stack trace, 而不是只看最終成功率.

可行的輔助方式包括把 agent transcript 保存下來, 用文字搜尋定位模式, 或由另一個 agent 先標示可能偏離的位置. 但問答中講者仍強調, 人工逐行閱讀完整 trace 最能理解模型為何做出某個選擇.

## Harness 必須隨模型調整

[00:34:14](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=2054s)

模型版本改變後, 舊 harness 可能包含不再需要的複雜度. 講者列出幾個調整案例:

| Harness 元件 | 較早模型 | 較新模型的觀察 |
| --- | --- | --- |
| session 間重設 context | 用來處理 context anxiety | 可改成長 session 加 compaction |
| 細粒度 sprint decomposition | 維持長任務一致性的關鍵 | 模型可連續實作約 2 小時, 不必每次只餵一個功能 |
| evaluator 執行頻率 | 每個 sprint 都檢查 | 可在一輪完整生成後再檢查 |
| 共享狀態 | 依賴對話 context | 偏好以檔案系統保存, 降低跨角色耦合 |

這不代表原本設計錯誤. 它對當時模型是正確的, 只是能力前緣移動了. 新模型發布後應重新跑 eval, 尋找可以移除的 scaffold, 避免舊限制變成新瓶頸.

## 自己建立 Harness 的最小元件

[00:37:56](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=2276s)

影片建議可以從現有 primitive 組合, 不必等待內部系統公開:

- Custom sub-agent, 分離 generator, evaluator 與 QA 角色.
- 嚴格且具體的 evaluator system prompt 與 rubric.
- Playwright MCP, Claude for Chrome MCP 或 Computer Use, 實際操作成果.
- Skills, 封裝可重用的評分規則與領域知識.
- 檔案系統, 保存合約, 進度, 嘗試過的方法與跨 context handoff.
- 權限與停止條件, 限制自主執行的風險與時數.

## 五項核心結論

[00:39:01](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=2341s)

1. 自我評估是一個陷阱, 使用獨立且對抗式的 evaluator.
2. Compaction 不等於 coherence, 有損摘要仍會漂移. 使用結構化 handoff 與乾淨 context.
3. 主觀品質可以評分, 前提是團隊能把 "好" 寫成明確 rubric.
4. 詳讀 trace, 找到模型判斷偏離人類的位置.
5. 模型進步後, 重新決定 scaffold 哪些要刪, 哪些要留.

## 問答補充

[00:40:05](https://www.youtube.com/watch?v=mR-WAvEPRwE&t=2405s)

### Agent Teams 與 generator-evaluator 並不衝突

Generator-evaluator 可視為 agent team 的一種專門化配置. 前端, 後端或整合 agent 甚至可以各自配一個 critic. Claude Code 適合快速實驗這些角色, Agent SDK 則較適合部署到雲端或 sandbox 中長時間執行.

### 不要把 generator 的完整思考直接交給 evaluator

講者測試過共享更多 generator context, 但認為容易混濁兩個角色的判斷. 更有效的方式是讓 evaluator 只判斷輸出並描述問題, 再讓 generator 自己推導成因與修法.

### Traceability 仍是未完全解決的問題

團隊會用另一個 agent 對大量 trace 做第一輪定位, 但主要方法仍是人工閱讀. 讀 trace 的能力包含對模型工作條件的同理, 例如 browser agent 只能間歇看到靜態畫面, 其操作困難與人類持續觀看頁面不同.

### Greenfield 與 brownfield

影片展示的模式較適合 greenfield 專案, 因為技術棧與 rubric 都很有主見. Brownfield 專案仍能使用相同原則, 但必須依既有架構, 測試與品質標準客製化. 在既有系統中, agent 也可以串接監控, issue 生成, PR 實作與 review, 自動化完整軟體生命週期的一部分.

### Human-in-the-loop

若需要人工檢查點, 可用 hook 在 evaluator 的特定停止條件注入人類回饋. 不過講者研究的目標是先找出能全自主運作的模式. 他們會同時執行多個版本, 閱讀失敗 trace, 修正 harness, 直到可靠度足以降低人工介入.

### 長期產品的持久化資訊

問答中提到以 JSON 保存嘗試, 評估, bug, 修正與結果, 形成可供下一個模型或人類接手的 breadcrumbs. 再搭配持續更新的高層文件與檔案結構說明, 通常足以讓 Claude Code 和開發者延續工作.

### Context 策略沒有單一答案

1M context 與較強模型使單一 session 加 compaction 成為可行方案, 但 fresh session 是否更適合仍取決於 use case 與 eval. Context rot 是當代模型的限制, 不應把針對它的 workaround 當成永久架構.

## 實作檢查表

### 開始前

- 用高層 spec 定義產品邊界, 避免提前鎖死所有技術細節.
- 把品質拆成可評分項目, 每項附具體正反例.
- 分離 generator 與 evaluator 的 context, prompt 和工具權限.
- 設定最大時間, 成本, 迭代次數與停止條件.

### 每個工作週期

- Generator 與 evaluator 先協商可測試的完成契約.
- 將合約和進度寫入持久化檔案.
- 實作後執行 unit, integration 與實際介面操作測試.
- Evaluator 只依輸出與契約評分, 不沿用 generator 的自我敘事.
- 保留 trace, 失敗原因與已嘗試修法, 供下一輪接手.

### 模型升級後

- 用相同代表性任務重跑 eval.
- 檢查是否仍需要頻繁 context reset.
- 調整 sprint 大小與 evaluator cadence.
- 重新校準 rubric 權重.
- 移除已由模型本身穩定處理的 scaffold.

## 編輯補充

影片強調全自主 agent, 但實際導入時還應依錯誤成本調整權限. 可逆的開發工作適合較長的自主迴圈, 涉及 production deployment, 資料刪除, 金流或外部溝通時, 應保留清楚的人工核准點. 這是根據影片架構所做的風險管理延伸, 不是講者逐字提出的規則.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕可能誤辨人名, 產品名稱與較新的模型術語. 內容已轉寫成學習筆記, 省略開場寒暄, 重複語句與部分較細的現場問答, 不是逐字稿.

影片中的模型能力, 價格, context 規格與產品功能反映錄影當時的狀態. Retro Forge 的時數與成本是示範案例, 不應直接推廣為其他任務的預估值.
