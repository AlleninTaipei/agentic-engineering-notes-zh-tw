# 從 Ambient Documentation 到 Clinical Intelligence

> [!info] 來源
> - 影片: [From Ambient Documentation to Clinical Intelligence — Chaitanya Asawa, Abridge](https://www.youtube.com/watch?v=u6q-byPWUuo)
> - 頻道: AI Engineer
> - 發布日期: 2026-08-19
> - 片長: 21:35
> - Video ID: `u6q-byPWUuo`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

Abridge 先以減少臨床文件負擔切入醫療體系, 再把看診對話、EHR 與醫學文獻結合成臨床智慧。要在高風險、大規模環境落地, 評估必須成為產品的 operating system, 並透過專家 rubric、分階段發布、小模型專門化與事件式路由共同控制品質、延遲及成本。

## 為什麼醫療是困難且重要的技術領域

講者 Chaitanya Asawa 希望反駁「醫療技術問題不夠前沿」的印象 ([02:08](https://www.youtube.com/watch?v=u6q-byPWUuo&t=128s))。醫療確實仍依賴 fax 等舊系統, 但 AI 一旦進入真實診療, 同時面對高風險、複雜 context、即時互動、成本與嚴格整合需求。

他回顧自己從 robotics、enterprise search 到 healthcare 的經歷, 並指出醫療長期面臨成本增加、行政生產力停滯、醫院低利潤或關閉、病患債務與臨床人員 burnout 等問題。

## Clinical documentation 是切入點

每次看診後, 臨床人員都要產生 clinical note, 常見格式是 SOAP note。許多人每天另外花約兩小時在下班後完成文件, 又稱 pajama time ([07:04](https://www.youtube.com/watch?v=u6q-byPWUuo&t=424s))。

這些 note 不只是行政文件, 還會影響:

- Billing 與 audit。
- 下一位臨床人員理解病患狀況。
- 跨醫療體系的 longitudinal record。
- 後續治療與風險判斷。

Abridge 以 ambient documentation 為 wedge, 從醫病對話產生個人化 clinical note。講者表示此產品已進入約 300 個美國大型 health systems。這是公司在影片中的規模陳述, 並非本文獨立查核。

## 所有下游工作都源於對話

Abridge 的核心 thesis 是, 醫療工作流圍繞醫病對話 ([08:48](https://www.youtube.com/watch?v=u6q-byPWUuo&t=528s))。Clinical note、billing、trial matching、decision support 與 order 都是對話下游產物。

產品方向因此從「把 conversation 轉成 note」擴展為完整 visit lifecycle:

- Pre-visit: 準備可能需要討論的臨床或行政議題。
- In-visit: 聽取即時對話, 回答臨床問題或辨識 order。
- Post-visit: 產生 clinical note、patient summary, 並準備待簽核 order。

影片示範醫師以語音詢問病患是否符合 clinical trial, 系統根據條件指出仍需新的 echocardiogram, 接著依語音要求建立 order ([09:38](https://www.youtube.com/watch?v=u6q-byPWUuo&t=578s))。

## 系統需要讀取的三類 context

要完成這類工作, 系統結合 ([10:27](https://www.youtube.com/watch?v=u6q-byPWUuo&t=627s)):

1. EHR context: 既有病歷、lab、過去 note 與長期健康紀錄。
2. Live conversation: 病患目前的症狀、變化與醫師判斷。
3. Medical literature and guidelines: 作為推理與回答的專業依據。

Context 的重要性同時構成隱私與安全責任。資料範圍、來源、時效性、授權與引用都必須被治理。

## Agentic healthcare 的三個 KPI

影片將一般 agentic product 的三項指標帶入醫療 hard mode ([11:27](https://www.youtube.com/watch?v=u6q-byPWUuo&t=687s)):

| KPI | 醫療情境的特殊要求 |
| --- | --- |
| Quality | 錯誤可能造成臨床後果及信任崩潰 |
| Latency | 必須在看診中的正確時機提供資訊 |
| Cost | 大量即時醫療對話使每次推理成本被放大 |

三者不能只靠單一 frontier model 同時解決, 系統需要任務分解、評估與分層模型策略。

## Evals 是產品的 operating system

Abridge 把 evals 視為產品運作核心, 而不是發布前最後一步 ([13:08](https://www.youtube.com/watch?v=u6q-byPWUuo&t=788s)):

```text
內部 benchmark 與 offline evaluation
                ↓
可信任臨床人員的小型 alpha
                ↓
beta / A-B testing / staged rollout
                ↓
正式上線後 continual monitoring
```

Offline dataset 無法完整代表真實診療, 因此必須逐步接觸 reality, 控制曝險範圍並觀察線上訊號。

## 把 clinician judgement 編碼進 LM judges

公司內的 clinician 參與建立與校準 LM judges ([13:57](https://www.youtube.com/watch?v=u6q-byPWUuo&t=837s))。Judge 應反映臨床人員真正期待的產品行為, 讓非臨床工程師也能在明確回饋下改善模型、agent architecture 或 search ranking。

線上訊號還包含:

- 臨床人員如何修改產生的 note。
- Thumbs up/down 與 star rating。
- Free-form feedback。

這些訊號不能單獨證明臨床正確性, 但能補足離線 benchmark 看不到的使用行為。

## Contextual clinical decision support

影片以「病患是否符合 febrile neutropenia 標準」為例 ([14:48](https://www.youtube.com/watch?v=u6q-byPWUuo&t=888s))。問題本身未提供足夠資訊, 系統需要從 EHR 取回 lab, 結合即時對話, 再以 clinical guidelines 與 medical journals 作為推理來源。

這類任務的困難在於 generator-verifier gap 很小 ([15:38](https://www.youtube.com/watch?v=u6q-byPWUuo&t=938s))。Sudoku 的答案難產生但容易驗證, 臨床回答則連「判斷答案是否正確」本身都需要高度專業。如果 verifier 足夠強, 它可能已接近 generator 的能力。

## 用多個訊號評估難以驗證的答案

團隊不依賴單一 judge, 而是從不同面向建立訊號:

- Clinical quality judge。
- Boundary 與 adversarial judge。
- Clinical safety judge。
- Tone、style 等 product judge。

每個 judge 只捕捉問題的一部分, 組合後才較能覆蓋高風險輸出。

## 四位 physician 建立一份 rubric

Clinical quality judge 的 reference 不是唯一標準答案, 而是好答案應包含的元素 ([16:27](https://www.youtube.com/watch?v=u6q-byPWUuo&t=987s))。

影片描述的建立流程:

1. 從真實 clinical cases 建立問題。
2. 兩位獨立 physician 各自撰寫 rubric。
3. 第三位 physician adjudicate 兩份結果, 建立最終 rubric。
4. 第四位 clinician 進行 QA。
5. LM judge 將 agent response 與 rubric elements 做語意比對。

Rubric 描述必要要素, 而不是要求每次回答與 golden response 字面相同。這容許臨床回答存在合理表達差異。

## 大規模下的成本問題

講者表示系統以每年約一億次 medical conversations 的 run rate 運作 ([18:05](https://www.youtube.com/watch?v=u6q-byPWUuo&t=1085s))。在此規模下, 每個 token、每次 model call 與 latency 都會累積為重大成本。

這個數字是公司於影片中的陳述, 沒有在字幕中附獨立驗證資料。

## 將 clinical note 拆給專門小模型

Clinical note 包含 history of present illness、past medical history、assessment and plan 等不同 section。團隊不一定用單一 foundation model 生成全部內容, 而是把問題拆成較小 workflow, 對不同 section post-train 專門小模型 ([18:56](https://www.youtube.com/watch?v=u6q-byPWUuo&t=1136s))。

這個策略的理由是:

- 特定任務不需要 frontier-level intelligence。
- 專門模型可降低 latency 與 cost。
- 大量領域資料可能讓專門模型在狹窄任務上維持品質。
- 可針對每個 section 分別評估與更新。

講者提出「right to win」概念: 若團隊擁有獨特資料與專注場景, 模型改善速度可能在特定問題上追上或超越通用模型的進步。

## 事件式 routing 避免每秒呼叫大型模型

在看診過程中, 系統會辨識醫師口頭提出的藥物或非藥物 order, 先準備後讓醫師在 EHR 中簽核 ([19:49](https://www.youtube.com/watch?v=u6q-byPWUuo&t=1189s))。

若每幾秒都用大型模型掃描完整對話, 成本會過高。團隊使用多層較便宜、快速的 gate, 先找出可能發生 order 的事件, 再觸發較大型模型完成匹配與端到端處理。

```text
即時對話
   ↓
低成本事件偵測 gate
   ↓ 只有疑似 order 時
較大型模型理解並匹配核准項目
   ↓
建立待簽核 order, 由醫師確認
```

## 實務檢查表

- 產品是否從明確、高價值且可評估的 workflow 切入?
- 是否同時管理 EHR、即時對話與醫學依據的來源及權限?
- Quality、latency、cost 是否各有獨立指標?
- 是否在開發前建立 internal benchmark?
- Rollout 是否從 alpha、beta 到 continual monitoring 分階段進行?
- LM judge 是否由 domain expert 校準?
- 難以驗證的任務是否使用多種互補 judge?
- Rubric 是否描述必要元素, 而非限制唯一措辭?
- 工作是否能拆給較小、專門的模型?
- 是否以低成本 gate 只在必要事件觸發大型模型?
- 臨床 order 與高風險行動是否保留專業人員簽核?

## 時效性與限制

影片發布於 2026 年 8 月。Abridge 的客戶規模、conversation run rate、產品功能與模型策略均可能更新。本文記錄的是講者當時陳述, 不是對醫療效果、安全性或法規符合性的獨立評估。

本筆記依英文原始自動字幕與 YouTube 章節整理。涉及醫療名詞、研究、機構數字或臨床用途時, 應查閱正式產品文件、臨床驗證資料與適用法規。本文不能取代醫療或合規專業意見。
