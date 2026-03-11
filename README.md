# AIDC 開發規範手冊

> AI 驅動工作流程 — 從需求到交付的完整開發指引

---

## 1. 專案架構圖（三個 Repo 的分工）

```mermaid
graph TB
    subgraph ORG["🏢 Rexshen-Co GitHub Organization"]
        direction TB

        subgraph DOCS["📚 docs（系統規格與設計中心）"]
            D1["/requirements — 使用者故事"]
            D2["/diagrams — Mermaid 圖表<br/>Use Case / Flowchart / Class"]
            D3["/database — DDL .sql 檔案"]
        end

        subgraph PROTO["🎨 prototype（功能雛形與 UI 驗證）"]
            P1["V0 匯出的 React / Tailwind 程式碼"]
            P2["UI 原型供利害關係人確認"]
        end

        subgraph BACKEND["⚙️ backend（正式開發主專案）"]
            B1["工程師正式開發的程式碼"]
            B2["Code Review / CI/CD"]
            B3["最終上線產品"]
        end

        DOCS -- "規格參考" --> PROTO
        DOCS -- "唯一真理來源 (Single Source of Truth)" --> BACKEND
        PROTO -- "視覺參考" --> BACKEND
    end

    PM["👤 PM / SA"] -- "建立 Issues & 看板" --> ORG
    DEV["👩‍💻 工程師"] -- "開發 & PR" --> BACKEND
```

| Repo | 定位 | 主要內容 | 管理重點 |
|------|------|----------|----------|
| **docs** | 專案大腦 | 規格文件、Mermaid 圖表、SQL Schema | 所有 Issue 都應連結至此 |
| **prototype** | 專案臉面 | V0 匯出的 React 雛形 | 確認 UI 後供開發者視覺參考 |
| **backend** | 專案身體 | 正式上線程式碼 | 執行 Project 看板最頻繁之處 |

---

## 2. 開發全流程圖（SA → 設計 → 雛形 → 分工）

```mermaid
flowchart TD
    START([🚀 需求啟動]) --> SA

    subgraph SA["階段一：需求分析與邏輯建模（SA 核心）"]
        SA1["與 Claude 討論需求"] --> SA2
        SA2["產出 Mermaid 圖表<br/>Use Case / Flowchart / Class Diagram"] --> SA3
        SA3["存成 .md 檔案 → docs/diagrams/"]
    end

    SA --> DB

    subgraph DB["階段二：資料建模與技術規格（技術深耕）"]
        DB1["DrawSQL 設計 ER 圖"] --> DB2
        DB2["Export SQL (DDL)"] --> DB3
        DB3["存入 docs/database/schema.sql"]
        DB4["Claude 將 SQL 轉為 Mermaid ERD 貼於 GitHub Issue"]
    end

    DB --> UI

    subgraph UI["階段三：功能雛形與 UI 實作（視覺驗證）"]
        UI1["將需求貼給 V0 by Vercel"] --> UI2
        UI2["V0 產出 React + Tailwind 程式碼"] --> UI3
        UI3["Claude Code 自動上傳 → prototype repo"]
        UI4["利害關係人確認畫面"]
        UI3 --> UI4
    end

    UI --> PM

    subgraph PM["階段四：團隊分工與任務控管（PM 管理）"]
        PM1["建立 GitHub Teams 分配權限"] --> PM2
        PM2["建立 Project 看板"] --> PM3
        PM3["拆解 Issues 附上 Mermaid 圖"] --> PM4
        PM4["分配 Issue 給對應 Team 成員"]
    end

    PM --> DEV([👩‍💻 工程師開始執行])
```

---

## 3. 工具與文件對照表

