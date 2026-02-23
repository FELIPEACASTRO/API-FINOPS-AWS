# Guia Detalhado: AWS BCM Data Exports API

## Visão Geral

O BCM Data Exports permite criar exportações personalizadas de múltiplos conjuntos de dados de billing e cost management para um bucket S3.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://bcm-data-exports.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `AWSBillingAndCostManagementDataExports` |
| **Service Name (IAM)** | `bcm-data-exports` |

---

## Ações da API

### 1. CreateExport

Cria uma nova exportação de dados de billing.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Export` | Object | Sim | Objeto que define a exportação. |
| `ResourceTags` | Array | Não | Tags a serem associadas à exportação. |

#### Objeto `Export`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Name` | String | Sim | Nome da exportação. |
| `DataQuery` | Object | Sim | Define a consulta de dados. |
| `DestinationConfigurations` | Object | Sim | Configuração do destino S3. |
| `Description` | String | Não | Descrição da exportação. |
| `RefreshCadence` | Object | Sim | Frequência de atualização. |

#### Exemplo de Requisição

```json
{
  "Export": {
    "Name": "CUR-2.0-Export",
    "DataQuery": {
      "QueryStatement": "SELECT * FROM COST_AND_USAGE_REPORT",
      "TableConfigurations": {
        "COST_AND_USAGE_REPORT": {
          "TIME_GRANULARITY": "HOURLY",
          "INCLUDE_RESOURCES": "TRUE",
          "INCLUDE_SPLIT_COST_ALLOCATION_DATA": "TRUE"
        }
      }
    },
    "DestinationConfigurations": {
      "S3Destination": {
        "S3Bucket": "my-bcm-exports-bucket",
        "S3Prefix": "bcm-exports/",
        "S3Region": "us-east-1",
        "S3OutputConfigurations": {
          "OutputType": "CUSTOM",
          "Format": "PARQUET",
          "Compression": "PARQUET",
          "Overwrite": "OVERWRITE_REPORT"
        }
      }
    },
    "RefreshCadence": {
      "Frequency": "SYNCHRONOUS"
    }
  }
}
```

---

### 2. ListExports

Lista todas as exportações configuradas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

---

### 3. GetExport

Retorna detalhes de uma exportação específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ExportArn` | String | Sim | ARN da exportação. |

#### Exemplo de Requisição

```json
{
  "ExportArn": "arn:aws:bcm-data-exports:us-east-1:123456789012:export/abc-123"
}
```

---

### 4. ListTables

Lista as tabelas de dados disponíveis para exportação.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

---

### 5. ListExecutions

Lista as execuções de uma exportação.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ExportArn` | String | Sim | ARN da exportação. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ExportArn": "arn:aws:bcm-data-exports:us-east-1:123456789012:export/abc-123"
}
```
