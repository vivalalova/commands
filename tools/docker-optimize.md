# Docker 優化

您是 Docker 優化專家，專精於建立高效、安全和最小的容器映像。優化 Dockerfile 的大小、建置速度、安全性和運行時性能，同時遵循容器最佳實踐。

## 背景
使用者需要優化 Docker 映像和容器以用於生產環境。專注於減少映像大小、縮短建置時間、實施安全最佳實踐，並確保高效的運行時性能。

## 要求
$ARGUMENTS

## 指示

### 1. 容器優化策略選擇

根據您的應用程式類型和要求選擇正確的優化方法：

**優化策略矩陣**
```python
from typing import Dict, List, Any, Optional
from dataclasses import dataclass
from pathlib import Path
import docker
import json
import subprocess
import tempfile

@dataclass
class OptimizationRecommendation:
    category: str
    priority: str
    impact: str
    effort: str
    description: str
    implementation: str
    validation: str

class SmartDockerOptimizer:
    def __init__(self):
        self.client = docker.from_env()
        self.optimization_strategies = {
            'web_application': {
                'priorities': ['security', 'size', 'startup_time', 'build_speed'],
                'recommended_base': 'alpine 或 distroless',
                'patterns': ['multi_stage', 'layer_caching', 'dependency_optimization']
            },
            'microservice': {
                'priorities': ['size', 'startup_time', 'security', 'resource_usage'],
                'recommended_base': 'scratch 或 distroless',
                'patterns': ['minimal_dependencies', 'static_compilation', 'health_checks']
            },
            'data_processing': {
                'priorities': ['performance', 'resource_usage', 'build_speed', 'size'],
                'recommended_base': 'slim 或特定運行時',
                'patterns': ['parallel_processing', 'volume_optimization', 'memory_tuning']
            },
            'machine_learning': {
                'priorities': ['gpu_support', 'model_size', 'inference_speed', 'dependency_mgmt'],
                'recommended_base': 'nvidia/cuda 或 tensorflow/tensorflow',
                'patterns': ['model_optimization', 'cuda_optimization', 'multi_stage_ml']
            }
        }
    
    def detect_application_type(self, project_path: str) -> str:
        """從專案結構自動檢測應用程式類型"""
        path = Path(project_path)
        
        # 檢查 ML 指標
        ml_indicators = ['requirements.txt', 'environment.yml', 'model.pkl', 'model.h5']
        ml_keywords = ['tensorflow', 'pytorch', 'scikit-learn', 'keras', 'numpy', 'pandas']
        
        if any((path / f).exists() for f in ml_indicators):
            if (path / 'requirements.txt').exists():
                with open(path / 'requirements.txt') as f:
                    content = f.read().lower()
                    if any(keyword in content for keyword in ml_keywords):
                        return 'machine_learning'
        
        # 檢查微服務指標
        if any(f.name in ['go.mod', 'main.go', 'cmd'] for f in path.iterdir()):
            return 'microservice'
        
        # 檢查資料處理
        data_indicators = ['airflow', 'kafka', 'spark', 'hadoop']
        if any((path / f).exists() for f in ['docker-compose.yml', 'k8s']):
            return 'data_processing'
        
        # 預設為 Web 應用程式
        return 'web_application'
    
    def analyze_dockerfile_comprehensively(self, dockerfile_path: str, project_path: str) -> Dict[str, Any]:
        """
        全面 Dockerfile 分析與現代優化建議
        """
        app_type = self.detect_application_type(project_path)
        
        with open(dockerfile_path, 'r') as f:
            content = f.read()
        
        analysis = {
            'application_type': app_type,
            'current_issues': [],
            'optimization_opportunities': [],
            'security_risks': [],
            'performance_improvements': [],
            'size_optimizations': [],
            'build_optimizations': [],
            'recommendations': []
        }
        
        # 全面分析
        self._analyze_base_image_strategy(content, analysis)
        self._analyze_layer_efficiency(content, analysis)
        self._analyze_security_posture(content, analysis)
        self._analyze_build_performance(content, analysis)
        self._analyze_runtime_optimization(content, analysis)
        self._generate_strategic_recommendations(analysis, app_type)
        
        return analysis
    
    def _analyze_base_image_strategy(self, content: str, analysis: Dict):
        """分析基礎映像選擇和優化機會"""
        base_image_patterns = {
            'outdated_versions': {
                'pattern': r'FROM\s+([^:]+):(?!latest)([0-9]+\.[0-9]+)(?:\s|$)',
                'severity': 'medium',
                'recommendation': '考慮更新到最新的穩定版本'
            },
            'latest_tag': {
                'pattern': r'FROM\s+([^:]+):latest',
                'severity': 'high',
                'recommendation': '固定到特定版本以實現可重現的建置'
            },
            'large_base_images': {
                'patterns': [
                    r'FROM\s+ubuntu(?!.*slim)',
                    r'FROM\s+centos',
                    r'FROM\s+debian(?!.*slim)',
                    r'FROM\s+node(?!.*alpine)'
                ],
                'severity': 'medium',
                'recommendation': '考慮使用較小的替代方案（alpine、slim、distroless）'
            },
            'missing_multi_stage': {
                'pattern': r'FROM\s+(?!.*AS\s+)',
                'count_threshold': 1,
                'severity': 'low',
                'recommendation': '考慮多階段建置以獲得更小的最終映像'
            }
        }
        
        # 檢查基礎映像優化機會
        for issue_type, config in base_image_patterns.items():
            if 'patterns' in config:
                for pattern in config['patterns']:
                    if re.search(pattern, content, re.IGNORECASE):
                        analysis['size_optimizations'].append({
                            'type': issue_type,
                            'severity': config['severity'],
                            'description': config['recommendation'],
                            'potential_savings': self._estimate_size_savings(issue_type)
                        })
            elif 'pattern' in config:
                matches = re.findall(config['pattern'], content, re.IGNORECASE)
                if matches:
                    analysis['current_issues'].append({
                        'type': issue_type,
                        'severity': config['severity'],
                        'instances': len(matches),
                        'description': config['recommendation']
                    })
    
    def _analyze_layer_efficiency(self, content: str, analysis: Dict):
        """分析 Docker 層效率和快取機會"""
        lines = content.split('\n')
        run_commands = [line for line in lines if line.strip().startswith('RUN')]
        
        # 多個 RUN 命令分析
        if len(run_commands) > 3:
            analysis['build_optimizations'].append({
                'type': '過多層',
                'severity': 'medium',
                'current_count': len(run_commands),
                'recommended_count': '1-3',
                'description': f'發現 {len(run_commands)} 個 RUN 命令。考慮合併相關操作。',
                'implementation': '使用 && 合併 RUN 命令以減少層'
            })
        
        # 套件管理器清理分析
        package_managers = {
            'apt': {'install': r'apt-get\s+install', 'cleanup': r'rm\s+-rf\s+/var/lib/apt/lists'},
            'yum': {'install': r'yum\s+install', 'cleanup': r'yum\s+clean\s+all'},
            'apk': {'install': r'apk\s+add', 'cleanup': r'rm\s+-rf\s+/var/cache/apk'}
        }
        
        for pm_name, patterns in package_managers.items():
            if re.search(patterns['install'], content) and not re.search(patterns['cleanup'], content):
                analysis['size_optimizations'].append({
                    'type': f'{pm_name}_缺少清理',
                    'severity': 'medium',
                    'description': f'缺少 {pm_name} 快取清理',
                    'potential_savings': '50-200MB',
                    'implementation': '在同一 RUN 層中添加清理命令'
                })
        
        # 複製優化分析
        copy_commands = [line for line in lines if line.strip().startswith(('COPY', 'ADD'))]
        if any('.' in cmd for cmd in copy_commands):
            analysis['build_optimizations'].append({
                'type': '低效複製',
                'severity': 'low',
                'description': '考慮使用 .dockerignore 和特定的 COPY 命令',
                'implementation': '僅複製必要的檔案以提高建置快取效率'
            })
    
    def _generate_strategic_recommendations(self, analysis: Dict, app_type: str):
        """根據應用程式類型生成戰略優化建議"""
        strategy = self.optimization_strategies[app_type]
        
        # 基於優先級的建議
        for priority in strategy['priorities']:
            if priority == 'security':
                analysis['recommendations'].append(OptimizationRecommendation(
                    category='安全',
                    priority='高',
                    impact='關鍵',
                    effort='中',
                    description='實施安全掃描和強化',
                    implementation=self._get_security_implementation(app_type),
                    validation='運行 Trivy 和 Hadolint 掃描'
                ))
            elif priority == 'size':
                analysis['recommendations'].append(OptimizationRecommendation(
                    category='大小優化',
                    priority='高',
                    impact='高',
                    effort='低',
                    description=f'使用 {strategy["recommended_base"]} 基礎映像',
                    implementation=self._get_size_implementation(app_type),
                    validation='比較映像大小前後'
                ))
            elif priority == 'startup_time':
                analysis['recommendations'].append(OptimizationRecommendation(
                    category='啟動性能',
                    priority='中',
                    impact='高',
                    effort='中',
                    description='優化應用程式啟動時間',
                    implementation=self._get_startup_implementation(app_type),
                    validation='測量容器啟動時間'
                ))
    
    def _estimate_size_savings(self, optimization_type: str) -> str:
        """估計優化的潛在大小節省"""
        savings_map = {
            'large_base_images': '200-800MB',
            'apt_cleanup_missing': '50-200MB',
            'yum_cleanup_missing': '100-300MB',
            'apk_cleanup_missing': '20-100MB',
            'excessive_layers': '10-50MB',
            'multi_stage_optimization': '100-500MB'
        }
        return savings_map.get(optimization_type, '10-50MB')
    
    def _get_security_implementation(self, app_type: str) -> str:
        """根據應用程式類型獲取安全實施"""
        implementations = {
            'web_application': '非 root 使用者，安全掃描，最小套件',
            'microservice': 'Distroless 基礎，靜態編譯，能力丟棄',
            'data_processing': '安全資料處理，加密卷，網路策略',
            'machine_learning': '模型加密，安全模型服務，GPU 安全'
        }
        return implementations.get(app_type, '標準安全強化')
```

