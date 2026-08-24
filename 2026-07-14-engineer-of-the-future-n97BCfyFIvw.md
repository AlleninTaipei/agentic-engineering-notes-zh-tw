# 未來工程師: 選擇值得做的事, 並為結果負責

> 來源: ["The engineer of the future is the person who is able to choose what is worth doing." — Addy Osmani](https://www.youtube.com/watch?v=n97BCfyFIvw)
> 頻道: AI Engineer
> 發布日期: 2026-07-14
> 片長: 18:26
> Video ID: `n97BCfyFIvw`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

Addy Osmani 認為, 當 agents 能大量生成、修改與驗證軟體時, 未來工程師的價值不再主要來自 keystrokes 或單一技術能力, 而是選擇值得做的事、要求正確證據、理解風險, 並為 production verdict 負責。

這場演講將人類與 agents 的分工劃成兩層:

```text
Inner loop: capability
  -> Agent 調查、實作、測試、回報

Outer loop: agency
  -> Human 決定、驗證、核准、承擔責任
```

核心問題不再只是「agent 能做什麼」, 因為這份清單持續擴大。更長期的問題是「哪些決策需要某個人、團隊或組織具名負責」。

演講也提醒三項風險:

- Cognitive debt: 系統成長速度超過團隊理解速度。
- Cognitive surrender: 在形成自己判斷之前就接受 AI 的答案。
- Orchestration tax: Agent 數量提高, 但人的注意力和整合能力沒有同步擴張。

## 未來工程師擁有 Evidence、Understanding 與 Verdict

[00:00](https://www.youtube.com/watch?v=n97BCfyFIvw&t=0s)

Addy 將未來工程師定義為能選擇「什麼值得做」的人。隨 agents 接管更多 execution, 人類仍要擁有三項責任:

| 責任 | 內容 |
| --- | --- |
| Evidence | 支持結果的 tests、diffs、logs、traces 與其他證據 |
| Understanding | 對 intent、constraints、trade-offs 和風險的理解 |
| Verdict | 決定 ship、block、redirect 或 accept risk |

Quality 能產生 evidence, 但 verdict 才把責任指派給某個主體。Answerability 表示工程師能說明為何做出決定, 並在結果進入 production 後站在它後面。

這裡的 verdict 不是要求工程師扮演法官, 而是確認誰對 production decision 負責。

## 角色正在圍繞 Work 重新組合

[01:46](https://www.youtube.com/watch?v=n97BCfyFIvw&t=106s)

Addy 引用 Boris Cherny 對工程角色變化的描述。舊有 craft boundaries 逐漸模糊, 工作可能重新分為 prototype、build、sweep、grow 和 maintain 等 modes。

因此更重要的問題從「你的 title 是什麼」轉為:

- 你能擁有系統的哪一部分?
- 目前產品需要哪種 engineering mode?
- 這項工作的 quality bar 是什麼?
- 結果最後由誰負責?

Agents 可以參與所有 modes, 但選擇 mode、設定品質標準和承擔結果仍是稀缺能力。

## 從 Harness 到 Loop Engineering 與 Software Factory

[02:34](https://www.youtube.com/watch?v=n97BCfyFIvw&t=154s)

模型不是完整 agent。Harness 將 model 和 context、tools、file system、Git 等能力組合, 讓 intelligence 變成可委派的系統。

Loop engineering 再把單次 prompt 擴展為持續運作的流程:

```text
Prompt
  -> Act
  -> Check
  -> Remember
  -> Decide next step
  -> Repeat
```

當多個 loops、tools 和 feedback mechanisms 組合起來, 就形成 software factory。Agent 在 inner loop 產生工作和 evidence, 人類則在高槓桿 checkpoint 做 production decision。

風向不是把人類完全移出系統, 而是把 human judgment 移到更高槓桿的位置。

## Answerability 已成為 Engineering Requirement

[03:34](https://www.youtube.com/watch?v=n97BCfyFIvw&t=214s)

AI-assisted code 正從少數實驗變成一般 codebase 的常態。當模型經常修改 production systems, answerability 便不再是抽象倫理詞彙, 而是實際工程需求。

演講援引 Sonar 的研究, 表示 clean 與 messy repositories 的 pass rates 可能接近, 但 clean code 讓 agent 使用較少 tokens、減少 revisits。這意味 maintainability 不只服務下一名人類維護者, 也能提高下一個 agent 的效率。

Clean code 的價值因此擴張:

- 人類比較容易 review。
- Agent 比較容易理解局部變更。
- Context 可放入更高密度資訊。
- 減少重複搜尋與修正。
- Software factory 的每輪成本更低。

## Generation 變便宜, Review 沒有自動變便宜

[04:26](https://www.youtube.com/watch?v=n97BCfyFIvw&t=266s)

Agent 可以快速生成大量 code, 但人類 review capacity 不會同步成長。演講引用的調查顯示, 幾乎所有人都對 AI-generated code 保持一定懷疑, 但只有約一半的人總是在 commit 前驗證。

這形成「distrust without bandwidth」:

```text
不完全信任輸出
  + 沒有足夠時間驗證
  = 容易跳過 review 或只做表面檢查
```

安全不能只依賴工程師「更小心」。Verification 必須變得:

- 更便宜。
- 更清楚。
- 更自動化。
- 更難以跳過。

當 adoption 速度超過 organization governance, review 和 validation 會成為新瓶頸。組織需要回答:

- Model 是否修改了這個 file?
- 當時有哪些 constraints?
- Agent 產生了哪些 evidence?
- 哪些風險被接受?
- 誰擁有最終結果?

## 稀缺資源是 Evidence-backed Judgment

[05:55](https://www.youtube.com/watch?v=n97BCfyFIvw&t=355s)

如果 generation 的擴張速度超過 comprehension, 人類的高槓桿不再是寫更多 code, 而是做有證據支持的判斷。

因此問題應從:

```text
Agent 可以做多少?
```

改成:

```text
Human judgment 在哪裡仍能改變結果?
```

這個 framing 不否認 agent capability, 而是把注意力放在 product value、risk 和 accountability 上。

## Alpha 與 Decay

[06:15](https://www.youtube.com/watch?v=n97BCfyFIvw&t=375s)

Addy 使用兩個詞描述職涯優勢:

- Alpha: 你今天能做到, 但目前 models 還做不到的能力差距。
- Decay: 這個差距隨模型進步而縮小的速度。

如果個人優勢只建立在某項 capability 上, frontier models 最終可能追上。過去的 speed 和 recall 已快速 decay; verification 正逐步移入 harnesses、evals、static checks 和 model critique。

策略不是找到一項永不消失的技能, 而是持續把自己的 edge 往更高層移動。

## Taste 的價值與限制

[07:02](https://www.youtube.com/watch?v=n97BCfyFIvw&t=422s)

當每個人都能產生十種方案時, 判斷哪個方案值得存在會更加重要。Addy 接受 taste 的價值, 但提醒它很容易變成「目前還不想解釋的部分」的神祕標籤。

他引用 Mitchell Hashimoto 的定義:

> Taste 是在尚無客觀 metric 的領域做出高品質 qualitative judgment 的能力。

Taste 位於 benchmark 和市場完整驗證之前。工程師試用模型、檢查 UX 或比較產品方案時, 往往能先感覺出品質差異。

但 taste 必須能轉換成:

- 具體 critique。
- 可比較 examples。
- 明確 preferences。
- 團隊可學習的 decision record。

如果只保留 mystique, taste 無法擴展。它也不是永恆 moat, 因為 models 會從 examples 和 preferences 學習。

## 對各種能力套用 Decay Test

[08:10](https://www.youtube.com/watch?v=n97BCfyFIvw&t=490s)

| 能力 | 變化方向 |
| --- | --- |
| Speed | 已快速自動化 |
| Recall | Harness memory 與 retrieval 持續改善 |
| Verification | Evals、static analysis 和 model critique 逐步承接 |
| Taste | 可能 decay 較慢, 但仍會被 examples 與 preference learning 推進 |
| Judgment | 不是永久牆壁, 而是一條持續移動的 slope |

因此「agent 還不能做什麼」不是穩定的職涯策略。更有韌性的問題是「哪些決定需要 owner 接受後果」。

## 現代 Software Engineer 的更嚴格定義

[08:49](https://www.youtube.com/watch?v=n97BCfyFIvw&t=529s)

更多人能讓電腦完成工作是正面發展, builder 的 total addressable market 因此擴大。但 engineer 不能只定義為「能寫 code 或讓東西存在的人」。

工程師還需要:

- Reason about systems。
- 理解 constraints。
- Defend trade-offs。
- Manage risk。
- 在系統失效時接受追問並修復。

AI 降低 implementation barrier, 不會自動承接這些 responsibilities。

## Cognitive Debt: Code 超過團隊理解

[09:50](https://www.youtube.com/watch?v=n97BCfyFIvw&t=590s)

Cognitive debt 是人類對問題和系統的理解、記憶逐漸流失。對 codebase 而言, 它是:

```text
Repository 中存在的 code
  - 團隊真正理解的 code
  = Cognitive debt
```

Agent 可以產生 passing build、green tests 和可合併 PR, 團隊仍可能失去解釋系統的能力。Addy 也把這稱為 delegation debt 的一部分。

Long-horizon tasks 使風險放大。30 秒 run 像單次 interaction; 一小時或一天的 task 已是獨立 workstream。當多個 agents 平行執行, 人類更容易失去事件順序與 decision context。

Review 不能只是在最後看一眼 diff, 而必須成為完整 control system, 包含中間 evidence、checkpoints、ownership 和 escalation。

## Cognitive Surrender: 借來的自信

[11:02](https://www.youtube.com/watch?v=n97BCfyFIvw&t=662s)

Delegation 與 surrender 不同:

| Delegation | Cognitive surrender |
| --- | --- |
| 要求 AI 完成工作並提供 evidence | 在形成自己意見前接受 AI 答案 |
| 人類仍做 verdict | AI 的答案直接變成人類答案 |
| 可要求修改或拒絕 | 借用模型 confidence |

演講援引 Wharton 的研究指出, AI 提供錯誤答案時, 許多受試者仍選擇錯誤選項, 而且更有信心。真正風險不是使用 AI, 而是 borrowed confidence。

工程師應先建立至少最低限度的 independent view, 再讓 AI 的分析成為 evidence 或 counterargument。

## Orchestration Tax: 注意力無法像 Agents 一樣平行化

[11:51](https://www.youtube.com/watch?v=n97BCfyFIvw&t=711s)

同時執行更多 agents 不等於有更多「你」。Human cognitive bandwidth 不會因 agent 數量增加而自動平行化。

每新增一個 loop, 都增加:

- Decisions 需要 routing。
- Conflicting changes 需要 merge。
- Results 需要 verification。
- Work streams 需要 integration。
- Failures 需要 attribution。

解法不一定是減少 agents, 而是「像設計系統一樣設計注意力」:

- 人類在哪些 checkpoints 進入?
- 每個 agent 必須回傳哪些 evidence?
- 哪些 decisions 可以重用 policy?
- 哪些風險必須 escalation?
- 同時可管理多少 active streams?

## Accountability 是 Scaling Mechanism

[12:39](https://www.youtube.com/watch?v=n97BCfyFIvw&t=759s)

Accountability 不是 agents 變強後剩下的麻煩, 而是讓整個系統能擴張的基礎。

當 agents 更快、更多並能平行工作時, 稀缺能力變成:

- Explain intent。
- Inspect evidence。
- Accept risk。
- 在 verdict 錯誤時改善 system。

沒有 owner 的 automation 很難建立長期 trust。清楚責任讓組織能放大 delegation, 因為每項 production decision 都有可追蹤的政策、證據和決策者。

## Career Math: Capability 的半衰期與 Signature 的半衰期

[13:16](https://www.youtube.com/watch?v=n97BCfyFIvw&t=796s)

技術 edge 的 half-life 可能只有一次 model release。速度、recall、verification 甚至 taste 都會隨 frontier 移動。

相較之下, signature 的 half-life 較長。Signature 是站在 work 後面的 name:

- 個人。
- 團隊。
- 專業角色。
- Institution。

Skills 能取得 leverage, accountability 則把 leverage 轉換為 trust。當同一個 owner 長期做出可解釋、可驗證的好決定, credibility 會形成較持久的職涯資產。

## High Agency 不是親自做完所有事情

[14:13](https://www.youtube.com/watch?v=n97BCfyFIvw&t=853s)

Agent 可以 choose、route、merge、escalate, 並在 policy 內行動, 但 execution 和 responsibility 不相同。Agent 可以遵循 runbook, 卻不能真正繼承失敗後果。

High agency 是主動擁有 outcomes, 包括知道:

- 何時 delegate。
- 何時 inspect。
- 何時 stop。
- 何時接受 risk。
- 何時把自己的 name 放在結果上。

它不是「所有事都自己做」, 也不是 hustle theater。真正的 high agency 是帶有 judgment 的 ownership。

### Agency Ladder

Addy 描述一條從低到高的 agency ladder:

```text
Flag a problem
  -> Execute
  -> Diagnose
  -> Propose
  -> Recommend
  -> Resolve
  -> Discern whether the problem deserves investment
```

最高層不是追逐每一條可能路徑, 而是 discernment。當 agents 讓更多 paths 成為可能, 稀缺能力是判斷哪些 paths 值得 attention 與 ownership。

## 人機邊界不是 Output Review, 而是 Evidence 與 Responsibility

[15:13](https://www.youtube.com/watch?v=n97BCfyFIvw&t=913s)

Agent 適合在 inner execution loop 中:

- Investigate。
- Implement。
- Test。
- Report。

Human outer loop 則負責:

- Decide。
- Verify。
- Approve or redirect。
- Own what reaches production。

Agent 應回傳與工作相符的 evidence:

- Diffs。
- Test results。
- Logs。
- Rationale。
- Traces 與 trajectories。
- Screenshots。

拿到 output 之後才真正進入 engineering judgment。人類要判斷工作是否值得做、證據是否充分, 以及是否願意承擔 production risk。

因此 boundary 不是簡單的「human looks at AI output」, 而是 evidence 和 responsibility 的明確交接。

## Operational Rule: Explain It or Don't Ship It

[16:13](https://www.youtube.com/watch?v=n97BCfyFIvw&t=973s)

「能解釋, 才能 ship」不表示人類必須親自輸入或閱讀每一行 code。它要求至少有 owner 能:

- 說明變更目的。
- 解釋主要實作方式。
- 指出 evidence 與其限制。
- 說明已知 risks 和 blast radius。
- 知道失敗時如何回應。

大型 codebase 常用 owners file 指定 subdirectory 的責任人。AI-generated changes 也應採用相似 ownership model。Model 可以寫 code, 但 architecture 或 service owner 仍需為其進入 production 負責。

## Automation 提高工程工作的層級

[17:20](https://www.youtube.com/watch?v=n97BCfyFIvw&t=1040s)

每次軟體開發變容易, 市場都曾預測工程需求會下降。Higher-level languages、frameworks、cloud 和 low-code 的歷史卻常呈現相反結果: 成本下降後, latent demand 被釋放, 原本不值得建立的系統開始變得可行。

Agents 可能再次產生相同效果。工程瓶頸將從:

```text
Can we build this?
```

移向:

```text
Should this exist?
Can we answer for it?
```

未來工作可能更集中在:

- Loop design。
- Evidence design。
- Brownfield stewardship。
- Risk management。
- Taste 與 product judgment。
- Ownership 和 care。

更少 keystrokes 不代表更少 engineering, 而是更多 surface area 需要驗證與負責。

## 實作框架

### 為 Agent 定義 Evidence Contract

在委派 task 前先決定 agent 必須回傳:

- 修改內容與理由。
- 執行過的 tests 和結果。
- 未通過或未執行的 checks。
- Assumptions 與 unresolved risks。
- 影響範圍和 rollback 方式。

### 控制 Cognitive Debt

- 限制單一 agent workstream 的 scope。
- Long-running task 設置中間 checkpoints。
- 要求 decision log, 不只要求 final diff。
- 讓 system owner 定期解釋 architecture 和 recent changes。
- 發現沒有任何人能說明某區域時, 將其視為 debt 而非正常狀態。

### 避免 Cognitive Surrender

- 在詢問 AI 前先寫下自己的 initial hypothesis。
- 要求 evidence 和 alternatives, 不只要求答案。
- 對高風險 verdict 使用 independent review。
- 將 model confidence 與實際 verification 分開。

### 管理 Orchestration Tax

- 限制同時 active 的 agent streams。
- 為每個 stream 指定 owner。
- 將共同 policy 和 checks 自動化。
- 統一 evidence format, 降低 review switching cost。
- 只在需要 judgment 的 checkpoint 中斷人類。

### Production Verdict Checklist

- 這項工作值得做嗎?
- Intent 和 scope 是否清楚?
- Evidence 是否足以支持結果?
- 風險與 blast radius 是否可接受?
- 誰具名擁有這項決定?
- 失敗後誰負責回應與改善 loop?

## 核心結論

Agent capability 會持續增加, 但 capability 並不等於 accountability。未來工程師的長期價值來自 evidence-backed judgment、system ownership 和 answerability。

真正可擴展的 human-in-the-loop 不是要求人類逐行檢查所有 agent output, 而是建立清楚的 evidence contract 與 responsibility boundary。Agent 運行 factory 的 inner loops, 人類則選擇值得追求的 paths, 做 production verdict, 並在結果出錯時站在系統後面。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕與公開章節整理, 並非逐字稿。內容已移除口語贅詞、重複字幕與現場串場, 以繁體中文重新組織。

演講引用 Sonar 與 Wharton 的調查和研究數據, 本筆記只忠實整理講者的陳述, 未另行核對原始研究方法、樣本或完整限制。因此百分比不應脫離影片語境作為普遍定律。Alpha、decay、taste、cognitive debt 和 signature 等概念是講者用來解釋職涯與責任變化的分析框架, 不是標準化工程規範。
