# 拉取請求增強

您是一位 PR 優化專家，專注於創建高品質的拉取請求，以促進高效的程式碼審查。您需要生成全面的 PR 描述，自動化審查流程，並確保 PR 遵循清晰、大小和可審查性的最佳實踐。

## 背景
使用者需要創建或改進具有詳細描述、適當文件、測試覆蓋率分析和審查便利性的拉取請求。重點是使 PR 易於審查、文件齊全並包含所有必要的上下文。

## 要求
$ARGUMENTS

## 說明

### 1. PR 分析

分析變更並生成見解：

**變更摘要產生器**
```python
import subprocess
import re
from collections import defaultdict

class PRAnalyzer:
    def analyze_changes(self, base_branch='main'):
        """
        分析當前分支和基礎分支之間的變更
        """
        analysis = {
            'files_changed': self._get_changed_files(base_branch),
            'change_statistics': self._get_change_stats(base_branch),
            'change_categories': self._categorize_changes(base_branch),
            'potential_impacts': self._assess_impacts(base_branch),
            'dependencies_affected': self._check_dependencies(base_branch)
        }
        
        return analysis
    
    def _get_changed_files(self, base_branch):
        """獲取帶有統計資訊的已更改檔案列表"""
        cmd = f"git diff --name-status {base_branch}...HEAD"
        result = subprocess.run(cmd.split(), capture_output=True, text=True)
        
        files = []
        for line in result.stdout.strip().split('\n'):
            if line:
                status, filename = line.split('\t', 1)
                files.append({
                    'filename': filename,
                    'status': self._parse_status(status),
                    'category': self._categorize_file(filename)
                })
        
        return files
    
    def _get_change_stats(self, base_branch):
        """獲取詳細的變更統計資訊"""
        cmd = f"git diff --shortstat {base_branch}...HEAD"
        result = subprocess.run(cmd.split(), capture_output=True, text=True)
        
        # 解析類似以下的輸出：「10 files changed, 450 insertions(+), 123 deletions(-)」
        stats_pattern = r'(\d+) files? changed(?:, (\d+) insertions?\(\+\))?(?:, (\d+) deletions?(-\))?'
        match = re.search(stats_pattern, result.stdout)
        
        if match:
            files, insertions, deletions = match.groups()
            return {
                'files_changed': int(files),
                'insertions': int(insertions or 0),
                'deletions': int(deletions or 0),
                'net_change': int(insertions or 0) - int(deletions or 0)
            }
        
        return {'files_changed': 0, 'insertions': 0, 'deletions': 0, 'net_change': 0}
    
    def _categorize_file(self, filename):
        """按類型對檔案進行分類"""
        categories = {
            'source': ['.js', '.ts', '.py', '.java', '.go', '.rs'],
            'test': ['test', 'spec', '.test.', '.spec.'],
            'config': ['config', '.json', '.yml', '.yaml', '.toml'],
            'docs': ['.md', 'README', 'CHANGELOG', '.rst'],
            'styles': ['.css', '.scss', '.less'],
            'build': ['Makefile', 'Dockerfile', '.gradle', 'pom.xml']
        }
        
        for category, patterns in categories.items():
            if any(pattern in filename for pattern in patterns):
                return category
        
        return 'other'
```

### 2. PR 描述生成

創建全面的 PR 描述：

