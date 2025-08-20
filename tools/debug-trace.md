# 除錯與追蹤配置

您是除錯專家，專精於設定全面的除錯環境、分散式追蹤和診斷工具。配置除錯工作流程、實施追蹤解決方案，並為開發和生產環境建立故障排除實踐。

## 背景
使用者需要設定除錯和追蹤功能，以有效診斷問題、追蹤錯誤並了解系統行為。專注於開發人員生產力、生產除錯、分散式追蹤和全面的日誌記錄策略。

## 要求
$ARGUMENTS

## 指示

### 1. 開發環境除錯

設定全面的除錯環境：

**VS Code 除錯配置**
```json
// .vscode/launch.json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "除錯 Node.js 應用程式",
            "type": "node",
            "request": "launch",
            "runtimeExecutable": "node",
            "runtimeArgs": ["--inspect-brk", "--enable-source-maps"],
            "program": "${workspaceFolder}/src/index.js",
            "env": {
                "NODE_ENV": "development",
                "DEBUG": "*",
                "NODE_OPTIONS": "--max-old-space-size=4096"
            },
            "sourceMaps": true,
            "resolveSourceMapLocations": [
                "${workspaceFolder}/**",
                "!**/node_modules/**"
            ],
            "skipFiles": [
                "<node_internals>/**",
                "node_modules/**"
            ],
            "console": "integratedTerminal",
            "outputCapture": "std"
        },
        {
            "name": "除錯 TypeScript",
            "type": "node",
            "request": "launch",
            "program": "${workspaceFolder}/src/index.ts",
            "preLaunchTask": "tsc: build - tsconfig.json",
            "outFiles": ["${workspaceFolder}/dist/**/*.js"],
            "sourceMaps": true,
            "smartStep": true,
            "internalConsoleOptions": "openOnSessionStart"
        },
        {
            "name": "除錯 Jest 測試",
            "type": "node",
            "request": "launch",
            "program": "${workspaceFolder}/node_modules/.bin/jest",
            "args": [
                "--runInBand",
                "--no-cache",
                "--watchAll=false",
                "--detectOpenHandles"
            ],
            "console": "integratedTerminal",
            "internalConsoleOptions": "neverOpen",
            "env": {
                "NODE_ENV": "test"
            }
        },
        {
            "name": "附加到進程",
            "type": "node",
            "request": "attach",
            "processId": "${command:PickProcess}",
            "protocol": "inspector",
            "restart": true,
            "sourceMaps": true
        }
    ],
    "compounds": [
        {
            "name": "全棧除錯",
            "configurations": ["除錯後端", "除錯前端"],
            "stopAll": true
        }
    ]
}
```

**Chrome DevTools 配置**
```javascript
// debug-helpers.js
class DebugHelper {
    constructor() {
        this.setupDevTools();
        this.setupConsoleHelpers();
        this.setupPerformanceMarkers();
    }
    
    setupDevTools() {
        if (typeof window !== 'undefined') {
            // 添加除錯命名空間
            window.DEBUG = window.DEBUG || {};
            
            // 儲存重要物件的引用
            window.DEBUG.store = () => window.__REDUX_STORE__;
            window.DEBUG.router = () => window.__ROUTER__;
            window.DEBUG.components = new Map();
            
            // 性能除錯
            window.DEBUG.measureRender = (componentName) => {
                performance.mark(`${componentName}-start`);
                return () => {
                    performance.mark(`${componentName}-end`);
                    performance.measure(
                        componentName,
                        `${componentName}-start`,
                        `${componentName}-end`
                    );
                };
            };
            
            // 記憶體除錯
            window.DEBUG.heapSnapshot = async () => {
                if ('memory' in performance) {
                    const snapshot = await performance.measureUserAgentSpecificMemory();
                    console.table(snapshot);
                    return snapshot;
                }
            };
        }
    }
    
    setupConsoleHelpers() {
        // 增強的控制台日誌記錄
        const styles = {
            error: 'color: #ff0000; font-weight: bold;',
            warn: 'color: #ff9800; font-weight: bold;',
            info: 'color: #2196f3; font-weight: bold;',
            debug: 'color: #4caf50; font-weight: bold;',
            trace: 'color: #9c27b0; font-weight: bold;'
        };
        
        Object.entries(styles).forEach(([level, style]) => {
            const original = console[level];
            console[level] = function(...args) {
                if (process.env.NODE_ENV === 'development') {
                    const timestamp = new Date().toISOString();
                    original.call(console, `%c[${timestamp}] ${level.toUpperCase()}:`, style, ...args);
                }
            };
        });
    }
}

// React DevTools 整合
if (process.env.NODE_ENV === 'development') {
    // 暴露 React 內部
    window.__REACT_DEVTOOLS_GLOBAL_HOOK__ = {
        ...window.__REACT_DEVTOOLS_GLOBAL_HOOK__,
        onCommitFiberRoot: (id, root) => {
            // 自定義提交日誌記錄
            console.debug('React 提交:', root);
        }
    };
}
```

