# Guia Devastadoramente Detalhado: AWS Cost Optimization Hub API

## Visão Geral

O Cost Optimization Hub é um serviço centralizador que agrega e consolida recomendações de otimização de custos de múltiplos serviços da AWS (como Cost Explorer, Compute Optimizer e Trusted Advisor) em um único local. Ele ajuda a quantificar e priorizar as oportunidades de economia em toda a sua organização AWS, tornando-se um ponto de partida crucial para a tomada de decisões em FinOps.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cost-optimization-hub.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `CostOptimizationHubService` |
| **Service Name (IAM)** | `cost-optimization-hub` |

---

## 1. ListRecommendations

Lista as recomendações de otimização de custos disponíveis, permitindo filtros e ordenação para priorizar as ações mais impactantes.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `filter` | Objeto | Não | Um objeto complexo para filtrar as recomendações. Você pode filtrar por `accountId`, `region`, `resourceType`, `recommendationId`, `actionType` (ex: `Rightsize`, `Terminate`), `tag`, `implementationEffort` (`VeryLow`, `Low`, `Medium`, `High`, `VeryHigh`).<br>**Uso**: Essencial para focar. Ex: filtrar por `resourceType: "Ec2Instance"` e `implementationEffort: "VeryLow"` para encontrar as vitórias fáceis e de baixo risco. |
| `orderBy` | Objeto | Não | Ordena os resultados. É um objeto com `dimension` (ex: `savingsAmount`, `costAmount`) e `order` (`Asc` ou `Desc`).<br>**Uso**: **Fundamental para priorização**. Ordene por `savingsAmount` em ordem `Desc` para ver as maiores oportunidades de economia primeiro. |
| `includeMemberAccounts` | Boolean | Não | Se `true` e você for a conta de gerenciamento, a busca incluirá recomendações de todas as contas membro. Padrão: `false`. Essencial para uma visão centralizada. |
| `maxResults` | Integer | Não | Número máximo de resultados por página. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Top 10 maiores economias em EC2 com baixo esforço)

```json
{
  "filter": {
    "resourceType": "Ec2Instance",
    "implementationEffort": "VeryLow"
  },
  "orderBy": {
    "dimension": "savingsAmount",
    "order": "Desc"
  },
  "maxResults": 10,
  "includeMemberAccounts": true
}
```

---

## 2. ListRecommendationSummaries

Fornece um resumo agregado das economias estimadas, agrupadas por uma dimensão específica. É perfeito para criar visões de alto nível e dashboards para a liderança.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `groupBy` | String | **Sim** | A dimensão pela qual agrupar os resumos.<br>**Valores**: `accountId`, `region`, `resourceType`, `actionType`, `implementationEffort`, `tag`.<br>**Uso**: Use `groupBy: "resourceType"` para ver o potencial de economia por serviço (EC2 vs RDS vs Lambda). Use `groupBy: "accountId"` para comparar o potencial entre contas e identificar quais times precisam de mais apoio. |
| `filter` | Objeto | Não | Filtra as recomendações a serem incluídas no resumo. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Potencial de economia por tipo de recurso)

```json
{
  "groupBy": "resourceType"
}
```

---

## 3. GetRecommendation

Recupera os detalhes completos de uma única recomendação de otimização de custos, incluindo o recurso específico, a ação recomendada e a economia estimada.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `recommendationId` | String | **Sim** | O ID da recomendação que você deseja inspecionar. Este ID é obtido na resposta da chamada `ListRecommendations`. |

### Exemplo de Requisição

```json
{
  "recommendationId": "rec-a1b2c3d4-e5f6-7890-1234-567890abcdef"
}
```

---

## 4. UpdateEnrollmentStatus

Ativa ou desativa o Cost Optimization Hub para a conta. O serviço precisa estar ativo para coletar e agregar recomendações.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `status` | String | **Sim** | O novo status de inscrição.<br>**Valores**: `Active` para ativar, `Inactive` para desativar. |
| `includeMemberAccounts` | Boolean | Não | Se `true` e você for a conta de gerenciamento, o status será aplicado a todas as contas membro. |

### Exemplo de Requisição (Ativar o serviço para toda a organização)

```json
{
  "status": "Active",
  "includeMemberAccounts": true
}
```

---

## 5. GetPreferences / UpdatePreferences

Permite visualizar e configurar suas preferências para o tipo de recomendações que você deseja receber, como a forma de calcular a economia.

### `GetPreferences`

Não possui parâmetros de entrada.

### `UpdatePreferences`

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `savingsEstimationMode` | String | Não | O modo de estimativa de economia.<br>**Valores**: `BeforeDiscounts` (antes de descontos de RI/SP, mostra a economia bruta) ou `AfterDiscounts` (depois dos descontos, mostra a economia líquida real).<br>**Uso**: `AfterDiscounts` é geralmente mais útil para entender o impacto real no seu bolso. |
| `memberAccountDiscountVisibility` | String | Não | Controla a visibilidade dos descontos de contas membro.<br>**Valores**: `All` (todos veem) ou `None` (ninguém vê). |

### Exemplo de Requisição (Atualizar Preferências para mostrar economia líquida)

```json
{
  "savingsEstimationMode": "AfterDiscounts"
}
```
