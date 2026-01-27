# **Divination — LINE × Notion × Cloudflare 全自動塔羅占卜系統**

（2025  Side Project / Full-stack Serverless Architecture）

一個以 Cloudflare Workers + R2 + n8n + Notion 打造的低成本、高效能、可擴充、前後台分離的塔羅占卜服務。

![images/n8n.png](/images/n8n.png)
![images/line-ex1.PNG](/images/line-ex1.png)
![images/line-ex2.PNG](/images/line-ex2.png)
![images/notion.PNG](/images/notion.png)

---

## **專案亮點**

### **1. 完整 Serverless 架構，低延遲、低成本**

整個系統採用 Serverless 設計：

Cloudflare Workers 作為 API Gateway

R2 作為 CDN 與快取層

Notion 作為內容 CMS

n8n 作為 Orchestrator（排程、後台、自動化）

架構文件：

n8n 全流程：

Worker API 程式碼：

### **✔**

### **2. 抽牌體驗採用快取優先策略（R2 Cache），單次 API < 30ms**

我設計了一套「三層快取」：

1️⃣ Notion → 圖片同步至 R2（CDN）

2️⃣ Notion → 牌組 IDs 依 deck 存成 JSON 快取

3️⃣ /checkDaily 同時完成每日限額檢查與抽牌 → 避免多次 API call

抽牌完全不查 Notion，大幅降低延遲與 API 費用。

### **✔**

### **3. n8n 自動化流程（Orchestrator）**

我在 n8n 中設計了三大核心流程：

### **A. 圖片 CDN 同步（Notion → R2）**

- 解析 Notion 的圖片欄位
- 轉成 base64
- 上傳至 R2
- 回寫公開的 cdnUrl 至 Notion
  （自動批次、避免重複上傳）

### **B. 卡牌 ID 快取**

- 每小時同步一次
- 依分類轉成 { deck: love, ids: [pageId…] } 格式
- 寫入 R2：cache/card-ids-love.json

### **C. LINE Webhook 處理 + 回應 Flex message**

- 拆 LINE events
- Switch 判斷 message or postback
- 先 checkDaily（每日 1 次）
- 再取得 ID → 查單筆 Notion → 組 Flex → 回 LINE

完整流程皆可從 workflow trace 觀察。

### **✔**

### **4. Cloudflare Worker API 設計（高穩定 + 完整驗證 + 防呆處理）**

Worker 提供 4 大端點：

- /upload — 圖片上傳 R2
- /updateCacheCardId — 更新快取
- /getCardId — 隨機抽卡
- /checkDaily — 每日提領限制（含自動抽卡）

每一個端點都含：

- Token 驗證
- Content-Type 驗證
- Payload size 防護
- Key path 安全檢查
- R2 操作錯誤處理
- 統一 Response wrapper

程式碼證明：

（面試官一看就知道你懂 API 安全、架構、錯誤處理）

---

# **技術貢獻**

### **✔ 雲端架構設計與快取策略制定**

- 設計 R2 物件快取架構，確保抽牌 API 毫秒級回應
- 使用 Worker 隔離 API 層，減少 n8n 工作量、提升可靠性

### **✔ API 開發（Cloudflare Worker）**

- 撰寫完整 REST API
- 具備驗證、限流思維、payload 檢查、防止 key injection

### **✔ Orchestration / 自動化流程（n8n）**

- 建立可維護的 Workflow（60+ nodes）
- 強化可觀察性（增加 log, status flag, 分流設計）

### **✔ 後台 CMS 設計（Notion）**

- 定義卡牌資料模型（名稱、副標題、描述、建議、分類、image…）
- 使用 R2 作為靜態圖檔 CDN，降低 Notion 壓力

### **✔ 前端（LINE Flex）**

- 設計完整 Flex UI
- 可動態生成卡牌內容（含圖片、描述、建議等）

---
