# AI 時代, 軟體工程基本功比以往更重要

> 來源: ["Software Fundamentals Matter More Than Ever" — Matt Pocock](https://www.youtube.com/watch?v=v4F1gFy-hqg)  
> 頻道: AI Engineer  
> 發布日期: 2026-04-23  
> 影片長度: 18 分 26 秒  
> Video ID: `v4F1gFy-hqg`  
> 內容依據: YouTube 英文原語自動字幕, 經去重、編輯與繁體中文整理.

## 核心論點

Matt Pocock 認為, AI 並沒有讓軟體工程基本功失去價值. 相反地, AI 能快速產生大量程式碼之後, 系統設計、需求溝通、測試、模組邊界與持續重構變得更加重要.

他的理由是:

- AI 在良好 codebase 中能快速工作.
- AI 在複雜、難以修改的 codebase 中會更快累積錯誤.
- 程式碼產生速度提高, 也提高了 software entropy 的速度.
- 人類因此更需要從策略層級掌握系統設計, 而不是退出程式碼本身.

## 1. Specs-to-code 的問題

[01:12](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=72s)

`specs-to-code` 的想法是: 人只維護規格, AI 將規格轉成程式碼. 如果結果不對, 就修改規格並重新生成, 不必深入查看程式碼.

講者實際嘗試後發現, 每次重新生成都可能讓程式碼品質繼續下降. 若人完全忽略 codebase, 這種做法只是換了名稱的 vibe coding. 問題並不只是「編譯器還不夠好」, 而是團隊放棄了對系統結構的管理.

他引用 John Ousterhout 在《A Philosophy of Software Design》中的觀點: 軟體複雜度是任何讓系統變得難以理解與修改的結構因素. 因此:

- 壞的 codebase 難以安全修改.
- 好的 codebase 容易理解與修改.

《The Pragmatic Programmer》談到的 software entropy 也適用於 AI coding. 若每次變更只關注眼前功能, 不考慮整體設計, 系統會逐漸失序.

## 2. 「Code is cheap」並不成立

[03:46](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=226s)

AI 讓產生程式碼變便宜, 但不代表擁有、理解和維護程式碼的成本也變低. 講者甚至認為, 壞程式碼現在比以往更昂貴.

原因是 AI 在結構良好的 codebase 中可以帶來很大的生產力. 如果系統難以修改, 團隊便無法充分利用 AI, 還會讓代理快速放大既有問題. 真正有價值的資產不是大量程式碼, 而是能持續安全變更的系統.

## 3. 失敗模式一: AI 沒有做出你想要的東西

[04:42](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=282s)

需求往往不是一開始就完全明確. 人與 AI 之間存在溝通障礙, AI 必須透過對話完成 requirements gathering.

講者引用 Frederick P. Brooks《The Design of Design》的 `design concept`: 多人共同設計時, 參與者之間會形成一個共享的設計概念. 它不是單一 Markdown 文件, 而是大家對「正在建造什麼」的共同理解.

### Grill Me skill

[05:50](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=350s)

為了讓人與 AI 建立共同的 design concept, 講者設計了 `Grill Me` skill. 它要求 AI:

- 持續訪談使用者, 直到雙方理解一致.
- 沿著 design tree 逐一處理不同決策分支.
- 按依賴順序解決問題.
- 不急著產出計畫並立即開始實作.

一次訪談可能包含數十個甚至上百個問題. 這會把 AI 變成提出挑戰的協作者, 主動找出模糊處與未說明的假設. 對話完成後, 再將共識整理成 PRD 或 issues, 交給可離線工作的代理執行.

講者認為, 先取得共同理解通常比直接進入工具預設的 plan mode 更好. Plan mode 容易過早追求產出一份文件, 但文件不等於雙方真的已經對齊.

## 4. 失敗模式二: AI 過度冗長, 雙方語言不一致

[07:31](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=451s)

當開發者、domain expert 與 AI 對同一個詞有不同理解時, 對話會變得冗長, 實作也容易偏離原意.

講者從 Domain-Driven Design 引入 `ubiquitous language`. 同一組術語應一致地出現在:

- Domain expert 與開發者的溝通.
- 開發者彼此的討論.
- 程式碼中的命名與結構.
- 人與 AI 的規劃和實作對話.

### Ubiquitous Language skill

[09:01](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=541s)

這個 skill 掃描 codebase 中的術語, 將核心詞彙整理成 Markdown 表格. 人可以在規劃時開著這份文件, AI 也能用它理解一致的 domain model.

講者觀察到, 共同語言不只改善規劃, 也會讓 AI 的推理較精簡, 並使實作更貼近原先討論的設計.

## 5. 失敗模式三: AI 做對了功能, 但程式不能運作

[10:06](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=606s)

AI 需要能觀察結果的 feedback loops, 例如:

- Static types.
- Automated tests.
- Linting 與編譯檢查.
- 前端應用的瀏覽器存取與實際操作.

只有回饋工具還不夠. AI 常一次產生太多程式碼, 到最後才執行 type check 或測試. 《The Pragmatic Programmer》把這種行為稱為 `outrunning your headlights`, 也就是前進速度超過你能看清問題的距離.

> The rate of feedback is your speed limit.

中文意譯: 回饋速度決定了你能安全前進的最高速度.

## 6. TDD 強迫代理採取小步驟

[11:03](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=663s)

講者建議用 test-driven development 約束 AI:

1. 先建立會失敗的測試.
2. 寫出讓測試通過的最小實作.
3. 在測試保護下重構並改善設計.

TDD 能阻止代理一次做過多工作, 讓每一步都有可觀察回饋. 但測試設計本身仍很困難, 因為必須決定:

- 測試單位應該多大.
- 哪些依賴需要 mock.
- 應驗證哪些外部行為.
- 如何避免緩慢或不穩定的測試.

這些決策彼此相關. 因此, 測試品質最終仍取決於 codebase 的架構.

## 7. 失敗模式四: Codebase 很難探索和測試

[12:45](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=765s)

講者再次引用 John Ousterhout 的 `deep modules`:

| 類型 | 介面 | 內部功能 | 結果 |
| --- | --- | --- | --- |
| Deep module | 簡單、穩定 | 豐富, 複雜度被封裝 | 容易使用與形成測試邊界 |
| Shallow module | 相對複雜 | 功能很少 | 依賴零碎, 探索與測試困難 |

大量 shallow modules 會形成許多小檔案、函式和交錯依賴. AI 需要追蹤整張依賴圖, 容易漏掉正確修改位置或不了解完整流程. 而且 AI 本身很容易繼續生成這種零碎架構.

Deep modules 將相關行為收進清楚邊界, 對外只提供少量介面. 測試可以從介面驗證整個模組, AI 也更容易探索、實作與取得完整回饋.

### Improve Codebase Architecture skill

[14:17](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=857s)

講者將反覆使用的架構改善步驟整理成 skill:

1. 探索 codebase.
2. 找出彼此相關但散落的程式碼.
3. 尋找可以建立清楚邊界的位置.
4. 將行為封裝成 deep module.
5. 從公開介面建立測試.

這不是一次完成的架構重寫, 而是持續尋找能加深模組的機會.

## 8. 失敗模式五: 開發速度提高, 人腦卻跟不上

[15:09](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=909s)

AI 讓團隊產生更多程式碼, 開發者卻可能更疲憊, 也更難掌握整個系統. Deep modules 同時能降低 AI 與人類的認知負擔.

講者建議把非關鍵模組視為 `gray boxes`:

- 人設計並理解公開介面.
- 人掌握模組目的和外部行為.
- 測試從外部驗證契約.
- 內部實作可委派給 AI, 不必逐行記住所有細節.

涉及金融或其他高風險邏輯時, 不能任意降低審查程度. 這種委派方式適合風險較低、邊界明確且能可靠測試的模組.

## 9. Design the interface, delegate the implementation

[15:45](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=945s)

這是講者整套方法的濃縮原則: 人應設計系統形狀與模組介面, AI 則可負責戰術層級的內部實作.

因此, module map 應納入 ubiquitous language 與規劃流程. PRD 不只描述使用者功能, 還要明確寫出:

- 哪些模組會新增或修改.
- 模組對外提供哪些介面.
- 介面如何改變.
- 哪些行為構成測試邊界.

## 10. 人類仍負責策略層級

[17:01](https://www.youtube.com/watch?v=v4F1gFy-hqg&t=1021s)

講者引用 Kent Beck 的原則: 每天都要投資於系統設計. Specs-to-code 的危險在於團隊把設計責任交出去, 只關注規格輸入和功能輸出.

他把 AI 比喻為地面上的優秀戰術執行者. AI 可以迅速完成具體修改, 但仍需要有人在更高層思考:

- 系統長期應形成什麼結構.
- 哪些介面值得穩定下來.
- 哪些複雜度應被封裝.
- 哪些回饋迴圈能保證品質.
- 每次變更是否改善或侵蝕整體設計.

這個策略角色就是人類工程師. 它依賴的正是多年來的軟體工程基本功.

## 六項實務原則

| 問題 | 對應原則 |
| --- | --- |
| AI 沒做出真正想要的功能 | 先用密集問答建立 shared design concept |
| 人與 AI 溝通冗長、術語混亂 | 建立 ubiquitous language |
| 實作結果不能運作 | 提供 types、tests、browser 等 feedback loops |
| AI 一次修改太多 | 使用 TDD, 讓回饋速度限制前進速度 |
| Codebase 難以探索和測試 | 將 shallow modules 改善成 deep modules |
| 人腦跟不上 AI 產碼速度 | 設計介面, 測試契約, 委派內部實作 |

## 一句話總結

AI 降低的是產生程式碼的成本, 不是擁有壞程式碼的成本. 越能快速生成程式碼, 團隊越需要需求對齊、共同語言、測試、模組化與持續設計.

## 來源與信心限制

- 原影片沒有人工字幕, 本文依 YouTube 英文原語自動字幕整理.
- 影片沒有公開章節. 本文的段落與時間標記依演講內容轉折編排, 不是創作者提供的章節名稱.
- 自動字幕將部分專有名詞辨識錯誤. 例如講者談論的工具名稱依上下文應是 Claude Code, 但本文避免依不可靠字幕擴寫相關細節.
- 中文標題、表格、分類與解釋是編輯整理. 短句翻譯已明確標為中文意譯.
- 本文是學習摘要, 已移除寒暄、現場互動、宣傳與重複內容, 不取代完整演講.
