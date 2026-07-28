---
name: supabase-setup
description: 把 Supabase 接進 OpenCode（註冊 → 建專案 → 裝 MCP → 認證 → 建表並開啟 RLS）。使用者說「接 Supabase」「要即時資料庫」「做課堂文字雲」「加碼任務 B」時載入。三天研習第二天下午的進階選配。
---

# Supabase 連線

> **這份是寫給 AI agent 照著執行的**，不是給人讀的教學。
> 對應的人類版說明：<https://mathruffian-dot.github.io/opencode-workshop-3day/supabase.html>

---

## 你面對的使用者

台灣的國中小老師，**從沒用過資料庫**，正在三天 AI Agent 研習的第二天下午。這是**加碼選配**，他做完作品④ 之後自己來的。

- 全程**繁體中文**。**不要丟術語**：說「表格」不說「schema」，說「權限保護」再括號註明 RLS。
- 註冊、建專案、點授權這些**只能他本人做**，你負責給明確指示然後**等他回覆**。
- Windows 一律用 PowerShell，**不要用 `&&`**，用 `;`。
- **不要幫他選付費方案。** 全程 Free。

---

## 🔴 五條硬性紅線

### 1. 建完表格一定要開 RLS，而且要驗證

**這是本包最重要的一條。**

Supabase 官方行為：用 **Table Editor** 建表，RLS 預設**開啟**；用 **SQL** 建表，RLS 預設**關閉**。而**你建表一律走 SQL**。

所以每次建完表，**你必須**：

1. 立刻啟用該表的 RLS
2. 建立適當的 policy
3. **實際查詢確認 RLS 已啟用**，把結果告訴使用者
4. 請他自己去 Table Editor 看一眼有沒有「RLS enabled」標記

**沒做完這四步，不准回報「完成」。** 沒有 RLS 的 public schema 表格 = 任何拿到公開金鑰的人都讀得到裡面的學生資料。

### 2. 只存座號，不存姓名

不要建立「姓名」欄位。使用者要求要有，**主動說明風險並建議**：姓名對照表留在本機，雲端只放座號。

也不要放身分證字號、家長聯絡方式。

### 3. 設定檔要合併，不能覆蓋

寫 `~/.config/opencode/opencode.json` 時，**先讀出現有內容**，把 `supabase` 這一塊**併進**既有的 `mcp` 物件。

**絕對不要整個覆寫檔案**——使用者可能已經接了好幾個 MCP，覆蓋掉就全毀了。

### 4. URL 一定要帶 `project_ref`

官方建議的範圍限制。加了它，你只能碰這一個專案。

新手預設**再加上 `read_only=true`**，並告訴他：「先設成唯讀，我只能查不能改。等你熟悉了要我幫你寫入，再把它拿掉。」

### 5. 改完設定檔要重啟 OpenCode

**設定檔不重啟不生效。** 改完就明講：「請你關掉 OpenCode 再打開，不然這個設定不會生效。」

---

## 前置檢查

| 檢查 | 怎麼做 |
|---|---|
| 使用者有沒有 Supabase 帳號 | 直接問。沒有 → 走步驟 1 |
| 有沒有建好專案、拿到專案 ID | 直接問。沒有 → 走步驟 2 |
| `~/.config/opencode/opencode.json` 現有內容 | **先讀出來**，準備合併不是覆蓋 |

---

## 步驟

### 1. 註冊（他本人做）

給他 <https://supabase.com>，點「Start your project」，用 **個人 Gmail** 或 GitHub 登入。

⚠️ **提醒他不要用學校公務帳號**，常被管理員限制第三方登入。不綁信用卡。

**等他回報完成。**

### 2. 建立專案（他本人做）

給他這張表：

| 欄位 | 填什麼 |
|---|---|
| Organization | 用預設 |
| Name | 例如 `my-class` |
| Database Password | 自己設一組**記下來**（用 MCP 不會用到，但重設麻煩） |
| Region | `Northeast Asia (Tokyo)` |
| Pricing Plan | **Free** |

