# Kubernetes 清單生成

您是 Kubernetes 專家，專精於建立生產就緒的清單、Helm Chart 和雲原生部署配置。遵循最佳實踐和 GitOps 原則，生成安全、可擴展和可維護的 Kubernetes 資源。

## 背景
使用者需要建立或優化 Kubernetes 清單以部署應用程式。專注於生產就緒、安全強化、資源優化、可觀察性和多環境配置。

## 要求
$ARGUMENTS

## 指示

### 1. 應用程式分析

分析應用程式以確定 Kubernetes 要求：

**框架特定分析**
```python
import yaml
import json
from pathlib import Path
from typing import Dict, List, Any

class AdvancedK8sAnalyzer:
    def __init__(self):
        self.framework_patterns = {
            'react': {
                'files': ['package.json', 'src/App.js', 'src/index.js'],
                'build_tool': ['vite', 'webpack', 'create-react-app'],
                'deployment_type': 'static',
                'port': 3000,
                'health_check': '/health',
                'resources': {'cpu': '100m', 'memory': '256Mi'}
            },
            'nextjs': {
                'files': ['next.config.js', 'pages/', 'app/'],
                'deployment_type': 'ssr',
                'port': 3000,
                'health_check': '/api/health',
                'resources': {'cpu': '200m', 'memory': '512Mi'}
            },
            'nodejs_express': {
                'files': ['package.json', 'server.js', 'app.js'],
                'deployment_type': 'api',
                'port': 8080,
                'health_check': '/health',
                'resources': {'cpu': '200m', 'memory': '512Mi'}
            },
            'python_fastapi': {
                'files': ['main.py', 'requirements.txt', 'pyproject.toml'],
                'deployment_type': 'api',
                'port': 8000,
                'health_check': '/health',
                'resources': {'cpu': '250m', 'memory': '512Mi'}
            },
            'python_django': {
                'files': ['manage.py', 'settings.py', 'wsgi.py'],
                'deployment_type': 'web',
                'port': 8000,
                'health_check': '/health/',
                'resources': {'cpu': '300m', 'memory': '1Gi'}
            },
            'go': {
                'files': ['main.go', 'go.mod', 'go.sum'],
                'deployment_type': 'api',
                'port': 8080,
                'health_check': '/health',
                'resources': {'cpu': '100m', 'memory': '128Mi'}
            },
            'java_spring': {
                'files': ['pom.xml', 'build.gradle', 'src/main/java'],
                'deployment_type': 'api',
                'port': 8080,
                'health_check': '/actuator/health',
                'resources': {'cpu': '500m', 'memory': '1Gi'}
            },
            'dotnet': {
                'files': ['*.csproj', 'Program.cs', 'Startup.cs'],
                'deployment_type': 'api',
                'port': 5000,
                'health_check': '/health',
                'resources': {'cpu': '300m', 'memory': '512Mi'}
            }
        }
    
    def analyze_application(self, app_path: str) -> Dict[str, Any]:
        """
        使用框架檢測進行進階應用程式分析
        """
        framework = self._detect_framework(app_path)
        analysis = {
            'framework': framework,
            'app_type': self._detect_app_type(app_path),
            'services': self._identify_services(app_path),
            'dependencies': self._find_dependencies(app_path),
            'storage_needs': self._analyze_storage(app_path),
            'networking': self._analyze_networking(app_path),
            'resource_requirements': self._estimate_resources(app_path, framework),
            'security_requirements': self._analyze_security_needs(app_path),
            'observability_needs': self._analyze_observability(app_path),
            'scaling_strategy': self._recommend_scaling(app_path, framework)
        }
        
        return analysis
    
    def _detect_framework(self, app_path: str) -> str:
        """檢測應用程式框架以進行優化部署"""
        app_path = Path(app_path)
        
        for framework, config in self.framework_patterns.items():
            if all((app_path / f).exists() for f in config['files'][:1]):
                if any((app_path / f).exists() for f in config['files']):
                    return framework
        
        return 'generic'
    
    def generate_framework_optimized_manifests(self, analysis: Dict[str, Any]) -> Dict[str, str]:
        """生成針對特定框架優化的清單"""
        framework = analysis['framework']
        if framework in self.framework_patterns:
            return self._generate_specialized_manifests(framework, analysis)
        return self._generate_generic_manifests(analysis)
    
    def _detect_app_type(self, app_path):
        """檢測應用程式類型和堆疊"""
        indicators = {
            'web': ['nginx.conf', 'httpd.conf', 'index.html'],
            'api': ['app.py', 'server.js', 'main.go'],
            'database': ['postgresql.conf', 'my.cnf', 'mongod.conf'],
            'worker': ['worker.py', 'consumer.js', 'processor.go'],
            'frontend': ['package.json', 'webpack.config.js', 'angular.json']
        }
        
        detected_types = []
        for app_type, files in indicators.items():
            if any((Path(app_path) / f).exists() for f in files):
                detected_types.append(app_type)
                
        return detected_types
    
    def _identify_services(self, app_path):
        """識別微服務結構"""
        services = []
        
        # 檢查 docker-compose.yml
        compose_file = Path(app_path) / 'docker-compose.yml'
        if compose_file.exists():
            with open(compose_file) as f:
                compose = yaml.safe_load(f)
                for service_name, config in compose.get('services', {}).items():
                    services.append({
                        'name': service_name,
                        'image': config.get('image', 'custom'),
                        'ports': config.get('ports', []),
                        'environment': config.get('environment', {}),
                        'volumes': config.get('volumes', [])
                    })
        
        return services
```

### 2. 部署清單生成

建立生產就緒的部署清單：

**部署模板**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
  labels:
    app: ${APP_NAME}
    version: ${VERSION}
    component: ${COMPONENT}
    managed-by: kubectl
spec:
  replicas: ${REPLICAS}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: ${APP_NAME}
      component: ${COMPONENT}
  template:
    metadata:
      labels:
        app: ${APP_NAME}
        version: ${VERSION}
        component: ${COMPONENT}
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "${METRICS_PORT}"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: ${APP_NAME}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: ${APP_NAME}
        image: ${IMAGE}:${TAG}
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: ${PORT}
          protocol: TCP
        - name: metrics
          containerPort: ${METRICS_PORT}
          protocol: TCP
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        envFrom:
        - configMapRef:
            name: ${APP_NAME}-config
        - secretRef:
            name: ${APP_NAME}-secrets
        resources:
          requests:
            memory: "${MEMORY_REQUEST}"
            cpu: "${CPU_REQUEST}"
          limits:
            memory: "${MEMORY_LIMIT}"
            cpu: "${CPU_LIMIT}"
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        startupProbe:
          httpGet:
            path: /startup
            port: http
          initialDelaySeconds: 0
          periodSeconds: 10
          timeoutSeconds: 3
          failureThreshold: 30
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
          capabilities:
            drop:
            - ALL
        volumeMounts:
        - name: tmp
          mountPath: /tmp
        - name: cache
          mountPath: /app/cache
      volumes:
      - name: tmp
        emptyDir: {}
      - name: cache
        emptyDir: {}
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - ${APP_NAME}
              topologyKey: kubernetes.io/hostname
      topologySpreadConstraints:
      - maxSkew: 1
        topologyKey: topology.kubernetes.io/zone
        whenUnsatisfiable: DoNotSchedule
        labelSelector:
          matchLabels:
            app: ${APP_NAME}
