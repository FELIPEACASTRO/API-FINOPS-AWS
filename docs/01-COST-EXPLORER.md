# 1. AWS Cost Explorer API

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

## Ações Disponíveis

### Consultas de Custo e Uso

#### GetCostAndUsage

Esta é a ação mais utilizada do Cost Explorer. Ela retorna dados agregados de custo e uso para o período especificado, com suporte a filtros, agrupamentos e múltiplas métricas.

**Parâmetros Principais:**

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período de consulta com `Start` e `End` no formato `YYYY-MM-DD`. |
| `Granularity` | String | Sim | Granularidade: `DAILY`, `MONTHLY` ou `HOURLY`. |
| `Metrics` | Array | Sim | Métricas a retornar: `UnblendedCost`, `BlendedCost`, `AmortizedCost`, `NetAmortizedCost`, `NetUnblendedCost`, `UsageQuantity`, `NormalizedUsageAmount`. |
| `GroupBy` | Array | Não | Agrupamento por dimensão (`SERVICE`, `REGION`, `LINKED_ACCOUNT`, `USAGE_TYPE`, `INSTANCE_TYPE`, `RECORD_TYPE`) ou por `TAG` (ex: `{"Type": "TAG", "Key": "Project"}`). Máximo de 2 grupos. |
| `Filter` | Object | Não | Filtro por dimensão, tag ou expressão de custo. |
| `NextPageToken` | String | Não | Token para paginação de resultados. |

**Métricas Disponíveis e Quando Usar:**

| Métrica | Descrição | Quando Usar |
| :--- | :--- | :--- |
| `UnblendedCost` | Custo real sem distribuição de descontos. | Análise de custo por recurso individual. |
| `BlendedCost` | Custo com descontos de RI/SP distribuídos proporcionalmente. | Visão de custo em contas consolidadas. |
| `AmortizedCost` | Custo com pagamentos antecipados de RI/SP distribuídos ao longo do tempo. | Análise financeira e orçamentária. |
| `NetAmortizedCost` | Amortizado menos descontos negociados. | Custo líquido real para a organização. |
| `UsageQuantity` | Quantidade de uso (horas, GB, requisições, etc.). | Análise de consumo de recursos. |

**Exemplo de Requisição - Custo Mensal por Serviço:**

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
  ]
}
```

#### GetCostAndUsageWithResources

Similar ao `GetCostAndUsage`, mas retorna dados no nível de recurso individual (Resource ID). Requer obrigatoriamente um filtro para limitar o escopo da consulta, pois os dados no nível de recurso são muito volumosos.

**Diferença chave**: Enquanto `GetCostAndUsage` retorna dados agregados, esta ação retorna o custo de cada recurso individual (ex: cada instância EC2, cada bucket S3, etc.).

#### GetCostForecast

Retorna uma previsão de custos para um período futuro, baseada nos padrões históricos de gastos. Utiliza modelos de machine learning internos da AWS.

**Parâmetros Principais:**

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Object | Sim | Período futuro para a previsão. |
| `Metric` | String | Sim | Métrica para previsão: `UNBLENDED_COST`, `BLENDED_COST`, `AMORTIZED_COST`, `NET_AMORTIZED_COST`, `NET_UNBLENDED_COST`. |
| `Granularity` | String | Sim | `DAILY` ou `MONTHLY`. |
| `PredictionIntervalLevel` | Integer | Não | Nível de confiança da previsão (51-99). Padrão: 80. |
| `Filter` | Object | Não | Filtro para previsão de um subconjunto de custos. |

#### GetUsageForecast

Similar ao `GetCostForecast`, mas para previsão de uso (quantidade) em vez de custo. Útil para planejamento de capacidade.

### Dimensões e Tags

#### GetDimensionValues

Retorna os valores disponíveis para uma dimensão específica. Útil para descobrir quais serviços, regiões, contas ou tipos de uso existem nos dados de custo.

**Dimensões Disponíveis**: `SERVICE`, `LINKED_ACCOUNT`, `TAG`, `REGION`, `INSTANCE_TYPE`, `USAGE_TYPE`, `USAGE_TYPE_GROUP`, `RECORD_TYPE`, `OPERATING_SYSTEM`, `TENANCY`, `SCOPE`, `PLATFORM`, `SUBSCRIPTION_ID`, `LEGAL_ENTITY_NAME`, `INVOICING_ENTITY`, `DEPLOYMENT_OPTION`, `DATABASE_ENGINE`, `CACHE_ENGINE`, `INSTANCE_TYPE_FAMILY`, `BILLING_ENTITY`, `RESERVATION_ID`, `SAVINGS_PLANS_TYPE`, `SAVINGS_PLAN_ARN`, `PAYMENT_OPTION`, `AGREEMENT_END_DATE_TIME_AFTER`, `AGREEMENT_END_DATE_TIME_BEFORE`.

#### GetTags

Retorna todas as chaves e valores de tags de alocação de custos disponíveis para o período especificado.

### Reserved Instances

#### GetReservationUtilization

Retorna dados de utilização de Reserved Instances, mostrando quanto das RIs compradas está sendo efetivamente utilizado.

#### GetReservationCoverage

Retorna dados de cobertura de Reserved Instances, mostrando quanto do uso total está coberto por RIs.

#### GetReservationPurchaseRecommendation

Retorna recomendações de compra de Reserved Instances baseadas no uso histórico, incluindo estimativas de economia.

### Savings Plans

#### GetSavingsPlansCoverage

Retorna dados de cobertura de Savings Plans, mostrando quanto do uso elegível está coberto por SPs.

#### GetSavingsPlansUtilizationDetails

Retorna detalhes de utilização de cada Savings Plan individual.

#### GetSavingsPlansPurchaseRecommendation

Retorna recomendações de compra de Savings Plans baseadas no uso histórico.

### Right-Sizing

#### GetRightsizingRecommendation

Retorna recomendações de right-sizing para instâncias EC2, identificando instâncias ociosas ou subutilizadas e sugerindo tipos de instância mais adequados.

### Cost Categories

#### CreateCostCategoryDefinition / UpdateCostCategoryDefinition / DeleteCostCategoryDefinition

Gerencia definições de categorias de custo, que permitem mapear custos da AWS para estruturas de negócio internas (times, projetos, centros de custo, etc.).

#### ListCostCategoryDefinitions

Lista todas as definições de categorias de custo existentes.

#### GetCostCategories

Retorna os nomes e valores das categorias de custo para o período especificado.

### Cost Anomaly Detection

#### CreateAnomalyMonitor

Cria um monitor de anomalias que rastreia custos por dimensão (serviço, conta, tag, etc.) usando machine learning para detectar padrões anômalos.

#### CreateAnomalySubscription

Cria uma assinatura para receber notificações quando anomalias são detectadas, com configuração de limiar e frequência.

#### GetAnomalies

Retorna as anomalias de custo detectadas no período especificado.

#### GetAnomalyMonitors / GetAnomalySubscriptions

Lista os monitores e assinaturas de anomalia configurados.

#### ProvideAnomalyFeedback

Permite enviar feedback sobre uma anomalia detectada (confirmando se é real ou falso positivo), melhorando a precisão do modelo de ML.

## Referências

- [AWS Cost Explorer API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations.html)
- [Using the Cost Explorer API](https://docs.aws.amazon.com/cost-management/latest/userguide/ce-api.html)
