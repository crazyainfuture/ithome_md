# Day7 拆解 RLHF 全流程：SFT、Reward Model 到 PPO 的 Pipeline

講 Reward Model 之前，我們先來聊聊RLHF流程，看看reward model用在哪裡?
### RLHF（Reinforcement Learning from Human Feedback，人類回饋強化學習）的完整流程通常分成五個階段：
#### 預訓練 → 監督式微調(SFT) → 訓練獎勵模型(RM) → 強化學習優化(RL) → 評估與迭代
依序是預訓練語言模型、在精選範例上做監督式微調、根據人類偏好資料訓練獎勵模型，最後用強化學習根據獎勵模型的回饋來微調語言模型

---
進入正題 Reward Model!!!

### 如何訓練Reward Model (RM)?
#### 訓練一個模型，輸入prompt，輸出一個「分數」（scalar），分數越高代表越符合人類偏好


做法是拿人類標註過的「一組回答中誰比較好」的成對偏好資料（pairwise preference data），讓模型學會替 chosen（較好）回答打的分數高於 rejected（較差）回答。
1.	收集偏好資料
2.	模型打分：把 (prompt+chosen) 和 (prompt+ rejected) 分別餵進RM模型，各自吐出一個 scalar 分數 r(x, y_chosen)、r(x,y_ rejected)
3.	用 Bradley-Terry 公式算出 y_chosen 比 y_ rejected 好的機率
4.	算 loss
5.	更新參數
---
`偏好資料格式`
必須是「同一個 prompt，一個 chosen（人類選的較好回答）、一個 rejected（較差回答）」。
例:
{
    "prompt": "請解釋什麼是遞迴？",
    "chosen": "遞迴是函式呼叫自己來解決問題...(完整、清楚的回答)",
    "rejected": "遞迴就是迴圈啦。" 
}

---
`Bradley-Terry 公式`
用來定義訓練目標函數——告訴模型「什麼叫做學得好」。
給定 prompt x，模型生成兩個回答 y₁、y₂，人類標註 y₁ 比較好。獎勵模型學的目標函數是：
> $$p\left(y_{1}\succ y_{2}\mid x\right)=\sigma \left(r\left(x,\; y_{1}\right)-r\left(x,\; y_{2}\right)\right)$$
> - r(x, y)：獎勵函數（Reward function），也就是獎勵模型本身。輸入是 prompt x 和某個回答 y，輸出一個純量分數（scalar），分數越高代表模型認為這個回答越好。所以 r(x,y1) 就是獎勵模型對回答 y1 打的分數，r(x,y2) 是對 y2 打的分數。

---
### Takeaway
- RLHF 流程
- Reward Model 用來判斷哪個回答更符合人類偏好，就給越高的分數

---

明天會詳細說明關於偏好資料的注意事項，難道，有陷阱???




