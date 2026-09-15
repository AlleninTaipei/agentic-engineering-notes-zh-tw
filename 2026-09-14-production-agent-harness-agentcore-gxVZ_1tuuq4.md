# Production Agent 的 Harness Engineering: 從本機工具到獨立擴展的 Runtime 與 Memory

> - 影片: [Harness Engineering: Building the Production Cage for Powerful Domain Agents - Mike Chambers, AWS](https://www.youtube.com/watch?v=gxVZ_1tuuq4).
> - 頻道: [AI Engineer](https://www.youtube.com/channel/UCLKPca3kwwd-B59HNr-_lvA).
> - 講者: Mike Chambers, AWS Senior AI Specialist Developer Advocate, 職稱依口述與公開說明.
> - 發布日期: 2026-09-14. 此為 YouTube 上傳日期, 不代表錄影日期.
> - 片長: 20:46.
> - Video ID: `gxVZ_1tuuq4`.
> - 內容依據: 完整英文自動字幕 `en-orig`, 搭配公開 metadata 與章節. 無創作者字幕.
> - 來源性質: AWS 第一方產品架構分享與現場操作示範. 未逐幀核對程式碼, 未執行部署, 未驗證產品現況或規模化成效.

## 核心觀點

Mike 將 agent 分成「我們使用的」與「我們建造的」兩種視角. 使用 coding assistant 時, 團隊主要關心指引, 記憶與工具配置. 建造要服務大量使用者的 agent 時, 還要處理 runtime, identity, scaling, context management, observability 與 evaluations.

他的 harness 定義很寬: 從 agent 拿掉模型, 剩下的部分就是 harness. 在這個定義下, harness engineering 包含將不同責任拆成可獨立運作與擴展的元件, 而不只是撰寫模型的工具呼叫迴圈.

演講透過 Strands Agents SDK 與 Amazon Bedrock AgentCore, 從簡單本機 agent, 加入 session manager, 再走到獨立記憶服務與雲端 runtime. 結尾則展示以 JSON 設定建立 agent 的另一條路徑.

## 1. 使用 Agent 與建造 Agent, 面對不同責任

[02:45](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=165s) / [04:24](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=264s)

這兩種分類可以相互連接: 一個團隊建造的 agent, 會成為另一個人的工作工具. 差別在於當下承擔的是使用責任, 還是服務提供者的責任.

| 視角 | 演講關注的問題 |
| --- | --- |
| 使用 agent | 如何讓 coding assistant 使用團隊的 memory, skills, tools 與 MCP servers |
| 建造 agent | 如何為實際使用者安排執行, 身分, 記憶, 擴展與驗證 |

[05:16](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=316s) 起, 講者把 coding standards 延伸為 harness standards: 團隊需要一致配送 coding assistant 應使用的工具與規範. 他提到 Agent Toolkit for AWS 作為輔助入口, 但影片公開說明未附對應 repository, 本文不推測 QR code 的網址.

## 2. 避免讓 Agent 直接操作雲端, 卻沒有留下部署定義

[06:07](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=367s)

講者將 agent 隨手建立雲端資源的做法稱為 slop ops, 類比過去在 console 手動操作的 click ops. 他區分了探索環境與正式部署:

- 透過 console 或 agent 查明系統狀態, 可以幫助理解問題.
- 正式部署應讓 agent 產生 infrastructure as code, 再由該定義建立所需資源.

影片以建立 S3 bucket 或 EC2 instance 為例. 講者希望團隊保有自己的部署定義, 而非只得到一組 agent 已經建立的資源.

編輯整理: 這是對部署工作流的建議. 影片沒有展示 IaC 的審查, 狀態漂移偵測或 rollback, 因此不把這些後續能力當作已完成的示範.

## 3. Production Harness 包含可獨立擴展的元件

[06:57](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=417s)

講者列出的責任包括 loop management, scaling, payments, memory, identity, skills, runtime 與 context management. 他最後提到 observability 和 evaluations, 同時強調應該最先考慮它們.

面對數千名使用者, 他不建議將所有責任都寫進同一個 container, 再將整體一起擴展. 不同元件應能按需求獨立擴展. 演講接下來實際聚焦的是 runtime 與 memory 的分離, 沒有逐一實作上述所有元件.

以下是依口述整理的概念關係, 非原片架構圖重製:

```text
Agent 應用
  -> 模型與工具互動
  -> Runtime 與 session 執行
  -> 獨立的記憶服務
  -> 身分與隔離機制
  -> Observability 與 evaluations
```

這段是在說明架構方向. 影片未提供負載測試或瓶頸分析, 不能據此判定每個小型 agent 都需要拆成多個服務.

## 4. 第一版: 有工具與迴圈的本機 Agent

[08:38](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=518s)

講者以 Strands Agents SDK 建立最小範例. Agent 接收 system prompt 與工具, 包括套件提供的 calculator, 以及以 tool decorator 定義的時間工具. Framework 管理 agent loop.

這已包含基本 harness, 但尚未處理 production 所需的其他責任. 程式在講者筆電上執行, 沒有展示多使用者部署或擴展能力. 他在這一段主要展示程式結構, 未實際執行時間提問.

## 5. 第二版: Session Manager 與檔案記憶

[10:16](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=616s)

第二版由講者請 Kiro 產生, 仍使用 Strands. 主要新增 session manager, 在不同 invocation 之間保存狀態, 並於下次互動時重新載入 conversation history.

範例同時包含 `remember` 工具, 讓 agent 決定是否把使用者資訊存入檔案. 這兩者的角色不同:

| 機制 | 演講中的用途 |
| --- | --- |
| Session manager | 恢復先前對話, 延續 session context |
| `remember` 工具與檔案 | 保存可在之後取用的使用者資訊 |

講者問誰會贏得世界盃時, Agent 的回答提到他希望澳洲獲勝. 講者以此說明先前對話可影響後續回覆, 並指出模型並不知道真正結果.

這是記憶使用的示範, 不是預測能力展示. 筆記無法僅依口述判斷該資訊究竟來自 conversation history 或 `remember` 寫入的檔案. 演講也沒有提供記憶提取準確率, 保存政策或使用者間隔離測試.

## 6. 透過 AgentCore CLI 建立部署專案

[12:50](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=770s)

接下來的目標是把 agent 部署至雲端, 並讓 memory 成為與 agent 分開運作的基礎設施. 講者使用 AgentCore CLI 的互動式流程建立專案, 大致選擇如下:

1. 建立 agent, 暫時跳過名為 harness 的選項.
2. 由 CLI 產生初始程式, 而非匯入既有程式.
3. 選擇 Python; 介面也提供 TypeScript.
4. 選擇 HTTP 連接方式; 口述另提到 MCP 與 AG-UI.
5. 使用 Strands Agents SDK 與預設模型.
6. 選擇部署 short-term 與 long-term memory.

講者表示可使用不同 framework 或自寫 agent, 模型也不限定來自 Amazon Bedrock. 這是演講對平台範圍的描述, 未逐一示範整合.

Memory 選項會建立獨立的雲端記憶基礎設施, 與 agent 連接並可分開擴展. 講者提到非同步運作, 但沒有展開寫入時機, 一致性或提取流程.

範例中的預設模型名稱在自動字幕中不清楚, 本文不將其還原為確定版本. 初始建立命令也未從口述完整取得, 因此不補寫可執行的初始化步驟.

## 7. Runtime 接手隔離與擴展, 應用保留工具與記憶連接

[15:18](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=918s)

講者切換到預先準備的專案, 展示與 AgentCore app 的連接, 工具, MCP, session manager 與 memory 設定. 相較純本機版, 應用多了與託管平台整合的部分.

他表示, 這讓開發者能先寫服務單一使用者的 agent, 再由 runtime 提供多租戶隔離與擴展, 減少自行撰寫相關程式的工作.

這是平台能力的第一方說明. 影片未展示 tenant identity 如何傳遞, memory 的租戶範圍, 配額, 隔離測試或高併發結果. 因此不能把「不用自行寫多租戶程式」解讀為完全不必設計或驗證使用者權限.

## 8. 本機開發與已部署環境的觀察入口

[16:59](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=1019s)

講者口述的兩個命令如下, 僅保留演講中的形式, 未在本次環境執行:

```bash
agentcore dev
agentcore deploy
```

依演講描述, `agentcore dev` 提供與本機 agent 互動的瀏覽器介面, 程式變更會反映在開發過程中. 示範以簡單問候確認能與 agent 對話.

`agentcore deploy` 則使用 infrastructure as code 部署 agent runtime, memory 與選定元件. 同一介面可切換至已部署版本, 並查看 traces 與已存記憶, 協助除錯.

現場使用了預先部署的環境, 並沒有完整呈現從空白帳號到成功部署的全流程. Observability 的入口有被介紹, 但 eval dataset, 指標與上線門檻沒有展開.

## 9. JSON 設定式 Harness 與按需採用元件

[18:40](https://www.youtube.com/watch?v=gxVZ_1tuuq4&t=1120s)

結尾回到初始化流程中跳過的 harness 選項. 講者展示簡單 JSON 設定, 指定模型與 system prompt, 也可透過 `agentcore deploy` 部署. 這條路徑讓範例不必自行撰寫 agentic code.

影片沒有口述完整 JSON schema, 本文不重建設定檔. 這也不表示底層不需要程式, 而是由既有 harness 提供執行能力.

講者推測, 若許多工作只需 system prompt 與 MCP tools, 也許約 80% 的 agent 開發已經有現成解法. 這是概括性主張, 未附使用案例統計或成功率評估, 不宜當作導入估算.

另一個重點是元件可組合採用. 若已有運作良好的 production agent, 也可以只整合託管 long-term memory, 不必一次遷移整套系統. 演講未提供這類局部整合的實際程式或遷移成本.

## 編輯整理: 從示範走向 Production 前的檢查問題

以下為依影片內容整理的檢查問題, 非原講者交付的完整 checklist.

1. 部署結果是否由版本控制中的 IaC 定義, 而不是只有 agent 執行過的操作紀錄?
2. Session history 與長期記憶各自保存什麼, 何時寫入與讀取?
3. 哪個元件實際遇到擴展瓶頸, 是否需要與 runtime 分開配置?
4. 使用者身分如何傳入工具與 memory, 如何驗證不會讀到其他租戶資訊?
5. Traces 能否定位失敗步驟, evals 是否涵蓋記憶錯誤與工具失敗?
6. 若只採用一個託管元件, 整合成本, 延遲與故障處理是否可接受?

## 來源與限制

- 筆記依完整英文自動字幕轉述, 移除寒暄與重複片段. 時間連結採影片公開章節起點.
- Kiro, Strands, AgentCore 與 slop ops 等明顯誤辨依上下文整理. 未納入不影響主線且難以確認的開場人名與機構資訊.
- 未逐幀檢視程式碼或 QR codes, 也未取得範例 repository. 不推測畫面中未口述的設定與連結.
- CLI, 支援範圍與平台行為均為演講當時描述, 不是現行操作文件. 本次未查證影片以外的產品資訊.
- 影片有本機互動與既有部署的操作展示, 但缺少負載測試, 成本, 多租戶安全測試與正式 eval 結果. 規模化與隔離成效仍屬供應商陳述.
