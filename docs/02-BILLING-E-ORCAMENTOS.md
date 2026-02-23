# 2. APIs de Billing e Orçamentos

## 2.1 AWS Billing API

A API de Billing permite gerenciar visualizações de billing, que são representações dos dados de faturamento da conta.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://billing.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **X-Amz-Target Prefix** | `AWSBilling` |
| **Service Name (IAM)** | `billing` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `ListBillingViews` | Lista as visualizações de billing disponíveis para o período. |
| `GetBillingView` | Retorna detalhes de uma visualização de billing específica. |
| `CreateBillingView` | Cria uma nova visualização de billing personalizada. |
| `UpdateBillingView` | Atualiza uma visualização de billing existente. |
| `DeleteBillingView` | Remove uma visualização de billing. |

---

## 2.2 AWS Budgets API

O AWS Budgets permite criar orçamentos para monitorar custos e uso, com notificações automáticas e ações quando limites são atingidos. É uma ferramenta essencial para o controle proativo de custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://budgets.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSBudgetServiceGateway` |
| **Service Name (IAM)** | `budgets` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `CreateBudget` | Cria um novo orçamento com limites, filtros e notificações. |
| `DescribeBudgets` | Lista todos os orçamentos da conta. |
| `DescribeBudget` | Retorna detalhes de um orçamento específico. |
| `UpdateBudget` | Atualiza um orçamento existente. |
| `DeleteBudget` | Remove um orçamento. |
| `CreateBudgetAction` | Cria uma ação automática vinculada a um orçamento (ex: parar instâncias). |
| `DescribeBudgetAction` | Retorna detalhes de uma ação de orçamento. |
| `DescribeBudgetActionsForAccount` | Lista todas as ações de orçamento da conta. |
| `DescribeBudgetActionsForBudget` | Lista as ações de um orçamento específico. |
| `DescribeBudgetPerformanceHistory` | Retorna o histórico de performance de um orçamento ao longo do tempo. |
| `DescribeNotificationsForBudget` | Lista as notificações configuradas para um orçamento. |
| `DescribeSubscribersForNotification` | Lista os assinantes de uma notificação. |

### Tipos de Orçamento

| Tipo | Descrição |
| :--- | :--- |
| `COST` | Monitora custos em dólares. |
| `USAGE` | Monitora quantidade de uso (horas, GB, etc.). |
| `RI_UTILIZATION` | Monitora a utilização de Reserved Instances. |
| `RI_COVERAGE` | Monitora a cobertura de Reserved Instances. |
| `SAVINGS_PLANS_UTILIZATION` | Monitora a utilização de Savings Plans. |
| `SAVINGS_PLANS_COVERAGE` | Monitora a cobertura de Savings Plans. |

### Exemplo de Criação de Orçamento com Alerta

```json
{
  "AccountId": "123456789012",
  "Budget": {
    "BudgetName": "Monthly-Total-Budget",
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
          "Address": "finops@example.com"
        }
      ]
    }
  ]
}
```

---

## 2.3 Cost and Usage Reports (CUR) API

O Cost and Usage Report (CUR) é o relatório mais detalhado e granular de custos da AWS. Ele é entregue em um bucket S3 e pode ser integrado com Athena, Redshift ou QuickSight para análise.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cur.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **X-Amz-Target Prefix** | `AWSOrigamiServiceGatewayService` |
| **Service Name (IAM)** | `cur` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `PutReportDefinition` | Cria uma nova definição de relatório CUR. |
| `DescribeReportDefinitions` | Lista todas as definições de relatórios CUR existentes. |
| `DeleteReportDefinition` | Remove uma definição de relatório. |
| `ModifyReportDefinition` | Modifica uma definição de relatório existente. |
| `ListTagsForResource` | Lista as tags de um relatório. |
| `TagResource` | Adiciona tags a um relatório. |
| `UntagResource` | Remove tags de um relatório. |

### Parâmetros Importantes do CUR

| Parâmetro | Descrição |
| :--- | :--- |
| `TimeUnit` | `HOURLY`, `DAILY` ou `MONTHLY`. Hourly fornece a maior granularidade. |
| `AdditionalSchemaElements` | `RESOURCES` (inclui IDs de recursos), `SPLIT_COST_ALLOCATION_DATA` (dados de alocação dividida). |
| `AdditionalArtifacts` | `ATHENA` (formato Parquet para Athena), `REDSHIFT`, `QUICKSIGHT`. |
| `RefreshClosedReports` | Se `true`, atualiza relatórios de meses fechados quando há ajustes. |

---

## 2.4 BCM Data Exports API

O BCM Data Exports é um serviço mais recente que permite criar exportações personalizadas de múltiplos conjuntos de dados de billing e cost management.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://bcm-data-exports.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **X-Amz-Target Prefix** | `AWSBillingAndCostManagementDataExports` |
| **Service Name (IAM)** | `bcm-data-exports` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `CreateExport` | Cria uma nova exportação de dados. |
| `GetExport` | Retorna detalhes de uma exportação. |
| `ListExports` | Lista todas as exportações configuradas. |
| `UpdateExport` | Atualiza uma exportação existente. |
| `DeleteExport` | Remove uma exportação. |
| `GetTable` | Retorna metadados de uma tabela de dados. |
| `ListTables` | Lista as tabelas de dados disponíveis para exportação. |
| `ListExecutions` | Lista as execuções de uma exportação. |

---

## 2.5 AWS Billing Conductor API

O Billing Conductor permite criar versões pro forma dos dados de billing, útil para organizações que precisam redistribuir custos entre unidades de negócio ou clientes.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://billingconductor.us-east-1.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `billingconductor` |

### Ações Disponíveis

| Ação | Método HTTP | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `ListBillingGroups` | POST | `/list-billing-groups` | Lista os grupos de billing. |
| `ListBillingGroupCostReports` | POST | `/list-billing-group-cost-reports` | Relatórios de custo por grupo. |
| `ListPricingPlans` | POST | `/list-pricing-plans` | Lista os planos de precificação. |
| `ListPricingRules` | POST | `/list-pricing-rules` | Lista as regras de precificação. |
| `ListAccountAssociations` | POST | `/list-account-associations` | Lista associações de contas. |
| `ListCustomLineItems` | POST | `/list-custom-line-items` | Lista itens de linha personalizados. |
| `CreateBillingGroup` | POST | `/create-billing-group` | Cria um grupo de billing. |
| `CreatePricingPlan` | POST | `/create-pricing-plan` | Cria um plano de precificação. |
| `CreatePricingRule` | POST | `/create-pricing-rule` | Cria uma regra de precificação. |

## Referências

- [AWS Budgets API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations_AWS_Budgets.html)
- [CUR API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations_AWS_Cost_and_Usage_Report_Service.html)
- [BCM Data Exports API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations_AWS_Billing_and_Cost_Management_Data_Exports.html)
- [Billing Conductor API Reference](https://docs.aws.amazon.com/billingconductor/latest/APIReference/Welcome.html)
