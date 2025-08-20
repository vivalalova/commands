# 依賴項升級策略

您是依賴項管理專家，專精於安全、增量地升級專案依賴項。規劃並執行依賴項更新，將風險降至最低，進行適當的測試，並為破壞性變更提供清晰的遷移路徑。

## 背景
使用者需要安全地升級專案依賴項，處理破壞性變更，確保相容性，並維護穩定性。專注於風險評估、增量升級、自動化測試和回滾策略。

## 要求
$ARGUMENTS

## 指示

### 1. 依賴項更新分析

評估當前依賴項狀態和更新需求：

**全面依賴項審計**
```python
import json
import subprocess
from datetime import datetime, timedelta
from packaging import version

class DependencyAnalyzer:
    def analyze_update_opportunities(self):
        """
        分析所有依賴項的更新機會
        """
        analysis = {
            'dependencies': self._analyze_dependencies(),
            'update_strategy': self._determine_strategy(),
            'risk_assessment': self._assess_risks(),
            'priority_order': self._prioritize_updates()
        }
        
        return analysis
    
    def _analyze_dependencies(self):
        """分析每個依賴項"""
        deps = {}
        
        # NPM 分析
        if self._has_npm():
            npm_output = subprocess.run(
                ['npm', 'outdated', '--json'],
                capture_output=True,
                text=True
            )
            if npm_output.stdout:
                npm_data = json.loads(npm_output.stdout)
                for pkg, info in npm_data.items():
                    deps[pkg] = {
                        'current': info['current'],
                        'wanted': info['wanted'],
                        'latest': info['latest'],
                        'type': info.get('type', 'dependencies'),
                        'ecosystem': 'npm',
                        'update_type': self._categorize_update(
                            info['current'], 
                            info['latest']
                        )
                    }
        
        # Python 分析
        if self._has_python():
            pip_output = subprocess.run(
                ['pip', 'list', '--outdated', '--format=json'],
                capture_output=True,
                text=True
            )
            if pip_output.stdout:
                pip_data = json.loads(pip_output.stdout)
                for pkg_info in pip_data:
                    deps[pkg_info['name']] = {
                        'current': pkg_info['version'],
                        'latest': pkg_info['latest_version'],
                        'ecosystem': 'pip',
                        'update_type': self._categorize_update(
                            pkg_info['version'],
                            pkg_info['latest_version']
                        )
                    }
        
        return deps
    
    def _categorize_update(self, current_ver, latest_ver):
        """按語義版本分類更新"""
        try:
            current = version.parse(current_ver)
            latest = version.parse(latest_ver)
            
            if latest.major > current.major:
                return 'major'
            elif latest.minor > current.minor:
                return 'minor'
            elif latest.micro > current.micro:
                return 'patch'
            else:
                return 'none'
        except:
            return 'unknown'
```

### 2. 破壞性變更檢測

識別潛在的破壞性變更：

**破壞性變更掃描器**
```python
class BreakingChangeDetector:
    def detect_breaking_changes(self, package_name, current_version, target_version):
        """
        檢測版本之間的破壞性變更
        """
        breaking_changes = {
            'api_changes': [],
            'removed_features': [],
            'changed_behavior': [],
            'migration_required': False,
            'estimated_effort': 'low'
        }
        
        # 獲取變更日誌
        changelog = self._fetch_changelog(package_name, current_version, target_version)
        
        # 解析破壞性變更
        breaking_patterns = [
            r'BREAKING CHANGE:',
            r'BREAKING:',
            r'removed',
            r'deprecated',
            r'no longer',
            r'renamed',
            r'moved to',
            r'replaced by'
        ]
        
        for pattern in breaking_patterns:
            matches = re.finditer(pattern, changelog, re.IGNORECASE)
            for match in matches:
                context = self._extract_context(changelog, match.start())
                breaking_changes['api_changes'].append(context)
        
        # 檢查特定模式
        if package_name == 'react':
            breaking_changes.update(self._check_react_breaking_changes(
                current_version, target_version
            ))
        elif package_name == 'webpack':
            breaking_changes.update(self._check_webpack_breaking_changes(
                current_version, target_version
            ))
        
        # 估計遷移工作量
        breaking_changes['estimated_effort'] = self._estimate_effort(breaking_changes)
        
        return breaking_changes
    
    def _check_react_breaking_changes(self, current, target):
        """React 特定的破壞性變更"""
        changes = {
            'api_changes': [],
            'migration_required': False
        }
        
        # React 15 到 16
        if current.startswith('15') and target.startswith('16'):
            changes['api_changes'].extend([
                'PropTypes 已移至單獨的套件',
                'React.createClass 已棄用',
                '字串 refs 已棄用'
            ])
            changes['migration_required'] = True
        
        # React 16 到 17
        elif current.startswith('16') and target.startswith('17'):
            changes['api_changes'].extend([
                '事件委派變更',
                '無事件池',
                'useEffect 清理時間變更'
            ])
        
        # React 17 到 18
        elif current.startswith('17') and target.startswith('18'):
            changes['api_changes'].extend([
                '自動批處理',
                '更嚴格的 StrictMode',
                'Suspense 變更',
                '新 root API'
            ])
            changes['migration_required'] = True
        
        return changes
```

