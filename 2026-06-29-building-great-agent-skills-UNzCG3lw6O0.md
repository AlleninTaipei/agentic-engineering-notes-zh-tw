# 建立優秀 Agent Skills 的實作手冊

來源: [Building Great Agent Skills: The Missing Manual](https://www.youtube.com/watch?v=UNzCG3lw6O0), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=UNzCG3lw6O0
- 上傳日期: 2026-06-29
- 片長: 20:43
- Video ID: `UNzCG3lw6O0`
- 講者: Matt Pocock
- 內容依據: 英文原始語言自動字幕
- 講者提供的參考實作: [writing-great-skills/SKILL.md](https://github.com/mattpocock/skills/blob/main/skills/productivity/writing-great-skills/SKILL.md)

## 摘要

Agent skills 愈來愈容易取得, 但缺少共同品質標準時, 使用者很難判斷哪些 skill 有效, 哪些只是堆積大量指令. Matt Pocock 將這種狀態稱為 "skill hell", 並提出一套四階段 checklist:

```text
Trigger -> Structure -> Steering -> Pruning
```

Trigger 決定 skill 由使用者明確呼叫, 還是由模型依 description 自行選擇. Structure 將內容分成 procedure steps 與 supporting reference, 並用 context pointers 延後載入只屬於特定 branch 的資料. Steering 使用帶有濃縮語意的 leading words 引導模型, 必要時拆分流程並隱藏未來步驟, 讓 agent 對目前階段投入更多 legwork. Pruning 最後移除 duplication, sediment, stale content 與不改變行為的 no-ops.

影片的中心思想是, 好 skill 不是資訊愈多愈好. 它應該在正確時機觸發, 只載入目前需要的 context, 用少量高密度語言改變 agent 行為, 並維持單一事實來源.

## Skill Hell 是什麼

[00:00](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=0s)

講者把 skill hell 類比為 tutorial hell 與 framework hell. 社群提供大量可下載的 skills, 但開發者無法判斷:

- Skill 是否真的會改善 agent 行為.
- 多個 skills 如何共同運作.
- 為何同一 skill 在別人環境有效, 在自己環境卻沒有結果.
- 組織的 SOP 應如何轉成可維護的 skill.
- Skill 變大後, 哪些內容該留, 哪些應刪.

問題不只發生在個人. 組織若沒有共同 rubric, 不同團隊會用不同方式編寫 procedural knowledge, 最後形成難以 audit, reuse 與改進的 Markdown 集合.

## 一套共同的 Skill Checklist

[02:12](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=132s)

影片將 skill review 分成四個階段:

| 階段 | 核心問題 |
| --- | --- |
| Trigger | 誰應在何時呼叫這個 skill? |
| Structure | 程序與參考資料如何安排? |
| Steering | 哪些文字真的能改變 agent 行為? |
| Pruning | 哪些內容可以刪除而不降低效果? |

順序很重要. 如果 trigger 設計錯誤, 再好的內容也不會在正確時機出現. 如果 structure 沒有分支與 progressive disclosure, steering instructions 會被大量無關 context 稀釋. Pruning 則應在 skill 能運作後進行, 避免在尚未理解行為前過早壓縮.

## Trigger, User-Invoked 與 Model-Invoked

[03:16](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=196s)

Skill 有兩種主要觸發方式.

### User-invoked skill

使用者在 prompt 中明確要求 agent 使用某個 skill. 不同 harness 的語法可能是 slash command, skill name 或其他引用方式.

優點:

- 觸發時機由使用者控制.
- 不依賴模型是否正確理解 description.
- 不必把所有 skill descriptions 放入每次 context.
- 移除一類 "模型明明應該用卻沒用" 的不確定性.

代價:

- 使用者必須知道有哪些 skills.
- 使用者要記得何時呼叫哪一個.
- Skill 數量愈多, pilot 的 cognitive load 愈高.

### Model-invoked skill

Skill description 常駐 agent context, 作為指向完整 `SKILL.md` 的 context pointer. 模型判斷目前任務符合 description 時, 自行載入完整內容.

優點:

- 使用者不必記住所有 skills.
- Agent 可以依任務主動選擇能力.
- 對初學者與多樣任務較方便.

代價:

- 每個 description 都增加固定 token cost.
- 每多一個可選 skill, 模型就多一項 routing decision.
- 模型可能漏掉適合的 skill, 或錯誤觸發不相關 skill.
- 團隊需要 eval invocation precision 與 recall.

## Context Load 與 Cognitive Load 的交換

[05:07](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=307s)

講者把兩種成本對照為:

```text
More model-invoked skills
  -> Higher context load
  -> Lower user cognitive load
  -> Higher routing unpredictability

More user-invoked skills
  -> Lower context load
  -> Higher user cognitive load
  -> More explicit control
```

例如, 100 個 model-invoked skills 可能意味每次 request 都帶入 100 個 descriptions. 即使 descriptions 很短, 也會消耗 tokens, 增加選擇干擾與錯誤觸發面積.

反過來, 全部改為 user-invoked 雖可保持 context 精簡, 卻要求使用者熟悉完整 skill catalog. 因此沒有普遍最佳答案.

一種實務分法是:

- 高頻, 明確且應自動發現的能力, 考慮 model-invoked.
- 低頻, 高成本, 高風險或需要使用者意圖的 workflow, 考慮 user-invoked.
- 觸發條件高度重疊的 skills, 優先合併, 改寫 description 或轉為 explicit invocation.

最後三點是根據影片 trade-off 所做的編輯整理.

## Description 是 Context Pointer

[04:13](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=253s)

Model-invoked skill 的 description 並不是完整操作手冊. 它的工作是讓模型判斷:

1. 這個 skill 能處理什麼.
2. 目前任務是否符合.
3. 是否值得載入完整 `SKILL.md`.

因此 description 應清楚描述 trigger conditions 與 scope, 不應把完整 procedure 塞入其中. 如果 description 含糊, 模型可能漏用. 如果過度寬廣, 模型可能在不相關任務中反覆載入.

可用下列問題審查:

- 使用者會用哪些語句提出這類需求?
- 哪些相似需求不應觸發?
- Skill 是否需要特定 inputs 或環境?
- 說明是否與其他 skill description 重疊?
- 自動觸發失敗的成本多高?

## Structure, Steps 與 Reference

[07:29](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=449s)

講者把 skill 的主要內容分為兩種 units:

### Steps

Agent 應依序執行的 procedure. Steps 表達 control flow, checkpoints 與完成條件.

### Reference

支援 steps 的資料, 例如 definitions, templates, examples, schemas 或 domain rules.

Skill 可以只有 steps, 也可以只是 reference knowledge. 但將兩者清楚區分, 能幫助作者判斷每段文字的目的.

```text
Skill
├── Steps
│   ├── Find relevant context
│   ├── Confirm test seams with user
│   └── Write PRD
└── Reference
    ├── Definition of test seam
    └── PRD template
```

影片以 `2PRD` skill 為例. 它從目前 context 建立 product requirements document, 包含尋找相關 context, 與使用者確認 test seams, 再套用 PRD template 輸出文件.

Human-in-the-loop checkpoint 是 procedure 的正式部分, 不是臨時插入的聊天. 這讓 skill 能明確阻止 agent 在測試邊界尚未對齊時直接完成規格.

## 讓主 `SKILL.md` 儘量小

[09:00](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=540s)

主 `SKILL.md` 每次 skill 啟用時都會進入 context. 它愈大:

- 維護與 audit 愈困難.
- 每次 invocation 的 token cost 愈高.
- 核心 instructions 愈容易被無關資料稀釋.
- 重複內容與 stale rules 愈難發現.

但 "小" 不代表刪除必要限制. 真正目標是只在主檔保留所有執行路徑都需要的內容, 將只屬於特定 branch 的 reference 延後載入.

## Branches 與 External References

[10:03](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=603s)

如果 skill 只有一條路徑, 且每次都需要相同 template, 將 reference 放在主檔可能合理. 如果 skill 有多個 branches, 某份 reference 只在一條 branch 使用, 就適合移到獨立檔案.

影片以 domain modeling skill 為例. 它可能更新 local glossary, 也可能建立 architectural decision record, 或兩者都不做. Glossary template 與 ADR template 不應每次全部載入.

```text
SKILL.md
├── Shared procedure
├── Decide branch
├── If updating glossary -> references/context-template.md
└── If creating ADR -> references/adr-template.md
```

主檔使用 context pointer 告訴 agent, 只有進入特定 branch 時才讀取相應 reference. 這就是 progressive disclosure, 先載入導航與共同規則, 再按需取得細節.

判斷 reference 是否應外移, 可以問:

- 每次 invocation 都需要它嗎?
- 只有特定 task type 或 branch 才需要嗎?
- 它是 procedure, 還是支援資料?
- 外移後, pointer 是否足以讓 agent 知道何時讀取?

## Steering, 使用 Leading Words

[11:54](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=714s)

Skill 寫得清楚, agent 卻仍不照做, 可能不是因為文字不夠多, 而是缺乏能喚起模型既有概念的高密度詞彙. 講者稱這些詞為 leading words.

Leading word 或短語在少量 tokens 中承載一整套實務意義. 它進入 skill 後, 模型可能在 reasoning trace 與 plan 中重複使用, 進而讓該概念持續影響行為.

### Vertical Slice 範例

Agent 面對大型功能時, 常按技術層水平實作:

```text
Database -> Schema -> API -> Frontend
```

這會延遲可執行 feedback. 可以寫一長段 instructions 要求先建立小型 end-to-end flow, 也可以使用業界已有明確含義的短語 `vertical slice`.

```text
Thin vertical slice
  -> One small path through database, API and UI
  -> Run it
  -> Get feedback
  -> Expand iteratively
```

如果 agent 在 plan 或 trace 中開始主動說 "先做 thin vertical slice", 就表示它採用了 skill 的概念框架. 接著應檢查實際 actions 是否也符合, 不能只因模型重複關鍵字就認定成功.

## 如何挑選好的 Leading Words

[13:26](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=806s)

好的 leading words 通常具備:

- 在該 domain 中已有相對穩定的含義.
- 能取代一段重複說明.
- 與希望 agent 採用的 strategy 直接相關.
- 容易在 plans, traces 與 outputs 中觀察.
- 不會與其他 instructions 產生歧義.

如果 agent 沒採用預期行為, 可嘗試:

1. 統一術語, 不在 skill 中用多個近義詞描述同一方法.
2. 將 leading word 放在 step 名稱與 success criteria.
3. 在 reference 提供一個正例與反例.
4. 觀察 trace 是否採用概念, 再檢查行為是否改變.
5. 用 eval 比較修改前後, 不只憑感覺判斷.

其中正反例與 eval 是依影片方法延伸的編輯建議.

## 增加每個 Step 的 Legwork

[14:56](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=896s)

Agent 知道後面還有主要 deliverable 時, 可能草率完成前置階段. 影片以 plan mode 為例:

```text
Ask clarifying questions -> Create plan
```

模型的最終目標是 create plan, 因此可能只問少量問題就急著規劃. 講者的解法不是在同一 skill 中反覆強調 "多問一些", 而是拆成兩個 skills:

```text
Skill 1: grill-with-docs
  -> Only investigate and ask clarifying questions

Skill 2: 2PRD
  -> Convert agreed context into a PRD
```

執行第一階段時, agent 看不到第二階段的 goal, 因此更能專注於 discovery. 完成後再由使用者或 workflow 明確進入下一 skill.

講者將這個技巧描述為 "hide the future goal". 它適合需要深入研究, 問答, review 或資料蒐集的階段, 但不表示每個 step 都要拆成獨立 skill. 過度拆分會增加 invocation 與 handoff cost.

## 何時應拆分 Skill

可以考慮拆分的訊號:

- Agent 穩定地跳過或縮短某一階段.
- 一個 phase 有自己的 inputs, outputs 與 completion criteria.
- Phase 之間需要 human approval.
- 不同階段需要不同 tools 或 permissions.
- 中間產物值得保存並獨立 review.

不一定要拆分的情況:

- Steps 很短且有強依賴.
- Handoff 會失去重要 context.
- 額外 invocation 比省下的 tokens 更多.
- 單一 eval 顯示 agent 已能穩定依序完成.

後一組判斷是根據影片 trade-off 所做的編輯整理.

## Pruning, 移除 Skill 中的雜質

[16:48](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=1008s)

Massive skill 通常不是優點, 而是其他設計問題的症狀. 講者列出三類常見膨脹來源.

### Duplication

同一 definition, template 或 rule 出現在多個段落和 reference files. Skill 應讓每項資訊有 single source of truth, 其他位置只用 pointer 引用.

### Sediment

多人長期向 shared Markdown 追加內容, 卻沒有人敢刪除或重構舊內容. 文件逐漸累積 irrelevant, misplaced 與 stale material, 像沉積物一樣愈來愈厚.

處理 sediment 時應先重新檢查 structure:

- 內容是否對所有 branches 都必要?
- 是否應移到特定 branch 的 reference?
- 是否已經 stale?
- 是否與其他規則矛盾?
- 是否可以直接刪除?

### No-Ops

看起來像 instruction, 實際刪除後卻不改變 agent 行為的文字. Agent 自動產生 skill 時尤其容易加入這類泛泛規則.

例如, implementation skill 用一整段要求 agent 撰寫詳細 commit message, 但即使刪除, agent 仍會產生相同品質的 message. 這段文字就可能是 no-op.

## Deletion Test

[18:10](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=1090s)

講者讓 skills 維持精簡的重要方法是 deletion test:

```text
刪除一段 instruction
  -> 用代表性任務重新執行
  -> 比較 behavior 與 output
  -> 若無可觀察差異, 保持刪除
```

更嚴謹的實作應使用固定 eval set, 多次執行並比較 invocation, tool use, success rate, token cost 與 quality. 單次沒有差異, 不足以證明 instruction 永遠沒有作用. 這是根據講者方法所做的編輯補充.

## 完整 Skill Review Checklist

[19:06](https://www.youtube.com/watch?v=UNzCG3lw6O0&t=1146s)

### Trigger

- Skill 是 user-invoked 還是 model-invoked?
- 自動觸發真的值得增加 context load 嗎?
- User invocation 是否造成過高 cognitive load?
- Description 是否清楚區分 in-scope 與 out-of-scope tasks?
- 是否測試漏觸發與誤觸發?

### Structure

- Skill 是否清楚分成 steps 與 reference?
- Main `SKILL.md` 是否只包含共同且必要的內容?
- 有哪些 branches?
- Branch-specific reference 是否藏在 context pointers 後?
- Human checkpoints, inputs 與 outputs 是否明確?
- 每項 reference 是否有 single source of truth?

### Steering

- Skill 想改變的具體 agent behavior 是什麼?
- 是否有能濃縮該策略的 leading words?
- Agent 是否在 reasoning / plan 中採用相同概念?
- 實際 actions 是否真的跟著改變?
- 哪個 step 的 legwork 不足?
- 是否需要拆分 skill 並暫時隱藏 future steps?

### Pruning

- 哪些內容重複?
- 哪些是歷史 sediment 或 stale rule?
- 哪些 reference 放錯 branch?
- 哪些 instructions 是 no-ops?
- 哪些段落可通過 deletion test?
- 精簡後是否仍保留 safety, permissions 與 completion criteria?

## 建議的 Skill 檔案結構

以下是依影片原則整理的概念範例, 不是講者 repository 的逐字重建:

```text
my-skill/
├── SKILL.md
├── references/
│   ├── branch-a-template.md
│   └── branch-b-rules.md
├── scripts/
│   └── deterministic-check.ps1
└── evals/
    ├── should-trigger.md
    ├── should-not-trigger.md
    └── behavior-cases.md
```

主 `SKILL.md` 可只保留:

```markdown
---
name: my-skill
description: 說明能力, trigger 與邊界
---

# Goal

一句話描述完成狀態.

# Steps

1. 確認必要輸入.
2. 選擇 branch.
3. 只讀取該 branch 的 reference.
4. 執行並驗證.

# Completion

列出必要 evidence 與停止條件.
```

Frontmatter 欄位與 invocation controls 會因 harness 而異, 實際格式應以使用平台的最新規格為準.

## 核心觀念回顧

```text
Great skill
├── Trigger
│   ├── Context load
│   ├── User cognitive load
│   └── Invocation predictability
├── Structure
│   ├── Steps
│   ├── Reference
│   ├── Branches
│   └── Context pointers
├── Steering
│   ├── Leading words
│   └── Legwork per phase
└── Pruning
    ├── Single source of truth
    ├── Remove sediment
    ├── Remove stale content
    └── Delete no-ops
```

最值得帶走的三個原則:

1. User-invoked 與 model-invoked 是 cognitive load 和 context load 的交換, 不是單純的功能多寡.
2. Main `SKILL.md` 應保持精簡, branch-specific material 用 context pointers 延後載入.
3. Skill 品質要看是否改變行為. 使用 leading words, traces, evals 與 deletion tests 驗證, 不以篇幅判斷.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕多次誤辨 Matt Pocock 姓名, repository 名稱及部分 skill 名稱, 本筆記只在影片說明, 畫面語境或公開連結足以確認時修正.

影片以講者自己的 skills 與 Superpowers 作為設計風格比較, 不是完整 benchmark. User-invoked 與 model-invoked 的實際行為, metadata 欄位, context loading 與 reasoning trace 可見度取決於 agent harness. 使用前應查閱目標平台規格, 並以自己的 invocation 與 behavior evals 驗證.
