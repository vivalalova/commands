# 配置驗證

您是配置管理專家，專精於驗證、測試和確保應用程式配置的正確性。建立全面的驗證模式、實施配置測試策略，並確保所有環境中的配置安全、一致且無錯誤。

## 背景
使用者需要驗證配置文件、實施配置模式、確保環境之間的一致性，並防止與配置相關的錯誤。專注於建立強大的驗證規則、類型安全、安全檢查和自動化驗證流程。

## 要求
$ARGUMENTS

## 指示

### 1. 配置分析

分析現有配置結構並識別驗證需求：

**配置掃描器**
```python
import os
import yaml
import json
import toml
import configparser
from pathlib import Path
from typing import Dict, List, Any, Set

class ConfigurationAnalyzer:
    def analyze_project(self, project_path: str) -> Dict[str, Any]:
        """
        分析專案配置文件和模式
        """
        analysis = {
            'config_files': self._find_config_files(project_path),
            'config_patterns': self._identify_patterns(project_path),
            'security_issues': self._check_security_issues(project_path),
            'consistency_issues': self._check_consistency(project_path),
            'validation_coverage': self._assess_validation(project_path),
            'recommendations': []
        }
        
        self._generate_recommendations(analysis)
        return analysis
    
    def _find_config_files(self, project_path: str) -> List[Dict]:
        """在專案中尋找所有配置文件"""
        config_patterns = [
            '**/*.json', '**/*.yaml', '**/*.yml', '**/*.toml',
            '**/*.ini', '**/*.conf', '**/*.config', '**/*.env*', 
            '**/*.properties', '**/config.js', '**/config.ts'
        ]
        
        config_files = []
        for pattern in config_patterns:
            for file_path in Path(project_path).glob(pattern):
                if not self._should_ignore(file_path):
                    config_files.append({
                        'path': str(file_path),
                        'type': self._detect_config_type(file_path),
                        'size': file_path.stat().st_size,
                        'environment': self._detect_environment(file_path)
                    })
        
        return config_files
    
    def _check_security_issues(self, project_path: str) -> List[Dict]:
        """檢查配置中的安全問題"""
        issues = []
        
        # 可能指示秘密的模式
        secret_patterns = [
            r'(api[_-]?key|apikey)',
            r'(secret|password|passwd|pwd)',
            r'(token|auth)',
            r'(private[_-]?key)',
            r'(aws[_-]?access|aws[_-]?secret)',
            r'(database[_-]?url|db[_-]?connection)'
        ]
        
        for config_file in self._find_config_files(project_path):
            content = Path(config_file['path']).read_text()
            
            for pattern in secret_patterns:
                if re.search(pattern, content, re.IGNORECASE):
                    # 檢查是否為佔位符或實際秘密
                    if self._looks_like_real_secret(content, pattern):
                        issues.append({
                            'file': config_file['path'],
                            'type': '潛在秘密',
                            'pattern': pattern,
                            'severity': '高'
                        })
        
        return issues
    
    def _check_consistency(self, project_path: str) -> List[Dict]:
        """檢查環境之間的配置一致性"""
        inconsistencies = []
        
        # 按基本名稱分組配置
        config_groups = defaultdict(list)
        for config in self._find_config_files(project_path):
            base_name = self._get_base_config_name(config['path'])
            config_groups[base_name].append(config)
        
        # 檢查每個組的矛盾之處
        for base_name, configs in config_groups.items():
            if len(configs) > 1:
                keys_by_env = {}
                for config in configs:
                    env = config.get('environment', 'default')
                    keys = self._extract_config_keys(config['path'])
                    keys_by_env[env] = keys
                
                # 尋找缺少的鍵
                all_keys = set()
                for keys in keys_by_env.values():
                    all_keys.update(keys)
                
                for env, keys in keys_by_env.items():
                    missing = all_keys - keys
                    if missing:
                        inconsistencies.append({
                            'config_group': base_name,
                            'environment': env,
                            'missing_keys': list(missing),
                            'severity': '中'
                        })
        
        return inconsistencies
```

### 2. 模式定義與驗證

實施配置模式驗證：

