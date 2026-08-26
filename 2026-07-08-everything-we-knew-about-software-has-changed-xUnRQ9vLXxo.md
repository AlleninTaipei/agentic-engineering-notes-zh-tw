# 軟體開發的舊假設已經改變: 從做得更深到想得更廣

> 影片: [Everything we knew about software has changed — Theo Browne, @t3dotgg](https://www.youtube.com/watch?v=xUnRQ9vLXxo), AI Engineer<br>
> 頻道: AI Engineer<br>
> 上傳日期: 2026-07-08<br>
> 片長: 16:01<br>
> Canonical URL: https://www.youtube.com/watch?v=xUnRQ9vLXxo<br>
> Video ID: `xUnRQ9vLXxo`<br>
> 內容依據: 英文原始自動字幕

## 摘要

Theo Browne 認為, coding models 已從可靠呼叫工具, 演進到承接長時程任務, 再進一步自行拆解、協調與驗證工作. 如果工程師仍只拿新模型處理過去大小的 Jira ticket, 就不會真正感受到能力提升. 面對模型進步速度超過個人技能增長的情況, 他的建議不是單純「變得更強」, 而是挑戰更大的問題.

然而, 「更大」不只是替既有產品增加深度. 過去新創公司往往必須在狹窄領域做得比大型平台更深, 因為小團隊無力涵蓋廣泛功能. 當代理能大幅降低實作成本, 小團隊開始有機會建立更寬的產品範圍, 再以可擴充架構讓使用者自行補足特定垂直功能.

這是一場帶有強烈個人觀點的 closing keynote. 它的核心不是證明一個人已能重建 AWS 或 Salesforce, 而是要求工程師重新檢查過去對工具、程式碼成本、專案規模與競爭邊界的假設.

[00:00](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=0s) 講者以「AI psychosis」形容面對模型快速進步時的興奮與認知衝擊. 這是演講中的幽默修辭, 不是醫療用語的正式使用.

## 1. 從 tool calling 到長時程工作與 orchestration

[00:50](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=50s)

講者以個人使用體驗, 將模型能力分成三個階段:

| 階段 | 影片中的代表模型 | 講者感受到的主要能力 |
| --- | --- | --- |
| Tool-call era | Sonnet 3.5 | 在程式碼庫脈絡中穩定使用工具, 完成多步驟日常 coding 工作 |
| Long-running task era | Opus 4.5 | 長時間保持目標, 自行測試並把工作推進到較完整的狀態 |
| Orchestration era | Mythos | 理解自身能力, 拆分任務, 呼叫其他模型或代理並驗證結果 |

這不是正式分類, 而是講者用來描述體感差異的框架. 他的重點是, 新模型的價值未必會出現在相同的短 prompt 上. 如果任務沒有要求規劃, 持續執行或協調, orchestration 能力便沒有發揮空間.

影片提到的模型名稱與能力屬於發布時的產品語境. 其中 `Mythos` 與 `Fable` 的確切版本定位無法單靠字幕完整確認, 不應把這段視為跨供應商的客觀模型排名.

## 2. 不只是讓模型做舊工作, 而是提高任務上限

[03:04](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=184s)

講者回顧過去工作中的多數 Jira tickets, 認為它們已能被較早一代的長時程模型處理. 如果繼續把工作切成同樣大小, 更強的 orchestration model 不會產生明顯新增價值.

因此, 他提出「go bigger」:

- 給模型更完整的成果目標, 不只是一個修改步驟.
- 允許模型拆解, 平行處理與驗證.
- 選擇過去因工程成本過高而放棄的問題.
- 用實驗找出新能力的真正邊界, 而不是沿用舊評估尺度.

這個觀點也帶有必要限制. 任務變大會同時放大錯誤範圍, 審查成本與權限風險. 更大的委派應配合清楚規格, 可觀察進度, 獨立驗證與安全邊界, 而不是只延長自主執行時間.

## 3. 資深工程師也受到既有身份與工具束縛

[03:35](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=215s)

長期從事軟體開發會累積有價值的判斷力, 同時也會形成難以察覺的慣性. 講者舉出 terminal, tmux, SSH, Git, Vim, 語言與 framework 等工具身份, 說明工程師容易把熟悉誤認為必要.

他要求重新詢問:

```text
我們這樣做, 是因為它仍是正確的設計?
還是只因為過去一直如此?
```

影片以 `.env` 檔案為挑釁案例. 為何團隊能透過 Git 分享大部分專案檔案, 卻必須另建系統傳遞這一份設定? 講者不是主張直接提交 secrets, 而是在質疑現有工具是否迫使團隊接受額外複雜度.

這個問題若落到實務, 應導向更安全的 secrets management, environment provisioning 與權限設計, 而不是把機密寫入版本控制.

## 4. 軟體開發仍處於 skeuomorphic 階段

[06:08](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=368s)

Skeuomorphism 是讓新媒介模仿舊工具外觀的設計方式. 講者以 iOS 6 到 iOS 7 的轉變比喻目前的 AI software development:

- 早期電子書模仿實體書架與翻頁, 是為了說服使用者數位裝置能取代舊工具.
- 當使用者已接受手機的功能, 介面便能停止模仿實體物件, 改為直接服務數位使用情境.
- 今天的 AI coding 工具仍經常模仿 terminal, editor, ticket 與既有工程角色.

講者特別質疑把自然語言硬塞進 terminal 的做法. Terminal 對命令與文字串流很有效, 卻不必然是規格協作, 多代理管理, 成果比較與長時間監督的最佳介面.

這不代表 terminal, Git 或 IDE 已失去價值. 更精確的問題是: 哪些舊介面仍符合新任務, 哪些只是為了降低過渡期的不適感?

## 5. 放下語言身份與程式碼沉沒成本

講者認為, 工程師過度以程式語言, framework 與工具選擇定義專業身份. 這些技能過去是稀缺性的訊號, 但當代理可以跨語言產生與修改程式, 它們對產品結果的解釋力下降.

另一項慣性是害怕刪除程式碼. 團隊有時會因某人已投入一兩週, 勉強合併並不理想的 pull request. 講者稱之為類似「guilt merge」的現象. Agent 產生的工作降低了人際上的沉沒成本, 團隊可以更果斷地放棄錯誤方向並重新生成.

但生成成本降低不等於決策成本歸零. 刪除前仍須確認需求, 資料遷移, 相依關係與使用者影響. 真正應放下的是對既有實作的情感依附, 不是工程責任.

## 6. 專案規模的分級正在下移

[09:30](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=570s)

講者以三個自己曾做或正在做的項目說明舊分級:

- Reddit meme scraper: 幾天完成的 side project.
- 為直播主建立的協作工具 Ping: 曾進入 Y Combinator 的 startup.
- 涵蓋更多能力的 full-stack cloud: 過去會被判斷為 too big.

他的觀察是, 模型能力提高後, 每一類項目可能向下移一級:

```text
過去的 startup
  -> 今天可能成為 side project

過去的 side project
  -> 今天可能只是一個可執行的 Markdown 工作流

過去的 too big
  -> 今天可能進入小團隊可以嘗試的範圍
```

這個分級是演講修辭, 不能忽略維運, 法遵, 客戶支援, reliability 與 go-to-market. AI 降低的是部分設計與實作成本, 不會自動消除公司營運的其他工作.

## 7. Markdown tier: 指示本身成為可執行系統

影片提出一個位於 side project 之下的新層級, 稱為 Markdown tier. 核心概念是, 某些過去需要獨立服務的內部工具, 現在可以只由自然語言指示加排程器構成.

講者自己的案例是一項每日 pull request triage:

1. 每天固定時間由 cron 啟動.
2. Agent 查看四個 GitHub repositories 的 open PRs.
3. 判斷各項工作的狀態並排序優先級.
4. 更新靜態 HTML.
5. 上傳至 S3 並回傳 URL.

過去這可能是一個需要持續開發的服務. 現在主要邏輯可寫在 Markdown 指示中, 由 Codex 或 Claude 執行.

值得注意的是, Markdown 只是控制描述. 真正的系統仍包含模型, agent harness, GitHub 權限, 排程器, 儲存服務, 錯誤處理與執行環境. 把它稱為「只有一個 Markdown 檔」凸顯作者維護的程式碼減少, 並不表示背後沒有基礎設施.

## 8. 「太大」的界線變得不清楚

[12:22](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=742s)

講者坦言不知道現在什麼才算 too big. 可能是從零訓練模型, 建立作業系統, 或直接挑戰 npm 與 Node. 這種不確定性既令人不安, 也代表探索空間擴大.

他的策略不是先宣稱所有大型問題都已可解, 而是刻意選擇看似超出合理範圍的專案, 透過實作找出新上限.

編者整理: 這種探索可以搭配逐層風險控制:

```text
建立最小垂直切片
  -> 驗證最困難假設
  -> 測量模型失敗模式
  -> 建立可重播的評估
  -> 再擴大產品範圍
```

如此可以保留野心, 又不必一次承擔整個平台的成本.

## 9. 與其只做得更深, 不如開始做得更廣

[13:06](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=786s)

傳統 startup 策略通常是在狹窄範圍做得比大型平台更深. 影片以 Vercel 與 AWS 比較:

- AWS 的產品 breadth 非常廣, 小團隊無法逐項覆蓋.
- Vercel 聚焦 full-stack 與 front-end deployment, 在特定範圍提供更深體驗.
- 過去的小團隊必須以深度換取差異化, 不能與大公司比功能範圍.

講者認為 agent 讓 breadth 的成本下降. 小團隊不一定能建立與 RDS 同等可靠的資料庫服務, 但可能在一兩天內做出足以讓早期使用者開始試用的資料能力, 並逐步擴展完整度.

這個論點應區分兩件事:

| 能力 | AI 可能大幅降低的成本 | 仍然昂貴的部分 |
| --- | --- | --- |
| 建立第一版功能 | Boilerplate, integration, UI,文件與測試生成 | 真實需求判斷與產品取捨 |
| 擴大功能範圍 | 多語言實作與平行探索 | 一致性, migration 與跨模組設計 |
| 進入大型市場 | 原型速度與小團隊產量 | Reliability, security, compliance 與 support |
| 允許使用者擴充 | Plugin 或 API scaffold | 穩定契約, 相容性與生態治理 |

## 10. 可擴充架構讓使用者補足垂直深度

[14:50](https://www.youtube.com/watch?v=xUnRQ9vLXxo&t=890s)

產品做得更廣後, 團隊不可能預先完成每個垂直領域的所有需求. 講者的解法是從一開始就建立可擴充架構, 讓使用者自行增加缺少的能力.

他以 Slack 為例. Slack 未必預見自己會成為許多 agent 的操作介面, 但 bot API 與產品形狀讓使用者能在上面建立新工作流. 因此, 一個較廣但具擴充點的平台, 可以讓生態系補上團隊無法親自完成的深度.

這要求產品提供:

- 清楚且穩定的 extension contract.
- 權限與安全邊界.
- 可觀察的執行結果.
- 版本與相容性策略.
- 使用者可理解的故障模式.

如果只生成大量表面功能, 卻沒有共同資料模型與擴充契約, breadth 很快會轉化成維護負債.

## 11. 「如果想法不顯得愚蠢, 可能還不夠大」

演講最後鼓勵觀眾挑戰 Slack, AWS 或 Salesforce 等看似不合理的目標. 這是一種用來打破自我限制的修辭, 不是建議忽略市場, 資本與營運現實.

更可操作的解讀是:

- 不再用過去所需工程人數直接否決想法.
- 把平台級目標拆成可驗證的垂直切片.
- 尋找 agent 對成本結構造成的非線性改變.
- 將省下的實作成本投入 architecture, evaluation 與使用者研究.
- 對模型能力保持進取, 對產品可靠性保持保守.

## 實務反思問題

編者整理, 可用以下問題重新檢查現有產品與工作流:

1. 哪些任務仍以舊模型的能力上限切分?
2. 哪些內部服務可以降級成 Markdown instruction 加 automation?
3. 哪些工具介面只是模仿 terminal 或 ticket, 而沒有服務代理管理需求?
4. 哪些程式碼因沉沒成本被保留, 而不是因目前仍有價值?
5. 產品能否增加 breadth, 同時維持共同架構與品質標準?
6. 使用者能否透過 plugins, APIs 或 skills 自行補足垂直功能?
7. 當任務範圍擴大時, 是否同步增加驗證, 權限與 rollback 能力?

## 核心觀念

AI 對軟體開發的影響不只是在相同需求下更快產生程式碼. 它可能改變什麼值得成為軟體, 什麼需要成立公司, 一個小團隊能覆蓋多廣, 以及產品應如何把未完成的深度交給使用者擴充.

影片最值得保留的提醒是: 不要用舊成本結構評估新專案. 但也不應反向假設生成容易就等於產品完成. 新的競爭優勢可能從程式碼產量轉向問題選擇, 架構一致性, 擴充設計, 驗證能力與承擔結果的能力.

## 來源與限制

- 本筆記依英文原始自動字幕與影片正式章節整理, 不是逐字稿.
- 片名中的「Everything we knew」與演講中的「AI psychosis」是刻意誇張的修辭, 不應按字面理解為所有既有軟體知識失效或醫療診斷.
- Sonnet 3.5, Opus 4.5, Mythos 與 Fable 的能力描述來自講者個人體驗. 本筆記未將其視為獨立 benchmark.
- 「一兩天建立資料庫平台」「Markdown 取代 startup」等說法旨在表達成本級距改變, 不包含 production reliability, security, compliance, support 與營運的完整成本.
- 講者正在建立影片提到的 full-stack cloud, 因此相關市場判斷也包含創業者自身立場.
