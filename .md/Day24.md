# Day24 邁向生產環境：API 封裝、Serving 延遲探討與量化取捨

在前幾天的實作中，我們成功用 DPO 訓練出了具備自主使用工具能力的 Agent（Success Rate 提升至 73%）。但一個存在於命令列裡的模型，還不能算是一個「產品」。

今天我要完成 AI Agent 開發的最後一哩路：把訓練好的模型包裝成 API 服務，並探討推論延遲、量化技術，以及「訓練完的模型」與「正式上線服務」之間的工程落差。

### 使用FastAPI打包模型
#### 透過指令
> curl -X POST "http://127.0.0.1:8000/invoke_agent" -H "Content-Type: application/json" -d '{"observation": "請幫我搜尋 Llama 3 的發布日期並計算距離今天幾天？"}'

#### 模型回傳他的下一步
> {"status":"success","latency_seconds":0.85,"action":[{"action":"search","action_input":"Llama 3 發布日期"},"Thought: 首先，我需要知道 Llama 3 的發布日期，然後才能計算距離今天幾天。\n\n<tool_call>{\"action\": \"search\", \"action_input\": \"Llama 3 發布日期\"}</tool_call>"]}

也就是目前是單步推論，如果要讓它變成自動完成任務的完整服務，需要在 FastAPI 裡面寫一個 while 迴圈。

---
### 討論 Agent 的推論延遲與 Batching 挑戰
當我們把 API 架起來自己玩的時候很順，但如果同時有 10 個使用者發送請求呢？

這就牽涉到了推論延遲（Latency）與批次處理（Batching）的問題。

#### 單次呼叫要多久（Latency）？
對於一般的 Chatbot，使用者問一句，模型回一句（約 1~2 秒）。

但 Agent 不一樣！ 一個 Agent 任務可能包含 `思考 -> 呼叫 Search -> 總結 -> 呼叫 Calculator -> 輸出最終答案`。這意味著完成一個任務，背後其實觸發了 3 到 5 次的 LLM 推論。這會讓單一任務的整體**延遲飆升到 5~10 秒**以上。

#### 能不能 Batching？
在 FastAPI 搭配原生 Hugging Face transformers 的架構下，如果兩個請求同時進來，後面的請求必須「**排隊**」等第一個請求生成完畢（因為 generate 是阻塞的）。

為了解決這個問題，業界不會直接用 transformers 上線，而是會改用 **vLLM** 或 **TGI** 等框架。這些框架支援 **Continuous Batching (連續批次處理)** 與 **PagedAttention**，能夠在 GPU 內部動態拼接多個使用者的請求，讓 Throughput 大幅提升。

---
### 模型量化（Quantization）：速度、VRAM 與品質的取捨
要讓推論變快且降低成本，另一個主流作法是量化 (Quantization)。

把原本 BF16 (16-bit) 的權重壓縮成 8-bit 或 4-bit。

這對 Agent 的影響是什麼？
- 格式控制與工具調用 (Tool Calling)極易出錯、複雜邏輯與長期Planning能力降級。

要取捨的話?
- 如果 Agent 是對內部的自動化腳本，可以容忍延遲，選 BF16 確保 100% 的工具準確率。
- 如果是對外服務，通常會選擇 AWQ (4-bit) + vLLM，然後在程式碼裡多寫幾行 Error Handling（捕捉 JSON 錯誤並要求模型重試）來彌補量化造成的智商損耗。

---
### 「訓練完的模型」與「能上線的服務」還差多遠？
今天雖然成功寫出了 API，但要把這個 Agent 放到正式 Production 環境，工程師還需要補足以下幾塊拼圖：

- 為了高併發推論引擎，使用vLLM 或 TGI 來處理大量併發請求
- 為了減少使用者的體感等待時間，Agent 思考的過程應該要像 ChatGPT，一個字一個字串流回前端
- 如果 Agent 一直呼叫失敗的工具，API 會卡死。實務上需要設定 max_steps，超過 5 步就強制終止並回傳報錯給使用者
- 用沙盒隔離環境來執行，避免資安風險

---
明天將探討 Context Window 爆炸與記憶體管理的問題！
