# Guia Detalhado: AWS Compute Optimizer API

## Visão Geral

O Compute Optimizer utiliza machine learning para analisar métricas de utilização e fornecer recomendações de right-sizing para diversos tipos de recursos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://compute-optimizer.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `ComputeOptimizerService` |
| **Service Name (IAM)** | `compute-optimizer` |

---

## Ações da API

### 1. GetEC2InstanceRecommendations

Retorna recomendações de right-sizing para instâncias EC2.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `instanceArns` | Array | Não | ARNs das instâncias para obter recomendações. |
| `nextToken` | String | Não | Token para paginação. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `filters` | Array | Não | Filtros por `finding`, `recommendationSourceType`, etc. |
| `accountIds` | Array | Não | IDs de contas para obter recomendações (requer permissão de organização). |
| `recommendationPreferences` | Object | Não | Preferências de recomendação (ex: `cpuVendorArchitectures`). |

#### Exemplo de Requisição

```json
{
  "filters": [
    {
      "name": "finding",
      "values": ["Overprovisioned"]
    }
  ],
  "maxResults": 100
}
```

---

### 2. GetEBSVolumeRecommendations

Retorna recomendações de otimização para volumes EBS.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `volumeArns` | Array | Não | ARNs dos volumes para obter recomendações. |
| `nextToken` | String | Não | Token para paginação. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `filters` | Array | Não | Filtros por `finding`. |
| `accountIds` | Array | Não | IDs de contas para obter recomendações. |

#### Exemplo de Requisição

```json
{
  "filters": [
    {
      "name": "finding",
      "values": ["NotOptimized"]
    }
  ]
}
```

---

### 3. GetLambdaFunctionRecommendations

Retorna recomendações de memória para funções Lambda.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `functionArns` | Array | Não | ARNs das funções para obter recomendações. |
| `nextToken` | String | Não | Token para paginação. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `filters` | Array | Não | Filtros por `finding`. |
| `accountIds` | Array | Não | IDs de contas para obter recomendações. |

#### Exemplo de Requisição

```json
{
  "maxResults": 100
}
```

---

### 4. GetRecommendationSummaries

Retorna um resumo das recomendações por tipo de recurso e por tipo de achado (finding).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `accountIds` | Array | Não | IDs de contas para obter resumos. |
| `nextToken` | String | Não | Token para paginação. |
| `maxResults` | Integer | Não | Número máximo de resultados. |

#### Exemplo de Requisição

```json
{
  "accountIds": ["123456789012"]
}
```

---

### 5. ExportEC2InstanceRecommendations

Exporta as recomendações de instâncias EC2 para um bucket S3 em formato CSV.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `s3DestinationConfig` | Object | Sim | Configuração do bucket S3 de destino. |
| `fileFormat` | String | Não | `Csv` (padrão). |
| `includeMemberAccounts` | Boolean | Não | Se deve incluir contas membro da organização. |
| `filters` | Array | Não | Filtros para exportar um subconjunto de recomendações. |
| `fieldsToExport` | Array | Não | Campos específicos a serem exportados. |
| `recommendationPreferences` | Object | Não | Preferências de recomendação. |

#### Exemplo de Requisição

```json
{
  "s3DestinationConfig": {
    "bucket": "my-compute-optimizer-exports",
    "keyPrefix": "ec2-recommendations/"
  },
  "fileFormat": "Csv",
  "includeMemberAccounts": true,
  "fieldsToExport": [
    "AccountId",
    "InstanceArn",
    "InstanceName",
    "Finding",
    "CurrentInstanceType",
    "RecommendationOptionsInstanceType",
    "RecommendationOptionsProjectedUtilizationMetricsCpuMaximum",
    "EstimatedMonthlySavingsAmount"
  ]
}
```

---

### 6. GetEnrollmentStatus

Verifica se a conta está inscrita no Compute Optimizer e se a coleta de dados está ativa.

#### Parâmetros de Entrada

Nenhum.

#### Exemplo de Requisição

```json
{}
```
