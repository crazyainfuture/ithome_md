# Day28 擴充 Streamlit 實作軌跡檢視器與人工除錯介面

萬一 Agent 失敗時，該怎麼辦?今天我們要在 Streamlit 中展開 Agent 的每一步，並加入人工修正功能。

### 新增三個功能:
- **挑選錯誤樣本**：在介面上增加一個下拉選單或過濾器，專門撈出 `status == "error"` 或未得出最終答案的紀錄。
- **展開對話樹 (Trace Inspector)**：利用 st.expander，將 Agent 的 Prompt 以及原始錯誤的 Thought / Tool Call 視覺化。
- **動態編輯與匯出**：在錯誤的那一步提供 st.text_area，讓工程師填寫「正確的推理與動作」。點擊按鈕後，自動將這組對比存成 DPO 專用的 chosen 與 rejected 格式。

---
以下是我們將agent 輸出錯誤的用人工修正功能修正，並將原本失敗的軌跡打包成 rejected，修正過的軌跡打包成 chosen，並把這筆偏好資料加入訓練集。

![image](https://hackmd.io/_uploads/H1s_luqdGx.png)

這樣維護人員就可以直接在監控介面完成錯誤抓取與標註，省去在伺服器終端機翻找 Log 的痛苦!

---
明天我們要把這份得來不易的資料，進行資料飛輪與持續微調 (Continuous DPO)。
