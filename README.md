# 30 天打造 Agentic RL：從強化學習到 AgentOps

這是一系列參加 iThome 鐵人賽的技術文章，從 AI Agent 為什麼需要強化學習（Reinforcement Learning, RL）開始，逐步介紹 RL、RLHF、PPO、DPO、工具呼叫、模型微調，以及生產環境中的 AgentOps 與持續對齊流程。

這 30 天不只整理理論，也記錄實際實驗、Reward 設計、Reward Hacking、訓練不穩定、Context 管理與部署監控等過程，希望用一條完整路線理解如何讓 Agent 從「能回答問題」走向「能完成任務，並從錯誤中持續改善」。

## 系列重點

- 理解 AI Agent 與一般 LLM 的差異
- 建立 MDP、Reward、Policy、Policy Gradient 與 PPO 的 RL 基礎
- 拆解 RLHF、Reward Model、DPO 與 GRPO
- 實作工具呼叫環境、評估資料集與 Agent baseline
- 使用 TRL 進行 Reward 設計與 DPO 微調
- 分析 Reward Hacking、KL Explosion 與模型 Collapse
- 建立 API、監控、Tracing、人工除錯與資料飛輪
- 思考 Agent 從實驗走向生產環境時的工程取捨

## 文章目錄

### Part 1：RL 與 Agent 基礎

1. [Day 1：為什麼 Agent 需要 RL？從 0 到 Agentic RL 實作的起點](.md/Day1.md)
2. [Day 2：掌握 RL 基礎：什麼是 MDP、Reward、Policy？](.md/Day2.md)
3. [Day 3：Value-based：為什麼 ChatGPT 不能直接用 Q-learning？](.md/Day3.md)
4. [Day 4：Policy Gradient 演進：Actor-Critic 與 OpenAI 選擇 PPO 的關鍵](.md/Day4.md)
5. [Day 5：避免模型崩潰的秘密：PPO Clip 機制與 KL Penalty 解析](.md/Day5.md)
6. [Day 6：邁向 LLM 時代：Clip 大小實驗與第一階段 RL 脈絡總結](.md/Day6.md)

### Part 2：RLHF、DPO 與獎勵設計

7. [Day 7：拆解 RLHF 全流程：SFT、Reward Model 到 PPO 的 Pipeline](.md/Day7.md)
8. [Day 8：人工標註的陷阱：Reward Model 訓練實戰與資料品質探討](.md/Day8.md)
9. [Day 9：DPO vs PPO：為什麼 DPO 不需要 Reward Model 且更適合微調？](.md/Day9.md)
10. [Day 10：DeepSeek 背後的演算法：GRPO 與 PPO 差異比較](.md/Day10.md)
11. [Day 11：Agent RL 與傳統 RLHF 的本質差異及 Credit Assignment 難題](.md/Day11.md)
12. [Day 12：獎勵機制的抉擇：Outcome vs Process Reward Model 與可驗證獎勵](.md/Day12.md)
13. [Day 13：成功的一半：為什麼對 Agent 來說「環境設計」比演算法更重要？](.md/Day13.md)
14. [Day 14：當 Agent 開始作弊：故意設計錯誤 Reward 誘發 Reward Hacking](.md/Day14.md)
15. [Day 15：自我進化的開端：Self-play 迴圈與 Multi-Agent 協作入門](.md/Day15.md)

### Part 3：Agent 實作與模型微調

16. [Day 16：打造 Agent 競技場：手刻工具呼叫環境與評估資料集建立](.md/Day16.md)
17. [Day 17：建立 Baseline 基準點：純 SFT 版本 Agent 的任務表現與極限](.md/Day17.md)
18. [Day 18：TRL 訓練實戰：Reward 第一版設計與被 Hack 全紀錄](.md/Day18.md)
19. [Day 19：Reward 第二版修正](.md/Day19.md)
20. [Day 20：使用 TRL DPOTrainer 從日誌生成資料微調 Agent](.md/Day20.md)
21. [Day 21：RL 訓練血淚史：不穩定現象（KL Explosion、Collapse）踩坑](.md/Day21.md)
22. [Day 22：如何定義 Agent 成功？Evaluation Framework 與 Metrics 評估設計](.md/Day22.md)
23. [Day 23：完整 DPO Pipeline 復盤](.md/Day23.md)
24. [Day 24：邁向生產環境：API 封裝、Serving 延遲探討與量化取捨](.md/Day24.md)
25. [Day 25：解決 Token 爆炸：多步驟任務下的 Context 管理與裁剪策略](.md/Day25.md)

### Part 4：AgentOps 與持續對齊

26. [Day 26：探討生產環境的維運：Multi-LoRA 部署架構與 Prometheus + Grafana 監控](.md/Day26.md)
27. [Day 27：用 Streamlit 模擬 Grafana 視覺化：建構 Agent Controller 迴圈與 API 批次測試](.md/Day27.md)
28. [Day 28：擴充 Streamlit 實作軌跡檢視器與人工除錯介面](.md/Day28.md)
29. [Day 29：打造資料飛輪：收集 Bad Case、模擬 RLAIF 與持續對齊（Continuous DPO）](.md/Day29.md)
30. [Day 30：從 SFT、RLHF 到 MLOps 完整旅程總結與未來展望](.md/Day30.md)

## 建議閱讀方式

- 想快速了解 Agent 為什麼需要 RL：從 Day 1 開始。
- 想補足 RL 與 PPO 基礎：閱讀 Day 2–6。
- 想理解 RLHF、DPO 與 GRPO：閱讀 Day 7–15。
- 想看實作、訓練與評估：閱讀 Day 16–25。
- 想了解部署、監控與持續改善：閱讀 Day 26–30。

## 專案結構

```text
.
├── README.md
└── .md/
	├── Day1.md
	├── Day2.md
	├── ...
	└── Day30.md
```

## 關鍵字

`AI Agent` `Agentic RL` `Reinforcement Learning` `RLHF` `PPO` `DPO` `GRPO` `TRL` `Reward Hacking` `AgentOps` `MLOps`
