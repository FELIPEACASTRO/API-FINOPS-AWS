# Guia Devastadoramente Detalhado: AWS Savings Plans API

## Visão Geral

Savings Plans são um modelo de preço flexível que oferece descontos significativos (semelhantes a RIs) em troca de um compromisso de uso de computação (medido em USD/hora) por um período de 1 ou 3 anos. A API do Savings Plans permite que você gerencie seus planos, descreva as ofertas disponíveis e analise seu inventário de planos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://savingsplans.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `savingsplans` |

---

## 1. DescribeSavingsPlansOfferings

Descreve as ofertas de Savings Plans disponíveis para compra. Use esta ação para explorar as opções antes de se comprometer.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `offeringIds` | Array de Strings | Não | Uma lista de IDs de ofertas específicas, se você já sabe quais quer ver. |
| `paymentOptions` | Array de Strings | Não | Filtra por opção de pagamento: `No Upfront` (sem pagamento adiantado), `Partial Upfront` (parcialmente adiantado), `All Upfront` (totalmente adiantado). |
| `planTypes` | Array de Strings | Não | Filtra por tipo de plano: `Compute` (flexível entre EC2, Fargate, Lambda), `EC2Instance` (preso a uma família de instância EC2 em uma região), `SageMaker`. |
| `products` | Array de Strings | Não | Filtra por produto coberto pelo plano: `EC2`, `Fargate`, `Lambda`, `SageMaker`. |
| `durations` | Array de Inteiros | Não | Filtra pela duração do termo em segundos. Use `31536000` para 1 ano e `94608000` para 3 anos. |
| `currencies` | Array de Strings | Não | Filtra por moeda: `USD`, `CNY`. |
| `filters` | Array de Objetos | Não | Filtros mais genéricos com `name` (ex: `region`, `instanceFamily`) e `values`. Útil para encontrar ofertas para uma família de instância específica. |
| `maxResults` | Integer | Não | Número máximo de resultados por página. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Ofertas de Compute SP de 1 ano, sem pagamento adiantado)

```json
{
  "planTypes": ["Compute"],
  "paymentOptions": ["No Upfront"],
  "durations": [31536000]
}
```

---

## 2. DescribeSavingsPlans

Descreve os Savings Plans que você já possui, incluindo seu status, compromisso e período de validade.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `savingsPlanArns` | Array de Strings | Não | ARNs de Savings Plans específicos que você deseja descrever. |
| `savingsPlanIds` | Array de Strings | Não | IDs de Savings Plans específicos. |
| `states` | Array de Strings | Não | Filtra por estado do plano: `payment-pending`, `payment-failed`, `active`, `retired`. Use `active` para ver seus compromissos atuais. |
| `filters` | Array de Objetos | Não | Filtros mais genéricos com `name` (ex: `region`, `ec2-instance-family`) e `values`. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Listar todos os planos ativos)

```json
{
  "states": ["active"]
}
```

---

## 3. DescribeSavingsPlanRates

Descreve as taxas (preços com desconto) de um Savings Plan específico para diferentes produtos. Isso permite que você veja exatamente qual o preço por hora de uma instância `t3.micro`, por exemplo, sob o seu plano.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `savingsPlanId` | String | **Sim** | O ID do Savings Plan cujas taxas você deseja ver. |
| `filters` | Array de Objetos | Não | Filtra as taxas por `region`, `instanceType`, `productDescription`, etc. Essencial para encontrar a taxa de um produto específico. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Taxas de um SP para instâncias t3 na Virgínia)

```json
{
  "savingsPlanId": "sp-12345abcdef",
  "filters": [
    {
      "name": "region",
      "values": ["us-east-1"]
    },
    {
      "name": "instanceType",
      "values": ["t3.micro", "t3.small"]
    }
  ]
}
```

---

## 4. CreateSavingsPlan

Cria (compra) um novo Savings Plan. **Atenção: Esta é uma transação financeira que gera um compromisso de pagamento.**

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `savingsPlanOfferingId` | String | **Sim** | O ID da oferta do Savings Plan que você deseja comprar. Obtido de `DescribeSavingsPlansOfferings`. |
| `commitment` | String | **Sim** | O valor do compromisso horário em USD (ex: `"0.50"` para 50 centavos por hora). Este é o valor que você se compromete a gastar por hora durante o termo do plano. |
| `upfrontPaymentAmount` | String | Não | O valor do pagamento adiantado. Obrigatório se a oferta for `Partial Upfront` ou `All Upfront`. |
| `clientToken` | String | Não | Um token de idempotência para evitar compras duplicadas acidentais. A API gerará um se você não fornecer. |
| `tags` | Objeto | Não | Tags para associar ao Savings Plan, úteis para alocação de custos interna. |

### Exemplo de Requisição (Comprar um SP com compromisso de $1.25/hora)

```json
{
  "savingsPlanOfferingId": "offering-12345abcdef",
  "commitment": "1.25",
  "tags": {
    "Owner": "FinOpsTeam",
    "Project": "Global-Compute-SP"
  }
}
```
