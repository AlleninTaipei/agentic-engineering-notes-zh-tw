# State of the Claw: OpenClaw 的成長、安全與個人 Agent 願景

> 來源: [State of the Claw — Peter Steinberger](https://www.youtube.com/watch?v=zgNvts_2TUE)
> 頻道: AI Engineer
> 發布日期: 2026-04-17
> 片長: 44:12
> Video ID: `zgNvts_2TUE`
> 內容依據: YouTube 英文自動字幕 (`en-orig`)

## 摘要

OpenClaw 創作者 Peter Steinberger 回顧專案爆發式成長所帶來的治理與安全壓力, 並在問答中說明加入 OpenAI 後, OpenClaw 如何維持獨立、開源、跨模型的定位。

這場分享的核心不只是專案近況, 而是三個更廣泛的 agent 工程問題:

1. 強大 agent 的安全風險, 往往來自資料、非信任內容和對外通訊三種能力同時存在。
2. AI 可以大量產生安全報告與程式碼, 但判斷真實風險、產品方向和系統整體性, 仍需要人類維護者。
3. Agentic coding 的瓶頸正從「寫程式」移向 taste、系統設計、取捨和拒絕功能的能力。

Peter 也描繪個人 agent 的長期方向: 使用者掌控自己的資料, agent 能跨裝置存在於生活空間, 個人與企業 agent 能在雙方接受的權限邊界內溝通。

## 專案的爆發式成長

[00:00](https://www.youtube.com/watch?v=zgNvts_2TUE&t=0s)

演講時 OpenClaw 約五個月大。Peter 表示, 專案已累積約 3 萬次 commit, 接近 2,000 名 contributor, pull request 數量也將近 3 萬。這種快速成長並未減速, 但也形成典型的維護與治理壓力。

為降低 bus factor, 專案逐步引入更多維護者。參與者來自 Nvidia、Microsoft、Red Hat、Tencent、ByteDance 等公司, 分別協助安全、容器化、Windows、Microsoft Teams 及其他整合。

### OpenClaw Foundation 的治理挑戰

[02:23](https://www.youtube.com/watch?v=zgNvts_2TUE&t=143s)

Peter 一方面加入 OpenAI, 另一方面成立 OpenClaw Foundation。他把基金會形容成「困難模式的公司」: 同樣需要處理組織和營運事務, 但多數貢獻者是志工, 無法像企業員工一樣直接分派工作。

基金會的目標是建立獨立且跨公司的治理結構, 聘用全職人員, 維持開發速度、改善品質, 並讓核心維護者重新騰出時間探索產品方向。

## 安全通報的規模與訊噪比

[03:47](https://www.youtube.com/watch?v=zgNvts_2TUE&t=227s)

Peter 表示, 專案截至當時共收到 1,142 份安全 advisory, 平均每天約 16.6 份, 其中 99 份被標示為 critical。團隊公開了約 469 份, 並關閉約 60%。

這些數字看似驚人, 但他的重點是: 嚴重性分數、通報語氣和實際可被利用的風險不一定相等。AI 安全工具能快速找出多步驟、低機率或特殊設定下才成立的漏洞, 因而大幅提高通報量。維護者仍需逐一檢查前提、部署方式與實際影響。

### AI 同時擴大攻擊與防禦能力

Peter 舉 Nvidia NemoClaw 的整合測試為例。據他描述, 將產品接上 Codex Security 後, 半小時內找到數種突破 sandbox 的方式。這說明更強的模型能發現傳統人工檢查難以串連的 exploit chain, 也意味著軟體產業需要重新思考安全開發方式。

常見通報類型包括:

- Remote code execution。
- 繞過核准機制。
- Code injection。
- Path traversal。
- 權限提升。

然而 CVSS 分數只描述標準化條件, 未必反映產品的典型部署。例如某個權限提升問題可能得到最高分, 即使受影響的功能尚未正式發布, 或多數部署根本不使用細分權限模式。這不表示問題可以忽略, 而是修補排序不能只看分數。

### 真實風險也可能來自供應鏈

演講提到兩類專案本身難以完全控制的風險:

- 惡意套件使用相似名稱誘導下載, 安裝後植入惡意程式。
- 間接 dependency 未鎖定安全版本, 使上游漏洞透過 Teams 或 Slack 等整合傳入。

這些案例提醒維護者, agent 安全不只涵蓋 prompt injection, 也包括傳統軟體供應鏈、套件來源與依賴版本管理。

## 安全報導、部署前提與責任邊界

[10:33](https://www.youtube.com/watch?v=zgNvts_2TUE&t=633s)

Peter 批評部分研究與媒體在測試時刻意偏離安全建議, 卻未在結論中清楚揭露。例如 OpenClaw 被定位為 personal agent, 官方建議是只讓擁有者與它交談。如果要放進群組聊天, 就應啟用 sandbox, 並只提供該群組本來就能存取的資料。

安全邊界可以整理成以下原則:

| 使用情境 | 建議權限與隔離方式 |
| --- | --- |
| 個人 agent | 僅限擁有者使用, gateway 保持本機或私人網路可達 |
| 團隊 agent | 只存取團隊原本有權查看的資料 |
| 群組聊天 | 啟用 sandbox, 避免任一成員能間接控制高權限工具 |
| 小型或缺乏防禦訓練的模型 | 避免直接接觸郵件、網頁等非信任內容和高權限工具 |

演講舉出一個被政府資安單位警告的 RCE 案例。其攻擊需要惡意網站取得 gateway token, 但官方預設與建議設定讓 gateway 只在本機或私人網路中可見。Peter 的立場是, 問題仍應修補, 但風險說明必須包含非預設設定這個前提。

### Agentic System 的「致命三要素」

Peter 將最需要警戒的組合描述為:

```text
可存取私人資料
  + 可接觸非信任內容
  + 可對外通訊或採取行動
  = 資料外洩與間接操控的高風險組合
```

這並非 OpenClaw 獨有。任何 powerful agent 同時擁有三種能力, 都需要權限分層、sandbox、信任標記與模型層防禦。能力越強, 能完成的工作越多, 使用者就越需要理解其行為與風險。

## AI 產生的安全通報正在壓垮開源維護

[14:50](https://www.youtube.com/watch?v=zgNvts_2TUE&t=890s)

即使維護者知道多數通報由 agent 生成, 仍必須投入人力閱讀, 因為目前還不能完全信任 agent 自行判斷真假和優先級。

Peter 觀察到幾個常見問題:

- 通報數量增加, 但內容品質差異很大。
- 通報者很少同時提出可用修正。
- 隨附的修正經常品質不佳。
- 維護者在資訊過載下匆忙合併修正, 反而容易破壞產品。

因此 AI 降低了發現和提交問題的成本, 卻把驗證、整合與承擔後果的成本集中到維護者身上。只依靠志工很難長期吸收這種負擔, 也是基金會希望聘用全職人員的原因。

## OpenAI 與 OpenClaw 的關係

[16:12](https://www.youtube.com/watch?v=zgNvts_2TUE&t=972s)

Peter 明確表示, OpenAI 並未收購 OpenClaw。OpenAI 支持專案, 是因為讓更多人實際使用 AI, 能同時幫助社會理解其可能性和風險。但 OpenClaw 若要成功, 不應隸屬單一公司。

他提出的原則包括:

- OpenClaw 維持開源。
- 支援不同公司的模型及本機模型。
- 維護團隊由多家公司共同參與。
- OpenAI 可以提供資源, 但不應形成接管專案的外觀或事實。

問答中他也表示, OpenAI 對開源的態度正變得更積極。不過關於特定開放模型計畫, 他沒有提供未公開的內部資訊。

## 為什麼開放模型與本機模型重要

[22:28](https://www.youtube.com/watch?v=zgNvts_2TUE&t=1348s)

OpenClaw 的一項原始動機是資料主權。大型平台通常要求把 Gmail 等個人資料交給外部服務, 再由服務提供有限介面。Peter 更偏好的架構是:

- 使用者掌控完整資料。
- 只有必要片段送往頂級雲端模型。
- 可依任務在本機模型與雲端模型之間分層或 fallback。
- Agent 代表使用者操作現有網站, 減少被平台 connector 和 API 審核流程限制。

這種 hacker-style 路徑能打破資料孤島, 也能實作大型公司因法務或市場不確定性而不願率先推出的自動化。但它同時提高安全責任, 不能把「可做到」等同於「任何部署方式都安全」。

## Peter 的 Agentic Coding 工作流

[24:57](https://www.youtube.com/watch?v=zgNvts_2TUE&t=1497s)

Peter 曾同時執行接近十個 coding agent session。隨著模型推理和 token 生成速度提升, 典型工作流降到約五、六個視窗。他認為大量平行 session 是等待模型時的權宜之計, 並非人類最自然的工作方式。

### 為何不完全採用 Dark Factory

他不認同先完整定義產品, 再讓自動化 pipeline 寫完並合併所有程式碼的做法。產品探索不像直線登山:

1. 先做一小步。
2. 實際使用並感受結果。
3. 發現新的方向、問題或捷徑。
4. 修改 prompt 和產品想法。
5. 再進入下一輪。

如果一開始就固定完整規格, 本質上容易回到 waterfall model。第一版想法通常不會是最終產品, 而 AI 也無法可靠判斷每個外部 pull request 是否符合產品方向。

特定重複工作可以建立 pipeline, 但產品方向仍受限於思考與 taste。

## Taste 是 Agentic Engineering 的護城河

[28:28](https://www.youtube.com/watch?v=zgNvts_2TUE&t=1708s)

Peter 對 taste 的最低限度描述是: 成果不能帶有明顯的 AI 味。這包含制式寫作風格、相似 UI 模板、缺乏個性的產品語言等。

更高層次的 taste 則表現在細節。當軟體的大部分實作都能自動化, 人類反而能把更多時間花在微小但令人愉悅的互動上。例如 OpenClaw 啟動時偶爾出現帶有個性的訊息, 這類細節不太會從高層、泛化的 prompt 自然產生。

## Agent Personality 與 `SOUL.md`

[30:31](https://www.youtube.com/watch?v=zgNvts_2TUE&t=1831s)

早期聊天產品模仿搜尋引擎, 使用者只期待輸入問題和得到答案, 不期待工具具有個性。當 AI 從搜尋介面走向長期共處的 agent, personality 變得重要。

Peter 在把 coding agent 接上 WhatsApp 後發現, 原本的回覆方式不像真人訊息: 太冗長、標點和節奏不自然。這種「感覺不對」促使他反覆調整 `SOUL.md`, 讓 agent 的語氣更符合聊天情境。

這個案例說明 personality 並非只靠一次 prompt 設計, 而是透過實際使用、感受落差與持續修正形成。

## 無所不在的個人 Agent

[33:22](https://www.youtube.com/watch?v=zgNvts_2TUE&t=2002s)

Peter 的理想不是只在手機上使用 agent, 而是讓 agent 存在於每個空間:

- 使用者可以在任何房間以語音呼叫。
- Agent 知道使用者所在位置。
- 回答需要視覺資訊時, 使用最近的螢幕呈現內容。
- 眼鏡、耳機、手機與家中顯示器只是不同輸入輸出端點。
- 個人 agent 與公司 agent 可在雙方認可的資料和權限範圍內溝通。

手機是方便的入口, 但長期目標是 ubiquitous agents, 而非被單一裝置或單一聊天視窗限制。

## Prompt Injection 與模型選擇

[35:58](https://www.youtube.com/watch?v=zgNvts_2TUE&t=2158s)

Peter 表示, 頂級 frontier models 對網頁或郵件中隨機出現的惡意指令已有較好的辨識能力。如果系統清楚標記 untrusted content, 直接誘導資料外洩的難度會提高。

但風險沒有消失:

- 攻擊者若能持續與 agent 互動或大量餵入內容, 仍可能成功。
- 某些小型本機模型缺少相關防禦訓練, 可能直接服從惡意內容。
- 將這類模型同時接上瀏覽器、郵件和高權限工具尤其危險。

因此 OpenClaw 支援本機模型, 但會在使用小型模型時提出警告。這不是反對 local models, 而是避免一般使用者在不了解能力差異時建立高風險組合。

問答還提及用信任隨時間累積的權限系統, 讓較可信的來源取得較高權限。Peter 表示這只是整體方案的一部分, 並未宣稱 prompt injection 已被完全解決。

## Dreaming、記憶與模組化

[38:33](https://www.youtube.com/watch?v=zgNvts_2TUE&t=2313s)

OpenClaw 的 dreaming 概念模仿人類睡眠期間的記憶整理:

```text
白天的 session logs
  -> 回顧與整理
  -> 將部分短期經驗轉成長期記憶
  -> 丟棄不重要資訊
  -> 形成 dream log 或可再利用的知識
```

Peter 將已發布功能視為第一步。Dreaming 與 wiki、memory 的界線仍可能交疊, 需要透過實驗找出有效形式。

專案架構也正從早期的單體、混亂 codebase 轉向 extension 和 plugin。使用者可以替換記憶系統、加入 wiki、dreaming 或其他實驗, 不必讓所有客製功能進入主專案。這種方向更接近 Linux 的模組化生態。

## AI 時代工程師仍需培養的能力

[40:24](https://www.youtube.com/watch?v=zgNvts_2TUE&t=2424s)

Peter 最後提出三項關鍵能力。

### 1. Taste

能辨認制式 AI 產物, 並在互動、語氣和細節上做出有意識的選擇。

### 2. System Design

Agent 擁有大量程式知識, 但仍需要人類提出正確問題、定義邊界與指出系統關聯。否則它容易只看到局部, 建立彼此衝突的 localized solutions。

良好的提示不是只說「加入 user profiles」, 還會提醒 agent 檢查:

- 現有架構中有哪些相關模組。
- 新功能如何與權限、資料模型和其他功能互動。
- 哪些設計慣例必須延續。
- 如何避免在局部加入第二套解法。

### 3. 說不

每個想法如今都只差一個 prompt 就能實作, 因此單一功能的成本不再是主要限制。真正的問題是大量功能如何共同形成一致、可維護的產品。

人類仍需負責 big-picture thinking, 拒絕不符合方向的功能。Agent 被丟進 codebase 時, 可能只有過時的 `AGENTS.md` 和局部檔案, 無法自然理解完整產品願景。工程師的工作是提供脈絡、指出關聯並維護整體性。

## 實作啟示

### 部署個人或團隊 Agent

- 先辨識資料、非信任內容和對外行動是否同時存在。
- Personal agent 預設只允許擁有者控制。
- 團隊 agent 不應能讀取超出團隊權限的資料。
- Gateway 保持本機或私人網路可達, 不任意暴露到公網。
- 高權限工具搭配 sandbox、核准流程和清楚的信任標記。
- 依模型實際防禦能力決定可提供的工具, 不把模型大小或開源與否當成唯一標準。

### 維護 AI 高速生成的程式碼

- 不以通報數量或 CVSS 分數取代 threat model。
- 合併 AI 修正前, 驗證攻擊前提、典型部署和回歸風險。
- 用 extension boundary 隔離客製功能, 降低主專案維護負擔。
- 讓 agent 查看跨模組關係, 避免只解決眼前局部問題。
- 把產品方向、taste 和拒絕功能視為不可省略的人工責任。

## 核心結論

OpenClaw 的案例顯示, agent 工程的難點已超出「模型能不能完成任務」。當模型可以大量產生程式碼、漏洞報告與自動化時, 真正稀缺的是風險判斷、治理能力、系統整體性和產品 taste。

開放、可組合、由使用者掌控資料的 agent 具有吸引力, 但其能力與風險是同一件事的兩面。有效的系統需要根據部署情境縮小權限, 讓模型能力、工具、資料和信任邊界彼此匹配。

## 來源與可信度限制

本筆記依據 YouTube 提供的英文自動字幕與公開章節整理, 並非逐字稿。內容已移除口語贅詞、重複字幕、串場與無關宣傳, 以繁體中文重新組織。

演講中的專案規模、安全通報數量、漏洞實際影響及各公司參與情況, 均為講者在當時的陳述, 本筆記未另行獨立驗證。部分產品與專有名詞可能被自動字幕誤辨, 已在語境足夠明確時修正, 不確定處則避免延伸推論。
