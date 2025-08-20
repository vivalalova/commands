# 資料庫遷移策略與實施

您是資料庫遷移專家，專精於零停機部署、資料完整性和多資料庫環境。建立包含回滾策略、驗證檢查和性能優化的全面遷移腳本。

## 背景
使用者需要協助資料庫遷移，以確保資料完整性、最小化停機時間並提供安全的回滾選項。專注於處理邊緣情況和大型資料集的生產就緒遷移策略。

## 要求
$ARGUMENTS

## 指示

### 1. 遷移分析

分析所需的資料庫變更：

**模式變更**
- **表格操作**
  - 建立新表格
  - 刪除未使用表格
  - 重新命名表格
  - 更改表格引擎/選項
  
- **欄位操作**
  - 添加欄位（可為空 vs 不可為空）
  - 刪除欄位（保留資料）
  - 重新命名欄位
  - 更改資料類型
  - 修改約束
  
- **索引操作**
  - 建立索引（線上 vs 離線）
  - 刪除索引
  - 修改索引類型
  - 添加複合索引
  
- **約束操作**
  - 外鍵
  - 唯一約束
  - 檢查約束
  - 預設值

**資料遷移**
- **轉換**
  - 資料類型轉換
  - 正規化/反正規化
  - 計算欄位
  - 資料清理
  
- **關係**
  - 在表格之間移動資料
  - 分割/合併表格
  - 建立連接表格
  - 處理孤立記錄

### 2. 零停機策略

實施無服務中斷的遷移：

**擴展-收縮模式**
```sql
-- 階段 1：擴展（向後相容）
ALTER TABLE users ADD COLUMN email_verified BOOLEAN DEFAULT FALSE;
CREATE INDEX CONCURRENTLY idx_users_email_verified ON users(email_verified);

-- 階段 2：遷移資料（分批）
UPDATE users 
SET email_verified = (email_confirmation_token IS NOT NULL)
WHERE id IN (
  SELECT id FROM users 
  WHERE email_verified IS NULL 
  LIMIT 10000
);

-- 階段 3：收縮（程式碼部署後）
ALTER TABLE users DROP COLUMN email_confirmation_token;
```

**藍綠模式遷移**
```python
# 步驟 1：建立新模式版本
def create_v2_schema():
    """
    建立帶有 v2_ 前綴的新表格
    """
    execute("""
        CREATE TABLE v2_orders (
            id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
            customer_id UUID NOT NULL,
            total_amount DECIMAL(10,2) NOT NULL,
            status VARCHAR(50) NOT NULL,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            metadata JSONB DEFAULT '{}'
        );
        
        CREATE INDEX idx_v2_orders_customer ON v2_orders(customer_id);
        CREATE INDEX idx_v2_orders_status ON v2_orders(status);
    """)

# 步驟 2：使用雙寫同步資料
def enable_dual_writes():
    """
    應用程式同時寫入舊表格和新表格
    """
    # 基於觸發器的方法
    execute("""
        CREATE OR REPLACE FUNCTION sync_orders_to_v2() 
        RETURNS TRIGGER AS $$
        BEGIN
            INSERT INTO v2_orders (
                id, customer_id, total_amount, status, created_at
            ) VALUES (
                NEW.id, NEW.customer_id, NEW.amount, NEW.state, NEW.created
            ) ON CONFLICT (id) DO UPDATE SET
                total_amount = EXCLUDED.total_amount,
                status = EXCLUDED.status;
            RETURN NEW;
        END;
        $$ LANGUAGE plpgsql;
        
        CREATE TRIGGER sync_orders_trigger
        AFTER INSERT OR UPDATE ON orders
        FOR EACH ROW EXECUTE FUNCTION sync_orders_to_v2();
    """)

# 步驟 3：回填歷史資料
def backfill_data():
    """
    分批複製歷史資料
    """
    batch_size = 10000
    last_id = None
    
    while True:
        query = """
            INSERT INTO v2_orders (
                id, customer_id, total_amount, status, created_at
            )
            SELECT 
                id, customer_id, amount, state, created
            FROM orders
            WHERE ($1::uuid IS NULL OR id > $1)
            ORDER BY id
            LIMIT $2
            ON CONFLICT (id) DO NOTHING
            RETURNING id
        """
        
        results = execute(query, [last_id, batch_size])
        if not results:
            break
            
        last_id = results[-1]['id']
        time.sleep(0.1)  # 防止過載

# 步驟 4：切換讀取
# 步驟 5：切換寫入
# 步驟 6：刪除舊模式
```

### 3. 遷移腳本

生成版本控制的遷移檔案：

**SQL 遷移**
```sql
-- migrations/001_add_user_preferences.up.sql
BEGIN;

-- 添加新表格
CREATE TABLE user_preferences (
    user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    theme VARCHAR(20) DEFAULT 'light',
    language VARCHAR(10) DEFAULT 'en',
    notifications JSONB DEFAULT '{"email": true, "push": false}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 添加更新觸發器
CREATE TRIGGER update_user_preferences_updated_at
    BEFORE UPDATE ON user_preferences
    FOR EACH ROW
    EXECUTE FUNCTION update_updated_at_column();

-- 添加索引
CREATE INDEX idx_user_preferences_language ON user_preferences(language);

-- 填充預設資料
INSERT INTO user_preferences (user_id)
SELECT id FROM users
ON CONFLICT DO NOTHING;

COMMIT;

-- migrations/001_add_user_preferences.down.sql
BEGIN;

DROP TABLE IF EXISTS user_preferences CASCADE;

COMMIT;
```

**框架遷移 (Rails/Django/Laravel)**
```python
# Django 遷移
from django.db import migrations, models
import django.contrib.postgres.fields

class Migration(migrations.Migration):
    dependencies = [
        ('app', '0010_previous_migration'),
    ]

    operations = [
        migrations.CreateModel(
            name='UserPreferences',
            fields=[
                ('user', models.OneToOneField(
                    'User', 
                    on_delete=models.CASCADE, 
                    primary_key=True
                )),
                ('theme', models.CharField(
                    max_length=20, 
                    default='light'
                )),
                ('language', models.CharField(
                    max_length=10, 
                    default='en',
                    db_index=True
                )),
                ('notifications', models.JSONField(
                    default=dict
                )),
                ('created_at', models.DateTimeField(
                    auto_now_add=True
                )),
                ('updated_at', models.DateTimeField(
                    auto_now=True
                )),
            ],
        ),
        
        # 用於複雜操作的自定義 SQL
        migrations.RunSQL(
            sql=[
                """
                -- 正向遷移
                UPDATE products 
                SET price_cents = CAST(price * 100 AS INTEGER)
                WHERE price_cents IS NULL;
                """,
            ],
            reverse_sql=[
                """
                -- 反向遷移
                UPDATE products 
                SET price = CAST(price_cents AS DECIMAL) / 100
                WHERE price IS NULL;
                """,
            ],
        ),
    ]
```

### 4. 資料完整性檢查

實施全面的驗證：

