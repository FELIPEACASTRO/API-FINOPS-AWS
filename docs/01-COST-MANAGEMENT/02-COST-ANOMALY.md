# Guia Devastadoramente Detalhado: AWS Cost Anomaly Detection API

## Visão Geral

A detecção de anomalias é uma capacidade proativa do FinOps. Esta API permite que você crie monitores que vigiam seus padrões de gastos e o alertam automaticamente sobre custos inesperados, permitindo uma ação rápida para evitar surpresas na fatura. O serviço usa machine learning para aprender seus padrões de gastos e identificar desvios significativos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ce.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSInsightsIndexService` |
| **Service Name (IAM)** | `ce` |

---

## 1. CreateAnomalyMonitor

Cria um novo monitor para começar a detectar anomalias nos seus custos. Pense em um monitor como um "vigia" que você configura para olhar para uma parte específica dos seus custos.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AnomalyMonitor` | Objeto | **Sim** | O objeto principal que define a configuração do monitor. Contém as seguintes chaves:<br>- `MonitorName` (String, **Sim**): O nome que você dará ao monitor para identificá-lo facilmente (ex: "Monitor-Custo-EC2", "Monitor-Conta-Dev").<br>- `MonitorType` (String, **Sim**): O tipo de monitor. `DIMENSIONAL` para monitorar uma dimensão padrão da AWS, ou `CUSTOM` para usar um agrupamento mais complexo definido por uma Cost Category.<br>- `MonitorDimension` (String, **Obrigatório se `MonitorType` for `DIMENSIONAL`**): A dimensão a ser monitorada. Valores comuns: `SERVICE` (para monitorar cada serviço AWS como uma unidade), `LINKED_ACCOUNT` (para monitorar cada conta vinculada).<br>- `MonitorSpecification` (Objeto, **Obrigatório se `MonitorType` for `CUSTOM`**): A especificação da Cost Category a ser usada, permitindo criar monitores para grupos lógicos de negócio (ex: por time, projeto, etc.). |

### Exemplo de Requisição (Monitor por Serviço)

```json
{
  "AnomalyMonitor": {
    "MonitorName": "Monitor-Geral-Por-Servico",
    "MonitorType": "DIMENSIONAL",
    "MonitorDimension": "SERVICE"
  }
}
```

---

## 2. GetAnomalies

Recupera uma lista de anomalias que foram detectadas dentro de um intervalo de datas. Use esta ação para investigar picos de custos passados.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `DateInterval` | Objeto | **Sim** | O intervalo de datas para buscar anomalias. Contém `StartDate` e `EndDate` no formato `YYYY-MM-DD`. Lembre-se que `EndDate` é exclusivo. |
| `MonitorArn` | String | Não | O ARN de um monitor específico. Use para focar a investigação em uma área, como os custos de uma conta específica, se você tiver um monitor para ela. Se não for fornecido, a API retorna anomalias de todos os monitores. |
| `Feedback` | String | Não | Filtra as anomalias com base no feedback que você forneceu anteriormente.<br>**Valores**: `YES` (anomalia confirmada), `NO` (não é uma anomalia), `PLANNED_ACTIVITY` (atividade planejada). Útil para revisar suas análises passadas. |
| `TotalImpact` | Objeto | Não | Filtra anomalias com base no impacto financeiro total. Contém `NumericOperator` (`GREATER_THAN_OR_EQUAL`, `LESS_THAN`, etc.) e `StartValue`/`EndValue`.<br>**Uso**: Essencial para focar apenas nas anomalias mais caras e ignorar ruídos de baixo valor. |
| `NextPageToken` | String | Não | Token para paginação, caso a lista de anomalias seja muito longa. |
| `MaxResults` | Integer | Não | Número máximo de resultados a serem retornados por página. |

### Exemplo de Requisição (Buscando anomalias com impacto > $100)

```json
{
  "DateInterval": {
    "StartDate": "2026-01-01",
    "EndDate": "2026-02-23"
  },
  "TotalImpact": {
    "NumericOperator": "GREATER_THAN_OR_EQUAL",
    "StartValue": 100
  },
  "MaxResults": 50
}
```

---

## 3. CreateAnomalySubscription

Cria uma assinatura para receber notificações (alertas) quando um monitor detecta uma anomalia. É a parte "proativa" do serviço.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AnomalySubscription` | Objeto | **Sim** | O objeto principal que define a assinatura. Contém as seguintes chaves:<br>- `SubscriptionName` (String, **Sim**): O nome da sua assinatura de alerta (ex: "Alertas-Time-FinOps").<br>- `MonitorArnList` (Array de Strings, **Sim**): Uma lista contendo os ARNs dos monitores que esta assinatura irá cobrir. Você pode ter uma única assinatura para múltiplos monitores.<br>- `Subscribers` (Array de Objetos, **Sim**): Uma lista de destinatários. Cada objeto `Subscriber` tem `Type` (`EMAIL` ou `SNS`) e `Address` (o endereço de e-mail ou o ARN do tópico SNS). Use `SNS` para integrações automatizadas (ex: postar no Slack, criar um ticket no Jira).<br>- `Frequency` (String, **Sim**): A frequência dos alertas. `IMMEDIATE` para ação rápida em anomalias críticas, `DAILY` para um resumo diário gerenciável, `WEEKLY` para um relatório de alto nível.<br>- `ThresholdExpression` (Objeto, Não): Uma expressão poderosa para definir um limiar de notificação e reduzir ruído. Por exemplo, só enviar alerta se o impacto absoluto for maior que $100. Se não definido, você será notificado sobre todas as anomalias. |

### Exemplo de Requisição (Alerta Imediato por E-mail para anomalias > $50)

```json
{
  "AnomalySubscription": {
    "SubscriptionName": "Alertas-Criticos-FinOps-Email",
    "MonitorArnList": [
      "arn:aws:ce::123456789012:anomalymonitor/MONITOR_ID_AQUI"
    ],
    "Subscribers": [
      {
        "Type": "EMAIL",
        "Address": "equipe.finops@exemplo.com"
      }
    ],
    "Frequency": "IMMEDIATE",
    "ThresholdExpression": {
      "Dimensions": {
        "Key": "ANOMALY_TOTAL_IMPACT_ABSOLUTE",
        "MatchOptions": ["GREATER_THAN_OR_EQUAL"],
        "Values": ["50"]
      }
    }
  }
}
```

---

## 4. ProvideAnomalyFeedback

Permite que você forneça feedback sobre uma anomalia detectada. Este feedback é **crucial**, pois ajuda o modelo de Machine Learning da AWS a se tornar mais inteligente e a reduzir falsos positivos no futuro, personalizando a detecção para o seu padrão de gastos.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AnomalyId` | String | **Sim** | O ID da anomalia para a qual você está fornecendo feedback. Este ID é obtido na resposta da chamada `GetAnomalies`. |
| `Feedback` | String | **Sim** | Sua avaliação da anomalia.<br>**Valores**: `YES` (Sim, isso foi um pico de custo inesperado e indesejado), `NO` (Não, isso não é uma anomalia relevante para mim), `PLANNED_ACTIVITY` (Sim, mas era uma atividade planejada, como um teste de carga ou uma migração de dados, e não deve ser considerada uma anomalia no futuro). |

### Exemplo de Requisição

```json
{
  "AnomalyId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "Feedback": "YES"
}
```