### 2. 遠端除錯設定

配置遠端除錯功能：

**遠端除錯伺服器**
```javascript
// remote-debug-server.js
const inspector = require('inspector');
const WebSocket = require('ws');
const http = require('http');

class RemoteDebugServer {
    constructor(options = {}) {
        this.port = options.port || 9229;
        this.host = options.host || '0.0.0.0';
        this.wsPort = options.wsPort || 9230;
        this.sessions = new Map();
    }
    
    start() {
        // 開啟檢查器
        inspector.open(this.port, this.host, true);
        
        // 為遠端連接建立 WebSocket 伺服器
        this.wss = new WebSocket.Server({ port: this.wsPort });
        
        this.wss.on('connection', (ws) => {
            const sessionId = this.generateSessionId();
            this.sessions.set(sessionId, ws);
            
            ws.on('message', (message) => {
                this.handleDebugCommand(sessionId, message);
            });
            
            ws.on('close', () => {
                this.sessions.delete(sessionId);
            });
            
            // 發送初始會話資訊
            ws.send(JSON.stringify({
                type: 'session',
                sessionId,
                debugUrl: `chrome-devtools://devtools/bundled/inspector.html?ws=${this.host}:${this.port}`
            }));
        });
        
        console.log(`遠端除錯伺服器正在監聽 ws://${this.host}:${this.wsPort}`);
    }
    
    handleDebugCommand(sessionId, message) {
        const command = JSON.parse(message);
        
        switch (command.type) {
            case 'evaluate':
                this.evaluateExpression(sessionId, command.expression);
                break;
            case 'setBreakpoint':
                this.setBreakpoint(command.file, command.line);
                break;
            case 'heapSnapshot':
                this.takeHeapSnapshot(sessionId);
                break;
            case 'profile':
                this.startProfiling(sessionId, command.duration);
                break;
        }
    }
    
    evaluateExpression(sessionId, expression) {
        const session = new inspector.Session();
        session.connect();
        
        session.post('Runtime.evaluate', {
            expression,
            generatePreview: true,
            includeCommandLineAPI: true
        }, (error, result) => {
            const ws = this.sessions.get(sessionId);
            if (ws) {
                ws.send(JSON.stringify({
                    type: 'evaluateResult',
                    result: result || error
                }));
            }
        });
        
        session.disconnect();
    }
}

// Docker 遠端除錯設定
FROM node:18
RUN apt-get update && apt-get install -y \
    chromium \
    gdb \
    strace \
    tcpdump \
    vim
    
EXPOSE 9229 9230
ENV NODE_OPTIONS="--inspect=0.0.0.0:9229"
CMD ["node", "--inspect-brk=0.0.0.0:9229", "index.js"]
```

### 3. 分散式追蹤

實施全面的分散式追蹤：

**OpenTelemetry 設定**
```javascript
// tracing.js
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { Resource } = require('@opentelemetry/resources');
const { SemanticResourceAttributes } = require('@opentelemetry/semantic-conventions');
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger');
const { BatchSpanProcessor } = require('@opentelemetry/sdk-trace-base');

class TracingSystem {
    constructor(serviceName) {
        this.serviceName = serviceName;
        this.sdk = null;
    }
    
