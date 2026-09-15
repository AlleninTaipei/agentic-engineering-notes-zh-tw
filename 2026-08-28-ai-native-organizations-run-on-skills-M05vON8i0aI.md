# AI-Native 組織如何用 Skills 運作: 從個人工具到治理平台

- 影片: [AI-Native Organisations Run on Skills: How to Structure and Scale Them - Imad Touil, QuantumBlack](https://www.youtube.com/watch?v=M05vON8i0aI)
- 頻道: AI Engineer
- 講者: Imad Touil, QuantumBlack Distinguished Engineer
- 發布日期: 2026-08-28
- 片長: 20:30
- Video ID: `M05vON8i0aI`
- 內容依據: YouTube 英文原語自動字幕 (`en`)

## 摘要

這場演講討論的不是如何寫一個 skill, 而是當組織內每個團隊都開始建立 skills 後, 如何避免重複, 品質衰退, 無人維護, 權限失控和供應鏈風險.

Imad Touil 的核心主張是, skills 會逐漸成為組織 know-how 的可執行單位. 它們能把領域規則, 工程慣例和操作程序放入 agent runtime, 並透過 progressive disclosure 在需要時載入. 但如果每個團隊各自保存一份, skills 很快會形成新的 technical debt.

他的治理方案分成三層. 個人可以建立與測試 skill, 團隊共同改善和分享, 組織則以中央平台管理 catalog, metadata, dependencies, versions, access control, evaluation, observability 和 named ownership. 同一套方法最後還要擴展到完整 workflows, 不能只管理其中的 skills.

影片提供清楚的 platform blueprint 和治理問題清單, 但沒有 production rollout 的實際指標. 其中 15 個團隊, 6 個月的比較是模擬情境, 只能用來解釋機制, 不能證明治理平台必然改善生產力, 成本或安全.

## Agentic Software Stack 有內外兩層

[01:33](https://www.youtube.com/watch?v=M05vON8i0aI&t=93s)

講者把 agentic software stack 分為 inner loop 和 outer loop.

Inner loop 是 coding agent harness 的 runtime, 主要包含:

- Context manager.
- Tools 和 MCP integrations.
- Memory 與 state.
- Skills loader.

Outer loop 是組織為完成工作而設計的 workflows, 其中可能組合:

- Skills.
- Subagents.
- MCP servers.
- Event hooks.

兩層之下還需要 enablement infrastructure, 例如 sandbox, MCP gateway, model gateway, knowledge graph, skills registry 和 workflow marketplace. Context layer 則把 project instructions, tool schemas, conversation history, human input 和 retrieved content 組成每次執行需要的 context.

這是一張概念架構圖, 不是 QuantumBlack 已部署系統的完整技術規格. Knowledge graph 和 workflow marketplace 也是講者提出的 stack components, 不表示每個組織都必須採用.

## Coding Workflow 只是產品生命週期的一段

[02:56](https://www.youtube.com/watch?v=M05vON8i0aI&t=176s)

常見的 agentic coding flow 是:

```text
specify -> plan -> tasks -> implement
```

講者提醒, 這只描述一個 product increment. 組織真正的 end-to-end lifecycle 還包括:

```text
product strategy and success metrics
  -> market research and customer interviews
  -> problem and solution discovery
  -> experiments and user stories
  -> data preparation and integration
  -> product implementation
  -> infrastructure provisioning
  -> launch
  -> monitoring, incident response and optimization
  -> next iteration
```

同一家公司通常還有多種 lifecycle. Mobile product, internal platform, customer-facing service 和 data product 的流程, 風險與控制點不同. 因此, 組織不應期待一個 universal workflow 處理所有工作.

講者根據 18 年服務不同組織的經驗指出, 圖中的 lifecycle 仍可能只涵蓋實際複雜度的一小部分. 這是經驗性估計, 不是經過量測的固定比例.

## 為什麼 Skills 是重要的 Know-How 載體

[07:03](https://www.youtube.com/watch?v=M05vON8i0aI&t=423s)

在講者的分類中, hooks 只在特定事件觸發動作, MCP servers 提供工具介面, subagents 常被用來隔離工作和保護主要 context. 真正描述「這項工作應如何完成」的規則, 程序和領域判斷, 往往集中在 skills.

因此, workflow 可以理解為塑造 agent runtime 行為的 harness blueprint, skills 則提供其中可重用的執行知識.

這個區分很有用, 但不是嚴格邊界. Know-how 也可能存在 code, tests, data schemas, policies, tool implementations 和人員經驗中. Subagent 也不只負責縮短 context. 閱讀這項主張時, 應把它視為強調 skills 治理重要性的架構觀點.

## 用 Microservices 原則設計 Skills

[08:27](https://www.youtube.com/watch?v=M05vON8i0aI&t=507s)

講者把 skill ecosystem 類比為 microservices ecosystem. 他提出一組設計性質:

| 性質 | 對 skill 的意義 |
| --- | --- |
| Reusable | 不應只服務一次性 prompt |
| Modular | 將單一責任和依賴邊界說清楚 |
| Discoverable | 使用者與 agent 能找到合適的 skill |
| Portable | 可跨 workflow 與相容的 agent harness 使用 |
| Specialized | 避免把所有知識塞進單一 monolithic skill |
| Composable | 能與其他 skills 串接, 不重複也不互相衝突 |
| Consistent | 對相同條件產生穩定且可檢查的行為 |
| Cost-efficient | 只在需要時載入必要內容 |

Microservices analogy 提供治理語言, 但不能直接等同. Skill 通常是 instructions, scripts 和 supporting resources 的組合, 不一定有獨立 runtime 或 network contract. 是否需要與 microservice 相同程度的 versioning 和 ownership, 仍取決於風險與使用規模.

## Progressive Disclosure 控制 Context 成本

[09:48](https://www.youtube.com/watch?v=M05vON8i0aI&t=588s)

Skill loader 可以先暴露少量名稱與描述, 只有在 task 需要時才載入完整 instructions 和 resources. 講者把這稱為 progressive disclosure, 目的包括:

- 不把所有組織規則預先塞入 system prompt.
- 降低每次 request 的 token usage.
- 減少不相關 instructions 之間的干擾.
- 讓 agent 依任務選取專門知識.

成本是否真的下降, 不能只看 prompt tokens. Registry search, skill selection errors, tool calls, retries 和 evaluation 也會產生成本. 實務上應比較整個 task 的成功成本, 而不是單次 context 長度.

## 將法規 Skills 組成可稽核 Workflow

[11:09](https://www.youtube.com/watch?v=M05vON8i0aI&t=669s)

講者以 regulatory disclosure review 為例. 一個 workflow 可能組合:

- Data retention policy.
- Disclosure standards.
- GDPR rules.
- Filing templates.

Agent 在處理 web, mobile 或其他產品功能時載入相關 skills, 執行 review, 產生 audit report, 列出需要修改的項目, 再把結果回饋到 codebase.

這個例子展示 composability 的目的: 不為每個 workflow 複製完整法規知識, 而是讓不同 workflows 重用具有清楚責任邊界的 domain skills.

然而, 影片沒有展示實際法規 verifier, false-positive rate 或 legal approval flow. 可重現的合規系統還需要版本化政策來源, evidence links, deterministic checks, human sign-off 和不可竄改的 audit records. Skill 輸出本身不能等同法律合規證明.

## Ungoverned Skills 會形成新的 Technical Debt

[12:30](https://www.youtube.com/watch?v=M05vON8i0aI&t=750s)

講者列出六類風險:

1. Duplication: 多個團隊為相同 stack 重複建立相似 skill.
2. Quality decay: Model, tool 和 task 改變後, 原本有效的 skill 不再可靠.
3. Discoverability: 沒有 catalog, 其他團隊不知道資產存在.
4. Ownership: 沒有具名 owner, 問題出現時無人維護.
5. Composability: Skills 各自設計, 串接後可能重複或衝突.
6. Security and permissions: Skill 可能包含 scripts, prompt injection 或敏感 business logic.

下載公開 skill 是一項 supply-chain decision. 審查不能只閱讀 Markdown prompt, 還要檢查 scripts, dependencies, network access, filesystem permissions 和 installation behavior.

Access control 也不應只控制誰能下載 skill. 平台需要區分 discover, read, invoke, modify, approve 和 publish 等權限, 並記錄哪個版本在何時被哪個 workflow 使用.

## 三層採用模型

[13:52](https://www.youtube.com/watch?v=M05vON8i0aI&t=832s)

講者建議循序建立能力:

### Individual

允許工程師建立, 測試, 改善和使用 skills, 但先約定結構, 工具和基本 quality checks, 避免每個人建立不相容格式.

### Team

把個人 skill 分享給處理相同產品和技術棧的團隊, 共同修正內容, 測試和適用邊界. Team layer 是快速取得 feedback 的地方.

### Organization

中央平台讓所有團隊 discover, pull, execute, improve 和 republish 經治理的 skills. 平台不等於由中央團隊撰寫所有內容, 而是提供共同 distribution 和 control plane.

這個模式需要處理 federation. 過度中央化可能讓發布流程太慢, 完全分散則會回到重複和失控. 較合理的做法是由 domain owners 管理內容, 中央平台提供共同 metadata, policy 和 enforcement.

## Central Skills Platform 的必要能力

[13:52](https://www.youtube.com/watch?v=M05vON8i0aI&t=832s)

講者提出的 central platform 包含:

- Catalog 和 searchable metadata.
- CLI 與 MCP interface, 供 IDE 或 sandbox 搜尋和拉取 skill.
- Dependency graph.
- Versioning 與 lifecycle state.
- Access control.
- Evaluation.
- Observability.
- Governance 和 named ownership.

可以把最小 metadata schema 整理為:

```text
skill_id
name and description
owner and domain
version and lifecycle state
compatible harnesses and models
dependencies
permissions and risk class
evaluation suite and latest result
source repository and change history
```

這段 schema 是依講者列出的平台需求所做的編者整理, 不是影片中的正式 specification.

Runtime 不應無條件拉取 `latest`. 對 production workflow, 更安全的做法通常是 pin approved version, 在 staging 重新評估新版本, 通過後再更新 lockfile 或 deployment manifest. 否則 registry 的正常更新可能直接改變 production behavior.

## 技術平台無法取代治理責任

[15:14](https://www.youtube.com/watch?v=M05vON8i0aI&t=914s)

Catalog, CLI 和 policy engine 建好後, 最難的問題仍然是誰有權決定內容. 講者建議 architects, engineering leads, infrastructure leads 和 cybersecurity leads 分別擁有相關 domains.

可操作的責任分工包括:

- Domain owner 定義規則和適用邊界.
- Skill maintainer 實作, 測試和處理回報.
- Security reviewer 檢查 scripts, dependencies 和 permissions.
- Platform team 維護 registry, distribution 和 observability.
- Workflow owner 決定 production 採用版本並承擔 outcome.

若 skill 沒有 owner, 發現錯誤時就無法判斷誰應修正, 誰能批准新版本, 以及哪些 workflows 需要回滾.

## 15 個團隊的情境是 Simulation

[16:34](https://www.youtube.com/watch?v=M05vON8i0aI&t=994s)

講者建立一個 15-team, 6-month simulation, 變數包括每隊人數, 每位工程師建立的 skills, 使用頻率, duplication ratio, quality 和 security. 畫面用它比較未治理與治理後的組織狀態.

這個 simulation 的用途是呈現假設中的 feedback loop:

```text
shared catalog
  -> reuse instead of duplicate creation
  -> more shared improvement
  -> higher consistency
  -> lower repeated prompting and rework
```

但影片沒有提供模型方程式, input distributions, calibration data 或與真實公司結果的對照. 因此不能引用它宣稱治理造成特定 productivity uplift, cost reduction 或 security improvement. 真正 rollout 應先記錄 baseline, 再量測 reuse rate, duplicate rate, task success, maintenance load 和 incidents.

## 治理範圍最後要擴展到 Workflows

[17:58](https://www.youtube.com/watch?v=M05vON8i0aI&t=1078s)

Skills 只是 workflow 的一個 component. 即使 skills 已被治理, hooks, agents, tool permissions, model configuration 和 execution sequence 仍可能造成風險.

講者因此把 central platform 延伸為 workflow marketplace. 工程師可以取得一個已治理的 infrastructure provisioning 或 regulatory review workflow, 在 sandbox 測試, 改善後再把新版貢獻回平台.

Workflow catalog 至少需要額外保存 trigger, input and output contracts, referenced skill versions, tool permissions, model policy, human approval points, rollback procedure 和 end-to-end evals.

## Skill Evaluation 與 Auto-Evolution

[19:22](https://www.youtube.com/watch?v=M05vON8i0aI&t=1162s)

講者認為接下來的發展重點包括 skills registries, skills evaluation 和 auto-evolving skills. 目前較容易執行的第一層檢查是 static evaluation, 例如 skill 是否能被正確觸發, 結構是否符合約定, instructions 是否符合供應商 best practices.

但結構正確不代表 task outcome 正確. 完整 evaluation 應分層:

```text
static checks
  -> trigger and selection tests
  -> task-level behavioral evals
  -> composition and conflict tests
  -> model and harness compatibility matrix
  -> security and permission tests
  -> production monitoring
```

Auto-evolution 會放大治理的重要性. 若系統能根據 traces 或 failures 自動修改 skill, 就必須限制 training data, mutation scope 和發布權限. 每個候選版本應保留 diff, evidence 和 eval result, 並先通過 approval gate, 不能直接覆寫 production version.

## 實務落地順序

依影片主張與上述限制, 可以採取以下漸進路線:

1. Inventory 現有 skills, owners, users 和重複項目.
2. 定義最小結構, metadata 和 lifecycle states.
3. 先建立 searchable catalog, 不急著自動更新 production.
4. 為高使用率 skills 建立 task-level evals 和 compatibility matrix.
5. 加入 signed releases, dependency scanning 和 permission review.
6. 以 domain ownership 取代單一中央內容團隊.
7. 在 staging 測試 skill composition 和 workflow behavior.
8. 量測 reuse, success rate, token cost, maintenance time 和 incidents.
9. 證明 skill governance 有效後, 再擴展到 workflow registry.
10. 最後才考慮具 approval gate 的 auto-evolution.

## 核心結論

Skills 能把組織知識從散落 prompts 轉成可搜尋, 可重用及可執行的資產. 當使用規模擴大後, 最重要的問題會從「如何建立 skill」轉成「誰擁有它, 如何驗證它, 哪個版本能進 production, 發生錯誤時如何追溯」.

中央 registry 是必要基礎, 但不是完整答案. 真正的可靠性來自 federated ownership, version pinning, task-level evals, security review, runtime observability 和 workflow-level controls. 若缺少這些機制, auto-evolving skills 只會更快累積無人理解的技術債.

## 來源與信心限制

- 本筆記依據 YouTube 英文自動字幕整理, 不是逐字稿. 講者姓名等明顯轉錄錯誤已依影片 metadata 修正.
- Central platform 架構和 microservices analogy 是講者的設計主張, 影片沒有展示 QuantumBlack production implementation 或 migration results.
- 15-team, 6-month 案例是 simulation, 不是客戶案例或受控實驗.
- 影片提及的 skills 數量成長和 benchmark 改善沒有在演講中提供足以獨立查核的完整來源與方法.
- Metadata schema, version pinning, evaluation layers 和落地順序標示為編者整理, 用於把演講原則轉成可檢查的工程做法.
- Skills 和 agent platforms 仍快速演進. 實作時應重新確認使用中的 harness, registry protocol, permission model 和供應商文件版本.