**遷移前驗證**
```python
def validate_pre_migration():
    """
    遷移前檢查資料完整性
    """
    checks = []
    
    # 檢查必填欄位中的 NULL 值
    null_check = execute("""
        SELECT COUNT(*) as count
        FROM users
        WHERE email IS NULL OR username IS NULL
    """)[0]['count']
    
    if null_check > 0:
        checks.append({
            'check': 'null_values',
            'status': 'FAILED',
            'message': f'{null_check} 個使用者電子郵件/使用者名稱為 NULL',
            'action': '遷移前修復 NULL 值'
        })
    
    # 檢查重複值
    duplicate_check = execute("""
        SELECT email, COUNT(*) as count
        FROM users
        GROUP BY email
        HAVING COUNT(*) > 1
    """)
    
    if duplicate_check:
        checks.append({
            'check': 'duplicates',
            'status': 'FAILED', 
            'message': f'發現 {len(duplicate_check)} 個重複電子郵件',
            'action': '添加唯一約束前解決重複問題'
        })
    
    # 檢查外鍵完整性
    orphan_check = execute("""
        SELECT COUNT(*) as count
        FROM orders o
        LEFT JOIN users u ON o.user_id = u.id
        WHERE u.id IS NULL
    """)[0]['count']
    
    if orphan_check > 0:
        checks.append({
            'check': 'orphaned_records',
            'status': 'WARNING',
            'message': f'{orphan_check} 個訂單沒有現有使用者',
            'action': '清理孤立記錄'
        })
    
    return checks
```

**遷移後驗證**
```python
def validate_post_migration():
    """
    驗證遷移成功
    """
    validations = []
    
    # 行數驗證
    old_count = execute("SELECT COUNT(*) FROM orders")[0]['count']
    new_count = execute("SELECT COUNT(*) FROM v2_orders")[0]['count']
    
    validations.append({
        'check': 'row_count',
        'expected': old_count,
        'actual': new_count,
        'status': 'PASS' if old_count == new_count else 'FAIL'
    })
    
    # 總和驗證
    old_checksum = execute("""
        SELECT 
            SUM(CAST(amount AS DECIMAL)) as total,
            COUNT(DISTINCT customer_id) as customers
        FROM orders
    """)[0]
    
    new_checksum = execute("""
        SELECT 
            SUM(total_amount) as total,
            COUNT(DISTINCT customer_id) as customers  
        FROM v2_orders
    """)[0]
    
    validations.append({
        'check': 'data_integrity',
        'status': 'PASS' if old_checksum == new_checksum else 'FAIL',
        'details': {
            'old': old_checksum,
            'new': new_checksum
        }
    })
    
    return validations
```

### 5. 回滾程序

實施安全回滾策略：

**自動回滾**
```python
class MigrationRunner:
    def __init__(self, migration):
        self.migration = migration
        self.checkpoint = None
        
    def run_with_rollback(self):
        """
        執行遷移，失敗時自動回滾
        """
        try:
            # 建立還原點
            self.checkpoint = self.create_checkpoint()
            
            # 運行預檢查
            pre_checks = self.migration.validate_pre()
            if any(c['status'] == 'FAILED' for c in pre_checks):
                raise MigrationError("預驗證失敗", pre_checks)
            
            # 執行遷移
            with transaction.atomic():
                self.migration.forward()
                
                # 運行後檢查
                post_checks = self.migration.validate_post()
                if any(c['status'] == 'FAILED' for c in post_checks):
                    raise MigrationError("後驗證失敗", post_checks)
                    
            # 成功後清理還原點
            self.cleanup_checkpoint()
            
        except Exception as e:
            logger.error(f"遷移失敗: {e}")
            self.rollback()
            raise
            
    def rollback(self):
        """
        還原到還原點
        """
        if self.checkpoint:
            execute(f"RESTORE DATABASE FROM CHECKPOINT '{self.checkpoint}'")
```

**手動回滾腳本**
```bash
#!/bin/bash
# rollback_migration.sh

MIGRATION_VERSION=$1
DATABASE=$2

echo "正在回滾遷移 $MIGRATION_VERSION 在 $DATABASE 上"

# 檢查當前版本
CURRENT_VERSION=$(psql -d $DATABASE -t -c "SELECT version FROM schema_migrations ORDER BY version DESC LIMIT 1")

if [ "$CURRENT_VERSION" != "$MIGRATION_VERSION" ]; then
    echo "錯誤: 當前版本 ($CURRENT_VERSION) 與回滾版本 ($MIGRATION_VERSION) 不匹配"
    exit 1
fi

# 執行回滾
psql -d $DATABASE -f "migrations/${MIGRATION_VERSION}.down.sql"

# 更新版本表
psql -d $DATABASE -c "DELETE FROM schema_migrations WHERE version = '$MIGRATION_VERSION'"

echo "回滾成功完成"
```

### 6. 性能優化

最小化遷移影響：

**批次處理**
```python
def migrate_large_table(batch_size=10000):
    """
    分批遷移大型表格
    """
    total_rows = execute("SELECT COUNT(*) FROM source_table")[0]['count']
    processed = 0
    
    while processed < total_rows:
        # 處理批次
        execute("""
            INSERT INTO target_table (columns...)
            SELECT columns...
            FROM source_table
            ORDER BY id
            OFFSET %s
            LIMIT %s
            ON CONFLICT DO NOTHING
        """, [processed, batch_size])
        
        processed += batch_size
        
        # 進度追蹤
        progress = (processed / total_rows) * 100
        logger.info(f"遷移進度: {progress:.1f}%")
        
        # 防止過載
        time.sleep(0.5)
```

**索引管理**
```sql
-- 批量插入前刪除索引
ALTER TABLE large_table DROP INDEX idx_column1;
ALTER TABLE large_table DROP INDEX idx_column2;

-- 批量插入
INSERT INTO large_table SELECT * FROM temp_data;

-- 並發重新建立索引
CREATE INDEX CONCURRENTLY idx_column1 ON large_table(column1);
CREATE INDEX CONCURRENTLY idx_column2 ON large_table(column2);
```

### 7. NoSQL 和跨平台遷移支援

處理 SQL、NoSQL 和混合環境中的現代資料庫遷移：

