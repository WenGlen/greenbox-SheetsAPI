# Google Sheets API Backend (Vercel Serverless)

Vercel Serverless Functions API，用於串接多個 Google Sheets 進行 CRUD 操作。

## 專案結構

```
SheetsAPI/
├── api/                          # Vercel Serverless Functions
│   ├── index.js                  # GET /api - 根路由
│   ├── health.js                 # GET /api/health - 健康檢查
│   └── sheets/[sheetName]/
│       ├── data.js               # CRUD operations
│       ├── batch-get.js          # 批次讀取
│       └── batch-update.js       # 批次更新
├── services/
│   └── googleSheets.js           # Google Sheets 服務模組
├── credentials/                  # Google Service Account 憑證（本地開發用，不上傳）
├── server.js                     # 本地開發用 Express 伺服器（保留）
├── vercel.json                   # Vercel 部署配置
├── .env                          # 環境變數（不上傳）
├── .gitignore
├── package.json
└── README.md
```

## 環境設定

### 1. 取得 Google Service Account 憑證

1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 建立新專案或選擇現有專案
3. 啟用 **Google Sheets API**
4. 建立 Service Account：
   - 導航至「IAM 與管理」→「服務帳戶」
   - 點擊「建立服務帳戶」
   - 填寫名稱後建立
   - 在服務帳戶的「金鑰」頁籤，點擊「新增金鑰」→「建立新的金鑰」
   - 選擇 **JSON** 格式下載
5. 將下載的 JSON 檔案：
   - **本地開發**：放到專案的 `credentials/` 資料夾
   - **Vercel 部署**：將整個 JSON 內容複製，稍後設定到 Vercel 環境變數

### 2. 設定 Google Sheets 權限

1. 開啟你要串接的 Google Sheets
2. 點擊右上角「共用」
3. 將 Service Account 的 email（在 JSON 檔案的 `client_email` 欄位）加入為**編輯者**

### 3. 本地開發環境設定

支援三種環境變數配置方式，選擇其中一種即可：

#### 方式 1：分開的環境變數（推薦，適合本地開發）

```env
# Server Configuration
PORT=3000

# Google Service Account Credentials（從下載的 JSON 檔案複製）
GOOGLE_PROJECT_ID=your-project-id
GOOGLE_CLIENT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"

# Google Sheets IDs
SHEET_ID_TEST=1vQ5JuVS7oauhh78NnkmjpR2-nwCG31LUvgyGyHdR4U4
SHEET_ID_USERS=1ABC123def456GHI789jkl
SHEET_ID_ORDERS=1XYZ789abc012MNO345pqr
```

#### 方式 2：使用 JSON 檔案路徑

```env
PORT=3000
GOOGLE_CREDENTIALS_PATH=./credentials/service-account.json

SHEET_ID_TEST=your_google_sheet_id_here
```

#### 方式 3：完整 JSON 字串（適合 Vercel 部署）

```env
GOOGLE_CREDENTIALS_JSON={"type":"service_account","project_id":"...","private_key":"...","client_email":"..."}

SHEET_ID_TEST=your_google_sheet_id_here
```

**Sheet ID 取得方式**：
- 從 Google Sheets URL 取得：`https://docs.google.com/spreadsheets/d/1ABC...XYZ/edit`
- Sheet ID 就是 `/d/` 和 `/edit` 之間的部分：`1ABC...XYZ`

## 本地開發

```bash
# 安裝依賴
npm install

# 方式 1: 使用 Vercel CLI 本地測試（推薦）
npm install -g vercel
vercel dev

# 方式 2: 使用 Express 伺服器（保留的舊版）
npm start
```

本地伺服器會在 `http://localhost:3000` 啟動。

## 部署到 Vercel

### 方式 1: 使用 Vercel Dashboard（推薦）

