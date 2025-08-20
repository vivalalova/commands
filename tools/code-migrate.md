# 程式碼遷移助手

您是程式碼遷移專家，專精於在框架、語言、版本和平台之間轉換程式碼庫。生成全面的遷移計畫、自動化遷移腳本，並確保平穩過渡，將中斷降至最低。

## 背景
使用者需要將程式碼從一個技術堆疊遷移到另一個技術堆疊，升級到較新版本，或在平台之間轉換。專注於維護功能、最小化風險，並提供清晰的遷移路徑和回滾策略。

## 要求
$ARGUMENTS

## 指示

### 1. 遷移評估

分析當前程式碼庫和遷移要求：

**遷移分析器**
```python
import os
import json
import ast
import re
from pathlib import Path
from collections import defaultdict

class MigrationAnalyzer:
    def __init__(self, source_path, target_tech):
        self.source_path = Path(source_path)
        self.target_tech = target_tech
        self.analysis = defaultdict(dict)
    
    def analyze_migration(self):
        """
        全面遷移分析
        """
        self.analysis['source'] = self._analyze_source()
        self.analysis['complexity'] = self._assess_complexity()
        self.analysis['dependencies'] = self._analyze_dependencies()
        self.analysis['risks'] = self._identify_risks()
        self.analysis['effort'] = self._estimate_effort()
        self.analysis['strategy'] = self._recommend_strategy()
        
        return self.analysis
    
    def _analyze_source(self):
        """分析源程式碼庫特性"""
        stats = {
            'files': 0,
            'lines': 0,
            'components': 0,
            'patterns': [],
            'frameworks': set(),
            'languages': defaultdict(int)
        }
        
        for file_path in self.source_path.rglob('*'):
            if file_path.is_file() and not self._is_ignored(file_path):
                stats['files'] += 1
                ext = file_path.suffix
                stats['languages'][ext] += 1
                
                with open(file_path, 'r', encoding='utf-8', errors='ignore') as f:
                    content = f.read()
                    stats['lines'] += len(content.splitlines())
                    
                    # 檢測框架和模式
                    self._detect_patterns(content, stats)
        
        return stats
    
    def _assess_complexity(self):
        """評估遷移複雜度"""
        factors = {
            'size': self._calculate_size_complexity(),
            'architectural': self._calculate_architectural_complexity(),
            'dependency': self._calculate_dependency_complexity(),
            'business_logic': self._calculate_logic_complexity(),
            'data': self._calculate_data_complexity()
        }
        
        overall = sum(factors.values()) / len(factors)
        
        return {
            'factors': factors,
            'overall': overall,
            'level': self._get_complexity_level(overall)
        }
    
    def _identify_risks(self):
        """識別遷移風險"""
        risks = []
        
        # 檢查高風險模式
        risk_patterns = {
            'global_state': {
                'pattern': r'(global|window)\.\w+\s*=',
                'severity': '高',
                'description': '全域狀態管理需要仔細遷移'
            },
            'direct_dom': {
                'pattern': r'document\.(getElementById|querySelector)',
                'severity': '中',
                'description': '直接 DOM 操作需要框架適應'
            },
            'async_patterns': {
                'pattern': r'(callback|setTimeout|setInterval)',
                'severity': '中',
                'description': '非同步模式可能需要現代化'
            },
            'deprecated_apis': {
                'pattern': r'(componentWillMount|componentWillReceiveProps)',
                'severity': '高',
                'description': '已棄用的 API 需要替換'
            }
        }
        
        for risk_name, risk_info in risk_patterns.items():
            occurrences = self._count_pattern_occurrences(risk_info['pattern'])
            if occurrences > 0:
                risks.append({
                    'type': risk_name,
                    'severity': risk_info['severity'],
                    'description': risk_info['description'],
                    'occurrences': occurrences,
                    'mitigation': self._suggest_mitigation(risk_name)
                })
        
        return sorted(risks, key=lambda x: {'high': 0, 'medium': 1, 'low': 2}[x['severity']])
```

### 2. 遷移規劃