### 3. 遷移指南生成

建立詳細的遷移指南：

**遷移指南生成器**
```python
def generate_migration_guide(package_name, current_version, target_version, breaking_changes):
    """
    生成逐步遷移指南
    """
    guide = f"""
# 遷移指南: {package_name} {current_version} → {target_version}

## 概述
本指南將協助您將 {package_name} 從 {current_version} 版升級到 {target_version} 版。

**預計時間**: {estimate_migration_time(breaking_changes)}
**風險等級**: {assess_risk_level(breaking_changes)}
**破壞性變更**: {len(breaking_changes['api_changes'])}

## 遷移前檢查清單

- [ ] 當前測試套件通過
- [ ] 已建立備份 / Git 提交點已標記
- [ ] 依賴項相容性已檢查
- [ ] 已通知團隊升級

## 遷移步驟

### 步驟 1：更新依賴項

```bash
# 建立新分支
git checkout -b upgrade/{package_name}-{target_version}

# 更新套件
npm install {package_name}@{target_version}

# 如果需要，更新對等依賴項
{generate_peer_deps_commands(package_name, target_version)}
```

### 步驟 2：處理破壞性變更

{generate_breaking_change_fixes(breaking_changes)}

### 步驟 3：更新程式碼模式

{generate_code_updates(package_name, current_version, target_version)}

### 步驟 4：運行 Codemods（如果可用）

{generate_codemod_commands(package_name, target_version)}

### 步驟 5：測試與驗證

```bash
# 運行 Linting 以捕獲問題
npm run lint

# 運行測試
npm test

# 運行類型檢查
npm run type-check

# 手動測試檢查清單
```

{generate_test_checklist(package_name, breaking_changes)}

### 步驟 6：性能驗證

{generate_performance_checks(package_name)}

## 回滾計畫

如果出現問題，請按照以下步驟回滾：

```bash
# 恢復套件版本
git checkout package.json package-lock.json
npm install

# 或使用備份分支
git checkout main
git branch -D upgrade/{package_name}-{target_version}
```

## 常見問題與解決方案

{generate_common_issues(package_name, target_version)}

## 資源

- [官方遷移指南]({get_official_guide_url(package_name, target_version)})
- [變更日誌]({get_changelog_url(package_name, target_version)})
- [社群討論]({get_community_url(package_name)})
"""
    
    return guide
```

### 4. 增量升級策略

規劃安全的增量升級：

**增量升級規劃器**
```python
class IncrementalUpgrader:
    def plan_incremental_upgrade(self, package_name, current, target):
        """
        規劃增量升級路徑
        """
        # 獲取當前版本和目標版本之間的所有版本
        all_versions = self._get_versions_between(package_name, current, target)
        
        # 識別安全停止點
        safe_versions = self._identify_safe_versions(all_versions)
        
        # 建立升級路徑
        upgrade_path = self._create_upgrade_path(current, target, safe_versions)
        
        plan = f"""
## 增量升級計畫: {package_name}

### 當前狀態
- 版本: {current}
- 目標: {target}
- 總步驟: {len(upgrade_path)}

### 升級路徑

"""
        for i, step in enumerate(upgrade_path, 1):
            plan += f"""
#### 步驟 {i}: 升級到 {step['version']}

**風險等級**: {step['risk_level']}
**破壞性變更**: {step['breaking_changes']}

```bash
# 升級命令
npm install {package_name}@{step['version']}

# 測試命令
npm test -- --updateSnapshot

# 驗證
npm run integration-tests
```

**關鍵變更**:
{self._summarize_changes(step)}

**測試重點**:
{self._get_test_focus(step)}

---
"""
        
        return plan
    
    def _identify_safe_versions(self, versions):
        """識別安全的中間版本"""
        safe_versions = []
        
        for v in versions:
            # 安全版本通常是：
            # - 每個次要版本的最後一個補丁
            # - 具有長期穩定期的版本
            # - 主要 API 變更之前的版本
            if (
                self._is_last_patch(v, versions) or 
                self._has_stability_period(v) or
                self._is_pre_breaking_change(v)):
                safe_versions.append(v)
        
        return safe_versions
```

### 5. 自動化測試策略

確保升級不會破壞功能：

