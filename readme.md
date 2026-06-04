# 🛍️ Yulu 小舖 — Yulu Store

一個純前端單頁電商應用，具備賣家市集、訂單管理、商品評價、拉霸小遊戲等功能 — 所有資料皆透過 `localStorage` 儲存在用戶端，無需後端伺服器。

部署於 GitHub Pages：**[wuulu.github.io/yulu-store](https://wuulu.github.io/yulu-store)**

---

## 功能列表

### 買家功能
- **商品瀏覽** — 網格佈局，支援關鍵字搜尋與賣家篩選（全部商品 / 官方 / 各賣家）
- **商品詳情** — 彈窗檢視圖片、說明、評分、評價列表
- **購物車** — 側滑面板，數量增減，折價券套用
- **結帳流程** — 收件資訊、付款方式（信用卡 / ATM 轉帳 / 貨到付款）、信用卡號格式化
- **訂單查詢** — 檢視歷史訂單，每件商品獨立顯示狀態
- **完成訂單與評價** — 已出貨商品可點擊「完成訂單」後留下評分與文字評價

### 賣家功能
- **成為賣家** — 在賣家後台點擊按鈕即可註冊為賣家
- **商品管理** — 新增 / 編輯 / 刪除商品，可設定名稱、價格、描述、初始銷量，並上傳商品圖片
- **訂單管理** — 檢視包含自己商品的所有訂單，可將商品標記為「出貨」
- **商品圖片** — 透過檔案選擇器上傳，以 base64 格式儲存，在商品卡片、詳情頁、購物車、賣家商品列表皆會顯示

### 迷你遊戲 🎰
- **每日免費拉霸** — 每天一次遊玩機會
- 6 種表情符號，三軸相同即中獎，隨機獲得一張折價券
- 中獎折價券自動套用到購物車，重新整理頁面後仍保留（結帳或登出時清除）

### 多語系
- 支援 **繁體中文 (zh)**、**English (en)**、**日本語 (ja)**
- 頁首語言切換按鈕，所有 UI 文字即時響應

### 折價券一覽

| 代碼 | 折扣金額 | 最低消費 |
|------|----------|----------|
| `SAVE50` | NT$50 | NT$200 |
| `VIP100` | NT$100 | NT$500 |
| `WELCOME` | NT$80 | NT$300 |
| `FREESHIP` | NT$60 | NT$0 |
| `LUCKY30`（遊戲） | NT$30 | NT$100 |
| `GAME50`（遊戲） | NT$50 | NT$200 |
| `JACKPOT`（遊戲） | NT$100 | NT$400 |

---

## 技術架構

| 層級 | 技術 |
|------|------|
| **語言** | 原生 JavaScript (ES6) |
| **標記** | HTML5 |
| **樣式** | CSS3（單一 `<style>` 區塊，自適應 RWD，無框架） |
| **儲存** | `localStorage`（JSON 序列化） |
| **圖示** | Emoji |
| **部署** | GitHub Pages（`gh-pages` 分支） |

零外部依賴 — 無 jQuery、無 React、無後端。

---

## 架構說明

### 資料流程（100% 客戶端）

```
使用者操作 → JavaScript 函式 → localStorage (讀/寫) → 重新渲染 UI
```

每次資料異動都會寫入 `localStorage` 並重新渲染對應的 DOM 區塊。

### localStorage 資料結構

| Key | 類型 | 說明 |
|-----|------|------|
| `store_users` | `Object<email→user>` | 使用者帳號 `{ name, email, password, isSeller }` |
| `store_session` | `sessionStorage` | 當前登入的使用者 Email |
| `store_lang` | `string` | 當前語言 (`zh` / `en` / `ja`) |
| `store_cart_{email}` | `Object<id→qty>` | 每位使用者的購物車內容 |
| `store_seller_products` | `Array<product>` | 賣家建立的商品（全域） |
| `store_orders_{email}` | `Array<order>` | 每位使用者的訂單紀錄 |
| `store_reviews_{productId}` | `Array<review>` | 各商品的評價列表 |
| `store_extra_sold` | `Object<id→number>` | 官方商品的累計銷售增量 |
| `store_game_play_{email}` | `string` | 最後遊玩日期（日期字串） |
| `store_game_coupon_{email}` | `object` | 遊戲獲得的折價券物件 |

### 核心函式

| 函式 | 用途 |
|------|------|
| `getAllProducts()` | 合併官方商品與賣家商品為單一陣列 |
| `getProductById(id)` | 在合併清單中依 ID 查找商品 |
| `pn(product, field)` | 依目前語言解析多語系欄位（`name` / `desc`） |
| `t(key, vars)` | 多語系字串查詢，支援變數替換 |
| `renderProducts()` | 渲染商品網格，包含搜尋與賣家篩選 |
| `updateCart()` | 重新計算購物車 UI、金額、折價券狀態 |
| `renderOrderHistory()` | 渲染買方訂單列表，每項商品顯示狀態 |

### 賣家商品 ID 規則
- 官方商品：ID 1～8
- 賣家商品：從 **1000** 開始自動遞增（不會與官方衝突）

### 商品圖片處理
- 賣家透過 `<input type="file">` 選擇圖片 → `FileReader.readAsDataURL()` → base64 字串存入商品 `image` 欄位
- 顯示時檢查 `p.image`：若有值則以 `background-image` 呈現於縮圖 / 完整尺寸元素；否則退回顏色背景 + Emoji

---

## 使用者流程

### 購物流程
```
開啟頁面 → 瀏覽商品（搜尋 / 篩選）→ 點擊商品 → 檢視詳情 →
加入購物車 → 開啟購物車 → 套用折價券 → 結帳 → 填寫收件資訊 →
選擇付款方式 → 送出訂單 → 訂單成立確認 → 清空購物車
```

### 賣家流程
```
登入 → 點擊 🏪 賣家中心 → 點擊「成為賣家」→
切換「商品管理」分頁 → 新增 / 編輯 / 刪除商品（可上傳圖片）→
切換「訂單管理」分頁 → 檢視含有自己商品的訂單 → 點擊「出貨」
```

### 訂單生命週期
```
待出貨（買方看到 ⏳）→ 賣家點擊「出貨」→ 已出貨（買方看到 ✅ +「完成訂單」）→
買方點擊「完成訂單」→ 內嵌評價表單 → 送出 → 已完成（狀態 ✅ + 評價已儲存）
```

### 遊戲流程
```
點擊 🎰 → 開啟拉霸機 → 點擊「旋轉」→ 1.5 秒動畫 →
中獎（三軸相同）：隨機獲得折價券，自動套用到購物車 →
沒中獎：顯示「明天再來」→ 下次造訪若還有未使用的折價券則顯示提示
```

---

## 部署方式

透過 `gh-pages` 分支部署至 GitHub Pages：

```
git checkout -b gh-pages
git push origin gh-pages
```

儲存庫：`github.com/wuulu/yulu-store.git`  
線上網址：`https://wuulu.github.io/yulu-store/`

---

## 檔案結構

```
yulu-store/
├── index.html          # 單一檔案 SPA（HTML + CSS + JS，約 1800 行）
├── README.md           # 本說明文件
```

---

## 開發備註

- 整份應用為**單一 HTML 檔案** — 無需建置工具或打包器。
- 所有文字內容皆透過 `t()` 翻譯函式輸出；新增字串至 `langData` 物件即可完整支援多語系。
- 官方 `products` 陣列中的商品使用 `{ zh, en, ja }` 多語系物件儲存名稱與描述。
- 賣家商品的 name / desc 儲存為純字串（非多語系物件），遊戲折價券亦同。
- 新增折價券：全域折價券加入 `coupons` 陣列，遊戲獎勵折價券加入 `gameCoupons` 陣列。
