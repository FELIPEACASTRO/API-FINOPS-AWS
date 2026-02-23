# Guia Detalhado: AWS Cost Explorer API

## Visão Geral

O **AWS Cost Explorer** é o serviço central para análise de custos e uso na AWS. Ele permite consultar dados agregados de custo e uso, gerar previsões, obter recomendações de compra de Reserved Instances e Savings Plans, e identificar oportunidades de right-sizing. É a API mais importante para qualquer prática de FinOps.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ce.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSInsightsIndexService` |
| **Service Name (IAM)** | `ce` |
| **Região** | `us-east-1` (global) |

---

## Ações da API

### 1. GetCostAndUsage

Esta é a ação mais utilizada do Cost Explorer. Ela retorna dados agregados de custo e uso para o período especificado, com suporte a filtros, agrupamentos e múltiplas métricas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período de consulta com `Start` e `End` no formato `YYYY-MM-DD`. |
| `Granularity` | String | Sim | Granularidade: `DAILY`, `MONTHLY` ou `HOURLY`. |
| `Metrics` | Array | Sim | Métricas a retornar: `UnblendedCost`, `BlendedCost`, `AmortizedCost`, `NetAmortizedCost`, `NetUnblendedCost`, `UsageQuantity`, `NormalizedUsageAmount`. |
| `GroupBy` | Array | Não | Agrupamento por dimensão (`SERVICE`, `REGION`, `LINKED_ACCOUNT`, `USAGE_TYPE`, `INSTANCE_TYPE`, `RECORD_TYPE`) ou por `TAG` (ex: `{"Type": "TAG", "Key": "Project"}`). Máximo de 2 grupos. |
| `Filter` | Object | Não | Filtro por dimensão, tag ou expressão de custo. |
| `NextPageToken` | String | Não | Token para paginação de resultados. |

#### Exemplo de Requisição

```json
{
  "TimePeriod": {
    "Start": "2026-01-01",
    "End": "2026-02-01"
  },
  "Granularity": "MONTHLY",
  "Metrics": ["UnblendedCost", "UsageQuantity"],
  "GroupBy": [
    {"Type": "DIMENSION", "Key": "SERVICE"}
  ],
  "Filter": {
    "Not": {
      "Dimensions": {
        "Key": "SERVICE",
        "Values": ["AWS Support (Business)"]
      }
    }
  }
}
```

---

### 2. GetCostForecast

Retorna uma previsão de custos para um período futuro, baseada nos padrões históricos de gastos.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período futuro para a previsão. |
| `Metric` | String | Sim | Métrica para previsão: `UNBLENDED_COST`, `BLENDED_COST`, `AMORTIZED_COST`, `NET_AMORTIZED_COST`, `NET_UNBLENDED_COST`. |
| `Granularity` | String | Sim | `DAILY` ou `MONTHLY`. |
| `PredictionIntervalLevel` | Integer | Não | Nível de confiança da previsão (51-99). Padrão: 80. |
| `Filter` | Object | Não | Filtro para previsão de um subconjunto de custos. |

#### Exemplo de Requisição

```json
{
  "TimePeriod": {
    "Start": "2026-03-01",
    "End": "2026-04-01"
  },
  "Metric": "UNBLENDED_COST",
  "Granularity": "MONTHLY",
  "PredictionIntervalLevel": 95
}
```

---

### 3. GetRightsizingRecommendation

Retorna recomendações de right-sizing para instâncias EC2, identificando instâncias ociosas ou subutilizadas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Service` | String | Sim | O serviço para o qual obter recomendações (atualmente, apenas `AmazonEC2`). |
| `Configuration` | Object | Não | Configurações como `RecommendationTarget` (`SAME_INSTANCE_FAMILY` ou `CROSS_INSTANCE_FAMILY`) e `BenefitsConsidered` (se deve considerar benefícios de RI/SP). |
| `NextPageToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "Service": "AmazonEC2",
  "Configuration": {
    "RecommendationTarget": "CROSS_INSTANCE_FAMILY",
    "BenefitsConsidered": true
  }
}
```