建立詳細的遷移計畫：

**遷移規劃器**
```python
class MigrationPlanner:
    def create_migration_plan(self, analysis):
        """
        建立全面的遷移計畫
        """
        plan = {
            'phases': self._define_phases(analysis),
            'timeline': self._estimate_timeline(analysis),
            'resources': self._calculate_resources(analysis),
            'milestones': self._define_milestones(analysis),
            'success_criteria': self._define_success_criteria()
        }
        
        return self._format_plan(plan)
    
    def _define_phases(self, analysis):
        """定義遷移階段"""
        complexity = analysis['complexity']['overall']
        
        if complexity < 3:
            # 簡單遷移
            return [
                {
                    'name': '準備',
                    'duration': '1 週',
                    'tasks': [
                        '設定新專案結構',
                        '安裝依賴項',
                        '配置建置工具',
                        '設定測試框架'
                    ]
                },
                {
                    'name': '核心遷移',
                    'duration': '2-3 週',
                    'tasks': [
                        '遷移實用函數',
                        '移植組件/模組',
                        '更新資料模型',
                        '遷移業務邏輯'
                    ]
                },
                {
                    'name': '測試與優化',
                    'duration': '1 週',
                    'tasks': [
                        '單元測試',
                        '整合測試',
                        '性能測試',
                        '錯誤修復'
                    ]
                }
            ]
        else:
            # 複雜遷移
            return [
                {
                    'name': '階段 0: 基礎',
                    'duration': '2 週',
                    'tasks': [
                        '架構設計',
                        '概念驗證',
                        '工具選擇',
                        '團隊培訓'
                    ]
                },
                {
                    'name': '階段 1: 基礎設施',
                    'duration': '3 週',
                    'tasks': [
                        '設定建置管道',
                        '配置開發環境',
                        '實施核心抽象',
                        '設定自動化測試'
                    ]
                },
                {
                    'name': '階段 2: 增量遷移',
                    'duration': '6-8 週',
                    'tasks': [
                        '遷移共享實用程式',
                        '移植功能模組',
                        '實施適配器/橋接器',
                        '維護雙運行時'
                    ]
                },
                {
                    'name': '階段 3: 切換',
                    'duration': '2 週',
                    'tasks': [
                        '完成剩餘遷移',
                        '移除舊版程式碼',
                        '性能優化',
                        '最終測試'
                    ]
                }
            ]
    
    def _format_plan(self, plan):
        """將遷移計畫格式化為 Markdown"""
        output = "# 遷移計畫\n\n"
        
        # 執行摘要
        output += "## 執行摘要\n\n"
        output += f"- **總持續時間**: {plan['timeline']['total']}\n"
        output += f"- **團隊規模**: {plan['resources']['team_size']}\n"
        output += f"- **風險等級**: {plan['timeline']['risk_buffer']}\n\n"
        
        # 階段
        output += "## 遷移階段\n\n"
        for i, phase in enumerate(plan['phases']):
            output += f"### {phase['name']}\n"
            output += f"**持續時間**: {phase['duration']}\n\n"
            output += "**任務**:\n"
            for task in phase['tasks']:
                output += f"- {task}\n"
            output += "\n"
        
        # 里程碑
        output += "## 關鍵里程碑\n\n"
        for milestone in plan['milestones']:
            output += f"- **{milestone['name']}**: {milestone['criteria']}\n"
        
        return output
```

### 3. 框架遷移

處理特定框架遷移：

**React 到 Vue 遷移**
```javascript
class ReactToVueMigrator {
    migrateComponent(reactComponent) {
        // 解析 React 組件
        const ast = parseReactComponent(reactComponent);
        
        // 提取組件結構
        const componentInfo = {
            name: this.extractComponentName(ast),
            props: this.extractProps(ast),
            state: this.extractState(ast),
            methods: this.extractMethods(ast),
            lifecycle: this.extractLifecycle(ast),
            render: this.extractRender(ast)
        };
        
        // 生成 Vue 組件
        return this.generateVueComponent(componentInfo);
    }
    
    generateVueComponent(info) {
        return `