**JSON 模式驗證器**
```typescript
// config-validator.ts
import Ajv from 'ajv';
import ajvFormats from 'ajv-formats';
import ajvKeywords from 'ajv-keywords';
import { JSONSchema7 } from 'json-schema';

interface ValidationResult {
  valid: boolean;
  errors?: Array<{ 
    path: string;
    message: string;
    keyword: string;
    params: any;
  }>;
}

export class ConfigValidator {
  private ajv: Ajv;
  private schemas: Map<string, JSONSchema7> = new Map();
  
  constructor() {
    this.ajv = new Ajv({
      allErrors: true,
      verbose: true,
      strict: false,
      coerceTypes: true
    });
    
    // 添加格式支援
    ajvFormats(this.ajv);
    ajvKeywords(this.ajv);
    
    // 添加自定義格式
    this.addCustomFormats();
  }
  
  private addCustomFormats() {
    // 帶有協議驗證的 URL 格式
    this.ajv.addFormat('url-https', {
      type: 'string',
      validate: (data: string) => {
        try {
          const url = new URL(data);
          return url.protocol === 'https:';
        } catch {
          return false;
        }
      }
    });
    
    // 環境變數引用
    this.ajv.addFormat('env-var', {
      type: 'string',
      validate: /^\$\{[A-Z_][A-Z0-9_]*\}$/
    });
    
    // 語義版本
    this.ajv.addFormat('semver', {
      type: 'string',
      validate: /^\d+\.\d+\.\d+(-[0-9A-Za-z-]+(\.[0-9A-Za-z-]+)*)?$/
    });
    
    // 埠號
    this.ajv.addFormat('port', {
      type: 'number',
      validate: (data: number) => data >= 1 && data <= 65535
    });
    
    // 持續時間格式（例如，「5m」、「1h」、「30s」）
    this.ajv.addFormat('duration', {
      type: 'string',
      validate: /^\d+[smhd]$/
    });
  }
  
  registerSchema(name: string, schema: JSONSchema7): void {
    this.schemas.set(name, schema);
    this.ajv.addSchema(schema, name);
  }
  
  validate(configData: any, schemaName: string): ValidationResult {
    const validate = this.ajv.getSchema(schemaName);
    
    if (!validate) {
      throw new Error(`未找到模式 '${schemaName}'`);
    }
    
    const valid = validate(configData);
    
    if (!valid && validate.errors) {
      return {
        valid: false,
        errors: validate.errors.map(error => ({
          path: error.instancePath || '/',
          message: error.message || '驗證錯誤',
          keyword: error.keyword,
          params: error.params
        }))
      };
    }
    
    return { valid: true };
  }
  
  generateSchema(sampleConfig: any): JSONSchema7 {
    // 從範例配置自動生成模式
    const schema: JSONSchema7 = {
      type: 'object',
      properties: {},
      required: []
    };
    
    for (const [key, value] of Object.entries(sampleConfig)) {
      schema.properties![key] = this.inferSchema(value);
      
      // 預設情況下，所有頂層屬性都是必需的
      if (schema.required && !key.startsWith('optional_')) {
        schema.required.push(key);
      }
    }
    
    return schema;
  }
  
  private inferSchema(value: any): JSONSchema7 {
    if (value === null) {
      return { type: 'null' };
    }
    
    if (Array.isArray(value)) {
      return {
        type: 'array',
        items: value.length > 0 ? this.inferSchema(value[0]) : {}
      };
    }
    
    if (typeof value === 'object') {
      const properties: Record<string, JSONSchema7> = {};
      const required: string[] = [];
      
      for (const [k, v] of Object.entries(value)) {
        properties[k] = this.inferSchema(v);
        if (v !== null && v !== undefined) {
          required.push(k);
        }
      }
      
      return {
        type: 'object',
        properties,
        required
      };
    }
    
    // 從值模式推斷格式
    if (typeof value === 'string') {
      if (value.match(/^https?:\/\/ /)) {
        return { type: 'string', format: 'uri' };
      }
      if (value.match(/^\d{4}-\d{2}-\d{2}$/)) {
        return { type: 'string', format: 'date' };
      }
      if (value.match(/^[a-f0-9]{8}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{4}-[a-f0-9]{12}$/i)) {
        return { type: 'string', format: 'uuid' };
      }
    }
    
    return { type: typeof value as JSONSchema7['type'] };
  }
}

// 範例模式
export const schemas = {
  database: {
    type: 'object',
    properties: {
      host: { type: 'string', format: 'hostname' },
      port: { type: 'integer', format: 'port' },
      database: { type: 'string', minLength: 1 },
      user: { type: 'string', minLength: 1 },
      password: { type: 'string', minLength: 8 },
      ssl: {
        type: 'object',
        properties: {
          enabled: { type: 'boolean' },
          ca: { type: 'string' },
          cert: { type: 'string' },
          key: { type: 'string' }
        },
        required: ['enabled']
      },
      pool: {
        type: 'object',
        properties: {
          min: { type: 'integer', minimum: 0 },
          max: { type: 'integer', minimum: 1 },
          idleTimeout: { type: 'string', format: 'duration' }
        }
      }
    },
    required: ['host', 'port', 'database', 'user', 'password'],
    additionalProperties: false
  },
  
  api: {
    type: 'object',
    properties: {
      server: {
        type: 'object',
        properties: {
          host: { type: 'string', default: '0.0.0.0' },
          port: { type: 'integer', format: 'port', default: 3000 },
          cors: {
            type: 'object',
            properties: {
              enabled: { type: 'boolean' },
              origins: {
                type: 'array',
                items: { type: 'string', format: 'uri' }
              },
              credentials: { type: 'boolean' }
            }
          }
        },
        required: ['port']
      },
      auth: {
        type: 'object',
        properties: {
          jwt: {
            type: 'object',
            properties: {
              secret: { type: 'string', minLength: 32 },
              expiresIn: { type: 'string', format: 'duration' },
              algorithm: {
                type: 'string',
                enum: ['HS256', 'HS384', 'HS512', 'RS256', 'RS384', 'RS512']
              }
            },
            required: ['secret', 'expiresIn']
          }
        }
      },
      rateLimit: {
        type: 'object',
        properties: {
          windowMs: { type: 'integer', minimum: 1000 },
          max: { type: 'integer', minimum: 1 },
          message: { type: 'string' }
        }
      }
    },
    required: ['server', 'auth']
  }
};
```

### 3. 環境特定驗證

驗證環境之間的配置：

