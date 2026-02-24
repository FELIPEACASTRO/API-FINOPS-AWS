# Guia Devastadoramente Detalhado: AWS Billing Conductor API

## Visão Geral

O AWS Billing Conductor é um serviço avançado para cenários de showback e chargeback. Ele permite que você crie uma versão "pro forma" (ou seja, uma simulação) da sua fatura, aplicando regras de precificação personalizadas e agrupando contas de forma lógica, independentemente da estrutura da sua AWS Organization. Isso é útil para empresas que precisam redistribuir custos internamente ou para clientes de provedores de soluções.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://billingconductor.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `billingconductor` |

---

## 1. CreateBillingGroup

Cria um grupo de billing, que é um conjunto lógico de contas AWS cujos custos serão calculados juntos sob um plano de precificação específico.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Name` | String | **Sim** | O nome do seu grupo de billing (ex: "Grupo-Cliente-A", "Grupo-Time-Marketing"). |
| `AccountGrouping` | Objeto | **Sim** | Define como as contas são agrupadas. Contém:<br>- `LinkedAccountIds` (Array de Strings, **Sim**): A lista dos IDs de 12 dígitos das contas que farão parte deste grupo.<br>- `AutoAssociate` (Boolean, Não): Se `true`, associa automaticamente novas contas da organização a este grupo. |
| `ComputationPreference` | Objeto | **Sim** | Define como os custos do grupo são calculados. Contém:<br>- `PricingPlanArn` (String, **Sim**): O ARN do plano de precificação que será aplicado a este grupo. |
| `PrimaryAccountId` | String | **Sim** | O ID da conta que será a "dona" deste grupo de billing. Os custos calculados serão visíveis a partir desta conta. |
| `Description` | String | Não | Uma descrição opcional para o grupo. |
| `Tags` | Objeto | Não | Tags para associar ao grupo de billing. |

### Exemplo de Requisição

```json
{
  "Name": "Grupo-Marketing-Chargeback",
  "AccountGrouping": {
    "LinkedAccountIds": [
      "111111111111",
      "222222222222"
    ]
  },
  "ComputationPreference": {
    "PricingPlanArn": "arn:aws:billingconductor::123456789012:pricingplan/PLAN_ID_AQUI"
  },
  "PrimaryAccountId": "123456789012"
}
```

---

## 2. CreatePricingPlan

Cria um plano de precificação, que é um contêiner para uma ou mais regras de precificação.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Name` | String | **Sim** | O nome do seu plano de precificação (ex: "Plano-Standard-Com-Markup", "Plano-Desconto-Parceiros"). |
| `Description` | String | Não | Uma descrição opcional. |
| `PricingRuleArns` | Array de Strings | Não | Uma lista de ARNs de regras de precificação a serem associadas a este plano no momento da criação. |
| `Tags` | Objeto | Não | Tags para associar ao plano. |

### Exemplo de Requisição

```json
{
  "Name": "Plano-Standard-Markup-10-porcento",
  "Description": "Aplica um markup de 10% sobre os custos públicos da AWS."
}
```

---

## 3. CreatePricingRule

Cria uma regra de precificação, que define como modificar a taxa pública de um serviço (aplicando um markup ou um desconto).

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Name` | String | **Sim** | O nome da sua regra (ex: "Markup-Global-10%", "Desconto-EC2-5%"). |
| `Scope` | String | **Sim** | O escopo da regra.<br>**Valores**: `GLOBAL` (aplica a tudo), `SERVICE` (aplica a um serviço específico como "AmazonEC2"), `BILLING_ENTITY` (aplica ao pagador). |
| `Type` | String | **Sim** | O tipo de modificação.<br>**Valores**: `MARKUP` (adiciona uma porcentagem ao custo), `DISCOUNT` (subtrai uma porcentagem do custo). |
| `ModifierPercentage` | Double | **Sim** | O valor percentual da modificação. Ex: `10.0` para 10%. |
| `Service` | String | Não | Se o `Scope` for `SERVICE`, este campo é obrigatório e deve conter o nome do serviço (ex: "AmazonEC2"). |
| `Description` | String | Não | Uma descrição opcional. |
| `Tags` | Objeto | Não | Tags para associar à regra. |

### Exemplo de Requisição (Regra de Markup Global)

```json
{
  "Name": "Markup-Global-10-porcento",
  "Scope": "GLOBAL",
  "Type": "MARKUP",
  "ModifierPercentage": 10.0,
  "Description": "Markup global de 10% para todos os serviços."
}
```

---

## 4. ListBillingGroupCostReports

Lista os relatórios de custo gerados para um grupo de billing específico, mostrando os custos pro forma calculados.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `BillingPeriod` | String | Não | O período de faturamento no formato `YYYY-MM`. Se não especificado, usa o período atual. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Objeto | Não | Permite filtrar os relatórios por `BillingGroupArns`. |

### Exemplo de Requisição

```json
{
  "BillingPeriod": "2026-02",
  "Filters": {
    "BillingGroupArns": [
      "arn:aws:billingconductor::123456789012:billinggroup/GROUP_ID_AQUI"
    ]
  }
}
```

---

## 5. CreateCustomLineItem

Cria um item de linha personalizado (uma cobrança ou crédito manual) que aparecerá na fatura pro forma de um grupo de billing. Útil para cobrar por serviços de suporte, licenças de software, ou aplicar créditos manualmente.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Name` | String | **Sim** | O nome do item de linha (ex: "Taxa de Suporte Premium", "Crédito de Boa Vontade"). |
| `Description` | String | **Sim** | Uma descrição detalhada do que é a cobrança/crédito. |
| `BillingGroupArn` | String | **Sim** | O ARN do grupo de billing onde este item de linha aparecerá. |
| `BillingPeriodRange` | Objeto | Não | O intervalo de períodos de faturamento em que este item se aplicará. Contém `InclusiveStartBillingPeriod` e `ExclusiveEndBillingPeriod`. |
| `ChargeDetails` | Objeto | **Sim** | Define os detalhes da cobrança. Contém `Type` (`FEE` ou `CREDIT`), e `Flat` (para um valor fixo) ou `Percentage` (para um valor percentual sobre os custos do grupo). |
| `AccountId` | String | Não | O ID da conta a ser associada a este item de linha. |
| `Tags` | Objeto | Não | Tags para associar ao item de linha. |

### Exemplo de Requisição (Cobrança Fixa de Suporte)

```json
{
  "Name": "Suporte-Premium-Fevereiro",
  "Description": "Taxa mensal fixa para suporte premium",
  "BillingGroupArn": "arn:aws:billingconductor::123456789012:billinggroup/GROUP_ID_AQUI",
  "BillingPeriodRange": {
    "InclusiveStartBillingPeriod": "2026-02"
  },
  "ChargeDetails": {
    "Type": "FEE",
    "Flat": {
      "Value": 500.00
    }
  }
}
```
