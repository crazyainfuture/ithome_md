# Day22 如何定義 Agent 成功？Evaluation Framework 與 Metrics 評估設計

前天訓練好 DPO model，今天拿 Day16 寫好的評估集做測試，並與 Day17 的baseline比較!

### 我先定義了多項關鍵評估指標來檢驗模型表現： 
- Success Rate：檢查 Agent 的最終答案是否包含預期的關鍵字。 
- Tool Accuracy：評估 Agent 使用的工具序列是否與預期完全相符。 
- Invalid Tool Calls：透過捕捉 JSON 解析錯誤，計算模型發生幻覺或格式錯誤的次數。 
- Average Steps：記錄模型完成任務所需的平均步驟數。 

根據實驗結果，SFT與DPO 實驗結果比較
|評估指標|SFT Baseline|DPO Agent|
|------|-------------|---------|
|Success Rate|50.00%|73.33%|
|Tool Accuracy|	0.00%|73.33%|
|Avg Invalid Calls|0|0.23|
|Avg Steps|0.00|3.633|

---
### 實驗結果分析：SFT Baseline vs. DPO Agent
本次實驗基於 Llama-3.1-8B-Instruct 模型，比較了純 SFT（監督式微調）與經過 DPO（直接偏好最佳化）微調後的 Agent 表現差異。從評估數據中，我們可以觀察到模型行為產生了本質上的轉變：

- 任務成功率與工具準確率大幅雙升： SFT Baseline 雖然擁有 50.00% 的 Success Rate，但其 Tool Accuracy 卻是 0.00%。這意味著 SFT 模型很可能只是單純依賴自身預訓練的知識「盲猜」或直接生成答案，並沒有真正學會如何正確呼叫工具。 相對地，經過 DPO 微調後，Agent 的 Success Rate 提升至 73.33%，且 Tool Accuracy 更迎來了質的飛躍，同樣達到 73.33%。這證明 DPO 成功教會了模型對齊預期的工具使用邏輯，讓它能精準依照步驟使用工具來解決問題。

- 代理行為（Agentic Behavior）的真正成型： 在 SFT 階段，Avg Steps 為 0.00，顯示模型未能展開任何有效的工具呼叫循環；而 DPO Agent 的 Avg Steps 來到 3.633 步，明確展現了 Agent 拆解任務、多步驟推理與行動的能力。

- 輸出格式控制穩定： 儘管 DPO Agent 開始大量且連續地呼叫工具，其格式或幻覺錯誤（Avg Invalid Calls）僅微幅上升至 0.23 次/題。這顯示模型在遵循 JSON 格式輸出的規範上，依然保持著高度的穩定性，沒有因為任務變複雜而崩潰。

這份成績單直觀地展示了 DPO 的威力。它成功激活了模型的 Agent 潛能，讓模型從一個「只會直接回答的對話助手（SFT）」，真正蛻變成「懂得自主運用工具解決複雜問題的 AI Agent」。

---
明天會詳細整理 DPO 的流程~


