# TaskMind — LINE 智慧任務管理機器人

基於 **n8n 自動化工作流程** 與 **Google Gemini AI** 打造的 LINE Bot，讓你直接用自然語言在 LINE 上管理所有待辦任務。

---

## 功能特色

- **自然語言輸入**：直接用中文說「明天要交報告」，AI 自動解析並新增任務
- **圖片辨識（VLM）**：傳送圖片，自動識別內容並存入任務
- **完整任務管理**：新增、查詢、完成、刪除任務
- **待辦確認機制**：支援確認或取消待辦中的任務
- **自動提醒**：每日、逾期、每週五、每週日四種排程提醒
- **雲端儲存**：所有任務存放於 Google Sheets

---

## 系統架構

```
用戶 LINE 訊息
    ↓
LINE Messaging API (Webhook)
    ↓
n8n 主工作流程
    ↓
Google Gemini AI（意圖識別）
    ↓
路由判斷（If 節點）
    ↓
對應處理流程 → Google Sheets → LINE 回覆
```

---

## 支援的意圖

| 意圖 | 說明 | 範例 |
|------|------|------|
| `create_task` | 新增任務 | 「明天要交報告」 |
| `query_task` | 查詢任務 | 「今天有什麼任務」 |
| `complete_task` | 完成任務 | 「報告做完了」 |
| `delete_task` | 刪除任務 | 「取消健身房」 |
| `confirm_pending` | 確認待辦 | 「確認」 |
| `cancel_pending` | 取消待辦 | 「取消」 |

---

## 工作流程說明

| 檔案 | 觸發方式 | 說明 |
|------|----------|------|
| `TaskMind - 主工作流.json` | LINE Webhook | 處理所有用戶訊息與意圖路由 |
| `TaskMind - 每日提醒.json` | 每天 09:00 | 推播今日待辦任務 |
| `TaskMind - 逾期提醒.json` | 每天 21:00 | 推播已逾期未完成的任務 |
| `TaskMind - 每週五提醒.json` | 每週五 21:00 | 推播本週任務進度 |
| `TaskMind - 每週摘要.json` | 每週日 21:00 | 推播本週任務完成率統計 |

---

## 使用技術

- **[n8n](https://n8n.io/)** — 視覺化自動化工作流程平台
- **[LINE Messaging API](https://developers.line.biz/)** — Webhook 接收與 Reply API 回覆
- **[Google Gemini AI](https://deepmind.google/technologies/gemini/)** — 自然語言意圖識別與圖片辨識
- **[Google Sheets](https://sheets.google.com/)** — 輕量任務資料儲存
- **[ngrok](https://ngrok.com/)** — 本地端 Webhook 穿透
- **Docker** — 容器化部署

---

## 部署方式

### 前置需求

- Docker & Docker Compose
- ngrok 帳號（取得 Authtoken）
- LINE Developers 帳號（建立 Messaging API Channel）
- Google Cloud 帳號（啟用 Sheets API 與 Gemini API）

### 步驟

**1. 複製此專案**
```bash
git clone https://github.com/your-username/taskmind-n8n.git
cd taskmind-n8n
```

**2. 設定環境變數**
```bash
cp .env.example .env
```
編輯 `.env`，填入你的 ngrok token：
```env
NGROK_AUTHTOKEN=your_ngrok_authtoken_here
```

**3. 啟動服務**
```bash
docker compose up -d
```

**4. 開啟 n8n**

前往 `http://localhost:5678`，匯入 `workflows/` 資料夾中的五個工作流程 JSON 檔案。

**5. 設定 LINE Webhook**

將 ngrok 提供的網址填入 LINE Developers Console 的 Webhook URL：
```
https://your-ngrok-domain.ngrok-free.app/webhook/taskmind
```

**6. 設定 Google Sheets 連線**

在 n8n 中設定 Google Sheets credentials，並將 Spreadsheet ID 填入對應節點。

---

## 注意事項

- `.env` 檔案包含敏感資訊，請勿上傳至版本控制
- n8n 需持續運行才能接收 Webhook 與執行排程，建議部署於常駐主機
- LINE 的 `replyToken` 有效期限為 30 秒，請確認 n8n 使用即時回應模式（`responseMode: onReceived`）

---

## 授權

MIT License
