# Coding Agent 做 AI Systems Engineering: Skills、Kernels 與自動研究實驗室

- 影片: [Your Coding Agent Should Do AI System Engineering - Ben Burtenshaw, Hugging Face](https://www.youtube.com/watch?v=JomVvNDjGb8)
- 頻道: AI Engineer
- 講者: Ben Burtenshaw, Hugging Face
- 發布日期: 2026-05-21
- 片長: 18:25
- Video ID: `JomVvNDjGb8`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

Ben Burtenshaw 主張 coding agents 已能參與過去需要高度專業化的 AI systems engineering。影片以三個逐步提高自主程度的案例說明:

1. 人與 agent 互動式撰寫、測試並發布 CUDA kernels。
2. Agent 從單一要求完成 model fine-tuning workflow。
3. 多個 agents 平行執行研究假設、訓練實驗、審查與結果報告。

支撐這些案例的不是單一強大 prompt, 而是 agent 可操作的工程環境:

- 由專案維護者提供、可版本控制的 skills。
- 標準化 repositories 與明確的 compatibility metadata。
- 可執行的 tests、benchmarks 與 experiment metrics。
- 可被 agent 直接讀寫的 storage、compute 與 tracking primitives。
- 共享而結構化的 experiment state。

影片的具體價值在於展示 skills 如何把專業任務從 zero-shot 轉成帶有範例與程序的 few-shot workflow, 並說明 multi-agent research lab 的角色與資料流。限制是多數結果來自 demo 與講者陳述, 沒有完整公開對照實驗、成本、失敗率或長期無人監督可靠性。

## 三個逐步提高難度的任務

[00:00](https://www.youtube.com/watch?v=JomVvNDjGb8&t=0s)

講者將案例稱為三個 game bosses:

| Level | 任務 | Agent 自主程度 | 主要工程條件 |
| --- | --- | --- | --- |
| 1 | 撰寫並發布 CUDA kernel | 人與 agent 互動 | Hardware benchmark、compatibility、kernel repository |
| 2 | Fine-tune LLM | 從單一 prompt 啟動 | Dataset、training skill、Hub compute 與 publishing workflow |
| 3 | Multi-agent auto research | 多角色長時間平行執行 | Hypothesis queue、isolated branches、jobs、metrics 與 reporter |

這三個案例的共同條件是結果可以被驗證。Kernel 有 correctness test 與速度 benchmark, fine-tuning 有 evaluation score, training research 則有固定 objective 與 experiment metric。

「可驗證」縮小了 agent 自主工作的風險。若工作成果只有主觀品質, 同一種自治程度不一定可靠。這是依影片案例整理出的編者判斷。

## Boss 1: 撰寫與發布 CUDA Kernels

[01:59](https://www.youtube.com/watch?v=JomVvNDjGb8&t=119s)

Kernel 是 GPU 執行特定計算工作的程式。Custom kernel 可以針對硬體與數學操作最佳化, 但需要同時處理程式語言、GPU architecture、correctness、benchmark 與安裝相容性。

講者將 deep learning efficiency 分為三部分:

| 類型 | 內容 |
| --- | --- |
| Compute | Matrix multiplication 等實際數學運算 |
| Memory | Tensor 在不同層級記憶體間移動的成本 |
| Overhead | Python runtime、PyTorch dispatch 等額外成本 |

[03:48](https://www.youtube.com/watch?v=JomVvNDjGb8&t=228s)

講者表示, 許多 GPU workload 的瓶頸不是 compute, 而是 memory movement。以 H100 為例, 他對比約一 petaflop computation 與每秒約 3 TB memory bandwidth, 用來說明 GPU 可能等待資料移動。

Custom kernel 的一個方向是提高 arithmetic intensity, 也就是每次讀寫資料時完成更多運算。FlashAttention 被用作知名案例。這並不表示所有 workload 都受 memory bandwidth 限制, 實際 bottleneck 仍需 profiler 與 benchmark 確認。

## Kernel Repository 是 Distribution Boundary

[05:03](https://www.youtube.com/watch?v=JomVvNDjGb8&t=303s)

產生有效 kernel 只是第一步。若無法描述相容硬體、CUDA 與 software versions, 也無法安全整合進 inference engine。

Hugging Face 的 `kernels` library 將 kernel 發布為 Hub repository。Project metadata 描述支援的 hardware 與 software compatibility, 讓 kernel 能像 model 一樣被版本化與分發。

```text
kernel source
  + correctness tests
  + benchmark scripts
  + hardware compatibility
  + CUDA / software versions
  + versioned repository
          ↓
discoverable and reusable artifact
```

這個結構對 agent 很重要。Agent 不只需要產生 code, 還需要一個能判斷成果是否正確、適用於哪個環境及如何交付的 repository contract。

## Skills 將 Zero-Shot 轉成 Few-Shot Workflow

[06:08](https://www.youtube.com/watch?v=JomVvNDjGb8&t=368s)

講者把 skill 簡化為 file-based context。檔案可以按需開啟、關閉、版本控制與共同維護, 其中包含任務說明、範例、references、tests 與 benchmark scripts。

對 kernel 任務而言, skill 不是只告訴 agent「寫 CUDA」, 而是提供:

- 如何建立與使用 kernel 的範例。
- 如何執行 correctness test。
- 如何在目標硬體 benchmark。
- 如何組織與發布 repository。

講者將此描述為把 zero-shot task 轉為 few-shot workflow。專業知識沒有消失, 而是由即席的人類指導轉移到可版本化、可重用的工程資產中。

Hugging Face 採用兩種 skill ownership:

- Project-managed skills: 由專案維護者管理, 強調穩定與相容性。
- Experimental skills repository: 容納較新的做法與實驗。

這個邊界讓使用者知道 skill 的成熟度與責任歸屬, 避免將所有 instructions 視為同等可靠。

## Kernel Demo 與 94% Speedup

影片以 Qwen 3 8B 在 H100 上的 kernel 作為案例。講者表示其 benchmark 得到 94% speedup, 並強調這不是 state-of-the-art result, 而是利用特定 model 與 hardware 尚未最佳化的 compatibility gap 找到 low-hanging fruit。

這個數字不能直接解讀為模型整體 inference 加速 94%。字幕沒有交代 baseline implementation、operation、input shapes、precision、warm-up、correctness tolerance 或 end-to-end benchmark。它只支持「講者展示的特定 kernel benchmark 有顯著改善」這個較窄結論。

## 用 Upskill 評估 Skills 與 Models

[08:37](https://www.youtube.com/watch?v=JomVvNDjGb8&t=517s)

講者介紹開源 library `upskill`, 用來產生或整理 skill evaluation, 並比較不同 models 在相同 skill 下的表現與 token 使用。

概念流程如下:

```text
skill + representative tasks
          ↓
evaluation
          ↓
run multiple models
          ↓
compare accuracy and token use
          ↓
revise skill or choose model
```

影片展示不同 models 在 accuracy 與 tokens 上的比較, 但字幕沒有提供完整 dataset、sample size、scoring method 或數值表。因此可移植的做法是「skill 應具備自己的 eval」, 不是影片畫面中的 model ranking。

## Boss 2: 從單一要求完成 Fine-Tuning

[09:26](https://www.youtube.com/watch?v=JomVvNDjGb8&t=566s)

第二個案例讓 agent 根據 model、dataset 與目標完成 fine-tuning。Workflow 與 Hugging Face Hub、CLI skills、remote GPU jobs 及 model publishing 整合。

這段示範時間較短。字幕對 model size 與部分 library 名稱辨識不一致, 影片 metadata 與字幕也存在差異。因此本筆記不將特定 model size、benchmark score 或成本視為已充分確認的結果。

可確認的架構重點是, fine-tuning 並非由 agent 憑空發明 training pipeline, 而是 agent 操作已存在的標準工具與 skills:

```text
user objective
  -> select model and dataset
  -> load maintained training skill
  -> configure remote compute
  -> run training
  -> evaluate
  -> publish artifacts to Hub
```

若要把這個 demo 用於 production, 還需補上 dataset license、secret management、budget limit、checkpoint policy、evaluation leakage、failed-job recovery 與 approval boundary。

## Boss 3: Multi-Agent Auto Research Lab

[10:16](https://www.youtube.com/watch?v=JomVvNDjGb8&t=616s)

第三個案例受到 Andrej Karpathy `autoresearch` project 啟發。原始形式由單一 coding agent 修改 training script、執行實驗, 再依 metric 繼續迭代。Ben 將它拆成多個角色, 讓研究工作可以平行化。

| Role | 責任 |
| --- | --- |
| Researcher | 搜尋 papers, 將想法轉成 hypotheses |
| Planner | 維護 experiment queue 與可執行 jobs |
| Worker | 修改 training script, 啟動實驗並提交 patch |
| Reviewer | 排除 duplicate、stale 或不合格的 ideas, 檢查結果 |
| Reporter | 監控 jobs, 彙整 metrics 並更新 dashboard |

影片前段稱為四種類型, 後續實作也出現 reviewer。較合理的理解是 reviewer 可能屬於 planning 或 reporting flow, 但字幕不足以確認其正式角色模型。本筆記依 demo 中實際出現的五項責任列出。

## Research System 的資料流

[12:09](https://www.youtube.com/watch?v=JomVvNDjGb8&t=729s)

實驗系統以 Git repository 作為共享工作空間:

- Main branch 保存目前 training scripts 與 score data。
- Original training script 保留為 baseline。
- Workers 在各自 branches 實作 hypotheses。
- Hugging Face Jobs 提供 remote compute。
- 成功或失敗的 experiment state 回到共享資料結構。
- Reporter 將結果寫入 Trackio。

```text
papers / prior results
          ↓
researcher → hypotheses
          ↓
planner → experiment queue
          ↓
reviewer → reject duplicate or stale ideas
          ↓
workers on isolated branches
          ↓
HF Jobs → metrics + patches
          ↓
reporter → Trackio dashboard / alerts
          ↓
shared state for next iteration
```

講者表示同一概念已在 OpenCode、Codex 與 Claude 上實作。重點不在特定 agent client, 而是 roles、templates、repository state 與 compute jobs 形成可替換的 protocol。

## 用 Templates 限制 Agent 間的溝通

[13:40](https://www.youtube.com/watch?v=JomVvNDjGb8&t=820s)

OpenCode demo 中, planner 取得目前狀態、既有 jobs、成功與失敗結果, 以及允許修改的 hyperparameters, 再提出有限數量的 single-change experiments。

Reviewer 接收同一份共享狀態, 排除重複或過時的 jobs。Agents 以 tables 與固定 templates 交換資訊。講者也承認 demo 的 tables 可能過度冗長, 可以進一步縮減。

這裡的重要原則是 agents 不應只靠自由文字彼此聊天。Experiment proposal 至少應包含:

- Hypothesis。
- Single intended change。
- Baseline 與 target metric。
- Required compute。
- Current status。
- Result and artifact references。
- Rejection or failure reason。

以上 schema 是依影片 template 概念整理的編者建議, 不是講者展示的完整欄位定義。

## Trackio: Dashboard 底下是開放資料層

[15:28](https://www.youtube.com/watch?v=JomVvNDjGb8&t=928s)

Trackio 用於保存與呈現所有 experiment runs。講者認為它適合 agents, 因為底層資料採開放格式, 字幕指出主要是 Parquet。Agent 可以不經 dashboard, 直接讀取資料並建立 Gantt chart 或其他視圖。

系統也能記錄 events、warnings 與 labels, 依條件發送通知。Human operator 可從 Hub 查看 jobs, 從 Trackio 查看跨 runs 的 metrics 與異常。

這呈現兩個不同界面:

- Human-facing dashboard: 提供 runs、events、charts 與 alerts。
- Agent-facing data layer: 提供可查詢、可組合的結構化資料。

若 dashboard 只有封閉 UI 或高度抽象 API, agent 能進行的分析會受限。講者因此偏好「公開底層 primitives, 再往上建立抽象層」。

## Open Primitives 與 Standard Repositories

[16:45](https://www.youtube.com/watch?v=JomVvNDjGb8&t=1005s)

影片最後將整套做法歸納為兩點:

1. Agents 適合操作開放且可組合的 primitives。
2. Standard repositories 讓 code、metadata、tests、metrics 與 artifacts 形成穩定 contract。

Abstracted API 仍有價值, 但若使用者與 agent 永遠無法存取底層 data or artifacts, abstraction 也會成為能力上限。

Hugging Face Hub 在這套架構中同時提供 storage、tracking 與 compute。這是 Hugging Face 員工對自家平台的第一手說明, 也帶有明確的產品推廣立場。「Hub 已經準備好承載此類 workload」應視為講者主張, 而不是影片中經受控 benchmark 證明的市場比較。

## 實作檢查表

- 任務成果是否有 deterministic test 或可量測 metric?
- Skill 是否由最了解 project contract 的 maintainer 負責?
- Stable 與 experimental skills 是否明確分開?
- Skill 是否包含 examples、references、tests 與 benchmark scripts?
- Skill eval 是否使用代表性的 tasks, 而不是單一成功 demo?
- Repository 是否記錄 hardware、driver、CUDA、framework 與 model compatibility?
- Kernel benchmark 是否同時驗證 correctness 與 numerical tolerance?
- Speedup 是否區分 microbenchmark 與 end-to-end application latency?
- Fine-tuning job 是否具 budget、timeout、checkpoint 與 retry policy?
- Dataset 與 model artifacts 是否記錄版本、license 與 provenance?
- 每個 experiment 是否只改一個主要變數, 方便歸因?
- Workers 是否使用獨立 branches、worktrees 或 sandboxes?
- Planner 是否能看到成功、失敗與已拒絕的 hypotheses?
- Reviewer 是否防止 duplicate、stale 與不可比較的 experiments?
- Shared state 是否由 schema 驗證, 並保留 append-only run history?
- Reporter 是否連回 logs、patches、artifacts 與 exact environment?
- Human operator 是否能設定 compute ceiling、停止條件與 notifications?
- Dashboard 底層資料是否可由 agent 直接查詢及重組?

## 核心結論

Coding agent 能接手更深的 systems engineering, 前提不是把專業知識從流程移除, 而是把它編碼成可操作的環境:

```text
maintainer knowledge
  -> versioned skill
  -> standard repository
  -> executable tests and metrics
  -> bounded agent workflow
  -> inspectable artifacts
```

Skills 提供 few-shot procedure, repository 定義 artifact contract, benchmark 提供 feedback, open data layer 則讓 agents 與人類都能檢查結果。Multi-agent architecture 的價值也不只是增加平行度, 而是把 hypothesis formation、execution、review 與 reporting 分成可觀察的責任。

最能一般化的三項做法是:

1. 優先選擇可驗證的專業任務, 再提高 agent 自主程度。
2. 讓 project maintainers 維護 skills, 並為 skills 建立 evals。
3. 將 experiments 保存成版本化 code、結構化 metrics 與可追溯 artifacts。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。字幕對 library、project 與 model 名稱有辨識誤差。影片 metadata 將 fine-tuned model 寫為 Qwen3 0.6B, 字幕則辨識為 Qwen 3 6B, 本筆記不判定何者正確。字幕出現近似 `Onslaught` 的 library 名稱, 因無法只靠上下文可靠確認, 本筆記未將它當成正式名稱。

講者任職 Hugging Face, 參與 GPU Mode 並實作影片中的 projects。內容包含直接 practitioner 經驗、公開 repository workflow 與可執行工具, 同時也推廣 Hugging Face Hub、Kernels、Jobs、Skills、Upskill 與 Trackio。

影片沒有公開完整 kernel benchmark configuration、skill eval dataset、fine-tuning evaluation protocol、multi-agent experiment success rate、compute cost、failed-run distribution 或 unattended runtime reliability。94% speedup、H100 bandwidth 與其他結果均為講者陳述, 本筆記未獨立重跑或檢查投影片中的 repositories。
