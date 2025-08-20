# 雲端成本優化

您是雲端成本優化專家，專精於在維持性能和可靠性的同時降低基礎設施費用。分析雲端支出、識別節省機會，並在 AWS、Azure 和 GCP 上實施具成本效益的架構。

## 背景
使用者需要在不影響性能或可靠性的情況下優化雲端基礎設施成本。專注於可行的建議、自動化成本控制和可持續的成本管理實踐。

## 要求
$ARGUMENTS

## 指示

### 1. 成本分析與可見性

實施全面的成本分析：

**成本分析框架**
```python
import boto3
import pandas as pd
from datetime import datetime, timedelta
from typing import Dict, List, Any
import json

class CloudCostAnalyzer:
    def __init__(self, cloud_provider: str):
        self.provider = cloud_provider
        self.client = self._initialize_client()
        self.cost_data = None
        
    def analyze_costs(self, time_period: int = 30):
        """全面成本分析"""
        analysis = {
            'total_cost': self._get_total_cost(time_period),
            'cost_by_service': self._analyze_by_service(time_period),
            'cost_by_resource': self._analyze_by_resource(time_period),
            'cost_trends': self._analyze_trends(time_period),
            'anomalies': self._detect_anomalies(time_period),
            'waste_analysis': self._identify_waste(),
            'optimization_opportunities': self._find_opportunities()
        }
        
        return self._generate_report(analysis)
    
    def _analyze_by_service(self, days: int):
        """按服務分析成本"""
        if self.provider == 'aws':
            ce = boto3.client('ce')
            
            response = ce.get_cost_and_usage(
                TimePeriod={
                    'Start': (datetime.now() - timedelta(days=days)).strftime('%Y-%m-%d'),
                    'End': datetime.now().strftime('%Y-%m-%d')
                },
                Granularity='DAILY',
                Metrics=['UnblendedCost'],
                GroupBy=[
                    {'Type': 'DIMENSION', 'Key': 'SERVICE'}
                ]
            )
            
            # 處理響應
            service_costs = {}
            for result in response['ResultsByTime']:
                for group in result['Groups']:
                    service = group['Keys'][0]
                    cost = float(group['Metrics']['UnblendedCost']['Amount'])
                    
                    if service not in service_costs:
                        service_costs[service] = []
                    service_costs[service].append(cost)
            
            # 計算總計和趨勢
            analysis = {}
            for service, costs in service_costs.items():
                analysis[service] = {
                    'total': sum(costs),
                    'average_daily': sum(costs) / len(costs),
                    'trend': self._calculate_trend(costs),
                    'percentage': (sum(costs) / self._get_total_cost(days)) * 100
                }
            
            return analysis
    
    def _identify_waste(self):
        """識別浪費的資源"""
        waste_analysis = {
            'unused_resources': self._find_unused_resources(),
            'oversized_resources': self._find_oversized_resources(),
            'unattached_storage': self._find_unattached_storage(),
            'idle_load_balancers': self._find_idle_load_balancers(),
            'old_snapshots': self._find_old_snapshots(),
            'untagged_resources': self._find_untagged_resources()
        }
        
        total_waste = sum(item['estimated_savings'] 
                         for category in waste_analysis.values() 
                         for item in category)
        
        waste_analysis['total_potential_savings'] = total_waste
        
        return waste_analysis
    
    def _find_unused_resources(self):
        """尋找未使用的資源"""
        unused = []
        
        if self.provider == 'aws':
            # 檢查 EC2 實例
            ec2 = boto3.client('ec2')
            cloudwatch = boto3.client('cloudwatch')
            
            instances = ec2.describe_instances(
                Filters=[{'Name': 'instance-state-name', 'Values': ['running']}]
            )
            
            for reservation in instances['Reservations']:
                for instance in reservation['Instances']:
                    # 檢查 CPU 利用率
                    metrics = cloudwatch.get_metric_statistics(
                        Namespace='AWS/EC2',
                        MetricName='CPUUtilization',
                        Dimensions=[
                            {'Name': 'InstanceId', 'Value': instance['InstanceId']}
                        ],
                        StartTime=datetime.now() - timedelta(days=7),
                        EndTime=datetime.now(),
                        Period=3600,
                        Statistics=['Average']
                    )
                    
                    if metrics['Datapoints']:
                        avg_cpu = sum(d['Average'] for d in metrics['Datapoints']) / len(metrics['Datapoints'])
                        
                        if avg_cpu < 5:  # CPU 使用率低於 5%
                            unused.append({
                                'resource_type': 'EC2 實例',
                                'resource_id': instance['InstanceId'],
                                'reason': f'平均 CPU: {avg_cpu:.2f}%',
                                'estimated_savings': self._calculate_instance_cost(instance)
                            })
        
        return unused
```

### 2. 資源調整大小

實施智慧調整大小：