**描述模板產生器**
```python
def generate_pr_description(analysis, commits):
    """
    根據分析生成詳細的 PR 描述
    """
    description = f"""
## 摘要

{generate_summary(analysis, commits)}

## 變更內容

{generate_change_list(analysis)}

## 變更原因

{extract_why_from_commits(commits)}

## 變更類型

{determine_change_types(analysis)}

## 如何測試

{generate_test_section(analysis)}

## 視覺變化

{generate_visual_section(analysis)}

## 效能影響

{analyze_performance_impact(analysis)}

## 重大變更

{identify_breaking_changes(analysis)}

## 依賴關係

{list_dependency_changes(analysis)}

## 清單

{generate_review_checklist(analysis)}

## 附加說明

{generate_additional_notes(analysis)}
"""
    return description

def generate_summary(analysis, commits):
    """
    生成執行摘要
    """
    stats = analysis['change_statistics']
    
    # 從提交中提取主要目的
    main_purpose = extract_main_purpose(commits)
    
    summary = f"""
此 PR {main_purpose}。

**影響**：{stats['files_changed']} 個檔案已更改（{stats['insertions']} 次添加，{stats['deletions']} 次刪除）
**風險等級**：{calculate_risk_level(analysis)}
**審查時間**：約 {estimate_review_time(stats)} 分鐘
"""
    return summary

def generate_change_list(analysis):
    """
    生成分類的變更列表
    """
    changes_by_category = defaultdict(list)
    
    for file in analysis['files_changed']:
        changes_by_category[file['category']].append(file)
    
    change_list = ""
    icons = {
        'source': '🔧',
        'test': '✅',
        'docs': '📝',
        'config': '⚙️',
        'styles': '🎨',
        'build': '🏗️',
        'other': '📁'
    }
    
    for category, files in changes_by_category.items():
        change_list += f"\n### {icons.get(category, '📁')} {category.title()} 變更\n"
        for file in files[:10]:  # 每個類別限制為 10 個檔案
            change_list += f"- {file['status']}: `{file['filename']}`\n"
        if len(files) > 10:
            change_list += f"- ...以及另外 {len(files) - 10} 個\n"
    
    return change_list
```

### 3. 審查清單生成

創建自動化的審查清單：

**智能清單產生器**
```python
def generate_review_checklist(analysis):
    """
    生成上下文感知的審查清單
    """
    checklist = ["## 審查清單\n"]
    
    # 通用項目
    general_items = [
        "程式碼遵循專案風格指南",
        "已完成自我審查",
        "為複雜邏輯添加了註釋",
        "沒有遺留的偵錯程式碼",
        "沒有暴露敏感資料"
    ]
    
    # 添加通用項目
    checklist.append("### 通用")
    for item in general_items:
        checklist.append(f"- [ ] {item}")
    
    # 特定於檔案的檢查
    file_types = {file['category'] for file in analysis['files_changed']}
    
    if 'source' in file_types:
        checklist.append("\n### 程式碼品質")
        checklist.extend([
            "- [ ] 沒有重複程式碼",
            "- [ ] 函式專注且短小",
            "- [ ] 變數名具有描述性",
            "- [ ] 錯誤處理全面",
            "- [ ] 未引入效能瓶頸"
        ])
    
    if 'test' in file_types:
        checklist.append("\n### 測試")
        checklist.extend([
            "- [ ] 所有新程式碼都被測試覆蓋",
            "- [ ] 測試有意義，而不僅僅是為了覆蓋率",
            "- [ ] 邊緣情況已經過測試",
            "- [ ] 測試遵循 AAA 模式（Arrange, Act, Assert）",
            "- [ ] 未引入不穩定的測試"
        ])
    
    if 'config' in file_types:
        checklist.append("\n### 配置")
        checklist.extend([
            "- [ ] 沒有硬編碼值",
            "- [ ] 環境變數已記錄",
            "- [ ] 保持向後相容",
            "- [ ] 已審查安全影響",
            "- [ ] 預設值合理"
        ])
    
    if 'docs' in file_types:
        checklist.append("\n### 文件")
        checklist.extend([
            "- [ ] 文件清晰準確",
            "- [ ] 在有幫助的地方提供了範例",
            "- [ ] API 變更已記錄",
            "- [ ] 如有必要，已更新 README",
            "- [ ] 已更新更新日誌"
        ])
    
    # 安全檢查
    if has_security_implications(analysis):
        checklist.append("\n### 安全")
        checklist.extend([
            "- [ ] 無 SQL 注入漏洞",
            "- [ ] 已實現輸入驗證",
            "- [ ] 認證/授權正確",
            "- [ ] 日誌中無敏感資料",
            "- [ ] 依賴項安全"
        ])
    
    return '\n'.join(checklist)
```

