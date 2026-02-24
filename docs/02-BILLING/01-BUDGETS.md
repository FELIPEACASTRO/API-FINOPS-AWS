# Guia Devastadoramente Detalhado: AWS Budgets API

## Visão Geral

A API do AWS Budgets é a principal ferramenta para controle de custos e governança financeira. Ela permite que você defina orçamentos para seus custos e uso, e crie alertas que notificam quando seus gastos (ou a previsão de gastos) excedem um limiar definido. É uma ferramenta essencial para evitar surpresas na fatura e manter a responsabilidade financeira entre as equipes.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://budgets.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSBudgetServiceGateway` |
| **Service Name (IAM)** | `budgets` |

---

## 1. CreateBudget

Cria um novo orçamento para monitorar custos ou uso.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | **Sim** | O ID da conta de 12 dígitos à qual o orçamento se aplica. |
| `Budget` | Objeto | **Sim** | O objeto principal que define o orçamento. Veja a tabela detalhada abaixo. |
| `NotificationsWithSubscribers` | Array de Objetos | Não | Uma lista de alertas a serem configurados para este orçamento. Essencial para a proatividade. Veja a tabela detalhada abaixo. |

#### O Objeto `Budget`

| Chave | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `BudgetName` | String | **Sim** | O nome do seu orçamento. Deve ser único na sua conta. Ex: "Orçamento-Mensal-Total", "Orçamento-Projeto-X-Dev". |
| `BudgetType` | String | **Sim** | O tipo de orçamento. <br>**Valores**: `COST` (para custos em USD), `USAGE` (para quantidade de uso, ex: horas de EC2), `RI_UTILIZATION` (para utilização de RIs), `RI_COVERAGE` (para cobertura de RIs), `SAVINGS_PLANS_UTILIZATION`, `SAVINGS_PLANS_COVERAGE`.<br>**Uso**: `COST` é o mais comum. Os outros são para casos de uso avançados de otimização. |
| `TimeUnit` | String | **Sim** | A periodicidade do orçamento.<br>**Valores**: `DAILY`, `MONTHLY`, `QUARTERLY`, `ANNUALLY`.<br>**Uso**: `MONTHLY` é o mais comum para orçamentos de custo. |
| `BudgetLimit` | Objeto | **Sim** | O valor do orçamento. Contém `Amount` (String) e `Unit` (String, ex: "USD"). |
| `CostFilters` | Objeto | Não | Permite que o orçamento se aplique a um subconjunto dos seus custos. A estrutura é `{"CHAVE": ["valor1", "valor2"]}`. <br>**Chaves Comuns**: `Service`, `LinkedAccount`, `TagKeyValue` (formato: `"TagKey$TagValue"`), `Region`.<br>**Uso**: Fundamental para criar orçamentos para projetos, times ou contas específicas. |
| `CostTypes` | Objeto | Não | Define quais tipos de custo incluir no cálculo do orçamento. <br>**Chaves (Boolean)**: `IncludeTax`, `IncludeSubscription`, `UseBlended`, `IncludeRefund`, `IncludeCredit`, `UseAmortized`.<br>**Uso**: Por padrão, a maioria é `true`. `UseAmortized: true` é importante para orçamentos que precisam refletir o custo real após a distribuição de RIs/SPs. |

#### O Objeto `NotificationsWithSubscribers`

