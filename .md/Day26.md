# Day26 探討生產環境的維運：Multi-LoRA 部署架構與 Prometheus + Grafana 監控

當我們的 Agent 完成了前幾天的微調與訓練後，我們不能只靠簡單的腳本跑模型，更不能只用肉眼盯著 Terminal 抓 Bug。

今天，我們不寫繁瑣的實驗程式碼，而是從架構設計的視角，探討現代 LLM 部署的兩大神器：Multi-LoRA 部署策略，以及 Prometheus + Grafana 的生產級監控體系。


### 為什麼需要 Multi-LoRA 部署？
在 Agent 的應用場景中，通常希望模型具備多種能力（例如：規劃、寫程式、API 呼叫）。如果把所有知識都塞進同一個模型進行全參數微調，不僅成本高昂，還容易發生「災難性遺忘」。

Multi-LoRA 部署架構解決了這些痛點：
- **架構拆分**： 記憶體中只需要載入「一個」龐大的 Base Model，但在推理時，可以根據不同的 Request 動態掛載不同的 LoRA 權重（Adapter）。
- **極致的延展性與成本控管**： 假設有「負責聊天的 Agent A」與「負責查資料庫的 Agent B」，可以同時伺服這兩個 Agent，而不需要部署兩顆巨大的完整模型，大幅節省 VRAM。
- **無痛的版本控制（Versioning）**： 當業務邏輯改變，只需要訓練並抽換幾 MB 的新版 LoRA 權重即可。如果新版上線出問題，也能在毫秒級別切換回舊版 LoRA。

---
### 工具: Prometheus + Grafana
在實驗室裡，我們用 TensorBoard 看 Loss 曲線；但在生產環境，我們需要的是 Prometheus 採集即時數據，並用 Grafana 打造絢麗且實用的監控儀表板。
- Prometheus: 採集即時數據
- Grafana: 監控儀表板

#### 監控指標：
#### A. 系統與推論效能指標 (Inference Metrics)
- **TTFT (Time To First Token)**： 系統吐出第一個字的時間。
    - 如果過高，代表系統負載太重或 KV Cache 命中率低。
- **TPOT (Time Per Output Token) / TPS (Tokens Per Second)**： 生成每個 Token 的速度，反映了 GPU 的吞吐量。
- **LoRA 切換延遲 (Adapter Swap Latency)**： 在 Multi-LoRA 架構下，動態掛載不同權重所消耗的時間。
#### B. Agent 行為與業務指標 (Agent Behavior Metrics)
- **工具呼叫失敗率 (Tool Call Error Rate)**： 統計 Agent 生成的 JSON 格式錯誤，或是呼叫外部 API 超時的頻率。
- **Token 消耗成本統計**： 追蹤 Input / Output Token 的總量，甚至可以細分到「哪一個 LoRA Adapter」消耗了最多的運算資源，藉此抓出「過度思考」的異常請求。
#### C. 警告機制 (Alerting Rules)
透過 Prometheus 的 Alertmanager，我們可以設定防線。例如：「當某個 API 工具連續 5 次被 Agent 呼叫失敗」，或是「GPU VRAM 使用率持續 3 分鐘超過 95%」時，自動發送 Slack 或信件通知維運團隊。


---
### 架構總結：可維護性的終極型態
將 Multi-LoRA 與 Prometheus + Grafana 結合，我們得到了一個極具彈性且透明的系統。

當使用者回報「今天的 Agent 回答變笨了」，我們不再需要翻遍整個 Log。我們可以打開 Grafana，確認是不是某個 LoRA Adapter 發生了推論延遲，或是特定的 Prompt 觸發了極高的失敗率；接著，我們可以透過 Multi-LoRA 架構，瞬間降級（Rollback）到昨天穩定的版本，甚至完全不需中斷服務。

---
Summary:
一個真正具備商業價值的 AI 應用，不僅要「夠聰明」，更要「好維護、好除錯、好擴展」。掌握了這套工業級的部署與監控架構，系統才能在真實世界中從容應對未來的流量挑戰與頻繁的業務迭代。

---
明天我會用Streamlit 模擬Grafana 視覺化!
