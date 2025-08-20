# API 模擬框架

您是 API 模擬專家，專精於為開發、測試和演示目的建立逼真的模擬服務。設計全面的模擬解決方案，模擬真實的 API 行為，實現平行開發，並促進徹底的測試。

## 背景
使用者需要為開發、測試或演示目的建立模擬 API。專注於建立靈活、逼真的模擬，精確模擬生產 API 行為，同時實現高效的開發工作流程。

## 要求
$ARGUMENTS

## 指示

### 1. 模擬伺服器設定

建立全面的模擬伺服器基礎設施：

**模擬伺服器框架**
```python
from typing import Dict, List, Any, Optional
import json
import asyncio
from datetime import datetime
from fastapi import FastAPI, Request, Response
import uvicorn

class MockAPIServer:
    def __init__(self, config: Dict[str, Any]):
        self.app = FastAPI(title="模擬 API 伺服器")
        self.routes = {}
        self.middleware = []
        self.state_manager = StateManager()
        self.scenario_manager = ScenarioManager()
        
    def setup_mock_server(self):
        """設定全面的模擬伺服器"""
        # 配置中介軟體
        self._setup_middleware()
        
        # 載入模擬定義
        self._load_mock_definitions()
        
        # 設定動態路由
        self._setup_dynamic_routes()
        
        # 初始化場景
        self._initialize_scenarios()
        
        return self.app
    
    def _setup_middleware(self):
        """配置伺服器中介軟體"""
        @self.app.middleware("http")
        async def add_mock_headers(request: Request, call_next):
            response = await call_next(request)
            response.headers["X-Mock-Server"] = "true"
            response.headers["X-Mock-Scenario"] = self.scenario_manager.current_scenario
            return response
        
        @self.app.middleware("http")
        async def simulate_latency(request: Request, call_next):
            # 模擬網路延遲
            latency = self._calculate_latency(request.url.path)
            await asyncio.sleep(latency / 1000)  # 轉換為秒
            response = await call_next(request)
            return response
        
        @self.app.middleware("http")
        async def track_requests(request: Request, call_next):
            # 追蹤請求以進行驗證
            self.state_manager.track_request({
                'method': request.method,
                'path': str(request.url.path),
                'headers': dict(request.headers),
                'timestamp': datetime.now()
            })
            response = await call_next(request)
            return response
    
    def _setup_dynamic_routes(self):
        """設定動態路由處理"""
        @self.app.api_route("/{path:path}", methods=["GET", "POST", "PUT", "DELETE", "PATCH"])
        async def handle_mock_request(path: str, request: Request):
            # 尋找匹配的模擬
            mock = self._find_matching_mock(request.method, path, request)
            
            if not mock:
                return Response(
                    content=json.dumps({"error": "此端點找不到模擬"}),
                    status_code=404,
                    media_type="application/json"
                )
            
            # 處理模擬響應
            response_data = await self._process_mock_response(mock, request)
            
            return Response(
                content=json.dumps(response_data['body']),
                status_code=response_data['status'],
                headers=response_data['headers'],
                media_type="application/json"
            )
    
    async def _process_mock_response(self, mock: Dict[str, Any], request: Request):
        """處理並生成模擬響應"""
        # 檢查條件響應
        if mock.get('conditions'):
            for condition in mock['conditions']:
                if self._evaluate_condition(condition, request):
                    return await self._generate_response(condition['response'], request)
        
        # 使用預設響應
        return await self._generate_response(mock['response'], request)
    
    def _generate_response(self, response_template: Dict[str, Any], request: Request):
        """從模板生成響應"""
        response = {
            'status': response_template.get('status', 200),
            'headers': response_template.get('headers', {}),
            'body': self._process_response_body(response_template['body'], request)
        }
        
        # 應用響應轉換
        if response_template.get('transformations'):
            response = self._apply_transformations(response, response_template['transformations'])
        
        return response
```

### 2. 請求/響應存根

實施靈活的存根系統：