**進階多資料庫遷移框架**
```python
from abc import ABC, abstractmethod
from typing import Dict, List, Any, Optional
import asyncio
from dataclasses import dataclass

@dataclass
class MigrationOperation:
    operation_type: str
    collection_or_table: str
    data: Dict[str, Any]
    conditions: Optional[Dict[str, Any]] = None
    batch_size: int = 1000

class DatabaseAdapter(ABC):
    @abstractmethod
    async def connect(self, connection_string: str):
        pass
    
    @abstractmethod
    async def execute_migration(self, operation: MigrationOperation):
        pass
    
    @abstractmethod
    async def validate_migration(self, operation: MigrationOperation) -> bool:
        pass
    
    @abstractmethod
    async def rollback_migration(self, operation: MigrationOperation):
        pass

class MongoDBAdapter(DatabaseAdapter):
    def __init__(self):
        self.client = None
        self.db = None
    
    async def connect(self, connection_string: str):
        from motor.motor_asyncio import AsyncIOMotorClient
        self.client = AsyncIOMotorClient(connection_string)
        self.db = self.client.get_default_database()
    
    async def execute_migration(self, operation: MigrationOperation):
        collection = self.db[operation.collection_or_table]
        
        if operation.operation_type == 'add_field':
            await self._add_field(collection, operation)
        elif operation.operation_type == 'rename_field':
            await self._rename_field(collection, operation)
        elif operation.operation_type == 'migrate_data':
            await self._migrate_data(collection, operation)
        elif operation.operation_type == 'create_index':
            await self._create_index(collection, operation)
        elif operation.operation_type == 'schema_validation':
            await self._add_schema_validation(collection, operation)
    
    async def _add_field(self, collection, operation):
        """將新欄位添加到所有文件"""
        field_name = operation.data['field_name']
        default_value = operation.data.get('default_value')
        
        # 將欄位添加到沒有它的文件
        result = await collection.update_many(
            {field_name: {"$exists": False}},
            {"$set": {field_name: default_value}}
        )
        
        return {
            'matched_count': result.matched_count,
            'modified_count': result.modified_count
        }
    
    async def _rename_field(self, collection, operation):
        """在所有文件中重新命名欄位"""
        old_name = operation.data['old_name']
        new_name = operation.data['new_name']
        
        result = await collection.update_many(
            {old_name: {"$exists": True}},
            {"$rename": {old_name: new_name}}
        )
        
        return {
            'matched_count': result.matched_count,
            'modified_count': result.modified_count
        }
    
    async def _migrate_data(self, collection, operation):
        """遷移期間轉換資料"""
        pipeline = operation.data['pipeline']
        
        # 使用聚合管道進行複雜轉換
        cursor = collection.aggregate([
            {"$match": operation.conditions or {}},
            *pipeline,
            {"$merge": {
                "into": operation.collection_or_table,
                "on": "_id",
                "whenMatched": "replace"
            }}
        ])
        
        return [doc async for doc in cursor]
    
    async def _add_schema_validation(self, collection, operation):
        """將 JSON 模式驗證添加到集合"""
        schema = operation.data['schema']
        
        await self.db.command({
            "collMod": operation.collection_or_table,
            "validator": {"$jsonSchema": schema},
            "validationLevel": "strict",
            "validationAction": "error"
        })

class DynamoDBAdapter(DatabaseAdapter):
    def __init__(self):
        self.dynamodb = None
    
    async def connect(self, connection_string: str):
        import boto3
        self.dynamodb = boto3.resource('dynamodb')
    
    async def execute_migration(self, operation: MigrationOperation):
        table = self.dynamodb.Table(operation.collection_or_table)
        
        if operation.operation_type == 'add_gsi':
            await self._add_global_secondary_index(table, operation)
        elif operation.operation_type == 'migrate_data':
            await self._migrate_table_data(table, operation)
        elif operation.operation_type == 'update_capacity':
            await self._update_capacity(table, operation)
    
    async def _add_global_secondary_index(self, table, operation):
        """添加全域次要索引"""
        gsi_spec = operation.data['gsi_specification']
        
        table.update(
            GlobalSecondaryIndexUpdates=[
                {
                    'Create': gsi_spec
                }
            ]
        )
    
    async def _migrate_table_data(self, table, operation):
        """在 DynamoDB 表格之間遷移資料"""
        scan_kwargs = {
            'ProjectionExpression': operation.data.get('projection'),
            'FilterExpression': operation.conditions
        }
        
        target_table = self.dynamodb.Table(operation.data['target_table'])
        
        # 掃描源表格並寫入目標
        while True:
            response = table.scan(**scan_kwargs)
            
            # 轉換並寫入項目
            with target_table.batch_writer() as batch:
                for item in response['Items']:
                    transformed_item = self._transform_item(item, operation.data['transformation'])
                    batch.put_item(Item=transformed_item)
            
            if 'LastEvaluatedKey' not in response:
                break
            scan_kwargs['ExclusiveStartKey'] = response['LastEvaluatedKey']

class CassandraAdapter(DatabaseAdapter):
    def __init__(self):
        self.session = None
    
    async def connect(self, connection_string: str):
        from cassandra.cluster import Cluster
        from cassandra.auth import PlainTextAuthProvider
        
        # 解析連接字串以進行身份驗證
        cluster = Cluster(['127.0.0.1'])
        self.session = cluster.connect()
    
    async def execute_migration(self, operation: MigrationOperation):
        if operation.operation_type == 'add_column':
            await self._add_column(operation)
        elif operation.operation_type == 'create_materialized_view':
            await self._create_materialized_view(operation)
        elif operation.operation_type == 'migrate_data':
            await self._migrate_data(operation)
    
    async def _add_column(self, operation):
        """將欄位添加到 Cassandra 表格"""
        table = operation.collection_or_table
        column_name = operation.data['column_name']
        column_type = operation.data['column_type']
        
        cql = f"ALTER TABLE {table} ADD {column_name} {column_type}"
        self.session.execute(cql)
    
    async def _create_materialized_view(self, operation):
        """建立具體化視圖以進行反正規化"""
        view_spec = operation.data['view_specification']
        self.session.execute(view_spec)

class CrossPlatformMigrator:
    def __init__(self):
        self.adapters = {
            'postgresql': PostgreSQLAdapter(),
            'mysql': MySQLAdapter(),
            'mongodb': MongoDBAdapter(),
            'dynamodb': DynamoDBAdapter(),
            'cassandra': CassandraAdapter(),
            'redis': RedisAdapter(),
            'elasticsearch': ElasticsearchAdapter()
        }
    
    async def migrate_between_platforms(self, source_config, target_config, migration_spec):
        """在不同資料庫平台之間遷移資料"""
        source_adapter = self.adapters[source_config['type']]
        target_adapter = self.adapters[target_config['type']]
        
        await source_adapter.connect(source_config['connection_string'])
        await target_adapter.connect(target_config['connection_string'])
        
        # 執行遷移計畫
        for step in migration_spec['steps']:
            if step['type'] == 'extract':
                data = await self._extract_data(source_adapter, step)
            elif step['type'] == 'transform':
                data = await self._transform_data(data, step)
            elif step['type'] == 'load':
                await self._load_data(target_adapter, data, step)
    
    async def _extract_data(self, adapter, step):
        """從源資料庫提取資料"""
        extraction_op = MigrationOperation(
            operation_type='extract',
            collection_or_table=step['source_table'],
            data=step.get('extraction_params', {}),
            conditions=step.get('conditions'),
            batch_size=step.get('batch_size', 1000)
        )
        
        return await adapter.execute_migration(extraction_op)
    
    async def _transform_data(self, data, step):
        """在格式之間轉換資料"""
        transformation_rules = step['transformation_rules']
        
        transformed_data = []
        for record in data:
            transformed_record = {}
            
            for target_field, source_mapping in transformation_rules.items():
                if isinstance(source_mapping, str):
                    # 簡單的欄位映射
                    transformed_record[target_field] = record.get(source_mapping)
                elif isinstance(source_mapping, dict):
                    # 複雜轉換
                    if source_mapping['type'] == 'function':
                        func = source_mapping['function']
                        args = [record.get(arg) for arg in source_mapping['args']]
                        transformed_record[target_field] = func(*args)
                    elif source_mapping['type'] == 'concatenate':
                        fields = source_mapping['fields']
                        separator = source_mapping.get('separator', ' ')
                        values = [str(record.get(field, '')) for field in fields]
                        transformed_record[target_field] = separator.join(values)
            
            transformed_data.append(transformed_record)
        
        return transformed_data
    
    async def _load_data(self, adapter, data, step):
        """將資料載入到目標資料庫"""
        load_op = MigrationOperation(
            operation_type='load',
            collection_or_table=step['target_table'],
            data={'records': data},
            batch_size=step.get('batch_size', 1000)
        )
        
        return await adapter.execute_migration(load_op)

# 使用範例
async def migrate_sql_to_nosql():
    """範例：從 PostgreSQL 遷移到 MongoDB"""
    migrator = CrossPlatformMigrator()
    
    source_config = {
        'type': 'postgresql',
        'connection_string': 'postgresql://user:pass@localhost/db'
    }
    
    target_config = {
        'type': 'mongodb',
        'connection_string': 'mongodb://localhost:27017/db'
    }
    
    migration_spec = {
        'steps': [
            {
                'type': 'extract',
                'source_table': 'users',
                'conditions': {'active': True},
                'batch_size': 5000
            },
            {
                'type': 'transform',
                'transformation_rules': {
                    '_id': 'id',
                    'full_name': {
                        'type': 'concatenate',
                        'fields': ['first_name', 'last_name'],
                        'separator': ' '
                    },
                    'metadata': {
                        'type': 'function',
                        'function': lambda created, updated: {
                            'created_at': created,
                            'updated_at': updated
                        },
                        'args': ['created_at', 'updated_at']
                    }
                }
            },
            {
                'type': 'load',
                'target_table': 'users',
                'batch_size': 1000
            }
        ]
    }
    
    await migrator.migrate_between_platforms(source_config, target_config, migration_spec)
```

