# Day17 建立 Baseline 基準點：純 SFT 版本 Agent 的任務表現與極限

昨天我們寫好agent的基礎架構，今天我們要用 SFT 建立第一個 Benchmark，也是我們的Baseline

### 為什麼需要 Baseline？
在 RLHF 或 Agent RL 的實驗中，我們追求的是模型的「決策能力」而非僅是「語言生成能力」。

建立Baseline 的意義是：
1.	定量比較：量化 RL 優化前後的成長空間
2.	實驗證明：確認目前的任務對於純 SFT 而言具有挑戰性，有透過 RL 引入 Reward 進行引導的必要

---
### 建立 Benchmark 基準環境
我使用 Day 16 所做的工具呼叫環境，選定 30 個核心任務作為 Benchmark。
在 SFT 訓練階段，我們訓練 Agent 學習如何從輸入 Prompt 生成正確的 Action sequence。

---
### 本日實驗
我們已經建立好了第一份 Benchmark 評估報告。透過對 SFT Agent 的測試，我們記錄下了以下指標：

![image](https://hackmd.io/_uploads/H1B5WpwdMl.png)

---
### SFT 版本 Agent 的極限

經過 SFT 訓練的 Agent，通常會呈現出以下特徵，這也是我們後續想要透過 RL 解決的痛點：

#### 1.	過度依賴訓練模式 (Pattern Matching)：
SFT 模型因為大量閱讀對話模板，常常會忽略當下其實只需要回答一個單一問題，盲目背誦接下來會出現的對話格式

![image](https://hackmd.io/_uploads/SkFLfTDuzl.png)

在任務「蘋果公司(apple inc.)創辦人是誰?」中。模型不僅沒有給出賈伯斯，還生硬地接上毫不相干的對話模版：「使用者：2+2等於幾?\n助理：你可以使用 Calculator 來計算 2+2。」

#### 2.	缺乏對「錯誤」的反饋學習：
目前的模型完全沒有「呼叫工具 -> 等待真實環境回傳 -> 繼續推論」的機制，而是直接在同一段生成的文字中，把工具可能的回傳結果也「幻想 (Hallucinate)」出來，導致一錯再錯

![image](https://hackmd.io/_uploads/HyAiMTw_Gg.png)

在任務「距離地球最近的恆星是哪一顆？」中。模型呼叫搜尋後，自己編造出一段荒謬的回傳內容：「答案：距離地球最近的恆星是太陽外星人稱為普羅克斯米娜（Proxima Centauri）...」

#### 3.	工具呼叫的盲點：
在沒有 Reward 引導下，SFT Agent 很難學會何時該「停止呼叫」，模型常常會陷入無限迴圈、產出奇怪的選項，或者面對無效輸入時不知道該交給工具報錯

![image](https://hackmd.io/_uploads/BJ1lm6Ddfe.png)

在任務「請問台灣的最高峰是什麼山？」中。模型除了幻想出錯誤答案（雪山），還無法停止輸出，陷入不斷重複的迴圈：「## 最終答案：雪山。 # 最終答案：雪山。 # 最終答案：雪山。」。

---
### 改進方向
這份 Baseline 數據將成為我們接下來 Day 18 引入 Reward Model 與 RL 訓練後的「對照組」。

---
### Takeaway


---

明天預告：我們將正式進入 RL 訓練階段，並觀察第一版 Reward 設計如何讓 Agent 進入「被 Hack」的瘋狂狀態。

---

`🧪 Experiment Summary`

| 項目                | 內容                                                                                  |
| ------------------- |:------------------------------------------------------------------------------------- |
| **Model**           | Llama-3.1-8B-Instruct                                                                 |
| **Method**          | SFT                                                                                   |
| **Dataset**         | 30 Agent Tasks                                                                        |
| **Evaluation**      | Success Rate：56.67%                                                                  |
| **Reason(Failure)** | 過度依賴訓練模式 **(Pattern Matching)**<br>缺乏對「錯誤」的反饋學習<br>工具呼叫的盲點 |
| **Improvement**     | 使用RL，利用Reward 引導                                                               |
