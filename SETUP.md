# Google Sheets API 設定指南

伺服器程式碼已經完成，但需要完成 Google Cloud 和 Google Sheets 的權限設定才能正常運作。

## 當前狀態

✅ 伺服器已啟動成功（http://localhost:3000）
✅ Google Sheets API 認證已初始化
❌ **需要完成以下設定才能存取 Google Sheets**

## 錯誤訊息

```
Method doesn't allow unregistered callers (callers without established identity).
Please use API Key or other form of API consumer identity to call this API.
```

這表示需要完成 Google Cloud 的設定和權限授權。

---

## 必要設定步驟

### 步驟 1：啟用 Google Sheets API

1. 前往 [Google Cloud Console](https://console.cloud.google.com/)
2. 選擇你的專案：**sheetsapi-489508**
3. 點擊左側選單 →「API 和服務」→「資料庫」
4. 搜尋「**Google Sheets API**」
5. 點擊進入後，確認狀態為「**已啟用**」
   - 如果未啟用，點擊「**啟用**」按鈕

### 步驟 2：將 Service Account 加入 Google Sheet 的編輯者

你的 Service Account email：
```
sheetsapi@sheetsapi-489508.iam.gserviceaccount.com
```

**操作步驟**：

1. 開啟你的 Google Sheet：
   - Sheet ID: `1vQ5JuVS7oauhh78NnkmjpR2-nwCG31LUvgyGyHdR4U4`
   - 或直接開啟網址：
     ```
     https://docs.google.com/spreadsheets/d/1vQ5JuVS7oauhh78NnkmjpR2-nwCG31LUvgyGyHdR4U4/edit
     ```

2. 點擊右上角「**共用**」按鈕

3. 在「新增使用者和群組」欄位貼上：
   ```
   sheetsapi@sheetsapi-489508.iam.gserviceaccount.com
   ```

4. 權限設定為「**編輯者**」

5. **取消勾選**「通知使用者」（Service Account 不需要通知）

6. 點擊「**傳送**」或「**共用**」

---

## 完成設定後測試

### 1. 測試健康檢查

```bash
curl http://localhost:3000/api/health
```

應該回傳：
```json
{
  "status": "OK",
  "timestamp": "2026-03-07T09:16:19.940Z"
}
```

### 2. 測試讀取 Google Sheet

```bash
curl "http://localhost:3000/api/sheets/test/data?range=Sheet1!A1:D10"
```

成功時應該回傳：
```json
{
  "success": true,
  "sheetName": "test",
  "range": "Sheet1!A1:D10",
  "data": [
    ["欄位1", "欄位2", "欄位3", "欄位4"],
    ["資料1", "資料2", "資料3", "資料4"]
  ],
  "rowCount": 2
}
```

### 3. 測試新增資料

```bash
curl -X POST http://localhost:3000/api/sheets/test/data \
  -H "Content-Type: application/json" \
  -d '{
    "range": "Sheet1!A:D",
    "values": [["測試", "資料", "新增", "成功"]]
  }'
```

---

## 環境變數說明

你的 `.env` 檔案目前設定：

```env
GOOGLE_PROJECT_ID=sheetsapi-489508
GOOGLE_CLIENT_EMAIL=sheetsapi@sheetsapi-489508.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
SHEET_ID_TEST=1vQ5JuVS7oauhh78NnkmjpR2-nwCG31LUvgyGyHdR4U4
port=3000
```

- `SHEET_ID_TEST` 對應 API endpoint 的 `test` sheetName
- 如果要新增其他 Sheet，格式為：`SHEET_ID_XXX`，例如：
  - `SHEET_ID_USERS` → API 使用 `/api/sheets/users/data`
  - `SHEET_ID_ORDERS` → API 使用 `/api/sheets/orders/data`

---

## 常見問題

### Q: 為什麼需要將 Service Account 加為編輯者？

Service Account 是程式用來存取 Google Sheets 的身份。如果沒有加入編輯者，它就沒有權限讀寫該 Sheet。

### Q: 需要在每個 Sheet 都加一次嗎？

是的，每個不同的 Google Sheet 都需要將 Service Account email 加入編輯者。

### Q: 可以設定為「檢視者」而非「編輯者」嗎？

如果只需要讀取功能，可以設為「檢視者」。但如果要使用 POST/PUT/DELETE API（新增、更新、刪除資料），就必須是「編輯者」。

---

## 下一步

完成上述設定後：

1. ✅ 本地開發環境已可使用
2. 📤 將程式碼部署到 Vercel
3. 🔧 在 Vercel Dashboard 設定相同的環境變數
4. 🌐 使用 Vercel 提供的網址存取 API

---

## 需要協助？

如果完成設定後仍有問題，請檢查：

1. Google Cloud Console → API 和服務 → 已啟用的 API 和服務
   - 確認「Google Sheets API」在列表中且狀態為「已啟用」

2. Google Sheet → 共用設定
   - 確認 `sheetsapi@sheetsapi-489508.iam.gserviceaccount.com` 在共用列表中
   - 確認權限為「編輯者」

3. 重新啟動伺服器
   ```bash
   pkill -f "node server.js"
   npm start
   ```
