# 哪些 AI Startups 能取得 Enterprise Contracts: 企業買方的採購, 資安與可靠性門檻

- 影片: [Which AI startups actually land enterprise contracts? - Brian Lewis, Millennium](https://www.youtube.com/watch?v=7A65O-0lvKE)
- 頻道: AI Engineer
- 講者: Brian Lewis, Millennium
- 發布日期: 2026-08-29
- 片長: 18:44
- Video ID: `7A65O-0lvKE`
- 內容依據: YouTube 英文原語自動字幕 (`en`)
- 身分聲明: 講者表示內容僅代表個人, 不代表 Millennium

## 摘要

Brian Lewis 從大型 hedge fund 的 enterprise buyer 角度, 說明 AI startup 為何經常無法從 demo 走到正式合約. 真正的門檻不只是 model capability, 還包括能否解決問題, pricing 是否對應價值, integration 是否已經存在, security architecture 是否可信, control plane 是否可管理, SLA 是否可執行, data retention 是否與合約一致.

他描述的採購漏斗是, 每個 pain point 可能先找到 10 到 15 家 startups, 經初步審查後安排 2 到 3 場 demos, 再進入 0 或 1 個 pilot, 約每 4 個 pilots 才有 1 個成為合約. 依此估算, 約 5% 的 demo calls 最後簽約. 這是講者個人工作經驗, 雖稱與外部 benchmark 接近, 影片沒有提供可核對的研究來源.

演講的訊號密度來自具體失敗案例. 有 vendor 要求客戶回報自有 LLM gateway telemetry, 再對沒有經過 vendor infrastructure 的流量加價. 有 integration 只有取得所有資料的 read/write permissions 才能運作. 有 vendor 宣稱 zero data retention, 後來卻提到只有保留客戶資料才可能知道的內容. 還有團隊面對 breach response 問題時只回答「目前沒有發生過 breach」.

核心結論是, AI 會放大既有 enterprise foundations. Entitlements, integration, architecture, auditability, data hygiene 和 change management 若原本就有缺口, agents 只會讓問題以更高速度和更大權限擴散.

## Enterprise 採購漏斗

[02:52](https://www.youtube.com/watch?v=7A65O-0lvKE&t=172s)

講者以單一內部 pain point 為單位, 描述自己的典型篩選流程:

```text
10 to 15 candidate startups
  -> 2 to 3 demo calls
  -> 0 or 1 pilot
  -> about 1 contract per 4 pilots
  -> roughly 5% of demos become signed contracts
```

這個 funnel 不是嚴格 cohort analysis. 各階段數字是範圍和近似值, 也沒有說明樣本數, 觀察期間或不同產品類型的差異. 它仍然揭示一項重要現實: demo conversion 不是 enterprise readiness, pilot 也不是採購承諾.

影片將失敗來源分為四類:

- Efficacy 和 commercial fit.
- Security.
- Reliability.
- Legal.

講者提到約 40% 與 efficacy 有關, 但沒有提供完整分類資料. 不應把這個比例當成適用所有企業採購的統計結果.

## Efficacy: 先證明解決問題

[04:16](https://www.youtube.com/watch?v=7A65O-0lvKE&t=256s)

買方的基本要求包括:

- Product 確實解決已定義的問題.
- Pricing model 與可觀察價值相符.
- Day one 就能展示真實 integration, 而不是 future promise.
- Success criteria 由客戶依業務結果定義, 不是 vendor 自行挑選有利指標.

### Vaporware 與 Build-versus-Buy

講者看過 startups 提案的功能, 內部 platform team 約 6 週就能重建. 這不代表企業一定選擇 build. 即使技術上可重建, vendor 若能降低維護, 支援和 time-to-value, buy 仍可能更合理.

真正的風險是把 demo prototype 當成 ready product. Vendor 若在兩個月後仍無法提出承諾功能的 ETA, 或反覆推銷客戶已明確拒絕的 feature, 代表需求理解與 delivery discipline 都不足.

### Upside-Down Pricing

一個 vendor 的 wrapper 使用客戶自己的 LLM gateway, model traffic 不經過 vendor infrastructure. Vendor 卻要求客戶回報 gateway telemetry, 再對這些 traffic 收取高額 margin.

這種定價把客戶自己承擔的 inference cost 和 infrastructure load 再次計價, 卻沒有相對應的運算或風險承擔. 合理 pricing 應清楚區分 software value, managed service, inference usage, support 和 enterprise controls.

## Pilot Window 正在縮短

[05:36](https://www.youtube.com/watch?v=7A65O-0lvKE&t=336s)

依講者兩年多的個人觀察, 常見 pilot 期間從約 6 個月縮短到 3 個月, 後來甚至可能只有 2 週. 原因是 AI 市場和產品變化很快, 企業不願用半年評估可能已經改版的 capability.

短 pilot 對雙方的要求更高:

- Pilot 前先完成 security 和 integration prerequisites.
- 使用 non-production data 和隔離環境.
- 預先約定 success criteria, baseline 和 measurement method.
- 在第一天就能存取 logs, admin APIs 和 support channel.
- 明確定義 pilot 結束時的 buy, extend 或 stop decision.

後面這份清單是依演講要求所做的編者整理, 不是影片中的正式採購 policy.

## Security: Enterprise Ready 的基本條件

[06:57](https://www.youtube.com/watch?v=7A65O-0lvKE&t=417s)

講者自稱不是 security specialist, 但負責許多 vendor security 對話的第一線篩選. 他列出的買方需求包括:

### Data Protection

- 優先要求 zero data retention, 簡稱 ZDR.
- 若無法 ZDR, 希望使用 customer-managed encryption keys.
- Encryption configuration 不能讓核心產品功能失效.
- 客戶資料不得用於 model training.

### Deployment 和 Network Control

- Bring your own gateway, 讓企業透過自己的 LLM gateway 路由 traffic.
- Bring your own infrastructure, 能部署到客戶管理的 cloud environment.
- 不應在未取得同意時把 pilot data 傳到 vendor cloud.

### Identity 和 Entitlements

- Role-based access control 與 SCIM 或 enterprise directory groups 整合.
- 不為全公司預設開啟新功能.
- Permissions 和 feature rollout 可由 API 管理.
- Integration 採 least privilege, 不要求對所有系統的 read/write all.

### Security Ownership

對小型公司, 買方至少希望看到一位真正負責 security 的 hire. 文件可以補足部分資訊, 但沒有 owner 的 security architecture 很難通過 incident, questionnaire 和 remediation 的持續協作.

## 重複出現的 Security 失敗

[08:19](https://www.youtube.com/watch?v=7A65O-0lvKE&t=499s)

講者列出的真實 vendor 經驗包括:

- Pilot data 直接送往 vendor cloud, 沒有遵守買方限制.
- Integration 只能以 read/write all scopes 運作.
- 每次 release 都預設開啟 beta features.
- Security architecture diagram 一週又一週延遲交付.
- 被問及 breach response 時, 只回答尚未發生 breach.

講者補充, 他們的 pilots 使用 non-production data, 因此 vendor 違反 data-routing expectation 時降低了實際暴露. 這也說明 pilot isolation 不是形式要求, 而是預防供應商控制失效的 containment boundary.

「尚未發生 breach」不能取代 incident response plan. Enterprise buyer 需要知道 detection, containment, notification, forensics, credential rotation, recovery 和 post-incident reporting 的責任與時限.

## Reliability: Control Plane 必須可以管理

[09:42](https://www.youtube.com/watch?v=7A65O-0lvKE&t=582s)

買方希望看到:

- 所有 admin settings 都能透過 API 管理.
- Configuration changes 有 audit logs.
- Enterprise 可以控制 release rollout.
- 有具體 SLA.
- 能聯絡到真正的 support engineer.
- 有可靠 status page.

這些能力讓企業可以自動化 deployment, 重現設定, 追查事故和分批發布. 只有 UI button 的 control plane 無法支援大規模環境.

### 快速發布造成版本碎片

講者描述某類 desktop apps 一天多次發布 update, 3,000 名使用者各自點擊 relaunch. Enterprise 無法知道誰在哪個版本, 也無法控制 rollout. 若新版同時破壞 SSL certificates, 問題就很難定位和回復.

更新速度本身不是可靠性. Vendor 需要 stable channels, version inventory, staged rollout, rollback 和 compatibility policy.

### Documentation 也需要 Versioning

有些 vendor 在 support page 加入新的 terms 或 risks, 卻沒有把它們納入 legal contract. 頁面只顯示「昨天更新」, 客戶無法知道變更前後內容.

Documentation 若影響 security, retention, subprocessors 或 supported behavior, 就是可稽核 control surface. 它需要 change history, effective date 和 contract hierarchy.

### 交易日的 API Outage

[11:04](https://www.youtube.com/watch?v=7A65O-0lvKE&t=664s)

講者曾遇到 core APIs 在繁忙交易日中斷數小時. 對管理數十億美元交易的 production system, 這不是一般 feature bug, 而是直接影響業務連續性的風險.

Enterprise readiness 因此必須包含 SLO, incident communication, degradation mode, recovery target 和 support escalation. Demo accuracy 無法抵消 production unavailability.

## Legal: 合約與產品行為必須一致

[12:23](https://www.youtube.com/watch?v=7A65O-0lvKE&t=743s)

講者列出的 legal requirements 包括:

- 不以任何 feature 或 product 的客戶資料訓練模型.
- 完整揭露 subprocessors.
- 管理 fourth-party risk.
- 提供合理的 IP indemnification 和 liability caps.

企業不控制底層 models. 若模型輸出侵害 intellectual property, 買方不希望獨自承擔全部責任. 合約需要說明 model provider, application vendor 和 customer 各自控制的部分.

### 宣稱 ZDR, 行為卻顯示保留資料

某 vendor 法律文件聲稱 ZDR, 後來卻主動提到客戶資料中的內容. 買方追問後, vendor 才承認確實看到了不應保存的資料.

這是一個比 policy wording 更強的驗證原則: 必須核對實際 data flow, logs, support access 和 deletion behavior. ZDR 應是可測試的 system property, 不是 sales label.

### 永久 Beta 作為 Retention Loophole

另一種模式是新功能長期只以 beta 發布, beta terms 則允許較寬鬆的 data retention. 使用者開啟新功能時, 可能在沒有清楚理解的情況下改變資料處理條件.

Enterprise 應要求 feature-level retention matrix, explicit opt-in 和一致的 contract coverage. Vendor 不應以 beta label 規避主要產品承諾.

### Fourth-Party Risk 不應藏在網頁

Vendor 使用的 subprocessor 還可能再依賴其他服務. 這些 fourth parties 的風險最後仍傳遞給企業. 若資訊只存在未版本化的隱蔽網頁, 買方無法持續追蹤供應鏈變化.

## 最好與最差的 Enterprise Vendor

[13:44](https://www.youtube.com/watch?v=7A65O-0lvKE&t=824s)

講者將兩端特徵對照如下:

| 較成熟的 vendor | 高風險 vendor |
| --- | --- |
| Security architecture 實際可用 | 沒有 security architecture diagram |
| Support engineer 能快速回應 | 找不到能處理問題的 engineer |
| 從早期就提供 admin API | 沒有 deployment control 和 audit logs |
| 有部署到客戶 cloud 的 90-day plan | Feature promises 沒有 ETA |
| 由客戶定義 success criteria | Salesmanship 優先於 product delivery |

90-day deployment plan 是講者提出的理想特徵, 不是所有 enterprise 的固定期限. 關鍵是 vendor 能提出具 owner, milestones, dependencies 和 risks 的落地路徑.

## 新 Models 無法修補舊 Foundations

[13:44](https://www.youtube.com/watch?v=7A65O-0lvKE&t=824s)

講者指出, frontier models 發布速度可能以天或週計算, enterprise architecture 卻可能已有十年以上歷史. ERP migration 也可能持續多年. Model cycle 與 organizational change cycle 因此極不對稱.

他形容 AI 是 flashlight, 不是 band-aid:

- 基礎良好時, AI 能加速既有優勢.
- Integration, permissions 或 data quality 不佳時, AI 會更快暴露問題.
- Model capability 不能自行完成 architecture modernization 和 change management.

影片稱平均每 11 天出現一個 frontier model, 但沒有提供計算範圍或來源. 這個數字適合表達更新頻率很高, 不宜作為精確產業統計.

## 「無趣的 60%」是方向性假設

[15:07](https://www.youtube.com/watch?v=7A65O-0lvKE&t=907s)

講者用一個刻意簡化的分配表達優先順序:

- 40% 是 AI models 和 products.
- 60% 是 data hygiene, clean architecture, integration, enablement 和 change management.

他明確表示這些比例不具科學性. 真正主張是, 成為 AI-native organization 至少一半工作不屬於模型本身. 讀者應保留「先改善基礎」的決策原則, 不應引用 `60%` 作 ROI forecast.

## Agents 會放大 Entitlement 問題

[16:28](https://www.youtube.com/watch?v=7A65O-0lvKE&t=988s)

大型企業常同時存在 over-entitled 和 under-entitled users. Agents 能高速跨系統操作並運用判斷, 若直接繼承這套不精確權限, 風險不只是複製人類問題, 而是放大其速度和範圍.

講者認為 enterprise foundations 需要調整:

- Entitlements 採用更細緻, 可撤銷和可觀察的模式.
- Cross-platform integration 提升為核心 platform capability.
- Documentation 和 support knowledge 成為 agent-readable centralized knowledge.
- Legacy environment 與 experiment ecosystem 必要時隔離.
- Thin agents 依賴更可靠的 shared substrate.

實務上, agent identity 不應長期借用人類的 broad credentials. 應使用 task-scoped permissions, short-lived tokens, approval gates, tool allowlists 和完整 action logs. 這些控制方式是依講者 entitlement 原則所做的編者延伸.

## 給 Seller 與 Buyer 的決策清單

[17:54](https://www.youtube.com/watch?v=7A65O-0lvKE&t=1074s)

### Seller 應先準備

- 以客戶定義的 outcome 證明 efficacy.
- 展示已能運作的 integration.
- 讓 pricing 對應 vendor 實際提供的價值與成本.
- 提供 security architecture, data-flow diagram 和 incident response plan.
- 支援 least privilege, SCIM, RBAC, admin API 和 audit logs.
- 明確承諾 retention, training use 和 subprocessors.
- 提供 SLA, status page, escalation path 和 rollout controls.
- 準備部署到客戶 gateway 或 cloud 的可執行計畫.

### Buyer 應先修復

- 清理 identity, entitlements 和 access review.
- 建立可管理的 model gateway 和 integration layer.
- 整理 data quality 和 agent-readable knowledge.
- 使用 non-production data 與隔離環境執行 pilots.
- 由內部定義 success criteria 和 stop conditions.
- 驗證產品行為是否符合 security 與 legal claims.
- 建立 release governance, audit logging 和 incident ownership.
- 不把新的 AI product 當成 legacy architecture 的替代修復方案.

## 核心結論

Enterprise AI contract 的勝負通常不在 demo 最炫目的時刻, 而在 vendor 是否願意承擔 production outcome. 買方要的不是一個能偶爾完成任務的 model wrapper, 而是一個可部署, 可限制, 可觀察, 可回復及可透過合約追究的系統.

對 startup 而言, security, admin APIs, audit logs, support 和 legal transparency 不是成交後再補的 enterprise features, 而是產品架構的一部分. 對 buyer 而言, 採購更強模型也不會自動修正 entitlements, integrations 和 change management. Agents 最終會繼承雙方已經建立的基礎.

## 來源與信心限制

- 本筆記依據 YouTube 英文自動字幕整理, 不是逐字稿.
- 講者表示內容僅代表個人, 且因公司 compliance 限制而刻意保持案例匿名和一般化.
- 採購 funnel, 5% demo conversion, pilot duration 和 6-week internal rebuild 都是講者個人經驗的近似值. 影片沒有提供樣本數或原始資料.
- 「每 11 天一個 frontier model」和外部 conversion benchmark 沒有在影片中附上可核對來源.
- 40% models 與 60% foundations 是講者明確標為 unscientific 的方向性假設.
- 匿名案例無法由本筆記獨立驗證, 但講者宣稱全部來自實際經驗.
- 部分 security, pilot 和 agent-control 清單標示為編者整理, 不代表 Millennium 的正式 policy.
