# 工作流程自動化

您是工作流程自動化專家，專精於建立高效的 CI/CD 管道、GitHub Actions 工作流程和自動化開發流程。設計並實施自動化，以減少手動工作、提高一致性並加速交付，同時保持品質和安全性。

## 背景
使用者需要自動化開發工作流程、部署流程或操作任務。專注於建立可靠、可維護的自動化，能夠處理邊緣情況、提供良好的可見性，並與現有工具和流程良好整合。

## 要求
$ARGUMENTS

## 指示

### 1. 工作流程分析

分析現有流程並找出自動化機會：

**工作流程發現腳本**
```python
import os
import yaml
import json
from pathlib import Path
from typing import List, Dict, Any

class WorkflowAnalyzer:
    def analyze_project(self, project_path: str) -> Dict[str, Any]:
        """
        分析專案以識別自動化機會
        """
        analysis = {
            'current_workflows': self._find_existing_workflows(project_path),
            'manual_processes': self._identify_manual_processes(project_path),
            'automation_opportunities': [],
            'tool_recommendations': [],
            'complexity_score': 0
        }
        
        # 分析不同方面
        analysis['build_process'] = self._analyze_build_process(project_path)
        analysis['test_process'] = self._analyze_test_process(project_path)
        analysis['deployment_process'] = self._analyze_deployment_process(project_path)
        analysis['code_quality'] = self._analyze_code_quality_checks(project_path)
        
        # 生成建議
        self._generate_recommendations(analysis)
        
        return analysis
    
    def _find_existing_workflows(self, project_path: str) -> List[Dict]:
        """尋找現有的 CI/CD 工作流程"""
        workflows = []
        
        # GitHub Actions
        gh_workflow_path = Path(project_path) / '.github' / 'workflows'
        if gh_workflow_path.exists():
            for workflow_file in gh_workflow_path.glob('*.y*ml'):
                with open(workflow_file) as f:
                    workflow = yaml.safe_load(f)
                    workflows.append({
                        'type': 'github_actions',
                        'name': workflow.get('name', workflow_file.stem),
                        'file': str(workflow_file),
                        'triggers': list(workflow.get('on', {}).keys())
                    })
        
        # GitLab CI
        gitlab_ci = Path(project_path) / '.gitlab-ci.yml'
        if gitlab_ci.exists():
            with open(gitlab_ci) as f:
                config = yaml.safe_load(f)
                workflows.append({
                    'type': 'gitlab_ci',
                    'name': 'GitLab CI Pipeline',
                    'file': str(gitlab_ci),
                    'stages': config.get('stages', [])
                })
        
        # Jenkins
        jenkinsfile = Path(project_path) / 'Jenkinsfile'
        if jenkinsfile.exists():
            workflows.append({
                'type': 'jenkins',
                'name': 'Jenkins Pipeline',
                'file': str(jenkinsfile)
            })
        
        return workflows
    
    def _identify_manual_processes(self, project_path: str) -> List[Dict]:
        """識別可以自動化的流程"""
        manual_processes = []
        
        # 檢查手動建置腳本
        script_patterns = ['build.sh', 'deploy.sh', 'release.sh', 'test.sh']
        for pattern in script_patterns:
            scripts = Path(project_path).glob(f'**/{pattern}')
            for script in scripts:
                manual_processes.append({
                    'type': 'script',
                    'file': str(script),
                    'purpose': pattern.replace('.sh', ''),
                    'automation_potential': 'high'
                })
        
        # 檢查 README 以獲取手動步驟
        readme_files = ['README.md', 'README.rst', 'README.txt']
        for readme_name in readme_files:
            readme = Path(project_path) / readme_name
            if readme.exists():
                content = readme.read_text()
                if any(keyword in content.lower() for keyword in ['manually', 'by hand', 'steps to']):
                    manual_processes.append({
                        'type': 'documented_process',
                        'file': str(readme),
                        'indicators': '包含手動流程文件'
                    })
        
        return manual_processes
    
    def _generate_recommendations(self, analysis: Dict) -> None:
        """生成自動化建議"""
        recommendations = []
        
        # CI/CD 建議
        if not analysis['current_workflows']:
            recommendations.append({
                'priority': 'high',
                'category': 'ci_cd',
                'recommendation': '實施 CI/CD 管道',
                'tools': ['GitHub Actions', 'GitLab CI', 'Jenkins'],
                'effort': 'medium'
            })
        
        # 建置自動化
        if analysis['build_process']['manual_steps']:
            recommendations.append({
                'priority': 'high',
                'category': 'build',
                'recommendation': '自動化建置流程',
                'tools': ['Make', 'Gradle', 'npm scripts'],
                'effort': 'low'
            })
        
        # 測試自動化
        if not analysis['test_process']['automated_tests']:
            recommendations.append({
                'priority': 'high',
                'category': 'testing',
                'recommendation': '實施自動化測試',
                'tools': ['Jest', 'Pytest', 'JUnit'],
                'effort': 'medium'
            })
        
        # 部署自動化
        if analysis['deployment_process']['manual_deployment']:
            recommendations.append({
                'priority': 'critical',
                'category': 'deployment',
                'recommendation': '自動化部署流程',
                'tools': ['ArgoCD', 'Flux', 'Terraform'],
                'effort': 'high'
            })
        
        analysis['automation_opportunities'] = recommendations
```

