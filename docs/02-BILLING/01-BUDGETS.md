# Guia Detalhado: AWS Budgets API

## Visão Geral

O AWS Budgets permite criar orçamentos para monitorar custos e uso, com notificações automáticas e ações quando limites são atingidos. É uma ferramenta essencial para o controle proativo de custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://budgets.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSBudgetServiceGateway` |
| **Service Name (IAM)** | `budgets` |

---

## Ações da API

### 1. CreateBudget

Cria um novo orçamento com limites, filtros e notificações.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta onde o orçamento será criado. |
| `Budget` | Object | Sim | Objeto complexo que define o orçamento. |
| `NotificationsWithSubscribers` | Array | Não | Lista de notificações e seus assinantes. |

#### Objeto `Budget`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BudgetName` | String | Sim | Nome único para o orçamento. |
| `BudgetType` | String | Sim | `COST`, `USAGE`, `RI_UTILIZATION`, `RI_COVERAGE`, `SAVINGS_PLANS_UTILIZATION`, `SAVINGS_PLANS_COVERAGE`. |
| `TimeUnit` | String | Sim | `DAILY`, `MONTHLY`, `QUARTERLY`, `ANNUALLY`. |
| `BudgetLimit` | Object | Sim | Objeto com `Amount` e `Unit`. |
| `CostFilters` | Object | Não | Filtros por dimensão (serviço, conta, tag, etc.). |
| `CostTypes` | Object | Não | Incluir/excluir impostos, créditos, reembolsos, etc. |
| `TimePeriod` | Object | Não | Período de início e fim para orçamentos fixos. |

#### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "Budget": {
    "BudgetName": "Monthly-EC2-Budget",
    "BudgetType": "COST",
    "TimeUnit": "MONTHLY",
    "BudgetLimit": {
      "Amount": "1000.0",
      "Unit": "USD"
    },
    "CostFilters": {
      "Service": ["Amazon Elastic Compute Cloud - Compute"]
    },
    "CostTypes": {
      "IncludeTax": true,
      "IncludeSubscription": true,
      "UseBlended": false
    }
  },
  "NotificationsWithSubscribers": [
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 80,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [
        {
          "SubscriptionType": "EMAIL",
          "Address": "finops@example.com"
        },
        {
          "SubscriptionType": "SNS",
          "Address": "arn:aws:sns:us-east-1:123456789012:MyTopic"
        }
      ]
    }
  ]
}
```

---

### 2. DescribeBudgets

Lista os orçamentos de uma conta.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta. |
| `MaxResults` | Integer | Não | Número máximo de resultados por página. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "MaxResults": 100
}
```

---

### 3. DescribeBudgetPerformanceHistory

Retorna o histórico de performance de um orçamento, comparando o custo/uso real com o limite orçado.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta. |
| `BudgetName` | String | Sim | O nome do orçamento. |
| `TimePeriod` | Object | Não | Período de início e fim para o histórico. Se não especificado, retorna os últimos 12 meses. |
| `MaxResults` | Integer | Não | Número máximo de resultados por página. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "BudgetName": "Monthly-EC2-Budget"
}
```

---

### 4. CreateBudgetAction

Cria uma ação automática a ser executada quando um orçamento atinge um determinado limite. A ação pode ser aplicar uma política IAM, desanexar uma política ou parar instâncias EC2/RDS.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta. |
| `BudgetName` | String | Sim | O nome do orçamento ao qual a ação está vinculada. |
| `NotificationType` | String | Sim | `ACTUAL` ou `FORECASTED`. |
| `ActionType` | String | Sim | `APPLY_IAM_POLICY`, `APPLY_SCP_POLICY`, `RUN_SSM_DOCUMENTS`. |
| `ActionThreshold` | Object | Sim | Objeto com `Value` e `Type` (`PERCENTAGE` ou `ABSOLUTE_VALUE`). |
| `Definition` | Object | Sim | Define a ação (ex: qual política aplicar, quais instâncias parar). |
| `ExecutionRoleArn` | String | Sim | ARN do perfil que o Budgets usará para executar a ação. |
| `ApprovalModel` | String | Sim | `AUTOMATIC` ou `MANUAL`. |
| `Subscribers` | Array | Sim | Lista de contatos a serem notificados sobre a execução da ação. |

#### Exemplo de Requisição (Parar Instâncias EC2)

```json
{
  "AccountId": "123456789012",
  "BudgetName": "Daily-Dev-Budget",
  "NotificationType": "ACTUAL",
  "ActionType": "RUN_SSM_DOCUMENTS",
  "ActionThreshold": {
    "Value": 100,
    "Type": "PERCENTAGE"
  },
  "Definition": {
    "SsmActionDefinition": {
      "ActionSubType": "STOP_EC2_INSTANCES",
      "Region": "us-east-1",
      "InstanceIds": [
        "i-0123456789abcdef0"
      ]
    }
  },
  "ExecutionRoleArn": "arn:aws:iam::123456789012:role/MyBudgetActionRole",
  "ApprovalModel": "AUTOMATIC",
  "Subscribers": [
    {
      "SubscriptionType": "EMAIL",
      "Address": "devops@example.com"
    }
  ]
}
```

---

### 5. DescribeBudgetActionsForBudget

Lista as ações configuradas para um orçamento específico.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta. |
| `BudgetName` | String | Sim | O nome do orçamento. |
| `MaxResults` | Integer | Não | Número máximo de resultados por página. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "BudgetName": "Daily-Dev-Budget"
}
```