    initialize() {
        const jaegerExporter = new JaegerExporter({
            endpoint: process.env.JAEGER_ENDPOINT || 'http://localhost:14268/api/traces',
        });
        
        const resource = Resource.default().merge(
            new Resource({
                [SemanticResourceAttributes.SERVICE_NAME]: this.serviceName,
                [SemanticResourceAttributes.SERVICE_VERSION]: process.env.SERVICE_VERSION || '1.0.0',
                [SemanticResourceAttributes.DEPLOYMENT_ENVIRONMENT]: process.env.NODE_ENV || 'development',
            })
        );
        
        this.sdk = new NodeSDK({
            resource,
            spanProcessor: new BatchSpanProcessor(jaegerExporter),
            instrumentations: [
                getNodeAutoInstrumentations({
                    '@opentelemetry/instrumentation-fs': {
                        enabled: false, // 太吵
                    },
                    '@opentelemetry/instrumentation-http': {
                        requestHook: (span, request) => {
                            span.setAttribute('http.request.body', JSON.stringify(request.body));
                        },
                        responseHook: (span, response) => {
                            span.setAttribute('http.response.size', response.length);
                        },
                    },
                    '@opentelemetry/instrumentation-express': {
                        requestHook: (span, req) => {
                            span.setAttribute('user.id', req.user?.id);
                            span.setAttribute('session.id', req.session?.id);
                        },
                    },
                }),
            ],
        });
        
        this.sdk.start();
        
        // 優雅關閉
        process.on('SIGTERM', () => {
            this.sdk.shutdown()
                .then(() => console.log('追蹤已終止'))
                .catch((error) => console.error('終止追蹤時出錯', error))
                .finally(() => process.exit(0));
        });
    }
    
    // 自定義 Span 建立
    createSpan(name, fn, attributes = {}) {
        const tracer = trace.getTracer(this.serviceName);
        return tracer.startActiveSpan(name, async (span) => {
            try {
                // 添加自定義屬性
                Object.entries(attributes).forEach(([key, value]) => {
                    span.setAttribute(key, value);
                });
                
                // 執行函數
                const result = await fn(span);
                
                span.setStatus({ code: SpanStatusCode.OK });
                return result;
            } catch (error) {
                span.recordException(error);
                span.setStatus({
                    code: SpanStatusCode.ERROR,
                    message: error.message,
                });
                throw error;
            } finally {
                span.end();
            }
        });
    }
}

// 分散式追蹤中介軟體
class TracingMiddleware {
    constructor() {
        this.tracer = trace.getTracer('http-middleware');
    }
    
    express() {
        return (req, res, next) => {
            const span = this.tracer.startSpan(`${req.method} ${req.path}`, {
                kind: SpanKind.SERVER,
                attributes: {
                    'http.method': req.method,
                    'http.url': req.url,
                    'http.target': req.path,
                    'http.host': req.hostname,
                    'http.scheme': req.protocol,
                    'http.user_agent': req.get('user-agent'),
                    'http.request_content_length': req.get('content-length'),
                },
            });
            
            // 將追蹤上下文注入請求
            req.span = span;
            req.traceId = span.spanContext().traceId;
            
            // 將追蹤 ID 添加到響應標頭
            res.setHeader('X-Trace-Id', req.traceId);
            
            // 覆寫 res.end 以捕獲響應資料
            const originalEnd = res.end;
            res.end = function(...args) {
                span.setAttribute('http.status_code', res.statusCode);
                span.setAttribute('http.response_content_length', res.get('content-length'));
                
                if (res.statusCode >= 400) {
                    span.setStatus({
                        code: SpanStatusCode.ERROR,
                        message: `HTTP ${res.statusCode}`,
                    });
                }
                
                span.end();
                originalEnd.apply(res, args);
            };
            
            next();
        };
    }
}
```

### 4. 除錯日誌記錄框架

實施結構化除錯日誌記錄：

**進階日誌記錄器**
```javascript
// debug-logger.js
const winston = require('winston');
const { ElasticsearchTransport } = require('winston-elasticsearch');

class DebugLogger {
    constructor(options = {}) {
        this.service = options.service || 'app';
        this.level = process.env.LOG_LEVEL || 'debug';
        this.logger = this.createLogger();
    }
    
    createLogger() {
        const formats = [
            winston.format.timestamp(),
            winston.format.errors({ stack: true }),
            winston.format.splat(),
            winston.format.json(),
        ];
        
        if (process.env.NODE_ENV === 'development') {
            formats.push(winston.format.colorize());
            formats.push(winston.format.printf(this.devFormat));
        }
        
        const transports = [
            new winston.transports.Console({
                level: this.level,
                handleExceptions: true,
                handleRejections: true,
            }),
        ];
        
        // 添加檔案傳輸以進行除錯
        if (process.env.DEBUG_LOG_FILE) {
            transports.push(
                new winston.transports.File({
                    filename: process.env.DEBUG_LOG_FILE,
                    level: 'debug',
                    maxsize: 10485760, // 10MB
                    maxFiles: 5,
                })
            );
        }
        
        // 為生產環境添加 Elasticsearch
        if (process.env.ELASTICSEARCH_URL) {
            transports.push(
                new ElasticsearchTransport({
                    level: 'info',
                    clientOpts: {
                        node: process.env.ELASTICSEARCH_URL,
                    },
                    index: `logs-${this.service}`,
                })
            );
        }
        
        return winston.createLogger({
            level: this.level,
            format: winston.format.combine(...formats),
            defaultMeta: {
                service: this.service,
                environment: process.env.NODE_ENV,
                hostname: require('os').hostname(),
                pid: process.pid,
            },
            transports,
        });
    }
    
