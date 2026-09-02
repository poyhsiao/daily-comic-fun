# 漫畫人生 — 設定總覽

> 建立時間：2026-09-02
> 更新時間：2026-09-02

---

## 一、儲存架構

```
daily-comic-fun/          ← Git repo（poyhsiao/daily-comic-fun）
│
├── books/
│   ├── my-comic-life/    ← 書籍根目錄（主要工作區）
│   │   ├── index.html
│   │   ├── series-manifest.json
│   │   └── publication-profile.json
│   ├── episodes/         ← 所有章節的獨立資料夾
│   │   ├── {episodeId}/
│   │   │   ├── cover.png
│   │   │   ├── beat-*.png
│   │   │   ├── episode.json
│   │   │   ├── index.html
│   │   │   └── narration-*.ogg
│   │   └── ...
│   ├── monthly-volumes/  ← 月冊（連續版 + 精裝版）
│   │   └── {YYYY-MM}/
│   │       ├── index.html
│   │       ├── volume-manifest.json
│   │       └── continuous-edition/
│   │           ├── index.html
│   │           ├── book-data.json
│   │           └── assets/{episodeId}/
│   ├── parts/            ← 季度合集
│   └── annuals/         ← 年度合集
└── runtime/
    ├── locks/            ← 鎖檔（防止併發寫入）
    └── quarantine/       ← 隔離區（待確認素材）
```

---

## 二、publication-profile.json（書籍設定）

**路徑：** `books/my-comic-life/publication-profile.json`

```json
{
  "schemaVersion": "1.1.0",
  "bookId": "my-comic-life",
  "title": "我的漫畫人生",
  "style": {
    "id": "02-snow-pastel",
    "lifecycle": "validated_preset"
  },
  "character": {
    "mode": "none",
    "ids": []
  },
  "publication": {
    "primary": "local-html",
    "localRoot": "/Users/kimhsiao/git/kimhsiao/daily-comic-fun"
  },
  "visualFallback": null,
  "continuity": "weak",
  "episodeCover": true,
  "budget": "standard"
}
```

**寫入規則：**
- `localRoot` 為漫畫產出的根目錄（絕對路徑）
- 新增章節時，所有圖片、音頻寫入 `books/my-comic-life/episodes/{episodeId}/`
- `monthly-volumes/` 為月冊引用圖，獨立於章節資料夾
- 禁止將任何內容寫入 `localRoot` 根目錄（已有 `.gitkeep` 或 `LICENSE` 保護）

---

## 三、章節流程（idempotency 機制）

### 3.1 新增章節時

1. 從用戶輸入中提取 `idempotencyKey`（格式：`YYYY-MM-DD-daily` 或自由格式）
2. 檢查 `series-manifest.json` 的 `idempotency` map：
   - 若已存在 → 覆寫該 `episodeId` 下的所有檔案
   - 若不存在 → 分配新 `episodeId`，寫入 `idempotency` map
3. 章節號（`episodeNumber`）：從 `series-manifest.json` 的 `nextEpisodeNumber` 遞增取得
4. 完成後更新 `series-manifest.json` 的 `nextEpisodeNumber` + `updatedAt`

### 3.2 寫入時的必要欄位（episode.json）

| 欄位 | 必填 | 說明 |
|---|---|---|
| `idempotencyKey` | ✓ | 與 series-manifest.json 對應 |
| `episodeId` | ✓ | 目錄名稱（唯一） |
| `episodeNumber` | ✓ | 遞增章節號 |
| `title` | ✓ | 章節標題 |
| `text` | ✓ | 完整敘述文 |
| `route` | ✓ | S / P / K / M / L |
| `recordedAt` | ✓ | ISO 8601（帶時區） |
| `pageAssets[]` | ✓ | 含 assetId / src / role / 完整 schema |
| `style` | ✓ | 固定 `02-snow-pastel` validated_preset |
| `palette` | ✓ | 來自 style 的 6 色 |
| `visualStatus` | ✓ | `ready` |
| `committedAt` | ✓ | ISO 8601（帶時區） |

---

## 四、路由規則（input-routing）

> 詳見 `../skills/ache-life-to-comic-skill/references/input-routing.md`

| 路由 | 適用場景 |
|---|---|
| `S` | 日常、想法、心得、情緒、輕事件 |
| `P` | 照片是主要證據 |
| `K` | 知識學習、概念解釋、讀書事實 |
| `M` | 會議、訪談、討論與行動記錄 |
| `L` | 原文已足夠，只需排版與輕插圖 |

**S-route 多事件鐵律：** 用戶列舉多件事 → 主動提取 `beats` + `requestedBodyPages`；否則只生 1 圖。

---

## 五、月冊更新流程

每新增一章時，必須同時更新：

1. `books/monthly-volumes/{YYYY-MM}/volume-manifest.json`
2. `books/monthly-volumes/{YYYY-MM}/continuous-edition/book-data.json`
3. `books/monthly-volumes/{YYYY-MM}/continuous-edition/index.html`
4. `books/monthly-volumes/{YYYY-MM}/index.html`
5. `books/parts/{YYYY-QN}/index.json`（如涉及季度彙整）
6. `books/annuals/{YYYY}/index.json`（如涉及年度彙整）

---

## 六、Git 協作約定

- **推送頻率：** 每次章節完成後立即 commit + push
- **Commit 訊息格式：** `feat: add episode-YYYY-MM-DD — {標題}`
- **分支策略：** 單一 `main` 分支（個人專案）
- **嚴禁：** 將 `.env`、憑證、敏感資料寫入 repo

---

## 七、視覺風格（02-snow-pastel）

```json
{
  "ink": "#202B37",
  "soft": "#667487",
  "primary": "#78A8DC",
  "pale": "#DBE9F7",
  "accent": "#C96355",
  "paper": "#FFFFFF"
}
```

- 軟調粉蠟筆質感
- 白色紙張背景
- 主角色為 `accent`（赤陶色）
- 字體：系統中文字體（無需嵌入）

---

## 八、目前章節現況

| 章節 ID | 標題 | 日期 | 路由 |
|---|---|---|---|
| `5d843add-c3ad-40d0-9908-ee4be98dff84` | 未知標題 | 2026-09-01 | - |
| `ep-qa-meeting-20260901` | QA 離職 meeting | 2026-09-01 | M |
| `episode-2026-09-02` | 肅穆與交付 | 2026-09-02 | S |

`nextEpisodeNumber`：4