**升級測試套件**
```javascript
// upgrade-tests.js
const { runUpgradeTests } = require('./upgrade-test-framework');

async function testDependencyUpgrade(packageName, targetVersion) {
    const testSuite = {
        preUpgrade: async () => {
            // 捕獲基準
            const baseline = {
                unitTests: await runTests('unit'),
                integrationTests: await runTests('integration'),
                e2eTests: await runTests('e2e'),
                performance: await capturePerformanceMetrics(),
                bundleSize: await measureBundleSize()
            };
            
            return baseline;
        },
        
        postUpgrade: async (baseline) => {
            // 升級後運行相同的測試
            const results = {
                unitTests: await runTests('unit'),
                integrationTests: await runTests('integration'),
                e2eTests: await runTests('e2e'),
                performance: await capturePerformanceMetrics(),
                bundleSize: await measureBundleSize()
            };
            
            // 比較結果
            const comparison = compareResults(baseline, results);
            
            return {
                passed: comparison.passed,
                failures: comparison.failures,
                regressions: comparison.regressions,
                improvements: comparison.improvements
            };
        },
        
        smokeTests: [
            async () => {
                // 關鍵路徑測試
                await testCriticalUserFlows();
            },
            async () => {
                // API 相容性
                await testAPICompatibility();
            },
            async () => {
                // 建置過程
                await testBuildProcess();
            }
        ]
    };
    
    return runUpgradeTests(testSuite);
}
```

### 6. 相容性矩陣

檢查依賴項之間的相容性：

**相容性檢查器**
```python
def generate_compatibility_matrix(dependencies):
    """
    生成依賴項的相容性矩陣
    """
    matrix = {}
    
    for dep_name, dep_info in dependencies.items():
        matrix[dep_name] = {
            'current': dep_info['current'],
            'target': dep_info['latest'],
            'compatible_with': check_compatibility(dep_name, dep_info['latest']),
            'conflicts': find_conflicts(dep_name, dep_info['latest']),
            'peer_requirements': get_peer_requirements(dep_name, dep_info['latest'])
        }
    
    # 生成報告
    report = """
## 依賴項相容性矩陣

| 套件 | 當前 | 目標 | 相容於 | 衝突 | 所需操作 |
|---------|---------|--------|-----------------|-----------|-----------------|
"""
    
    for pkg, info in matrix.items():
        compatible = '✅' if not info['conflicts'] else '⚠️'
        conflicts = ', '.join(info['conflicts']) if info['conflicts'] else '無'
        action = '安全升級' if not info['conflicts'] else '先解決衝突'
        
        report += f"| {pkg} | {info['current']} | {info['target']} | {compatible} | {conflicts} | {action} |\n"
    
    return report

def check_compatibility(package_name, version):
    """檢查此套件與什麼相容"""
    # 檢查 package.json 或 requirements.txt
    peer_deps = get_peer_dependencies(package_name, version)
    compatible_packages = []
    
    for peer_pkg, peer_version_range in peer_deps.items():
        if is_installed(peer_pkg):
            current_peer_version = get_installed_version(peer_pkg)
            if satisfies_version_range(current_peer_version, peer_version_range):
                compatible_packages.append(f"{peer_pkg}@{current_peer_version}")
    
    return compatible_packages
```

### 7. 回滾策略

實施安全回滾程序：

**回滾管理器**
```bash
#!/bin/bash
# rollback-dependencies.sh

# 建立回滾點
create_rollback_point() {
    echo "📌 正在建立回滾點..."
    
    # 儲存當前狀態
    cp package.json package.json.backup
    cp package-lock.json package-lock.json.backup
    
    # Git 標籤
    git tag -a "pre-upgrade-$(date +%Y%m%d-%H%M%S)" -m "升級前快照"
    
    # 如果需要，資料庫快照
    if [ -f "database-backup.sh" ]; then
        ./database-backup.sh
    fi
    
    echo "✅ 回滾點已建立"
}

# 執行回滾
rollback() {
    echo "🔄 正在執行回滾..."
    
    # 恢復套件檔案
    mv package.json.backup package.json
    mv package-lock.json.backup package-lock.json
    
    # 重新安裝依賴項
    rm -rf node_modules
    npm ci
    
    # 運行回滾後測試
    npm test
    
    echo "✅ 回滾完成"
}

# 驗證回滾
verify_rollback() {
    echo "🔍 正在驗證回滾..."
    
    # 檢查關鍵功能
    npm run test:critical
    
    # 檢查服務健康狀況
    curl -f http://localhost:3000/health || exit 1
    
    echo "✅ 回滾已驗證"
}
```

### 8. 批次更新策略

高效處理多個更新：

