# 6. APIs de Monitoramento e Métricas

## 6.1 Amazon CloudWatch API

O CloudWatch é o serviço central de monitoramento da AWS. Para FinOps, ele é essencial para coletar métricas de utilização que fundamentam decisões de right-sizing e otimização.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://monitoring.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `monitoring` |
| **Versão da API** | `2010-08-01` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `ListMetrics` | Lista as métricas disponíveis para um namespace (serviço). |
| `GetMetricData` | Retorna dados de uma ou mais métricas em um período. **Ação principal para coleta de métricas.** |
| `GetMetricStatistics` | Retorna estatísticas de uma métrica específica (mais simples que GetMetricData). |
| `DescribeAlarms` | Lista os alarmes configurados. |
| `PutMetricAlarm` | Cria ou atualiza um alarme. |
| `GetInsightRuleReport` | Retorna relatórios de regras de insight. |

### Namespaces Mais Relevantes para FinOps

| Namespace | Serviço | Métricas Chave |
| :--- | :--- | :--- |
| `AWS/EC2` | EC2 | `CPUUtilization`, `NetworkIn`, `NetworkOut`, `DiskReadOps`, `DiskWriteOps` |
| `AWS/EBS` | EBS | `VolumeReadOps`, `VolumeWriteOps`, `VolumeIdleTime`, `BurstBalance` |
| `AWS/RDS` | RDS | `CPUUtilization`, `DatabaseConnections`, `FreeableMemory`, `ReadIOPS`, `WriteIOPS` |
| `AWS/Lambda` | Lambda | `Invocations`, `Duration`, `Errors`, `ConcurrentExecutions`, `Throttles` |
| `AWS/ECS` | ECS | `CPUUtilization`, `MemoryUtilization` |
| `AWS/ElastiCache` | ElastiCache | `CPUUtilization`, `CurrConnections`, `CacheHitRate` |
| `AWS/DynamoDB` | DynamoDB | `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits`, `ThrottledRequests` |
| `AWS/ELB` | ELB | `RequestCount`, `ActiveConnectionCount`, `ProcessedBytes` |
| `AWS/NATGateway` | NAT Gateway | `BytesInFromSource`, `BytesOutToDestination`, `PacketsDropCount` |
| `AWS/S3` | S3 | `BucketSizeBytes`, `NumberOfObjects` |
| `AWS/Billing` | Billing | `EstimatedCharges` (métrica especial, disponível apenas em us-east-1) |

### Formato da Requisição CloudWatch

Todas as requisições CloudWatch usam o formato Query API:

```
POST / HTTP/1.1
Host: monitoring.us-east-1.amazonaws.com
Content-Type: application/x-www-form-urlencoded

Action=ListMetrics&Version=2010-08-01&Namespace=AWS/EC2
```

### Exemplo: GetMetricData para CPU de EC2

```
Action=GetMetricData
&Version=2010-08-01
&MetricDataQueries.member.1.Id=cpu
&MetricDataQueries.member.1.MetricStat.Metric.Namespace=AWS/EC2
&MetricDataQueries.member.1.MetricStat.Metric.MetricName=CPUUtilization
&MetricDataQueries.member.1.MetricStat.Metric.Dimensions.member.1.Name=InstanceId
&MetricDataQueries.member.1.MetricStat.Metric.Dimensions.member.1.Value=i-1234567890abcdef0
&MetricDataQueries.member.1.MetricStat.Period=3600
&MetricDataQueries.member.1.MetricStat.Stat=Average
&MetricDataQueries.member.1.ReturnData=true
&StartTime=2026-02-22T00:00:00Z
&EndTime=2026-02-23T00:00:00Z
```

### Métricas Estratégicas para Right-Sizing

Para fundamentar decisões de right-sizing, as seguintes combinações de métricas são recomendadas:

| Recurso | Métricas a Coletar | O que Analisar |
| :--- | :--- | :--- |
| **EC2** | `CPUUtilization`, `NetworkIn/Out`, `DiskReadOps/WriteOps` | CPU < 10% por 14 dias = candidato a downsizing. |
| **EBS** | `VolumeReadOps`, `VolumeWriteOps`, `VolumeIdleTime` | IOPS baixo + idle time alto = candidato a mudança de tipo. |
| **RDS** | `CPUUtilization`, `DatabaseConnections`, `FreeableMemory` | CPU < 20% + poucas conexões = candidato a downsizing. |
| **Lambda** | `Duration`, `MemorySize` (via API Lambda) | Duration muito baixa com memória alta = candidato a redução de memória. |
| **ElastiCache** | `CPUUtilization`, `CurrConnections`, `CacheHitRate` | Baixa utilização = candidato a downsizing ou remoção. |
| **DynamoDB** | `ConsumedReadCapacityUnits`, `ConsumedWriteCapacityUnits` | Consumo muito abaixo do provisionado = candidato a on-demand. |

### Métrica Especial: EstimatedCharges

A métrica `EstimatedCharges` no namespace `AWS/Billing` fornece uma estimativa dos custos acumulados da conta no mês corrente. Ela está disponível apenas na região `us-east-1` e pode ser usada para criar alarmes de custo simples.

```
Action=GetMetricStatistics
&Version=2010-08-01
&Namespace=AWS/Billing
&MetricName=EstimatedCharges
&Dimensions.member.1.Name=Currency
&Dimensions.member.1.Value=USD
&StartTime=2026-02-01T00:00:00Z
&EndTime=2026-02-23T23:59:59Z
&Period=86400
&Statistics.member.1=Maximum
```

## Referências

- [CloudWatch API Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/APIReference/Welcome.html)
- [CloudWatch Metrics and Dimensions Reference](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html)