**環境驗證器**
```python
# environment_validator.py
from typing import Dict, List, Set, Any
import os
import re

class EnvironmentValidator:
    def __init__(self):
        self.environments = ['development', 'staging', 'production']
        self.environment_rules = self._define_environment_rules()
    
    def _define_environment_rules(self) -> Dict[str, Dict]:
        """定義環境特定的驗證規則"""
        return {
            'development': {
                'allow_debug': True,
                'require_https': False,
                'allow_wildcards': True,
                'min_password_length': 8,
                'allowed_log_levels': ['debug', 'info', 'warn', 'error']
            },
            'staging': {
                'allow_debug': True,
                'require_https': True,
                'allow_wildcards': False,
                'min_password_length': 12,
                'allowed_log_levels': ['info', 'warn', 'error']
            },
            'production': {
                'allow_debug': False,
                'require_https': True,
                'allow_wildcards': False,
                'min_password_length': 16,
                'allowed_log_levels': ['warn', 'error'],
                'require_encryption': True,
                'require_backup': True
            }
        }
    
    def validate_config(self, config: Dict, environment: str) -> List[Dict]:
        """驗證特定環境的配置"""
        if environment not in self.environment_rules:
            raise ValueError(f"未知環境: {environment}")
        
        rules = self.environment_rules[environment]
        violations = []
        
        # 檢查除錯設定
        if not rules['allow_debug'] and config.get('debug', False):
            violations.append({
                'rule': '不允許在生產環境中除錯',
                'message': '生產環境中不允許除錯模式',
                'severity': '關鍵',
                'path': 'debug'
            })
        
        # 檢查 HTTPS 要求
        if rules['require_https']:
            urls = self._extract_urls(config)
            for url_path, url in urls:
                if url.startswith('http://') and 'localhost' not in url:
                    violations.append({
                        'rule': '需要 HTTPS',
                        'message': f'{url_path} 需要 HTTPS',
                        'severity': '高',
                        'path': url_path,
                        'value': url
                    })
        
        # 檢查日誌級別
        log_level = config.get('logging', {}).get('level')
        if log_level and log_level not in rules['allowed_log_levels']:
            violations.append({
                'rule': '無效日誌級別',
                'message': f"日誌級別 '{log_level}' 在 {environment} 中不允許",
                'severity': '中',
                'path': 'logging.level',
                'allowed': rules['allowed_log_levels']
            })
        
        # 檢查生產特定要求
        if environment == 'production':
            violations.extend(self._validate_production_requirements(config))
        
        return violations
    
    def _validate_production_requirements(self, config: Dict) -> List[Dict]:
        """生產環境的額外驗證"""
        violations = []
        
        # 檢查加密設定
        if not config.get('security', {}).get('encryption', {}).get('enabled'):
            violations.append({
                'rule': '需要加密',
                'message': '生產環境中必須啟用加密',
                'severity': '關鍵',
                'path': 'security.encryption.enabled'
            })
        
        # 檢查備份配置
        if not config.get('backup', {}).get('enabled'):
            violations.append({
                'rule': '需要備份',
                'message': '生產環境中必須配置備份',
                'severity': '高',
                'path': 'backup.enabled'
            })
        
        # 檢查監控
        if not config.get('monitoring', {}).get('enabled'):
            violations.append({
                'rule': '需要監控',
                'message': '生產環境中必須啟用監控',
                'severity': '高',
                'path': 'monitoring.enabled'
            })
        
        return violations
    
    def _extract_urls(self, obj: Any, path: str = '') -> List[tuple]:
        """從配置中遞歸提取 URL"""
        urls = []
        
        if isinstance(obj, dict):
            for key, value in obj.items():
                new_path = f"{path}.{key}" if path else key
                urls.extend(self._extract_urls(value, new_path))
        elif isinstance(obj, list):
            for i, item in enumerate(obj):
                new_path = f"{path}[{i}]"
                urls.extend(self._extract_urls(item, new_path))
        elif isinstance(obj, str) and re.match(r'^https?://', obj):
            urls.append((path, obj))
        
        return urls

# 跨環境一致性檢查器
class ConsistencyChecker:
    def check_consistency(self, configs: Dict[str, Dict]) -> List[Dict]:
        """檢查環境之間的配置一致性"""
        issues = []
        
        # 獲取所有環境中的所有唯一鍵
        all_keys = set()
        env_keys = {}
        
        for env, config in configs.items():
            keys = self._flatten_keys(config)
            env_keys[env] = keys
            all_keys.update(keys)
        
        # 檢查缺少的鍵
        for env, keys in env_keys.items():
            missing_keys = all_keys - keys
            if missing_keys:
                issues.append({
                    'type': '缺少鍵',
                    'environment': env,
                    'keys': list(missing_keys),
                    'severity': '中'
                })
        
        # 檢查類型不一致
        for key in all_keys:
            types_by_env = {}
            for env, config in configs.items():
                value = self._get_nested_value(config, key)
                if value is not None:
                    types_by_env[env] = type(value).__name__
            
            unique_types = set(types_by_env.values())
            if len(unique_types) > 1:
                issues.append({
                    'type': '類型不匹配',
                    'key': key,
                    'types': types_by_env,
                    'severity': '高'
                })
        
        return issues
```

### 4. 配置測試框架

實施配置測試：

