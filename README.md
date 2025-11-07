# n8n Render Flow 部署指南

本指南將帶你一步步在 Render 平台上部署 n8n，並使用 Supabase 作為資料庫，讓你的自動化工作流程穩定運行。

## 目錄

- [前言](#前言)
- [一、Supabase 建立資料庫](#一supabase-建立資料庫)
- [二、Render 上利用 Docker 安裝 n8n](#二render-上利用-docker-安裝-n8n)
- [三、設定定時排程 (cron job)](#三設定定時排程-cron-job)
- [常見問題](#常見問題)

---

## 前言

### 為什麼需要 Supabase？

資料庫可以想成「n8n 的記事本」，它會記錄你的工作流程。Render 免費方案常常會自動關機，如果記事本也放在那裡就會不見。**把記事本放到 Supabase，就算 Render 關機了，你的東西還在。**

### 為什麼需要 Render？

Render 就像「租一台雲端電腦」，讓你的 n8n 可以 24/7 運行在雲端，不需要自己的電腦一直開著。

### 為什麼需要 cron-job？

因為 Render 免費方案會在一段時間沒有活動後自動休眠，cron-job 會定時「叫醒」它，確保服務持續運行。

---

## 一、Supabase 建立資料庫

### 步驟 1：建立組織

1. 前往 [Supabase](https://supabase.com/) 並登入
2. 如果沒有組織，點擊 **「New organization」** 建立新的組織
3. 輸入組織名稱並完成建立

### 步驟 2：建立專案

1. 進入組織後，點擊 **「New project」** 建立專案
2. 輸入以下資訊：
   - **Project name**：為你的專案命名（例如：n8n-database）
   - **Database password**：設定資料庫密碼（**請務必記下來！**）
   - **Region**：選擇離你最近的區域（例如：Northeast Asia (Tokyo)）
3. 點擊 **「Create new project」**，等待專案建立完成

### 步驟 3：取得資料庫連線資訊

1. 專案建立完成後，點擊右上角的 **「Connect」** 按鈕
2. 在彈出視窗中，選擇 **「Transaction pooler」** 標籤
3. 點擊 **「View parameters」**
4. 記下以下資訊（等等會用到）：
   - **host**：主機位址
   - **port**：通常是 `6543`
   - **database**：通常是 `postgres`
   - **user**：使用者名稱
   - **password**：剛剛設定的資料庫密碼

> **重要提醒**：這些資訊請妥善保存，後續在 Render 設定時會需要用到。

---

## 二、Render 上利用 Docker 安裝 n8n

### 步驟 1：建立 Render 專案

1. 前往 [Render.com](https://render.com/) 並註冊/登入
![註冊/登入 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/1.png)
![選擇 github 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/2.png)
![選擇 github 2](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/3.png)

2. 點擊 **「Create your first project」** 或 **「New +」** 新建專案
3. 輸入專案名稱後建立

### 步驟 2：建立 Web Service

1. 在專案內，點擊 **「New」** → **「Web Service」**
![建立網頁服務](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/4.png)
![建立網頁服務](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/5.png)
2. 選擇 **「Deploy an existing image from a registry」**
![docker 部署 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/6.png)
3. 在 **Image URL** 輸入：
   ```
   n8nio/n8n:latest
   ```
   > `latest` 代表使用 n8n 官方最新版本
![docker 部署 2](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/6.png)

### 步驟 3：基本設定

1. **Name**：為你的服務命名（例如：my-n8n-service）
2. **Region**：選擇離你最近的區域
![選擇區域](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/7.png)
3. **Instance Type**：選擇 **「Free」** 免費方案
![免費方案](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/7.png)

### 步驟 4：設定環境變數
![設定環境變數](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/8.png)
![設定環境變數1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/9.png)
在 **Environment Variables** 區域，點擊 **「Add Environment Variable」**，逐一新增以下變數：

```bash
GENERIC_TIMEZONE=Asia/Taipei
TZ=Asia/Taipei
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
N8N_RUNNERS_ENABLED=true
DB_TYPE=postgresdb
DB_POSTGRESDB_SCHEMA=public
DB_POSTGRESDB_DATABASE=postgres
DB_POSTGRESDB_USER=<Supabase的user>
DB_POSTGRESDB_PORT=6543
DB_POSTGRESDB_HOST=<Supabase的host>
DB_POSTGRESDB_PASSWORD=<Supabase的password>
N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
N8N_ENABLE_COMMUNITY_NODES=true
N8N_PORT=443
N8N_PROTOCOL=https
N8N_ENCRYPTION_KEY=<隨機生成的SHA256 key>
```

#### 如何生成 N8N_ENCRYPTION_KEY？

1. 前往線上 SHA256 產生器，例如：
   - https://coding.tools/tw/sha256
   - 或其他線上 SHA256 工具
2. 輸入任意隨機文字
3. 複製生成的 SHA256 雜湊值
4. 將此值填入 `N8N_ENCRYPTION_KEY`

> **重要**：請將此 key 妥善保存，遺失將無法解密已加密的資料。


### 步驟 5：部署服務

1. 確認所有設定無誤後，點擊 **「Deploy Web Service」**
2. Render 開始建置和部署 n8n（此過程約需 3-5 分鐘）
3. 部署完成後，Render 會提供一個網址（例如：`https://xxxxx.onrender.com`）

### 步驟 6：新增額外環境變數

1. 複製剛才生成的完整網址
2. 回到 Render 服務頁面，點擊左側的 **「Environment」**
3. 新增以下兩個環境變數：

```bash
WEBHOOK_URL=https://xxxxx.onrender.com
N8N_HOST=xxxxx.onrender.com
```
![設定環境變數 3](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/10.png)
![設定環境變數 4](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/11.png)
![設定環境變數 5](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/12.png)
> **注意**：`WEBHOOK_URL` 要包含 `https://`，而 `N8N_HOST` 則不包含。

4. 點擊 **「Save Changes」**
5. Render 會自動重新部署服務（約需 3-5 分鐘）

### 步驟 7：驗證安裝

1. 等待約 5 分鐘讓服務完全啟動
2. 開啟你的 n8n 網址
3. 如果看到 **「Set up owner account」** 畫面，代表安裝成功！
4. 輸入你的帳號資訊完成設定

---

## 三、設定定時排程 (cron job)

因為 Render 免費方案會在 15 分鐘無活動後自動休眠，我們需要定時「叫醒」它。

### 步驟 1：註冊 cron-job.org

1. 前往 [cron-job.org](https://cron-job.org/)
2. 註冊並登入帳號，信像驗證帳號

![信箱驗證 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/13.png)
![信箱驗證 2](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/14.png)

### 步驟 2：建立 Cronjob

1. 登入後，點擊 **「CREATE CRONJOB」**
2. 填寫以下資訊：
   - **Title**：為任務命名（例如：Keep n8n Alive）
   - **Address**：填入你的 n8n 完整網址（例如：`https://xxxxx.onrender.com`）
   - **Schedule**：選擇執行頻率
     - 點擊 **「Every 5 minutes」** 或自訂為 `*/5 * * * *`
![建立定時任務 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/16.png)
![建立定時任務 2](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/17.png)

### 步驟 3：啟用並測試

1. 確認 **「Enable job」** 選項已開啟（開關為綠色）
2. 點擊 **「Test Run」** 進行測試
3. 如果顯示綠色的 **「200 OK」**，代表連線成功
4. 點擊 **「Create」** 完成建立
![測試 1](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/18.png)
![測試 2](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/19.png)
![創建](https://github.com/yycquant0812/n8n-render-flow/blob/main/image/20.png)


### 驗證排程

- 在 cron-job.org 的控制面板可以看到執行歷史
- 確認每 5 分鐘都有成功執行記錄

---

## 常見問題

### Q1：部署後網站顯示 503 或無法連線？

**A**：這是正常現象，Render 免費方案首次啟動需要時間。請等待 5-10 分鐘後再試。

### Q2：設定 cron job 後還是會休眠？

**A**：請確認：
1. cron-job 的網址是否正確
2. Enable job 是否已開啟
3. 執行歷史是否顯示成功（200 狀態碼）

### Q3：忘記 N8N_ENCRYPTION_KEY 怎麼辦？

**A**：如果遺失 encryption key：
1. 可以在 Render 的 Environment 中查看
2. 如果已經加密資料但遺失 key，將無法解密，需要重新設定

### Q4：想要更新 n8n 版本怎麼做？

**A**：
1. 前往 Render 服務頁面
2. 點擊 **「Manual Deploy」** → **「Deploy latest commit」**
3. Render 會自動拉取最新的 `n8n:latest` 映像檔

### Q5：資料庫連線失敗？

**A**：請檢查：
1. Supabase 資料庫連線資訊是否正確
2. 確認使用的是 **Transaction pooler** 的連線參數（port 6543）
3. 檢查密碼是否正確（注意大小寫）

### Q6：Render 免費方案有什麼限制？

**A**：
- 每月 750 小時免費運行時間（足夠單一服務 24/7 運行）
- 15 分鐘無活動後會自動休眠
- 重新啟動需要 30-60 秒

---

## 進階設定（選用）

### 自訂網域

如果你有自己的網域，可以在 Render 的 **Settings** → **Custom Domain** 中設定。

### 備份工作流程

建議定期匯出你的 n8n 工作流程：
1. 進入 n8n 介面
2. 點擊右上角的設定
3. 選擇 **「Export Workflows」**

### 升級到付費方案

如果需要更穩定的服務，可以考慮 Render 的付費方案（$7/月起），優點：
- 不會自動休眠
- 更快的啟動速度
- 更多的運算資源

---

## 參考資源

- [n8n 官方文件](https://docs.n8n.io/)
- [Render 官方文件](https://render.com/docs)
- [Supabase 官方文件](https://supabase.com/docs)
- [cron-job.org 使用說明](https://cron-job.org/en/documentation/)

---

## 授權

本指南採用 MIT 授權，歡迎自由使用和分享。

## 貢獻

如有任何問題或建議，歡迎提交 Issue 或 Pull Request。

---

**祝你部署順利！如有問題，歡迎隨時詢問。**
