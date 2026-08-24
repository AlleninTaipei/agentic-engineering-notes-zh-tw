# 每家公司都應該有一個 Brain

來源: [Every company should have a Brain, Garry Tan, Y Combinator](https://www.youtube.com/watch?v=eBUyTS7SzV4), AI Engineer

- 正規網址: https://www.youtube.com/watch?v=eBUyTS7SzV4
- 上傳日期: 2026-07-17
- 片長: 21:08
- Video ID: `eBUyTS7SzV4`
- 內容依據: 英文原始語言自動字幕

## 摘要

Garry Tan 從創辦人, 投資人與 Y Combinator 經營者的角度, 主張真正的 AI-native company 不只是讓員工使用聊天機器人, 而是把角色, 流程, 路由規則, 評估標準與組織知識編碼成 agent 可以執行的基礎設施.

這套方法有兩個核心. 第一, 將重複工作整理成可重用的 skill, 讓組織不必每天重新教模型相同的事. 第二, 建立兼具 library 與 librarian 的 company brain, 讓 agent 能依任務載入正確 context, 而不是把所有資料塞入 context window.

講者以自己估算的 400 倍程式產出與多家 YC 公司作為例子, 認為差距不只來自模型能力, 更來自工作如何被編排. 不過這些數字主要是講者的經驗與現場主張, 不能直接推論為普遍生產力或 AI 導致營收成長的證據.

## 從一千人的公司能力, 走向一人的槓桿

[00:00](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=0s)

講者設定的目標是, 讓一個人完成過去需要一千人才能做的工作. 他認為這不是遙遠預測, 而是當代創業者已能開始實驗的組織設計.

這裡的關鍵並不是一個人親自執行所有步驟, 而是由一個人設計流程, 分派任務, 提供 context, 評估結果, 並管理由 agent 組成的數位勞動力.

## 400 倍產能的主張

[01:25](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=85s)

Garry Tan 比較自己在 2013 年與當年的程式產出. 他表示, 2013 年擔任 YC partner 並開發內部社交網路時, 每天約能完成 14 行可用的邏輯程式碼. 當年常見估計大約落在每天 15 至 50 行.

他依目前產出估算約有 400 倍提升. 即使大幅扣除 agent 產生的冗長程式, scaffolding 與個人高估, 他仍認為保守下限約為 8 倍, 中間估計約為 80 倍.

這項比較有幾個限制:

- 程式行數不是軟體價值或品質的可靠單一指標.
- 兩個年代的工作內容, 工具與衡量方式不同.
- 數字是講者的自我估算, 不是控制實驗.
- 講者自己也明確表示, 無法證明 AI 生成程式碼造成 YC 公司成長.

因此, 這段最值得保留的不是精確倍數, 而是後續命題: 使用同一模型的人可能產生極大差距, 因為槓桿來自如何設計工作系統.

## 槓桿不只在模型權重, 而在工作接線

[03:06](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=186s)

講者觀察到, 產能提升 2 倍與 100 倍的人可能使用相同的 Claude, context window 與 API. 差異在於是否把 AI 當成 autocomplete, 還是當成需要被聘用, 訓練, 分工與評估的 workforce.

他援引 YC Winter 2025 batch 的觀察, 表示約四分之一公司的 codebase 有 95% 由 AI 生成, 並稱該 batch 是 YC 成長最快且獲利最好的 batch. 這是講者的現場陳述, 而且他明確沒有主張兩者存在已證實的因果關係.

## 把 Agent 基礎設施對應到組織

[03:38](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=218s)

講者將常見 agent 元件映射到公司的管理結構:

| Agent 元件 | 組織對應 | 功能 |
| --- | --- | --- |
| Skill file | 員工或職務 | 清楚描述一項能力與工作方法 |
| Resolver table | 組織圖與派工規則 | 判斷任務應由哪個 skill 或角色處理 |
| Filing rules | 內部流程 | 規定資訊如何分類, 保存與使用 |
| Trigger evals | 績效考核 | 驗證特定情境是否載入正確規則並產生合格結果 |
| Code | 確定性執行機制 | 負責需要一致性, 計算或硬性約束的部分 |

例如, 當任務涉及修改測試時, resolver 應載入 `tests.md`. Trigger eval 則驗證這個路由是否真的發生. 這不只是 prompt engineering, 而是把職責分工與品質管理寫成可測試的系統.

講者因此提出一個鮮明比喻: 當你使用 Claude Code 或 Codex 時, 不只是在寫軟體, 也在聘用, 訓練與管理由 Markdown 組成的 workforce.

## AI-native Organization

[04:11](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=251s)

AI-native company 的特徵不是擁有一個 AI 專案, 而是讓 agent 能執行銷售, 客服, 營運, 財務與工程中的標準化流程. 團隊聘用工程師維護 skills, 並處理 skills 暫時無法完成的工作.

講者列出多家 YC 公司及其營收與人數, 用來說明極高 revenue per head 的可能性. 這些公司資料是影片中的案例陳述, 本筆記未以外部來源驗證. 它們支持的是講者的組織設計觀點, 不能單獨證明成果全由 AI-native 架構造成.

這種轉型也不限於工程師. 講者表示, YC 的媒體, 活動與財務人員也開始建立 skill files 與 cron jobs. 其中一位財務人員把約一百個 Excel workbook 整合成一個內部應用程式. 重點在於, 非程式設計者也可能從工具使用者轉變為 agent manager.

## Latent Space 與 Deterministic Space

[08:38](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=518s)

講者認為, 許多 AI 系統問題來自把運算放錯地方. 系統設計者應區分兩種空間:

| 空間 | 適合處理 | 不適合處理 |
| --- | --- | --- |
| Latent space, LLM | 品味, 判斷, 理解模糊意圖, 非確定性決策 | 精確保存大量狀態, 嚴格計算, 必須完全一致的規則 |
| Deterministic space, code | 資料結構, 計算, 驗證, 排程, 硬性約束 | 理解含糊需求, 需要語意與人類判斷的工作 |

Startup School 的座位安排是講者的例子. 系統要從 800 人中找出適合彼此交流的鄰座, LLM 可以判斷人物之間的語意相似性與互補性, 但 800 個座位的配置狀態不應只存在 context window. 座位陣列, 衝突檢查與配置限制應由 deterministic code 管理.

可概括為:

```text
LLM 決定什麼安排具有意義
程式確保安排在結構上有效且可重現
```

## 人類工作記憶與 Agent Context

[10:53](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=653s)

講者用人類工作記憶的經典說法 "7 plus or minus 2" 作為對比. 清單, 組織圖與檔案櫃都是人類為有限工作記憶建立的外部輔具.

相較之下, 一百萬 token 的 context 約可容納上千頁文字. 他用三本 Harry Potter 小說同時攤開來比喻 agent 可以搜尋與綜合的資訊量. 這只是協助理解規模的比喻, 並不表示模型能對 context 中所有內容保持完全可靠的理解.

即使三本書很多, 一家公司仍遠大於三本書. 郵件, 會議, 決策理由, 客戶對話與 postmortem 共同形成一座 library. 決定 agent 是聰明助手還是容易失憶的 goldfish, 關鍵在於誰替它選出此刻該放在桌上的三本書.

## Context Engineering 是選書, 不是塞滿 Context

[12:53](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=773s)

講者將 company brain 定義為兩部分:

```text
Company brain = Library + Librarian
```

- Library 保存組織知識與歷史.
- Librarian 依目前任務選取, 排序與壓縮最相關資訊.

這與 RAG 有關, 但 retrieval 只是底層 primitive. 真正困難的是:

- 哪些資訊一開始就值得寫入.
- 如何補充 metadata, 關聯與 provenance.
- 哪些內容應進入 hot memory, 哪些只作 cold reference.
- 新舊事實衝突時由誰裁決.
- 如何讓過時資訊失效或被移除.

講者的濃縮判斷是, retrieval 本身容易, 建立值得被 retrieval 的知識才是產品.

## GBrain 與個人第二大腦

[13:28](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=808s)

講者介紹自己開源建立的 GBrain, 將其定位為 agent 的 retrieval layer, 負責針對任務挑出應載入的資訊. 他表示, 個人系統已累積約 220,000 頁, 來源包括郵件, 會議, 二十年的筆記與 agent 整理的經驗.

當創辦人寄來危機求助信時, agent 能先找出過往對話, 遭遇過類似問題的 portfolio company, 以及當時有效的方法. 講者認為, 能在行動前使用擁有者既有知識, 是 assistant 與 colleague 之間的重要差別.

這裡的頁數與效果為講者自述. 大型知識庫的實際品質仍取決於擷取, 權限, 時效性與清理機制.

## Company Brain 的失敗模式

[14:20](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=860s)

講者主動指出, 沒有人維護的 brain 只會變成 "搜尋功能很好的垃圾場". 常見風險包括:

- Retrieval 以很高信心取回過時事實.
- 錯誤 skill 把壞流程永久自動化.
- 新舊資料互相矛盾, 卻沒有仲裁機制.
- Agent 產生錯誤結論, 但缺少可追查的來源.

因此真正需要的 primitive 不是單純 memory, 而是:

```text
Memory + Hygiene + Provenance + Contradiction Checks + Pruning
```

每項事實應保留來源, 新資訊進入時應檢查衝突, 並由人類與 agent 組成的 librarian 定期修剪. Company brain 應被視為 production infrastructure, 而不是無限制收集資料的 dumping ground.

## 永遠不要做一次性工作

[15:13](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=913s)

講者提出最具操作性的紀律: "Never do one-off work."

工作流程是:

1. 讓 agent 完成任務.
2. 檢查結果並要求修正, 直到品質可接受.
3. 不在這裡停止, 把成功流程整理成 skill.
4. 將 skill 放回 harness, 下次自動重用.
5. 為 skill 增加路由規則與 eval, 避免只保存文字而沒有驗證.

他稱這個動作為 "skillify it". 如果同一件事需要詢問兩次, 就表示第一次沒有把學習沉澱成組織能力.

```text
一次性成果 -> 經過修正的流程 -> Skill -> Eval -> 可重複能力
```

能持續捕捉學習的組織會每天變聰明. 沒有沉澱機制的組織, 即使租用最好的模型, 隔天仍像失憶一樣重新開始. 講者用一句話區分兩者: 模型品質是租來的, 自己建立的 brain 才是組織擁有的資產.

## 該建立什麼

[16:40](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=1000s)

講者給創業者的方向是, 從第一天就建立 AI-native company:

- 維持精簡團隊.
- 讓創辦人持續接近程式與實際工作.
- 為重複流程建立 skill files.
- 從第一週開始建立可複利的 library.
- 建立 librarian, 依任務提供正確 context.
- 不把方法綁死在單一模型, harness 或 repository.

他也認為 company brain, personal context 與 librarian layer 仍是開放的產品機會. GBrain 是其開源實作之一, 但講者強調概念比特定工具重要, 其他 stack 也能採用相同原則.

## Abundance 必須成為已交付的軟體

[18:56](https://www.youtube.com/watch?v=eBUyTS7SzV4&t=1136s)

講者最後將 AI 生產力連結到更廣泛的社會問題. 他的主張是, abundance 不能只停留在政策論述, 必須透過實際交付的軟體實現.

他舉一位朋友為患有罕見癲癇的兒子建立 80,000 個 Markdown 檔案為例. 這位父親把相關知識建成專門 library, 再由 librarian 為當下問題挑選資訊. 影片用此例說明, 相同架構不只服務商業自動化, 也能協助個人探索原本需要大量專業資源的問題.

這類高風險醫療用途仍需要專業醫療判斷, 來源查核與隱私保護. Knowledge brain 可以支援研究與整理, 不等於醫療診斷系統. 這項提醒是編輯補充.

## 實作藍圖

### 1. 找出可重複工作

優先盤點每週重複出現, 有明確輸入與輸出的流程. 不要一開始就自動化最模糊, 風險最高的決策.

### 2. 分離 latent 與 deterministic 工作

- 讓 LLM 處理語意, 分類, 摘要與模糊判斷.
- 讓 code 處理資料保存, 計算, 驗證, 權限與硬性限制.
- 不把唯一狀態只留在對話 context.

### 3. 建立第一個 skill

Skill 至少應寫清楚:

- 何時觸發.
- 需要哪些輸入與 context.
- 執行步驟與工具.
- 預期輸出.
- 失敗時如何停止或升級給人類.
- 如何驗證成功.

### 4. 建立 resolver 與 eval

讓任務能被路由到正確 skill, 再用測試確認特定情境真的載入相應規則. Skill 沒有 eval, 就像職務說明存在, 卻從不檢查是否照做.

### 5. 建立可維護的 library

- 保存來源與時間.
- 區分 hot memory 與 cold reference.
- 對矛盾事實建立仲裁流程.
- 設定資料過期與刪除政策.
- 保護機密, 個資與存取權限.

### 6. 每次成功後 skillify

將一次性互動中的修正與判斷整理成可重跑流程, 並用真實案例測試. 只有保存下來且可驗證的學習, 才能形成組織複利.

## 核心觀念回顧

```text
AI-native company
├── Workforce: 可分工的 skills
├── Org chart: resolver 與路由規則
├── Performance review: trigger evals
├── Deterministic systems: code, state 與約束
└── Company brain
    ├── Library: 可追溯的組織知識
    └── Librarian: 依任務選取 context
```

影片的中心論點可以濃縮成三句話:

1. 不要只升級模型, 要重新設計工作如何被編排.
2. 不要讓成果停留在一次性對話, 要把有效方法轉成 skill.
3. 不要把公司資料堆進 context, 要建立能選出正確資訊的 librarian.

## 來源與限制

本筆記依 YouTube 提供的英文原始語言自動字幕整理, 並參照影片章節與公開中繼資料. 自動字幕可能誤辨產品名稱, 人名與技術術語. 內容已轉寫為學習筆記, 省略開場互動, 重複語句與宣傳性措辭, 不是逐字稿.

影片中的產能倍數, 公司營收, 人數, 知識庫頁數與 YC batch 表現是講者在演講中的陳述, 本筆記未使用外部資料獨立驗證. 講者也明確承認, AI 生成程式碼與公司成長之間的因果關係不能由這些觀察證明.
