---
name: slide-setup
description: 幫老師安裝並使用兩包免費簡報技能——HTML 互動簡報（雙擊就播）與可編輯 .pptx（SOIL 教學版型）。使用者說「安裝簡報技能」「我要做簡報」「把教材做成投影片」「做成 PPT」「做互動簡報」時載入。三天研習第三天的帶得走技能。
---

# 簡報技能安裝與使用

> **這份是寫給 AI agent 照著執行的**，不是給人讀的教學。
> 對應的人類版說明：<https://mathruffian-dot.github.io/opencode-workshop-3day/slides.html>

---

## 你面對的使用者

台灣的國中小老師，**零程式基礎**，正在三天 AI Agent 研習的第三天。這兩包是**帶回學校長期用**的東西，不是當天的作品。

- 全程**繁體中文**，不要丟術語。
- Windows 一律用 **PowerShell**，**不要用 `&&` 串指令**（PowerShell 5.1 會語法錯誤），用 `;`。
- 兩包都**零金鑰**。使用者問到要不要申請 OpenAI 帳號時，明確回答**不用**。

---

## 兩包差在哪（先問清楚他要哪一種，不要兩包都裝了才問）

| | `html-slide-builder-lite` | `soil-teaching-deck` |
|---|---|---|
| 產出 | 單一 `index.html` | 可編輯 `.pptx` |
| 怎麼播 | **雙擊，瀏覽器全螢幕** | PowerPoint 開 |
| 能不能事後改 | 要改程式碼 | **PowerPoint 直接改** |
| 互動 | ✅ 滑桿、揭露效果 | ❌ |
| 額外需求 | Python | Python **＋ Node.js** |

**判斷法**：他說「上課直接播」→ HTML。他說「回去還要改」「要給同事」「學校要收檔」→ pptx。**兩個都要也可以**，兩包不衝突。

---

## 🔴 四條硬性紅線

### 1. 跑 `install.py` 之前，一定要先手動建好 OpenCode 的技能資料夾

```powershell
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
```

**這一步不能跳，跳了會靜默失敗。**

理由（2026-07-29 實測）：`install.py` 的 `find_skills_dir()` 是「**取第一個存在的資料夾**」，順序是
`~/.claude/skills` → `~/.claude-skills` → `~/.config/opencode/skills` → `~/.agents/skills`；
**四個都不存在時，它會建 `~/.claude/skills` 並裝進去，然後印出「✔ 已安裝！」**。

老師的機器只裝了 OpenCode，四個通常都不存在 → **裝進 Claude Code 的資料夾，畫面顯示成功，OpenCode 永遠讀不到**。
先把 OpenCode 的資料夾建出來，它就會正確落在那裡。

### 2. 一律用 PowerShell 跑 `install.py`，不要用 bash

Git Bash 的 stdout 在繁體中文 Windows 是 **cp950**，`install.py` 裡的 `✔` 會直接
`UnicodeEncodeError` 崩掉（實測）。PowerShell 沒有這個問題。

真的只能用 bash 時，前面加 `PYTHONUTF8=1`。

### 3. 裝完一定要「看檔案」驗證，不要相信畫面上的成功訊息

見下方步驟 3。**紅線 1 的失敗模式就是「畫面說成功但裝錯地方」**，所以唯一可信的是檔案真的在不在。

### 4. 不要去申請任何 API 金鑰

這兩包是刻意挑的**零金鑰**版本。底圖用 CSS 漸層、圖標用 emoji／SVG、pptx 用 PptxGenJS 原生繪製。
使用者如果說「網路上教學說要 OpenAI Key」，告訴他**那是完整版，我們用的是研習簡易版，不需要**。

---

## 前置檢查（依序跑，任一不過就停下來說）

| 檢查 | 怎麼做 | 不過的話 |
|---|---|---|
| Python | `python --version` | 要 3.8 以上。D1 的 `00-env-setup` 已裝過，多半是走個確認 |
| git | `git --version` | D1 已裝過 |
| Node.js（**只有裝 pptx 那包才要**）| `node --version` | D2 教 clasp 時已裝過。沒有 → 問他「要我幫你裝 Node.js LTS 嗎？」 |

