# LAVA · Living Dashboard

對外的即時儀表板：白底日間設計，會自己保持更新，並支援手動增刪改 + 自動待審清單。
> 詳細部署/API 步驟看 **`部署與自動更新_完整教學.md`**（給不熟 GitHub 的人）。

## 它由三個資料檔驅動（頁面會 fetch 並合併）
| 檔案 | 內容 | 誰更新 |
|---|---|---|
| `data.json` | 產品指標（WAU/MAU/配對/成交/留存）+ 今日焦點 | GitHub Action（每 3 小時，server 端）|
| `inbox.json` | 待審清單：從 ClickUp 頻道 + Drive 會議記錄 AI 判別的 任務/Bug/他部門 | 桌面排程 `lava-inbox-ingest`（每日 07:30）|
| `board.json` | 你手動管理的 任務 / 目標 / Agenda | 你在頁面按「⬆ 同步雲端」寫回 |

## 三種功能對應
- **看（live 指標）**：`data.json`，頂部顯示最後更新時間 + 綠色 LIVE 燈（<26h）。頁面每 60 秒自動重抓。
- **改（手動 CRUD）**：任務/目標/Agenda 可新增/編輯/刪除，先存本機瀏覽器（localStorage），按「⬆ 同步雲端」推成 `board.json`（跨裝置一致）；換裝置按「⬇ 從雲端載入」。
- **收（自動待審）**：每天自動把可能的任務/Bug/他部門事項放進「📥 待審清單」，你按「✓ 採納」才進任務看板，「✕ 略過」則不再出現。

## 一次性部署（GitHub Pages，免終端機）
見《部署與自動更新_完整教學.md》§1。重點：把 `lava-live` 的**內容**（不是資料夾本身）上傳到 repo 根目錄 → Settings → Pages → main /root。預設密碼 `lava2026`。

## 讓三個自動更新都動起來
1. **指標（data.json）**：在 repo Settings → Secrets → Actions 加 `POSTHOG_API_KEY`、`POSTHOG_PROJECT_ID=98614`、`CLICKUP_API_TOKEN`、`CLICKUP_LIST_ID=901818809467`。Action 會每 3 小時更新。
2. **待審清單（inbox.json）**：由桌面版 Cowork 排程 `lava-inbox-ingest` 產生。要讓它自動推上 GitHub，在 `lava-live/` 放一個 **`.sync.json`**（已被 .gitignore 忽略、不會公開）：
   ```json
   { "owner": "你的GitHub帳號", "repo": "lava-dashboard", "branch": "main", "token": "github_pat_..." }
   ```
   token 用 GitHub **fine-grained token**，只授權該 repo 的 **Contents: Read and write**。沒放 `.sync.json` 也沒關係，排程會把 inbox.json 更新在本機資料夾，你再手動上傳。
3. **手動看板（board.json）**：在頁面「⚙ 同步設定」填 owner/repo/branch/token（同一顆 fine-grained token），之後「⬆ 同步雲端」即可。token 只存在你這台裝置的瀏覽器。

> 安全：`data.json` / `inbox.json` / `board.json` 都是公開可讀，所以**只放聚合指標、任務標題、你自己的待辦**，不放姓名/電話/客戶個資。前端密碼僅為輕度遮蔽。

## 換密碼
密碼 SHA-256 存在 `index.html`（搜 `PW_HASH`）。`printf '新密碼' | shasum -a 256` 取雜湊貼回，或叫 Claude 幫你算。

## 本機預覽
別直接雙擊（`file://` 會擋 fetch）。`cd lava-live && python3 -m http.server 8080` → 開 http://localhost:8080（fetch 失敗會用內建種子資料顯示）。

## 檔案結構
```
lava-live/
├─ index.html                       # 看板：密碼門檻 + 指標 + CRUD + 待審 + 雲端同步
├─ data.json                        # 指標（GitHub Action 更新）
├─ inbox.json                       # 待審清單（排程 lava-inbox-ingest 更新）
├─ board.json                       # 你的任務/目標/agenda（同步雲端後產生）
├─ .sync.json                       # （你自建、被 gitignore）排程推 inbox 用的 token
├─ DESIGN_PHILOSOPHY.md             # 設計哲學
├─ README.md / 部署與自動更新_完整教學.md
├─ scripts/refresh_data.py          # PostHog + ClickUp → data.json
└─ .github/workflows/refresh.yml    # 每 3 小時自動刷新 data.json
```
