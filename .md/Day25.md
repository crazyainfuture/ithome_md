# Day25 解決 Token 爆炸：多步驟任務下的 Context 管理與裁剪策略

今天，我們就來探討如何在多步驟任務中進行**上下文管理 (Context Management)**。

先來看看會遇到甚麼問題!

### 當工具回傳結果撐爆 Context Window 怎麼辦？
在打造 Agent 的過程中，經常會遇到一個煩惱：工具 (Tools) 運作得太好了，以至於回傳了海量的資料

假設讓 Agent 呼叫一個 fetch_webpage 工具，它直接把整個維基百科網頁的原始 HTML 或上萬筆的 JSON 塞回對話歷史中。

這不僅會瞬間撐爆 LLM 的 Context Window（上下文視窗），導致 API 報錯 (Token Limit Exceeded)，更會帶來昂貴的 API 費用，甚至引發 LLM 的「迷失在中間 (Lost in the middle)」效應，導致模型抓錯重點或產生幻覺。

---
### 策略設計：簡單的裁剪與摘要機制
為了解決這個問題，可以嘗試在「工具執行完畢」與「將結果交給 LLM」之間，插入一個中介層 (Middleware)。
最常見的基礎策略：

#### 1.	粗暴截斷 (Hard Truncation)：
設定一個字元上限，超過的部分直接切掉，並在句尾加上提示，讓 LLM 知道資料被截斷了

#### 2.	簡易過濾 (Rule-based Filtering)：
針對特定格式（如 JSON 或 HTML）拔除無用的標籤或空值

---
### 不裁剪 vs 裁剪後 有什麼影響?
雖然「裁剪後」成功救下了我們的錢包，且避免了 Context Window 爆炸，但直接截斷的代價是破壞了資料的完整性。如果關鍵資訊剛好在被切掉的後半段，Agent 就會給出錯誤答案。

---
### 長任務下的進階記憶策略
當單純的截斷不夠用時，需要更聰明的記憶體積控管方式。以下是三種在長任務中常見的 Context 管理策略：

#### 1. 對話摘要緩衝區 (Conversation Summary Buffer)
當 messages 陣列的長度超過設定閾值（例如 10 輪對話），就在背景觸發一次輕量級的 LLM 呼叫，將前 8 輪的對話「總結」成一小段核心記憶（Core Memory），然後清空舊訊息，只保留摘要與最新的 2 輪對話

- 優點：兼顧了歷史脈絡與 Token 消耗
- 缺點：每次總結都會流失些微的細節
#### 2. 工具輸出的 LLM 摘要 (LLM-based Summarization)
如果工具回傳的結果很長，不要自己用 Python 切斷，而是寫一個小型的獨立 Agent（使用較便宜的模型），專門負責把長篇大論濃縮成關鍵字或 Markdown 條列，再把摘要結果丟給主 Agent
#### 3. RAG 記憶體 (Vector Store Memory)
對於超長期的任務，將每次工具的回傳結果與對話，寫入向量資料庫 (Vector DB) 中。主 Agent 的 Context 永遠保持乾淨，只有當它需要回憶先前的步驟時，才透過 search_memory 工具去檢索過去的紀錄

---
Takeaway:
- 在實作 Agent 時，我們很容易陷入「給它越多資料越好」的迷思
- 優秀的上下文管理，不僅是為了省錢或繞過 Token 限制，更是為了提高訊噪比 (Signal-to-Noise Ratio)
- 不管是透過程式邏輯的硬性裁剪，還是導入摘要機制，幫 Agent 「過濾雜訊」，才是讓任務成功率穩定提升的關鍵

---
明天我們將進一步探討 Agent 部屬起來之後，要怎麼對它進行監控與維護。