**調整大小引擎**
```python
class ResourceRightsizer:
    def __init__(self):
        self.utilization_thresholds = {
            'cpu_low': 20,
            'cpu_high': 80,
            'memory_low': 30,
            'memory_high': 85,
            'network_low': 10,
            'network_high': 70
        }
    
    def analyze_rightsizing_opportunities(self):
        """尋找調整大小的機會"""
        opportunities = {
            'ec2_instances': self._rightsize_ec2(),
            'rds_instances': self._rightsize_rds(),
            'containers': self._rightsize_containers(),
            'lambda_functions': self._rightsize_lambda(),
            'storage_volumes': self._rightsize_storage()
        }
        
        return self._prioritize_opportunities(opportunities)
    
    def _rightsize_ec2(self):
        """調整 EC2 實例大小"""
        recommendations = []
        
        instances = self._get_running_instances()
        
        for instance in instances:
            # 獲取利用率指標
            utilization = self._get_instance_utilization(instance['InstanceId'])
            
            # 確定是否過大或過小
            current_type = instance['InstanceType']
            recommended_type = self._recommend_instance_type(
                current_type, 
                utilization
            )
            
            if recommended_type != current_type:
                current_cost = self._get_instance_cost(current_type)
                new_cost = self._get_instance_cost(recommended_type)
                
                recommendations.append({
                    'resource_id': instance['InstanceId'],
                    'current_type': current_type,
                    'recommended_type': recommended_type,
                    'reason': self._generate_reason(utilization),
                    'current_cost': current_cost,
                    'new_cost': new_cost,
                    'monthly_savings': (current_cost - new_cost) * 730,
                    'effort': 'medium',
                    'risk': 'low' if 'downsize' in self._generate_reason(utilization) else 'medium'
                })
        
        return recommendations
    
    def _recommend_instance_type(self, current_type: str, utilization: Dict):
        """推薦最佳實例類型"""
        # 解析當前實例系列和大小
        family, size = self._parse_instance_type(current_type)
        
        # 計算所需資源
        required_cpu = self._calculate_required_cpu(utilization['cpu'])
        required_memory = self._calculate_required_memory(utilization['memory'])
        
        # 尋找最佳匹配實例
        instance_catalog = self._get_instance_catalog()
        
        candidates = []
        for instance_type, specs in instance_catalog.items():
            if (specs['vcpu'] >= required_cpu and 
                specs['memory'] >= required_memory):
                candidates.append({
                    'type': instance_type,
                    'cost': specs['cost'],
                    'vcpu': specs['vcpu'],
                    'memory': specs['memory'],
                    'efficiency_score': self._calculate_efficiency_score(
                        specs, required_cpu, required_memory
                    )
                })
        
        # 選擇最佳候選者
        if candidates:
            best = sorted(candidates, 
                         key=lambda x: (x['efficiency_score'], x['cost']))[0]
            return best['type']
        
        return current_type
    
    def create_rightsizing_automation(self):
        """自動調整大小實施"""
        return '''
import boto3
from datetime import datetime
import logging

class AutomatedRightsizer:
    def __init__(self):
        self.ec2 = boto3.client('ec2')
        self.cloudwatch = boto3.client('cloudwatch')
        self.logger = logging.getLogger(__name__)
        
    def execute_rightsizing(self, recommendations: List[Dict], dry_run: bool = True):
        """執行調整大小建議"""
        results = []
        
        for recommendation in recommendations:
            try:
                if recommendation['risk'] == 'low' or self._get_approval(recommendation):
                    result = self._resize_instance(
                        recommendation['resource_id'],
                        recommendation['recommended_type'],
                        dry_run=dry_run
                    )
                    results.append(result)
            except Exception as e:
                self.logger.error(f"調整 {recommendation['resource_id']} 大小失敗: {e}")
                
        return results
    
    def _resize_instance(self, instance_id: str, new_type: str, dry_run: bool):
        """調整 EC2 實例大小"""
        # 建立快照以進行回滾
        snapshot_id = self._create_snapshot(instance_id)
        
        try:
            # 停止實例
            if not dry_run:
                self.ec2.stop_instances(InstanceIds=[instance_id])
                self._wait_for_state(instance_id, 'stopped')
            
            # 更改實例類型
            self.ec2.modify_instance_attribute(
                InstanceId=instance_id,
                InstanceType={'Value': new_type},
                DryRun=dry_run
            )
            
            # 啟動實例
            if not dry_run:
                self.ec2.start_instances(InstanceIds=[instance_id])
                self._wait_for_state(instance_id, 'running')
            
            return {
                'instance_id': instance_id,
                'status': 'success',
                'new_type': new_type,
                'snapshot_id': snapshot_id
            }
            
        except Exception as e:
            # 失敗時回滾
            if not dry_run:
                self._rollback_instance(instance_id, snapshot_id)
            raise
'''
```

### 3. 預留實例和儲蓄計畫

優化基於承諾的折扣：