**進階多框架 Dockerfile 生成器**
```python
class FrameworkOptimizedDockerfileGenerator:
    def __init__(self):
        self.templates = {
            'node_express': self._generate_node_express_optimized,
            'python_fastapi': self._generate_python_fastapi_optimized,
            'python_django': self._generate_python_django_optimized,
            'golang_gin': self._generate_golang_optimized,
            'java_spring': self._generate_java_spring_optimized,
            'rust_actix': self._generate_rust_optimized,
            'dotnet_core': self._generate_dotnet_optimized
        }
    
    def generate_optimized_dockerfile(self, framework: str, config: Dict[str, Any]) -> str:
        """為特定框架生成高度優化的 Dockerfile"""
        if framework not in self.templates:
            raise ValueError(f"不支援的框架: {framework}")
        
        return self.templates[framework](config)
    
    def _generate_node_express_optimized(self, config: Dict) -> str:
        """
        生成優化的 Node.js Express Dockerfile
        """
        node_version = config.get('node_version', '20')
        use_bun = config.get('use_bun', False)
        
        if use_bun:
            return f"""
# 使用 Bun 優化的 Node.js - 超快速建置和運行時
FROM oven/bun:{config.get('bun_version', 'latest')} AS base

# 安裝依賴項 (Bun 比 npm 快得多)
WORKDIR /app
COPY package.json bun.lockb* ./
RUN bun install --frozen-lockfile --production

# 建置階段
FROM base AS build
COPY . .
RUN bun run build

# 生產階段
FROM gcr.io/distroless/nodejs{node_version}-debian11
WORKDIR /app

# 複製已建置的應用程式
COPY --from=build --chown=nonroot:nonroot /app/dist ./dist
COPY --from=build --chown=nonroot:nonroot /app/node_modules ./node_modules
COPY --from=build --chown=nonroot:nonroot /app/package.json ./

# 安全：以非 root 運行
USER nonroot

# 健康檢查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["node", "-e", "require('http').get('http://localhost:3000/health', (res) => process.exit(res.statusCode === 200 ? 0 : 1)).on('error', () => process.exit(1))"]

EXPOSE 3000
CMD ["node", "dist/index.js"]
"""
        
        return f"""
# 優化的 Node.js Express - 生產就緒的多階段建置
FROM node:{node_version}-alpine AS deps

# 安裝 dumb-init 以正確處理信號
RUN apk add --no-cache dumb-init

# 建立應用程式目錄並設定適當權限
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
USER nodejs

# 複製套件檔案並安裝依賴項
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production --no-audit --no-fund && npm cache clean --force

# 建置階段
FROM node:{node_version}-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --no-audit --no-fund
COPY . .
RUN npm run build && npm run test

# 生產階段
FROM node:{node_version}-alpine AS production

# 安裝 dumb-init
RUN apk add --no-cache dumb-init

# 建立使用者和應用程式目錄
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
USER nodejs

# 複製已建置的應用程式
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=nodejs:nodejs /app/package.json ./

# 健康檢查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node healthcheck.js || exit 1

# 暴露埠
EXPOSE 3000

# 使用 dumb-init 以正確處理信號
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
"""
    
    def _generate_python_fastapi_optimized(self, config: Dict) -> str:
        """
        生成優化的 Python FastAPI Dockerfile
        """
        python_version = config.get('python_version', '3.11')
        use_uv = config.get('use_uv', True)
        
        if use_uv:
            return f"""
# 使用 uv 套件管理器實現超快速 Python
FROM python:{python_version}-slim AS base

# 安裝 uv - 最快的 Python 套件管理器
RUN pip install uv

# 建置依賴項
FROM base AS build
WORKDIR /app

# 複製 requirements 並使用 uv 安裝依賴項
COPY requirements.txt ./
RUN uv venv /opt/venv && \
    . /opt/venv/bin/activate && \
    uv pip install --no-cache-dir -r requirements.txt

# 生產階段
FROM python:{python_version}-slim AS production

# 安裝安全更新
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends dumb-init && \
    rm -rf /var/lib/apt/lists/*

# 建立非 root 使用者
RUN useradd -m -u 1001 appuser
WORKDIR /app

# 複製虛擬環境
COPY --from=build /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 複製應用程式
COPY --chown=appuser:appuser . ./

# 安全：以非 root 運行
USER appuser

# 健康檢查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health', timeout=5)"

EXPOSE 8000

# 使用 dumb-init 和 Gunicorn 進行生產
ENTRYPOINT ["dumb-init", "--"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "app.main:app"]
"""
        
        # 標準優化 Python Dockerfile
        return f"""
# 優化的 Python FastAPI - 生產就緒
FROM python:{python_version}-slim AS build

# 安裝建置依賴項
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 建立虛擬環境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安裝 Python 依賴項
COPY requirements.txt ./
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# 生產階段
FROM python:{python_version}-slim AS production

# 安裝運行時依賴項和安全更新
RUN apt-get update && apt-get install -y --no-install-recommends \
    dumb-init \
    && apt-get upgrade -y \
    && rm -rf /var/lib/apt/lists/*

# 建立非 root 使用者
RUN useradd -m -u 1001 appuser
WORKDIR /app

# 複製虛擬環境
COPY --from=build /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONOPTIMIZE=2

# 複製應用程式
COPY --chown=appuser:appuser . ./

# 安全：以非 root 運行
USER appuser

# 健康檢查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health', timeout=5)"

EXPOSE 8000

# 帶有正確信號處理的生產伺服器
ENTRYPOINT ["dumb-init", "--"]
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "app.main:app"]
"""
    
    def _generate_golang_optimized(self, config: Dict) -> str:
        """
        生成優化的 Go Dockerfile，具有最小的最終映像
        """
        go_version = config.get('go_version', '1.21')
        
        return f"""
# 優化的 Go 建置 - 超最小最終映像
FROM golang:{go_version}-alpine AS build

# 安裝 git 以用於 Go 模組
RUN apk add --no-cache git ca-certificates tzdata

# 建立建置目錄
WORKDIR /build

# 複製 go mod 檔案並下載依賴項
COPY go.mod go.sum ./
RUN go mod download && go mod verify

# 複製原始碼
COPY . .

# 使用優化建置靜態二進位檔
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -installsuffix cgo \
    -o app .

# 使用 UPX 壓縮二進位檔（可選，大小減少 50-70%）
RUN upx --best --lzma app

# 最終階段 - 最小 scratch 映像
FROM scratch

# 複製必要的檔案
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=build /build/app /app

# 健康檢查 (使用應用程式本身)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/app", "--health-check"]

EXPOSE 8080

# 運行二進位檔
ENTRYPOINT ["/app"]
"""
    
    def _check_base_image(self, content, analysis):
        """增強的基礎映像優化分析"""
        from_match = re.search(r'^FROM\s+(.+?)(?:\s+AS\s+\w+)?$', content, re.MULTILINE)
        if from_match:
            base_image = from_match.group(1)
            
            # 檢查最新標籤
            if ':latest' in base_image or not ':' in base_image:
                analysis['security_risks'].append({
                    'issue': '使用 latest 或無標籤',
                    'severity': 'HIGH',
                    'fix': '固定到特定版本',
                    'example': f'FROM {base_image.split(":")[0]}:1.2.3',
                    'impact': '不可預測的建置，安全漏洞'
                })
            
            # 增強的基礎映像建議
            optimization_recommendations = {
                'ubuntu': {
                    'alternatives': ['ubuntu:22.04-slim', 'debian:bullseye-slim', 'alpine:3.18'],
                    'savings': '400-600MB',
                    'notes': '生產環境考慮 distroless'
                },
                'debian': {
                    'alternatives': ['debian:bullseye-slim', 'alpine:3.18', 'gcr.io/distroless/base'],
                    'savings': '300-500MB',
                    'notes': 'Distroless 提供更好的安全性'
                },
                'centos': {
                    'alternatives': ['alpine:3.18', 'gcr.io/distroless/base', 'ubuntu:22.04-slim'],
                    'savings': '200-400MB',
                    'notes': 'CentOS 已棄用，遷移到替代方案'
                },
                'node': {
                    'alternatives': ['node:20-alpine', 'node:20-slim', 'gcr.io/distroless/nodejs20'],
                    'savings': '300-700MB',
                    'notes': 'Alpine 最小，distroless 最安全'
                },
                'python': {
                    'alternatives': ['python:3.11-slim', 'python:3.11-alpine', 'gcr.io/distroless/python3'],
                    'savings': '400-800MB',
                    'notes': 'Slim 平衡大小和相容性'
                }
            }
            
            for base_name, config in optimization_recommendations.items():
                if base_name in base_image and 'slim' not in base_image and 'alpine' not in base_image:
                    analysis['size_impact'].append({
                        'issue': f'大型基礎映像: {base_image}',
                        'impact': config['savings'],
                        'alternatives': config['alternatives'],
                        'recommendation': f"切換到 {config['alternatives'][0]} 以獲得最佳大小/相容性平衡",
                        'notes': config['notes']
                    })
            
            # 檢查已棄用或不安全的基礎映像
            deprecated_images = {
                'centos:7': '已達 EOL，遷移到 Rocky Linux 或 Alpine',
                'ubuntu:18.04': 'LTS 已結束，升級到 ubuntu:22.04',
                'node:14': 'Node 14 已 EOL，升級到 node:18 或 node:20',
                'python:3.8': 'Python 3.8 即將達到 EOL，升級到 3.11+'
            }
            
            for deprecated, message in deprecated_images.items():
                if deprecated in base_image:
                    analysis['security_risks'].append({
                        'issue': f'已棄用的基礎映像: {deprecated}',
                        'severity': 'MEDIUM',
                        'fix': message,
                        'impact': '安全漏洞，無安全更新'
                    })
    
    def _check_layer_optimization(self, content: str, analysis: Dict):
        """使用現代最佳實踐增強層優化分析"""
        lines = content.split('\n')
        
        # 檢查多個 RUN 命令
        run_commands = [line for line in lines if line.strip().startswith('RUN')]
        if len(run_commands) > 5:
            analysis['build_performance'].append({
                'issue': f'過多的 RUN 命令 ({len(run_commands)})',
                'impact': f'建立 {len(run_commands)} 個不必要的層',
                'fix': '使用 && 合併相關的 RUN 命令',
                'optimization': f'可以減少到 2-3 層，節省約 {len(run_commands) * 10}MB'
            })
        
        # 增強的套件管理器清理檢查
        package_managers = {
            'apt': {
                'install_pattern': r'RUN.*apt-get.*install',
                'cleanup_pattern': r'rm\s+-rf\s+/var/lib/apt/lists',
                'update_pattern': r'apt-get\s+update',
                'combined_check': r'RUN.*apt-get\s+update.*&&.*apt-get\s+install.*&&.*rm\s+-rf\s+/var/lib/apt/lists/*',
                'recommended_pattern': 'RUN apt-get update && apt-get install -y --no-install-recommends <packages> && rm -rf /var/lib/apt/lists/*'
            },
            'yum': {
                'install_pattern': r'RUN.*yum.*install',
                'cleanup_pattern': r'yum\s+clean\s+all',
                'recommended_pattern': 'RUN yum install -y <packages> && yum clean all'
            },
            'apk': {
                'install_pattern': r'RUN.*apk.*add',
                'cleanup_pattern': r'--no-cache|rm\s+-rf\s+/var/cache/apk',
                'recommended_pattern': 'RUN apk add --no-cache <packages>'
            },
            'pip': {
                'install_pattern': r'RUN.*pip.*install',
                'cleanup_pattern': r'--no-cache-dir|pip\s+cache\s+purge',
                'recommended_pattern': 'RUN pip install --no-cache-dir <packages>'
            }
        }
        
        for pm_name, patterns in package_managers.items():
            has_install = re.search(patterns['install_pattern'], content)
            has_cleanup = re.search(patterns['cleanup_pattern'], content)
            
            if has_install and not has_cleanup:
                potential_savings = {
                    'apt': '50-200MB',
                    'yum': '100-300MB', 
                    'apk': '5-50MB',
                    'pip': '20-100MB'
                }.get(pm_name, '10-50MB')
                
                analysis['size_impact'].append({
                    'issue': f'{pm_name} 套件管理器缺少清理',
                    'impact': potential_savings,
                    'fix': '在同一 RUN 命令中添加清理',
                    'example': patterns['recommended_pattern'],
                    'severity': 'MEDIUM'
                })
        
        # 檢查低效的 COPY 操作
        copy_commands = [line for line in lines if line.strip().startswith(('COPY', 'ADD'))]
        for cmd in copy_commands:
            if 'COPY . .' in cmd or 'COPY ./ ./' in cmd:
                analysis['build_performance'].append({
                    'issue': '低效的 COPY 命令複製整個上下文',
                    'impact': '建置快取效率低，建置速度慢',
                    'fix': '使用特定的 COPY 命令和 .dockerignore',
                    'example': 'COPY package*.json ./ && COPY src/ ./src/',
                    'note': '先複製依賴項檔案以獲得更好的快取'
                })
        
        # 檢查 BuildKit 優化
        if '--mount=type=cache' not in content:
            analysis['build_performance'].append({
                'issue': '缺少 BuildKit 快取掛載',
                'impact': '建置速度慢，無依賴項快取',
                'fix': '使用 BuildKit 快取掛載用於套件管理器',
                'example': 'RUN --mount=type=cache,target=/root/.cache/pip pip install -r requirements.txt',
                'note': '需要 DOCKER_BUILDKIT=1'
            })
        
        # 檢查多階段建置機會
        from_statements = re.findall(r'FROM\s+([^\s]+)', content)
        if len(from_statements) == 1 and any(keyword in content.lower() for keyword in ['build', 'compile', 'npm install', 'pip install']):
            analysis['size_impact'].append({
                'issue': '單階段建置與開發依賴項',
                'impact': '100-500MB 來自建置工具和開發依賴項',
                'fix': '實施多階段建置',
                'example': '分離建置和運行時階段',
                'potential_savings': '200-800MB'
            })
```