**存根引擎**
```python
class StubbingEngine:
    def __init__(self):
        self.stubs = {}
        self.matchers = self._initialize_matchers()
        
    def create_stub(self, method: str, path: str, **kwargs):
        """建立新存根"""
        stub_id = self._generate_stub_id()
        
        stub = {
            'id': stub_id,
            'method': method,
            'path': path,
            'matchers': self._build_matchers(kwargs),
            'response': kwargs.get('response', {}),
            'priority': kwargs.get('priority', 0),
            'times': kwargs.get('times', -1),  # -1 表示無限
            'delay': kwargs.get('delay', 0),
            'scenario': kwargs.get('scenario', 'default')
        }
        
        self.stubs[stub_id] = stub
        return stub_id
    
    def _build_matchers(self, kwargs):
        """建立請求匹配器"""
        matchers = []
        
        # 路徑參數匹配
        if 'path_params' in kwargs:
            matchers.append({
                'type': 'path_params',
                'params': kwargs['path_params']
            })
        
        # 查詢參數匹配
        if 'query_params' in kwargs:
            matchers.append({
                'type': 'query_params',
                'params': kwargs['query_params']
            })
        
        # 標頭匹配
        if 'headers' in kwargs:
            matchers.append({
                'type': 'headers',
                'headers': kwargs['headers']
            })
        
        # 主體匹配
        if 'body' in kwargs:
            matchers.append({
                'type': 'body',
                'body': kwargs['body'],
                'match_type': kwargs.get('body_match_type', 'exact')
            })
        
        return matchers
    
    def match_request(self, request: Dict[str, Any]):
        """為請求尋找匹配的存根"""
        candidates = []
        
        for stub in self.stubs.values():
            if self._matches_stub(request, stub):
                candidates.append(stub)
        
        # 按優先級排序並返回最佳匹配
        if candidates:
            return sorted(candidates, key=lambda x: x['priority'], reverse=True)[0]
        
        return None
    
    def _matches_stub(self, request: Dict[str, Any], stub: Dict[str, Any]):
        """檢查請求是否匹配存根"""
        # 檢查方法
        if request['method'] != stub['method']:
            return False
        
        # 檢查路徑
        if not self._matches_path(request['path'], stub['path']):
            return False
        
        # 檢查所有匹配器
        for matcher in stub['matchers']:
            if not self._evaluate_matcher(request, matcher):
                return False
        
        # 檢查存根是否仍然有效
        if stub['times'] == 0:
            return False
        
        return True
    
    def create_dynamic_stub(self):
        """建立帶有回調的動態存根"""
        return '''
class DynamicStub:
    def __init__(self, path_pattern: str):
        self.path_pattern = path_pattern
        self.response_generator = None
        self.state_modifier = None
        
    def with_response_generator(self, generator):
        """設定動態響應生成器"""
        self.response_generator = generator
        return self
    
    def with_state_modifier(self, modifier):
        """設定狀態修改回調"""
        self.state_modifier = modifier
        return self
    
    async def process_request(self, request: Request, state: Dict[str, Any]):
        """動態處理請求"""
        # 提取請求資料
        request_data = {
            'method': request.method,
            'path': request.url.path,
            'headers': dict(request.headers),
            'query_params': dict(request.query_params),
            'body': await request.json() if request.method in ['POST', 'PUT'] else None
        }
        
        # 如果需要，修改狀態
        if self.state_modifier:
            state = self.state_modifier(state, request_data)
        
        # 生成響應
        if self.response_generator:
            response = self.response_generator(request_data, state)
        else:
            response = {'status': 200, 'body': {}}
        
        return response, state

# 使用範例
dynamic_stub = DynamicStub('/api/users/{user_id}')
dynamic_stub.with_response_generator(lambda req, state: {
    'status': 200,
    'body': {
        'id': req['path_params']['user_id'],
        'name': state.get('users', {}).get(req['path_params']['user_id'], '未知'),
        'request_count': state.get('request_count', 0)
    }
}).with_state_modifier(lambda state, req: {
    **state,
    'request_count': state.get('request_count', 0) + 1
})
'''
```

### 3. 動態資料生成

生成逼真的模擬資料：

