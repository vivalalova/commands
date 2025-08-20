# AI 助理開發

您是 AI 助理開發專家，專精於建立智慧對話介面、聊天機器人和 AI 驅動的應用程式。設計包含自然語言理解、上下文管理和無縫整合的全面 AI 助理解決方案。

## 背景
使用者需要開發具有自然語言功能、智慧響應和實用功能的 AI 助理或聊天機器人。專注於建立為使用者提供真正價值的生產就緒助理。

## 要求
$ARGUMENTS

## 指示

### 1. AI 助理架構

設計全面的助理架構：

**助理架構框架**
```python
from typing import Dict, List, Optional, Any
from dataclasses import dataclass
from abc import ABC, abstractmethod
import asyncio

@dataclass
class ConversationContext:
    """維護對話狀態和上下文"""
    user_id: str
    session_id: str
    messages: List[Dict[str, Any]]
    user_profile: Dict[str, Any]
    conversation_state: Dict[str, Any]
    metadata: Dict[str, Any]

class AIAssistantArchitecture:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.components = self._initialize_components()
        
    def design_architecture(self):
        """設計全面的 AI 助理架構"""
        return {
            'core_components': {
                'nlu': self._design_nlu_component(),
                'dialog_manager': self._design_dialog_manager(),
                'response_generator': self._design_response_generator(),
                'context_manager': self._design_context_manager(),
                'integration_layer': self._design_integration_layer()
            },
            'data_flow': self._design_data_flow(),
            'deployment': self._design_deployment_architecture(),
            'scalability': self._design_scalability_features()
        }
    
    def _design_nlu_component(self):
        """自然語言理解組件"""
        return {
            'intent_recognition': {
                'model': '基於 Transformer 的分類器',
                'features': [
                    '多意圖檢測',
                    '置信度評分',
                    '回退處理'
                ],
                'implementation': '''
class IntentClassifier:
    def __init__(self, model_path: str):
        self.model = self.load_model(model_path)
        self.intents = self.load_intent_schema()
    
    async def classify(self, text: str) -> Dict[str, Any]:
        # 預處理文本
        processed = self.preprocess(text)
        
        # 獲取模型預測
        predictions = await self.model.predict(processed)
        
        # 提取帶有置信度的意圖
        intents = []
        for intent, confidence in predictions:
            if confidence > self.config['threshold']:
                intents.append({
                    'name': intent,
                    'confidence': confidence,
                    'parameters': self.extract_parameters(text, intent)
                })
        
        return {
            'intents': intents,
            'primary_intent': intents[0] if intents else None,
            'requires_clarification': len(intents) > 1
        }
'''
            },
            'entity_extraction': {
                'model': '帶有自定義實體的 NER',
                'features': [
                    '領域特定實體',
                    '上下文提取',
                    '實體解析'
                ]
            },
            'sentiment_analysis': {
                'model': '微調情感分類器',
                'features': [
                    '情感檢測',
                    '緊急性分類',
                    '使用者滿意度追蹤'
                ]
            }
        }
    
    def _design_dialog_manager(self):
        """對話管理系統"""
        return '''
class DialogManager:
    """管理對話流程和狀態"""
    
    def __init__(self):
        self.state_machine = ConversationStateMachine()
        self.policy_network = DialogPolicy()
        
    async def process_turn(self, 
                          context: ConversationContext, 
                          nlu_result: Dict[str, Any]) -> Dict[str, Any]:
        # 確定當前狀態
        current_state = self.state_machine.get_state(context)
        
        # 應用對話策略
        action = await self.policy_network.select_action(
            current_state, 
            nlu_result, 
            context
        )
        
        # 執行動作
        result = await self.execute_action(action, context)
        
        # 更新狀態
        new_state = self.state_machine.transition(
            current_state, 
            action, 
            result
        )
        
        return {
            'action': action,
            'new_state': new_state,
            'response_data': result
        }
    
    async def execute_action(self, action: str, context: ConversationContext):
        """執行對話動作"""
        action_handlers = {
            'greet': self.handle_greeting,
            'provide_info': self.handle_information_request,
            'clarify': self.handle_clarification,
            'confirm': self.handle_confirmation,
            'execute_task': self.handle_task_execution,
            'end_conversation': self.handle_conversation_end
        }
        
        handler = action_handlers.get(action, self.handle_unknown)
        return await handler(context)
'''
```