```

### 3. 服務和網路

生成服務和網路資源：

**服務配置**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
  labels:
    app: ${APP_NAME}
    component: ${COMPONENT}
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: ClusterIP
  selector:
    app: ${APP_NAME}
    component: ${COMPONENT}
  ports:
  - name: http
    port: 80
    targetPort: http
    protocol: TCP
  - name: grpc
    port: 9090
    targetPort: grpc
    protocol: TCP
---
apiVersion: v1
kind: Service
metadata:
  name: ${APP_NAME}-headless
  namespace: ${NAMESPACE}
  labels:
    app: ${APP_NAME}
spec:
  type: ClusterIP
  clusterIP: None
  selector:
    app: ${APP_NAME}
  ports:
  - name: http
    port: 80
    targetPort: http
```

**Ingress 配置**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - ${DOMAIN}
    secretName: ${APP_NAME}-tls
  rules:
  - host: ${DOMAIN}
    http:
      paths:
      - path: / 
        pathType: Prefix
        backend:
          service:
            name: ${APP_NAME}
            port:
              name: http
```

### 4. 配置管理

建立 ConfigMaps 和 Secrets：

**ConfigMap 生成器**
```python
def generate_configmap(app_name, config_data):
    """
    生成 ConfigMap 清單
    """
    configmap = {
        'apiVersion': 'v1',
        'kind': 'ConfigMap',
        'metadata': {
            'name': f'{app_name}-config',
            'namespace': 'default',
            'labels': {
                'app': app_name
            }
        },
        'data': {}
    }
    
    # 處理不同的配置格式
    for key, value in config_data.items():
        if isinstance(value, dict):
            # 巢狀配置為 YAML
            configmap['data'][key] = yaml.dump(value)
        elif isinstance(value, list):
            # 列表為 JSON
            configmap['data'][key] = json.dumps(value)
        else:
            # 純字串
            configmap['data'][key] = str(value)
    
    return yaml.dump(configmap)

def generate_secret(app_name, secret_data):
    """
    生成 Secret 清單
    """
    import base64
    
    secret = {
        'apiVersion': 'v1',
        'kind': 'Secret',
        'metadata': {
            'name': f'{app_name}-secrets',
            'namespace': 'default',
            'labels': {
                'app': app_name
            }
        },
        'type': 'Opaque',
        'data': {}
    }
    
    # Base64 編碼所有值
    for key, value in secret_data.items():
        encoded = base64.b64encode(value.encode()).decode()
        secret['data'][key] = encoded
    
    return yaml.dump(secret)
```

### 5. 持久儲存

配置持久卷：

**帶有儲存的 StatefulSet**
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
spec:
  serviceName: ${APP_NAME}-headless
  replicas: ${REPLICAS}
  selector:
    matchLabels:
      app: ${APP_NAME}
  template:
    metadata:
      labels:
        app: ${APP_NAME}
    spec:
      containers:
      - name: ${APP_NAME}
        image: ${IMAGE}:${TAG}
        ports:
        - containerPort: ${PORT}
          name: http
        volumeMounts:
        - name: data
          mountPath: /data
        - name: config
          mountPath: /etc/config
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: ${APP_NAME}-config
  volumeClaimTemplates:
  - metadata:
      name: data
      labels:
        app: ${APP_NAME}
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: ${STORAGE_CLASS}
      resources:
        requests:
          storage: ${STORAGE_SIZE}
```

### 6. Helm Chart 生成

建立生產 Helm Chart：

**Chart 結構**
```bash
#!/bin/bash
# generate-helm-chart.sh

create_helm_chart() {
    local chart_name="$1"
    
    mkdir -p "$chart_name"/{{templates,charts}}
    
    # Chart.yaml
    cat > "$chart_name"/Chart.yaml << EOF
apiVersion: v2
name: $chart_name
description: A Helm chart for $chart_name
type: application
version: 0.1.0
appVersion: "1.0.0"
keywords:
  - $chart_name
home: https://github.com/org/$chart_name
sources:
  - https://github.com/org/$chart_name
maintainers:
  - name: Team Name
    email: team@example.com
dependencies: []
EOF

    # values.yaml
    cat > "$chart_name"/values.yaml << 'EOF'
# 應用程式的預設值
replicaCount: 2

image:
  repository: myapp
  pullPolicy: IfNotPresent
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}
podSecurityContext:
  fsGroup: 2000
  runAsNonRoot: true
  runAsUser: 1000

securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
    - ALL
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 1000

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: false
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: chart-example.local
      paths:
        - path: / 
          pathType: ImplementationSpecific
  tls: []

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
  targetMemoryUtilizationPercentage: 80

persistence:
  enabled: false
  storageClass: ""
  accessMode: ReadWriteOnce
  size: 8Gi

nodeSelector: {}
tolerations: []
affinity: {}

# 應用程式配置
config:
  logLevel: info
  debug: false

# 秘密 - 在生產環境中使用外部秘密
secrets: {}

# 健康檢查路徑
healthcheck:
  liveness:
    path: /health
    initialDelaySeconds: 30
    periodSeconds: 10
  readiness:
    path: /ready
    initialDelaySeconds: 5
    periodSeconds: 5
EOF

    # _helpers.tpl
    cat > "$chart_name"/templates/_helpers.tpl << 'EOF'
{{/*
展開 Chart 的名稱。
*/}}
{{- define "app.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
建立預設的完全限定應用程式名稱。
*/}}
{{- define "app.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}

{{/*
建立 Chart 名稱和版本，如 Chart 標籤所用。
*/}}
{{- define "app.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
通用標籤
*/}}
{{- define "app.labels" -}}
helm.sh/chart: {{ include "app.chart" . }}
{{ include "app.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
選擇器標籤
*/}}
{{- define "app.selectorLabels" -}}
app.kubernetes.io/name: {{ include "app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
建立要使用的服務帳戶名稱
*/}}
{{- define "app.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "app.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
EOF
}
```