    devFormat(info) {
        const { timestamp, level, message, ...meta } = info;
        const metaString = Object.keys(meta).length ? 
            '\n' + JSON.stringify(meta, null, 2) : '';
        
        return `${timestamp} [${level}]: ${message}${metaString}`;
    }
    
    // 除錯特定方法
    trace(message, meta = {}) {
        const stack = new Error().stack;
        this.logger.debug(message, {
            ...meta,
            trace: stack,
            timestamp: Date.now(),
        });
    }
    
    timing(label, fn) {
        const start = process.hrtime.bigint();
        const result = fn();
        const end = process.hrtime.bigint();
        const duration = Number(end - start) / 1000000; // 轉換為毫秒
        
        this.logger.debug(`計時: ${label}`, {
            duration,
            unit: 'ms',
        });
        
        return result;
    }
    
    memory() {
        const usage = process.memoryUsage();
        this.logger.debug('記憶體使用量', {
            rss: `${Math.round(usage.rss / 1024 / 1024)}MB`,
            heapTotal: `${Math.round(usage.heapTotal / 1024 / 1024)}MB`,
            heapUsed: `${Math.round(usage.heapUsed / 1024 / 1024)}MB`,
            external: `${Math.round(usage.external / 1024 / 1024)}MB`,
        });
    }
}

// 除錯上下文管理器
class DebugContext {
    constructor() {
        this.contexts = new Map();
    }
    
    create(id, metadata = {}) {
        const context = {
            id,
            startTime: Date.now(),
            metadata,
            logs: [],
            spans: [],
        };
        
        this.contexts.set(id, context);
        return context;
    }
    
    log(contextId, level, message, data = {}) {
        const context = this.contexts.get(contextId);
        if (context) {
            context.logs.push({
                timestamp: Date.now(),
                level,
                message,
                data,
            });
        }
    }
    
    export(contextId) {
        const context = this.contexts.get(contextId);
        if (!context) return null;
        
        return {
            ...context,
            duration: Date.now() - context.startTime,
            logCount: context.logs.length,
        };
    }
}
```

### 5. 原始碼映射配置

設定生產除錯的原始碼映射支援：

**原始碼映射設定**
```javascript
// webpack.config.js
module.exports = {
    mode: 'production',
    devtool: 'hidden-source-map', // 生成原始碼映射但不引用它們
    
    output: {
        filename: '[name].[contenthash].js',
        sourceMapFilename: 'sourcemaps/[name].[contenthash].js.map',
    },
    
    plugins: [
        // 將原始碼映射上傳到錯誤追蹤服務
        new SentryWebpackPlugin({
            authToken: process.env.SENTRY_AUTH_TOKEN,
            org: 'your-org',
            project: 'your-project',
            include: './dist',
            ignore: ['node_modules'],
            urlPrefix: '~/',
            release: process.env.RELEASE_VERSION,
            deleteAfterCompile: true,
        }),
    ],
};

// 運行時原始碼映射支援
require('source-map-support').install({
    environment: 'node',
    handleUncaughtExceptions: false,
    retrieveSourceMap(source) {
        // 生產環境的自定義原始碼映射檢索
        if (process.env.NODE_ENV === 'production') {
            const sourceMapUrl = getSourceMapUrl(source);
            if (sourceMapUrl) {
                const map = fetchSourceMap(sourceMapUrl);
                return {
                    url: source,
                    map: map,
                };
            }
        }
        return null;
    },
});

