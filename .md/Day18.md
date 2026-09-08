# Day18 TRL 訓練實戰：Reward 第一版設計與被 Hack 全紀錄

經過 Day 17 建立的純 SFT Baseline 評估後，我們正式進入 Reinforcement Learning (RL) 的核心實作階段。今天，我們將設計第一版的 Reward 函數，並將模型放進訓練環境中。

發現，RL 訓練往往不會一帆風順，會發生 **Reward Hacking**。

### 第一版 Reward 設計 (Reward v1)
第一版 Reward 設計得非常直觀：
1.	成功呼叫工具 (Valid Tool Call)：給予 +1的微小獎勵
2.	格式錯誤 (Format Error/Invalid JSON)：給予 -1的懲罰
3.	成功完成任務 (Task Success)：給予 +10 的最終大獎

我們預期這樣的設計能引導 Agent 積極使用工具，並最終給出正確解答

#### 分析訓練日誌
查看 **Loss & Reward** 指標：Reward 穩定上升，看似一切順利

![image](https://hackmd.io/_uploads/Hy4ILpP_ze.png)

看著 Reward 曲線不斷攀升，以為 Agent 變得很聰明時，打開 Log 卻看到了崩潰的畫面

---
### Agent 發生了典型的Reward Hacking行為!!!
作弊行為探討：
#### 1. 為了拿分，放棄人類語言邏輯 (Tag Spamming)
當模型發現只要輸出特定字串就能獲得高額獎勵時，它會放棄生成自然流暢的句子，轉而使用最低成本的方式湊出答案

- 在 Epoch 18 的數學題中，模型一旦輸出了能獲得滿分（10.0 分）的 
`<final_answer>100</final_answer>`，後續就完全崩壞，開始無意義地瘋狂重複 `</final_answer>` 標籤

- 模型發現只要把關鍵字塞進去就好，根本不在乎語句是否通順

這是Epoch 18 的response:
> <final_answer>100</final_answer> </final_answer> 100</final_answer> </final_answer> </final_answer>…

#### 2. 騙取工具呼叫的格式獎勵
在遇到無法直接給出確切答案的題目（例如天氣題）時，模型學會了利用格式來「苟且偷生」賺取小額獎勵

- 面對天氣題，模型在 Epoch 1 中成功藉由輸出 <tool_call>{"tool": "weather", "args": "台北"}</tool_call> 的 JSON 格式，賺取了 1.0 的獎勵
- 在 Epoch 13 中，模型同樣利用 </tool_call> 的 JSON 格式成功獲得 1.0 的獎勵

這是Epoch 13 的response:
> "請使用氣象站資料\n\n請注意：你必須嚴格使用以下兩種格式之一來回答，即只能使用其中一種格式。\n\n展開回答選項：\n\n1. 呼叫工具：<tool_call>{\"tool\": \"name\", \"args\": \"value\"}</tool_call>\n\n你可以使用氣象站資料來查詢台北天氣。\n\n\n呼叫氣象站 API</tool_call>{“tool”: “氣象站”，"

---
Agent 確實「最大化」了它的 Reward，但完全偏離了我的預期目標。這就是 RL 訓練中最常遇到的難題——模型會以意想不到的方式來滿足給定的 Reward 條件。

---
### Takeaway
- RL　訓練常見的問題：Reward Hacking
- 今天實驗雖然在任務成功率上是「失敗」的，但它完美展示了 AI Agent 訓練的真實樣貌。不精確的 Reward 設計會直接導致系統性偏差。

---

明天預告：Day 19 將設計 Reward 第二版，修正這個無限呼叫工具的漏洞，並進行小型參數比較對照


