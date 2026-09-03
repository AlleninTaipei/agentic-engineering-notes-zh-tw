# Agentic Applications 實戰 Evals: 從 Traces, Failure Taxonomy 到 Experiments

- 影片: [Ship Real Agents: Hands-On Evals for Agentic Applications - Laurie Voss, Arize](https://www.youtube.com/watch?v=Xfl50508LZM)
- 頻道: AI Engineer
- 講者: Laurie Voss, Arize AI Developer Experience 負責人
- 發布日期: 2026-05-14
- 片長: 2:04:18
- Video ID: `Xfl50508LZM`
- 內容依據: YouTube 英文自動字幕 (`en`)

## 摘要

這場 workshop 用一個兩階段 financial analysis agent 示範完整 eval workflow. Agent 先搜尋公司資料, 再撰寫投資報告. 講者使用較便宜的 Claude Haiku 執行 agent, 使用較強的 Claude Sonnet 擔任 judge, 並以 Arize Phoenix 收集 OpenTelemetry 與 OpenInference traces.

核心流程不是先挑一個現成 eval, 而是先 instrument agent, 執行多樣化 queries, 人工閱讀 traces, 把 failures 依 root cause 分類, 再決定需要什麼 eval. Workshop 依序建立 deterministic code eval, Phoenix 內建 LLM-as-a-judge eval 與自訂 actionability rubric, 最後用 dataset 和 experiment 比較 prompt 變更.

最重要的反例發生在同一批 13 個 agent runs. Correctness evaluator 全部判定失敗, 因為 judge 不具備 2026 年即時金融事實. 改用只判斷報告是否忠於 research context 的 faithfulness evaluator 後, 13 筆全部通過. 這不代表 agent 的內容必然正確, 而是證明 eval 必須只判斷 judge 真正擁有證據的性質.

Workshop 也示範 eval 本身需要測試. 自訂 rubric 必須以 human-labeled golden dataset 做 meta-evaluation, 觀察 precision, recall 與各類 bias. 未驗證的 judge 只是把錯誤判斷自動放大.

## Vibe Checking 無法支援 Regression

[00:05:17](https://www.youtube.com/watch?v=Xfl50508LZM&t=317s)

許多團隊以幾筆 queries 測試 AI feature, 覺得輸出看起來合理就發布. 講者稱之為 vibes problem. 這種方法無法涵蓋 edge cases, adversarial inputs 或使用者未採用開發者預期詞彙的情況, 也不能在 CI 中反覆執行.

Agent 的 prompt 是共享控制面. 為了改善 tone 而修改一句指令, 可能同時讓 factuality, tool use 或 policy compliance 退步. 更換 model 也不只是能力升級, 不同 model 可能對相同 prompt 產生不同 execution behavior.

Evals 的基本作用是把「看起來不錯」轉成可重跑的變更判準:

```text
prompt, model or tool change
            ↓
run a representative dataset
            ↓
apply stable evaluators
            ↓
compare capability and regressions
            ↓
ship, revise or roll back
```

固定 expected string 通常不適合自然語言輸出, 因為多個不同答案可能同時正確. 但這不代表所有測試都必須使用 LLM judge.

## 三種互補的 Evaluation

### Code Evals

Code eval 是 deterministic Python, TypeScript 或其他程式. 它執行快, 成本低且可重現, 適合檢查:

- Output 是否為合法 JSON.
- Required fields 是否存在.
- 長度或 token count 是否在限制內.
- 是否含 forbidden phrase.
- 是否提到 requested entity.
- 價格或識別碼能否由 database 或 API 精確驗證.

缺點是容易過度依賴格式或字串. 當有效輸出空間很大時, code eval 可能把語意正確的答案誤判為失敗.

### LLM-as-a-Judge

LLM judge 使用另一個 model 按 rubric 判斷 semantic quality, 例如 factual correctness, faithfulness, tone 或 completeness. 它能輸出 score, label 與 explanation, 後者可用來理解失敗原因及尋找重複模式.

代價包括 latency, 費用與 judge 自身的 non-determinism. Judge prompt 和被測 agent prompt 一樣需要設計, 版本控制及 regression testing.

### Human Evaluation

Humans 能提供 domain judgment 並辨識尚未編碼的 failure modes, 但不能經濟地對所有 production traces 做持續判讀. 較適合的角色是:

- 定義 success criteria.
- 標記代表性 examples.
- 建立 golden dataset.
- 解決 LLM judge 與 domain expectation 的分歧.
- 定期抽樣發現新型 failures.

三種方法不是替代關係. Code eval 先攔截便宜且明確的錯誤, LLM judge 處理語意, humans 校準目標與未知風險.

## Agent Evals 比單一 LLM Call 更難

Agent 的結果來自多個相依 decisions. 加入一次 tool call 後, 至少需要分辨:

- 是否選對 tool.
- 是否傳入正確 parameters.
- 是否正確解讀 tool result.
- 是否在適當時機繼續或停止.
- 最後輸出是否達成 task outcome.

Multi-agent system 還要測 routing, subagent task understanding, handoff data 與 termination. 早期一步選錯, 後續 steps 即使各自合理也可能把錯誤擴大.

但 eval 也不能把 path 寫死. 更好的 model 可能用較少 tool calls 達到相同或更佳 outcome. 若測試規定必須依序呼叫 A, B, C, model 找到更短路徑時反而會被判失敗.

因此, 預設先測 outcome. 只有當某個中間行為本身具有安全或合規意義時, 才把它設為 hard requirement. 例如退款前驗證身分, 就不能因最終金額正確而忽略 verification step.

## Capability Evals 與 Regression Evals

講者用 hill climbing 解釋兩者關係:

- Capability eval: 針對 agent 尚未可靠具備的能力, 預期會有足夠 failures 可供改善.
- Regression eval: 保護已經能可靠完成的能力, 防止後續變更破壞它.

當 capability eval 達到目標後, 它就能加入 regression suite. 團隊再選下一項能力建立新的 hill.

```text
new capability eval
  -> inspect failures
  -> improve data, prompt or system
  -> reach acceptance threshold
  -> promote to regression eval
  -> select the next capability
```

這種區分也影響成本配置. Q&A 中, 講者建議壓縮代表性 regression set, 把較昂貴的 judge 或較密集測試留給當前 capability eval. Production traffic 通常執行 regression monitoring, 因為未改變的 capability experiment 沒有必要在每筆 live trace 上重跑.

## Trace 是 Eval 的原始材料

[00:18:44](https://www.youtube.com/watch?v=Xfl50508LZM&t=1124s)

Trace 記錄 AI application 在 runtime 的行為. Span 是其中一個 execution step, 例如 LLM call, tool call 或包含多個 child spans 的完整 agent turn.

每個 span 可包含:

- Input 與 output.
- Prompt 和 completion.
- Model identity.
- Tool invocation.
- Timing 與 latency.
- Token counts 與 cost.
- Parent-child relationship.
- Application-defined attributes.

Workshop 使用 Phoenix 接收 traces. OpenTelemetry 提供一般 observability 結構, OpenInference 在其上增加 prompt, completion, token 和 tool 等 LLM-specific semantics. 範例以 auto-instrumentation 收集 Claude Agent SDK 的資料, 並另外建立 parent span, 把 research 與 report-writing 兩個 agent turns 組成同一條 trace.

講者提醒, demo 使用適合即時顯示的 span processor. Production 高流量情境應使用 batching, 避免把 workshop 設定直接當成 deployment configuration.

## Demo Agent 與測試資料

[00:26:58](https://www.youtube.com/watch?v=Xfl50508LZM&t=1618s)

Financial analysis agent 有兩個 sequential turns:

```text
stock ticker + focus area
          ↓
research turn with web search
          ↓ shared conversation context
report-writing turn
          ↓
financial report
```

Agent 使用 Haiku, 因為成本與速度適合 workshop, 也較容易產生可供分析的 failures. 這是刻意構造的教學系統, 不是 production financial advisor.

測試包含 12 組 queries, 加上最初的 Tesla run 共 13 條 traces. Cases 涵蓋單一 ticker, 不同分析焦點, Apple 與 Microsoft comparative analysis, 資料較少的公司, 以及成熟公司與 growth stocks.

講者強調測試資料要涵蓋不同語言表達. 同一意圖可能被寫成正式分析要求, 也可能只是「這支股票現在能買嗎」. 還應加入不存在的 ticker, multi-part question, jailbreak 與 adversarial input.

理想來源是 production traces. 尚未上線或直接上線風險太高時, 可以用 LLM 生成 synthetic queries, 但仍需確認它們能代表真實使用者, 不能把 synthetic diversity 當成 production coverage.

## 先讀 Traces, 再寫 Eval

[00:38:12](https://www.youtube.com/watch?v=Xfl50508LZM&t=2292s)

在寫任何 evaluator 前, 團隊必須先定義 success. Workshop 的需求不是模糊的「深入」或「有幫助」, 而是 report 必須提供可採取行動的 buy, sell 或 hold recommendation, 並區分 historical summary 與 forward-looking analysis.

這個定義不能只由 engineer 決定. Product managers, domain experts, customer-facing teams 與 users 更接近真正 outcome, 應共同把「好」轉成 observable criteria.

人工閱讀 traces 後, workshop 找到幾種不同 failures:

### 寫到 Disk 而不是回傳結果

Apple case 完成 research 並產生 report, 卻因「write a report」指令含糊, 嘗試把 Markdown 寫入 disk. Notebook environment 沒有相應能力, 最終 output 失敗.

若只看 final response, 可能以為 research 或 report generation 失敗. 展開 trace 才能看到實際 root cause 是輸出位置與 permission mode.

### 有分析但沒有 Recommendation

Apple 與 Nvidia reports 提供資料和敘述, 卻沒有回答是否應該買入. 這是 requirement gap, 不是單純 factual error.

### 看似精確但無法驗證

Rivian report 提供非常具體的未來交付數字. 講者無法從現有 context 判斷它是否來自可靠 source, 因此標記為 suspicious, 而不是直接宣布 hallucination.

### Entity Scope 錯誤

Amazon report 全部聚焦 AWS, 沒有處理 Amazon 整體. 即使文字流暢, 它仍偏離 requested entity scope.

這些 cases 說明 final output 只是 symptom. Bad retrieval, tool misuse, missing instruction, unsupported claim 與 wrong conclusion 需要不同修正.

## 用 Frequency 和 Severity 排定 Failure 優先級

讀完代表性 traces 後, 應建立 failure taxonomy, 再計算各類頻率. 但頻率不能單獨決定優先順序. 罕見但造成重大安全或信任損失的錯誤, 可能比常見的小缺陷更重要.

可將初步優先級表示為:

```text
priority ≈ frequency × consequence severity × exposure
```

這是依講者說明做的編者整理, 不是 workshop 提供的正式公式.

講者以 Swiss cheese model 說明 layered defenses. 每一層 eval 都有盲點, 但 code checks, semantic judges 與 human review 的盲點不完全相同. 可靠性來自多層互補, 不是尋找一個完美 evaluator.

## 第一個 Code Eval: 是否提到 Stock Ticker

[00:49:52](https://www.youtube.com/watch?v=Xfl50508LZM&t=2992s)

Workshop 第一個 evaluator 只檢查 report 是否提到 requested stock ticker. Phoenix 中每個 log entry 是 span, 無 parent 的 spans 被選為 13 個完整 agent runs.

這個簡單 regex check 通過 11/13, 找到兩個問題:

- Tesla case 因寫檔流程失敗, 沒有產生預期 output.
- Amazon case 只談 AWS, 沒有提到 Amazon ticker.

這證明 code eval 不只適合 toy checks. 只要 grading rule deterministic, evaluator 也可以查 database 或 API 驗證 product price, identifier 或外部狀態.

但 code eval 應容許等價表達. 例如 2 hours, 120 minutes 與相同秒數都可能有效. 測試應聚焦 required outcome, 避免把單一路徑或單一字串格式誤當成 correctness.

## Correctness 0/13, Faithfulness 13/13

[00:57:51](https://www.youtube.com/watch?v=Xfl50508LZM&t=3471s)

Workshop 接著使用 Phoenix 內建 correctness evaluator. 每個 LLM-as-a-judge eval 包含:

- Judge model.
- Prompt template 或 rubric.
- Criteria.
- 被評估的 data.

結果是 13 筆全部判定 incorrect. 閱讀 explanations 後發現, 問題不一定在 agent. Judge 的知識無法驗證 2026 年即時或 forward-looking financial figures, 因此把超出自身 knowledge cutoff 的內容全部視為可疑.

團隊改用 faithfulness evaluator, 不再問「世界上的事實是否正確」, 而是問「report 是否只根據 research turn 提供的 context」. 因為 agent 已分成 research 與 writing 兩步, 第一階段輸出能直接成為 judge context.

Faithfulness 結果為 13/13. 這組對照的正確解讀是:

- Correctness eval 無權威資料可判斷當下金融事實, 因此 signal 無效.
- Faithfulness eval 能判斷 report 是否忠於給定 research, 因此 task 和 evidence 對齊.
- Faithfulness 通過不證明 research source 本身正確.
- 需要另一層 retrieval quality, source authority 或 external fact verification 才能涵蓋 end-to-end correctness.

選擇 evaluator 前, 應先問 judge 擁有什麼 evidence, 以及該 evidence 能支持哪種 claim.

## 自訂 LLM Judge 的五個組件

[01:04:35](https://www.youtube.com/watch?v=Xfl50508LZM&t=3875s)

Phoenix 沒有預先知道何謂 actionable financial report, 因此 workshop 建立自訂 actionability rubric. 講者整理五個組件:

1. Define the role: 提供 domain 與 evaluation task context.
2. State observable criteria: 不寫「good」或「helpful」, 改寫成可觀察條件.
3. Delimit input data: 清楚區分 user query, source context 與 agent response.
4. Add labeled examples: 同時展示符合與不符合條件的 outputs.
5. Constrain the output: 優先使用明確 labels, 避免含義不清的 1 到 10 評分.

Actionable criteria 包含具體 recommendation, forward-looking analysis 與風險解讀. Not actionable 則包括只重述 public data 而沒有 interpretation.

Examples 特別重要, 因為它們把抽象 instruction 轉成可模仿的 decision boundary. Labels 可使用 `actionable` 和 `not_actionable`, 並讓 judge 先產生 rationale, 再輸出受限 label.

講者不建議建立一個同時判斷 accuracy, tone, completeness, policy 與 format 的 god evaluator. 多維 rubric 難以校準, failure 時也不知道哪個 dimension 出錯. 應為每個重要 dimension 建立獨立 evaluator.

評分也應分成兩類:

- Guardrail: Failure 會阻止發布, 例如虛構價格或違反政策.
- North Star metric: 用來持續改善, 但單次 failure 不一定阻止發布.

## Meta-Evaluation: Judge 也必須被測試

[01:15:16](https://www.youtube.com/watch?v=Xfl50508LZM&t=4516s)

LLM judge 可視為 classifier. 要知道它是否可信, 必須把 predictions 和 human-labeled ground truth 比較.

Golden dataset 不只是一般 test inputs, 而是 domain experts 對 good and bad outcomes 的明確編碼. Human annotators 也需要相同 rubric, examples 與標記程序, 否則 ground truth 會混入未說明的個人偏好.

建議資料同時包含:

- Behavior 應該發生的 positive cases.
- Behavior 不應發生的 negative cases.
- 常見 production failures.
- Edge cases 和 adversarial cases.
- Reference solution 或可接受 outcome.

若只測「應搜尋 web」的 cases, agent 可能學會每次都搜尋. 加入不需要搜尋的 questions, 才能防止 evaluator 鼓勵無條件 tool use.

Golden set 也要分成 development 與 held-out evaluation subsets, 避免 rubric 或 prompt 只記住反覆使用的 examples. Workshop 建議的 75/25 split 是示意性做法, 實際比例取決於資料量與風險.

## Precision, Recall 與 Judge Bias

[01:24:27](https://www.youtube.com/watch?v=Xfl50508LZM&t=5067s)

Meta-evaluation 應檢查 judge 的 false positives 與 false negatives:

- Precision 關心被標成 failure 的 cases 中, 有多少真的是 failure.
- Recall 關心所有真實 failures 中, judge 成功抓到多少.

應最佳化哪一項取決於 consequence. 若漏掉 failure 會直接傷害使用者, 通常提高 recall 並接受更多人工複查較合理. 若 false alarm 本身會造成重大損害, precision 可能更重要.

常見 LLM judge biases 包括:

- Position bias: 偏好先出現或後出現的 option.
- Length bias: 偏好較長 response.
- Confidence bias: 被語氣肯定但證據薄弱的文字影響.
- Self-preference bias: 偏好同一 model family 產生的 output.

可以交換 option 順序, 按 response length 和 task category 分析 judge accuracy, 或使用不同 provider 的 model 交叉判讀. Pairwise comparison 通常也比沒有清楚 anchor 的 1 到 10 rating 更穩定.

Workshop 提及 human annotator error, inter-rater reliability 與 Anthropic CoreBench 的數字, 但影片沒有提供可直接查核的研究或完整 benchmark reference. 這些數字不應脫離原始研究作為通用常數.

## Failure 應該看起來合理

[01:30:40](https://www.youtube.com/watch?v=Xfl50508LZM&t=5440s)

若 domain reviewer 看到一個 failed case, 卻無法說明 agent 哪裡錯, 問題可能在 task 或 evaluator. 講者引用 Anthropic 對 CoreBench 的檢查案例: Exact expected value 與更高精度 numerical answer 不一致, 被 evaluator 當成錯誤. 修正多項 eval defects 後, 影片稱 Opus score 從 42% 變為 95%.

這個案例的重點不是特定 model 分數, 而是 benchmark failure 需要被 audit. Eval output 不能因為具有數字就自動取得權威性.

可採用的 review question 是:

```text
Given the task, evidence and rubric,
can a domain reviewer explain exactly why this case failed?
```

若答案是否定的, 應先檢查 dataset, parser, reference answer, tolerance 和 rubric, 再修改 agent.

## Dataset 與 Experiment 把 Eval 變成改善迴圈

[01:32:03](https://www.youtube.com/watch?v=Xfl50508LZM&t=5523s)

只有 scores 仍不足以證明 prompt fix 有效. Workshop 把六個 actionability failures 加入 dataset, 再依 judge explanations 修改兩段 prompts:

- Research prompt 要求 financial ratios, 近期消息與 current price data.
- Writing prompt 明確要求 buy, sell 或 hold recommendation.

每項修改都對應已觀察到的 failure, 而不是無方向調整措辭. Experiment 使用同一組 inputs 與同一 evaluator, 只改 agent prompt. 這提供 controlled comparison.

Demo 中, 原先 failing cases 在修改後全部通過. 講者明確說這種 one-shot improvement 是為 workshop 節省時間, 不能代表 production 的一般改善幅度.

Experiment 不必每次執行整個 agent. 若問題只在 tool selection, 可以抽出該 step 單獨測試, 降低 latency 與費用. 達到局部目標後, 再回到完整 dataset 執行 end-to-end regression, 檢查有沒有 overfitting 或 collateral damage.

## 樣本數與 Shipping Confidence

[01:40:33](https://www.youtube.com/watch?v=Xfl50508LZM&t=6033s)

講者用 defect rate 的 confidence interval 說明 sample size tradeoff. 假設 200 個 samples 觀察到 3% failures, 影片給出的 95% interval 約為 0.6% 到 5.4%. 增加到 400 samples 後, interval 約縮小到 1.3% 到 4.7%, 才能較有把握低於 5% threshold.

Workshop 提供的 rough guidance 是:

- 12 到 20 examples: 取得方向性 signal.
- 200 到 400 examples: 較適合 shipping decision.

這些數量不是固定門檻. 所需 samples 取決於 baseline rate, 容許誤差, confidence level, class imbalance, 分群方式, run-to-run variance 和風險. Rare catastrophic failures 也不能只靠隨機抽樣保證覆蓋, 需要 targeted adversarial cases.

## 改善順序: Data 先於 Prompt 和 Model

[01:42:04](https://www.youtube.com/watch?v=Xfl50508LZM&t=6124s)

Workshop 建議依 impact hierarchy 投入資源:

1. Data quality: 修正錯誤 sources, stale knowledge 或 retrieval failure.
2. Prompting: 加入 examples, explicit constraints 與 task instructions.
3. Model selection: 當能力不足無法靠 prompt 解決時再升級 model.
4. Hyperparameters: Temperature 和 top-p 通常不是最高影響項目.

這是講者的 practitioner heuristic, 不是所有系統都適用的實驗定律. 真正的優先順序仍應由 traces 和 ablation results 決定.

Evals 也能在 feature 前建立. 先定義「退款前一定驗證身分」的 evaluator, 再讓 agent 攀爬 capability hill, 相當於 agentic system 的 eval-driven development.

## Production Monitoring 與 Cost-Aware Evaluation

[01:46:18](https://www.youtube.com/watch?v=Xfl50508LZM&t=6378s)

上線後可以抽樣 production traffic 持續執行 eval, 監測:

- Model 或 agent quality drift.
- Product 與 use-case changes.
- 新的 adversarial patterns.
- Cost, latency 與 tool-call inflation.

Workshop 區分 Phoenix 與 Arize AX. Phoenix 是本場使用的 open-source observability and evaluation platform. Continuous production evaluation, enterprise compliance, SSO, large-scale storage, session-aware tracing, monitoring 與 dashboards 等能力, 部分被講者定位在商業產品 Arize AX. 實作前應查閱當前產品文件, 不能假設 workshop 當時的 feature boundary 永久不變.

Cost-normalized accuracy 將 quality 和 inference cost 一起考量. 影片舉例, 92% accuracy 且每 query 2 cents 的系統, 可能比 95% accuracy 且每 query 15 cents 更符合產品價值. 是否值得取決於錯誤後果, 不能只比較百分比.

## Pass@k 與 Pass^k 衡量不同可靠性

[01:48:45](https://www.youtube.com/watch?v=Xfl50508LZM&t=6525s)

講者區分兩種多次嘗試指標:

- `pass@k`: K 次內至少成功一次.
- `pass^k`: K 次全部成功.

Coding assistant 若能重試並由 tests 驗證, `pass@k` 可能符合需求. Customer support agent 面對每位顧客都必須穩定, 偶爾一次嚴重錯誤就不可接受, 因此更接近 `pass^k` 的問題.

選擇 reliability metric 前, 應先理解產品是否允許 retry, 是否有 verifier, 以及單次 failure 是否可逆.

## 最小可執行 Eval Workflow

講者最後建議從小規模開始. 可整理為以下實作順序:

1. Instrument agent, capture complete traces.
2. 人工閱讀至少一批真實 outputs 與 intermediate steps.
3. 和 stakeholders 定義 observable success criteria.
4. 建立 failure taxonomy, 以頻率和後果排序.
5. 先寫一個 cheap deterministic code eval.
6. 為需要 semantic judgment 的單一 dimension 建 LLM evaluator.
7. 加入清楚 criteria, positive/negative examples 與 constrained labels.
8. 建立 human-labeled golden dataset, 評估 precision, recall 和 bias.
9. 保存 prompt, rubric, model, dataset 與 evaluator versions.
10. 把 failures 放入 experiment dataset, 每次只比較明確變更.
11. 局部改善後執行完整 regression suite.
12. 上線後持續抽樣 traces, 把新 failure 轉為明天的 test case.

完整 loop 是:

```text
instrument
    ↓
trace real behavior
    ↓
read and categorize failures
    ↓
evaluate with layered methods
    ↓
human-validate the evaluators
    ↓
run controlled experiments
    ↓
improve agent and repeat
```

## Production Eval 檢查表

### Task 與 Dataset

- Success 是否可被 domain expert 清楚描述?
- Dataset 是否包含常見, 邊界, negative 與 adversarial cases?
- Test language 是否反映實際 users 的詞彙?
- 是否保存 reference solution 或 acceptable outcome?
- Golden set 是否有 held-out subset?
- Production failures 是否持續加入 regression set?

### Trace 與 Root Cause

- 是否能連結完整 agent turn, LLM calls 與 tool calls?
- 每個 span 是否保留 model, tokens, latency, cost 與 attributes?
- Failure 是 data, tool, reasoning, output 還是 evaluator 問題?
- 是否依 category 聚合 frequency?
- 是否用 consequence 而非只有 frequency 排序?

### Evaluator

- 能用 deterministic code 判斷的條件, 是否避免使用 LLM?
- Judge 是否真的擁有判斷該 claim 所需 evidence?
- 每個 evaluator 是否只處理一個 dimension?
- Criteria 是否 observable, 並附正反 examples?
- Labels 是否清楚, 避免沒有 anchor 的連續評分?
- Judge 是否經 human-labeled data 做 meta-evaluation?
- 是否監測 position, length, confidence 和 self-preference bias?

### Experiment 與 Release

- Agent prompt, judge prompt, models 和 dataset 是否 versioned?
- 每項 change 是否對應一個已觀察 failure?
- 比較時是否保持 inputs 和 evaluators 相同?
- 是否執行多次 runs 評估 non-determinism?
- Capability improvement 是否同時通過 regression suite?
- Shipping threshold 是否反映 confidence interval 與 failure severity?
- Production 是否具抽樣 monitoring, rollback 與 incident feedback path?

## 核心結論

這場 workshop 最有價值的不是 Phoenix API 操作, 而是 evaluation 的診斷順序:

1. 不先假設什麼會壞, 而是 instrument 並閱讀 traces.
2. 不只標記 output 好壞, 而是找出 root cause 和重複 failure pattern.
3. 不因 evaluator 產生數字就相信它, 而是驗證 judge 的 evidence, precision, recall 與 bias.
4. 不用新的 prompt 再做一次 vibe check, 而是以固定 dataset 和 evaluator 執行 controlled experiment.

Correctness 0/13 與 faithfulness 13/13 的對照濃縮了整場課程. Eval 不是對 output 加一個分數, 而是明確定義「要判斷哪個性質, 憑什麼證據, 錯誤代價是什麼」. 若這三件事沒有對齊, 自動 evaluation 只會讓錯誤信心更快進入 CI.

## 時效性與限制

本筆記依 YouTube 英文自動字幕與 18 個正式 chapters 整理. 字幕將 Arize, Phoenix, Claude, OpenTelemetry, OpenInference, code eval 等名稱多次誤辨, 本筆記只在上下文足以確認時修正.

講者 Laurie Voss 是 Arize AI Developer Experience 負責人, 對 Phoenix workflow 具直接產品經驗, 同時也有推廣 Arize 產品的商業誘因. Workshop 的 code, 13 條 traces, eval results 與 prompt experiment 是具體 demo, 但不是獨立 production study.

Demo 刻意使用能力較弱的 model, 少量 cases 與預先挑選的 failures. Human labels 在 meta-evaluation 示範中是隨機指定, 講者也明確承認該資料不能支持真實 precision/recall 結論. 最後的 one-shot prompt improvement 是教學壓縮, 不代表一般 production agent 能同樣改善.

關於 human annotator error, inter-rater reliability, CoreBench score, sample-size guidance 和多家公司的 eval practices, 影片沒有附完整研究方法或可直接驗證來源. 採用這些數字前應回到原始研究或以自身資料重新計算.

影片發布於 2026-05-14. Claude models, Claude Agent SDK, Phoenix APIs, OpenInference integrations 與 Phoenix/Arize AX feature boundary 都可能改變. 實作時應以目前版本文件, 真實 traces 和自己的 risk threshold 驗證.