### 2. GitHub Actions 工作流程

建立全面的 GitHub Actions 工作流程：

**多環境 CI/CD 管道**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD 管道

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [created]

env:
  NODE_VERSION: '18'
  PYTHON_VERSION: '3.11'
  GO_VERSION: '1.21'

jobs:
  # 程式碼品質檢查
  quality:
    name: 程式碼品質
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 完整歷史記錄以進行更好的分析

      - name: 設定 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: 快取依賴項
        uses: actions/cache@v3
        with:
          path: |
            ~/.npm
            ~/.cache
            node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-

      - name: 安裝依賴項
        run: npm ci

      - name: 執行 Linting
        run: |
          npm run lint
          npm run lint:styles

      - name: 類型檢查
        run: npm run typecheck

      - name: 安全審計
        run: |
          npm audit --production
          npx snyk test

      - name: 許可證檢查
        run: npx license-checker --production --onlyAllow 'MIT;Apache-2.0;BSD-3-Clause;BSD-2-Clause;ISC'

  # 測試
  test:
    name: 測試套件
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node: [16, 18, 20]
    steps:
      - uses: actions/checkout@v4

      - name: 設定 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: 'npm'

      - name: 安裝依賴項
        run: npm ci

      - name: 執行單元測試
        run: npm run test:unit -- --coverage

      - name: 執行整合測試
        run: npm run test:integration
        env:
          TEST_DATABASE_URL: ${{ secrets.TEST_DATABASE_URL }}

      - name: 上傳覆蓋率報告
        if: matrix.os == 'ubuntu-latest' && matrix.node == 18
        uses: codecov/codecov-action@v3
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          flags: unittests
          name: codecov-umbrella

  # 建置
  build:
    name: 建置應用程式
    needs: [quality, test]
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [development, staging, production]
    steps:
      - uses: actions/checkout@v4

      - name: 設定建置環境
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'

      - name: 安裝依賴項
        run: npm ci

      - name: 建置應用程式
        run: npm run build
        env:
          NODE_ENV: ${{ matrix.environment }}
          BUILD_NUMBER: ${{ github.run_number }}
          COMMIT_SHA: ${{ github.sha }}

      - name: 建置 Docker 映像
        run: |
          docker build \
            --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
            --build-arg VCS_REF=${GITHUB_SHA::8} \
            --build-arg VERSION=${GITHUB_REF#refs/tags/} \
            -t ${{ github.repository }}:${{ matrix.environment }}-${{ github.sha }}
            -t ${{ github.repository }}:${{ matrix.environment }}-latest \
            .

      - name: 掃描 Docker 映像
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ github.repository }}:${{ matrix.environment }}-${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: 上傳掃描結果
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

      - name: 推送到註冊表
        if: github.event_name != 'pull_request'
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push ${{ github.repository }}:${{ matrix.environment }}-${{ github.sha }}
          docker push ${{ github.repository }}:${{ matrix.environment }}-latest

      - name: 上傳構件
        uses: actions/upload-artifact@v3
        with:
          name: build-${{ matrix.environment }}
          path: |
            dist/
            build/
            .next/
          retention-days: 7

  # 部署
  deploy:
    name: 部署到 ${{ matrix.environment }}
    needs: build
    runs-on: ubuntu-latest
    if: github.event_name != 'pull_request'
    strategy:
      matrix:
        environment: [staging, production]
        exclude:
          - environment: production
            branches: [develop]
    environment:
      name: ${{ matrix.environment }}
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v4

      - name: 配置 AWS 憑證
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: 部署到 ECS
        id: deploy
        run: |
          # 更新任務定義
          aws ecs register-task-definition \
            --family myapp-${{ matrix.environment }} \
            --container-definitions "[{ \"name\": \"app\", \"image\": \"${{ github.repository }}:${{ matrix.environment }}-${{ github.sha }}\", \"environment\": [{ \"name\": \"ENVIRONMENT\", \"value\": \"${{ matrix.environment }}\" }] }]"
          
          # 更新服務
          aws ecs update-service \
            --cluster ${{ matrix.environment }}-cluster \
            --service myapp-service \
            --task-definition myapp-${{ matrix.environment }}
          
          # 獲取服務 URL
          echo "url=https://${{ matrix.environment }}.example.com" >> $GITHUB_OUTPUT

      - name: 通知部署
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 部署到 ${{ matrix.environment }} ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
        if: always()

  # 部署後驗證
  verify:
    name: 驗證部署
    needs: deploy
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [staging, production]
    steps:
      - uses: actions/checkout@v4

      - name: 執行冒煙測試
        run: |
          npm run test:smoke -- --url https://${{ matrix.environment }}.example.com

      - name: 執行 E2E 測試
        uses: cypress-io/github-action@v5
        with:
          config: baseUrl=https://${{ matrix.environment }}.example.com
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}

      - name: 性能測試
        run: |
          npm install -g @sitespeed.io/sitespeed.io
          sitespeed.io https://${{ matrix.environment }}.example.com \
            --budget.configPath=.sitespeed.io/budget.json \
            --plugins.add=@sitespeed.io/plugin-lighthouse

      - name: 安全掃描
        run: |
          npm install -g @zaproxy/action-baseline
          zaproxy/action-baseline -t https://${{ matrix.environment }}.example.com
