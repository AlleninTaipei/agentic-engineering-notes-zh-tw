# KV Cache 與 Paged Attention 如何加速 LLM Inference

來源: [How KV Cache Speeds Up LLMs for Faster AI Models on GPUs](https://www.youtube.com/watch?v=o0gkdZBtwEg), IBM Technology

- 正規網址: https://www.youtube.com/watch?v=o0gkdZBtwEg
- 上傳日期: 2026-06-30
- 片長: 11:14
- Video ID: `o0gkdZBtwEg`
- 內容依據: 創作者提供的英文字幕

## 摘要

LLM 在單一使用者時可能回應很快, 但併發量上升後, GPU memory 用量與 latency 可能急遽增加. 問題不一定在模型本身, 而可能在 inference 過程如何保存與讀取每個請求逐步累積的 context.

影片介紹兩個核心機制. KV cache 用 GPU memory 換取 compute, 保存先前 token 已算好的 key 與 value, 避免每個 decode step 重算整段歷史. Paged attention 則像作業系統管理虛擬記憶體一樣, 把 KV cache 分成小型 blocks, 按需配置到不連續的 VRAM, 降低 fragmentation 與預留浪費.

在這個基礎上, vLLM 還能使用 prefix caching, chunked prefill 與 speculative decoding. 不同技巧最佳化的目標不同, 有些提高 concurrency 與 throughput, 有些降低互動 latency, 因此應依真實 workload benchmark, 而不是一次全部開到最大.

## 問題從併發開始顯現

[00:00](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=0s)

同一模型只服務一位使用者時, time to first token 可能很短. 當同時有十個或一百個請求, 每個請求都擁有不同且持續增長的 context, GPU 不只要執行模型運算, 還要保存與搬移大量中間狀態.

常見症狀包括:

- GPU memory 快速上升.
- TTFT 隨 concurrency 增加.
- 串流 token 出現停頓.
- Throughput 在高負載下反而下降.
- 尚有總 free memory, 卻因 fragmentation 無法配置新請求.

因此, serving optimization 的問題不只是模型每秒能做多少 FLOPs, 還包括有限 VRAM 如何在 weights, KV cache 與 runtime buffers 之間分配.

## LLM Inference 的兩個階段

[01:05](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=65s)

### Prefill

使用者送出 prompt 後, 模型必須先讓輸入通過所有 transformer layers, 建立 attention 所需的表示, 才能產生第一個輸出 token. 這段等待主要發生在 prefill.

Prefill 同時處理整段輸入, 通常具有較高的 compute intensity, 因此影片稱它為 compute-bound phase. 長 prompt 會增加 prefill 工作, 直接影響 time to first token.

### Decode

第一個 token 產生後, 模型以 autoregressive 方式逐 token 生成. 每一步都要讀取先前 token 的 context, 再預測下一個 token.

Decode 每次只新增少量運算, 卻反覆讀取 weights 與 KV cache, 因此更容易受 memory bandwidth 限制. 影片將其稱為 memory-bound phase.

```text
Prompt
  -> Prefill, 處理完整輸入
  -> First token
  -> Decode, 逐 token 生成並讀取 KV cache
  -> Completed response
```

## Attention 中的 Query, Key 與 Value

[02:05](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=125s)

Transformer 在每一層為 token 計算 query, key 與 value. 影片用直覺方式說明:

- Query 是目前 token 在詢問 "哪些資訊與我相關?"
- Key 幫助判斷歷史 token 是否相關.
- Value 提供被關注 token 的內容表示.

在 autoregressive generation 中, 第 `n` 個輸出 token 需要對先前 token 做 attention. 若沒有 cache, 每一步都要重新計算所有先前 token 的 key 與 value. 當輸出愈長, 重複工作就愈大.

## KV Cache 用 Memory 換 Compute

[02:48](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=168s)

KV cache 保存先前 token 已計算好的 key 與 value matrices. 生成新 token 時, 模型只需計算新 token 自己的 Q, K 與 V, 再讓 query 對 cached K/V 做 attention.

```text
沒有 KV cache:
每個 decode step 重算整段歷史的 K/V

使用 KV cache:
重用歷史 K/V, 只計算新 token 的 Q/K/V
```

這大幅減少長序列中的重複運算, 但也讓 KV cache 隨著下列因素增加:

- Active requests 數量.
- Prompt length.
- Generated output length.
- Transformer layers 與 hidden dimensions.
- KV heads 數量.
- Cache precision.

KV cache 因此是一項明確的 memory-for-compute trade-off. 它加快 decode, 但使 VRAM 容量與管理方式成為 LLM serving 的核心瓶頸.

## 模型權重與 KV Cache 爭奪 VRAM

[03:45](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=225s)

影片以約 13B parameters 的 FP16 模型與 40 GB GPU 為例. 權重大約需要 26 GB, 約佔卡片容量的 65%. 剩餘空間還要容納所有 active requests 的 KV cache 與 runtime overhead.

這是一個簡化估算:

```text
13 billion parameters x 2 bytes per FP16 weight
≈ 26 GB
```

實際 GPU usage 還會受到量化格式, allocator, kernels, activations 與 framework overhead 影響. 影片例子旨在說明, 大模型權重載入後, 可供並行 sequence 使用的空間可能比總 VRAM 小很多.

## 傳統連續配置的浪費

[04:19](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=259s)

影片描述的 naive serving system 會依最大 sequence length, 為每個 request 預留一塊固定且連續的 KV cache memory.

如果最大長度是 2,048 tokens, 但使用者只有 200 個 input tokens, 再生成 300 tokens, 系統可能為剩餘的 1,548 tokens 保留空間. 這些未使用空間暫時不能提供給其他 request.

影片將傳統配置中的浪費歸納為三類:

### Internal fragmentation

每個 request 被分配的 block 大於實際使用量, block 內部留下無法共享的空間.

### External fragmentation

不同長度的 request 進出後, free memory 被切成許多不連續區段. 總容量可能足夠, 卻沒有單一連續區塊能滿足新 request.

### Redundant duplication

多個 request 使用相同 system prompt 時, naive system 可能各自保存相同 prefix 的 KV cache.

影片引用傳統系統可能浪費 KV cache 區域中約 60% 至 80% memory 的說法. 這個比例取決於 allocator, sequence distribution 與 serving engine, 不應視為所有部署的固定值.

## Paged Attention

[05:33](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=333s)

Paged attention 借用作業系統 virtual memory paging 的概念. 它不再要求每個 request 的 KV cache 使用單一連續區塊, 而是切成固定大小的 logical blocks, 再按需對應到分散於 VRAM 的 physical blocks.

```text
Model 看見:
Request A -> logical block 0, 1, 2, 3

實際 VRAM:
logical 0 -> physical 9
logical 1 -> physical 2
logical 2 -> physical 14
logical 3 -> physical 6
```

Block table 負責 logical-to-physical mapping. 新 token 到來時才配置需要的 block, 不必一開始依最大長度預留完整空間.

影片以預設 16 tokens per block 說明 vLLM 的配置. 這是影片所述版本的預設例子, 實際版本與設定可能不同.

Paged attention 的主要收益:

- 將 KV cache memory 按需配置.
- 降低 internal fragmentation.
- 不要求實體 memory 連續, 降低 external fragmentation.
- 更快回收已完成 request 的 blocks.
- 讓更多 sequence 同時留在 GPU 中.

它不是縮小單一 token 的 K/V 表示, 而是改善 cache 的配置與共享效率. Cache quantization 則是另一種降低每個 block 大小的方法.

## 調整 GPU Memory Utilization

[06:55](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=415s)

影片提出第一個 vLLM 調整項目, GPU memory utilization. 它控制 engine 願意把多少 GPU memory 用於模型服務與 KV cache.

影片所述參考值:

- 預設約 `0.9`.
- 穩定 workload 可嘗試提高到 `0.95`, 容納更多 concurrent requests.
- 若 burst load 容易 OOM, 可降低到 `0.8` 留出餘裕.

這些不是通用最佳值. 提高比例會增加 cache capacity, 但減少 runtime spike, temporary buffers 或其他 process 的安全空間. 應以模型, quantization, max sequence length 與尖峰流量進行壓力測試.

## Prefix Caching

[07:29](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=449s)

許多請求共享相同前綴, 例如:

- 相同 system prompt.
- RAG pipeline 中固定的 instruction template.
- Multi-turn chat 的共同歷史.
- Coding agent 重複載入的 repository instructions.

Prefix caching 對 KV blocks 的 token sequence 建立識別, 讓共享相同 prefix 的 requests 指向已計算的 blocks. 系統不必為每個 request 重做同一段 prefill, 也不必重複保存同一份 cache.

影片聲稱, 在 RAG, multi-turn chat 與 coding agents 中, 75% 至 95% hit rate 可能出現, 並能顯著降低 TTFT. 這高度依賴 prompt 穩定性. 任何 token 差異, template 變更或 context 排序改變, 都可能降低 reuse.

實務上應量測:

- Prefix cache hit rate.
- 被重用的 tokens 數量.
- TTFT hit / miss 差異.
- Cache eviction 與 memory overhead.

## Chunked Prefill

[08:16](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=496s)

長 prompt 的 prefill 可能長時間佔用 GPU, 使既有 decode requests 的串流 token 暫停. Chunked prefill 將長 prefill 拆成多個 chunks, scheduler 先安排 latency-sensitive decode, 再用剩餘 compute budget 處理 prefill chunks.

它嘗試改善 prefill 與 decode 互相干擾的問題:

```text
沒有 chunking:
長 prefill 完整執行 -> decode 等待

使用 chunking:
decode 優先 + 剩餘 budget 執行 prefill chunk
```

影片提到 production deployment 曾觀察到約 50% throughput improvement, 並建議搭配調整 `max-num-batched-tokens`. 這是工作負載相關結果, 不代表所有模型或 latency SLO 都能得到相同提升. Chunk size 與 token budget 需要同時觀察 throughput, TTFT 與 inter-token latency.

## Speculative Decoding

[09:00](https://www.youtube.com/watch?v=o0gkdZBtwEg&t=540s)

Decode 常受 memory bandwidth 限制, GPU 的部分 compute capacity 可能閒置. Speculative decoding 使用較小的 draft model 一次提出多個候選 tokens, 再由 target model 在一次 forward pass 中驗證.

- 被接受的 tokens 可以一次向前提交.
- 從第一個不符合的 token 起, 依演算法拒絕並由 target model 修正.
- 使用正確的 acceptance scheme 時, 輸出分布可保持與單獨執行 target model 相同.

這種方法適合 interactive latency 比 raw throughput 更重要的情境. 在非常高的 concurrency 下, batching 已使 GPU 保持忙碌, draft 與驗證的額外成本可能縮小收益.

影片也提到 vLLM 的 ngram-based zero-model-cost speculation, 適合結構化或重複性較高的輸出. 字幕將 `ngram` 誤辨為其他字詞, 此處依命令語境修正.

## 各技術最佳化的目標不同

| 技術 | 主要問題 | 主要收益 | 典型代價或風險 |
| --- | --- | --- | --- |
| KV cache | Decode 重算歷史 K/V | 大幅減少重複 compute | 消耗並持續增加 VRAM |
| Paged attention | KV cache fragmentation | 提高 memory utilization 與 concurrency | Block table 與管理 overhead |
| Prefix caching | 相同 prefix 重複 prefill | 降低 TTFT 與重複 cache | Prompt 差異會降低 hit rate |
| Chunked prefill | 長 prefill 阻塞 decode | 改善 scheduling 與串流穩定性 | 需要調整 chunk 與 token budget |
| Speculative decoding | 單請求 decode latency | 一次接受多個 tokens | Draft overhead, 高 concurrency 下收益下降 |

這些技術不是互斥選項, 但也不是開啟愈多就一定愈快. 它們會改變 memory, scheduling 與 latency trade-offs, 必須在相同 workload 與品質要求下比較.

## 實作與驗證清單

### 建立 baseline

- 固定模型, precision, GPU 與 vLLM 版本.
- 記錄 input / output length distribution.
- 測量 TTFT, inter-token latency, total latency 與 throughput.
- 記錄 GPU memory, KV cache usage, OOM 與 request queue.
- 分別測試低 concurrency, 典型 load 與 burst load.

### 評估 paged attention 與 memory utilization

- 確認可容納的最大 concurrent sequences.
- 觀察 cache blocks 使用率與 eviction.
- 逐步提高 memory utilization, 不直接跳到極限.
- 用長短混合 requests 測 fragmentation 與 OOM.

### 評估 prefix caching

- 將穩定 system prompt 放在 prefix 前段.
- 避免不必要的 timestamp, random ID 或順序變化破壞 prefix.
- 同時報告 hit rate 與實際重用 token 數.
- 驗證 tenant isolation, 權限與敏感資料不會因 cache sharing 洩漏.

### 評估 chunked prefill

- 建立 long-prefill 與 active-decode 同時存在的測試.
- 比較 throughput, TTFT 與 token streaming jitter.
- 調整 batched token budget, 尋找符合 SLO 的折衷點.

### 評估 speculative decoding

- 選擇與 target model 輸出分布相容的 draft strategy.
- 記錄 acceptance rate.
- 分別測試低 concurrency 與高 concurrency.
- 驗證 end-to-end latency, 不只看 accepted tokens per step.

## 核心觀念回顧

```text
LLM serving latency
├── Prefill
│   ├── Input length
│   ├── Attention compute
│   └── Prefix reuse
└── Decode
    ├── Model weight reads
    ├── KV cache reads and growth
    ├── Memory allocation
    └── Request scheduling

GPU memory
├── Model weights
├── KV cache blocks
└── Runtime overhead and safety margin
```

最值得帶走的三個原則:

1. KV cache 以 VRAM 換取 decode compute, 它解決重算問題, 也創造 memory capacity 問題.
2. Paged attention 的重點是按需, 非連續地配置 cache, 而不是單純縮小模型.
3. 設定必須用真實 concurrency 與 sequence distribution 驗證, 單一 request benchmark 不足以代表 production.

## 編輯補充

影片著重效能, 但 prefix cache sharing 也涉及安全邊界. 多租戶服務應確認 cache key, isolation 與 eviction 實作不會讓一個租戶推測或取得另一個租戶的內容. 這是根據影片架構所做的 production 安全延伸, 不是影片逐字提出的建議.

## 來源與限制

本筆記依創作者提供的英文字幕整理. 影片沒有 YouTube chapters, 章節與時間戳由內容結構編排. 字幕中少數技術術語有明顯轉錄錯誤, 本筆記僅在上下文足以確認時修正為 `paged attention`, `vLLM`, `GuideLLM` 與 `ngram`.

影片中的預設值, block size, cache hit rate 與 throughput improvement 反映特定 vLLM 版本和工作負載. vLLM 的 CLI, scheduler 與預設設定可能變更, 實際部署前應以使用版本的文件與 benchmark 為準.