**配置測試套件**
```typescript
// config-test.ts
import { describe, it, expect, beforeEach } from '@jest/globals';
import { ConfigValidator } from './config-validator';
import { loadConfig } from './config-loader';

interface ConfigTestCase {
  name: string;
  config: any;
  environment: string;
  expectedValid: boolean;
  expectedErrors?: string[];
}

export class ConfigTestSuite {
  private validator: ConfigValidator;
  
  constructor() {
    this.validator = new ConfigValidator();
  }
  
  async runTests(testCases: ConfigTestCase[]): Promise<TestResults> {
    const results: TestResults = {
      passed: 0,
      failed: 0,
      errors: []
    };
    
    for (const testCase of testCases) {
      try {
        const result = await this.runTestCase(testCase);
        if (result.passed) {
          results.passed++;
        } else {
          results.failed++;
          results.errors.push({
            testName: testCase.name,
            errors: result.errors
          });
        }
      } catch (error) {
        results.failed++;
        results.errors.push({
          testName: testCase.name,
          errors: [error.message]
        });
      }
    }
    
    return results;
  }
  
  private async runTestCase(testCase: ConfigTestCase): Promise<TestResult> {
    // 載入並驗證配置
    const validationResult = this.validator.validate(
      testCase.config,
      testCase.environment
    );
    
    const result: TestResult = {
      passed: validationResult.valid === testCase.expectedValid,
      errors: []
    };
    
    // 檢查預期錯誤
    if (testCase.expectedErrors && validationResult.errors) {
      for (const expectedError of testCase.expectedErrors) {
        const found = validationResult.errors.some(
          error => error.message.includes(expectedError)
        );
        
        if (!found) {
          result.passed = false;
          result.errors.push(`預期錯誤未找到: ${expectedError}`);
        }
      }
    }
    
    return result;
  }
}

// Jest 測試範例
describe('配置驗證', () => {
  let validator: ConfigValidator;
  
  beforeEach(() => {
    validator = new ConfigValidator();
  });
  
  describe('資料庫配置', () => {
    it('應驗證有效的資料庫配置', () => {
      const config = {
        host: 'localhost',
        port: 5432,
        database: 'myapp',
        user: 'dbuser',
        password: 'securepassword123'
      };
      
      const result = validator.validate(config, 'database');
      expect(result.valid).toBe(true);
    });
    
    it('應拒絕無效的埠號', () => {
      const config = {
        host: 'localhost',
        port: 70000, // 無效埠號
        database: 'myapp',
        user: 'dbuser',
        password: 'securepassword123'
      };
      
      const result = validator.validate(config, 'database');
      expect(result.valid).toBe(false);
      expect(result.errors?.[0].path).toBe('/port');
    });
    
    it('應在生產環境中要求 SSL', () => {
      const config = {
        host: 'prod-db.example.com',
        port: 5432,
        database: 'myapp',
        user: 'dbuser',
        password: 'securepassword123',
        ssl: { enabled: false }
      };
      
      const envValidator = new EnvironmentValidator();
      const violations = envValidator.validate_config(config, 'production');
      
      expect(violations).toContainEqual(
        expect.objectContaining({
          rule: 'production_ssl_required'
        })
      );
    });
  });
  
  describe('API 配置', () => {
    it('應驗證 CORS 設定', () => {
      const config = {
        server: {
          port: 3000,
          cors: {
            enabled: true,
            origins: ['https://example.com', 'https://app.example.com'],
            credentials: true
          }
        },
        auth: {
          jwt: {
            secret: 'a'.repeat(32),
            expiresIn: '1h',
            algorithm: 'HS256'
          }
        }
      };
      
      const result = validator.validate(config, 'api');
      expect(result.valid).toBe(true);
    });
    
    it('應拒絕短 JWT 秘密', () => {
      const config = {
        server: { port: 3000 },
        auth: {
          jwt: {
            secret: 'tooshort',
            expiresIn: '1h'
          }
        }
      };
      
      const result = validator.validate(config, 'api');
      expect(result.valid).toBe(false);
      expect(result.errors?.[0].path).toBe('/auth/jwt/secret');
    });
  });
});
```

### 5. 運行時配置驗證

實施運行時驗證和熱重載：

