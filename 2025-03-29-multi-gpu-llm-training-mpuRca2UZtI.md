# 利用多張 GPU 訓練大型語言模型: DeepSpeed, FlashAttention, Liger Kernel 與 Quantization

來源: [【生成式 AI 時代下的機器學習 (2025)】助教課: 利用多張 GPU 訓練大型語言模型](https://www.youtube.com/watch?v=mpuRca2UZtI), Hung-yi Lee

- 影片網址: https://www.youtube.com/watch?v=mpuRca2UZtI
- 上傳日期: 2025-03-29
- 長度: 54:56
- 講者: 王秀軒 (Hsiu-Hsuan Wang)
- Video ID: `mpuRca2UZtI`
- 內容依據: 創作者提供的繁體中文字幕與影片章節

## 摘要

大型語言模型難以訓練, 不只是因為權重很大。訓練時還必須容納梯度, Adam optimizer 狀態, 混合精度使用的權重副本, 以及隨 batch size 和 context length 增長的 activations。解法不是單一技巧, 而是先辨認瓶頸屬於哪一類, 再選擇對應工具。

- 權重, 梯度與 optimizer states 放不下時, 用 DeepSpeed ZeRO 將它們分片到多張 GPU。
- 長 context 造成 attention activations 過大時, 用 gradient checkpointing, FlashAttention 與 Liger Kernel 降低記憶體需求或加速 kernel。
- 推論環境的 VRAM 有限時, 用 quantization 降低每個參數的儲存位元數。

整體原則是以記憶體換運算, 以通訊換單卡容量, 或以精度換模型大小。每種交換都有成本, 因此不能只看模型能否載入, 還要看通訊頻寬, step time 與可接受的精度損失。

## 為什麼一個 8B 模型也很難完整訓練

[04:38](https://www.youtube.com/watch?v=mpuRca2UZtI&t=278s)

影片用 8B 模型與 Adam optimizer 說明基本記憶體帳目。在簡化估算中:

| 項目 | 精度 | 約略容量 |
| --- | ---: | ---: |
| FP32 主權重 | 32 bit | 32 GB |
| FP16 計算用權重 | 16 bit | 16 GB |
| FP16 gradients | 16 bit | 16 GB |
| Adam momentum | 32 bit | 32 GB |
| Adam variance | 32 bit | 32 GB |
| 合計, 尚未計 activations |  | 128 GB |

FP16 不只是讓資料量減半。多數 NVIDIA GPU 對低精度運算有硬體加速, 因此混合精度也能提高運算速度。不過訓練仍可能保留 FP32 主權重與 optimizer states, 所以不能把「8B 乘以 2 bytes」當作完整訓練的總需求。

這份估算是教學用近似值。實際占用還取決於 optimizer, dtype, framework buffers, allocator, batch, sequence length 與平行化策略。

## Activations 才可能是長 context 的記憶體殺手

[09:50](https://www.youtube.com/watch?v=mpuRca2UZtI&t=590s)

反向傳播需要前向傳播的中間結果, 因此每層都會保留 activations。標準 attention 會形成與序列長度平方相關的中間資料, context 越長, 記憶體壓力就越明顯。模型參數相關的記憶體大致固定, activations 則會隨輸入長度和 batch size 改變。

影片提出兩個基礎技巧:

### Gradient checkpointing

前向傳播時不保存所有 activations, 只留下必要的 checkpoint。反向傳播需要某段中間結果時再重新計算。這能大幅降低記憶體, 代價是額外運算與較慢的訓練速度。

### Gradient accumulation

若全域 batch 無法一次放入 GPU, 可拆成多個 micro-batches。每次 forward/backward 先累積梯度, 達到指定次數後才更新一次參數。

影片以 1,920 個 32K context 為例, 約等於 6,144 萬 tokens。若 micro-batch size 是 16, 就累積 120 次才做一次 optimizer update。它模擬較大的 global batch, 但不能降低完成整個 global batch 所需的總運算量。

## DeepSpeed ZeRO: 把訓練狀態分散到多張 GPU

[20:20](https://www.youtube.com/watch?v=mpuRca2UZtI&t=1220s)

傳統 data parallelism 讓每張 GPU 都保留完整模型與訓練狀態。ZeRO, Zero Redundancy Optimizer, 逐階段移除這些重複資料:

| ZeRO 階段 | 分片內容 | 效果與代價 |
| --- | --- | --- |
| Stage 1 | optimizer states | 先處理最大的 optimizer 狀態, 通訊增量較小 |
| Stage 2 | optimizer states 與 gradients | 進一步省記憶體, 需要更多跨卡資料交換 |
| Stage 3 | optimizer states, gradients 與 parameters | 每卡占用最低, 但參數使用前也可能需要跨卡收集 |

Stage 越高不代表在所有環境都越好。省下的顯存越多, 通訊與排程也越複雜。高速 GPU interconnect, 例如 NVLink, 以及傳輸和計算重疊的排程, 會直接影響實際效能。

### CPU offload

如果分片後仍放不下, DeepSpeed 可把 optimizer states, 甚至部分參數移到 CPU RAM, 使用時再搬回 GPU。這讓原本不可能執行的訓練變得可行, 但 CPU RAM 與 GPU VRAM 間的頻寬和延遲可能讓 step time 大幅增加。

影片的實測案例使用 8B 模型與 V100 GPU:

- 4 張 V100 配合 optimizer offload, 每張實際約使用 15 GB VRAM, 每 step 約 74 秒。
- 8 張 V100 不使用 offload, 每張約使用 24 至 32 GB VRAM, 每 step 約 7.3 秒。

這是特定硬體與極短輸入設定的示例, 不宜直接當作其他叢集的效能預測。它支持的實務結論是: offload 是容量上的退路, 若 GPU 數量與顯存足夠, 避免 offload 通常能得到更好的速度。

## GPU kernel 的抽象層次

[33:14](https://www.youtube.com/watch?v=mpuRca2UZtI&t=1994s)

為改善 attention 與其他常用操作, 可以在不同抽象層次優化 GPU kernel:

1. PyTorch: 容易使用, 但對運算順序與記憶體配置的控制較少。
2. `torch.compile`: 編譯計算圖並進行融合, 記憶體排程與預取等最佳化。
3. Triton: 用 Python 風格介面撰寫 GPU kernel, 在可用性與底層控制之間取得平衡。
4. CUDA: 以 NVIDIA 的底層平台和 C/C++ 工具取得高度控制, 開發成本也最高。

越低階不保證應用程式一定越快。成果取決於 kernel 品質, 硬體特性, 資料形狀與整體 pipeline。

## FlashAttention: 融合並重排 attention 計算

[36:10](https://www.youtube.com/watch?v=mpuRca2UZtI&t=2170s)

標準 scaled dot-product attention 包含矩陣乘法, scaling, masking, softmax, dropout 及另一個矩陣乘法。若每一步都各自形成完整中間張量並反覆讀寫較慢的 GPU 高頻寬記憶體, 資料搬移會成為瓶頸。

FlashAttention 的核心是 IO-aware tiling 與 kernel fusion。它把 attention 分塊計算, 讓資料盡量停留在更靠近運算單元的快速片上記憶體, 並避免物化完整的 attention matrix。結果通常是:

- 降低 HBM 讀寫量。
- 將 attention 的額外記憶體需求由平方量級降到近似線性。
- 在長序列上同時改善速度與可訓練長度。

這裡補充釐清字幕中的表述: FlashAttention 的重點不是把資料 offload 到 CPU RAM, 而是減少 GPU 記憶體階層之間的資料搬移, 特別是 HBM 與 on-chip SRAM 間的 IO。這是依影片所引用的 FlashAttention 技術概念所做的編輯校正。

## Liger Kernel: 替換 Transformer 的常用運算

[41:15](https://www.youtube.com/watch?v=mpuRca2UZtI&t=2475s)

Liger Kernel 使用 Triton 實作多個適合 LLM 訓練的 fused kernels, 將常用 PyTorch 運算替換成更有效率的版本。目標是提高吞吐量, 降低記憶體占用, 並讓較長 context 的訓練成為可能。

影片強調它的整合介面相對簡單, 可透過 Liger 提供的 Hugging Face 相容模型類別載入支援模型。實際採用前仍應檢查模型支援清單, 套件版本與 benchmark, 因為加速幅度會隨模型與硬體改變。

FlashAttention 專注於 attention, Liger Kernel 則覆蓋 Transformer 訓練中的多種常用運算。兩者不是必然互斥的替代品。

## Quantization: 以較低精度縮小推論模型

[44:36](https://www.youtube.com/watch?v=mpuRca2UZtI&t=2676s)

Quantization 是有損壓縮。它用較少位元表示 tensor, 計算時再反量化或直接使用低精度 kernel。常見格式包含 8-bit 與 4-bit, 更低位元也存在, 但壓縮越積極, 精度與相容性風險通常越高。

影片以 8B 模型的 8-bit 權重做直觀估算: 權重本體約需 8 GB, 因而可能放入具有約 15 GB VRAM 的 Colab T4, 並留下部分空間給輸入和 runtime buffers。

本課將 quantization 主要定位於 inference。若目標是訓練, 還需要區分完整低精度訓練, quantization-aware training 與 QLoRA 等不同方法, 不能直接把量化推論模型等同於完整訓練配置。

## 如何選擇工具

| 症狀 | 主要瓶頸 | 優先考慮 |
| --- | --- | --- |
| 模型, 梯度與 optimizer states 放不下 | 固定訓練狀態 | ZeRO Stage 1 至 3 |
| 多卡仍放不下 | GPU 容量 | CPU offload, 並接受速度代價 |
| 長 context 導致 OOM | activations 與 attention 中間資料 | gradient checkpointing, FlashAttention, Liger Kernel |
| global batch 無法一次載入 | batch 記憶體 | gradient accumulation |
| 單張小 GPU 只需執行推論 | 權重容量 | 8-bit 或 4-bit quantization |
| 已經放得下, 但訓練太慢 | 運算, IO 或通訊 | profiling 後調整 kernels, compile, batch 與並行策略 |

推薦的決策順序是先量測, 再最佳化。先將記憶體拆成 parameters, gradients, optimizer states, activations 與 buffers, 接著確認瓶頸在單卡容量, 跨卡通訊, GPU kernel, CPU-GPU 傳輸或資料 pipeline。工具名稱相同, 在不同硬體拓撲上的結果可能完全不同。

## Q&A 重點

[48:45](https://www.youtube.com/watch?v=mpuRca2UZtI&t=2925s)

- CPU offload 慢的主因是 CPU RAM 與 GPU 之間的頻寬和延遲, 以及記憶體系統針對容量或速度的不同設計取捨。
- PyTorch distributed 提供多程序與多 GPU 的通訊基礎。DeepSpeed 可建立在這些能力之上, 額外處理 ZeRO 分片與訓練狀態調度。
- ZeRO 優先分片 optimizer states, 因為它們容量大, 但只在 optimizer update 時使用。相較於每次 forward/backward 都頻繁使用的參數, 先分片 optimizer states 能以較少通訊換取可觀的記憶體節省。

## 實務結論

1. 「模型放得下」和「模型訓練得動」是兩個問題。前者看容量, 後者還要看吞吐量與通訊。
2. ZeRO 的階段應按需求逐步提高, 不必一開始就選 Stage 3。
3. CPU offload 能救容量, 但可能讓訓練慢一個數量級, 應以實測決定。
4. 長 context 的主要壓力常在 activations。gradient checkpointing 用額外運算換記憶體, FlashAttention 與 fused kernels 則減少中間資料和 IO。
5. Quantization 適合小顯存推論, 但應把模型品質, kernel 支援與 runtime buffers 一併納入評估。

## 來源限制與信心

本筆記主要根據創作者提供的繁體中文字幕整理, 並以影片章節作為時間定位。字幕對英語技術詞有多處辨識錯誤, 例如將 `forward/backward pass`, `Adam`, `FlashAttention`, `Liger Kernel`, `inference` 等轉寫成相近讀音。筆記已在上下文足以確認時校正, 對無法可靠還原的細節則不納入。

影片中的容量與速度數據是簡化估算或特定實驗結果, 適合用來理解取捨, 不應視為所有硬體與軟體版本的固定基準。
