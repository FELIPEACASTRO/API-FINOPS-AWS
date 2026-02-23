# 4. APIs de Inventário de Recursos

Para uma prática de FinOps eficaz, é essencial ter um inventário completo de todos os recursos provisionados. Este capítulo cobre as APIs dos principais serviços AWS para listar e descrever recursos que geram custos.

## 4.1 Amazon EC2

O EC2 é tipicamente o maior gerador de custos na AWS. Monitorar instâncias, volumes EBS, snapshots e IPs elásticos é fundamental.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ec2.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `ec2` |
| **Versão da API** | `2016-11-15` |

### Ações Relevantes para FinOps

| Ação | Descrição | Relevância FinOps |
| :--- | :--- | :--- |
| `DescribeInstances` | Lista todas as instâncias EC2. | Inventário principal, identificar instâncias ociosas. |
| `DescribeVolumes` | Lista todos os volumes EBS. | Identificar volumes não anexados ou subutilizados. |
| `DescribeSnapshots` | Lista snapshots EBS. | Identificar snapshots antigos ou desnecessários. |
| `DescribeAddresses` | Lista Elastic IPs. | IPs não associados geram custo ($0.005/hora). |
| `DescribeNatGateways` | Lista NAT Gateways. | Recurso com custo significativo ($0.045/hora + dados). |
| `DescribeReservedInstances` | Lista RIs ativas. | Monitorar compromissos de reserva. |
| `DescribeReservedInstancesOfferings` | Lista ofertas de RIs. | Avaliar opções de compra. |
| `DescribeSpotInstanceRequests` | Lista Spot Instances. | Monitorar uso de instâncias spot. |

### Formato da Requisição EC2

Todas as requisições EC2 usam o formato Query API (form-urlencoded):

```
POST / HTTP/1.1
Host: ec2.us-east-1.amazonaws.com
Content-Type: application/x-www-form-urlencoded

Action=DescribeInstances&Version=2016-11-15
```

---

## 4.2 Amazon RDS

O RDS é outro serviço com custos significativos. Monitorar instâncias de banco de dados, clusters Aurora e snapshots é essencial.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://rds.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `rds` |
| **Versão da API** | `2014-10-31` |

### Ações Relevantes para FinOps

| Ação | Descrição | Relevância FinOps |
| :--- | :--- | :--- |
| `DescribeDBInstances` | Lista todas as instâncias RDS. | Inventário, identificar instâncias ociosas. |
| `DescribeDBClusters` | Lista clusters Aurora/RDS. | Monitorar clusters e réplicas. |
| `DescribeDBSnapshots` | Lista snapshots de banco de dados. | Identificar snapshots antigos. |
| `DescribeReservedDBInstances` | Lista RIs de RDS ativas. | Monitorar compromissos. |
| `DescribeReservedDBInstancesOfferings` | Lista ofertas de RIs RDS. | Avaliar opções de compra. |

---

## 4.3 Amazon S3

O S3 é um dos serviços mais utilizados e seus custos podem crescer significativamente com o volume de dados armazenados.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://s3.{region}.amazonaws.com/` |
| **Protocolo** | REST (XML) |
| **Service Name (IAM)** | `s3` |

### Ações Relevantes para FinOps

| Ação | Descrição | Relevância FinOps |
| :--- | :--- | :--- |
| `ListBuckets` | Lista todos os buckets S3. | Inventário básico de armazenamento. |
| `GetBucketMetricsConfiguration` | Obtém métricas de um bucket. | Análise de padrões de acesso. |

Para análise avançada de armazenamento S3, utilize o **S3 Storage Lens** (via `s3control`), que fornece métricas organizacionais de uso e atividade.

---

## 4.4 AWS Lambda

O Lambda é cobrado por número de invocações e duração de execução. Monitorar funções é importante para otimizar memória e identificar funções não utilizadas.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://lambda.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `lambda` |

### Ações Relevantes para FinOps

| Ação | Método | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `ListFunctions` | GET | `/2015-03-31/functions` | Lista todas as funções Lambda. |
| `GetFunction` | GET | `/2015-03-31/functions/{name}` | Retorna detalhes de uma função (memória, runtime, etc.). |
| `GetAccountSettings` | GET | `/2016-08-19/account-settings` | Retorna limites e configurações da conta para Lambda. |

