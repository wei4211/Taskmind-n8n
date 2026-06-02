# TaskMind

> 用自然語言在 LINE 上管理所有任務的 AI 助理

![](https://img.shields.io/badge/n8n-orange) ![](https://img.shields.io/badge/Google%20Gemini-4285F4) ![](https://img.shields.io/badge/LINE-00C300) ![](https://img.shields.io/badge/Docker-2496ED)

基於 **n8n 工作流程自動化** 與 **Google Gemini AI**，你只需要傳一句話，TaskMind 會自動理解你的意圖、記錄任務、並在對的時間提醒你。

---

## 功能

| 功能 | 說明 |
|------|------|
| 自然語言輸入 | 「明天要交報告」→ 自動新增任務 |
| 圖片辨識 | 傳圖片，AI 識別內容存為任務 |
| 任務管理 | 新增、查詢、完成、刪除 |
| 待辦確認 | 支援確認或取消待辦中的任務 |
| 自動提醒 | 每日、逾期、每週五、每週日共四種排程 |
| 雲端儲存 | 所有資料存於 Google Sheets |

---

## 系統架構

```
用戶 LINE 訊息
    │
    ▼
LINE Messaging API   (接收 Webhook)
    │
    ▼
n8n 主工作流程       (解析 userId、replyToken)
    │
    ▼
Google Gemini AI     (識別意圖，輸出結構化 JSON)
    │
    ▼
路由判斷 (If 節點)   (根據 intent 分流)
    │
    ▼
對應處理流程 ──────► Google Sheets ──────► LINE 回覆
```

---

## 支援意圖

| 意圖 | 觸發範例 |
|------|---------|
| `create_task` | 「明天要交報告」 |
| `query_task` | 「今天有什麼任務」 |
| `complete_task` | 「報告做完了」 |
| `delete_task` | 「取消健身房」 |
| `confirm_pending` | 「確認」 |
| `cancel_pending` | 「取消」 |

---

## 工作流程

| 檔案 | 觸發時間 | 說明 |
|------|----------|------|
| `TaskMind - 主工作流.json` | LINE Webhook | 處理所有用戶訊息與意圖路由 |
| `TaskMind - 每日提醒.json` | 每天 09:00 | 推播今日待完成任務 |
| `TaskMind - 逾期提醒.json` | 每天 21:00 | 推播已逾期的任務 |
| `TaskMind - 每週五提醒.json` | 每週五 21:00 | 推播本週任務進度 |
| `TaskMind - 每週摘要.json` | 每週日 21:00 | 推播本週任務完成率統計 |

---

## 技術棧

- **[n8n](https://n8n.io/)** — 視覺化工作流程自動化平台
- **[LINE Messaging API](https://developers.line.biz/)** — Webhook 接收、Reply API 回覆
- **[Google Gemini AI](https://aistudio.google.com/)** — 自然語言意圖識別與圖片辨識
- **[Google Sheets](https://sheets.google.com/)** — 任務資料儲存
- **[ngrok](https://ngrok.com/)** — 將本機服務對外公開，讓 LINE Webhook 可以連進來
- **Docker** — 容器化部署

---

## 部署

### 前置準備

**1. 帳號與金鑰**

- Docker & Docker Compose
- [ngrok](https://ngrok.com/) 帳號 → 取得 Authtoken
- [LINE Developers](https://developers.line.biz/) → 建立 Messaging API Channel，取得 Channel Access Token
- Google 帳號

**2. Google Cloud Console**（[console.cloud.google.com](https://console.cloud.google.com/)）

- 建立或選擇一個專案
- 啟用 **Google Sheets API**
- 建立 **OAuth 2.0 用戶端 ID**

**3. Google AI Studio**（[aistudio.google.com](https://aistudio.google.com/)）

- 前往 **Get API key** → 建立 Gemini API 金鑰

---

### 啟動步驟

**Clone 專案**
```bash
git clone https://github.com/your-username/taskmind-n8n.git
cd taskmind-n8n
```

**設定環境變數**
```bash
cp .env.example .env
```
編輯 `.env`：
```env
NGROK_AUTHTOKEN=your_ngrok_authtoken_here
```

**啟動服務**
```bash
docker compose up -d
```

n8n 會在 `http://localhost:5678` 啟動，ngrok 自動建立對外連線。

---

### 匯入工作流程

前往 `http://localhost:5678`，將專案根目錄中的五個 JSON 檔案逐一匯入 n8n。

---

### 設定 Credentials

在 n8n 的 **Settings → Credentials** 建立以下四組：

| 名稱 | 類型 | 說明 |
|------|------|------|
| Google account | Google OAuth2 | Google 帳號授權（OAuth 連線基礎） |
| Google Sheets account | Google Sheets OAuth2 | 連接試算表，主工作流使用 |
| Google Gemini(PaLM) Api account | Google PaLM API | 填入 Gemini API 金鑰 |
| Header Auth account | Header Auth | Header 名稱：`Authorization`，值：`Bearer <LINE Token>` |

---

### 設定 LINE Webhook

將以下網址填入 LINE Developers Console 的 Webhook URL：
```
https://your-ngrok-domain.ngrok-free.app/webhook/taskmind
```

---

### 設定 Google Sheets

建立任務試算表，並將各工作流程節點中的 Spreadsheet ID 替換為你的試算表 ID。

---

## 注意事項

- `.env` 包含敏感資訊，已加入 `.gitignore`，請勿手動上傳
- n8n 需持續運行才能接收 Webhook 與執行排程，建議部署於常駐主機
- LINE 的 `replyToken` 有效期限為 30 秒，Webhook 回應模式須設為 `onReceived`（立即回應）

---

## 授權

MIT License
