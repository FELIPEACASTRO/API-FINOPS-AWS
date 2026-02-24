# Guia Devastadoramente Detalhado: AWS Cost Explorer API

## Visão Geral

A API do Cost Explorer é o coração do FinOps na AWS. Ela permite consultar, analisar, agrupar e filtrar seus dados de custo e uso com alta granularidade. Dominar esta API é essencial para entender para onde seu dinheiro está indo, identificar oportunidades de otimização e criar relatórios financeiros precisos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ce.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSInsightsIndexService` |
| **Service Name (IAM)** | `ce` |

---

## 1. GetCostAndUsage

Esta é a ação mais fundamental. Ela retorna dados de custo e uso agregados para um período de tempo, permitindo agrupar e filtrar por múltiplas dimensões.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Objeto | **Sim** | Define o intervalo de tempo para a consulta. É um objeto que contém duas chaves: `Start` e `End`.<br>**Formato**: `YYYY-MM-DD`.<br>**Regras**: O `Start` é inclusivo e o `End` é exclusivo. Por exemplo, para obter dados de Janeiro, use `Start: '2026-01-01'` e `End: '2026-02-01'`.<br>**Uso**: Essencial para qualquer análise de custo, permitindo focar em períodos específicos como o último mês, o último trimestre, etc. |
| `Granularity` | String | **Sim** | Define o nível de agregação dos dados no tempo.<br>**Valores Aceitos**: `DAILY` (diário), `MONTHLY` (mensal), `HOURLY` (horário - requer habilitação e tem custo adicional).<br>**Uso**: Use `MONTHLY` para relatórios de alto nível, `DAILY` para análises de tendências e detecção de picos, e `HOURLY` para investigações forenses de custos em recursos específicos. |
| `Metrics` | Array de Strings | **Sim** | Define quais métricas financeiras ou de uso você deseja obter.<br>**Valores Comuns**: `UnblendedCost` (custo bruto), `BlendedCost` (custo misturado para contas em uma Organization), `AmortizedCost` (custo amortizado de RIs/SPs), `UsageQuantity` (quantidade de uso).<br>**Uso**: `UnblendedCost` é o mais comum para análise direta. `AmortizedCost` é crucial para entender o custo real após a aplicação de descontos de compromissos. `UsageQuantity` é usado em conjunto com filtros para entender o consumo de um recurso específico (ex: horas de EC2, GB de S3). |
| `GroupBy` | Array de Objetos | Não | Agrupa os resultados por uma ou mais dimensões. Cada objeto no array tem `Type` e `Key`.<br>**Tipos**: `DIMENSION` (para dimensões padrão como `SERVICE` ou `LINKED_ACCOUNT`) ou `TAG` (para tags de alocação de custos).<br>**Chaves Comuns**: `SERVICE`, `LINKED_ACCOUNT`, `REGION`, `INSTANCE_TYPE`, `USAGE_TYPE`, ou o nome de uma tag (ex: `Project`).<br>**Uso**: Fundamental para quebrar os custos. `GroupBy: SERVICE` mostra o custo por serviço. `GroupBy: LINKED_ACCOUNT` mostra o custo por conta. Você pode combinar, por exemplo, agrupar por conta e depois por serviço. |
| `Filter` | Objeto | Não | Filtra os resultados para incluir apenas os dados que correspondem a critérios específicos. É um objeto complexo que pode conter `Dimensions`, `Tags`, `CostCategories`, ou operadores lógicos como `And`, `Or`, `Not`.<br>**Uso**: Essencial para focar sua análise. Por exemplo, filtrar por um serviço específico (`SERVICE: AmazonEC2`), uma conta (`LINKED_ACCOUNT: 123456789012`), ou uma tag (`TAG: Environment=Production`). |
| `NextPageToken` | String | Não | Token usado para paginação. Se a resposta anterior foi truncada, ela conterá um `NextPageToken`. Use este token no campo `NextPageToken` da próxima requisição para obter a página seguinte de resultados.<br>**Uso**: Indispensável ao lidar com grandes volumes de dados que não cabem em uma única resposta. |

### Exemplo de Requisição (Custo Mensal por Serviço)

```json
{
  "TimePeriod": {
    "Start": "2026-01-01",
    "End": "2026-02-01"
  },
  "Granularity": "MONTHLY",
  "Metrics": [
    "UnblendedCost"
  ],
  "GroupBy": [
    {
      "Type": "DIMENSION",
      "Key": "SERVICE"
    }
  ]
}
```

---

## 2. GetCostAndUsageWithResources

Similar ao `GetCostAndUsage`, mas permite agrupar os custos até o nível de recurso individual (ex: por ID de instância EC2). **Atenção**: esta ação exige que você filtre por **exatamente uma** dimensão (ex: um serviço, uma conta). 

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `TimePeriod`, `Granularity`, `Metrics`, `NextPageToken` | - | - | Idênticos ao `GetCostAndUsage`. |
| `Filter` | Objeto | **Sim** | **Regra Específica**: O filtro DEVE conter uma dimensão (`Dimensions` ou `CostCategories` ou `Tags`) e essa dimensão deve ter **apenas um valor**. Por exemplo, você deve filtrar por `SERVICE: "AmazonEC2"`, mas não por `SERVICE: ["AmazonEC2", "AmazonS3"]`.<br>**Uso**: É a única forma de obter custos por recurso. Use para encontrar qual instância EC2 ou banco de dados RDS específico está gerando mais custos. |
| `GroupBy` | Array de Objetos | **Sim** | Deve conter um objeto com `Type: "DIMENSION"` e `Key: "RESOURCE_ID"`. Você pode adicionar outras dimensões se desejar. |

### Exemplo de Requisição (Custo por Instância EC2)

```json
{
  "TimePeriod": {
    "Start": "2026-02-01",
    "End": "2026-02-02"
  },
  "Granularity": "DAILY",
  "Metrics": [
    "UnblendedCost"
  ],
  "Filter": {
    "Dimensions": {
      "Key": "SERVICE",
      "Values": [
        "Amazon Elastic Compute Cloud - Compute"
      ]
    }
  },
  "GroupBy": [
    {
      "Type": "DIMENSION",
      "Key": "RESOURCE_ID"
    }
  ]
}
```

---

## 3. GetCostForecast

Prevê qual será o seu custo total para um período futuro, com base no seu histórico de gastos.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `TimePeriod` | Objeto | **Sim** | Define o período para o qual a previsão será gerada. O `Start` deve ser no futuro (a partir do dia seguinte ao atual) e o `End` também no futuro.<br>**Exemplo**: Para prever o custo de Março, use `Start: '2026-03-01'` e `End: '2026-04-01'`. |
| `Metric` | String | **Sim** | A métrica de custo a ser prevista.<br>**Valores Aceitos**: `UNBLENDED_COST`, `BLENDED_COST`, `AMORTIZED_COST`, `NET_UNBLENDED_COST`, `NET_AMORTIZED_COST`.<br>**Uso**: `UNBLENDED_COST` é o mais comum para previsões diretas. |
| `Granularity` | String | **Sim** | `DAILY` ou `MONTHLY`. Define se a previsão será um total mensal ou uma série de valores diários. |
| `Filter` | Objeto | Não | Permite gerar uma previsão para um subconjunto dos seus custos (ex: prever o custo apenas do serviço EC2 ou de uma conta específica). |
| `PredictionIntervalLevel` | Integer | Não | Define o nível de confiança do intervalo de previsão (a faixa de valores prováveis).<br>**Valores Comuns**: `95` (padrão), `80`. Um valor de `95` significa que há 95% de chance de o custo real cair dentro do intervalo previsto. |

### Exemplo de Requisição (Previsão de Custo para Março)

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

## 4. GetReservationPurchaseRecommendation

Analisa seu histórico de uso de instâncias On-Demand e recomenda a compra de Reserved Instances (RIs) para economizar dinheiro.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Service` | String | **Sim** | O serviço para o qual você quer recomendações.<br>**Exemplo**: `"Amazon Elastic Compute Cloud - Compute"`, `"Amazon Relational Database Service"`. |
| `LookbackPeriodInDays` | String | Não | O período de histórico de uso a ser analisado.<br>**Valores Aceitos**: `SEVEN_DAYS`, `THIRTY_DAYS`, `SIXTY_DAYS` (padrão).<br>**Uso**: `SIXTY_DAYS` oferece uma base mais estável para recomendações, suavizando picos de uso de curto prazo. |
| `TermInYears` | String | Não | O termo do compromisso da RI.<br>**Valores Aceitos**: `ONE_YEAR` (padrão), `THREE_YEARS`. |
| `PaymentOption` | String | Não | A opção de pagamento da RI.<br>**Valores Aceitos**: `NO_UPFRONT` (padrão), `PARTIAL_UPFRONT`, `ALL_UPFRONT`. Pagamentos adiantados maiores resultam em descontos maiores. |
| `AccountId` | String | Não | Se você for a conta de gerenciamento, pode solicitar recomendações para uma conta membro específica. |
| `ServiceSpecification` | Objeto | Não | Para serviços como EC2, permite especificar atributos como `OfferingClass` (`STANDARD` ou `CONVERTIBLE`). |