### 2. 進階多階段建置策略

實施複雜的多階段建置，並採用現代優化技術：

**超優化多階段模式**
```dockerfile
# 模式 1：帶有 Bun 的 Node.js - 下一代 JavaScript 運行時
# 安裝速度快 5 倍，運行時快 4 倍，映像小 90%
FROM oven/bun:1.0-alpine AS base

# 階段 1：使用 Bun 進行依賴項解析
FROM base AS deps
WORKDIR /app

# Bun 鎖定檔案用於確定性建置
COPY package.json bun.lockb* ./

# 超快速依賴項安裝
RUN bun install --frozen-lockfile --production

# 階段 2：使用開發依賴項建置
FROM base AS build
WORKDIR /app

# 複製套件檔案
COPY package.json bun.lockb* ./

# 安裝所有依賴項（包括開發）
RUN bun install --frozen-lockfile

# 複製原始碼並建置
COPY . .
RUN bun run build && bun test

# 階段 3：安全掃描（可選但建議）
FROM build AS security-scan
RUN apk add --no-cache curl
# 下載並運行 Trivy 進行漏洞掃描
RUN curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin && \
    trivy fs --exit-code 1 --no-progress --severity HIGH,CRITICAL /app

# 階段 4：使用 distroless 實現超最小生產
FROM gcr.io/distroless/nodejs20-debian11 AS production

# 僅複製生產所需的內容
COPY --from=deps --chown=nonroot:nonroot /app/node_modules ./node_modules
COPY --from=build --chown=nonroot:nonroot /app/dist ./dist
COPY --from=build --chown=nonroot:nonroot /app/package.json ./

# Distroless 已以非 root 運行
USER nonroot

# 健康檢查
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
    CMD ["node", "-e", "require('http').get('http://localhost:3000/health',(r)=>process.exit(r.statusCode===200?0:1)).on('error',()=>process.exit(1))"]

EXPOSE 3000
CMD ["node", "dist/index.js"]
```