### 7. 進階多環境配置

使用 GitOps 處理環境特定配置：

**FluxCD GitOps 設定**
```yaml
# infrastructure/flux-system/gotk-sync.yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 1m0s
  ref:
    branch: main
  url: https://github.com/org/k8s-gitops
---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: flux-system
  namespace: flux-system
spec:
  interval: 10m0s
  path: "./clusters/production"
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  validation: client
  healthChecks:
    - apiVersion: apps/v1
      kind: Deployment
      name: myapp
      namespace: production

---
# 帶有 Helm 的進階 Kustomization
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - namespace.yaml
  - ../../base
  - monitoring/
  - security/

namespace: production

helmCharts:
  - name: prometheus
    repo: https://prometheus-community.github.io/helm-charts
    version: 15.0.0
    releaseName: prometheus
    namespace: monitoring
    valuesInline:
      server:
        persistentVolume:
          size: 50Gi
      alertmanager:
        enabled: true

patchesStrategicMerge:
  - deployment-patch.yaml
  - service-patch.yaml

patchesJson6902:
  - target:
      version: v1
      kind: Deployment
      name: myapp
    patch: |
      - op: replace
        path: /spec/replicas
        value: 10
      - op: add
        path: /spec/template/spec/containers/0/resources/limits/nvidia.com~1gpu
        value: "1"

configMapGenerator:
  - name: app-config
    behavior: merge
    literals:
      - ENV=production
      - LOG_LEVEL=warn
      - DATABASE_POOL_SIZE=20
      - CACHE_TTL=3600
    files:
      - config/production.yaml

secretGenerator:
  - name: app-secrets
    behavior: replace
    type: Opaque
    options:
      disableNameSuffixHash: true
    files:
      - .env.production

replicas:
  - name: myapp
    count: 10

images:
  - name: myapp
    newTag: v1.2.3

commonLabels:
  app: myapp
  env: production
  version: v1.2.3

commonAnnotations:
  deployment.kubernetes.io/revision: "1"
  prometheus.io/scrape: "true"

resources:
  - hpa.yaml
  - vpa.yaml
  - pdb.yaml
  - networkpolicy.yaml
  - servicemonitor.yaml
  - backup.yaml
```

### 8. 進階安全清單

建立全面的安全導向資源：

**Pod 安全標準 (PSS)**
```yaml
# 帶有 Pod 安全標準的命名空間
apiVersion: v1
kind: Namespace
metadata:
  name: ${NAMESPACE}
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
    name: ${NAMESPACE}
---
# 進階網路策略
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ${APP_NAME}-netpol
  namespace: ${NAMESPACE}
spec:
  podSelector:
    matchLabels:
      app: ${APP_NAME}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
      podSelector: {}
    - namespaceSelector:
        matchLabels:
          name: monitoring
      podSelector:
        matchLabels:
          app: prometheus
    ports:
    - protocol: TCP
      port: 8080
    - protocol: TCP
      port: 9090  # 指標
  egress:
  # 資料庫存取
  - to:
    - namespaceSelector:
        matchLabels:
          name: database
    ports:
    - protocol: TCP
      port: 5432
    - protocol: TCP
      port: 6379  # Redis
  # 外部 API 存取
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 80
  # DNS 解析
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
---
# Open Policy Agent Gatekeeper 限制
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: requiredlabels
spec:
  crd:
    spec:
      names:
        kind: RequiredLabels
      validation:
        properties:
          labels:
            type: array
            items:
              type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package requiredlabels
        
        violation[{"msg": msg}] {
          required := input.parameters.labels
          provided := input.review.object.metadata.labels
          missing := required[_]
          not provided[missing]
          msg := sprintf("缺少必要標籤: %v", [missing])
        }
---
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: RequiredLabels
metadata:
  name: must-have-app-label
spec:
  match:
    kinds:
      - apiGroups: ["apps"]
        kinds: ["Deployment"]
  parameters:
    labels: ["app", "version", "component"]
---
# Falco 安全規則
apiVersion: v1
kind: ConfigMap
metadata:
  name: falco-rules
  namespace: falco
data:
  application_rules.yaml: |
    - rule: 可疑網路活動
      desc: 偵測可疑網路連線
      condition: >
        (spawned_process and container and
         ((proc.name in (nc, ncat, netcat, netcat.traditional) and
           proc.args contains "-l") or
          (proc.name = socat and proc.args contains "TCP-LISTEN")))
      output: >
        在容器中啟動可疑網路工具
        (使用者=%user.name 命令=%proc.cmdline 映像=%container.image.repository)
      priority: WARNING
      tags: [network, mitre_lateral_movement]
    
    - rule: 意外的出站連線
      desc: 建立了意外的出站連線
      condition: >
        outbound and not proc.name in (known_outbound_processes) and
        not fd.sip in (allowed_external_ips)
      output: >
        意外的出站連線
        (命令=%proc.cmdline 連線=%fd.name 使用者=%user.name)
      priority: WARNING
      tags: [network, mitre_exfiltration]
---
# 服務網格安全 (Istio)
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: ${APP_NAME}-authz
  namespace: ${NAMESPACE}
spec:
  selector:
    matchLabels:
      app: ${APP_NAME}
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/frontend/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
  - from:
    - source:
        principals: ["cluster.local/ns/monitoring/sa/prometheus"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/metrics"]
---
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: ${APP_NAME}-peer-authn
  namespace: ${NAMESPACE}
spec:
  selector:
    matchLabels:
      app: ${APP_NAME}
  mtls:
    mode: STRICT
```

### 9. 進階可觀察性設定

配置全面的監控、日誌記錄和追蹤：