### Exemplo de Requisição (Recomendação de RIs EC2 para 1 Ano, Sem Pagamento Adiantado)

```json
{
  "Service": "Amazon Elastic Compute Cloud - Compute",
  "LookbackPeriodInDays": "SIXTY_DAYS",
  "TermInYears": "ONE_YEAR",
  "PaymentOption": "NO_UPFRONT"
}
```

---

## 5. GetSavingsPlansPurchaseRecommendation

Similar à recomendação de RIs, mas para Savings Plans, que são mais flexíveis.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `SavingsPlansType` | String | **Sim** | O tipo de Savings Plan.<br>**Valores Aceitos**: `COMPUTE_SP` (cobre EC2, Fargate, Lambda), `EC2_INSTANCE_SP` (cobre uma família de instâncias EC2 específica em uma região), `SAGEMAKER_SP`. |
| `TermInYears` | String | **Sim** | `ONE_YEAR` ou `THREE_YEARS`. |
| `PaymentOption` | String | **Sim** | `NO_UPFRONT`, `PARTIAL_UPFRONT`, `ALL_UPFRONT`. |
| `LookbackPeriodInDays` | String | **Sim** | `SEVEN_DAYS`, `THIRTY_DAYS`, `SIXTY_DAYS`. |
| `AccountScope` | String | Não | `PAYER` (padrão, para a conta de gerenciamento) ou `LINKED` (para uma conta membro). |
| `Filter` | Objeto | Não | Filtra o uso a ser considerado para a recomendação. |

### Exemplo de Requisição (Recomendação de Compute SP para 1 Ano, Sem Pagamento Adiantado)

```json
{
  "SavingsPlansType": "COMPUTE_SP",
  "TermInYears": "ONE_YEAR",
  "PaymentOption": "NO_UPFRONT",
  "LookbackPeriodInDays": "SIXTY_DAYS"
}
```

---

## 6. GetRightsizingRecommendation

Analisa o uso de CPU e memória de instâncias EC2 e recomenda mudar para um tipo de instância menor (right-sizing) se estiverem subutilizadas.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Service` | String | **Sim** | O único valor aceito atualmente é `"AmazonEC2"`. |
| `Configuration` | Objeto | Não | Permite customizar a recomendação.<br>**Chaves**: `RecommendationTarget` (`SAME_INSTANCE_FAMILY` ou `CROSS_INSTANCE_FAMILY`), `BenefitsConsidered` (`true` para incluir economia estimada). |
| `NextPageToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{
  "Service": "AmazonEC2",
  "Configuration": {
    "RecommendationTarget": "CROSS_INSTANCE_FAMILY",
    "BenefitsConsidered": true
  }
}
```