**使用 UV 套件管理器實現進階 Python 多階段**
```dockerfile
# 模式 2：帶有 UV 的 Python - 比 pip 快 10-100 倍
FROM python:3.11-slim AS base

# 安裝 UV - 下一代 Python 套件管理器
RUN pip install uv

# 階段 1：使用 UV 進行依賴項解析
FROM base AS deps
WORKDIR /app

# 複製 requirements
COPY requirements.txt requirements-dev.txt ./

# 建立虛擬環境並使用 UV 安裝生產依賴項
RUN uv venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
RUN uv pip install --no-cache-dir -r requirements.txt

# 階段 2：建置和測試
FROM base AS build
WORKDIR /app

# 安裝所有依賴項，包括開發
RUN uv venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"
COPY requirements*.txt ./
RUN uv pip install --no-cache-dir -r requirements.txt -r requirements-dev.txt

# 複製原始碼並運行測試
COPY . .
RUN python -m pytest tests/ --cov=src --cov-report=term-missing
RUN python -m black --check src/
RUN python -m isort --check-only src/
RUN python -m mypy src/

# 階段 3：安全和合規性掃描
FROM build AS security
RUN uv pip install safety bandit
RUN safety check
RUN bandit -r src/ -f json -o bandit-report.json

# 階段 4：使用 distroless 優化生產
FROM gcr.io/distroless/python3-debian11 AS production

# 複製虛擬環境和應用程式
COPY --from=deps /opt/venv /opt/venv
COPY --from=build /app/src ./src
COPY --from=build /app/requirements.txt ./

# 設定生產環境
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONOPTIMIZE=2

# 健康檢查
HEALTHCHECK --interval=30s --timeout=10s --start-period=10s --retries=3 \
    CMD ["python", "-c", "import urllib.request; urllib.request.urlopen('http://localhost:8000/health', timeout=5)"]

EXPOSE 8000
CMD ["python", "-m", "gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "uvicorn.workers.UvicornWorker", "src.main:app"]
```

**帶有 Scratch 基礎的 Go 靜態二進位檔**
```dockerfile
# 模式 3：帶有超最小 scratch 基礎的 Go
FROM golang:1.21-alpine AS base

# 安裝 git 以用於 Go 模組
RUN apk add --no-cache git ca-certificates tzdata

# 建立建置目錄
WORKDIR /build

# 複製 go mod 檔案並下載依賴項
COPY go.mod go.sum ./
RUN go mod download && go mod verify

# 複製原始碼
COPY . .

# 使用優化建置靜態二進位檔
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build \
    -ldflags='-w -s -extldflags "-static"' \
    -a -installsuffix cgo \
    -o app .

# 使用 UPX 壓縮二進位檔（可選，大小減少 50-70%）
RUN upx --best --lzma app

# 最終階段 - 最小 scratch 映像
FROM scratch

# 複製必要的檔案
COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build /usr/share/zoneinfo /usr/share/zoneinfo
COPY --from=build /build/app /app

# 健康檢查 (使用應用程式本身內建的健康端點)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/app", "--health-check"]

EXPOSE 8080

# 運行二進位檔
ENTRYPOINT ["/app"]
```

**帶有交叉編譯和安全的 Rust**
```dockerfile
# 模式 4：帶有 musl 的 Rust 用於靜態連結
FROM rust:1.70-alpine AS base

# 安裝 musl 開發工具
RUN apk add --no-cache musl-dev openssl-dev

# 階段 1：依賴項快取
FROM base AS deps
WORKDIR /app

# 複製 Cargo 檔案
COPY Cargo.toml Cargo.lock ./

# 建立虛擬 main 並建置依賴項
RUN mkdir src && echo 'fn main() {}' > src/main.rs
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/app/target \
    cargo build --release --target x86_64-unknown-linux-musl

# 階段 2：建置應用程式
FROM base AS build
WORKDIR /app

# 複製快取中的依賴項
COPY --from=deps /usr/local/cargo /usr/local/cargo
COPY . .

# 建置優化靜態二進位檔
RUN --mount=type=cache,target=/usr/local/cargo/registry \
    --mount=type=cache,target=/app/target \
    cargo build --release --target x86_64-unknown-linux-musl && \
    cp target/x86_64-unknown-linux-musl/release/app /app/app

# 剝離二進位檔以減小大小
RUN strip /app/app

# 階段 3：安全掃描
FROM build AS security
RUN cargo audit
RUN cargo clippy -- -D warnings

# 階段 4：最小 scratch 映像
FROM scratch AS production

# 複製靜態二進位檔
COPY --from=build /app/app /app

# 健康檢查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD ["/app", "--health"]

EXPOSE 8000
ENTRYPOINT ["/app"]
```

**帶有 GraalVM Native Image 的 Java Spring Boot**
```dockerfile
# 模式 5：帶有 GraalVM Native Image 的 Java（亞秒級啟動）
FROM ghcr.io/graalvm/graalvm-ce:java17 AS base

# 安裝 native-image 組件
RUN gu install native-image

# 階段 1：依賴項
FROM base AS deps
WORKDIR /app

# 複製 Maven/Gradle 檔案
COPY pom.xml ./
COPY .mvn .mvn
COPY mvnw ./

# 下載依賴項
RUN ./mvnw dependency:go-offline

# 階段 2：建置應用程式
FROM base AS build
WORKDIR /app

# 複製依賴項和原始碼
COPY --from=deps /root/.m2 /root/.m2
COPY . .

# 建置 JAR
RUN ./mvnw clean package -DskipTests

# 建置原生映像
RUN native-image \
    -jar target/*.jar \
    --no-fallback \
    --static \
    --libc=musl \
    -H:+ReportExceptionStackTraces \
    -H:+AddAllCharsets \
    -H:IncludeResourceBundles=sun.util.resources.TimeZoneNames \
    app

# 階段 3：測試
FROM build AS test
RUN ./mvnw test

# 階段 4：超最小最終映像（20-50MB vs 200-300MB）
FROM scratch AS production

# 複製原生二進位檔
COPY --from=build /app/app /app

# 健康檢查
HEALTHCHECK --interval=30s --timeout=3s --start-period=2s --retries=3 \
    CMD ["/app", "--health"]

EXPOSE 8080
ENTRYPOINT ["/app"]
```

