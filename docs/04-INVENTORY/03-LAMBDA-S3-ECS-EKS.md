# Guia Devastadoramente Detalhado: Inventário de Lambda, S3, ECS e EKS

## Visão Geral

Este guia cobre as chamadas de API essenciais para inventariar recursos em serviços de computação serverless, contêineres e armazenamento, que são fundamentais para uma visão completa do seu ambiente na AWS.

---

## AWS Lambda

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://lambda.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `lambda` |

### 1. ListFunctions

Retorna uma lista das suas funções Lambda, incluindo informações de configuração como tamanho da memória, runtime e última modificação.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MasterRegion` | String | Não | Para funções em regiões replicadas, especifica a região principal. Geralmente não é necessário para a maioria dos casos de uso de inventário. |
| `FunctionVersion` | String | Não | Use `ALL` para retornar todas as versões de todas as funções. Se omitido, retorna apenas a versão `$LATEST` de cada função. Listar todas as versões pode ser útil para entender o histórico, mas aumenta o volume de dados. |
| `Marker` | String | Não | Token de paginação para obter a próxima página de resultados se a lista for muito longa. |
| `MaxItems` | Integer | Não | Número máximo de resultados por página. |

#### Exemplo de Requisição (Listar todas as funções)

```
GET /2015-03-31/functions/
```

---

## Amazon S3

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://s3.{region}.amazonaws.com` |
| **Protocolo** | REST (XML) |
| **Service Name (IAM)** | `s3` |

### 1. ListBuckets

Retorna uma lista de todos os buckets S3 que pertencem à conta que fez a chamada.

#### Parâmetros de Entrada

Nenhum.

#### Exemplo de Requisição

```
GET / HTTP/1.1
Host: s3.amazonaws.com
```

### 2. GetBucketLifecycleConfiguration

Retorna a configuração do ciclo de vida de um bucket, que define como os objetos são transicionados para classes de armazenamento mais baratas (ex: Standard-IA, Glacier) ou expirados. Essencial para otimização de custos de armazenamento.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Bucket` | String | **Sim** | O nome do bucket cuja configuração de ciclo de vida você deseja inspecionar. |
| `ExpectedBucketOwner` | String | Não | O ID da conta do proprietário esperado do bucket. Usado para validação em cenários de acesso complexos. |

#### Exemplo de Requisição

```
GET /?lifecycle HTTP/1.1
Host: <BucketName>.s3.<Region>.amazonaws.com
```

---

## Amazon ECS (Elastic Container Service)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ecs.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `ecs` |

### 1. ListClusters / DescribeClusters

`ListClusters` retorna uma lista de ARNs de clusters. `DescribeClusters` usa esses ARNs para retornar informações detalhadas sobre cada cluster.

#### Parâmetros de `DescribeClusters`

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `clusters` | Array de Strings | **Sim** | Uma lista de ARNs ou nomes curtos dos clusters a serem descritos. Você pode descrever até 100 clusters por chamada. |
| `include` | Array de Strings | Não | Permite incluir informações adicionais. Para FinOps, `TAGS` é o mais importante para associar custos a projetos ou equipes. `STATISTICS` pode fornecer contagens de tarefas em execução. |

#### Exemplo de Requisição

```json
{
   "clusters": ["arn:aws:ecs:region:aws_account_id:cluster/MyCluster"],
   "include": ["TAGS", "STATISTICS"]
}
```

### 2. ListServices / DescribeServices

Similarmente, `ListServices` retorna os ARNs dos serviços dentro de um cluster, e `DescribeServices` fornece os detalhes.

#### Parâmetros de `DescribeServices`

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `cluster` | String | Não | O nome ou ARN do cluster. Se omitido, o cluster `default` é usado. É uma boa prática sempre especificar o cluster. |
| `services` | Array de Strings | **Sim** | A lista de ARNs ou nomes curtos dos serviços a serem descritos (até 10 por chamada). |
| `include` | Array de Strings | Não | `TAGS` para obter as tags associadas ao serviço. |

#### Exemplo de Requisição

```json
{
   "cluster": "MyCluster",
   "services": ["MyService"],
   "include": ["TAGS"]
}
```

---

## Amazon EKS (Elastic Kubernetes Service)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://eks.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `eks` |

### 1. ListClusters

Retorna uma lista dos nomes de todos os seus clusters EKS.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `maxResults` | Integer | Não | Número máximo de resultados por página. |
| `nextToken` | String | Não | Token de paginação. |
| `include` | Array de Strings | Não | Use `["all"]` para incluir clusters em qualquer estado (criando, deletando, etc.). Por padrão, retorna apenas clusters `ACTIVE`. |

#### Exemplo de Requisição

```
GET /clusters
```

### 2. DescribeCluster

Retorna informações detalhadas sobre um cluster EKS específico, incluindo sua versão do Kubernetes, status e configuração de rede.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `name` | String | **Sim** | O nome do cluster que você deseja descrever. |

#### Exemplo de Requisição

```
GET /clusters/MyCluster
```

### 3. ListNodegroups / DescribeNodegroup

`ListNodegroups` retorna os grupos de nós gerenciados de um cluster. `DescribeNodegroup` retorna os detalhes de um grupo de nós, incluindo tipo de instância, configuração de auto-scaling e versão do AMI.

#### Parâmetros de `DescribeNodegroup`

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `clusterName` | String | **Sim** | O nome do cluster pai do grupo de nós. |
| `nodegroupName` | String | **Sim** | O nome do grupo de nós a ser descrito. |

#### Exemplo de Requisição

```
GET /clusters/MyCluster/nodegroups/MyNodegroup
```