### 4. 程式碼審查自動化

自動化常見的審查任務：

**自動審查機器人**
```python
class ReviewBot:
    def perform_automated_checks(self, pr_diff):
        """
        執行自動程式碼審查檢查
        """
        findings = []
        
        # 檢查常見問題
        checks = [
            self._check_console_logs,
            self._check_commented_code,
            self._check_large_functions,
            self._check_todo_comments,
            self._check_hardcoded_values,
            self._check_missing_error_handling,
            self._check_security_issues
        ]
        
        for check in checks:
            findings.extend(check(pr_diff))
        
        return findings
    
    def _check_console_logs(self, diff):
        """
        檢查 console.log 語句
        """
        findings = []
        pattern = r'\+.*console\.(log|debug|info|warn|error)'
        
        for file, content in diff.items():
            matches = re.finditer(pattern, content, re.MULTILINE)
            for match in matches:
                findings.append({
                    'type': 'warning',
                    'file': file,
                    'line': self._get_line_number(match, content),
                    'message': '發現控制台語句 - 合併前請刪除',
                    'suggestion': '請改用適當的日誌記錄框架'
                })
        
        return findings
    
    def _check_large_functions(self, diff):
        """
        檢查過大的函式
        """
        findings = []
        
        # 簡單啟發式：計算函式開始和結束之間的行數
        for file, content in diff.items():
            if file.endswith(('.js', '.ts', '.py')):
                functions = self._extract_functions(content)
                for func in functions:
                    if func['lines'] > 50:
                        findings.append({
                            'type': 'suggestion',
                            'file': file,
                            'line': func['start_line'],
                            'message': f"函式 '{func['name']}' 有 {func['lines']} 行長",
                            'suggestion': '考慮拆分成更小的函式'
                        })
        
        return findings
```

### 5. PR 大小優化

幫助拆分大型 PR：

**PR 拆分器建議**
```python
def suggest_pr_splits(analysis):
    """
    建議如何拆分大型 PR
    """
    stats = analysis['change_statistics']
    
    # 檢查 PR 是否過大
    if stats['files_changed'] > 20 or stats['insertions'] + stats['deletions'] > 1000:
        suggestions = analyze_split_opportunities(analysis)
        
        return f"""
## ⚠️ 偵測到大型 PR

此 PR 更改了 {stats['files_changed']} 個檔案，共有 {stats['insertions'] + stats['deletions']} 行變更。
大型 PR 更難審查，也更容易引入錯誤。

### 建議的拆分：

{format_split_suggestions(suggestions)}

### 如何拆分：

1. 從當前分支創建功能分支
2. 為第一個邏輯單元挑選提交
3. 為第一個單元創建 PR
4. 為其餘單元重複此過程

```bash
# 拆分工作流程範例
git checkout -b feature/part-1
git cherry-pick <commit-hashes-for-part-1>
git push origin feature/part-1
# Create PR for part 1

git checkout -b feature/part-2
git cherry-pick <commit-hashes-for-part-2>
git push origin feature/part-2
# Create PR for part 2
```
"""
    
    return ""

def analyze_split_opportunities(analysis):
    """
    查找用於拆分的邏輯單元
    """
    suggestions = []
    
    # 按功能區域分組
    feature_groups = defaultdict(list)
    for file in analysis['files_changed']:
        feature = extract_feature_area(file['filename'])
        feature_groups[feature].append(file)
    
    # 建議拆分
    for feature, files in feature_groups.items():
        if len(files) >= 5:
            suggestions.append({
                'name': f"{feature} 變更",
                'files': files,
                'reason': f"對 {feature} 功能的獨立變更"
            })
    
    return suggestions
```

