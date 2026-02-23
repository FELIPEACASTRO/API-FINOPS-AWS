# Guia Detalhado: AWS Billing Conductor API

## Visão Geral

O Billing Conductor permite criar versões pro forma dos dados de billing, útil para organizações que precisam redistribuir custos entre unidades de negócio ou clientes (showback/chargeback).

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://billingconductor.us-east-1.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `billingconductor` |

---

## Ações da API

### 1. ListBillingGroups

Lista os grupos de billing configurados.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BillingPeriod` | String | Não | Período de billing no formato `YYYY-MM`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Object | Não | Filtros por `Arns`, `PricingPlan`, `Statuses`, `AutoAssociate`. |

**Método HTTP**: POST  
**Path**: `/list-billing-groups`

#### Exemplo de Requisição

```json
{
  "BillingPeriod": "2026-02",
  "MaxResults": 100
}
```

---

### 2. ListBillingGroupCostReports

Retorna relatórios de custo por grupo de billing.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BillingPeriod` | String | Não | Período de billing. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Object | Não | Filtros por `BillingGroupArns`. |

**Método HTTP**: POST  
**Path**: `/list-billing-group-cost-reports`

#### Exemplo de Requisição

```json
{
  "BillingPeriod": "2026-01"
}
```

---

### 3. CreateBillingGroup

Cria um novo grupo de billing.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Name` | String | Sim | Nome do grupo de billing. |
| `AccountGrouping` | Object | Sim | Objeto com `LinkedAccountIds` (lista de IDs de contas) e `AutoAssociate` (boolean). |
| `ComputationPreference` | Object | Sim | Objeto com `PricingPlanArn` (ARN do plano de precificação). |
| `PrimaryAccountId` | String | Sim | ID da conta primária do grupo. |
| `Description` | String | Não | Descrição do grupo. |
| `Tags` | Object | Não | Tags a serem associadas. |

**Método HTTP**: POST  
**Path**: `/create-billing-group`

#### Exemplo de Requisição

```json
{
  "Name": "Grupo-Producao",
  "AccountGrouping": {
    "LinkedAccountIds": ["111111111111", "222222222222"],
    "AutoAssociate": false
  },
  "ComputationPreference": {
    "PricingPlanArn": "arn:aws:billingconductor::123456789012:pricingplan/abc123"
  },
  "PrimaryAccountId": "123456789012",
  "Description": "Grupo de billing para contas de produção"
}
```

---

### 4. ListPricingPlans

Lista os planos de precificação.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BillingPeriod` | String | Não | Período de billing. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Object | Não | Filtros por `Arns`. |

**Método HTTP**: POST  
**Path**: `/list-pricing-plans`

#### Exemplo de Requisição

```json
{}
```

---

### 5. CreatePricingRule

Cria uma regra de precificação personalizada.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Name` | String | Sim | Nome da regra. |
| `Scope` | String | Sim | `GLOBAL`, `SERVICE`, `BILLING_ENTITY`, `SKU`. |
| `Type` | String | Sim | `MARKUP`, `DISCOUNT`, `TIERING`. |
| `ModifierPercentage` | Double | Sim (se MARKUP/DISCOUNT) | Percentual de markup ou desconto. |
| `Service` | String | Sim (se scope SERVICE) | Código do serviço AWS. |
| `Description` | String | Não | Descrição da regra. |
| `BillingEntity` | String | Não | Entidade de billing. |
| `Tiering` | Object | Não | Configuração de tiering. |
| `UsageType` | String | Não | Tipo de uso. |
| `Operation` | String | Não | Operação. |
| `Tags` | Object | Não | Tags. |

**Método HTTP**: POST  
**Path**: `/create-pricing-rule`

#### Exemplo de Requisição

```json
{
  "Name": "Markup-10-Percent",
  "Scope": "GLOBAL",
  "Type": "MARKUP",
  "ModifierPercentage": 10.0,
  "Description": "Markup de 10% para chargeback interno"
}
```

---

### 6. ListAccountAssociations

Lista as associações de contas com grupos de billing.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BillingPeriod` | String | Não | Período de billing. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Object | Não | Filtros por `Association`, `AccountId`, `AccountIds`. |

**Método HTTP**: POST  
**Path**: `/list-account-associations`

#### Exemplo de Requisição

```json
{}
```
