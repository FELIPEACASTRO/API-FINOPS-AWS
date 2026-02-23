# Guia Detalhado: AWS Savings Plans API

## Visão Geral

A API de Savings Plans permite gerenciar e consultar informações sobre Savings Plans, que oferecem descontos significativos em troca de compromisso de uso por 1 ou 3 anos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://savingsplans.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `savingsplans` |

---

## Ações da API

### 1. DescribeSavingsPlans

Lista todos os Savings Plans da conta com detalhes.

**Método HTTP**: POST  
**Path**: `/DescribeSavingsPlans`

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `savingsPlanArns` | Array | Não | ARNs dos SPs a serem descritos. |
| `savingsPlanIds` | Array | Não | IDs dos SPs a serem descritos. |
| `states` | Array | Não | Filtro por estado: `payment-pending`, `payment-failed`, `active`, `retired`, `queued`, `queued-deleted`, `returned`. |
| `filters` | Array | Não | Filtros adicionais com `name` e `values`. |
| `maxResults` | Integer | Não | Número máximo de resultados (1-1000). |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "states": ["active"],
  "maxResults": 100
}
```

---

### 2. DescribeSavingsPlansOfferings

Lista as ofertas de Savings Plans disponíveis para compra.

**Método HTTP**: POST  
**Path**: `/DescribeSavingsPlansOfferings`

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `offeringIds` | Array | Não | IDs de ofertas específicas. |
| `paymentOptions` | Array | Não | `All Upfront`, `Partial Upfront`, `No Upfront`. |
| `productType` | String | Não | `EC2`, `Fargate`, `Lambda`, `SageMaker`. |
| `planTypes` | Array | Não | `Compute`, `EC2Instance`, `SageMaker`. |
| `durations` | Array | Não | Duração em segundos (ex: `31536000` para 1 ano, `94608000` para 3 anos). |
| `currencies` | Array | Não | `USD`, `CNY`. |
| `descriptions` | Array | Não | Filtro por descrição. |
| `serviceCodes` | Array | Não | Códigos de serviço. |
| `usageTypes` | Array | Não | Tipos de uso. |
| `operations` | Array | Não | Operações. |
| `filters` | Array | Não | Filtros adicionais. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "planTypes": ["Compute"],
  "paymentOptions": ["No Upfront"],
  "durations": [31536000],
  "productType": "EC2",
  "maxResults": 50
}
```

---

### 3. DescribeSavingsPlansOfferingRates

Lista as taxas detalhadas de uma oferta de Savings Plan.

**Método HTTP**: POST  
**Path**: `/DescribeSavingsPlansOfferingRates`

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `savingsPlanOfferingIds` | Array | Não | IDs de ofertas. |
| `savingsPlanPaymentOptions` | Array | Não | Opções de pagamento. |
| `savingsPlanTypes` | Array | Não | Tipos de SP. |
| `products` | Array | Não | Produtos. |
| `serviceCodes` | Array | Não | Códigos de serviço. |
| `usageTypes` | Array | Não | Tipos de uso. |
| `operations` | Array | Não | Operações. |
| `filters` | Array | Não | Filtros adicionais. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "savingsPlanTypes": ["Compute"],
  "products": ["EC2"],
  "maxResults": 100
}
```

---

### 4. CreateSavingsPlan

Compra um novo Savings Plan.

**Método HTTP**: POST  
**Path**: `/CreateSavingsPlan`

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `savingsPlanOfferingId` | String | Sim | ID da oferta a ser comprada. |
| `commitment` | String | Sim | Valor do compromisso por hora em USD (ex: `"10.0"`). |
| `upfrontPaymentAmount` | String | Não | Valor do pagamento antecipado. |
| `purchaseTime` | Timestamp | Não | Data/hora para compra futura (até 7 dias). |
| `clientToken` | String | Não | Token de idempotência. |
| `tags` | Object | Não | Tags a serem associadas. |

#### Exemplo de Requisição

```json
{
  "savingsPlanOfferingId": "offering-abc123",
  "commitment": "10.0"
}
```