```

### 3. 發布自動化

自動化發布流程：

**語義化發布工作流程**
```yaml
# .github/workflows/release.yml
name: 發布

on:
  push:
    branches:
      - main

jobs:
  release:
    name: 建立發布
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
          persist-credentials: false

      - name: 設定 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18

      - name: 安裝依賴項
        run: npm ci

      - name: 執行語義化發布
        env:
          GITHUB_TOKEN: ${{ secrets.SEMANTIC_RELEASE_TOKEN }}
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}
        run: npx semantic-release

      - name: 更新文件
        if: steps.semantic-release.outputs.new_release_published == 'true'
        run: |
          npm run docs:generate
          npm run docs:publish

      - name: 建立發布說明
        if: steps.semantic-release.outputs.new_release_published == 'true'
        uses: actions/github-script@v6
        with:
          script: |
            const { data: releases } = await github.rest.repos.listReleases({
              owner: context.repo.owner,
              repo: context.repo.repo,
              per_page: 1
            });
            
            const latestRelease = releases[0];
            const changelog = await generateChangelog(latestRelease);
            
            # 更新發布說明
            await github.rest.repos.updateRelease({
              owner: context.repo.owner,
              repo: context.repo.repo,
              release_id: latestRelease.id,
              body: changelog
            });
```

**發布配置**
```javascript
// .releaserc.js
module.exports = {
  branches: [
    'main',
    { name: 'beta', prerelease: true },
    { name: 'alpha', prerelease: true }
  ],
  plugins: [
    '@semantic-release/commit-analyzer',
    '@semantic-release/release-notes-generator',
    ['@semantic-release/changelog', {
      changelogFile: 'CHANGELOG.md'
    }],
    '@semantic-release/npm',
    ['@semantic-release/git', {
      assets: ['CHANGELOG.md', 'package.json'],
      message: 'chore(release): ${nextRelease.version} [skip ci]\n\n${nextRelease.notes}'
    }],
    '@semantic-release/github'
  ]
};
```

### 4. 開發工作流程自動化

自動化常見開發任務：

**預提交鉤子**
```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files
        args: ['--maxkb=1000']
      - id: check-case-conflict
      - id: check-merge-conflict
      - id: detect-private-key

  - repo: https://github.com/psf/black
    rev: 23.10.0
    hooks:
      - id: black
        language_version: python3.11

  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort
        args: ["--profile", "black"]

  - repo: https://github.com/pycqa/flake8
    rev: 6.1.0
    hooks:
      - id: flake8
        additional_dependencies: [flake8-docstrings]

  - repo: https://github.com/pre-commit/mirrors-eslint
    rev: v8.52.0
    hooks:
      - id: eslint
        files: \.[jt]sx?$
        types: [file]
        additional_dependencies:
          - eslint@8.52.0
          - eslint-config-prettier@9.0.0
          - eslint-plugin-react@7.33.2

  - repo: https://github.com/pre-commit/mirrors-prettier
    rev: v3.0.3
    hooks:
      - id: prettier
        types_or: [css, javascript, jsx, typescript, tsx, json, yaml]

  - repo: local
    hooks:
      - id: unit-tests
        name: 執行單元測試
        entry: npm run test:unit -- --passWithNoTests
        language: system
        pass_filenames: false
        stages: [commit]
