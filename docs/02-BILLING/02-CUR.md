# Guia Detalhado: AWS Cost and Usage Reports (CUR) API

## Visão Geral

O Cost and Usage Report (CUR) é o relatório mais detalhado e granular de custos da AWS. Ele é entregue em um bucket S3 e pode ser integrado com Athena, Redshift ou QuickSight para análise avançada.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cur.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSOrigamiServiceGatewayService` |
| **Service Name (IAM)** | `cur` |

---

## Ações da API

### 1. PutReportDefinition

Cria uma nova definição de relatório CUR.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReportDefinition` | Object | Sim | Objeto que define o relatório. |

#### Objeto `ReportDefinition`

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReportName` | String | Sim | Nome único do relatório. |
| `TimeUnit` | String | Sim | `HOURLY`, `DAILY` ou `MONTHLY`. |
| `Format` | String | Sim | `textORcsv`, `Parquet`. |
| `Compression` | String | Sim | `ZIP`, `GZIP`, `Parquet`. |
| `S3Bucket` | String | Sim | Nome do bucket S3 de destino. |
| `S3Prefix` | String | Sim | Prefixo no bucket S3. |
| `S3Region` | String | Sim | Região do bucket S3. |
| `AdditionalSchemaElements` | Array | Não | `RESOURCES` (inclui IDs de recursos), `SPLIT_COST_ALLOCATION_DATA`. |
| `AdditionalArtifacts` | Array | Não | `ATHENA`, `REDSHIFT`, `QUICKSIGHT`. |
| `RefreshClosedReports` | Boolean | Não | Se `true`, atualiza relatórios de meses fechados. |
| `ReportVersioning` | String | Não | `CREATE_NEW_REPORT` ou `OVERWRITE_REPORT`. |
| `BillingViewArn` | String | Não | ARN de uma visualização de billing personalizada. |

#### Exemplo de Requisição

```json
{
  "ReportDefinition": {
    "ReportName": "CUR-Athena-Hourly",
    "TimeUnit": "HOURLY",
    "Format": "Parquet",
    "Compression": "Parquet",
    "S3Bucket": "my-cur-bucket",
    "S3Prefix": "cur-reports/",
    "S3Region": "us-east-1",
    "AdditionalSchemaElements": ["RESOURCES"],
    "AdditionalArtifacts": ["ATHENA"],
    "RefreshClosedReports": true,
    "ReportVersioning": "OVERWRITE_REPORT"
  }
}
```

---

### 2. DescribeReportDefinitions

Lista todas as definições de relatórios CUR existentes.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados (5-5). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

---

### 3. ModifyReportDefinition

Modifica uma definição de relatório existente.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReportName` | String | Sim | Nome do relatório a ser modificado. |
| `ReportDefinition` | Object | Sim | Nova definição do relatório (mesma estrutura de `PutReportDefinition`). |

#### Exemplo de Requisição

```json
{
  "ReportName": "CUR-Athena-Hourly",
  "ReportDefinition": {
    "ReportName": "CUR-Athena-Hourly",
    "TimeUnit": "DAILY",
    "Format": "Parquet",
    "Compression": "Parquet",
    "S3Bucket": "my-cur-bucket",
    "S3Prefix": "cur-reports/",
    "S3Region": "us-east-1",
    "AdditionalSchemaElements": ["RESOURCES"],
    "AdditionalArtifacts": ["ATHENA"],
    "RefreshClosedReports": true,
    "ReportVersioning": "OVERWRITE_REPORT"
  }
}
```

---

### 4. DeleteReportDefinition

Remove uma definição de relatório.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ReportName` | String | Sim | Nome do relatório a ser removido. |

#### Exemplo de Requisição

```json
{
  "ReportName": "CUR-Athena-Hourly"
}
```
