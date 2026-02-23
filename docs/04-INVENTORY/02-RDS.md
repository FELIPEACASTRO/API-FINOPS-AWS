# Guia Detalhado: Amazon RDS API (FinOps)

## Visão Geral

O RDS é outro serviço com custos significativos. Monitorar instâncias de banco de dados, clusters Aurora e snapshots é essencial para right-sizing e otimização.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://rds.{region}.amazonaws.com/` |
| **Protocolo** | Query API (form-urlencoded) |
| **Service Name (IAM)** | `rds` |
| **Versão da API** | `2014-10-31` |

---

## Ações da API

### 1. DescribeDBInstances

Lista todas as instâncias RDS com detalhes completos.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `DBInstanceIdentifier` | String | Não | Identificador de uma instância específica. |
| `Filters` | Array | Não | Filtros por `db-cluster-id`, `db-instance-id`, `dbi-resource-id`, `domain`, `engine`. |
| `MaxRecords` | Integer | Não | Número máximo de resultados (20-100). |
| `Marker` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeDBInstances
&Version=2014-10-31
&MaxRecords=100
```

---

### 2. DescribeDBClusters

Lista clusters Aurora/RDS com detalhes.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `DBClusterIdentifier` | String | Não | Identificador de um cluster específico. |
| `Filters` | Array | Não | Filtros por `db-cluster-id`, `db-cluster-resource-id`, `domain`, `engine`. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token para paginação. |
| `IncludeShared` | Boolean | Não | Incluir clusters compartilhados. |

#### Exemplo de Requisição

```
Action=DescribeDBClusters
&Version=2014-10-31
```

---

### 3. DescribeReservedDBInstances

Lista as Reserved DB Instances ativas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReservedDBInstanceId` | String | Não | ID de uma RI específica. |
| `DBInstanceClass` | String | Não | Filtro por classe de instância. |
| `Duration` | String | Não | Duração em segundos. |
| `ProductDescription` | String | Não | Descrição do produto. |
| `OfferingType` | String | Não | Tipo de oferta. |
| `MultiAZ` | Boolean | Não | Filtro por Multi-AZ. |
| `LeaseId` | String | Não | ID do lease. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```
Action=DescribeReservedDBInstances
&Version=2014-10-31
```

---

### 4. DescribeDBSnapshots

Lista snapshots de banco de dados.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `DBInstanceIdentifier` | String | Não | Filtro por instância. |
| `DBSnapshotIdentifier` | String | Não | ID de um snapshot específico. |
| `SnapshotType` | String | Não | `automated`, `manual`, `shared`, `public`, `awsbackup`. |
| `Filters` | Array | Não | Filtros adicionais. |
| `MaxRecords` | Integer | Não | Número máximo de resultados. |
| `Marker` | String | Não | Token para paginação. |
| `IncludeShared` | Boolean | Não | Incluir snapshots compartilhados. |
| `IncludePublic` | Boolean | Não | Incluir snapshots públicos. |

#### Exemplo de Requisição

```
Action=DescribeDBSnapshots
&Version=2014-10-31
&SnapshotType=manual
```
