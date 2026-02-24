># Guia Devastadoramente Detalhado: BCM Data Exports API

## Visão Geral

A API BCM Data Exports é a evolução do CUR. Ela oferece uma maneira mais moderna e flexível de exportar dados de custo e uso, incluindo a capacidade de criar exportações com base em consultas SQL personalizadas. É a direção futura para exportação de dados de faturamento na AWS.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://bcm-data-exports.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `AWSBillingAndCostManagementDataExports` |
| **Service Name (IAM)** | `bcm-data-exports` |

---

## 1. CreateExport

Cria uma nova configuração de exportação de dados.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Export` | Objeto | **Sim** | O objeto principal que define a exportação. Veja a tabela detalhada abaixo. |

#### O Objeto `Export`

| Chave | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Name` | String | **Sim** | O nome da sua exportação. |
| `Description` | String | Não | Uma descrição opcional para a exportação. |
| `DataQuery` | Objeto | **Sim** | Define a consulta que gera os dados. Contém:<br>- `QueryStatement` (String, **Sim**): A consulta SQL para selecionar os dados. Ex: `SELECT * FROM COST_AND_USAGE_REPORT`.<br>- `TableConfigurations` (Objeto, Não): Permite configurar parâmetros para as tabelas na consulta. |
| `DestinationConfigurations` | Objeto | **Sim** | Define para onde os dados serão enviados. Contém:<br>- `S3Destination` (Objeto, **Sim**): Configura a entrega para um bucket S3. |
| `RefreshCadence` | Objeto | **Sim** | Define a frequência de atualização dos dados. Contém:<br>- `Frequency` (String, **Sim**): `SYNCHRONOUS` (atualiza sempre que há novos dados). |

#### O Objeto `S3Destination`

| Chave | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `S3Bucket` | String | **Sim** | O nome do bucket S3 de destino. |
| `S3Prefix` | String | **Sim** | O prefixo (subpasta) dentro do bucket. |
| `S3Region` | String | **Sim** | A região do bucket S3. |
| `S3OutputConfigurations` | Objeto | **Sim** | Define o formato do arquivo de saída. Contém `OutputType` (`CUSTOM`), `Format` (`PARQUET` ou `CSV`), `Compression` (`PARQUET` ou `GZIP`), e `Overwrite` (`OVERWRITE_REPORT` ou `CREATE_NEW_REPORT`).<br>**Uso**: Assim como no CUR, a combinação `PARQUET`/`PARQUET`/`OVERWRITE_REPORT` é a ideal para análise. |

### Exemplo de Requisição (Exportação Padrão do CUR 2.0)

```json
{
  "Export": {
    "Name": "Exportacao-CUR-2.0-Padrao",
    "Description": "Exportação completa dos dados de custo e uso",
    "DataQuery": {
      "QueryStatement": "SELECT * FROM COST_AND_USAGE_REPORT",
      "TableConfigurations": {
        "COST_AND_USAGE_REPORT": {
          "TIME_GRANULARITY": "HOURLY",
          "INCLUDE_RESOURCES": "TRUE"
        }
      }
    },
    "DestinationConfigurations": {
      "S3Destination": {
        "S3Bucket": "meu-bucket-bcm-exports",
        "S3Prefix": "cur-2.0/",
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

## 2. ListExports

Lista todas as exportações de dados que foram configuradas na conta.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | O número máximo de exportações a serem retornadas por página. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{
  "MaxResults": 10
}
```

---

## 3. GetExport

Recupera os detalhes completos de uma configuração de exportação específica.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ExportArn` | String | **Sim** | O ARN da exportação que você deseja inspecionar. Este ARN é obtido na resposta da chamada `ListExports`. |

### Exemplo de Requisição

```json
{
  "ExportArn": "arn:aws:bcm-data-exports:us-east-1:123456789012:export/EXPORT_ID_AQUI"
}
```

---

## 4. ListTables

Lista as tabelas de dados que estão disponíveis para consulta e exportação através do serviço.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de tabelas a retornar. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição

```json
{}
```
