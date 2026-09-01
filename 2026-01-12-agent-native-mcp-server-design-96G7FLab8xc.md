# Agent-Native MCP Server 設計: 從 REST Wrapper 到可用產品

- 影片: [Your MCP Server is Bad (and you should feel bad) - Jeremiah Lowin, Prefect](https://www.youtube.com/watch?v=96G7FLab8xc)
- 頻道: AI Engineer
- 講者: Jeremiah Lowin, Prefect founder and CEO、FastMCP creator
- 發布日期: 2026-01-12
- 片長: 54:32
- Video ID: `96G7FLab8xc`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

Jeremiah Lowin 認為, 許多 MCP servers 只是把既有 REST endpoints 自動轉成 tools, 卻沒有針對 agent 的限制重新設計介面。這種 server 可能在技術上符合 protocol, 但 tool 太多、參數太複雜、需要多次呼叫才能完成一個簡單 workflow, 最終增加 token、latency 與錯誤選擇。

他將 agent-native product design 歸納為三個差異面向:

- Discovery: 人類通常只讀一次 API 文件, agent 每次連線可能都要取得全部 tools 與 descriptions。
- Iteration: 人類可用程式快速串接 endpoints, agent 每次 tool call 都可能增加一次昂貴且隨機的 reasoning round trip。
- Context: Agent 只有有限 context window, 每個 schema、description 與 tool result 都在競爭同一份預算。

因此 MCP server 不應被視為 transport wrapper, 而應視為 agent 使用的產品介面。影片提出五項核心原則:

1. Outcomes over operations。
2. Flatten arguments。
3. Instructions are context。
4. Respect the token budget。
5. Curate ruthlessly。

這些原則來自 FastMCP 作者維護大量 servers 的第一手觀察, 具有強烈實務價值。但影片沒有提供完整 benchmark dataset 或受控實驗, `50 tools` 等門檻是講者 heuristic, 不是 MCP protocol 的固定限制。

## MCP Server 是 Agent 的 User Interface

[03:23](https://www.youtube.com/watch?v=96G7FLab8xc&t=203s)

講者反對「人能使用 API, AI 為何不能」這個假設。人類通常不直接操作 raw API, 而是使用 website、SDK、client 或 mobile app。這些介面會隱藏不必要的 operations, 引導使用者完成 outcome。

MCP server 對 agent 扮演相似角色:

```text
backend API
  -> agent-oriented curation
  -> MCP tools, resources and instructions
  -> agent
```

Protocol compliance 只代表雙方能交換資料, 不代表 agent 能穩定選到正確工具、生成合法參數或以合理成本完成任務。Server author 仍需做 product design。

## 三個 Agent-Specific Design Constraints

[05:16](https://www.youtube.com/watch?v=96G7FLab8xc&t=316s)

### Discovery 很昂貴

人類開發者通常讀一次 API docs, 找到少數 endpoints 後將選擇固定在 code 中。Agent client 則常在 handshake 時取得 server 的 tools、schemas 與 descriptions。這些 discovery metadata 可能在每個 session 占用 context。

### Iteration 很慢

人類寫好的程式能依序快速呼叫多個 endpoints。若把相同 orchestration 交給 agent, 每一步都需要模型判斷下一個 tool 與參數, 產生更多 latency、tokens 與 stochastic failure points。

Agent 適合處理無法先寫成 deterministic algorithm 的部分。若流程已完全知道, 例如先找 user、再找 order、最後查 status, 應優先用一般 code 在 tool 內完成。

### Context 很有限

Agent 的 working information 受 context window 限制。工具數量、argument schema、instructions、examples、errors 與 tool results 都在競爭相同預算。

講者用一個直覺比喻說明: Agent 或許能在 haystack 中找到 needle, 但可能要檢視很多 hay 才能判斷。較好的 server 應先縮小 haystack。

## 最重要的動詞: Curate

[09:16](https://www.youtube.com/watch?v=96G7FLab8xc&t=556s)

Agent-facing interface 的核心不是暴露所有可能資訊, 而是從完整 domain 中挑出當前 agent stories 需要的最小有效 surface。

```text
complete API surface
  -> select valuable workflows
  -> combine deterministic operations
  -> simplify inputs
  -> document choices and recovery
  -> expose compact agent surface
```

這種 curation 會犧牲部分底層組合自由, 換取較低的 discovery cost、較少 iteration 與更可靠的 tool selection。若使用者確實需要 open-ended composition, 才讓 agent 承擔更多 orchestration。

## 原則一: Outcomes Over Operations

[15:45](https://www.youtube.com/watch?v=96G7FLab8xc&t=945s)

影片以 order status 為例。REST API 可能提供:

- Lookup user。
- List orders。
- Filter orders。
- Get order status。

若逐項轉成 MCP tools, agent 至少要完成三次以上選擇與 round trips, 還要自己推導呼叫順序及 argument format。

Agent-facing tool 應從 workflow top-down 設計, 例如 `track_latest_order(email)`, 並在 tool 內用 deterministic code 串接既有 API calls。

```text
bad agent surface
  lookup_user -> list_orders -> filter -> get_status

curated surface
  track_latest_order(email)
    -> deterministic API orchestration inside tool
```

這不是刪除 backend operations, 而是把已知 orchestration 移出模型 reasoning loop。講者稱每個 tool 應對應一個 agent story, 並用 agent 能理解的 outcome 命名。

Agent 不應成為預設 glue layer。只有當流程本身無法預先編碼、需要語意判斷或探索時, LLM orchestration 才值得其成本與不確定性。

## 原則二: Flatten Arguments

[18:40](https://www.youtube.com/watch?v=96G7FLab8xc&t=1120s)

複雜 dictionary、深層 object 或相互依賴的 arguments 會增加模型生成錯誤。講者建議優先使用 top-level primitives, 並在選項固定時使用 literals 或 enums。

```text
避免:
config: dict

偏好:
email: string
include_cancelled: boolean
format: "basic" | "detailed"
```

Tightly coupled arguments 也應避免。例如第一個 argument 選擇 file type, 第二個 argument 的合法值卻依 file type 改變。模型必須同時維持額外 constraint, 容易產生表面合法但組合無效的 input。

若複雜結構不可避免, schema、tool description 與 examples 必須維持單一事實來源。不要在 system prompt、sub-agent definition 與 server docstring 分別保存不同版本的 argument rules。

講者也提到 client implementations 可能錯誤序列化 structured arguments。FastMCP 曾因實際 client compatibility 而加入額外轉換。這說明理論上清楚的 schema 不代表所有 client 都會正確傳送。

## 原則三: Instructions Are Context

[21:54](https://www.youtube.com/watch?v=96G7FLab8xc&t=1314s)

未提供說明時, agent 仍會猜測工具用途。猜錯的 attempts 又會進入 conversation history, 擴大後續 context 與偏差。

講者建議同時考慮:

- Server-level instructions。
- Tool name 與 description。
- Argument descriptions。
- Examples。
- Tool result 與 error messages。
- MCP annotations。

### Examples 近似 Contracts

[22:40](https://www.youtube.com/watch?v=96G7FLab8xc&t=1360s)

Examples 能明確展示合法使用方式, 但模型可能模仿其中未明說的偶然特徵。講者的案例是 tags argument: example 只放兩個 tags, 即使 instruction 要求至少十個, agent 仍反覆產生兩個。

因此 example 不只是 syntax sample, 也可能暗示數量、長度、順序與風格。建立 examples 時應刻意涵蓋 variation, 並以 eval 檢查模型是否過度複製。

### Errors 也是 Prompts

[25:05](https://www.youtube.com/watch?v=96G7FLab8xc&t=1505s)

Agent 會把 tool error 當成下一輪 context。只有 exception name 或數字 code, 通常不足以支持修正。可回復的 error 應說明:

- 哪個輸入無效。
- 合法範圍或 format。
- 是否值得 retry。
- 下一次呼叫應修改什麼。
- 是否需要換用其他 tool。

講者將 error-specific guidance 視為一種 progressive disclosure: 不必在初始 description 放入所有 edge cases, 可在 agent 實際遇到某類錯誤時提供相應修復資訊。

這個做法有代價。若刻意依賴 first-call failure 才揭露正常操作方式, 會增加 latency 與 tokens。Error guidance 應用來改善真正的例外恢復, 不應取代清楚的 happy-path schema。

### Read-Only Annotation

MCP tool annotations 可以提示 tool 是否 read-only。支援此 annotation 的 clients 可依副作用風險採取不同 permission policy。影片提到 ChatGPT developer mode 會利用這類訊號, 但 client support 並不一致。

Annotation 是 hint, 不是 security boundary。Server 仍需自行執行 authentication、authorization 與 input validation。這是依協定用途延伸的編者提醒。

## 原則四: Respect the Token Budget

[27:27](https://www.youtube.com/watch?v=96G7FLab8xc&t=1647s)

MCP client 常在 handshake 取得全部 tool definitions, 而不是自動 progressive disclosure。若 server 暴露數百個 tools, names、schemas 與 descriptions 可能在 task 開始前就占掉大量 context。

影片舉例, 一家公司希望暴露 800 endpoints。若 agent 只有 200,000-token context, 平均到每個 tool 的 name、schema 與 documentation 已非常有限, 而且沒有替 system instructions、user task 與 results 保留空間。

這只是用來展示資源競爭的算術例子, 不是每個 client 都有相同 context window 或 handshake behavior。

### Progressive Disclosure 的限制

可以建立 meta-tools, 讓 agent 先搜尋或讀取少量 tool descriptions, 再取得完整 schema。但這會引入新的設計問題:

- Agent 必須先理解如何使用 discovery tool。
- Tool discovery 本身成為額外 round trip。
- Client 可能 cache 初始 tool list 或忽略 protocol notifications。
- 不同 clients 對動態 tools 的支援可能不一致。

講者特別批評部分 clients 未完整遵守當時 spec behavior。這類相容性敘述具有版本時效性, 實作時應針對目標 clients 重新測試。

## 原則五: Curate Ruthlessly

[35:32](https://www.youtube.com/watch?v=96G7FLab8xc&t=2132s)

講者把超過 50 tools 視為可能出現 performance degradation 的警訊, 隨後修正為「agent 能看到的總 tools」, 而不是單一 server 的 tools。若 agent 同時連接兩個各有 50 tools 的 servers, 它面對的是 100 個 choices。

`50` 是講者經驗門檻, 並非普遍上限。Tool schema 長度、model、client routing、task distribution 與是否使用 deferred loading 都會影響結果。

影片提到 Fiverr MCP server 曾從約 188 tools 精簡到五個, 用來說明 product journey 可能先讓功能運作, 再持續刪減到核心 outcomes。講者也承認自己常在探索階段建立過多 tools, 後續才進行 curation。

可採取的拆分方式包括:

- 將 admin 與 general-user tools 分離。
- 依 domain 或 workflow namespace。
- 拆成多個 servers。
- 只依 authenticated user's permissions 回傳可用 tools。
- 使用 semantic routing 或 tool search。
- 只保留最常見且高價值的 agent stories。

## 不要直接將 REST API 當成成品

[38:20](https://www.youtube.com/watch?v=96G7FLab8xc&t=2300s)

FastMCP 提供 OpenAPI 或 REST conversion capability, 而講者公開建議不要把自動轉換結果直接當成 production MCP server。他承認這是 FastMCP 很受歡迎的功能, 自己也對大量低品質 wrappers 負有部分責任。

自動轉換仍適合 bootstrap:

1. 先挑少數關鍵 endpoints。
2. 驗證 client 能連線並正確呼叫。
3. 確認 authentication、transport 與基本 result flow。
4. 找出真正需要的 agent stories。
5. 合併 deterministic operations。
6. 簡化 arguments、instructions 與 results。
7. 移除不必要 tools 後再進入 production。

因此問題不是 generator 本身, 而是將 scaffold 誤認為設計完成的產品。

## 是否控制 Client 會改變設計空間

[43:55](https://www.youtube.com/watch?v=96G7FLab8xc&t=2635s)

Q&A 中, 講者反覆用「是否控制 client」作為決策邊界。

若團隊控制 client, 可以:

- 把低頻 workflow 放入 client-side files 或 skills, 避免常駐 server context。
- 自訂 progressive disclosure 與 semantic routing。
- 確保 server instructions 被讀取。
- 配合特定 permission、cache 與 notification behavior。
- 針對已知 model 與 context budget 最佳化。

若 server 面向未知 external clients, 就必須假設較差的相容性。關鍵 instructions 應盡量放在 tool name、schema 與 docstring 等普遍可見的位置, 並以多個 clients 做 contract tests。

Server-level `instructions` field 可提供高層說明, 但影片指出並非所有 clients 都可靠採用, 而且不宜塞入過長內容。

## Async Tasks 與 Elicitation

[46:31](https://www.youtube.com/watch?v=96G7FLab8xc&t=2791s)

講者討論當時名為 SEP-1686 的 asynchronous background tasks proposal。其設計讓 client 選擇非同步執行並承擔 polling 與取得 result 的責任。由於 tool discovery 與 invocation interface 基本不變, 前述 outcomes、arguments、instructions 與 token principles 仍適用。

Elicitation 則允許 tool execution 中途向 client 要求更多 structured input。適用情境包括:

- 取得缺少的資料。
- 對 destructive action 要求 confirmation。
- 將部分 tightly coupled choice 延後到已有更多資訊時。

限制是 client 不一定支援 elicitation, 而 background or automated client 也未必有人可回答。Server 若直接依賴它, 可能在不支援的 client 上失敗。

Protocol proposal 名稱、狀態與 client support 可能在影片發布後改變。本節只記錄講者當時的說明, 不是目前 MCP 規格確認。

## 何時不需要 MCP

[51:15](https://www.youtube.com/watch?v=96G7FLab8xc&t=3075s)

觀眾詢問, 若自己完全控制 agent loop, 是否仍需要建立 MCP server。講者回答, 這種情況不必為 MCP 而 MCP。直接使用 local tools 或 framework-native functions 是合理做法。

他預期成熟的 MCP infrastructure 未來可能帶來較好的 observability 與 tool-call debugging, 但這是未來判斷。影片當下的實務原則是:

```text
fully controlled client + private tools
  -> use the simplest suitable interface

multiple clients / reusable integration / protocol boundary
  -> MCP becomes more valuable
```

影片的五項 tool-design 原則仍適用於普通 Python tools, 並不依賴 MCP。

## Code Mode 的取捨

[52:27](https://www.youtube.com/watch?v=96G7FLab8xc&t=3147s)

Code mode 讓 LLM 產生程式碼, 再由程式串接多個 MCP tools。這能把部分逐步 tool orchestration 移出對話式 round trips, 也可能降低 context 壓力。

代價是引入 code execution、sandboxing、permissions 與新的 failure modes。它能繞過部分 interface 限制, 但不能消除 outcome design、schema quality、security 或 verification 的需求。

講者表示 FastMCP 團隊已實驗相關 extension, 並考慮放入 experimental CLI capability。這是影片當時的 roadmap, 不是目前功能保證。

## MCP Server Review Checklist

### Outcomes

- 每個 tool 是否對應一個清楚的 agent story?
- 已知的 API orchestration 是否移入 deterministic code?
- Agent 是否只負責真正需要語意判斷的部分?
- Tool name 是否描述 outcome, 而不是 backend operation?

### Arguments

- 是否能用 top-level primitives 取代 nested configuration?
- 固定選項是否使用 enum 或 literal?
- 是否存在依其他 argument 才能判斷合法性的 tightly coupled fields?
- Schema 是否只有一個可維護的事實來源?

### Instructions 與 Recovery

- Server、tool 與 arguments 是否都有必要說明?
- Examples 是否意外限制數量、格式或風格?
- Error 是否說明如何修復, 而不只是回傳 exception code?
- Read-only 與 side-effect annotations 是否正確?
- Client 不採用 server instructions 時, 關鍵資訊是否仍可取得?

### Context Budget

- Handshake 時 agent 會收到多少 tools、schemas 與 tokens?
- Agent 同時連接的其他 servers 是否也計入總預算?
- Large tool results 是否可 filter、paginate 或 summarize?
- Tool discovery 是否支援 target clients, 還是只在理論上符合 spec?
- 是否能依 role、permission 或 workflow 隱藏無關 tools?

### Evaluation

- 是否測量 tool selection accuracy、argument validity 與 task success?
- 是否記錄平均 tool calls、retry count、tokens 與 latency?
- 是否測試多 server 情境下的 tool confusion?
- 是否針對不同 clients 執行 compatibility tests?
- 刪除某個 tool 後, 核心 agent stories 是否仍能完成?

## 核心結論

一個 technically valid MCP server 不一定是 agent 可用的產品。REST API 追求完整、可組合的 atomic operations, agent interface 則必須同時考慮 discovery cost、reasoning round trips 與有限 context。

最具一般性的設計轉換是:

```text
API-first thinking
  "有哪些 endpoints 可以暴露?"

             ↓

agent-product thinking
  "Agent 要完成哪些 outcomes, 最少需要看見什麼?"
```

自動 conversion 可以快速驗證 transport, 但 production surface 需要重新設計。將 deterministic orchestration 放進 tool、簡化 arguments、把 instructions 和 errors 當成 context、控制 agent 看見的 tools, 才能建立穩定且節省資源的介面。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。影片沒有 YouTube chapters, 本文段落與時間戳依演講內容轉折編排。字幕將 MCP、FastMCP、Anthropic、Claude 等名稱多次辨識為近似詞, 本筆記只在上下文明確時修正。

講者是 FastMCP creator 與 Prefect CEO, 能直接觀察大量使用該 framework 建立的 servers, 並坦承 FastMCP 自動 REST conversion 助長部分問題。這提供高度第一手的 maintainer 經驗, 同時也包含 FastMCP 與 Prefect 相關產品立場。

影片沒有提供 tool-count degradation benchmark、完整 client compatibility matrix、task success comparison、token measurements 或 REST wrapper 與 curated server 的受控實驗。FastMCP downloads、GitHub server token size、50-tool threshold、Fiverr tools reduction 與 client behavior 均為講者陳述, 本筆記未獨立驗證。

MCP spec、SEPs、client support、tool discovery、elicitation、async tasks 與 code mode 仍快速演進。實作前應以目標 client 與當前官方 protocol 文件重新驗證, 不應把本影片中的狀態當成永久規格。