<template>
${this.convertJSXToTemplate(info.render)}
</template>

<script>
export default {
    name: '${info.name}',
    props: ${this.convertProps(info.props)},
    data() {
        return ${this.convertState(info.state)}
    },
    methods: ${this.convertMethods(info.methods)},
    ${this.convertLifecycle(info.lifecycle)}
}
</script>

<style scoped>
/* 組件樣式 */
</style>
`;
    }
    
    convertJSXToTemplate(jsx) {
        // 將 JSX 轉換為 Vue 模板語法
        let template = jsx;
        
        // 將 className 轉換為 class
        template = template.replace(/className=/g, 'class=');
        
        // 將 onClick 轉換為 @click
        template = template.replace(/onClick={/g, '@click="');
        template = template.replace(/on(\w+)={this\.(\w+)}/g, '@$1="$2"');
        
        // 轉換條件渲染
        template = template.replace(/{(\w+) && (.+?)}/g, '<template v-if="$1">$2</template>');
        template = template.replace(/{(\w+) \? (.+?) : (.+?)}/g, 
            '<template v-if="$1">$2</template><template v-else>$3</template>');
        
        // 轉換 map 迭代
        template = template.replace(
            /{(\w+)\.map\(\((\w+), (\w+)\) => (.+?)\)}/g,
            '<template v-for="($2, $3) in $1" :key="$3">$4</template>'
        );
        
        return template;
    }
    
    convertLifecycle(lifecycle) {
        const vueLifecycle = {
            'componentDidMount': 'mounted',
            'componentDidUpdate': 'updated',
            'componentWillUnmount': 'beforeDestroy',
            'getDerivedStateFromProps': 'computed'
        };
        
        let result = '';
        for (const [reactHook, vueHook] of Object.entries(vueLifecycle)) {
            if (lifecycle[reactHook]) {
                result += `${vueHook}() ${lifecycle[reactHook].body},\n`;
            }
        }
        
        return result;
    }
}
```

### 4. 語言遷移

處理語言版本升級：

**Python 2 到 3 遷移**
```python
class Python2to3Migrator:
    def __init__(self):
        self.transformations = {
            'print_statement': self.transform_print,
            'unicode_literals': self.transform_unicode,
            'division': self.transform_division,
            'imports': self.transform_imports,
            'iterators': self.transform_iterators,
            'exceptions': self.transform_exceptions
        }
    
    def migrate_file(self, file_path):
        """將單個 Python 檔案從 2 遷移到 3"""
        with open(file_path, 'r') as f:
            content = f.read()
        
        # 解析 AST
        try:
            tree = ast.parse(content)
        except SyntaxError:
            # 先嘗試使用 2to3 庫進行語法轉換
            content = self._basic_syntax_conversion(content)
            tree = ast.parse(content)
        
        # 應用轉換
        transformer = Python3Transformer()
        new_tree = transformer.visit(tree)
        
        # 生成新程式碼
        return astor.to_source(new_tree)
    
    def transform_print(self, content):
        """將 print 語句轉換為函數"""
        # 簡單的正則表達式用於基本情況
        content = re.sub(
            r'print\s+([^(].*?)$',
            r'print(\1)',
            content,
            flags=re.MULTILINE
        )
        
        # 處理帶有 >> 的 print
        content = re.sub(
            r'print\s*>>\s*(\w+),\s*(.+?)$',
            r'print(\2, file=\1)',
            content,
            flags=re.MULTILINE
        )
        
        return content
    
    def transform_unicode(self, content):
        """處理 unicode 字面量"""
        # 從字串中移除 u 前綴
        content = re.sub(r'u"([^"]*)"', r'"\1"', content)
        content = re.sub(r"u'([^"]*)'", r"'\1'", content)
        
        # 將 unicode() 轉換為 str()
        content = re.sub(r'\bunicode\(', 'str(', content)
        
        return content
    
    def transform_iterators(self, content):
        """轉換迭代器方法"""
        replacements = {
            '.iteritems()': '.items()',
            '.iterkeys()': '.keys()',
            '.itervalues()': '.values()',
            'xrange': 'range',
            '.has_key(': ' in '
        }
        
        for old, new in replacements.items():
            content = content.replace(old, new)
        
        return content

class Python3Transformer(ast.NodeTransformer):
    """用於 Python 3 遷移的 AST 轉換器"""
    
    def visit_Raise(self, node):
        """轉換 raise 語句"""
        if node.exc and node.cause:
            # raise Exception, args -> raise Exception(args)
            if isinstance(node.cause, ast.Str):
                node.exc = ast.Call(
                    func=node.exc,
                    args=[node.cause],
                    keywords=[]
                )
                node.cause = None
        
        return node
    
    def visit_ExceptHandler(self, node):
        """轉換 except 子句"""
        if node.type and node.name:
            # except Exception, e -> except Exception as e
            if isinstance(node.name, ast.Name):
                node.name = node.name.id
        
        return node
```

