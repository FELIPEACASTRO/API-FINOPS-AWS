# Guia Devastadoramente Detalhado: Inventário de Outros Serviços Relevantes

## Visão Geral

Este guia finaliza a seção de inventário cobrindo serviços de caching, banco de dados NoSQL, data warehousing e balanceamento de carga. Embora possam não ser tão onipresentes quanto o EC2, esses serviços podem representar custos significativos e otimizações importantes.

---

## Amazon ElastiCache

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticache.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `elasticache` |

### 1. DescribeCacheClusters

Retorna informações sobre clusters de cache provisionados (Redis ou Memcached).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `CacheClusterId` | String | Não | O ID de um cluster de cache específico. Se omitido, retorna todos os clusters. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token de paginação. |
| `ShowCacheNodeInfo` | Boolean | Não | Se `true`, inclui informações detalhadas sobre cada nó no cluster. Útil para verificar o estado individual dos nós. |

#### Exemplo de Requisição (Listar todos os clusters)

```json
{}
```

---

## Amazon DynamoDB

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://dynamodb.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `dynamodb` |

### 1. ListTables

Retorna uma lista dos nomes de todas as suas tabelas DynamoDB.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ExclusiveStartTableName` | String | Não | O nome da tabela a partir da qual continuar uma lista paginada. Usado para paginação. |
| `Limit` | Integer | Não | Número máximo de nomes de tabela a serem retornados por página. |

#### Exemplo de Requisição

```json
{}
```

### 2. DescribeTable

Retorna informações detalhadas sobre uma tabela, incluindo seu status, esquema de chave, índices e, mais importante para FinOps, o `BillingModeSummary` (On-Demand ou Provisioned) e as `ProvisionedThroughput` (RCUs/WCUs provisionadas).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `TableName` | String | **Sim** | O nome da tabela a ser descrita. |

#### Exemplo de Requisição

```json
{
  "TableName": "MyDataTable"
}
```

---

## Amazon Redshift

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://redshift.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `redshift` |

### 1. DescribeClusters

Retorna propriedades de clusters provisionados do Amazon Redshift.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ClusterIdentifier` | String | Não | O identificador de um cluster específico. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token de paginação. |
| `TagKeys` / `TagValues` | Array de Strings | Não | Filtra clusters com base em chaves e/ou valores de tags. Essencial para inventário por projeto. |

#### Exemplo de Requisição (Listar todos os clusters)

```json
{}
```

---

## Elastic Load Balancing (ELB)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticloadbalancing.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `elasticloadbalancing` |

### 1. DescribeLoadBalancers

Retorna informações sobre seus Application Load Balancers (ALB), Network Load Balancers (NLB) e Gateway Load Balancers (GWLB). A API para Classic Load Balancers é separada e mais antiga.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `LoadBalancerArns` | Array de Strings | Não | ARNs de load balancers específicos. |
| `Names` | Array de Strings | Não | Nomes de load balancers específicos. |
| `Marker` | String | Não | Token de paginação. |
| `PageSize` | Integer | Não | Tamanho da página. |

#### Exemplo de Requisição

```json
{}
```

---

## EC2 Auto Scaling

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://autoscaling.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `autoscaling` |

### 1. DescribeAutoScalingGroups

Retorna informações sobre seus grupos de Auto Scaling, incluindo o número desejado, mínimo e máximo de instâncias, e os tipos de instância usados. Crucial para entender a elasticidade e o custo potencial de seus workloads.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AutoScalingGroupNames` | Array de Strings | Não | Nomes de grupos de Auto Scaling específicos. |
| `Filters` | Array de Objetos | Não | Filtra por tag. Ex: `Name: "tag-key", Values: ["MyTagKey"]`. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Listar todos os grupos)

```json
{}
```