---

### 4. GetReservationPurchaseRecommendation

Retorna recomendações de compra de Reserved Instances baseadas no uso histórico.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Service` | String | Sim | Serviço para o qual obter recomendações (ex: `Amazon Elastic Compute Cloud - Compute`). |
| `AccountId` | String | Não | ID da conta para análise. Se não especificado, usa a conta pagadora. |
| `LookbackPeriodInDays` | String | Não | Período de análise: `SEVEN_DAYS`, `THIRTY_DAYS`, `SIXTY_DAYS`. Padrão: `SIXTY_DAYS`. |
| `TermInYears` | String | Não | Duração do compromisso: `ONE_YEAR`, `THREE_YEARS`. |
| `PaymentOption` | String | Não | Opção de pagamento: `NO_UPFRONT`, `PARTIAL_UPFRONT`, `ALL_UPFRONT`. |
| `OfferingClass` | String | Não | Classe da oferta: `standard` ou `convertible`. |

#### Exemplo de Requisição

```json
{
  "Service": "Amazon Elastic Compute Cloud - Compute",
  "LookbackPeriodInDays": "SIXTY_DAYS",
  "TermInYears": "ONE_YEAR",
  "PaymentOption": "NO_UPFRONT",
  "OfferingClass": "standard"
}
```

---

### 5. GetSavingsPlansPurchaseRecommendation

Retorna recomendações de compra de Savings Plans baseadas no uso histórico.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `SavingsPlansType` | String | Sim | Tipo de SP: `COMPUTE_SP`, `EC2_INSTANCE_SP`, `SAGEMAKER_SP`. |
| `TermInYears` | String | Sim | Duração do compromisso: `ONE_YEAR`, `THREE_YEARS`. |
| `PaymentOption` | String | Sim | Opção de pagamento: `NO_UPFRONT`, `PARTIAL_UPFRONT`, `ALL_UPFRONT`. |
| `LookbackPeriodInDays` | String | Sim | Período de análise: `SEVEN_DAYS`, `THIRTY_DAYS`, `SIXTY_DAYS`. |
| `AccountScope` | String | Não | `PAYER` (padrão) ou `LINKED`. |
| `Filter` | Object | Não | Filtro por região, instância, etc. |

#### Exemplo de Requisição

```json
{
  "SavingsPlansType": "COMPUTE_SP",
  "TermInYears": "ONE_YEAR",
  "PaymentOption": "NO_UPFRONT",
  "LookbackPeriodInDays": "SIXTY_DAYS",
  "AccountScope": "PAYER"
}
```

---

### 6. GetDimensionValues

Retorna os valores disponíveis para uma dimensão específica. Útil para construir filtros dinâmicos em dashboards.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período de consulta. |
| `Dimension` | String | Sim | A dimensão a ser consultada (ex: `SERVICE`, `REGION`, `LINKED_ACCOUNT`). |
| `Context` | String | Não | `COST_AND_USAGE` (padrão) ou `RESERVATIONS`. |
| `SearchString` | String | Não | Filtra os valores retornados. |
| `NextPageToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "TimePeriod": {
    "Start": "2026-01-01",
    "End": "2026-02-01"
  },
  "Dimension": "SERVICE",
  "SearchString": "Amazon"
}
```

---

### 7. GetTags

Retorna todas as chaves e valores de tags de alocação de custos disponíveis para o período especificado.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período de consulta. |
| `TagKey` | String | Não | Filtra para obter valores de uma chave de tag específica. |
| `SearchString` | String | Não | Filtra os valores retornados. |
| `NextPageToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "TimePeriod": {
    "Start": "2026-01-01",
    "End": "2026-02-01"
  },
  "TagKey": "Project"
}
```
