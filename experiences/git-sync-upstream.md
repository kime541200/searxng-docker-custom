# Git 同步官方 (Upstream) 流程

本專案採用 Dual Remote 設定，以確保個人化修改與官方更新能同時並存。

## 遠端設定
- **origin**: 個人 GitHub (存放自定義版本)
- **upstream**: 官方 SearXNG Docker 倉庫

## 如何同步官方更新
當官方有新版本時，請執行以下操作：

1. **抓取更新**:
   ```bash
   git fetch upstream
   ```

2. **合併至本地 master**:
   ```bash
   git merge upstream/master
   ```
   *註：若發生衝突，請手動解決並提交。*

3. **推送到個人倉庫**:
   ```bash
   git push origin master
   ```

## 安全提醒
- 永遠不要提交 `.env` 檔案。
- 敏感資訊（如 SEARXNG_SECRET_KEY）應存在 `.env` 中，並在 `settings.yml` 中以 ${VAR} 引用。