### 2. 自然語言處理

實施進階 NLP 功能：

**NLP 管道實施**
```python
class NLPPipeline:
    def __init__(self):
        self.tokenizer = self._initialize_tokenizer()
        self.embedder = self._initialize_embedder()
        self.models = self._load_models()
    
    async def process_message(self, message: str, context: ConversationContext):
        """透過 NLP 管道處理使用者訊息"""
        # 分詞和預處理
        tokens = self.tokenizer.tokenize(message)
        
        # 生成嵌入
        embeddings = await self.embedder.embed(tokens)
        
        # NLP 任務的平行處理
        tasks = [
            self.detect_intent(embeddings),
            self.extract_entities(tokens, embeddings),
            self.analyze_sentiment(embeddings),
            self.detect_language(tokens),
            self.check_spelling(tokens)
        ]
        
        results = await asyncio.gather(*tasks)
        
        return {
            'intent': results[0],
            'entities': results[1],
            'sentiment': results[2],
            'language': results[3],
            'corrections': results[4],
            'original_message': message,
            'processed_tokens': tokens
        }
    
    async def detect_intent(self, embeddings):
        """進階意圖檢測"""
        # 多標籤分類
        intent_scores = await self.models['intent_classifier'].predict(embeddings)
        
        # 分層意圖檢測
        primary_intent = self.get_primary_intent(intent_scores)
        sub_intents = self.get_sub_intents(primary_intent, embeddings)
        
        return {
            'primary': primary_intent,
            'secondary': sub_intents,
            'confidence': max(intent_scores.values()),
            'all_scores': intent_scores
        }
    
    def extract_entities(self, tokens, embeddings):
        """提取和解析實體"""
        # 命名實體識別
        entities = self.models['ner'].extract(tokens, embeddings)
        
        # 實體連結和解析
        resolved_entities = []
        for entity in entities:
            resolved = self.resolve_entity(entity)
            resolved_entities.append({
                'text': entity['text'],
                'type': entity['type'],
                'resolved_value': resolved['value'],
                'confidence': resolved['confidence'],
                'alternatives': resolved.get('alternatives', [])
            })
        
        return resolved_entities
    
    def build_semantic_understanding(self, nlu_result, context):
        """建立使用者意圖的語義表示"""
        return {
            'user_goal': self.infer_user_goal(nlu_result, context),
            'required_information': self.identify_missing_info(nlu_result),
            'constraints': self.extract_constraints(nlu_result),
            'preferences': self.extract_preferences(nlu_result, context)
        }
```

### 3. 對話流程設計

設計智慧對話流程：