**OpenTelemetry 整合**
```yaml
# OpenTelemetry 收集器
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: otel-collector
  namespace: ${NAMESPACE}
spec:
  mode: daemonset
  serviceAccount: otel-collector
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
            endpoint: 0.0.0.0:4317
          http:
            endpoint: 0.0.0.0:4318
      prometheus:
        config:
          scrape_configs:
            - job_name: 'kubernetes-pods'
              kubernetes_sd_configs:
                - role: pod
      k8s_cluster:
        auth_type: serviceAccount
      kubeletstats:
        collection_interval: 20s
        auth_type: "serviceAccount"
        endpoint: "${env:K8S_NODE_NAME}:10250"
        insecure_skip_verify: true
    
    processors:
      batch:
        timeout: 1s
        send_batch_size: 1024
      memory_limiter:
        limit_mib: 512
      k8sattributes:
        auth_type: "serviceAccount"
        passthrough: false
        filter:
          node_from_env_var: KUBE_NODE_NAME
        extract:
          metadata:
            - k8s.pod.name
            - k8s.pod.uid
            - k8s.deployment.name
            - k8s.namespace.name
            - k8s.node.name
            - k8s.pod.start_time
    
    exporters:
      prometheus:
        endpoint: "0.0.0.0:8889"
      jaeger:
        endpoint: jaeger-collector:14250
        tls:
          insecure: true
      loki:
        endpoint: http://loki:3100/loki/api/v1/push
    
    service:
      pipelines:
        traces:
          receivers: [otlp]
          processors: [memory_limiter, k8sattributes, batch]
          exporters: [jaeger]
        metrics:
          receivers: [otlp, prometheus, k8s_cluster, kubeletstats]
          processors: [memory_limiter, k8sattributes, batch]
          exporters: [prometheus]
        logs:
          receivers: [otlp]
          processors: [memory_limiter, k8sattributes, batch]
          exporters: [loki]
---
# 增強的 ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
  labels:
    app: ${APP_NAME}
    prometheus: kube-prometheus
spec:
  selector:
    matchLabels:
      app: ${APP_NAME}
  endpoints:
  - port: metrics
    interval: 15s
    path: /metrics
    honorLabels: true
    metricRelabelings:
    - sourceLabels: [__name__]
      regex: 'go_.*'
      action: drop
    - sourceLabels: [__name__]
      regex: 'promhttp_.*'
      action: drop
    relabelings:
    - sourceLabels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
      action: keep
      regex: true
    - sourceLabels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
      action: replace
      targetLabel: __metrics_path__
      regex: (.*)
    - sourceLabels: [__meta_kubernetes_pod_ip]
      action: replace
      targetLabel: __address__
      regex: (.*)
      replacement: $1:9090
---
# 自定義 Prometheus 規則
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: ${APP_NAME}-rules
  namespace: ${NAMESPACE}
spec:
  groups:
  - name: ${APP_NAME}.rules
    rules:
    - alert: HighErrorRate
      expr: |
        (
          rate(http_requests_total{job="${APP_NAME}",status=~"5.."}[5m])
          /
          rate(http_requests_total{job="${APP_NAME}"}[5m])
        ) > 0.05
      for: 5m
      labels:
        severity: warning
        service: ${APP_NAME}
      annotations:
        summary: "檢測到高錯誤率"
        description: "錯誤率為 {{ $value | humanizePercentage }}，適用於 {{ $labels.job }}"
    
    - alert: HighResponseTime
      expr: |
        histogram_quantile(0.95,
          rate(http_request_duration_seconds_bucket{job="${APP_NAME}"}[5m])
        ) > 0.5
      for: 5m
      labels:
        severity: warning
        service: ${APP_NAME}
      annotations:
        summary: "檢測到高響應時間"
        description: "第 95 個百分位數響應時間為 {{ $value }} 秒，適用於 {{ $labels.job }}"
    
    - alert: PodCrashLooping
      expr: |
        increase(kube_pod_container_status_restarts_total{pod=~"${APP_NAME}-.*"}[1h]) > 5
      for: 5m
      labels:
        severity: critical
        service: ${APP_NAME}
      annotations:
        summary: "Pod 正在崩潰循環"
        description: "Pod {{ $labels.pod }} 在過去一小時內已重啟 {{ $value }} 次"
---
# Grafana 儀表板 ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: ${APP_NAME}-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  dashboard.json: |
    {
      "dashboard": {
        "title": "${APP_NAME} 儀表板",
        "panels": [
          {
            "title": "請求率",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(http_requests_total{job=\"$\"{APP_NAME}\"}[5m])",
                "legendFormat": "{{ $labels.method }} {{ $labels.status }}"
              }
            ]
          },
          {
            "title": "響應時間",
            "type": "graph", 
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{job=\"$\"{APP_NAME}\"}[5m]))",
                "legendFormat": "第 95 個百分位數"
              },
              {
                "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket{job=\"$\"{APP_NAME}\"}[5m]))",
                "legendFormat": "第 50 個百分位數"
              }
            ]
          }
        ]
      }
    }
```

### 10. 進階 GitOps 整合

為企業 GitOps 準備清單：

**多叢集 ArgoCD 應用程式**
```yaml
# 多環境部署的應用程式集
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: ${APP_NAME}-appset
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          argocd.argoproj.io/secret-type: cluster
  - git:
      repoURL: https://github.com/org/k8s-manifests
      revision: HEAD
      directories:
      - path: apps/${APP_NAME}/overlays/*
  template:
    metadata:
      name: '${APP_NAME}-{{path.basename}}'
      labels:
        app: ${APP_NAME}
        env: '{{path.basename}}'
    spec:
      project: default
      source:
        repoURL: https://github.com/org/k8s-manifests
        targetRevision: HEAD
        path: 'apps/${APP_NAME}/overlays/{{path.basename}}'
      destination:
        server: '{{server}}'
        namespace: '${APP_NAME}-{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
          allowEmpty: false
        syncOptions:
        - CreateNamespace=true
        - PrunePropagationPolicy=foreground
        - RespectIgnoreDifferences=true
        - ApplyOutOfSyncOnly=true
        managedNamespaceMetadata:
          labels:
            pod-security.kubernetes.io/enforce: restricted
            managed-by: argocd
        retry:
          limit: 5
          backoff:
            duration: 5s
            factor: 2
            maxDuration: 3m
      ignoreDifferences:
      - group: apps
        kind: Deployment
        jsonPointers:
        - /spec/replicas
      - group: autoscaling
        kind: HorizontalPodAutoscaler
        jsonPointers:
        - /spec/minReplicas
        - /spec/maxReplicas
---
# 使用 Argo Rollouts 進行漸進式發布
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: ${APP_NAME}
  namespace: ${NAMESPACE}
spec:
  replicas: 10
  strategy:
    canary:
      maxSurge: "25%"
      maxUnavailable: 0
      analysis:
        templates:
        - templateName: success-rate
        startingStep: 2
        args:
        - name: service-name
          value: ${APP_NAME}
      steps:
      - setWeight: 10
      - pause: {duration: 60s}
      - setWeight: 20
      - pause: {duration: 60s}
      - analysis:
          templates:
          - templateName: success-rate
          args:
          - name: service-name
            value: ${APP_NAME}
      - setWeight: 40
      - pause: {duration: 60s}
      - setWeight: 60
      - pause: {duration: 60s}
      - setWeight: 80
      - pause: {duration: 60s}
      trafficRouting:
        istio:
          virtualService:
            name: ${APP_NAME}
            routes:
            - primary
          destinationRule:
            name: ${APP_NAME}
            canarySubsetName: canary
            stableSubsetName: stable
  selector:
    matchLabels:
      app: ${APP_NAME}
  template:
    metadata:
      labels:
        app: ${APP_NAME}
    spec:
      containers:
      - name: ${APP_NAME}
        image: ${IMAGE}:${TAG}
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
# Rollouts 的分析模板
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
  namespace: ${NAMESPACE}
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    interval: 60s
    count: 5
    successCondition: result[0] >= 0.95
    failureLimit: 3
    provider:
      prometheus:
        address: http://prometheus:9090
        query: |
          sum(
            rate(http_requests_total{job="{{args.service-name}}",status!~"5.."}[5m])
          ) /
          sum(
            rate(http_requests_total{job="{{args.service-name}}"}[5m])
          )
---
# 多叢集服務鏡像 (Linkerd)
apiVersion: linkerd.io/v1alpha2
kind: Link
metadata:
  name: ${APP_NAME}-west
  namespace: ${NAMESPACE}
  annotations:
    multicluster.linkerd.io/target-cluster-name: west
spec:
  targetClusterName: west
  targetClusterDomain: cluster.local
  selector:
    matchLabels:
      app: ${APP_NAME}
      mirror.linkerd.io/exported: "true"
```

