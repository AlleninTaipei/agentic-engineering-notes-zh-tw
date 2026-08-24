# 如何打造有效的 AI Agents

> 來源: [How We Build Effective Agents: Barry Zhang, Anthropic](https://www.youtube.com/watch?v=D7_ipDqhtwk)
> 頻道: AI Engineer
> 發布日期: 2025-04-04
> 片長: 15:09
> Video ID: `D7_ipDqhtwk`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

Anthropic 的 Barry Zhang 將打造有效 agent 的經驗濃縮成三項原則:

1. 不要為所有問題都打造 agent。
2. 找到適合的場景後, 盡可能長時間維持簡單架構。
3. 迭代時站在 agent 的視角, 檢查它實際能看到的上下文。

Agent 適合處理複雜、模糊且高價值的任務, 但其自主性也會放大成本、延遲與錯誤後果。實作的核心不需要很複雜: 先建立環境、工具和 system prompt, 讓模型在迴圈中行動與取得回饋。行為穩定之後, 再針對成本、延遲和使用者信任進行最佳化。

## 從單次呼叫到 Agentic Systems

[01:04](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=64s)

AI 應用大致經歷三個階段:

| 階段 | 控制方式 | 適合的工作 |
| --- | --- | --- |
| 單次模型呼叫 | 一次輸入與輸出 | 摘要、分類、資訊擷取 |
| Workflow | 開發者預先定義多次模型呼叫與控制流程 | 決策路徑可描述、常見情境可枚舉的任務 |
| Agent | 模型根據環境回饋自行決定行動路徑 | 開放、模糊、難以事先列出完整決策樹的任務 |

Workflow 透過增加成本與延遲換取較好的表現, 也是 agentic system 的起點。Agent 則能依照環境回饋決定自己的 trajectory, 以近乎獨立的方式運作。

系統擁有越多 agency, 通常越有用、能力也越強, 但同時會提高三種代價:

- 執行成本。
- 回應延遲。
- 錯誤造成的後果。

## 原則一: 不要為所有問題都打造 Agent

[02:35](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=155s)

Agent 是擴展複雜、高價值任務的手段, 不是每個 use case 都應採用的直接升級。許多問題以 workflow 就能更具體、更可控地交付價值。

### 判斷清單

#### 1. 任務是否足夠複雜或模糊

Agent 擅長探索模糊的問題空間。如果能輕易畫出完整決策樹, 就應直接實作流程, 再逐一最佳化節點。這樣成本更低, 控制力也更高。

#### 2. 任務價值是否足以支付探索成本

Agent 的探索會消耗大量 token。演講以高流量客服為例: 如果每項任務的預算只有約 0.1 美元, 能使用的可能只有 3 萬至 5 萬 token。這時可用 workflow 處理最常見情境, 先取得大部分價值。

相反地, 若首要目標是無論花多少 token 都要完成任務, agent 就比較可能合理。

#### 3. 關鍵能力是否已去除風險

先確認 agent trajectory 上沒有重大瓶頸。以 coding agent 為例, 至少要驗證它能:

- 寫出良好的程式碼。
- 除錯。
- 從自身錯誤中恢復。

瓶頸不一定讓系統完全失敗, 但會成倍增加成本與延遲。遇到瓶頸時, 可先縮小範圍、簡化任務, 再重新驗證。

#### 4. 錯誤的代價與可發現性如何

如果錯誤後果嚴重, 而且又難以察覺, 就很難信任 agent 代表使用者採取自主行動。可以用唯讀權限或 human-in-the-loop 降低風險, 但這也會限制 agent 的可擴展程度。

### 為什麼 Coding Agent 是好場景

[05:05](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=305s)

從設計文件產出 pull request 是高度複雜且模糊的任務。高品質程式碼具有明確價值, 模型也已能處理許多 coding workflow 環節。更重要的是, 程式碼能透過 unit tests 和 CI 驗證, 讓錯誤較容易被發現。

## 原則二: 盡可能保持簡單

[05:47](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=347s)

Barry 將 agent 描述為「模型在迴圈中使用工具」。其基本結構只有三個部分:

```text
環境 Environment
  -> Agent 所操作的系統與當前狀態

工具 Tools
  -> Agent 採取行動及接收回饋的介面

System prompt
  -> Agent 的目標、限制與理想行為

模型反覆推理 -> 呼叫工具 -> 讀取環境回饋 -> 繼續推理
```

不同 agent 在產品介面、範圍和能力上可能差異很大, 底層骨架卻可以幾乎相同。環境通常由 use case 決定, 最重要的兩項設計決策是:

- 要提供哪些工具。
- 要用什麼 prompt 指示 agent。

過早加入複雜設計會拖慢迭代。先調整環境、工具和 prompt, 往往能得到最高投資報酬率。等到理想行為成形後, 再進行針對性最佳化:

- Coding 或 computer-use agent: 快取 trajectory 以降低成本。
- Search agent: 平行執行大量工具呼叫以降低延遲。
- 各類 agent: 清楚呈現進度, 建立使用者信任。

## 原則三: 像 Agent 一樣思考

[08:09](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=489s)

開發者常從自己的視角設計系統, 再對 agent 看似不合理的錯誤感到困惑。更有效的方法是把自己放進 agent 的 context window。

Agent 即使展現出複雜行為, 每一步仍是在有限上下文上進行 inference。它對世界現況的全部理解, 可能只來自當下的 1 萬至 2 萬 token。開發者應檢查這些資訊是否充分、連貫且沒有歧義。

### Computer-use Agent 的視角實驗

[09:04](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=544s)

想像自己只能取得一張靜態螢幕截圖和一段寫得不好的任務描述。你可以推理, 但只有工具呼叫能改變環境。送出點擊後, 在推理與工具執行的數秒內看不到畫面, 就像閉著眼睛操作電腦。下一張截圖出現時, 先前操作可能成功, 也可能造成完全不同的結果。

親自用這種限制完成一次任務, 會凸顯 agent 真正缺少的資訊。例如 computer-use agent 可能需要:

- 螢幕解析度, 才能正確計算點擊位置。
- 建議行動與限制, 以減少無效探索並建立 guardrails。
- 工具執行後足夠清楚的狀態回饋。

### 用模型協助理解模型

[10:32](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=632s)

因為模型能理解自然語言, 開發者可以請模型檢查:

- System prompt 是否含糊或互相矛盾。
- 工具說明是否足以讓 agent 正確使用。
- 工具需要更多參數, 還是可以減少參數。
- 完整 trajectory 中, agent 為何在某一步做出特定決策。
- 能提供哪些資訊, 幫助它做出更好的決策。

這種做法不能取代開發者對上下文的理解, 但能縮短人類視角與 agent 視角之間的差距。

## 三個尚待解決的方向

[11:34](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=694s)

### 1. 讓 Agent 具備預算意識

Workflow 的成本與延遲相對容易控制, agent 則缺乏同等精準的控制方式。若能定義並強制執行時間、金錢和 token 預算, 就能讓更多 agent use case 安全進入 production。

### 2. Self-evolving Tools

[12:12](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=732s)

模型已能協助改善工具說明。這個能力可以延伸成 meta-tool, 讓 agent 設計或改善自己的工具 ergonomics, 並依不同 use case 調整所需工具。這可能使 agent 更具通用性。

### 3. Multi-agent Collaboration

[12:40](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=760s)

多 agent 系統適合平行處理工作, 也能清楚分離關注點。Sub-agent 還能保護主要 agent 的 context window, 避免被所有細節塞滿。

核心問題在於 agent 之間如何溝通。多數系統仍建立在同步的 user-assistant turn 上, 未來需要支援非同步通訊、更多角色, 以及 agent 彼此辨識和協作的機制。

## 實作檢查表

開始打造 agent 前:

- 任務是否真的複雜且模糊?
- 任務價值能否負擔探索成本?
- 關鍵能力是否已通過獨立驗證?
- 錯誤是否容易被發現與修復?
- 能否先用 workflow 解決大多數情境?

開始迭代後:

- 先只聚焦環境、工具和 system prompt。
- 從 agent 可見的 context window 重播失敗過程。
- 確認工具說明、參數和回饋沒有歧義。
- 行為穩定後, 再最佳化成本、延遲與進度呈現。
- 對高風險行動限制權限, 或加入 human-in-the-loop。

## 核心結論

[13:41](https://www.youtube.com/watch?v=D7_ipDqhtwk&t=821s)

Agent 的價值來自自主探索, 但自主性並非免費。有效的設計不是堆疊複雜架構, 而是選對值得自主探索的問題, 建立最小可行的 agent loop, 再從 agent 實際擁有的資訊出發持續改善。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕整理, 並非逐字稿。內容已移除口語贅詞、重複字幕片段、開場致意與宣傳資訊, 以繁體中文重新組織。自動字幕可能誤辨專有名詞, 因此保留可回查的段落時間戳。演講沒有提供章節, 本文段落為編輯整理。