**Python 多階段範例**
```dockerfile
# 階段 1：建置依賴項
FROM python:3.11-slim AS builder

# 安裝建置依賴項
RUN apt-get update && apt-get install -y --no-install-recommends \
    gcc \
    libc6-dev \
    && rm -rf /var/lib/apt/lists/*

# 建立虛擬環境
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 安裝 Python 依賴項
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt

# 階段 2：運行時
FROM python:3.11-slim AS runtime

# 從建置器複製虛擬環境
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# 建立非 root 使用者
RUN useradd -m -u 1001 appuser

WORKDIR /app

# 複製應用程式
COPY --chown=appuser:appuser . ./

USER appuser

# Gunicorn 用於生產
CMD ["gunicorn", "--bind", "0.0.0.0:8000", "--workers", "4", "app:application"]
```

### 3. 映像大小優化

最小化 Docker 映像大小：

**大小縮減技術**
```dockerfile
# 基於 Alpine 的優化
FROM alpine:3.18

# 僅安裝必要的套件
RUN apk add --no-cache \
    python3 \
    py3-pip \
    && pip3 install --no-cache-dir --upgrade pip

# 對於 pip 使用 --no-cache-dir
COPY requirements.txt ./
RUN pip3 install --no-cache-dir -r requirements.txt

# 移除不必要的檔案
RUN find /usr/local -type d -name __pycache__ -exec rm -rf {} + \
    && find /usr/local -type f -name '*.pyc' -delete

# 帶有 scratch 映像的 Golang 範例
FROM golang:1.21-alpine AS builder
WORKDIR /build
COPY . .
# 建置靜態二進位檔
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-s -w' -o app .

# 最終階段：scratch
FROM scratch
# 僅複製二進位檔
COPY --from=builder /build/app /app
# 複製 SSL 憑證以用於 HTTPS
COPY --from=builder /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
ENTRYPOINT ["/app"]
```

**層優化腳本**
```python
def optimize_dockerfile_layers(dockerfile_content):
    """
    優化 Dockerfile 層
    """
    optimizations = []
    
    # 合併 RUN 命令
    run_commands = re.findall(r'^RUN\s+(.+?)(?=^(?:RUN|FROM|COPY|ADD|ENV|EXPOSE|CMD|ENTRYPOINT|WORKDIR)|\Z)', 
                             dockerfile_content, re.MULTILINE | re.DOTALL)
    
    if len(run_commands) > 1:
        combined = ' && \
    '.join(cmd.strip() for cmd in run_commands)
        optimizations.append({
            'original': '\n'.join(f'RUN {cmd}' for cmd in run_commands),
            'optimized': f'RUN {combined}',
            'benefit': f'將 {len(run_commands)} 層減少到 1 層'
        })
    
    # 優化套件安裝
    apt_install = re.search(r'RUN\s+apt-get\s+update.*?apt-get\s+install\s+(.+?)(?=^(?:RUN|FROM)|\Z)', 
                           dockerfile_content, re.MULTILINE | re.DOTALL)
    
    if apt_install:
        packages = apt_install.group(1)
        optimized = f"""RUN apt-get update && apt-get install -y --no-install-recommends \
    {packages.strip()} \
    && rm -rf /var/lib/apt/lists/*"""
        
        optimizations.append({
            'original': apt_install.group(0),
            'optimized': optimized,
            'benefit': '透過清理 apt 快取減少映像大小'
        })
    
    return optimizations
```

### 4. 建置性能優化

加速 Docker 建置：

**.dockerignore 優化**
```
# .dockerignore
# 版本控制
.git
.gitignore

# 開發
.vscode
.idea
*.swp
*.swo

# 依賴項
node_modules
vendor
venv
__pycache__

# 建置工件
dist
build
*.egg-info
target

# 測試
test
tests
*.test.js
*.spec.js
coverage
.pytest_cache

# 文件
docs
*.md
LICENSE

# 環境
.env
.env.*

# 日誌
*.log
logs

# 作業系統檔案
.DS_Store
Thumbs.db

# CI/CD
.github
.gitlab
.circleci
Jenkinsfile

# Docker
Dockerfile*
docker-compose*
.dockerignore
```

**建置快取優化**
```dockerfile
# 優化建置快取
FROM node:18-alpine

WORKDIR /app

# 先複製套件檔案（更改頻率較低）
COPY package*.json ./

# 安裝依賴項（如果套件檔案未更改，則快取）
RUN npm ci --only=production

# 複製原始碼（更改頻率較高）
COPY . .

# 建置應用程式
RUN npm run build

# 使用 BuildKit 快取掛載
FROM node:18-alpine AS builder
WORKDIR /app

# 掛載套件管理器快取
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# 掛載建置工件快取
RUN --mount=type=cache,target=/app/.cache \
    npm run build
```

### 5. 安全強化

實施安全最佳實踐：

**安全強化 Dockerfile**
```dockerfile
# 使用特定版本和最小基礎映像
FROM alpine:3.18.4

# 安裝安全更新
RUN apk update && apk upgrade && apk add --no-cache \
    ca-certificates \
    && rm -rf /var/cache/apk/*

# 建立非 root 使用者
RUN addgroup -g 1001 -S appgroup && \
    adduser -S appuser -u 1001 -G appgroup

# 設定安全權限
RUN mkdir /app && chown -R appuser:appgroup /app
WORKDIR /app

# 以正確的所有權複製
COPY --chown=appuser:appgroup . .

# 丟棄所有能力
USER appuser

# 只讀根檔案系統
# 為可寫目錄添加卷
VOLUME ["/tmp", "/app/logs"]

# 安全標籤
LABEL security.scan="trivy" \
      security.updates="auto"

# 帶有超時的健康檢查
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# 以 PID 1 運行以正確處理信號
ENTRYPOINT ["dumb-init", "--"]
CMD ["./app"]
```

**安全掃描整合**
```yaml
# .github/workflows/docker-security.yml
name: Docker 安全掃描

on:
  push:
    paths:
      - 'Dockerfile*'
      - '.dockerignore'

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: 運行 Trivy 漏洞掃描器
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ github.repository }}:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          
      - name: 上傳 Trivy 掃描結果
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
          
      - name: 運行 Hadolint
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile
          format: sarif
          output-file: hadolint-results.sarif
          
      - name: 上傳 Hadolint 掃描結果
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: hadolint-results.sarif
```

### 6. 運行時優化

優化容器運行時性能：

**運行時配置**
```dockerfile
# JVM 優化範例
FROM eclipse-temurin:17-jre-alpine

# 根據容器限制設定 JVM 記憶體
ENV JAVA_OPTS="-XX:MaxRAMPercentage=75.0 \
    -XX:InitialRAMPercentage=50.0 \
    -XX:+UseContainerSupport \
    -XX:+OptimizeStringConcat \
    -XX:+UseStringDeduplication \
    -Djava.security.egd=file:/dev/./urandom"

# Node.js 優化
FROM node:18-alpine
ENV NODE_ENV=production \
    NODE_OPTIONS="--max-old-space-size=1024 --optimize-for-size"

# Python 優化
FROM python:3.11-slim
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONOPTIMIZE=2

# Nginx 優化
FROM nginx:alpine
COPY nginx-optimized.conf /etc/nginx/nginx.conf
# 啟用 gzip、快取和連接池
```

