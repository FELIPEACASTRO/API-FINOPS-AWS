# Guia Detalhado: Resource Groups Tagging e AWS Config APIs (FinOps)

---

## 1. Resource Groups Tagging API

A Tagging API é essencial para governança de tags, que é a base da alocação de custos em FinOps.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://tagging.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `ResourceGroupsTaggingAPI_20170126` |
| **Service Name (IAM)** | `tagging` |

### 1.1 GetResources

Retorna recursos que possuem as tags especificadas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TagFilters` | Array | Não | Filtros por tag. Cada filtro tem `Key` (obrigatório) e `Values` (opcional). |
| `ResourceTypeFilters` | Array | Não | Filtro por tipo de recurso (ex: `ec2:instance`, `rds:db`, `s3`). |
| `TagsPerPage` | Integer | Não | Número de tags por página (padrão: 100). |
| `PaginationToken` | String | Não | Token para paginação. |
| `IncludeComplianceDetails` | Boolean | Não | Se `true`, inclui detalhes de compliance de tags. |
| `ExcludeCompliantResources` | Boolean | Não | Se `true`, retorna apenas recursos não compliant. |
| `ResourceARNList` | Array | Não | Lista de ARNs específicos para consultar. |

#### Exemplo de Requisição

```json
{
  "TagFilters": [
    {
      "Key": "Environment",
      "Values": ["Production"]
    }
  ],
  "ResourceTypeFilters": ["ec2:instance", "rds:db"],
  "IncludeComplianceDetails": true
}
```

### 1.2 GetTagKeys

Lista todas as chaves de tag em uso na conta e região.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `PaginationToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

### 1.3 GetTagValues

Lista todos os valores para uma chave de tag específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Key` | String | Sim | A chave de tag para listar os valores. |
| `PaginationToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "Key": "Environment"
}
```

### 1.4 GetComplianceSummary

Retorna um resumo de compliance de tags, agrupado por dimensão.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TargetIdFilters` | Array | Não | IDs de contas para filtrar. |
| `RegionFilters` | Array | Não | Regiões para filtrar. |
| `ResourceTypeFilters` | Array | Não | Tipos de recurso para filtrar. |
| `TagKeyFilters` | Array | Não | Chaves de tag para filtrar. |
| `GroupBy` | Array | Não | Agrupar por: `TARGET_ID`, `REGION`, `RESOURCE_TYPE`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `PaginationToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "GroupBy": ["RESOURCE_TYPE"],
  "TagKeyFilters": ["CostCenter", "Environment"]
}
```

---

## 2. AWS Config API

O AWS Config permite avaliar, auditar e monitorar as configurações dos recursos AWS. Para FinOps, o inventário avançado e consultas SQL são particularmente úteis.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://config.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `StarlingDoveService` |
| **Service Name (IAM)** | `config` |

### 2.1 SelectResourceConfig

Executa uma consulta SQL sobre os recursos configurados. Esta é a ação mais poderosa para inventário.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Expression` | String | Sim | Consulta SQL. |
| `Limit` | Integer | Não | Número máximo de resultados (0-100). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplos de Consultas SQL

**Listar instâncias EC2 com tipo e tags:**
```json
{
  "Expression": "SELECT resourceId, resourceType, configuration.instanceType, tags WHERE resourceType = 'AWS::EC2::Instance'"
}
```

**Encontrar volumes EBS não anexados:**
```json
{
  "Expression": "SELECT resourceId, configuration.size, configuration.volumeType WHERE resourceType = 'AWS::EC2::Volume' AND configuration.state = 'available'"
}
```

**Listar recursos sem a tag CostCenter:**
```json
{
  "Expression": "SELECT resourceId, resourceType WHERE tags.tag('CostCenter') IS NULL"
}
```

### 2.2 GetDiscoveredResourceCounts

Retorna a contagem de recursos descobertos por tipo.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `resourceTypes` | Array | Não | Tipos de recurso para filtrar. |
| `limit` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

### 2.3 BatchGetResourceConfig

Retorna a configuração atual de um ou mais recursos.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `resourceKeys` | Array | Sim | Lista de objetos com `resourceType` e `resourceId`. Máximo 100. |

#### Exemplo de Requisição

```json
{
  "resourceKeys": [
    {
      "resourceType": "AWS::EC2::Instance",
      "resourceId": "i-1234567890abcdef0"
    },
    {
      "resourceType": "AWS::EC2::Volume",
      "resourceId": "vol-049df61146c4d7901"
    }
  ]
}
```
