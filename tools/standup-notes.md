# 站立會議筆記生成器

透過審查 Obsidian 保險庫上下文和 Jira 票證來生成每日站立會議筆記。

## 用法
```
/standup-notes
```

## 流程

1. **從 Obsidian 收集上下文**
   - 使用 `mcp__mcp-obsidian__obsidian_get_recent_changes` 尋找最近修改的檔案
   - 使用 `mcp__mcp-obsidian__obsidian_get_recent_periodic_notes` 獲取最近的每日筆記
   - 尋找專案更新、已完成的任務和正在進行的工作

2. **檢查 Jira 票證**
   - 使用 `mcp__atlassian__searchJiraIssuesUsingJql` 尋找分配給當前使用者的票證
   - 過濾：
     - 進行中的票證（當前工作）
     - 最近解決/關閉的票證（昨天完成的成就）
     - 即將到來/待辦的票證（今天計畫的工作）

3. **生成站立會議筆記**
   格式：
   ```
   早安！
   昨天：
   
   • [來自 Jira 和 Obsidian 筆記的已完成任務]
   • [主要成就和里程碑]
   
   今天：
   
   • [進行中的 Jira 票證]
   • [來自票證和筆記的計畫工作]
   • [來自日曆/筆記的會議]
   
   備註：[任何阻礙、依賴或重要上下文]
   ```

4. **寫入 Obsidian**
   - 以 `Standup Notes/YYYY-MM-DD.md` 格式建立檔案
   - 使用 `mcp__mcp-obsidian__obsidian_append_content` 寫入生成的筆記

## 實施步驟

1. 從 Atlassian 獲取當前使用者資訊
2. 搜尋最近的 Obsidian 變更（最近 2 天）
3. 查詢 Jira：
   - `assignee = currentUser() AND (status CHANGED FROM "In Progress" TO "Done" DURING (-1d, now()) OR resolutiondate >= -1d)`
   - `assignee = currentUser() AND status = "In Progress"`
   - `assignee = currentUser() AND status in ("To Do", "Open") AND (sprint in openSprints() OR priority in (High, Highest))`
4. 解析並分類發現
5. 生成格式化的站立會議筆記
6. 儲存到 Obsidian 保險庫

## 上下文提取模式

- 尋找關鍵字：「completed」、「finished」、「deployed」、「released」、「fixed」、「implemented」
- 提取會議筆記和行動項目
- 識別提及的阻礙或依賴
- 提取衝刺目標和目的
