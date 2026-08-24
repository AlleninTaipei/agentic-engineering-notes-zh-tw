# 企業如何擁有自己的 Intelligence, 從租用模型到掌控 Weights

來源: [How Companies Are Building Their Own Intelligence, Sonya Huang, Sequoia Capital](https://www.youtube.com/watch?v=bMMv0bZzONg), Sequoia Capital

- 正規網址: https://www.youtube.com/watch?v=bMMv0bZzONg
- 上傳日期: 2026-08-11
- 片長: 17:29
- Video ID: `bMMv0bZzONg`
- 講者: Sonya Huang, Sequoia Capital
- 內容依據: 英文原始語言自動字幕

## 摘要

Sonya Huang 將 sovereign AI 定義為企業逐步擁有自身 intelligence, 最深可以延伸到 model weights, training pipeline, data 與 serving stack. 這不代表所有公司都應停止使用 Opus, GPT 或其他 closed APIs. 真正的策略問題是, 哪些能力值得擁有, 哪些能力繼續租用更合理.

影片提出四個 ownership 驅動因素: cost, speed, performance 與 control over destiny. 當某項 AI capability 高頻, latency-sensitive, 使用高度 proprietary data, 或直接決定產品差異時, 自建模型與 stack 的價值較高. 反之, 需要通用 frontier capability, 尚未形成穩定需求或缺乏訓練資料時, 租用通常有更高起點與較低複雜度.

從零開始的建議路徑包含四個面向: strategy, team, legibility 與 technical roadmap. 技術上則從 evals 開始, 接著探索 model routing, harness 與 context, 再依必要性進入 post-training, mid-training 或 pre-training, 最後建立 production feedback 與 online learning.

## Sovereign AI 是什麼

[00:00](https://www.youtube.com/watch?v=bMMv0bZzONg&t=0s)

Sovereign 原指擁有完整自我治理權的獨立主體. 影片將 sovereign AI 延伸為, 公司不完全依賴外部模型供應商, 而能控制自己的 intelligence, 最深可擁有並保管 model weights.

這個定義有一項重要限定. 講者並不是要求公司全面離開 closed models. 對 coding agents, desktop work 與需要 frontier-level general capability 的任務, closed APIs 仍然很好用. 企業可以在產品的部分環節垂直整合, 同時在其他環節繼續租用模型.

```text
Sovereignty spectrum

Rent everything
  -> Own prompts and evals
  -> Own harness and context
  -> Own routing and serving
  -> Own post-training and data flywheel
  -> Own model weights
  -> Own pre-training
```

這是一條 ownership spectrum, 不是 0% 與 100% 的二元選擇.

## Centralized 與 Decentralized Intelligence

[01:24](https://www.youtube.com/watch?v=bMMv0bZzONg&t=84s)

講者對比兩種 AI 生態:

### Centralized intelligence

少數大型模型以 black box 形式支撐愈來愈多經濟活動, 並吸收來自各產業的 data exhaust 與 feedback. Intelligence 的控制權與資料飛輪集中在少數供應商.

### Decentralized intelligence

世界共享強大的基礎模型, 但個人與企業再依自己的 data, domain, workflow, personalization 與 taste 建立專屬 intelligence. 不同組織可以形成各自的能力與產品差異.

影片明確偏好第二種路徑, 認為它能讓生態系保持多樣性, 避免單一公司控制所有應用 intelligence. 這是講者的產業與投資觀點, 不是技術上已證明的必然結果.

## 企業擁有模型的四個理由

[02:54](https://www.youtube.com/watch?v=bMMv0bZzONg&t=174s)

### 1. Cost

AI product 愈成功, inference usage 可能愈高, AI COGS 也可能同步上升. 對低毛利, 零毛利或負毛利公司, 每次 request 的模型成本會直接限制商業模式.

自建 stack 可能透過小模型, distillation, quantization, batching 與專用 serving 降低單位成本. 但 ownership 也會新增 GPU, data, research, platform 與 on-call 成本, 所以不能只比較 API token price.

### 2. Speed

Coding autocomplete 與 security 等場景對 latency 很敏感. 專門化的小型 distilled model 可能比大型 general model 更快, 即使後者整體能力更強.

### 3. Performance

講者認為 open-weight models 已進步到, 在特定 domain 中配合 proprietary data 與 post-training 後, 有機會超越 closed general models. 這項優勢不是免費取得, 前提是公司有可靠 evals, 高品質資料與足夠訓練能力.

### 4. Control over destiny

公司若完全依賴外部 API, 會承受 pricing, rate limits, availability, policy, model deprecation 與 roadmap 改變的風險. 擁有 weights 可增加供應商獨立性, 也讓 serving, fine-tuning 與 release cadence 更可控.

## "Not Your Weights, Not Your Product"

[04:22](https://www.youtube.com/watch?v=bMMv0bZzONg&t=262s)

講者借用 crypto 的 "not your keys, not your crypto" 口號, 提出 "not your weights, not your product". 她的立場是, 如果 intelligence 是產品最核心的功能, 公司應考慮是否需要控制並保管自己的 weights.

這句話是刻意鮮明的 rallying call, 不宜當成普遍規則. 許多成功產品的差異可能來自 workflow, distribution, proprietary context, data network effects, user experience 或 domain trust, 而不是 model weights 本身.

較務實的問題是:

```text
如果供應商明天改變價格, 模型或政策,
哪些產品能力會失去競爭力或無法運作?
```

答案若涉及核心 revenue path, 企業就有理由提高該能力的 ownership.

## Application Company 正在成為新型 AI Lab

[05:32](https://www.youtube.com/watch?v=bMMv0bZzONg&t=332s)

過去的競爭常被描述為 foundation model labs 從模型層往應用層移動, application companies 則從使用者介面往下建立 intelligence. 講者認為新的 battleground 不只在 application layer, 也在 intelligence layer.

Application companies 掌握特定 domain 的使用行為, feedback 與 outcome data, 因此能進行高度應用導向的研究. 影片列舉法律, coding, enterprise search, 醫療, security 與 finance 公司, 並把下列工作視為 applied research:

- 建立 domain evals 與 benchmarks.
- Harness engineering.
- Fine-tuning 與 post-training algorithms.
- Synthetic data 與 expert trajectories.
- Context, retrieval 與 memory design.
- Production feedback 與 online learning.

影片將這些 application companies 稱為新的 "neo labs". 這是講者對產業趨勢的定位, 不是正式組織類型.

## 從零開始的四步框架

[07:05](https://www.youtube.com/watch?v=bMMv0bZzONg&t=425s)

講者提出一套 opinionated framework:

```text
1. Strategy
2. Team
3. Legibility
4. Technical roadmap
```

她多次提醒, 每家公司不同, 而且這是一個起點而非固定 playbook.

## Step 1, 決定 Own 什麼, Rent 什麼

[07:05](https://www.youtube.com/watch?v=bMMv0bZzONg&t=425s)

每一項 AI capability 都可以用四個維度評估:

| 維度 | 適合提高 ownership 的訊號 | 適合繼續 rent 的訊號 |
| --- | --- | --- |
| Cost | 高頻呼叫, API COGS 侵蝕毛利 | 用量低或需求仍不穩定 |
| Speed | Latency 是 P0 product requirement | 可接受較高延遲, 重視通用能力 |
| Performance | Domain tuning 可形成明顯優勢 | Frontier general capability 更重要 |
| Proprietary data | Data 高度專屬且可用於學習 | 缺少合法, 乾淨或足量資料 |

### Coding 案例

影片區分 coding agent 與 tab autocomplete:

- Coding agent 需要強大的 out-of-the-box reasoning 與 tool use, latency 未必是最高優先, 因此當時多數仍租用 frontier models.
- Tab autocomplete 呼叫頻率高, 對即時 latency 極敏感, 小型專用模型更容易在成本與速度上成立.

### Security 與 Biology 案例

Security 公司可能為 speed, domain performance 與 bespoke post-training 擁有模型. Biology 公司則可能因 proprietary scientific data 的價值, 希望在自有 stack 中訓練與部署.

這些是策略例子, 不是對整個產業的統計結論.

## Step 2, 建立專門團隊

[09:51](https://www.youtube.com/watch?v=bMMv0bZzONg&t=591s)

Lab leader 可以來自 research 或 engineering, 適合背景取決於公司做的是 fundamental research 還是 applied research.

講者不建議直接把 sovereign AI 工作塞給既有的 hub-and-spoke AI platform team. Platform team 通常以提供穩定服務, 回應內部需求與維護 shared infrastructure 為主. Own intelligence 團隊則需要主動探索 frontier, 快速實驗並產出研究.

她建議:

- 從小型 de novo team 開始.
- 讓團隊具有獨立研究任務與清楚 ownership.
- 招募能在 research 與 production 間移動的人.
- 避免讓團隊只成為其他 product teams 的 service desk.

影片以 Harvey 約七人的研究團隊作為小團隊也能產出研究的例子. 人數與組織成效是講者在演講中的陳述, 本筆記未以外部資料驗證.

## Step 3, 讓研究具有 Legibility

[11:17](https://www.youtube.com/watch?v=bMMv0bZzONg&t=677s)

Legibility 指外界能否理解公司具備哪些研究能力, 為何其 intelligence 值得信任. 內部有優秀研究但完全不可見, 對 buyer, talent 與 ecosystem 的說服力有限.

影片建議透過下列方式提高 legibility:

- 建立具品牌辨識的 labs 或 research group.
- 發布高品質 research, evals 與 technical reports.
- 用 technical marketing 解釋方法與結果.
- 讓市場看見產品改善背後的研究能力.

講者將 CEO 的責任分成 substantive results 與 narrative / legibility. 這是偏向 go-to-market 的建議. 發布研究時仍應保護 proprietary data, security details, customer confidentiality 與真正的 trade secrets.

## Step 4, Technical Roadmap

[12:33](https://www.youtube.com/watch?v=bMMv0bZzONg&t=753s)

影片提出一條常見但非強制的技術路線:

```text
Strategy
  -> Evals
  -> Model routing
  -> Harness and context
  -> Post-training
  -> Mid-training, if needed
  -> Pre-training, in rare cases
  -> Production feedback loop
  -> Online learning
```

### Evals 應該最先建立

講者特別強調 evals. 如果沒有 domain-specific measurement, 團隊就無法回答 open model 是否真的更好, fine-tuning 是否提升產品, 或 production behavior 是否正在 drift.

Evals 雖然不顯眼, 卻是後續 model selection, routing, data generation 與 training 的共同基礎.

### 先嘗試較淺的 ownership

有些公司只靠 out-of-the-box open models, routing, prompt, harness 與 context 就能達到目標. 若仍有穩定且可量測的 performance gap, 再進入 post-training. Mid-training 與 pre-training 應在更少數, 有明確理由的情況下考慮.

### 建立 feedback loop

最終目標是讓 production customer interactions 轉化為可治理的 learning signal, 持續改善 intelligence. 這需要 consent, privacy, labeling, quality control 與 rollback, 不能把所有使用者資料未經篩選地送入 training.

最後一句是根據影片架構所做的編輯補充.

## Open Weights 提高了 Performance Ceiling

[13:14](https://www.youtube.com/watch?v=bMMv0bZzONg&t=794s)

講者認為 2026 年的 open-weight models 已接近 frontier baseline, 且 weights 可修改, 因此結合 domain data, post-training, prompt / harness engineering 與 online learning 後, 有機會在特定 eval 上超越 closed frontier models.

影片字幕提到兩個當時的新 open-weight model 名稱, 但自動字幕可能誤辨版本, 因此本筆記不將其作為可依賴的產品選型資訊.

"Better than frontier" 必須附帶範圍:

- 是哪一組 domain evals.
- 與哪個 closed model 版本比較.
- 是否在相同 latency, cost 與 context 限制下測試.
- 是否包含安全性, robustness 與 long-tail cases.
- 結果能否在 production distribution 重現.

Open weights 提供更高的 customization ceiling, 但不保證每家公司都能達到更好表現.

## Production Stack 與 Development Stack

[13:56](https://www.youtube.com/watch?v=bMMv0bZzONg&t=836s)

影片把完整系統分成兩側.

### Production stack

這是使用者實際接觸的 intelligence, 可概括為:

```text
User request
  -> Harness
      -> Routing
      -> Tools
      -> Context
      -> Policies
      -> Verification
  -> Model
  -> User-visible result
```

講者認為 production intelligence 基本上是 model 加上 harness. 最終表現不只取決於 weights, 還取決於模型在什麼 context 中, 能使用哪些 tools, 如何被路由與驗證.

### Development stack

這是讓 intelligence 持續改善的系統:

- Evals 與 production monitoring.
- Training data pipeline.
- Expert trajectories.
- Synthetic data.
- RL environments.
- Post-training infrastructure.
- Online learning.

Closed API stack 通常較簡單. 供應商負責 weights, serving 與多數 model development, 應用公司專注 prompt, context, harness 與 evals. 講者稱之為 higher floor, lower ceiling.

Own-weight stack 的 floor 較低, 因為團隊要自己選 base model, 訓練, serving 與維護. 但 customization 與 data flywheel 帶來的 ceiling 可能更高.

## 打開 Own-Weight Stack 的 Pandora's Box

[15:16](https://www.youtube.com/watch?v=bMMv0bZzONg&t=916s)

從單一 API call 走向自有 intelligence 後, 企業需要處理更多選擇:

- 選擇 open-weight base model.
- 設計 post-training recipe.
- 選擇或建立 open harness.
- 配置 tools, logic 與 context.
- 維護 inference infrastructure.
- 建立 data, eval 與 observability systems.
- 控制 production drift 與 release lifecycle.

Context 也是 performance 的重要來源. 影片提到幾種 context layer:

- Vector database.
- Enterprise knowledge graph.
- MCP-based connectors.
- 將特定 context 或知識編碼進 weights 的研究方法.

這些方式並非互斥. Dynamic context 適合經常變動, 需要 provenance 的知識. Encoding into weights 可能降低 retrieval latency 或改變模型行為, 但更新, 刪除, 稽核與資料權利會更困難.

## Own 與 Rent 的決策矩陣

以下是根據影片框架整理的實務問題:

| 問題 | 若答案為是, 傾向 Own | 若答案為否, 傾向 Rent |
| --- | --- | --- |
| 這項能力是否直接決定產品差異? | 是 | 否 |
| API cost 是否顯著侵蝕毛利? | 是 | 否 |
| Latency 是否為 P0? | 是 | 否 |
| 是否有合法, 專屬且高品質的 training signal? | 是 | 否 |
| Domain eval 是否顯示專用模型有優勢? | 是 | 否 |
| 供應商變更是否會威脅核心產品? | 是 | 否 |
| 團隊是否能維護 training 與 serving? | 是 | 否, 即使前面符合也應謹慎 |
| Workload 是否穩定到值得最佳化? | 是 | 否 |

常見答案可能不是全 own 或全 rent, 而是 hybrid:

```text
Frontier API for complex reasoning
  + Owned small model for high-frequency classification
  + Owned autocomplete model for low latency
  + Router selects by task, cost and SLO
```

## 落地檢查表

### Strategy

- 列出所有 AI capabilities, 不以單一 "AI stack" 籠統處理.
- 分別量測 cost, latency, quality 與 data sensitivity.
- 建立供應商失效或政策變動的 contingency plan.
- 先選一項高價值, 可量測的 capability 作為 pilot.

### Evals

- 建立與產品 outcome 相關的 domain evals.
- 同時比較 quality, latency, cost, safety 與 robustness.
- 保存 model, prompt, harness, data 與 infra version.
- 將 offline eval 與 production monitoring 連接起來.

### Team

- 指定單一 labs leader 與清楚 mission.
- 建立 research engineering, data 與 inference 能力.
- 讓團隊能直接接觸 product feedback, 不只提供平台服務.
- 從小團隊開始, 以可重現成果擴張.

### Technical roadmap

- 先建立 baseline 與 router, 再決定是否 training.
- 優先嘗試 harness, context 與 post-training.
- 只有 eval 證明必要時才向更深 training 移動.
- 為 model release 準備 canary, rollback 與 regression gates.
- 將 production data 的 consent, privacy 與 retention 納入設計.

### Legibility

- 發布可重現且有清楚限制的技術成果.
- 避免只發布 leaderboard number, 同時說明 eval scope.
- 平衡 external credibility 與 proprietary advantage.
- 讓 research narrative 與真正的產品成果一致.

## 核心觀念回顧

```text
Own your intelligence
├── Strategy
│   ├── Cost
│   ├── Speed
│   ├── Performance
│   └── Proprietary data and control
├── Team
│   ├── Small de novo lab
│   └── Research + engineering
├── Legibility
│   └── Research and technical marketing
└── Technical roadmap
    ├── Evals
    ├── Routing
    ├── Harness and context
    ├── Post-training
    ├── Deeper training when justified
    └── Production feedback and online learning
```

最值得帶走的三個原則:

1. Sovereign AI 不是全面自建, 而是逐項決定 intelligence 的 ownership boundary.
2. Evals 應先於 training, 沒有可靠量測就無法證明自建更好.
3. Own weights 提高控制與 customization ceiling, 也同時承擔 data, training, serving 與維運複雜度.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕可能誤辨公司, 人名與模型版本. 本筆記只在上下文足以確認時修正, 並刻意不依賴字幕中無法可靠辨識的 model version 作為選型依據.

這場演講是 Sequoia Capital 活動的開場與策略框架, 帶有明確的投資觀點和 rallying-call 性質. 影片沒有提供完整 cost model, benchmark methodology 或各公司案例的外部證據. Open-weight 與 closed-model 能力, 價格及產品功能變動很快, 實際決策應以自己的 evals, workload 與最新官方資訊為準.
