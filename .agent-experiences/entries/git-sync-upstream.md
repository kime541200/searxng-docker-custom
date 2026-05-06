---
id: git-sync-upstream
title: Git 雙遠端同步官方 upstream 流程
created: 2026-05-06
updated: 2026-05-06
tags: [git, upstream, workflow, deployment]
keywords: [dual remote, origin, upstream, merge upstream/master, secret key, env]
related_files: [experiences/git-sync-upstream.md, .gitignore, .env.example]
related_entries: []
---

## 症狀

這個 fork 需要同時保留個人化部署設定，並定期接收官方 SearXNG Docker 倉庫的更新。如果只使用單一 remote，容易混淆「自己的版本」與「官方來源」，同步時也較難明確知道要從哪裡抓取更新。

## 根因

專案採用 dual remote 工作流：

- `origin` 指向個人 GitHub 倉庫，用來保存自定義版本。
- `upstream` 指向官方 SearXNG Docker 倉庫，用來取得官方更新。

官方更新需要先從 `upstream` fetch，再合併到本地 `master`，最後推送回 `origin`。

## 解法

同步官方更新時依序執行：

```bash
git fetch upstream
git merge upstream/master
git push origin master
```

若 `git merge upstream/master` 發生衝突，先手動解決衝突並提交，再推送到 `origin`。

## 試過但失敗的路

沒有在舊紀錄中留下已嘗試但失敗的替代做法。後續若遇到衝突處理、rebase、或 remote 設定錯誤等踩坑，應補充到這一節。

## 備註

- 永遠不要提交 `.env`。
- 敏感資訊，例如 `SEARXNG_SECRET_KEY`，應存在 `.env`，並在 `settings.yml` 中以 `${VAR}` 引用。
- 原始舊紀錄位於 `experiences/git-sync-upstream.md`。
