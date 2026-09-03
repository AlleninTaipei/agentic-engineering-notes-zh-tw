# Reinforcement Learning 與 GRPO 實戰: Reward Functions, PPO, Unsloth 和 Dynamic Quantization

- 影片: [[Full Workshop] Reinforcement Learning, Kernels, Reasoning, Quantization & Agents - Daniel Han](https://www.youtube.com/watch?v=OkEGJ5G3foU)
- 頻道: AI Engineer
- 講者: Daniel Han, Unsloth
- 發布日期: 2025-07-19
- 片長: 2:42:27
- Video ID: `OkEGJ5G3foU`
- 內容依據: YouTube 英文自動字幕 (`en`)

## 摘要

這場 workshop 從 LLM 的訓練階段切入, 逐步解釋 REINFORCE, PPO 與 GRPO, 再用 Unsloth, vLLM 和 Qwen 3 base model 示範 reinforcement fine-tuning. 核心不是背誦演算法, 而是理解三個實務問題: 如何定義可驗證的 reward, 如何讓 rollouts 保持足夠多樣性, 以及如何避免所有樣本都得到相同 reward 而失去學習訊號.

GRPO 的價值在於省去 learned value model. 對同一 prompt 產生一組 responses, 計算各自 reward, 再以組內平均和標準差形成 relative advantage. 這降低了 PPO 的模型與記憶體負擔, 但沒有消除 reward design, sampling cost 或 reward hacking.

Workshop 的 demo 也揭示一項重要取捨. Base model 理論上可以直接做 RL, 但若它幾乎無法產生符合格式或答對問題的樣本, 整組 rollout 會得到相同 reward. 講者因此先用少量 SFT examples 做 priming, 再進行 GRPO. SFT 在此不是最終解法, 而是提高探索到有效軌跡的機率.

最後一段介紹 dynamic quantization. 重點不是所有 layers 一律壓到相同 bit width, 而是衡量 activation 和 weight quantization error, 對敏感 layers 保留較高精度. 影片中的壓縮率, 記憶體數字與品質結果主要來自講者及 Unsloth 的展示, 應視為待重現的產品方自述, 不是跨模型的保證.

## LLM 訓練不是單一步驟

[00:09:47](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=587s)

講者以 Yann LeCun 的 cake analogy 回顧 LLM pipeline, 並把現代流程整理為:

```text
random initialization
  -> pretraining
  -> mid-training and long-context extension
  -> supervised fine-tuning
  -> preference optimization or RLHF
  -> reinforcement fine-tuning with verifiable rewards
```

Pretraining 從大規模 token prediction 取得一般能力. Mid-training 可能提高高品質資料權重, 加入 domain data, 並延長 context window. SFT 教導模型遵守指令和輸出格式. Preference optimization 再調整回答風格與人類偏好.

DeepSeek R1-Zero 類型的結果說明 base model 可以直接透過可驗證 reward 學出推理行為. 但「可以直接訓練」不等於「對每個資料集都最有效率」. 若 base model 的成功率接近零, 少量 SFT priming 通常能減少 RL 初期的無效探索.

## 把 Agent 問題寫成 Reinforcement Learning

[00:16:56](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=1016s)

傳統 RL 反覆經歷 environment state, action, reward 和下一個 state. 在 workshop 的單輪 LLM 簡化模型中:

- Prompt 對應 state.
- 完整 reasoning 與 answer 對應 action 或 trajectory.
- Verifier 對輸出計分, 形成 reward.
- Training objective 提高高 reward 輸出的機率.

這個映射容易理解, 但會隱藏 multi-turn agent 的複雜度. 真實 agent 具有多個 observations, tool calls 和 delayed outcomes. 若只對最後答案給分, credit assignment 仍然困難.

數學與程式題適合早期 RLVR, 因為答案可由 calculator, test suite 或 symbolic checker 驗證. 開放式寫作, UX 品質或「好不好玩」沒有單一可靠 verifier. 使用 LLM judge 可以擴大範圍, 但也把 judge bias 和可被操弄的弱點帶進 reward loop.

## Reward Model 與 Reward Function

[00:48:12](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=2892s)

Reward model 是從人類偏好資料學得的模型. 它能評估難以用規則描述的輸出, 代價是需要標註資料, 額外推論資源, 並可能把偏好資料的偏差轉成訓練目標.

Reward function 則直接以可執行規則計分, 例如:

- 正規表示式檢查輸出格式.
- Unit tests 或 sandbox execution 驗證程式碼.
- Exact match 驗證答案.
- Numerical distance 給予部分分數.
- 檢查 required delimiters 或 structured fields.

RLVR 通常指以 verifiable reward function 取代 learned reward model. Verifiable 不代表無風險. 只要 reward 與真正目標存在落差, model 就可能學會滿足 checker, 卻沒有完成使用者想要的工作.

## REINFORCE 的核心直覺

[00:51:22](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=3082s)

Policy gradient 的簡化直覺是, 提高帶來好結果之 actions 的 log probability, 降低帶來壞結果之 actions 的機率:

```text
gradient J(theta) ~= E[gradient log pi_theta(a | s) * R]
```

直接使用 reward 的 variance 很高. 加入 baseline 後, 真正控制更新方向的是 advantage:

```text
A(s, a) = R(s, a) - V(s)
```

高於預期的 action 得到正 advantage, 低於預期的 action 得到負 advantage. Baseline 不改變想要最大化 expected reward 的目標, 但可降低梯度估計的 variance.

影片以教學直覺簡化了推導. 講者在部分 likelihood-ratio 問答中也明確保留. 若要實作 optimizer 或核對符號, 應以原始 REINFORCE, PPO 和 GRPO 論文為準, 不應把本段視為完整數學證明.

## PPO 如何限制更新幅度

[01:08:50](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=4130s)

PPO 通常需要 trainable policy, 用來產生資料的 old policy, reference policy, reward 訊號和 learned value model. 它比較新舊 policy 對同一 action 的機率:

```text
r_t(theta) = pi_theta(a_t | s_t) / pi_old(a_t | s_t)
```

Clipped objective 把 ratio 限制在信任區間附近. 若一次 update 試圖大幅提高或降低某個 action 的機率, clipping 會限制額外收益:

```text
L_clip = E[min(r_t * A_t, clip(r_t, 1 - epsilon, 1 + epsilon) * A_t)]
```

Reference model 和 KL penalty 則抑制 policy 偏離原模型太遠. 這些機制不是保證穩定, 而是降低單次更新過大, catastrophic drift 或針對有限 reward 過度最佳化的風險.

PPO 的主要工程成本之一是 value model. 它需要訓練和保存另一組權重, 並增加 GPU memory 與系統複雜度.

## GRPO 用組內比較取代 Value Model

[01:16:29](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=4589s)

GRPO 對每個 prompt 產生 `G` 個 rollouts, 分別計算 reward, 再以 group statistics 建立 advantage. Workshop 使用的簡化表示為:

```text
for one prompt:
  responses = sample(policy, count=G)
  rewards = verify(responses)

A_i = (r_i - mean(rewards)) / (std(rewards) + epsilon)
```

高於同組平均的 response 被強化, 低於平均者被抑制. 因為 baseline 直接來自同一 prompt 的 sampled group, 不需要 learned critic 或 value model.

這帶來幾個直接取捨:

- 優點: 少一個模型, 降低 memory 和 implementation burden.
- 成本: 每個 prompt 必須產生多個 completions, rollout inference 可能成為主要瓶頸.
- 風險: Group 太小時 relative estimate noisy. Group 越大則探索較完整, 但 compute cost 線性增加.
- 限制: 所有 samples reward 相同時, standardized advantage 接近零, 沒有方向可學.

GRPO 仍可使用 probability ratio, clipping 和 KL regularization. 它不是完全不同於 PPO 的獨立思想, 而是改變 advantage estimation 和模型配置.

## Zero-Reward 問題與 SFT Priming

[02:00:20](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=7220s)

Demo 使用 Qwen 3 base model. System prompt 要求模型把推理與答案放入指定 delimiters, reward function 同時檢查格式和數值正確性.

若未經指令調整的 base model 不會遵守格式, 四個 rollouts 可能全部得到零分. 組內 mean 也是零, 所有 advantages 都相同, 因此 GRPO 不知道哪個方向更好. 提高 rollout 數只能增加撞到成功樣本的機會, 不能保證解決問題.

講者的處理方式是先從 DeepSeek R1 examples 取一個很小的 subset 做 SFT. 影片後段提到 demo 使用約 118 筆 rows. 自動字幕對資料筆數的轉錄有不一致, 所以這個數字只能視為 workshop 設定, 不是可泛化的最低需求.

```text
base model
  -> small SFT set teaches format and initial behavior
  -> GRPO samples diverse trajectories
  -> verifiers distinguish better and worse outputs
```

若起點是已具備 instruction-following 能力的 instruct model, 可嘗試略過 priming. 判斷標準不應是 model 名稱, 而是初始 rollouts 是否能產生非零 reward variance.

## Unsloth 實作流程

[02:00:20](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=7220s)

Workshop 的實作組合包括 Unsloth, vLLM, Qwen 3, LoRA 和 GRPO trainer. 高階流程如下:

1. 載入 base model, tokenizer 與 chat template.
2. 以 LoRA 限制 trainable parameters.
3. 建立含 expected answer 的 prompts dataset.
4. 定義嚴格格式和部分格式 rewards.
5. 定義 exact answer 與 numerical-distance rewards.
6. 先跑小量 SFT, 確認 model 能產生預期結構.
7. 每個 prompt 產生多個 responses, 計算 rewards 後執行 GRPO update.
8. 持續檢查 samples, reward distribution 和 evaluator loopholes.

Demo 中的 reward values, 例如 exact match 給固定正分, 格式不完整給部分分數或負分, 都是 task-specific heuristics. 它們的相對尺度會改變學習訊號, 不能直接複製到其他任務.

Regex verifier 也需要獨立測試. 應涵蓋缺少 closing delimiter, 多個答案區塊, comma-formatted numbers, scientific notation, 空白差異和 prompt injection 等 cases. Parser 若抽錯答案, optimizer 只會更有效率地學習錯誤目標.

## Sampling Diversity 不是免費午餐

GRPO 需要同一 prompt 的 rollouts 有差異. Temperature 設為零會讓 samples 幾乎相同, 失去 group comparison 的意義. 但 temperature 太高也可能產生大量無效文字.

講者提到 temperature 約 `1.0` 到 `1.5`, 搭配 `min_p=0.1` 等設定. 這些是 workshop 經驗值, 不是普遍最佳參數. 應以以下 signals 調整:

- Unique completion ratio.
- Reward mean, variance 和 saturation.
- Valid-format rate.
- Pass rate 和 false-positive rate.
- Tokens per successful trajectory.
- Training stability 和 KL drift.

若 reward 很快全部滿分, task 或 verifier 可能太容易. 若長期全部零分, 可能是 task 太難, prompt 格式不清, verifier 有 bug, 或需要 curriculum 和 SFT priming.

## Reward Function 才是主要產品工作

影片反覆指出, PPO 或 GRPO 並不是最難複製的部分. 真正稀缺的是能代表 desired outcome 的資料和 reward functions.

一個可部署的 reward pipeline 至少要問:

- Reward 是否驗證真正 outcome, 還是只驗證表面格式?
- 是否有 model 可以鑽的 loophole?
- Partial credit 是否讓 model 朝正確方向移動?
- 不同 reward components 的 scale 是否互相壓過?
- Verifier 自己有沒有 tests 和 version control?
- Offline reward 是否與 production outcome 相關?
- 是否保留 human review 來發現新型 reward hacking?

在可驗證領域, execution-based reward 通常比文字 judge 更可靠. 在主觀領域, 必須承認評估的不確定性, 並避免把單一 judge score 當成 objective truth.

## Dynamic Quantization

[02:33:07](https://www.youtube.com/watch?v=OkEGJ5G3foU&t=9187s)

Naive quantization 對所有 layers 使用相同 precision. Dynamic quantization 則先估計各 layer 的 sensitivity, 再分配不同 bit width. Workshop 提到兩類 signals:

- Activation quantization error.
- Weight quantization error.

Attention layers, shared experts 或少數敏感 weights 可能需要保留較高 precision, 而大量 MoE expert layers 可以更積極量化. Layer sensitivity 是 model-specific, 不應把一個 architecture 的配置直接套到另一個 model.

講者以 vision-language model 為例. 全層 4-bit 的較小版本誤判圖片內容, 保留部分敏感 layers 精度後, 檔案稍大但回答恢復正確. 這個例子說明平均 benchmark 之外, layer selection 可能直接影響特定能力.

影片也提到 1.58-bit weights, Blackwell FP4, MXFP4 和 `torch.compile`. 其中 GPU precision 的未來上限是講者推測. `torch.compile` 也不保證每個 workload 都更快, 仍需針對實際 shapes, kernels, compilation overhead 和 serving path benchmark.

## 實務採用清單

開始 GRPO 前:

- 先建立 deterministic verifier tests.
- 檢查 base policy 的 reward distribution, 不只看幾個漂亮 examples.
- 確認同一 prompt 能產生有意義的多樣 responses.
- 若 reward variance 為零, 先調整 task difficulty, curriculum 或 SFT priming.
- 分開監控 format reward 和 outcome reward.
- 保留未參與訓練的 eval set.

訓練期間:

- 追蹤每個 reward component 的 mean 和 variance.
- 閱讀高 reward samples, 搜尋 reward hacking.
- 監控 KL drift, response length 和 mode collapse.
- 記錄 model, tokenizer, template, dataset, verifier 和 sampling config 的版本.
- 用實際 wall-clock 和 cost 比較 group size, 不只比較 training loss.

量化前:

- 先定義必須保留的 capabilities 和 latency target.
- 對 layer sensitivity 做 measurement, 不以單一 bit width 一刀切.
- 同時測 quality, memory, prefill latency 和 decode throughput.
- 對產品關鍵 cases 做 task-level regression, 不只看 aggregate benchmark.

## 核心結論

Reinforcement fine-tuning 的關鍵瓶頸不是知道 GRPO 公式, 而是建立能產生探索空間, 可可靠驗證, 且不容易被利用的 feedback loop. GRPO 以組內相對分數省去 value model, 讓實驗更容易進入單機或有限 GPU 環境, 但 rollout 成本和 reward engineering 仍然存在.

最可操作的起點是選擇一項真正可驗證的窄任務. 先測試 verifier, 再觀察 base model 的 reward variance. 必要時用少量 SFT 建立初始成功軌跡, 然後才讓 GRPO 擴大探索. Quantization 也遵循相同原則: 不追求最小 bit 數本身, 而是以 task-level evidence 決定哪些部分可以壓縮.

## 內容限制

- 本筆記依據英文自動字幕, 專有名詞與數字可能有轉錄誤差. 例如字幕多次把 `vLLM` 轉成其他拼法.
- Unsloth 的下載量, GitHub stars, 記憶體節省, 訓練速度和 quantization 結果主要是講者自述, 本筆記未做獨立 benchmark 驗證.
- 影片中的 sampling parameters, reward weights 和 dataset size 是 demo choices, 不構成通用建議.
- PPO 和 GRPO 數學以概念教學為主. 精確 objective, estimator 和 implementation details 應回查原始論文與使用中的 trainer source code.
- 影片對 open model 發展和未來 GPU precision 的部分判斷帶有個人觀點, 不應當作已驗證事實.