// 堆疊追蹤增強
Error.prepareStackTrace = (error, stack) => {
    const mapped = stack.map(frame => {
        const fileName = frame.getFileName();
        const lineNumber = frame.getLineNumber();
        const columnNumber = frame.getColumnNumber();
        
        // 嘗試獲取原始位置
        const original = getOriginalPosition(fileName, lineNumber, columnNumber);
        
        return {
            function: frame.getFunctionName() || '<anonymous>',
            file: original?.source || fileName,
            line: original?.line || lineNumber,
            column: original?.column || columnNumber,
            native: frame.isNative(),
            async: frame.isAsync(),
        };
    });
    
    return {
        message: error.message,
        stack: mapped,
    };
};
```

### 6. 性能分析

實施性能分析工具：

**性能分析器**
```javascript
// performance-profiler.js
const v8Profiler = require('v8-profiler-next');
const fs = require('fs');
const path = require('path');

class PerformanceProfiler {
    constructor(options = {}) {
        this.outputDir = options.outputDir || './profiles';
        this.profiles = new Map();
        
        // 確保輸出目錄存在
        if (!fs.existsSync(this.outputDir)) {
            fs.mkdirSync(this.outputDir, { recursive: true });
        }
    }
    
    startCPUProfile(id, options = {}) {
        const title = options.title || `cpu-profile-${id}`;
        v8Profiler.startProfiling(title, true);
        
        this.profiles.set(id, {
            type: 'cpu',
            title,
            startTime: Date.now(),
        });
        
        return id;
    }
    
    stopCPUProfile(id) {
        const profileInfo = this.profiles.get(id);
        if (!profileInfo || profileInfo.type !== 'cpu') {
            throw new Error(`未找到 CPU 分析 ${id}`);
        }
        
        const profile = v8Profiler.stopProfiling(profileInfo.title);
        const duration = Date.now() - profileInfo.startTime;
        
        // 匯出分析
        const fileName = `${profileInfo.title}-${Date.now()}.cpuprofile`;
        const filePath = path.join(this.outputDir, fileName);
        
        profile.export((error, result) => {
            if (!error) {
                fs.writeFileSync(filePath, result);
                console.log(`CPU 分析已儲存到 ${filePath}`);
            }
            profile.delete();
        });
        
        this.profiles.delete(id);
        
        return {
            id,
            duration,
            filePath,
        };
    }
    
    takeHeapSnapshot(tag = '') {
        const fileName = `heap-${tag}-${Date.now()}.heapsnapshot`;
        const filePath = path.join(this.outputDir, fileName);
        
        const snapshot = v8Profiler.takeSnapshot();
        
        // 匯出快照
        snapshot.export((error, result) => {
            if (!error) {
                fs.writeFileSync(filePath, result);
                console.log(`堆記憶體快照已儲存到 ${filePath}`);
            }
            snapshot.delete();
        });
        
        return filePath;
    }
    
    measureFunction(fn, name = 'anonymous') {
        const measurements = {
            name,
            executions: 0,
            totalTime: 0,
            minTime: Infinity,
            maxTime: 0,
            avgTime: 0,
            lastExecution: null,
        };
        
        return new Proxy(fn, {
            apply(target, thisArg, args) {
                const start = process.hrtime.bigint();
                
                try {
                    const result = target.apply(thisArg, args);
                    
                    if (result instanceof Promise) {
                        return result.finally(() => {
                            this.recordExecution(start);
                        });
                    }
                    
                    this.recordExecution(start);
                    return result;
                } catch (error) {
                    this.recordExecution(start);
                    throw error;
                }
            },
            
            recordExecution(start) {
                const end = process.hrtime.bigint();
                const duration = Number(end - start) / 1000000; // 轉換為毫秒
                
                measurements.executions++;
                measurements.totalTime += duration;
                measurements.minTime = Math.min(measurements.minTime, duration);
                measurements.maxTime = Math.max(measurements.maxTime, duration);
                measurements.avgTime = measurements.totalTime / measurements.executions;
                measurements.lastExecution = new Date();
                
                // 記錄慢速執行
                if (duration > 100) {
                    console.warn(`慢速函數執行: ${name} 花費 ${duration} 毫秒`);
                }
            },
            
            get(target, prop) {
                if (prop === 'measurements') {
                    return measurements;
                }
                return target[prop];
            },
        });
    }
}

// 記憶體洩漏檢測器
class MemoryLeakDetector {
    constructor() {
        this.snapshots = [];
        this.threshold = 50 * 1024 * 1024; // 50MB
    }
    
    start(interval = 60000) {
        this.interval = setInterval(() => {
            this.checkMemory();
        }, interval);
    }
    
