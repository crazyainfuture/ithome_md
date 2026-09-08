# Day4 Policy Gradient 演進：Actor-Critic 與 OpenAI 選擇 PPO 的關鍵

昨天提到除 Valeu-based，還有 Policy-Based。而 Policy-Based 的代表演算法是REINFORCE，我們來看看REINFORCE吧!
但在這之前我們還需要先看看Policy Gradient

### Policy Gradient 是什麼?
#### Policy Gradient（策略梯度） 是一種直接最佳化策略的框架。
#### 如果某個動作帶來好的結果，就提高它未來出現的機率；反之，如果表現差，就降低它的機率
Policy Gradient Theorem:
> ${\nabla }_{\theta }J\left(\theta \right)={\mathbb{E}}_{\tau \sim {\pi }_{\theta }}\left[\sum_{t=0}^{T} {\nabla }_{\theta }{\log{\pi }}_{\theta }\left(a_{t}|s_{t}\right)G_{t}\right]$
> - θ：神經網路（策略模型）的參數
> - ${\pi }_{\theta }\left(a_{t}|s_{t}\right)$：在狀態 $s_{t}$ 下採取行動 $a_{t}$ 的機率
> - $G_{t}$：從時間步 t 開始到回合結束的累積回報 (Cumulative Return)
> 
把每次做的動作，依照『這條軌跡最終獲得的總回報好不好』來加權，回報越高的動作，就讓它未來出現的機率提高得越多

---
### REINFORCE 是什麼?
#### 使用Policy Gradient 的最基礎演算法是REINFORCE
#### 邏輯很直觀：「如果剛才那個動作得分高，我就調高它出現的機率；得分低，就調低機率。」

步驟:
1. 初始化策略網路
2. 在環境中實際玩完一整局遊戲（episode）:
    - 記錄下每一步的資料：$\left(s_{0},a_{0},r_{1}\right),\left(s_{1},a_{1},r_{2}\right),\ldots ,\left(s_{T},a_{T},r_{T+1}\right)$
    - 這串資料稱為一個軌跡 (Trajectory, $τ$)
3. 計算每一步的未來總回報 ($G_{t}$)
> $G_{t}=r_{t+1}+\gamma r_{t+2}+{\gamma }^{2}r_{t+3}+\ldots =\sum_{k=0}^{\infty } {\gamma }^{k}r_{t+k+1}$ 
> - 對於時間點 $t$ 的動作，計算它之後所獲得的所有獎勵總和（通常會乘上衰減因子 $γ$ 以看重眼前利益）
> - 這就是標準的 **Monte Carlo (蒙地卡羅, MC)** 方法：必須等整局遊戲結束，才能算出某個動作之後所獲得的所有獎勵總和
	
4. 計算梯度並更新神經網路

#### 但它有三個極大的問題：
- 因為 REINFORCE 依賴 MC 方法，必須等整局打完才知道輸贏。假設 AI 在一盤棋中下了 99 步神之一手，卻在最後 1 步失誤導致滿盤皆輸，MC 會盲目地將這整局**所有的動作**機率都調低。這種夾雜大量雜訊的評估方式，讓訓練過程非常不穩定且沒效率。

- 步子邁太大，容易掉下懸崖（Step Size Problem）
在深度學習中，我們通常用 Learning Rate（學習率）來控制每次更新的幅度。但在強化學習中，參數（權重）的一點點微小改變，可能會導致「策略行為」發生翻天覆地的巨變。

- 一失足成千古恨
因為 REINFORCE 是 On-policy（邊做邊學），它依賴「當下的策略」去收集資料。一旦前一次更新把策略搞壞了（掉下懸崖），它接下來收集到的全都是「爛資料」。用爛資料繼續訓練，只會越來越爛，永遠訓練不好。

---
不用MC的話有沒有其他方法?於是就有人提出 Actor-Critic 架構!

### Actor-Critic 架構
#### Critic 會即時給 Actor 打分數，Actor 根據分數馬上調整策略，不用等整個回合結束，兼顧「穩定」與「即時」