**運行時配置驗證器**
```typescript
// runtime-validator.ts
import { EventEmitter } from 'events';
import * as chokidar from 'chokidar';
import { ConfigValidator } from './config-validator';

export class RuntimeConfigValidator extends EventEmitter {
  private validator: ConfigValidator;
  private currentConfig: any;
  private watchers: Map<string, chokidar.FSWatcher> = new Map();
  private validationCache: Map<string, ValidationResult> = new Map();
  
  constructor() {
    super();
    this.validator = new ConfigValidator();
  }
  
  async initialize(configPath: string): Promise<void> {
    // 載入初始配置
    this.currentConfig = await this.loadAndValidate(configPath);
    
    // 設定檔案監控器以進行熱重載
    this.watchConfig(configPath);
  }
  
  private async loadAndValidate(configPath: string): Promise<any> {
    try {
      // 載入配置
      const config = await this.loadConfig(configPath);
      
      // 驗證配置
      const validationResult = this.validator.validate(
        config,
        this.detectEnvironment()
      );
      
      if (!validationResult.valid) {
        this.emit('validation:error', {
          path: configPath,
          errors: validationResult.errors
        });
        
        // 在開發環境中，記錄錯誤但繼續
        if (this.isDevelopment()) {
          console.error('配置驗證錯誤:', validationResult.errors);
          return config;
        }
        
        // 在生產環境中，拋出錯誤
        throw new ConfigValidationError(
          '配置驗證失敗',
          validationResult.errors
        );
      }
      
      this.emit('validation:success', { path: configPath });
      return config;
    } catch (error) {
      this.emit('validation:error', { path: configPath, error });
      throw error;
    }
  }
  
  private watchConfig(configPath: string): void {
    const watcher = chokidar.watch(configPath, {
      persistent: true,
      ignoreInitial: true
    });
    
    watcher.on('change', async () => {
      console.log(`配置文件已更改: ${configPath}`);
      
      try {
        const newConfig = await this.loadAndValidate(configPath);
        
        // 檢查配置是否實際更改
        if (JSON.stringify(newConfig) !== JSON.stringify(this.currentConfig)) {
          const oldConfig = this.currentConfig;
          this.currentConfig = newConfig;
          
          this.emit('config:changed', {
            oldConfig,
            newConfig,
            changedKeys: this.findChangedKeys(oldConfig, newConfig)
          });
        }
      } catch (error) {
        this.emit('config:error', { error });
      }
    });
    
    this.watchers.set(configPath, watcher);
  }
  
  private findChangedKeys(oldConfig: any, newConfig: any): string[] {
    const changed: string[] = [];
    
    const findDiff = (old: any, new_: any, path: string = '') => {
      // 檢查舊配置中的所有鍵
      for (const key in old) {
        const currentPath = path ? `${path}.${key}` : key;
        
        if (!(key in new_)) {
          changed.push(`${currentPath} (已移除)`);
        } else if (typeof old[key] === 'object' && typeof new_[key] === 'object') {
          findDiff(old[key], new_[key], currentPath);
        } else if (old[key] !== new_[key]) {
          changed.push(currentPath);
        }
      }
      
      // 檢查新鍵
      for (const key in new_) {
        if (!(key in old)) {
          const currentPath = path ? `${path}.${key}` : key;
          changed.push(`${currentPath} (已添加)`);
        }
      }
    };
    
    findDiff(oldConfig, newConfig);
    return changed;
  }
  
  validateValue(path: string, value: any): ValidationResult {
    // 使用快取模式以提高性能
    const cacheKey = `${path}:${JSON.stringify(value)}`;
    if (this.validationCache.has(cacheKey)) {
      return this.validationCache.get(cacheKey)!;
    }
    
    // 提取特定路徑的模式
    const schema = this.getSchemaForPath(path);
    if (!schema) {
      return { valid: true }; // 此路徑未定義模式
    }
    
    const result = this.validator.validateValue(value, schema);
    this.validationCache.set(cacheKey, result);
    
    return result;
  }
  
  async shutdown(): Promise<void> {
    // 關閉所有監控器
    for (const watcher of this.watchers.values()) {
      await watcher.close();
    }
    
    this.watchers.clear();
    this.validationCache.clear();
  }
}

// 類型安全配置存取
export class TypedConfig<T> {
  constructor(
    private config: T,
    private validator: RuntimeConfigValidator
  ) {}
  
  get<K extends keyof T>(key: K): T[K] {
    const value = this.config[key];
    
    // 在開發環境中存取時驗證
    if (process.env.NODE_ENV === 'development') {
      const result = this.validator.validateValue(String(key), value);
      if (!result.valid) {
        console.warn(`鍵 ${String(key)} 的配置值無效:`, result.errors);
      }
    }
    
    return value;
  }
  
  getOrDefault<K extends keyof T>(key: K, defaultValue: T[K]): T[K] {
    return this.config[key] ?? defaultValue;
  }
  
  require<K extends keyof T>(key: K): NonNullable<T[K]> {
    const value = this.config[key];
    
    if (value === null || value === undefined) {
      throw new Error(`必需的配置 '${String(key)}' 缺失`);
    }
    
    return value as NonNullable<T[K]>;
  }
}
```

### 6. 配置遷移

實施配置遷移和版本控制：

