# Forward Deployed Engineering 101

> 影片: [Forward Deployed Engineering 101 - Kevin Bai, Anthropic, ex Palantir & Rippling Founding FDE](https://www.youtube.com/watch?v=KwhgfwOSToQ)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=KwhgfwOSToQ  
> 發布日期: 2026-07-28  
> 片長: 17:48  
> Video ID: `KwhgfwOSToQ`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Anthropic Applied AI 團隊成員 Kevin Bai 以 Palantir 和 Rippling 的經驗, 說明 Forward Deployed Engineering, 簡稱 FDE, 為何存在及何時適用。

FDE 不是一般顧問服務, 也不是替每位客戶從零開發客製軟體。它由具備客戶溝通能力的軟體工程師, 在可重用的平台和 shared primitives 上, 與客戶共同建立能交付商業成果的解決方案。

講者提出兩項導入前必答問題:

1. 公司是否必須把技術複雜的產品賣給非技術買家?
2. 公司是否已擁有, 或願意投資建立可供 FDE 重用的平台?

若任一答案是否定, FDE 可能不是合適的 go-to-market motion。

## FDE 的歷史背景

Kevin Bai 在 Anthropic Applied AI 團隊任職。在此之前, 他是 Rippling FDE 團隊的第一位成員, 並表示該團隊在一年內成長至約 25 人。他更早曾在 Palantir 工作。[00:00](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=0s)

這場演講從 Palantir 的商業模式出發, 解釋 FDE 如何成為銷售技術平台的 go-to-market function, 再將這套思路延伸至其他公司。

## Palantir Foundry 面臨的銷售問題

依講者說明, Palantir Foundry 是一套讓大型組織集中資料, 建立 ontology 並在其上建造應用程式的平台。[01:47](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=107s)

Ontology 的作用是將分散資料轉成業務可理解的實體。例如, 與其保留多張名稱不明確的資料表, 組織可以建立一個代表 warehouse 的單一真實來源, 再用這些實體建立應用程式。

但只向產業主管說明資料集中和 ontology, 並不能直接回答他最關心的問題:

- 商品能否取得更多貨架位置?
- 銷售 throughput 能否提高?
- 營運效率能否改善?
- 投資這套平台後何時能產生商業結果?

平台成功還取決於客戶是否會用。客戶除了購買軟體, 還要訓練員工, 理解平台並自行建立應用。若採用成本過高, 強大的平台也可能無法轉換成成果。

## 客戶買的是 Outcome, 不是 Product 或工時

Palantir 的解法是把產品和服務結合, 但不把它們分別出售。客戶既不是只買一套軟體, 也不是只購買工程師工時, 而是購買可驗收的結果。[03:14](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=194s)

FDE 會:

1. 深入理解客戶業務.
2. 找出真正需要改善的 outcome.
3. 使用平台既有能力組裝解決方案.
4. 與客戶共同部署和調整.
5. 以商業成效證明平台價值.

資料組織, schema 與其他技術細節對客戶而言只是 implementation details。FDE 的工作是把平台能力翻譯成客戶實際關心的成果。

## 何時真的需要 FDE

講者用兩個維度判斷是否需要 FDE: 產品的技術複雜度, 以及買家的技術能力。[04:18](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=258s)

| 產品 | 買家或使用者 | 較適合的 GTM |
| --- | --- | --- |
| 技術複雜 | 技術型 | Developer relations, solution engineering 或 developer-led motion |
| 較標準化或主要靠設定 | 非技術型 | 傳統 sales-led SaaS motion |
| 技術複雜且高度可開發 | 非技術型 | FDE 特別有價值 |

GitHub 或 Datadog 等複雜工具通常賣給 CTO, CIO 和工程師。買方有能力理解技術, 並把產品整合進日常工作。

Rippling, Jira 或 Slack 也可能有複雜功能, 但主要依靠 configuration, 不要求客戶在平台上大量開發。因此非技術買家也能採用。

FDE 最適合第三種組合: 平台很複雜, 客戶卻沒有足夠工程能力自行把平台轉成解決方案。

## 為何 Palantir 需要這種模式

Foundry 是 application-building platform。大型科技公司通常已有深厚工程團隊, 能自行建立內部應用, 因此不一定需要外部平台和 FDE。[05:14](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=314s)

相較之下, 油氣, 消費品或其他大型產業公司的核心專長不是軟體開發。它們可能有重要資料與高價值問題, 卻缺少足夠工程深度將平台轉為日常應用。

FDE 相當於提供一組客戶不必自行招募, 訓練, 管理與留任的工程能力。這些工程師同時熟悉平台, 並與客戶密切合作理解產業問題。

## 商業價值: 提高合約規模

講者用 Average Contract Value, 簡稱 ACV, 說明 FDE motion 可能帶來的大型合約。[05:59](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=359s)

他表示, 依自己當時查看的公開 SaaS 公司資料:

- Palantir 的 ACV 約為 400 萬美元.
- ServiceNow 約為 120 萬美元.
- Workday 約為 60 萬美元.
- 其他公開 SaaS 公司沒有超過 50 萬美元.

這些數字是講者在演講中的概略比較, 字幕沒有提供資料日期, 樣本定義或原始來源。本筆記不將它們視為已獨立驗證的財務資料。

背後的商業邏輯是, 單純販售軟體功能通常只能取得軟體價值的一部分。若供應商能對 outcome 負責, 並直接參與解決高價值業務問題, 客戶願意承諾的合約規模可能更大。

## FDE 是可規模化的 Design Partnership

早期 B2B startup 常透過 design partnership 找到 product-market fit。公司與少數客戶密切合作, 客戶提供問題 context, 團隊投入時間, 技術和資源建立解決方案。[07:16](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=436s)

講者將 FDE 定義為把 design partnership 延伸到 enterprise scale。Palantir 的核心假設是, 這種高度合作不必只存在於創業初期, 也可以成為長期的企業 go-to-market 模式。

但要讓它可規模化, 必須解決客製維護問題。

## FDE 與 Dev Shop 的分界

若每位 FDE 都為每位客戶從零開發一套獨立軟體, 那就不是可規模化的 FDE function, 而是 dev shop。[08:05](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=485s)

Dev shop 本身可以是有利潤的商業模式, 但它面臨:

- 每位客戶都有不同 codebase.
- 維護成本隨客戶數量線性或更快增加.
- 知識集中在個別工程師身上.
- 工程師必須學習大量 repositories.
- 客製成果難以轉化為產品改進.
- 人力成本最終侵蝕 P&L.

真正的 FDE 應建立在平台與 shared primitives 上。工程師不是從零開始, 而是組合既有元件, 建立對客戶有高價值的 application, workflow 或 solution。

可以將差異整理如下:

| 面向 | FDE | Dev shop |
| --- | --- | --- |
| 建造基礎 | 共用平台與 primitives | 常為客戶從零建立 |
| 客製範圍 | 組合與局部延伸 | 大量獨立客製 |
| 重用程度 | 高 | 通常較低 |
| 產品回饋 | 可形成新 primitives | 容易停留在單一客戶專案 |
| 擴張限制 | 平台能力與 FDE 效率 | 人力和維護成本 |

## 導入 FDE 前的兩個問題

講者要求公司先問 "need" 而不是 "want"。FDE 受到關注並不代表每家公司都應建立這個職能。[09:48](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=588s)

### 1. 是否必須把複雜技術賣給非技術買家?

公司是否真的面臨只有 FDE 才能解決的特殊 GTM 問題?

若買家本身是技術團隊, developer relations 和 developer engagement 可能更有效。若產品是標準 SaaS, 傳統 sales-led growth 可能足夠。

### 2. 是否有平台, 或願意投資建立平台?

FDE 是否能在 shared primitives 上工作? 如果沒有, 公司是否願意長期投資平台化?

若答案是否定, 每個客戶專案都會製造新維護負債。即使短期營收快速增加, 後續維護與人力成本仍可能使模式失控。

## AI 時代改變了什麼

Kevin 認為, FDE 再次受到關注不只是市場終於理解 Palantir, 而是軟體產業的產品性質已改變。[11:34](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=694s)

AI 讓寫程式與建立高度可調整軟體變得更容易。大量產品成為 agentic platforms, 而 agentic 往往意味著可客製。

這造成新的採用問題:

- 平台能做的事情更廣, 更難用簡短功能清單說明.
- 客戶未必知道應該如何配置 agent 解決自己的問題.
- 產品成功更依賴客戶的 implementation 能力.
- 公司向 enterprise 或新垂直市場擴張時, 需要把抽象能力轉成具體成果.

因此, 更多 AI 公司可能落入 "技術複雜產品賣給非技術買家" 的象限。若把成功完全交給客戶自行實作, adoption 和 expansion 都可能受阻。

這是講者的產業假說, 不代表每個 agentic product 都必須建立 FDE 團隊。是否需要仍取決於前述兩個判斷問題。

## Shared Primitives 應有多細

Q&A 中有人問 shared primitives 的 granularity。Kevin 的回答是依產業與使用者而定。[13:08](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=788s)

有些產品可以先完成 60%, FDE 只客製剩餘 40%。其他產業則需要非常 granular 的 configuration 和 tooling。

講者以 AWS 為類比。AWS 服務的客群極廣, 因此提供 DynamoDB 等較底層的 primitives, 讓工程師不必從硬體和資料庫引擎開始建造, 又保留足夠組合空間。

選擇 primitive granularity 時可考慮:

- 客戶需求差異有多大.
- 哪些能力在多個客戶間穩定重複.
- FDE 是否能在合理時間內組合出解法.
- 平台團隊能否維護這些 abstractions.
- Primitive 是否過於特定, 只服務單一客戶.
- Primitive 是否過於底層, 使 FDE 仍需重建大量基礎功能.

## 客戶專案如何回饋平台

講者提出一項產品邊界原則。[15:16](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=916s)

- 只對單一客戶有意義的需求, 可以保留在 forward-deployed solution 中.
- 能泛化到多個客戶的能力, 長期應回收到核心平台.

FDE 因此也是產品探索機制。它在客戶現場先發現問題, 驗證解法, 再辨識哪些模式值得成為新的 primitives 或 product services。

這形成一個循環:

```text
客戶問題
  → FDE 建立解法
  → 觀察重複模式
  → 平台團隊泛化能力
  → 新 primitives 降低後續交付成本
  → FDE 能處理更高階問題
```

若公司只讓 FDE 不斷交付專案, 卻沒有把重複能力回收到平台, 維護負擔仍會逐漸接近 dev shop。

## 團隊協作與單點失敗

Kevin 鼓勵多位 FDE 共同參與客戶專案, 避免只有一人掌握全部 context。若唯一負責人休假或離職, 團隊可能無法維護解決方案。[14:25](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=865s)

這表示 FDE 團隊仍需要標準工程實務:

- Shared ownership.
- Code review.
- 文件與 handoff.
- 共用 repositories 和 conventions.
- 客戶 context 的結構化保存.
- 平台元件與客製層的清楚邊界.

外部合作夥伴也能參與類似模式, 但需像管理 contractors 一樣明確界定責任與交付界面。

## 理想 FDE 人才輪廓

講者用一句話描述 FDE: "customer-facing software engineer"。[17:08](https://www.youtube.com/watch?v=KwhgfwOSToQ&t=1028s)

候選人首先必須達到公司聘用軟體工程師的技術標準, 同時又是團隊願意信任其直接面對客戶的人。

這個角色需要同時具備:

- 能在平台上快速建立可靠解法的工程能力.
- 能從模糊對話中理解業務目標的需求探索能力.
- 能向非技術買家解釋 trade-offs 的溝通能力.
- 能在現場迭代又不破壞平台邊界的產品判斷.
- 能辨識 bespoke request 與 generalizable primitive 的抽象能力.
- 對客戶 outcome 負責, 而不只是完成 tickets.

講者沒有給出更細的固定履歷模板, 因為不同公司的客戶, 平台和銷售模式會影響理想人才。

## 編輯整理: 建立 FDE Function 的實務框架

以下是依演講內容整理的執行框架, 不是講者逐字提供的完整 playbook。

### 第一階段: 驗證必要性

1. 描述產品中最難被非技術客戶理解與實作的部分.
2. 確認現有 solution engineering, professional services 或 sales 是否真的無法解決.
3. 定義客戶願意付費的 business outcome, 不以功能部署作為唯一成功標準.

### 第二階段: 建立平台邊界

1. 盤點已存在的 shared primitives.
2. 定義哪些元件由核心平台團隊維護.
3. 限制客戶專屬程式碼的範圍.
4. 建立把重複模式回收至平台的正式流程.

### 第三階段: 小規模 Design Partnership

1. 選擇問題價值高且願意密切合作的客戶.
2. 由至少兩人共享專案 context.
3. 記錄交付時間, primitives 重用率與後續維護成本.
4. 以 outcome, adoption 和 contract expansion 評估成功.

### 第四階段: 規模化

1. 把成熟解法轉成 templates, primitives 或 platform services.
2. 建立工程 review, security, observability 與 handoff 標準.
3. 追蹤 FDE 人力增加是否帶來非線性維護負擔.
4. 讓 FDE 洞察持續影響核心產品 roadmap.

## FDE 與相鄰職能

不同公司對職稱的用法可能不同, 但依影片論點可做如下區分:

| 職能 | 主要責任 | 成功衡量 |
| --- | --- | --- |
| Software engineer | 建立核心產品與平台 | 品質, 可靠性, roadmap delivery |
| Solutions engineer | 協助技術評估, integration 與售前 | Technical win, deployment readiness |
| Professional services | 依範圍完成導入或專案 | Project delivery, utilization |
| FDE | 在平台上與客戶共建 outcome, 並回饋產品 | Outcome, adoption, ACV, reusable learning |
| Dev shop engineer | 為客戶建立專屬軟體 | Project margin, client delivery |

這些邊界不是業界統一標準。實際組織設計應根據權責, 平台成熟度和商業模式判斷, 不應只依職稱分類。

## 重點回顧

- FDE 適合把技術複雜的平台賣給缺乏實作能力的非技術買家.
- 客戶購買的是 outcome, 不只是產品授權或工程師工時.
- FDE 是可規模化的 enterprise design partnership.
- 可規模化的前提是 shared platform 和 reusable primitives.
- 若每位客戶都從零開發, 公司經營的是 dev shop, 不是平台型 FDE.
- 導入前必須確認公司真的需要 FDE, 並願意投資平台.
- AI 使產品更可客製, 同時也增加客戶理解與實作的難度.
- Bespoke work 可留在客戶層, 可泛化能力應回收到核心平台.
- FDE 的基本人才定義是具備客戶溝通能力的軟體工程師.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 字幕多次將 Palantir, FDE, FTE, DevRel, SaaS 和 Datadog 辨識成相似詞, 本筆記只在上下文足以確認時修正.
- YouTube metadata 標題中的分隔符出現編碼異常, 本筆記將其正規化為 `Forward Deployed Engineering 101 - Kevin Bai, Anthropic, ex Palantir & Rippling Founding FDE`.
- 影片中的 ACV 數字由講者概略陳述, 未附資料來源和計算方法, 本筆記未獨立驗證.
- FDE, solutions engineering, professional services 與 dev shop 的職責在不同公司可能重疊. 本筆記呈現講者的框架, 不是業界強制分類.
- AI 平台將普遍需要 FDE 是講者的產業假說. 個別公司仍應根據買家, 產品複雜度與平台成熟度判斷.