**模擬資料生成器**
```python
from faker import Faker
import random
from datetime import datetime, timedelta

class MockDataGenerator:
    def __init__(self):
        self.faker = Faker()
        self.templates = {}
        self.generators = self._init_generators()
        
    def generate_data(self, schema: Dict[str, Any]):
        """根據模式生成資料"""
        if isinstance(schema, dict):
            if '$ref' in schema:
                # 引用另一個模式
                return self.generate_data(self.resolve_ref(schema['$ref']))
            
            result = {}
            for key, value in schema.items():
                if key.startswith('$'):
                    continue
                result[key] = self._generate_field(value)
            return result
        
        elif isinstance(schema, list):
            # 生成陣列
            count = random.randint(1, 10)
            return [self.generate_data(schema[0]) for _ in range(count)]
        
        else:
            return schema
    
    def _generate_field(self, field_schema: Dict[str, Any]):
        """根據模式生成欄位值"""
        field_type = field_schema.get('type', 'string')
        
        # 檢查自定義生成器
        if 'generator' in field_schema:
            return self._use_custom_generator(field_schema['generator'])
        
        # 檢查枚舉
        if 'enum' in field_schema:
            return random.choice(field_schema['enum'])
        
        # 根據類型生成
        generators = {
            'string': self._generate_string,
            'number': self._generate_number,
            'integer': self._generate_integer,
            'boolean': self._generate_boolean,
            'array': self._generate_array,
            'object': lambda s: self.generate_data(s)
        }
        
        generator = generators.get(field_type, self._generate_string)
        return generator(field_schema)
    
    def _generate_string(self, schema: Dict[str, Any]):
        """生成字串值"""
        # 檢查格式
        format_type = schema.get('format', '')
        
        format_generators = {
            'email': self.faker.email,
            'name': self.faker.name,
            'first_name': self.faker.first_name,
            'last_name': self.faker.last_name,
            'phone': self.faker.phone_number,
            'address': self.faker.address,
            'url': self.faker.url,
            'uuid': self.faker.uuid4,
            'date': lambda: self.faker.date().isoformat(),
            'datetime': lambda: self.faker.date_time().isoformat(),
            'password': lambda: self.faker.password()
        }
        
        if format_type in format_generators:
            return format_generators[format_type]()
        
        # 檢查模式
        if 'pattern' in schema:
            return self._generate_from_pattern(schema['pattern'])
        
        # 預設字串生成
        min_length = schema.get('minLength', 5)
        max_length = schema.get('maxLength', 20)
        return self.faker.text(max_nb_chars=random.randint(min_length, max_length))
    
    def create_data_templates(self):
        """建立可重用資料模板"""
        return {
            'user': {
                'id': {'type': 'string', 'format': 'uuid'},
                'username': {'type': 'string', 'generator': 'username'},
                'email': {'type': 'string', 'format': 'email'},
                'profile': {
                    'type': 'object',
                    'properties': {
                        'firstName': {'type': 'string', 'format': 'first_name'},
                        'lastName': {'type': 'string', 'format': 'last_name'},
                        'avatar': {'type': 'string', 'format': 'url'},
                        'bio': {'type': 'string', 'maxLength': 200}
                    }
                },
                'createdAt': {'type': 'string', 'format': 'datetime'},
                'status': {'type': 'string', 'enum': ['active', 'inactive', 'suspended']}
            },
            'product': {
                'id': {'type': 'string', 'format': 'uuid'},
                'name': {'type': 'string', 'generator': 'product_name'},
                'description': {'type': 'string', 'maxLength': 500},
                'price': {'type': 'number', 'minimum': 0.01, 'maximum': 9999.99},
                'category': {'type': 'string', 'enum': ['電子產品', '服裝', '食品', '書籍']},
                'inStock': {'type': 'boolean'},
                'rating': {'type': 'number', 'minimum': 0, 'maximum': 5}
            }
        }
    
    def generate_relational_data(self):
        """生成帶有關係的資料"""
        return '''
class RelationalDataGenerator:
    def generate_related_entities(self, schema: Dict[str, Any], count: int):
        """生成相關實體並維護參照完整性"""
        entities = {}
        
        # 第一遍：生成主要實體
        for entity_name, entity_schema in schema['entities'].items():
            entities[entity_name] = []
            for i in range(count):
                entity = self.generate_entity(entity_schema)
                entity['id'] = f"{entity_name}_{i}"
                entities[entity_name].append(entity)
        
        # 第二遍：建立關係
        for relationship in schema.get('relationships', []):
            self.establish_relationship(entities, relationship)
        
        return entities
    
    def establish_relationship(self, entities: Dict[str, List], relationship: Dict):
        """在實體之間建立關係"""
        source = relationship['source']
        target = relationship['target']
        rel_type = relationship['type']
        
        if rel_type == '一對多':
            for source_entity in entities[source['entity']]:
                # 選擇隨機目標
                num_targets = random.randint(1, 5)
                target_refs = random.sample(
                    entities[target['entity']], 
                    min(num_targets, len(entities[target['entity']]))
                )
                source_entity[source['field']] = [t['id'] for t in target_refs]
        
        elif rel_type == '多對一':
            for target_entity in entities[target['entity']]:
                # 選擇一個源
                source_ref = random.choice(entities[source['entity']])
                target_entity[target['field']] = source_ref['id']
'''
```