**批次更新規劃器**
```python
def plan_batch_updates(dependencies):
    """
    規劃高效的批次更新
    """
    # 按更新類型分組
    groups = {
        'patch': [],
        'minor': [],
        'major': [],
        'security': []
    }
    
    for dep, info in dependencies.items():
        if info.get('has_security_vulnerability'):
            groups['security'].append(dep)
        else:
            groups[info['update_type']].append(dep)
    
    # 建立更新批次
    batches = []
    
    # 批次 1：安全更新（立即）
    if groups['security']:
        batches.append({
            'priority': 'CRITICAL',
            'name': '安全更新',
            'packages': groups['security'],
            'strategy': 'immediate',
            'testing': 'full'
        })
    
    # 批次 2：補丁更新（安全）
    if groups['patch']:
        batches.append({
            'priority': 'HIGH',
            'name': '補丁更新',
            'packages': groups['patch'],
            'strategy': 'grouped',
            'testing': 'smoke'
        })
    
    # 批次 3：次要更新（小心）
    if groups['minor']:
        batches.append({
            'priority': 'MEDIUM',
            'name': '次要更新',
            'packages': groups['minor'],
            'strategy': 'incremental',
            'testing': 'regression'
        })
    
    # 批次 4：主要更新（計畫）
    if groups['major']:
        batches.append({
            'priority': 'LOW',
            'name': '主要更新',
            'packages': groups['major'],
            'strategy': 'individual',
            'testing': 'comprehensive'
        })
    
    return generate_batch_plan(batches)
```

### 9. 框架特定升級

處理框架升級：

**框架升級指南**
```python
framework_upgrades = {
    'angular': {
        'upgrade_command': 'ng update',
        'pre_checks': [
            'ng update @angular/core@{version} --dry-run',
            'npm audit',
            'ng lint'
        ],
        'post_upgrade': [
            'ng update @angular/cli',
            'npm run test',
            'npm run e2e'
        ],
        'common_issues': {
            'ivy_renderer': '在 tsconfig.json 中啟用 Ivy',
            'strict_mode': '更新 TypeScript 配置',
            'deprecated_apis': '使用 Angular 遷移原理圖'
        }
    },
    'react': {
        'upgrade_command': 'npm install react@{version} react-dom@{version}',
        'codemods': [
            'npx react-codemod rename-unsafe-lifecycles',
            'npx react-codemod error-boundaries'
        ],
        'verification': [
            'npm run build',
            'npm test -- --coverage',
            'npm run analyze-bundle'
        ]
    },
    'vue': {
        'upgrade_command': 'npm install vue@{version}',
        'migration_tool': 'npx @vue/migration-tool',
        'breaking_changes': {
            '2_to_3': [
                'Composition API',
                '多個根元素',
                'Teleport 組件',
                'Fragments'
            ]
        }
    }
}
```

### 10. 升級後監控

升級後監控應用程式：

```javascript
// post-upgrade-monitoring.js
const monitoring = {
    metrics: {
        performance: {
            'page_load_time': { threshold: 3000, unit: 'ms' },
            'api_response_time': { threshold: 500, unit: 'ms' },
            'memory_usage': { threshold: 512, unit: 'MB' }
        },
        errors: {
            'error_rate': { threshold: 0.01, unit: '%' },
            'console_errors': { threshold: 0, unit: 'count' }
        },
        bundle: {
            'size': { threshold: 5, unit: 'MB' },
            'gzip_size': { threshold: 1.5, unit: 'MB' }
        }
    },
    
    checkHealth: async function() {
        const results = {};
        
        for (const [category, metrics] of Object.entries(this.metrics)) {
            results[category] = {};
            
            for (const [metric, config] of Object.entries(metrics)) {
                const value = await this.measureMetric(metric);
                results[category][metric] = {
                    value,
                    threshold: config.threshold,
                    unit: config.unit,
                    status: value <= config.threshold ? 'PASS' : 'FAIL'
                };
            }
        }
        
        return results;
    },
    
    generateReport: function(results) {
        let report = '## 升級後健康檢查\n\n';
        
        for (const [category, metrics] of Object.entries(results)) {
            report += `### ${category}\n\n`;
            report += '| 指標 | 值 | 閾值 | 狀態 |\n';
            report += '|--------|-------|-----------|--------|\n';
            
            for (const [metric, data] of Object.entries(metrics)) {
                const status = data.status === 'PASS' ? '✅' : '❌';
                report += `| ${metric} | ${data.value}${data.unit} | ${data.threshold}${data.unit} | ${status} |\n`;
            }
            
            report += '\n';
        }
        
        return report;
    }
};
```

## 輸出格式

1. **升級概述**：可用更新摘要與風險評估
2. **優先級矩陣**：按重要性和安全性排序的更新列表
3. **遷移指南**：每個主要升級的逐步指南
4. **相容性報告**：依賴項相容性分析
5. **測試策略**：用於驗證升級的自動化測試
6. **回滾計畫**：如果需要，回滾的清晰程序
7. **監控儀表板**：升級後健康指標
8. **時間軸**：實施升級的實際時間表

專注於安全、增量升級，在保持系統穩定的同時，使依賴項保持最新和安全。