```

**開發環境設定**
```bash
#!/bin/bash
# scripts/setup-dev-environment.sh

set -euo pipefail

echo "🚀 正在設定開發環境..."

# 檢查先決條件
check_prerequisites() {
    echo "正在檢查先決條件..."
    
    commands=("git" "node" "npm" "docker" "docker-compose")
    for cmd in "${commands[@]}"; do
        if ! command -v "$cmd" &> /dev/null;
        then
            echo "❌ $cmd 未安裝"
            exit 1
        fi
    done
    
    echo "✅ 所有先決條件已安裝"
}

# 安裝依賴項
install_dependencies() {
    echo "正在安裝依賴項..."
    npm ci
    
    # 安裝全域工具
    npm install -g @commitlint/cli @commitlint/config-conventional
    npm install -g semantic-release
    
    # 安裝 pre-commit
    pip install pre-commit
    pre-commit install
    pre-commit install --hook-type commit-msg
}

# 設定本地服務
setup_services() {
    echo "正在設定本地服務..."
    
    # 建立 docker 網路
    docker network create dev-network 2>/dev/null || true
    
    # 啟動服務
    docker-compose -f docker-compose.dev.yml up -d
    
    # 等待服務
    echo "正在等待服務準備就緒..."
    ./scripts/wait-for-services.sh
}

# 初始化資料庫
initialize_database() {
    echo "正在初始化資料庫..."
    npm run db:migrate
    npm run db:seed
}

# 設定環境變數
setup_environment() {
    echo "正在設定環境變數..."
    
    if [ ! -f .env.local ]; then
        cp .env.example .env.local
        echo "✅ 已從 .env.example 建立 .env.local"
        echo "⚠️ 請使用您的值更新 .env.local"
    fi
}

# 主要執行
main() {
    check_prerequisites
    install_dependencies
    setup_services
    setup_environment
    initialize_database
    
    echo "✅ 開發環境設定完成！"
    echo ""
    echo "Next steps:"
    echo "1. Update .env.local with your configuration"
    echo "2. Run 'npm run dev' to start the development server"
    echo "3. Visit http://localhost:3000"
}

main
```

### 5. 基礎設施自動化

自動化基礎設施佈建：

**Terraform 工作流程**
```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  pull_request:
    paths:
      - 'terraform/**'
      - '.github/workflows/terraform.yml'
  push:
    branches:
      - main
    paths:
      - 'terraform/**'

env:
  TF_VERSION: '1.6.0'
  TF_VAR_project_name: ${{ github.event.repository.name }}

