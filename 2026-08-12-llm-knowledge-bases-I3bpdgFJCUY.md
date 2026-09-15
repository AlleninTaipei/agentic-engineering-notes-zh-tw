# 用 LLM 建立個人 Knowledge Base: 從語音筆記到自動更新 Wiki

- 影片: [LLM Knowledge Bases: a practical guide - Ben Holmes, Warp](https://www.youtube.com/watch?v=I3bpdgFJCUY)
- 頻道: AI Engineer
- 講者: Ben Holmes, Warp Developer Relations Lead
- 發布日期: 2026-08-12
- 片長: 21:17
- Video ID: `I3bpdgFJCUY`
- 內容依據: YouTube 英文原始自動字幕 (`en-orig`)

## 摘要

影片展示一套以 Markdown、agent skills 與排程組成的個人 knowledge base workflow。使用者先以語音快速記錄未整理的 thoughts, enrichment agent 再加入來源、固定 tags、處理時間與 related-note backlinks。累積足夠內容後, 另一個 agent 依指定 focus area 建立可瀏覽 wiki, 也能額外生成 graph view 或 habit visualization。

整體流程可簡化為:

```text
voice or text capture
        ↓
raw Markdown notes
        ↓
enrichment skill
  ├─ source
  ├─ fixed tags
  ├─ enriched timestamp
  └─ related-note links
        ↓
topic-specific wiki
        ↓
scheduled refresh and visualization
```

這套方法的價值不在某個特定筆記軟體, 而是把原始 Markdown 當成可攜資料層, 把整理規則放進 versionable skills, 再讓 local 或 cloud agent 按需處理。影片提供一個可實作的雛形, 但沒有評估 source matching、tagging、backlinks 與 wiki synthesis 的準確率, 也沒有深入處理敏感資料、同步衝突及自動改寫原文的風險。

## 問題不是沒有筆記, 而是無法重新使用

[00:00](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=0s)

Apple Notes 或其他快速記錄工具容易累積大量零散內容, 但使用者之後常無法回答:

- 這段想法存在哪裡?
- 哪些筆記談到同一個人或主題?
- 某個來源與後續想法之間有什麼關係?
- 過去已經研究過哪些內容?
- 哪些興趣或知識區域仍有明顯缺口?

講者希望把 scattered thoughts 轉成能搜尋、瀏覽與探索的 personal wiki。示範以 Markdown 作為底層格式, 但可使用 Hubble、Obsidian、Warp 或其他 Markdown viewer。

## 第一階段: 降低 Raw Capture 的摩擦

[02:46](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=166s)

Knowledge base 的上游瓶頸是把 thoughts 留下來。若每次記錄都要求完整標題、分類、格式與連結, capture friction 會讓使用者減少輸入。

影片建議先追求 volume, 不要求 raw note 立即整潔。輸入可能來自:

- 閱讀後的想法。
- Podcast 聆聽筆記。
- Meeting transcript。
- Research passage 的反應。
- 臨時出現的個人觀察。

### Voice Dictation

[03:25](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=205s)

講者認為 voice dictation 比鍵盤更容易快速輸入長篇 context, 並口述約每分鐘 200 words 的估計。影片未提供測量來源, 實際速度會受語言、環境、校正與編輯需求影響。

他示範兩類 local voice tools:

- Handy, open-source local dictation tool。
- VoiceInk, 提供 hotkey 與 mobile app 的 on-device workflow。

Local transcription 的潛在好處是減少 subscription 與資料上傳, 但影片沒有比較模型準確率、硬體需求或不同語言的表現。

核心原則是先取得 raw material。段落、punctuation、tags 與 backlinks 可以之後處理, 不必在靈感出現時完成。

## 第二階段: 用 Enrichment Skill 增加結構

[05:53](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=353s)

講者展示一篇口述的 Ferrari 創業故事筆記。Enrichment agent 保留主要內容, 並加入:

| 欄位 | 用途 |
| --- | --- |
| Tags | 依主題或內容類型分類 |
| Source | 找回 podcast、文章或其他原始來源 |
| Enriched timestamp | 記錄 agent 已在何時處理過 |
| Related notes | 建立可點擊 backlinks |

Timestamp 同時充當簡單的 processing marker。下次批次執行時, agent 可先尋找沒有 marker 的 notes, 避免每次重做整個資料庫。

### Enrich Note Skill

[06:32](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=392s)

Skill 將 enrichment procedure 寫成可重用 instructions。依影片示範, 程序大致包含:

1. 讀取指定 Markdown note。
2. 加入或更新 enrichment timestamp。
3. 從既有 taxonomy 選擇 tags。
4. 必要時用 web search 找出可能來源。
5. 以 filesystem keyword search 尋找 related notes。
6. 在筆記末尾加入 backlinks。

這種設計把整理規則從 prompt history 移到可版本化的 skill, 因而可以手動呼叫, 也能交給 scheduled agent。

## 固定 Tag Taxonomy, 避免無限增生

[07:11](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=431s)

如果每次 enrichment 都讓模型自由產生 tags, 相近概念容易形成不同拼法、單複數或過細分類。最後每個 tag 只對應少量 notes, 無法提供有效導航。

講者將允許的 tags 放入 reference folder, 並要求 agent 優先從固定清單選擇。只有發現真正重複的新 pattern 時, 才考慮擴充 taxonomy。

這裡可以進一步區分兩種操作:

- Assign existing tag: 低風險, 可由 agent 自動執行。
- Create new tag: 會修改整體 ontology, 適合設更高門檻或人工 review。

後者是依影片原則延伸的編者整理, 不是 demo 中明確展示的 approval mechanism。

## Backlinks 建立個人 Rabbit Holes

[09:07](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=547s)

完成 enrichment 後, 使用者可以從一篇 note 點入相關 podcast、書籍章節或人物筆記, 形成類似 Wikipedia rabbit hole 的閱讀方式。

Backlinks 的實際品質取決於 related-note retrieval。影片使用 key-term filesystem search, 因此可能遇到:

- 同一概念使用不同詞彙而漏掉連結。
- 相同名稱指向不同人物或事物。
- 關鍵字重疊但語意不相關。
- Agent 建立大量弱連結, 讓 graph 失去辨識度。

影片展示幾個成功 links, 沒有提供 precision、recall 或人工修正量。可靠系統應保存 agent 為何建立連結的證據, 並讓使用者容易移除錯誤關係。

## 第三階段: 從 Notes 生成 Topic Wiki

[09:45](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=585s)

Backlinks 適合逐篇探索, wiki 則提供聚合後的導覽層。講者展示依 focus area 生成的 wiki, 例如 latest AI news 與 Bible study。

Wiki 可以將 notes 整理成:

- Sources。
- Concepts。
- People。
- Organizations。
- Related notes 與原始資料連結。

使用者先描述關心的 focus area, agent 再從 raw directory 選取相關內容, 建立 topic pages 與 navigation。講者表示這個想法受 Andrej Karpathy 發布的 wiki gist 啟發。

Wiki 是 derived view, 不應取代 raw notes。理想結構是 wiki entry 都能連回來源, 讓讀者區分原始觀察與 agent synthesis。

### 工作情境的延伸

[12:54](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=774s)

講者推測相同方法可以用在工作筆記。例如以 meeting notes 建立 people pages, 整理 customer conversations, 再連回每次 meeting 與資料來源。

這是應用構想, 不是影片實際展示的 production workflow。若涉及客戶、員工或內部會議, 還需要權限、retention、consent、PII 與資料分類政策。

## 第四階段: 讓 Knowledge Base 自動更新

[13:31](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=811s)

手動執行 enrichment 或 wiki generation 可能耗時。講者建議將 skills 放進 daily 或 weekly schedule, 讓 agent 在背景處理新 notes。

影片比較兩種執行位置:

| 模式 | 優點 | 限制 |
| --- | --- | --- |
| Local automation | Notes 不必離開本機, 可使用本地工具 | 排程時電腦必須開機並可執行 |
| Cloud sandbox | 不依賴個人電腦在線, 容易定期執行 | 必須同步資料, 並處理 secrets、privacy 與 storage policy |

講者提到 Codex app automations 作為 local example, 以及 Warp 的 Oz 作為 cloud scheduling example。這些產品能力可能隨版本改變。

## Markdown 同步至 Sandbox 再寫回

[15:29](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=929s)

Cloud automation 的資料流如下:

```text
local Markdown folder
        ↓ sync
cloud sandbox
        ↓
run enrichment or wiki skill
        ↓
agent file changes
        ↓ sync back
updated local knowledge base
```

影片使用 Obsidian headless CLI 同步 Markdown, 也指出 Git repository 是另一個選擇。排程 prompt 會要求 agent:

1. 將 notebook 同步到 sandbox。
2. 尋找尚未 enrich 的 notes。
3. 執行 skill 並產生 file changes。
4. 更新 wiki 或 backlinks。
5. 將結果同步回原始 notebook。

講者描述的體驗是每天早上看到已更新的 wiki。Demo 顯示 cloud run 能處理新 notes 並補上來源與 related notes, 但沒有討論 concurrent edits、partial failure、重複執行及 rollback。

## 第五階段: 生成視覺化介面

[18:01](https://www.youtube.com/watch?v=I3bpdgFJCUY&t=1081s)

Markdown 保留可攜資料, HTML 則可以建立不同閱讀介面。講者讓 agent 使用 HTML 與 Tailwind 生成 graph view, 將 notes 依 AI、engineering、startup、books 與 faith 等主題群聚。

Graph nodes 可以點擊並連回 notes, 也能改變 visual style。其他可生成的 views 包含:

- Habit or contribution chart。
- Topic distribution。
- Research coverage map。
- Timeline。
- Reading or meeting dashboard。

這些圖表能幫助探索, 但圖形位置與群聚未必代表可靠的 semantic structure。若 graph generation 沒有明確演算法、資料欄位與 validation, 它較適合作為導航介面, 不應直接當成研究結論。

## 建議的安全資料模型

以下是根據影片 workflow 整理的改良結構, 不是影片原始 schema:

```text
notes/
  raw/              # 使用者原始內容, agent 預設不可覆寫
  enriched/         # tags、sources、backlinks 與 processing metadata
  wiki/             # 可重新產生的 derived views
  visualizations/   # 可重新產生的 HTML 或資料檔
  taxonomy/         # 受控 tag vocabulary
  reviews/          # 待人工確認的來源、連結與新 tags
```

將 raw、enriched 與 derived content 分離, 可以避免 enrichment agent 無意改變使用者原意, 也能讓 wiki generation 在失敗時安全重跑。

## 實作檢查表

- Raw capture 是否足夠快速, 不要求使用者先完成分類?
- 原始 notes 是否保持 immutable, 或至少有完整 version history?
- Enrichment metadata 是否記錄時間、agent、model 與 skill version?
- Tags 是否來自固定 taxonomy?
- 建立新 tag 是否需要較高 confidence 或人工確認?
- Source matching 是否保留候選 URL 與匹配理由?
- Backlinks 是否能顯示 evidence, 並方便移除錯誤關係?
- Wiki entries 是否連回 raw notes, 避免 synthesis 失去 provenance?
- Scheduled job 是否具有 idempotency、retry、timeout 與 failure report?
- Local 與 cloud 同時編輯時如何處理 sync conflict?
- Sensitive notes 是否被排除於 web search、cloud sandbox 與外部 models?
- Agent 是否只能修改 derived files, 而不能靜默覆寫原始想法?
- Taxonomy、skills 與 generated views 是否納入 version control?
- Visualizations 的 clustering 是否有可說明的方法與資料來源?

## 核心結論

LLM knowledge base 的重點不是讓模型替使用者記住一切, 而是降低 capture friction, 保存可攜的 raw material, 再把重複整理工作交給具明確規則的 agents。

Markdown 提供可擁有與可搬移的資料層, skills 保存 enrichment procedure, wiki 與 graph 則是可以重建的 derived views。若再加入排程, 系統就能從一次性筆記工具演進為持續更新的個人知識管線。

自動化程度提高時, provenance、privacy、idempotency、rollback 與 human review 也必須同步增加。否則整齊的 wiki 可能只是把 source mismatch、錯誤 backlinks 與 agent-generated interpretations 包裝得更可信。

## 時效性與限制

本筆記依 YouTube 英文原始自動字幕整理。Handy、VoiceInk、Hubble、Obsidian headless CLI、Warp、Oz 與 Codex automations 的名稱、價格與功能可能變動, 實作前應查閱目前官方資訊。

Ben Holmes 是 Warp Developer Relations Lead。影片包含 Warp 與 Oz demo, 具有明確產品推廣性。它能證明講者使用這些工具建立個人 workflow, 不能證明該組合比其他 knowledge management systems 更準確或更有效。

影片展示少量個人 notes 的成功案例, 沒有提供 tagging accuracy、source matching accuracy、backlink quality、wiki hallucination rate、長期維護成本或使用者研究。Voice dictation speed、減少人工工作及 knowledge-gap discovery 等說法也未經影片中的正式測量。