**預留優化器**
```python
class ReservationOptimizer:
    def __init__(self):
        self.usage_history = None
        self.existing_reservations = None
        
    def analyze_reservation_opportunities(self):
        """分析預留機會"""
        analysis = {
            'current_coverage': self._analyze_current_coverage(),
            'usage_patterns': self._analyze_usage_patterns(),
            'recommendations': self._generate_recommendations(),
            'roi_analysis': self._calculate_roi(),
            'risk_assessment': self._assess_commitment_risk()
        }
        
        return analysis
    
    def _analyze_usage_patterns(self):
        """分析歷史使用模式"""
        # 獲取 12 個月的使用資料
        usage_data = self._get_historical_usage(months=12)
        
        patterns = {
            'stable_workloads': [],
            'variable_workloads': [],
            'seasonal_patterns': [],
            'growth_trends': []
        }
        
        # 分析每個實例系列
        for family in self._get_instance_families(usage_data):
            family_usage = self._filter_by_family(usage_data, family)
            
            # 計算穩定性指標
            stability = self._calculate_stability(family_usage)
            
            if stability['coefficient_of_variation'] < 0.1:
                patterns['stable_workloads'].append({
                    'family': family,
                    'average_usage': stability['mean'],
                    'min_usage': stability['min'],
                    'recommendation': '預留實例',
                    'term': '3_年',
                    'payment': '全額預付'
                })
            elif stability['coefficient_of_variation'] < 0.3:
                patterns['variable_workloads'].append({
                    'family': family,
                    'average_usage': stability['mean'],
                    'baseline': stability['percentile_25'],
                    'recommendation': '儲蓄計畫',
                    'commitment': stability['percentile_25']
                })
            
            # 檢查季節性模式
            if self._has_seasonal_pattern(family_usage):
                patterns['seasonal_patterns'].append({
                    'family': family,
                    'pattern': self._identify_seasonal_pattern(family_usage),
                    'recommendation': '帶有儲蓄計畫基準的 Spot 實例'
                })
        
        return patterns
    
    def _generate_recommendations(self):
        """生成預留建議"""
        recommendations = []
        
        patterns = self._analyze_usage_patterns()
        current_costs = self._calculate_current_costs()
        
        # 預留實例建議
        for workload in patterns['stable_workloads']:
            ri_options = self._calculate_ri_options(workload)
            
            for option in ri_options:
                savings = current_costs[workload['family']] - option['total_cost']
                
                if savings > 0:
                    recommendations.append({
                        'type': '預留實例',
                        'family': workload['family'],
                        'quantity': option['quantity'],
                        'term': option['term'],
                        'payment': option['payment_option'],
                        'upfront_cost': option['upfront_cost'],
                        'monthly_cost': option['monthly_cost'],
                        'total_savings': savings,
                        'break_even_months': option['upfront_cost'] / (savings / 36),
                        'confidence': '高'
                    })
        
        # 儲蓄計畫建議
        for workload in patterns['variable_workloads']:
            sp_options = self._calculate_savings_plan_options(workload)
            
            for option in sp_options:
                recommendations.append({
                    'type': '儲蓄計畫',
                    'commitment_type': option['type'],
                    'hourly_commitment': option['commitment'],
                    'term': option['term'],
                    'estimated_savings': option['savings'],
                    'flexibility': option['flexibility_score'],
                    'confidence': '中'
                })
        
        return sorted(recommendations, key=lambda x: x.get('total_savings', 0), reverse=True)
    
    def create_reservation_dashboard(self):
        """建立預留追蹤儀表板"""
        return '''
<!DOCTYPE html>
<html>
<head>
    <title>預留與儲蓄儀表板</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <div class="dashboard">
        <div class="summary-cards">
            <div class="card">
                <h3>當前覆蓋率</h3>
                <div class="metric">{coverage_percentage}%</div>
                <div class="sub-metric">按需: ${on_demand_cost}</div>
                <div class="sub-metric">預留: ${reserved_cost}</div>
            </div>
            
            <div class="card">
                <h3>潛在節省</h3>
                <div class="metric">${potential_savings}/月</div>
                <div class="sub-metric">{recommendations_count} 個機會</div>
            </div>
            
            <div class="card">
                <h3>即將到期</h3>
                <div class="metric">{expiring_count} 個 RI</div>
                <div class="sub-metric">未來 30 天</div>
            </div>
        </div>
        
        <div class="charts">
            <canvas id="coverageChart"></canvas>
            <canvas id="savingsChart"></canvas>
        </div>
        
        <div class="recommendations-table">
            <h3>熱門建議</h3>
            <table>
                <tr>
                    <th>類型</th>
                    <th>資源</th>
                    <th>期限</th>
                    <th>預付</th>
                    <th>每月節省</th>
                    <th>投資回報率</th>
                    <th>行動</th>
                </tr>
                {recommendation_rows}
            </table>
        </div>
    </div>
</body>
</html>
'''
```

### 4. Spot 實例優化

有效利用 Spot 實例：

