# Guia Devastadoramente Detalhado: Tagging e AWS Config

## Visão Geral

Governança de tags e conformidade de configuração são pilares para um programa de FinOps maduro. A API de Resource Groups Tagging permite consultar recursos por tags em toda a sua conta, enquanto o AWS Config permite que você audite e avalie as configurações dos seus recursos AWS.

---

## Resource Groups Tagging API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://tagging.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `tag` |

### 1. GetResources

Retorna todos os recursos que correspondem a um filtro de tag. Esta é a chamada mais poderosa para encontrar recursos com base em sua taxonomia de tags.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `TagFilters` | Array de Objetos | Não | Filtra recursos com base em chaves e valores de tags. Cada objeto tem `Key` e `Values`. Você pode filtrar por existência de uma chave ou por um valor específico. |
| `ResourceTypeFilters` | Array de Strings | Não | Restringe a pesquisa a tipos de recursos específicos (ex: `ec2:instance`, `s3:bucket`). Se omitido, pesquisa em todos os tipos de recursos suportados. |
| `ResourcesPerPage` | Integer | Não | Número máximo de resultados por página. |
| `PaginationToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Encontrar todas as instâncias EC2 com a tag `Environment=Production`)

```json
{
  "ResourceTypeFilters": ["ec2:instance"],
  "TagFilters": [
    {
      "Key": "Environment",
      "Values": ["Production"]
    }
  ]
}
```

---

## AWS Config

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://config.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `config` |

### 1. SelectResourceConfig

Executa uma consulta SQL avançada contra os dados de inventário de recursos do AWS Config. Permite fazer perguntas complexas sobre a configuração dos seus recursos.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Expression` | String | **Sim** | A consulta SQL a ser executada. A sintaxe é semelhante ao SQL padrão, mas opera sobre os Itens de Configuração (CIs) do Config. Ex: `SELECT resourceId, resourceType, configuration.instanceType WHERE resourceType = 'AWS::EC2::Instance' AND configuration.instanceType = 't2.micro'`. |
| `Limit` | Integer | Não | O número máximo de resultados a serem retornados. |
| `NextToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Encontrar todos os volumes EBS que não estão criptografados)

```json
{
  "Expression": "SELECT resourceId, awsRegion, configuration.encrypted WHERE resourceType = 'AWS::EC2::Volume' AND configuration.encrypted = false"
}
```

### 2. GetDiscoveredResourceCounts

Retorna a contagem de recursos descobertos pelo AWS Config, opcionalmente agrupados por tipo de recurso.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `resourceTypes` | Array de Strings | Não | Uma lista de tipos de recursos para incluir na contagem. Se omitido, conta todos os tipos. |
| `groupByKey` | String | Não | A chave para agrupar os resultados. Use `resourceType` para ver a contagem por tipo de recurso. |
| `limit` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Contar recursos por tipo)

```json
{
  "groupByKey": "resourceType"
}
```

### 3. ListDiscoveredResources

Aceita um tipo de recurso e retorna uma lista de Itens de Configuração (CIs) para esse tipo de recurso.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `resourceType` | String | **Sim** | O tipo de recurso a ser listado (ex: `AWS::EC2::Instance`, `AWS::S3::Bucket`). |
| `resourceIds` | Array de Strings | Não | IDs de recursos específicos para listar. |
| `resourceName` | String | Não | O nome do recurso. |
| `limit` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Listar todas as instâncias EC2)

```json
{
  "resourceType": "AWS::EC2::Instance"
}
```
