# Research-Plan-Implement 哪裡做錯了: 從 Magic Words 到可控工作流

> 影片: [Everything We Got Wrong About Research-Plan-Implement - Dexter Horthy](https://www.youtube.com/watch?v=YwZR6tc7qYg), AAIF Live<br>
> 頻道: AAIF Live<br>
> 上傳日期: 2026-03-24<br>
> 活動日期: 2026-03-03<br>
> 片長: 26:46<br>
> Canonical URL: https://www.youtube.com/watch?v=YwZR6tc7qYg<br>
> Video ID: `YwZR6tc7qYg`<br>
> 內容依據: 英文原始自動字幕

## 摘要

Dexter Horthy 回顧 HumanLayer 推廣的 Research-Plan-Implement (RPI) 方法, 並公開修正其中幾項重要主張. RPI 對熟練使用者有效, 但團隊大規模採用時出現不一致: 研究混入實作意見, planning agent 跳過人類問答, 使用者必須知道 magic words, 千行 plan 的審查沒有降低總工作量, 而不讀程式碼最終造成大量重寫.

新版方向不是尋找更強的單一 prompt, 而是把研究與規劃拆成多個小型 context windows, 以真正的 control flow 串接 questions、research、design、structure outline、plan、worktree、implementation 與 pull request. 每個階段只承擔較少指示, 並把共享理解寫入持久 Markdown artifacts.

人類應把注意力放在高槓桿位置: 先審查約 200 行的 design discussion 與較短的 structure outline, 在模型寫出數千行程式碼前修正方向. Tactical plan 主要供代理執行, 人類只需 spot-check, 但正式環境的最終程式碼仍必須由工程師閱讀並負責.

## 1. RPI 原本想解決什麼問題

[01:10](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=70s)

RPI 的背景是 coding agent 雖能提高產出, 也可能帶來大量 rework. 講者引用較早的產業觀察: 團隊看似多交付約 50%, 卻可能把其中一半時間用在清理前一週產生的低品質程式碼. 舊模型在低複雜度 greenfield 任務表現較好, 面對高複雜度 brownfield codebase 則更容易失敗.

影片也提醒, 這些數字來自較早的模型世代, 講者認為新模型表現應更好. 因此這段不是 2026 年模型的正式 benchmark, 而是 RPI 方法形成時的問題背景.

RPI 希望先研究既有系統, 再建立計畫, 最後才讓代理實作, 以減少對大型 codebase 的誤解和重工.

## 2. 哪些原則仍然成立

[02:35](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=155s)

講者認為 RPI 有幾項核心原則仍然正確:

- 不存在能解決所有問題的 magic prompt.
- 不要把思考責任外包給模型.
- 工程師是流程中的必要參與者.
- 應尋找 leverage, 在錯誤變成大量程式碼之前重新導向.
- 正式環境不應接受無人負責的 slop.

問題不是研究、規劃與實作的順序本身, 而是原版工具如何依賴使用者經驗, 長 prompt 與不穩定的模型 instruction following.

## 3. 研究階段為何容易得到錯誤結果

[03:11](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=191s)

熟練工程師不會只把 ticket 交給 agent 並要求「研究 codebase」. 他們會先把需求拆成能迫使模型走過相關模組的問題, 例如:

- 系統中的 endpoint 通常如何建立?
- 某個 domain object 的資料與控制流程經過哪些層?
- 哪些 worker 會處理相關背景工作?
- 相同能力在其他模組採用什麼 pattern?

理想的 research artifact 應該壓縮目前程式碼庫的客觀事實, 而不是提出新功能應如何實作.

### Ticket 會污染研究脈絡

如果研究 agent 同時知道「我們準備建立什麼」, 它容易從描述事實轉向替未來方案辯護. 研究文件便混入偏好與實作決策, 後續規劃會把未經人類確認的假設當成現況.

新版做法先在一個 context window 根據 ticket 產生 research questions, 再把問題交給全新的 context window. 第二個 agent 看不到 ticket, 只負責依問題查明 codebase 現況.

```text
Ticket
  -> Question planner: 產生需要查明的問題
  -> Fresh research context: 只看問題與 codebase
  -> Objective research artifact
```

這與 query planning 類似. 先決定要查什麼, 再讓檢索或探索程序在不受目標答案暗示的情況下收集證據.

## 4. Planning prompt 為何跳過人類對齊

[05:20](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=320s)

原版 planning prompt 是一個包含約 85 項指示的 monolithic prompt. 它要求模型:

1. 讀取 ticket 與研究文件.
2. 啟動 subagents 補充 codebase 事實.
3. 提出設計選項.
4. 詢問使用者並取得回饋.
5. 顯示實作階段順序.
6. 根據使用者意見修改 outline.
7. 最後才寫出 plan file.

熟練使用者會明確要求 agent 先提出 open questions, 並與自己來回討論後才寫 plan. 但許多使用者只執行 `create plan`, 模型便直接替他們做完所有設計決定.

## 5. 需要 Magic Words 代表工具設計失敗

[07:06](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=426s)

團隊發現, 只要加入類似以下要求, 結果就明顯改善:

```text
在寫計畫前, 先從你的開放問題與 outline 開始, 和我來回討論.
```

問題是新使用者不知道必須說這句話. 當企業導入一套工具, 卻要靠工作坊傳授 magic words 才能穩定執行設計問答, 缺陷應歸於工具, 不能責怪使用者不會 prompting.

可重複的工程流程應把必要步驟做成結構, 而不是期待模型從一組龐大指示中穩定遵循所有隱含順序.

## 6. Instruction budget 是有限資源

[07:38](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=458s)

講者提出 instruction budget. 他引用較早研究, 稱當時 frontier models 大約只能對 150 至 200 項指示維持較好的一致性. 超過之後, 模型不一定完全忽略指示, 但遵循率會下降.

一次 coding session 的指示來源不只使用者 prompt:

- System prompt.
- `CLAUDE.md` 或 repository instructions.
- Tool definitions.
- MCP server descriptions.
- Skills.
- 當前任務與對話歷史.
- 85 項 planning workflow instructions.

因此, 即使 85 小於研究所稱的上限, 加上其他來源後仍可能超出模型能穩定處理的範圍. 這些數字會隨模型與測試方法變化, 重點是把 instruction attention 視為有限預算.

## 7. 審查千行 Plan 並沒有產生槓桿

[08:30](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=510s)

原版方法要求工程師仔細閱讀 plan, 有些團隊甚至為 plan 建立 pull request. 但講者觀察到, 一份約 1,000 行的 plan 經常對應約 1,000 行程式碼, 而 implementation 仍會出現 plan 沒有寫到的變化.

結果是工程師先花一小時閱讀計畫, 接著又必須閱讀不同的實作. 這不是 leverage, 因為沒有用更少工作換取更多可靠產出.

新建議是:

- 不要逐行審查長篇 tactical plan.
- 在更短的 design 與 structure artifacts 上做深度對齊.
- Plan 只做 spot-check.
- 把完整審查時間留給實際程式碼.

## 8. 講者撤回「可以不讀程式碼」的主張

[09:03](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=543s)

Dexter 明確承認, 自己在 2025 年 8 月曾主張只讀 plan 就足夠, 但實際嘗試約六個月後, 團隊不得不移除並重建系統的大部分內容.

他區分兩種情境:

- 實驗性 open-source project 可以承受較高變動, 維護者也未必逐行閱讀每個 PR.
- 會影響客戶, 讓工程師凌晨被 paging, 或涉及重大罰款的 production SaaS 與受監管系統, 必須有人理解和負責程式碼.

他對 agent swarms 與完全自動化 software factory 保持保留. 速度提升十倍沒有意義, 如果六個月後必須丟棄全部成果. 相較之下, 維持接近人工品質的兩到三倍提升可能帶來更好的商業結果.

## 9. Context engineering 的另一種讀法

[11:26](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=686s)

Context engineering 常被理解為加入更多相關資訊, 例如建立 RAG pipeline. 講者認為另一個同樣重要的方向是:

- 更好的指示.
- 更簡單的任務.
- 更小的 context window.
- 更少且更相關的 tools.

即使 context window 尚未填滿, 模型品質也可能隨內容增加而下降. Session 同時載入太多 MCP servers 時, 大量工具說明會占用脈絡與 instruction attention, 削弱實際 coding 指示的遵循率.

影片以「dumb zone」稱呼品質開始下降的區域, 但講者在 Q&A 也強調, 具體百分比會依任務複雜度, instructions 與資料比例而變化, 不能當成固定物理界線.

## 10. 不要用 Prompt 模擬 Control Flow

[13:20](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=800s)

許多 LLM workflow 把所有分支寫進同一個 prompt:

```text
如果是 complaint, 執行 A.
如果是 product feedback, 執行 B.
如果是 billing issue, 執行 C.
```

更可靠的方式是先分類輸入, 再由程式把它送往較短, 更專注的 prompt. 每個模型呼叫只需要少量指示與少量 action choices.

同一原理可套用到 RPI. 原版是一個有 85 項指示的 mega prompt, 新版則把工作拆成:

```text
Questions
  -> Research
  -> Design
  -> Structure outline
  -> Plan
  -> Worktree
  -> Implement
  -> Pull request
```

每個 planning phase 的 prompt 都降到 40 項指示以下, 並仍在持續縮減.

講者的總結是:

> 如果能用 control flow 表達控制流程, 就不要用 prompt 表達控制流程.

這是短句形式的影片主張, 本筆記保留其核心措辭並以中文翻譯呈現.

## 11. Design discussion: 在寫程式前對 Agent 做「腦部手術」

[15:08](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=908s)

Design discussion 回答「我們要去哪裡?」, 內容包括:

- Current state.
- Desired end state.
- 應遵循的既有 patterns.
- 已解決的設計決策.
- 尚未回答的開放問題.

模型可能在 codebase 中找到過時或錯誤的 pattern. 例如它準備仿照某處的 SQL update, 但那其實是團隊已不再使用的做法. Design discussion 讓工程師能在寫出大量程式碼前指出正確參考位置.

講者引用 Matt Pocock 的 design concept, 指人與 agent 在 context window 中形成的共享理解. 將它輸出成 Markdown artifact, 就能讓人類直接修改, 追蹤決策並跨 session 保存.

這份 artifact 約 200 行, 比千行 plan 更適合審查. 講者將此過程比喻為在 agent 繼續往下游之前, 先對它做「brain surgery」, 把錯誤理解修正掉.

## 12. Structure outline: 決定如何抵達目標

[16:42](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1002s)

如果 design discussion 類似 architecture review, structure outline 就像 sprint planning. 它不列出每一行實作, 而是說明:

- 變更要分成哪些 phase.
- Phase 的執行順序.
- 每個階段改變哪些高階結構, type 或 signature.
- 何時及如何驗證中間成果.

講者用 C header file 比喻 structure outline. Plan 像完整 implementation, outline 則只揭示介面與型別變化, 足以讓工程師看出 agent 是否走錯方向.

## 13. Horizontal plan 與 Vertical plan

[17:58](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1078s)

模型經常產生 horizontal plan:

```text
完成全部 database changes
  -> 完成全部 service layer
  -> 完成全部 API
  -> 完成全部 frontend
```

問題是直到跨層工作全部完成後, 系統才形成可測試成果. 如果 1,200 行程式碼之後才發現功能失敗, 人類和 agent 都難以定位是哪一層首先出錯.

[18:32](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1112s)

Vertical plan 改以可運作切片推進:

```text
建立最小 API path 與 mock
  -> 讓 frontend 走通第一條路徑
  -> 接上 service behavior
  -> 加入 database migration
  -> 驗證完整切片
  -> 擴展下一個切片
```

總程式碼量可能相同, 但每兩三百行就有 checkpoint. 對敏感或複雜功能, 人類可以在下一階段開始前確認結果. 這讓驗證發生在錯誤仍局部時, 而不是最後一次處理所有問題.

## 14. Plan 重新成為供 Agent 使用的 Tactical Document

[19:20](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1160s)

Design 與 structure 已完成高層對齊後, agent 才根據 ticket, research 與前述 artifacts 建立完整 plan. 這份 plan 的 template 可以維持不變, 但角色改變了:

- 它主要是 agent 的戰術執行文件.
- 人類不再逐行深度審查.
- 工程師確認關鍵方向沒有漂移後, 把注意力保留給實際 code review.

因此, 新方法不是不要 plan, 而是不再把長 plan 當作最終品質保證.

## 15. 團隊在較短文件上提前對齊

[19:50](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1190s)

Design discussion 與 structure outline 不只用於人機對齊, 也適合團隊協作. 講者會把 design discussion 主動交給真正的 code owner, 在尚未產生程式碼前取得回饋.

這能避免一種常見情況: 開發者已花時間讓 feature 運作, 也對方案產生情感依附, code owner 卻在 PR 階段才指出根本架構問題.

AI 對開發時程的最大幫助也不只縮短 coding:

| 階段 | 只用 AI 寫程式 | 同時用 AI 協助結構化對齊 |
| --- | --- | --- |
| 規劃 | 仍需大量人工整理與會議 | 以短 artifacts 加速 architecture review |
| Coding | 從數小時縮短到數十分鐘 | 同樣縮短 |
| Review | 方向問題在 PR 才出現 | Reviewer 已提前 re-steer |
| Rework | 可能需要重做完整方案 | 多數錯誤在 design 或 outline 階段修正 |

影片沒有完整處理 testing 與 verification, 講者明確說那需要另一場演講. 因此不能把這套 planning 流程視為取代測試策略.

## 16. 從 RPI 到更多結構化階段

[21:34](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1294s)

新版流程包含 questions、research、design、structure、plan、worktree、implement 與 PR. 講者以近似「crispy」的讀音稱呼這套新方法. 影片說明與字幕對正式縮寫的呈現不完全一致, 因此本筆記以階段名稱為準, 不強行判定唯一拼法.

增加階段也帶來導入問題. 原本三步已經讓部分團隊覺得複雜, 現在變成七至八個明確步驟. 後續工作包括:

- 如何讓 IDE 或 orchestration tool 自動管理階段.
- 如何衡量對工程團隊的真實影響.
- 如何讓中央平台團隊更新 shared skills, 又不破壞個別團隊工作流.
- 如何合併各團隊累積的 prompts 與 skills.

## 17. Q&A: 到底應該讀多少程式碼

[23:24](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1404s)

被問到人工讀 code 是否無法擴展時, 講者承認這仍是一個正在探索的問題. 他形容業界正在 binary search「人類需要閱讀多少程式碼」.

目前立場是, 閱讀程式碼仍能帶來兩到三倍加速, 而且比追求十倍生成速度後留下大量 slop 更可能產生長期商業成果. 他也預測, 現在完全不讀 code 的團隊可能在數月後遇到重寫問題, 但這仍是個人判斷, 不是已證實的普遍結論.

## 18. Software factory 與形式驗證

[24:40](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1480s)

另一個問題指出, 完全自動化 software factory 可能不讓人類閱讀 plan 或 code, 而改以 evals 與 formal verification 保證品質.

講者承認 TLA+ 與新型形式驗證方法值得探索, 未來可能出現不依賴逐行閱讀的可靠系統. 但對今天必須把程式碼送進正式環境的團隊, 他不再支持「只讀 specification, 把 code 當 assembly」的主張.

這裡存在兩個不同時間尺度:

- 當下實務: 工程師仍要閱讀並擁有 production code.
- 研究方向: 建立足以替代人工閱讀的規格, evals 與形式化證據.

不應把未來研究可能性當成今天已具備的安全保證.

## 19. Q&A: Context Window 百分比不是硬規則

[25:40](https://www.youtube.com/watch?v=YwZR6tc7qYg&t=1540s)

對經驗豐富, 每週大量使用 coding agents 的工程師, dumb zone 的固定百分比不一定有用. 他們會依任務在 30%, 40% 或 60% context usage 間調整.

對初學者, 講者提供的 heuristic 是:

- 儘量把 context 維持在約 40% 以下.
- 接近 60% 時, 考慮結束當前 session 並重新開始.
- 把重要內容持續寫進 static artifacts, 而不是依賴內建 auto-compaction 保存所有理解.

這些數字是教學用經驗法則, 不是所有模型與任務的固定限制. 真正變因包括任務複雜度, instruction 數量, 使用者訊息比例, 載入檔案與工具描述.

## 實務工作流

編者整理, 可將影片方法轉成以下執行方式:

1. 從 ticket 產生客觀 research questions.
2. 在不知道目標方案的 fresh context 中研究 codebase.
3. 建立短 design discussion, 明確列出 current state、desired state、patterns 與 open questions.
4. 由工程師與 code owners 審查 design, 保留技術決策責任.
5. 建立 vertical structure outline, 為每個切片安排驗證點.
6. 讓 agent 產生詳細 tactical plan, 人類只檢查關鍵風險.
7. 在隔離 worktree 中實作.
8. 執行測試與其他獨立 verification.
9. 工程師閱讀最終程式碼並對 production 結果負責.
10. 將研究、設計與結構 artifacts 留在 repository, 支援跨 session 恢復和團隊協作.

## 核心觀念

可靠的 agentic engineering 不是把更多規則塞進一個超長 prompt. 它把認知工作分階段, 限制每個 context 的目的, 以程式控制流程, 並在決策成本最低的位置要求人類介入.

最值得保留的修正有三項:

- 不要依賴使用者知道 magic words.
- 不要把審查長 plan 誤認為品質槓桿.
- 不要因生成速度提升而放棄對 production code 的理解與所有權.

Design 與 structure artifacts 的目的不是增加文件數量, 而是讓錯誤在變成大量程式碼前可見. 如果文件只增加形式負擔, 卻沒有讓人類更早改變決策, 就沒有達成這套方法的目標.

## 來源與限制

- 本筆記依英文原始自動字幕整理, 不是逐字稿. YouTube metadata 沒有提供正式 chapters, 時間點依字幕內容選取.
- 影片中的 50% 產出、rework 比例、150 至 200 項 instructions、context 百分比與兩到三倍速度等數字, 多數是講者引用的舊研究或實務 heuristic. 本筆記未獨立驗證完整研究來源.
- 「slop」「magic words」「dumb zone」「brain surgery」與「crispy」是講者使用的口語或修辭, 不是標準工程術語.
- QRSPI 或近似 `crispy` 的正式拼法無法從自動字幕完全確認. 筆記以 questions、research、design、structure、plan 等實際階段為準.
- 講者是 HumanLayer 成員, 並在結尾介紹團隊正在建立的 IDE. 對 RPI 與新版 workflow 的評價包含產品開發者自身立場.
