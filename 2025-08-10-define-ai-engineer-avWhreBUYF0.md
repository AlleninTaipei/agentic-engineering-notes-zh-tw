# `#define AI Engineer`: Greg Brockman 談工程、研究與 AI 開發的未來

> 來源: [#define AI Engineer - Greg Brockman, OpenAI (ft. Jensen Huang)](https://www.youtube.com/watch?v=avWhreBUYF0)
> 頻道: AI Engineer
> 發布日期: 2025-08-10
> 片長: 41:05
> Video ID: `avWhreBUYF0`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

OpenAI 共同創辦人 Greg Brockman 從個人經歷出發, 說明為什麼程式設計、獨立學習、工程與研究的結合, 是推動 AI 發展的重要力量。訪談後半則轉向 vibe coding、Codex、long-horizon agents、AI 基礎設施與未來開發工作流。

這場訪談的主要觀點是:

1. 程式設計能把腦中的抽象想法變成其他人不必理解內部細節也能使用的成果。
2. AI 研究已不再是研究員提出演算法、工程師只負責實作的線性流程。大型系統需要研究與工程共同探索。
3. Vibe coding 是降低創作門檻的起點, 但 cloud agents 和長時間自主工作會進一步改變軟體工程。
4. 要充分利用 coding agents, codebase 應具備小型模組、快速測試和清楚邊界。
5. 未來不是單一模型完成所有工作, 而可能是成本與能力不同的多模型系統, 再加上大量 domain-specific agents。

## 從數學轉向程式設計

[00:00](https://www.youtube.com/watch?v=avWhreBUYF0&t=0s)

Greg 小時候原本想成為數學家, 希望研究能在數百年的時間尺度上留下影響。高中畢業後, 他撰寫了一本化學教材, 但朋友認為傳統出版不切實際, 建議他改做網站。他因此透過 W3Schools 的 PHP 教學自學網頁開發。

第一個讓他感受到程式設計力量的作品, 是表格排序元件。當點擊欄位後, 畫面真的依照腦中構想重新排序, 他感覺抽象想法第一次直接成為現實。

Greg 將數學與程式設計做了以下對照:

| 數學 | 程式設計 |
| --- | --- |
| 深入理解問題並寫成 proof | 深入理解問題並寫成 program |
| 可能只有少數人讀懂證明 | 可能只有少數人閱讀原始碼 |
| 影響通常透過知識傳播 | 所有人可直接使用程式成果 |

真正吸引他的, 是使用者不必理解內部細節, 仍能獲得想法帶來的效益。

## Stripe 與突破不必要的組織限制

[02:50](https://www.youtube.com/watch?v=avWhreBUYF0&t=170s)

Greg 在大學期間透過共同朋友被早期 Stripe 找到。他先離開 Harvard 前往 MIT, 後來又離開 MIT 加入當時只有約三人的 Stripe, 並在公司從四人成長到約 250 人的階段擔任技術領導角色。

他認為外界容易只記得 Stripe 到客戶現場安裝產品的故事, 卻忽略早期創業的艱難。團隊真正持續做的, 是透過即時聊天與客戶保持密切聯繫, 快速發現並解決問題。

### 24 小時完成銀行整合

Stripe 曾被告知, 與 Wells Fargo 的技術整合通常需要九個月。團隊把工作當作大學 problem set, 在約 24 小時內完成主要實作與測試:

- Greg 負責實作。
- 一名成員從測試腳本頂端往下驗證。
- 另一名成員從底端往上測試。
- Patrick Collison 在認證出錯時維持通話, 為團隊爭取即時修正機會。

團隊第一次驗證失敗, 但在兩小時後重新測試並通過。Greg 將這段經驗解讀為: 不要盲目接受其他組織因歷史流程形成的時間尺度。

這不代表所有限制都能忽略。First-principles thinking 的作用, 是區分真正必要的約束與已不適用的行政成本。AI 提高生產力後, 這種重新檢查流程假設的能力更加重要。

## 獨立學習為何能產生複利

[08:04](https://www.youtube.com/watch?v=avWhreBUYF0&t=484s)

Greg 很早就習慣 independent study。父親先教他代數, 正規課程跟不上進度後, 他透過線上課程在一年內完成約三年的高中數學, 後來再到 University of North Dakota 修讀更進階課程。

這種進度會產生複利: 提前完成基礎課程, 讓後續幾年能投入更高階內容。他學習程式設計的方式也類似, 不是先完整讀完教材, 而是透過建造實際作品持續深入。

Greg 的建議是:

- 有機會探索真正感興趣的題目時, 應該深入追究。
- 興趣不表示過程永遠有趣。
- 遇到無聊或困難階段時, 不要立刻把阻力誤判為方向錯誤。
- 自學的成果來自持續建造與接觸真實回饋。

## 從自學 Machine Learning 到相信 AGI

[10:18](https://www.youtube.com/watch?v=avWhreBUYF0&t=618s)

Greg 在 2013 至 2014 年於 Hacker News 反覆看到 deep learning 進展。他從認識的一名研究者開始詢問, 再經由介紹接觸更多人, 發現許多大學時期最聰明的朋友都進入了這個領域。這讓他意識到, 電腦開始具備過去沒有的實質能力。

離開 Stripe 後, 他自行組裝多張 Titan X GPU 的機器, 透過 Kaggle 和實作學習 machine learning。

### Turing 的 Child Machine

Greg 很早便受到 Alan Turing 1950 年論文〈Computing Machinery and Intelligence〉影響。除了 Turing Test, 他特別關注「child machine」構想: 與其人工寫下所有智慧規則, 不如建立能像兒童一樣學習、再透過獎勵與懲罰成長的機器。

傳統 programming 要求人類理解問題並明確寫出解法。Learning machine 則有機會解決程式設計者自己也無法完整理解的問題, 這對他而言是根本性的轉變。

早期 NLP 研究仍以 parse trees 和手工知識結構為主, 與這個願景有明顯距離。Deep learning 的突破則改變了判斷:

- 2012 年 AlexNet 以 GPU 上的快速 convolution kernels 大幅提升 ImageNet 表現。
- 相似學習方法後來跨入 machine translation、NLP 等領域。
- 原本分立的研究領域逐漸被通用學習方法串聯。

Greg 將這視為延續數十年的 neural networks 路線。關鍵並非神經網路概念突然出現, 而是運算規模終於讓它展現能力。

## AI Engineer: 工程與研究共同推動進展

[16:10](https://www.youtube.com/watch?v=avWhreBUYF0&t=970s)

Greg 曾主張優秀工程師可以和優秀研究者以相同程度貢獻 AI 進展。在訪談中, 他認為這一點今天更加成立。

AlexNet 本身就是研究直覺與工程能力的組合。一方面需要認知到應把方法套用到 ImageNet, 另一方面也要實作高速 GPU convolution kernels。缺少任何一邊, 突破都不會發生。

隨著系統規模增加, 工程不再只是把研究原型做快一點。訓練、資料、分散式系統、可靠性與部署方式都會改變哪些研究實驗可行。研究問題與工程限制因此需要反覆共同演化。

### 不存在永久固定的分工邊界

[18:47](https://www.youtube.com/watch?v=avWhreBUYF0&t=1127s)

工程與研究不能完全分開交接。實際工作更像:

```text
研究假設
  -> 工程系統驗證可行性
  -> 規模化後暴露新問題
  -> 新的研究方向
  -> 新的系統需求
  -> 持續循環
```

AI engineer 的價值, 正是在這個迴圈中理解模型行為、系統限制和產品目標, 再把它們連接起來。

## Scaling 會在不同數量級打破系統

[21:11](https://www.youtube.com/watch?v=avWhreBUYF0&t=1271s)

Greg 表示, scaling 的難處在於每提高一個數量級, 不同元件會以不同方式失效。原本合理的排程、儲存、網路或資源配置, 到下一個規模可能成為瓶頸。

這也會造成研究與產品之間的資源競爭。演講提到, 某些重要產品發布期間需要暫時從研究工作移用 compute。這類做法可以解決短期容量問題, 卻不是希望反覆依賴的常態。

工程團隊必須為新的 scale 重新檢查所有假設, 而不能只把原系統等比例放大。

## Vibe Coding 是入口, Agentic Coding 是下一階段

[24:32](https://www.youtube.com/watch?v=avWhreBUYF0&t=1472s)

Greg 認為 vibe coding 是強大的 empowerment mechanism。它降低了從想法到應用程式的門檻, 讓更多人能透過互動迴圈創造軟體。

但目前對 vibe coding 的理解仍偏向人與模型即時來回、從零建立炫目的小型 app。未來 agentic systems 可能超越這種模式:

- Agent 在 cloud 中持續工作。
- 使用者關閉筆電或睡覺時, 任務仍可進行。
- 不只執行一個或十個副本, 而可能同時調度數百到數萬個 agents。
- 使用方式更接近委派給 coworkers, 而不是盯著每次 token 輸出。

更重要的轉變不是產生全新的 demo, 而是改造既有軟體。Legacy code migration、library upgrade 或 COBOL 系統轉換等工作既困難又不吸引人, 卻具有很高的企業價值。Greg 預期 AI 將從快速做 app, 逐步走向 serious software engineering。

## Codex 如何改變 Codebase 設計

[26:06](https://www.youtube.com/watch?v=avWhreBUYF0&t=1566s)

Coding agent 的效果取決於 codebase 結構。既有系統主要針對人類能力設計, 但模型的能力分布不同:

- 能處理大量、多樣化細節。
- 對深層概念連結與全域架構理解仍可能不穩定。
- 能不厭其煩地重跑測試。
- 在邊界清楚時, 很擅長填入實作細節。

因此適合 coding agents 的 codebase 具有以下特徵:

1. 模組較小, 責任單一。
2. 元件邊界和介面清楚。
3. 每個模組都有可靠測試。
4. 測試能快速執行。
5. 架構關係能以簡潔文件說明。

這些原則其實也是良好軟體工程實務。人類有時能在腦中維持較大的概念模型, 因而容忍缺乏測試或模糊邊界。模型可能執行測試數百或數千次, 因此改善測試速度與模組化的報酬更高。

Greg 將方向概括為: 把 codebase 設計成資淺工程師也能安全工作。這同時有利於模型與人類維護者。

訪談當時, Greg 表示 OpenAI 內部已有低雙位數百分比的 pull requests 完全由 Codex 撰寫。他也強調, 這仍是早期數據, 而且內部不只使用單一系統。

## Long-horizon Agents 的可靠性與 Checkpointing

[29:20](https://www.youtube.com/watch?v=avWhreBUYF0&t=1760s)

當 agent rollout 只有約 30 秒時, 執行失敗後重來的成本不高。如果任務延長到數天, 中途失敗就會成為核心問題。

系統需要逐步提升 checkpoint 能力:

- 保存模型與訓練狀態。
- 保存 cache, 避免重新計算。
- 避免 checkpoint 造成大量複製與阻塞。
- 保存 agent trajectory 和必要的 VM 狀態。
- 判斷外部工具是否能恢復至一致狀態。

語言模型的狀態相對明確, 可以序列化保存。真正棘手的是具狀態的外部工具。例如 agent 已修改遠端服務, 單純重啟模型並不能讓整個世界回到同一時間點。

另一方面, 系統不一定需要完美重現。具備足夠能力的模型有時能接受小幅狀態差異並重新規劃。因此工程上需要衡量 checkpoint 成本、任務價值和模型恢復能力。

## 未來 AI Infrastructure 的兩類負載

[32:06](https://www.youtube.com/watch?v=avWhreBUYF0&t=1926s)

Jensen Huang 透過預錄問題提出兩種差異很大的 workload:

| 負載 | 特性 |
| --- | --- |
| Deep research 與長時間 agents | 大量 reasoning、planning、memory、large context 與 test-time compute |
| 即時多模態 companion | 隨時在線, 語音與視覺互動, 要求極低 latency |

直觀方案是準備兩類 accelerator: 一類偏重 compute throughput, 另一類針對 latency 最佳化, 並分別配置所需的 HBM 與運算能力。

問題在於很難提前預測兩種資源的比例。如果比例錯誤, 可能留下難以利用的 fleet。Greg 認為研究與工程團隊通常能重新設計 workload 來吸收不均衡資源。Mixture of Experts 就可理解為利用大量記憶體容納更多參數, 同時避免所有參數都參與每次計算。

因此 homogeneous accelerators 是穩健起點, 但當資料中心資本支出極高、workload 趨於穩定時, purpose-built accelerators 也可能合理。困難在於研究進展速度可能比硬體規劃週期更快。

## Scaling Bottlenecks 與 Basic Research 回歸

[35:24](https://www.youtube.com/watch?v=avWhreBUYF0&t=2124s)

主持人要求 Greg 排列 compute、data、algorithms、power 和 money 等限制。他沒有給出固定排名, 而是強調所有因素都需保持平衡, 每一天的主要瓶頸都可能不同。

他認為 basic research 再次變得重要。曾經一段時間, 產業看似只需把 Transformer 和 scaling laws 推得更遠。但當 compute 與 data 已被推到很大規模, algorithms 又成為長期進展的重要限制。

Reinforcement learning 是他舉出的例子。GPT-4 雖然語言流暢, 卻仍會以難以預測的方式偏離任務。Greg 的解讀是, 只從資料中觀察世界, 不等於親身在環境中行動並承受結果。RL 為模型提供不同的學習範式, 有助於改善可靠性。

## AGI 時代的開發工作流

[38:06](https://www.youtube.com/watch?v=avWhreBUYF0&t=2286s)

Jensen Huang 的第二個問題是: 當基礎模型愈來愈強, 工程師開發 domain-specific agents 的方式會如何變化?

Greg 沒有假設未來只有單一超強模型。他認為目前證據更偏向由多種模型組成的 menagerie:

- 模型具有不同 inference cost。
- 不同能力和 latency 形成不同 trade-offs。
- Distillation 能把部分能力轉移到更小、更便宜的模型。
- 強模型可以規劃, 再呼叫專門或低成本模型完成子任務。

模型更強不會消除所有應用工程。未來經濟活動包含大量領域、制度和利害關係人。Healthcare 需要責任與安全設計; education 同時涉及學生、教師和家長。這些場景仍需要 domain expertise、workflow、資料治理和產品判斷。

重點也不是把現有工作中的人類比例換成 AI, 而是創造十倍的活動、產出與社會效益。模型能力提高和開發門檻下降後, 人類想建造的系統數量也會增加。

## 實作啟示

### 個人學習與職涯

- 以實際作品驅動自學, 讓成果接受外部回饋。
- 用 first principles 區分真正限制與歷史流程。
- 將工程和研究視為同一迭代循環的不同能力。
- 選擇能長期複利的基礎技能, 而不是只追逐當下工具。

### 為 Coding Agents 整理 Codebase

- 拆分責任單一的小型模組。
- 為每個模組建立快速、可靠的測試。
- 清楚記錄架構、介面和跨模組關係。
- 讓 agent 能在本地獨立驗證結果。
- 為長任務設計 checkpoint、retry 和外部狀態恢復策略。

### 設計 AI 產品與平台

- 同時考慮長時間、高 throughput 與即時、低 latency 負載。
- 不把硬體比例建立在單一、短期 workload 假設上。
- 讓多種模型依成本和能力互相調度。
- 將 domain expertise 和利害關係人需求放入 agent workflow。
- 把 AI 視為擴大可完成工作總量的工具, 不只用來替換既有步驟。

## 核心結論

Greg 對 AI engineer 的定義不是單純會呼叫模型 API 的應用工程師, 而是能把研究直覺、系統工程、產品需求和快速建造結合起來的人。

隨著 coding agents 變強, 寫出局部程式碼會愈來愈便宜。更重要的工作將是建立讓模型可靠發揮的 codebase、測試與基礎設施, 並在特定領域中定義正確問題。工程不會因模型進步而消失, 而會向更大的系統邊界與更高層次的設計移動。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕與公開章節整理, 並非逐字稿。內容已移除口語贅詞、重複字幕、串場與無關宣傳, 以繁體中文重新組織。

訪談中的 OpenAI 內部 pull request 比例、公開 GitHub 使用量及基礎設施經驗, 均為講者在當時的陳述, 本筆記未另行獨立驗證。自動字幕對人名、學術名詞和產品名稱可能有誤辨, 已在上下文足夠明確時修正, 不確定處則避免延伸推論。