### 11. 驗證與測試

驗證生成的清單：

**清單驗證腳本**
```python
#!/usr/bin/env python3
import yaml
import sys
from kubernetes import client, config
from kubernetes.client.rest import ApiException

class ManifestValidator:
    def __init__(self):
        try:
            config.load_incluster_config()
        except:
            config.load_kube_config()
        
        self.api_client = client.ApiClient()
    
    def validate_manifest(self, manifest_file):
        """
        驗證 Kubernetes 清單
        """
        with open(manifest_file) as f:
            manifests = list(yaml.safe_load_all(f))
        
        results = []
        for manifest in manifests:
            result = {
                'kind': manifest.get('kind'),
                'name': manifest.get('metadata', {}).get('name'),
                'valid': False,
                'errors': []
            }
            
            # 乾運行驗證
            try:
                self._dry_run_apply(manifest)
                result['valid'] = True
            except ApiException as e:
                result['errors'].append(str(e))
            
            # 安全檢查
            security_issues = self._check_security(manifest)
            if security_issues:
                result['errors'].extend(security_issues)
            
            # 最佳實踐檢查
            bp_issues = self._check_best_practices(manifest)
            if bp_issues:
                result['errors'].extend(bp_issues)
            
            results.append(result)
        
        return results
    
    def _check_security(self, manifest):
        """檢查安全最佳實踐"""
        issues = []
        
        if manifest.get('kind') == 'Deployment':
            spec = manifest.get('spec', {}).get('template', {}).get('spec', {})
            
            # 檢查安全上下文
            if not spec.get('securityContext'):
                issues.append("缺少 Pod 安全上下文")
            
            # 檢查容器安全
            for container in spec.get('containers', []):
                if not container.get('securityContext'):
                    issues.append(f"容器 {container['name']} 缺少安全上下文")
                
                sec_ctx = container.get('securityContext', {})
                if not sec_ctx.get('runAsNonRoot'):
                    issues.append(f"容器 {container['name']} 未配置為非 root 運行")
                
                if not sec_ctx.get('readOnlyRootFilesystem'):
                    issues.append(f"容器 {container['name']} 具有可寫的根檔案系統")
        
        return issues
```

### 11. 進階擴展和性能

實施智慧擴展策略：

**KEDA 自動擴展**
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: ${APP_NAME}-scaler
  namespace: ${NAMESPACE}
spec:
  scaleTargetRef:
    name: ${APP_NAME}
  pollingInterval: 30
  cooldownPeriod: 300
  idleReplicaCount: 2
  minReplicaCount: 2
  maxReplicaCount: 50
  fallback:
    failureThreshold: 3
    replicas: 5
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://prometheus:9090
      metricName: http_requests_per_second
      threshold: '100'
      query: sum(rate(http_requests_total{job="${APP_NAME}"}[2m]))
  - type: memory
    metadata:
      type: Utilization
      value: "70"
  - type: cpu
    metadata:
      type: Utilization
      value: "70"
---
# 垂直 Pod 自動擴展器
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: ${APP_NAME}-vpa
  namespace: ${NAMESPACE}
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ${APP_NAME}
  updatePolicy:
    updateMode: "Auto"
    minReplicas: 2
  resourcePolicy:
    containerPolicies:
    - containerName: ${APP_NAME}
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]
      controlledValues: RequestsAndLimits
---
# Pod 中斷預算
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: ${APP_NAME}-pdb
  namespace: ${NAMESPACE}
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: ${APP_NAME}
```

### 12. CI/CD 整合

現代部署管道整合：

**GitHub Actions 工作流程**
```yaml
# .github/workflows/deploy.yml
name: 部署到 Kubernetes

on:
  push:
    branches: [main]
    paths: ['src/**', 'k8s/**', 'Dockerfile']
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
      id-token: write
    
    steps:
    - name: 結帳
      uses: actions/checkout@v4
      with:
        fetch-depth: 0
    
    - name: 設定 GitVersion
      uses: gittools/actions/gitversion/setup@v0.9.15
      with:
        versionSpec: '5.x'
    
    - name: 確定版本
      uses: gittools/actions/gitversion/execute@v0.9.15
      id: gitversion
    
    - name: 設定 Docker Buildx
      uses: docker/setup-buildx-action@v3
    
    - name: 登入容器註冊表
      uses: docker/login-action@v3
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: 建置並推送 Docker 映像
      uses: docker/build-push-action@v5
      with:
        context: .
        platforms: linux/amd64,linux/arm64
        push: true
        tags: |
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.gitversion.outputs.semVer }}
          ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
        cache-from: type=gha
        cache-to: type=gha,mode=max
        build-args: |
          VERSION=${{ steps.gitversion.outputs.semVer }}
          COMMIT_SHA=${{ github.sha }}
    
    - name: 運行 Trivy 漏洞掃描器
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: '${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.gitversion.outputs.semVer }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
    
    - name: 上傳 Trivy 掃描結果
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: 'trivy-results.sarif'
    
    - name: 安裝 kubectl 和 kustomize
      run: |
        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
        chmod +x kubectl && sudo mv kubectl /usr/local/bin/
        curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
        sudo mv kustomize /usr/local/bin/
    
    - name: 驗證 Kubernetes 清單
      run: |
        kubectl --dry-run=client --validate=true apply -k k8s/overlays/staging
    
    - name: 部署到 staging
      if: github.ref == 'refs/heads/main'
      run: |
        cd k8s/overlays/staging
        kustomize edit set image app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.gitversion.outputs.semVer }}
        kubectl apply -k .
        kubectl rollout status deployment/${APP_NAME} -n staging --timeout=300s
    
    - name: 運行整合測試
      if: github.ref == 'refs/heads/main'
      run: |
        # 等待部署準備就緒
        kubectl wait --for=condition=available --timeout=300s deployment/${APP_NAME} -n staging
        # 運行測試
        npm run test:integration
    
    - name: 部署到 production
      if: github.ref == 'refs/heads/main' && success()
      run: |
        cd k8s/overlays/production
        kustomize edit set image app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ steps.gitversion.outputs.semVer }}
        kubectl apply -k .
        kubectl rollout status deployment/${APP_NAME} -n production --timeout=600s
