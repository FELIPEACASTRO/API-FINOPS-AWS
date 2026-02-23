# Guia Detalhado: AWS Cost Optimization Hub API

## Visão Geral

O Cost Optimization Hub centraliza recomendações de otimização de custos de múltiplos serviços AWS em um único lugar, permitindo identificar, filtrar, agregar e quantificar economias potenciais.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cost-optimization-hub.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `CostOptimizationHubService` |
| **Service Name (IAM)** | `cost-optimization-hub` |

---

## Ações da API

### 1. ListRecommendations

Lista todas as recomendações de otimização de custos disponíveis.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `filter` | Object | Não | Filtros por `accountIds`, `regions`, `resourceTypes`, `actionTypes`, `implementationEfforts`, `restartNeeded`, `rollbackPossible`, `tags`. |
| `orderBy` | Object | Não | Ordenação com `dimension` e `order` (`Asc`/`Desc`). |
| `maxResults` | Integer | Não | Número máximo de resultados (1-1000). |
| `nextToken` | String | Não | Token para paginação. |
| `includeAllRecommendations` | Boolean | Não | Se `true`, inclui recomendações já implementadas. |

#### Exemplo de Requisição

```json
{
  "filter": {
    "actionTypes": ["Rightsize", "Terminate"],
    "resourceTypes": ["Ec2Instance"]
  },
  "orderBy": {
    "dimension": "SavingsAmount",
    "order": "Desc"
  },
  "maxResults": 50
}
```

---

### 2. ListRecommendationSummaries

Lista resumos de recomendações agrupados por uma dimensão específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `groupBy` | String | Sim | Dimensão de agrupamento: `Region`, `ResourceType`, `AccountId`, `ImplementationEffort`, `ActionType`, `CurrencyCode`, `Tag`. |
| `filter` | Object | Não | Mesmos filtros de `ListRecommendations`. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |
| `metrics` | Array | Não | Métricas adicionais a incluir. |

#### Exemplo de Requisição

```json
{
  "groupBy": "ResourceType",
  "maxResults": 100
}
```

---

### 3. GetRecommendation

Retorna detalhes completos de uma recomendação específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `recommendationId` | String | Sim | O ID da recomendação. |

#### Exemplo de Requisição

```json
{
  "recommendationId": "rec-abc123def456"
}
```

---

### 4. GetPreferences

Retorna as preferências configuradas no Cost Optimization Hub.

#### Parâmetros de Entrada

Nenhum parâmetro obrigatório.

#### Exemplo de Requisição

```json
{}
```

---

### 5. UpdatePreferences

Atualiza as preferências do Cost Optimization Hub.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `memberAccountDiscountVisibility` | String | Não | `All` ou `None`. Controla se contas membro podem ver descontos. |
| `savingsEstimationMode` | String | Não | `BeforeDiscounts` ou `AfterDiscounts`. |

#### Exemplo de Requisição

```json
{
  "savingsEstimationMode": "AfterDiscounts"
}
```