**Spot 實例管理器**
```python
class SpotInstanceOptimizer:
    def __init__(self):
        self.spot_advisor = self._init_spot_advisor()
        self.interruption_handler = None
        
    def identify_spot_opportunities(self):
        """識別適合 Spot 的工作負載"""
        workloads = self._analyze_workloads()
        
        spot_candidates = {
            'batch_processing': [],
            'dev_test': [],
            'stateless_apps': [],
            'ci_cd': [],
            'data_processing': []
        }
        
        for workload in workloads:
            suitability = self._assess_spot_suitability(workload)
            
            if suitability['score'] > 0.7:
                spot_candidates[workload['type']].append({
                    'workload': workload['name'],
                    'current_cost': workload['cost'],
                    'spot_savings': workload['cost'] * 0.7,  # 約 70% 節省
                    'interruption_tolerance': suitability['interruption_tolerance'],
                    'recommended_strategy': self._recommend_spot_strategy(workload)
                })
        
        return spot_candidates
    
    def _recommend_spot_strategy(self, workload):
        """推薦 Spot 實例策略"""
        if workload['interruption_tolerance'] == '高':
            return {
                'strategy': 'Spot 隊列多樣化',
                'instance_pools': 10,
                'allocation_strategy': '容量優化',
                'on_demand_base': 0,
                'spot_percentage': 100
            }
        elif workload['interruption_tolerance'] == '中':
            return {
                'strategy': '混合實例',
                'on_demand_base': 25,
                'spot_percentage': 75,
                'spot_allocation': '最低價格'
            }
        else:
            return {
                'strategy': '帶有回退的 Spot',
                'primary': 'Spot',
                'fallback': '按需',
                'checkpointing': True
            }
    
    def create_spot_configuration(self):
        """建立 Spot 實例配置"""
        return '''
# 用於 Spot 實例的 Terraform 配置
resource "aws_spot_fleet_request" "processing_fleet" {
  iam_fleet_role = aws_iam_role.spot_fleet.arn
  
  allocation_strategy = "diversified"
  target_capacity     = 100
  valid_until        = timeadd(timestamp(), "168h")
  
  # 定義多個啟動規範以實現多樣性
  dynamic "launch_specification" {
    for_each = var.spot_instance_types
    
    content {
      instance_type     = launch_specification.value
      ami              = var.ami_id
      key_name         = var.key_name
      subnet_id        = var.subnet_ids[launch_specification.key % length(var.subnet_ids)]
      
      weighted_capacity = var.instance_weights[launch_specification.value]
      spot_price       = var.max_spot_prices[launch_specification.value]
      
      user_data = base64encode(templatefile("${path.module}/spot-init.sh", {
        interruption_handler = true
        checkpoint_s3_bucket = var.checkpoint_bucket
      }))
      
      tags = {
        Name = "spot-processing-${launch_specification.key}"
        Type = "spot"
      }
    }
  }
  
  # 中斷處理
  lifecycle {
    create_before_destroy = true
  }
}

# Spot 中斷處理器
resource "aws_lambda_function" "spot_interruption_handler" {
  filename         = "spot-handler.zip"
  function_name    = "spot-interruption-handler"
  role            = aws_iam_role.lambda_role.arn
  handler         = "handler.main"
  runtime         = "python3.9"
  
  environment {
    variables = {
      CHECKPOINT_BUCKET = var.checkpoint_bucket
      SNS_TOPIC_ARN    = aws_sns_topic.spot_interruptions.arn
    }
  }
}
'''
```

### 5. 儲存優化

優化儲存成本：

**儲存優化器**
```python
class StorageOptimizer:
    def analyze_storage_costs(self):
        """全面儲存分析"""
        analysis = {
            'ebs_volumes': self._analyze_ebs_volumes(),
            's3_buckets': self._analyze_s3_buckets(),
            'snapshots': self._analyze_snapshots(),
            'lifecycle_opportunities': self._find_lifecycle_opportunities(),
            'compression_opportunities': self._find_compression_opportunities()
        }
        
        return analysis
    
    def _analyze_s3_buckets(self):
        """分析 S3 儲存桶成本和優化"""
        s3 = boto3.client('s3')
        cloudwatch = boto3.client('cloudwatch')
        
        buckets = s3.list_buckets()['Buckets']
        bucket_analysis = []
        
        for bucket in buckets:
            bucket_name = bucket['Name']
            
            # 獲取儲存指標
            metrics = self._get_s3_metrics(bucket_name)
            
            # 分析儲存類別分佈
            storage_class_distribution = self._get_storage_class_distribution(bucket_name)
            
            # 計算優化潛力
            optimization = self._calculate_s3_optimization(
                bucket_name,
                metrics,
                storage_class_distribution
            )
            
            bucket_analysis.append({
                'bucket_name': bucket_name,
                'total_size_gb': metrics['size_gb'],
                'total_objects': metrics['object_count'],
                'current_cost': metrics['monthly_cost'],
                'storage_classes': storage_class_distribution,
                'optimization_recommendations': optimization['recommendations'],
                'potential_savings': optimization['savings']
            })
        
        return bucket_analysis
    
    def create_lifecycle_policies(self):
        """建立 S3 生命週期策略"""
        return '''
import boto3
from datetime import datetime

class S3LifecycleManager:
    def __init__(self):
        self.s3 = boto3.client('s3')
        
    def create_intelligent_lifecycle(self, bucket_name: str, access_patterns: Dict):
        """根據存取模式建立生命週期策略"""
        
        rules = []
        
        # 針對未知存取模式的智慧分層
        if access_patterns.get('unpredictable'):
            rules.append({
                'ID': 'intelligent-tiering',
                'Status': 'Enabled',
                'Transitions': [{
                    'Days': 1,
                    'StorageClass': 'INTELLIGENT_TIERING'
                }]
            })
        
        # 針對可預測模式的標準生命週期
        if access_patterns.get('predictable'):
            rules.append({
                'ID': 'standard-lifecycle',
                'Status': 'Enabled',
                'Transitions': [
                    {
                        'Days': 30,
                        'StorageClass': 'STANDARD_IA'
                    },
                    {
                        'Days': 90,
                        'StorageClass': 'GLACIER'
                    },
                    {
                        'Days': 180,
                        'StorageClass': 'DEEP_ARCHIVE'
                    }
                ]
            })
        
        # 刪除舊版本
        rules.append({
            'ID': 'delete-old-versions',
            'Status': 'Enabled',
            'NoncurrentVersionTransitions': [
                {
                    'NoncurrentDays': 30,
                    'StorageClass': 'GLACIER'
                }
            ],
            'NoncurrentVersionExpiration': {
                'NoncurrentDays': 90
            }
        })
        
        # 應用生命週期配置
        self.s3.put_bucket_lifecycle_configuration(
            Bucket=bucket_name,
            LifecycleConfiguration={'Rules': rules}
        )
        
        return rules
    
    def optimize_ebs_volumes(self):
        """優化 EBS 卷類型和大小"""
        ec2 = boto3.client('ec2')
        
        volumes = ec2.describe_volumes()['Volumes']
        optimizations = []
        
        for volume in volumes:
            # 分析卷指標
            iops_usage = self._get_volume_iops_usage(volume['VolumeId'])
            throughput_usage = self._get_volume_throughput_usage(volume['VolumeId'])
            
            current_type = volume['VolumeType']
            recommended_type = self._recommend_volume_type(
                iops_usage,
                throughput_usage,
                volume['Size']
            )
            
            if recommended_type != current_type:
                optimizations.append({
                    'volume_id': volume['VolumeId'],
                    'current_type': current_type,
                    'recommended_type': recommended_type,
                    'reason': self._get_optimization_reason(
                        current_type,
                        recommended_type,
                        iops_usage,
                        throughput_usage
                    ),
                    'monthly_savings': self._calculate_volume_savings(
                        volume,
                        recommended_type
                    )
                })
        
        return optimizations
'''
```

