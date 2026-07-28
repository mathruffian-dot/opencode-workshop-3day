---
name: video-specs
description: 帶老師用 claude-video-specs-lite 做影片（活動紀錄／教學影片／社群科普三類），零金鑰、免費 Edge-TTS 旁白。使用者說「我要做影片」「做一支教學影片」「把這個做成短片」「啟動影片規範」時載入。三天研習第三天的帶得走技能。
---

# 影片製作（repo 模式）

> **這份是寫給 AI agent 照著執行的**，不是給人讀的教學。
> 對應的人類版說明：<https://mathruffian-dot.github.io/opencode-workshop-3day/video.html>

---

## 你面對的使用者

台灣的國中小老師，**零程式基礎**，正在三天 AI Agent 研習的第三天。這是**帶回學校長期用**的東西。

- 全程**繁體中文**，不要丟術語。
- Windows 一律用 **PowerShell**，**不要用 `&&` 串指令**，用 `;`。
- 這一包**零金鑰**：圖用 Unsplash 免費直連，旁白用免費 Edge-TTS。

---

## 🔴 這一包最重要的一件事：走 repo 模式，不要裝技能

**做法**：把 repo clone 下來，**在那個資料夾裡開 OpenCode**，然後跟 agent 說「我要做影片」。
repo 根目錄的 `opencode.json` 會自動把 `AGENTS.md` 載進來，agent 依五階段帶著使用者跑。

**不要跑 `install/pack_skill.sh`。** 理由有兩個，都是實測出來的：

1. **它產出的 SKILL.md 裡寫死 repo 的絕對路徑。** repo 一旦搬家、改名或刪掉，技能就壞了，
   而且壞得很安靜——agent 只會說「找不到規範」。老師整理電腦時一定會踩到。
2. **它的用法說明是錯的。** 腳本註解寫 `--target opencode`（空格），但解析器只吃
   `--target=opencode`（**等號**）。照文件寫法會 **靜默裝到 `~/.claude/skills/`**，
   畫面還會印「目標 agent : claude」——不盯著看就過去了。

> `AGENTS.md` 階段 5 會問「要不要打包成技能」。**研習現場一律回答不要**（選項 4）。
> 那裡寫的「打包成 OpenCode plugin（`.opencode/plugin/`）」也和腳本實際行為不符，不要照著做。
> 使用者回校後自己真的想打包，才告訴他正確寫法是 `--target=opencode`（等號），
> 而且 **repo 從此不能搬家**。

---

## 🔴 另外三條紅線

### 1. 不要 clone 到 Google 雲端硬碟

