# 組織如何擴大 Skills 的使用: 結構設計, 集中目錄與治理責任

> - 影片: [AI-Native Organisations Run on Skills: How to Structure and Scale Them](https://www.youtube.com/watch?v=M05vON8i0aI)
> - 頻道: [AI Engineer](https://www.youtube.com/@aiDotEngineer)
> - 講者: Imad Touil, QuantumBlack Distinguished Engineer
> - 發布日期: 2026-08-28
> - 片長: 20:30
> - Video ID: `M05vON8i0aI`
> - 內容依據: YouTube 英文原語自動字幕 (`en-orig`), 公開影片資訊與章節
> - 整理語言: 繁體中文

## 核心觀點

Touil 將 skills 視為組織操作知識的可執行載體. 個人能建立 skill, 不代表團隊能找到, 重用與維護它. 當採用範圍擴大, 需要一起處理目錄, 版本, 依賴, 評測, 權限與負責人.

他的主要警告是, 缺少治理的 skills 會累積成新的技術債. 各團隊可能重複建置相同能力, 沒有人維護舊版本, 也不知道哪些內容能安全地組合使用. 中央平台可以支援管理, 但跨團隊責任仍要由人決定.

## Coding workflow 只是完整交付生命週期的一部分

[01:33](https://www.youtube.com/watch?v=M05vON8i0aI&t=93s)

講者將 agentic stack 分成內外兩層:

| 層次 | 影片中的組件 | 用途 |
| --- | --- | --- |
| 內層 harness | Context manager, tools/MCP, memory/state, skills loader | 支援 agent 執行與上下文管理 |
| 外層 workflow | Skills, subagents, MCP servers, hooks | 組織任務步驟與執行方式 |
| 支援設施 | Sandbox, MCP gateway, model gateway, knowledge graph, skills registry, workflow marketplace | 提供環境, 存取與共用能力 |

Context 則來自專案指令, 工具 schema, 對話記憶與取回的文件或程式碼. 這是講者提出的架構視角, 不是所有團隊都必須具備的最小建置清單.

[02:56](https://www.youtube.com/watch?v=M05vON8i0aI&t=176s), [04:19](https://www.youtube.com/watch?v=M05vON8i0aI&t=259s)

常見的 specify, plan, task, implement 流程, 在他看來只是建立一次 product increment. 完整交付還包含:

1. 產品策略, 成功指標與 roadmap.
2. 市場研究, 競爭分析與客戶訪談.
3. 問題探索, 解法驗證與 user stories.
4. 資料管線, 品質檢查, 資料目錄與系統整合準備.
5. 功能實作.
6. 基礎設施佈建, 發布與營運.
7. 效能改善, incident 處理與下一輪回饋.

[05:39](https://www.youtube.com/watch?v=M05vON8i0aI&t=339s)

同一組織也可能同時存在行動應用, 內部平台與客戶產品等不同生命週期. 講者據其服務組織的經驗, 反對假設一條 workflow 能適用所有產品. 共用 skills 的設計需要容納這些差異.

## Skills 承載什麼組織知識?

[07:03](https://www.youtube.com/watch?v=M05vON8i0aI&t=423s)

講者區分 workflow 中四種組件的主要用途: hooks 在事件發生時觸發動作, MCP 提供工具介面, subagents 分派任務與減少主 context 負擔, skills 則承載如何完成工作的 know-how.

這是為突出 skills 角色所做的簡化. 他也將 workflow 比喻成 harness blueprint, 在執行時塑造 agent 行為.

編輯說明: 組織知識也可能存在程式碼, 工具實作與其他指令中. 結構化 skills 有助於一致性, 但包含模型判斷的 workflow 不會因此自動變成確定性流程. 應區分一般程式腳本的確定性與模型執行結果的可變性.

## 借用微服務的設計原則

[08:27](https://www.youtube.com/watch?v=M05vON8i0aI&t=507s)

Touil 認為這類管理問題並不陌生, 可以借用微服務與軟體工程的既有原則:

| 原則 | 對 skill 的要求 |
| --- | --- |
| 可重用 | 相同知識能服務多個任務或團隊 |
| 模組化與專責 | 一個 skill 聚焦明確任務, 避免包辦所有工作的巨大 skill |
| 可發現 | 其他團隊能找到已有能力與負責人 |
| 可攜 | 讓知識能在不同 workflows 與 harnesses 中使用 |
| 可組合 | 明確設計邊界, 避免重複步驟與互相衝突的指令 |
| 一致性 | 將組織方法與要求表達成可遵循的程序 |
| 成本效率 | 只在需要時載入相關內容 |

講者以共用格式支持跨 harness 可攜性. 但影片沒有展示跨平台相容性測試, 因此不能據此保證所有工具, 腳本與權限設定都能原樣移植.

### Progressive disclosure 控制載入成本

[09:48](https://www.youtube.com/watch?v=M05vON8i0aI&t=588s)

講者強調在正確時機載入適量內容, 而不是讓全部操作知識持續占用 context. 這使 skills 能將組織方法封裝成可取用的單位.

他也提到一項 skills benchmark, 用來支持提供 skills 後任務表現可能改善. 口述沒有完整給出研究連結, 測試配置或數值, 本筆記不據此推算成功率或 token 節省幅度.

## 組合範例: 規範審查 workflow

[11:09](https://www.youtube.com/watch?v=M05vON8i0aI&t=669s)

講者以資料保留政策作為 skill 範例, 再將它與揭露標準, GDPR 相關規則及填報範本組合到 regulatory disclosure review workflow.

```text
Skills catalog
  -> 資料保留政策
  -> 揭露標準
  -> 相關規則
  -> 填報範本
          |
          v
Review workflow
          |
          v
稽核報告 + 待改善事項
          |
          v
回到程式碼與產品修正
```

此例說明跨應用共用規則與產出可保存報告的設計方式. 影片未提供可檢查的規則實作或審查正確率, 也沒有證明這個組合能保證合規. 本節僅記錄架構示例, 不提供法律判斷.

## 未治理的 skills 如何累積技術債

[12:30](https://www.youtube.com/watch?v=M05vON8i0aI&t=750s)

| 問題 | 講者描述的成因 | 管理方向 |
| --- | --- | --- |
| 重複建置 | 同技術棧的團隊不知道彼此已有能力 | 共用目錄與跨團隊協作 |
| 品質退化 | Skills 未隨任務與模型變更重新驗證 | 持續測試與維護 |
| 難以發現 | 缺少 metadata 與查找入口 | 可搜尋 catalog |
| 無人負責 | 不知道誰擁有與維護內容 | 指定 owner |
| 組合衝突 | 各自設計, 沒有協調 domain 邊界 | 事先設計組合方式 |
| 安全風險 | 外部 skill 可能帶有不當指令或腳本 | 納入安全檢查流程 |
| 存取不當 | 部分 skills 包含敏感商業邏輯 | Access control |

講者特別指出, skill 不只是文字, 還可能帶有可執行 scripts. 因此取得公開 skill 也涉及軟體供應來源與執行權限的判斷.

## 從個人建立走向組織共用

[13:52](https://www.youtube.com/watch?v=M05vON8i0aI&t=832s)

講者提出三層推進方式:

1. 個人: 用一致的方法建立, 測試與改善 skills.
2. 團隊: 分享並共同維護相同產品與技術棧需要的能力.
3. 組織: 透過中央平台支援跨團隊發現, 發布與管理.

他認為中央平台應包含:

- 可搜尋的 catalog 與 metadata.
- Skills 之間的 dependencies.
- Versioning 與 lifecycle 管理.
- Access control.
- Evaluation 與 observability.

他舉例以 MCP 搜尋目錄, 再用 CLI 將 skill 取回 IDE 或 sandbox. Harness 也可識別並取得較新版本. 這是建議的整合方式, 影片沒有交付完整平台實作.

## 平台之外, 還需要明確的治理責任

[15:14](https://www.youtube.com/watch?v=M05vON8i0aI&t=914s)

技術可以提供目錄與版本控制, 但無法自行決定誰有權維護某個 domain. 講者建議由 architects, engineering leads, infrastructure leads 與 security leads 依組織結構分工, 負責內容與政策的更新.

理想流程是各團隊從同一入口取得維護中的 skills, 改善後再回饋共用平台. Domain owner 讓這個循環具有責任歸屬, 不只是把檔案放到一起.

編輯說明: 能取得最新版本與應自動採用最新版本是不同決策. 影片沒有完整討論版本固定, 相容性, 升級核准與 rollback, 這些仍是實際平台設計需要補齊的問題.

## 十五個團隊的六個月模擬

[16:34](https://www.youtube.com/watch?v=M05vON8i0aI&t=994s)

講者展示一個十五團隊, 每隊約五到十二人的模擬. 輸入包含 skill 貢獻量, 使用頻率, 團隊間重複比例與品質安全等假設, 用來呈現六個月後不同治理情境的差異.

未治理情境中, 各團隊的成本與品質差異持續存在. 治理情境則示意工程師建立新 skill 前, harness 可先發現既有能力並重用, 減少重複工作.

這是講者自行建立的情境模擬, 不是十五個真實團隊的六個月追蹤研究. 口述未提供完整模型, 參數來源與校準方式, 因此不能把展示的 productivity, cost 或 security 變化當成已驗證的治理成效.

## 同樣的治理方式也要延伸到 workflows

[17:58](https://www.youtube.com/watch?v=M05vON8i0aI&t=1078s)

講者提醒, skills 只是 workflow 的一部分. 即使每個 skill 都整理妥當, 完整流程仍可能有問題.

他建議對 workflows 採用相似的共用, 測試與回饋方式. 例如需要佈建基礎設施的工程師, 可取得既有 workflow 及其 skills, 執行測試後再回饋改善.

## 評測與自動演進仍需治理

[19:22](https://www.youtube.com/watch?v=M05vON8i0aI&t=1162s)

講者最後提出三個方向: skills registry, skills evaluation 與 auto-evolving skills.

他將依最佳實務做靜態檢查視為評測起點, 檢查 skill 結構與觸發設計是否合理. 同時提醒, 自動改善 skills 的閉環若沒有治理, 會放大原有的維護問題.

編輯說明: 靜態結構檢查不能單獨證明 skill 會在正確時機觸發, 或能提高任務品質. 動態行為與不同模型下的結果仍需實測. 影片也沒有提供可直接使用的自動演進機制或量化成果.

## 編輯整理: 可用來盤點現有 skills 的問題

以下依影片主題整理, 不是講者交付的完整治理規格:

1. 每個 skill 是否有明確任務, owner 與適用範圍?
2. 建立新 skill 前, 能否搜尋到相近的既有能力?
3. 共用 skills 是否記錄版本, 依賴與變更原因?
4. 更新模型或 skill 後, 是否有代表性任務能檢查退步?
5. 外部腳本與敏感內容是否經過相應的檢查與存取控制?
6. 多個 skills 組合後, 是否仍能完成 workflow 的整體目標?
7. 自動產生的修改由誰評估, 如何發布, 出問題時如何回復?

## 延伸閱讀

- [建立優秀 Agent Skills 的實作手冊](2026-06-29-building-great-agent-skills-UNzCG3lw6O0.md): 補充單一 skill 的 trigger, structure, steering 與 pruning. 本篇延伸到跨團隊目錄與治理.
- [不要重建 Agent, 改為建立 Skills](2025-12-08-build-skills-instead-of-agents-CEvIs9y1uog.md): 補充 skills 封裝程序知識與共用 harness 的設計方向.
- [Agentic Applications 實戰 Evals](2026-05-14-hands-on-agent-evals-Xfl50508LZM.md): 補充 traces, 評測資料集與受控實驗, 可作為行為驗證方法的參考.

## 來源與限制

本筆記依完整英文自動字幕重新整理, 時間戳採公開章節. 講者以 QuantumBlack 工程職務與多年服務組織的經驗提出建議, 但未在本場提供具名企業的治理前後追蹤資料, 完整平台程式碼或 rollout 成效.

本次未逐格核對投影片或外部 skills benchmark. 十五團隊案例是模擬, 規範審查是架構示例, 兩者皆不等同 production 驗證. 筆記將可直接追溯的主張與編輯補充分開, 不把 deterministic, 可攜與品質改善等主張視為無條件保證.

Skills 生態, harness 支援與平台功能均記錄影片當時的說法, 未作現況驗證. 部分自動字幕對人名與工具名稱有誤辨, 僅在 metadata 或上下文足夠明確時修正.
