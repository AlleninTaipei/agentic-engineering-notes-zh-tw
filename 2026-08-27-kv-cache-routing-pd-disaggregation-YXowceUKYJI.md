# Agentic Inference 的 KV Cache Routing 與 Prefill/Decode 分離

> - 影片: [KV Cache-Aware Routing and P/D Disaggregation on Kubernetes](https://www.youtube.com/watch?v=YXowceUKYJI)
> - 頻道: [AI Engineer](https://www.youtube.com/@aiDotEngineer)
> - 講者: Yuchen Fama, Red Hat Inference 產品經理; Ashish Kamra, Red Hat 效能工程資深經理
> - 發布日期: 2026-08-27
> - 片長: 21:47
> - Video ID: `YXowceUKYJI`
> - 內容依據: YouTube 英文原語自動字幕 (`en-orig`), 公開影片資訊與章節
> - 整理語言: 繁體中文

## 核心觀點

Agentic workloads 經常重用長上下文, 但 session 長度, 請求到達時間與分支行為變化很大. 講者認為, 部署決策應同時考慮 cache 重用, 首個 token 的等待時間與後續 token 的穩定性, 不能只看穩態吞吐量.

影片以 llm-d 說明兩種互補的方法:

- KV cache-aware routing: 將請求導向有可重用 cache 且負載合適的 pod, 減少重算.
- Prefill/decode disaggregation, 簡稱 P/D 分離: 將 prompt 處理與逐 token 生成放到不同 workers, 降低兩階段的資源干擾.

兩者可組合, 但 P/D 分離需要付出 KV cache 傳輸與資源配置成本. 講者也展示了保留合併部署較合理的條件.

## 先理解工作負載, 再解讀 benchmark

[00:00](https://www.youtube.com/watch?v=YXowceUKYJI&t=0s), [03:29](https://www.youtube.com/watch?v=YXowceUKYJI&t=209s)

講者指出, 公開 inference benchmark 常呈現隔離環境中的穩態結果, 難以涵蓋多輪互動與上下文波動. 他們從 agentic 任務與 coding sessions 觀察到:

- Session 從少數幾輪到約 3,000 輪都有.
- System prompt 與工具定義反覆出現, cache hit rate 常超過 90%.
- 輸入與輸出 token 比例可能超過 100:1.
- Subagents 的並行活動使排程更加複雜.

這些是講者所分析資料的特性, 不是所有 agents 的固定分布. 他們主張容量規劃要看分布與 P90 等分位數, 避免平均值掩蓋長尾, 並提到與 Google 及 IBM 合作在 `inference-perf` 加入 trace replay 能力.

### 本文使用的指標

以下為閱讀用的術語整理:

| 指標 | 意義 | 對應問題 |
| --- | --- | --- |
| TTFT | Time to first token, 首個 token 延遲 | 使用者開始看到回覆前要等多久 |
| ITL | Inter-token latency, token 間隔延遲 | 回覆開始後是否流暢, 有沒有停頓 |
| P99 ITL | Token 間隔延遲的第 99 百分位 | 尾端延遲是否過高 |
| ISL/OSL | 輸入/輸出序列長度 | 工作負載偏向處理長 prompt 還是長時間生成 |
| Cache hit rate | Cache 命中比例 | 既有計算能重用多少, 具體分母仍需依測試定義 |

## Cache 的生命週期受到 client 行為影響

[05:12](https://www.youtube.com/watch?v=YXowceUKYJI&t=312s)

Prompt 結構由使用者與 client 決定, 上下文的變動使 cache eviction 與重建更頻繁. 因此, 推論引擎與上層 routing 需要協調, 將 prefix 重用與延遲納入排程.

講者也建議分開衡量 cached throughput. 她以演講當時展示的 API 價格說明 cached 與 uncached tokens 可能有約十倍差距, 強調重用的經濟價值. 這是影片中的價格例子, 並非本筆記確認的現行費率, 也不能直接換算成自架 GPU 成本降幅.

## llm-d 如何兼顧 cache locality 與負載

[06:28](https://www.youtube.com/watch?v=YXowceUKYJI&t=388s)

講者介紹 llm-d router 的 endpoint picker 擴充機制. 它會依各 pod 的執行與等待請求, KV cache 使用狀況與 prefix cache 可用性等訊號評分, 尋找負載與 cache 命中機會較合適的目的地.

這表示選擇目的地時需要同時考慮重用與排隊成本. 講者沒有提供這場示範所用的完整評分公式與權重.

在 cache 管理方面, 她介紹團隊正在投入的方向, 包括更多 offload 儲存層, NVMe SSD 與 KV 專用儲存, 以及 priority 和 session pinning 等 session-aware eviction 策略. 這些內容包含進行中的工作, 不宜全數視為已可直接啟用的功能清單.

### 示範: 相同 prefix 回到相同 pod

[07:46](https://www.youtube.com/watch?v=YXowceUKYJI&t=466s)

| 步驟 | 請求變化 | 講者描述的觀察 |
| --- | --- | --- |
| 第一輪 | 首次傳入 prompt | 沒有 cache hit, 約 3 秒 |
| 第二輪 | 保持相同 system prompt | 路由到相同 pod 並重用 cache, 約 1 秒 |
| 第三輪 | 改用不同 system prompt | 另一個 pod, 沒有 cache hit, 約 3 秒 |
| 後續一輪 | 保持該 system prompt, 改變 user prompt | 再度重用 cache, 約 1 秒 |

這個例子展示 cache 重用與 pod 選擇的關係. 口述沒有完整定義畫面計時的起訖, 因此此處只記為示範時間, 不將其當成精確的 TTFT benchmark 或普遍三倍加速.

## Prefill 與 decode 為何互相干擾

[09:28](https://www.youtube.com/watch?v=YXowceUKYJI&t=568s)

講者將 llm-d 描述為分散式 LLM inference 的控制與編排框架. 除了 router, 它還透過 workload APIs 編排多節點執行, 並以 autoscaling 調整資源. 本段描述的是影片當時的架構定位.

P/D 分離要處理的是兩個階段不同的資源需求:

| 階段 | 工作 | 講者強調的資源特性 |
| --- | --- | --- |
| Prefill | 處理初始 prompt, 建立 KV cache | 偏重計算, 可利用批次平行處理, 負載突發 |
| Decode | 逐個產生後續 tokens | 偏重記憶體頻寬與 cache 駐留, 對延遲敏感 |

在 aggregated serving 中, 同一 pod 同時承擔兩者. 長 prompt 的 prefill 可能干擾正在進行的 decode, 造成串流停頓與抖動. P/D 分離將兩階段放到可獨立擴縮的 workers, 讓資源配置更有針對性.

## P/D 分離的請求流程

[12:03](https://www.youtube.com/watch?v=YXowceUKYJI&t=723s)

1. 請求進入 gateway/router.
2. Endpoint picker 評估叢集狀態, 選擇 prefill 與 decode workers.
3. Prefill worker 處理 prompt, 建立 KV cache 並提供傳輸 metadata.
4. Decode worker 依 metadata, 經網路取得已計算的 KV cache.
5. Decode worker 接續生成 tokens.

```text
Request -> Router / Endpoint picker
                     |
                     v
              Prefill worker
              計算初始 KV cache
                     |
              KV cache 傳輸
                     v
               Decode worker
               持續生成 tokens
```

這個分工將 GPU 上的階段干擾轉成跨 worker 的協調與資料搬移問題. 網路能力因而成為部署決策的一部分.

## 實驗結果: 尾端延遲與 concurrency 必須一起看

[13:08](https://www.youtube.com/watch?v=YXowceUKYJI&t=788s)

講者展示 Red Hat 內部 gpt-oss 測試. 使用 16 張 H100, aggregated 配置為四個 replicas, 每個 tensor parallelism (TP) 為 4; 分離配置為兩個 prefill 與兩個 decode workers, 同樣各為 TP4.

工作負載是多輪互動, 有約 10,000 tokens 的 prefix 與每輪 128 tokens. 口述未明確界定後者是新增輸入或輸出, 因此不據此推算完整序列長度.

講者回報 P99 ITL 從 aggregated 的約 900 ms 降到分離部署的約 100 ms, 曲線也更平滑. 這是特定配置與工作負載的內部結果.

[14:14](https://www.youtube.com/watch?v=YXowceUKYJI&t=854s)

接著的比較包含三類配置:

- Aggregated 加上一般 Kubernetes 排程, 作為基準.
- Aggregated 加上 llm-d cache-aware routing.
- P/D 分離部署.

講者指出, routing 本身就有收益. 在其中一組曲線中, P/D 在中段 concurrency 的優勢最明顯, 低端與高端則接近 aggregated. 這提醒讀者不要只挑一個有利的 concurrency 點代表所有流量.

另一組 64 張 H100 的 prefill-heavy 測試, 平均輸入約 5,000 tokens, 輸出約 500 tokens. Aggregated 使用八個 TP8 replicas, 分離配置為三個 prefill 與五個 decode workers, 各為 TP8. 講者表示這組 P/D 曲線在展示的互動性範圍內優於 aggregated.

兩組結果共同說明收益會隨工作負載與配置改變. 字幕中的 gpt-oss 規模辨識不明確, 本筆記不填入未核對的參數量.

## 何時採用 P/D, 何時保留 aggregated

[15:45](https://www.youtube.com/watch?v=YXowceUKYJI&t=945s)

下表整理講者的決策矩陣, 是起始判斷方向, 不是普遍門檻:

| 條件 | 較值得測試 P/D 分離 | 較值得先保留 aggregated |
| --- | --- | --- |
| 上下文 | 長上下文, 高 ISL/OSL | 短到中等上下文 |
| Concurrency | 測試顯示能受益的中段區間 | 低 concurrency |
| 延遲目標 | 嚴格 ITL 與串流平滑度要求 | 主要追求 TTFT, 可先在合併部署調整 |
| 模型配置 | 大模型, 可運用多種平行策略 | 不需以階段分離處理目前瓶頸 |
| 網路 | 具備 RDMA/RoCE 等高速傳輸條件 | 缺少支援有效 KV 傳輸的網路 |

講者特別強調網路條件. 沒有合適 fabric 時, 建議保留 aggregated. 這是其部署建議, 不應擴張成所有 P/D 實作在其他網路上一律無法運作.

實際排程還要平衡 SLO, 請求速率, cache locality, P/D 比例與網路拓樸. 流量變動時, 固定比例未必持續適用, 因此需要考慮兩個 pools 的獨立 autoscaling 與模型平行策略.

## 進行中的案例: 在 H200 部署 GLM 5.2

[18:10](https://www.youtube.com/watch?v=YXowceUKYJI&t=1090s)

講者表示, 客戶現有資源常是 H200, 因此團隊研究如何在這類硬體上組合 routing, P/D 分離與不同平行策略, 而不只引用 B200 的效能結果.

她描述的架構可配置最多三個 prefill workers 與一個 decode worker. 每個 worker 使用 TP1, data parallelism (DP) 8 與 expert parallelism (EP) 8 的組合, 並以傳輸機制在兩個 pools 間搬移 KV cache. 字幕對傳輸元件名稱辨識不清, 此處不猜測拼字.

在 ISL/OSL 約 45:1 的資料集上, 講者口述以兩個 prefill 與一個 decode worker 得到約四倍更快的 TTFT, 以及約 60% 更多 requests. 影片說明欄則把三對一架構與這些成果連在一起. 因兩者未完全對齊, 本筆記分開記錄可配置架構與口述結果, 不將成果指定為已確認的三對一測試.

此外, 口述未完整交代基準配置與 requests 的統計期間, 不能直接將後者換算成已確認的每秒吞吐量提升.

講者還分享一項近期觀察: 長 prefill 情境中, BF16 KV cache 曾比 FP8 KV cache 更快. 這是仍在探索的結果, 不是低精度 cache 一定較慢的通則. 整個案例在演講時仍持續進行.

## 編輯整理: 部署前的驗證順序

以下是依影片方法整理的實作檢查, 並非講者提供的完整 benchmark 腳本:

1. 收集多輪 traces, 看 ISL/OSL, cache 命中, concurrency 與 session 長度的分布.
2. 先測 aggregated 基準與 cache-aware routing, 分開辨識路由帶來的收益.
3. 若 ITL 抖動仍是問題, 再以可比 GPU 資源測試 P/D 分離.
4. 同時記錄 TTFT, P99 ITL, 吞吐量與 KV 傳輸成本, 避免只最佳化單一指標.
5. 掃描不同 concurrency 與 P/D 比例, 找出符合自身 SLO 的區間.
6. 測試 cache eviction, 冷啟動與流量變化, 確認穩態收益能否延續到實際 session.

## 延伸閱讀

- [KV Cache 與 Paged Attention 如何加速 LLM Inference](2026-06-30-kv-cache-paged-attention-o0gkdZBtwEg.md): 先理解 cache, prefix 重用與 prefill/decode 基礎.
- [掌握 LLM Inference 最佳化](2025-01-01-llm-inference-optimization-9tvJ_GYJA-o.md): 補充延遲, batching 與 parallelism 的整體取捨.

## 來源與限制

筆記依完整英文自動字幕整理, 時間戳採公開章節. 講者直接參與 Red Hat 推論產品與效能工程, 但內部測試仍有產品方立場. 本次未逐格檢視投影片, 未取得原始 benchmark 配置與 traces, 也未重跑實驗.

字幕對部分模型規模, 元件名稱與測試配置有辨識歧義. 本文省略無法可靠確認的精確名稱, 並標示口述與說明欄對 GLM 案例配置的差異. 所有加速與延遲數據都保留為影片中的個別結果, 不代表一般部署保證.

llm-d 功能, 模型版本, 硬體支援與價格均記錄影片當時的敘述, 未作現況驗證. 文中的 ongoing 工作不應直接視為已發布能力.
