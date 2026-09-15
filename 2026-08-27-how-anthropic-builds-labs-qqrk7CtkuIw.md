# Anthropic Labs 如何建造產品: 大膽委派、兩週實驗與理解導向的 Review

- 影片: [How Anthropic Builds: Lessons from Labs - Mike Krieger, Anthropic](https://www.youtube.com/watch?v=qqrk7CtkuIw)
- 頻道: AI Engineer
- 講者: Mike Krieger, Anthropic Member of Technical Staff、Instagram 共同創辦人
- 發布日期: 2026-08-27
- 片長: 26:10
- Video ID: `qqrk7CtkuIw`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

Mike Krieger 從 Anthropic Chief Product Officer 轉為 Labs 的 individual contributor, 因為他希望直接參與模型時代的產品建造。他描述的工作模式已從「人先拆解任務, 再逐步交給模型」轉向「人說明期望的終點, 模型自行探索方法與取捨, 人再針對結果討論」。

這種更大尺度的委派不等於放棄工程控制。Krieger 分享他如何讓 Claude 在一個週末將數十萬行 Python port 到 TypeScript, 反覆比對、驗證並做到可部署。同時, 他延續 Instagram 時期的原則: 預先量測、建立 feature flags 與 runtime knobs、使用 production data 驗證, 並尋找可漸進遷移的邊界。

Anthropic Labs 的組織也配合快速實驗。每兩週對所有 bets 進行 `persevere or pivot` review, 幾乎每個週期都會關閉專案。人員依實驗臨時組隊, bet lead 通常不直接管理其他成員, 專案有 traction 後才形成較固定的團隊。

當 agents 能產生大型變更時, code review 的瓶頸不只是時間, 而是人是否還能在腦中理解變更。Anthropic 因此讓大型 PR 搭配解釋 intent 與 trade-offs 的 artifact, 再由人透過 Claude 調查問題。程式碼仍需驗證, 但團隊討論逐漸移向設計意圖、取捨與 production measurements。

## 從高階主管回到 Individual Contributor

[00:00](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=0s)

Krieger 加入 Anthropic 的前兩年擔任 Chief Product Officer。他會用 Claude 批評產品策略文件或執行部分 workflow, 但這與直接建造產品的體驗不同。隨著他看見內部團隊使用更強模型, 自己也持續在週末實驗, 最後轉入 Labs 擔任 individual contributor。

他觀察到的模型使用方式也發生變化:

```text
較早的方式
  -> 人先在腦中拆解問題
  -> 把各步驟逐項交給模型
  -> 人密集參與每輪實作

新的方式
  -> 人描述期望終點
  -> Agent 自行探索與執行
  -> 遇到重要選擇時揭露問題
  -> 人檢查結果、理解取捨並決定下一步
```

這也提高了對解說能力的需求。模型完成工作後列出的 trade-offs 可能超過使用者當下能掌握的範圍, 人需要要求模型用更適合自己的抽象層級重新說明。

## Be unreasonable: 不再沿用舊時代的任務尺度

[02:55](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=175s)

Krieger 認為, 目前很多人對 AI 的要求仍過於保守。第一代 AI 產品把模型限制在狹窄介面中, 缺少工具、環境存取與執行能力。使用者因為多次遇到「我不能執行」而養成只委派小任務的習慣, 即使新的 agent 已具備更完整的 environment, 這種直覺仍然存在。

他所說的 `be unreasonable` 不是忽略風險, 而是重新測試任務上限。例如 knowledge worker 表面上未必需要能執行 Bash 的 virtual machine, 但這個環境讓 agent 在內建 PDF parser 失敗時, 可以自行寫 script 改用另一條路徑完成工作。工具自由度增加後, agent 不必在第一個受阻點停止。

### 週末完成 Python 到 TypeScript 的大型 Port

[04:51](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=291s)

Krieger 的 Labs 專案原本以 Python 實作, 規模約數十萬行。為改善使用 Bun 的 deployment story, 他決定將整個系統 port 到 TypeScript。按照過去的工程直覺, 這類重寫成本過高、風險也很大。

他為 Claude 建立動態 workflow, 讓它在週末期間:

1. 進行跨語言轉換。
2. 驗證輸出。
3. 比對兩邊的程式碼與行為。
4. 對問題持續反覆修正。
5. 將結果整理到可部署狀態。

週一時他取得完成的 port。影片沒有展示 workflow 內容、測試覆蓋率、production rollout 或後續缺陷率, 因此這是一個具有第一手性的成功案例, 不是可直接重現的 migration recipe。

## 大規模變更仍需要舊有工程基本功

[06:48](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=408s)

談到是否能對 Instagram 這類 production product 進行相同轉換時, Krieger 沒有主張一次全面替換。他回顧 Instagram 從 Python 2 過渡至 Python 3 與 type hints 時建立的 `MonkeyType`, 透過 production runtime 捕捉實際型別, 再回填至 codebase。

他提出的方向是讓 LLM migration 結合:

- Production data。
- Segmented tests。
- Runtime types 或其他實際行為訊號。
- 可逐步轉換、驗證與 rollout 的邊界。

真正困難的部分不是模型能否一次生成大量程式碼, 而是找出不必一次替換整套系統的 incremental boundary。

### Measure everything and build knobs

Instagram 上線第一週遇到嚴重 scaling 問題。Krieger 從當年的 infrastructure 討論保留兩項長期原則:

1. 預先量測可能需要的訊號。事故發生後才新增 metric, 無法判斷當前數字究竟偏高還是正常。
2. 預先建立 feature flags、rollout controls 與 dynamic configuration, 讓團隊能在數秒內調整行為及負載。

他認為這些原則對 AI 系統同樣重要。模型與產品行為持續改變, 團隊需要能逐步 rollout、比較結果並快速調整 runtime trade-offs。

## Multiplayer delegation: 從個人 CLI 走向共享委派

[08:06](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=486s)

Krieger 描述 Anthropic 內部透過 Slack tagging 將工作交給 Claude。它的重要性不只在非同步執行, 而在於委派行為對其他人可見。類似 Midjourney 早期在 Discord 中的使用方式, 團隊成員能看見別人如何提出更大膽的要求, 從而擴張自己的使用想像。

簡單用法是請 Claude 修正一個 bug。更進階的委派則是讓它長期負責某部分 codebase, 監控 feedback channel, 主動接收任務, 並在 API 改變時跟進修改。此時 agent 不再只是 Slack 中的 Claude Code, 而是具有 context、memory 與 proactive behavior 的非同步 teammate。

這種共享空間也讓 agent 使用方式成為可觀察的組織知識, 不必只靠每位工程師自己摸索 prompt。

## Code review 的瓶頸是理解, 不只是時間

[10:37](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=637s)

Anthropic 對涉及 architecture 的變更仍受 review 限制, 但 Krieger 認為更深層的問題是人是否能完整 conceptualize 大型變更。面對 2,000 行 PR, reviewer 即使抽出時間逐行閱讀, 也不一定能形成清楚的系統模型。

團隊開始讓 PR 搭配 Claude Code artifact, 內容包括:

- 變更的解釋。
- 實作 intent。
- 過程中的主要 trade-offs。
- Reviewer 應優先調查的問題。

Krieger 坦承自己不會逐行閱讀每個 PR。他會針對原本想問的問題與 Claude 對話, 請它調查程式碼, 維持 human-driven、Claude-powered 的 review。對重要架構變更仍採取較嚴格的審查, cosmetic visual changes 則可能接受 `fix forward`。

這顯示 review depth 應與風險相稱。程式碼可透過 tests 與其他工具驗證的部分逐漸自動化, 人類討論則集中在 intent、trade-offs 與 production behavior。

## Anthropic Labs 的兩週實驗組織

[11:53](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=713s)

Labs 每兩週檢查所有 projects, 稱為 `persevere or pivot` review。每項實驗必須決定繼續、轉向或關閉。Krieger 表示幾乎每個週期都有專案被結束, 這不是失敗, 而是 Labs 的設計目的: 快速 prototype、嘗試內部上線或 early access, 沒有效果便停止投入。

如果 org chart 直接跟著每個實驗專案排列, 組織將被迫每兩週重組。因此 Labs 將 people management 與 project leadership 分離:

| 結構 | 責任 |
| --- | --- |
| Bet team | 依當期實驗從 product、engineering 等職能臨時組成 |
| Bet lead / DRI | 對實驗方向與結果負責, 通常不直接管理其他成員 |
| Engineering manager | 負責 coaching、個人成長與協助成員找到適合的工作 |
| 固定產品團隊 | 只有當實驗取得 traction 並進入持續發展後才逐漸形成 |

這種結構使 project shutdown 不必同時變成人事重組。人員可以快速轉移到下一個 bet, 而 managerial support 仍保持穩定。

## 產品應能刪除上一代 AI 的限制

[13:43](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=823s)

談到 Claude Design 的未來時, Krieger 指出不同 Anthropic surfaces 之間尚未充分互通。使用者可能先討論 design, 再把內容手動複製到 coding surface。當 agent 能理解需求並自行選擇工具時, 要求人先判斷應進入 chat、code 或 cowork 的哪個產品入口, 反而形成額外負擔。

在 [15:37](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=937s), 他提到 Anthropic 內部有討論 `project unship` 的 Slack channel。產品團隊必須願意移除上一代模型能力不足時建立的 primitive。例如較為 prescriptive 的 styles, 可能被更靈活的 skills 取代。

這個原則可以表達為:

```text
模型限制產生 workaround
  -> Workaround 形成產品功能
  -> 模型或新 primitive 改善
  -> 重新檢查舊功能是否仍有必要
  -> 移除重複入口與產品複雜度
```

## 為什麼 AI Labs 不會吃掉所有 Startups

[17:34](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=1054s)

Krieger 認為 coding 變快會釋放更多 startup 實驗, 並不代表基礎模型公司會掌握所有產品機會。大型 labs 必須服務廣泛使用者、維護多個產品與既有 integrations, 不可能對每個 vertical 都達到創業團隊的專注程度。

小型團隊仍能建立優勢的地方包括:

- 對特定產業及使用者問題的深入理解。
- 快速接觸、傾聽並迭代使用者需求。
- Distribution 與 adoption。
- Taste 與清楚的產品取捨。
- 對單一問題長期且高度集中的注意力。

部分薄層產品確實可能被轉化為 skill, 不再需要獨立產品。但 Krieger 的核心判斷是, 寫程式從來不是 startup 成敗的唯一限制, 最難的仍是選對問題並獲得使用者採用。

## Vertical AI: 自由生成與可驗證資料的邊界

[20:06](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=1206s)

以 finance 為例, 更強模型可以即時建立分析、dashboard 或 workflow, 但企業不會接受所有內容都完全 free-form。系統需要把可變的 agentic application 與較穩定、可驗證的資料層結合。

講者提到的重要要求包括:

- Verifiability。
- Audit logging。
- Data provenance。
- 經過確認的資料來源。
- 不過度限制上層應用的彈性。

既有金融系統通常為 auditability 設計, 卻未必適合高彈性的 agentic workloads。產品機會可能同時存在於上下兩層: 一方面改善可靠的資料與稽核基礎, 另一方面讓 agents 在其上自由組合分析與流程。

## Burnout 不是交付速度的附帶問題

[22:00](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=1320s)

AI 產業的發布與競爭週期遠比早期社群產品密集。模型、產品、競爭者與監管消息可能在同一週多次改變, 若個人把自我評價綁在每次發布的市場反應上, 很容易陷入持續的 `it is so over / we are so back` 循環。

Krieger 建議刻意安排離線時間。他強調沒有任何工作重要到不能離線幾天; 若一個人真的無法離開, 應尋求 mentor 或團隊協助消除單點依賴。Burnout 一旦發生, 復原可能需要很長時間。

他也用運動競賽比喻長期視角: 一個人不會等同於自己表現最好或最差的那一場。AI 是快速變動但長期的賽局, 團隊需要建立能穿越短期高低起伏的文化。

### 將情緒說出口

[25:04](https://www.youtube.com/watch?v=qqrk7CtkuIw&t=1504s)

Krieger 從 coach 得到的一項領導建議是, 自己感受到的壓力、難過或挫折, 團隊其他成員往往也正在經歷。領導者直接說出「我對關閉這個實驗感到難過與挫折」, 能允許其他人表達相同情緒。承認現況後, 團隊才比較容易進入「接下來要怎麼做」的討論。

這與 Labs 頻繁關閉實驗的制度相互配合。把 shutdown 正常化不能只靠流程名稱, 還必須讓人能處理投入沒有轉化為產品的失落。

## 可採用的實務原則

以下是依影片內容整理的操作方式:

1. 定期選擇一項原本因成本過高而不考慮的工程工作, 用當前 agent 能力重新評估可行性。
2. 大型 migration 不只要求生成, 還要提供比對、測試、production signals 與漸進 rollout 的邊界。
3. 讓團隊能看見彼此如何委派 agents, 將高品質 delegation 變成共享知識。
4. 大型 PR 同時交付 intent、trade-offs 與風險導向的 explainer artifact。
5. 依變更風險調整 review depth, 不把 cosmetic change 與 architecture change 使用同一套門檻。
6. 對實驗設置固定的 continue、pivot 或 stop cadence, 並將人員管理與短期 project leadership 分離。
7. 模型或 primitive 升級後, 重新檢查產品入口與舊 workaround 是否應刪除。
8. 在 vertical AI 中分離可自由生成的 application layer 與可驗證、可稽核的 data layer。
9. 為真正的離線時間及人員備援建立制度, 不把持續在線視為高績效。

## 證據與限制

Krieger 直接參與 Anthropic Labs 的產品與工作流程, 對 Labs 組織、內部 agent 使用與個人 migration 案例具有第一手性。他也是 Anthropic 員工, 對產品能力、公司文化與 startup 生態的描述存在公司視角與推廣誘因。

Python-to-TypeScript port、Slack delegation 與 artifact-based review 都是具體案例, 但影片沒有提供原始 code、workflow、測試結果、缺陷資料、成本或 production impact。`persevere or pivot` 則是組織設計說明, 不是已證明優於其他研發模式的比較研究。

影片中的模型、Labs projects 與 Anthropic product surfaces 反映發布時狀態, 可能快速變動。部分較新的產品名稱由英文自動字幕辨識不穩定, 本文只保留能從 metadata 或上下文可靠確認的名稱, 並避免把不確定名稱當作核心事實。
