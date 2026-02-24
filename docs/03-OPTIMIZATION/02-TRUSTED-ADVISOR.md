# Guia Devastadoramente Detalhado: AWS Trusted Advisor API

## Visão Geral

O AWS Trusted Advisor inspeciona seu ambiente AWS e faz recomendações para seguir as melhores práticas em cinco categorias: otimização de custos, performance, segurança, tolerância a falhas e limites de serviço. A API permite que você acesse os resultados dessas verificações de forma programática. **Importante**: O acesso programático ao Trusted Advisor requer um plano de suporte **Business, Enterprise On-Ramp ou Enterprise**.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://support.{region}.amazonaws.com` (API Legada) / `https://trustedadvisor.{region}.amazonaws.com` (API v2) |
| **Protocolo** | JSON-RPC (POST) / REST (JSON) |
| **Content-Type** | `application/x-amz-json-1.1` / `application/json` |
| **X-Amz-Target Prefix** | `AWSSupport_20130415` (API Legada) |
| **Service Name (IAM)** | `support` / `trustedadvisor` |

---

## API Legada (via AWS Support)

Esta é a API mais antiga, mas ainda funcional.

### 1. DescribeTrustedAdvisorChecks

Descreve as verificações disponíveis no Trusted Advisor, retornando seus nomes, IDs, descrições e categorias.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `language` | String | **Sim** | O idioma para a descrição das verificações. Use `"en"` para inglês, pois é o mais completo. Outros idiomas como `"ja"` (japonês) e `"fr"` (francês) são suportados, mas o português não é garantido para todas as descrições na API. |

#### Exemplo de Requisição

```json
{
  "language": "en"
}
```

### 2. DescribeTrustedAdvisorCheckResult

Retorna o resultado detalhado de uma verificação específica, incluindo a lista de recursos sinalizados e metadados sobre cada recurso.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `checkId` | String | **Sim** | O ID da verificação que você deseja executar. Este ID é obtido da resposta da chamada `DescribeTrustedAdvisorChecks`. Cada verificação (ex: "Low Utilization Amazon EC2 Instances") tem um ID único. |
| `language` | String | Não | O idioma para o resultado. Use `"en"` para consistência. |

#### Exemplo de Requisição (Verificação de Instâncias EC2 Ociosas)

```json
{
  "checkId": "L4_T1_OP_EC2_Idle_Instances",
  "language": "en"
}
```

### 3. RefreshTrustedAdvisorCheck

Solicita uma atualização para uma verificação específica do Trusted Advisor. As verificações não são em tempo real e os resultados podem ficar em cache por algum tempo. Use esta chamada para forçar uma nova análise.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `checkId` | String | **Sim** | O ID da verificação que você deseja atualizar. |

#### Exemplo de Requisição

```json
{
  "checkId": "L4_T1_OP_EC2_Idle_Instances"
}
```

---

## API v2 (Trusted Advisor)

Esta é a API mais moderna e recomendada, oferecendo mais filtros e uma estrutura RESTful.

### 1. ListChecks

Lista as verificações disponíveis no Trusted Advisor, com mais opções de filtro.

**Método**: `GET`
**Path**: `/v2/checks`

#### Parâmetros de Query

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `pillar` | String | Não | Filtra por pilar: `cost_optimizing`, `performance`, `security`, `fault_tolerance`, `service_limits`. Essencial para focar nas verificações de FinOps (`cost_optimizing`). |
| `language` | String | Não | Suporta mais idiomas, incluindo `pt_BR` para português do Brasil. |
| `awsService` | String | Não | Filtra as verificações por um serviço AWS específico (ex: `Amazon EC2`). |
| `maxResults` | Integer | Não | Número máximo de resultados por página. |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição (Verificações de Otimização de Custo em Português)

```
GET /v2/checks?pillar=cost_optimizing&language=pt_BR
```

### 2. ListRecommendations

Lista as recomendações (ou seja, os recursos que foram sinalizados em alguma verificação).

**Método**: `GET`
**Path**: `/v2/recommendations`

#### Parâmetros de Query

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `pillar` | String | Não | Filtra por pilar. `cost_optimizing` é o mais relevante para FinOps. |
| `status` | String | Não | Filtra pelo status do recurso na verificação: `ok` (verde), `warning` (amarelo), `error` (vermelho).<br>**Uso**: Filtrar por `warning` e `error` para encontrar problemas ativos. |
| `checkIdentifier` | String | Não | O ID de uma verificação específica para obter apenas as suas recomendações. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição (Recursos com status de 'erro' no pilar de otimização de custo)

```
GET /v2/recommendations?pillar=cost_optimizing&status=error
```

### 3. UpdateRecommendationLifecycle

Atualiza o ciclo de vida de uma recomendação, permitindo que você marque um item como "em andamento" ou "resolvido". Isso ajuda a gerenciar o fluxo de trabalho de otimização.

**Método**: `PUT`
**Path**: `/v2/recommendations/{recommendationIdentifier}/lifecycle`

#### Parâmetros

| Parâmetro | Local | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- | :--- |
| `recommendationIdentifier` | Path | String | **Sim** | O ID da recomendação que você está atualizando. |
| `lifecycleStage` | Body | String | **Sim** | O novo estágio do ciclo de vida.<br>**Valores**: `in_progress` (em andamento), `dismissed` (ignorado), `resolved` (resolvido). |
| `updateReason` | Body | String | Não | Um texto livre explicando o motivo da atualização (ex: "Instância terminada pelo time de dev"). |

#### Exemplo de Requisição (Marcar uma recomendação como resolvida)

```json
{
  "lifecycleStage": "resolved",
  "updateReason": "A instância ociosa foi terminada em 22/02/2026."
}
```
