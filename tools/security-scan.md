# 安全掃描與漏洞評估

您是安全專家，專精於應用程式安全、漏洞評估和安全編碼實踐。執行全面的安全審計，以識別漏洞、提供補救指導並實施安全最佳實踐。

## 背景
使用者需要徹底的安全分析，以識別漏洞、評估風險並實施保護措施。專注於 OWASP Top 10、依賴項漏洞和安全配置錯誤，並提供可行的補救步驟。

## 要求
$ARGUMENTS

## 指示

### 1. 安全掃描工具選擇

根據您的技術堆疊和要求選擇適當的安全掃描工具：

**工具選擇矩陣**
```python
security_tools = {
    'python': {
        'sast': {
            'bandit': {
                'strengths': ['專為 Python 設計', '快速', '良好的預設值', '基於 AST'],
                'best_for': ['Python 程式碼庫', 'CI/CD 管道', '快速掃描'],
                'command': 'bandit -r . -f json -o bandit-report.json',
                'config_file': '.bandit'
            },
            'semgrep': {
                'strengths': ['多語言', '自定義規則', '低誤報'],
                'best_for': ['複雜專案', '自定義安全模式', '企業'],
                'command': 'semgrep --config=auto --json --output=semgrep-report.json',
                'config_file': '.semgrep.yml'
            }
        },
        'dependency_scan': {
            'safety': {
                'command': 'safety check --json --output safety-report.json',
                'database': 'PyUp.io 漏洞資料庫',
                'best_for': 'Python 套件漏洞'
            },
            'pip_audit': {
                'command': 'pip-audit --format=json --output=pip-audit-report.json',
                'database': 'OSV 資料庫',
                'best_for': '全面的 Python 漏洞掃描'
            }
        }
    },
    
    'javascript': {
        'sast': {
            'eslint_security': {
                'command': 'eslint . --ext .js,.jsx,.ts,.tsx --format json > eslint-security.json',
                'plugins': ['@eslint/plugin-security', 'eslint-plugin-no-secrets'],
                'best_for': 'JavaScript/TypeScript 安全 Linting'
            },
            'sonarjs': {
                'command': 'sonar-scanner -Dsonar.projectKey=myproject',
                'best_for': '全面的程式碼品質和安全',
                'features': ['漏洞檢測', '程式碼異味', '技術債務']
            }
        },
        'dependency_scan': {
            'npm_audit': {
                'command': 'npm audit --json > npm-audit-report.json',
                'fix': 'npm audit fix',
                'best_for': 'NPM 套件漏洞'
            },
            'yarn_audit': {
                'command': 'yarn audit --json > yarn-audit-report.json',
                'best_for': 'Yarn 套件漏洞'
            },
            'snyk': {
                'command': 'snyk test --json > snyk-report.json',
                'fix': 'snyk wizard',
                'best_for': '全面的漏洞管理'
            }
        }
    },
    
    'container': {
        'trivy': {
            'image_scan': 'trivy image --format json --output trivy-image.json myimage:latest',
            'fs_scan': 'trivy fs --format json --output trivy-fs.json .',
            'repo_scan': 'trivy repo --format json --output trivy-repo.json .',
            'strengths': ['快速', '準確', '多目標', 'SBOM 生成'],
            'best_for': '容器和檔案系統漏洞掃描'
        },
        'grype': {
            'command': 'grype dir:. -o json > grype-report.json',
            'strengths': ['快速', '準確的漏洞檢測'],
            'best_for': '容器映像和檔案系統掃描'
        },
        'clair': {
            'api_based': True,
            'strengths': ['基於 API', '持續監控'],
            'best_for': '註冊表整合，自動化掃描'
        }
    },
    
    'infrastructure': {
        'checkov': {
            'command': 'checkov -d . --framework terraform --output json > checkov-report.json',
            'supports': ['Terraform', 'CloudFormation', 'Kubernetes', 'Helm', 'Serverless'],
            'best_for': '基礎設施即程式碼安全'
        },
        'tfsec': {
            'command': 'tfsec . --format json > tfsec-report.json',
            'supports': ['Terraform'],
            'best_for': 'Terraform 特定的安全掃描'
        },
        'kube_score': {
            'command': 'kube-score score *.yaml --output-format json > kube-score.json',
            'supports': ['Kubernetes'],
            'best_for': 'Kubernetes 清單安全和最佳實踐'
        }
    },
    
    'secrets': {
        'truffleHog': {
            'command': 'trufflehog git file://. --json > trufflehog-report.json',
            'strengths': ['Git 歷史掃描', '高準確性', '自定義正則表達式'],
            'best_for': 'Git 儲存庫中的秘密檢測'
        },
        'gitleaks': {
            'command': 'gitleaks detect --report-format json --report-path gitleaks-report.json',
            'strengths': ['快速', '可配置', '預提交鉤子'],
            'best_for': '即時秘密檢測'
        },
        'detect_secrets': {
            'command': 'detect-secrets scan --all-files . > .secrets.baseline',
            'strengths': ['基準管理', '減少誤報'],
            'best_for': '企業秘密管理'
        }
    }
}
```

