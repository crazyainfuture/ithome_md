# Day9 DPO vs PPO：為什麼 DPO 不需要 Reward Model 且更適合微調？

有了乾淨的偏好資料，我們今天來探討如何把這批資料變成真正能訓練模型的訊號?

### 先回顧傳統 RLHF（PPO 路線）在做什麼 
詳細可以到 Day 5 看看
1.	SFT：先用高品質資料微調出一個基礎模型 $π_{ref}$
2.	訓練 Reward Model（RM）
3.	PPO 強化學習

這個流程有幾個問題：要在訓練迴圈裡即時採樣生成文字、PPO 超參數難調、訓練不穩定、GPU 記憶體吃很兇。

---
一定要用 Reward Model 嗎???還是其實不用呢?

---
### DPO（Direct Preference Optimization）
DPO是 2023 年史丹佛團隊提出的方法，用來把語言模型對齊人類偏好，核心貢獻是提出一種新的 reward model 參數化方式，讓 RLHF 問題可以用一個簡單的分類損失（classification loss）直接求解，不需要額外訓練 reward model，也不需要跑強化學習。

#### 為什麼 DPO 不需要 Reward Model?
因為數學推導證明 reward 可以用「policy 相對 reference model 的機率比值」封閉表達，所以 reward 已經隱含在 policy 本身，不用另外訓練一個 reward model。
#### DPO 適合哪些場景?
適合已經有高品質、成對標註偏好資料，且希望訓練穩定、資源有限、不想維護 RM 和 PPO 採樣迴圈的團隊，是目前多數開源模型偏好對齊的主流做法。

---
####  DPO 不需要 Reward Model 的數學推導
在 RLHF 的目標函數下，最優 policy 跟 reward model 之間存在一個「封閉形式」的對應關係—— reward 可以直接用 policy 自己（即相對於 reference model 的機率比值）表達出來:
$$r(x,y)=\beta log\frac{{\pi }_{\theta }\! (y\mid x)}{{\pi }_{ref}\! (y\mid x)}\! +\beta logZ(x)$$

把這個式子代回 Bradley-Terry 偏好模型 
> $$p\left(y_{1}\succ y_{2}\mid x\right)=\sigma \left(r\left(x,\; y_{1}\right)-r\left(x,\; y_{2}\right)\right)$$

其中
>$r\left(x,y_{1}\right)-r\left(x,y_{2}\right)=\ \underset{r\left(x,y_{1}\right)}{\underbrace{\left[\beta log\frac{{\pi }_{\theta }\! \left(y_{1}\mid x\right)}{{\pi }_{ref}\! \left(y_{1}\mid x\right)}\! +\beta logZ\left(x\right)\right]}}-\underset{r\left(x,y_{2}\right)}{\underbrace{\left[\beta log\frac{{\pi }_{\theta }\! \left(y_{2}\mid x\right)}{{\pi }_{ref}\! \left(y_{2}\mid x\right)}\! +\beta logZ\left(x\right)\right]}}$


相減時直接抵銷:
> $r\left(x,y_{1}\right)-r\left(x,y_{2}\right)=\ \beta log\frac{{\pi }_{\theta }\! \left(y_{1}\mid x\right)}{{\pi }_{ref}\! \left(y_{1}\mid x\right)}\! -\beta log\frac{{\pi }_{\theta }\! \left(y_{2}\mid x\right)}{{\pi }_{ref}\! \left(y_{2}\mid x\right)}$

DPO Loss
把上式代回 Bradley-Terry 機率，並對整個偏好資料集 D 做最大概似估計（maximum likelihood estimation，即最小化負對數概似）：
$${\mathcal{L}}_{DPO}\left({\pi }_{\theta };\; {\pi }_{ref}\right)=-{\mathbb{E}}_{\left(x,\; y_{1},\; y_{2}\right)\sim D}\left[\log{\sigma }\left(\ \beta log\frac{{\pi }_{\theta }\! \left(y_{1}\mid x\right)}{{\pi }_{ref}\! \left(y_{1}\mid x\right)}\! -\beta log\frac{{\pi }_{\theta }\! \left(y_{2}\mid x\right)}{{\pi }_{ref}\! \left(y_{2}\mid x\right)}\right)\right]$$
其中
- $x$：使用者的 prompt 
- $y_{1}$：人類偏好的回答 
- $y_{2}$：人類不偏好的回答 
- $π_θ$：正在訓練的模型 
- $π_{ref}$：固定不動的參考模型（通常就是 SFT 完的初始模型，訓練時凍結權重） 
- $β$：控制 policy 可以偏離 π_ref 多遠的溫度參數，越小允許偏離越多 
- $σ$：sigmoid 函數


---
### Takeaway
- 從兩大方面看 Preference Data 品質問題
- 關鍵在於先正確診斷問題屬於「規格層」還是「執行層」(或兩者皆有)，再對症下藥——規格層用「拆軸 + 規範 + 校準」，執行層用「多重標註 + 品管 + 穩健訓練演算法」，同時保留一份「這是有效多元分歧」的例外名單

---

明天我們將正式進入 GRPO（Group Relative Policy Optimization），看看它如何突破 DPO跟PPO!