```

## 輸出格式

1. **框架優化清單**：定制的部署配置
2. **進階安全套件**：PSS、OPA、Falco、服務網格策略
3. **GitOps 儲存庫結構**：帶有 FluxCD/ArgoCD 的多環境
4. **可觀察性堆疊**：OpenTelemetry、Prometheus、Grafana、Jaeger
5. **漸進式交付設定**：帶有金絲雀部署的 Argo Rollouts
6. **自動擴展配置**：HPA、VPA、KEDA 用於智慧擴展
7. **多叢集設定**：服務網格和跨叢集通訊
8. **CI/CD 管道**：帶有安全掃描的完整 GitHub Actions 工作流程
9. **災難恢復計畫**：備份策略和恢復程序
10. **性能基準**：負載測試和優化建議

## 跨指令整合

### 完整的雲原生部署工作流程

**企業 Kubernetes 管道**
```bash
# 1. 生成雲原生 API 腳手架
/api-scaffold
framework: "fastapi"
deployment_target: "kubernetes"
cloud_native: true
observability: ["prometheus", "jaeger", "grafana"]

# 2. 優化容器以用於 Kubernetes
/docker-optimize
optimization_level: "kubernetes"
multi_arch_build: true
security_hardening: true

# 3. 全面安全掃描
/security-scan
scan_types: ["k8s", "container", "iac", "rbac"]
compliance: ["cis", "nsa", "pci"]

# 4. 生成生產 K8s 清單
/k8s-manifest
environment: "production"
security_level: "enterprise"
auto_scaling: true
service_mesh: true
```

**整合 Kubernetes 配置**
```python
# k8s-integration-config.py - 所有指令共享
class IntegratedKubernetesConfig:
    def __init__(self):
        self.api_config = self.load_api_config()           # 來自 /api-scaffold
        self.container_config = self.load_container_config() # 來自 /docker-optimize
        self.security_config = self.load_security_config() # 來自 /security-scan
        self.test_config = self.load_test_config()         # 來自 /test-harness
        
    def generate_application_manifests(self):
        """生成應用程式堆疊的完整 K8s 清單"""
        manifests = {
            'namespace': self.generate_namespace_manifest(),
            'secrets': self.generate_secrets_manifests(),
            'configmaps': self.generate_configmap_manifests(),
            'deployments': self.generate_deployment_manifests(),
            'services': self.generate_service_manifests(),
            'ingress': self.generate_ingress_manifests(),
            'security': self.generate_security_manifests(),
            'monitoring': self.generate_monitoring_manifests(),
            'autoscaling': self.generate_autoscaling_manifests()
        }
        return manifests
    
    def generate_deployment_manifests(self):
        """從 API 和容器配置生成部署清單"""
        deployments = []
        
        # API 部署
        if self.api_config.get('framework'):
            api_deployment = {
                'apiVersion': 'apps/v1',
                'kind': 'Deployment',
                'metadata': {
                    'name': f"{self.api_config['name']}-api",
                    'namespace': self.api_config.get('namespace', 'default'),
                    'labels': {
                        'app': f"{self.api_config['name']}-api",
                        'framework': self.api_config['framework'],
                        'version': self.api_config.get('version', 'v1.0.0'),
                        'component': 'backend'
                    }
                },
                'spec': {
                    'replicas': self.calculate_replica_count(),
                    'selector': {
                        'matchLabels': {
                            'app': f"{self.api_config['name']}-api"
                        }
                    },
                    'template': {
                        'metadata': {
                            'labels': {
                                'app': f"{self.api_config['name']}-api"
                            },
                            'annotations': self.generate_pod_annotations()
                        },
                        'spec': self.generate_pod_spec()
                    }
                }
            }
            deployments.append(api_deployment)
        
        return deployments
    
    def generate_pod_spec(self):
        """生成優化的 Pod 規範"""
        containers = []
        
        # 主應用程式容器
        app_container = {
            'name': 'app',
            'image': self.container_config.get('image_name', 'app:latest'),
            'imagePullPolicy': 'Always',
            'ports': [
                {
                    'name': 'http',
                    'containerPort': self.api_config.get('port', 8000),
                    'protocol': 'TCP'
                }
            ],
            'env': self.generate_environment_variables(),
            'resources': self.calculate_resource_requirements(),
            'securityContext': self.generate_security_context(),
            'livenessProbe': self.generate_health_probes('liveness'),
            'readinessProbe': self.generate_health_probes('readiness'),
            'startupProbe': self.generate_health_probes('startup'),
            'volumeMounts': self.generate_volume_mounts()
        }
        containers.append(app_container)
        
        # Sidecar 容器（監控、安全等）
        if self.should_include_monitoring_sidecar():
            containers.append(self.generate_monitoring_sidecar())
        
        if self.should_include_security_sidecar():
            containers.append(self.generate_security_sidecar())
        
        pod_spec = {
            'serviceAccountName': f"{self.api_config['name']}-sa",
            'securityContext': self.generate_pod_security_context(),
            'containers': containers,
            'volumes': self.generate_volumes(),
            'initContainers': self.generate_init_containers(),
            'nodeSelector': self.generate_node_selector(),
            'tolerations': self.generate_tolerations(),
            'affinity': self.generate_affinity_rules(),
            'topologySpreadConstraints': self.generate_topology_constraints()
        }
        
        return pod_spec
    
    def generate_security_context(self):
        """從安全掃描結果生成容器安全上下文"""
        security_level = self.security_config.get('level', 'standard')
        
        base_context = {
            'allowPrivilegeEscalation': False,
            'readOnlyRootFilesystem': True,
            'runAsNonRoot': True,
            'runAsUser': 1001,
            'capabilities': {
                'drop': ['ALL']
            }
        }
        
        if security_level == 'enterprise':
            base_context.update({
                'seccompProfile': {'type': 'RuntimeDefault'},
                'capabilities': {
                    'drop': ['ALL'],
                    'add': ['NET_BIND_SERVICE'] if self.api_config.get('privileged_port') else []
                }
            })
        
        return base_context
