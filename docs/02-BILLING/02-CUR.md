# Guia Devastadoramente Detalhado: AWS Cost and Usage Reports (CUR) API

## Visão Geral

O AWS Cost and Usage Reports (CUR) é a fonte de dados mais granular e completa para análise de custos na AWS. Ele gera arquivos CSV ou Parquet com dados detalhados de uso e custo, que podem ser carregados no Amazon Athena, Redshift ou S3 para análise. A API do CUR permite que você gerencie as definições de relatório de forma programática.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cur.us-east-1.amazonaws.com` (Global, apenas us-east-1) |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSOrigamiServiceGatewayService` |
| **Service Name (IAM)** | `cur` |

---

## 1. PutReportDefinition

Cria ou atualiza uma definição de relatório CUR.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ReportDefinition` | Objeto | **Sim** | O objeto que define todas as configurações do relatório. |

#### Objeto `ReportDefinition`

| Campo | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ReportName` | String | **Sim** | O nome do relatório. Deve ser único na conta. |
| `TimeUnit` | String | **Sim** | A granularidade temporal do relatório. `HOURLY` (por hora) ou `DAILY` (por dia) ou `MONTHLY` (por mês). |
| `Format` | String | **Sim** | O formato do arquivo. `textORcsv` para CSV ou `Parquet` para Parquet. |
| `Compression` | String | **Sim** | O tipo de compressão. `ZIP`, `GZIP` ou `Parquet` (quando o formato é Parquet). |
| `AdditionalSchemaElements` | Array de Strings | **Sim** | Elementos adicionais a incluir no relatório. Use `RESOURCES` para incluir o ARN de cada recurso. |
| `S3Bucket` | String | **Sim** | O nome do bucket S3 onde o relatório será armazenado. |
| `S3Prefix` | String | **Sim** | O prefixo do caminho no S3 para os arquivos do relatório. |
| `S3Region` | String | **Sim** | A região do bucket S3. |
| `AdditionalArtifacts` | Array de Strings | Não | Artefatos adicionais a serem gerados. `REDSHIFT` (manifesto para Redshift), `QUICKSIGHT`, `ATHENA`. |
| `RefreshClosedReports` | Boolean | Não | Se `true`, o relatório dos últimos 3 meses será atualizado quando novos dados chegarem. |
| `ReportVersioning` | String | Não | `CREATE_NEW_REPORT` (cria um novo arquivo a cada entrega) ou `OVERWRITE_REPORT` (sobrescreve o arquivo existente). |

### Exemplo de Requisição (Criar um relatório CUR para Athena)

```json
{
  "ReportDefinition": {
    "ReportName": "finops-cur-report",
    "TimeUnit": "HOURLY",
    "Format": "Parquet",
    "Compression": "Parquet",
    "AdditionalSchemaElements": ["RESOURCES"],
    "S3Bucket": "meu-bucket-finops-cur",
    "S3Prefix": "reports/cur/",
    "S3Region": "us-east-1",
    "AdditionalArtifacts": ["ATHENA"],
    "RefreshClosedReports": true,
    "ReportVersioning": "OVERWRITE_REPORT"
  }
}
```

---

## 2. DescribeReportDefinitions

Lista as definições de relatório CUR existentes na conta.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | O número máximo de resultados a serem retornados. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição

```json
{}
```

---

## 3. DeleteReportDefinition

Exclui uma definição de relatório CUR.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ReportName` | String | **Sim** | O nome do relatório a ser excluído. |

### Exemplo de Requisição

```json
{
  "ReportName": "finops-cur-report"
}
```