**對話流程引擎**
```python
class ConversationFlowEngine:
    def __init__(self):
        self.flows = self._load_conversation_flows()
        self.state_tracker = StateTracker()
        
    def design_conversation_flow(self):
        """設計多輪對話流程"""
        return {
            'greeting_flow': {
                'triggers': ['你好', '嗨', '問候'],
                'nodes': [
                    {
                        'id': '問候使用者',
                        'type': '響應',
                        'content': self.personalized_greeting,
                        'next': '詢問如何協助'
                    },
                    {
                        'id': '詢問如何協助',
                        'type': '問題',
                        'content': "今天我能為您提供什麼協助？",
                        'expected_intents': ['請求協助', '提問'],
                        'timeout': 30,
                        'timeout_action': '提供建議'
                    }
                ]
            },
            'task_completion_flow': {
                'triggers': ['任務請求'],
                'nodes': [
                    {
                        'id': '理解任務',
                        'type': 'nlu_處理',
                        'extract': ['任務類型', '參數'],
                        'next': '檢查要求'
                    },
                    {
                        'id': '檢查要求',
                        'type': '驗證',
                        'validate': self.validate_task_requirements,
                        'on_success': '確認任務',
                        'on_missing': '請求缺少資訊'
                    },
                    {
                        'id': '請求缺少資訊',
                        'type': '槽位填充',
                        'slots': self.get_required_slots,
                        'prompts': self.get_slot_prompts,
                        'next': '確認任務'
                    },
                    {
                        'id': '確認任務',
                        'type': '確認',
                        'content': self.generate_task_summary,
                        'on_confirm': '執行任務',
                        'on_deny': '澄清任務'
                    }
                ]
            }
        }
    
    async def execute_flow(self, flow_id: str, context: ConversationContext):
        """執行對話流程"""
        flow = self.flows[flow_id]
        current_node = flow['nodes'][0]
        
        while current_node:
            result = await self.execute_node(current_node, context)
            
            # 確定下一個節點
            if result.get('user_input'):
                next_node_id = self.determine_next_node(
                    current_node, 
                    result['user_input'],
                    context
                )
            else:
                next_node_id = current_node.get('next')
            
            current_node = self.get_node(flow, next_node_id)
            
            # 更新上下文
            context.conversation_state.update(result.get('state_updates', {}))
        
        return context
```

### 4. 響應生成

建立智慧響應生成：

**響應生成器**
```python
class ResponseGenerator:
    def __init__(self, llm_client=None):
        self.llm = llm_client
        self.templates = self._load_response_templates()
        self.personality = self._load_personality_config()
        
    async def generate_response(self, 
                               intent: str, 
                               context: ConversationContext,
                               data: Dict[str, Any]) -> str:
        """生成上下文響應"""
        
        # 選擇響應策略
        if self.should_use_template(intent):
            response = self.generate_from_template(intent, data)
        elif self.should_use_llm(intent, context):
            response = await self.generate_with_llm(intent, context, data)
        else:
            response = self.generate_hybrid_response(intent, context, data)
        
        # 應用個性和語氣
        response = self.apply_personality(response, context)
        
        # 確保響應適當性
        response = self.validate_response(response, context)
        
        return response
    
    async def generate_with_llm(self, intent, context, data):
        """使用 LLM 生成響應"""
        # 建構提示
        prompt = self.build_llm_prompt(intent, context, data)
        
        # 設定生成參數
        params = {
            'temperature': self.get_temperature(intent),
            'max_tokens': 150,
            'stop_sequences': ['\n\n', '使用者:', '人類:']
        }
        
        # 生成響應
        response = await self.llm.generate(prompt, **params)
        
        # 後處理響應
        return self.post_process_llm_response(response)
    
    def build_llm_prompt(self, intent, context, data):
        """為 LLM 建立上下文感知提示"""
        return f"""
您是一位樂於助人的 AI 助理，具有以下特點：
{self.personality.description}

對話歷史：
{self.format_conversation_history(context.messages[-5:])}

使用者意圖：{intent}
相關資料：{json.dumps(data, indent=2)}

生成一個有幫助、簡潔的響應，該響應：
1. 解決使用者的意圖
2. 適當使用提供的資料
3. 維持對話連續性
4. 遵循個性指南

響應："""
    
    def generate_from_template(self, intent, data):
        """從模板生成響應"""
        template = self.templates.get(intent)
        if not template:
            return self.get_fallback_response()
        
        # 選擇模板變體
        variant = self.select_template_variant(template, data)
        
        # 填充模板槽位
        response = variant
        for key, value in data.items():
            response = response.replace(f"{{{key}}}", str(value))
        
        return response
    
    def apply_personality(self, response, context):
        """將個性特徵應用於響應"""
        # 添加個性標記
        if self.personality.get('友好'):
            response = self.add_friendly_markers(response)
        
        if self.personality.get('專業'):
            response = self.ensure_professional_tone(response)
        
        # 根據使用者偏好調整
        if context.user_profile.get('偏好簡潔'):
            response = self.make_concise(response)
        
        return response
```