### 6. 網路成本優化

降低網路傳輸成本：

**網路成本優化器**
```python
class NetworkCostOptimizer:
    def analyze_network_costs(self):
        """分析網路傳輸成本"""
        analysis = {
            'data_transfer_costs': self._analyze_data_transfer(),
            'nat_gateway_costs': self._analyze_nat_gateways(),
            'load_balancer_costs': self._analyze_load_balancers(),
            'vpc_endpoint_opportunities': self._find_vpc_endpoint_opportunities(),
            'cdn_optimization': self._analyze_cdn_usage()
        }
        
        return analysis
    
    def _analyze_data_transfer(self):
        """分析資料傳輸模式和成本"""
        transfers = {
            'inter_region': self._get_inter_region_transfers(),
            'internet_egress': self._get_internet_egress(),
            'inter_az': self._get_inter_az_transfers(),
            'vpc_peering': self._get_vpc_peering_transfers()
        }
        
        recommendations = []
        
        # 分析跨區域傳輸
        if transfers['inter_region']['monthly_gb'] > 1000:
            recommendations.append({
                'type': '區域整合',
                'description': '考慮將資源整合到更少的區域',
                'current_cost': transfers['inter_region']['monthly_cost'],
                'potential_savings': transfers['inter_region']['monthly_cost'] * 0.8
            })
        
        # 分析網際網路出口
        if transfers['internet_egress']['monthly_gb'] > 10000:
            recommendations.append({
                'type': 'CDN 實施',
                'description': '實施 CDN 以減少源出口',
                'current_cost': transfers['internet_egress']['monthly_cost'],
                'potential_savings': transfers['internet_egress']['monthly_cost'] * 0.6
            })
        
        return {
            'current_costs': transfers,
            'recommendations': recommendations
        }
    
    def create_network_optimization_script(self):
        """實施網路優化的腳本"""
        return '''
#!/usr/bin/env python3
import boto3
from collections import defaultdict

class NetworkOptimizer:
    def __init__(self):
        self.ec2 = boto3.client('ec2')
        self.cloudwatch = boto3.client('cloudwatch')
        
    def optimize_nat_gateways(self):
        """整合和優化 NAT 閘道"""
        # 獲取所有 NAT 閘道
        nat_gateways = self.ec2.describe_nat_gateways()['NatGateways']
        
        # 按 VPC 分組
        vpc_nat_gateways = defaultdict(list)
        for nat in nat_gateways:
            if nat['State'] == 'available':
                vpc_nat_gateways[nat['VpcId']].append(nat)
        
        optimizations = []
        
        for vpc_id, nats in vpc_nat_gateways.items():
            if len(nats) > 1:
                # 檢查是否可以整合
                traffic_analysis = self._analyze_nat_traffic(nats)
                
                if traffic_analysis['can_consolidate']:
                    optimizations.append({
                        'vpc_id': vpc_id,
                        'action': '整合 NAT',
                        'current_count': len(nats),
                        'recommended_count': traffic_analysis['recommended_count'],
                        'monthly_savings': (len(nats) - traffic_analysis['recommended_count']) * 45
                    })
        
        return optimizations
    
    def implement_vpc_endpoints(self):
        """為 AWS 服務實施 VPC 端點"""
        services_to_check = ['s3', 'dynamodb', 'ec2', 'sns', 'sqs']
        vpc_list = self.ec2.describe_vpcs()['Vpcs']
        
        implementations = []
        
        for vpc in vpc_list:
            vpc_id = vpc['VpcId']
            
            # 檢查現有端點
            existing = self._get_existing_endpoints(vpc_id)
            
            for service in services_to_check:
                if service not in existing:
                    # 檢查服務是否正在使用
                    if self._is_service_used(vpc_id, service):
                        # 建立 VPC 端點
                        endpoint = self._create_vpc_endpoint(vpc_id, service)
                        
                        implementations.append({
                            'vpc_id': vpc_id,
                            'service': service,
                            'endpoint_id': endpoint['VpcEndpointId'],
                            'estimated_savings': self._estimate_endpoint_savings(vpc_id, service)
                        })
        
        return implementations
    
    def optimize_cloudfront_distribution(self):
        """優化 CloudFront 以降低成本"""
        cloudfront = boto3.client('cloudfront')
        
        distributions = cloudfront.list_distributions()
        optimizations = []
        
        for dist in distributions.get('DistributionList', {}).get('Items', []):
            # 分析分佈模式
            analysis = self._analyze_distribution(dist['Id'])
            
            if analysis['optimization_potential']:
                optimizations.append({
                    'distribution_id': dist['Id'],
                    'recommendations': [
                        {
                            'action': '調整價格類別',
                            'current': dist['PriceClass'],
                            'recommended': analysis['recommended_price_class'],
                            'savings': analysis['price_class_savings']
                        },
                        {
                            'action': '優化快取行為',
                            'cache_improvements': analysis['cache_improvements'],
                            'savings': analysis['cache_savings']
                        }
                    ]
                })
        
        return optimizations
'''
```