**多工具安全掃描器**
```python
import json
import subprocess
import os
from pathlib import Path
from typing import Dict, List, Any
from dataclasses import dataclass
from datetime import datetime

@dataclass
class VulnerabilityFinding:
    tool: str
    severity: str
    category: str
    title: str
    description: str
    file_path: str
    line_number: int
    cve: str
    cwe: str
    remediation: str
    confidence: str

class SecurityScanner:
    def __init__(self, project_path: str):
        self.project_path = Path(project_path)
        self.findings = []
        self.scan_results = {}
        
    def detect_project_type(self) -> List[str]:
        """檢測專案技術以選擇適當的掃描器"""
        technologies = []
        
        # Python
        if (self.project_path / 'requirements.txt').exists() or \
           (self.project_path / 'setup.py').exists() or \
           (self.project_path / 'pyproject.toml').exists():
            technologies.append('python')
            
        # JavaScript/Node.js
        if (self.project_path / 'package.json').exists():
            technologies.append('javascript')
            
        # Go
        if (self.project_path / 'go.mod').exists():
            technologies.append('golang')
            
        # Docker
        if (self.project_path / 'Dockerfile').exists():
            technologies.append('container')
            
        # Terraform
        if list(self.project_path.glob('*.tf')):
            technologies.append('terraform')
            
        # Kubernetes
        if list(self.project_path.glob('*.yaml')) or list(self.project_path.glob('*.yml')):
            technologies.append('kubernetes')
            
        return technologies
    
    def run_comprehensive_scan(self) -> Dict[str, Any]:
        """運行所有適用的安全掃描器"""
        technologies = self.detect_project_type()
        
        scan_plan = {
            'timestamp': datetime.now().isoformat(),
            'technologies': technologies,
            'scanners_used': [],
            'findings': []
        }
        
        # 始終運行秘密檢測
        self.run_secret_scan()
        scan_plan['scanners_used'].append('secret_detection')
        
        # 特定技術掃描
        if 'python' in technologies:
            self.run_python_scans()
            scan_plan['scanners_used'].extend(['bandit', 'safety', 'pip_audit'])
            
        if 'javascript' in technologies:
            self.run_javascript_scans()
            scan_plan['scanners_used'].extend(['eslint_security', 'npm_audit'])
            
        if 'container' in technologies:
            self.run_container_scans()
            scan_plan['scanners_used'].append('trivy')
            
        if 'terraform' in technologies:
            self.run_terraform_scans()
            scan_plan['scanners_used'].extend(['checkov', 'tfsec'])
            
        # 生成統一報告
        scan_plan['findings'] = self.findings
        scan_plan['summary'] = self.generate_summary()
        
        return scan_plan
    
    def run_secret_scan(self):
        """運行秘密檢測工具"""
        try:
            # TruffleHog
            result = subprocess.run([
                'trufflehog', 'filesystem', str(self.project_path),
                '--json', '--no-update'
            ], capture_output=True, text=True, timeout=300)
            
            if result.stdout:
                for line in result.stdout.strip().split('\n'):
                    if line:
                        finding = json.loads(line)
                        self.findings.append(VulnerabilityFinding(
                            tool='trufflehog',
                            severity='CRITICAL',
                            category='secrets',
                            title=f"檢測到秘密: {finding.get('DetectorName', '未知')}",
                            description=finding.get('Raw', ''),
                            file_path=finding.get('SourceMetadata', {}).get('Data', {}).get('Filesystem', {}).get('file', ''),
                            line_number=finding.get('SourceMetadata', {}).get('Data', {}).get('Filesystem', {}).get('line', 0),
                            cve='',
                            cwe='CWE-798',
                            remediation='移除秘密並輪換憑證',
                            confidence=str(finding.get('Verified', False))
                        ))
        except (subprocess.TimeoutExpired, subprocess.CalledProcessError, FileNotFoundError):
            print("TruffleHog 不可用或掃描失敗")
            
        try:
            # GitLeaks
            result = subprocess.run([
                'gitleaks', 'detect', '--source', str(self.project_path),
                '--report-format', 'json', '--no-git'
            ], capture_output=True, text=True, timeout=300)
            
            if result.stdout:
                findings = json.loads(result.stdout)
                for finding in findings:
                    self.findings.append(VulnerabilityFinding(
                        tool='gitleaks',
                        severity='HIGH',
                        category='secrets',
                        title=f"秘密模式: {finding.get('RuleID', '未知')}",
                        description=finding.get('Description', ''),
                        file_path=finding.get('File', ''),
                        line_number=finding.get('StartLine', 0),
                        cve='',
                        cwe='CWE-798',
                        remediation='移除秘密並添加到 .gitignore',
                        confidence='high'
                    ))
        except (subprocess.TimeoutExpired, subprocess.CalledProcessError, FileNotFoundError):
            print("GitLeaks 不可用或掃描失敗")
    
    def run_python_scans(self):
        """運行 Python 特定的安全掃描器"""
        # Bandit
        try:
            result = subprocess.run([
                'bandit', '-r', str(self.project_path),
                '-f', 'json', '--severity-level', 'medium'
            ], capture_output=True, text=True, timeout=300)
            
            if result.stdout:
                bandit_results = json.loads(result.stdout)
                for result_item in bandit_results.get('results', []):
                    self.findings.append(VulnerabilityFinding(
                        tool='bandit',
                        severity=result_item.get('issue_severity', 'MEDIUM'),
                        category='sast',
                        title=result_item.get('test_name', ''),
                        description=result_item.get('issue_text', ''),
                        file_path=result_item.get('filename', ''),
                        line_number=result_item.get('line_number', 0),
                        cve='',
                        cwe=result_item.get('test_id', ''),
                        remediation=result_item.get('more_info', ''),
                        confidence=result_item.get('issue_confidence', 'MEDIUM')
                    ))
        except (subprocess.TimeoutExpired, subprocess.CalledProcessError, FileNotFoundError):
            print("Bandit 不可用或掃描失敗")
        
        # Safety
        try:
            result = subprocess.run([
                'safety', 'check', '--json'
            ], capture_output=True, text=True, timeout=300, cwd=self.project_path)
            
            if result.stdout:
                safety_results = json.loads(safety_result.stdout)
                for vuln in safety_results:
                    self.findings.append(VulnerabilityFinding(
                        tool='safety',
                        severity='HIGH',
                        category='dependencies',
                        title=f"易受攻擊的套件: {vuln.get('package_name', '')}",
                        description=vuln.get('advisory', ''),
                        file_path='requirements.txt',
                        line_number=0,
                        cve=vuln.get('cve', ''),
                        cwe='',
                        remediation=f"更新到版本 {vuln.get('analyzed_version', 'latest')}",
                        confidence='high'
                    ))
        except (subprocess.TimeoutExpired, subprocess.CalledProcessError, FileNotFoundError):
            print("Safety 不可用或掃描失敗")
    
    def generate_summary(self) -> Dict[str, Any]:
        """生成摘要統計"""
        severity_counts = {'CRITICAL': 0, 'HIGH': 0, 'MEDIUM': 0, 'LOW': 0}
        category_counts = {}
        
        for finding in self.findings:
            severity_counts[finding.severity] = severity_counts.get(finding.severity, 0) + 1
            category_counts[finding.category] = category_counts.get(finding.category, 0) + 1
        
        return {
            'total_findings': len(self.findings),
            'severity_breakdown': severity_counts,
            'category_breakdown': category_counts,
            'risk_score': self.calculate_risk_score(severity_counts)
        }
    
    def calculate_risk_score(self, severity_counts: Dict[str, int]) -> int:
        """計算總體風險分數 (0-100)"""
        weights = {'CRITICAL': 10, 'HIGH': 7, 'MEDIUM': 4, 'LOW': 1}
        total_score = sum(weights[severity] * count for severity, count in severity_counts.items())
        max_possible = 100  # 任意上限
        return min(100, int((total_score / max_possible) * 100))
```

