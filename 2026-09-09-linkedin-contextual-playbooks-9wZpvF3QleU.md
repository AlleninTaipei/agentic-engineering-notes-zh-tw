# LinkedIn 的 Contextual Playbooks: 用 MCP 傳遞操作知識, 按需發現上千工具

> - 影片: [500 Skills, Zero Fine-Tuning: LinkedIn's Playbook for AI Agents — Ajay Prakash, LinkedIn](https://www.youtube.com/watch?v=9wZpvF3QleU)
> - 頻道: [AI Engineer](https://www.youtube.com/channel/UCLKPca3kwwd-B59HNr-_lvA)
> - 講者: Ajay Prakash, LinkedIn Senior Staff Software Engineer, 職稱依影片公開說明.
> - 發布日期: 2026-09-09.
> - 片長: 20:24.
> - Video ID: `9wZpvF3QleU`.
> - 內容依據: YouTube 原始英文自動字幕 `en-orig`, 搭配公開資訊與章節. 無創作者提供字幕.
> - 來源性質: 內部系統建造團隊的第一手架構分享. 規模與使用成效為講者陳述, 未另行稽核.

## 核心觀點

LinkedIn 早期導入 coding agents 時, 模型不了解內部框架與操作方式, 工程師花在糾正輸出上的時間甚至超過自己實作. 開放程式碼搜尋與其他工具後, Agent 雖能回答局部問題, 仍不容易可靠地完成整段工作.

團隊的下一步是把「如何使用工具完成任務」也透過 MCP 提供. 這些 playbooks 以工具形式被發現與呼叫, 將指引作為工具輸出送入 context. 大型指引拆成可重用的小指引, 需要時才載入, 並讓工程師與 agent 透過 PR 持續修正.

工具數量擴大後, 對外入口收斂為 search, get schema 與 execute 三個 meta tools. Agent 先找到需要的能力, 再取得 schema 與執行, 避免一開始將全部工具定義塞入 context.

## 1. 從 on-call 工作說明系統目標

[00:00](https://www.youtube.com/watch?v=9wZpvF3QleU&t=0s)

Ajay 以服務錯誤率上升的情境開場. 工程師將告警連結交給 coding agent, 後者依序:

1. 取得公司處理此類問題的指引.
2. 識別服務, 再取得該服務專用的除錯 context.
3. 讀取 logs 與 metrics, 調查根因.
4. 彙整問題與緩解方案, 等待人確認.
5. 確認後執行緩解動作.
6. 更新 incident 紀錄, 並為根因修正建立 PR.

講者表示, LinkedIn 團隊已用這種方式讓 agent 參與工作, 將原本可能需要數小時的流程縮短到數分鐘. 但開場沒有提供具體 incident ID, trace, 測量方法或成功率. 應將它視為使用情境與成效自述, 不能當作已完整呈現的事故復盤.

這個情境也保留了人機界線: 緩解操作在人確認之後才執行, 建立修正 PR 不等於自動合併或部署.

## 2. 公開程式碼知識不足以理解內部系統

[02:48](https://www.youtube.com/watch?v=9wZpvF3QleU&t=168s), [05:10](https://www.youtube.com/watch?v=9wZpvF3QleU&t=310s)

LinkedIn 有超過一千個 repositories, 支援數千個微服務與應用. 系統建立在內部框架, 函式庫與自建基礎設施之上, 包括資料庫, 實驗與追蹤平台, 以及設定管理系統. 講者表示新進工程師也需要一週的 boot camp 熟悉這些環境.

早期 Agent 缺少這些 context, 會卡住或編造不存在的做法. 工程師需要反覆提示修正, 導致部分人回到手動 coding.

團隊因此把目標定為讓各種 coding agents 理解內部系統, 並產生工程師願意信任的程式碼. 所謂信任包含正確性與程式碼品質, 不只是輸出速度.

## 3. MCP 提供存取能力, 但工具不是完整工作知識

[06:34](https://www.youtube.com/watch?v=9wZpvF3QleU&t=394s)

第一個內部 MCP 工具是 code search. Agent 可以利用既有搜尋系統, 透過關鍵字, filters 與正規表示式等方式尋找跨 repository 的內部範例, 再依範例回答問題或實作.

之後團隊接入文件, Jira, Slack, 資料平台與 feature flags. 工程師可以把 PRD, 設計文件與工作事項的 context 帶入同一段任務.

[08:30](https://www.youtube.com/watch?v=9wZpvF3QleU&t=510s) 起, 講者指出仍有三個障礙:

| 障礙 | 具體問題 |
| --- | --- |
| 操作知識分散 | 怎麼除錯與設定系統的經驗散落在文件, wiki 與 Slack, 還可能過時或重複 |
| Context 過載 | 工具輸出持續占用空間, 壓縮 context 後可能遺失工作資訊, 需要重查 |
| 無法持續保留學習 | 即使完成一次任務, 下一次 session 仍可能重新探索相同知識 |

能存取資料與能完成工作是兩個不同需求. Agent 還需要知道應該找什麼, 依什麼順序處理, 以及如何使用查到的資訊.

## 4. Playbook 是透過工具取得的指引

[10:34](https://www.youtube.com/watch?v=9wZpvF3QleU&t=634s)

Playbook 有名稱與描述, 外觀如同一般工具. Agent 判斷與任務相關後呼叫它, 取得的輸出是操作指引與 context, 再依指引呼叫真正執行工作的工具.

原片以在 LinkedIn 建立 Airflow DAG 為例:

```text
使用者要求建立 Airflow DAG
  -> 找到適用的 playbook
  -> 取得 LinkedIn 內部的建立方式與限制
  -> 依指引使用相關工具
  -> 完成任務
```

工程師可以將 playbook 寫入 repository, 讓其他人重用. 這讓組織知識成為可按任務取用的內容, 而不必每次由使用者手動補進提示.

講者表示 playbooks 與 skills 的概念相近, 但團隊在 skills 成為相關形式之前就建立了這套系統. 本文保留其命名, 不假設兩者使用相同檔案格式或可直接互換.

## 5. 聚焦單一任務, 大型流程拆成小型指引

[12:27](https://www.youtube.com/watch?v=9wZpvF3QleU&t=747s)

團隊要求 playbook 遵循兩個原則:

- Self-contained: 聚焦明確任務, 讓 Agent 容易判斷何時選用. 例如建立 Airflow DAG 的指引應專注於該工作.
- 拆分與引用: 把大型 playbook 拆成較小的 playbooks, 由上層引用它們.

拆分同時支援重用與漸進式取得 context. 多個工作可以引用同一份小型指引, Agent 也只需在進入相關步驟時才讀取細節.

這裡的 self-contained 不代表禁止依賴或引用. 依講者後續說明, 它強調任務邊界清楚, 同時允許透過引用組合較大的流程.

## 6. 以 PR 將任務中的修正回饋給 playbook

[14:23](https://www.youtube.com/watch?v=9wZpvF3QleU&t=863s)

知識庫的維護問題不會因儲存形式改變而消失. 團隊鼓勵 Agent 在使用 playbook 後, 找出過時資訊, 不一致與缺漏, 再提出更新:

```text
工程師建立 playbook
  -> Agent 在任務中使用
  -> 辨識缺漏與學到的修正
  -> Checkout repository 並修改
  -> 建立 PR
  -> 經核准後更新共用指引
```

這個循環讓實際工作成為維護知識的機會. 演講未交代完整 reviewer 政策, 自動測試或錯誤更新的回復方式, 因此不能把它解讀為已驗證的全自動知識演進系統.

## 7. 本機配送, 中央共用與 repository 專用內容

[15:34](https://www.youtube.com/watch?v=9wZpvF3QleU&t=934s)

依講者描述, LinkedIn 筆電預先安裝 local MCP server. Server, tools 與 playbooks 的更新每小時自動配送至筆電.

| 類型 | 範圍 | 使用方式 |
| --- | --- | --- |
| Central playbooks | 跨 repository 的共用工作 | 集中維護, 支援多個專案 |
| Local playbooks | 特定 repository 的工作 | 與專案一起版本控制, Agent 在該專案工作時才自動取得 |

Local playbooks 讓專案團隊不必為每項局部知識都修改中央 repository. 統一的 MCP server 則提供共用的 authentication 與 telemetry, 支援觀察及改善整套系統.

這裡的「統一」是共同入口與管理方式, 不表示所有工具都在同一個程序內執行. 影片未詳述後端服務拓樸, 授權粒度或憑證流向.

## 8. 以三個 meta tools 按需發現能力

[17:23](https://www.youtube.com/watch?v=9wZpvF3QleU&t=1043s)

講者表示, 同時暴露超過約 30 至 40 個工具時, 他們遇到 context 或系統表現下降的問題. 解法是將入口換成三個 meta tools:

| 入口 | 任務 |
| --- | --- |
| Search | 用關鍵字與 tags 尋找相關 tools 和 playbooks |
| Get schema | 取得候選工具的詳細規格 |
| Execute | 執行選定的工具或 playbook |

Coding agents 也預先配置 system instructions, 說明如何使用這些入口與有效搜尋. 因此能力發現並非完全依賴模型自行猜測流程.

這個設計將「可取得的能力總量」與「當前 context 需要呈現的能力」分開. Playbook 的漸進式載入處理指引內容大小, meta tools 則處理工具目錄規模, 兩者作用在不同層次.

30 至 40 是講者對其系統經驗的概述, 不是 MCP 協定規定的工具上限. 影片沒有提供各種工具數量下的對照評測, schema 大小或模型配置.

### 使用規模與標題差異

結尾講者表示, 系統每天有超過 8,000 名使用者, 包括工程師, PM, 設計師與 TPM. 工具數約 1,300 個, playbooks 超過 600 份. 工具數的自動字幕辨識不完整, 約 1,300 由影片公開說明補足.

標題使用「500 Skills」, 但口述與公開說明提到超過 600 playbooks. 本文不自行推定差異原因. 這些數字顯示講者描述的採用規模, 不等於任務成功率或生產力提升的證據.

標題也強調「Zero Fine-Tuning」, 但演講主體著重 context 與工具基礎設施, 沒有提供微調方案的對照實驗. 不能由此推論微調在其他情境沒有價值.

## 編者整理: 可用於設計審查的問題

以下依影片架構整理, 非講者逐字交付的 checklist:

1. Agent 失敗是因為缺少工具, 還是缺少使用工具的組織知識?
2. 每份指引是否聚焦明確任務, 名稱與描述是否足以讓 Agent 找到它?
3. 哪些重複步驟值得抽成小型指引, 並在需要時載入?
4. 任務中發現的知識缺漏, 能否形成可審查的更新 PR?
5. 哪些內容屬於中央共用, 哪些應與 repository 一起維護?
6. 是否真的需要一次公開所有工具定義, 還是可以先搜尋再取得 schema?
7. 除了使用人數, 是否也能觀察任務品質, 失敗與重複搜尋成本?

## 來源與限制

本文依完整英文自動字幕去重與重述, 時間戳採影片公開章節. 未逐幀核對投影片或操作畫面, 也未取得 LinkedIn 內部程式碼與 playbook 原文. MCP, Claude Code 等明顯誤辨依上下文校正.

系統起源時間依講者回顧, 不作為 MCP 公開發布歷史的精確紀錄. 更新頻率, 採用人數與工具規模皆是影片當時的部署自述, 未驗證現況.

本場提供了從工具接入, 指引配送到能力發現的架構脈絡, 但沒有完整 eval 方法, 品質數據, context token 成本或事故時間對照. 對安全操作, PR 審查, 更新回復與搜尋品質的實作也只概述或未說明. 這些限制不代表系統沒有處理, 而是不能僅憑演講確認.
