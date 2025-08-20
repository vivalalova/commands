# 依賴項審計與安全分析

您是依賴項安全專家，專精於漏洞掃描、許可證合規性和供應鏈安全。分析專案依賴項中的已知漏洞、許可證問題、過時套件，並提供可行的補救策略。

## 背景
使用者需要全面的依賴項分析，以識別其專案依賴項中的安全漏洞、許可證衝突和維護風險。專注於可操作的見解，並在可能的情況下提供自動化修復。

## 要求
$ARGUMENTS

## 指示

### 1. 依賴項發現

掃描並盤點所有專案依賴項：

**多語言檢測**
```python
import os
import json
import toml
import yaml
from pathlib import Path

class DependencyDiscovery:
    def __init__(self, project_path):
        self.project_path = Path(project_path)
        self.dependency_files = {
            'npm': ['package.json', 'package-lock.json', 'yarn.lock'],
            'python': ['requirements.txt', 'Pipfile', 'Pipfile.lock', 'pyproject.toml', 'poetry.lock'],
            'ruby': ['Gemfile', 'Gemfile.lock'],
            'java': ['pom.xml', 'build.gradle', 'build.gradle.kts'],
            'go': ['go.mod', 'go.sum'],
            'rust': ['Cargo.toml', 'Cargo.lock'],
            'php': ['composer.json', 'composer.lock'],
            'dotnet': ['*.csproj', 'packages.config', 'project.json']
        }
        
    def discover_all_dependencies(self):
        """
        發現不同套件管理器中的所有依賴項
        """
        dependencies = {}
        
        # NPM/Yarn 依賴項
        if (self.project_path / 'package.json').exists():
            dependencies['npm'] = self._parse_npm_dependencies()
            
        # Python 依賴項
        if (self.project_path / 'requirements.txt').exists():
            dependencies['python'] = self._parse_requirements_txt()
        elif (self.project_path / 'Pipfile').exists():
            dependencies['python'] = self._parse_pipfile()
        elif (self.project_path / 'pyproject.toml').exists():
            dependencies['python'] = self._parse_pyproject_toml()
            
        # Go 依賴項
        if (self.project_path / 'go.mod').exists():
            dependencies['go'] = self._parse_go_mod()
            
        return dependencies
    
    def _parse_npm_dependencies(self):
        """
        解析 NPM package.json 和鎖定檔案
        """
        with open(self.project_path / 'package.json', 'r') as f:
            package_json = json.load(f)
            
        deps = {}
        
        # 直接依賴項
        for dep_type in ['dependencies', 'devDependencies', 'peerDependencies']:
            if dep_type in package_json:
                for name, version in package_json[dep_type].items():
                    deps[name] = {
                        'version': version,
                        'type': dep_type,
                        'direct': True
                    }
        
        # 解析鎖定檔案以獲取確切版本
        if (self.project_path / 'package-lock.json').exists():
            with open(self.project_path / 'package-lock.json', 'r') as f:
                lock_data = json.load(f)
                self._parse_npm_lock(lock_data, deps)
                
        return deps
```

**依賴項樹分析**
```python
def build_dependency_tree(dependencies):
    """
    建置完整的依賴項樹，包括傳遞性依賴項
    """
    tree = {
        'root': {
            'name': 'project',
            'version': '1.0.0',
            'dependencies': {}
        }
    }
    
    def add_dependencies(node, deps, visited=None):
        if visited is None:
            visited = set()
            
        for dep_name, dep_info in deps.items():
            if dep_name in visited:
                # 檢測到循環依賴
                node['dependencies'][dep_name] = {
                    'circular': True,
                    'version': dep_info['version']
                }
                continue
                
            visited.add(dep_name)
            
            node['dependencies'][dep_name] = {
                'version': dep_info['version'],
                'type': dep_info.get('type', 'runtime'),
                'dependencies': {}
            }
            
            # 遞歸添加傳遞性依賴項
            if 'dependencies' in dep_info:
                add_dependencies(
                    node['dependencies'][dep_name],
                    dep_info['dependencies'],
                    visited.copy()
                )
    
    add_dependencies(tree['root'], dependencies)
    return tree
```

### 2. 漏洞掃描

根據漏洞資料庫檢查依賴項：