### 8. 現代遷移工具和變更資料擷取

與企業遷移工具和即時同步整合：

**Atlas 模式遷移 (MongoDB)**
```javascript
// atlas-migration.js
const { MongoClient } = require('mongodb');

class AtlasMigration {
    constructor(connectionString) {
        this.client = new MongoClient(connectionString);
        this.migrations = new Map();
    }
    
    register(version, migration) {
        this.migrations.set(version, migration);
    }
    
    async migrate() {
        await this.client.connect();
        const db = this.client.db();
        
        // 獲取當前版本
        const versionsCollection = db.collection('schema_versions');
        const currentVersion = await versionsCollection
            .findOne({}, { sort: { version: -1 } });
        
        const startVersion = currentVersion?.version || 0;
        
        // 運行待處理的遷移
        for (const [version, migration] of this.migrations) {
            if (version > startVersion) {
                console.log(`正在運行遷移 ${version}`);
                
                const session = this.client.startSession();
                
                try {
                    await session.withTransaction(async () => {
                        await migration.up(db, session);
                        await versionsCollection.insertOne({
                            version,
                            applied_at: new Date(),
                            checksum: migration.checksum
                        });
                    });
                } catch (error) {
                    console.error(`遷移 ${version} 失敗:`, error);
                    if (migration.down) {
                        await migration.down(db, session);
                    }
                    throw error;
                } finally {
                    await session.endSession();
                }
            }
        }
    }
}

// 範例 MongoDB 模式遷移
const migration_001 = {
    checksum: 'sha256:abc123...',
    
    async up(db, session) {
        // 將新欄位添加到現有文件
        await db.collection('users').updateMany(
            { email_verified: { $exists: false } },
            { 
                $set: { 
                    email_verified: false,
                    verification_token: null,
                    verification_expires: null
                }
            },
            { session }
        );
        
        // 建立新索引
        await db.collection('users').createIndex(
            { email_verified: 1, verification_expires: 1 },
            { session }
        );
        
        // 添加模式驗證
        await db.command({
            collMod: 'users',
            validator: {
                $jsonSchema: {
                    bsonType: 'object',
                    required: ['email', 'email_verified'],
                    properties: {
                        email: { bsonType: 'string' },
                        email_verified: { bsonType: 'bool' },
                        verification_token: { 
                            bsonType: ['string', 'null'] 
                        }
                    }
                }
            }
        }, { session });
    },
    
    async down(db, session) {
        // 移除模式驗證
        await db.command({
            collMod: 'users',
            validator: {}
        }, { session });
        
        // 刪除索引
        await db.collection('users').dropIndex(
            { email_verified: 1, verification_expires: 1 },
            { session }
        );
        
        // 移除欄位
        await db.collection('users').updateMany(
            {},
            { 
                $unset: {
                    email_verified: '',
                    verification_token: '',
                    verification_expires: ''
                }
            },
            { session }
        );
    }
};
```

