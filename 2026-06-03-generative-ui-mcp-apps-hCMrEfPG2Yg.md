# Beyond Components: MCP Apps 與生成式 UI 的下一步

- 影片: [Beyond Components: Designing Generative UI for MCP Apps - Ruben Casas, Postman](https://www.youtube.com/watch?v=hCMrEfPG2Yg)
- 頻道: [AI Engineer](https://www.youtube.com/@aiDotEngineer)
- 講者: Ruben Casas, Postman Staff Engineer
- 上傳日期: 2026-06-03
- 片長: 16:58
- Video ID: `hCMrEfPG2Yg`
- 內容依據: YouTube 英文自動字幕與影片章節

## 摘要

Ruben Casas 將 agent UI 分成三個層級: static UI, declarative UI 與 generative UI。Static UI 讓 agent 選擇預先建立的 component 並填入 props。Declarative UI 讓 agent 產生 JSON 或 YAML descriptor, 再由 renderer 組合既有 design system。Generative UI 則讓 model 在 runtime 直接產生 HTML, CSS 與 JavaScript。

影片認為 declarative UI 是現階段在彈性, 一致性, 效能與成本之間最實用的平衡。完全生成式 UI 的想像空間更大, 但不能直接信任 model 產生的程式碼, 因此需要 containment, sandbox 與受控訊息通道。講者主張 MCP Apps 很適合作為這類 UI 的 delivery mechanism, 因為它已提供驗證, tool calling, message passing 與隔離執行環境。

更長期的方向不是持續增加 component, 而是建立人類和 agent 能共同操作的 shared artifact。影片以 Excalidraw MCP App 為例, 說明 canvas 可以同時接受 agent 生成與使用者直接編輯, 把 agent 從回應產生器轉變為協作夥伴。

## 從複製程式碼到高擬真 UI

[00:00](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=0s)

2022 年底的典型 AI 輔助前端流程, 是要求 ChatGPT 在 code block 中產生 component, 手動複製到專案, 發現問題後再反覆修正。講者稱之為「poor man's vibe coding」。當時能產生可執行的幾行程式碼已令人興奮, 但 model 還無法獨立負責高品質 UI。

到了 2025 年末, 講者觀察到 model 的長時間任務與高擬真 UI 生成能力快速提升。他以改寫個人 blog 為例, model 在未被明確要求的情況下建立了搜尋框, 模糊動畫與 accessibility 支援。這使他提出核心問題:

> 如果 model 已經很擅長撰寫前端程式碼, 為什麼 agent 介面仍主要由固定 UI 和文字聊天構成?

這是影片後續討論的前提, 不是經 benchmark 證明所有 model 已普遍超越專業前端工程師。

## 新電腦還沒有自己的 GUI

[02:56](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=176s)

講者引用 Andrej Karpathy 對 LLM 的比喻: 我們正在面對一種新的電腦, 目前與它互動的方式近似早期直接操作 terminal。能力已經出現, 但成熟的 graphical interface language 尚未形成。

現在常見的做法是把 chat 放進每個 SaaS 首頁。Chat 是可用的過渡介面, 但講者不認為它必然是最終形式。另一條路則是由 ChatGPT, Claude 或 Gemini 一類 super app 統一承載第三方 UI, 讓使用者在一個 agent environment 中操作多個服務。

影片把問題拆成兩個獨立維度:

1. UI 在哪裡執行: 每個產品都有自己的 chat, 或由 super app 承載第三方 UI.
2. UI 如何產生: 使用固定 component, descriptor 動態組合, 或在 runtime 生成程式碼.

講者沒有斷言哪種承載模式一定勝出, 並明確表示最終選擇仍會由使用者行為決定。影片主要聚焦第二個問題。

## UI 生成的三個層級

[05:52](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=352s)

| 層級 | Model 產生什麼 | Client 如何呈現 | 主要特性 |
| --- | --- | --- | --- |
| Static UI | Tool call, data 與 props | 對應預先建立的 component | 可預測, 但組合空間有限 |
| Declarative UI | JSON, YAML 或其他 descriptor | Renderer 將 descriptor 映射到 design system | 更動態且個人化, 仍受 component catalog 約束 |
| Generative UI | HTML, CSS, JavaScript 或 framework code | 在隔離環境執行 model 產生的 UI | 彈性最高, 同時帶來信任與安全問題 |

三者不是單純的新舊替代關係。對不同產品而言, 可預測性, 品牌一致性, latency, token 成本與自由度的優先順序不同。

## Static UI: Agent 只負責選擇與填值

[06:03](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=363s)

Static UI 是目前最常見的 agent UI 模式。Developer 先建立 React 等 component, agent 作為 orchestrator 呼叫 tool, 將資料與 props 傳給 component, client 再負責 render。

```text
user intent
  -> agent 選擇 tool
  -> tool call 產生 data 和 props
  -> client 映射到預先建立的 component
  -> render UI
```

這和傳統 client-server UI 很接近。主要差異在於 props 可以由 agent 根據自然語言意圖產生。

### AG-UI

影片以 AG-UI SDK 為例。開發者註冊一個 client tool, 將它映射到 React component。Agent 呼叫 tool 並提供 props, client 再把 props 傳給固定 component。

### Goose Auto Visualizer

Goose 是支援 MCP 的 client。它的 Auto Visualizer 可以接收不同類型的資料, 判斷適合的排列方式, 再從 Goose 團隊預先建立的 visual components 中挑選並呈現。它具備自動選擇能力, 但 component 本身不是 model 即時創造的。

## Declarative UI: Model 產生介面描述

[07:42](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=462s)

Declarative UI 保留 developer 建立的 component 與 design system, 但 agent 不只提供單一 component 的 props, 而是產生描述整體 UI 結構的 descriptor。Descriptor 可以使用 JSON, YAML, 甚至 Python 表示, 再由 translation/rendering engine 轉換成實際畫面。

```text
user intent
  -> agent 產生 JSON/YAML descriptor
  -> renderer 驗證並解析 descriptor
  -> 從核准的 component catalog 組合 UI
  -> render 個人化介面
```

這個概念並非完全新創。Netflix 的 personalized, server-driven UI 已長期根據個別使用者組合首頁, 但組合結果仍映射到 Netflix 自己的 UI elements。

### JSON Render

影片介紹 Vercel 的 JSON Render。它讓 model 透過 JSON 或 YAML 描述 UI, 再映射到既有 components, 可以建立比單一 static component 更動態的互動流程。

限制也很清楚: LLM 生成的是 descriptor, 不是 component code。它能自由組合 catalog 中的元素, 但不能越過 catalog 創造全新的 primitive。

### 為什麼它是現階段的平衡點

講者認為 declarative UI 在當下可能是彈性與一致性之間最好的平衡:

- 能根據任務與使用者生成不同 layout.
- 保留產品 design system 與品牌一致性.
- 輸出結構可驗證, 行為較可預測.
- 通常比生成完整前端程式碼更快.
- Descriptor 較短, token 與 inference 成本可能較低.

這是講者的工程判斷, 影片沒有提供 latency, token 或成本 benchmark。

## Generative UI: 在 Runtime 產生 Component

[10:06](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=606s)

Generative UI 直接利用 coding model 的前端能力。Agent 可以透過 tool call 觸發同一 model 的 reverse sampling, 或呼叫另一個 model, 按需求產生 HTML, CSS 和 JavaScript, 再交給 client 顯示。

```text
user intent + domain data
  -> agent 或 UI-generation model
  -> 產生 HTML/CSS/JavaScript
  -> containment 與安全檢查
  -> sandbox 中執行
  -> UI 事件透過受控通道回到 agent
```

這種方式不需要預先準備完整 component catalog, 可以針對當下資料與情境創造介面。代價是輸出變得不確定, 每次生成的視覺, 程式結構和行為可能不同。

### Postman Weather Agent 實驗

講者在 Postman 製作 weather agent 實驗。Agent 呼叫 weather API, 生成笑話, 並在一次 tool call 中產生 HTML, CSS 與 JavaScript。結果是每次都可能不同, 但具有想像力的天氣介面, 中間沒有固定 component 或 descriptor translation layer。

這個示範證明 runtime generation 的流程可行, 但影片沒有提供 production deployment, 安全測試或可靠性數據。

## 最大障礙不是生成能力, 而是信任

[11:25](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=685s)

如果第三方程式碼不應直接取得 client 權限, LLM 即時產生的程式碼更不應被無條件信任。Generative UI 因此需要的不只是 rendering API, 而是一個具備安全邊界的 distribution model。

影片點出的必要條件包含:

- Containment: 限制生成程式碼可影響的範圍.
- Sandbox: 將 UI 與 host application 隔離.
- Controlled message passing: UI 不能任意存取 agent 或 host 狀態.
- Authentication: 使用既有身分與服務授權機制.
- Tool mediation: 需要外部能力時, 經受控 tool call 執行.

影片沒有深入說明 CSP, network policy, capability permission, data exfiltration 或 generated-code validation。這些仍是從 prototype 走向 production 時必須補上的安全設計。

## 為什麼 MCP Apps 適合傳送 Generative UI

[12:22](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=742s)

講者主張 MCP Apps 是 generative UI 的合適 delivery mechanism, 原因是它整合了 MCP 的 authentication 與 tool calling, 並定義 UI 和 agent 之間的 message passing。UI 預設在雙層 iframe 形成的 sandbox 中執行, 延續第三方 UI 的隔離模型。

這項能力不只服務第三方 UI。影片提到 Anthropic 的 visualizer 使用 MCP Apps 承載第一方生成式 UI。講者認為這具有策略意義: 即使 host 能自行建立專用 renderer, 採用 MCP Apps 仍可重用既有的隔離, 互動與協議能力。

需要區分的是, 影片提出的是架構適配性的論點, 不是 MCP Apps 已解決所有生成程式碼安全問題的證明。Sandbox 是必要邊界, 但不是完整的安全結論。

## 不要用舊媒體想像新介面

[13:21](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=801s)

Chat, floating windows 或類似 Jarvis 的介面可能只是我們用現有模式想像未來。講者以早期電視節目為例: 電視剛出現時, 節目常像「裝上攝影機的廣播」, 因為創作者尚未發展出新媒體自己的表達語言。

同樣地, 今天的 agent UI 仍可能處於「radio shows with cameras」階段。Static component, chat panel 與 dashboard 是已知形式的延伸, 不一定代表 agent-native interface 的最終樣貌。

講者明確將這段視為推測。他沒有宣稱 chat 或 MCP Apps 已經是最終介面。

## Beyond Components: 人類與 Agent 共用 Artifact

[14:46](https://www.youtube.com/watch?v=hCMrEfPG2Yg&t=886s)

影片認為比「讓 agent 產生更多 visualizations」更重要的方向, 是建立 human-agent collaboration space。UI 不再只是 agent 的輸出, 而是雙方能持續修改的共享工作物。

### Excalidraw MCP App

Excalidraw MCP App 建立可共同操作的 canvas:

- Agent 能生成或修改 diagram.
- 使用者能以熟悉的直接操作方式移動, 新增或刪除元素.
- 使用者能再用自然語言要求 agent 調整內容.
- 雙方針對同一份 artifact 來回迭代.

```text
agent generation
       \
        -> shared canvas -> persistent artifact
       /
human direct manipulation
```

這與一次性的 response UI 不同。共享 artifact 同時保留自然語言的高階控制與 GUI 的精確直接操作, 可能成為 agent interface 的重要模式。

## 編輯整理: 選擇 UI 生成策略

以下是根據影片內容整理的決策框架, 不是講者逐字提供的 checklist。

### 選擇 Static UI

- 操作流程固定, component 數量可控.
- 法規, 品牌或測試要求高度可預測.
- Agent 只需選擇介面並填入資料.
- 團隊希望沿用成熟 component library.

### 選擇 Declarative UI

- 同一套 design system 需要支援大量動態組合.
- 介面要依任務或使用者個人化.
- 團隊需要 schema validation 與可預測的能力邊界.
- 完整 code generation 的 latency, token 成本或風險仍太高.

### 考慮 Generative UI

- 任務形態無法用既有 component catalog 預先涵蓋.
- 高度個人化或創意呈現能產生實質價值.
- Host 已有足夠的 sandbox, permissions 與 message mediation.
- 團隊能測試不確定輸出, 並接受額外 latency 和成本.

### 優先考慮 Shared Artifact

- 任務需要多輪修改, 而非一次展示結果.
- 人類需要直接操作細節, agent 負責較高階的生成與重構.
- 成果需要在互動後持續保存, 驗證或交付.

## 核心結論

影片的核心不是宣稱 generative UI 應立即取代 static UI, 而是將設計空間分層:

```text
Static UI
  可預測, 成熟, 自由度有限
       |
Declarative UI
  動態組合, 保留 design system
       |
Generative UI
  runtime code generation, 需要強隔離
       |
Shared Artifact
  從展示結果走向人機共同工作
```

短期內, declarative UI 可能是較務實的預設。Generative UI 的價值取決於安全 delivery mechanism, 而 MCP Apps 提供了可重用的基礎。長期來看, 真正的突破可能不是產生更漂亮的 component, 而是讓使用者與 agent 能在同一個互動空間中共同建立成果。

## 來源與可信度限制

本文依據 YouTube 英文自動字幕與影片章節整理, 並非逐字稿。自動字幕可能誤辨產品名稱, 人名與技術詞彙。本文只在上下文足夠明確時修正, 例如 `MCP Apps`, `AG-UI`, `Goose`, `JSON Render`, `Excalidraw` 與 `Postman`。

影片主要是架構觀點, prototype 與未來方向, 未提供效能 benchmark, token 成本比較, 安全評估, production incident 或使用者研究。Model 進步速度, 產品功能與 MCP Apps 規格都具有時效性。實作前應查閱最新官方規格, SDK 與目標 host 的安全模型。
