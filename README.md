# AI Agent 與軟體工程影片筆記

這個 repository 使用 codex skill 收錄知識型 YouTube 影片的繁體中文學習筆記, 主題涵蓋 AI coding、agent engineering、Claude Code、Codex、MCP、skills、軟體工程治理及高風險 AI 應用。

這些文件不是逐字稿。每份筆記會保留影片的重要論點、案例、方法與限制, 再重新整理成適合閱讀、複習及實作的 Markdown 內容。點選下方標題即可開啟對應筆記。

## 筆記如何製作

製作流程以來源可追溯性為優先:

1. 核對影片 ID、標題、頻道、發布日期、片長與字幕狀態。
2. 優先採用創作者字幕, 其次使用原語自動字幕。沒有可驗證內容來源時, 不根據標題自行補寫影片觀點。
3. 清除字幕中的重複片段、語助詞、寒暄與無關宣傳。
4. 依影片的論證順序重新組織內容, 保留關鍵案例、條件、反例與講者保留意見。
5. 為重要定義、示範與主張加入可返回原影片的時間戳連結。
6. 將影片內容與編輯補充分開, 對收音不清、字幕誤辨或缺乏原始引文的部分明確標示限制。
7. 完成後核對影片連結、Video ID、metadata 與筆記中的主要主張。

自動字幕可能誤辨人名、產品名稱與技術詞彙。筆記只在上下文足夠明確時修正, 並在各文件末尾說明來源與可信度限制。產品功能、API、模型版本及政策可能在影片發布後改變, 實際使用時應再查閱最新官方文件。

## 建議閱讀路徑

如果你第一次接觸這些主題, 可以依下列順序閱讀:

```text
軟體工程基本功
  -> AI coding 工作流
  -> Agent 與 harness
  -> MCP、skills 與外部工具
  -> Agent memory 與 multimodal retrieval
  -> 長時程任務與多 agent 協作
  -> 組織治理與高風險應用
```

## 重要文章標記

使用 `★` 標記值得優先閱讀、定期重讀或長期保留的文章。這是個人知識優先級, 不代表影片內容已經過獨立驗證, 也不是品質評分。

建議在文章符合下列任一條件時加上標記:

- 明顯改變對某個主題的理解框架。
- 對目前工作或未來決策具有反覆參考價值。
- 提供其他筆記沒有的 production 經驗、反例或方法。
- 適合作為該主題的入口或核心閱讀材料。

標記格式固定放在項目開頭:

```text
- ★ | [文章標題](實際筆記檔名)
```

只使用「重要」與「未標記」兩種狀態, 不採用多星評分。當文章不再符合優先級時, 直接移除標記即可。

## 全部筆記

### AI coding 與軟體工程方法

- [Claude Code 與 Agentic Coding 的演進](2025-07-04-claude-code-agentic-coding-evolution-Lue8K2jqfKk.md)  
  從程式設計介面的歷史切入, 解釋自然語言 coding agent 的產品定位, 以及探索、規劃、TDD、context 與平行 agents 的實務方法。

- [`#define AI Engineer`: Greg Brockman 談工程、研究與 AI 開發的未來](2025-08-10-define-ai-engineer-avWhreBUYF0.md)
  從獨立學習與 Stripe 經歷談到 AI 工程和研究的協作, 並探討 vibe coding、Codex、long-horizon agents、多模型工作流與未來 AI 基礎設施。

- [Anthropic Claude Prompt Workshop: 從失敗案例迭代提示詞](2024-08-17-anthropic-claude-prompt-workshop-hkhDdcM5V94.md)  
  以真實失敗案例說明 prompt 如何進行回歸測試, 並探討指令結構、範例、格式驗證與確定性程式的分工。

- [Claude Code 如何運作: Coding Agent 的簡單迴圈、Tools 與 Context Engineering](2025-12-26-how-claude-code-works-RFKCzGlAU6Q.md)
  從外部研究與實際使用經驗拆解 Claude Code, 並比較 coding agents 的簡單 loop、通用 tools、context management、subagents 與安全設計。

- [Research-Plan-Implement 哪裡做錯了: 從 Magic Words 到可控工作流](2026-03-24-everything-wrong-about-research-plan-implement-YwZR6tc7qYg.md)
  回顧 RPI 的團隊採用問題, 並以 instruction budget、分離 context、design discussion、vertical plans 與 code ownership 建立更可靠的 agentic coding 流程。

