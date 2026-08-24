# 掌握 LLM Inference 最佳化, 從原理到具成本效益的部署

來源: [Mastering LLM Inference Optimization From Theory to Cost Effective Deployment: Mark Moyou](https://www.youtube.com/watch?v=9tvJ_GYJA-o), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=9tvJ_GYJA-o
- 上傳日期: 2025-01-01
- 片長: 33:39
- Video ID: `9tvJ_GYJA-o`
- 講者: Mark Moyou, NVIDIA
- 內容依據: 英文原始語言自動字幕

## 摘要

LLM inference 與一般深度學習 inference 的關鍵差異, 是它不只要把模型權重放進 GPU, 還要為每個請求保存持續增長的 KV cache, 並逐 token 產生輸出. 模型大小, 輸入與輸出長度, 並行策略, batching, quantization 與 GPU memory 因此共同決定延遲, 吞吐量與成本.

影片先以 tokenization, embedding, attention, prefill 與 decode 建立直覺, 再說明如何用 time to first token, inter-token latency 與 total generation time 觀察 production deployment. 最實用的觀點是, 不應只看模型的最大 context 或單一 benchmark, 而要量測真實使用者的 input sequence length 與 output sequence length 分布, 再據此建置 inference engine.

後半段介紹 TensorRT-LLM, Triton Inference Server, inflight batching, paged KV cache, tensor parallelism 與低精度推論. 產品名稱與硬體世代反映演講錄製時的 NVIDIA 生態, 實際部署前應重新核對最新支援矩陣與 benchmark.

## LLM Inference 的基本工作負載

[01:56](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=116s)

當 prompt 送入 GPU 後, inference 大致包含兩個階段:

1. Prefill, 處理完整 prompt 並計算 attention.
2. Decode, 根據先前內容逐次產生下一個 token.

輸出通常是自回歸的. 第 `t` 個 token 依賴輸入 prompt 與之前已產生的 token, 因此不能一次獨立生成所有輸出.

```text
Text prompt
  -> Tokenization
  -> Token IDs
  -> Embeddings
  -> Prefill / Attention
  -> First token
  -> Decode one token at a time
  -> Detokenization
  -> User-visible text
```

服務同時面對大量使用者時, GPU 一方面要為新請求處理完整 prompt, 另一方面還要替既有請求持續 decode. Production inference 的挑戰正是如何在兩種不同運算特性之間有效排程.

## Token 不是人類語言中的單字

[04:35](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=275s)

每個模型有自己的 tokenizer 與 vocabulary. Token 可能是完整單字, 子詞, 字元群組, 符號或程式碼片段. 影片用約四個字元對應一個 token 作為英文的粗略心智模型, 但實際比例會因語言, 內容與 tokenizer 而異.

文字進入模型前會轉成 token IDs, 再查表取得每個 token 的 embedding vector. 一段 prompt 最終成為矩陣, GPU 擅長的正是大規模矩陣運算.

影片提到的 tokenizer vocabulary 數量是特定模型例子, 不應推廣到所有 Llama 或其他模型版本.

## Attention 的直覺

[07:14](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=434s)

Attention 要回答的問題是, 對目前要生成的 token 而言, 先前哪些 token 最重要. 輸入 embedding 分別乘上模型學到的權重矩陣, 形成 query, key 與 value:

```text
Q = XWq
K = XWk
V = XWv
```

影片沒有深入公式, 但建立了兩個重要直覺:

- Query 表示目前正在尋找什麼資訊.
- Key 與 value 保存可供後續 token 參照的表示.

多個 attention head 會以不同投影理解 token 關係, 再合併結果預測下一個 token. 因此, 每個請求都涉及多組矩陣運算與持續更新的中間狀態.

## KV Cache 為什麼重要

[09:40](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=580s)

如果每生成一個新 token 都重新計算整段歷史的 key 與 value, decode 會浪費大量運算. KV cache 保存先前 token 已計算過的 K 與 V, 新步驟只需要計算最新 token 的表示, 再讓它對既有 cache 做 attention.

```text
沒有 KV cache:
每一步重新處理 prompt + 所有已生成 token

有 KV cache:
重用過去的 K/V + 只計算最新 token
```

代價是 KV cache 會隨 sequence length, batch 中的請求數, layer 數與表示精度增長. 一個請求保留愈久, 或輸入與輸出愈長, 佔用的 GPU memory 就愈多.

這形成 production inference 的主要資源競爭:

```text
GPU memory = Model weights + KV cache + Runtime overhead
```

影片為建立快速估算, 建議在 FP16 下用參數量乘以 2 bytes 估計權重大小. 例如 8B parameters 約需 16 GB 存放原始 FP16 權重. 這只是近似值, 還未包含 runtime buffer, allocator overhead 與其他中間張量.

## Prefill 與 Decode 的運算特性

[11:23](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=683s)

Prefill 一次處理整段 prompt, 以較大的 matrix-matrix operations 為主, 通常具有較高 compute intensity. Decode 每一步只新增一個 token, 對單一請求而言更接近 matrix-vector operations, 而且反覆讀取模型權重與 KV cache, 常受 memory bandwidth 與排程影響.

Inference engine 會將多個請求的 decode 工作 batch 在一起, 讓 GPU 同時產生不同請求的下一個 token. 因此, 單一請求的理論速度並不能完整代表高併發服務的表現.

## 三個主要延遲指標

[13:05](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=785s)

### Time to First Token, TTFT

從請求進入系統到第一個輸出 token 出現的時間. 它包含排隊, prompt processing 與第一步 decode, 可用來觀察 prefill 與排程在負載下的表現.

### Inter-Token Latency, ITL

第一個 token 之後, 相鄰輸出 token 之間的延遲. 對串流文字而言, ITL 決定使用者看到內容連續出現時是否流暢.

### Time to Total Generation

從請求開始到整段輸出完成的時間. 它同時受輸入長度, 輸出長度, queueing, batching 與每 token decode 速度影響.

影片也使用兩個工作負載縮寫:

- ISL, input sequence length.
- OSL, output sequence length.

## 一般模型部署與 LLM 部署的差異

[14:20](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=860s)

對許多傳統深度學習模型, 一張 GPU 可能同時放置多個較小模型, 以提高硬體使用率. LLM deployment 常由一個大型模型佔用主要記憶體, 剩餘空間則決定能容納多少 KV cache 與並行請求.

這使 optimization 的目標不只是提高單次 forward pass 的速度, 還包括:

- 縮小模型權重.
- 壓縮 KV cache.
- 改善 batch manager 與 scheduler.
- 控制請求的 input / output 長度.
- 讓被釋放的 cache 空間能立即服務新請求.

## 四種 Query Pattern

[15:16](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=916s)

影片將 production requests 按 ISL 與 OSL 分成四種組合. 這是容量規劃最重要的視角之一:

| Pattern | Prefill 特性 | Decode 與 memory 特性 | 可能場景 |
| --- | --- | --- | --- |
| Long input, short output | Prompt processing 較重 | 生成很快結束 | 長文件分類, 摘要式回答 |
| Long input, long output | Prefill 重 | KV cache 大且存活久 | 長文件改寫, 大型程式生成 |
| Short input, long output | TTFT 可能較低 | 持續 decode 並逐步增加 cache | 文章或程式生成 |
| Short input, short output | 單次負擔較小 | 容易快速完成 | 分類, 簡短問答 |

長 input, 長 output 通常最消耗 serving capacity, 因為它同時佔用較多 prompt cache 與較長生成時間. 如果所有使用者都使用最大 context, 單張 GPU 可同時服務的請求數會顯著下降.

Benchmark 若只挑一種 pattern, 可能無法代表 production traffic. 系統應記錄每個請求的 ISL 與 OSL, 觀察聯合分布, 而不只看平均值.

## 用真實分布設計 Inference Engine

[17:42](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1062s)

影片建議以 ISL 與 OSL 的二維 histogram 分析使用者行為. 即使 API 宣稱支援很長的最大 context, 真實流量也可能集中在遠低於上限的區域.

從分布可以回答:

- 常見 input 與 output 長度是多少.
- 需要覆蓋到哪個 percentile.
- 是否有少數超長請求拖累整體服務.
- Engine 的 max input / output sequence length 應如何設定.
- 是否應將極端請求路由到不同 deployment.

Engine 對最大長度的設定會影響底層 buffer 與執行配置. 用過大的上限編譯所有服務, 可能浪費 memory 或降低效率. 用過小的上限則會拒絕合法流量, 因此需要用統計分布與服務目標共同選擇.

## 在負載下分析 TTFT

[19:02](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1142s)

單一 request 的 TTFT benchmark 只能呈現理想條件. 真正要觀察的是 GPU 滿載時, TTFT 如何隨 input tokens 與 concurrent requests 改變.

影片建議繪製:

- TTFT distribution.
- TTFT versus number of input tokens.
- Total completion time distribution.
- ITL versus generated token position.

正常情況下, input 愈長, TTFT 通常會增加. 系統最佳化的目標是降低曲線斜率, 並控制高負載下的尾端延遲. 如果相同長度請求的完成時間差異很大, 應檢查 batching, scheduling 與資源競爭.

## Inter-Token Latency 的漂移

[20:44](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1244s)

隨著輸出序列變長, KV cache 增加, 系統負擔也可能上升. 如果 ITL 對 token position 呈現明顯惡化或變異擴大, 表示 scheduler, memory pressure 或 throttling 可能影響 decode.

理想服務應讓 token generation cadence 儘量穩定. 但實務上還要分解 queueing, request mix, batch size 與硬體差異, 不能單憑一張圖直接判斷根因.

## Quantization 的成本效益

[23:08](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1388s)

Quantization 使用較低 precision 表示 weights, activations 或 KV cache. 主要收益包括:

- 減少模型權重佔用的 GPU memory.
- 為 KV cache 與更多 concurrent requests 留出空間.
- 降低 memory bandwidth 壓力.
- 在硬體支援下提高運算速度.

影片以 FP16 降到 FP8 為例, 將權重儲存需求大致減半, 並指出 Hopper 與 Ada Lovelace 世代對 FP8 的支援. 它也提到當時公布的 Blackwell FP4 能力.

較低 precision 並非免費. 部署前需要在代表性資料上驗證品質, latency 與 throughput. "幾乎相同 accuracy" 只在特定模型, 校準方法, 工作負載與硬體組合下成立.

## Inflight Batching 與 KV Cache 管理

[24:27](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1467s)

傳統 static batching 可能等待 batch 中所有請求完成後才接收新請求, 但生成長度不一致會讓較短請求完成後留下空槽. Inflight batching 允許某個 request 結束時立即插入新 request, 提高 GPU utilization.

影片另外提到:

- Quantized KV cache, 以較低 precision 儲存 K/V.
- Paged KV cache, 以分頁方式管理不規則且動態增長的 cache.
- Batch manager, 在 prefill 與 decode request 之間排程工作.

這些技術的共同目標是減少 memory fragmentation, 提高可同時服務的 sequence 數, 並避免長短請求互相拖累.

## Parallelism

[25:12](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1512s)

當模型無法放入單張 GPU, 或需要降低 latency 時, 可以分散到多張 GPU:

### Tensor Parallelism

將同一 layer 的 tensor operations 切分到多張 GPU, 各 GPU 共同完成一層. 通訊頻繁, 因此通常在單一 node 內使用高速互連.

### Pipeline Parallelism

將不同 layer 或 stage 放在不同裝置, 資料依序通過. 可跨 node 擴展大型模型, 但要處理 pipeline bubble 與 stage 間負載平衡.

影片也簡短提及 data, sequence 與 expert parallelism. 選擇策略時不能只看模型能否載入, 還要考量 GPU interconnect, 通訊成本, latency SLO 與 concurrency.

更多 GPU 不必然帶來更便宜的服務. Tensor parallelism 可以縮短單一 request latency, 卻可能降低每 GPU 的整體成本效率. 應以目標 workload 實測.

## TensorRT-LLM 與 Triton 的角色

[22:16](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1336s)

影片將 NVIDIA software stack 分為兩層:

```text
Model
  -> TensorRT-LLM 編譯與最佳化
  -> GPU-specific inference engine
  -> Triton Inference Server 託管
  -> HTTP / gRPC requests, batching, scheduling, metrics
```

TensorRT-LLM 專門針對 NVIDIA GPU 編譯與最佳化 LLM inference, 包含 attention kernels, quantization, KV cache 與 parallelism 等能力. 編譯出的 engine 會針對特定 GPU architecture 與設定, 不能假設能原封不動移到另一種 GPU.

Triton Inference Server 負責載入與服務 engine, 接收請求, batching 並提供統一部署介面. 它也能服務其他 framework 與模型類型, 讓平台團隊不必為 PyTorch, TensorFlow 或自訂 Python 模型各維護一套 server.

影片也介紹 NVIDIA Inference Microservice, 作為預先最佳化的企業方案. 這部分帶有供應商產品介紹性質, 且名稱與功能可能已更新.

## Seasonal Engine 的構想

[29:27](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1767s)

問答中, 講者延伸出 seasonal engines 的想法. 同一服務在不同時段可能有不同 query pattern, 例如白天多短問答, 夜間多批次長文件處理. 團隊可以為不同流量分布建立不同 engine configuration, 再隨 scaling event 切換.

是否值得這樣做, 要比較:

- 切換後是否仍符合所有必要 request shapes.
- 對 TTFT, ITL 與 throughput 的實際改善.
- 節省的 GPU 成本是否高於編譯, 驗證與營運複雜度.
- Engine 切換期間如何維持服務與 cache 一致性.

影片仍指出, 常見做法是先尋找一個能涵蓋主要 request mix 的配置. Seasonal engine 是進一步的工程與研究方向, 不是所有服務都需要的預設架構.

## 長 Context 的代價

[31:35](https://www.youtube.com/watch?v=9tvJ_GYJA-o&t=1895s)

問答最後談到, context 長度會放大 attention 計算與 memory 需求. 訓練與服務超長 context 都很昂貴, 因此產業會研究更有效率的 attention kernels, interpolation 方法, 新架構與高速 GPU interconnect.

影片提到 FlashAttention 與 NVIDIA 高速互連作為例子. 這裡的硬體數字與產品描述具有時間性, 但長 context 並非免費的原則仍成立. API 支援某個 context 上限, 不代表每個請求都應使用上限.

## Production Capacity Planning 流程

### 1. 收集真實 workload

至少記錄:

- Model 與版本.
- Input tokens 與 output tokens.
- TTFT, ITL 與 total generation time.
- Request arrival rate 與 concurrency.
- Batch size 與 queue time.
- GPU memory utilization 與 KV cache usage.
- 成功, 拒絕, timeout 與 OOM 狀態.

### 2. 建立 request shape 分布

以 ISL x OSL heatmap 找出主要流量, 尾端請求與時段差異. 平均值不足以描述 inference workload, 應搭配 percentile 與聯合分布.

### 3. 選擇模型表示

比較 FP16, BF16, FP8, INT8 或其他受支援格式. 用代表性 eval 驗證品質, 不只比較 engine size.

### 4. 選擇 parallelism 與 batching

依模型大小, interconnect, latency SLO 與 request rate 決定是否使用 tensor / pipeline parallelism, 並調整 inflight batching 與 scheduler.

### 5. 壓力測試

同時覆蓋四種 ISL / OSL pattern, 尤其是 long input, long output. 在接近飽和時觀察 TTFT 與 ITL 尾端延遲, 而不是只測單一 request.

### 6. 以成本衡量結果

部署選擇應用相同品質門檻比較:

```text
Cost per successful request
Cost per output token
Requests per second per GPU
Tokens per second per GPU
SLO attainment rate
```

只有速度快但品質下降, 大量 timeout 或需要更多 GPU 的配置, 不一定更具成本效益.

## 核心觀念回顧

```text
成本與效能
├── Model weights
│   ├── Parameter count
│   └── Precision / quantization
├── KV cache
│   ├── Concurrent requests
│   ├── Input sequence length
│   ├── Output sequence length
│   └── Cache precision / memory manager
├── Compute
│   ├── Prefill attention
│   ├── Decode
│   └── Parallelism
└── Serving
    ├── Queueing
    ├── Inflight batching
    ├── Scheduling
    └── Traffic distribution
```

最值得帶走的三個原則:

1. LLM serving 的稀缺資源不只有權重空間, 還有會隨請求變化的 KV cache.
2. 用真實 ISL / OSL 分布規劃容量, 不要只依賴最大 context 或理想 benchmark.
3. 在 production load 下同時觀察 TTFT, ITL, throughput, 品質與成本.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理. 影片沒有 YouTube chapters, 章節與時間戳由內容結構編排. 自動字幕可能誤辨套件名稱, GPU 型號與技術縮寫. 本筆記已修正上下文中可可靠辨識的術語, 但不是逐字稿.

影片錄製於 AI Engineer World's Fair 2024, 並於 2025-01-01 上傳. 其中 NVIDIA GPU, precision 支援, TensorRT-LLM, Triton, NVIDIA Inference Microservice 與開源工具的功能會持續變動. 這些內容適合作為 inference 原理與量測方法的學習材料, 不應直接視為目前產品選型文件.