### 4. 模擬場景

實施基於場景的模擬：

**場景管理器**
```python
class ScenarioManager:
    def __init__(self):
        self.scenarios = {}
        self.current_scenario = 'default'
        self.scenario_states = {}
        
    def define_scenario(self, name: str, definition: Dict[str, Any]):
        """定義模擬場景"""
        self.scenarios[name] = {
            'name': name,
            'description': definition.get('description', ''),
            'initial_state': definition.get('initial_state', {}),
            'stubs': definition.get('stubs', []),
            'sequences': definition.get('sequences', []),
            'conditions': definition.get('conditions', [])
        }
    
    def create_test_scenarios(self):
        """建立常見測試場景"""
        return {
            'happy_path': {
                'description': '所有操作成功',
                'stubs': [
                    {
                        'path': '/api/auth/login',
                        'response': {
                            'status': 200,
                            'body': {
                                'token': '有效令牌',
                                'user': {'id': '123', 'name': '測試使用者'}
                            }
                        }
                    },
                    {
                        'path': '/api/users/{id}',
                        'response': {
                            'status': 200,
                            'body': {
                                'id': '{id}',
                                'name': '測試使用者',
                                'email': 'test@example.com'
                            }
                        }
                    }
                ]
            },
            'error_scenario': {
                'description': '各種錯誤條件',
                'sequences': [
                    {
                        'name': '速率限制',
                        'steps': [
                            {'repeat': 5, 'response': {'status': 200}},
                            {'repeat': 10, 'response': {'status': 429, 'body': {'error': '速率限制超出'}}}
                        ]
                    }
                ],
                'stubs': [
                    {
                        'path': '/api/auth/login',
                        'conditions': [
                            {
                                'match': {'body': {'username': '鎖定使用者'}},
                                'response': {'status': 423, 'body': {'error': '帳戶已鎖定'}}
                            }
                        ]
                    }
                ]
            },
            'degraded_performance': {
                'description': '響應緩慢和超時',
                'stubs': [
                    {
                        'path': '/api/*',
                        'delay': 5000,  # 5 秒延遲
                        'response': {'status': 200}
                    }
                ]
            }
        }
    
    def execute_scenario_sequence(self):
        """執行場景序列"""
        return '''
class SequenceExecutor:
    def __init__(self):
        self.sequence_states = {}
        
    def get_sequence_response(self, sequence_name: str, request: Dict):
        """根據序列狀態獲取響應"""
        if sequence_name not in self.sequence_states:
            self.sequence_states[sequence_name] = {'step': 0, 'count': 0}
        
        state = self.sequence_states[sequence_name]
        sequence = self.get_sequence_definition(sequence_name)
        
        # 獲取當前步驟
        current_step = sequence['steps'][state['step']]
        
        # 檢查是否應前進到下一步
        state['count'] += 1
        if state['count'] >= current_step.get('repeat', 1):
            state['step'] = (state['step'] + 1) % len(sequence['steps'])
            state['count'] = 0
        
        return current_step['response']
    
    def create_stateful_scenario(self):
        """建立具有狀態行為的場景"""
        return {
            'shopping_cart': {
                'initial_state': {
                    'cart': {},
                    'total': 0
                },
                'stubs': [
                    {
                        'method': 'POST',
                        'path': '/api/cart/items',
                        'handler': 'add_to_cart',
                        'modifies_state': True
                    },
                    {
                        'method': 'GET',
                        'path': '/api/cart',
                        'handler': 'get_cart',
                        'uses_state': True
                    }
                ],
                'handlers': {
                    'add_to_cart': lambda state, request: {
                        'state': {
                            **state,
                            'cart': {
                                **state['cart'],
                                request['body']['product_id']: request['body']['quantity']
                            },
                            'total': state['total'] + request['body']['price']
                        },
                        'response': {
                            'status': 201,
                            'body': {'message': '商品已添加到購物車'}
                        }
                    },
                    'get_cart': lambda state, request: {
                        'response': {
                            'status': 200,
                            'body': {
                                'items': state['cart'],
                                'total': state['total']
                            }
                        }
                    }
                }
            }
        }
'''
```

