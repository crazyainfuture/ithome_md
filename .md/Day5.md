# Day5 避免模型崩潰的秘密：PPO Clip 機制與 KL Penalty 解析

昨天提到模型一次走太大步，會崩潰，那有沒有什麼方法可以避免呢?
有的! 我們今天要介紹非常有名的PPO，但在這之前先介紹一下TRPO~

### TRPO 是什麼?
他引入了兩個東西!!!
#### 1. Surrogate Objective (代理目標函數)
> $$L(\theta) = \mathbb{E}_{s \sim \rho_{\theta_{old}}, a \sim \pi_{\theta_{old}}} \left[ \frac{\pi_{\theta}(a\vert{}s)}{\pi_{\theta_{old}}(a\vert{}s)} A_{\theta_{old}}(s, a) \right]$$
> - $\pi_{\theta}(a\vert{}s)$：新策略的機率
> - $\pi_{\theta_{old}}(a\vert{}s)$：舊策略的機率
> - $A_{\theta_{old}}(s, a)$：優勢函數（Advantage Function），代表在狀態 $s$ 下執行動作 $a$ 比平均表現好多少

負責利用舊經驗來評估新策略的好壞，指引 AI 變強的方向

[文末補充] `優勢函數（Advantage Function）`

#### 2. Trust Region Constraint (信任區域限制)
為了確保新策略不要偏離舊策略太遠，TRPO 引入了 **KL 散度**（Kullback-Leibler Divergence） 作為約束條件：
> $$\mathbb{E}_{s \sim \rho_{\theta_{old}}} \left[ D_{KL}(\pi_{\theta_{old}}(\cdot\vert{}s) \parallel \pi_{\theta}(\cdot\vert{}s)) \right] \le \delta$$
> - $D_{KL}$ 用來衡量兩個機率分佈的差異程度
> - $\delta$ 是一個超參數，定義了我們可以信任的「區域」大小

負責控制每次策略改變的幅度不能太大，避免模型崩潰

---
#### 但它也有兩個極大的問題：
- **計算太慢且極度耗能**（二階最佳化的代價）：
為了解出那個「KL 散度限制」，TRPO 必須計算極度複雜的神經網路「二階導數（費雪資訊矩陣）」。每走一步都要花費龐大的運算資源和時間。

- **程式碼極度難寫與除錯**：
TRPO 的實作包含了共軛梯度法和線性搜尋等複雜步驟，只要程式碼稍微有一點 Bug，模型就會完全無法收斂，對工程師來說非常不友善。

---
所以才有了PPO!!!

### PPO（Proximal Policy Optimization，近端策略優化）
#### 保留了 TRPO「不讓步伐跨太大」的核心精神，但把複雜的二階數學計算，換成了一個極度簡單的「剪裁（Clipping）機制」

#### Clipped Surrogate Objective（裁剪代理目標函數）
> $$L^{CLIP}\left(\theta \right)={\hat{\mathbb{E}}}_{t}\left[\min{\left(r_{t}\left(\theta \right){\hat{A}}_{t},\text{clip}\left(r_{t}\left(\theta \right),1-\epsilon ,1+\epsilon \right){\hat{A}}_{t}\right)}\right]$$
> 
1. 先用「舊策略 ${\pi }_{old}$」收集一批資料（多個 episode） 
2. 用這批資料計算「新策略 ${\pi }_{new}$」相對「舊策略」的機率比值：
> $r\left(\theta \right)=\frac{{\pi }_{new}\left(a\mid s\right)}{{\pi }_{old}\left(a\mid s\right)}$
3. 用一個 Clip函數把這個比值夾在$[1-ϵ,1+ϵ]$之間（通常 $ϵ=0.2$），避免更新過頭

---
`例子`
具體數字範例(假設 Critic 算出這個動作的 Advantage $A=3.8$):

`情境 A：機率比值在安全範圍內`
- 舊策略選這個動作的機率 ${\pi }_{old}=0.2$
- 新策略選這個動作的機率 ${\pi }_{new}=0.22$
- 機率比值：$r\left(\theta \right)=0.22/0.2=1.1$