    checkMemory() {
        const usage = process.memoryUsage();
        const snapshot = {
            timestamp: Date.now(),
            heapUsed: usage.heapUsed,
            external: usage.external,
            rss: usage.rss,
        };
        
        this.snapshots.push(snapshot);
        
        // 只保留最近 10 個快照
        if (this.snapshots.length > 10) {
            this.snapshots.shift();
        }
        
        // 檢查記憶體洩漏模式
        if (this.snapshots.length >= 5) {
            const trend = this.calculateTrend();
            if (trend.increasing && trend.delta > this.threshold) {
                console.error('檢測到潛在記憶體洩漏!', {
                    trend,
                    current: snapshot,
                });
                
                // 拍攝堆記憶體快照以進行分析
                const profiler = new PerformanceProfiler();
                profiler.takeHeapSnapshot('洩漏檢測');
            }
        }
    }
    
    calculateTrend() {
        const recent = this.snapshots.slice(-5);
        const first = recent[0];
        const last = recent[recent.length - 1];
        
        const delta = last.heapUsed - first.heapUsed;
        const increasing = recent.every((s, i) => 
            i === 0 || s.heapUsed > recent[i - 1].heapUsed
        );
        
        return {
            increasing,
            delta,
            rate: delta / (last.timestamp - first.timestamp) * 1000 * 60, // MB 每分鐘
        };
    }
}
```

### 7. 除錯配置管理

集中除錯配置：

**除錯配置**
```javascript
// debug-config.js
class DebugConfiguration {
    constructor() {
        this.config = {
            // 除錯級別
            levels: {
                error: 0,
                warn: 1,
                info: 2,
                debug: 3,
                trace: 4,
            },
            
            // 功能標誌
            features: {
                remoteDebugging: process.env.ENABLE_REMOTE_DEBUG === 'true',
                tracing: process.env.ENABLE_TRACING === 'true',
                profiling: process.env.ENABLE_PROFILING === 'true',
                memoryMonitoring: process.env.ENABLE_MEMORY_MONITORING === 'true',
            },
            
            // 除錯端點
            endpoints: {
                jaeger: process.env.JAEGER_ENDPOINT || 'http://localhost:14268',
                elasticsearch: process.env.ELASTICSEARCH_URL || 'http://localhost:9200',
                sentry: process.env.SENTRY_DSN,
            },
            
            // 採樣率
            sampling: {
                traces: parseFloat(process.env.TRACE_SAMPLING_RATE || '0.1'),
                profiles: parseFloat(process.env.PROFILE_SAMPLING_RATE || '0.01'),
                logs: parseFloat(process.env.LOG_SAMPLING_RATE || '1.0'),
            },
        };
    }
    
    isEnabled(feature) {
        return this.config.features[feature] || false;
    }
    
    getLevel() {
        const level = process.env.DEBUG_LEVEL || 'info';
        return this.config.levels[level] || 2;
    }
    
    shouldSample(type) {
        const rate = this.config.sampling[type] || 1.0;
        return Math.random() < rate;
    }
}

// 除錯中介軟體工廠
class DebugMiddlewareFactory {
    static create(app, config) {
        const middlewares = [];
        
        if (config.isEnabled('tracing')) {
            const tracingMiddleware = new TracingMiddleware();
            middlewares.push(tracingMiddleware.express());
        }
        
        if (config.isEnabled('profiling')) {
            middlewares.push(this.profilingMiddleware());
        }
        
        if (config.isEnabled('memoryMonitoring')) {
            const detector = new MemoryLeakDetector();
            detector.start();
        }
        
        // 除錯路由
        if (process.env.NODE_ENV === 'development') {
            app.get('/debug/heap', (req, res) => {
                const profiler = new PerformanceProfiler();
                const path = profiler.takeHeapSnapshot('manual');
                res.json({ heapSnapshot: path });
            });
            
            app.get('/debug/profile', async (req, res) => {
                const profiler = new PerformanceProfiler();
                const id = profiler.startCPUProfile('manual');
                
                setTimeout(() => {
                    const result = profiler.stopCPUProfile(id);
                    res.json(result);
                }, 10000);
            });
            
            app.get('/debug/metrics', (req, res) => {
                res.json({
                    memory: process.memoryUsage(),
                    cpu: process.cpuUsage(),
                    uptime: process.uptime(),
                });
            });
        }
        
        return middlewares;
    }
    
