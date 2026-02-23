# Guia Detalhado: Amazon CloudWatch API

## Visão Geral

O CloudWatch é o serviço central de monitoramento da AWS. Para FinOps, ele é essencial para coletar métricas de utilização que fundamentam decisões de right-sizing e otimização.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://monitoring.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `monitoring` |
| **Versão da API** | `2010-08-01` |

---

## Ações da API

### 1. GetMetricData

Retorna dados de uma ou mais métricas em um período. Esta é a ação principal para coleta de métricas para análise de FinOps.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MetricDataQueries` | Array | Sim | Lista de estruturas que definem as métricas a serem consultadas. |
| `StartTime` | Timestamp | Sim | Início do período de consulta. |
| `EndTime` | Timestamp | Sim | Fim do período de consulta. |
| `NextToken` | String | Não | Token para paginação. |
| `ScanBy` | String | Não | `TimestampDescending` ou `TimestampAscending` (padrão). |
| `MaxDatapoints` | Integer | Não | Número máximo de pontos de dados a retornar. |
| `LabelOptions` | Object | Não | Opções para formatação de labels. |

#### Estrutura `MetricDataQuery`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Id` | String | Sim | ID único para a consulta. |
| `MetricStat` | Object | Sim (se não for `Expression`) | Define a métrica e a estatística. |
| `Expression` | String | Sim (se não for `MetricStat`) | Expressão matemática para combinar métricas. |
| `Label` | String | Não | Rótulo para a série temporal. |
| `ReturnData` | Boolean | Não | Se `true`, retorna os dados desta consulta. |
| `Period` | Integer | Não | Granularidade em segundos. |
| `AccountId` | String | Não | ID da conta para consulta cross-account. |

#### Exemplo de Requisição (CPU e Rede de uma Instância EC2)

```
Action=GetMetricData
&Version=2010-08-01
&StartTime=2026-02-22T00:00:00Z
&EndTime=2026-02-23T00:00:00Z
&MetricDataQueries.member.1.Id=cpu_util
&MetricDataQueries.member.1.MetricStat.Metric.Namespace=AWS/EC2
&MetricDataQueries.member.1.MetricStat.Metric.MetricName=CPUUtilization
&MetricDataQueries.member.1.MetricStat.Metric.Dimensions.member.1.Name=InstanceId
&MetricDataQueries.member.1.MetricStat.Metric.Dimensions.member.1.Value=i-1234567890abcdef0
&MetricDataQueries.member.1.MetricStat.Period=3600
&MetricDataQueries.member.1.MetricStat.Stat=Average
&MetricDataQueries.member.1.ReturnData=true
&MetricDataQueries.member.2.Id=network_out
&MetricDataQueries.member.2.MetricStat.Metric.Namespace=AWS/EC2
&MetricDataQueries.member.2.MetricStat.Metric.MetricName=NetworkOut
&MetricDataQueries.member.2.MetricStat.Metric.Dimensions.member.1.Name=InstanceId
&MetricDataQueries.member.2.MetricStat.Metric.Dimensions.member.1.Value=i-1234567890abcdef0
&MetricDataQueries.member.2.MetricStat.Period=3600
&MetricDataQueries.member.2.MetricStat.Stat=Sum
&MetricDataQueries.member.2.ReturnData=true
```

---

### 2. ListMetrics

Lista as métricas disponíveis para um namespace (serviço).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Namespace` | String | Não | O namespace do serviço (ex: `AWS/EC2`). |
| `MetricName` | String | Não | O nome da métrica. |
| `Dimensions` | Array | Não | Filtro por dimensões. |
| `NextToken` | String | Não | Token para paginação. |
| `RecentlyActive` | String | Não | `PT3H` para listar apenas métricas ativas nas últimas 3 horas. |
| `IncludeLinkedAccounts` | Boolean | Não | Incluir métricas de contas vinculadas. |
| `OwningAccount` | String | Não | ID da conta proprietária das métricas. |

#### Exemplo de Requisição

```
Action=ListMetrics
&Version=2010-08-01
&Namespace=AWS/EC2
&MetricName=CPUUtilization
&RecentlyActive=PT3H
```

---

### 3. PutMetricAlarm

Cria ou atualiza um alarme que monitora uma métrica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AlarmName` | String | Sim | Nome único para o alarme. |
| `MetricName` | String | Sim | Nome da métrica a ser monitorada. |
| `Namespace` | String | Sim | Namespace da métrica. |
| `Statistic` | String | Sim | Estatística a ser aplicada (`Average`, `Sum`, `Maximum`, etc.). |
| `Period` | Integer | Sim | Período em segundos para avaliação. |
| `EvaluationPeriods` | Integer | Sim | Número de períodos consecutivos para o alarme disparar. |
| `Threshold` | Double | Sim | Valor limite para a métrica. |
| `ComparisonOperator` | String | Sim | `GreaterThanOrEqualToThreshold`, `GreaterThanThreshold`, `LessThanThreshold`, `LessThanOrEqualToThreshold`. |
| `AlarmDescription` | String | Não | Descrição do alarme. |
| `ActionsEnabled` | Boolean | Não | Se as ações do alarme estão ativas. |
| `OKActions` | Array | Não | ARNs de ações a serem executadas quando o alarme vai para o estado OK. |
| `AlarmActions` | Array | Não | ARNs de ações a serem executadas quando o alarme vai para o estado ALARM. |
| `InsufficientDataActions` | Array | Não | ARNs de ações a serem executadas quando não há dados suficientes. |
| `Dimensions` | Array | Não | Dimensões da métrica. |
| `Unit` | String | Não | Unidade da métrica. |

#### Exemplo de Requisição (Alarme de Custo Estimado)

```
Action=PutMetricAlarm
&Version=2010-08-01
&AlarmName=EstimatedCharges-GreaterThan-1000
&AlarmDescription="Alarme quando os custos estimados excederem $1000"
&MetricName=EstimatedCharges
&Namespace=AWS/Billing
&Statistic=Maximum
&Period=21600
&EvaluationPeriods=1
&Threshold=1000
&ComparisonOperator=GreaterThanThreshold
&Dimensions.member.1.Name=Currency
&Dimensions.member.1.Value=USD
&AlarmActions.member.1=arn:aws:sns:us-east-1:123456789012:MyFinOpsTopic
```

---

### 4. DescribeAlarms

Lista os alarmes configurados.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AlarmNames` | Array | Não | Lista de nomes de alarmes para descrever. |
| `AlarmNamePrefix` | String | Não | Prefixo para filtrar alarmes por nome. |
| `StateValue` | String | Não | Filtra por estado (`OK`, `ALARM`, `INSUFFICIENT_DATA`). |
| `ActionPrefix` | String | Não | Filtra por ARN de ação. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeAlarms
&Version=2010-08-01
&StateValue=ALARM
```
