# Guia Detalhado: AWS Cost Anomaly Detection API

## Visão Geral

O Cost Anomaly Detection utiliza machine learning para identificar automaticamente gastos anômalos na sua conta AWS. Ele monitora continuamente os padrões de custo e envia alertas quando detecta desvios significativos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ce.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSInsightsIndexService` |
| **Service Name (IAM)** | `ce` |

---

## Ações da API

### 1. CreateAnomalyMonitor

Cria um monitor de anomalias que rastreia custos por dimensão usando machine learning.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AnomalyMonitor` | Object | Sim | Objeto que define o monitor. |
| `ResourceTags` | Array | Não | Tags a serem associadas ao monitor. |

#### Objeto `AnomalyMonitor`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MonitorName` | String | Sim | Nome do monitor. |
| `MonitorType` | String | Sim | `DIMENSIONAL` (por serviço/conta) ou `CUSTOM` (com filtro personalizado). |
| `MonitorDimension` | String | Sim (se DIMENSIONAL) | `SERVICE` é o único valor suportado. |
| `MonitorSpecification` | Object | Sim (se CUSTOM) | Expressão de filtro personalizada. |

#### Exemplo de Requisição

```json
{
  "AnomalyMonitor": {
    "MonitorName": "MonitorPorServico",
    "MonitorType": "DIMENSIONAL",
    "MonitorDimension": "SERVICE"
  }
}
```

---

### 2. CreateAnomalySubscription

Cria uma assinatura para receber notificações quando anomalias são detectadas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AnomalySubscription` | Object | Sim | Objeto que define a assinatura. |
| `ResourceTags` | Array | Não | Tags a serem associadas à assinatura. |

#### Objeto `AnomalySubscription`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `SubscriptionName` | String | Sim | Nome da assinatura. |
| `Frequency` | String | Sim | `DAILY`, `IMMEDIATE`, `WEEKLY`. |
| `MonitorArnList` | Array | Sim | Lista de ARNs dos monitores a serem rastreados. |
| `Subscribers` | Array | Sim | Lista de destinatários (email ou SNS). |
| `ThresholdExpression` | Object | Não | Expressão para definir o limiar de notificação. |

#### Exemplo de Requisição

```json
{
  "AnomalySubscription": {
    "SubscriptionName": "AlertasFinOps",
    "Frequency": "DAILY",
    "MonitorArnList": [
      "arn:aws:ce::123456789012:anomalymonitor/abc123"
    ],
    "Subscribers": [
      {"Address": "finops@example.com", "Type": "EMAIL"}
    ],
    "ThresholdExpression": {
      "Dimensions": {
        "Key": "ANOMALY_TOTAL_IMPACT_ABSOLUTE",
        "Values": ["100"],
        "MatchOptions": ["GREATER_THAN_OR_EQUAL"]
      }
    }
  }
}
```

---

### 3. GetAnomalies

Retorna as anomalias de custo detectadas no período especificado.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `DateInterval` | Object | Sim | Período com `StartDate` e `EndDate` (formato `YYYY-MM-DD`). |
| `MonitorArn` | String | Não | Filtra anomalias de um monitor específico. |
| `Feedback` | String | Não | Filtra por feedback: `YES`, `NO`, `PLANNED_ACTIVITY`. |
| `TotalImpact` | Object | Não | Filtra por impacto total com `NumericOperator` e `StartValue`/`EndValue`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextPageToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "DateInterval": {
    "StartDate": "2026-01-01",
    "EndDate": "2026-02-23"
  },
  "TotalImpact": {
    "NumericOperator": "GREATER_THAN_OR_EQUAL",
    "StartValue": 50
  },
  "MaxResults": 100
}
```

---

### 4. ProvideAnomalyFeedback

Envia feedback sobre uma anomalia detectada, melhorando a precisão do modelo de ML.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AnomalyId` | String | Sim | O ID da anomalia. |
| `Feedback` | String | Sim | `YES` (anomalia real), `NO` (falso positivo), `PLANNED_ACTIVITY` (atividade planejada). |

#### Exemplo de Requisição

```json
{
  "AnomalyId": "abc123-def456",
  "Feedback": "YES"
}
```
