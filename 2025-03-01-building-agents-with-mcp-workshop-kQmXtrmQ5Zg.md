# 使用 Model Context Protocol 建立 Agents

> 影片: [Building Agents with Model Context Protocol - Full Workshop with Mahesh Murag of Anthropic](https://www.youtube.com/watch?v=kQmXtrmQ5Zg)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=kQmXtrmQ5Zg  
> 發布日期: 2025-03-01  
> 片長: 1:44:11  
> Video ID: `kQmXtrmQ5Zg`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Anthropic Applied AI 團隊的 Mahesh Murag 介紹 Model Context Protocol, 簡稱 MCP, 的設計動機, 核心 primitive, server 建置方式, agent framework 整合與早期 roadmap。

MCP 的核心目標是把 AI application 與外部 tools, data sources 和 reusable interactions 之間的整合標準化。它不取代模型, agent loop 或一般 API, 而是提供共同連接層, 將原本每個 client 對每個 service 都要客製的 N x M integration problem, 簡化為 client 與 server 分別實作一次 MCP。

影片將 MCP 能力分成三類:

- Tools: 由模型決定何時呼叫.
- Resources: 由 application 決定如何取得與呈現資料.
- Prompts: 由使用者主動選擇的預設互動模板.

工作坊也展示 MCP 如何支援 multi-agent research workflow, sampling, composability, remote servers, OAuth, registry discovery 和 self-evolving agents。不過影片發布於 MCP 發展初期, 多項 roadmap 和 transport 細節可能已改變。

## MCP 的基本動機: 模型只和 Context 一樣好

演講從一句原則開始: 模型的品質受提供給它的 context 限制。[00:00:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=0s)

早期 AI applications 主要依靠使用者複製貼上或手動輸入 context。之後 applications 開始直接連接企業資料, local files 和 SaaS tools, 使模型能產生更個人化的結果並採取行動。

問題是每個 application 和 data source 都建立自己的 integration, 導致:

- 不同團隊重複實作 authentication, tools 和 prompt logic.
- 同一公司對同一 database 建立多套 wrappers.
- Client 與 server 的 interface 不一致.
- Access control, logging 和 maintenance 分散.
- 新 AI application 很難重用既有 integrations.

MCP 希望提供開放協定, 讓 AI apps 和 agents 以一致方式連接 tools 與 data sources。

## 從 API 與 LSP 得到的啟發

講者用兩個先例解釋 MCP 的角色。

### API

API 標準化前端, 後端, services 和 databases 之間的請求與回應。前端不必知道 server 內部實作, 只需遵循 contract。

### Language Server Protocol

LSP 標準化 IDE 與各程式語言工具的互動。一個 Go language server 建立一次後, 所有支援 LSP 的 IDE 都能重用 diagnostics, completion 和 navigation 能力。

MCP 想把同一種 one-to-many 重用帶到 AI 生態。AI client 實作 MCP 後, 可以連接不同 MCP servers。Tool 或 data provider 建立 server 後, 也能被不同 clients 使用。

## MCP 解決 N x M Integration Problem

在沒有共同協定時, N 個 applications 連接 M 個 systems, 理論上可能需要 N x M 種整合。MCP 在中間提供共同 layer。[00:04:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=240s)

```text
AI clients
  ├── Claude applications
  ├── IDEs
  └── Agents
        ↕ MCP
MCP servers
  ├── Databases
  ├── Salesforce or other SaaS
  ├── Git and local filesystem
  └── Internal enterprise services
```

對不同角色的價值如下:

| 角色 | MCP 帶來的價值 |
| --- | --- |
| Application developer | Client 相容後可接入多個 servers, 不必逐一建立 integration |
| Tool or API provider | Server 建立一次後可被多個 AI applications 使用 |
| End user | Agent 能取得更多個人化 context 並執行外部操作 |
| Enterprise | 基礎設施團隊可集中維護 data access, 產品團隊專注 AI UX 和 agent logic |

企業可讓一個團隊維護 vector database 的 MCP server, 統一 chunking, prompting 和 access interface。其他團隊只需使用 server, 不再各自直接連接 database。這與 microservices 分離服務 ownership 的概念相似。

## MCP Architecture

MCP client 與 MCP server 有不同責任。[00:09:39](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=579s)

### Client

Client 屬於 AI application 或 agent runtime, 主要負責:

- Discover server capabilities.
- 將 tool descriptions 提供給模型.
- 呼叫 tools.
- 取得 resources.
- 將 prompts 提供給使用者.
- 決定資料如何進入模型 context 或 UI.

### Server

Server 包裝外部 system, 並公開:

- Tools.
- Resources.
- Prompts.
- 與目標系統互動所需的 business logic.
- Authentication, retries, data transformation 和 logging 等實作.

講者的設計傾向是, server 應較接近 end system 並掌握相關邏輯。Client 應能在第一次見到 server 時, 只依 protocol 與 capability metadata 正確操作。

## 三個核心 Primitive

### Tools: Model Controlled

Tool 由 server 公開, 模型根據 tool name, description 和 schema 決定何時呼叫。[00:11:30](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=690s)

Tools 可用於:

- 讀取 database records.
- 搜尋 web 或 documents.
- 寫入 SaaS applications.
- 更新 database.
- 操作 local filesystem.
- 執行其他具有副作用的 actions.

Tool 特別適合呼叫時機不固定的能力。例如, model 可能已經有答案, 也可能需要查 vector database, 或先向使用者取得更多資訊。

若每次任務都必須固定呼叫某項服務, application 可以用 deterministic workflow 直接呼叫, 不必將它交給模型選擇。

### Resources: Application Controlled

Resource 是 server 公開給 application 的資料。它可以是 text, image, JSON, file 或動態產生的內容。[00:13:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=780s)

Application 可以:

- 在 UI 中列出可選 resources.
- 將 resource 顯示為 attachment.
- 自動附加與任務相關的 resource.
- 根據規則或另一次 model call 決定是否載入.
- 訂閱 resource changes 並接收 notifications.

Resources 與 tools 分開, 是為了讓 application 不必把所有 context 選擇都交由模型控制。Application 可以建立更豐富且可預測的 UX。

### Prompts: User Controlled

Prompt 是由 server 提供的 reusable interaction template, 由使用者選擇何時啟動。[00:14:30](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=870s)

演講以 IDE slash command 為例。使用者輸入類似 `/pr` 並提供 pull request ID, client 取得 server 定義的完整 prompt, 再送給模型執行摘要。

Prompts 適合封裝:

- Document Q&A 格式.
- PR review 或 summarization workflow.
- 特定組織的輸出規則.
- 使用 server 的建議步驟.
- 重複且由使用者明確觸發的任務.

## 為何不把所有能力都做成 Tools

理論上 tools 也能回傳 files, prompts 或其他資料, 但 MCP 想區分三種控制權。[00:16:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=960s)

| Primitive | 主要控制者 | 核心問題 |
| --- | --- | --- |
| Tool | Model | 何時應採取這個操作? |
| Resource | Application | 哪些資料應顯示或放入 context? |
| Prompt | User | 我現在要啟動哪個預設 workflow? |

這個分離讓 server 不只替模型提供函式, 也能與 application UI 和使用者建立標準化互動。

Resources 和 prompts 也不必是 static。Server 可以根據 user, application 或 task context 動態產生內容。Resource notification 還能讓 server 在資料更新時通知 client。

## MCP 不取代 Agent Framework

MCP 與 LangGraph 等 agent frameworks 是互補關係。Framework 負責 agent loop, memory, context management, orchestration 和 decision flow。MCP 負責以標準方式提供 tools, resources 和 prompts。[00:20:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=1200s)

```text
Agent framework
  ├── Model selection
  ├── Loop and state
  ├── Memory and context compression
  ├── Multi-agent orchestration
  └── MCP client adapter
          ↕
      MCP servers
```

既有 framework 只要加入 MCP adapter, 就能接入 servers, 不必重寫整個 agent。

MCP 可能取代 framework 中部分 tool integration 和 context plumbing, 但不負責完整 autonomous behavior。

## GitHub Triage Demo

工作坊展示 Claude Desktop 作為 MCP client, 連接 GitHub MCP server。[00:22:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=1320s)

使用者提供 repository URL, 要求列出 issues 並建議優先處理項目。模型自行判斷應呼叫 `list issues` tool, 取得資料後完成 triage。

Demo 的重點不是 GitHub 功能本身, 而是 client 並未為這一個 repository 建立專用 integration。只要 server 公開適當 tools, model 就能根據自然語言目標選擇操作。

## 建立 MCP Server 的基本流程

講者表示, tools 通常是理解 server 最容易的起點, 之後再加入 resources 與 prompts。[00:24:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=1440s)

可將建置流程整理為:

1. 選擇要公開的 end system 或 API.
2. 定義最小且清楚的 tool capabilities.
3. 為每個 tool 撰寫 name, description 和 input schema.
4. 實作 validation, authentication, retries 和 errors.
5. 視需要加入 application-controlled resources.
6. 為常見使用者流程加入 prompts.
7. 使用 inspector 連接並檢查 calls, logs 和 responses.
8. 以真實 agent tasks 建立 evals.
9. Package, version 和發布 server.

影片提到, 講者曾使用 Claude 在約 45 分鐘內建立多個早期 servers。簡單 API wrapper 確實可能由模型快速產生, 但需要 logging, transformation, security 和 complex behavior 的 server 仍需工程設計。

## MCP Agent Research Demo

工作坊使用 LastMile AI 的開源 `mcp-agent` framework, 建立量子運算對 cybersecurity 影響的研究 agent。[00:26:25](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=1585s)

系統大約以 80 行 Python 定義三個 subagents:

### Research agent

負責 web research, 使用 Brave Search, fetch 和 filesystem MCP servers。它搜尋 authoritative sources, 讀取內容並把研究資料保存至 files。

### Fact-checker agent

驗證 research agent 產生的資料, 同樣可使用 search, fetch 和 filesystem。

### Report-writer agent

讀取 research 與 fact-checking artifacts, 綜合成指定格式的 final report。它只需要 filesystem 和 fetch, 不必重新進行廣泛搜尋。

外層 orchestrator 先建立 plan, 再將 steps 分配給 agents。MCP 使 builder 能以 declarative 方式指定每個 agent 可用 servers, 把注意力放在 task, loop 和 orchestration, 而不是重新建立 web search 或 filesystem integration。

## Proprietary Data 與 Private Deployment

MCP 是開放協定, server 可以在企業 VPC, internal network 或員工 local machine 上執行。使用 MCP 不代表資料必須公開或離開組織。[00:34:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=2040s)

企業仍需自行設計:

- Network boundaries.
- Authentication and authorization.
- User and service identity.
- Audit logging.
- Data masking.
- Tool-level read and write permissions.
- Secret storage and token rotation.

MCP 統一 interface, 但不自動消除 data governance 責任。

## Tools 規模與 Progressive Discovery

講者表示, 當時 Claude 在數十至數百個 tools 的範圍通常仍能良好選擇, 但數量持續增加後, tool descriptions 會占用大量 context, 並提高選擇難度。[00:44:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=2640s)

可能的解法包括:

- Tool-search tool: 使用 RAG, fuzzy search 或 keywords 搜尋可用 tools.
- Hierarchical tools: 先選 Finance, read 或 write 等類別, 再揭露子工具.
- Task-specific exposure: 只將目前任務相關 tools 放入 context.
- Registry discovery: 執行時再搜尋並安裝需要的 server.

因此不是協定存在固定 tool 數上限, 而是 context 和 model selection quality 形成實務限制。

## Server Logic 應放在哪裡

針對 retries, authentication 和 data transformation, 講者傾向由 server 負責, 因為 server 最了解 end application。[00:48:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=2880s)

這個原則的理由是:

- Client 第一次連接時可能完全不認識 server.
- Client 不應知道每個 service 的 retry semantics.
- Server 最清楚 rate limits, errors 和 logging requirements.
- Tool provider 可以集中更新 business logic.

但 agent-specific orchestration 是否也應放在 server, 沒有單一答案。Server provider 可能只想公開 API, 讓 agent framework 決定 loop 和 reasoning。

## Sampling: Server 向 Client 借用模型

Sampling 讓 MCP server 向 client 要求 LLM completion, 不必自行持有 model provider credentials 或部署模型。[00:53:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=3180s)

Server 可以提供:

- System and task prompts.
- Model preferences.
- Temperature.
- Maximum tokens.
- 所需能力或模型大小偏好.

Client 保有最終控制權:

- 選擇實際模型.
- 拒絕可疑 requests.
- 限制成本和 request 次數.
- 管理 privacy policy.
- 決定哪些資料可提供給模型.

Sampling 的價值在於, client 與 server 即使彼此陌生, server 仍能使用 client 已配置的 intelligence, 而不必每個 server 都自行整合 LLM。

## Composability: 同一元件可同時是 Client 與 Server

MCP 中的 client 和 server 是 logical roles, 不必是不同 physical services。Agent 可以對上一層扮演 server, 同時作為 client 呼叫其他 servers。[00:58:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=3480s)

```text
User-facing client
  → Research agent, MCP server and client
      ├── Web search server
      ├── Fetch server
      └── Filesystem server
```

這讓系統可以建立 hierarchical agents。每個節點專注特定任務, 收集下游資料後驗證, 壓縮並傳回上游。

複雜性也會增加:

- Errors 可能跨層累積.
- Observability 變困難.
- User input 可能要跨多層往返.
- Rate limits 和 cost control 需要明確 ownership.
- Server 內部是否又呼叫其他 servers, client 未必知道.

MCP 提供 communication interface, 但不規定 primary election, distributed control flow 或完整 observability。這些仍是 system builder 的責任。

## Verification, Debugging 與 Evals

MCP Inspector 用於連接 server, 查看 capabilities, logs 和 interactions, 並協助除錯。[01:08:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=4080s)

Protocol 標準化不代表 server 自動可靠。團隊仍需評估:

- Model 是否在正確時機選擇 tool.
- Input arguments 是否有效.
- Tool response 是否被 agent 正確解讀.
- Tool description 改變後是否造成 regression.
- Write actions 是否符合 permissions.
- Multi-server workflow 是否能處理 partial failure.

版本更新可在相同 eval suite 上比較 server 1.0 與 1.1。MCP 改善測試和替換的 ergonomics, 但不降低建立 robust evals 的必要性。

## Governance 與 Tool Annotations

影片討論用 metadata 區分 read 與 write tools, 或提供 limit 等 tool annotations, 讓 client 更容易判斷風險和控制行為。[01:10:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=4200s)

即使有 annotations, server 仍應執行真正 authorization。不能只依 tool description 或 prompt 宣稱 read-only, 後端必須阻止不允許的 operations。

Enterprise governance 還可以透過:

- Approved-server allowlist.
- Private registry.
- Scoped identities.
- Environment isolation.
- Human approval for sensitive tools.
- Audit trail.
- Version pinning.

## Remote Servers 與 OAuth

影片後段展示早期 remote server 和 OAuth 2.0 流程。[01:13:15](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=4395s)

Demo 使用 Inspector 連接 remote Slack MCP server。Server 協調 OAuth handshake, user 在 browser 授權, server 保存目標 service token, client 則使用 session token 與 server 互動。

Remote servers 的產品意義是:

- 使用者不必在本機安裝和 build server.
- Provider 可以集中部署, 更新與監控.
- Agent 與 server 可以運行在不同系統.
- Server 像 web service 一樣透過 URL 被發現與使用.

這段影片使用 SSE 描述 remote transport。MCP transport 和 authorization 規格在 2025 年後快速演進, 因此不可將影片中的 OAuth handshake, token ownership 或 SSE 建議直接當成現行實作指南。

## MCP 與 REST API 的關係

講者明確表示 REST APIs 不會消失。MCP 可以建立在既有 APIs 上, 為 LLM interaction 加入 data transformation, context curation 和 richer protocol capabilities。[01:17:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=4620s)

| REST API | MCP Server |
| --- | --- |
| 通用 application-to-service contract | AI client-to-context and capability contract |
| 常提供較原始, stateless data | 可為模型整理資料和操作說明 |
| 不假設 LLM 在 loop 中 | 明確支援 tools, resources, prompts 和 sampling |
| 適合一般 software clients | 適合 AI applications 與 agents |

很多 MCP servers 本質上會包裝 REST APIs, 並增加 authentication, validation, formatting 和 model-facing semantics。

## Registry: 發現與信任 MCP Servers

早期 MCP ecosystem 快速增長後, servers 分散在 repository, npm, PyPI, Docker 和不同社群目錄中。使用者難以判斷 server 安裝方式, transport, 作者和可信度。[01:18:30](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=4710s)

影片描述中的 registry 目標包括:

- Unified metadata API.
- Package or remote URL location.
- Transport information.
- Publisher identity and verification.
- Official provider status.
- Version history and change log.
- 對 npm, PyPI, Docker 和其他 ecosystems 的索引.

企業可以像使用 Artifactory 一樣維護 private registry, 只允許 approved servers。Existing IDE 或 marketplace 也能透過 registry API 顯示 MCP servers。

影片中的 registry 還在開發階段, 不是對目前 registry implementation 的準確說明。

## Self-Evolving Agents

Registry 不只改善人工安裝, 也讓 agent 在 runtime 發現新能力。[01:26:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=5160s)

講者以 Grafana debugging 為例:

1. 使用者要求 agent 檢查 Grafana logs 並修復 bug.
2. Agent 原本沒有 Grafana tools.
3. Agent 搜尋 registry, 找到官方或 verified Grafana server.
4. 經過 policy 和 permission 檢查後安裝或連接.
5. Agent 使用新 tools 查 logs 並處理問題.

這種 agent 不必在建立時預先裝入所有 capabilities, 而可以按需擴充。

安全前提是 registry trust, allowlists, verification, scoped permissions 和 human approval。功能上能選擇正確 tool, 不等於可以信任任意 server code 或資料存取。

## Well-Known Discovery 與 Computer Use

除了 registry 的 bottom-up search, 影片還提出 well-known endpoint。網站可以在標準 URL 公開 MCP metadata, 讓 agent 已知要操作某個 domain 時, 直接發現官方 server。[01:31:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=5460s)

講者想像 Shopify 公開 well-known MCP JSON, 描述 endpoint, capabilities 和 OAuth 方法。

MCP 與 computer use 可以互補:

- 有結構化 MCP capability 時, 優先呼叫可靠 API.
- 沒有 API 的 long tail actions, 再使用 computer use 操作 UI.

這讓 agent 同時具有精確的 programmatic access 和處理未知 UI 的 fallback。

## 當時的 Roadmap

影片最後列出多項 2025 年初仍在探索的方向。[01:35:00](https://www.youtube.com/watch?v=kQmXtrmQ5Zg&t=5700s)

### Stateful vs Stateless Connections

長期連線適合 server-to-client notifications 和 sampling, 短期 HTTP request 則適合單次 tool invocation。團隊當時考慮依 capability 分離兩種模式。

### Streaming

讓 server 能把多個 data chunks 持續送回 client, 適合長時間執行和大型輸出。

### Namespacing

多個 servers 可能公開同名 tools。暫時做法是在 tool name 前加 server name, 但協定需要更正式的 namespacing 和 logical groups。

### Proactive Server Behavior

Server 可能因外部 event 主動通知 client, 詢問使用者或開始 interaction。Resource notifications 已提供部分能力, 更完整 elicitation 尚待設計。

這些項目只反映影片當時 roadmap, 不代表現行規格仍採同一名稱或設計。

## 編輯整理: MCP 系統設計檢查表

以下清單根據 workshop 內容整理, 不是講者逐字提供的規格要求。

### Capability design

- 操作應由 model, application 還是 user 控制?
- 是否真的需要 tool, 或 deterministic call 更適合?
- Resource 是否應 static, dynamic 或 subscribable?
- Prompt 是否代表可重複且由使用者觸發的 workflow?

### Server quality

- Tool names, descriptions 和 schemas 是否清楚?
- Errors 是否能讓 agent 修正?
- Server 是否負責 retries, rate limits 和 data transformation?
- Read 和 write capabilities 是否清楚分離?
- 是否提供足夠 logs 與 metadata 便於除錯?

### Security

- Server publisher 是否可信?
- OAuth 或 service identity 是否採最小權限?
- Client 是否對敏感 tool 呼叫要求人類核准?
- Server 是否在後端執行 authorization?
- 是否有 private registry, allowlist 和 version pinning?

### Agent integration

- Agent framework 如何管理 context 和 memory?
- Tools 數量是否需要 progressive discovery?
- Subagents 如何驗證和壓縮下游結果?
- Sampling requests 的 cost, privacy 和 model choice 由誰控制?
- Multi-server failures 如何隔離?

### Lifecycle

- 是否有 inspector 和 automated evals?
- Server 升級是否執行 regression tests?
- Tool description 和 behavior 變更是否記錄版本?
- Deprecated capabilities 如何通知 clients?

## 重點回顧

- MCP 是 AI clients 與外部 systems 之間的標準連接層, 用來降低 N x M integration cost.
- Tools, resources 和 prompts 分別對應 model, application 和 user control.
- MCP 不取代 agent framework, 它標準化 framework 取得 context 和 capabilities 的方式.
- Server 應集中維護與 end system 有關的 authentication, validation 和 business logic.
- Sampling 讓 server 借用 client 的模型能力, client 仍控制成本, privacy 和 model selection.
- Client-server roles 可組合, 使 agents 能形成 hierarchical systems.
- Registry 和 well-known discovery 讓 agent 按需發現 capabilities, 也帶來新的 trust 問題.
- Protocol interoperability 不等於 security, observability 或 reliability 已自動解決.
- 這支影片屬於 MCP 早期歷史資料, 實作前必須查閱現行規格.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 字幕多次誤辨 MCP, Claude, Anthropic, LangGraph, LastMile AI, SSE, OAuth, PyPI 和 Grafana 等技術名稱, 本筆記只在上下文足以確認時修正.
- 影片發布於 2025-03-01, 當時 MCP 仍快速演進. Remote transport, OAuth, registry, tool annotations, sampling, namespacing 和 server-initiated behavior 等內容可能已過時.
- 影片中的 adoption counts, server 數量與 roadmap 時程只代表錄影當時的講者陳述, 本筆記未將它們視為現況.
- Demo 使用的 `mcp-agent` framework, Claude Desktop 和 Inspector 行為可能已改版.
- 本筆記旨在保存設計理念和早期架構脈絡. 若要實作 production MCP server, 應另行查閱最新版官方 specification, SDK 文件和 security guidance.