因為 $1.1$ 落在 $\left[0.8,\; 1.2\right]$ 範圍內（$ϵ=0.2$），不會被裁剪，直接用 $r\left(\theta \right)\times A=1.1\times 3.8=4.18$ 當作更新力道

`情境 B：機率比值暴衝，觸發裁剪`
- 新策略突然把機率拉到 ${\pi }_{new}=0.5$
- 機率比值：$r\left(\theta \right)=0.5/0.2=2.5$（暴衝！）

因為 $2.5$ 超過上限 $1.2$，會被裁剪成 $1.2$
更新力道變成 $1.2×3.8=4.56$，而不是 $2.5×3.8=9.5$

---
### 為什麼 LLM 選擇 PPO？

#### 因為在訓練千億參數的語言模型時，「穩定性」與「計算效率」是首要考量。其他演算法要嘛極不穩定，要嘛計算過於複雜。PPO 憑藉獨特的「Clipping機制」，確保模型每次只做安全的微幅更新，不僅大幅降低運算成本，且不需繁瑣的參數調校即可獲得極佳效能。

---
### Takeaway
- TRPO 透過 Surrogate Objective 告訴模型「往哪裡變強」，再利用 KL Divergence 的 Trust Region 限制模型「不要一次走太遠」

- TRPO 的問題是二階最佳化讓 TRPO 計算成本高、實作也複雜

- PPO 則把這個複雜的限制，簡化成直觀的 Clipping 機制：當新舊策略的機率差距太大，就直接限制更新幅度，避免模型因為「一步走太大」而崩潰
- TRPO 告訴我們「不要走太遠」，PPO 則用 Clipping 讓模型真的能安全地走

---

明天我們直接動手做一個小型實驗，看看 PPO 最核心的 Clip 大小 $ε$ 到底會如何影響模型訓練！

實際比較不同 Clip 大小下的訓練效果，觀察「限制太嚴格」或「放得太寬」會帶來什麼差異，進一步理解 PPO 為什麼能在穩定性與學習效率之間取得平衡。

最後也會把這幾天的內容串起來，整理 RL → RLHF → Agent RL 的演進脈絡。

---
補充 `優勢函數（Advantage Function）`

利用Generalized Advantage Estimation（GAE）方法計算 Advantage 。

完全依賴 Critic 的估計的話,如果 Critic 估不準,誤差會被放大(偏差大);且每次計算只看一步，波動小(方差小)。

用 GAE 的話，不只看 1 步，也不看到底，而是把「未來所有步數的 TD Error」都加權平均起來，用一個參數 λ(lambda)控制「要看多遠」。

公式:
> $${\hat{A}}_{t}^{GAE\left(\gamma ,\lambda \right)}=\sum_{l=0}^{\infty } {\left(\gamma \lambda \right)}^{l}{\delta }_{t+l}$$

GAE加入了 $λ$ 這個參數 (介於 0 到 1 之間)，它就像一個調音旋鈕，在「瞎猜 (Bias)」與「運氣 (Variance)」之間找到完美的平衡：
公式展開:
> ${\hat{A}}_{t}={\delta }_{t}+\left(\gamma \lambda \right){\delta }_{t+1}+{\left(\gamma \lambda \right)}^{2}{\delta }_{t+2}+{\left(\gamma \lambda \right)}^{3}{\delta }_{t+3}+\ldots$
> - 當 λ=0 時（只看眼前的 TD Error）：
${\hat{A}}_{t}={\delta }_{t}=r_{t}+\gamma V\left(s_{t+1}\right)-V\left(s_{t}\right)$
> - 當 λ=1 時（蒙地卡羅估計）： 這時公式會退化成一路加到遊戲結束的實際回報（類似 REINFORCE 的作法）

在訓練 PPO 等演算法時，我們通常不會走極端，而是將 λ 設定在 0.95 左右。

---
`TD Error`

TD Error（δ）公式：\delta =r+\gamma V\left(s^{\prime }\right)-V\left(s\right)代表「實際結果」與「原本預期」的落差。