### 5. 契約測試

實施基於契約的模擬：

**契約測試框架**
```python
class ContractMockServer:
    def __init__(self):
        self.contracts = {}
        self.validators = self._init_validators()
        
    def load_contract(self, contract_path: str):
        """載入 API 契約 (OpenAPI, AsyncAPI 等)"""
        with open(contract_path, 'r') as f:
            contract = yaml.safe_load(f)
        
        # 解析契約
        self.contracts[contract['info']['title']] = {
            'spec': contract,
            'endpoints': self._parse_endpoints(contract),
            'schemas': self._parse_schemas(contract)
        }
    
    def generate_mocks_from_contract(self, contract_name: str):
        """從契約規範生成模擬"""
        contract = self.contracts[contract_name]
        mocks = []
        
        for path, methods in contract['endpoints'].items():
            for method, spec in methods.items():
                mock = self._create_mock_from_spec(path, method, spec)
                mocks.append(mock)
        
        return mocks
    
    def _create_mock_from_spec(self, path: str, method: str, spec: Dict):
        """從端點規範建立模擬"""
        mock = {
            'method': method.upper(),
            'path': self._convert_path_to_pattern(path),
            'responses': {}
        }
        
        # 為每個狀態碼生成響應
        for status_code, response_spec in spec.get('responses', {}).items():
            mock['responses'][status_code] = {
                'status': int(status_code),
                'headers': self._get_response_headers(response_spec),
                'body': self._generate_response_body(response_spec)
            }
        
        # 添加請求驗證
        if 'requestBody' in spec:
            mock['request_validation'] = self._create_request_validator(spec['requestBody'])
        
        return mock
    
    def validate_against_contract(self):
        """根據契約驗證模擬響應"""
        return '''
class ContractValidator:
    def validate_response(self, contract_spec, actual_response):
        """根據契約驗證響應"""
        validation_results = {
            'valid': True,
            'errors': []
        }
        
        # 尋找狀態碼的響應規範
        response_spec = contract_spec['responses'].get(
            str(actual_response['status']),
            contract_spec['responses'].get('default')
        )
        
        if not response_spec:
            validation_results['errors'].append({
                'type': '意外狀態',
                'message': f"狀態 {actual_response['status']} 未在契約中定義"
            })
            validation_results['valid'] = False
            return validation_results
        
        # 驗證標頭
        if 'headers' in response_spec:
            header_errors = self.validate_headers(
                response_spec['headers'],
                actual_response['headers']
            )
            validation_results['errors'].extend(header_errors)
        
        # 驗證主體模式
        if 'content' in response_spec:
            body_errors = self.validate_body(
                response_spec['content'],
                actual_response['body']
            )
            validation_results['errors'].extend(body_errors)
        
        validation_results['valid'] = len(validation_results['errors']) == 0
        return validation_results
    
    def validate_body(self, content_spec, actual_body):
        """根據模式驗證響應主體"""
        errors = []
        
        # 獲取內容類型的模式
        schema = content_spec.get('application/json', {}).get('schema')
        if not schema:
            return errors
        
        # 根據 JSON 模式驗證
        try:
            validate(instance=actual_body, schema=schema)
        except ValidationError as e:
            errors.append({
                'type': '模式驗證',
                'path': e.json_path,
                'message': e.message
            })
        
        return errors
'''
```

### 6. 性能測試

建立性能測試模擬：