    static profilingMiddleware() {
        const profiler = new PerformanceProfiler();
        
        return (req, res, next) => {
            if (Math.random() < 0.01) { // 1% 採樣
                const id = profiler.startCPUProfile(`請求-${Date.now()}`);
                
                res.on('finish', () => {
                    profiler.stopCPUProfile(id);
                });
            }
            
            next();
        };
    }
}
```

### 8. 生產除錯

啟用安全生產除錯：

**生產除錯工具**
```javascript
// production-debug.js
class ProductionDebugger {
    constructor(options = {}) {
        this.enabled = process.env.PRODUCTION_DEBUG === 'true';
        this.authToken = process.env.DEBUG_AUTH_TOKEN;
        this.allowedIPs = (process.env.DEBUG_ALLOWED_IPS || '').split(',');
    }
    
    middleware() {
        return (req, res, next) => {
            if (!this.enabled) {
                return next();
            }
            
            // 檢查授權
            const token = req.headers['x-debug-token'];
            const ip = req.ip || req.connection.remoteAddress;
            
            if (token !== this.authToken || !this.allowedIPs.includes(ip)) {
                return next();
            }
            
            // 添加除錯標頭
            res.setHeader('X-Debug-Enabled', 'true');
            
            // 為此請求啟用除錯模式
            req.debugMode = true;
            req.debugContext = new DebugContext().create(req.id);
            
            // 覆寫此請求的控制台
            const originalConsole = { ...console };
            ['log', 'debug', 'info', 'warn', 'error'].forEach(method => {
                console[method] = (...args) => {
                    req.debugContext.log(req.id, method, args[0], args.slice(1));
                    originalConsole[method](...args);
                };
            });
            
            // 在響應時恢復控制台
            res.on('finish', () => {
                Object.assign(console, originalConsole);
                
                // 如果請求，發送除錯資訊
                if (req.headers['x-debug-response'] === 'true') {
                    const debugInfo = req.debugContext.export(req.id);
                    res.setHeader('X-Debug-Info', JSON.stringify(debugInfo));
                }
            });
            
            next();
        };
    }
}

// 生產環境中的條件斷點
class ConditionalBreakpoint {
    constructor(condition, callback) {
        this.condition = condition;
        this.callback = callback;
        this.hits = 0;
    }
    
    check(context) {
        if (this.condition(context)) {
            this.hits++;
            
            // 記錄斷點命中
            console.debug('條件斷點命中', {
                condition: this.condition.toString(),
                hits: this.hits,
                context,
            });
            
            // 執行回調
            if (this.callback) {
                this.callback(context);
            }
            
            // 在生產環境中，實際上不要中斷
            if (process.env.NODE_ENV === 'production') {
                // 而是拍攝快照
                const profiler = new PerformanceProfiler();
                profiler.takeHeapSnapshot(`斷點-${Date.now()}`);
            } else {
                // 在開發環境中，使用除錯器
                debugger;
            }
        }
    }
}

// 用法
const breakpoints = new Map();

// 設定條件斷點
breakpoints.set('高記憶體', new ConditionalBreakpoint(
    (context) => context.memoryUsage > 500 * 1024 * 1024, // 500MB
    (context) => {
        console.error('檢測到高記憶體使用量', context);
        // 發送警報
        alerting.send('高記憶體', context);
    }
));