**變更資料擷取 (CDC) 以實現即時同步**
```python
# cdc-migration.py
import asyncio
from kafka import KafkaConsumer, KafkaProducer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
import json

class CDCMigrationManager:
    def __init__(self, config):
        self.config = config
        self.consumer = None
        self.producer = None
        self.schema_registry = None
        self.active_migrations = {}
    
    async def setup_cdc_pipeline(self):
        """設定變更資料擷取管道"""
        # Kafka 消費者用於 CDC 事件
        self.consumer = KafkaConsumer(
            'database.changes',
            bootstrap_servers=self.config['kafka_brokers'],
            auto_offset_reset='earliest',
            enable_auto_commit=True,
            group_id='migration-consumer',
            value_deserializer=lambda m: json.loads(m.decode('utf-8'))
        )
        
        # Kafka 生產者用於處理後的事件
        self.producer = KafkaProducer(
            bootstrap_servers=self.config['kafka_brokers'],
            value_serializer=lambda v: json.dumps(v).encode('utf-8')
        )
        
        # 模式註冊表用於資料驗證
        self.schema_registry = SchemaRegistryClient({
            'url': self.config['schema_registry_url']
        })
    
    async def process_cdc_events(self):
        """處理 CDC 事件並應用到目標資料庫"""
        for message in self.consumer:
            event = message.value
            
            # 解析 CDC 事件
            operation = event['operation']  # INSERT, UPDATE, DELETE
            table = event['table']
            data = event['data']
            
            # 檢查此表格是否有活動遷移
            if table in self.active_migrations:
                migration_config = self.active_migrations[table]
                await self.apply_migration_transformation(event, migration_config)
            else:
                # 標準複製
                await self.replicate_change(event)
    
    async def apply_migration_transformation(self, event, migration_config):
        """遷移期間應用資料轉換"""
        transformation_rules = migration_config['transformation_rules']
        target_tables = migration_config['target_tables']
        
        # 根據遷移規則轉換資料
        transformed_data = {}
        for target_field, rule in transformation_rules.items():
            if isinstance(rule, str):
                # 簡單的欄位映射
                transformed_data[target_field] = event['data'].get(rule)
            elif isinstance(rule, dict):
                # 複雜轉換
                if rule['type'] == 'function':
                    func_name = rule['function']
                    func = getattr(self, f'transform_{func_name}')
                    args = [event['data'].get(arg) for arg in rule['args']]
                    transformed_data[target_field] = func(*args)
                elif rule['type'] == 'concatenate':
                    fields = rule['fields']
                    separator = rule.get('separator', ' ')
                    values = [str(event['data'].get(field, '')) for field in fields]
                    transformed_data[target_field] = separator.join(values)
        
        # 應用到目標表格
        for target_table in target_tables:
            await self.apply_to_target(target_table, event['operation'], transformed_data)
    
    async def setup_debezium_connector(self, source_db_config):
        """配置 Debezium 以進行 CDC"""
        connector_config = {
            "name": f"migration-connector-{source_db_config['name']}",
            "config": {
                "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
                "database.hostname": source_db_config['host'],
                "database.port": source_db_config['port'],
                "database.user": source_db_config['user'],
                "database.password": source_db_config['password'],
                "database.dbname": source_db_config['database'],
                "database.server.name": source_db_config['name'],
                "table.include.list": ",".join(source_db_config['tables']),
                "plugin.name": "pgoutput",
                "slot.name": f"migration_slot_{source_db_config['name']}",
                "publication.name": f"migration_pub_{source_db_config['name']}",
                "transforms": "route",
                "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
                "transforms.route.regex": "([^.]+)\.([^.]+)\.([^.]+)",
                "transforms.route.replacement": "database.changes"
            }
        }
        
        # 將連接器提交到 Kafka Connect
        import requests
        response = requests.post(
            f"{self.config['kafka_connect_url']}/connectors",
            json=connector_config,
            headers={'Content-Type': 'application/json'}
        )
        
        if response.status_code != 201:
            raise Exception(f"建立連接器失敗: {response.text}")
```

**進階監控和可觀察性**
```python
class EnterpriseeMigrationMonitor:
    def __init__(self, config):
        self.config = config
        self.metrics_client = self.setup_metrics_client()
        self.alerting_client = self.setup_alerting_client()
        self.migration_state = {
            'current_migrations': {},
            'completed_migrations': {},
            'failed_migrations': {}
        }
    
    def setup_metrics_client(self):
        """設定 Prometheus/Datadog 指標客戶端"""
        from prometheus_client import Counter, Gauge, Histogram, CollectorRegistry
        
        registry = CollectorRegistry()
        
        self.metrics = {
            'migration_duration': Histogram(
                'migration_duration_seconds',
                '遷移花費時間',
                ['migration_id', 'source_db', 'target_db'],
                registry=registry
            ),
            'rows_migrated': Counter(
                'migration_rows_total',
                '遷移的總行數',
                ['migration_id', 'table_name'],
                registry=registry
            ),
            'migration_errors': Counter(
                'migration_errors_total',
                '總遷移錯誤數',
                ['migration_id', 'error_type'],
                registry=registry
            ),
            'active_migrations': Gauge(
                'active_migrations_count',
                '活動遷移數',
                registry=registry
            ),
            'data_lag': Gauge(
                'migration_data_lag_seconds',
                '源和目標之間的資料延遲',
                ['migration_id'],
                registry=registry
            )
        }
        
        return registry
    
    async def track_migration_progress(self, migration_id):
        """即時遷移進度追蹤"""
        migration = self.migration_state['current_migrations'][migration_id]
        
        while migration['status'] == 'running':
            # 計算進度指標
            progress_stats = await self.calculate_progress_stats(migration)
            
            # 更新 Prometheus 指標
            self.metrics['rows_migrated'].labels(
                migration_id=migration_id,
                table_name=migration['table']
            ).inc(progress_stats['rows_processed_delta'])
            
            self.metrics['data_lag'].labels(
                migration_id=migration_id
            ).set(progress_stats['lag_seconds'])
            
            # 檢查異常
            await self.detect_migration_anomalies(migration_id, progress_stats)
            
            # 如果需要，生成警報
            await self.check_alert_conditions(migration_id, progress_stats)
            
            await asyncio.sleep(30)  # 每 30 秒檢查一次
    
    async def detect_migration_anomalies(self, migration_id, stats):
        """AI 驅動的遷移異常檢測"""
        # 簡單的統計異常檢測
        if stats['rows_per_second'] < stats['expected_rows_per_second'] * 0.5:
            await self.trigger_alert(
                'migration_slow',
                f"遷移 {migration_id} 運行速度慢於預期",
                {'stats': stats}
            )
        
        if stats['error_rate'] > 0.01:  # 1% 錯誤率閾值
            await self.trigger_alert(
                'migration_high_error_rate',
                f"遷移 {migration_id} 錯誤率高: {stats['error_rate']}",
                {'stats': stats}
            )
        
        if stats['memory_usage'] > 0.8:  # 80% 記憶體使用率
            await self.trigger_alert(
                'migration_high_memory',
                f"遷移 {migration_id} 記憶體使用率高: {stats['memory_usage']}",
                {'stats': stats}
            )
    
    async def setup_migration_dashboard(self):
        """設定 Grafana 儀表板以進行遷移監控"""
        dashboard_config = {
            "dashboard": {
                "title": "資料庫遷移監控",
                "panels": [
                    {
                        "title": "遷移進度",
                        "type": "graph",
                        "targets": [
                            {
                                "expr": "rate(migration_rows_total[5m])",
                                "legendFormat": "{{migration_id}} - {{table_name}}"
                            }
                        ]
                    },
                    {
                        "title": "資料延遲",
                        "type": "singlestat",
                        "targets": [
                            {
                                "expr": "migration_data_lag_seconds",
                                "legendFormat": "延遲 (秒)"
                            }
                        ]
                    },
                    {
                        "title": "錯誤率",
                        "type": "graph",
                        "targets": [
                            {
                                "expr": "rate(migration_errors_total[5m])",
                                "legendFormat": "{{error_type}}"
                            }
                        ]
                    },
                    {
                        "title": "遷移持續時間",
                        "type": "heatmap",
                        "targets": [
                            {
                                "expr": "migration_duration_seconds",
                                "legendFormat": "持續時間"
                            }
                        ]
                    }
                ]
            }
        }
        
        # 將儀表板提交到 Grafana API
        import requests
        response = requests.post(
            f"{self.config['grafana_url']}/api/dashboards/db",
            json=dashboard_config,
            headers={
                'Authorization': f"Bearer {self.config['grafana_token']}",
                'Content-Type': 'application/json'
            }
        )
        
        return response.json()
```

### 9. 事件溯源和 CQRS 遷移

處理事件驅動架構遷移：