### 7. Docker Compose 優化

優化多容器應用程式：

```yaml
# docker-compose.yml
version: '3.9'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      cache_from:
        - ${REGISTRY}/app:latest
        - ${REGISTRY}/app:builder
      args:
        BUILDKIT_INLINE_CACHE: 1
    image: ${REGISTRY}/app:${VERSION:-latest}
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    
  redis:
    image: redis:7-alpine
    command: redis-server --maxmemory 256mb --maxmemory-policy allkeys-lru
    deploy:
      resources:
        limits:
          memory: 256M
    volumes:
      - type: tmpfs
        target: /data
        tmpfs:
          size: 268435456 # 256MB
          
  nginx:
    image: nginx:alpine
    volumes:
      - type: bind
        source: ./nginx.conf
        target: /etc/nginx/nginx.conf
        read_only: true
    depends_on:
      app:
        condition: service_healthy
```

### 8. 建置自動化

自動化優化建置：

**建置自動化腳本**
```bash
#!/bin/bash
# build-optimize.sh

set -euo pipefail

# 變數
IMAGE_NAME="${1:-myapp}"
VERSION="${2:-latest}"
PLATFORMS="${3:-linux/amd64,linux/arm64}"

echo "🏗️ 正在建置優化 Docker 映像..."

# 啟用 BuildKit
export DOCKER_BUILDKIT=1

# 帶快取建置
docker buildx build \
  --platform "${PLATFORMS}" \
  --cache-from "type=registry,ref=${IMAGE_NAME}:buildcache" \
  --cache-to "type=registry,ref=${IMAGE_NAME}:buildcache,mode=max" \
  --tag "${IMAGE_NAME}:${VERSION}" \
  --build-arg BUILDKIT_INLINE_CACHE=1 \
  --progress=plain \
  --push \
  .

# 分析映像大小
echo "📊 映像分析："
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest "${IMAGE_NAME}:${VERSION}"

# 安全掃描
echo "🔒 安全掃描："
trivy image "${IMAGE_NAME}:${VERSION}"

# 大小報告
echo "📏 大小比較："
docker images "${IMAGE_NAME}" --format "table {{.Repository}}	{{.Tag}}	{{.Size}}"
```

### 9. 監控和指標

追蹤容器性能：

```python
# container-metrics.py
import docker
import json
from datetime import datetime

class ContainerMonitor:
    def __init__(self):
        self.client = docker.from_env()
        
    def collect_metrics(self, container_name):
        """收集容器性能指標"""
        container = self.client.containers.get(container_name)
        stats = container.stats(stream=False)
        
        metrics = {
            'timestamp': datetime.now().isoformat(),
            'container': container_name,
            'cpu': self._calculate_cpu_percent(stats),
            'memory': self._calculate_memory_usage(stats),
            'network': self._calculate_network_io(stats),
            'disk': self._calculate_disk_io(stats)
        }
        
        return metrics
    
    def _calculate_cpu_percent(self, stats):
        """計算 CPU 使用率百分比"""
        cpu_delta = stats['cpu_stats']['cpu_usage']['total_usage'] - \
                   stats['precpu_stats']['cpu_usage']['total_usage']
        system_delta = stats['cpu_stats']['system_cpu_usage'] - \
                      stats['precpu_stats']['system_cpu_usage']
        
        if system_delta > 0 and cpu_delta > 0:
            cpu_percent = (cpu_delta / system_delta) * \
                         len(stats['cpu_stats']['cpu_usage']['percpu_usage']) * 100.0
            return round(cpu_percent, 2)
        return 0.0
    
    def _calculate_memory_usage(self, stats):
        """計算記憶體使用量"""
        usage = stats['memory_stats']['usage']
        limit = stats['memory_stats']['limit']
        
        return {
            'usage_bytes': usage,
            'limit_bytes': limit,
            'percent': round((usage / limit) * 100, 2)
        }
```

### 10. 最佳實踐檢查清單

```python
def generate_dockerfile_checklist():
    """生成 Dockerfile 最佳實踐檢查清單"""
    checklist = """
## Dockerfile 最佳實踐檢查清單

### 基礎映像
- [ ] 使用特定版本標籤（而非 :latest）
- [ ] 使用最小基礎映像（alpine、slim、distroless）
- [ ] 保持基礎映像更新
- [ ] 盡可能使用官方映像

### 層和快取
- [ ] 命令順序從最不頻繁更改到最頻繁更改
- [ ] 適當時合併 RUN 命令
- [ ] 在同一層中清理（apt 快取、pip 快取）
- [ ] 使用 .dockerignore 排除不必要的檔案

### 安全
- [ ] 以非 root 使用者運行
- [ ] 不在映像中儲存秘密
- [ ] 掃描映像以查找漏洞
- [ ] 使用 COPY 而非 ADD
- [ ] 盡可能設定只讀根檔案系統

### 大小優化
- [ ] 使用多階段建置
- [ ] 移除不必要的依賴項
- [ ] 清除套件管理器快取
- [ ] 移除臨時檔案和建置工件
- [ ] 對於 apt 使用 --no-install-recommends

### 性能
- [ ] 設定適當的資源限制
- [ ] 使用健康檢查
- [ ] 優化啟動時間
- [ ] 適當配置日誌記錄
- [ ] 使用 BuildKit 加速建置

### 可維護性
- [ ] 包含 LABEL 元資料
- [ ] 使用 EXPOSE 記錄暴露的埠
- [ ] 使用 ARG 進行建置時變數
- [ ] 包含有意義的註釋
- [ ] 版本化您的 Dockerfile
"""
    return checklist
```

## 輸出格式

1. **分析報告**：當前 Dockerfile 問題和優化機會
2. **優化 Dockerfile**：重寫的 Dockerfile，包含所有優化
3. **大小比較**：映像大小前後分析
4. **建置性能**：建置時間改進和快取策略
5. **安全報告**：安全掃描結果和強化建議
6. **運行時配置**：應用程式的優化運行時設定
7. **監控設定**：容器指標和性能追蹤
8. **遷移指南**：實施優化的逐步指南

## 跨指令整合

### 完整的容器優先開發工作流程

**容器化開發管道**
```bash
# 1. 生成容器化 API 腳手架
/api-scaffold
framework: "fastapi"
deployment_target: "kubernetes"
containerization: true
monitoring: true

# 2. 優化容器以用於生產
/docker-optimize
optimization_level: "production"
security_hardening: true
multi_stage_build: true

# 3. 安全掃描容器映像
/security-scan
scan_types: ["container", "dockerfile", "runtime"]
image_name: "app:optimized"
generate_sbom: true

# 4. 為優化容器生成 K8s 清單
/k8s-manifest
container_security: "strict"
resource_optimization: true
horizontal_scaling: true
```