---

## 4.5 Amazon ECS

O ECS é cobrado pelos recursos subjacentes (EC2 ou Fargate). Monitorar clusters, serviços e tarefas é essencial para otimização.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ecs.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AmazonEC2ContainerServiceV20141113` |
| **Service Name (IAM)** | `ecs` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `ListClusters` | Lista todos os clusters ECS. |
| `DescribeClusters` | Retorna detalhes de clusters (com estatísticas). |
| `ListServices` | Lista os serviços em um cluster. |
| `DescribeServices` | Retorna detalhes de serviços (desired/running count). |
| `ListTasks` | Lista as tarefas em execução. |
| `DescribeTasks` | Retorna detalhes de tarefas (CPU/memória alocada). |

---

## 4.6 Amazon EKS

O EKS cobra uma taxa fixa por cluster ($0.10/hora) mais os recursos de computação (EC2 ou Fargate).

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://eks.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `eks` |

### Ações Relevantes para FinOps

| Ação | Método | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `ListClusters` | GET | `/clusters` | Lista todos os clusters EKS. |
| `DescribeCluster` | GET | `/clusters/{name}` | Retorna detalhes de um cluster. |
| `ListNodegroups` | GET | `/clusters/{name}/node-groups` | Lista os node groups de um cluster. |
| `DescribeNodegroup` | GET | `/clusters/{name}/node-groups/{ng}` | Retorna detalhes de um node group. |

---

## 4.7 Amazon ElastiCache

O ElastiCache é cobrado por nó de cache. Monitorar clusters e nós é importante para right-sizing.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticache.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `elasticache` |
| **Versão da API** | `2015-02-02` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeCacheClusters` | Lista todos os clusters ElastiCache com informações de nós. |
| `DescribeReservedCacheNodes` | Lista as Reserved Cache Nodes ativas. |
| `DescribeReservedCacheNodesOfferings` | Lista ofertas de Reserved Cache Nodes. |

---

## 4.8 Amazon DynamoDB

O DynamoDB é cobrado por capacidade provisionada ou por requisição (modo on-demand). Monitorar tabelas e sua capacidade é essencial.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://dynamodb.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `DynamoDB_20120810` |
| **Service Name (IAM)** | `dynamodb` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `ListTables` | Lista todas as tabelas DynamoDB. |
| `DescribeTable` | Retorna detalhes de uma tabela (capacidade, tamanho, modo de billing). |
| `DescribeReservedCapacity` | Lista a capacidade reservada ativa. |
| `DescribeReservedCapacityOfferings` | Lista ofertas de capacidade reservada. |

---

## 4.9 Amazon Redshift

O Redshift é cobrado por nó de cluster. Monitorar clusters e nós reservados é importante.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://redshift.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `redshift` |
| **Versão da API** | `2012-12-01` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeClusters` | Lista todos os clusters Redshift. |
| `DescribeReservedNodes` | Lista as Reserved Nodes ativas. |
| `DescribeReservedNodeOfferings` | Lista ofertas de Reserved Nodes. |

---

## 4.10 Elastic Load Balancing

Load balancers são cobrados por hora e por unidade de capacidade (LCU). Identificar load balancers ociosos é uma oportunidade de economia.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://elasticloadbalancing.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `elasticloadbalancing` |
| **Versão da API** | `2015-12-01` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeLoadBalancers` | Lista todos os ALBs, NLBs e GLBs. |
| `DescribeTargetGroups` | Lista todos os Target Groups. |

---

## 4.11 Auto Scaling

Monitorar grupos de Auto Scaling é importante para entender a capacidade provisionada e identificar oportunidades de otimização.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://autoscaling.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `autoscaling` |
| **Versão da API** | `2011-01-01` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeAutoScalingGroups` | Lista todos os grupos de Auto Scaling (min, max, desired). |
| `DescribePolicies` | Lista as políticas de escalabilidade. |

## Referências

- [EC2 API Reference](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/Welcome.html)
- [RDS API Reference](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/Welcome.html)
- [Lambda API Reference](https://docs.aws.amazon.com/lambda/latest/api/Welcome.html)
- [ECS API Reference](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/Welcome.html)
- [EKS API Reference](https://docs.aws.amazon.com/eks/latest/APIReference/Welcome.html)