jobs:
  terraform:
    name: Terraform 規劃與應用
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: terraform
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 設定 Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: ${{ env.TF_VERSION }}
          terraform_wrapper: false
      
      - name: 配置 AWS 憑證
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Terraform 格式檢查
        run: terraform fmt -check -recursive
      
      - name: Terraform 初始化
        run: |
          terraform init \
            -backend-config="bucket=${{ secrets.TF_STATE_BUCKET }}" \
            -backend-config="key=${{ github.repository }}/terraform.tfstate" \
            -backend-config="region=us-east-1"
      
      - name: Terraform 驗證
        run: terraform validate
      
      - name: Terraform 規劃
        id: plan
        run: |
          terraform plan -out=tfplan -no-color | tee plan_output.txt
          
          # 提取規劃摘要
          echo "PLAN_SUMMARY<<EOF" >> $GITHUB_ENV
          grep -E '(Plan:|No changes.|# )' plan_output.txt >> $GITHUB_ENV
          echo "EOF" >> $GITHUB_ENV
      
      - name: 評論 PR
        if: github.event_name == 'pull_request'
        uses: actions/github-script@v6
        with:
          script: |
            const output = `#### Terraform 規劃 📖
            ```
            ${process.env.PLAN_SUMMARY}
            ```
            
            *由 @${{ github.actor }} 推送，動作: `${{ github.event_name }}`*`;
            
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
      
      - name: Terraform 應用
        if: github.ref == 'refs/heads/main' && github.event_name == 'push'
        run: terraform apply tfplan
```

### 6. 監控與警報自動化

自動化監控設定：

**監控堆疊部署**
```yaml
# .github/workflows/monitoring.yml
name: 部署監控

on:
  push:
    paths:
      - 'monitoring/**'
      - '.github/workflows/monitoring.yml'
    branches:
      - main

jobs:
  deploy-monitoring:
    name: 部署監控堆疊
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 設定 Helm
        uses: azure/setup-helm@v3
        with:
          version: '3.12.0'
      
      - name: 配置 Kubernetes
        run: |
          echo "${{ secrets.KUBE_CONFIG }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig
      
      - name: 添加 Helm 儲存庫
        run: |
          helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
          helm repo add grafana https://grafana.github.io/helm-charts
          helm repo update
      
      - name: 部署 Prometheus
        run: |
          helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \
            --namespace monitoring \
            --create-namespace \
            --values monitoring/prometheus-values.yaml \
            --wait
      
      - name: 部署 Grafana 儀表板
        run: |
          kubectl apply -f monitoring/dashboards/
      
      - name: 部署警報規則
        run: |
          kubectl apply -f monitoring/alerts/
      
      - name: 設定警報路由
        run: |
          helm upgrade --install alertmanager prometheus-community/alertmanager \
            --namespace monitoring \
            --values monitoring/alertmanager-values.yaml
```

### 7. 依賴項更新自動化

自動化依賴項更新：

**Renovate 配置**
```json
{
  "extends": [
    "config:base",
    ":dependencyDashboard",
    ":semanticCommits",
    ":automergeDigest",
    ":automergeMinor"
  ],
  "schedule": ["after 10pm every weekday", "before 5am every weekday", "every weekend"],
  "timezone": "America/New_York",
  "vulnerabilityAlerts": {
    "labels": ["security"],
    "automerge": true
  },
  "packageRules": [
    {
      "matchDepTypes": ["devDependencies"],
      "automerge": true
    },
    {
      "matchPackagePatterns": ["^@types/"],
      "automerge": true
    },
    {
      "matchPackageNames": ["node"],
      "enabled": false
    },
    {
      "matchPackagePatterns": ["^eslint"],
      "groupName": "eslint packages",
      "automerge": true
    },
    {
      "matchManagers": ["docker"],
      "pinDigests": true
    }
  ],
  "postUpdateOptions": [
    "npmDedupe",
    "yarnDedupeHighest"
  ],
  "prConcurrentLimit": 3,
  "prCreation": "not-pending",
  "rebaseWhen": "behind-base-branch",
  "semanticCommitScope": "deps"
}
```

### 8. 文件自動化

自動化文件生成：

**文件工作流程**
```yaml
# .github/workflows/docs.yml
name: 文件

on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'docs/**'
      - 'README.md'

jobs:
  generate-docs:
    name: 生成文件
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 設定 Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 18
      
      - name: 安裝依賴項
        run: npm ci
      
      - name: 生成 API 文件
        run: |
          npm run docs:api
          npm run docs:typescript
      
      - name: 生成架構圖
        run: |
          npm install -g @mermaid-js/mermaid-cli
          mmdc -i docs/architecture.mmd -o docs/architecture.png
      
      - name: 建置文件網站
        run: |
          npm run docs:build
      
      - name: 部署到 GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./docs/dist
          cname: docs.example.com
