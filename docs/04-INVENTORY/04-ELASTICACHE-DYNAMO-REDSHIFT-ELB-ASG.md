# Guia Detalhado: ElastiCache, DynamoDB, Redshift, ELB e Auto Scaling APIs (FinOps)

---

## 1. Amazon ElastiCache API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticache.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `elasticache` |
| **Versão da API** | `2015-02-02` |

### 1.1 DescribeCacheClusters

Lista todos os clusters ElastiCache com informações de nós.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `CacheClusterId` | String | Não | ID de um cluster específico. |
| `ShowCacheNodeInfo` | Boolean | Não | Se `true`, retorna informações detalhadas de cada nó. |
| `ShowCacheClustersNotInReplicationGroups` | Boolean | Não | Se `true`, mostra apenas clusters standalone. |
| `MaxRecords` | Integer | Não | Número máximo de resultados (20-100). |
| `Marker` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeCacheClusters
&Version=2015-02-02
&ShowCacheNodeInfo=true
```

### 1.2 DescribeReservedCacheNodes

Lista as Reserved Cache Nodes ativas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReservedCacheNodeId` | String | Não | ID de uma reserva específica. |
| `CacheNodeType` | String | Não | Filtro por tipo de nó. |
| `Duration` | String | Não | Duração em segundos. |
| `ProductDescription` | String | Não | Descrição do produto. |
| `OfferingType` | String | Não | Tipo de oferta. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeReservedCacheNodes
&Version=2015-02-02
```

---

## 2. Amazon DynamoDB API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://dynamodb.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `DynamoDB_20120810` |
| **Service Name (IAM)** | `dynamodb` |

### 2.1 ListTables

Lista todas as tabelas DynamoDB.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ExclusiveStartTableName` | String | Não | Nome da tabela a partir da qual continuar a listagem. |
| `Limit` | Integer | Não | Número máximo de resultados (1-100). |

#### Exemplo de Requisição

```json
{
  "Limit": 100
}
```

### 2.2 DescribeTable

Retorna detalhes de uma tabela (capacidade, tamanho, modo de billing).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `TableName` | String | Sim | Nome da tabela. |

#### Exemplo de Requisição

```json
{
  "TableName": "my-table"
}
```

### 2.3 DescribeReservedCapacity

Lista a capacidade reservada ativa do DynamoDB.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReservedCapacityId` | String | Não | ID de uma reserva específica. |

#### Exemplo de Requisição

```json
{}
```

---

## 3. Amazon Redshift API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://redshift.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `redshift` |
| **Versão da API** | `2012-12-01` |

### 3.1 DescribeClusters

Lista todos os clusters Redshift.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ClusterIdentifier` | String | Não | Identificador de um cluster específico. |
| `MaxRecords` | Integer | Não | Número máximo de resultados (20-100). |
| `Marker` | String | Não | Token para paginação. |
| `TagKeys` | Array | Não | Filtro por chaves de tag. |
| `TagValues` | Array | Não | Filtro por valores de tag. |

#### Exemplo de Requisição

```
Action=DescribeClusters
&Version=2012-12-01
```

### 3.2 DescribeReservedNodes

Lista as Reserved Nodes ativas do Redshift.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReservedNodeId` | String | Não | ID de uma reserva específica. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeReservedNodes
&Version=2012-12-01
```

---

## 4. Elastic Load Balancing API (v2)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticloadbalancing.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `elasticloadbalancing` |
| **Versão da API** | `2015-12-01` |

### 4.1 DescribeLoadBalancers

Lista todos os ALBs, NLBs e GLBs.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `LoadBalancerArns` | Array | Não | ARNs de LBs específicos. |
| `Names` | Array | Não | Nomes de LBs específicos. |
| `Marker` | String | Não | Token para paginação. |
| `PageSize` | Integer | Não | Número máximo de resultados (1-400). |

#### Exemplo de Requisição

```
Action=DescribeLoadBalancers
&Version=2015-12-01
```

### 4.2 DescribeTargetGroups

Lista todos os Target Groups.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `LoadBalancerArn` | String | Não | Filtro por LB. |
| `TargetGroupArns` | Array | Não | ARNs de TGs específicos. |
| `Names` | Array | Não | Nomes de TGs específicos. |
| `Marker` | String | Não | Token para paginação. |
| `PageSize` | Integer | Não | Número máximo de resultados. |

#### Exemplo de Requisição

```
Action=DescribeTargetGroups
&Version=2015-12-01
```

---

## 5. Auto Scaling API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://autoscaling.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `autoscaling` |
| **Versão da API** | `2011-01-01` |

### 5.1 DescribeAutoScalingGroups

Lista todos os grupos de Auto Scaling (min, max, desired).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AutoScalingGroupNames` | Array | Não | Nomes de ASGs específicos. |
| `Filters` | Array | Não | Filtros por tag. |
| `MaxRecords` | Integer | Não | Número máximo de resultados (1-100). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeAutoScalingGroups
&Version=2011-01-01
&MaxRecords=100
```

### 5.2 DescribePolicies

Lista as políticas de escalabilidade.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AutoScalingGroupName` | String | Não | Nome do ASG. |
| `PolicyNames` | Array | Não | Nomes de políticas específicas. |
| `PolicyTypes` | Array | Não | Tipos de política. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribePolicies
&Version=2011-01-01
&AutoScalingGroupName=my-asg
```