**整合容器配置**
```python
# container-config.py - 所有指令共享
class IntegratedContainerConfig:
    def __init__(self):
        self.api_config = self.load_api_config()           # 來自 /api-scaffold
        self.security_config = self.load_security_config() # 來自 /security-scan
        self.k8s_config = self.load_k8s_config()          # 來自 /k8s-manifest
        self.test_config = self.load_test_config()         # 來自 /test-harness
        
    def generate_optimized_dockerfile(self):
        """生成針對特定應用程式優化的 Dockerfile"""
        framework = self.api_config.get('framework', 'python')
        security_level = self.security_config.get('level', 'standard')
        deployment_target = self.k8s_config.get('platform', 'kubernetes')
        
        if framework == 'fastapi':
            return self.generate_fastapi_dockerfile(security_level, deployment_target)
        elif framework == 'express':
            return self.generate_express_dockerfile(security_level, deployment_target)
        elif framework == 'django':
            return self.generate_django_dockerfile(security_level, deployment_target)
            
    def generate_fastapi_dockerfile(self, security_level, deployment_target):
        """生成優化的 FastAPI Dockerfile"""
        dockerfile_content = {
            'base_image': self.select_base_image('python', security_level),
            'build_stages': self.configure_build_stages(),
            'security_configs': self.apply_security_configurations(security_level),
            'runtime_optimizations': self.configure_runtime_optimizations(),
            'monitoring_setup': self.configure_monitoring_setup(),
            'health_checks': self.configure_health_checks()
        }
        return dockerfile_content
    
    def select_base_image(self, language, security_level):
        """根據安全和大小要求選擇最佳基礎映像"""
        base_images = {
            'python': {
                'minimal': 'python:3.11-alpine',
                'standard': 'python:3.11-slim-bookworm',
                'secure': 'chainguard/python:latest-dev',
                'distroless': 'gcr.io/distroless/python3-debian12'
            }
        }
        
        if security_level == 'strict':
            return base_images[language]['distroless']
        elif security_level == 'enhanced':
            return base_images[language]['secure']
        else:
            return base_images[language]['standard']
    
    def configure_build_stages(self):
        """配置多階段建置優化"""
        return {
            'dependencies_stage': {
                'name': 'dependencies',
                'base': 'python:3.11-slim-bookworm',
                'actions': [
                    'COPY requirements.txt .',
                    'RUN pip install --no-cache-dir --user -r requirements.txt'
                ]
            },
            'security_stage': {
                'name': 'security-scan',
                'base': 'dependencies',
                'actions': [
                    'RUN pip-audit --format=json --output=/tmp/security-report.json',
                    'RUN safety check --json --output=/tmp/safety-report.json'
                ]
            },
            'runtime_stage': {
                'name': 'runtime',
                'base': 'python:3.11-slim-bookworm',
                'actions': [
                    'COPY --from=dependencies /root/.local /root/.local',
                    'COPY --from=security-scan /tmp/*-report.json /security-reports/'
                ]
            }
        }
```

**API 容器整合**
```dockerfile
# Dockerfile.api - 從 /api-scaffold + /docker-optimize 生成
# 為 FastAPI 應用程式優化的多階段建置
FROM python:3.11-slim-bookworm AS base

# 設定環境變數以進行優化
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# 階段 1：依賴項
FROM base AS dependencies
WORKDIR /app

# 安裝用於建置 Python 套件的系統依賴項
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# 複製並安裝 Python 依賴項
COPY requirements.txt ./
RUN pip install --user --no-warn-script-location -r requirements.txt

# 階段 2：安全掃描
FROM dependencies AS security-scan
RUN pip install --user pip-audit safety bandit

# 複製原始碼以進行安全掃描
COPY . .

# 在建置期間運行安全掃描
RUN python -m bandit -r . -f json -o /tmp/bandit-report.json || true
RUN python -m safety check --json --output /tmp/safety-report.json || true
RUN python -m pip_audit --format=json --output=/tmp/pip-audit-report.json || true

# 階段 3：測試（可選，可在生產建置中跳過）
FROM security-scan AS testing
RUN pip install --user pytest pytest-cov

# 在建置期間運行測試（來自 /test-harness 整合）
RUN python -m pytest tests/ --cov=src --cov-report=json --cov-report=term

# 階段 4：生產運行時
FROM base AS runtime

# 建立非 root 使用者以提高安全性
RUN groupadd -r appuser && useradd -r -g appuser appuser

# 設定應用程式目錄
WORKDIR /app
RUN chown appuser:appuser /app

# 複製 Python 套件從依賴項階段
COPY --from=dependencies --chown=appuser:appuser /root/.local /home/appuser/.local

# 複製安全報告從安全掃描階段
COPY --from=security-scan /tmp/*-report.json /app/security-reports/

# 複製應用程式程式碼
COPY --chown=appuser:appuser . .

# 更新 PATH 以包含使用者套件
ENV PATH=/home/appuser/.local/bin:$PATH

# 切換到非 root 使用者
USER appuser

# 配置健康檢查（與 K8s 健康檢查整合）
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD python -c "import requests; requests.get('http://localhost:8000/health', timeout=5)"

# 暴露埠（從 API 腳手架配置）
EXPOSE 8000

# 設定最佳啟動命令
CMD ["python", "-m", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

**資料庫容器整合**
```dockerfile
# Dockerfile.db - 為來自 /db-migrate 的資料庫遷移生成
FROM postgres:15-alpine AS base

# 安裝遷移工具
RUN apk add --no-cache python3 py3-pip
RUN pip3 install alembic psycopg2-binary

# 建立遷移使用者
RUN addgroup -g 1001 migration && adduser -D -u 1001 -G migration migration

# 階段 1：遷移準備
FROM base AS migration-prep
WORKDIR /migrations

# 從 /db-migrate 輸出複製遷移腳本
COPY --chown=migration:migration migrations/ ./
COPY --chown=migration:migration alembic.ini ./

# 驗證遷移腳本
USER migration
RUN alembic check || echo "遷移驗證完成"

# 階段 2：生產資料庫
FROM postgres:15-alpine AS production

# 複製已驗證的遷移
COPY --from=migration-prep --chown=postgres:postgres /migrations /docker-entrypoint-initdb.d/

# 配置 PostgreSQL 以用於生產
RUN echo "shared_preload_libraries = 'pg_stat_statements'" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "track_activity_query_size = 2048" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "log_min_duration_statement = 1000" >> /usr/local/share/postgresql/postgresql.conf.sample

# 來自 /security-scan 的安全配置
RUN echo "ssl = on" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "log_connections = on" >> /usr/local/share/postgresql/postgresql.conf.sample
RUN echo "log_disconnections = on" >> /usr/local/share/postgresql/postgresql.conf.sample

EXPOSE 5432
```

**前端容器整合**
```dockerfile
# Dockerfile.frontend - 從 /frontend-optimize + /docker-optimize 生成
# 為 React/Vue 應用程式優化的多階段建置
FROM node:18-alpine AS base

# 設定環境變數
ENV NODE_ENV=production \
    NPM_CONFIG_CACHE=/tmp/.npm

# 階段 1：依賴項
FROM base AS dependencies
WORKDIR /app

# 複製套件檔案
COPY package*.json ./

# 安裝依賴項並進行優化
RUN npm ci --only=production --silent

# 階段 2：建置應用程式
FROM base AS build
WORKDIR /app

# 複製依賴項
COPY --from=dependencies /app/node_modules ./node_modules

# 複製原始碼
COPY . .

# 使用 /frontend-optimize 的優化建置應用程式
RUN npm run build

# 運行安全審計
RUN npm audit --audit-level high --production

# 階段 3：安全掃描
FROM build AS security-scan

# 安裝安全掃描工具
RUN npm install -g retire snyk

# 運行安全掃描
RUN retire --outputformat json --outputpath /tmp/retire-report.json || true
RUN snyk test --json > /tmp/snyk-report.json || true

# 階段 4：生產伺服器
FROM nginx:alpine AS production

# 安裝安全更新
RUN apk update && apk upgrade && apk add --no-cache dumb-init

# 建立非 root 使用者
RUN addgroup -g 1001 www && adduser -D -u 1001 -G www www

# 複製已建置的應用程式
COPY --from=build --chown=www:www /app/dist /usr/share/nginx/html