```

**文件生成腳本**
```typescript
// scripts/generate-docs.ts
import { Application, TSConfigReader, TypeDocReader } from 'typedoc';
import { generateMarkdown } from './markdown-generator';
import { createApiReference } from './api-reference';

async function generateDocumentation() {
  // TypeDoc 用於 TypeScript 文件
  const app = new Application();
  app.options.addReader(new TSConfigReader());
  app.options.addReader(new TypeDocReader());
  
  app.bootstrap({
    entryPoints: ['src/index.ts'],
    out: 'docs/api',
    theme: 'default',
    includeVersion: true,
    excludePrivate: true,
    readme: 'README.md',
    plugin: ['typedoc-plugin-markdown']
  });
  
  const project = app.convert();
  if (project) {
    await app.generateDocs(project, 'docs/api');
    
    // 生成自定義 Markdown 文件
    await generateMarkdown(project, {
      output: 'docs/guides',
      includeExamples: true,
      generateTOC: true
    });
    
    // 建立 API 參考
    await createApiReference(project, {
      format: 'openapi',
      output: 'docs/openapi.json',
      includeSchemas: true
    });
  }
  
  // 生成架構文件
  await generateArchitectureDocs();
  
  // 生成部署指南
  await generateDeploymentGuides();
}

async function generateArchitectureDocs() {
  const mermaidDiagrams = `
    graph TB
      A[客戶端] --> B[負載平衡器]
      B --> C[網頁伺服器]
      C --> D[應用程式伺服器]
      D --> E[資料庫]
      D --> F[快取]
      D --> G[訊息佇列]
  `;
  
  // 儲存圖表並生成文件
  await fs.writeFile('docs/architecture.mmd', mermaidDiagrams);
}
```

### 9. 安全自動化

自動化安全掃描和合規性：

**安全掃描工作流程**
```yaml
# .github/workflows/security.yml
name: 安全掃描

on:
  push:
    branches: [main, develop]
  pull_request:
  schedule:
    - cron: '0 0 * * 0'  # 每週日

jobs:
  security-scan:
    name: 安全掃描
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: 執行 Trivy 漏洞掃描器
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
      
      - name: 上傳 Trivy 結果
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
      
      - name: 執行 Snyk 安全掃描
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: 執行 OWASP 依賴項檢查
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: ${{ github.repository }}
          path: '.'
          format: 'ALL'
          args: >
            --enableRetired
            --enableExperimental
      
      - name: SonarCloud 掃描
        uses: SonarSource/sonarcloud-github-action@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      
      - name: 執行 Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
      
      - name: GitLeaks 秘密掃描
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 10. 工作流程編排

建立複雜的工作流程編排：