```

**與 Kubernetes 的資料庫整合**
```yaml
# database-k8s-manifests.yaml - 來自 /db-migrate + /k8s-manifest
apiVersion: v1
kind: Secret
metadata:
  name: database-credentials
  namespace: production
type: Opaque
data:
  username: cG9zdGdyZXM=  # postgres (base64)
  password: <ENCODED_PASSWORD>
  database: YXBwX2Ri  # app_db (base64)

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: database-config
  namespace: production
data:
  postgresql.conf: |
    # 來自 /db-migrate 分析的性能調整
    shared_buffers = 256MB
    effective_cache_size = 1GB
    work_mem = 4MB
    maintenance_work_mem = 64MB
    
    # 來自 /security-scan 的安全設定
    ssl = on
    log_connections = on
    log_disconnections = on
    log_statement = 'all'
    
    # 監控設定
    shared_preload_libraries = 'pg_stat_statements'
    track_activity_query_size = 2048

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-pvc
  namespace: production
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-ssd
  resources:
    requests:
      storage: 100Gi

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
  namespace: production
spec:
  serviceName: database-headless
  replicas: 3  # 高可用性設定
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      serviceAccountName: database-sa
      securityContext:
        runAsUser: 999
        runAsGroup: 999
        fsGroup: 999
      containers:
      - name: postgresql
        image: postgres:15-alpine
        ports:
        - name: postgresql
          containerPort: 5432
        env:
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: username
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: password
        - name: POSTGRES_DB
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: database
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
        volumeMounts:
        - name: database-storage
          mountPath: /var/lib/postgresql/data
        - name: database-config
          mountPath: /etc/postgresql/postgresql.conf
          subPath: postgresql.conf
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "2000m"
        livenessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - pg_isready
            - -U
            - postgres
          initialDelaySeconds: 5
          periodSeconds: 5
      # 來自 /db-migrate 的遷移初始化容器
      initContainers:
      - name: migration
        image: migration-runner:latest
        env:
        - name: DATABASE_URL
          value: "postgresql://$(POSTGRES_USER):$(POSTGRES_PASSWORD)@localhost:5432/$(POSTGRES_DB)"

```

**前端 + 後端整合**
```yaml
# fullstack-k8s-deployment.yaml - 整合所有指令
apiVersion: v1
kind: Namespace
metadata:
  name: fullstack-app
  labels:
    name: fullstack-app
    security-policy: strict

---
# API 部署（來自 /api-scaffold + 優化）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  namespace: fullstack-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api
      tier: backend
  template:
    metadata:
      labels:
        app: api
        tier: backend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: api-service-account
      containers:
      - name: api
        image: registry.company.com/api:optimized-latest
        ports:
        - containerPort: 8000
          name: http
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-credentials
              key: url
        - name: REDIS_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: redis-url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
# 前端部署（來自 /frontend-optimize + 容器優化）
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
  namespace: fullstack-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
      tier: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: frontend
    spec:
      containers:
      - name: frontend
        image: registry.company.com/frontend:optimized-latest
        ports:
        - containerPort: 80
          name: http
        env:
        - name: API_URL
          value: "http://api-service:8000"
        - name: NODE_ENV
          value: "production"
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1001
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

---
# 服務
apiVersion: v1
kind: Service
metadata:
  name: api-service
  namespace: fullstack-app
spec:
  selector:
    app: api
    tier: backend
  ports:
  - name: http
    port: 8000
    targetPort: 8000
  type: ClusterIP

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
  namespace: fullstack-app
spec:
  selector:
    app: frontend
    tier: frontend
  ports:
  - name: http
    port: 80
    targetPort: 80
  type: ClusterIP

---
# Ingress 帶有來自 /security-scan 的安全配置
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: fullstack-app
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
    nginx.ingress.kubernetes.io/rate-limit-window: "1m"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/add-base-url: "true"
    nginx.ingress.kubernetes.io/proxy-buffer-size: "8k"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - app.company.com
    secretName: app-tls-secret
  rules:
  - host: app.company.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8000
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**安全整合**
```yaml
# security-k8s-manifests.yaml - 來自 /security-scan 整合
apiVersion: v1
kind: ServiceAccount
metadata:
  name: api-service-account
  namespace: fullstack-app
automountServiceAccountToken: false

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: api-role
  namespace: fullstack-app
rules:
- apiGroups: [""]
  resources: ["secrets", "configmaps"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: api-role-binding
  namespace: fullstack-app
subjects:
- kind: ServiceAccount
  name: api-service-account
  namespace: fullstack-app
roleRef:
  kind: Role
  name: api-role
  apiGroup: rbac.authorization.k8s.io

---
# API 的網路策略
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-network-policy
  namespace: fullstack-app
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []  # 允許 DNS
    ports:
    - protocol: UDP
      port: 53

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-network-policy
  namespace: fullstack-app
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 80
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: api
    ports:
    - protocol: TCP
      port: 8000

---
# Pod 安全標準
apiVersion: v1
kind: LimitRange
metadata:
  name: resource-limits
  namespace: fullstack-app
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
      ephemeral-storage: "1Gi"
    defaultRequest:
      cpu: "100m"
      memory: "128Mi"
      ephemeral-storage: "500Mi"
    type: Container
  - max:
      cpu: "2"
      memory: "4Gi"
      ephemeral-storage: "10Gi"
    type: Container

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: api-pdb
  namespace: fullstack-app
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: api

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
  namespace: fullstack-app
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: frontend
```

**監控和可觀察性整合**
```yaml
# monitoring-k8s-manifests.yaml - 完整的可觀察性堆疊
apiVersion: v1
kind: ServiceMonitor
metadata:
  name: api-monitor
  namespace: fullstack-app
  labels:
    app: api
spec:
  selector:
    matchLabels:
      app: api
  endpoints:
  - port: http
    path: /metrics
    interval: 30s

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
  namespace: monitoring
data:
  app-dashboard.json: |
    {
      "dashboard": {
        "title": "Application Metrics",
        "panels": [
          {
            "title": "API Response Time",
            "targets": [
              {
                "expr": "http_request_duration_seconds{job=\"api-service\"}",
                "legendFormat": "Response Time"
              }
            ]
          },
          {
            "title": "Error Rate",
            "targets": [
              {
                "expr": "rate(http_requests_total{job=\"api-service\",status=~\"5..\"}[5m])",
                "legendFormat": "5xx Errors"
              }
            ]
          }
        ]
      }
    }

---
# Jaeger 追蹤配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: jaeger-config
  namespace: fullstack-app
data:
  jaeger.yaml: |
    sampling:
      type: probabilistic
      param: 0.1
    reporter:
      logSpans: true
      localAgentHostPort: jaeger-agent:6831

---
# 應用程式日誌記錄配置
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: fullstack-app
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf
    
    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5
        Mem_Buf_Limit     5MB
        Skip_Long_Lines   On
    
    [FILTER]
        Name                kubernetes
        Match               kube.*
        Kube_URL            https://kubernetes.default.svc:443
        Kube_CA_File        /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        Kube_Token_File     /var/run/secrets/kubernetes.io/serviceaccount/token
        Merge_Log           On
    
    [OUTPUT]
        Name  es
        Match *
        Host  elasticsearch.logging.svc.cluster.local
        Port  9200
        Index app-logs
```