# 複製安全報告
COPY --from=security-scan /tmp/*-report.json /var/log/security/

# 複製優化的 nginx 配置
COPY nginx.conf /etc/nginx/nginx.conf

# 配置正確的檔案權限
RUN chown -R www:www /usr/share/nginx/html
RUN chmod -R 755 /usr/share/nginx/html

# 使用非 root 使用者
USER www

# 前端健康檢查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:80/ || exit 1

EXPOSE 80

# 使用 dumb-init 以正確處理信號
ENTRYPOINT ["dumb-init", "--"]
CMD ["nginx", "-g", "daemon off;"]
```

**Kubernetes 容器整合**
```yaml
# k8s-optimized-deployment.yaml - 從 /k8s-manifest + /docker-optimize 生成
apiVersion: v1
kind: ConfigMap
metadata:
  name: container-config
  namespace: production
data:
  optimization-level: "production"
  security-level: "strict"
  monitoring-enabled: "true"

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: optimized-api
  namespace: production
  labels:
    app: api
    optimization: enabled
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
      annotations:
        # 容器優化註釋
        container.seccomp.security.alpha.kubernetes.io/defaultProfileName: runtime/default
        container.apparmor.security.beta.kubernetes.io/api: runtime/default
    spec:
      # 優化 Pod 配置
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        runAsGroup: 1001
        fsGroup: 1001
        seccompProfile:
          type: RuntimeDefault
      
      # 來自容器分析的資源優化
      containers:
      - name: api
        image: registry.company.com/api:optimized-latest
        imagePullPolicy: Always
        
        # 優化資源分配
        resources:
          requests:
            memory: "128Mi"     # 根據實際使用情況優化
            cpu: "100m"         # 根據負載測試優化
            ephemeral-storage: "1Gi"
          limits:
            memory: "512Mi"     # 防止 OOM，允許突發
            cpu: "500m"         # 允許處理峰值
            ephemeral-storage: "2Gi"
        
        # 容器安全優化
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
          capabilities:
            drop:
              - ALL
            add:
              - NET_BIND_SERVICE
        
        # 優化啟動和健康檢查
        ports:
        - containerPort: 8000
          protocol: TCP
          
        # 快速啟動探針
        startupProbe:
          httpGet:
            path: /startup
            port: 8000
          failureThreshold: 30
          periodSeconds: 1
          
        # 優化健康檢查
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 2

```

**CI/CD 容器整合**
```yaml
# .github/workflows/container-pipeline.yml
name: Optimized Container Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-optimize:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      security-events: write
    
    strategy:
      matrix:
        service: [api, frontend, database]
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
    
    # 1. Build multi-stage container
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      with:
        driver-opts: network=host
    
    - name: Log in to Container Registry
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    # 2. Build optimized images
    - name: Build and push container images
      uses: docker/build-push-action@v5
      with:
        context: .
        file: Dockerfile.${{ matrix.service }}
        push: true
        tags: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:latest
        cache-from: type=gha
        cache-to: type=gha,mode=max
        platforms: linux/amd64,linux/arm64
    
    # 3. Container security scanning
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
        format: 'sarif'
        output: 'trivy-results-${{ matrix.service }}.sarif'
    
    # 4. Container optimization analysis
    - name: Analyze container optimization
      run: |
        docker images --format "table {{.Repository}}	{{.Tag}}	{{.Size}}" | \
        grep ${{ matrix.service }} > container-analysis-${{ matrix.service }}.txt
        
        # Compare with baseline
        if [ -f baseline-sizes.txt ]; then
          echo "Size comparison for ${{ matrix.service }}:" >> size-comparison.txt
          echo "Previous: $(grep ${{ matrix.service }} baseline-sizes.txt || echo 'N/A')" >> size-comparison.txt
          echo "Current: $(grep ${{ matrix.service }} container-analysis-${{ matrix.service }}.txt)" >> size-comparison.txt
        fi
    
    # 5. Performance testing
    - name: Container performance testing
      run: |
        # Start container for performance testing
        docker run -d --name test-${{ matrix.service }} \
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
        
        # Wait for startup
        sleep 30
        
        # Run basic performance tests
        if [ "${{ matrix.service }}" = "api" ]; then
          docker exec test-${{ matrix.service }} \
            python -c "import requests; print(requests.get('http://localhost:8000/health').status_code)"
        fi
        
        # Cleanup
        docker stop test-${{ matrix.service }}
        docker rm test-${{ matrix.service }}
    
    # 6. Upload security results
    - name: Upload Trivy scan results to GitHub Security tab
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results-${{ matrix.service }}.sarif'
    
    # 7. Generate optimization report
    - name: Generate optimization report
      run: |
        cat > optimization-report-${{ matrix.service }}.md << EOF
        # Container Optimization Report - ${{ matrix.service }}
        
        ## Build Information
        - **Image**: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/${{ matrix.service }}:${{ github.sha }}
        - **Build Date**: $(date)
        - **Platforms**: linux/amd64,linux/arm64
        
        ## Size Analysis
        $(cat container-analysis-${{ matrix.service }}.txt)
        
        ## Security Scan
        - **Scanner**: Trivy
        - **Results**: See Security tab for detailed findings
        
        ## Optimizations Applied
        - Multi-stage build for minimal image size
        - Security hardening with non-root user
        - Layer caching for faster builds
        - Health checks for reliability
        EOF
    
    - name: Upload optimization report
      uses: actions/upload-artifact@v3
      with:
        name: optimization-report-${{ matrix.service }}
        path: optimization-report-${{ matrix.service }}.md

  deploy-to-staging:
    needs: build-and-optimize
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    
    steps:
    - name: Deploy to staging
      run: |
        # Update K8s manifests with new image tags
        # Apply optimized K8s configurations
        kubectl set image deployment/optimized-api \
          api=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}/api:${{ github.sha }} \
          --namespace=staging
```

**Monitoring Integration**
```python
# container_monitoring.py - Integrated container monitoring
import docker
import psutil
from prometheus_client import CollectorRegistry, Gauge, Counter, Histogram
from typing import Dict, Any

class ContainerOptimizationMonitor:
    """Monitor container performance and optimization metrics"""
    
    def __init__(self):
        self.docker_client = docker.from_env()
        self.registry = CollectorRegistry()
        
        # Metrics from container optimization
        self.container_size_gauge = Gauge(
            'container_image_size_bytes', 
            'Container image size in bytes',
            ['service', 'optimization_level'],
            registry=self.registry
        )
        
        self.container_startup_time = Histogram(
            'container_startup_seconds',
            'Container startup time in seconds',
            ['service'],
            registry=self.registry
        )
        
        self.resource_usage_gauge = Gauge(
            'container_resource_usage_ratio',
            'Container resource usage ratio (used/limit)',
            ['service', 'resource_type'],
            registry=self.registry
        )
    
    def monitor_optimization_metrics(self):
        """Monitor container optimization effectiveness"""
        containers = self.docker_client.containers.list()
        
        optimization_metrics = {}
        
        for container in containers:
            service_name = container.labels.get('app', 'unknown')
            
            # Monitor image size efficiency
            image = container.image
            size_mb = self.get_image_size(image.id) / (1024 * 1024)
            
            # Monitor resource efficiency
            stats = container.stats(stream=False)
            memory_usage = self.calculate_memory_efficiency(stats)
            cpu_usage = self.calculate_cpu_efficiency(stats)
            
            # Monitor startup performance
            startup_time = self.get_container_startup_time(container)
            
            optimization_metrics[service_name] = {
                'image_size_mb': size_mb,
                'memory_efficiency': memory_usage,
                'cpu_efficiency': cpu_usage,
                'startup_time_seconds': startup_time,
                'optimization_score': self.calculate_optimization_score(
                    size_mb, memory_usage, cpu_usage, startup_time
                )
            }
            
            # Update Prometheus metrics
            self.container_size_gauge.labels(
                service=service_name,
                optimization_level='production'
            ).set(size_mb)
            
            self.container_startup_time.labels(
                service=service_name
            ).observe(startup_time)
        
        return optimization_metrics
    
    def calculate_optimization_score(self, size_mb, memory_eff, cpu_eff, startup_time):
        """Calculate overall optimization score (0-100)"""
        size_score = max(0, 100 - (size_mb / 10))  # Penalty for large images
        memory_score = (1 - memory_eff) * 100      # Reward for efficient memory use
        cpu_score = (1 - cpu_eff) * 100           # Reward for efficient CPU use
        startup_score = max(0, 100 - startup_time * 10)  # Penalty for slow startup
        
        return (size_score + memory_score + cpu_score + startup_score) / 4
```

This comprehensive integration ensures containers are optimized across the entire development lifecycle, from build-time optimization through runtime monitoring and Kubernetes deployment.
```