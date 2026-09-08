# Day3 Value-based：為什麼 ChatGPT 不能直接用 Q-learning？

昨天最後補充如何算在某個state下有多好的數學理論公式，今天我們來討論如何使用到演算法中。
### Valeu-based 是什麼?
決定 Agent 行為主要分為兩大流派：**Value-Based** 與 **Policy-Based**

|  | Value-Based (價值導向) | Policy-Based (策略導向) |
| -------- | -------- | -------- |
| 概念     | 把表格裡的 $Q(s,a)$ 算得越精準越好 | 直接學習「在狀態 $s$ 下，應該採取什麼動作 $a$ 的機率分佈」    |
|Policy|每次都選分數最高的那個動作|模型會直接輸出一個機率，根據這個機率來抽樣決定動作|
|代表算法|Q-Learning、DQN |REINFORCE|

---

### Q-Learning 是什麼?

#### Q-Learning 會維護一張表（Q-Table），記錄「在某個 State，選某個 Action，大概值多少分」

回顧之前算 $V(s)$ 時，其實是「上帝視角」，已經知道整條路徑的 Reward 是多少。但真實世界的 Agent 不是這樣——它必須先做動作，才知道會發生什麼事。這時就需要Q-Learning解決這個問題。

**Q-Table** 像是:

|     | Action1 | Action2 | Action3 |
| --- | ------ | ---- |-------|
| State1 | 0.0 |  0.0   | 0.0 |
| State2 | 0.0 |  0.0   | 0.0 |
| State3 | 0.0 |  0.0   | 0.0 |

一開始全部是 0，因為什麼都不知道，Agent 透過不斷嘗試、觀察 Reward、更新表格，讓這張表格越來越準確

---
### Q-Learning 怎麼更新 ?

Q-Learning更新公式：
> $Q(s,a) ← Q(s,a) + α × [ R + γ × max(Q(下一狀態的所有 Action)) − Q(s,a) ]$

每次做完一個動作，就拿「新觀察到的結果」去修正「舊的猜測」，改一點點，慢慢逼近真相

但真實世界的可能性無限多，Q-Table會記不下無限多種可能，有甚麼方法可以解決此問題?

---
### DQN（Deep Q-Network）是什麼?

#### DQN 用一個神經網路取代 Q-Table，解決「State 太多、表格存不下」的問題
Q-Table 本來是**查表**，改用DQN 是「**用一個函數去『猜』表格裡本來該有的值**」，而且這個函數可以類推到沒見過的 State

#### 用神經網路當「函數近似器」，不再查表，而是訓練一個神經網路，輸入 State，輸出「每個 Action 的 Q 值」

Q-Learning 的更新公式相同，但 DQN 把這個「差距」變成神經網路的 Loss Function（損失函數），透過**梯度下降**去縮小這個差距：
> $Loss = ( R + γ × max(Q(s', a')) − Q(s,a) )^2$

流程如下:
```
輸入：State (例如對話向量、畫面像素)
↓
神經網路 (DQN)
↓
輸出：Q(s, Action1), Q(s, Action2), Q(s, Action3) ...
```

#### DQN 優勢:
即使這句話是網路從沒見過的全新句子，只要語意跟訓練過的句子相似，DQN 依然能給出合理的 Q 值估計——這就是「函數近似」相對於「查表」的巨大優勢，也是為什麼 LLM 的強化學習不可能用 Q-Table，一定要用類似 DQN 這種神經網路方法的原因。

---
### LLM Agent 也無法直接用 Q-learning、DQN？
#### Action 也太多

DQN 的核心操作 max Q(s',a') 需要「把每個可能的 Action 都試算一次 Q 值,再挑最大的那個」——這件事在 Action 是「一段自由文字」的情況下做不到，因為不可能窮舉所有可能的句子。

#### LLM 訓練幾乎都用 Policy Gradient / PPO，而不是 Q-Learning、DQN 

---
### Takeaway
- 決定 Agent 行為主要介紹：**Value-Based** 與 **Policy-Based**
- Q-Learning 記錄在某個 State，選某個 Action，大概值多少分
- DQN 用神經網路取代 Q-Table，解決 State 空間過大、連續、無法窮舉的問題，訓練邏輯仍然是 Bellman Equation，只是把「改表格」換成「用 Loss Function 做梯度下降」。

---

Day3 介紹 Value-Based 相關算法，明天會介紹 Policy-Based 相關算法!

---
補充 `Experience Replay`

Agent 是一步一步順著時間在跟環境互動的：S1→A1→S2→A2→S3...。
如果直接拿「剛剛發生的這一步」馬上拿去訓練神經網路，會有兩個嚴重問題：
1.	Catastrophic Forgetting（災難性遺忘）：連續幾步的 State 通常很相似（例如同一段對話裡連續幾句話），神經網路連續看到一堆「長得很像」的資料，會過度擬合最近發生的情況，忘記早期學到的東西
2.	資料用完就丟，浪費：每個 (s,a,r,s') 只被用來訓練一次就丟掉，效率很差

解法: `記憶庫（Replay Buffer）`

準備一個**記憶庫（Replay Buffer）**，把 Agent 每一步的經驗 (s, a, r, s') 都存進去，訓練時不是用剛發生的那一步，而是從整個記憶庫裡隨機抽一批（mini-batch）出來訓練

![image](https://hackmd.io/_uploads/ryZRw1l_Gl.png)

---
補充 `Target Network`
問題: 每更新一次網路權重，不只是 Q(s,a) 變了，連「訓練目標」max(Q(s',·)) 也跟著變了——因為它用的是同一組權重算出來的

解法: `準備兩個網路`

|網路|用途|更新頻率|
|---|----|------|
|Online Network（線上網路）|平常用來選動作、被訓練更新|	每一步都更新
|Target Network（目標網路）|	專門用來算 Loss 公式裡的 max(Q(s',·)) 也就是「訓練目標」|	每隔一段時間（例如每 1000 步）才把 Online Network 的權重複製過來，其他時間完全「凍結不動」

![image](https://hackmd.io/_uploads/H1kZc1guMe.png)



