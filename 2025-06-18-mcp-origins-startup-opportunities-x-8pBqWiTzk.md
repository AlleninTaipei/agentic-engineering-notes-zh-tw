# MCP 的起源、設計取捨與創業機會

> [!info] 來源
> - 影片: [MCP: Origins and Requests For Startups — Theodora Chu, Model Context Protocol PM, Anthropic](https://www.youtube.com/watch?v=x-8pBqWiTzk)
> - 頻道: AI Engineer
> - 發布日期: 2025-06-18
> - 片長: 17:45
> - Video ID: `x-8pBqWiTzk`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

MCP 的出發點不是再造一套普通的 API 呼叫方式, 而是建立開放標準, 讓模型能發現、取得並操作外部世界。真正有價值的 MCP server 也不應只是逐一包裝既有 endpoint, 而要同時為終端使用者、client 開發者與模型三種使用者設計工具介面。

## MCP 要解決的原始問題

MCP 共同創作者 David 與 Justin 注意到一個反覆出現的工作模式: 使用者必須不斷把 Slack 訊息、Sentry error log 等外部資料複製到模型的 context window ([01:44](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=104s))。

他們提出的問題不只是「如何放入更多 context」, 而是:

> 如何讓模型離開封閉的對話框, 主動取得外部資訊並對外部系統採取行動?

影片把這項能力稱為 model agency。Context 是模型知道什麼, agency 則進一步包含模型能選擇與執行什麼動作。

## 為什麼需要開放且標準化的協定

若每個模型產品都是封閉生態系, integration 開發者往往要先經過商務合作, 再與每個 client 個別協調介面, 最後重複實作連接。這種 N 對 M 的整合成本會限制生態系規模。

MCP 選擇開源標準的理由是:

- Server 開發者可以面向共同介面建置能力。
- Client 與模型提供者可採用同一套連接方式。
- 社群能直接指出協定問題、提交 proposal 或 pull request。
- 工具能力不必完全依賴雙邊商務合作才能進入產品。

標準並不會因為被宣布為標準就自然成功。講者強調, 它必須先對 builders 真正有用。

## 從內部 Hack Week 到產業採用

MCP 團隊先在 Anthropic 內部 Hack Week 推出協定。工程師開始建立 server, 自動化自己與其他團隊的工作流程。之後 MCP 於 2024 年 11 月開源。

最初外界的反應包括:

- MCP 是什麼?
- 為什麼還需要新的 protocol?
- 模型原本不是就能 tool calling 嗎?
- 為什麼一定要 open source?

真正的第二個轉折是 Cursor 採用 MCP, 接著 VS Code、Sourcegraph 等 coding tools 跟進。到影片錄製時, 講者也提到 Google、Microsoft、OpenAI 等公司已採用 MCP。

這段歷程說明, 協定的價值需要透過可實作的 use case 與多方 client 支援被證明, 而不是單靠規格文件。

## Agent 的核心是讓模型選擇行動

影片對 agent 的理解是: 系統依賴模型的 intelligence 來選擇下一步行動 ([07:22](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=442s))。開發者不會預先知道每次輸出與行動, 而是設計任務、工具與邊界, 讓模型在其中做決策。

這個前提影響 MCP 的技術取捨。協定不能只服務固定的 request-response 工具呼叫, 還要考慮 agent 之間與 server/client 之間更動態的互動。

## 兩項重要設計取捨

### Streamable HTTP 與雙向互動

影片提到 MCP 當時將主要 transport 從 SSE 調整為 Streamable HTTP。講者認為更強的雙向互動能力符合 agent 未來需要互相溝通的方向。

這是 2025 年 6 月時的專案狀態陳述。Transport 規格之後可能已有變更, 實作時應查閱目前 MCP specification。

### 優先降低 server 端複雜度

團隊當時的假設是, 未來 server 數量會遠多於 client。基於這項假設, 協定傾向讓 server 容易建立, 必要時把更多複雜度放到 client。

這是刻意的生態系取捨, 但講者也承認預測可能錯誤。是否合理要看實際 server/client 數量、重用程度與 client 實作者的維護成本。

## 2025 年中時的專案更新

影片快速列出當時六個月內的進展 ([09:46](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=586s)):

- 支援 remote MCP server。
- 修正最初 OAuth 設計中的問題, 更新至當時的 draft spec。
- 將 Streamable HTTP 作為主要 transport。
- 更新 SDK 的 developer experience。
- 改善 MCP Inspector, 讓 server 更容易除錯。

講者特別說明, OAuth 修正來自社群成員對 identity provider 實務的貢獻。這也呼應 MCP 以公開協作方式演進的目標。

上述項目是影片當時的狀態快照, 不是目前版本保證。

## 當時正在規劃的方向

### Elicitation

Elicitation 讓 server 在資訊不足時向終端使用者提出澄清問題 ([11:01](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=661s))。

影片以訂機票為例:

```text
使用者: 幫我訂去 Atlanta 最好的航班。
Server: 「最好」是指最便宜, 還是最快?
使用者回答後, 再把選擇送回 server。
```

這使 server 不必假設含糊偏好, 也讓工具執行過程能保留人在迴路中。

### Registry API

Registry API 的方向是讓模型發現事先沒有明確提供的 MCP server。這會進一步提高 model agency, 但也同步帶來信任、版本、權限與 server 品質辨識問題。

### Open examples 與治理

團隊希望建立公開範例, 讓官方與社群共同沉澱 server 設計模式。另一項方向是建立下一階段治理機制, 確保 MCP 長期保持開放。

### Agent 可能同時是 server 與 client

影片提出一種簡化視角: 一個 agent 可以是「作為 client 的 server」, 也可以反過來。Agent 透過 MCP 接收請求, 同時又呼叫其他 agent、server 或 client, 形成互相協作的網路。

## 應該在 MCP 生態系建立什麼

講者把創業與建置機會分成三類 ([13:20](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=800s))。她給出的主觀權重是:

| 方向 | 講者權重 | 核心需求 |
| --- | ---: | --- |
| 更多、更高品質且跨垂直領域的 server | 80% | 補齊生態系能力與品質 |
| 簡化 server 建置的工具 | 10% | Hosting、testing、eval、deployment |
| 自動產生 MCP server | 10% | 讓高能力模型即時建立所需 integration |

這些比例是講者用來表達優先順序的判斷, 不是市場規模預測。

## 高品質 server 不是一對一包裝 API

講者認為, 把每個 REST endpoint 直接一對一暴露成 tool, 通常不是正確的 MCP server 設計。Server 實際上有三種使用者:

1. 終端使用者: 想完成實際工作的人。
2. Client developer: 把 server 接進產品的人。
3. Model: 必須理解工具目的、參數與使用時機的決策者。

因此設計順序應該是:

```text
終端使用者有哪些工作情境?
        ↓
他們會如何向模型描述需求?
        ↓
模型需要哪些語意清楚的工具才能完成需求?
        ↓
再映射到底層 API 與資料操作
```

API 通常反映後端資源結構, tool 則應反映使用者意圖。兩者未必是一對一。

## 值得補足的垂直領域

影片指出當時許多 MCP server 集中在 developer tools, 生態系需要更多非工程領域能力, 例如:

- Sales
- Finance
- Legal
- Education

垂直 server 的價值不只在連接資料, 還包括把領域工作流程、權限與正確操作邊界轉化為模型可使用的工具。

## Server 工具鏈的機會

若 server 數量真的遠多於 client, 建置與維護 server 的工具會形成重要基礎層。影片列舉的方向包括:

- Hosting
- Testing
- Evaluation
- Deployment
- Debugging
- 企業內部 MCP 管理

這些工具既服務公開 server, 也服務企業內部把 MCP 當作團隊介面的情境。

## 自動產生 MCP server

較長期的構想是, 當模型足夠擅長寫程式並理解外部系統, 它可以在需要時即時建立自己的 MCP server。這是對模型 intelligence 與 agency 持續提升的押注。

要讓這個想法安全落地, 還需要處理產生程式的測試、權限最小化、憑證注入、供應鏈風險、生命週期與稽核。影片將它定位為較早期、偏 moonshot 的方向。

## 安全、可觀測性與稽核

模型能接觸的真實系統與資料越多, 安全與隱私風險就越高。講者認為 AI security、observability、auditing 等工具具有廣泛機會 ([16:42](https://www.youtube.com/watch?v=x-8pBqWiTzk&t=1002s))。

這些需求不只屬於 MCP, 而是所有能對外部世界採取行動的 AI application 都必須處理的基礎問題。

## 實作檢查表

- Server 解決的是使用者工作, 還是只把 endpoint 換一個名稱?
- Tool 名稱與描述是否足以讓模型判斷何時使用?
- 參數是否符合任務語意, 而非直接洩漏後端資料模型?
- 是否同時測試終端使用者、client 與模型三種體驗?
- 資訊不足時, 是否能澄清而不是自行猜測?
- 是否限制資料範圍、操作權限與高風險 action?
- 是否有 Inspector、logs、traces 與可重現測試?
- 是否針對 prompt injection、惡意 server 與敏感資料外洩進行評估?
- Remote server 的 authentication 與 authorization 是否遵循目前規格?
- Server 版本、來源與治理是否能被 client 驗證?

## 時效性與限制

本片發布於 2025 年 6 月, 是 MCP 快速演進早期的狀態快照。影片提到的 draft spec、OAuth、transport、elicitation、registry、SDK、Inspector 與治理規劃可能已經更新。本文保留其設計理由與歷史脈絡, 不把當時 roadmap 視為目前產品文件。

本筆記以英文原始自動字幕與 YouTube 章節整理。部分公司、工具名稱與技術詞可能有字幕辨識誤差。涉及目前 MCP 規格或實作細節時, 應再查閱最新版官方 specification 與 SDK 文件。

## 結語

MCP 的長期價值取決於它是否能成為模型與外部世界之間真正可用的公共介面。協定本身只是起點, 生態系還需要符合使用者意圖的高品質 server、讓 builders 容易測試與部署的工具鏈, 以及能約束模型 agency 的安全與稽核能力。
