# Guia Detalhado: AWS Trusted Advisor APIs

## Visão Geral

O Trusted Advisor fornece recomendações em cinco categorias: otimização de custos, performance, segurança, tolerância a falhas e limites de serviço. Existem duas APIs: a API Legacy (via Support API) e a nova API REST.

---

## API 1: Trusted Advisor via Support API (Legacy)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://support.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSSupport_20130415` |
| **Service Name (IAM)** | `support` |
| **Requisito** | Plano de suporte Business ou Enterprise |

### 1. DescribeTrustedAdvisorChecks

Lista todas as verificações disponíveis do Trusted Advisor.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `language` | String | Sim | Idioma dos resultados: `en` (inglês), `ja` (japonês), `fr` (francês), `zh` (chinês). |

#### Exemplo de Requisição

```json
{
  "language": "en"
}
```

---

### 2. DescribeTrustedAdvisorCheckResult

Retorna o resultado detalhado de uma verificação específica, incluindo os recursos afetados.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `checkId` | String | Sim | O ID da verificação. |
| `language` | String | Não | Idioma dos resultados. |

#### Exemplo de Requisição

```json
{
  "checkId": "Qch7DwouX1",
  "language": "en"
}
```

---

### 3. DescribeTrustedAdvisorCheckSummaries

Retorna resumos de uma ou mais verificações.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `checkIds` | Array | Sim | Lista de IDs de verificações. |

#### Exemplo de Requisição

```json
{
  "checkIds": ["Qch7DwouX1", "DAvU99Dc4C", "Z4AUBRNSmz"]
}
```

---

### 4. RefreshTrustedAdvisorCheck

Solicita a atualização de uma verificação.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `checkId` | String | Sim | O ID da verificação a ser atualizada. |

#### Exemplo de Requisição

```json
{
  "checkId": "Qch7DwouX1"
}
```

---

## API 2: Trusted Advisor REST API (Nova)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://trustedadvisor.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `trustedadvisor` |

### 1. ListChecks

Lista todas as verificações disponíveis.

**Método HTTP**: GET  
**Path**: `/v2/checks`

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |
| `pillar` | String | Não | Filtro por pilar: `cost_optimizing`, `performance`, `security`, `fault_tolerance`, `service_limits`, `operational_excellence`. |
| `language` | String | Não | Idioma: `en`, `ja`, `zh`, `fr`, `de`, `ko`, `zh_TW`, `it`, `pt_BR`, `es`, `id`. |
| `awsService` | String | Não | Filtro por serviço AWS. |
| `source` | String | Não | Filtro por fonte: `aws_config`, `compute_optimizer`, `cost_explorer`, `lse`, `manual`, `pse`, `rds`, `resilience`, `resilience_hub`, `security_hub`, `stir`, `ta_check`. |

#### Exemplo de Requisição

```
GET /v2/checks?pillar=cost_optimizing&maxResults=100
```

---

### 2. ListRecommendations

Lista todas as recomendações ativas.

**Método HTTP**: GET  
**Path**: `/v2/recommendations`

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |
| `pillar` | String | Não | Filtro por pilar. |
| `status` | String | Não | `ok`, `warning`, `error`. |
| `awsService` | String | Não | Filtro por serviço AWS. |
| `source` | String | Não | Filtro por fonte. |
| `type` | String | Não | `standard` ou `priority`. |
| `checkIdentifier` | String | Não | Filtro por ID de verificação. |
| `afterLastUpdatedAt` | Timestamp | Não | Filtra recomendações atualizadas após esta data. |
| `beforeLastUpdatedAt` | Timestamp | Não | Filtra recomendações atualizadas antes desta data. |

#### Exemplo de Requisição

```
GET /v2/recommendations?pillar=cost_optimizing&status=warning
```

---

### 3. GetRecommendation

Retorna detalhes de uma recomendação específica.

**Método HTTP**: GET  
**Path**: `/v2/recommendations/{recommendationIdentifier}`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `recommendationIdentifier` | String | Sim | ARN da recomendação. |

---

### 4. ListRecommendationResources

Lista os recursos afetados por uma recomendação.

**Método HTTP**: GET  
**Path**: `/v2/recommendations/{recommendationIdentifier}/resources`

#### Parâmetros

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `recommendationIdentifier` | String (path) | Sim | ARN da recomendação. |
| `maxResults` | Integer (query) | Não | Número máximo de resultados. |
| `nextToken` | String (query) | Não | Token para paginação. |
| `status` | String (query) | Não | `ok`, `warning`, `error`. |
| `regionCode` | String (query) | Não | Filtro por região. |
| `exclusionStatus` | String (query) | Não | `excluded` ou `included`. |

---

### 5. UpdateRecommendationLifecycle

Atualiza o ciclo de vida de uma recomendação (ex: marcar como resolvida).

**Método HTTP**: PUT  
**Path**: `/v2/recommendations/{recommendationIdentifier}/lifecycle`

#### Parâmetros

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `recommendationIdentifier` | String (path) | Sim | ARN da recomendação. |
| `lifecycleStage` | String (body) | Sim | `pending_response`, `in_progress`, `dismissed`, `resolved`. |
| `updateReason` | String (body) | Não | Motivo da atualização. |
| `updateReasonCode` | String (body) | Não | Código do motivo: `non_critical_account`, `temporary_account`, `valid_business_case`, `other_methods_available`, `low_priority`, `not_applicable`, `other`. |

#### Exemplo de Requisição

```json
{
  "lifecycleStage": "resolved",
  "updateReason": "Instância foi terminada"
}
```