---

## 步驟

### 1. 建資料夾（紅線 1）

```powershell
New-Item -ItemType Directory -Force "$HOME\.config\opencode\skills"
```

### 2. 下載並安裝

先問他要裝在哪個資料夾，**不要裝在 Google 雲端硬碟裡**（同步會咬住檔案）。建議 `$HOME\agent-skills\`。

HTML 那包：

```powershell
git clone https://github.com/mathruffian-dot/claude-html-slide-builder-lite.git
Set-Location claude-html-slide-builder-lite
python install.py
```

pptx 那包：

```powershell
git clone https://github.com/mathruffian-dot/soil-teaching-deck.git
Set-Location soil-teaching-deck
python install.py
```

### 3. 驗證（🔴 這一步不要跳過）

```powershell
Get-ChildItem "$HOME\.config\opencode\skills" -Directory | Select-Object Name
```

- 看到 `html-slide-builder-lite` 或 `soil-teaching-deck` → ✅ 成功
- **只看到別的、或看到空的** → ❌ 去 `$HOME\.claude\skills` 找，找到就整個資料夾搬過來：

```powershell
Move-Item "$HOME\.claude\skills\html-slide-builder-lite" "$HOME\.config\opencode\skills\"
```

### 4. 請他重啟 OpenCode

**技能是啟動時載入的，不重啟不會生效。** 這句要主動講，不然他會以為裝壞了。

### 5. 實際做一份給他看

**停下來問他要用哪份教材**，不要拿範例主題交差。拿到教材後：

- HTML：做完直接**雙擊 `index.html`** 開給他看
- pptx：做完用 PowerPoint 開起來，**逐頁確認有沒有文字溢出**

---

## 錯誤處理

| 錯誤 | 意義 | 你要做的 |
|---|---|---|
| 畫面印「✔ 已安裝」但 OpenCode 找不到技能 | 🔴 **裝到 `~/.claude/skills` 了**（紅線 1） | 照步驟 3 搬過去，再重啟 |
| `UnicodeEncodeError: 'cp950'` | 用 bash 跑了（紅線 2） | 改用 PowerShell |
| 技能裝好了但叫不動 | 沒重啟 OpenCode | 請他關掉重開 |
| `node: 找不到指令`（做 pptx 時） | 沒裝 Node | 回前置檢查 |
| pptx 打開是空白／破圖 | PptxGenJS 版本或字體問題 | 先確認 Node 版本，再重跑一次；兩次不過就改用 HTML 那包 |
| 使用者問「圖片怎麼不是 AI 生的」 | 這是刻意的 | 說明：零金鑰版用 CSS 漸層＋emoji。要 AI 圖就自己去 Gemini／ChatGPT 網頁版生好再插入 |

**同一個錯誤重試兩次還不過 → 停下來問使用者，不要繼續耗。**

---

## 完成後回報

```
✅ 已安裝：html-slide-builder-lite（或 soil-teaching-deck）
✅ 位置：C:\Users\<你>\.config\opencode\skills\...
📋 待你確認：請關掉 OpenCode 重開一次，技能才會載入
📋 重開後跟我說「把這份教材做成互動簡報」，我就幫你做

下一步建議：
- 兩包可以都裝，上課播用 HTML、要給同事改用 pptx
```

**然後停下來。** 不要自作主張直接開始做簡報——先讓他確認技能載入了。

---

## 不要做的事

- ❌ **不要在沒建 `~/.config/opencode/skills` 的情況下跑 `install.py`**
- ❌ 不要相信畫面上的「✔ 已安裝」就宣告完成——一定要看檔案
- ❌ 不要用 bash 跑 `install.py`
- ❌ 不要把 repo clone 到 Google 雲端硬碟
- ❌ 不要引導使用者去申請 OpenAI／Firebase 金鑰
- ❌ 不要用完整版 `claude-html-slide-builder`（要金鑰），我們用的是 `-lite`
- ❌ 不要在使用者還沒重啟 OpenCode 之前，就說技能可以用了
