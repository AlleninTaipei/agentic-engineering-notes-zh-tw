# Claude Code 入門課程: 安裝、Goals、Skills、GitHub、MCP 與部署

> 來源: [Claude Code Full Course – Autonomous Goals, MCP, and VS Code Setup](https://www.youtube.com/watch?v=7l6bXLAKyEI)  
> 頻道: freeCodeCamp.org  
> 講者: Eric  
> 發布日期: 2026-08-05  
> 影片長度: 1 小時 21 分 37 秒  
> Video ID: `7l6bXLAKyEI`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 課程總覽

這是一門面向初學者的 Claude Code 操作課程. 講者從本機安裝與 VS Code 開始, 示範如何管理 session、選擇權限模式、使用自訂 `/goal` 工作流、安裝 skills、理解專案檔案、管理 context、使用 GitHub 版本控制, 最後透過 CLI 或 MCP 連接外部服務並部署應用程式.

完整學習路徑如下:

```text
安裝 Claude Code
  -> 在 VS Code terminal 啟動
  -> 理解 session 與權限模式
  -> Plan mode 釐清需求
  -> Goal / skill 執行可重複工作流
  -> 理解檔案與 context
  -> 用 Git / GitHub 保留版本
  -> 用 CLI / MCP 連接外部服務
  -> 部署並驗證應用程式
```

## 1. 安裝 Claude Code

[00:02:33](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=153s)

影片建議從 Claude Code 官方文件的 Quickstart 取得與作業系統相符的 native install 指令. 示範環境是 macOS, 流程為:

1. 開啟官方文件.
2. 選擇 native install.
3. 複製對應作業系統的安裝指令.
4. 在 terminal 執行.

安裝命令可能隨版本和作業系統變更. 實際操作時應以當下的官方文件為準, 不應從影片畫面抄錄舊指令.

## 2. 為什麼使用 VS Code

[00:03:24](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=204s)

影片比較三種使用環境:

| 環境 | 優點 | 限制 |
| --- | --- | --- |
| Desktop app | 容易入門 | 可見性與自訂能力較少 |
| 純 terminal | 功能完整 | 初學者較難瀏覽檔案與變更 |
| IDE + terminal | 同時查看 terminal、檔案和修改結果 | 需要先熟悉 IDE 基本操作 |

講者選擇 VS Code, 因為它能在同一個視窗中顯示 file explorer、編輯器和 terminal. 初學者可以清楚看到 Claude Code 建立或修改了哪些檔案.

### 最基本的專案概念

先建立一個專案資料夾, 再用 VS Code 開啟. Claude Code 從該目錄啟動後, 便能在授權範圍內讀寫其中的檔案.

影片也示範了建立、儲存和刪除文字檔, 用來說明 VS Code file explorer 中的變更會直接反映到本機檔案系統.

## 3. VS Code 外觀設定

[00:06:51](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=411s)

這一小節介紹縮放、開啟 Explorer、建立資料夾, 以及從設定選單選擇 color theme. 講者使用 Solarized Light, 但這只是個人偏好, 不影響 Claude Code 功能.

## 4. 啟動與管理 Claude Code session

[00:07:39](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=459s)

在 VS Code 開啟整合 terminal 後, 輸入 `claude` 啟動 Claude Code. 影片示範要求代理在目前資料夾建立範例檔案, 用來確認代理能存取工作目錄.

常見 session 操作:

| 操作 | 影片中的做法 |
| --- | --- |
| 啟動 | 在專案 terminal 執行 `claude` |
| 中止目前 session | 使用 `Ctrl+C`, 或關閉 terminal |
| 清理 terminal 畫面 | 執行 `clear` |
| 恢復舊對話 | 使用 `/resume` 選擇先前 session |

`/resume` 適合意外關閉 terminal 或需要回到舊工作時使用. 恢復前應確認選到正確專案與對話, 避免把舊脈絡帶入不相關工作.

## 5. 權限模式

[00:11:51](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=711s)

影片透過切換介面介紹數種權限模式. 名稱和精確行為可能隨版本變化, 但核心差異是代理可以採取哪些動作, 以及何時需要人類批准.

| 模式 | 影片描述的行為 | 適合情境 |
| --- | --- | --- |
| Plan mode | 允許探索和規劃, 不直接編輯或執行 | 複雜重構、code review、新專案規劃 |
| Accept Edits | 可接受檔案修改, 執行命令可能仍需批准 | 希望減少逐次核准編輯時 |
| Auto mode | 自動接受部分編輯與命令, 由額外安全判斷決定是否詢問 | 已理解風險且仍希望保留安全檢查時 |
| Bypass Permissions | 影片描述為不受限制地執行 | 僅限隔離、可丟棄且無敏感資料的環境 |

### 安全原則

- 一般專案不應預設使用 bypass.
- 代理若能執行 shell commands, 就可能修改、刪除或外傳資料.
- 連接 GitHub、部署平台或其他帳號後, 風險不再侷限於本機檔案.
- 高權限模式應搭配 sandbox、乾淨的測試帳號、版本控制和人工審查.

## 6. Plan mode 與自訂 `/goal` 工作流

[00:15:03](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=903s)

影片以製作 tier-list 網站為例. 講者先提供參考網站、截圖與要求, 在 Plan mode 中讓代理研究功能並提出問題, 包括:

- 使用哪一種技術棧.
- 是否保留 email gate.
- 要使用哪些內容.
- 只求功能可用, 還是要求接近 pixel-perfect.
- 圖片資產是否應先下載到本機.

代理把答案整理成 plan, 包含技術決策、design tokens、data model 與 file structure. 使用者仍可要求修改 plan, 例如先把圖片資產保存到本機, 再開始實作.

### `/goal` 的概念

影片中的 `/goal` 是已安裝的 skill 或自訂工作流, 不是本筆記所保證的標準內建命令. 它的設計概念是:

1. 接收明確的目標與完成條件.
2. 讓代理產出結果.
3. 由 evaluator 對照原始條件檢查結果.
4. 若未達標, 把回饋送回代理修正.
5. 重複迴圈, 直到完成或達到停止條件.

影片以它自動建立網站、啟動本機 dev server, 再於瀏覽器檢查拖放和 reset 等功能.

### 使用 autonomous loop 的前提

自主迴圈的品質取決於驗收條件是否可觀察. `做得漂亮` 很難自動判斷, `主要畫面與參考圖差異低於指定門檻, 且拖放與 reset 可運作` 則較具體.

## 7. Skills: 可重複使用的工作方法

[00:24:24](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=1464s)

影片把 skill 解釋為提供給 AI 的 workflow、guide 或 SOP. Skill 的價值在於把成功做法保存下來, 讓代理可以重複執行.

### Fix-ticket workflow 範例

講者描述了一條自動化 bug-fix pipeline:

1. 從 Jira、Linear 或 GitHub 讀取 ticket.
2. 用 Playwright 在瀏覽器重現問題.
3. 使用代理研究原因.
4. 實作修正.
5. 由其他代理 review.
6. 再以瀏覽器驗證.
7. Commit 並 push 到 GitHub.
8. 部署到 hosting platform.
9. 將 ticket 移到 QA, 交給人類確認.

這個例子呈現了四個階段: understand、fix、ship、handoff. Master skill 可以呼叫其他 subskills 與工具, 但最終仍保留人工 QA.

### 安裝與觸發

影片示範透過 plugin discovery 安裝 front-end design skill, 選擇安裝 scope, reload plugins, 再用兩種方式觸發:

- 在自然語言 prompt 中明確要求使用該 skill.
- 直接輸入對應的 slash command.

安裝 scope 可能是全域個人使用、專案內個人使用, 或隨專案分享給協作者. 選擇專案 scope 時, 相關 skill 檔案可能會進入 repository, 因此提交前應先檢查內容與授權.

影片另示範安裝 front-end design 與 React best-practice 類型的 skill. 第三方 skill 本質上是會影響代理行為的指令, 使用前應閱讀內容, 不要只依熱門排行直接信任.

## 8. 專案檔案與目錄

[00:34:48](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=2088s)

影片以 Next.js/React 專案解釋常見結構:

| 項目 | 用途 |
| --- | --- |
| `*.md` | Markdown 文件與專案說明 |
| `package.json` | Dependencies 與可執行 scripts |
| lock file | 固定已解析的 dependency versions |
| `public/` | 圖片、圖示等 public assets |
| `node_modules/` | 安裝後的 dependencies, 通常不提交 Git |
| `components/` | 可重複使用的 UI 元件 |
| `app/` | 頁面、routes 與應用組合 |
| `.gitignore` | 指定 Git 不追蹤的檔案 |
| `.env` | 常含 secrets 或環境設定, 通常不可提交 |

### Agent-specific 與共用設定

影片的示範專案同時有 `.claude/` 與 `.agents/` 類型的目錄:

- `.claude/`: Claude Code 專用設定或 skills.
- `.agents/`: 供多種 agent framework 共用的內容.
- `CLAUDE.md`: Claude Code 使用的專案指令.
- `AGENTS.md`: 可供多種 coding agent 使用的專案規則.

有箭頭的資料夾可能是 symbolic link, 代表它指向另一個實際來源. 修改前應確認真正的 source of truth, 避免同時編輯連結與來源造成混亂.

## 9. Context window 與 token 使用量

[00:44:36](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=2676s)

Context 包含系統指令、skills、memory、訊息、工具結果與代理正在處理的其他內容. 影片使用 status line 和 `/context` 查看目前 token 使用量及分類.

講者指出, 對話越長不一定越好. Context 持續膨脹後, 模型可能更難找出真正重要的資訊, 回答品質也可能下降.

### Context 管理方式

- `/context`: 查看使用量與來源.
- `/compact`: 摘要既有對話以釋放空間.
- `/clear`: 清除目前 conversation context, 重新開始.
- 新 session: 在新 context 中執行不同或較大的下一項工作.

影片建議在使用率明顯升高時考慮 compact 或新 session. 但 `/compact` 是有損摘要, 可能遺失關鍵細節. 重要決策應先寫入專案文件、issue 或其他可追溯狀態, 不應只依賴 conversation history.

## 10. 常用 slash commands

[00:50:00](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=3000s)

影片介紹的命令包括:

| 命令 | 用途 |
| --- | --- |
| `/clear` | 清除目前對話 context |
| `/compact` | 摘要對話並降低 context 佔用 |
| `/context` | 查看 context 使用狀況 |
| `/resume` | 恢復舊 session |
| `/model` | 選擇可用模型 |
| `/plugins` | 搜尋或管理 plugins |
| `/rewind` | 回到較早的 conversation/code checkpoint |
| `/mcp` | 管理或查看 MCP 相關功能 |

確切命令、名稱和介面會隨產品版本或已安裝 plugins 改變. Slash command 也可能是使用者自訂 skill, 不能只憑 `/名稱` 判定它一定是內建功能.

## 11. Git 與 GitHub 版本控制

[00:53:18](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=3198s)

Conversation rewind 適合短期修正, Git 則提供跨 session、跨日期和多人協作的完整歷史.

核心概念:

- Repository: 一個受版本控制的專案.
- Commit: 某一時間點的變更紀錄.
- Commit message: 說明這次變更的目的.
- Branch: 從共同歷史分出的工作線.
- Remote: GitHub 上的遠端 repository.
- `.gitignore`: 排除 dependencies、build output、secrets 等不應提交的內容.

影片讓 Claude Code 建立 private GitHub repository 並完成 initial commit, 接著刪除 eval 目錄, 再產生第二個 commit. GitHub 的 diff 中, 綠色代表新增, 紅色代表刪除.

### 提交前檢查

- `.env`、API keys 和其他 secrets 不可提交.
- `node_modules/` 與可重建的 build output 通常應忽略.
- 確認 repository 是 public 還是 private.
- 不要只相信代理說「沒有敏感資訊」, 應自行查看 staged diff.
- Revert、reset 和 force push 可能改變歷史或遺失工作, 操作前應確認目標 commit.

## 12. MCP 與 CLI

[01:02:44](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=3764s)

影片用兩種方式讓 Claude Code 操作第三方服務:

### CLI

CLI 是在 terminal 執行的 command-line interface. AI 可以產生命令並呼叫已安裝、已登入的 CLI.

優點:

- 通常較省 context tokens.
- 適合已有成熟 CLI 的服務.
- 本機除錯路徑直接.

風險:

- Shell command 的能力可能很廣.
- 權限常繼承目前登入使用者.
- 錯誤命令可能影響本機或遠端資源.

### MCP

Model Context Protocol 提供標準方式, 讓 AI client 呼叫 MCP server 暴露的工具與資料.

影片將它比喻成 AI 的 USB port. 相較於一般 shell, MCP 能提供較明確的 tool schemas、authentication boundaries 和 auditability, 適合團隊或需要限制能力範圍的環境. 代價是工具定義會占用 context, 可能使用更多 tokens.

### 如何選擇

| 需求 | 較適合的選項 |
| --- | --- |
| 已有成熟 CLI、重視 token 效率 | CLI |
| 需要結構化工具、權限邊界與團隊治理 | MCP |
| 只有少量本機操作 | CLI 可能較簡單 |
| 多帳號、多使用者或需 audit trail | MCP 通常較合適 |

這不是絕對規則. 實際選擇仍取決於服務支援、認證模型與組織安全要求.

## 13. 使用 Vercel 部署

[01:02:44](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=3764s)

影片確認本機已有 Vercel CLI 與 MCP 後, 要求 Claude Code 建立 Vercel project 並部署 tier-list 應用. 部署完成後, 講者開啟公開 URL 驗證功能.

接著, 他要求代理刪除剛建立的 project. 代理先列出帳號內的 projects, 區分測試 project 與既有正式資產, 再刪除目標, 最後確認 URL 回傳 404.

這段示範凸顯一項重要安全做法: 刪除或修改遠端資源前, 必須先解析精確目標並再次確認, 尤其帳號中同時存在 production projects 時.

## 14. FAQ 重點

[01:11:57](https://www.youtube.com/watch?v=7l6bXLAKyEI&t=4317s)

### Skills、MCP 與 agents 的關係

- Agent: 負責執行工作的角色, 可具有特定專長與指令.
- Skill: Agent 可遵循的可重複 SOP 或 workflow.
- MCP/CLI: Skill 執行過程中可呼叫的外部工具.

可概括為:

```text
Agent 執行任務
  -> 選擇或觸發 Skill
  -> Skill 指示 Agent 呼叫 MCP / CLI
  -> 工具對外部系統採取動作
```

### 初學者需要什麼

影片認為初學者不需要 Cursor, 可直接使用 VS Code terminal. Desktop app 較簡單, terminal/IDE 則有更多可見性和自訂能力.

### 是否需要 GitHub

短暫實驗不一定需要 remote repository, 但長期專案、多人協作或需要可靠 rollback 時, Git 與 GitHub 很重要.

### 費用、模型與限額

影片對方案價格、模型名稱、限額與其他 agent framework 的比較, 都屬於發布當時的講者看法. 這些資訊高度容易變動, 採購或選型前應另外查閱各產品的即時官方資料.

## 初學者建議的安全工作流

以下是根據影片內容整理的實作順序:

1. 建立獨立、可丟棄的練習資料夾.
2. 用 VS Code 開啟該資料夾, 從 terminal 啟動 Claude Code.
3. 先使用 Plan mode, 檢查代理理解是否正確.
4. 使用保守權限模式完成小型檔案操作.
5. 初始化 Git, 在重要修改前 commit.
6. 讀過 skill 內容後再安裝或觸發.
7. 定期查看 context, 大任務分成多個 session.
8. 連接 GitHub、Vercel 或 MCP 前, 使用測試帳號與最小權限.
9. 部署與刪除遠端資源時, 明確指定 project ID 或名稱並人工確認.
10. 最後在瀏覽器實際驗證功能, 不只依賴代理摘要.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 自動字幕頻繁將 `Claude Code` 辨識成 `Cloud Code`, `Claw Code` 或其他近音詞. 本文依影片標題與上下文統一為 `Claude Code`.
- 影片的 `/goal`、plugin discovery、permission modes 與部分 skills 來自發布時的環境和已安裝擴充, 不保證所有 Claude Code 安裝預設存在.
- 影片提到的模型、價格、方案限額、產品比較與 UI 可能快速變動. 本文只記錄課程內容, 不將它們視為目前官方規格.
- 影片示範包含 bypass permissions、建立 repository、部署及刪除 Vercel project. 這些操作會改變本機或遠端狀態, 實際執行時應採最小權限並逐項確認.
- 中文標題、表格、安全提醒與「初學者建議的安全工作流」是編輯整理. 本文不是逐字稿, 也不取代最新官方文件.