**性能模擬伺服器**
```python
class PerformanceMockServer:
    def __init__(self):
        self.performance_profiles = {}
        self.metrics_collector = MetricsCollector()
        
    def create_performance_profile(self, name: str, config: Dict):
        """建立性能測試配置檔"""
        self.performance_profiles[name] = {
            'latency': config.get('latency', {'min': 10, 'max': 100}),
            'throughput': config.get('throughput', 1000),  # 每秒請求數
            'error_rate': config.get('error_rate', 0.01),  # 1% 錯誤
            'response_size': config.get('response_size', {'min': 100, 'max': 10000})
        }
    
    async def simulate_performance(self, profile_name: str, request: Request):
        """模擬性能特性"""
        profile = self.performance_profiles[profile_name]
        
        # 模擬延遲
        latency = random.uniform(profile['latency']['min'], profile['latency']['max'])
        await asyncio.sleep(latency / 1000)
        
        # 模擬錯誤
        if random.random() < profile['error_rate']:
            return self._generate_error_response()
        
        # 生成指定大小的響應
        response_size = random.randint(
            profile['response_size']['min'],
            profile['response_size']['max']
        )
        
        response_data = self._generate_data_of_size(response_size)
        
        # 追蹤指標
        self.metrics_collector.record({
            'latency': latency,
            'response_size': response_size,
            'timestamp': datetime.now()
        })
        
        return response_data
    
    def create_load_test_scenarios(self):
        """建立負載測試場景"""
        return {
            'gradual_load': {
                'description': '逐漸增加負載',
                'stages': [
                    {'duration': 60, 'target_rps': 100},
                    {'duration': 120, 'target_rps': 500},
                    {'duration': 180, 'target_rps': 1000},
                    {'duration': 60, 'target_rps': 100}
                ]
            },
            'spike_test': {
                'description': '流量突然飆升',
                'stages': [
                    {'duration': 60, 'target_rps': 100},
                    {'duration': 10, 'target_rps': 5000},
                    {'duration': 60, 'target_rps': 100}
                ]
            },
            'stress_test': {
                'description': '尋找臨界點',
                'stages': [
                    {'duration': 60, 'target_rps': 100},
                    {'duration': 60, 'target_rps': 500},
                    {'duration': 60, 'target_rps': 1000},
                    {'duration': 60, 'target_rps': 2000},
                    {'duration': 60, 'target_rps': 5000},
                    {'duration': 60, 'target_rps': 10000}
                ]
            }
        }
    
    def implement_throttling(self):
        """實施請求節流"""
        return '''
class ThrottlingMiddleware:
    def __init__(self, max_rps: int):
        self.max_rps = max_rps
        self.request_times = deque()
        
    async def __call__(self, request: Request, call_next):
        current_time = time.time()
        
        # 移除舊請求
        while self.request_times and self.request_times[0] < current_time - 1:
            self.request_times.popleft()
        
        # 檢查是否超出限制
        if len(self.request_times) >= self.max_rps:
            return Response(
                content=json.dumps({
                    'error': '速率限制超出',
                    'retry_after': 1
                }),
                status_code=429,
                headers={'Retry-After': '1'}
            )
        
        # 記錄此請求
        self.request_times.append(current_time)
        
        # 處理請求
        response = await call_next(request)
        return response
'''
```

### 7. 模擬資料管理

有效管理模擬資料：

**模擬資料儲存**
```python
class MockDataStore:
    def __init__(self):
        self.collections = {}
        self.indexes = {}
        
    def create_collection(self, name: str, schema: Dict = None):
        """建立新資料集合"""
        self.collections[name] = {
            'data': {},
            'schema': schema,
            'counter': 0
        }
        
        # 在 'id' 上建立預設索引
        self.create_index(name, 'id')
    
    def insert(self, collection: str, data: Dict):
        """將資料插入集合"""
        collection_data = self.collections[collection]
        
        # 如果存在，則根據模式驗證
        if collection_data['schema']:
            self._validate_data(data, collection_data['schema'])
        
        # 如果未提供，則生成 ID
        if 'id' not in data:
            collection_data['counter'] += 1
            data['id'] = str(collection_data['counter'])
        
        # 儲存資料
        collection_data['data'][data['id']] = data
        
        # 更新索引
        self._update_indexes(collection, data)
        
        return data['id']
    
    def query(self, collection: str, filters: Dict = None):
        """使用過濾器查詢集合"""
        collection_data = self.collections[collection]['data']
        
        if not filters:
            return list(collection_data.values())
        
        # 如果可用，則使用索引
        if self._can_use_index(collection, filters):
            return self._query_with_index(collection, filters)
        
        # 全面掃描
        results = []
        for item in collection_data.values():
            if self._matches_filters(item, filters):
                results.append(item)
        
        return results
    
    def create_relationships(self):
        """定義集合之間的關係"""
        return '''
class RelationshipManager:
    def __init__(self, data_store: MockDataStore):
        self.store = data_store
        self.relationships = {}
        
    def define_relationship(self, 
                          source_collection: str,
                          target_collection: str,
                          relationship_type: str,
                          foreign_key: str):
        """定義集合之間的關係"""
        self.relationships[f"{source_collection}->{target_collection}"] = {
            'type': relationship_type,
            'source': source_collection,
            'target': target_collection,
            'foreign_key': foreign_key
        }
    
    def populate_related_data(self, entity: Dict, collection: str, depth: int = 1):
        """填充實體的相關資料"""
        if depth <= 0:
            return entity
        
        # 尋找此集合的關係
        for rel_key, rel in self.relationships.items():
            if rel['source'] == collection:
                # 獲取相關資料
                foreign_id = entity.get(rel['foreign_key'])
                if foreign_id:
                    related = self.store.get(rel['target'], foreign_id)
                    if related:
                        # 遞歸填充
                        related = self.populate_related_data(
                            related, 
                            rel['target'], 
                            depth - 1
                        )
                        entity[rel['target']] = related
        
        return entity
    
    def cascade_operations(self, operation: str, collection: str, entity_id: str):
        """處理級聯操作"""
        if operation == 'delete':
            # 尋找依賴關係
            for rel in self.relationships.values():
                if rel['target'] == collection:
                    # 刪除依賴實體
                    dependents = self.store.query(
                        rel['source'],
                        {rel['foreign_key']: entity_id}
                    )
                    for dep in dependents:
                        self.store.delete(rel['source'], dep['id'])
'''
```

