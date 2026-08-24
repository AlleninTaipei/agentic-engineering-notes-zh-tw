# Anthropic CCA Exam: Agentic Engineering 的實戰指南

> 來源: [Anthropic's CCA Exam as a Field-Guide for Agentic Engineering — Frank Coyle, UC Berkeley](https://www.youtube.com/watch?v=Z-c11pV_uvU)  
> 頻道: AI Engineer  
> 講者: Frank Coyle, UC Berkeley  
> 發布日期: 2026-08-08  
> 影片長度: 20 分 8 秒  
> Video ID: `Z-c11pV_uvU`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 核心摘要

Frank Coyle 將 Anthropic 的 Claude Certified Architect, CCA, exam 視為一份 agentic engineering field guide. 即使不準備應試, 考試涵蓋的領域和 production scenarios 仍能幫助工程師理解如何建造可靠的 agent systems.

影片特別強調 anti-patterns. 同一個問題通常有多種可行解法, 但先知道哪些做法會造成工具誤用、context 膨脹、groupthink 或 CI 卡住, 往往更容易找到可靠設計.

主要實務原則是:

- Agent loop 必須檢查 `stop_reason`.
- LLM 不會真的執行工具, harness 才是執行者.
- Instructions 應依 scope 分層管理.
- Agents 應專業化, 不要給單一 agent 過多工具與責任.
- Subtasks 應隔離 context, 只回傳必要摘要.
- 長 session 應控制 token growth 並適時 compact.
- CI pipeline 不應依賴 interactive approval.

## 1. 學習哲學: 先做, 再從失敗中找規律

講者引用 Sister Corita Kent 的觀點: 沒有勝負或失敗, 只有創作. 他鼓勵學習者不只閱讀, 還要反覆建造與實驗.

Thomas Edison 的名言也被用來說明: 沒成功的嘗試可以揭露哪些方法不工作. 在 agentic engineering 中, 這些反覆失敗就是 anti-patterns 的來源.

Design patterns 告訴我們可重複使用的有效結構, anti-patterns 則揭露看似合理、實際上容易失敗的做法. 考試題目的關鍵往往不是背誦唯一答案, 而是辨認不安全或不可靠的選項.

## 2. CCA Exam 的形式

影片表示, 該認證考試於當年 3 月推出, 具備以下特徵:

- Scenario-based.
- Timed.
- Proctored.
- Multiple choice, 但題目建立在 realistic constraints 與 production scenarios 上.
- 企業可透過 Anthropic ecosystem 參與.
- 個人可付費應試, 並受到重考間隔限制.

影片提到的價格為 99 美元, 每六個月可考一次. 這些屬於發布時的講者陳述, 考試名稱、資格、費用與重考政策都可能更新, 報名前應查閱官方頁面.

## 3. 五個考試領域

[03:26](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=206s)

講者列出五個 domains:

| Domain | 影片提到的比重或內容 |
| --- | --- |
| Agentic architecture | 約 27%, 包括 agent loops 與 system design |
| Claude Code configuration and workflows | 約 20%, 包括專案設定與工作方式 |
| Prompt engineering and structured output | Prompt 結構、JSON 與可靠輸出 |
| Tool design and MCP integration | 工具介面與 Model Context Protocol |
| Context management and reliability | Context 控制、隔離與穩定執行 |

影片只明確說出前兩項百分比. 其他項目在字幕中沒有可靠辨識到精確比例, 因此本文不補造數字.

這五個領域共同反映一件事: Agentic engineering 不只是寫 prompt. 它同時涉及 control flow、tool protocol、context architecture、execution environment 與 failure handling.

## 4. 六種 Production Scenarios

[04:20](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=260s)

影片表示考試提供六種 production scenarios, 實際考試會隨機選取其中四種作為題目背景:

1. Customer support resolution agent.
2. Code generation with Claude.
3. Multi-agent research system.
4. Developer productivity with Claude Code.
5. Claude Code for continuous integration.
6. Structured data extraction.

演講時間有限, 對前五項的 anti-patterns 有較具體說明; structured data extraction 只在總覽中被提及, 沒有足夠細節可可靠擴寫.

## 5. Loops 並不是全新的概念

[06:34](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=394s)

Agent 社群常說現在的工作是「設計 loops」. 講者同意 loop 是 agentic systems 的關鍵, 但提醒它並非新發明.

他回顧 Böhm 與 Jacopini 在 1966 年提出的 structured-programming 結果: 具有順序、條件分支和迴圈的控制結構, 足以表達一般可計算程序.

Agentic applications 同樣遵循這個骨架:

```text
依序執行步驟
  -> 根據結果分支
  -> 重複呼叫模型或工具
  -> 滿足停止條件後結束
```

Agent loop 的新意不在「有迴圈」, 而在 loop 中的決策者是 probabilistic model, 並需要 harness 處理工具執行、錯誤、token limits 和 human escalation.

## 6. Scenario 1: Customer Support Resolution Agent

### Anti-pattern: 呼叫模型一次就直接使用答案

模型可能要求使用工具、輸出被截斷、達到 token limit, 或產生低信心結果. 如果 harness 不檢查回傳狀態便直接把文字送給客戶, 系統可能使用不完整或未驗證的答案.

### 正確模式: 檢查 `stop_reason`

[08:17](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=497s)

影片示範一個 `while true` agent loop:

```text
呼叫模型
  -> 讀取 stop_reason
  -> 若為 tool_use, 由程式執行工具
  -> 把 tool result 加回 messages
  -> 再呼叫模型
  -> 若產生完整答案, 離開 loop
  -> 依 confidence 決定直接使用或交給人類
```

### LLM 不會執行工具

模型只會產生結構化的 tool request, 例如 tool name 與 arguments. 真正的執行由 application/harness 完成. 執行結果再以 tool-result message 放回 context, 讓模型繼續推理.

這個責任分界很重要:

- Model: 選擇工具並產生參數.
- Harness: 驗證參數、執行工具、處理權限與錯誤.
- Model: 讀取結果並決定下一步.

### Token exhaustion 也是 stop reason

[10:57](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=657s)

若模型因 token limit 停止, 它仍可能留下看起來像答案的部分文字. Harness 必須辨認這是 truncated output, 不能將其當作完成結果. 可採取的動作包括重試、縮減 context、要求續寫或 escalation.

## 7. Human-in-the-loop 的位置

Customer-support loop 結束後, 還可以依 confidence、風險或 policy 判斷是否交給人類.

適合 escalation 的情況包括:

- Confidence 低.
- 工具執行失敗.
- Output 被截斷.
- 涉及退款、帳戶權限或敏感資料.
- 模型結果與 policy 衝突.
- Loop 達到最大次數仍未完成.

影片只直接提到 confidence 與 token exhaustion. 其餘項目是依該控制模式整理的編輯補充, 並非影片逐項列舉.

## 8. Scenario 2: Code Generation with Claude

[11:19](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=679s)

影片介紹以 Markdown instructions 控制 Claude Code 行為. 講者描述三層 scope:

- 高層或全域專案規則.
- Project-level instructions.
- Directory-specific instructions.

這種 hierarchy 讓較通用的規則放在上層, 特定子目錄的語言、測試或架構規範則靠近相關 code.

實務上的設計原則是:

- 規則放在最小但足夠的適用 scope.
- 不要在每個目錄重複整份相同內容.
- 子層規則應補充或具體化上層規則.
- 避免彼此矛盾的 instructions.

影片所稱的檔名和三層精確位置可能因產品版本或專案設定而異, 因此本文保留概念, 不將其寫成固定安裝路徑.

## 9. Scenario 3: Multi-agent Research System

[12:04](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=724s)

### Anti-pattern: 一個 Agent 擁有所有工具與責任

講者用「同時帶著水電、木工與電氣工具的工人」比喻 overloaded agent. 工具太多、責任太廣時, agent 更難選擇正確行動, context 也會被大量 tool descriptions 占據.

### 正確模式: 專業化 Agents

每個 agent 應有狹窄、清楚的任務, 並只取得必要工具. 這呼應 functional programming 中函式應做一件事的原則.

例如 research system 可拆成:

- Researcher: 蒐集證據.
- Synthesizer: 整理多個來源.
- Critic: 檢查 claim 是否受 evidence 支持.
- Orchestrator: 分派工作並組合必要結果.

這些角色是依影片所述模式整理的範例. 影片具體展示的是 critic 只接收 `claim` 與 `evidence`.

## 10. Context Isolation 與 Groupthink

### Anti-pattern: Agents 共享完整思考過程

若 critic 同時看到原研究 agent 的完整 reasoning, 它容易沿用既有假設, 失去獨立檢查能力. 多個 agents 不斷交換全部內容, 也可能逐漸收斂成 groupthink.

### 正確模式: 只給完成角色所需的 slice

Critic 只需看到:

```json
{
  "claim": "需要驗證的主張",
  "evidence": "支持或反駁主張的資料"
}
```

不傳遞原始 chain of thought 或所有探索紀錄, 可以:

- 保留 critic 的獨立性.
- 減少 token cost.
- 避免主 context 污染.
- 讓輸入介面更容易測試.

## 11. Scenario 4: Developer Productivity

[15:22](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=922s)

### Anti-patterns

- 每個 subtask 都把完整輸出倒回 primary thread.
- 讓 context 無限制增長.
- 把所有 logs 和 intermediate reasoning 保留在主 session.

### Context Fork

[16:13](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=973s)

影片以分析 logs 為例:

1. 從主 agent 派生獨立 context.
2. Subagent 掃描全部 logs 並找 errors.
3. Subagent 在自己的 context 中處理大量細節.
4. 只將精簡 summary 傳回主 context.

這讓主 thread 保留任務方向和決策, 不被大量低階內容淹沒.

### Compaction

影片示範監控 token count, 超過某個門檻後執行 compact. 範例門檻是 150,000 tokens, 但這不是普遍建議值. 合理門檻取決於模型 context、任務類型和 compact 品質.

Compaction 會將長 conversation 壓縮成摘要. 它可以釋放空間, 也可能遺失資訊. 若領域有特殊需求, 團隊可以設計自訂 compression logic, 保留真正重要的狀態.

## 12. Scenario 5: Claude Code for CI

### Anti-pattern: 在 Pipeline 中使用 Interactive Mode

CI 無人值守. 若 agent 在 pipeline 中停下來詢問是否允許命令或要求下一步輸入, job 就會卡住或 timeout.

CI agent 應具備:

- Non-interactive configuration.
- 明確的 permissions.
- 固定 timeout 與最大迴圈次數.
- Machine-readable output.
- 失敗時的 logs 與 exit status.
- 對 secrets 與外部工具採最小權限.

前兩項來自影片直接說明, 後四項是依 non-interactive CI 模式整理的工程補充.

## 13. Batch Processing

[18:45](https://www.youtube.com/watch?v=Z-c11pV_uvU&t=1125s)

影片提到可將不需即時結果的 prompts 或工作提交為 batch. 講者表示, 發布當時 batch 可降低約 50% token cost, 結果承諾在 24 小時內返回.

適合 batch 的工作:

- 大量離線評估.
- 不緊急的資料抽取.
- 夜間或休假期間執行的分析.
- 可獨立重試的批次任務.

不適合 batch 的工作:

- 需要即時 human feedback.
- 高度互動的 debugging.
- 下一步依賴立即結果的 agent loop.

折扣、服務承諾與支援模型可能變動, 實際使用前應查閱當下官方 API 文件.

## 14. Scenario 6: Structured Data Extraction

影片在開頭把 structured data extraction 列為第六種 production scenario, 也提到 prompt engineering 與 JSON output, 但沒有在後續時間內完整展開.

因此可以可靠確認的只有:

- 結構化輸出是考試與 agentic system design 的重要主題.
- JSON 是模型與工具、agent 與 agent 之間常用的明確介面.

本文不依一般知識推測該 scenario 的考題或標準答案.

## Anti-pattern 對照表

| Anti-pattern | 可能問題 | 較佳模式 |
| --- | --- | --- |
| 只呼叫模型一次就使用答案 | 忽略 tool use、truncation 或錯誤狀態 | Loop 並檢查 `stop_reason` |
| 認為 LLM 自己執行工具 | 權限、驗證與錯誤責任不清 | Harness 驗證並執行 tool request |
| 單一 agent 配置所有工具 | Tool selection 混亂、context 過大 | 建立 specialized agents |
| Critic 讀取全部原始 reasoning | Groupthink、失去獨立性 | 只傳 claim 與 evidence |
| Subtasks 將完整輸出倒回主 thread | 主 context 被低階細節占滿 | Context fork + summary |
| Context 無限制增長 | 成本與回答品質惡化 | 監控 tokens, 必要時 compact |
| CI 使用 interactive approval | Pipeline 卡住 | Non-interactive、有限權限的執行模式 |
| 即時執行所有離線工作 | 成本較高 | 合適工作改用 batch |

## 一套可靠的 Agent Loop

```text
初始化 messages 與工具
  -> 呼叫模型
  -> 檢查 stop_reason
     -> tool_use: 驗證參數, 執行工具, 加入結果, 繼續 loop
     -> token limit: 縮減 context、重試或 escalation
     -> completed: 驗證結構與 confidence
  -> 低信心或高風險: Human-in-the-loop
  -> 通過驗證: 返回結果
```

## 學習重點

- 把 agent system 當作受控程式, 不要當作一次性的聊天回答.
- 明確區分 model decision 與 tool execution.
- 所有 loop 都要有停止、錯誤和資源上限.
- Instructions、agents 和 contexts 都要依責任分層.
- Specialized agents 只取得完成角色所需的資料與工具.
- 主 context 應保存決策, 大量探索留在 forked contexts.
- Compact 是有損壓縮, 重要狀態應另行保存.
- CI 必須可無人值守, 同時保持最小權限.
- 考試規格可以作為學習地圖, 但真正能力來自反覆建造與觀察失敗.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 影片沒有公開章節. 本文的章節與時間標記依內容轉折建立.
- 自動字幕將 `Claude`、`Claude Code`、`context` 與部分姓名辨識成近音詞. 本文只在上下文可可靠確認時修正.
- CCA exam 名稱、推出日期、價格、重考間隔、domain 權重與 scenario 選取方式來自講者現場陳述, 本文未用外部官方資料獨立驗證.
- Batch API 的 50% 折扣與 24 小時回應承諾也是發布時資訊, 可能已調整.
- Human escalation 清單、CI 的部分工程要求、agent 角色範例與 batch 適用性是清楚標示的編輯補充.
- 本文沒有補造影片未展開的 structured-data-extraction scenario.
- 中文標題、表格與流程圖式文字是編輯整理. 本文不是逐字稿, 也不取代完整演講、實際考試指南或最新官方文件.