Playwright 的 `node_modules` 放在雲端硬碟裡會被同步咬住，渲染直接失敗（repo 的 `GOTCHAS.md` D-1 有記）。
clone 到 `$HOME\video\` 之類的本機路徑。**渲染用的 Playwright 一律裝在 `%TEMP%\cvs-render\`**。

### 2. 腳本與視覺規範沒給使用者確認過，不准開始寫 code

這是 repo 自己列為「最高安全防線」的一條（`GOTCHAS.md` A-1／A-2／A-3）。順序是：

**`SCRIPT.md`（旁白＋字卡＋分鏡）→ 使用者確認 → `DESIGN.md`（字體／配色／節奏）→ 使用者確認 → 才動工。**

跳過的下場是做到一半整支重做，現場沒有那個時間。

### 3. 環境裝不起來就換題目，不要卡在安裝

這一包的安裝負擔是三包裡最重的（見下表）。**ffmpeg 或 Playwright 卡超過 15 分鐘就主動提退路**：

> 「渲染環境這條卡住了。我們改成只把腳本跟畫面做出來——你會有一份完整的 `SCRIPT.md` 和一個能在瀏覽器播放的 `index.html`，回學校再補渲染。要這樣嗎？」

**做出東西優先。** 沒渲染成 mp4 不代表沒有作品。

---

## 前置檢查（照 AGENTS.md 階段 1，把結果整理成表格給他看）

| 元件 | 檢查指令 | 必要 | 備註 |
|---|---|---|---|
| Python 3.8+ | `python --version` | ✅ | D1 已裝 |
| edge-tts | `pip show edge-tts` | ✅ 旁白 | 沒有 → `pip install edge-tts` |
| Node.js 18+ | `node --version` | ✅ Playwright | D2 教 clasp 時已裝 |
| **ffmpeg** | `ffmpeg -version` | ✅ 音視合成 | 🔴 **前兩天都沒裝過，這是新的**。`winget install Gyan.FFmpeg` |
| **Playwright** | 看 `%TEMP%\cvs-render\node_modules\playwright` | ⚠️ 只有要渲染才需要 | 🔴 同上，這是新的 |
| 源石黑體 | 看 `~\AppData\Local\Microsoft\Windows\Fonts\GenSekiGothic2TW-H.otf` | 視覺一致 | 免費開源，缺了就自動裝，不用問 |

**ffmpeg 和 Playwright 是這一包唯二的新安裝負擔，先講清楚再開始，不要裝到一半才說。**

---

## 步驟

### 1. clone 並在裡面開 OpenCode

```powershell
Set-Location $HOME
git clone https://github.com/mathruffian-dot/claude-video-specs-lite.git video
Set-Location video
```

然後請他**在這個資料夾重新開一個 OpenCode 專案**，`opencode.json` 才會生效。

### 2. 跟著 AGENTS.md 的五階段跑

| 階段 | 做什麼 | 停不停 |
|---|---|---|
| 0 | 識別 agent 環境（你是 OpenCode） | — |
| 1 | 環境檢查（上面那張表） | 🛑 缺元件要逐項問他裝不裝 |
| 2 | 介紹三類影片，讓他選 | 🛑 等他選 |
| 3 | 試作：`SCRIPT.md` → 確認 → `DESIGN.md` → 確認 → 實作 | 🛑 **兩個確認點都要停** |
| 4 | 調整 | 🛑 等他看過成品 |
| 5 | 打包成技能 | **一律選 4（不用）**，見上方紅線 |

**每階段都要等他回覆再往下，不要一口氣跑完五階段。**

### 3. 三類影片怎麼選

| 類型 | 片長 | 適合老師的什麼場景 |
|---|---|---|
| 01 活動紀錄 | 60–180 秒 | 校慶、運動會、畢業典禮、比賽紀錄 |
| 02 教學影片 | 4–8 分鐘 | 學科概念講解、翻轉教室的課前影片 |
| 03 社群科普 | 2–3 分鐘 | 班級經營宣導、給家長看的說明短片 |

**研習現場建議選 01 或 03**（短、做得完）。02 教學影片留給回校後做。

---

## 錯誤處理

| 錯誤 | 意義 | 你要做的 |
|---|---|---|
| `npm install` 失敗／Playwright 壞掉 | 裝在雲端硬碟了 | 改裝到 `%TEMP%\cvs-render\`（GOTCHAS D-1）|
| Edge-TTS 中途斷線 | 並行請求被擋 | **序列執行 ＋ retry 3 次**，不要並行 |
| `ffmpeg: 找不到指令` | 沒裝 | `winget install Gyan.FFmpeg`，裝完要**重開終端機** |
| 影片開頭有遮罩殘影 | 錄影模式沒開對 | 用 `?render=true` 模式 |
| 字體變成預設黑體 | 源石黑體沒裝或路徑錯 | 跑 `python install/setup.py fonts`；範例 HTML 的 `@font-face` 相對路徑是 `../../assets/fonts/` |
| 打包完技能叫不動 | 🔴 用了 `--target opencode`（空格），裝到 `~/.claude/skills` | 本來就不該打包；真要打包用 `--target=opencode` |

**同一個錯誤重試兩次還不過 → 走紅線 3 的退路。**

---

## 完成後回報

```
✅ repo 已 clone 到：C:\Users\<你>\video
✅ 環境：Python / edge-tts / Node / ffmpeg / Playwright（逐項打勾或標缺）
✅ 影片類型：01 活動紀錄（或 02 / 03）
✅ SCRIPT.md 已產出
📋 待你確認：先看過腳本，確認了我才做視覺規範

⚠️ 沒有打包成技能——這是刻意的。要用的時候，
   在 C:\Users\<你>\video 這個資料夾開 OpenCode，說「我要做影片」就會啟動。
```

**然後停下來。**

---

## 不要做的事

- ❌ **不要跑 `install/pack_skill.sh`**（絕對路徑寫死 ＋ 用法說明是錯的）
- ❌ 不要在 `AGENTS.md` 階段 5 選「打包成 Claude Skill」或「OpenCode plugin」
- ❌ 不要把 repo clone 進 Google 雲端硬碟
- ❌ 不要把 Playwright 的 `node_modules` 裝在 repo 或雲端硬碟裡
- ❌ **不要在使用者確認 `SCRIPT.md` 和 `DESIGN.md` 之前開始寫 code**
- ❌ 不要並行呼叫 Edge-TTS
- ❌ 不要用完整版 `claude-video-specs`（要 OpenAI 生圖 ＋ 付費 TTS），我們用的是 `-lite`
- ❌ 不要拿範例主題交差——用他自己的真實素材