### 5. 上下文管理

實施複雜的上下文管理：

**上下文管理系統**
```python
class ContextManager:
    def __init__(self):
        self.short_term_memory = ShortTermMemory()
        self.long_term_memory = LongTermMemory()
        self.working_memory = WorkingMemory()
        
    async def manage_context(self, 
                            new_input: Dict[str, Any],
                            current_context: ConversationContext) -> ConversationContext:
        """管理對話上下文"""
        
        # 更新對話歷史
        current_context.messages.append({
            'role': '使用者',
            'content': new_input['message'],
            'timestamp': datetime.now(),
            'metadata': new_input.get('metadata', {})
        })
        
        # 解析引用
        resolved_input = await self.resolve_references(new_input, current_context)
        
        # 更新工作記憶體
        self.working_memory.update(resolved_input, current_context)
        
        # 檢測主題變更
        topic_shift = self.detect_topic_shift(resolved_input, current_context)
        if topic_shift:
            current_context = self.handle_topic_shift(topic_shift, current_context)
        
        # 維護實體狀態
        current_context = self.update_entity_state(resolved_input, current_context)
        
        # 如果需要，修剪舊上下文
        if len(current_context.messages) > self.config['max_context_length']:
            current_context = self.prune_context(current_context)
        
        return current_context
    
    async def resolve_references(self, input_data, context):
        """解析代詞和引用"""
        text = input_data['message']
        
        # 代詞解析
        pronouns = self.extract_pronouns(text)
        for pronoun in pronouns:
            referent = self.find_referent(pronoun, context)
            if referent:
                text = text.replace(pronoun['text'], referent['resolved'])
        
        # 時間引用解析
        temporal_refs = self.extract_temporal_references(text)
        for ref in temporal_refs:
            resolved_time = self.resolve_temporal_reference(ref, context)
            text = text.replace(ref['text'], str(resolved_time))
        
        input_data['resolved_message'] = text
        return input_data
    
    def maintain_entity_state(self):
        """在對話中追蹤實體狀態"""
        return '''
class EntityStateTracker:
    def __init__(self):
        self.entities = {}
        
    def update_entity(self, entity_id: str, updates: Dict[str, Any]):
        """更新實體狀態"""
        if entity_id not in self.entities:
            self.entities[entity_id] = {
                'id': entity_id,
                'type': updates.get('type'),
                'attributes': {},
                'history': []
            }
        
        # 記錄歷史
        self.entities[entity_id]['history'].append({
            'timestamp': datetime.now(),
            'updates': updates
        })
        
        # 應用更新
        self.entities[entity_id]['attributes'].update(updates)
    
    def get_entity_state(self, entity_id: str) -> Optional[Dict[str, Any]]:
        """獲取當前實體狀態"""
        return self.entities.get(entity_id)
    
    def query_entities(self, entity_type: str = None, **filters):
        """按類型和屬性查詢實體"""
        results = []
        for entity in self.entities.values():
            if entity_type and entity['type'] != entity_type:
                continue
            
            matches = True
            for key, value in filters.items():
                if entity['attributes'].get(key) != value:
                    matches = False
                    break
            
            if matches:
                results.append(entity)
        
        return results
'''
```

### 6. 與 LLM 整合

與各種 LLM 提供者整合：

