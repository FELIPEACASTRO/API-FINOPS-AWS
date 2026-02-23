# Guia Detalhado: Lambda, S3, ECS e EKS APIs (FinOps)

---

## 1. AWS Lambda API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://lambda.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `lambda` |

### 1.1 ListFunctions

Lista todas as funções Lambda.

**Método HTTP**: GET  
**Path**: `/2015-03-31/functions`

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MasterRegion` | String | Não | Região para listar funções replicadas. |
| `FunctionVersion` | String | Não | `ALL` para incluir todas as versões. |
| `Marker` | String | Não | Token para paginação. |
| `MaxItems` | Integer | Não | Número máximo de resultados (1-10000). |

### 1.2 GetFunction

Retorna detalhes de uma função (memória, runtime, timeout, etc.).

**Método HTTP**: GET  
**Path**: `/2015-03-31/functions/{FunctionName}`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `FunctionName` | String | Sim | Nome ou ARN da função. |

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Qualifier` | String | Não | Versão ou alias da função. |

### 1.3 GetAccountSettings

Retorna limites e configurações da conta para Lambda.

**Método HTTP**: GET  
**Path**: `/2016-08-19/account-settings`

Nenhum parâmetro necessário.

---

## 2. Amazon S3 API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://s3.{region}.amazonaws.com/` |
| **Protocolo** | REST (XML) |
| **Service Name (IAM)** | `s3` |

### 2.1 ListBuckets

Lista todos os buckets S3 da conta.

**Método HTTP**: GET  
**Path**: `/`

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `bucket-region` | String | Não | Filtro por região. |
| `continuation-token` | String | Não | Token para paginação. |
| `max-buckets` | Integer | Não | Número máximo de resultados. |
| `prefix` | String | Não | Filtro por prefixo de nome. |

### 2.2 GetBucketTagging

Retorna as tags de um bucket.

**Método HTTP**: GET  
**Path**: `/{BucketName}?tagging`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `BucketName` | String | Sim | Nome do bucket. |

---

## 3. Amazon ECS API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ecs.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AmazonEC2ContainerServiceV20141113` |
| **Service Name (IAM)** | `ecs` |

### 3.1 ListClusters

Lista todos os clusters ECS.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `maxResults` | Integer | Não | Número máximo de resultados (1-100). |
| `nextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "maxResults": 100
}
```

### 3.2 DescribeClusters

Retorna detalhes de clusters.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `clusters` | Array | Sim | Lista de nomes ou ARNs de clusters. |
| `include` | Array | Não | Informações adicionais: `ATTACHMENTS`, `CONFIGURATIONS`, `SETTINGS`, `STATISTICS`, `TAGS`. |

#### Exemplo de Requisição

```json
{
  "clusters": ["my-cluster"],
  "include": ["STATISTICS", "TAGS"]
}
```

### 3.3 ListServices

Lista os serviços em um cluster.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `cluster` | String | Não | Nome ou ARN do cluster. |
| `maxResults` | Integer | Não | Número máximo de resultados. |
| `nextToken` | String | Não | Token para paginação. |
| `launchType` | String | Não | `EC2`, `FARGATE`, `EXTERNAL`. |
| `schedulingStrategy` | String | Não | `REPLICA` ou `DAEMON`. |

#### Exemplo de Requisição

```json
{
  "cluster": "my-cluster",
  "launchType": "FARGATE"
}
```

### 3.4 DescribeServices

Retorna detalhes de serviços (desired/running count, CPU/memória).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `services` | Array | Sim | Lista de nomes ou ARNs de serviços (máximo 10). |
| `cluster` | String | Não | Nome ou ARN do cluster. |
| `include` | Array | Não | `TAGS`. |

#### Exemplo de Requisição

```json
{
  "services": ["my-service"],
  "cluster": "my-cluster",
  "include": ["TAGS"]
}
```

---

## 4. Amazon EKS API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://eks.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `eks` |

### 4.1 ListClusters

Lista todos os clusters EKS.

**Método HTTP**: GET  
**Path**: `/clusters`

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `maxResults` | Integer | Não | Número máximo de resultados (1-100). |
| `nextToken` | String | Não | Token para paginação. |
| `include` | Array | Não | Incluir clusters de outros tipos. |

### 4.2 DescribeCluster

Retorna detalhes de um cluster EKS.

**Método HTTP**: GET  
**Path**: `/clusters/{name}`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `name` | String | Sim | Nome do cluster. |

### 4.3 ListNodegroups

Lista os node groups de um cluster.

**Método HTTP**: GET  
**Path**: `/clusters/{name}/node-groups`

#### Parâmetros

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `name` | String (path) | Sim | Nome do cluster. |
| `maxResults` | Integer (query) | Não | Número máximo de resultados. |
| `nextToken` | String (query) | Não | Token para paginação. |

### 4.4 DescribeNodegroup

Retorna detalhes de um node group (tipo de instância, scaling config, etc.).

**Método HTTP**: GET  
**Path**: `/clusters/{name}/node-groups/{nodegroupName}`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `name` | String | Sim | Nome do cluster. |
| `nodegroupName` | String | Sim | Nome do node group. |
