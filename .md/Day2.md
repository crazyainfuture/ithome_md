# Day2 掌握 RL 基礎：什麼是 MDP ? Reward ? Policy 又是什麼 ?

### 介紹 MDP / State / Action/ Reward 基本定義
先來介紹MDP / State / Action/ Reward基本定義與用途~

MDP（Markov Decision Process，馬可夫決策過程）
- 一套用來描述「在不確定環境中，如何做出一連串決策」的數學框架
- 由4個元素組成，寫成 ($S$, $A$, $P$, $R$)

| 名詞 | 定義 | 
| ------------- | -------- | 
| State狀態 ($S$)|環境「當下」的完整快照，決策所需的所有資訊都包含在裡面 | 
|Action動作 ($A$)|Agent 在某個狀態下可以做的選擇|
|Transition轉移機率($P$)|在某狀態做某動作後，會轉移到哪個新狀態的機率|
|Reward獎勵 ($R$)|做出動作後環境給的回饋分數，正代表好、負代表壞|

- Discount factor折扣因子 ($γ$):0~1 之間的數字，決定「未來獎勵」要打幾折
- Policy 策略($π$):決策者在狀態$s$時將選擇的動作π($s$)

---
### AI Agent 本質上也是一個 MDP嗎？
我們來看看AI Agent與MDP的對應關係!

首先先來看看AI Agent怎麼學習?

1. Agent 觀察環境，拿到一個 State（例如：目前的對話紀錄、當前的螢幕畫面） 
2.	Agent 的 Policy（通常是 LLM）根據這個 State，決定要採取哪個 Action（例如：呼叫某個工具、搜尋網頁） 
3.	環境根據這個 Action 做出反應，Agent 收到新的 State 以及一個 Reward 
4.	以上步驟不斷重複，直到任務結束
    
| 名詞 | Agent task | 
| ------------- | -------- | 
| State狀態 ($S$)|目前的對話紀錄、當前的螢幕畫面 (Prompt/Context)| 
|Action動作 ($A$)|可做的動作，例如:呼叫某個工具、搜尋網頁|
|Transition轉移機率 ($P$)|當在『目前的狀態』下做了『某個動作』後，事情發展成『下一個特定狀態』的可能性有多大？|
|Reward獎勵 ($R$)|量化「這一步做得好不好」|
|Policy 策略($π$)|LLM 本人，進行決策|

AI 代理（Agent）或強化學習（RL）的決策與執行流程圖:
![image](https://hackmd.io/_uploads/r1JilKRwfg.png)



---

### Takeaway
- MDP 用四元組 ($S$, $A$, $P$, $R$) 描述「觀察 → 決策 → 行動 → 回饋」的循環
- AI Agent 之所以本質上是 MDP，是因為它的運作迴圈（LLM 當 Policy、Context 當 State、輸出/工具呼叫當 Action、任務結果當 Reward）跟 MDP 的定義完全對應

---

Day2 介紹了 RL 的基礎知識，明天會再深入了解 RL 相關知識!


