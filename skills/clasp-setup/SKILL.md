---
name: clasp-setup
description: 把 Google Apps Script 接進 OpenCode（clasp 登入 → 建綁定試算表的專案 → 部署成網頁應用程式）。使用者說「接 Apps Script」「clasp 登入」「做課堂提問箱」「作品④」時載入。三天研習第二天下午的全班主線。
---

# Apps Script ＋ clasp 連線

> **這份是寫給 AI agent 照著執行的**，不是給人讀的教學。
> 對應的人類版說明：<https://mathruffian-dot.github.io/opencode-workshop-3day/apps-script.html>

---

## 你面對的使用者

台灣的國中小老師，**零程式基礎**，正在三天 AI Agent 研習的第二天下午做**作品④**。這是全班主線，**現場約 15 分鐘要跑完設定**，講師在顧整場節奏。

- 全程**繁體中文**，不要丟術語。
- 需要他本人操作的地方（選帳號、點允許、去日曆或試算表確認），**停下來講清楚，等他回覆**。
- Windows 一律用 PowerShell，**不要用 `&&` 串指令**（PowerShell 5.1 會語法錯誤），用 `;`。

---

## 🔴 五條硬性紅線

### 1. 一定要用個人 Gmail

`clasp login` 時**主動提醒他選個人 Gmail，不要選學校公務帳號**。

學校 Workspace 帳號會回 `admin_policy_enforced`，那是**校方管理員才能解的**（要進 Admin Console 把 clasp 的 OAuth Client ID 加白名單）。使用者自己弄不了，不要讓他在那邊試。

### 2. 用 `npx`，不要全域安裝

一律 `npx @google/clasp ...`。

理由：不用先安裝，而且**繞開 Windows 的執行原則限制**。使用者主動要求常用時，才建議 `npm install -g @google/clasp`，並準備處理執行原則。

### 3. 卡超過 15 分鐘，主動提出退路

如果 `clasp login` 或 API 授權反覆失敗，**不要繼續重試迴圈**。主動說：

> 「clasp 這條卡住了。我們換一條——我直接把程式碼給你，你貼到 script.google.com 就好，做出來的東西完全一樣，只是之後每次改都要重貼一次。要換嗎？」

**clasp 是效率升級，不是作品④ 的必要條件。** 保住作品優先。

### 4. 只用座號，不用姓名

試算表欄位不要有學生真名。使用者提供的資料裡有姓名欄，**主動指出並建議移除**。
作品④ 的網址等一下要發給全班，那是公開網頁。

### 5. 不要覆蓋既有的 clasp 專案

執行 `clasp create` 前先確認當前資料夾**沒有** `.clasp.json`。有的話先問使用者這是不是他要接的專案，不要直接蓋掉。

---

## 前置檢查（依序跑，任一不過就停下來說）

| 檢查 | 怎麼做 | 不過的話 |
|---|---|---|
| Node 版本 | `node --version` | 要 **v22 以上**。低於或找不到 → 問他「要我幫你裝 Node.js LTS 嗎？」<br>（D1 的 `00-env-setup` 懶人包已裝過，多半是走個確認） |
| Apps Script API 開關 | 無法用指令查，**直接問使用者** | 「昨晚的功課有把 Apps Script API 開關打開嗎？」<br>沒有 → 給他 <https://script.google.com/home/usersettings>，**告訴他要等 1–2 分鐘才生效**，這段時間先做別的 |
| 資料夾 | 當前目錄是不是他的專案資料夾 | 不是 → 先確認要在哪裡建 |

---

## 步驟

### 1. clasp 登入

```
npx @google/clasp login
```

**執行前先講**：「等一下瀏覽器會打開，**請選你的個人 Gmail**，然後點允許。」

### 2. 驗證登入

```
npx @google/clasp login --status
```

看到帳號 = 成功。**沒看到就不要往下做**，先照錯誤表處理。

### 3. 建立綁定試算表的專案

問使用者要叫什麼名字，然後建立綁定試算表（container-bound）的專案。

### 4. 產出程式碼並推上去

作品④ 的預設題目是**課堂提問箱**：學生用手機填座號與問題，資料進試算表，老師能看到全部。

- 架構用**純 GAS ＋ `google.script.run` 同源溝通**，不要用 `fetch` 打自己的 `/exec`（會有 CORS 問題）。
- `clasp push` 推上去。

### 5. 部署為網頁應用程式

部署後拿到 `.../exec` 網址，**產生 QR Code** 給他。

⚠️ 首次部署會跳授權，**要先跟他說**：

> 「會跳一個紅色警告說『Google 尚未驗證這個應用程式』。那是正常的——那支腳本就是你自己寫的。請點左下角『進階』→『前往〈你的專案名〉（不安全）』→ 允許。」

### 6. 請他實測

**停下來**，請他用手機掃 QR Code 填一筆，然後去試算表確認資料有進去。**等他回報再繼續。**

---

## 錯誤處理

| 錯誤 | 意義 | 你要做的 |
|---|---|---|
| `admin_policy_enforced` | 用到學校帳號 | `npx @google/clasp logout`，重來並**強調選個人 Gmail** |
| `User has not enabled the Apps Script API` | 開關沒開或**還沒生效** | 給設定頁網址，請他開，**等 1–2 分鐘**再重試 |
| 「因為這個系統上已停用指令碼執行」 | Windows 執行原則 | 用 `npx` 而非全域安裝即可繞開；若已全域安裝，用 `Set-ExecutionPolicy -Scope Process -ExecutionPolicy RemoteSigned` |
| 「Google 尚未驗證這個應用程式」 | 正常，自己的腳本 | 引導：進階 → 前往〈專案名〉（不安全）→ 允許 |
| `node: 找不到指令` | 沒裝 Node | 回前置檢查 |
| 指令名稱找不到 | clasp v3 改過指令名（`open` → `open-script` 等） | 以 <https://github.com/google/clasp> 為準 |

**同一個錯誤重試兩次還不過 → 走紅線 3 的退路，不要繼續耗。**

---

## 完成後回報

```
✅ clasp 已登入（帳號：xxx@gmail.com）
✅ 專案已建立並綁定試算表「課堂提問箱」
✅ 已部署，網址：https://script.google.com/.../exec
✅ QR Code 已產生
📋 待你確認：用手機掃一下填一筆，看試算表有沒有進資料

下一步建議：
- 做完可以往上一階：加碼任務 A（行事曆與寄信）
- 收工前記得 push 到 GitHub
```

**然後停下來。** 不要自作主張繼續做加碼任務或改功能——研習現場是跟著講師進度走的。

---

## 不要做的事

- ❌ 不要用學校 Google 帳號登入
- ❌ 不要在 `clasp login` 失敗時無限重試——兩次不過就提退路
- ❌ 不要把學生姓名寫進試算表或程式碼
- ❌ 不要用 `fetch` 打自己的 `/exec`（用 `google.script.run`）
- ❌ 不要主動幫他建定時觸發器
- ❌ 不要在使用者還沒確認「資料真的有進試算表」之前，就宣告完成
