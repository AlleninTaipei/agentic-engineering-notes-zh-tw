# MCP Apps: 把互動式 UI 帶進 AI 對話介面

- 影片: [MCP UI: Extending the frontier - Liad Yosef and Ido Salomon, MCP Apps](https://www.youtube.com/watch?v=o-zkvb0iFDQ)
- 頻道: [AI Engineer](https://www.youtube.com/@aiDotEngineer)
- 講者: Ido Salomon, Liad Yosef
- 上傳日期: 2026-05-06
- 片長: 22:20
- Video ID: `o-zkvb0iFDQ`
- 內容依據: YouTube 英文自動字幕與影片章節

## 摘要

MCP Apps 是 Model Context Protocol 的官方延伸, 讓 MCP server 不只回傳文字或結構化資料, 還能把可互動的 HTML UI 資源交給支援它的 host, 例如 ChatGPT, Claude, VS Code 或其他 agent 介面。Host 會在沙箱中呈現 UI, 並管理 UI, model 與 MCP server 之間的訊息傳遞。

這項設計想解決兩個問題。第一, 純文字不適合呈現圖表, 商品卡片, 表單或複雜操作。第二, 如果嵌入式 UI 直接呼叫服務後端, model 就無法得知使用者做了什麼, 對話狀態也會斷裂。MCP Apps 因此把互動事件送回 host, 由 host 決定是否執行 tool call, 送出 prompt 或只接收通知, 讓重要操作留在 agent 的上下文與控制範圍內。

講者將它描述為應用程式的新分發方式。服務提供者可以保留品牌, 領域知識與成熟的使用者體驗, host 不必自行重建每一種專業介面, 使用者則可在同一個助理對話中組合多個服務。MCP Apps 也不限制 UI 的產生方式, 預先定義 UI, 宣告式 UI 與模型即時生成 UI 都能使用相同的互動與傳輸機制。

## 為什麼文字回應不夠

[01:02](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=62s)

傳統 MCP tool call 通常回傳文字。文字適合簡短回答, 但面對漏斗分析, 行程安排, 商品比較或媒體內容時, 容易變成難以閱讀的資訊牆。資料來源的品牌與視覺識別也會消失, 使用者不容易分辨內容來自 Shopify, Booking 或其他服務。

講者提出的替代方式是讓每個服務送出自己的 UI 片段。這些片段不只負責顯示, 還能接受點擊, 表單輸入與後續探索。既有產品累積的 UI/UX 知識不需要在 agent 時代全部捨棄, 而是能轉化為可嵌入對話的互動元件。

## 從 MCP UI 到 MCP Apps

[02:06](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=126s)

Ido Salomon 在 2025 年 5 月發布 MCP UI, 核心問題是如何透過 MCP 傳送 UI, 以及如何建立 UI 與 host 之間的通訊。之後 MCP UI 與 Anthropic, OpenAI 合作, 將概念納入 MCP 標準, 成為第一個官方 extension, 即 MCP Apps。

影片列舉的採用者包含 Claude, ChatGPT, VS Code, Cursor, Microsoft Copilot, GitHub, Postman 與 Goose。更早期的 MCP UI 使用案例則包含 Shopify 與 Hugging Face。除了 host 與服務供應商, 社群也開始建立 SDK, plugin, workshop 與專門協助企業製作 MCP Apps 的服務。

講者強調, 規格仍在演進中。官方工作小組定期開會, 透過公開 issue, pull request 與社群回饋推進互通性。

## 核心機制: 透過 MCP 傳送 UI 資源

[05:14](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=314s)

傳統流程如下:

```text
user prompt
  -> host 呼叫 MCP tool
  -> MCP server 回傳文字
  -> host 顯示文字
```

使用 MCP Apps 後, tool result 可以指向一個 UI resource:

```text
user prompt
  -> host 呼叫 MCP tool
  -> MCP server 回傳 UI resource, 例如 HTML
  -> host 在沙箱中呈現互動式應用程式
  -> 使用者操作 UI
  -> UI 將事件送回 host
  -> host 決定是否呼叫 tool, 傳送 prompt 或取得其他 resource
  -> model 與對話取得後續結果
```

這裡的關鍵不是單純把網頁塞入聊天視窗, 而是由規格定義雙向訊息流程。Host 是安全與控制的邊界, UI 不應任意繞過 host 執行影響對話狀態的操作。

### 為什麼互動要經過 host

影片用收藏歌曲說明這個差異。如果 UI 直接呼叫 Spotify 後端完成收藏, 稍後使用者詢問 Claude 剛才收藏了哪首歌時, Claude 可能完全不知道那次操作。若 UI 把收藏意圖送回 host, host 再決定呼叫 MCP server 的 tool, 操作就能留在對話脈絡中。

因此 host 同時負責:

- 接收 UI 事件.
- 控制 tool call 與後續 model 行為.
- 維持對話上下文的一致性.
- 在沙箱中隔離並呈現第三方 UI.

## PostHog 漏斗分析示範

[06:49](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=409s)

講者在 Claude 中要求分析產品漏斗。傳統 tool result 會提供正確但冗長的文字, 使用者必須自行閱讀並重建整體趨勢。使用 MCP Apps 後, PostHog 可以回傳自己設計的視覺化元件, 讓使用者一眼看出漏斗各步驟的轉換狀況。

這個元件仍保有 PostHog 的產品識別與互動方式。使用者能點選特定漏斗階段繼續追問, 點擊事件再沿著 MCP Apps 的訊息流程回到 host 與 model, 形成互動式探索, 而非靜態圖表。

影片也展示第一方或生成式 UI 的可能性。假設使用者不理解什麼是 funnel, model 可以生成適合當下問題的解說介面, 取代另一段長篇文字。

## 技術架構

[08:54](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=534s)

Server 端將 UI 註冊成 resource, tool response 再指向該 resource。支援 MCP Apps 的 host 取得 resource 後, 透過相應的 UI component 呈現內容, 並提供 callback 處理 UI 與 host 間的訊息。

從實作責任來看:

| 元件 | 主要責任 |
| --- | --- |
| MCP server | 提供 tool, UI resource 與服務端行為 |
| UI resource | 呈現領域介面, 接收使用者互動, 發送標準化訊息 |
| Host | 沙箱化呈現 UI, 管理權限與訊息, 協調 model 和 tool call |
| Model | 理解使用者意圖與互動結果, 規劃下一步 |

使用者點擊 UI 後, 事件可以讓 host 發起其他 tool call, 送出後續訊息或取得額外 resource。這使流程同時具備 UI 的可操作性與 agent 的推理能力。

## 從網站導覽轉向 UI 組合

[10:23](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=623s)

講者認為 MCP Apps 代表的不只是技術改動, 而是 web 使用模式的改變。使用者不必為了完成一項任務開啟許多分頁, 學習不同 dashboard, 再把自己的意圖逐一轉譯成各網站的操作。

在影片的週年紀念日範例中, 個人助理可以在同一段對話內組合:

- Google Calendar 的行程片段.
- Amazon 的商品或禮物元件.
- Booking 的場地與地圖片段.

服務供應商仍掌握自己最擅長的領域介面。例如 Booking 知道如何設計訂房流程, agent 則更了解使用者偏好, 可以只選取接近自然環境的場地及相關地圖。Host 不必生成所有專業 UI, 服務也不會被簡化成沒有識別度的資料庫。

影片將這種關係描述為三方受益:

- 服務供應商保留品牌與產品經驗.
- 使用者看到熟悉而且與任務相關的介面.
- Host 重用領域專家的 UI, 不必重建所有使用者旅程.

## 新的互動控制模型

[12:32](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=752s)

當應用程式被拆成對話中的 UI 片段, 單一服務不再完全擁有使用者旅程。互動可能要先交給 host, 再由 host 與 model 判斷下一步。影片把 UI 訊息放在一條控制權光譜上:

| 訊息類型 | UI 與 host 的關係 | 影片中的意義 |
| --- | --- | --- |
| Notification | UI 保留較多控制權 | UI 或服務已發生事件, 僅通知 host |
| Tool call | UI 指定希望執行的能力 | Host 仍決定是否呼叫 server tool |
| Prompt | UI 交出較多控制權 | 將意圖交給 host 與 model 自行規劃 |

例如購物車數量的局部調整可能直接由服務處理, 再通知 host 狀態已變更。需要受控執行的操作可以要求 tool call。更開放的探索則可送出 prompt, 讓 model 決定後續行動。

這個光譜讓 UI 開發者能依互動性質分配控制權, 也讓 host 保留協調整體任務與安全政策的能力。

## 規格正在發展的方向

[14:56](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=896s)

### 可重用 View

目前每次呈現 app 時可能建立新的 view。對 Autodesk 一類較重的應用程式而言, 重複初始化會拖慢體驗。規格社群正在研究重用既有 view, 再把新資料推送進去, 以減少載入成本並保留介面狀態。

### 讓 Model 操作 View

現有主要方向是使用者操作 UI, UI 再把事件送給 model。另一個待標準化方向是反轉流程, 讓 app 對 model 暴露可用工具, 使 model 能按按鈕, 填表單或操作 view。這會補齊 model 與 UI 的雙向互動迴路。

影片提到 WebMCP 等現有方案, 並表示相關能力仍在公開 pull request 階段, 尚屬持續演進中的工作。

## MCP Apps 與生成式 UI

[16:18](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=978s)

MCP Apps 處理的是 UI 的傳輸, 呈現與互動協議, 並不限定 UI 必須由誰建立。影片將 UI 生成方式分成三類:

| 類型 | UI 的建立與控制方式 | 適用情境 |
| --- | --- | --- |
| 預先定義 UI | 服務供應商提供完整介面, host 視為相對封閉的元件 | 品牌與成熟使用者旅程很重要 |
| 宣告式 UI | App 宣告結構, host 使用自己的 component 呈現 | Host 想維持一致的外觀與操作方式 |
| 完全生成式 UI | Model 依需求即時產生介面 | 任務高度動態, 難以事先定義 |

預先定義 UI 的例子是 Airbnb 自行設計元件後送到 Claude 或 ChatGPT。宣告式 UI 類似以 JSON 描述結構, app 與 host 分享 UI 控制權。完全生成式 UI 則由 model 即時產生。

三種方式都可以使用 MCP Apps。講者以 Claude 的生成式 UI 功能為例, 說明生成結果可以串流進 MCP App, 再由 MCP Apps 補上呈現與互動迴路。社群也在推動與 Google A2UI, WebMCP 等 UI protocol 的互通。

## 一次開發, 在多個 Host 執行

[18:39](https://www.youtube.com/watch?v=o-zkvb0iFDQ&t=1119s)

標準化的直接價值是同一套 app codebase 可以在不同 host 使用。影片以 LibreChat, ChatGPT 與其他支援 MCP Apps 的 client 為例, 將 MCP Apps 定位成應用程式分發機制, 而不只是一項 UI 技術。

開發者有兩種主要切入方式:

1. 建立 server 或 app: 使用官方 MCP Apps repository 與 SDK 建立 application.
2. 建立 host: 使用 MCP UI 的 client SDK, 在自己的產品中加入呈現與通訊能力.

講者建議參與官方 repository, 工作小組與社群 Discord, 透過 issue, pull request 和討論協助規格演進。

## 實務設計原則

以下是根據影片內容整理出的實作檢查點:

- 先判斷回應是否真的需要 UI. 簡短答案仍適合文字, 圖表, 表單和探索流程才適合互動元件.
- 將 UI 註冊成明確的 MCP resource, 由 tool result 指向它.
- 不要讓會影響對話狀態的重要互動繞過 host.
- 依行為選擇 notification, tool call 或 prompt, 不要把所有點擊都視為同一種事件.
- 將第三方 UI 放在沙箱中, 並由 host 執行權限與訊息控制.
- 保留服務的領域優勢, 但只呈現當下任務需要的 UI 片段.
- 若 app 很重或會反覆使用, 留意 view 重建成本與狀態保存.
- 不要把 MCP Apps 與單一 UI 生成技術綁死, 依需求選擇預先定義, 宣告式或生成式 UI.
- 用多個 host 驗證相同 app 的互通性, 避免無意間依賴單一 client 的行為.

## 編輯整理: 對 Agentic Software 的意義

這段是依影片主張整理出的延伸解讀, 不是講者逐字說法。

MCP Apps 將 agent 的作用從「呼叫 API 並轉述結果」推進到「協調多個可見, 可操作的領域介面」。Model 不必取代所有 UI, UI 也不必繞過 model 各自執行。兩者可形成分工:

```text
model
  負責理解意圖, 組合服務, 規劃下一步

domain UI
  負責呈現專業資訊, 提供熟悉且可靠的操作

host
  負責安全邊界, 上下文, 權限與訊息協調
```

這種架構是否能取代大量傳統網站, 仍取決於授權, 支付, 身分, 無障礙, 效能, 跨 host 一致性與使用者信任等問題。影片對未來 web 的描述具有明顯願景性, 應視為方向判斷, 而非已完成驗證的產業結果。

## 來源與可信度限制

本文依據 YouTube 英文自動字幕與影片提供的章節整理, 並非逐字稿。自動字幕可能誤辨人名, 專案名稱與技術詞彙。本文僅在上下文足夠明確時修正, 例如 `MCP UI`, `MCP Apps`, `PostHog`, `WebMCP` 與 `A2UI`。

影片中的採用名單, 使用者規模, 規格狀態及未來時程是講者在錄影當下提出的資訊或主張, 本文未逐項以外部資料查核。尤其尚在 pull request 或工作小組討論中的功能可能隨後變更, 實作時應再查閱最新 MCP Apps 規格與 SDK 文件。