- [Agentic Engineering: 與 AI 一起工作, 不只是使用 AI](2026-04-07-agentic-engineering-working-with-ai-BEKc4P87XKo.md)
  將 coding agent 視為需要 context、分工與審查的協作者, 並以 research、plan、implement、Git diff 與權限管理建立可靠工作流。

- [Harness Engineering: 當人類掌舵、Agents 執行時, 如何建造軟體](2026-04-17-harness-engineering-am_oeAoUhew.md)  
  說明當程式碼生成不再稀缺時, 團隊應如何建立 tickets、文件、測試、觀測能力與 guardrails, 讓 agents 能長時間可靠工作。

- [AI 時代, 軟體工程基本功比以往更重要](2026-04-23-software-fundamentals-v4F1gFy-hqg.md)  
  探討 AI 為何會放大 codebase 的優點與缺陷, 以及系統設計、模組邊界、測試與持續重構為何更加重要。

- [AI Coding 完整工作流: 從需求對齊到代理實作與 QA](2026-04-24-ai-coding-workflow--QFHIoCo-Ko.md)  
  完整示範研究、原型、需求問答、PRD、issue 拆解、代理實作、人工 QA、部署與監控的端到端流程。

- [從 Vibe Coding 到 Agentic Engineering](2026-04-29-vibe-coding-to-agentic-engineering.md)  
  整理 Andrej Karpathy 對 Software 3.0 的觀察, 並說明工程師如何轉向規格、驗證、代理協調與品質責任。

- [Coding Is Solved 之後, 軟體開發會走向哪裡](2026-05-04-coding-is-solved-SlGRN8jh2RI.md)  
  討論程式碼生產成本下降後, 領域理解、產品品質、組織流程、資料與多代理協調如何成為新的差異來源。

- ★ | [軟體開發的舊假設已經改變: 從做得更深到想得更廣](2026-07-08-everything-we-knew-about-software-has-changed-xUnRQ9vLXxo.md)
  從模型的 tool calling、長時程工作與 orchestration 演進, 重新檢查工程工具慣性、Markdown tier、專案規模與產品 breadth 的舊假設。

- [未來工程師: 選擇值得做的事, 並為結果負責](2026-07-14-engineer-of-the-future-n97BCfyFIvw.md)
  說明 agents 普及後, 工程師的價值如何從程式碼產量轉向問題選擇、證據判讀、production verdict、高 agency 與結果責任。

- [Harness Engineering 還不夠: 為什麼軟體工廠會失敗](2026-07-23-harness-engineering-is-not-enough-Ib5GBkD555M.md)
  說明 agent harness 為何無法單獨解決可維護性問題, 並從軟體工廠、coding agent 訓練與長時程 benchmark 探討先規劃再生成的工程方法。

- ★ | [Claude Code 入門課程: 安裝、Goals、Skills、GitHub、MCP 與部署](2026-08-05-claude-code-course-7l6bXLAKyEI.md)
  面向初學者的完整操作課程, 從本機設定、session 和權限開始, 一路涵蓋 goals、skills、GitHub、MCP 與部署。

- ★ | [Anthropic CCA Exam: Agentic Engineering 的實戰指南](2026-08-08-cca-agentic-engineering-Z-c11pV_uvU.md)
  從認證考試的 production scenarios 提煉 agent loop、stop reason、工具執行、context 隔離、sub-agents 與 CI 的常見 anti-patterns。

- ★ | [Anthropic 如何使用 Claude Code: 大規模 Agentic Software Engineering](2026-08-11-agentic-software-engineering-shZgedW15vg.md)
  說明大型組織如何為 coding agents 提供 access、knowledge 與快速 feedback loops, 並在有限 context 中擴展客製化能力。

- [AI 生成程式碼的信任問題: Code Review、Context 與治理層](2026-08-20-ai-code-review-context-governance-s-aixZYJG4c.md)  
  分析 AI 提高程式碼產量後的驗證瓶頸, 以及如何把架構、服務契約、事故經驗與審查決策轉成可稽核的治理 context。

### Agent 架構與長時程工作

- [如何打造有效的 AI Agents](2025-04-04-building-effective-agents-D7_ipDqhtwk.md)
  Anthropic 從任務複雜度、價值、關鍵能力與錯誤風險說明何時適合使用 agent, 並以 environment、tools 和 system prompt 建立最小可行架構。

- [12-Factor Agents: 建立可靠 LLM 應用的軟體工程 Patterns](2025-07-03-12-factor-agents-8kMaTybvDUw.md)
  將 production agent 拆成 deterministic software 與短小 LLM loops, 並說明 prompts、context、state、control flow、human interaction 與 framework 的工程取捨。

