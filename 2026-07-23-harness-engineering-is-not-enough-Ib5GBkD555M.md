# Harness Engineering 還不夠, 為什麼 Software Factories 會失敗

來源: [Harness Engineering is not Enough: Why Software Factories Fail, Dex Horthy, HumanLayer](https://www.youtube.com/watch?v=Ib5GBkD555M), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=Ib5GBkD555M
- 上傳日期: 2026-07-23
- 片長: 19:17
- Video ID: `Ib5GBkD555M`
- 講者: Dex Horthy, HumanLayer
- 內容依據: 英文原始語言自動字幕

## 摘要

Agent harness 可以提高 tool use, context management, testing 與短期任務成功率, 但無法單獨解決 codebase 長期 maintainability. Dex Horthy 的核心論點是, 現有 coding models 主要被 reinforcement learning 訓練成通過 tests 且不破壞既有 behavior, reward 卻很少反映 architecture erosion, coupling 或數月後的 change cost.

這使 lights-off software factory 面臨結構性風險. Agent 可以快速生成大量通過測試的 changes, review bots 也能提高最低品質, 但 codebase 仍可能逐步累積 shotgun surgery, defensive patches, unnecessary casts 與局部 workaround. 當某個問題超出 agents 能處理的範圍, 人類才發現自己已經數月沒有讀過這套程式碼.

講者的短期解法不是放棄 agents, 而是把 lights 打開. 對重要 changes, 人類與 AI 先進行 product review, system architecture, program design 與 vertical-slice planning, 再由 agent 實作. Upfront alignment 讓 code 更接近共同決策, 因而縮短後續 review, 同時保留人類對 codebase 的理解與 ownership.

## 主流敘事, 你才是 Bottleneck

[00:00](https://www.youtube.com/watch?v=Ib5GBkD555M&t=0s)

AI coding 的激進敘事是:

- Models 已經夠好.
- Code generation 幾乎免費.
- 應投入更多 tokens 與 loops.
- 人類 review 是阻礙 throughput 的 bottleneck.
- 只要 harness, adversarial review 與 regression tests 夠強, 就能同時得到 10 至 100 倍速度和高品質.

這種觀點最終導向 lights-out software factory, 人類不再閱讀 code, 只管理 queue, tests, rollout 與 monitoring.

講者不同意把所有失敗都歸因為使用者不會 prompt 或 harness 設計不好. 他的主張是, 某些問題源於 model training objective 本身, 不能只靠在外層增加 loops 解決.

## 已開始出現的裂痕

[01:28](https://www.youtube.com/watch?v=Ib5GBkD555M&t=88s)

影片提到 AI coding adoption 後的幾種警訊:

- Coding-agent mishaps 導致 production outages.
- Codebase degradation 速度加快.
- Pull requests 增多, comments 變長.
- 更多 PR 未經充分 review 就 merge.
- Incidents 與 bugs per developer 增加.

講者引用 AI Engineer Europe 的發言與 Faros AI report 支持這些觀察. 影片沒有展示完整 methodology, sample 或 causal analysis, 因此本筆記將它們視為講者引用的產業訊號, 不視為 AI coding 必然造成上述結果的證明.

AI adoption 往往也伴隨 throughput, team size, project mix 與 review policy 改變. 若要建立因果關係, 需要控制這些因素.

## 核心 Thesis, Harness 不是完整答案

[02:20](https://www.youtube.com/watch?v=Ib5GBkD555M&t=140s)

Harness 擅長控制可觀察且能在 execution episode 內驗證的項目:

- Tool permissions.
- Context and state.
- Max steps and retries.
- Tests and lint.
- Sandbox and rollout.
- Verifier and review loops.

Maintainability 的代價卻常在數週, 數月甚至數年後出現. 如果 reward signal 只在當前 task 結束時判斷 tests 是否通過, 就很難教模型避免未來才顯現的 architecture cost.

```text
Harness can constrain execution
  !=
Harness can supply a missing long-term quality objective
```

Review agents 與更多 tokens 可以提高品質底線, 但若 evaluator 也缺乏可靠的 maintainability criterion, 它仍可能批准短期正確, 長期昂貴的設計.

## 傳統 Software Factory

[03:36](https://www.youtube.com/watch?v=Ib5GBkD555M&t=216s)

講者先回顧約 2022 年的典型 software delivery loop:

```text
Product need
  -> Tracker, Jira / Linear / other state machine
  -> Engineer builds
  -> Automated and manual tests
  -> Pull request
  -> Human code review
  -> Human acceptance test
  -> Production rollout
  -> Monitoring and user feedback
  -> New work
```

Building 與 reviewing 大型 change 都可能花數小時或數天. 團隊因此發展 upfront planning, architecture proposals 與 sprint planning, 先對 problem 和 design 達成共識, 降低 rework 與逐行 review 時的認知成本.

這些流程不是單純 bureaucracy. 在複雜系統中, 它們將高成本 decision 提前, 讓 implementation 成為已對齊設計的具體化.

## Agentic Software Factory

[05:52](https://www.youtube.com/watch?v=Ib5GBkD555M&t=352s)

Agentic factory 把 "someone builds the thing" 換成 "agent builds the thing":

```text
Issue queue
  -> Orchestrator
  -> Harness
  -> Model
  -> Sandbox and computer use
  -> Pull request
```

Implementation 從數天縮短到數分鐘或數小時, 但 human review 仍可能需要數小時. 團隊因此加入 agentic code review 和 automated regression testing.

下一步則把 incidents 與 user feedback 直接送入 factory. Agent 自動產生 fixes 和 features, 人類只剩下排定需求與驗收 changes.

當人類連 code review 都移除後, 就進入 lights-off factory. 系統將信任放在 tests, monitoring, rollout controls 與 automated verifiers, 而不再要求任何人理解每一行 code.

## Lights-Off Factory 的邊界

[07:30](https://www.youtube.com/watch?v=Ib5GBkD555M&t=450s)

講者明確區分兩種情境:

- 小型 side project 或短期 marketing site.
- 需要長期維護的複雜 brownfield system.

兩者的 constraints 幾乎不同. Vibe coding 在低風險, 短生命週期專案可能完全合理. 本片關注的是 hard problems in complex codebases.

講者也重新定義 brownfield. 它不一定是十年歷史的 Java system. 在 AI 加速下, codebase 只要高速演進三至六個月, 也可能累積足夠 complexity, 讓 agents 難以掌握.

這是其實務觀察, 不是所有 repository 都會在固定時間後失效.

## 2025 年 Lights-Off 實驗

[07:48](https://www.youtube.com/watch?v=Ib5GBkD555M&t=468s)

HumanLayer 在 2025 年 7 月嘗試完全 lights-off. 數月後出現 agent 無法解決的 issue. 團隊使用進階 prompting, research 與 reproduction 仍無法修復, 最後必須由人類重新深入閱讀已停止關注約三個月的 codebase.

同時, site down, users 不滿, 而工程師必須在危機中理解大量未曾 review 的 code. 講者將這次經驗濃縮為:

```text
Agent can solve many local tasks
  -> Humans stop reading code
  -> Maintainability debt accumulates
  -> A hard failure exceeds agent capability
  -> Humans must recover context under pressure
```

這是一個公司自身的案例, 不足以量化所有 software factories 的 failure rate, 但能說明 lights-off 模式的 tail risk.

## 模型無法自行維持 Codebase Quality

[08:56](https://www.youtube.com/watch?v=Ib5GBkD555M&t=536s)

講者認為模型仍需要大量 human steering 才能長期維持或改善 codebase quality. 他用 Martin Fowler 的 `shotgun surgery` 說明 maintainability degradation:

```text
一項小 change
  -> 必須修改許多不相關 modules
  -> 容易遺漏
  -> Regression risk 增加
  -> Future changes 更昂貴
```

Agent 可能每次都完成當前 ticket, tests 也都通過, 但仍逐步增加 coupling, duplication 與 hidden dependencies. 每個局部決策都看似可接受, 累積後才形成昂貴架構.

影片沒有可量化 benchmark 證明 models 在 maintainability 上沒有改善. 講者也承認這一點, 並將結論建立在使用經驗和社群觀察上.

## 為什麼 Claude Code 成功

[10:12](https://www.youtube.com/watch?v=Ib5GBkD555M&t=612s)

在 Claude Code 前已有 Aider, Codebuff 等 CLI coding agents, 也具備 read, write, edit, grep 和 shell tools. 講者認為 Claude Code 的重要差異是 model lab 能在實際要交付的 harness 中訓練模型, 讓 weights 適應 tool schemas, loop 與 context pattern.

```text
Model + Harness co-training
  -> Better tool selection
  -> Better argument generation
  -> Better recovery inside that loop
  -> More reliable task completion
```

不擁有 model weights 的 harness builder 無法直接用 RL 讓模型適應自己的 environment, 因此相較同時控制 weights 和 harness 的團隊可能處於劣勢.

影片同時提到 Claude Code 的快速營收成長數字. 該數字是演講當時的講者陳述, 具有高度時效性, 本筆記不將它用於技術結論.

## Coding Agent Reinforcement Learning

[11:29](https://www.youtube.com/watch?v=Ib5GBkD555M&t=689s)

講者用簡化流程解釋 coding RL:

1. 給模型一個 software issue.
2. 產生多條嘗試解題的 trajectories.
3. 對每條 trajectory 評分.
4. 提高成功 behavior 的機率.
5. 降低失敗 behavior 的機率.

```text
Problem
  -> Sample many agent traces
  -> Apply verifiable reward
  -> Update weights
```

常見 reward 是 binary correctness, 例如:

- 新 test 是否通過.
- 舊 tests 是否仍通過.
- 是否修復指定 behavior.

這種 reward 容易自動化, 能大規模執行, 因此適合 RL. 但它沒有直接懲罰 poor program design.

## SWE-bench 類 Task 如何評分

[12:09](https://www.youtube.com/watch?v=Ib5GBkD555M&t=729s)

影片以 SWE-bench Multilingual 類型的 task 說明:

- 從 real open-source repository 取得 issue.
- Checkout 人類修復前的 base commit.
- 對模型隱藏 golden patch 與 test patch.
- 讓 agent 修改 repository.
- 移除 agent 對 tests 的修改, 避免藉改 test 作弊.
- 套用 hidden test patch.
- 執行原有和新增 tests.
- 全部通過才取得 reward.

這能驗證 functional correctness 與 regression, 卻未必判斷 fix 是否放在正確 abstraction, 是否增加 coupling, 或是否建立 future maintenance trap.

## Reward 會塑造局部最佳解

[13:00](https://www.youtube.com/watch?v=Ib5GBkD555M&t=780s)

如果唯一可驗證目標是 test pass, 模型會偏向任何能快速滿足 tests 的 patch. 影片舉出幾種 code smell:

- 在不必要位置加入 try/catch.
- 以 type casting 繞過 type system.
- 針對 symptom 加 local guard, 不修 root cause.
- 只滿足當前 fixture 的 special case.

這不代表 RL 必然產生壞 code, 而是 reward 沒有包含的品質很難穩定最佳化.

```text
What gets verified
  -> What gets reinforced

What appears months later
  -> Weak or missing training signal
```

## Maintainability 為什麼難以 Verify

[13:18](https://www.youtube.com/watch?v=Ib5GBkD555M&t=798s)

Code 能否執行與 tests 是否通過, 通常能在數分鐘內得到 binary answer. Bad architecture 的 cost 可能幾個月後才出現:

- 新需求需要跨多處修改.
- 新 engineer 無法理解 ownership boundary.
- 原本獨立 components 被隱性 coupling.
- Performance 或 reliability 問題只在規模下顯現.
- 每次 change 都增加 regression surface.

要把數月後的代價回傳到某一次 training trajectory, credit assignment 非常困難. Maintainability 也涉及不同情境和 trade-offs, 很難用單一 binary test 表示.

## 新型 Long-Horizon Benchmarks

[13:42](https://www.youtube.com/watch?v=Ib5GBkD555M&t=822s)

影片提到幾個探索方向:

- Sweep Marathon, 超長時間的完整 application tasks.
- DeepSweep, 在未曾真實完成的 open-source tasks 上測試較大型 changes.
- Frontier Code, 使用 multi-PR tasks, 更嚴格的 test quality 和 judge models.

這些 benchmark 名稱與細節依自動字幕和講者口述整理, 可能存在版本或拼字差異. 它們共同嘗試把 evaluation 從單一 15-minute patch 擴展到 multi-step, long-horizon 與跨 PR work.

影片提到的改善包括:

- 如果 agent 寫的 test 在 pre-patch code 上也通過, 就給予懲罰.
- 以 judge model 檢查 code-quality rules.
- 評估多個 sequential changes 的一致性.
- 使用不在原始 training distribution 中的 tasks.

Judge model 仍有 ceiling. 如果模型無法穩定辨認好設計, 用另一個相近模型 review 不一定能可靠解決問題.

## Review Agents 能做什麼, 不能做什麼

[14:24](https://www.youtube.com/watch?v=Ib5GBkD555M&t=864s)

Review agents 和額外 tokens 仍然有價值. 它們可以:

- 找出明顯 bugs.
- 檢查 style 和 policy.
- 執行 tests.
- 比較 implementation 與 spec.
- 提醒常見 code smells.
- 提高最低品質.

但它們無法保證:

- Architecture 在未來需求下仍合適.
- Abstraction boundaries 能長期演進.
- 所有 hidden constraints 都已被 context 捕捉.
- 人類完全不需理解 code.

因此, adversarial reviewer 是輔助 verifier, 不是 code ownership 的替代品.

## 短期結論, 還是要讀 Code

[14:58](https://www.youtube.com/watch?v=Ib5GBkD555M&t=898s)

講者認為, 在 maintainability evaluation 尚未解決前, 團隊仍需閱讀 production code. 這不代表回到純手工開發, 而是重新設計 AI leverage 的位置.

```text
AI accelerates research, planning and implementation
Human aligns architecture and owns review
Automated tests verify behavior
Production feedback updates the plan
```

關鍵不是把所有工作交回人類, 而是透過 upfront alignment 讓人類 review 的 code 更少偏離, 更容易理解.

## 四層 Upfront Planning

[15:12](https://www.youtube.com/watch?v=Ib5GBkD555M&t=912s)

### 1. Product Review

先確認:

- 要解決哪個 user problem.
- Desired behavior 是什麼.
- Mockups 和 workflows 如何運作.
- 哪些 scope 明確不做.
- Success 如何觀察.

小 change 可以直接交給 agent, 不需要完整儀式. 重要且跨系統的 change 才值得增加 planning depth.

### 2. System Architecture

定義 high-level structure:

- Components and ownership.
- Contracts and interfaces.
- Data models.
- Constraints.
- Cross-service communication.
- Security and failure boundaries.

這一步讓團隊先對 systems 如何組合達成共識.

### 3. Program Design

講者認為這是 agentic coding 最被低估的階段. Architecture 正確, 不表示模型能自行產生好 program layout. 團隊還要設計:

- Types.
- Function and method signatures.
- Module layout.
- Call stacks and call graphs.
- Dependency direction.
- Error handling.
- State transitions.

Program design 位於 architecture 與 implementation 之間. 它把抽象 component contracts 轉成 agent 可直接實作的 code-level structure.

### 4. Vertical Slices

最後決定 implementation order:

- 先建立哪一條 end-to-end path.
- 多 repository 如何協調.
- 每個 phase 如何測試.
- 哪些 checkpoints 需要 human review.
- 何時擴展下一個 slice.

Vertical slice 穿過必要 layers, 早期產生可執行 feedback. 這比先完成所有 database, 再所有 APIs, 最後才整合 UI 的 horizontal plan 更容易驗證.

## 30 分鐘 Alignment 如何節省 Review

[16:32](https://www.youtube.com/watch?v=Ib5GBkD555M&t=992s)

講者主張, 約 30 分鐘的 pre-planning 與 alignment 可能節省數小時 review. 這個數字是經驗性說法, 實際效益取決於 task size, team familiarity 與 codebase complexity.

機制是:

```text
Align on problem
  -> Align on architecture
  -> Align on program structure
  -> Define vertical slices
  -> Agent implements agreed plan
  -> Reviewer recognizes expected design
  -> Faster code review
```

AI 也可以加速 planning, 搜集 codebase context, 產生 call graphs, 比較 alternatives 與起草 documents. 人類的責任是做 judgment 和 alignment, 不一定親手撰寫所有規格文字.

## 問題不是 PR 太多, 而是 Bad PR 太多

[17:16](https://www.youtube.com/watch?v=Ib5GBkD555M&t=1036s)

如果 reviewer 被 PR 淹沒, 直覺解法是加入更多 review bots 或取消 human review. 講者提出另一種診斷: PR 難以審查, 是因為 implementation 和共同理解偏離太大.

Good PR 具備:

- Scope 清楚.
- Design 已事先討論.
- Change 符合預期 types 和 boundaries.
- Tests 對應 desired behavior.
- Diff 能以可理解順序閱讀.
- 沒有不必要的 unrelated changes.

即使一個 PR 只需 20% rework, 仍會對 reviewer 和 submitter 造成顯著 intellectual 與 emotional cost. Upfront design 的目的就是降低這個比例.

## 實務工作流

### Small Change

```text
Issue
  -> Agent implements
  -> Tests
  -> Human reads diff
  -> Merge
```

### Complex Change

```text
Product review
  -> Architecture proposal
  -> Program design
  -> Vertical slice plan
  -> Agent implements one slice
  -> Automated verification
  -> Human code review
  -> Continue next slice
```

### Incident

```text
Monitoring detects issue
  -> Agent gathers evidence and proposes fix
  -> Human confirms root cause and risk
  -> Agent implements bounded patch
  -> Regression tests and rollout guardrails
  -> Postmortem updates architecture knowledge
```

後三個流程是依影片原則整理的實務模板, 不是講者投影片的逐字內容.

## Code Review Checklist

### Behavior

- Change 是否符合 product review 的 desired behavior?
- Tests 是否在 pre-patch code 上會失敗?
- 是否保留既有 behavior?
- Failure modes 是否被測試?

### Architecture

- Component ownership 是否清楚?
- 是否新增不必要 coupling?
- Data flow 與 dependency direction 是否符合設計?
- 是否出現 shotgun surgery 的前兆?

### Program Design

- Types 是否表達 domain constraints?
- Function signatures 是否穩定且明確?
- Call graph 是否比必要情況更複雜?
- Error handling 是否修 root cause, 還是只包 try/catch?
- Casting 或 guards 是否在掩蓋 modeling problem?

### Maintainability

- 下一個相似 change 會更容易還是更困難?
- 新 engineer 能否從 code 理解 decision?
- 是否需要同步更新 architecture docs?
- 是否引入 duplicated source of truth?

### Ownership

- Reviewer 是否能解釋這個 change 如何工作?
- 發生 production incident 時, 團隊知道從哪裡開始查嗎?
- 是否只是因 agent 和 tests 都說成功就 merge?

## 核心觀念回顧

```text
Software quality
├── Immediate correctness
│   ├── Tests pass
│   ├── No regression
│   └── Required behavior works
└── Long-term maintainability
    ├── Clear abstractions
    ├── Low coupling
    ├── Coherent program design
    ├── Evolvable architecture
    └── Human understanding and ownership
```

最值得帶走的三個原則:

1. Tests 和 harness 能驗證短期 behavior, 但不自動代表 codebase 長期健康.
2. Model training 會強化可快速驗證的 reward, maintainability 的 delayed cost 則難以回傳.
3. 目前最實用的速度來源是 AI-assisted upfront planning 加 human code ownership, 不是完全取消 review.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕可能誤辨 benchmark, company, product 與 speaker names. 本筆記對 `SWE-bench` 等上下文可確認的術語做了修正, 對較新的 benchmark 名稱保留不確定性.

演講反映 HumanLayer 創辦人的實務經驗與產品立場, 並包含 HumanLayer 產品介紹. Lights-off 實驗是單一公司的案例, 產業 report 與營收數字未在影片中提供完整驗證資料. 關於 coding models 尚未改善 maintainability 的判斷, 講者自己承認目前缺少成熟 benchmark, 因此應視為有實務依據的 thesis, 不是已定論的科學結果.
