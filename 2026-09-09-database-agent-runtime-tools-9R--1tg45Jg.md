# 資料庫 Agent 的 Build-Time 與 Run-Time 工具: 從任意 SQL 到身分綁定的操作

> - 影片: [Build-Time vs. Run-Time: Why Dev Tools Fail in Production — Averi Kitsch & Prerna Kakkar, Google](https://www.youtube.com/watch?v=9R--1tg45Jg)
> - 頻道: [AI Engineer](https://www.youtube.com/channel/UCLKPca3kwwd-B59HNr-_lvA)
> - 講者: Averi Kitsch, Google Cloud databases 的 MCP Toolbox 技術負責人; Prerna Kakkar, Google 的 Eval Bench 技術負責人與 MCP Toolbox 貢獻者. 身分依影片口述及公開說明.
> - 發布日期: 2026-09-09.
> - 片長: 20:26.
> - Video ID: `9R--1tg45Jg`.
> - 內容依據: YouTube 原始英文自動字幕 `en-orig`, 搭配公開資訊與章節. 無創作者提供字幕.
> - 來源性質: 工具建造團隊的第一手架構與設計分享. 本文為繁體中文重述, 非逐字稿.

## 核心觀點

開發者為了探索資料庫, 可能需要任意 SQL, 建立資源與管理設定等彈性能力. 面向終端使用者的 agent 應用, 則通常需要受限且符合使用者權限的業務操作. 把前者直接放進後者, 會讓模型能決定過多事情.

兩位講者透過 MCP Toolbox 說明如何逐步收窄工具: 先把連線資訊移出模型控制, 再限制資料範圍與 SQL, 最後由應用程式或經驗證的身分 token 提供使用者識別. 模型只負責任務需要的動態參數.

本場最重要的判斷問題是: 哪些輸入可以讓模型推導, 哪些限制必須由可信任的程式與身分系統決定?

## 1. 三種資料庫工具, 三種使用需求

[02:56](https://www.youtube.com/watch?v=9R--1tg45Jg&t=176s), [04:37](https://www.youtube.com/watch?v=9R--1tg45Jg&t=277s)

| 類型 | 能力與例子 | 講者定位 |
| --- | --- | --- |
| Control plane tools | 建立或管理 instance, database 等資源 | 開發者與 DBA 輔助, 重要操作需人監督 |
| NL2SQL tools | 模型依問題產生 raw SQL, 再交給 execute SQL 工具 | 未知查詢的探索與分析 |
| Structured SQL tools | 預先定義 SQL 邏輯, 只接受特定參數 | 面向終端使用者的受控操作 |

NL2SQL 的例子是找出某地區購買特定商品, 在指定期間內退貨的客戶, 再按原始行銷活動分組. 這類探索不一定能事先知道所有查詢形式.

Structured SQL 則適用於已知操作, 例如取消訂單. Agent 不需要重新發明取消訂單的 SQL, 只需在指定介面內提供必要資訊.

講者認為預定義邏輯有助於減少生成錯誤與延遲, 但本場沒有提供延遲數據或成功率對照.

## 2. Build-Time 與 Run-Time 是權限與使用情境的區分

[05:20](https://www.youtube.com/watch?v=9R--1tg45Jg&t=320s)

演講以 build-time 表示開發者輔助情境, 以 run-time 表示終端使用者應用. 這裡並非狹義的編譯時間與程式執行時間.

開發者可能需要彈性且底層的工具, 並由人判斷操作後果. Run-time 應用則應將可執行行為限制在事先設計好的業務範圍, 例如查詢自己的航班或取消自己的訂單.

[06:16](https://www.youtube.com/watch?v=9R--1tg45Jg&t=376s) 起, 講者指出一個失敗例子: Agent 遇到錯誤後, 決定刪除資料表並重新開始, 系統也沒有 guardrails 阻止它. 筆記依口述記錄此例, 未逐幀檢查畫面, 因此不推定具體 SQL, 資料損失範圍或事故環境.

接著預定展示航班 chatbot: 使用者在對話中冒稱另一位講者, 系統仍依已驗證身分處理自己的航班. 但現場因技術問題未能成功播放 demo, 最後改為口述. 不能將這段視為已觀看到的防冒用測試結果.

## 3. Confused deputy: Agent 用自己的權限替不該存取的人取資料

[08:46](https://www.youtube.com/watch?v=9R--1tg45Jg&t=526s), [09:13](https://www.youtube.com/watch?v=9R--1tg45Jg&t=553s)

講者以 confused deputy 描述這種失敗: 使用者或不可信內容, 誘導有權限的 agent 執行原本不應替該使用者執行的動作.

示意情境是一個處理 ticket 的 triage agent. 攻擊者在 ticket 中放入指令, 要求查詢薪資資料並把結果貼回 ticket. Agent 把 ticket 內容當成工作指令, 動用自己的資料庫權限, 導致原本沒有薪資存取權的人取得資料.

這個例子結合了三項能力, 講者稱為 lethal trifecta:

- 讀取私有資料.
- 接觸不可信內容.
- 將資料送出給外部或不具相應權限的接收者.

Ticket 位於內部系統, 不代表其中的文字都應成為可信指令. 這是講者的攻擊模式示例, 沒有聲稱為本場披露的真實薪資外洩事故.

## 4. 分開三種身分, 再分開兩種參數

[11:19](https://www.youtube.com/watch?v=9R--1tg45Jg&t=679s)

| 身分 | 演講中的角色 |
| --- | --- |
| User identity | 使用應用程式的人, 決定其可取得的資料範圍 |
| Application identity | 應用程式的 workload identity, 為整合服務可能具有較廣權限 |
| Agent identity | 執行當次任務的 agent, 資料存取應受終端使用者權限限制 |

Agent 不能因為運行在有廣泛服務權限的應用程式內, 就自動取得相同範圍的任意操作能力.

工具參數也需區分:

| 參數來源 | 例子 | 處理原則 |
| --- | --- | --- |
| 模型動態推導 | 查詢日期等任務輸入 | 當作不可信輸入驗證 |
| 應用程式提供 | 已驗證的 user ID 與存取限制 | 保持在模型控制範圍之外 |

對話中的「我是某個人」不能取代身分驗證. 這正是後續 bound parameters 設計要處理的問題.

## 5. 第一層收斂: 將資料庫連線移出模型控制

[12:33](https://www.youtube.com/watch?v=9R--1tg45Jg&t=753s)

最初的高權限工具讓模型同時接觸 credentials, host, port, 連線細節與 raw SQL. 模型一旦被誘導, 可能洩露連線資訊或將權限用於錯誤目的.

Toolbox 以 source primitive 將連線設定放到預先配置的 YAML, 由 MCP server 在啟動時使用. 這裡的重點是模型不再決定或接觸這些設定. 演講未完整交代 secret 儲存方式, 不能推定所有憑證都應以明文寫入 YAML.

Source 層再加入不同限制:

| 控制 | 講者說明的用途 |
| --- | --- |
| Read-only | 移除寫入工具, 並在資料庫 driver 層限制為唯讀查詢 |
| Allowed datasets | 在支援的資料庫設定可存取資料集, 收窄影響範圍 |
| Output size | 限制一次取出的資料量, 減少資料暴露與系統負擔 |

唯讀仍可能讀出不該取得的資料, 輸出上限也不能取代授權. 這是依上述控制範圍所做的編者說明, 不代表影片提供了完整防外洩方案.

## 6. 第二層收斂: 預定義 SQL 與 typed parameters

[14:55](https://www.youtube.com/watch?v=9R--1tg45Jg&t=895s)

移除連線參數後, 如果模型仍能自由提供 SQL 字串, 就仍有過大的查詢決定權.

Toolbox 的 custom tools 讓開發者在 YAML 定義實際執行的 SQL, 並提供工具名稱, 描述與 typed parameters. 系統使用 prepared statements 與型別驗證, 將參數作為值傳入預先定義的邏輯.

這使工具介面從「執行任何 SQL」轉為「完成特定業務操作」, 例如 lookup flights. 講者將其視為降低 SQL injection 風險與提高可預期性的方式.

Prepared statements 處理的是 SQL 結構與參數值的分離. 若模型仍可任意指定其他人的 user ID, 即使 SQL 沒有注入問題, 仍可能取得不應取得的資料. 因此還需要下一層身分綁定.

## 7. 第三層收斂: 身分參數由可信任端綁定

[17:39](https://www.youtube.com/watch?v=9R--1tg45Jg&t=1059s)

講者以原本接受 user ID 與 date 的 lookup flights 工具說明. 日期可以由模型依需求提供, user ID 則不應讓模型自行選擇.

### Bound parameters

應用程式先驗證使用者, 再將相應參數直接綁定到工具. 模型不需要接觸或生成 user ID, 也不能透過修改工具參數冒用他人.

### Authenticated parameters

工具收到身分 token 後, 先驗證 token, 再擷取其中的 claims, 例如 user ID 或 email, 作為可信任端提供的參數. 演講提及 OpenID 與簽章 JWT, 但未提供完整驗證設定或程式.

兩種方式的共同結果是, 對模型顯示的工具只需接受 date 等業務參數. 身分資料來自驗證流程, 而非對話中的自我宣稱.

以下為編者依演講整理的介面演進, 非實際 API schema:

```text
模型提供連線資訊 + 任意 SQL
  -> 連線由 source 配置, 模型仍提供 SQL
  -> SQL 由 custom tool 固定, 模型提供 user ID 與 date
  -> user ID 由可信任端綁定, 模型只提供 date
```

講者將最終設計描述為 zero trust 架構. 本文將其視為演講的架構定位, 不將一項參數綁定機制等同於完整的安全驗證.

## 8. 工具品質: 讓 Agent 容易選對, 用對與修正

[15:52](https://www.youtube.com/watch?v=9R--1tg45Jg&t=952s)

| 原則 | 設計方向 |
| --- | --- |
| 以成果為單位 | 工具對應實際要完成的動作, 避免直接照搬細碎 REST APIs, 減少往返 |
| 描述提供使用指引 | 說清楚何時與如何使用, 不重複 schema 已有的參數資訊 |
| 分離讀與寫 | 方便對不同操作設定核准流程, 並讓 Agent 更清楚副作用 |
| 提供可採取行動的錯誤 | 說明可修正或可重試的情況, 不只回傳泛用 HTTP 錯誤 |
| 使用簡單輸入 | 優先平坦結構與簡單型別, 減少模型建構複雜參數的負擔 |

講者建議讀取操作可自動核准, 寫入操作交給人確認. 這是依操作類型設計流程的建議, 不表示唯讀工具就不需要資料授權.

## 編者整理: 上線前可檢查的邊界

1. 工具是否仍讓模型決定 host, credentials 或任意 SQL?
2. 終端使用者真正需要哪些固定操作, 能否用受限工具表達?
3. 讀取限制是否在實際執行層落實, 而非只寫在提示中?
4. user ID 等身分值是否由驗證流程提供, 並排除模型覆寫?
5. 預定義 SQL 是否仍需額外的資料範圍與使用者授權檢查?
6. 工具副作用是否明確, 錯誤是否足以讓 Agent 決定修正或停止?
7. 是否分別測試參數注入, 冒用身分與不可信內容誘導等不同失敗模式?

以上依演講設計整理, 不是講者交付的完整測試規格.

## 來源與限制

本筆記依完整英文自動字幕去重與重述, 時間戳使用影片公開章節. 姓名依 metadata 校正, JWT 等明顯誤辨依上下文修正. 未逐幀核對 SQL, YAML 或投影片, 因此不提供看似可執行但未經核實的設定範例.

航班 demo 在現場未成功播放, 相關行為僅為講者口述. 刪除資料表與薪資 ticket 的案例也沒有完整 trace 或測試資料供本次核對. 不能據此聲稱已獨立驗證產品防護效果.

開場介紹 MCP Toolbox 與 Google managed MCP, 包括連線管理, authentication 與 observability 等能力. 產品功能與使用規模皆屬影片當時陳述, 本文未驗證現況. 結尾提及 Eval Bench, 但未展示完整 eval 配置與結果.

本場的價值是呈現工具權限如何逐層縮小. 不同資料庫支援的唯讀與資料集限制可能不同, 實際部署仍須確認對應實作. 本文不將 read-only, prepared statements, 人工確認或身分綁定中的任一項視為完整安全保證.