**自動擴展整合**
```yaml
# autoscaling-k8s-manifests.yaml - 基於多個指標的智慧擴展
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
  namespace: fullstack-app
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "1000"
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 25
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 15

---
apiVersion: autoscaling/v1
kind: VerticalPodAutoscaler
metadata:
  name: api-vpa
  namespace: fullstack-app
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-deployment
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: api
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2
        memory: 4Gi
      controlledResources: ["cpu", "memory"]

---
# KEDA 用於基於外部指標的高級自動擴展
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: api-scaled-object
  namespace: fullstack-app
spec:
  scaleTargetRef:
    name: api-deployment
  minReplicaCount: 2
  maxReplicaCount: 50
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://prometheus.monitoring.svc.cluster.local:9090
      metricName: http_requests_per_second
      threshold: '1000'
      query: sum(rate(http_requests_total{job="api-service"}[1m]))
  - type: redis
    metadata:
      address: redis.fullstack-app.svc.cluster.local:6379
      listName: task_queue
      listLength: '10'
```

**CI/CD 整合管道**
```yaml
# .github/workflows/k8s-deployment.yml
name: Kubernetes Deployment Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  CLUSTER_NAME: production-cluster

jobs:
  deploy-to-kubernetes:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: read
      id-token: write
    
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4
    
    # 1. 設定 kubectl 和 helm
    - name: Setup kubectl
      uses: azure/setup-kubectl@v3
      with:
        version: 'v1.28.0'
    
    - name: Setup Helm
      uses: azure/setup-helm@v3
      with:
        version: 'v3.12.0'
    
    # 2. 與叢集進行身份驗證
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        role-to-assume: ${{ secrets.AWS_ROLE_TO_ASSUME }}
        aws-region: us-west-2
    
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --region us-west-2 --name ${{ env.CLUSTER_NAME }}
    
    # 3. 驗證清單
    - name: Validate Kubernetes manifests
      run: |
        # 驗證語法
        kubectl --dry-run=client apply -f k8s/
        
        # 使用 kubesec 進行安全驗證
        docker run --rm -v $(pwd):/workspace kubesec/kubesec:latest scan /workspace/k8s/*.yaml
        
        # 使用 OPA Gatekeeper 進行策略驗證
        conftest verify --policy opa-policies/ k8s/
    
    # 4. 部署到 staging
    - name: Deploy to staging
      if: github.ref == 'refs/heads/develop'
      run: |
        # 更新映像標籤
        sed -i "s|registry.company.com/api:.*|registry.company.com/api:${{ github.sha }}|g" k8s/api-deployment.yaml
        sed -i "s|registry.company.com/frontend:.*|registry.company.com/frontend:${{ github.sha }}|g" k8s/frontend-deployment.yaml
        
        # 將清單應用到 staging 命名空間
        kubectl apply -f k8s/ --namespace=staging
        
        # 等待滾動更新完成
        kubectl rollout status deployment/api-deployment --namespace=staging --timeout=300s
        kubectl rollout status deployment/frontend-deployment --namespace=staging --timeout=300s
    
    # 5. 運行整合測試
    - name: Run integration tests
      if: github.ref == 'refs/heads/develop'
      run: |
        # 等待服務準備就緒
        kubectl wait --for=condition=ready pod -l app=api --namespace=staging --timeout=300s
        
        # 獲取服務 URL
        API_URL=$(kubectl get service api-service --namespace=staging -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        
        # 從 /test-harness 運行測試
        pytest tests/integration/ --api-url="http://${API_URL}:8000" -v
    
    # 6. 部署到 production（在 main 分支上）
    - name: Deploy to production
      if: github.ref == 'refs/heads/main'
      run: |
        # 更新映像標籤
        sed -i "s|registry.company.com/api:.*|registry.company.com/api:${{ github.sha }}|g" k8s/api-deployment.yaml
        sed -i "s|registry.company.com/frontend:.*|registry.company.com/frontend:${{ github.sha }}|g" k8s/frontend-deployment.yaml
        
        # 將清單應用到 production 命名空間並進行滾動更新
        kubectl apply -f k8s/ --namespace=production
        
        # 監控滾動更新
        kubectl rollout status deployment/api-deployment --namespace=production --timeout=600s
        kubectl rollout status deployment/frontend-deployment --namespace=production --timeout=600s
        
        # 驗證部署健康狀況
        kubectl get pods --namespace=production -l app=api
        kubectl get pods --namespace=production -l app=frontend
    
    # 7. 部署後驗證
    - name: Post-deployment verification
      if: github.ref == 'refs/heads/main'
      run: |
        # 健康檢查
        kubectl exec -n production deployment/api-deployment -- curl -f http://localhost:8000/health
        
        # 性能基準檢查
        kubectl run --rm -i --tty load-test --image=loadimpact/k6:latest --restart=Never -- run - <<EOF
        import http from 'k6/http';
        import { check } from 'k6';
        
        export let options = {
          stages: [
            { duration: '2m', target: 100 },
            { duration: '5m', target: 100 },
            { duration: '2m', target: 0 },
          ],
        };
        
        export default function () {
          let response = http.get('http://api-service.production.svc.cluster.local:8000/health');
          check(response, {
            'status is 200': (r) => r.status === 200,
            'response time < 500ms': (r) => r.timings.duration < 500,
          });
        }
        EOF
    
    # 8. 失敗時的清理
    - name: Rollback on failure
      if: failure()
      run: |
        # 回滾到先前版本
        kubectl rollout undo deployment/api-deployment --namespace=production
        kubectl rollout undo deployment/frontend-deployment --namespace=production
        
        # 通知團隊
        echo "Deployment failed and rolled back" >> $GITHUB_STEP_SUMMARY
```

此全面的整合確保 Kubernetes 部署能夠利用從容器建置、安全強化、資料庫遷移和監控配置中獲得的所有優化，同時提供企業級的可靠性和可觀察性。