**CVE 資料庫檢查**
```python
import requests
from datetime import datetime

class VulnerabilityScanner:
    def __init__(self):
        self.vulnerability_apis = {
            'npm': 'https://registry.npmjs.org/-/npm/v1/security/advisories/bulk',
            'pypi': 'https://pypi.org/pypi/{package}/json',
            'rubygems': 'https://rubygems.org/api/v1/gems/{package}.json',
            'maven': 'https://ossindex.sonatype.org/api/v3/component-report'
        }
        
    def scan_vulnerabilities(self, dependencies):
        """
        掃描依賴項中的已知漏洞
        """
        vulnerabilities = []
        
        for package_name, package_info in dependencies.items():
            vulns = self._check_package_vulnerabilities(
                package_name,
                package_info['version'],
                package_info.get('ecosystem', 'npm')
            )
            
            if vulns:
                vulnerabilities.extend(vulns)
                
        return self._analyze_vulnerabilities(vulnerabilities)
    
    def _check_package_vulnerabilities(self, name, version, ecosystem):
        """
        檢查特定套件的漏洞
        """
        if ecosystem == 'npm':
            return self._check_npm_vulnerabilities(name, version)
        elif ecosystem == 'pypi':
            return self._check_python_vulnerabilities(name, version)
        elif ecosystem == 'maven':
            return self._check_java_vulnerabilities(name, version)
            
    def _check_npm_vulnerabilities(self, name, version):
        """
        檢查 NPM 套件漏洞
        """
        # 使用 npm audit API
        response = requests.post(
            'https://registry.npmjs.org/-/npm/v1/security/advisories/bulk',
            json={name: [version]}
        )
        
        vulnerabilities = []
        if response.status_code == 200:
            data = response.json()
            if name in data:
                for advisory in data[name]:
                    vulnerabilities.append({
                        'package': name,
                        'version': version,
                        'severity': advisory['severity'],
                        'title': advisory['title'],
                        'cve': advisory.get('cves', []),
                        'description': advisory['overview'],
                        'recommendation': advisory['recommendation'],
                        'patched_versions': advisory['patched_versions'],
                        'published': advisory['created']
                    })
                    
        return vulnerabilities
```

**嚴重性分析**
```python
def analyze_vulnerability_severity(vulnerabilities):
    """
    分析並按嚴重性優先處理漏洞
    """
    severity_scores = {
        'critical': 9.0,
        'high': 7.0,
        'moderate': 4.0,
        'low': 1.0
    }
    
    analysis = {
        'total': len(vulnerabilities),
        'by_severity': {
            'critical': [],
            'high': [],
            'moderate': [],
            'low': []
        },
        'risk_score': 0,
        'immediate_action_required': []
    }
    
    for vuln in vulnerabilities:
        severity = vuln['severity'].lower()
        analysis['by_severity'][severity].append(vuln)
        
        # 計算風險分數
        base_score = severity_scores.get(severity, 0)
        
        # 根據因素調整分數
        if vuln.get('exploit_available', False):
            base_score *= 1.5
        if vuln.get('publicly_disclosed', True):
            base_score *= 1.2
        if 'remote_code_execution' in vuln.get('description', '').lower():
            base_score *= 2.0
            
        vuln['risk_score'] = base_score
        analysis['risk_score'] += base_score
        
        # 標記立即行動項目
        if severity in ['critical', 'high'] or base_score > 8.0:
            analysis['immediate_action_required'].append({
                'package': vuln['package'],
                'severity': severity,
                'action': f"更新到 {vuln['patched_versions']}"
            })
    
    # 按風險分數排序
    for severity in analysis['by_severity']:
        analysis['by_severity'][severity].sort(
            key=lambda x: x.get('risk_score', 0),
            reverse=True
        )
    
    return analysis
```

### 3. 許可證合規性

分析依賴項許可證的相容性：

