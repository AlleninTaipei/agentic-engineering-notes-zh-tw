# Karpathy 談 Code Agents、AutoResearch 與 Loopy AI 時代

- 影片: [Skill Issue: Andrej Karpathy on Code Agents, AutoResearch, and the Loopy Era of AI](https://www.youtube.com/watch?v=kwSVtQ7dziU)
- 頻道: No Priors: AI, Machine Learning, Tech, & Startups
- 訪談者: Sarah Guo
- 來賓: Andrej Karpathy
- 發布日期: 2026-03-20
- 片長: 1:06:31
- Video ID: `kwSVtQ7dziU`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

這場訪談從 Karpathy 個人使用 coding agents 的劇烈轉變開始, 延伸到 AutoResearch、研究組織自動化、模型能力的 jaggedness、工作市場、open-source models、robotics 與 agentic education。

Karpathy 表示, 自 2025 年 12 月左右開始, 自己從主要手寫程式轉成主要委派 coding agents。他現在以較大的「macro actions」操作 repository, 同時讓不同 agents 研究、規劃與實作。新的瓶頸不再只是打字或 compute, 而是人能否定義互不衝突的任務、維持多個 loops、審查結果並提供可驗證的 objective。

AutoResearch 是這種思路的具體案例。它將小型 language-model training 包裝成自主實驗 loop, 讓 agent 修改 training code、執行實驗並依 validation metric 保留改善。Karpathy 表示, 在自己已手動調校過的 NanoChat repository 上, agent overnight run 仍找到他忽略的 weight decay 與 Adam beta adjustments。

他同時提出清楚限制: 這類 loop 特別適合具有 objective metric、便宜 verification 與可重複實驗的工作。離開 verifiable domains 後, models 仍可能無法掌握 nuance、適時澄清或產生具有品味的內容。這種能力 jaggedness 讓「全部都是 skill issue」只能當成探索心態, 不能當成 production reliability 假設。

訪談後半關於 jobs、model speciation、open source、robotics 與教育的內容主要是個人預測。筆記保留其論證, 但不將其視為已驗證趨勢。

## Coding Workflow 從 Lines 變成 Macro Actions

[02:55](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=175s)

Karpathy 描述自己的工作比例在數月內翻轉。過去主要親手寫 code, 現在大多數工作由 Claude Code、Codex 等 agent harnesses 執行。他甚至表示自 2025 年 12 月以來幾乎沒有親自輸入 code line。

這是個人 workflow 自述, 不是軟體工程產業採用率資料。但它具體呈現 coding interface 的轉變:

```text
以前
  human writes lines and functions

現在
  human defines repository-level changes
  agent researches, plans and implements
  human reviews outcomes and redirects
```

Karpathy 以 Peter Steinberger 同時操作多個 Codex sessions 的畫面為例。每個 agent 約執行二十分鐘, 人則在多個 checked-out repositories 間派發不互相干擾的功能。

可並行的 macro actions 包括:

- Agent A 實作獨立 functionality。
- Agent B 研究解法或相關技術。
- Agent C 產生 implementation plan。
- Agent D 審視已完成工作。

Parallelism 不等於無限制增加 sessions。人仍需判斷 dependency、merge conflict、shared state 與 review priority。影片沒有提供這種多 session workflow 的 defect rate 或真正節省時間。

## 新瓶頸: 人類的 Attention 與 Token Throughput

[06:15](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=375s)

Karpathy 將未使用完的 agent subscription 類比為博士班時沒有充分使用 GPU。過去在意 available FLOPs 是否持續運轉, 現在則在意能指揮多少 token throughput。

這種心態帶來兩面性:

- 正面: 鼓勵建立可並行、可長時間自主運作的 workflows。
- 負面: 容易把持續消耗 tokens 誤認為生產力, 或製造「所有閒置都代表浪費」的焦慮。

編者整理: 真正需要最佳化的不是 token utilization, 而是 verified useful outcomes per unit of human attention、cost 與 risk。影片本身以探索速度為主, 沒有提出這個衡量公式。

## 從 Agent Session 到 Persistent Claw

Karpathy 使用 `claw` 描述比單次 coding session 更持久的 agentic entity。它具有 sandbox、持續 loop、memory、personality 與 messaging portal, 即使使用者不在場也能執行工作。

他認為 OpenClaw 類系統同時在多個方向創新:

- Persistent execution。
- 較完整 memory system。
- 以 persona 或 `SOUL.md` 建立穩定互動風格。
- 透過 WhatsApp 等單一入口協調多種 automation。
- 與使用者形成類似 teammate 的關係。

訪談對 personality 的討論偏主觀。Karpathy 認為 Claude 的 praise 比較有選擇性, Codex 則更乾燥。這反映個人 UX 偏好, 不是 model behavior 的系統性評估。

## Dobby: 三個 Prompts 接管智慧家庭

[11:16](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=676s)

Karpathy 建立一個稱為 Dobby 的 home automation agent。Agent 掃描 local network、發現 Sonos 與其他 smart-home systems、搜尋控制方法、建立 APIs 與 dashboard, 最後透過 WhatsApp 接受自然語言指令。

Dobby 能控制或監控:

- Sonos。
- Lights。
- HVAC。
- Shades。
- Pool 與 spa。
- Security camera。

Security camera 使用 change detection, 再將可能有事件的影像交給 Qwen model 判讀。例如偵測 FedEx truck 後, Dobby 會把 snapshot 與說明傳到 WhatsApp。

這個案例展示 agent 如何把六個分散 apps 合成一個 intent-based interface。但它同時暴露重大風險:

- Local devices 缺少 authentication。
- Agent 主動掃描 network。
- Security system 與 cameras 涉及敏感資料。
- Control APIs 可能有實體副作用。
- 自動搜尋到的 unofficial endpoints 可能不穩定。

Karpathy 也承認自己尚未讓 agent 存取 email、calendar 與完整 digital life, 主要原因是 security、privacy 與系統仍粗糙。這項保留比「三個 prompts 就完成」更接近 production boundary。

## Agent-First Software 與 Ephemeral UI

Karpathy 從 Dobby 推論, 部分 bespoke apps 未來可能被 APIs 加 agent layer 取代。Agent 代表使用者呼叫不同 tools, 再臨時產生需要的 UI 或 automation。

```text
today
  one device -> one app -> one workflow

agent-first possibility
  devices expose APIs
        ↓
  agent interprets intent
        ↓
  composes workflows and temporary UI
```

這種方向可以降低人類學習多套 UI 的成本, 但不表示產品只要公開 raw endpoints 即可。Agent-facing contracts、authentication、permissions、confirmation、error handling 與 audit 仍需設計。

這裡和 Karpathy 後續的 AutoResearch 形成共同主題: 將人從每個中間步驟移出, 只在目標、界線與例外處介入。

## AutoResearch: 先把人移出 Experiment Loop

[15:51](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=951s)

AutoResearch 的目標不是讓 agent 模仿研究員對話, 而是重新安排 experiment system, 讓 agent 能長時間自主改善一個客觀指標。

```text
objective + metric + boundaries
            ↓
agent modifies training code
            ↓
run experiment
            ↓
measure validation result
            ↓
keep or reject change
            ↓
repeat without human prompt
```

Karpathy 將 NanoChat 視為 training-LLM playground。他已用多年經驗手動執行 hyperparameter tuning, 認為 repository 相當成熟。但 AutoResearch overnight run 仍找到他遺漏的 value-embedding weight decay 與 Adam beta adjustments。

這個案例的證據界線是:

- 講者親自執行並觀察到改善。
- 改善項目與大致時間被具體說明。
- 影片沒有提供 baseline score、最終 score、experiment count、compute cost 或重現步驟。
- 多個 tuning changes 彼此交互作用, 不能只從訪談判斷單一因素貢獻。

因此它能證明 autonomous search 在一個明確 playground 中找到有用 candidate, 不能證明 agents 已普遍比資深 researchers 更好。

## Program.md: 研究組織也能成為可最佳化程式

AutoResearch 使用 `program.md` 說明 agent 如何研究、可嘗試哪些方向及如何運作。Karpathy 認為這份文件只是人工撰寫的第一版 research policy, 本身也能被最佳化。

他和 Sarah Guo 討論一種 meta-loop:

1. 多個參與者撰寫不同 `program.md`。
2. 在相同 hardware 與 task 下執行。
3. 比較各 research process 帶來的 improvement。
4. 將結果交給 model 分析。
5. 生成下一版更好的 `program.md`。

更一般地說, research organization 可以被描述為一組 Markdown files:

- Roles。
- Experiment queue。
- Risk preference。
- Review rules。
- Communication protocol。
- Merge policy。
- Stop conditions。

一旦這些 policy 變成 code or files, 就能做版本控制與 eval。這是訪談中的概念延伸, 影片沒有展示已運作的 meta-optimization system。

## AutoResearch 的第一項硬限制: 必須可評估

[22:45](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=1365s)

Karpathy 明確指出, AutoResearch 最適合 objective metric 容易計算的問題。例如 CUDA kernel optimization 同時具有:

- 相同 functional behavior。
- 可檢查 correctness。
- 可測量 speed。
- 可反覆執行 benchmark。

若無法可靠 evaluation, 就無法建立自動 improvement loop。主觀設計、產品 nuance、需求理解與需要澄清的任務仍難以放入同一模式。

這也是 production implementation 最重要的決策規則:

```text
cheap, objective verification
  -> autonomy and parallel search can increase

ambiguous or gameable evaluation
  -> preserve human judgment and adversarial checks
```

Metric 即使存在也可能被 overfit 或 gaming。訪談後段談 LLM training 時, Karpathy 也承認 autonomous loop 可能過度追逐既有 metrics, 因而需要更完整的 metric coverage。

## Models 仍像 Brilliant PhD 加十歲孩童

Karpathy 用「極聰明的 systems-programming PhD student 與十歲孩童同時存在」描述 models 的 jagged capability。

Agent 可以執行數小時、修改大型 repository, 卻也可能:

- 完全誤解功能。
- 在錯誤方向反覆 loop。
- 不知道何時應澄清。
- 浪費大量 compute 在明顯錯誤上。
- 對 nuance、taste 或柔性目標表現不穩定。

他推測部分原因是 reinforcement learning 特別容易改善 verifiable domains。例如 code 可執行 unit tests, 但幽默、意圖理解與品味缺少同等清楚 reward。

訪談用「Why don't scientists trust atoms? Because they make everything up」這個長期重複的 joke 說明, models 在 coding 能力大幅提升時, joke diversity 並未同步改善。

這是有啟發性的例子, 但不是 generalization research。無法從單一 prompt 斷言 code intelligence 不會遷移到其他能力。

## Model Speciation: 從單一 Oracle 走向專門智能

[28:25](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=1705s)

Karpathy 預測未來可能出現更多 model speciation。現況 frontier labs 傾向建立同時處理所有使用者問題的 general model, 但特定領域可能使用較小、更高 throughput 或更低 latency 的 specialized model。

可能驅動因素包括:

- Serving compute pressure。
- 高價值 niche applications。
- 對 Lean mathematics 等專業 domain 的需求。
- General model 的 jagged capability。
- 企業已知且狹窄的 workload。

目前障礙是對 weights 的控制仍不成熟。Fine-tuning 可能破壞原能力, continual learning 仍困難, context-window customization 比 weight updates 更便宜、更可靠。

這一章是未來判斷, 不是影片中的已部署系統比較。

## 從單一 Loop 到網際網路研究 Swarm

[32:30](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=1950s)

AutoResearch 的單一 loop 很容易複製到多個 trusted workers。Karpathy 更有興趣的是如何讓網路上的 untrusted compute 共同工作。

他將問題描述為:

```text
large untrusted worker pool
  -> proposes code commits

small trusted verifier pool
  -> reruns candidate
  -> validates metric and safety
  -> accepts useful commits
```

這適用於「search 很貴、verification 很便宜」的問題。一個 worker 可能嘗試一萬個 ideas 才找到 candidate, verifier 卻只需重跑成功 candidate。Karpathy 類比 Folding@home、SETI@home, 並鬆散類比 blockchain 的 proof-of-work 與 commits chain。

主要困難包括:

- Arbitrary code execution。
- Sandbox escape 與 supply-chain risk。
- Fraudulent scores。
- Reproducibility。
- Hardware differences。
- Duplicate search。
- Trusted verification cost。
- Attribution 與 incentives。

影片沒有提供可用 implementation。這是 AutoResearch@home 的方向性構想。

## Jobs Data: 先區分 Digital 與 Physical Work

[37:28](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=2248s)

Karpathy 使用美國 Bureau of Labor Statistics 資料探索不同 occupations 與官方 growth outlook。他不是建立自己的 labor forecast, 而是用 public data 促進思考。

他的主要分類是:

- Digital information work: 可透過電腦與遠端完成, 較快受到 agents 影響。
- Physical-world work: 涉及 atoms、現場互動與設備, 自動化速度可能較慢。

他強調 jobs 是 tasks 的 bundles。AI 可能先加速其中部分 tasks, 不必然直接刪除整個 occupation。Software 變便宜也可能因 latent demand 與 Jevons paradox 增加總需求。

Bank ATM 與 bank tellers 被用作典型例子: automation 降低 branch operating cost, 增加 branches, 因而未按直覺直接消除 teller jobs。

這種歷史類比不能決定 AI employment outcome。Demand elasticity、產業集中、技能轉換速度、政策與收入分配都可能不同。Karpathy 也明確表示自己不是 labor economist, 長期結果很難預測。

較穩健的短期建議是把 agents 當成新工具, 找出自身職務中可加速的 tasks, 而不是把單一 occupation label 當成是否被取代的判斷單位。

## Frontier Labs 與 Independent Researchers 的張力

訪談討論研究者是否應加入 frontier lab。Karpathy 認為兩邊都有不可替代的價值:

### Inside frontier lab

- 接近最新 capability 與 opaque training systems。
- 能理解即將出現的技術。
- 可參與高影響力研究與內部決策。

### Outside frontier lab

- 較少 financial、organizational 與 messaging pressure。
- 能自由評論不同 labs。
- 可建立 ecosystem-level projects。
- 更容易保持與單一公司利益的距離。

他也承認, 長期離開 frontier systems 可能使技術判斷 drift。理想模式可能是在內外角色間移動, 但這是個人價值取捨, 不是普遍職涯建議。

## Open Models 可能持續落後 Frontier 數月

[48:25](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=2905s)

Karpathy 將 open models 類比 Linux: 產業需要一個共同、可控制且較少依賴單一供應商的平台。他認為 closed frontier models 可能持續領先, open models 則落後若干月並覆蓋大量一般 use cases。

他猜測 frontier intelligence 未來可能更多用於極高難度問題, open models 則能在 local devices 或一般 products 中處理日常工作。

這段包含多項不穩定預測:

- 所謂 6 至 8 個月差距沒有在影片中定義 benchmark。
- Open-weight release policy 會受資本、法規與競爭改變。
- Local model adequacy 依 task、hardware 與 privacy requirement 而不同。
- Linux 類比忽略 model training 的高額 compute 與資料依賴。

可保留的核心價值判斷是: Open models 提供 ecosystem power balance 與可控制的共同平台, 即使它們不是能力 frontier。

## Robotics: Bits 先變, Atoms 後跟

[53:51](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=3231s)

Karpathy 以 self-driving 經驗判斷 robotics 需要長期 capital、hardware、data collection 與 conviction。十年前許多 autonomous-driving startups 最終未能持續, 顯示 physical deployment 比 digital software 困難。

他預測發展順序可能是:

```text
digital agents and software
        ↓
interfaces between digital and physical
  sensors + actuators + information markets
        ↓
general physical automation and robotics
```

Digital agents 先消化已數位化的 papers、code 與 data。當既有資訊不足時, agent 必須向世界提出新問題, 透過 lab equipment、cameras、human data collection 或 robots 取得 observations。

訪談提出 information market 構想: Agent 可為特定地點的 photo、video 或 measurement 出價, 讓人類成為 sensors and actuators。這涉及 consent、safety、misinformation、surveillance、labor standards 與 geopolitical risk, 影片沒有深入治理設計。

## MicroGPT 與 Agentic Education

[1:00:59](https://www.youtube.com/watch?v=kwSVtQ7dziU&t=3659s)

Karpathy 長期嘗試將 neural-network training 簡化到最小核心。MicroGPT 約 200 lines of Python, 包含:

- Dataset。
- Neural-network architecture。
- Forward pass。
- Autograd 與 backward pass。
- Adam optimizer。
- Training loop。

他認為 production training code 的大部分複雜度來自 efficiency, 而非核心 algorithm。若不追求速度, 基本方法可以用短程式表達。

真正的新變化是他沒有再把主要精力放在錄製固定教學。只要 code 對 agent 足夠清楚, learner 可以要求 agent:

- 用不同深度解釋同一 function。
- 依背景補足 prerequisite。
- 重複說明而不受耐心限制。
- 產生 exercise 或 alternative explanation。
- 沿著 learner 的問題動態調整順序。

作者的角色轉向提供少量高價值 bits:

- 找出真正簡潔且正確的 artifact。
- 說明不可任意改動的設計理由。
- 用 skill 描述教學 progression。
- 讓 agent 負責個人化 delivery。

Karpathy 表示 agent 能理解 MicroGPT, 卻無法自行產生同樣簡潔的最終設計。這是他對 human contribution boundary 的個人觀察, 沒有展示不同 models 的系統性測試。

## 從 Documentation for Humans 到 Context for Agents

MicroGPT 的教育觀點可以延伸到 software documentation:

```text
traditional
  author -> fixed HTML docs -> learner

agent-mediated
  author -> code + Markdown + skill
                   ↓
                 agent
                   ↓
       personalized explanation for learner
```

這不代表 human documentation 應全部刪除。Agent 仍需要 authoritative source, 人也需要可直接瀏覽、引用、審查與在 agent 不可用時使用的文件。

較合理的工程方向是撰寫同時適合人和 agent 的 source material:

- Clear Markdown structure。
- Stable terminology。
- Runnable examples。
- Explicit invariants。
- Versioned references。
- Tests 與 expected results。
- Skill 只描述教學 procedure, 不複製完整事實。

以上是依訪談論點整理的編者建議。

## 實作檢查表

### Multi-Agent Coding

- 任務是否能切成不衝突的 repository-level changes?
- 每個 agent 是否有清楚 output contract 與 branch boundary?
- 是否保留 human review priority, 而不是只最大化 token throughput?
- Research、planning、implementation 與 verification 是否適合不同 contexts?
- 並行 sessions 的成本、defect 與 merge overhead 是否量測?

### Autonomous Experiment Loop

- Objective 是否單一且可測量?
- Metric 是否便宜、可靠且難以 gaming?
- Baseline、environment 與 random seeds 是否固定?
- 每個 experiment 是否保存 code diff、config、logs 與 result?
- Agent 是否只能修改明確允許的 files and parameters?
- 是否設定 compute budget、timeout、stop condition 與 rollback?
- Accepted candidate 是否由 trusted verifier 獨立重跑?
- 是否使用多項 metrics 防止局部 overfitting?

### Persistent Personal Agent

- Local network discovery 是否明確授權?
- Smart-home、camera、email 與 calendar 是否分開權限?
- Physical side effects 是否具 confirmation、audit 與 emergency stop?
- Memory 是否能查看、修正與刪除?
- Agent-generated APIs 是否有 authentication 與 version pinning?
- External messaging 是否可能洩漏 private images or state?

### Agent-Mediated Education

- 原始 code 或 lesson 是否是 authoritative artifact?
- Agent 是否引用具體 source line 或 test?
- Skill 是否只負責 progression, 不重複事實內容?
- Learner 是否有方式檢驗自己真的理解, 而不只是獲得流暢解釋?
- Human-readable documentation 是否仍能獨立使用?

## 核心結論

訪談中最值得保留的不是「coding 已經結束」或「所有問題都是 skill issue」這類誇張說法, 而是三個可操作的轉變:

1. 人類開始用 macro actions 管理多個 agents, 主要工作轉向 task decomposition、context、verification 與 integration。
2. 當 objective 與 verification 足夠清楚時, 應重構 workflow, 讓 agent 在沒有逐輪 prompting 的情況下持續實驗。
3. 人類的稀缺貢獻逐漸集中到 models 尚無法自行產生的少量高價值 bits, 例如好的 abstraction、metric、curriculum 與 judgment。

AutoResearch 的成功條件也是它的限制。容易衡量的工作最先進入 autonomous loop, 難以衡量的 nuance、taste、goal formation 與安全責任仍不能因 agent 表面能力強大而被省略。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕與 13 個正式 chapters 整理。字幕對 OpenClaw、Qwen、NanoChat、`program.md`、Adam betas 等名稱可能有辨識誤差, 本筆記只在上下文足以確認時修正。

Karpathy 對自己使用 coding agents、Dobby、AutoResearch、NanoChat 與 MicroGPT 的描述屬於第一手 practitioner experience。影片沒有公開完整 code walkthrough、experiment logs、benchmarks、security audit 或 controlled comparison, 因此仍以講者自述為主要證據。

關於 model generalization、speciation、jobs、frontier labs、open-source gap、robotics、information markets 與教育未來的段落屬於訪談觀點與預測。影片沒有提供足以驗證因果關係的研究資料。BLS project 使用 public data, 但 Karpathy 明確表示自己不是 labor economist。

Coding-agent workflow、model capability、subscriptions、OpenClaw、AutoResearch 與 open-source model 差距都可能快速變化。實作或決策時應以當前工具、repository、security model 與實際 evals 重新確認。
