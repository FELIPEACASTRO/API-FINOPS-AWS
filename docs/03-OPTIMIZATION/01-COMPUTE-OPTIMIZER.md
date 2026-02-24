# Guia Devastadoramente Detalhado: AWS Compute Optimizer API

## Visão Geral

O AWS Compute Optimizer é um serviço que utiliza machine learning para analisar as métricas de configuração e utilização dos seus recursos (como instâncias EC2, volumes EBS, funções Lambda e serviços ECS) e gerar recomendações para otimizá-los. Ele ajuda a responder perguntas como "Estou usando o tipo de instância certo?" ou "Posso economizar dinheiro mudando o tamanho deste volume EBS?". A API permite que você acesse essas recomendações de forma programática para integrá-las em seus fluxos de trabalho de FinOps.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://compute-optimizer.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `ComputeOptimizerService` |
| **Service Name (IAM)** | `compute-optimizer` |

---

## 1. GetEC2InstanceRecommendations

Retorna recomendações de otimização para instâncias Amazon EC2.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `instanceArns` | Array de Strings | Não | Uma lista de ARNs de instâncias específicas para as quais você deseja obter recomendações. Se não for fornecido, retorna para todas as instâncias na conta/região. |
| `filters` | Array de Objetos | Não | Filtra as recomendações retornadas. Cada objeto de filtro tem `name` e `value`.<br>**Nomes de Filtro**: `Finding` (`Underprovisioned`, `Overprovisioned`, `Optimized`), `RecommendationSourceType` (`Ec2Instance`, `AutoScalingGroup`).<br>**Uso**: Essencial para focar. Use `Finding: Overprovisioned` para listar apenas as instâncias que podem ser reduzidas para economizar custos. |
| `accountIds` | Array de Strings | Não | Se você for a conta de gerenciamento, pode especificar para quais contas membro deseja obter recomendações. |
| `maxResults` | Integer | Não | Número máximo de resultados por página. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Instâncias superprovisionadas)

```json
{
  "filters": [
    {
      "name": "Finding",
      "value": "Overprovisioned"
    }
  ]
}
```

---

## 2. GetAutoScalingGroupRecommendations

Retorna recomendações de otimização para grupos de Auto Scaling.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `autoScalingGroupArns` | Array de Strings | Não | ARNs de grupos de Auto Scaling específicos. |
| `filters` | Array de Objetos | Não | Filtra as recomendações. Mesmos filtros de `GetEC2InstanceRecommendations`. |
| `accountIds` | Array de Strings | Não | IDs de contas membro. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{
  "filters": [
    {
      "name": "Finding",
      "value": "NotOptimized"
    }
  ]
}
```

---

## 3. GetEBSVolumeRecommendations

Retorna recomendações de otimização para volumes Amazon EBS.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `volumeArns` | Array de Strings | Não | ARNs de volumes EBS específicos. |
| `filters` | Array de Objetos | Não | Filtra as recomendações. `Finding` (`Optimized`, `NotOptimized`). |
| `accountIds` | Array de Strings | Não | IDs de contas membro. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Volumes não otimizados)

```json
{
  "filters": [
    {
      "name": "Finding",
      "value": "NotOptimized"
    }
  ]
}
```

---

## 4. GetLambdaFunctionRecommendations

Retorna recomendações de otimização de memória para funções AWS Lambda.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `functionArns` | Array de Strings | Não | ARNs de funções Lambda específicas. |
| `filters` | Array de Objetos | Não | Filtra as recomendações. `Finding` (`Optimized`, `NotOptimized`, `Unavailable`). |
| `accountIds` | Array de Strings | Não | IDs de contas membro. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{
  "filters": [
    {
      "name": "Finding",
      "value": "NotOptimized"
    }
  ]
}
```

---

## 5. GetECSServiceRecommendations

Retorna recomendações de otimização de CPU e memória para serviços Amazon ECS.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `serviceArns` | Array de Strings | Não | ARNs de serviços ECS específicos. |
| `filters` | Array de Objetos | Não | Filtra as recomendações. `Finding` (`Optimized`, `Underprovisioned`, `Overprovisioned`). |
| `accountIds` | Array de Strings | Não | IDs de contas membro. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Serviços ECS superprovisionados)

```json
{
  "filters": [
    {
      "name": "Finding",
      "value": "Overprovisioned"
    }
  ]
}
```

---

## 6. GetEnrollmentStatus

Verifica se o Compute Optimizer está ativado para a conta. O serviço precisa estar ativo para gerar recomendações.

### Parâmetros de Entrada

Nenhum.

### Exemplo de Requisição

```json
{}
```

---

## 7. UpdateEnrollmentStatus

Ativa ou desativa o Compute Optimizer.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `status` | String | **Sim** | O novo status.<br>**Valores**: `Active` (Ativo), `Inactive` (Inativo), `Pending` (Pendente).<br>**Uso**: Use `Active` para começar a coletar métricas e gerar recomendações. |
| `includeMemberAccounts` | Boolean | Não | Se `true` e você for a conta de gerenciamento, o status será aplicado a todas as contas membro. |

### Exemplo de Requisição (Ativar para toda a organização)

```json
{
  "status": "Active",
  "includeMemberAccounts": true
}
```

---

## 8. GetRecommendationSummaries

Fornece um resumo agregado do número de recursos analisados e o potencial de economia, agrupado por tipo de recurso e status da recomendação.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `accountIds` | Array de Strings | Não | IDs de contas membro para incluir no resumo. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{}
```