- [為 Agent 演進 Claude API](2025-12-04-evolving-claude-apis-for-agents-aqW68Is_Kj4.md)  
  從 harness capabilities、context management 與安全執行環境三層, 說明 API 如何承接模型能力並支援高效能 agents。

- [Claude Agent SDK 完整工作坊](2026-01-05-claude-agent-sdk-workshop-TqC1qOfiVcQ.md)  
  透過觀念與 live coding 介紹 tools、filesystem、Bash、skills、subagents、compaction、hooks、permissions、sandboxing 與 production hosting。

- [State of the Claw: OpenClaw 的成長、安全與個人 Agent 願景](2026-04-17-state-of-the-claw-zgNvts_2TUE.md)
  回顧 OpenClaw 的治理與安全壓力, 並討論資料主權、prompt injection、agent personality、dreaming、模組化及 AI 時代的工程 taste。

- ★ | [AI Agent Harness 深入解析, 用確定性工程約束非確定性模型](2026-05-17-ai-agent-harnesses-deep-dive-C_GG5g38vLU.md)
  從 browser agent 的失敗案例逐步加入 guardrails、context management、trace verification 與 deterministic login handler, 說明 harness 如何提高舊模型的可靠性。

- ★ | [建立能連續執行數小時的 Agent](2026-05-18-build-agents-that-run-for-hours-mR-WAvEPRwE.md)
  以 planner、generator 與 adversarial evaluator 拆分長時間工作, 並說明完成契約、rubric、trace debugging、持久化狀態及 harness 演進。

- [Claude 長時程任務: 非同步 Agent、Verifier 與自我學習記憶](2026-07-22-claude-long-horizon-tasks-9QebvrrY3KY.md)  
  解釋模型 task horizon 變長後, 為何需要持久 session、獨立 verifier、可修正記憶、背景執行與主動通知。

- ★ | [Anthropic Applied AI: Agentic Surfaces 的演進與 Managed Agents 架構](2026-08-11-anthropic-agentic-surfaces-K0X9QDRkIdg.md)
  從問題、任務到成果的產品演進, 介紹 agent、environment、session 及「腦與手分離」的 managed agents 架構。

### MCP、Skills 與工具生態

- [使用 Model Context Protocol 建立 Agents](2025-03-01-building-agents-with-mcp-workshop-kQmXtrmQ5Zg.md)  
  MCP 完整工作坊, 涵蓋 tools、resources、prompts、server 建置、agent framework 整合、sampling、OAuth 與 remote servers。

- [MCP 的起源、設計取捨與創業機會](2025-06-18-mcp-origins-startup-opportunities-x-8pBqWiTzk.md)  
  回顧 MCP 為何從複製外部 context 的問題誕生, 並討論開放協定、model agency、工具介面與生態系機會。

- [不要重建 Agent, 改為建立 Skills](2025-12-08-build-skills-instead-of-agents-CEvIs9y1uog.md)  
  主張通用 agent harness 已逐漸成熟, 差異化應放在封裝程序知識的 skills, 並解釋 skills 與 MCP 的互補關係。

- [建立優秀 Agent Skills 的實作手冊](2026-06-29-building-great-agent-skills-UNzCG3lw6O0.md)
  以 trigger、structure、steering 與 pruning 建立 skill review checklist, 並說明 invocation trade-offs、context pointers、leading words、legwork 與 deletion tests。

- [Web Automation 的進階方法: 讓代理像人類一樣操作網站](2026-08-14-web-automation-agents-26RtyAm9y_Q.md)
  以 CLI、Chrome DevTools Protocol 與 sense-act-verify 迴圈建立可重用的瀏覽器代理, 並說明確定性程式與視覺模型的分工及安全邊界。

### Agent Memory 與 Multimodal Retrieval

- [設計 Agent Memory, 原則、Patterns 與實務方法](2025-06-27-architecting-agent-memory-W2HVdB4Jbjs.md)
  將 persona、conversation、entity、workflow、episodic 與 toolbox memory 放入完整 lifecycle, 並整理 retrieval、ranking、forgetting、治理與 production architecture。

- [從零建立 Multimodal AI Agent](2025-06-27-building-multimodal-ai-agents-640KMYtxCeI.md)
  以 page-as-image pipeline 建立 mixed-media RAG agent, 涵蓋 multimodal embeddings、vector search、跨頁 context、tool loop 與 session memory。

### LLM Inference 與部署基礎設施