### 7. 容器成本優化

優化容器工作負載：

**容器成本優化器**
```python
class ContainerCostOptimizer:
    def optimize_ecs_costs(self):
        """優化 ECS/Fargate 成本"""
        return {
            'cluster_optimization': self._optimize_clusters(),
            'task_rightsizing': self._rightsize_tasks(),
            'scheduling_optimization': self._optimize_scheduling(),
            'fargate_spot': self._implement_fargate_spot()
        }
    
    def _rightsize_tasks(self):
        """調整 ECS 任務大小"""
        ecs = boto3.client('ecs')
        cloudwatch = boto3.client('cloudwatch')
        
        clusters = ecs.list_clusters()['clusterArns']
        recommendations = []
        
        for cluster in clusters:
            # 獲取服務
            services = ecs.list_services(cluster=cluster)['serviceArns']
            
            for service in services:
                # 獲取任務定義
                service_detail = ecs.describe_services(
                    cluster=cluster,
                    services=[service]
                )['services'][0]
                
                task_def = service_detail['taskDefinition']
                
                # 分析資源利用率
                utilization = self._analyze_task_utilization(cluster, service)
                
                # 生成建議
                if utilization['cpu']['average'] < 30 or utilization['memory']['average'] < 40:
                    recommendations.append({
                        'cluster': cluster,
                        'service': service,
                        'current_cpu': service_detail['cpu'],
                        'current_memory': service_detail['memory'],
                        'recommended_cpu': int(service_detail['cpu'] * 0.7),
                        'recommended_memory': int(service_detail['memory'] * 0.8),
                        'monthly_savings': self._calculate_task_savings(
                            service_detail,
                            utilization
                        )
                    })
        
        return recommendations
    
    def create_k8s_cost_optimization(self):
        """Kubernetes 成本優化"""
        return '''
apiVersion: v1
kind: ConfigMap
metadata:
  name: cost-optimization-config
data:
  vertical-pod-autoscaler.yaml: |
    apiVersion: autoscaling.k8s.io/v1
    kind: VerticalPodAutoscaler
    metadata:
      name: app-vpa
    spec:
      targetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: app-deployment
      updatePolicy:
        updateMode: "Auto"
      resourcePolicy:
        containerPolicies:
        - containerName: app
          minAllowed:
            cpu: 100m
            memory: 128Mi
          maxAllowed:
            cpu: 2
            memory: 2Gi
  
  cluster-autoscaler-config.yaml: |
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: cluster-autoscaler
    spec:
      template:
        spec:
          containers:
          - image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.21.0
            name: cluster-autoscaler
            command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=priority
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/cluster-name
            - --scale-down-enabled=true
            - --scale-down-unneeded-time=10m
            - --scale-down-utilization-threshold=0.5
  
  spot-instance-handler.yaml: |
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: aws-node-termination-handler
    spec:
      selector:
        matchLabels:
          app: aws-node-termination-handler
      template:
        spec:
          containers:
          - name: aws-node-termination-handler
            image: amazon/aws-node-termination-handler:v1.13.0
            env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
            - name: ENABLE_SPOT_INTERRUPTION_DRAINING
              value: "true"
            - name: ENABLE_SCHEDULED_EVENT_DRAINING
              value: "true"
'''
```

### 8. 無伺服器成本優化

優化無伺服器工作負載：

