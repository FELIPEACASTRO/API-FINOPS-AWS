# Guia Devastadoramente Detalhado: Amazon RDS API (para FinOps)

## Visão Geral

Bancos de dados são frequentemente uma parcela significativa dos custos na nuvem. A API do RDS é vital para inventariar instâncias de banco de dados, clusters, snapshots e RIs, permitindo a identificação de recursos subutilizados, desalocados ou superdimensionados.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://rds.{region}.amazonaws.com` |
| **Protocolo** | Query (GET/POST) |
| **Service Name (IAM)** | `rds` |

---

## 1. DescribeDBInstances

Retorna informações sobre instâncias de banco de dados provisionadas.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `DBInstanceIdentifier` | String | Não | O identificador de uma instância de banco de dados específica. Se omitido, retorna todas as instâncias. |
| `Filters` | Array de Objetos | Não | Filtra os resultados. Cada objeto tem `Name` e `Values`.<br>**Nomes de Filtro Úteis**: `db-instance-id`, `db-instance-class`, `engine`, `tag:<key>`.<br>**Uso**: Permite focar em um subconjunto de instâncias, como todas as instâncias MySQL ou todas as instâncias de um projeto específico via tags. |
| `MaxRecords` | Integer | Não | O número máximo de registros a serem retornados em uma única chamada. |
| `Marker` | String | Não | Um token de paginação fornecido em uma resposta anterior para obter a próxima página de resultados. |

### Exemplo de Requisição (Listar todas as instâncias de banco de dados MySQL)

```json
{
  "Filters": [
    {
      "Name": "engine",
      "Values": ["mysql"]
    }
  ]
}
```

---

## 2. DescribeDBClusters

Retorna informações sobre clusters de banco de dados provisionados (usado para Aurora, Multi-AZ, etc.).

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `DBClusterIdentifier` | String | Não | O identificador de um cluster específico. |
| `Filters` | Array de Objetos | Não | Filtra os resultados. `Name` e `Values`.<br>**Nomes de Filtro Úteis**: `db-cluster-id`, `engine` (ex: `aurora-mysql`), `status` (`available`, `creating`, `stopped`).<br>**Uso**: Filtrar por `status: stopped` pode ajudar a encontrar clusters parados que ainda podem estar incorrendo em custos de armazenamento. |
| `MaxRecords` | Integer | Não | Número máximo de registros. |
| `Marker` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar todos os clusters Aurora MySQL parados)

```json
{
  "Filters": [
    {
      "Name": "engine",
      "Values": ["aurora-mysql"]
    },
    {
      "Name": "status",
      "Values": ["stopped"]
    }
  ]
}
```

---

## 3. DescribeDBSnapshots

Retorna informações sobre snapshots de banco de dados. Snapshots manuais, em particular, podem se acumular e gerar custos se não forem gerenciados.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `DBInstanceIdentifier` | String | Não | Filtra snapshots de uma instância específica. |
| `DBSnapshotIdentifier` | String | Não | O identificador de um snapshot específico. |
| `SnapshotType` | String | Não | O tipo de snapshot.<br>**Valores**: `manual`, `automated`, `shared`, `public`.<br>**Uso**: Filtrar por `manual` é crucial para encontrar snapshots que não são gerenciados automaticamente pelo RDS e que podem ser candidatos à exclusão se forem antigos. |
| `Filters` | Array de Objetos | Não | Filtros adicionais, como por `engine` ou `status`. |
| `MaxRecords` | Integer | Não | Número máximo de registros. |
| `Marker` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar todos os snapshots manuais)

```json
{
  "SnapshotType": "manual"
}
```

---

## 4. DescribeReservedDBInstances

Retorna informações sobre as Reserved Instances (RIs) de banco de dados que você possui.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ReservedDBInstanceId` | String | Não | O ID de uma RI específica. |
| `Filters` | Array de Objetos | Não | Filtra as RIs.<br>**Nomes de Filtro Úteis**: `state` (`payment-pending`, `active`, `payment-failed`, `retired`), `db-instance-class`, `duration` (em segundos, ex: `31536000` para 1 ano).<br>**Uso**: Use `state: active` para inventariar seus compromissos de RI de banco de dados atuais. |
| `MaxRecords` | Integer | Não | Número máximo de registros. |
| `Marker` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar todas as RIs de RDS ativas)

```json
{
  "Filters": [
    {
      "Name": "state",
      "Values": ["active"]
    }
  ]
}
```