**許可證檢測**
```python
class LicenseAnalyzer:
    def __init__(self):
        self.license_compatibility = {
            'MIT': ['MIT', 'BSD', 'Apache-2.0', 'ISC'],
            'Apache-2.0': ['Apache-2.0', 'MIT', 'BSD'],
            'GPL-3.0': ['GPL-3.0', 'GPL-2.0'],
            'BSD-3-Clause': ['BSD-3-Clause', 'MIT', 'Apache-2.0'],
            'proprietary': []
        }
        
        self.license_restrictions = {
            'GPL-3.0': 'Copyleft - 需要原始碼公開',
            'AGPL-3.0': '強 Copyleft - 網路使用需要原始碼公開',
            'proprietary': '未經明確許可證不得使用',
            'unknown': '許可證不明確 - 需要法律審查'
        }
        
    def analyze_licenses(self, dependencies, project_license='MIT'):
        """
        分析許可證相容性
        """
        issues = []
        license_summary = {}
        
        for package_name, package_info in dependencies.items():
            license_type = package_info.get('license', 'unknown')
            
            # 追蹤許可證使用情況
            if license_type not in license_summary:
                license_summary[license_type] = []
            license_summary[license_type].append(package_name)
            
            # 檢查相容性
            if not self._is_compatible(project_license, license_type):
                issues.append({
                    'package': package_name,
                    'license': license_type,
                    'issue': f'與專案許可證 {project_license} 不相容',
                    'severity': 'high',
                    'recommendation': self._get_license_recommendation(
                        license_type,
                        project_license
                    )
                })
            
            # 檢查限制性許可證
            if license_type in self.license_restrictions:
                issues.append({
                    'package': package_name,
                    'license': license_type,
                    'issue': self.license_restrictions[license_type],
                    'severity': 'medium',
                    'recommendation': '審查使用情況並確保合規性'
                })
        
        return {
            'summary': license_summary,
            'issues': issues,
            'compliance_status': '失敗' if issues else '通過'
        }
```

**許可證報告**
```markdown
## 許可證合規報告

### 摘要
- **專案許可證**：MIT
- **總依賴項**：245
- **許可證問題**：3
- **合規狀態**：⚠️ 需要審查

### 許可證分佈
| 許可證 | 計數 | 套件 |
|---------|-------|----------|
| MIT | 180 | express, lodash, ... |
| Apache-2.0 | 45 | aws-sdk, ... |
| BSD-3-Clause | 15 | ... |
| GPL-3.0 | 3 | [問題] package1, package2, package3 |
| 未知 | 2 | [問題] mystery-lib, old-package |

### 合規問題

#### 高嚴重性
1. **GPL-3.0 依賴項**
   - 套件：package1, package2, package3
   - 問題：GPL-3.0 與 MIT 許可證不相容
   - 風險：可能需要開源您的整個專案
   - 建議：
     - 替換為 MIT/Apache 許可證的替代方案
     - 或將專案許可證更改為 GPL-3.0

#### 中等嚴重性
2. **未知許可證**
   - 套件：mystery-lib, old-package
   - 問題：無法確定許可證相容性
   - 風險：潛在的法律風險
   - 建議：
     - 聯繫套件維護者
     - 審查原始碼以獲取許可證資訊
     - 考慮替換為已知替代方案
```

### 4. 過時依賴項

識別並優先處理依賴項更新：

**版本分析**
```python
def analyze_outdated_dependencies(dependencies):
    """
    檢查過時的依賴項
    """
    outdated = []
    
    for package_name, package_info in dependencies.items():
        current_version = package_info['version']
        latest_version = fetch_latest_version(package_name, package_info['ecosystem'])
        
        if is_outdated(current_version, latest_version):
            # 計算過時程度
            version_diff = calculate_version_difference(current_version, latest_version)
            
            outdated.append({
                'package': package_name,
                'current': current_version,
                'latest': latest_version,
                'type': version_diff['type'],  # major, minor, patch
                'releases_behind': version_diff['count'],
                'age_days': get_version_age(package_name, current_version),
                'breaking_changes': version_diff['type'] == 'major',
                'update_effort': estimate_update_effort(version_diff),
                'changelog': fetch_changelog(package_name, current_version, latest_version)
            })
    
    return prioritize_updates(outdated)

def prioritize_updates(outdated_deps):
    """
    根據多個因素優先處理更新
    """
    for dep in outdated_deps:
        score = 0
        
        # 安全更新獲得最高優先級
        if dep.get('has_security_fix', False):
            score += 100
            
        # 主要版本更新
        if dep['type'] == 'major':
            score += 20
        elif dep['type'] == 'minor':
            score += 10
        else:
            score += 5
            
        # 年齡因素
        if dep['age_days'] > 365:
            score += 30
        elif dep['age_days'] > 180:
            score += 20
        elif dep['age_days'] > 90:
            score += 10
            
        # 落後發布數量
        score += min(dep['releases_behind'] * 2, 20)
        
        dep['priority_score'] = score
        dep['priority'] = 'critical' if score > 80 else 'high' if score > 50 else 'medium'
    
    return sorted(outdated_deps, key=lambda x: x['priority_score'], reverse=True)
```