**LLM 整合層**
```python
class LLMIntegrationLayer:
    def __init__(self):
        self.providers = {
            'openai': OpenAIProvider(),
            'anthropic': AnthropicProvider(),
            'local': LocalLLMProvider()
        }
        self.current_provider = None
        
    async def setup_llm_integration(self, provider: str, config: Dict[str, Any]):
        """設定 LLM 整合"""
        self.current_provider = self.providers[provider]
        await self.current_provider.initialize(config)
        
        return {
            'provider': provider,
            'capabilities': self.current_provider.get_capabilities(),
            'rate_limits': self.current_provider.get_rate_limits()
        }
    
    async def generate_completion(self, 
                                 prompt: str,
                                 system_prompt: str = None,
                                 **kwargs):
        """生成帶有回退處理的完成"""
        try:
            # 主要嘗試
            response = await self.current_provider.complete(
                prompt=prompt,
                system_prompt=system_prompt,
                **kwargs
            )
            
            # 驗證響應
            if self.is_valid_response(response):
                return response
            else:
                return await self.handle_invalid_response(prompt, response)
                
        except RateLimitError:
            # 切換到回退提供者
            return await self.use_fallback_provider(prompt, system_prompt, **kwargs)
        except Exception as e:
            # 記錄錯誤並使用快取響應（如果可用）
            return self.get_cached_response(prompt) or self.get_default_response()
    
    def create_function_calling_interface(self):
        """為 LLM 建立函數調用介面"""
        return '''
class FunctionCallingInterface:
    def __init__(self):
        self.functions = {}
        
    def register_function(self, 
                         name: str,
                         func: callable,
                         description: str,
                         parameters: Dict[str, Any]):
        """註冊 LLM 調用的函數"""
        self.functions[name] = {
            'function': func,
            'description': description,
            'parameters': parameters
        }
    
    async def process_function_call(self, llm_response):
        """處理 LLM 的函數調用"""
        if 'function_call' not in llm_response:
            return llm_response
        
        function_name = llm_response['function_call']['name']
        arguments = llm_response['function_call']['arguments']
        
        if function_name not in self.functions:
            return {'error': f'未知函數: {function_name}'}
        
        # 驗證參數
        validated_args = self.validate_arguments(
            function_name, 
            arguments
        )
        
        # 執行函數
        result = await self.functions[function_name]['function'](**validated_args)
        
        # 返回結果供 LLM 處理
        return {
            'function_result': result,
            'function_name': function_name
        }
'''
```

### 7. 對話式 AI 測試

實施全面的測試：