|角色|對應的 RL 學派|負責什麼|
|---|-------------|------|
|Actor|策略梯度法（Policy Gradient）|決定「該採取什麼行動」，輸出的是一個機率分佈（策略 $π$）|
|Critic|價值函數法（Value-based）|評估「這個行動好不好」，輸出的是一個數值（狀態價值 $V$ 或動作價值 $Q$）|

在原本的 REINFORCE 中，我們是用整局結束的真實總得分 $G_t$ 來更新模型。而在 **Actor-Critic** 中，我們把 $G_t$ 替換成了 **TD (Temporal Difference, 時序差分)** 目標

---
### TD (Temporal Difference, 時序差分)

#### 1. Actor 不用再苦苦等整局遊戲打完，每走一步，Critic 就會利用 TD 方法立刻告訴 Actor 這步做得好不好，馬上進行權重更新

#### 2. 大幅降低變異性 (Low Variance)： 因為只看「眼下這一步的獎勵 + 對未來的預期」，避免了因為遊戲後期其他無關動作導致的雜訊，讓訓練過程變得非常穩定

公式:
> $V^*(s) = max( Reward + γ × V^*(下一狀態) )$
> $V^*(s) = \max_{a} \left[ R(s,a) + \gamma V^*(s') \right]$

公式為什麼要這樣設計可以參考文末的補充: `Bellman Equation`

---
### Takeaway
- 從 Policy Gradient 出發，理解了 REINFORCE 如何直接透過回報來調整策略，但也發現它存在 高變異、更新步伐過大，以及 On-policy 導致訓練容易崩壞 等問題。

- 為了解決「一定要等整局結束才能知道好不好」的問題，進一步介紹了 **Actor-Critic**：讓 Actor 負責做決策，Critic 負責即時評估，透過 TD Learning 讓模型可以邊走邊學，在降低 Variance 的同時提高訓練效率。

---

但新的問題也隨之而來：如果每次更新策略的幅度太大，還是可能讓原本學好的策略直接崩掉，明天會介紹如何解決這類的問題!

---
補充 `Bellman Equation`

將一個複雜的長期決策問題，拆解成「當下的即時獎勵」加上「未來的預期價值」。讓agent不需要窮舉所有可能的未來路徑，就能計算出每個狀態的真實價值。

在介紹 Bellman Equation 之前要先介紹兩個名詞:

|名詞|中英對照|定義|
|---|-------|---|
|Value Function|狀態價值函數 $V(s)$|站在狀態 $s$，之後照著 policy 一直走下去，總共能拿到多少分（期望值）|
|Q-Function|動作價值函數 $Q(s,a)$|在狀態 $s$，選擇動作 $a$，之後照著 policy 走下去，總共能拿到多少分|

Bellman Equation 的公式:
> $V(s) = 這一步的 Reward + γ × 下一個狀態的 Value$

現在這裡值多少分 = 馬上能拿到的分數 + 打折後,未來還能拿到的分數

Bellman Optimality Equation:
> $V^*(s) = max( Reward + γ × V^*(下一狀態) )$

Agent 學習的目標，就是找到一個 policy，讓每個狀態的 Value 都逼近這個「最佳值」

---
`例子` 
用 Bellman Optimality Equation 算 $V(s)$

走 6 步，每一步 reward 分別是 $R$ = 0, -0.2, 0, 0, 0, +1\
$γ = 0.9$

| $State$ | $S_1$ | $S_2$ | $S_3$ |$S_4$ | $S_5$ | $S_6$ |
| ------- | ----- | ----- | ----- |----- | ----- | ----- |
| $Reward$| 0     | -0.2  | 0     |   0  |   0   |   +1  |


$V(S_6)$ = 1\
$V(S_5)$ = 0 + 0.9 × 1     = 0.9\
$V(S_4)$ = 0 + 0.9 × 0.9   = 0.81\
$V(S_3)$ = 0 + 0.9 × 0.81  = 0.729\
$V(S_2)$ = -0.2 + 0.9 × 0.729 = 0.456\
$V(S_1)$ = 0 + 0.9 × 0.456 = 0.41