**工作流程協調器**
```typescript
import { EventEmitter } from 'events';
import { Logger } from 'winston';

interface WorkflowStep {
  name: string;
  type: 'parallel' | 'sequential';
  steps?: WorkflowStep[];
  action?: () => Promise<any>;
  retries?: number;
  timeout?: number;
  condition?: () => boolean;
  onError?: 'fail' | 'continue' | 'retry';
}

export class WorkflowOrchestrator extends EventEmitter {
  constructor(
    private logger: Logger,
    private config: WorkflowConfig
  ) {
    super();
  }
  
  async execute(workflow: WorkflowStep): Promise<WorkflowResult> {
    const startTime = Date.now();
    const result: WorkflowResult = {
      success: true,
      steps: [],
      duration: 0
    };
    
    try {
      await this.executeStep(workflow, result);
    } catch (error) {
      result.success = false;
      result.error = error;
      this.emit('workflow:failed', result);
    }
    
    result.duration = Date.now() - startTime;
    this.emit('workflow:completed', result);
    
    return result;
  }
  
  private async executeStep(
    step: WorkflowStep,
    result: WorkflowResult,
    parentPath: string = ''
  ): Promise<void> {
    const stepPath = parentPath ? `${parentPath}.${step.name}` : step.name;
    
    this.emit('step:start', { step: stepPath });
    
    // 檢查條件
    if (step.condition && !step.condition()) {
      this.logger.info(`由於條件，跳過步驟 ${stepPath}`);
      this.emit('step:skipped', { step: stepPath });
      return;
    }
    
    const stepResult: StepResult = {
      name: step.name,
      path: stepPath,
      startTime: Date.now(),
      success: true
    };
    
    try {
      if (step.action) {
        // 執行單一動作
        await this.executeAction(step, stepResult);
      } else if (step.steps) {
        // 執行子步驟
        if (step.type === 'parallel') {
          await this.executeParallel(step.steps, result, stepPath);
        } else {
          await this.executeSequential(step.steps, result, stepPath);
        }
      }
      
      stepResult.endTime = Date.now();
      stepResult.duration = stepResult.endTime - stepResult.startTime;
      result.steps.push(stepResult);
      
      this.emit('step:complete', { step: stepPath, result: stepResult });
    } catch (error) {
      stepResult.success = false;
      stepResult.error = error;
      result.steps.push(stepResult);
      
      this.emit('step:failed', { step: stepPath, error });
      
      if (step.onError === 'fail') {
        throw error;
      }
    }
  }
  
  private async executeAction(
    step: WorkflowStep,
    stepResult: StepResult
  ): Promise<void> {
    const timeout = step.timeout || this.config.defaultTimeout;
    const retries = step.retries || 0;
    
    let lastError: Error;
    
    for (let attempt = 0; attempt <= retries; attempt++) {
      try {
        const result = await Promise.race([
          step.action!(),
          this.createTimeout(timeout)
        ]);
        
        stepResult.output = result;
        return;
      } catch (error) {
        lastError = error as Error;
        
        if (attempt < retries) {
          this.logger.warn(`步驟 ${step.name} 失敗，重試 ${attempt + 1}/${retries}`);
          await this.delay(this.calculateBackoff(attempt));
        }
      }
    }
    
    throw lastError!;
  }
  
  private async executeParallel(
    steps: WorkflowStep[],
    result: WorkflowResult,
    parentPath: string
  ): Promise<void> {
    await Promise.all(
      steps.map(step => this.executeStep(step, result, parentPath))
    );
  }
  
  private async executeSequential(
    steps: WorkflowStep[],
    result: WorkflowResult,
    parentPath: string
  ): Promise<void> {
    for (const step of steps) {
      await this.executeStep(step, result, parentPath);
    }
  }
  
  private createTimeout(ms: number): Promise<never> {
    return new Promise((_, reject) => {
      setTimeout(() => reject(new Error(`超時 ${ms} 毫秒後`)), ms);
    });
  }
  
  private calculateBackoff(attempt: number): number {
    return Math.min(1000 * Math.pow(2, attempt), 30000);
  }
  
  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// 工作流程定義範例
export const deploymentWorkflow: WorkflowStep = {
  name: '部署',
  type: 'sequential',
  steps: [
    {
      name: '部署前',
      type: 'parallel',
      steps: [
        {
          name: '備份資料庫',
          action: async () => {
            // 備份資料庫
          },
          timeout: 300000 // 5 分鐘
        },
        {
          name: '健康檢查',
          action: async () => {
            // 檢查系統健康狀況
          },
          retries: 3
        }
      ]
    },
    {
      name: '部署',
      type: 'sequential',
      steps: [
        {
          name: '藍綠部署切換',
          action: async () => {
            // 將流量切換到新版本
          },
          onError: 'retry',
          retries: 2
        },
        {
          name: '冒煙測試',
          action: async () => {
            // 執行冒煙測試
          },
          onError: 'fail'
        }
      ]
    },
    {
      name: '部署後',
      type: 'parallel',
      steps: [
        {
          name: '通知團隊',
          action: async () => {
            // 發送通知
          },
          onError: 'continue'
        },
        {
          name: '更新監控',
          action: async () => {
            // 更新監控儀表板
          }
        }
      ]
    }
  ]
};
```

## 輸出格式

1. **工作流程分析**：現有流程和自動化機會
2. **CI/CD 管道**：完整的 GitHub Actions/GitLab CI 配置
3. **發布自動化**：語義化版本控制和發布工作流程
4. **開發自動化**：預提交鉤子和設定腳本
5. **基礎設施自動化**：Terraform 和 Kubernetes 工作流程
6. **安全自動化**：掃描和合規性工作流程
7. **文件生成**：自動化文件和圖表
8. **工作流程編排**：複雜工作流程管理
9. **監控整合**：自動化警報和儀表板
10. **實施指南**：逐步設定說明

專注於建立可靠、可維護的自動化，以減少手動工作，同時保持品質和安全標準。
