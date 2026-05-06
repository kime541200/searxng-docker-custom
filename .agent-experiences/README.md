# 專案經驗庫 (`.agent-experiences/`)

本目錄記錄此專案累積的「事件式經驗」：解過的 bug、踩過的坑、關鍵配置、依賴陷阱、需特定順序才成功的工作流。架構決策、編碼規範、需求文件不在這裡，那些屬於 `AGENTS.md` / `docs/adr/` / `docs/`。

每條經驗 = 一個 markdown 檔（位於 `entries/`）。`_index.json` 由 scripts 自動從 entries 反向產生，不要手動編輯它。

## 給 Agent 的工作流（兩個重點）

### 一、開始任務前先 search

從使用者的任務描述抽出關鍵字、會觸碰的檔案路徑，呼叫 search：

```bash
python3 <SKILL_PATH>/scripts/search.py \
  --query "使用者描述中的關鍵詞" \
  --files "path/a.py,path/b.py"
```

看 Top 5 的 `symptom_snippet`，相關的展開讀整檔。在回覆中明確告訴使用者「我參考了經驗條目 X、Y」並說明套用方式。**沒命中相關條目就直接退場，不要硬找東西套**。

### 二、任務完成後檢查「學習事件」

下列任一命中才考慮新增條目：

- Bug 試了 ≥2 次才解
- 找到非顯而易見的環境/依賴/配置陷阱
- 使用者明確表達「原來如此」「卡很久」「沒想到是這樣」
- 試過某條路明確失敗才走另一條

命中後**先用相同關鍵字 search 去重**：

- 有看起來高度重疊的條目 → 問使用者：「現有條目 X 看起來相關，要 update 它還是新建？」
- 沒重疊 → 走 add 流程

## scripts 用法

所有 scripts 路徑為 `<SKILL_PATH>/scripts/<name>.py`，`<SKILL_PATH>` 通常是 `~/.claude/skills/agent-experiences/`（或對應 IDE 的 skills 目錄）。

### search

```bash
# 自由文字
python3 search.py --query "sqlite locking nfs"

# 標籤過濾
python3 search.py --tags sqlite,database

# 檔案反查（強訊號：使用者改了某檔，反查相關經驗）
python3 search.py --files docker-compose.yml,infra/db/init.sh

# 混合
python3 search.py --query "lock error" --files docker-compose.yml --top 10
```

輸出 JSON：每筆 match 含 `id`、`score`、`title`、`symptom_snippet`、`matched_signals`。

### add（draft 模式）

```bash
# 1. 產生空模板到暫存檔
python3 add.py --scaffold > /tmp/exp-$(uuidgen).md

# 2. 用 Edit/Write 工具填入內容
#    必填欄位：title, tags, keywords; 必填段落：症狀/根因/解法/試過但失敗的路

# 3. 收進 entries/ 並 reindex
python3 add.py --from-draft /tmp/exp-xxx.md
```

slug 衝突時 add 會直接報錯，附上既有條目資訊提示「這可能是 update 對象」。

### update（draft 模式）

```bash
# 1. 把現有條目原文 dump 到暫存檔
python3 update.py --id sqlite-wal-mode --extract > /tmp/exp-$(uuidgen).md

# 2. 用 Edit 工具改要改的部分（id 和 created 不要改）

# 3. 寫回；updated 會自動更新為今天
python3 update.py --id sqlite-wal-mode --from-draft /tmp/exp-xxx.md
```

### delete

```bash
python3 delete.py --id sqlite-wal-mode
```

**呼叫 delete 之前必須先跟使用者確認**。

### reindex

```bash
# 平時不需要呼叫，所有 mutation 操作會自動 reindex
# 使用者手動編輯 entries/*.md 後，再叫一次重建索引
python3 reindex.py
```

## 經驗條目格式

```markdown
---
id: sqlite-wal-mode-on-nfs                 # 等同檔名（不含 .md）
title: SQLite WAL 模式在 NFS 掛載上會壞
created: 2026-05-06
updated: 2026-05-06
tags: [sqlite, nfs, database, deployment]
keywords: [WAL, locking, EBUSY, mount]
related_files: [infra/db/init.sh, docker-compose.yml]
related_entries: []
---

## 症狀
（出現了什麼錯誤訊息、什麼行為、在什麼情境下）

## 根因
（為什麼會發生，越具體越好）

## 解法
（具體做了什麼，含關鍵指令/設定）

## 試過但失敗的路
（避免下次再走，含為什麼失敗）

## 備註
（可選：版本、環境、相關連結）
```

`tags` 是受控分類（如 `sqlite`、`docker`），`keywords` 是自由文本（錯誤碼、版本號、症狀詞）。搜尋時兩者都查，但權重不同。

## 給人類開發者

- 直接瀏覽 `entries/` 目錄即可看到所有經驗
- `_index.json` 是 scripts 維護的，不要手改
- 想新增經驗：可以直接在 `entries/` 建檔（照上面格式），然後跑 `python reindex.py`；或讓 Agent 走 add 流程

## 此檔案的角色

`AGENTS.md`（或 `CLAUDE.md`）的指針指向這裡，所以**所有 coding agent**（Claude Code、Codex、Gemini CLI、Cursor）都應該先讀過這份再開始工作。Claude Code 上的 `agent-experiences` Skill 會自動觸發；其他 agent 就靠這份 README 與 `_index.json` 自行查詢。