**配置遷移系統**
```python
# config_migration.py
from typing import Dict, List, Callable, Any
from abc import ABC, abstractmethod
import semver

class ConfigMigration(ABC):
    """配置遷移的基類"""
    
    @property
    @abstractmethod
    def version(self) -> str:
        """此遷移的目標版本"""
        pass
    
    @property
    @abstractmethod
    def description(self) -> str:
        """此遷移的描述"""
        pass
    
    @abstractmethod
    def up(self, config: Dict) -> Dict:
        """將遷移應用於配置"""
        pass
    
    @abstractmethod
    def down(self, config: Dict) -> Dict:
        """從配置中恢復遷移"""
        pass
    
    def validate(self, config: Dict) -> bool:
        """遷移後驗證配置"""
        return True

class ConfigMigrator:
    def __init__(self):
        self.migrations: List[ConfigMigration] = []
    
    def register_migration(self, migration: ConfigMigration):
        """註冊遷移"""
        self.migrations.append(migration)
        # 按版本排序
        self.migrations.sort(key=lambda m: semver.VersionInfo.parse(m.version))
    
    def migrate(self, config: Dict, target_version: str) -> Dict:
        """將配置遷移到目標版本"""
        current_version = config.get('_version', '0.0.0')
        
        if semver.compare(current_version, target_version) == 0:
            return config  # 已在目標版本
        
        if semver.compare(current_version, target_version) > 0:
            # 降級
            return self._downgrade(config, current_version, target_version)
        else:
            # 升級
            return self._upgrade(config, current_version, target_version)
    
    def _upgrade(self, config: Dict, from_version: str, to_version: str) -> Dict:
        """將配置從一個版本升級到另一個版本"""
        result = config.copy()
        
        for migration in self.migrations:
            if (
                semver.compare(migration.version, from_version) > 0 and
                semver.compare(migration.version, to_version) <= 0
            ):
                
                print(f"正在應用遷移到 v{migration.version}: {migration.description}")
                result = migration.up(result)
                
                if not migration.validate(result):
                    raise ValueError(f"遷移到 v{migration.version} 驗證失敗")
                
                result['_version'] = migration.version
        
        return result
    
    def _downgrade(self, config: Dict, from_version: str, to_version: str) -> Dict:
        """將配置從一個版本降級到另一個版本"""
        result = config.copy()
        
        # 以相反順序應用遷移
        for migration in reversed(self.migrations):
            if (
                semver.compare(migration.version, to_version) > 0 and
                semver.compare(migration.version, from_version) <= 0
            ):
                
                print(f"正在恢復從 v{migration.version} 的遷移: {migration.description}")
                result = migration.down(result)
                
                # 將版本更新到上一個遷移的版本
                prev_version = self._get_previous_version(migration.version)
                result['_version'] = prev_version
        
        return result
    
    def _get_previous_version(self, version: str) -> str:
        """獲取給定版本之前的版本"""
        for i, migration in enumerate(self.migrations):
            if migration.version == version:
                return self.migrations[i-1].version if i > 0 else '0.0.0'
        return '0.0.0'

# 範例遷移
class MigrationV1_0_0(ConfigMigration):
    @property
    def version(self) -> str:
        return '1.0.0'
    
    @property
    def description(self) -> str:
        return '初始配置結構'
    
    def up(self, config: Dict) -> Dict:
        # 添加預設結構
        return {
            '_version': '1.0.0',
            'app': config.get('app', {}),
            'database': config.get('database', {}),
            'logging': config.get('logging', {'level': 'info'})
        }
    
    def down(self, config: Dict) -> Dict:
        # 移除版本資訊
        result = config.copy()
        result.pop('_version', None)
        return result

class MigrationV1_1_0(ConfigMigration):
    @property
    def version(self) -> str:
        return '1.1.0'
    
    @property
    def description(self) -> str:
        return '將資料庫配置拆分為讀/寫連接'
    
    def up(self, config: Dict) -> Dict:
        result = config.copy()
        
        # 將單一資料庫配置轉換為讀/寫拆分
        if 'database' in result and not isinstance(result['database'], dict):
            old_db = result['database']
            result['database'] = {
                'write': old_db,
                'read': old_db.copy()  # 最初與寫入相同
            }
        
        return result
    
    def down(self, config: Dict) -> Dict:
        result = config.copy()
        
        # 恢復為單一資料庫配置
        if 'database' in result and 'write' in result['database']:
            result['database'] = result['database']['write']
        
        return result

class MigrationV2_0_0(ConfigMigration):
    @property
    def version(self) -> str:
        return '2.0.0'
    
    @property
    def description(self) -> str:
        return '添加安全配置部分'
    
    def up(self, config: Dict) -> Dict:
        result = config.copy()
        
        # 添加帶有預設值的安全部分
        if 'security' not in result:
            result['security'] = {
                'encryption': {
                    'enabled': True,
                    'algorithm': 'AES-256-GCM'
                },
                'tls': {
                    'minVersion': '1.2',
                    'ciphers': ['TLS_AES_256_GCM_SHA384', 'TLS_AES_128_GCM_SHA256']
                }
            }
        
        return result
    
    def down(self, config: Dict) -> Dict:
        result = config.copy()
        result.pop('security', None)
        return result
```

### 7. 配置安全

實施安全配置處理：

