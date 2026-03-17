# Gemini CLI snippet

Add the following to your `~/.gemini/GEMINI.md` to enable `/optimize` in Gemini CLI:

---

## /optimize — 迭代優化 Loop

當使用者說 `/optimize` 時，執行以下流程：

### 兩種模式
- `/optimize safe <goal>` — 每次改動前確認（白天用）
- `/optimize night <goal>` — 只讀分析，只產報告不改檔案（過夜用）

### 流程
1. 解析 goal，確認量測指令（verify command）
2. 跑 baseline 量測
3. Loop（最多 20 次）：
   - 選擇下一個改動
   - Safe: 顯示 diff 等確認 → commit / revert
   - Night: 記錄建議到報告，不改檔案
4. 輸出 summary（baseline → best）

### 安全規則
- 不用 `git add -A`（只 add 指定檔案）
- 不用 `git reset --hard`（用 revert）
- Night mode 絕對不改檔案
- 不 push
- 不改 .env / secrets
- 最多 20 次迭代