**對話測試框架**
```python
class ConversationTestFramework:
    def __init__(self):
        self.test_suites = []
        self.metrics = ConversationMetrics()
        
    def create_test_suite(self):
        """建立全面的測試套件"""
        return {
            'unit_tests': self._create_unit_tests(),
            'integration_tests': self._create_integration_tests(),
            'conversation_tests': self._create_conversation_tests(),
            'performance_tests': self._create_performance_tests(),
            'user_simulation': self._create_user_simulation()
        }
    
    def _create_conversation_tests(self):
        """測試多輪對話"""
        return '''
class ConversationTest:
    async def test_multi_turn_conversation(self):
        """測試完整的對話流程"""
        assistant = AIAssistant()
        context = ConversationContext(user_id="測試使用者")
        
        # 對話腳本
        conversation = [
            {
                'user': "你好，我需要協助處理我的訂單",
                'expected_intent': '訂單協助',
                'expected_action': '詢問訂單詳細資訊'
            },
            {
                'user': "我的訂單號是 12345",
                'expected_entities': [{'type': '訂單 ID', 'value': '12345'}],
                'expected_action': '檢索訂單'
            },
            {
                'user': "什麼時候會到？",
                'expected_intent': '交付查詢',
                'should_use_context': True
            }
        ]
        
        for turn in conversation:
            # 發送使用者訊息
            response = await assistant.process_message(
                turn['user'], 
                context
            )
            
            # 驗證意圖檢測
            if 'expected_intent' in turn:
                assert response['intent'] == turn['expected_intent']
            
            # 驗證實體提取
            if 'expected_entities' in turn:
                self.validate_entities(
                    response['entities'], 
                    turn['expected_entities']
                )
            
            # 驗證上下文使用
            if turn.get('should_use_context'):
                assert 'order_id' in response['context_used']
    
    def test_error_handling(self):
        """測試錯誤處理場景"""
        error_cases = [
            {
                'input': "askdjfkajsdf",
                'expected_behavior': '回退響應'
            },
            {
                'input': "我想要 [已編輯]",
                'expected_behavior': '安全響應'
            },
            {
                'input': "告訴我關於 " + "x" * 1000,
                'expected_behavior': '長度限制響應'
            }
        ]
        
        for case in error_cases:
            response = assistant.process_message(case['input'])
            assert response['behavior'] == case['expected_behavior']
'''
    
    def create_automated_testing(self):
        """自動化對話測試"""
        return '''
class AutomatedConversationTester:
    def __init__(self):
        self.test_generator = TestCaseGenerator()
        self.evaluator = ResponseEvaluator()
        
    async def run_automated_tests(self, num_tests: int = 100):
        """運行自動化對話測試"""
        results = {
            'total_tests': num_tests,
            'passed': 0,
            'failed': 0,
            'metrics': {}
        }
        
        for i in range(num_tests):
            # 生成測試案例
            test_case = self.test_generator.generate()
            
            # 運行對話
            conversation_log = await self.run_conversation(test_case)
            
            # 評估結果
            evaluation = self.evaluator.evaluate(
                conversation_log,
                test_case['expectations']
            )
            
            if evaluation['passed']:
                results['passed'] += 1
            else:
                results['failed'] += 1
                
            # 收集指標
            self.update_metrics(results['metrics'], evaluation['metrics'])
        
        return results
    
    def generate_adversarial_tests(self):
        """生成對抗性測試案例"""
        return [
            # 模糊輸入
            "我想要我們討論過的那件事",
            
            # 上下文切換
            "其實，忘記那個。告訴我天氣如何",
            
            #  একাধিক intents
            "取消我的訂單並更新我的地址",
            
            # 資訊不完整
            "預訂航班",
            
            # 矛盾
            "我想要一份帶培根的素食餐點"
        ]
'''
```

### 8. 部署與擴展

部署和擴展 AI 助理：

**部署架構**
```python
class AssistantDeployment:
    def create_deployment_architecture(self):
        """建立可擴展的部署架構"""
        return {
            'containerization': '''
# AI 助理的 Dockerfile
FROM python:3.11-slim

WORKDIR /app

# 安裝依賴項
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 複製應用程式
COPY . .

# 在建置時載入模型
RUN python -m app.model_loader

# 暴露埠
EXPOSE 8080

# 健康檢查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD python -m app.health_check

# 運行應用程式
CMD ["gunicorn", "--worker-class", "uvicorn.workers.UvicornWorker", \
     "--workers", "4", "--bind", "0.0.0.0:8080", "app.main:app"]
''',
            'kubernetes_deployment': '''
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ai-assistant
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ai-assistant
  template:
    metadata:
      labels:
        app: ai-assistant
    spec:
      containers:
      - name: assistant
        image: ai-assistant:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        env:
        - name: MODEL_CACHE_SIZE
          value: "1000"
        - name: MAX_CONCURRENT_SESSIONS
          value: "100"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: ai-assistant-service
spec:
  selector:
    app: ai-assistant
  ports:
  - port: 80
    targetPort: 8080
  type: LoadBalancer
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ai-assistant-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ai-assistant
  minReplicas: 3
  maxReplicas: 10
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
''',
            'caching_strategy': self._design_caching_strategy(),
            'load_balancing': self._design_load_balancing()
        }
    
    def _design_caching_strategy(self):
        """設計性能快取策略"""
        return '''
class AssistantCache:
    def __init__(self):
        self.response_cache = ResponseCache()
        self.model_cache = ModelCache()
        self.context_cache = ContextCache()
        
    async def get_cached_response(self, 
                                 message: str, 
                                 context_hash: str) -> Optional[str]:
        """獲取快取響應（如果可用）"""
        cache_key = self.generate_cache_key(message, context_hash)
        
        # 檢查響應快取
        cached = await self.response_cache.get(cache_key)
        if cached and not self.is_expired(cached):
            return cached['response']
        
        return None
    
    def cache_response(self, 
                      message: str,
                      context_hash: str,
                      response: str,
                      ttl: int = 3600):
        """快取帶有 TTL 的響應"""
        cache_key = self.generate_cache_key(message, context_hash);
        
        self.response_cache.set(
            cache_key,
            {
                'response': response,
                'timestamp': datetime.now(),
                'ttl': ttl
            }
        )
    
    def preload_model_cache(self):
        """預載入常用模型"""
        models_to_cache = [
            '意圖分類器',
            '實體提取器',
            '響應生成器'
        ]
        
        for model_name in models_to_cache:
            model = load_model(model_name)
            self.model_cache.store(model_name, model)
'''
```

