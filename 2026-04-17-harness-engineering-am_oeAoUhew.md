# Harness Engineering: 當人類掌舵、Agents 執行時, 如何建造軟體

> 來源: [Harness Engineering: How to Build Software When Humans Steer, Agents Execute — Ryan Lopopolo, OpenAI](https://www.youtube.com/watch?v=am_oeAoUhew)  
> 頻道: AI Engineer  
> 講者: Ryan Lopopolo, OpenAI Member of Technical Staff  
> 發布日期: 2026-04-17  
> 影片長度: 46 分 20 秒  
> Video ID: `am_oeAoUhew`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 核心摘要

Ryan Lopopolo 將 `Harness Engineering` 描述為一種新的工程工作: 人類不再親自完成每一行實作, 而是建造一套能讓 coding agents 長時間、可靠且可平行工作的環境.

這個 harness 不只是 agent 外層的 UI 或 orchestrator. 它包括:

- 清楚定義的 tickets 與 acceptance criteria.
- 對 agents 可讀的 repository 結構與文件.
- Skills、開發工具與 local observability.
- Tests、lints、架構規則與 reviewer agents.
- CI、PR 和 GitHub 上的人機協作流程.
- 能把過去錯誤轉成永久 guardrails 的團隊習慣.

講者的核心假設是 `code is free`, 更精確地說, 程式碼的產生與重寫不再是主要稀缺資源. 真正稀缺的是 human time、human/model attention 與 model context. 因此工程師的工作重心移向 systems thinking、delegation、驗收與約束設計.

## 1. Implementation 不再是主要瓶頸

[02:11](https://www.youtube.com/watch?v=am_oeAoUhew&t=131s)

講者表示, coding agents 的能力和可靠度已提升, 能處理更複雜、時間跨度更長的工作. 在他的團隊實驗中, 實作不再是軟體工程最稀缺的能力.

他用「每位工程師都像 staff engineer」描述新的角色: 每個人都可以同時調度許多 agents, 因此更應該思考一天、一週或數月後需要哪些結構, 讓這些執行能力能真正轉化成產品進展.

### 「Code is free」的限定

這不是說程式碼沒有維護成本, 而是講者認為 agents 也能負責產生、重構、維護與刪除程式碼. 過去必須以人力排序的 P3 工作, 現在可能同時啟動多個方案, 再選擇能解決問題的結果.

影片舉例, 內部工具可以從一開始就補齊 localization 與 internationalization, 因為這些功能不再必須與其他人工開發容量直接競爭.

這是講者基於特定 OpenAI 團隊、模型能力和大量 token budget 得出的工程假設. 不代表所有組織目前都能忽略 code ownership、審計、資安或維護責任.

## 2. 新的稀缺資源

[05:12](https://www.youtube.com/watch?v=am_oeAoUhew&t=312s)

講者列出三項稀缺資源:

| 資源 | 為何稀缺 | Harness 的應對方式 |
| --- | --- | --- |
| Human time | 人無法同步監督無限工作 | 把重複判斷自動化, 將人移到高槓桿工作 |
| Human/model attention | 注意力會被例外與不一致消耗 | 統一做法、命名、架構與回饋格式 |
| Model context window | 無法一次載入所有知識 | Progressive disclosure、locality 與持續刷新必要 context |

重要的不是最終 code 本身, 而是產生可接受 code 的 prompts、guardrails 與歷史脈絡. 因此 repository 應保存能讓 agents 理解「好工作長什麼樣子」的 breadcrumbs:

- Documentation.
- Architecture Decision Records.
- Persona-oriented guidelines.
- Tickets 與 code review 歷史.
- QA、可靠度與安全標準.

## 3. 讓系統對 Agents 可理解

Agent-friendly codebase 應讓完成任務所需的 tokens 容易預測. 講者特別強調 `make things the same`:

- 同類問題只保留一種標準解法.
- 共同 utilities 有單一 canonical implementation.
- 相似 packages 使用一致結構.
- 減少長期未完成的 migrations.
- 透過 agents 大規模完成統一與重構.

一致性可以降低模型每到一個新目錄都必須重新推理的成本. 大型 migration 過去可能因人力不足拖延數月, 現在可以分配多個 agents 並行完成, 但前提是目標狀態和驗證方式足夠清楚.

## 4. 把 Non-functional Requirements 寫下來

一個看似簡單的 patch 可能包含數百個沒有明說的小決策, 例如:

- Reliability 與 failure handling.
- Security boundaries.
- API 是否容易被誤用.
- Timeout、retry 與 observability.
- Naming、module boundaries 與 maintainability.
- QA plan 與應附上的驗證證據.

模型在訓練中見過品質差異極大的程式碼, 不會自動知道某個團隊接受哪一組選擇. 團隊必須將自己的 non-functional requirements 寫成 agents 可取得的規則.

如果 agent 反覆產生不可接受的結果, 人的工作不是永遠在 PR 上重複糾正, 而是找出失敗類別, 加入能系統性限制輸出的 guardrail.

## 5. 將個人專長轉成團隊槓桿

一個 cross-functional 團隊可能包含 front-end architecture、back-end scalability、product thinking 或 reliability 等不同專長. 傳統上, 這些知識要等到 code review 才傳遞, 而 reviewer 也可能成為同步瓶頸.

Harness Engineering 的做法是讓各領域專家把判斷標準寫下來. 一位熟悉產品 QA 的工程師只需定義一次好的 QA plan 應包含什麼, 此後每條 agent trajectory 都能使用相同標準.

這讓「團隊裡最擅長某件事的人」不必逐項參與每個 patch, 仍能把其經驗施加到所有實作上.

## 6. 多層 Prompt Injection

[11:07](https://www.youtube.com/watch?v=am_oeAoUhew&t=667s)

講者用較廣義的方式理解 prompts. 除了聊天輸入外, 下列機制都能在正確時刻把指令送回 agent:

- `AGENTS.md` 或其他 rules files.
- Skills.
- Lint error messages.
- Test failures.
- Reviewer-agent comments.
- PR requirements.
- 內嵌 agent SDK 的 source-code tests.

由於 context 會被 compact 或逐漸換出, 只在對話開頭說一次規則並不可靠. Harness 應在 agent 執行過程中, 於最相關的時間點重新注入必要 context.

## 7. Reviewer Agents 與 CI Guardrails

[12:04](https://www.youtube.com/watch?v=am_oeAoUhew&t=724s)

講者的 codebase 會在每次 push 後執行 security 和 reliability reviewer agents. 它們讀取團隊文件, 再用特定 persona 檢查 patch, 例如:

- Network code 是否包含 timeout 與 retry.
- 新介面是否安全且難以誤用.
- 變更是否符合 reliability expectations.
- 是否有足以證明功能完成的 QA plan.

這些 reviewer agents 並不是取代所有 tests, 而是把過去由人類 reviewer 重複提出的高層判斷自動化.

### 從事故修補到永久規則

傳統流程可能是在 production outage 後補上一個 timeout, 合併後便結束. 講者主張應再往前一步:

1. 記錄這類錯誤為 durable failure class.
2. 寫出文件說明正確模式.
3. 建立 codebase-specific lint 或 test.
4. 讓所有既有程式碼完成 migration.
5. 後續每個 patch 自動取得回饋.

## 8. Source-code Tests 與可修復的錯誤訊息

[13:56](https://www.youtube.com/watch?v=am_oeAoUhew&t=836s)

除了測試 runtime behavior, 也可以測試 source code 的結構. 影片中的例子包括限制檔案不超過約 350 行, 用以配合有限 context 並提高模型處理效率.

其他結構性 guardrails 可檢查:

- Package privacy.
- Stack layers 之間允許的 dependency edges.
- 跨檔案重複的 schemas.
- 是否使用 canonical async helpers.
- 是否違反統一的 parsing 或 validation 策略.

錯誤訊息不應只說「lint failed」. 它應解釋:

- 違反了哪個設計原則.
- 為什麼這個做法不被接受.
- 正確替代方式在哪裡.
- Agent 下一步應如何修復.

此時 lint/test failure 便成為一個具體、即時且可行動的 prompt.

## 9. QA Plan 降低人類同步監督

要寫出良好 QA plan, 團隊必須先記錄:

- 系統有哪些功能.
- 哪些是 critical user journeys.
- 使用者如何與 web app、API 或 services 互動.
- 每類變更需要什麼驗證媒體或證據.

當所有 user-facing work 都必須提供 QA plan, reviewer agent 才能判斷實作是否充分驗證. PR 中附上的 screenshots、videos、logs 或其他證據, 也讓人類更信任輸出, 減少逐步盯著 agent 工作的需要.

## 10. 演講結論: 讓 Agents 做完整工作

講者主張不要只讓 agent 補完局部程式碼. 應提供足夠的 tools、tokens、context 與 guardrails, 讓它完成從理解任務到測試、review 和交付的完整工作.

這並不等於沒有控制. 控制從逐行人工操作, 轉移到:

- 任務定義.
- Acceptance criteria.
- Repository architecture.
- 自動化 feedback loops.
- Exceptions 與 escalation policy.
- 最終產品指標.

## 11. 實際工作流

[20:32](https://www.youtube.com/watch?v=am_oeAoUhew&t=1232s)

問答中, Ryan 描述其團隊的典型流程:

1. 從 ticket 開始, 定義 feature 或 reliability work.
2. 把 ticket 與必要 skills 交給 agent.
3. 讓 Codex 成為開發流程的 entry point.
4. Skill 教 Codex 啟動 app 和 local observability stack.
5. Skill 或 local CLI 讓 agent 啟動 Chrome DevTools 並連接應用.
6. Repository 內的 mini harnesses 提供 ESLint、結構 tests 與其他 guardrails.
7. Agent 實作、測試、review 並建立 PR.

團隊不是先建一個把 Codex 包住的巨大外殼, 而是讓 Codex 從 repository 內按需呼叫工具. 這使每個工具都能獨立加入 guardrails, 並符合 progressive disclosure.

## 12. 為什麼偏好 First-party Harnesses

Ryan 認為, 模型供應商會在自家 harness 的實際工具語境中進行 post-training, 包括 patch 工具、shell invocation 與 quoting semantics. 因此直接使用 first-party harness, 再透過 SDK 或 app server 擴充, 可以承接這些訓練帶來的槓桿.

他的團隊不想投入大量時間重新實作 coding harness, 而是專注於定義「正確 code 長什麼樣子」以及不同模型版本的實際行為差異.

## 13. 人機協作中心: Markdown、Repository 與 GitHub

[28:41](https://www.youtube.com/watch?v=am_oeAoUhew&t=1721s)

團隊主要使用 repository 中的 Markdown 和 GitHub 作為 collaboration hub. PR 類似一個乾淨的協作空間, 人與 agents 都能提出 feedback.

為了 throughput, 並非每一條 feedback 都必須被接受. Implementation agent 可以 acknowledge、defer 或 reject 建議. 如果規定所有 reviewer comments 都必須實作, 多個 reviewer agents 可能共同把 patch 推向錯誤方向.

因此目標不是追求沒有任何意見的完美 PR, 而是偏向能安全接受並推進產品的 code, 同時只把重要問題設為 blocking.

## 14. 初學者如何開始

[30:08](https://www.youtube.com/watch?v=am_oeAoUhew&t=1808s)

Ryan 提供兩條入門路徑:

### 先提升對現有程式碼的信心

讓 agents 閱讀現有 code 與使用情境, 補上能驗證行為的 tests. 測試增加後, humans 與 agents 都能更安全地修改系統, 人也不必逐行審查所有輸出.

### 找出自己時間花在哪裡

觀察是否把時間耗在:

- 親自輸入程式碼.
- 等待 tests 或 CI.
- 等待 human review.
- 處理 flaky tests.
- 重複執行相同 release 或 QA 步驟.

逐步自動化最大的同步瓶頸, 將自己移向定義、排序、排程與授權工作.

## 15. Progressive Disclosure 與大規模 Repository

[33:54](https://www.youtube.com/watch?v=am_oeAoUhew&t=2034s)

Ryan 描述其專案從單一 Electron app 成長後, 因缺乏 package privacy 與 domain boundaries 形成混亂. 團隊後來採取較重的架構, 將 pnpm workspace 切成約 750 個 packages, 按 business domain 或 stack layer 隔離.

重點不一定是所有團隊都要使用 750 packages, 而是讓大多數變更能侷限在 repository 的一個 subtree. Agent 只需載入與任務相關的局部結構, 再透過一致模式把經驗轉移到其他區域.

講者建議盡量統一:

- Bounded-concurrency helper.
- Observable side-effectful command 的建立方式.
- ORM 與 programming language.
- CI scripts.
- 新增 lint rules 的方式.

程式碼本身也是 agent 讀到的 text, 因此一致的 code structure 也是一種 context engineering.

## 16. Code Review 與 Garbage Collection Day

[37:09](https://www.youtube.com/watch?v=am_oeAoUhew&t=2229s)

團隊早期每位工程師每天產生約 3 到 5 個 PR. 即使團隊只有三人, 大型 PR 與相同區域的修改仍造成 merge conflicts. 他們採取兩項措施:

- 將 code tree 分得更清楚, 降低互相碰撞.
- 縮短 PR 開啟時間, 減少衝突機率.

Human code review 是 PR 等待的主要原因. 團隊因此設立每週一次的 `garbage collection day`:

[37:57](https://www.youtube.com/watch?v=am_oeAoUhew&t=2277s)

1. 收集這一週讓 PR 難以合併的 slop.
2. 將 review comments 分成 front-end architecture、reliability、scalability 等 personas.
3. 找出 agent 缺少的 context.
4. 把判斷寫入共用文件.
5. 建立 tests、lints 或 persona reviewer agents.
6. 讓 agents 下次能自動發現並修正同類問題.

此流程把一次性的同步 review 轉化為永久、可重複的 repository leverage.

## 17. Token 用量與 Planning

主持人描述 Ryan 的用量超過每日十億 output tokens. Ryan 粗略估計其 tokens 分布在三類活動:

- Planning、ticket curation 與 documentation.
- Implementation.
- CI 中執行的 review 與驗證.

這是極端資源條件下的個人案例, 不是一般團隊的建議預算.

Ryan 很少使用互動式 Plan mode. 如果 ticket 與 harness 已經足夠好, 他希望 agent 直接完成工作. 若團隊要使用正式 plan, 他的建議是將 plan 單獨建立 PR, 讓人逐行 review 後再核准. 未閱讀就批准 plan, 等於把未驗證的錯誤指令固定進 rollout.

## 18. Code 是否是 Disposable Build Artifact

[41:51](https://www.youtube.com/watch?v=am_oeAoUhew&t=2511s)

Ryan 的回答是肯定的. 他提出 `LLM as a fuzzy compiler` 的 mental model:

- Specification、repository docs 與 guardrails 定義可接受的程式碼空間.
- Model 像 code-generation backend.
- 換模型可能產生不同實作.
- 只要所有 constraints 都成立, 不同 code 仍可視為有效輸出.

他把 harness 中的規則類比為 compiler 的 static analysis 與 optimization passes. 這種觀點依賴非常強的 specification、tests 與 acceptance criteria. 若這些不足, 把 code 當成 disposable artifact 會失去可驗證基礎.

## 19. 想建造的未來

[43:37](https://www.youtube.com/watch?v=am_oeAoUhew&t=2617s)

Ryan 想要的系統是: 人提供 token budget、數季的工作範圍、優先順序、success metrics 與 reliability metrics, agents 則持續推進產品, 不需要人手一直放在方向盤上.

這個未來不只包含寫程式碼. Agents 還要能:

- QA built artifacts 與 critical user journeys.
- Triage user feedback 和 production pages.
- 檢查 logs 是否洩漏 PII.
- 觀察使用者對產品的反應.
- 維護 user-operations runbooks.
- 把反覆發生的問題修進 codebase, 使其不再發生.

對工程師而言, 工作逐漸成為一種 meta-programming: 把流程、標準和 acceptance criteria 寫下來, 讓 agents 能執行軟體工程的完整生命週期.

## Harness Engineering 迴圈

```text
觀察 Human / Agent 的重複失敗
  -> 分類失敗與缺少的 Context
  -> 寫成 Documentation / ADR / Persona rule
  -> 建立 Test / Lint / Reviewer Agent
  -> 在正確時刻把規則注入 Agent
  -> Agent 自動修復並交付 PR
  -> 觀察新失敗, 繼續強化 Harness
```

## 可立即採用的做法

- 先讓 agents 補足現有系統的行為測試.
- 記錄自己在 code review 中反覆提出的三種意見.
- 將其中一種寫成可執行 lint、test 或 reviewer checklist.
- 改善錯誤訊息, 加入原因與修復方式.
- 讓每個 ticket 有明確 acceptance criteria 和 QA evidence.
- 將大 repository 按 domain 建立清楚 locality.
- 優先統一重複做法, 減少 agents 需要猜測的選項.
- 只將高信號、重大問題設為 blocking review.
- 定期安排時間把新出現的 slop 轉成永久 guardrails.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 影片沒有公開章節. 本文的段落與時間標記依 keynote 與問答的內容轉折編排.
- 自動字幕對講者姓氏、模型名稱與少數技術詞存在誤辨風險. 本文以影片標題、上下文和可可靠確認的拼法為準, 未可靠確認的細節不加以擴寫.
- 「Code is free」、模型已能完成完整工程工作、code 可視為 disposable artifact, 都是講者在特定團隊與資源條件下的主張. 本文保留其限定, 不將其視為所有專案都成立的客觀規則.
- 每日十億 output tokens、每位工程師每日 3 到 5 個 PR、約 750 個 workspace packages 等數據來自講者或主持人的現場陳述, 本文未以外部資料獨立驗證.
- 中文標題、表格、「Harness Engineering 迴圈」與實務清單是編輯整理. 本文不是逐字稿, 也不取代完整演講與問答.