| Chave | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Notification` | Objeto | **Sim** | Define a regra do alerta. Contém:<br>- `NotificationType` (String, **Sim**): `ACTUAL` (baseado no custo real acumulado) ou `FORECASTED` (baseado na previsão de custo para o final do período).<br>- `ComparisonOperator` (String, **Sim**): `GREATER_THAN`, `LESS_THAN`, `EQUAL_TO`.<br>- `Threshold` (Double, **Sim**): O valor do limiar para disparar o alerta.<br>- `ThresholdType` (String, Não): `PERCENTAGE` (padrão) ou `ABSOLUTE_VALUE`. |
| `Subscribers` | Array de Objetos | **Sim** | Lista de destinatários. Cada objeto tem `SubscriptionType` (`EMAIL` ou `SNS`) e `Address` (o e-mail ou ARN do tópico SNS). |

### Exemplo de Requisição (Orçamento de Custo Mensal com Alerta de Previsão)

```json
{
  "AccountId": "123456789012",
  "Budget": {
    "BudgetName": "Orcamento-Mensal-Total-5000",
    "BudgetType": "COST",
    "TimeUnit": "MONTHLY",
    "BudgetLimit": {
      "Amount": "5000.0",
      "Unit": "USD"
    }
  },
  "NotificationsWithSubscribers": [
    {
      "Notification": {
        "NotificationType": "FORECASTED",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 100,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [
        {
          "SubscriptionType": "EMAIL",
          "Address": "lider.equipe@exemplo.com"
        },
        {
          "SubscriptionType": "SNS",
          "Address": "arn:aws:sns:us-east-1:123456789012:AlarmesFinOps"
        }
      ]
    }
  ]
}
```

---

## 2. DescribeBudgets

Lista os orçamentos que foram criados na conta.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | **Sim** | O ID da conta de 12 dígitos. |
| `MaxResults` | Integer | Não | O número máximo de orçamentos a serem retornados por página. |
| `NextToken` | String | Não | Token para paginação, obtido de uma resposta anterior. |

### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "MaxResults": 100
}
```

---

## 3. DescribeBudgetPerformanceHistory

Retorna o histórico de performance de um orçamento, mostrando o custo real e o valor orçado para períodos passados. Útil para análises de variação (orçado vs. realizado).

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | **Sim** | O ID da conta de 12 dígitos. |
| `BudgetName` | String | **Sim** | O nome exato do orçamento que você quer analisar. |
| `TimePeriod` | Objeto | Não | Permite especificar um período (`Start` e `End`) para o histórico. Se não fornecido, retorna os últimos 5 períodos do orçamento. |
| `MaxResults` | Integer | Não | Número máximo de períodos históricos a retornar. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{
  "AccountId": "123456789012",
  "BudgetName": "Orcamento-Mensal-Total-5000"
}
```

---

## 4. CreateBudgetAction

Cria uma ação automatizada a ser executada quando um orçamento atinge um determinado limiar. Isso transforma o Budgets de uma ferramenta de monitoramento para uma ferramenta de controle ativo.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | **Sim** | ID da conta. |
| `BudgetName` | String | **Sim** | Nome do orçamento ao qual a ação está associada. |
| `NotificationType` | String | **Sim** | `ACTUAL` ou `FORECASTED`. O gatilho da ação. |
| `ActionType` | String | **Sim** | O tipo de ação a ser executada. <br>**Valores**: `APPLY_IAM_POLICY`, `APPLY_SCP_POLICY`, `RUN_SSM_DOCUMENTS`.<br>**Uso**: `APPLY_IAM_POLICY` é comum para restringir permissões de criação de recursos (ex: uma política de "somente leitura") quando o orçamento estoura. `RUN_SSM_DOCUMENTS` pode ser usado para parar instâncias EC2/RDS. |
| `ActionThreshold` | Objeto | **Sim** | O limiar que dispara a ação. Contém `Value` (Double) e `Type` (`PERCENTAGE` ou `ABSOLUTE_VALUE`). |
| `Definition` | Objeto | **Sim** | Define os detalhes da ação. Contém `IamActionDefinition` (com `PolicyArn` e `Roles`/`Users`/`Groups`), `ScpActionDefinition` (com `PolicyId` e `TargetIds`), ou `SsmActionDefinition` (com `ActionSubType`, `Region` e `InstanceIds`). |
| `ExecutionRoleArn` | String | **Sim** | O ARN da role IAM que o serviço Budgets usará para executar a ação. Esta role precisa ter permissão para realizar a ação definida. |
| `ApprovalModel` | String | **Sim** | `AUTOMATIC` ou `MANUAL`. Define se a ação requer aprovação manual antes de ser executada. |
| `Subscribers` | Array de Objetos | **Sim** | Lista de contatos para notificar sobre a execução da ação. |

### Exemplo de Requisição (Aplicar política de "ReadOnly" automaticamente)

```json
{
  "AccountId": "123456789012",
  "BudgetName": "Orcamento-Mensal-Total-5000",
  "NotificationType": "ACTUAL",
  "ActionType": "APPLY_IAM_POLICY",
  "ActionThreshold": {
    "Value": 110,
    "Type": "PERCENTAGE"
  },
  "Definition": {
    "IamActionDefinition": {
      "PolicyArn": "arn:aws:iam::123456789012:policy/FinOps-ReadOnly-Policy",
      "Groups": ["Developers"]
    }
  },
  "ExecutionRoleArn": "arn:aws:iam::123456789012:role/BudgetActionRole",
  "ApprovalModel": "AUTOMATIC",
  "Subscribers": [
    {
      "SubscriptionType": "EMAIL",
      "Address": "admin.finops@exemplo.com"
    }
  ]
}
```