- [掌握 LLM Inference 最佳化, 從原理到具成本效益的部署](2025-01-01-llm-inference-optimization-9tvJ_GYJA-o.md)
  從 tokenization、attention 與 KV cache 說明 LLM serving, 並以 TTFT、ITL、query patterns、quantization、batching 與 parallelism 規劃效能及成本。

- ★ | [KV Cache 與 Paged Attention 如何加速 LLM Inference](2026-06-30-kv-cache-paged-attention-o0gkdZBtwEg.md)
  解釋 prefill 與 decode 的資源差異, 以及 paged attention、prefix caching、chunked prefill 和 speculative decoding 如何改善 VRAM 使用與延遲。

- ★ | [Kernels、強化學習與 Agent Reward Hacking 進階研討](2026-07-17-kernels-rl-reward-hacking-agents-uIiA6DquRiE.md)
  從模型趨勢、開閉源差距與 inference provider 延伸到 benchmark verifier、kernel 最佳化、reinforcement learning 與 agent reward hacking。

### Codex 個人工作系統

- [OpenAI Codex Masterclass: 從程式碼代理到軟體工程系統](2026-04-29-openai-codex-masterclass-MhHEGMFCEB0.md)
  系統介紹 Codex 的模型與 harness、plugins、automations、程式碼審查、subagents、Guardian approvals、hooks 及雲端工作方式。

- [用 Codex 建立可長期運作的個人工作系統](2026-07-24-codex-setting-yourself-up-for-success-il1c1a2FufU.md)  
  介紹 personal monorepo、記憶、appshots、plugins、pinned threads、heartbeat、goals、thread 協作、computer use 與安全界線。

- ★ | [Codex Behind the Harness: 從 Context、Tools 到 Long-running Loops](2026-08-10-codex-behind-the-harness-shRR1e2HXMk.md)
  拆解 app server、Responses API、deferred tools、非同步 actions、sandbox、auto review、WebSocket、goal loop 與 server-side compaction。

### 組織落地與專業領域

- [每家公司都應該有一個 Brain](2026-07-17-every-company-should-have-a-brain-eBUyTS7SzV4.md)
  將 skills、resolver、evals 與 company brain 對應到 AI-native organization, 並討論 library、librarian、知識衛生與可重複工作的組織複利。

- ★ | [Forward Deployed Engineering 101](2026-07-28-forward-deployed-engineering-101-KwhgfwOSToQ.md)
  說明 FDE 如何在可重用平台上與客戶共同交付商業成果, 以及公司在採用這種 go-to-market 模式前應具備的條件。

- [企業如何擁有自己的 Intelligence, 從租用模型到掌控 Weights](2026-08-11-own-your-intelligence-bMMv0bZzONg.md)
  以 cost、speed、performance 與 control 評估 AI capability 的 own-vs-rent 邊界, 並整理團隊、legibility、evals、post-training 與 data flywheel 路線。

- [為什麼企業技術堆疊尚未準備好迎接 AI 代理](2026-08-19-enterprise-tech-stack-ai-agents-mav15aW9lLM.md)
  以醫療產業為例, 說明如何用不可變事件紀錄、敏感資料隔離、人機等價操作與可重播評估, 建立符合企業治理要求的代理架構。

- [從 Ambient Documentation 到 Clinical Intelligence](2026-08-19-clinical-intelligence-abridge-u6q-byPWUuo.md)  
  介紹 Abridge 如何從臨床文件切入醫療 AI, 再以評估、專家 rubric、小模型與事件式路由擴展到 clinical intelligence。

- [會員端醫療 AI 的 Guardrails: 架構、持續評估與上線決策](2026-08-19-health-ai-guardrails-YXEqC05WEI0.md)  
  聚焦高風險醫療 AI, 說明確定性規則、PHI 邊界、持續評估、人工判讀與分階段上線如何共同構成安全架構。

## Repository 使用方式

- 在 GitHub 上直接點選上述筆記標題閱讀。
- 每份文件開頭列出來源影片與 metadata。
- 筆記中的時間戳會連回 YouTube 對應段落。
- 可使用 GitHub repository 搜尋功能查找 `MCP`、`context`、`skills`、`guardrails` 等關鍵字。
- 建議先閱讀摘要, 再依需要進入技術章節與原影片驗證細節。

## 使用與引用

這些內容是對公開影片的編輯式學習筆記, 著作權仍屬原講者與影片發布者。引用重要觀點時, 請優先連結原始影片, 並回到對應時間戳確認完整語境。筆記不應被視為原講者的逐字陳述、正式文件或產品政策。