**SAST (靜態應用程式安全測試)**
```python
# 增強的程式碼漏洞模式與工具特定實施
security_rules = {
    "sql_injection": {
        "patterns": [
            r"query\s*\(\s*[\"'].*\+.*[\"']\s*\)",
            r"execute\s*\(\s*[\"'].*%[s|d].*[\"']\s*%",
            r"f[\"'].*SELECT.*{.*}.*FROM"
        ],
        "severity": "CRITICAL",
        "cwe": "CWE-89",
        "fix": "使用參數化查詢或預準備語句"
    }

    "xss": {
        "patterns": [
            r"innerHTML\s*=\s*[^\"']*\+",
            r"document\.write\s*\([^\"']*\+",
            r"dangerouslySetInnerHTML",
            r"v-html\s*=\s*[\"'][^\"']*\{" 
        ],
        "severity": "HIGH",
        "cwe": "CWE-79",
        "fix": "清理使用者輸入並使用安全渲染方法"
    },
    
    "hardcoded_secrets": {
        "patterns": [
            r"(?i)(api[_-]?key|apikey|secret|password)\s*[:=]\s*[\"'][^\"']{8,}[\"']",
            r"(?i)bearer\s+[a-zA-Z0-9\-\._~\+\/]{20,}",
            r"(?i)(aws[_-]?access[_-]?key[_-]?id|aws[_-]?secret)\s*[:=]",
            r"private[_-]?key\s*[:=]\s*[\"'][^\"']+[\"']"
        ],
        "severity": "CRITICAL",
        "cwe": "CWE-798",
        "fix": "使用環境變數或安全金鑰管理服務"
    },
    
    "path_traversal": {
        "patterns": [
            r"\.\.\/",
            r"readFile\s*\([^\"']*\+",
            r"include\s*\([^\"']*\$",
            r"require\s*\([^\"']*\+"
        ],
        "severity": "HIGH",
        "cwe": "CWE-22",
        "fix": "驗證和清理檔案路徑"
    },
    
    "insecure_random": {
        "patterns": [
            r"Math\.random\(\) ",
            r"rand\(\) ",
            r"mt_rand\(\) "
        ],
        "severity": "MEDIUM",
        "cwe": "CWE-330",
        "fix": "使用加密安全隨機函數"
    }
}

def scan_code_vulnerabilities(file_path, content):
    """
    使用框架特定模式增強程式碼漏洞掃描
    """
    vulnerabilities = []
    
    for vuln_type, rule in security_rules.items():
        for pattern in rule['patterns']:
            matches = re.finditer(pattern, content, re.MULTILINE)
            for match in matches:
                line_num = content[:match.start()].count('\n') + 1
                vulnerabilities.append({
                    'type': vuln_type,
                    'severity': rule['severity'],
                    'file': file_path,
                    'line': line_num,
                    'code': match.group(0),
                    'cwe': rule['cwe'],
                    'fix': rule['fix'],
                    'confidence': rule.get('confidence', 'medium'),
                    'owasp_category': rule.get('owasp', 'A03:2021-Injection')
                })
    
    return vulnerabilities

# 框架特定安全模式
framework_security_patterns = {
    'django': {
        'csrf_exempt': {
            'pattern': r'@csrf_exempt',
            'severity': 'HIGH',
            'description': 'CSRF 保護已禁用',
            'fix': '移除 @csrf_exempt 裝飾器並實施適當的 CSRF 保護'
        },
        'raw_sql': {
            'pattern': r'\.raw\([