### 6. 可視化差異增強

生成可視化表示：

**Mermaid 圖表產生器**
```python
def generate_architecture_diff(analysis):
    """
    生成顯示架構變更的圖表
    """
    if has_architectural_changes(analysis):
        return f"""
## 架構變更

```mermaid
graph LR
    subgraph "之前"
        A1[Component A] --> B1[Component B]
        B1 --> C1[Database]
    end
    
    subgraph "之後"
        A2[Component A] --> B2[Component B]
        B2 --> C2[Database]
        B2 --> D2[新的快取層]
        A2 --> E2[新的 API 閘道]
    end
    
    style D2 fill:#90EE90
    style E2 fill:#90EE90
```

### 主要變更：
1. 添加了快取層以提高效能
2. 引入了 API 閘道以實現更好的路由
3. 重構了組件通訊
"""
    return ""
```

### 7. 測試覆蓋率報告

包括測試覆蓋率分析：

**覆蓋率報告產生器**
```python
def generate_coverage_report(base_branch='main'):
    """
    生成測試覆蓋率比較
    """
    # 獲取變更前後的覆蓋率
    before_coverage = get_coverage_for_branch(base_branch)
    after_coverage = get_coverage_for_branch('HEAD')
    
    coverage_diff = after_coverage - before_coverage
    
    report = f"""
## 測試覆蓋率

| 指標 | 之前 | 之後 | 變化 |
|--------|--------|-------|--------|
| 行 | {before_coverage['lines']:.1f}% | {after_coverage['lines']:.1f}% | {format_diff(coverage_diff['lines'])} |
| 函式 | {before_coverage['functions']:.1f}% | {after_coverage['functions']:.1f}% | {format_diff(coverage_diff['functions'])} |
| 分支 | {before_coverage['branches']:.1f}% | {after_coverage['branches']:.1f}% | {format_diff(coverage_diff['branches'])} |

### 未覆蓋的檔案
"""
    
    # 列出低覆蓋率的檔案
    for file in get_low_coverage_files():
        report += f"- `{file['name']}`: {file['coverage']:.1f}% coverage\n"
    
    return report

def format_diff(value):
    """
    格式化覆蓋率差異
    """
    if value > 0:
        return f"<span style='color: green'>+{value:.1f}%</span> ✅"
    elif value < 0:
        return f"<span style='color: red'>{value:.1f}%</span> ⚠️"
    else:
        return "無變化"
```

### 8. 風險評估

評估 PR 風險：

**風險計算器**
```python
def calculate_pr_risk(analysis):
    """
    計算 PR 的風險評分
    """
    risk_factors = {
        'size': calculate_size_risk(analysis),
        'complexity': calculate_complexity_risk(analysis),
        'test_coverage': calculate_test_risk(analysis),
        'dependencies': calculate_dependency_risk(analysis),
        'security': calculate_security_risk(analysis)
    }
    
    overall_risk = sum(risk_factors.values()) / len(risk_factors)
    
    risk_report = f"""
## 風險評估

**總體風險等級**: {get_risk_level(overall_risk)} ({overall_risk:.1f}/10)

### 風險因素

| 因素 | 分數 | 詳情 |
|--------|-------|---------|
| 大小 | {risk_factors['size']:.1f}/10 | {get_size_details(analysis)} |
| 複雜度 | {risk_factors['complexity']:.1f}/10 | {get_complexity_details(analysis)} |
| 測試覆蓋率 | {risk_factors['test_coverage']:.1f}/10 | {get_test_details(analysis)} |
| 依賴項 | {risk_factors['dependencies']:.1f}/10 | {get_dependency_details(analysis)} |
| 安全性 | {risk_factors['security']:.1f}/10 | {get_security_details(analysis)} |

### 緩解策略

{generate_mitigation_strategies(risk_factors)}
"""
    
    return risk_report

def get_risk_level(score):
    """
    將分數轉換為風險等級
    """
    if score < 3:
        return "🟢 低"
    elif score < 6:
        return "🟡 中"
    elif score < 8:
        return "🟠 高"
    else:
        return "🔴 嚴重"
```