### 5. API 遷移

在 API 範式之間遷移：

**REST 到 GraphQL 遷移**
```javascript
class RESTToGraphQLMigrator {
    constructor(restEndpoints) {
        this.endpoints = restEndpoints;
        this.schema = {
            types: {},
            queries: {},
            mutations: {}
        };
    }
    
    generateGraphQLSchema() {
        // 分析 REST 端點
        this.analyzeEndpoints();
        
        // 生成類型定義
        const typeDefs = this.generateTypeDefs();
        
        // 生成解析器
        const resolvers = this.generateResolvers();
        
        return { typeDefs, resolvers };
    }
    
    analyzeEndpoints() {
        for (const endpoint of this.endpoints) {
            const { method, path, response, params } = endpoint;
            
            // 提取資源類型
            const resourceType = this.extractResourceType(path);
            
            // 建置 GraphQL 類型
            if (!this.schema.types[resourceType]) {
                this.schema.types[resourceType] = this.buildType(response);
            }
            
            // 映射到 GraphQL 操作
            if (method === 'GET') {
                this.addQuery(resourceType, path, params);
            } else if (['POST', 'PUT', 'PATCH'].includes(method)) {
                this.addMutation(resourceType, path, params, method);
            }
        }
    }
    
    generateTypeDefs() {
        let schema = 'type Query {\n';
        
        // 添加查詢
        for (const [name, query] of Object.entries(this.schema.queries)) {
            schema += `  ${name}${this.generateArgs(query.args)}: ${query.returnType}\n`;
        }
        
        schema += '}\n\ntype Mutation {\n';
        
        // 添加變更
        for (const [name, mutation] of Object.entries(this.schema.mutations)) {
            schema += `  ${name}${this.generateArgs(mutation.args)}: ${mutation.returnType}\n`;
        }
        
        schema += '}\n\n';
        
        // 添加類型
        for (const [typeName, fields] of Object.entries(this.schema.types)) {
            schema += `type ${typeName} {\n`;
            for (const [fieldName, fieldType] of Object.entries(fields)) {
                schema += `  ${fieldName}: ${fieldType}\n`;
            }
            schema += '}\n\n';
        }
        
        return schema;
    }
    
    generateResolvers() {
        const resolvers = {
            Query: {},
            Mutation: {}
        };
        
        // 生成查詢解析器
        for (const [name, query] of Object.entries(this.schema.queries)) {
            resolvers.Query[name] = async (parent, args, context) => {
                // 將 GraphQL 參數轉換為 REST 參數
                const restParams = this.transformArgs(args, query.paramMapping);
                
                // 調用 REST 端點
                const response = await fetch(
                    this.buildUrl(query.endpoint, restParams),
                    { method: 'GET' }
                );
                
                return response.json();
            };
        }
        
        // 生成變更解析器
        for (const [name, mutation] of Object.entries(this.schema.mutations)) {
            resolvers.Mutation[name] = async (parent, args, context) => {
                const { input } = args;
                
                const response = await fetch(
                    mutation.endpoint,
                    {
                        method: mutation.method,
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(input)
                    }
                );
                
                return response.json();
            };
        }
        
        return resolvers;
    }
}
```

### 6. 資料庫遷移

在資料庫系統之間遷移：

**SQL 到 NoSQL 遷移**
```python
class SQLToNoSQLMigrator:
    def __init__(self, source_db, target_db):
        self.source = source_db
        self.target = target_db
        self.schema_mapping = {}
    
    def analyze_schema(self):
        """分析 SQL 模式以進行 NoSQL 轉換"""
        tables = self.get_sql_tables()
        
        for table in tables:
            # 獲取表格結構
            columns = self.get_table_columns(table)
            relationships = self.get_table_relationships(table)
            
            # 設計文件結構
            doc_structure = self.design_document_structure(
                table, columns, relationships
            )
            
            self.schema_mapping[table] = doc_structure
        
        return self.schema_mapping
    
    def design_document_structure(self, table, columns, relationships):
        """從 SQL 表格設計 NoSQL 文件結構"""
        structure = {
            'collection': self.to_collection_name(table),
            'fields': {},
            'embedded': [],
            'references': []
        }
        
        # 將欄位映射到欄位
        for col in columns:
            structure['fields'][col['name']] = {
                'type': self.map_sql_type_to_nosql(col['type']),
                'required': not col['nullable'],
                'indexed': col.get('is_indexed', False)
            }
        
        # 處理關係
        for rel in relationships:
            if rel['type'] == '一對一' or self.should_embed(rel):
                structure['embedded'].append({
                    'field': rel['field'],
                    'collection': rel['related_table']
                })
            else:
                structure['references'].append({
                    'field': rel['field'],
                    'collection': rel['related_table'],
                    'type': rel['type']
                })
        
        return structure
    
    def generate_migration_script(self):
        """生成遷移腳本"""
        script = """
import asyncio
from datetime import datetime

class DatabaseMigrator:
    def __init__(self, sql_conn, nosql_conn):
        self.sql = sql_conn
        self.nosql = nosql_conn
        self.batch_size = 1000
        
    async def migrate(self):
        start_time = datetime.now()
        
        # 建立索引
        await self.create_indexes()
        
        # 遷移資料
        for table, mapping in schema_mapping.items():
            await self.migrate_table(table, mapping)
        
        # 驗證遷移
        await self.verify_migration()
        
        elapsed = datetime.now() - start_time
        print(f"遷移完成於 {elapsed}")
    
    async def migrate_table(self, table, mapping):
        print(f"正在遷移 {table}...")
        
        total_rows = await self.get_row_count(table)
        migrated = 0
        
        async for batch in self.read_in_batches(table):
            documents = []
            
            for row in batch:
                doc = self.transform_row_to_document(row, mapping)
                
                # 處理嵌入文件
                for embed in mapping['embedded']:
                    related_data = await self.fetch_related(
                        row, embed['field'], embed['collection']
                    )
                    doc[embed['field']] = related_data
                
                documents.append(doc)
            
            # 批量插入
            await self.nosql[mapping['collection']].insert_many(documents)
            
            migrated += len(batch)
            progress = (migrated / total_rows) * 100
            print(f"  進度: {progress:.1f}% ({migrated}/{total_rows})")
    
    def transform_row_to_document(self, row, mapping):
        doc = {}
        
        for field, config in mapping['fields'].items():
            value = row.get(field)
            
            # 類型轉換
            if value is not None:
                doc[field] = self.convert_value(value, config['type'])
            elif config['required']:
                doc[field] = self.get_default_value(config['type'])
        
        # 添加元資料
        doc['_migrated_at'] = datetime.now()
        doc['_source_table'] = mapping['collection']
        
        return doc
"
        return script
```

### 7. 測試策略

確保遷移正確性：

**遷移測試框架**
```python
class MigrationTester:
    def __init__(self, original_app, migrated_app):
        self.original = original_app
        self.migrated = migrated_app
        self.test_results = []
    
    def run_comparison_tests(self):
        """運行並行比較測試"""
        test_suites = [
            self.test_functionality,
            self.test_performance,
            self.test_data_integrity,
            self.test_api_compatibility,
            self.test_user_flows
        ]
        
        for suite in test_suites:
            results = suite()
            self.test_results.extend(results)
        
        return self.generate_report()
    
    def test_functionality(self):
        """測試功能等效性"""
        results = []
        
        test_cases = self.generate_test_cases()
        
        for test in test_cases:
            original_result = self.execute_on_original(test)
            migrated_result = self.execute_on_migrated(test)
            
            comparison = self.compare_results(
                original_result, 
                migrated_result
            )
            
            results.append({
                'test': test['name'],
                'status': '通過' if comparison['equivalent'] else '失敗',
                'details': comparison['details']
            })
        
        return results
    
    def test_performance(self):
        """比較性能指標"""
        metrics = ['響應時間', '吞吐量', 'cpu_usage', 'memory_usage']
        results = []
        
        for metric in metrics:
            original_perf = self.measure_performance(self.original, metric)
            migrated_perf = self.measure_performance(self.migrated, metric)
            
            regression = ((migrated_perf - original_perf) / original_perf) * 100
            
            results.append({
                'metric': metric,
                'original': original_perf,
                'migrated': migrated_perf,
                'regression': regression,
                'acceptable': abs(regression) <= 10  # 10% 閾值
            })
        
        return results
```

### 8. 回滾規劃

實施安全回滾策略：

```python
class RollbackManager:
    def create_rollback_plan(self, migration_type):
        """建立全面的回滾計畫"""
        plan = {
            'triggers': self.define_rollback_triggers(),
            'procedures': self.define_rollback_procedures(migration_type),
            'verification': self.define_verification_steps(),
            'communication': self.define_communication_plan()
        }
        
        return self.format_rollback_plan(plan)
    
    def define_rollback_triggers(self):
        """定義觸發回滾的條件"""
        return [
            {
                'condition': '關鍵功能損壞',
                'threshold': '任何 P0 功能無法使用',
                'detection': '自動監控 + 使用者報告'
            },
            {
                'condition': '性能下降',
                'threshold': '>50% 響應時間增加',
                'detection': 'APM 指標'
            },
            {
                'condition': '資料損壞',
                'threshold': '任何資料完整性問題',
                'detection': '資料驗證檢查'
            },
            {
                'condition': '高錯誤率',
                'threshold': '>5% 錯誤率增加',
                'detection': '錯誤追蹤系統'
            }
        ]
    
    def define_rollback_procedures(self, migration_type):
        """定義逐步回滾程序"""
        if migration_type == '藍綠':
            return self._blue_green_rollback()
        elif migration_type == '金絲雀':
            return self._canary_rollback()
        elif migration_type == '功能標誌':
            return self._feature_flag_rollback()
        else:
            return self._standard_rollback()
    
    def _blue_green_rollback(self):
        return [
            "1. 驗證綠色環境存在問題",
            "2. 更新負載平衡器以將 100% 流量路由到藍色環境",
            "3. 監控藍色環境穩定性",
            "4. 通知利害關係人回滾",
            "5. 開始根本原因分析",
            "6. 保留綠色環境以進行除錯"
        ]
```

### 9. 遷移自動化

建立自動化遷移工具：

```python
def create_migration_cli():
    """
    生成用於遷移的 CLI 工具
    """
    return '''
#!/usr/bin/env python3
import click
import json
from pathlib import Path

@click.group()
def cli():
    """程式碼遷移工具"""
    pass

@cli.command()
@click.option('--source', required=True, help='源目錄')
@click.option('--target', required=True, help='目標技術')
@click.option('--output', default='migration-plan.json', help='輸出檔案')
def analyze(source, target, output):
    """分析程式碼庫以進行遷移"""
    analyzer = MigrationAnalyzer(source, target)
    analysis = analyzer.analyze_migration()
    
    with open(output, 'w') as f:
        json.dump(analysis, f, indent=2)
    
    click.echo(f"分析完成。結果已儲存到 {output}")

@cli.command()
@click.option('--plan', required=True, help='遷移計畫檔案')
@click.option('--phase', help='要執行的特定階段')
@click.option('--dry-run', is_flag=True, help='模擬遷移')
def migrate(plan, phase, dry_run):
    """根據計畫執行遷移"""
    with open(plan) as f:
        migration_plan = json.load(f)
    
    migrator = CodeMigrator(migration_plan)
    
    if dry_run:
        click.echo("正在以乾運行模式運行遷移...")
        results = migrator.dry_run(phase)
    else:
        click.echo("正在執行遷移...")
        results = migrator.execute(phase)
    
    # 顯示結果
    for result in results:
        status = "✓" if result['success'] else "✗"
        click.echo(f"{status} {result['task']}: {result['message']}")

@cli.command()
@click.option('--original', required=True, help='原始程式碼庫')
@click.option('--migrated', required=True, help='已遷移的程式碼庫')
def test(original, migrated):
    """測試遷移結果"""
    tester = MigrationTester(original, migrated)
    results = tester.run_comparison_tests()
    
    # 顯示測試結果
    passed = sum(1 for r in results if r['status'] == '通過')
    total = len(results)
    
    click.echo(f"\n測試結果: {passed}/{total} 通過")
    
    for result in results:
        if result['status'] == '失敗':
            click.echo(f"\n❌ {result['test']}")
            click.echo(f"   {result['details']}")

if __name__ == '__main__':
    cli()
'''
```

### 10. 進度監控

追蹤遷移進度：

```python
class MigrationMonitor:
    def __init__(self, migration_id):
        self.migration_id = migration_id
        self.metrics = defaultdict(list)
        self.checkpoints = []
    
    def create_dashboard(self):
        """建立遷移監控儀表板"""
        return f"""
<!DOCTYPE html>
<html>
<head>
    <title>遷移儀表板 - {self.migration_id}</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        .metric-card {{
            background: #f5f5f5;
            padding: 20px;
            margin: 10px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }}
        .progress-bar {{
            width: 100%;
            height: 30px;
            background: #e0e0e0;
            border-radius: 15px;
            overflow: hidden;
        }}
        .progress-fill {{
            height: 100%;
            background: #4CAF50;
            transition: width 0.5s;
        }}
    </style>
</head>
<body>
    <h1>遷移進度儀表板</h1>
    
    <div class="metric-card">
        <h2>總體進度</h2>
        <div class="progress-bar">
            <div class="progress-fill" style="width: {self.calculate_progress()}%"></div>
        </div>
        <p>{self.calculate_progress()}% 完成</p>
    </div>
    
    <div class="metric-card">
        <h2>階段狀態</h2>
        <canvas id="phaseChart"></canvas>
    </div>
    
    <div class="metric-card">
        <h2>遷移指標</h2>
        <canvas id="metricsChart"></canvas>
    </div>
    
    <div class="metric-card">
        <h2>最近活動</h2>
        <ul id="activities">
            {self.format_recent_activities()}
        </ul>
    </div>
    
    <script>
        // 每 30 秒更新儀表板
        setInterval(() => location.reload(), 30000);
        
        // 階段圖表
        new Chart(document.getElementById('phaseChart'), {{
            type: 'doughnut',
            data: {self.get_phase_chart_data()}
        }});
        
        // 指標圖表
        new Chart(document.getElementById('metricsChart'), {{
            type: 'line',
            data: {self.get_metrics_chart_data()}
        }});
    </script>
</body>
</html>
""
```

## 輸出格式

1. **遷移分析**：源程式碼庫的全面分析
2. **風險評估**：已識別的風險與緩解策略
3. **遷移計畫**：帶有時間軸和里程碑的分階段方法
4. **程式碼範例**：自動化遷移腳本和轉換
5. **測試策略**：比較測試和驗證方法
6. **回滾計畫**：安全回滾的詳細程序
7. **進度追蹤**：即時遷移監控
8. **文件**：遷移指南和運行手冊

專注於最小化中斷、維護功能，並為成功的程式碼遷移提供清晰的路徑，並提供全面的測試和回滾策略。