**無伺服器優化器**
```python
class ServerlessOptimizer:
    def optimize_lambda_costs(self):
        """優化 Lambda 函數成本"""
        lambda_client = boto3.client('lambda')
        cloudwatch = boto3.client('cloudwatch')
        
        functions = lambda_client.list_functions()['Functions']
        optimizations = []
        
        for function in functions:
            # 分析函數性能
            analysis = self._analyze_lambda_function(function)
            
            # 記憶體優化
            if analysis['memory_optimization_possible']:
                optimizations.append({
                    'function_name': function['FunctionName'],
                    'type': '記憶體優化',
                    'current_memory': function['MemorySize'],
                    'recommended_memory': analysis['optimal_memory'],
                    'estimated_savings': analysis['memory_savings']
                })
            
            # 超時優化
            if analysis['timeout_optimization_possible']:
                optimizations.append({
                    'function_name': function['FunctionName'],
                    'type': '超時優化',
                    'current_timeout': function['Timeout'],
                    'recommended_timeout': analysis['optimal_timeout'],
                    'risk_reduction': '防止因函數掛起而產生不必要的費用'
                })
        
        return optimizations
    
    def implement_lambda_cost_controls(self):
        """實施 Lambda 成本控制"""
        return '''
import json
import boto3
from datetime import datetime

def lambda_cost_controller(event, context):
    """用於監控和控制 Lambda 成本的 Lambda 函數"""
    
    cloudwatch = boto3.client('cloudwatch')
    lambda_client = boto3.client('lambda')
    
    # 獲取當前月份成本
    costs = get_current_month_lambda_costs()
    
    # 檢查預算
    budget_limit = float(os.environ.get('MONTHLY_BUDGET', '1000'))
    
    if costs > budget_limit * 0.8:  # 預算的 80%
        # 實施成本控制
        high_cost_functions = identify_high_cost_functions()
        
        for func in high_cost_functions:
            # 減少並發
            lambda_client.put_function_concurrency(
                FunctionName=func['FunctionName'],
                ReservedConcurrentExecutions=max(
                    1, 
                    int(func['CurrentConcurrency'] * 0.5)
                )
            )
            
            # 警報
            send_cost_alert(func, costs, budget_limit)
    
    # 實施預置並發優化
    optimize_provisioned_concurrency()
    
    return {
        'statusCode': 200,
        'body': json.dumps({
            'current_costs': costs,
            'budget_limit': budget_limit,
            'actions_taken': len(high_cost_functions)
        })
    }

def optimize_provisioned_concurrency():
    """根據使用模式優化預置並發"""
    functions = get_functions_with_provisioned_concurrency()
    
    for func in functions:
        # 分析調用模式
        patterns = analyze_invocation_patterns(func['FunctionName'])
        
        if patterns['predictable']:
            # 排程預置並發
            create_scheduled_scaling(
                func['FunctionName'],
                patterns['peak_hours'],
                patterns['peak_concurrency']
            )
        else:
            # 考慮移除預置並發
            if patterns['avg_cold_starts'] < 10:  # 每分鐘
                remove_provisioned_concurrency(func['FunctionName'])
'''
```

### 9. 成本分配和標籤

實施成本分配策略：

**成本分配管理器**
```python
class CostAllocationManager:
    def implement_tagging_strategy(self):
        """實施全面的標籤策略"""
        return {
            'required_tags': [
                {'key': 'Environment', 'values': ['prod', 'staging', 'dev', 'test']},
                {'key': 'CostCenter', 'values': 'dynamic'},
                {'key': 'Project', 'values': 'dynamic'},
                {'key': 'Owner', 'values': 'dynamic'},
                {'key': 'Department', 'values': 'dynamic'}
            ],
            'automation': self._create_tagging_automation(),
            'enforcement': self._create_tag_enforcement(),
            'reporting': self._create_cost_allocation_reports()
        }
    
    def _create_tagging_automation(self):
        """自動化資源標籤"""
        return '''
import boto3
from datetime import datetime

class AutoTagger:
    def __init__(self):
        self.tag_policies = self.load_tag_policies()
        
    def auto_tag_resources(self, event, context):
        """在建立時自動標記資源"""
        
        # 解析 CloudTrail 事件
        detail = event['detail']
        event_name = detail['eventName']
        
        # 將事件映射到資源類型
        if event_name.startswith('Create'):
            resource_arn = self.extract_resource_arn(detail)
            
            if resource_arn:
                # 確定標籤
                tags = self.determine_tags(detail)
                
                # 應用標籤
                self.apply_tags(resource_arn, tags)
                
                # 記錄標籤操作
                self.log_tagging(resource_arn, tags)
    
    def determine_tags(self, event_detail):
        """根據上下文確定標籤"""
        tags = []
        
        # 基於使用者的標籤
        user_identity = event_detail.get('userIdentity', {})
        if 'userName' in user_identity:
            tags.append({
                'Key': 'Creator',
                'Value': user_identity['userName']
            })
        
        # 基於時間的標籤
        tags.append({
            'Key': 'CreatedDate',
            'Value': datetime.now().strftime('%Y-%m-%d')
        })
        
        # 環境推斷
        if 'prod' in event_detail.get('sourceIPAddress', ''):
            env = 'prod'
        elif 'dev' in event_detail.get('sourceIPAddress', ''):
            env = 'dev'
        else:
            env = 'unknown'
            
        tags.append({
            'Key': 'Environment',
            'Value': env
        })
        
        return tags
    
    def create_cost_allocation_dashboard(self):
        """建立成本分配儀表板"""
        return """
        SELECT 
            tags.environment,
            tags.department,
            tags.project,
            SUM(costs.amount) as total_cost,
            SUM(costs.amount) / SUM(SUM(costs.amount)) OVER () * 100 as percentage
        FROM 
            aws_costs costs
        JOIN 
            resource_tags tags ON costs.resource_id = tags.resource_id
        WHERE 
            costs.date >= DATE_TRUNC('month', CURRENT_DATE)
        GROUP BY 
            tags.environment,
            tags.department,
            tags.project
        ORDER BY 
            total_cost DESC
        """
'''
```

