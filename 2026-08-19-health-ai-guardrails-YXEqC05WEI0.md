# 會員端醫療 AI 的 Guardrails: 架構、持續評估與上線決策

> [!info] 來源
> - 影片: [Guardrails First: Engineering Member-Facing Health AI — Rashi Agrawal, Hinge Health](https://www.youtube.com/watch?v=YXEqC05WEI0)
> - 頻道: AI Engineer
> - 發布日期: 2026-08-19
> - 片長: 21:49
> - Video ID: `YXEqC05WEI0`
> - 內容依據: YouTube 英文原始自動字幕 (`en-orig`)
> - 整理語言: 繁體中文

## 一句話總結

高風險醫療 AI 的安全不能只靠模型或 system prompt。不可失敗的決策應由模型外的確定性程式控制, PHI 保護要從資料管線邊界開始, 上線後則以持續評估、真實使用者回饋與人工判讀維持安全。

## 醫療 AI 的生產基準

講者以消費者使用一般 AI 進行健康分流的風險開場。她舉出錯誤飲食建議導致溴中毒, 以及健康 AI 對危及生命狀況 under-triage 的案例 ([01:04](https://www.youtube.com/watch?v=YXEqC05WEI0&t=64s))。

影片提到約 4,000 萬人使用模型自行判斷健康問題, 並稱一項獨立測試發現系統在約一半的緊急案例中未正確建議立即急診。這些數字與事件是講者引用的資料, 字幕沒有提供完整研究方法或可核對引文, 應回到原始研究確認。

## 三個不可妥協的基礎

講者提出三項原則 ([02:35](https://www.youtube.com/watch?v=YXEqC05WEI0&t=155s)):

1. Constraint is the architecture: 多數安全失敗源於 token 產生前的架構選擇。
2. Deterministic rules belong above the model: 絕不能錯的行為不能交給機率系統。
3. Safety is a continuous evaluation layer: 上線是風險真正開始的時點, 不是安全工作的終點。

## Layer 1: 以架構保護 PHI

Policy 說明什麼需要保護, architecture 則讓某些違規路徑根本無法發生 ([03:41](https://www.youtube.com/watch?v=YXEqC05WEI0&t=221s))。

影片建議在 ingestion boundary 就移除 PHI, 不等資料進入 data lake 或 dashboard 後才做 log redaction。如此一來, 開發者查看非必要資料時並非「看得到但被要求不要用」, 而是資料根本不存在。

其他架構界線包括:

- Production 與 non-production 完全分離, 避免任何管線把會員資料送進開發環境。
- 存取同時依角色與地理區域限制。
- HIPAA、FDA Good Machine Learning Practice 與州法要求作為設計輸入, 而非完成產品後再附加。

核心原則是讓系統無法產生部分失敗模式, 而不是只相信人員會遵守政策。

## Layer 2: 確定性規則放在模型之前

生成模型適合處理對話的長尾, 但不可負責不可逆、高風險決策。影片提出每輪互動先經過 code layer, 再決定是否讓模型處理 ([05:53](https://www.youtube.com/watch?v=YXEqC05WEI0&t=353s))。

```text
會員輸入
   ↓
確定性 code layer
   ├─ 緊急事件、身份失敗、高風險路由 → 直接處理或升級
   └─ 一般情況 → 交給模型處理長尾對話
```

講者明確表示, model 不是 guardrail, 加上 system prompt 的 model 也不是安全邊界 ([07:27](https://www.youtube.com/watch?v=YXEqC05WEI0&t=447s))。Prompt injection 或指令衝突可能改變模型行為, 安全控制必須由系統強制執行。

### 三種應由 code layer 控制的行為

- Emergency escalation: 偵測自傷、輕生或急性醫療緊急情況後, 直接路由到 911、988 或臨床人員。
- Intent routing: 避免臨床問題被錯送到一般 tech support agent。
- Identity verification: 任何會員資料操作都先完成 authentication, 不把身份判斷交給 prompt。

實務上, 緊急狀況辨識本身也可能含有自然語言不確定性。影片強調的是最終高風險決策與路由應由可審核的系統控制, 並不代表所有前置分類都能以簡單規則達成零錯誤。

## Layer 3: 安全是持續運作的評估層

Pre-launch golden dataset 必要但不足。正式環境需要持續對真實對話評分 ([09:39](https://www.youtube.com/watch?v=YXEqC05WEI0&t=579s))。

影片列出三類訊號:

1. Automated judges: 評估 clinical accuracy、safety、escalation、relevance、drift、refusal 等面向。
2. Member feedback: 每則訊息的 thumbs up/down 與回饋, 能發現語氣或使用感受問題。
3. Sample traces: 隨機抽樣不同能力, 高風險案例採 100% 人工檢查。

真正瓶頸通常不是 compute, 而是是否有足夠人員閱讀訊號、調查原因並採取行動。每個新型 production failure 都應形成新的測試或 judge。

## 五個 stakeholder 的上線衝突

影片設計一個距離上線五天仍有缺陷的場景 ([12:47](https://www.youtube.com/watch?v=YXEqC05WEI0&t=767s))。五種角色看到不同風險:

| Stakeholder | 關注風險 |
| --- | --- |
| Clinical | 會員傷害 |
| Legal | 法律與監管責任 |
| Compliance | 稽核失敗 |
| Product | 採用與產品效果 |
| Engineering | 延期與交付速度 |

每個觀點都可能合理, 因此需要共同決策框架, 不能只由聲量最大的一方決定。

## 五條決策規則

### 1. Worst case always wins

嚴重度由最壞的合理結果設定, 不是由平均影響設定 ([14:02](https://www.youtube.com/watch?v=YXEqC05WEI0&t=842s))。造成少數人重大傷害的錯誤, 可能比影響所有人但只帶來輕微不便的錯誤更嚴重。

### 2. Severity is not capacity

Bug severity 取決於傷害, 不能因團隊沒有時間修正就降低級別。可選擇修正、延後上線, 或由適當負責人明確簽核接受風險。

### 3. Asymmetric default

不確定時選擇較安全的錯誤:

- Safety issue: 誤延後上線的成本小於放過真實安全缺陷, 因此傾向 hold and fix。
- Polish issue: 小瑕疵上線的成本可能低於延期, 因此傾向 ship。

### 4. Revealed risk tolerance

組織真正的風險容忍度是目前已在 production 接受的行為, 而不是口頭宣稱的「零缺陷」。新功能的標準應與既有產品一致, 但這不代表既有危險行為應永久合理化。

### 5. Humans are the constraint

Judge 能大量評分, 但 pattern interpretation 與處置能力仍有限。系統要預先規劃人員、輪值、調查與 escalation, 不能只建立 dashboard。

此外, fast follow 是已承諾的技術債, 不是可無限延後的 backlog。

## 先驗證 scorer, 再相信 score

Judge 本身也是非確定性系統。若 clinical accuracy 從 4.9 降至 4.5, 不應立刻修改 agent prompt, 而要先確認 judge 是否正確 ([18:10](https://www.youtube.com/watch?v=YXEqC05WEI0&t=1090s))。

影片用咖啡因建議說明兩種情況:

- Agent 給出一般 FDA 指引並補充孕期與藥物情境, judge 卻誤判為 hallucination。此時應修正 judge。
- Agent 宣稱每日 1,000 mg 咖啡因安全, judge 正確指出超出安全範圍。此時應修正 agent。

修改 judge prompt 不是作弊。Judge 也是 production software, 需要版本、測試與持續改善。

## 六句總結

影片最後把架構與決策濃縮成六個方向 ([20:24](https://www.youtube.com/watch?v=YXEqC05WEI0&t=1224s)):

- 能以 architecture 強制做到的, 不只寫成 policy。
- 能以 code 保證的, 不只寫進 prompt。
- 能持續 monitor 的, 不只設一次性 gate。
- 依最壞合理結果評估風險。
- 不確定時選擇較安全的錯誤。
- 依組織實際風險行為校準, 並始終保留 human in the loop。

## 實務檢查表

- PHI 是否在 ingestion boundary 即被移除或最小化?
- Production 與 non-production 是否存在任何資料通道?
- 身份驗證與高風險路由是否由程式強制?
- Prompt 是否被錯當成 security boundary?
- 緊急與高風險案例是否全部人工檢查?
- 是否結合 automated judges、會員回饋與 trace review?
- 每個 production failure 是否轉化成 regression test 或 judge?
- 風險嚴重度是否獨立於修復容量?
- Fast follow 是否有負責人與完成期限?
- 修改 agent 前是否先驗證 judge 的正確性?

## 時效性與限制

影片發布於 2026 年 8 月, 涉及醫療、法規與安全的內容可能隨司法管轄區及規範更新。本文是演講筆記, 不能取代醫療、法律、安全或合規專業意見。

來源是英文原始自動字幕。影片中的事件、比例、機構排名與法規名稱可能有字幕辨識誤差, 且未附完整原始引用。涉及實際產品決策時應核對研究、法規正文與組織的臨床治理程序。