1. 前往 [Vercel Dashboard](https://vercel.com/dashboard)
2. 點擊 **Add New** → **Project**
3. 選擇你的 GitHub repository（`WenGlen/SheetsAPI`）
4. 點擊 **Import**
5. 設定環境變數（Environment Variables）：

   | Name | Value |
   |------|-------|
   | `GOOGLE_CREDENTIALS_JSON` | 貼上完整的 Service Account JSON 內容 |
   | `SHEET_ID_EXAMPLE` | `your_google_sheet_id_here` |
   | `SHEET_ID_USERS` | `1ABC123def456GHI789jkl` |
   | `SHEET_ID_ORDERS` | `1XYZ789abc012MNO345pqr` |

6. 點擊 **Deploy**

### 方式 2: 使用 Vercel CLI

```bash
# 安裝 Vercel CLI
npm install -g vercel

# 登入
vercel login

# 部署
vercel

# 部署到正式環境
vercel --prod
```

部署後，你會得到一個網址，例如：`https://your-project.vercel.app`

### 設定環境變數（CLI）

```bash
# 新增環境變數
vercel env add GOOGLE_CREDENTIALS_JSON
# 然後貼上完整的 JSON 內容

vercel env add SHEET_ID_USERS
# 輸入 Sheet ID
```

## API 使用說明

### 基本概念

所有 API endpoint 都使用 `:sheetName` 參數來對應環境變數中的 Sheet ID：

- URL 中使用 `users` → 對應到 `SHEET_ID_USERS`
- URL 中使用 `orders` → 對應到 `SHEET_ID_ORDERS`

### 可用的 API Endpoints

部署後的 API 網址格式：`https://your-project.vercel.app/api/...`

#### 1. 根路由

```bash
GET /api
```

返回 API 資訊和可用的 endpoints。

#### 2. 健康檢查

```bash
GET /api/health
```

#### 3. 讀取資料

```bash
GET /api/sheets/{sheetName}/data?range=Sheet1!A1:D10
```

範例：
```bash
curl "https://your-project.vercel.app/api/sheets/users/data?range=Sheet1!A1:D10"
```

#### 4. 新增資料（Append）

```bash
POST /api/sheets/{sheetName}/data
Content-Type: application/json

{
  "range": "Sheet1!A:D",
  "values": [
    ["John", "Doe", "john@example.com", "25"],
    ["Jane", "Smith", "jane@example.com", "30"]
  ]
}
```

範例：
```bash
curl -X POST https://your-project.vercel.app/api/sheets/users/data \
  -H "Content-Type: application/json" \
  -d '{
    "range": "Sheet1!A:D",
    "values": [["John", "Doe", "john@example.com", "25"]]
  }'
```

#### 5. 更新資料

```bash
PUT /api/sheets/{sheetName}/data
Content-Type: application/json

{
  "range": "Sheet1!A2:D2",
  "values": [
    ["Updated", "Data", "new@example.com", "28"]
  ]
}
```

#### 6. 清空資料

```bash
DELETE /api/sheets/{sheetName}/data?range=Sheet1!A2:D10
```

#### 7. 批次讀取多個範圍

```bash
POST /api/sheets/{sheetName}/batch-get
Content-Type: application/json

{
  "ranges": ["Sheet1!A1:B10", "Sheet2!C1:D20"]
}
```

#### 8. 批次更新多個範圍

```bash
POST /api/sheets/{sheetName}/batch-update
Content-Type: application/json

{
  "data": [
    {
      "range": "Sheet1!A1",
      "values": [["New Value"]]
    },
    {
      "range": "Sheet2!B1",
      "values": [["Another Value"]]
    }
  ]
}
```

## 範圍格式說明

Google Sheets 的範圍格式：

- `Sheet1!A1:D10` - Sheet1 的 A1 到 D10 儲存格
- `Sheet1!A:D` - Sheet1 的 A 到 D 欄（整欄）
- `Sheet1` - 整個 Sheet1
- `A1:D10` - 預設第一個工作表的 A1 到 D10

## 注意事項

1. **不要將 `.env` 和 `credentials/` 資料夾上傳到 Git**
2. Service Account email 必須要有對應 Google Sheets 的編輯權限
3. Sheet ID 要在環境變數中正確設定
4. Vercel 會自動處理 CORS，所有 API 都支援跨域請求
5. 支援同時串接多個不同的 Google Sheets
6. 免費版 Vercel 有請求次數限制，請注意使用量

## Vercel 部署優勢

- ✅ **自動擴展**：根據流量自動調整資源
- ✅ **全球 CDN**：快速的 API 回應時間
- ✅ **HTTPS**：自動配置 SSL 憑證
- ✅ **零維護**：無需管理伺服器
- ✅ **Git 整合**：推送程式碼自動部署

## 錯誤處理

API 會返回以下格式的錯誤：

```json
{
  "success": false,
  "error": "錯誤訊息"
}
```

常見錯誤：
- `Sheet ID not found` - 檢查環境變數是否有設定對應的 `SHEET_ID_XXX`
- `The caller does not have permission` - Service Account 沒有該 Sheet 的權限
- `Credentials file not found` - 檢查環境變數 `GOOGLE_CREDENTIALS_JSON` 是否正確設定

## License

ISC