| 流程環節 | 產出物 | 存放位置（GitHub） | 可自動化？ | 外部對應工具 |
|----------|--------|-------------------|-----------|-------------|
| 需求展開 | Use Case / Flowchart / Class Diagram | `docs/diagrams/*.md`（Mermaid） | ✅ 生成代碼 | Claude 網頁版 |
| 資料設計 | SQL 檔案（DDL）、ER 圖 | `docs/database/schema.sql` | ✅ 修改與 Commit | DrawSQL 編輯器 |
| 介面雛形 | React / Tailwind 程式碼 | `prototype/src/` | ✅ 一鍵 Push | V0 編輯器 / Figma |
| 任務分工 | GitHub Issues / Project Board | Project Board + Issues | ⚠️ 部分（可用 CLI 開 Issue） | 無 |
| 正式開發 | 上線程式碼 | `backend/` | ✅ PR / CI/CD | IDE（VS Code 等）|

```mermaid
graph LR
    CLAUDE_D["🤖 Claude 網頁版<br/>（思考與生成）"]
    CLAUDE_C["⌨️ Claude Code CLI<br/>（執行與自動化）"]
    V0["⚡ V0 by Vercel<br/>（UI 雛形生成）"]
    DRAWSQL["🗄️ DrawSQL<br/>（DB 設計）"]
    FIGMA["🎨 Figma<br/>（UI/UX 精細設計）"]
    GITHUB["🐙 GitHub<br/>（版本控管中心）"]

    CLAUDE_D -- "Mermaid 語法、規格討論" --> GITHUB
    CLAUDE_C -- "git add/commit/push 自動化" --> GITHUB
    V0 -- "複製 Code → Claude Code 上傳" --> GITHUB
    DRAWSQL -- "Export SQL → 存入 Repo" --> GITHUB
    FIGMA -- "貼 Figma 連結於 Issue" --> GITHUB
```

---

## 4. 無法直接進入 Git 的外部程序清單

以下工具屬於「外部連結」或「編輯環境」，內容變動無法被 Git 追蹤，需手動串接：

| 工具 | 對應流程 | 無法進 Git 的原因 | 正確串接方式 |
|------|----------|-----------------|-------------|
| **Figma** | UI/UX 精細設計 | 雲端即時協作，Git 無法管轄內容變動 | 在 GitHub Issue 中貼上 Figma 連結 |
| **V0 互動介面** | 雛形展示 | 線上編輯環境不可匯出為 Git 物件 | 複製程式碼，由 Claude Code 上傳至 `prototype` Repo |
| **DrawSQL 畫布** | ERD 視覺化設計 | 拖拉操作無法存成 Git 可追蹤的格式 | Export SQL → 將 `.sql` 存入 `docs/database/` |
| **Claude 網頁版** | 需求討論、圖表生成 | 對話內容不進版控 | 將生成的 Mermaid 語法複製存為 `.md` 文件 |

```mermaid
flowchart LR
    subgraph EXTERNAL["⚠️ 無法直接進入 GitHub 的外部程序"]
        FIGMA["🎨 Figma\n→ 在 Issue 貼連結"]
        V0["⚡ V0\n→ 複製 Code 後上傳"]
        DRAWSQL["🗄️ DrawSQL\n→ Export SQL 後存檔"]
        CLAUDE_WEB["🤖 Claude 網頁版\n→ 複製 Mermaid 存 .md"]
    end

    FIGMA -->|"手動串接"| GH["🐙 GitHub"]
    V0 -->|"Claude Code 上傳"| GH
    DRAWSQL -->|"Export + Commit"| GH
    CLAUDE_WEB -->|"複製存檔"| GH
```

---

## 快速操作指令

```bash
# SA 啟動 — 將 Mermaid 圖存入 docs
git add diagrams/use-case.md && git commit -m "feat: add use case diagram"

# 雛形上傳 — V0 程式碼由 Claude Code 推送
git add . && git commit -m "feat: add V0 prototype" && git push origin main

# 資料庫 Schema 版控
git add database/schema.sql && git commit -m "feat: update db schema"
```

---

*本手冊由 Claude Code 根據 AI 驅動工作流程文件自動生成，並持續維護於 `docs` Repo。*