### 5. 依賴項大小分析

分析捆綁包大小影響：

**捆綁包大小影響**
```javascript
// 分析 NPM 套件大小
const analyzeBundleSize = async (dependencies) => {
    const sizeAnalysis = {
        totalSize: 0,
        totalGzipped: 0,
        packages: [],
        recommendations: []
    };
    
    for (const [packageName, info] of Object.entries(dependencies)) {
        try {
            // 獲取套件統計資訊
            const response = await fetch(
                `https://bundlephobia.com/api/size?package=${packageName}@${info.version}`
            );
            const data = await response.json();
            
            const packageSize = {
                name: packageName,
                version: info.version,
                size: data.size,
                gzip: data.gzip,
                dependencyCount: data.dependencyCount,
                hasJSNext: data.hasJSNext,
                hasSideEffects: data.hasSideEffects
            };
            
            sizeAnalysis.packages.push(packageSize);
            sizeAnalysis.totalSize += data.size;
            sizeAnalysis.totalGzipped += data.gzip;
            
            // 大小建議
            if (data.size > 1000000) { // 1MB
                sizeAnalysis.recommendations.push({
                    package: packageName,
                    issue: '捆綁包大小過大',
                    size: `${(data.size / 1024 / 1024).toFixed(2)} MB`,
                    suggestion: '考慮更輕量的替代方案或延遲載入'
                });
            }
        } catch (error) {
            console.error(`分析 ${packageName} 失敗:`, error);
        }
    }
    
    // 按大小排序
    sizeAnalysis.packages.sort((a, b) => b.size - a.size);
    
    // 添加主要問題
    sizeAnalysis.topOffenders = sizeAnalysis.packages.slice(0, 10);
    
    return sizeAnalysis;
};
```

### 6. 供應鏈安全

檢查依賴項劫持和錯字劫持：

**供應鏈檢查**
```python
def check_supply_chain_security(dependencies):
    """
    執行供應鏈安全檢查
    """
    security_issues = []
    
    for package_name, package_info in dependencies.items():
        # 檢查錯字劫持
        typo_check = check_typosquatting(package_name)
        if typo_check['suspicious']:
            security_issues.append({
                'type': '錯字劫持',
                'package': package_name,
                'severity': 'high',
                'similar_to': typo_check['similar_packages'],
                'recommendation': '驗證套件名稱拼寫'
            })
        
        # 檢查維護者變更
        maintainer_check = check_maintainer_changes(package_name)
        if maintainer_check['recent_changes']:
            security_issues.append({
                'type': '維護者變更',
                'package': package_name,
                'severity': 'medium',
                'details': maintainer_check['changes'],
                'recommendation': '審查最近的套件變更'
            })
        
        # 檢查可疑模式
        if contains_suspicious_patterns(package_info):
            security_issues.append({
                'type': '可疑行為',
                'package': package_name,
                'severity': 'high',
                'patterns': package_info['suspicious_patterns'],
                'recommendation': '審計套件原始碼'
            })
    
    return security_issues

def check_typosquatting(package_name):
    """
    檢查套件名稱是否可能是錯字劫持
    """
    common_packages = [
        'react', 'express', 'lodash', 'axios', 'webpack',
        'babel', 'jest', 'typescript', 'eslint', 'prettier'
    ]
    
    for legit_package in common_packages:
        distance = levenshtein_distance(package_name.lower(), legit_package)
        if 0 < distance <= 2:  # 接近但不完全匹配
            return {
                'suspicious': True,
                'similar_packages': [legit_package],
                'distance': distance
            }
    
    return {'suspicious': False}
```

### 7. 自動化補救

生成自動化修復：

**更新腳本**
```bash
#!/bin/bash
# 自動更新帶有安全修復的依賴項

echo "🔒 安全更新腳本"
echo "========================