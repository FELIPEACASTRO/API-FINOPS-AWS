 # Guia Devastadoramente Detalhado: Amazon CloudWatch API (para FinOps)

## Visão Geral

O CloudWatch é o serviço de monitoramento nativo da AWS. Para FinOps, ele é essencial para duas coisas principais: coletar métricas de utilização de recursos (como CPU de uma instância EC2) para alimentar decisões de right-sizing, e monitorar custos estimados para criar alarmes de faturamento.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://monitoring.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `cloudwatch` |

---

## 1. GetMetricData

Busca dados de métricas do CloudWatch. Esta é a chamada principal para obter dados de utilização de recursos como EC2, RDS, Lambda, etc.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MetricDataQueries` | Array de Objetos | **Sim** | A lista de consultas de métricas a serem executadas. Cada objeto de consulta define a métrica, o namespace, as dimensões, a estatística, etc. |
| `StartTime` | Timestamp | **Sim** | O início do período de tempo para a consulta. |
| `EndTime` | Timestamp | **Sim** | O fim do período de tempo para a consulta. |
| `ScanBy` | String | Não | `TimestampAscending` ou `TimestampDescending`. Define a ordem dos pontos de dados retornados. |
| `MaxDatapoints` | Integer | Não | O número máximo de pontos de dados a serem retornados. |
| `NextToken` | String | Não | Token de paginação. |

#### Objeto `MetricDataQuery`

| Campo | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Id` | String | **Sim** | Um ID único para a consulta (ex: `"q1"`). |
| `MetricStat` | Objeto | **Sim** | Define a métrica a ser buscada. Contém `Metric` (com `Namespace`, `MetricName`, `Dimensions`), `Period` (em segundos) e `Stat` (`Average`, `Maximum`, `Sum`, etc.). |
| `Label` | String | Não | Um rótulo para a série temporal nos resultados. |
| `ReturnData` | Boolean | Não | Se `true`, retorna os dados para esta consulta. Se `false`, a consulta pode ser usada apenas para cálculos matemáticos em outras consultas. |

### Exemplo de Requisição (Uso médio de CPU de uma instância EC2 nos últimos 7 dias)

```json
{
  "StartTime": "2026-02-16T00:00:00Z",
  "EndTime": "2026-02-23T00:00:00Z",
  "MetricDataQueries": [
    {
      "Id": "cpu_utilization",
      "MetricStat": {
        "Metric": {
          "Namespace": "AWS/EC2",
          "MetricName": "CPUUtilization",
          "Dimensions": [
            {
              "Name": "InstanceId",
              "Value": "i-1234567890abcdef0"
            }
          ]
        },
        "Period": 86400, // 1 dia em segundos
        "Stat": "Average"
      },
      "ReturnData": true
    }
  ]
}
```

---

## 2. ListMetrics

Lista as métricas que o CloudWatch tem dados. Útil para explorar quais métricas estão disponíveis para um determinado serviço ou recurso.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Namespace` | String | Não | O namespace do serviço (ex: `AWS/EC2`). |
| `MetricName` | String | Não | O nome da métrica. |
| `Dimensions` | Array de Objetos | Não | Filtra por dimensões específicas (ex: `Name=InstanceId, Value=i-123...`). |
| `RecentlyActive` | String | Não | `PT3H` para listar apenas métricas que reportaram dados nas últimas 3 horas. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar todas as métricas para uma instância EC2 específica)

```json
{
  "Namespace": "AWS/EC2",
  "Dimensions": [
    {
      "Name": "InstanceId",
      "Value": "i-1234567890abcdef0"
    }
  ]
}
```

---

## 3. PutMetricAlarm / DescribeAlarms

`PutMetricAlarm` cria ou atualiza um alarme, enquanto `DescribeAlarms` retorna informações sobre os alarmes existentes. Alarmes de faturamento são uma prática fundamental de FinOps.

### Parâmetros de `PutMetricAlarm`

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AlarmName` | String | **Sim** | O nome do alarme. |
| `MetricName` | String | **Sim** | O nome da métrica a ser monitorada. Para faturamento, use `EstimatedCharges`. |
| `Namespace` | String | **Sim** | O namespace. Para faturamento, use `AWS/Billing`. |
| `Statistic` | String | **Sim** | A estatística a ser aplicada (`Maximum` para faturamento). |
| `Period` | Integer | **Sim** | O período em segundos sobre o qual a estatística é aplicada. |
| `EvaluationPeriods` | Integer | **Sim** | O número de períodos consecutivos que a métrica deve violar o limiar para o alarme disparar. |
| `Threshold` | Double | **Sim** | O valor do limiar a ser violado. |
| `ComparisonOperator` | String | **Sim** | `GreaterThanOrEqualToThreshold`, `GreaterThanThreshold`, `LessThanThreshold`, `LessThanOrEqualToThreshold`. |
| `AlarmActions` | Array de Strings | Não | Uma lista de ARNs de ações a serem tomadas quando o alarme dispara (ex: ARN de um tópico SNS). |

### Exemplo de Requisição (Criar um alarme de faturamento para quando os custos excederem $1000)

```json
{
  "AlarmName": "Billing-Alarm-1000USD",
  "MetricName": "EstimatedCharges",
  "Namespace": "AWS/Billing",
  "Statistic": "Maximum",
  "Period": 21600, // 6 horas
  "EvaluationPeriods": 1,
  "Threshold": 1000,
  "ComparisonOperator": "GreaterThanOrEqualToThreshold",
  "Dimensions": [
    {
      "Name": "Currency",
      "Value": "USD"
    }
  ],
  "AlarmActions": ["arn:aws:sns:us-east-1:123456789012:MyBillingAlertsTopic"]
}
```