### 8. 測試框架整合

與流行測試框架整合：

**測試整合**
```python
class TestingFrameworkIntegration:
    def create_jest_integration(self):
        """Jest 測試整合"""
        return '''
// jest.mock.config.js
import { MockServer } from './mockServer';

const mockServer = new MockServer();

beforeAll(async () => {
    await mockServer.start({ port: 3001 });
    
    // 載入模擬定義
    await mockServer.loadMocks('./mocks/*.json');
    
    // 設定預設場景
    await mockServer.setScenario('test');
});

afterAll(async () => {
    await mockServer.stop();
});

beforeEach(async () => {
    // 重置模擬狀態
    await mockServer.reset();
});

// 測試輔助函數
export const setupMock = async (stub) => {
    return await mockServer.addStub(stub);
};

export const verifyRequests = async (matcher) => {
    const requests = await mockServer.getRequests(matcher);
    return requests;
};

// 範例測試
describe('使用者 API', () => {
    it('應獲取使用者詳細資訊', async () => {
        // 設定模擬
        await setupMock({
            method: 'GET',
            path: '/api/users/123',
            response: {
                status: 200,
                body: { id: '123', name: '測試使用者' }
            }
        });
        
        // 發出請求
        const response = await fetch('http://localhost:3001/api/users/123');
        const user = await response.json();
        
        // 驗證
        expect(user.name).toBe('測試使用者');
        
        // 驗證模擬是否被調用
        const requests = await verifyRequests({ path: '/api/users/123' });
        expect(requests).toHaveLength(1);
    });
});
'''
    
    def create_pytest_integration(self):
        """Pytest 整合"""
        return '''
# conftest.py
import pytest
from mock_server import MockServer
import asyncio

@pytest.fixture(scope="session")
def event_loop():
    loop = asyncio.get_event_loop_policy().new_event_loop()
    yield loop
    loop.close()

@pytest.fixture(scope="session")
async def mock_server(event_loop):
    server = MockServer()
    await server.start(port=3001)
    yield server
    await server.stop()

@pytest.fixture(autouse=True)
async def reset_mocks(mock_server):
    await mock_server.reset()
    yield
    # 驗證沒有意外調用
    unmatched = await mock_server.get_unmatched_requests()
    assert len(unmatched) == 0, f"未匹配的請求: {unmatched}"

# 測試工具
class MockBuilder:
    def __init__(self, mock_server):
        self.server = mock_server
        self.stubs = []
    
    def when(self, method, path):
        self.current_stub = {
            'method': method,
            'path': path
        }
        return self
    
    def with_body(self, body):
        self.current_stub['body'] = body
        return self
    
    def then_return(self, status, body=None, headers=None):
        self.current_stub['response'] = {
            'status': status,
            'body': body,
            'headers': headers or {}
        }
        self.stubs.append(self.current_stub)
        return self
    
    async def setup(self):
        for stub in self.stubs:
            await self.server.add_stub(stub)

# 範例測試
@pytest.mark.asyncio
async def test_user_creation(mock_server):
    # 設定模擬
    mock = MockBuilder(mock_server)
    mock.when('POST', '/api/users') \
        .with_body({'name': '新使用者'}) \
        .then_return(201, {'id': '456', 'name': '新使用者'})
    
    await mock.setup()
    
    # 在此測試程式碼
    response = await create_user({'name': '新使用者'})
    assert response['id'] == '456'
'''
```