**事件儲存遷移策略**
```python
class EventStoreMigrator:
    def __init__(self, event_store_config):
        self.event_store = EventStore(event_store_config)
        self.event_transformers = {}
        self.aggregate_rebuilders = {}
    
    def register_event_transformer(self, event_type, transformer):
        """為特定事件類型註冊轉換"""
        self.event_transformers[event_type] = transformer
    
    def register_aggregate_rebuilder(self, aggregate_type, rebuilder):
        """為聚合快照註冊重建器"""
        self.aggregate_rebuilders[aggregate_type] = rebuilder
    
    async def migrate_events(self, from_version, to_version):
        """將事件從一個模式版本遷移到另一個模式版本"""
        # 獲取所有需要遷移的事件
        events_cursor = self.event_store.get_events_by_version_range(
            from_version, to_version
        )
        
        migrated_events = []
        
        async for event in events_cursor:
            if event.event_type in self.event_transformers:
                transformer = self.event_transformers[event.event_type]
                migrated_event = await transformer.transform(event)
                migrated_events.append(migrated_event)
            else:
                # 不需要轉換
                migrated_events.append(event)
        
        # 將遷移的事件寫入新串流
        await self.event_store.append_events(
            f"migration-{to_version}",
            migrated_events
        )
        
        # 使用新事件重建聚合
        await self.rebuild_aggregates(migrated_events)
    
    async def rebuild_aggregates(self, events):
        """從遷移的事件重建聚合快照"""
        aggregates_to_rebuild = set()
        
        for event in events:
            aggregates_to_rebuild.add(event.aggregate_id)
        
        for aggregate_id in aggregates_to_rebuild:
            aggregate_type = self.get_aggregate_type(aggregate_id)
            
            if aggregate_type in self.aggregate_rebuilders:
                rebuilder = self.aggregate_rebuilders[aggregate_type]
                await rebuilder.rebuild(aggregate_id)

# 範例事件轉換
class UserEventTransformer:
    async def transform(self, event):
        """將 UserCreated 事件從 v1 轉換為 v2"""
        if event.event_type == 'UserCreated' and event.version == 1:
            # v1 有單獨的 first_name 和 last_name
            # v2 使用 full_name
            old_data = event.data
            new_data = {
                'user_id': old_data['user_id'],
                'full_name': f"{old_data['first_name']} {old_data['last_name']}",
                'email': old_data['email'],
                'created_at': old_data['created_at']
            }
            
            return Event(
                event_id=event.event_id,
                event_type='UserCreated',
                aggregate_id=event.aggregate_id,
                version=2,
                data=new_data,
                metadata=event.metadata
            )
        
        return event
```

### 10. 雲資料庫遷移自動化

使用基礎設施即程式碼自動化雲資料庫遷移：

**AWS 資料庫遷移與 CDK**
```typescript
// aws-db-migration.ts
import * as cdk from 'aws-cdk-lib';
import * as dms from 'aws-cdk-lib/aws-dms';
import * as rds from 'aws-cdk-lib/aws-rds';
import * as ec2 from 'aws-cdk-lib/aws-ec2';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as stepfunctions from 'aws-cdk-lib/aws-stepfunctions';
import * as sfnTasks from 'aws-cdk-lib/aws-stepfunctions-tasks';

export class DatabaseMigrationStack extends cdk.Stack {
    constructor(scope: cdk.App, id: string, props?: cdk.StackProps) {
        super(scope, id, props);
        
        // 建立用於遷移的 VPC
        const vpc = new ec2.Vpc(this, 'MigrationVPC', {
            maxAzs: 2,
            subnetConfiguration: [
                {
                    cidrMask: 24,
                    name: 'private',
                    subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS
                },
                {
                    cidrMask: 24,
                    name: 'public',
                    subnetType: ec2.SubnetType.PUBLIC
                }
            ]
        });
        
        // DMS 複製實例
        const replicationInstance = new dms.CfnReplicationInstance(this, 'ReplicationInstance', {
            replicationInstanceClass: 'dms.t3.medium',
            replicationInstanceIdentifier: 'migration-instance',
            allocatedStorage: 100,
            autoMinorVersionUpgrade: true,
            multiAz: false,
            publiclyAccessible: false,
            replicationSubnetGroupIdentifier: this.createSubnetGroup(vpc).ref
        });
        
        // 源和目標端點
        const sourceEndpoint = new dms.CfnEndpoint(this, 'SourceEndpoint', {
            endpointType: 'source',
            engineName: 'postgres',
            serverName: 'source-db.example.com',
            port: 5432,
            databaseName: 'source_db',
            username: 'migration_user',
            password: 'migration_password'
        });
        
        const targetEndpoint = new dms.CfnEndpoint(this, 'TargetEndpoint', {
            endpointType: 'target',
            engineName: 'postgres',
            serverName: 'target-db.example.com',
            port: 5432,
            databaseName: 'target_db',
            username: 'migration_user',
            password: 'migration_password'
        });
        
        // 遷移任務
        const migrationTask = new dms.CfnReplicationTask(this, 'MigrationTask', {
            replicationTaskIdentifier: 'full-load-and-cdc',
            sourceEndpointArn: sourceEndpoint.ref,
            targetEndpointArn: targetEndpoint.ref,
            replicationInstanceArn: replicationInstance.ref,
            migrationType: 'full-load-and-cdc',
            tableMappings: JSON.stringify({
                "rules": [
                    {
                        "rule-type": "selection",
                        "rule-id": "1",
                        "rule-name": "1",
                        "object-locator": {
                            "schema-name": "public",
                            "table-name": "%"
                        },
                        "rule-action": "include"
                    }
                ]
            }),
            replicationTaskSettings: JSON.stringify({
                "TargetMetadata": {
                    "TargetSchema": "",
                    "SupportLobs": true,
                    "FullLobMode": false,
                    "LobChunkSize": 0,
                    "LimitedSizeLobMode": true,
                    "LobMaxSize": 32,
                    "LoadMaxFileSize": 0,
                    "ParallelLoadThreads": 0,
                    "ParallelLoadBufferSize": 0,
                    "BatchApplyEnabled": false,
                    "TaskRecoveryTableEnabled": false
                },
                "FullLoadSettings": {
                    "TargetTablePrepMode": "DROP_AND_CREATE",
                    "CreatePkAfterFullLoad": false,
                    "StopTaskCachedChangesApplied": false,
                    "StopTaskCachedChangesNotApplied": false,
                    "MaxFullLoadSubTasks": 8,
                    "TransactionConsistencyTimeout": 600,
                    "CommitRate": 10000
                },
                "Logging": {
                    "EnableLogging": true,
                    "LogComponents": [
                        {
                            "Id": "SOURCE_UNLOAD",
                            "Severity": "LOGGER_SEVERITY_DEFAULT"
                        },
                        {
                            "Id": "TARGET_LOAD",
                            "Severity": "LOGGER_SEVERITY_DEFAULT"
                        }
                    ]
                }
            })
        });
        
        // 使用 Step Functions 進行遷移編排
        this.createMigrationOrchestration(migrationTask);
    }
    
    private createSubnetGroup(vpc: ec2.Vpc): dms.CfnReplicationSubnetGroup {
        return new dms.CfnReplicationSubnetGroup(this, 'ReplicationSubnetGroup', {
            replicationSubnetGroupDescription: '用於 DMS 的子網組',
            replicationSubnetGroupIdentifier: 'migration-subnet-group',
            subnetIds: vpc.privateSubnets.map(subnet => subnet.subnetId)
        });
    }
    
    private createMigrationOrchestration(migrationTask: dms.CfnReplicationTask): void {
        // 遷移步驟的 Lambda 函數
        const startMigrationFunction = new lambda.Function(this, 'StartMigration', {
            runtime: lambda.Runtime.PYTHON_3_9,
            handler: 'index.handler',
            code: lambda.Code.fromInline(`