// 在程式碼中檢查斷點
function checkBreakpoints(context) {
    breakpoints.forEach(breakpoint => {
        breakpoint.check(context);
    });
}
```

### 9. 除錯儀表板

建立用於監控的除錯儀表板：

**除錯儀表板**
```html
<!-- debug-dashboard.html -->
<!DOCTYPE html>
<html>
<head>
    <title>除錯儀表板</title>
    <style>
        body { font-family: monospace; background: #1e1e1e; color: #d4d4d4; }
        .container { max-width: 1200px; margin: 0 auto; padding: 20px; }
        .metric { background: #252526; padding: 15px; margin: 10px 0; border-radius: 5px; }
        .metric h3 { margin: 0 0 10px 0; color: #569cd6; }
        .chart { height: 200px; background: #1e1e1e; margin: 10px 0; }
        .log-entry { padding: 5px; border-bottom: 1px solid #3e3e3e; }
        .error { color: #f44747; }
        .warn { color: #ff9800; }
        .info { color: #4fc3f7; }
        .debug { color: #4caf50; }
    </style>
</head>
<body>
    <div class="container">
        <h1>除錯儀表板</h1>
        
        <div class="metric">
            <h3>系統指標</h3>
            <div id="metrics"></div>
        </div>
        
        <div class="metric">
            <h3>記憶體使用量</h3>
            <canvas id="memoryChart" class="chart"></canvas>
        </div>
        
        <div class="metric">
            <h3>請求追蹤</h3>
            <div id="traces"></div>
        </div>
        
        <div class="metric">
            <h3>除錯日誌</h3>
            <div id="logs"></div>
        </div>
    </div>
    
    <script>
        // WebSocket 連接用於即時更新
        const ws = new WebSocket('ws://localhost:9231/debug');
        
        ws.onmessage = (event) => {
            const data = JSON.parse(event.data);
            
            switch (data.type) {
                case 'metrics':
                    updateMetrics(data.payload);
                    break;
                case 'trace':
                    addTrace(data.payload);
                    break;
                case 'log':
                    addLog(data.payload);
                    break;
            }
        };
        
        function updateMetrics(metrics) {
            const container = document.getElementById('metrics');
            container.innerHTML = `
                <div>CPU: ${metrics.cpu.percent}%</div>
                <div>記憶體: ${metrics.memory.used}MB / ${metrics.memory.total}MB</div>
                <div>運行時間: ${metrics.uptime}s</div>
                <div>活躍請求: ${metrics.activeRequests}</div>
            `;
        }
        
        function addTrace(trace) {
            const container = document.getElementById('traces');
            const entry = document.createElement('div');
            entry.className = 'log-entry';
            entry.innerHTML = `
                <span>${trace.timestamp}</span>
                <span>${trace.method} ${trace.path}</span>
                <span>${trace.duration}ms</span>
                <span>${trace.status}</span>
            `;
            container.insertBefore(entry, container.firstChild);
        }
        
        function addLog(log) {
            const container = document.getElementById('logs');
            const entry = document.createElement('div');
            entry.className = `log-entry ${log.level}`;
            entry.innerHTML = `
                <span>${log.timestamp}</span>
                <span>[${log.level.toUpperCase()}]</span>
                <span>${log.message}</span>
            `;
            container.insertBefore(entry, container.firstChild);
            
            // 只保留最近 100 條日誌
            while (container.children.length > 100) {
                container.removeChild(container.lastChild);
            }
        }
        
        // 記憶體使用量圖表
        const memoryChart = document.getElementById('memoryChart').getContext('2d');
        const memoryData = [];
        
        function updateMemoryChart(usage) {
            memoryData.push({
                time: new Date(),
                value: usage,
            });
            
            // 保留最近 50 個點
            if (memoryData.length > 50) {
                memoryData.shift();
            }
            
            // 繪製圖表
            // ...圖表繪製邏輯
        }
    </script>
</body>
</html>
```

### 10. IDE 整合

配置 IDE 除錯功能：

**IDE 除錯擴展**
```json
// .vscode/extensions.json
{
    "recommendations": [
        "ms-vscode.vscode-js-debug",
        "msjsdiag.debugger-for-chrome",
        "ms-vscode.vscode-typescript-tslint-plugin",
        "dbaeumer.vscode-eslint",
        "ms-azuretools.vscode-docker",
        "humao.rest-client",
        "eamodio.gitlens",
        "usernamehw.errorlens",
        "wayou.vscode-todo-highlight",
        "formulahendry.code-runner"
    ]
}

// .vscode/tasks.json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "啟動除錯伺服器",
            "type": "npm",
            "script": "debug",
            "problemMatcher": [],
            "presentation": {
                "reveal": "always",
                "panel": "dedicated"
            }
        },
        {
            "label": "分析應用程式",
            "type": "shell",
            "command": "node --inspect-brk --cpu-prof --cpu-prof-dir=./profiles ${workspaceFolder}/src/index.js",
            "problemMatcher": []
        },
        {
            "label": "記憶體快照",
            "type": "shell",
            "command": "node --inspect --expose-gc ${workspaceFolder}/scripts/heap-snapshot.js",
            "problemMatcher": []
        }
    ]
}
```

## 輸出格式

1. **除錯配置**：所有除錯工具的完整設定
2. **整合指南**：逐步整合說明
3. **故障排除手冊**：常見除錯場景和解決方案
4. **性能基準**：用於比較的指標
5. **除錯腳本**：自動化除錯工具
6. **儀表板設定**：即時除錯介面
7. **文件**：團隊除錯指南
8. **緊急程序**：生產除錯協議

專注於建立全面的除錯環境，以提高開發人員生產力並在所有環境中實現快速問題解決。
