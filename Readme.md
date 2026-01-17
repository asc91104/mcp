# **Model Context Protocol (MCP) 深度解析報告**

## **第 1 頁：什麼是 MCP？ — 解決 LLM 的「孤島效應」**

**核心概念：** MCP 是一個開放標準，讓 AI 模型能安全地存取本機或遠端數據。

* **對新手來說：** 它就像是為 AI 準備的「USB 接口」，讓 AI 能直接讀取你的檔案、資料庫或 API，而不需要你手動複製貼上。  
* **對老手來說：** 它是將「Context (上下文)」標準化的協議，解決了過去每個 AI 工具都要重複開發 Connector 的痛點。

## **第 2 頁：MCP 資料流解析 — 從 Client 出發**

**核心概念：** Client 啟動請求，Host 協調，Server 提供內容。

**資料流向圖：**

1. **User Request:** 使用者在 Host (如 Claude Desktop) 輸入：「分析我的專案文件」。  
2. **MCP Client:** Host 內的 Client 識別到需要外部工具。  
3. **Request to Server:** Client 發送 JSON-RPC 請求到指定的 MCP Server。  
4. **Server Execution:** Server 執行本機腳本（如讀取檔案）。  
5. **Response:** Server 回傳數據，由 Client 餵給 LLM 進行運算。

**技術重點：** 所有的邏輯判斷與組合都在 Client 端完成，Server 僅負責資料提取。

## **第 3 頁：Token 的消耗在哪裡？ — 成本管理指南**

**核心概念：** MCP 會顯著增加上下文長度 (Context Window)，開發者需注意 Token 佔用。

1. **Protocol Overhead (協議開銷)：**  
   * 列出所有可用 Tools 的定義（Name, Description, Schema）都會消耗輸入 Token。  
2. **Resource Retrieval (資源提取)：**  
   * 當 Server 回傳一個大型 CSV 或長文件時，這些內容會直接注入 Context。  
3. **Multi-turn Planning (多輪對話)：**  
   * 模型在決定呼叫哪個工具時，中間產生的思考過程 (Chain of Thought) 也計入消耗。

**優化建議：** 經驗豐富的工程師應精簡 Tool 的 Description，避免一次回傳過多無關數據。

## **第 4 頁：傳輸層詳解 (Transport Layer) — 溝通的管道**

**核心概念：** MCP 支援多種傳輸協議，適應不同部署環境。

| 傳輸方式 | 適用場景 | 技術細節 |
| :---- | :---- | :---- |
| **Stdio** | 本機執行 (Local) | 最常見。Host 透過子進程 (Child Process) 的 stdin/stdout 與 Server 通訊。 |
| **SSE (Server-Sent Events)** | 遠端執行 (Remote) | 基於 HTTP。Server 向 Client 推送訊息，Client 則透過 POST 發送訊息。 |
| **Streamable HTTP** | 未來擴充 | 針對高流量或即時串流場景設計，確保大文件傳輸不阻塞。 |

## **第 5 頁：MCP Server 的三大支柱 (Tools/Resources/Prompts)**

**核心概念：** 這是 MCP 提供給 AI 的三種基本能力。

1. **Tools (工具) \- 「動詞」：**  
   * 模型可以「執行」的操作。例如：query\_database, send\_email。具有副作用 (Side effects)。  
2. **Resources (資源) \- 「名詞」：**  
   * 模型可以「讀取」的靜態數據。例如：本機 Log、API 文檔、專案代碼庫。  
3. **Prompts (提示模板) \- 「預設情境」：**  
   * 預先定義好的對話範本。例如：「幫我解釋這段代碼的 Bug」，包含結構化的指令與上下文引用。

## **第 6 頁：工程師的最佳實踐 — 安全與擴展**

**核心概念：** 構建強健的 MCP 生態系統。

* **安全性：** \- 始終在 Sandbox 環境運行 Server。  
  * 對於 Tools 執行，實施「人機協作 (Human-in-the-loop)」確認機制。  
* **版本控制：**  
  * 利用 JSON-RPC 的方法版本化，確保 Client 與 Server 協議匹配。  
* **可觀測性：**  
  * 使用 MCP Inspector 工具來進行 Debug，觀察傳輸層的原始 JSON 封包。