### 9. 監控與分析

監控助理性能：

**助理分析系統**
```python
class AssistantAnalytics:
    def __init__(self):
        self.metrics_collector = MetricsCollector()
        self.analytics_engine = AnalyticsEngine()
        
    def create_monitoring_dashboard(self):
        """建立監控儀表板配置"""
        return {
            'real_time_metrics': {
                '活躍會話': 'gauge',
                '每秒訊息數': 'counter',
                '響應時間 p95': 'histogram',
                '意圖準確度': 'gauge',
                '回退率': 'gauge'
            },
            'conversation_metrics': {
                '平均對話長度': 'gauge',
                '完成率': 'gauge',
                '使用者滿意度': 'gauge',
                '升級率': 'gauge'
            },
            'system_metrics': {
                '模型推斷時間': 'histogram',
                '快取命中率': 'gauge',
                '錯誤率': 'counter',
                '資源利用率': 'gauge'
            },
            'alerts': [
                {
                    'name': '高回退率',
                    'condition': '回退率 > 0.2',
                    'severity': 'warning'
                },
                {
                    'name': '響應時間慢',
                    'condition': '響應時間 p95 > 2000',
                    'severity': 'critical'
                }
            ]
        }
    
    def analyze_conversation_quality(self):
        """分析對話品質指標"""
        return '''
class ConversationQualityAnalyzer:
    def analyze_conversations(self, time_range: str):
        """分析對話品質"""
        conversations = self.fetch_conversations(time_range)
        
        metrics = {
            'intent_recognition': self.analyze_intent_accuracy(conversations),
            'response_relevance': self.analyze_response_relevance(conversations),
            'conversation_flow': self.analyze_conversation_flow(conversations),
            'user_satisfaction': self.analyze_satisfaction(conversations),
            'error_patterns': self.identify_error_patterns(conversations)
        }
        
        return self.generate_quality_report(metrics)
    
    def identify_improvement_areas(self, analysis):
        """識別改進領域"""
        improvements = []
        
        # 意圖準確度低
        if analysis['intent_recognition']['accuracy'] < 0.85:
            improvements.append({
                'area': '意圖識別',
                'issue': '意圖檢測準確度低',
                'recommendation': '使用更多範例重新訓練意圖分類器',
                'priority': '高'
            })
        
        # 回退率高
        if analysis['conversation_flow']['fallback_rate'] > 0.15:
            improvements.append({
                'area': '覆蓋率',
                'issue': '回退率高',
                'recommendation': '擴展訓練資料以涵蓋未涵蓋的意圖',
                'priority': '中'
            })
        
        return improvements
'''
```

### 10. 持續改進

實施持續改進循環：