import boto3
import json

def handler(event, context):
    dms = boto3.client('dms')
    task_arn = event['task_arn']
    
    response = dms.start_replication_task(
        ReplicationTaskArn=task_arn,
        StartReplicationTaskType='start-replication'
    )
    
    return {
        'statusCode': 200,
        'task_arn': task_arn,
        'task_status': response['ReplicationTask']['Status']
    }
            `)
        });
        
        const checkMigrationStatusFunction = new lambda.Function(this, 'CheckMigrationStatus', {
            runtime: lambda.Runtime.PYTHON_3_9,
            handler: 'index.handler',
            code: lambda.Code.fromInline(`
import boto3
import json

def handler(event, context):
    dms = boto3.client('dms')
    task_arn = event['task_arn']
    
    response = dms.describe_replication_tasks(
        Filters=[
            {
                'Name': 'replication-task-arn',
                'Values': [task_arn]
            }
        ]
    )
    
    task = response['ReplicationTasks'][0]
    status = task['Status']
    
    return {
        'task_arn': task_arn,
        'task_status': status,
        'is_complete': status in ['stopped', 'failed', 'ready']
    }
            `)
        });
        
        // Step Function 定義
        const startMigrationTask = new sfnTasks.LambdaInvoke(this, 'StartMigrationTask', {
            lambdaFunction: startMigrationFunction,
            inputPath: '$',
            outputPath: '$'
        });
        
        const checkStatusTask = new sfnTasks.LambdaInvoke(this, 'CheckMigrationStatusTask', {
            lambdaFunction: checkMigrationStatusFunction,
            inputPath: '$',
            outputPath: '$'
        });
        
        const waitTask = new stepfunctions.Wait(this, 'WaitForMigration', {
            time: stepfunctions.WaitTime.duration(cdk.Duration.minutes(5))
        });
        
        const migrationComplete = new stepfunctions.Succeed(this, 'MigrationComplete');
        const migrationFailed = new stepfunctions.Fail(this, 'MigrationFailed');
        
        // 定義狀態機
        const definition = startMigrationTask
            .next(waitTask)
            .next(checkStatusTask)
            .next(new stepfunctions.Choice(this, 'IsMigrationComplete?')
                .when(stepfunctions.Condition.booleanEquals('$.is_complete', true),
                      new stepfunctions.Choice(this, 'MigrationSuccessful?')
                          .when(stepfunctions.Condition.stringEquals('$.task_status', 'stopped'), migrationComplete)
                          .otherwise(migrationFailed))
                .otherwise(waitTask));
        
        new stepfunctions.StateMachine(this, 'MigrationStateMachine', {
            definition: definition,
            timeout: cdk.Duration.hours(24)
        });
    }
}
```

## 輸出格式

1. **全面遷移策略**：多資料庫平台支援與 NoSQL 整合
2. **跨平台遷移工具**：SQL 到 NoSQL、NoSQL 到 SQL 和混合遷移
3. **現代工具整合**：Atlas、Debezium、Flyway、Prisma 和雲原生解決方案
4. **變更資料擷取管道**：與 Kafka 和模式註冊表的即時同步
5. **事件溯源遷移**：事件儲存轉換和聚合重建
6. **雲基礎設施自動化**：AWS DMS、GCP 資料庫遷移服務、Azure DMS
7. **企業監控套件**：Prometheus 指標、Grafana 儀表板和異常檢測
8. **進階驗證框架**：多資料庫完整性檢查和性能基準
9. **自動化回滾程序**：平台特定恢復策略
10. **性能優化**：批次處理、平行執行和資源管理

專注於零停機遷移，並在所有支援的資料庫平台中提供全面的驗證、自動化回滾和企業級監控。

## 跨指令整合

此命令與其他開發工作流程命令無縫整合，以建立全面的資料庫優先開發管道：