**安全配置管理器**
```typescript
// secure-config.ts
import * as crypto from 'crypto';
import { SecretManagerServiceClient } from '@google-cloud/secret-manager';
import { KeyVaultSecret, SecretClient } from '@azure/keyvault-secrets';

interface EncryptedValue {
  encrypted: true;
  value: string;
  algorithm: string;
  iv: string;
  authTag?: string;
}

export class SecureConfigManager {
  private secretsCache: Map<string, any> = new Map();
  private encryptionKey: Buffer;
  
  constructor(private options: SecureConfigOptions) {
    this.encryptionKey = this.deriveKey(options.masterKey);
  }
  
  private deriveKey(masterKey: string): Buffer {
    return crypto.pbkdf2Sync(masterKey, 'config-salt', 100000, 32, 'sha256');
  }
  
  encrypt(value: any): EncryptedValue {
    const algorithm = 'aes-256-gcm';
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv(algorithm, this.encryptionKey, iv);
    
    const stringValue = JSON.stringify(value);
    let encrypted = cipher.update(stringValue, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    return {
      encrypted: true,
      value: encrypted,
      algorithm,
      iv: iv.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }
  
  decrypt(encryptedValue: EncryptedValue): any {
    const decipher = crypto.createDecipheriv(
      encryptedValue.algorithm,
      this.encryptionKey,
      Buffer.from(encryptedValue.iv, 'hex')
    );
    
    if (encryptedValue.authTag) {
      decipher.setAuthTag(Buffer.from(encryptedValue.authTag, 'hex'));
    }
    
    let decrypted = decipher.update(encryptedValue.value, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return JSON.parse(decrypted);
  }
  
  async processConfig(config: any): Promise<any> {
    const processed = {};
    
    for (const [key, value] of Object.entries(config)) {
      if (this.isEncryptedValue(value)) {
        // 解密加密值
        processed[key] = this.decrypt(value as EncryptedValue);
      } else if (this.isSecretReference(value)) {
        // 從秘密管理器獲取
        processed[key] = await this.fetchSecret(value as string);
      } else if (typeof value === 'object' && value !== null) {
        // 遞歸處理巢狀物件
        processed[key] = await this.processConfig(value);
      } else {
        processed[key] = value;
      }
    }
    
    return processed;
  }
  
  private isEncryptedValue(value: any): boolean {
    return typeof value === 'object' && 
           value !== null && 
           value.encrypted === true;
  }
  
  private isSecretReference(value: any): boolean {
    return typeof value === 'string' && 
           (value.startsWith('secret://') || 
            value.startsWith('vault://') ||
            value.startsWith('aws-secret://'));
  }
  
  private async fetchSecret(reference: string): Promise<any> {
    // 先檢查快取
    if (this.secretsCache.has(reference)) {
      return this.secretsCache.get(reference);
    }
    
    let secretValue: any;
    
    if (reference.startsWith('secret://')) {
      // Google Secret Manager
      secretValue = await this.fetchGoogleSecret(reference);
    } else if (reference.startsWith('vault://')) {
      // Azure Key Vault
      secretValue = await this.fetchAzureSecret(reference);
    } else if (reference.startsWith('aws-secret://')) {
      // AWS Secrets Manager
      secretValue = await this.fetchAWSSecret(reference);
    }
    
    // 快取秘密
    this.secretsCache.set(reference, secretValue);
    
    return secretValue;
  }
  
  private async fetchGoogleSecret(reference: string): Promise<any> {
    const secretName = reference.replace('secret://', '');
    const client = new SecretManagerServiceClient();
    
    const [version] = await client.accessSecretVersion({
      name: `projects/${this.options.gcpProject}/secrets/${secretName}/versions/latest`
    });
    
    const payload = version.payload?.data;
    if (!payload) {
      throw new Error(`秘密 ${secretName} 沒有有效負載`);
    }
    
    return JSON.parse(payload.toString());
  }
  
  validateSecureConfig(config: any): ValidationResult {
    const errors: string[] = [];
    
    const checkSecrets = (obj: any, path: string = '') => {
      for (const [key, value] of Object.entries(obj)) {
        const currentPath = path ? `${path}.${key}` : key;
        
        // 檢查純文字秘密
        if (this.looksLikeSecret(key) && typeof value === 'string') {
          if (!this.isEncryptedValue(value) && !this.isSecretReference(value)) {
            errors.push(`潛在的純文字秘密在 ${currentPath}`);
          }
        }
        
        // 遞歸檢查巢狀物件
        if (typeof value === 'object' && value !== null && !this.isEncryptedValue(value)) {
          checkSecrets(value, currentPath);
        }
      }
    };
    
    checkSecrets(config);
    
    return {
      valid: errors.length === 0,
      errors: errors.map(message => ({
        path: '',
        message,
        keyword: 'security',
        params: {}
      }))
    };
  }
  
  private looksLikeSecret(key: string): boolean {
    const secretPatterns = [
      'password', 'secret', 'key', 'token', 'credential',
      'api_key', 'apikey', 'private_key', 'auth'
    ];
    
    const lowerKey = key.toLowerCase();
    return secretPatterns.some(pattern => lowerKey.includes(pattern));
  }
}
```

### 8. 配置文件

生成配置文件：

**配置文檔生成器**
```python
# config_docs_generator.py
from typing import Dict, List, Any
import json
import yaml

class ConfigDocGenerator:
    def generate_docs(self, schema: Dict, examples: Dict) -> str:
        """生成全面的配置文檔"""
        docs = ["# 配置參考\n"]
        
        # 添加概述
        docs.append("## 概述\n")
        docs.append("本文檔描述了所有可用的配置選項。\n")
        
        # 添加目錄
        docs.append("## 目錄\n")
        toc = self._generate_toc(schema.get('properties', {}))
        docs.extend(toc)
        
        # 添加配置部分
        docs.append("\n## 配置選項\n")
        sections = self._generate_sections(schema.get('properties', {}), examples)
        docs.extend(sections)
        
        # 添加範例
        docs.append("\n## 完整範例\n")
        docs.extend(self._generate_examples(examples))
        
        # 添加驗證規則
        docs.append("\n## 驗證規則\n")
        docs.extend(self._generate_validation_rules(schema))
        
        return '\n'.join(docs)
    
    def _generate_sections(self, properties: Dict, examples: Dict, level: int = 3) -> List[str]:
        """為每個配置部分生成文檔"""
        sections = []
        
        for prop_name, prop_schema in properties.items():
            # 部分標題
            sections.append(f"{ '#' * level} {prop_name}\n")
            
            # 描述
            if 'description' in prop_schema:
                sections.append(f"{prop_schema['description']}\n")
            
            # 類型資訊
            sections.append(f"**類型:** `{prop_schema.get('type', 'any')}`\n")
            
            # 必需
            if prop_name in prop_schema.get('required', []):
                sections.append("**必需:** 是\n")
            
            # 預設值
            if 'default' in prop_schema:
                sections.append(f"**預設:** `{json.dumps(prop_schema['default'])}`\n")
            
            # 驗證約束
            constraints = self._extract_constraints(prop_schema)
            if constraints:
                sections.append("**約束:**")
                for constraint in constraints:
                    sections.append(f"- {constraint}")
                sections.append("")
            
            # 範例
            if prop_name in examples:
                sections.append("**範例:**")
                sections.append("```yaml")
                sections.append(yaml.dump({prop_name: examples[prop_name]}, default_flow_style=False))
                sections.append("```\n")
            
            # 巢狀屬性
            if prop_schema.get('type') == 'object' and 'properties' in prop_schema:
                nested = self._generate_sections(
                    prop_schema['properties'],
                    examples.get(prop_name, {}),
                    level + 1
                )
                sections.extend(nested)
        
        return sections
    
    def _extract_constraints(self, schema: Dict) -> List[str]:
        """從模式中提取驗證約束"""
        constraints = []
        
        if 'enum' in schema:
            constraints.append(f"必須是以下之一: {', '.join(map(str, schema['enum']))}")
        
        if 'minimum' in schema:
            constraints.append(f"最小值: {schema['minimum']}")
        
        if 'maximum' in schema:
            constraints.append(f"最大值: {schema['maximum']}")
        
        if 'minLength' in schema:
            constraints.append(f"最小長度: {schema['minLength']}")
        
        if 'maxLength' in schema:
            constraints.append(f"最大長度: {schema['maxLength']}")
        
        if 'pattern' in schema:
            constraints.append(f"必須匹配模式: `{schema['pattern']}`")
        
        if 'format' in schema:
            constraints.append(f"格式: {schema['format']}")
        
        return constraints

