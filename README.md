# TaskMind — LINE 智慧任務管理機器人

基於 **n8n 自動化工作流程** 與 **Google Gemini AI** 打造的 LINE Bot，讓你直接用自然語言在 LINE 上管理所有待辦任務。

---

## 功能特色

- **自然語言輸入**：直接用中文說「明天要交報告」，AI 自動解析並新增任務
- **圖片辨識**：傳送圖片，AI 自動識別內容並存入任務
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
- **[ngrok](https://ngrok.com/)** — 將本機服務對外公開，讓 LINE Webhook 可以連進來
- **Docker** — 容器化部署

---

## 部署方式

### 前置需求

- Docker & Docker Compose
- ngrok 帳號（取得 Authtoken）
- LINE Developers 帳號（建立 Messaging API Channel，取得 Channel Access Token）
- Google Cloud 帳號

### Google 服務設定

**Google Cloud Console**（[console.cloud.google.com](https://console.cloud.google.com/)）：

1. 建立或選擇一個專案
2. 啟用 **Google Sheets API**
3. 建立 **OAuth 2.0 用戶端 ID**（供 n8n 連接 Google Sheets）

**Google AI Studio**（[aistudio.google.com](https://aistudio.google.com/)）：

1. 登入後前往 **Get API key**
2. 建立 API 金鑰，後續填入 n8n 的 Gemini credentials

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

前往 `http://localhost:5678`，匯入專案根目錄中的五個工作流程 JSON 檔案。

**5. 在 n8n 設定 Credentials**

匯入工作流程後，需在 n8n 中建立以下四組憑證（Settings → Credentials）：

| 憑證名稱 | 類型 | 說明 |
|----------|------|------|
| Google account | Google OAuth2 | Google 帳號授權（OAuth 連線基礎） |
| Google Sheets account | Google Sheets OAuth2 | 連接試算表（主工作流使用） |
| Google Gemini(PaLM) Api account | Google PaLM API | 填入 Gemini API 金鑰 |
| Header Auth account | Header Auth | 填入 LINE Channel Access Token，Header 名稱設為 `Authorization`，值為 `Bearer YOUR_TOKEN` |

**6. 設定 LINE Webhook**

將 ngrok 提供的網址填入 LINE Developers Console 的 Webhook URL：
```
https://your-ngrok-domain.ngrok-free.app/webhook/taskmind
```

**7. 設定 Google Sheets**

在 Google Sheets 建立任務試算表，並將各工作流程節點中的 Spreadsheet ID 替換為你的試算表 ID。

---

## 注意事項

- `.env` 檔案包含敏感資訊，請勿上傳至版本控制
- n8n 需持續運行才能接收 Webhook 與執行排程，建議部署於常駐主機
- LINE 的 `replyToken` 有效期限為 30 秒，請確認 n8n 使用即時回應模式（`responseMode: onReceived`）

---

## 授權

MIT License