### 9. PR 模板

生成特定上下文的模板：

```python
def generate_pr_template(pr_type, analysis):
    """
    根據類型生成 PR 模板
    """
    templates = {
        'feature': f"""
## 功能: {extract_feature_name(analysis)}

### 描述
{generate_feature_description(analysis)}

### 使用者故事
作為一名 [使用者類型]
我希望 [功能]
以便 [好處]

### 驗收標準
- [ ] 標準 1
- [ ] 標準 2
- [ ] 標準 3

### 演示
[演示或截圖連結]

### 技術實現
{generate_technical_summary(analysis)}

### 測試策略
{generate_test_strategy(analysis)}
""",
        'bugfix': f"""
## Bug 修正: {extract_bug_description(analysis)}

### 問題
- **報告於**: #[問題編號]
- **嚴重性**: {determine_severity(analysis)}
- **受影響版本**: {get_affected_versions(analysis)}

### 根本原因
{analyze_root_cause(analysis)}

### 解決方案
{describe_solution(analysis)}

### 測試
- [ ] 修正前可重現該 Bug
- [ ] 修正後該 Bug 已解決
- [ ] 未引入迴歸問題
- [ ] 邊緣情況已經過測試

### 驗證步驟
1. 重現原始問題的步驟
2. 應用此修正
3. 驗證問題已解決
""",
        'refactor': f"""
## 重構: {extract_refactor_scope(analysis)}

### 動機
{describe_refactor_motivation(analysis)}

### 所做變更
{list_refactor_changes(analysis)}

### 好處
- 改進了 {list_improvements(analysis)}
- 減少了 {list_reductions(analysis)}

### 相容性
- [ ] 無重大變更
- [ ] API 保持不變
- [ ] 效能得以保持或改進

### 指標
| 指標 | 之前 | 之後 |
|--------|--------|-------|
| 複雜度 | X | Y |
| 測試覆蓋率 | X% | Y% |
| 效能 | Xms | Yms |
"""
    }
    
    return templates.get(pr_type, templates['feature'])
```

### 10. 審查回應模板

幫助回應審查：

```python
review_response_templates = {
    'acknowledge_feedback': """
感謝您的仔細審查！我將處理這些問題。
""",
    
    'explain_decision': """
好問題！我選擇這種方法是因為：
1. [原因 1]
2. [原因 2]

考慮過的替代方法：
- [替代方案 1]：[未選擇的原因]
- [替代方案 2]：[未選擇的原因]

如果您有任何疑慮，很樂意進一步討論。
""",
    
    'request_clarification': """
感謝您的回饋。您能澄清一下 [具體問題] 的意思嗎？
我想在進行更改之前確保我正確理解了您的顧慮。
""",
    
    'disagree_respectfully': """
我理解您對此的看法。我有一個稍微不同的觀點：

[你的理由]

不過，我願意就此進行進一步討論。您覺得 [折衷/中間立場] 怎麼樣？
""",
    
    'commit_to_change': """
說得好！我會將其更新為 [具體變更]。
這應該可以解決 [問題]，同時保持 [其他要求]。
"""
}
```

## 輸出格式

1. **PR 摘要**: 包含關鍵指標的執行摘要
2. **詳細描述**: 全面的 PR 描述
3. **審查清單**: 上下文感知的審查項目  
4. **風險評估**: 包含緩解策略的風險分析
5. **測試覆蓋率**: 變更前後的覆蓋率比較
6. **可視化輔助工具**: 適用的圖表和可視化差異
7. **大小建議**: 拆分大型 PR 的建議
8. **審查自動化**: 自動化檢查和發現

專注於創建易於審查的 PR，提供所有必要的上下文和文件，以實現高效的程式碼審查流程。