# 生成互動式配置建置器
class InteractiveConfigBuilder:
    def generate_html_builder(self, schema: Dict) -> str:
        """生成互動式 HTML 配置建置器"""
        html = """
<!DOCTYPE html>
<html>
<head>
    <title>配置建置器</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .config-section { margin: 20px 0; padding: 20px; border: 1px solid #ddd; }
        .config-field { margin: 10px 0; }
        label { display: inline-block; width: 200px; font-weight: bold; }
        input, select { width: 300px; padding: 5px; }
        .error { color: red; font-size: 12px; }
        .preview { background: #f5f5f5; padding: 20px; margin-top: 20px; }
        pre { background: white; padding: 10px; overflow-x: auto; }
    </style>
</head>
<body>
    <h1>配置建置器</h1>
    <div id="config-form"></div>
    <button onclick="validateConfig()">驗證</button>
    <button onclick="exportConfig()">匯出</button>
    
    <div class="preview">
        <h2>預覽</h2>
        <pre id="config-preview"></pre>
    </div>
    
    <script>
        const schema = """ + json.dumps(schema) + """;
        
        function buildForm() {
            const container = document.getElementById('config-form');
            container.innerHTML = renderSchema(schema.properties);
        }
        
        function renderSchema(properties, prefix = '') {
            let html = '';
            
            for (const [key, prop] of Object.entries(properties)) {
                const fieldId = prefix ? `${prefix}.${key}` : key;
                
                html += '<div class="config-field">';
                html += `<label for="${fieldId}">${key}:</label>`;
                
                if (prop.enum) {
                    html += `<select id="${fieldId}" onchange="updatePreview()">`;
                    for (const option of prop.enum) {
                        html += `<option value="${option}">${option}</option>`;
                    }
                    html += '</select>';
                } else if (prop.type === 'boolean') {
                    html += `<input type="checkbox" id="${fieldId}" onchange="updatePreview()">`;
                } else if (prop.type === 'number' || prop.type === 'integer') {
                    html += `<input type="number" id="${fieldId}" onchange="updatePreview()">`;
                } else {
                    html += `<input type="text" id="${fieldId}" onchange="updatePreview()">`;
                }
                
                html += `<div class="error" id="${fieldId}-error"></div>`;
                html += '</div>';
                
                if (prop.type === 'object' && prop.properties) {
                    html += '<div class="config-section">';
                    html += renderSchema(prop.properties, fieldId);
                    html += '</div>';
                }
            }
            
            return html;
        }
        
        function updatePreview() {
            const config = buildConfig();
            document.getElementById('config-preview').textContent = 
                JSON.stringify(config, null, 2);
        }
        
        function buildConfig() {
            // 從表單值建立配置
            const config = {};
            // 實施在此
            return config;
        }
        
        function validateConfig() {
            // 根據模式驗證
            const config = buildConfig();
            // 實施在此
        }
        
        function exportConfig() {
            const config = buildConfig();
            const blob = new Blob([JSON.stringify(config, null, 2)], 
                                 {type: 'application/json'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'config.json';
            a.click();
        }
        
        // 初始化
        buildForm();
        updatePreview();
    </script>
</body>
</html>
        """
        return html
```

## 輸出格式

1. **配置分析**：當前配置評估
2. **驗證模式**：所有配置的 JSON 模式定義
3. **環境規則**：環境特定的驗證規則
4. **測試套件**：全面的配置測試
5. **遷移腳本**：版本遷移實施
6. **安全報告**：安全問題和建議
7. **文件**：自動生成的配置參考
8. **驗證管道**：用於配置驗證的 CI/CD 整合
9. **互動工具**：配置建置器和驗證器

專注於防止配置錯誤、確保環境之間的一致性，並維護安全最佳實踐。