### 與 API 開發整合 (`/api-scaffold`)
```python
# integrated-db-api-config.py
class IntegratedDatabaseApiConfig:
    def __init__(self):
        self.api_config = self.load_api_config()        # 來自 /api-scaffold
        self.db_config = self.load_db_config()          # 來自 /db-migrate
        self.migration_config = self.load_migration_config()
    
    def generate_api_aware_migrations(self):
        """生成考慮 API 端點和模式的遷移"""
        return {
            # API 感知遷移策略
            'api_migration_strategy': f"""
-- 考慮 API 端點的遷移
-- 遷移: {datetime.now().strftime('%Y%m%d_%H%M%S')}_api_aware_schema_update.sql

-- 遷移前檢查 API 依賴項
DO $$
BEGIN
    -- 驗證依賴此模式的 API 端點
    IF EXISTS (
        SELECT 1 FROM api_endpoints 
        WHERE schema_dependencies @> '["users", "profiles"]'
        AND is_active = true
    ) THEN
        RAISE NOTICE '發現依賴此模式的活動 API 端點';
        
        -- 建立帶有 API 版本控制的遷移策略
        CREATE TABLE IF NOT EXISTS api_migration_log (
            id SERIAL PRIMARY KEY,
            migration_name VARCHAR(255) NOT NULL,
            api_version VARCHAR(50) NOT NULL,
            schema_changes JSONB,
            rollback_script TEXT,
            created_at TIMESTAMP DEFAULT NOW()
        );
        
        -- 記錄此遷移以進行 API 追蹤
        INSERT INTO api_migration_log (
            migration_name, 
            api_version, 
            schema_changes
        ) VALUES (
            'api_aware_schema_update',
            '{self.api_config.get("version", "v1")}',
            '{{"tables": ["users", "profiles"], "type": "schema_update"}}'::jsonb
        );
    END IF;
END $$;

-- 向後相容的模式變更
ALTER TABLE users ADD COLUMN IF NOT EXISTS new_field VARCHAR(255);

-- 為 API 向後相容性建立視圖
CREATE OR REPLACE VIEW users_api_v1 AS 
SELECT 
    id,
    username,
    email,
    -- 維護 API 相容性
    COALESCE(new_field, 'default_value') as new_field,
    created_at,
    updated_at
FROM users;

-- 授予 API 服務存取權限
GRANT SELECT ON users_api_v1 TO {self.api_config.get("db_user", "api_service")};

COMMIT;
            """,
            
            # 資料庫連接池優化用於 API
            'connection_pool_config': {
                'fastapi': f"""
# 帶有優化資料庫連接的 FastAPI
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
from sqlalchemy.pool import QueuePool

class DatabaseConfig:
    def __init__(self):
        self.database_url = "{self.db_config.get('url', 'postgresql://localhost/app')}"
        self.api_config = {self.api_config}
        
    def create_engine(self):
        return create_engine(
            self.database_url,
            poolclass=QueuePool,
            pool_size={self.api_config.get('db_pool_size', 20)},
            max_overflow={self.api_config.get('db_max_overflow', 0)},
            pool_pre_ping=True,
            pool_recycle=3600,
            echo={str(self.api_config.get('debug', False)).lower()}
        )
    
    def get_session_maker(self):
        engine = self.create_engine()
        return sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 遷移感知 API 依賴項
async def get_db_with_migration_check():
    # 檢查遷移是否正在運行
    async with get_db() as session:
        result = await session.execute(
            text("SELECT COUNT(*) FROM schema_migrations WHERE is_running = true")
        )
        running_migrations = result.scalar()
        
        if running_migrations > 0:
            raise HTTPException(
                status_code=503,
                detail="資料庫遷移正在進行中。API 暫時不可用。"
            )
        
        yield session
                """,
                
                'express': f"""
// 帶有資料庫遷移感知的 Express.js
const {{ Pool }} = require('pg');
const express = require('express');
const app = express();

class DatabaseManager {{
    constructor() {{
        this.pool = new Pool({{
            connectionString: '{self.db_config.get('url', 'postgresql://localhost/app')}',
            max: {self.api_config.get('db_pool_size', 20)},
            idleTimeoutMillis: 30000,
            connectionTimeoutMillis: 2000,
        }});
        
        this.migrationStatus = new Map();
    }}
    
    async checkMigrationStatus() {{
        try {{
            const client = await this.pool.connect();
            const result = await client.query(
                'SELECT COUNT(*) as count FROM schema_migrations WHERE is_running = true'
            );
            client.release();
            
            return result.rows[0].count === '0';
        }} catch (error) {{
            console.error('檢查遷移狀態失敗:', error);
            return false;
        }}
    }}
    
    // 檢查遷移狀態的中介軟體
    migrationStatusMiddleware() {{
        return async (req, res, next) => {{
            const isSafe = await this.checkMigrationStatus();
            
            if (!isSafe) {{
                return res.status(503).json({{
                    error: '資料庫遷移正在進行中',
                    message: '資料庫更新期間 API 暫時不可用'
                }});
            }}
            
            next();
        }};
    }}
}}

const dbManager = new DatabaseManager();
app.use('/api', dbManager.migrationStatusMiddleware());
                """
            }
        }
    
    def generate_api_schema_sync(self):
        """生成 API 模式與資料庫同步"""
        return f"""
# API 模式同步
import asyncio
import aiohttp
from sqlalchemy import text

class ApiSchemaSync:
    def __init__(self, api_base_url="{self.api_config.get('base_url', 'http://localhost:8000')}"):
        self.api_base_url = api_base_url
        self.db_config = {self.db_config}
    
    async def notify_api_of_schema_change(self, migration_name, schema_changes):
        '''通知 API 服務資料庫模式變更'''
        async with aiohttp.ClientSession() as session:
            payload = {{
                'migration_name': migration_name,
                'schema_changes': schema_changes,
                'timestamp': datetime.now().isoformat()
            }}
            
            try:
                async with session.post(
                    f"{{self.api_base_url}}/internal/schema-update",
                    json=payload,
                    timeout=30
                ) as response:
                    if response.status == 200:
                        print(f"API 已通知模式變更: {{migration_name}}")
                    else:
                        print(f"通知 API 失敗: {{response.status}}")
            except Exception as e:
                print(f"通知 API 時出錯: {{e}}")
    
    async def validate_api_compatibility(self, proposed_changes):
        '''驗證提議的模式變更不會破壞 API'''
        async with aiohttp.ClientSession() as session:
            try:
                async with session.post(
                    f"{{self.api_base_url}}/internal/validate-schema",
                    json={{'proposed_changes': proposed_changes}},
                    timeout=30
                ) as response:
                    result = await response.json()
                    return result.get('compatible', False), result.get('issues', [])
            except Exception as e:
                print(f"驗證 API 相容性時出錯: {{e}}")
                return False, [f"驗證服務不可用: {{e}}"]
        """
```

### 完整工作流程整合
```python
# complete-database-workflow.py
class CompleteDatabaseWorkflow:
    def __init__(self):
        self.configs = {
            'api': self.load_api_config(),           # 來自 /api-scaffold
            'testing': self.load_test_config(),      # 來自 /test-harness
            'security': self.load_security_config(), # 來自 /security-scan
            'docker': self.load_docker_config(),     # 來自 /docker-optimize
            'k8s': self.load_k8s_config(),          # 來自 /k8s-manifest
            'frontend': self.load_frontend_config(), # 來自 /frontend-optimize
            'database': self.load_db_config()        # 來自 /db-migrate
        }
    
    async def execute_complete_workflow(self):
        console.log("🚀 正在啟動完整的資料庫遷移工作流程...")
        
        # 1. 遷移前安全掃描
        security_scan = await self.run_security_scan()
        console.log("✅ 資料庫安全掃描完成")
        
        # 2. API 相容性檢查
        api_compatibility = await self.check_api_compatibility()
        console.log("✅ API 相容性已驗證")
        
        # 3. 基於容器的遷移測試
        container_tests = await self.run_container_tests()
        console.log("✅ 基於容器的遷移測試通過")
        
        # 4. 生產遷移與監控
        migration_result = await self.run_production_migration()
        console.log("✅ 生產遷移完成")
        
        # 5. 前端快取失效
        cache_invalidation = await self.invalidate_frontend_caches()
        console.log("✅ 前端快取已失效")
        
        # 6. Kubernetes 部署更新
        k8s_deployment = await self.update_k8s_deployment()
        console.log("✅ Kubernetes 部署已更新")
        
        # 7. 遷移後測試管道
        post_migration_tests = await self.run_post_migration_tests()
        console.log("✅ 遷移後測試完成")
        
        return {
            'status': 'success',
            'workflow_id': self.generate_workflow_id(),
            'components': {
                security_scan,
                api_compatibility,
                container_tests,
                migration_result,
                cache_invalidation,
                k8s_deployment,
                post_migration_tests
            },
            'migration_summary': {
                'zero_downtime': True,
                'rollback_plan': '可用',
                'performance_impact': '最小',
                'security_validated': True
            }
        }
```

此整合資料庫遷移工作流程確保資料庫變更在應用程式堆疊的所有層次（從 API 相容性到前端快取失效）進行協調，建立一個全面的資料庫優先開發管道，維護資料完整性和系統可靠性。

專注於零停機部署的企業級遷移，並在所有支援的資料庫平台中提供全面的驗證、自動化回滾和企業級監控。