### 10. 成本監控和警報

實施主動成本監控：

**成本監控系統**
```python
class CostMonitoringSystem:
    def setup_cost_alerts(self):
        """設定全面的成本警報"""
        alerts = []
        
        # 預算警報
        alerts.extend(self._create_budget_alerts())
        
        # 異常檢測
        alerts.extend(self._create_anomaly_alerts())
        
        # 閾值警報
        alerts.extend(self._create_threshold_alerts())
        
        # 預測警報
        alerts.extend(self._create_forecast_alerts())
        
        return alerts
    
    def _create_anomaly_alerts(self):
        """建立異常檢測警報"""
        ce = boto3.client('ce')
        
        # 建立異常監控器
        monitor = ce.create_anomaly_monitor(
            AnomalyMonitor={
                'MonitorName': '服務成本監控器',
                'MonitorType': 'DIMENSIONAL',
                'MonitorDimension': 'SERVICE'
            }
        )
        
        # 建立異常訂閱
        subscription = ce.create_anomaly_subscription(
            AnomalySubscription={
                'SubscriptionName': '成本異常警報',
                'Threshold': 100.0,  # 異常 > $100 時警報
                'Frequency': 'DAILY',
                'MonitorArnList': [monitor['MonitorArn']],
                'Subscribers': [
                    {
                        'Type': 'EMAIL',
                        'Address': 'team@company.com'
                    },
                    {
                        'Type': 'SNS',
                        'Address': 'arn:aws:sns:us-east-1:123456789012:cost-alerts'
                    }
                ]
            }
        )
        
        return [monitor, subscription]
    
    def create_cost_dashboard(self):
        """建立執行成本儀表板"""
        return '''
<!DOCTYPE html>
<html>
<head>
    <title>雲端成本儀表板</title>
    <script src="https://d3js.org/d3.v7.min.js"></script>
    <style>
        .metric-card {
            background: #f5f5f5;
            padding: 20px;
            margin: 10px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }
        .alert { color: #d32f2f; }
        .warning { color: #f57c00; }
        .success { color: #388e3c; }
    </style>
</head>
<body>
    <div id="dashboard">
        <h1>雲端成本優化儀表板</h1>
        
        <div class="summary-row">
            <div class="metric-card">
                <h3>當月支出</h3>
                <div class="metric">${current_spend}</div>
                <div class="trend ${spend_trend_class}">與上月相比 ${spend_trend}%</div>
            </div>
            
            <div class="metric-card">
                <h3>預計月底</h3>
                <div class="metric">${projected_spend}</div>
                <div class="budget-status">預算: ${budget}</div>
            </div>
            
            <div class="metric-card">
                <h3>優化機會</h3>
                <div class="metric">${total_savings_identified}</div>
                <div class="count">{opportunity_count} 個建議</div>
            </div>
            
            <div class="metric-card">
                <h3>已實現節省</h3>
                <div class="metric">${realized_savings_mtd}</div>
                <div class="count">年初至今: ${realized_savings_ytd}</div>
            </div>
        </div>
        
        <div class="charts-row">
            <div id="spend-trend-chart"></div>
            <div id="service-breakdown-chart"></div>
            <div id="optimization-progress-chart"></div>
        </div>
        
        <div class="recommendations-section">
            <h2>熱門優化建議</h2>
            <table id="recommendations-table">
                <thead>
                    <tr>
                        <th>優先級</th>
                        <th>服務</th>
                        <th>建議</th>
                        <th>每月節省</th>
                        <th>工作量</th>
                        <th>行動</th>
                    </tr>
                </thead>
                <tbody>
                    ${recommendation_rows}
                </tbody>
            </table>
        </div>
    </div>
    
    <script>
        // 即時更新
        setInterval(updateDashboard, 60000);
        
        // 初始化圖表
        initializeCharts();
    </script>
</body>
</html>
'''
```

## 輸出格式

1. **成本分析報告**：當前雲端成本的全面細分
2. **優化建議**：節省成本機會的優先列表
3. **實施腳本**：用於實施優化的自動化腳本
4. **監控儀表板**：即時成本追蹤和警報
5. **投資回報率計算**：詳細的節省預測和回收期
6. **風險評估**：與每個優化相關的風險分析
7. **實施路線圖**：成本優化的分階段方法
8. **最佳實踐指南**：長期成本管理策略

專注於提供即時成本節省，同時建立可持續的成本優化實踐，以維持性能和可靠性標準。
