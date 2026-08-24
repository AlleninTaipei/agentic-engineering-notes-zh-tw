# Claude Code 與 Agentic Coding 的演進

> 影片: [Claude Code & the evolution of agentic coding - Boris Cherny, Anthropic](https://www.youtube.com/watch?v=Lue8K2jqfKk)  
> 頻道: AI Engineer  
> 原始網址: https://www.youtube.com/watch?v=Lue8K2jqfKk  
> 發布日期: 2025-07-04  
> 片長: 18:12  
> Video ID: `Lue8K2jqfKk`  
> 內容依據: YouTube 原始英文自動字幕 (`en-orig`)

## 摘要

Claude Code 創作者 Boris Cherny 從程式設計介面的歷史切入, 說明 programming UX 如何從實體 switchboards, punch cards, text editors, IDEs 和 autocomplete, 演進到由自然語言驅動的 coding agents。

他的核心判斷是, 模型能力正在快速上升, 產品介面卻還在追趕。由於沒有人確定 agentic coding 的最終 UX, Claude Code 刻意保持簡單, 通用且低階, 以 terminal, IDE integration, GitHub integration 和 programmatic SDK 提供多種入口。

演講後半段提供幾項實務建議: 先讓 Claude 探索再規劃, 在取得 context 後才使用 extended thinking, 採用 TDD, 提供可觀測的驗證目標, 以 `CLAUDE.md` 和 slash commands 保存 context, 並用多個工作樹平行執行 agents。

## 程式設計一直在提高抽象層級

程式設計最初是實體操作。1930 至 1940 年代使用 switchboards, 之後轉向 punch cards。到了 1950 年代, assembly 和較高階語言逐步把程式設計從硬體配置轉成軟體表達。[01:59](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=119s)

接下來的演進包括:

- COBOL 等早期高階語言.
- Typed languages 與 C++.
- 1990 年代的 Haskell, JavaScript, Java 和 Python 等語言家族.
- TypeScript, Rust, Swift 和 Go 等現代語言.

Boris 認為, 現代語言的抽象和使用感逐漸收斂。語言本身的變化不像模型能力和 programming UX 一樣快速。

這不表示所有語言已經相同, 而是開發者常在不同語言中看到相似的 type systems, module systems, collections 和 control structures。

## Programming UX 的歷史

程式介面與語言一起演進。每次重要變化都降低了表達意圖所需的機械工作。[03:20](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=200s)

### Punch-card machines

早期程式設計使用 IBM 029 等 keypunch machines, 將指令打孔在實體卡片上。修改程式意味著重新製作卡片並管理整疊實體媒介。

### Ed

Ken Thompson 在 Bell Labs 建立的 `ed`, 是最早期的文字編輯器之一。它針對 teletype machine 設計, 沒有現代 cursor, scrollback 或 autocomplete, 但至今仍可在許多 Unix 系統中使用。

`ed` 的重要性不只在功能, 而是它把程式編輯介面從物理機器轉成軟體命令。

### Smalltalk-80

Smalltalk-80 將 graphical programming environment, object-oriented programming 和即時更新帶入更完整的整合體驗。Boris 特別提到, 1980 年代的環境已具備類似 live reload 的能力。

### Visual Basic 與 Eclipse

Visual Basic 將 graphical programming paradigm 帶向主流。Eclipse 則透過 static analysis 和 symbol indexing 普及 typeahead, 並建立大型 IDE plugin ecosystem。

### Copilot 與 Devin

GitHub Copilot 將 autocomplete 推進至 AI 驅動的單行及多行生成。Boris 將 Devin 視為率先把自然語言轉程式碼的 agentic IDE 概念推向主流的產品之一。

這條演進路徑可以概括為:

```text
實體配置
  → 文字指令
  → 圖形化 IDE
  → 靜態 autocomplete
  → AI code completion
  → 自然語言驅動的 autonomous coding
```

## Verification 也在演進

Programming UX 不只包含如何寫程式, 也包含如何確認程式正確。早期主要靠手動除錯和檢查輸出, 後來加入更自動化和機率式的方法, 例如 fuzzing, vulnerability testing 和 Netflix Chaos Engineering 類型的故障注入。[07:05](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=425s)

Agentic coding 進一步提高 verification 的重要性。模型能快速生成大量程式碼後, 開發瓶頸會從輸入程式碼轉向建立測試, 觀察輸出和判斷是否符合意圖。

## Claude Code 為何從 Terminal 開始

Claude Code 刻意從 terminal 出發, 讓使用者以較低階, 接近模型原始能力的方式操作。產品不加入大量 flashy UI 或固定 scaffolding, 因為 Anthropic 當時仍不確定最佳 agentic coding UX 應該長什麼樣。[07:45](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=465s)

這個選擇有三個目的:

- Simple: 減少介面本身的複雜度.
- General: 適用於不同工具, 工作流和程式庫.
- Unopinionated: 不過早把模型能力限制在某種固定開發模式.

Terminal 讓模型直接使用開發者現有 tools, commands 和 environments。它不是因為圖形介面沒有價值, 而是低階介面較適合在產品形態尚未收斂時探索用途。

## "The More General Model Always Wins"

Boris 將 "the more general model always wins" 貼在牆上提醒自己。模型能力快速提高後, 圍繞模型的產品和工具也傾向由通用方法勝出。[08:44](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=524s)

這個主張在 Claude Code 中表現為一個核心產品, 搭配多種使用方式, 而不是為每個環境建立完全不同的 agent。

這是講者的產品原則, 不是已被證明的普遍定律。高度受管制或需要特定互動保證的領域, 仍可能需要專用流程和介面。

## Claude Code 的四種入口

### Terminal

使用者可在 iTerm2, WSL, SSH, tmux, VS Code terminal 或其他終端環境執行 Claude Code。這使 agent 能進入開發者原有工作環境, 不必搬到全新 IDE。[09:05](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=545s)

### IDE integration

在 IDE 中執行時, Claude Code 會使用圖形介面的優勢, 例如顯示較完整的 diffs 並讀取 diagnostics。Boris 承認, 當時整合的 polish 不一定與 Cursor 或 Windsurf 相同, 目標仍是提供接近模型能力的最小介面。

### GitHub integration

演講當時剛推出 GitHub integration。使用者可安裝 GitHub app 並選擇 repository, 讓 Claude 在 GitHub 工作流程中執行任務。講者表示運算和資料留在使用者的 compute 環境。[10:07](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=607s)

這段描述反映 2025 年 7 月時的產品狀態, 現行部署和資料處理方式應以最新官方文件為準。

### Programmatic SDK and Unix utility

開發者可以 programmatically 呼叫 Claude Code, 自行建立 UI 和 integrations。Boris 舉例, 他會將 GCP logs pipe 到 Claude, 再用 `jq` 處理結果, 用於 incident triage。[11:05](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=665s)

這種用法把模型視為 Unix pipeline 中的 utility:

```text
logs or command output
  | Claude programmatic interface
  | jq or another Unix tool
  → structured result
```

Boris 認為這個方向當時只探索了約 10%, 如何把模型自然融入 shell composability 仍有很大空間。

## 使用方式 1: Codebase Q&A

對 coding-agent 新手而言, 最容易入門的工作不是直接要求大型修改, 而是先問 codebase 問題。[12:00](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=720s)

Anthropic 讓新進工程師在第一天學習 Claude Code。Boris 表示, onboarding 時間由約兩三週縮短至約兩天。這是講者的內部經驗陳述, 字幕未提供樣本或正式測量方法。

常見問題包括:

- 某項功能在哪裡實作?
- 某個 service 如何與其他元件互動?
- 最近一週完成了哪些 commits?
- 修改這段程式可能影響哪些部分?

Boris 自己會在每週 stand-up 前要求 Claude 檢查 Git commits, 整理當週交付內容。

## 使用方式 2: 教 Claude 使用既有工具

傳統 IDE 若要支援新工具, 通常需要建立 plugin 或 extension。Claude Code 可以直接使用 Bash CLI 或 MCP tools, 因而降低整合成本。[13:04](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=784s)

一個簡單流程是:

1. 告訴 Claude CLI 的名稱.
2. 要求它執行 `--help`.
3. 讓它整理所學內容.
4. 將穩定指令寫入 `CLAUDE.md`.

如果功能涉及大量工具, structured resources 或 streaming, 可以使用 MCP。重點是讓 agent 進入既有工具生態, 不必為每個工具建造專用 IDE bridge。

## 使用方式 3: 先探索與規劃, 再寫程式

Boris 建議讓 Claude 先探索 codebase, 建立計畫並交由人類審查, 再開始修改。[14:01](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=841s)

推薦順序如下:

1. 使用 tools 讀取 codebase 與相關 context.
2. 找出既有 patterns, tests 和限制.
3. 根據取得的資訊使用 extended thinking.
4. 產生具體 implementation plan.
5. 等待人類確認或修正.
6. 才執行程式碼變更.

他特別指出, extended thinking 在模型已擁有相關 context 時較有價值。若一開始尚未讀取任何資訊就要求長時間思考, 可能只是浪費 tokens。

## 使用方式 4: TDD 與可迭代目標

Coding agents 很適合 test-driven development, 因為測試替模型提供清楚且可重複執行的目標。[14:25](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=865s)

Boris 建議的流程是:

1. 先描述預期行為並要求 Claude 寫 tests.
2. 明確提醒 tests 此時預期失敗, 不要把失敗誤認為需要修改測試.
3. 先 commit tests.
4. 再實作功能.
5. 執行 tests 並迭代.
6. 完成後再 commit implementation.

TDD 是更廣泛原則的一個例子: 只要 agent 有可觀察的 target, 就能透過 feedback loop 改善結果。

可用的 targets 包括:

- Unit and integration tests.
- iOS simulator screenshots.
- Puppeteer browser screenshots.
- Lint 和 type checks.
- 實體裝置 camera feedback.

演講提到一個機器人和 3D printer 實驗。模型透過 camera 觀察實體輸出後繼續修改。第一次結果可能普通, 第二或第三次通常會改善。

## Plan Mode

演講當日推出的 plan mode, 是把 "先規劃, 後執行" 工作流產品化。當時使用者可以按 `Shift+Tab` 切換, Claude 只產生計畫並等待核准, 不立即修改程式碼。[15:30](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=930s)

Plan mode 的價值在於建立清楚的人機控制點:

- 模型負責探索和整合資訊.
- 人類負責檢查方向, 範圍與風險.
- 核准後才允許 agent 採取變更動作.

快捷鍵和實際行為可能已在影片發布後改變, 應以目前產品版本為準。

## 使用方式 5: 提供持久 Context

演講使用 `CLAUDE.md` 作為 repository context 的主要入口。[15:48](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=948s)

可以放置於:

- Repository root, 提供整個專案的通用指令.
- Subfolders, 在相關目錄按需載入.
- User home directory, 保存個人層級偏好.

適合記錄的資訊包括:

- Build, test 和 lint commands.
- Architecture 與重要 directories.
- Coding conventions.
- 常用 CLI tools.
- Verification requirements.
- Repository-specific limitations.

演講也提到 slash commands。把 Markdown 檔放進指定 commands directory 後, 可從 slash menu 呼叫, 適合封裝重複 workflows。

當時還能用 `#` 要求 Claude 記住資訊, 再選擇加入哪個 memory file。Boris 將這些功能視為 memory UX 的早期版本, 可以運作但仍很粗糙。

## 平行使用多個 Claude

當 agent 可以自主執行 10 分鐘, 使用者自然會同時啟動多個工作。Boris 表示, power users 常開啟數個 terminal tabs, 使用多份 checkout 或 Git worktrees 讓 Claude 平行處理任務。[17:20](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=1040s)

GitHub Actions 也能用來啟動多個獨立工作。多數任務不一定需要 agents 彼此即時協調, 只要切分成互不衝突的範圍。

如果確實需要交接, 最簡單的方法是讓 agent 將結果寫入 Markdown 檔, 再由另一個 agent 讀取。Filesystem 可作為低複雜度的協作介面。[17:44](https://www.youtube.com/watch?v=Lue8K2jqfKk&t=1064s)

## 編輯整理: 一個穩健的 Agentic Coding 流程

以下流程根據影片建議整理, 不是講者逐字提供的固定模板。

1. 在 `CLAUDE.md` 記錄 build, test, architecture 與工具使用方式.
2. 先用 codebase Q&A 確認 agent 能正確理解專案.
3. 要求 agent 探索相關檔案, 不立即寫入.
4. 取得 context 後再啟用較長 thinking.
5. 先產生 plan, 由人類審查範圍與風險.
6. 優先寫 failing tests 或建立其他可觀察 target.
7. 實作後讓 agent 執行 tests, screenshots, lint 和 type checks.
8. 使用 Git commits 或 worktrees 隔離變更.
9. 將獨立任務分派給多個 agents 平行處理.
10. 以 Markdown, commits 或 artifacts 進行 agents 之間的交接.

## 產品設計啟示

### 不要過早鎖定 Agent UX

模型能力變化速度快, 固定 UI 可能把下一代能力限制在舊工作流。低階且通用的 interface 能保留探索空間。

### Product overhang 仍然存在

模型已能完成的事情, 可能尚未有合適產品承接。Terminal, GitHub 和 SDK 是不同探索面, 不是最終答案。

### Verification 會成為主要介面

當生成成本下降, 好的 coding UX 越來越取決於模型能否看到結果, 取得 feedback 並安全重試。

### Filesystem 是通用協作層

`CLAUDE.md`, commands, logs, artifacts 和 handoff notes 都可透過檔案存在。這讓人類, agents 和既有 Unix tools 使用同一套可檢查媒介。

## 重點回顧

- 程式設計的歷史是不斷提高抽象層級, 自然語言 agent 是下一步介面.
- 模型能力增長速度高於產品 UX, 因此 Claude Code 刻意保持簡單和通用.
- Terminal, IDE, GitHub 和 SDK 是同一通用 agent 的不同入口.
- Codebase Q&A 是最容易採用 coding agent 的起點.
- 先取得 context, 再 thinking, planning 和 coding, 通常比直接生成更可靠.
- TDD 和 screenshots 等 verification targets 能讓模型反覆改善結果.
- `CLAUDE.md` 與 slash commands 將專案知識和工作流變成可重用 context.
- 多 agents 可透過 worktrees 隔離工作, 並以 Markdown 等 artifacts 交接.

## 來源與限制

- 本筆記依據 YouTube 原始英文自動字幕整理, 不是逐字稿. 字幕多次將 Claude, Claude Code, Anthropic, Devin, Datadog 與其他名稱辨識成相似詞, 本筆記只在上下文足以確認時修正.
- YouTube metadata 標題中的分隔符出現編碼異常, 本筆記將其正規化為 `Claude Code & the evolution of agentic coding - Boris Cherny, Anthropic`.
- 影片沒有章節, 時間連結依演講內容轉折選取.
- 影片發布於 2025-07-04. 安裝方式, plan 支援方案, keyboard shortcuts, GitHub integration, SDK, memory 與 commands 等產品細節可能已經變更.
- Anthropic onboarding 由數週縮短至數天, 以及模型迭代數次後品質改善等內容是講者經驗, 影片未提供正式實驗資料.
- "The more general model always wins" 是講者的產品設計原則, 不應視為所有安全, 合規與專業領域都適用的定律.