**改進管道**
```python
class ContinuousImprovement:
    def create_improvement_pipeline(self):
        """建立持續改進管道"""
        return {
            'data_collection': '''
class ConversationDataCollector:
    async def collect_feedback(self, session_id: str):
        """收集使用者回饋"""
        feedback_prompt = {
            '滿意度': '您對這次對話的滿意度如何？(1-5)',
            '已解決': '您的問題是否已解決？',
            '改進': '我們如何改進？'
        }
        
        feedback = await self.prompt_user_feedback(
            session_id, 
            feedback_prompt
        )
        
        # 儲存回饋
        await self.store_feedback({
            'session_id': session_id,
            'timestamp': datetime.now(),
            'feedback': feedback,
            'conversation_metadata': self.get_session_metadata(session_id)
        })
        
        return feedback
    
    def identify_training_opportunities(self):
        """識別用於訓練的對話機會"""
        # 尋找低置信度互動
        low_confidence = self.find_low_confidence_interactions()
        
        # 尋找失敗的對話
        failed = self.find_failed_conversations()
        
        # 尋找高評價的對話
        exemplary = self.find_exemplary_conversations()
        
        return {
            '需要改進': low_confidence + failed,
            '好範例': exemplary
        }
''',
            'model_retraining': '''
class ModelRetrainer:
    async def retrain_models(self, new_data):
        """使用新資料重新訓練模型"""
        # 準備訓練資料
        training_data = self.prepare_training_data(new_data)
        
        # 驗證資料品質
        validation_result = self.validate_training_data(training_data)
        if not validation_result['passed']:
            return {'error': '資料品質檢查失敗', 'issues': validation_result['issues']}
        
        # 重新訓練模型
        models_to_retrain = ['意圖分類器', '實體提取器']
        
        for model_name in models_to_retrain:
            # 載入當前模型
            current_model = self.load_model(model_name)
            
            # 建立新版本
            new_model = await self.train_model(
                model_name,
                training_data,
                base_model=current_model
            )
            
            # 評估新模型
            evaluation = await self.evaluate_model(
                new_model,
                self.get_test_set()
            )
            
            # 如果改進，則部署
            if evaluation['performance'] > current_model.performance:
                await self.deploy_model(new_model, model_name)
        
        return {'status': '已完成', 'models_updated': models_to_retrain}
''',
            'a_b_testing': '''
class ABTestingFramework:
    def create_ab_test(self, 
                      test_name: str,
                      variants: List[Dict[str, Any]],
                      metrics: List[str]):
        """為助理改進建立 A/B 測試"""
        test = {
            'id': generate_test_id(),
            'name': test_name,
            'variants': variants,
            'metrics': metrics,
            'allocation': self.calculate_traffic_allocation(variants),
            'duration': self.estimate_test_duration(metrics)
        }
        
        # 部署測試
        self.deploy_test(test)
        
        return test
    
    async def analyze_test_results(self, test_id: str):
        """分析 A/B 測試結果"""
        data = await self.collect_test_data(test_id)
        
        results = {}
        for metric in data['metrics']:
            # 統計分析
            analysis = self.statistical_analysis(
                data['control'][metric],
                data['variant'][metric]
            )
            
            results[metric] = {
                'control_mean': analysis['control_mean'],
                'variant_mean': analysis['variant_mean'],
                'lift': analysis['lift'],
                'p_value': analysis['p_value'],
                'significant': analysis['p_value'] < 0.05
            }
        
        return results
'''
        }
```

## 輸出格式

1. **架構設計**：帶有組件的完整 AI 助理架構
2. **NLP 實施**：自然語言處理管道和模型
3. **對話流程**：對話管理和流程設計
4. **響應生成**：帶有 LLM 整合的智慧響應建立
5. **上下文管理**：複雜的上下文和狀態管理
6. **測試框架**：對話式 AI 的全面測試
7. **部署指南**：可擴展的部署架構
8. **監控設定**：分析和性能監控
9. **改進管道**：持續改進流程

專注於建立生產就緒的 AI 助理，透過自然對話、智慧響應和從使用者互動中持續學習來提供真正的價值
