# 系統設計基礎課程: 從單機架構到 API, 安全與權限控制

> 影片: [System Design Course - APIs, Databases, Caching, CDNs, Load Balancing & Production Infra](https://www.youtube.com/watch?v=C842vFY5kRo)  
> 頻道: freeCodeCamp.org  
> 原始網址: https://www.youtube.com/watch?v=C842vFY5kRo  
> 發布日期: 2026-04-16  
> 片長: 2:05:22  
> Video ID: `C842vFY5kRo`  
> 內容依據: YouTube 英文原語言自動字幕 (`en-orig`)

## 摘要

這門課以逐步演進的方式介紹系統設計. 起點是一台同時承載應用程式, 資料庫與其他元件的 server, 接著依流量與可靠性需求拆分資料層, 進行水平擴展, 加入 load balancer, health checks 與 redundancy. 後半部轉向 API, 比較 REST, GraphQL, gRPC, HTTP, WebSocket, AMQP, TCP 與 UDP, 最後整理 authentication, authorization 與常見 API security controls.

課程最重要的方法不是背誦元件名稱, 而是從需求和失敗模式推導架構:

```text
先建立最小可行系統
  -> 找出容量, 延遲或可靠性瓶頸
  -> 確認失敗會影響哪些元件
  -> 選擇符合互動模式與一致性需求的技術
  -> 加入監控, 安全與維運邊界
```

內容定位是廣泛的基礎導覽. 它提供大量術語, 比較與例子, 但沒有 production metrics, 容量估算, 實作程式碼或完整案例. 部分安全與資料庫敘述也經過教學性簡化, 使用時仍需查閱正式文件.

## 從單機開始理解資料流

[00:03:05](https://www.youtube.com/watch?v=C842vFY5kRo&t=185s)

最初的系統將 web application, database, cache 與其他元件放在同一台 server. 使用者輸入 `app.demo.com` 後, DNS 將 domain name 解析成 IP address, client 再向該位置傳送 HTTP request. Server 可能向 browser 回傳 HTML, CSS 與 JavaScript, 或向 mobile app 回傳 JSON.

這個模型的價值是讓 request flow 保持可見:

```text
Client
  -> DNS resolution
  -> Server IP
  -> HTTP request
  -> Application logic and data access
  -> HTML or JSON response
```

單機架構適合初期或低流量服務, 但 application 與 database 共用資源, 無法獨立擴展, 任一關鍵元件失效也可能讓整個服務中斷. 課程因此先將 web tier 與 data tier 分離, 讓兩者依各自負載演進.

## 資料庫選擇: SQL 與多種 NoSQL 模型

[00:07:12](https://www.youtube.com/watch?v=C842vFY5kRo&t=432s)

### Relational database

PostgreSQL, MySQL, Oracle Database 與 SQLite 等 relational databases 將資料放入具有 columns 與 rows 的 tables, 並透過 SQL 查詢. 主要優勢包括:

- 使用 joins 表達不同資料實體間的關係.
- 使用 transactions 將多個操作視為同一單位.
- 透過 ACID properties 維持 atomicity, consistency, isolation 與 durability.

課程以 bank transfer 說明 transaction. 轉帳中的扣款與入帳應一起成功或一起失敗, 且 concurrent transactions 不應互相破壞狀態.

### Non-relational database

影片將常見 NoSQL database 分成四類:

| 類型 | 例子 | 主要特性 |
| --- | --- | --- |
| Document store | MongoDB | 以 JSON-like documents 保存彈性結構 |
| Wide-column store | Cassandra, Cosmos DB | 適合大量資料與高寫入量 |
| Key-value store | Redis, Memcached | 介面簡單且能提供低延遲存取 |
| Graph database | Neo4j, Amazon Neptune | 將 entities 與 relationships 建模為 graph |

課程給出的初步判斷是:

- 資料結構與關係清楚, 且需要 transaction integrity 時, 優先考慮 SQL.
- Schema 經常改變, 資料半結構化或需要大規模水平擴展時, 可評估 NoSQL.
- 關係本身是查詢核心時, graph database 可能比一般 table 或 document 更自然.
- 極低延遲的 lookup 或 cache-like workload 可考慮 key-value store.

這些是選型起點, 不是互斥規則. 實際系統常同時使用多種資料儲存, 而且 consistency, query pattern, operational maturity 與成本都會影響決定.

## 垂直擴展與水平擴展

[00:13:32](https://www.youtube.com/watch?v=C842vFY5kRo&t=812s)

Vertical scaling, 或 scale up, 是增加單機的 CPU, RAM 或其他資源. 它容易理解和部署, 適合低至中等負載, 但受單機上限限制, 且無法自然消除 single point of failure.

Horizontal scaling, 或 scale out, 是增加多個 service instances 共同承擔流量. 它能提高容量與 fault tolerance, 但同時引入新的問題:

- Client request 應送到哪個 instance?
- 如何確認 instance 是否健康?
- Session 或 state 是否能跨 instance 使用?
- Load balancer 本身是否又成為 single point of failure?

這些問題使 load balancing, health checks 與 state management 成為水平擴展的必要配套.

## Load balancing

[00:16:22](https://www.youtube.com/watch?v=C842vFY5kRo&t=982s)

Load balancer 位於 clients 與 service instances 之間, 將新 request 導向可用的 instance. 課程介紹七種常見策略:

| 策略 | 判斷方式 | 適合情況 |
| --- | --- | --- |
| Round robin | 依序輪流分配 request | Instances 規格與工作量相近 |
| Least connections | 選擇 active connections 最少者 | Connection duration 差異較大 |
| Least response time | 綜合 response time 與 connections | Instance 效能或目前負載不同 |
| IP hash | 依 client IP 決定 instance | 需要某種 session affinity |
| Weighted round robin | 依 instance 權重分配 | Instances 規格不同 |
| Random | 隨機選擇 instance | 架構簡單且 pool 足夠大 |
| Resource-based | 根據 CPU, memory 等即時資源選擇 | 能取得可靠資源 telemetry |

實作形式包括 Nginx, HAProxy 等 software load balancers, F5 等 hardware appliances, 以及 AWS, Azure, Google Cloud 提供的 managed load balancing services.

## Health checks 與 single point of failure

[00:25:08](https://www.youtube.com/watch?v=C842vFY5kRo&t=1508s)

Health check 讓 load balancer 判斷 instance 是否仍可接受流量. 檢查可以從單純的 process 或 port availability, 延伸到能否存取 database, dependency 或關鍵 application path. 若只確認 process 尚在執行, 可能仍把 request 送給實際上無法完成工作的 instance.

[00:28:00](https://www.youtube.com/watch?v=C842vFY5kRo&t=1680s)

Single point of failure 是任何一旦失效就會使整體服務失效的元件. 單一 database 或單一 load balancer 都可能形成這種風險. 課程提出三個處理方向:

1. Redundancy: 部署多個等價元件, 讓其他 instance 能接手流量.
2. Monitoring and health checks: 持續偵測元件狀態, 及早停止 routing.
3. Self-healing: 偵測失效後自動建立 replacement instance.

Redundancy 並不自動等於高可用性. State replication, failover trigger, DNS 或 routing 更新, split-brain protection 與恢復測試仍需另外設計.

## API 是系統間的 contract

[00:31:01](https://www.youtube.com/watch?v=C842vFY5kRo&t=1861s)

API 定義 software components 如何互動, 包含允許的 requests, 預期 responses, 資料格式與錯誤行為. 它隱藏內部 implementation details, 並形成清楚的 service boundaries.

課程比較三種常見 API styles:

| Style | 核心模型 | 常見使用情境 | 主要取捨 |
| --- | --- | --- | --- |
| REST | Resources, HTTP methods 與固定 response shapes | Web 與 mobile APIs | 簡單, HTTP tooling 完整, 但複雜 UI 可能需要多次 request |
| GraphQL | Client 透過 query 指定需要的 fields | 資料需求多變的複雜 UI | 減少 over-fetching 與 round trips, 但 query cost, cache 與 schema governance 更複雜 |
| gRPC | RPC methods, Protocol Buffers 與 HTTP/2 | Internal services, microservices | Payload 緊湊且支援 streaming, 但 browser compatibility 與除錯體驗不同 |

好的 API 應兼顧四個方向:

- Consistency: naming, casing, resource patterns 與 error format 一致.
- Simplicity: 讓主要 use cases 容易理解, 避免暴露不必要的內部複雜度.
- Security: authentication, authorization, input validation 與 rate limiting.
- Performance: caching, pagination, 適當 payload size 與合理 round trips.

API design 應先確認 requirements, scope, performance targets 與 security constraints, 再決定採 top-down, bottom-up 或 contract-first approach. API 上線後仍有 monitoring, maintenance, versioning, deprecation 與 retirement 的 lifecycle.

## Application protocols

[00:47:17](https://www.youtube.com/watch?v=C842vFY5kRo&t=2837s)

Application protocol 的選擇取決於 interaction pattern, latency, throughput, payload, client compatibility, security 與 developer experience.

### HTTP 與 HTTPS

HTTP 定義 request-response pattern, 包含 method, URL, headers, body 與 status code. HTTPS 在 HTTP communication 外加入 TLS, 保護 data in transit 的 confidentiality 與 integrity. 公開服務應以 HTTPS 為預設, 但 TLS 不會取代 authentication, authorization 或 application-level validation.

### WebSocket

HTTP polling 需要 client 反覆詢問是否有新資料, 可能產生空 request, 額外 latency 與 server load. WebSocket 在 handshake 後維持雙向 connection, 讓 server 能主動推送訊息, 適合 chat, live updates 等 real-time communication.

### AMQP

Advanced Message Queuing Protocol 用於 asynchronous messaging. Producer 將 message 發送至 broker 或 queue, consumer 在有 capacity 時處理. 這能 decouple producer 與 consumer, 也為 delivery, routing 與 retry 提供明確機制. 影片提到 direct, fan-out 與 topic-based exchanges.

### gRPC

gRPC 使用 Protocol Buffers 描述 services 與 messages, 並以 HTTP/2 提供高效傳輸與 streaming. 影片將它定位在 service-to-service communication, 特別是能控制雙方 client stack 的內部 microservices.

## Transport layer: TCP 與 UDP

[00:59:10](https://www.youtube.com/watch?v=C842vFY5kRo&t=3550s)

TCP 是 connection-oriented transport, 透過 handshake, acknowledgement, retransmission 與 ordering 提供可靠 byte stream. 這些保證增加 overhead, 但適合 payments, authentication, email 與一般需要完整資料的 APIs.

UDP 不建立相同形式的 connection, 也不保證 delivery 或 ordering, 因此 overhead 較低. Video call, online game 與 live stream 等即時場景, 有時寧可接受少量 packet loss, 也不希望等待過期資料重傳.

這個比較是概念層級. 現代協定可能在 UDP 上自行實作 reliability 與 congestion control, 例如 QUIC. 因此不能直接把 UDP 等同於永遠不可靠, 應檢查實際 application protocol 提供的保證.

## RESTful API design

[01:04:22](https://www.youtube.com/watch?v=C842vFY5kRo&t=3862s)

REST 將 domain concepts 表達為 resources. URL 使用 nouns 而不是 operations:

```http
GET /api/products
GET /api/products/123
GET /api/products/123/reviews
POST /api/orders
PATCH /api/orders/456
DELETE /api/orders/456
```

大量 collection 需要 filtering, sorting 與 pagination, 避免一次傳回完整資料集. Response 應使用符合語意的 status codes, 並保持一致的 error structure. 設計時也要處理 versioning, idempotency, caching 與 backward compatibility.

課程強調 statelessness, 也就是每個 request 應包含 server 處理它所需的資訊. 這不表示後端不能有 database 或 shared state, 而是 application session 不應依賴某一個 server instance 的隱藏 local state.

## GraphQL

[01:19:04](https://www.youtube.com/watch?v=C842vFY5kRo&t=4744s)

GraphQL 通常透過單一 endpoint 接收 query, 由 client 指定 response fields. 主要 operations 包括:

- Query: 讀取資料.
- Mutation: 建立或修改資料.
- Subscription: 接收 real-time updates.

它能讓 client 一次取得 nested related data, 減少 REST 情境中的多次 request, 但也允許 client 建立昂貴或過深的 query. 課程建議保持 schema 小而模組化, 使用 meaningful type names, 為 nested queries 設定 depth limits, 並以 input types 描述 mutations.

GraphQL response 可能同時包含 partial `data` 與 `errors`. Transport-level HTTP status 和 application-level field errors 因此需要分開理解. Caching 也通常無法只依賴 REST 常見的 URL-based HTTP cache.

## Authentication

[01:24:52](https://www.youtube.com/watch?v=C842vFY5kRo&t=5092s)

Authentication 回答 "requester 是誰?". 課程刻意區分 authentication method, token format, authorization framework 與 user experience pattern.

| 方法或概念 | 角色 | 主要限制 |
| --- | --- | --- |
| Basic authentication | 每次 request 傳送 Base64 encoded credentials | Base64 不是 encryption, 必須依賴 HTTPS, 且不適合多數現代公開服務 |
| Digest authentication | 使用 challenge 與 digest, 避免直接傳送 password | 常見 MD5 variants 已過時, 現代使用情境有限 |
| API key | 識別 calling application 或 client | Key 外洩即可被冒用, 需要 rotation, scope, expiration 與 secure storage |
| Session cookie | Server 保存 session state, browser 保存 session identifier | 需要 shared session storage 與 CSRF protection |
| Bearer token | 持有 token 者即可取得相應 access | Token 必須受到傳輸, 儲存與生命週期保護 |
| JWT | 可簽章並攜帶 claims 的 token format | Revocation, stale permissions, key rotation 與 token size 仍需治理 |

影片建議使用 short-lived access token 搭配較長效 refresh token. Access token 用於 API calls, refresh token 用於換取新 access token. 對 browser application, refresh token 通常應放在 secure, HTTP-only cookie, 而不是可被 JavaScript 直接讀取的 local storage.

### OAuth 2, OpenID Connect 與 SSO

OAuth 2 是 delegated authorization framework, 回答 application 可以代表 user 存取哪些 resources. OpenID Connect 在 OAuth 2 flows 上加入 identity layer, 透過 ID token 讓 client 驗證 user identity.

Single sign-on 是一次登入後存取多個相關 services 的使用體驗, 底層可使用 OpenID Connect 或 SAML 等 identity protocols. SAML 常見於 enterprise 與較早期系統, 使用 XML assertions. OpenID Connect 通常使用 JWT-formatted ID token.

## Authorization

[01:45:51](https://www.youtube.com/watch?v=C842vFY5kRo&t=6351s)

Authorization 回答 "已確認身分的 requester 可以做什麼?". 影片比較三種 access-control models:

| Model | 決策依據 | 優點 | 代價 |
| --- | --- | --- | --- |
| RBAC | User 所屬 role | 容易理解與管理 | Role 數量可能膨脹, 難表達細緻 context |
| ABAC | User, resource 與 environment attributes | 彈性高, 可表達細緻 policy | Policy 複雜, 容易出現衝突與難以稽核的條件 |
| ACL | 每個 resource 的 users 與 permissions list | Resource-specific control 清楚 | 大量 users 與 objects 下管理成本高 |

GitHub repository permissions 是 RBAC-like example, department, resource classification 與 location 的組合是 ABAC example, Google Drive document sharing 則接近 ACL.

OAuth 2 access token 與 JWT 是傳遞 identity, claims 或 delegated scopes 的機制, 不是 access-control model 本身. Server 仍需依 RBAC, ABAC, ACL 或其他 policy 判斷 action 是否允許.

## API security controls

[01:57:02](https://www.youtube.com/watch?v=C842vFY5kRo&t=7022s)

影片最後整理七種常見防護:

1. Rate limiting: 依 endpoint, user, IP 或整體 service 限制 request rate.
2. CORS: 控制 browser 是否允許某個 origin 的 frontend JavaScript 讀取 response.
3. Injection prevention: 使用 parameterized queries, input validation 與 ORM safeguards.
4. Firewall or WAF: 在 application 前方過濾已知或可疑流量.
5. VPN or private network: 限制 internal APIs 只能由受控 network 存取.
6. CSRF protection: 對 cookie-authenticated state-changing requests 驗證 CSRF token, origin 或其他 anti-forgery signal.
7. XSS prevention: 對 untrusted content 做 context-aware output encoding, 並搭配 sanitization 與 Content Security Policy.

需要注意 CORS 不是 authentication, 也不是一般性的 CSRF protection. 它主要是 browser-enforced read policy, 非 browser client 不受相同限制. Security controls 必須組合使用, 不能將其中任何一項視為完整防線.

## 建立系統設計的實務檢查表

以下是依影片內容整理的設計順序, 不是講者逐字提供的單一 checklist.

### Requirements

- 核心 use cases, users 與 service boundaries 是什麼?
- 預期 traffic, latency, throughput, availability 與 consistency 是多少?
- 哪些資料和 actions 具有較高 security 或 compliance 風險?

### Data

- 主要 access patterns 與 relationships 是什麼?
- 是否需要 multi-record transactions 或強一致性?
- Schema 會如何演進, 資料量與 write rate 如何成長?

### Scaling and reliability

- 目前最小架構能承受多少負載?
- 哪些元件需要 vertical 或 horizontal scaling?
- 哪裡存在 single point of failure?
- Health checks 是否檢查實際 service readiness?
- Failover, replication 與 recovery 是否定期測試?

### API and communication

- Interaction 是 request-response, real-time, asynchronous messaging 或 internal RPC?
- REST, GraphQL 或 gRPC 的選擇是否符合 client 與 operational constraints?
- 是否定義 pagination, error format, versioning 與 deprecation policy?

### Identity and security

- Authentication, token format, authorization policy 是否清楚分離?
- Credentials, sessions, access tokens 與 refresh tokens 如何儲存和撤銷?
- 是否具備 rate limiting, injection prevention, CSRF 與 XSS controls?
- Network boundary, least privilege, logging 與 incident response 是否明確?

## 來源與限制

- 本筆記依據 YouTube 英文原語言自動字幕整理, 不是逐字稿. 字幕可能誤辨人名, 縮寫與技術名詞, 本文只在上下文足以確認時修正.
- 影片有明確的 15 個 chapters, 時間連結依官方 chapter start 建立.
- 影片標題提到 caching, CDNs 與 production infrastructure, 但這支影片的實際 chapters 主要涵蓋單機架構, 資料庫, 擴展, load balancing, API, authentication, authorization 與 security. 結尾也說明 caching, CDNs 與更完整 production infrastructure 是頻道上的其他 deep dives, 因此本筆記沒有補寫正片未講授的內容.
- 課程是廣泛的基礎導覽, 沒有 capacity estimation, production incident, benchmark, 成本數據或可重現 implementation. 文中的工具選擇應視為起點, 不應取代正式規格, threat modeling 或 workload-specific evaluation.
- 對 NoSQL scalability, JWT statelessness, CORS, HTTP status, database consistency 與 protocol selection 的部分說明經過簡化. 本筆記已標示主要邊界, 但實作時仍應查閱目標技術的官方文件.