建好要等 1～2 分鐘。

然後**請他從瀏覽器網址列抄下專案 ID**：

```
https://supabase.com/dashboard/project/abcdefghijklmnop
                                      └──── 這一段 ────┘
```

**等他把 ID 給你。**

### 3. 寫設定檔（你做）

讀出現有的 `~/.config/opencode/opencode.json`，把這一塊**併進去**：

```json
"supabase": {
  "type": "remote",
  "url": "https://mcp.supabase.com/mcp?project_ref=<他的專案ID>&read_only=true",
  "enabled": true,
  "timeout": 300000
}
```

- `timeout` 必要——OpenCode **預設只等 5 秒**，資料庫操作常常不夠。
- 之後要寫入時再把 `&read_only=true` 拿掉（紅線 4）。

寫完**請他重啟 OpenCode**（紅線 5），然後等他回來。

### 4. 認證

```
opencode mcp auth supabase
```

瀏覽器會打開 → 登入 Supabase → 點 Allow。

### 5. 驗證連線

```
opencode mcp list
```

**必須看到 `✓ supabase connected`**。沒看到就照錯誤表處理，不要往下做。

### 6. 建第一張表 ＋ 開 RLS ＋ 驗證

建表（只放座號，紅線 2）→ **立刻啟用 RLS 並建 policy** → **查詢確認 RLS 真的開了** → 請他去 Table Editor 目視確認（紅線 1）。

要寫入資料的話，記得先把 `read_only=true` 拿掉並重啟。

> ⚠️ 只給寫入權限不夠，**還要給讀取**。少了讀取，資料其實進去了但畫面會報錯，使用者會以為壞掉而重複送。

---

## 要主動告知的三件事

不要等他撞牆才講：

1. **免費專案閒置 7 天會自動暫停。** 連不上不是壞掉，去 Dashboard 按 Restore 就回來，資料完整保留。
2. **免費層同時只能有 2 個 active 專案。**
3. **免費方案 500 MB 資料庫**，班級使用非常夠。

---

## 錯誤處理

| 錯誤 | 意義 | 你要做的 |
|---|---|---|
| `opencode mcp list` 沒有 supabase | 設定檔沒寫成功或沒重啟 | 重讀設定檔確認格式，請他重啟 |
| 連線逾時 | `timeout` 沒設或太短 | 確認有 `"timeout": 300000` |
| 認證後仍失敗 | OAuth 沒完成 | 重跑 `opencode mcp auth supabase`，請他確實點到 Allow |
| 寫入被拒 | `read_only=true` 還在 | 拿掉該參數並**請他重啟** |
| 資料寫進去了但讀不到 | 只給了寫入 policy | 補上讀取 policy |
| 專案連不上、Dashboard 顯示 paused | 閒置 7 天被暫停 | 請他按 Restore |

---

## 完成後回報

```
✅ Supabase 專案已連線（project_ref: xxxx，唯讀模式）
✅ 已建立表格「班級成績」，欄位：座號、國文、數學
🔒 RLS 已啟用並驗證通過
📋 待你確認：請去 Dashboard → Table Editor 看一眼，表格旁邊有沒有「RLS enabled」

提醒：
- 免費專案閒置 7 天會暫停，按 Restore 就回來
- 目前是唯讀模式，要我寫入資料的話告訴我，我幫你改設定（要重啟）
```

**然後停下來。**

---

## 不要做的事

- ❌ **不要在沒開 RLS 的情況下回報完成**（最嚴重的一條）
- ❌ 不要整個覆寫 `opencode.json`
- ❌ 不要建立姓名、身分證字號、家長聯絡方式欄位
- ❌ 不要幫他選付費方案
- ❌ 不要省略 `project_ref`
- ❌ 改完設定檔不要忘記提醒重啟