### 9. 模擬伺服器部署

部署模擬伺服器：

**部署配置**
```yaml
# 模擬服務的 docker-compose.yml
version: '3.8'

services:
  mock-api:
    build:
      context: .
      dockerfile: Dockerfile.mock
    ports:
      - "3001:3001"
    environment:
      - MOCK_SCENARIO=production
      - MOCK_DATA_PATH=/data/mocks
    volumes:
      - ./mocks:/data/mocks
      - ./scenarios:/data/scenarios
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  mock-admin:
    build:
      context: .
      dockerfile: Dockerfile.admin
    ports:
      - "3002:3002"
    environment:
      - MOCK_SERVER_URL=http://mock-api:3001
    depends_on:
      - mock-api

# Kubernetes 部署
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mock-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mock-server
  template:
    metadata:
      labels:
        app: mock-server
    spec:
      containers:
      - name: mock-server
        image: mock-server:latest
        ports:
        - containerPort: 3001
        env:
        - name: MOCK_SCENARIO
          valueFrom:
            configMapKeyRef:
              name: mock-config
              key: scenario
        volumeMounts:
        - name: mock-definitions
          mountPath: /data/mocks
      volumes:
      - name: mock-definitions
        configMap:
          name: mock-definitions
```

### 10. 模擬文件

生成模擬 API 文件：

**文件生成器**
```python
class MockDocumentationGenerator:
    def generate_documentation(self, mock_server):
        """生成全面的模擬文件"""
        return f"""
# 模擬 API 文件

## 概述
{self._generate_overview(mock_server)}

## 可用端點
{self._generate_endpoints_doc(mock_server)}

## 場景
{self._generate_scenarios_doc(mock_server)}

## 資料模型
{self._generate_models_doc(mock_server)}

## 使用範例
{self._generate_examples(mock_server)}

## 配置
{self._generate_config_doc(mock_server)}
"""
    
    def _generate_endpoints_doc(self, mock_server):
        """生成端點文件"""
        doc = ""
        for endpoint in mock_server.get_endpoints():
            doc += f"""
### {endpoint['method']} {endpoint['path']}

**描述**: {endpoint.get('description', '無描述')}

**請求**:
```json
{json.dumps(endpoint.get('request_example', {}), indent=2)}
```

**響應**:
```json
{json.dumps(endpoint.get('response_example', {}), indent=2)}
```

**場景**:
{self._format_endpoint_scenarios(endpoint)}
"""
        return doc
    
    def create_interactive_docs(self):
        """建立互動式 API 文件"""
        return '''
<!DOCTYPE html>
<html>
<head>
    <title>模擬 API 互動文件</title>
    <script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
    <link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css">
</head>
<body>
    <div id="swagger-ui"></div>
    <script>
        window.onload = function() {
            const ui = SwaggerUIBundle({
                url: "/api/mock/openapi.json",
                dom_id: '#swagger-ui',
                presets: [
                    SwaggerUIBundle.presets.apis,
                    SwaggerUIBundle.SwaggerUIStandalonePreset
                ],
                layout: "BaseLayout",
                tryItOutEnabled: true,
                requestInterceptor: (request) => {
                    request.headers['X-Mock-Scenario'] = 
                        document.getElementById('scenario-select').value;
                    return request;
                }
            });
        }
    </script>
    
    <div class="scenario-selector">
        <label>場景:</label>
        <select id="scenario-select">
            <option value="default">預設</option>
            <option value="error">錯誤條件</option>
            <option value="slow">慢響應</option>
        </select>
    </div>
</body>
</html>
'''
```

## 輸出格式

1. **模擬伺服器設定**：完整的模擬伺服器實施
2. **存根配置**：靈活的請求/響應存根
3. **資料生成**：逼真的模擬資料生成
4. **場景定義**：全面的測試場景
5. **契約測試**：基於契約的模擬驗證
6. **性能模擬**：性能測試功能
7. **資料管理**：模擬資料儲存和關係
8. **測試整合**：框架整合範例
9. **部署指南**：模擬伺服器部署配置
10. **文件**：自動生成的模擬 API 文件

專注於建立靈活、逼真的模擬服務，以實現高效開發、徹底測試和可靠的 API 模擬，適用於開發生命週期的所有階段。
```