# Guia Detalhado: AWS Pricing API

## Visão Geral

A Pricing API permite consultar os preços de todos os produtos e serviços da AWS de forma programática, essencial para análises de custo-benefício e comparações.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://api.pricing.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSPriceListService` |
| **Service Name (IAM)** | `pricing` |

---

## Ações da API

### 1. DescribeServices

Lista os serviços para os quais há informações de preço disponíveis, incluindo os atributos de cada serviço.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Não | Código do serviço (ex: `AmazonEC2`). Se não especificado, retorna todos os serviços. |
| `FormatVersion` | String | Não | Versão do formato de resposta. |
| `MaxResults` | Integer | Não | Número máximo de resultados (1-100). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "AmazonEC2",
  "FormatVersion": "aws_v1"
}
```

---

### 2. GetAttributeValues

Lista os valores disponíveis para um atributo de um serviço.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço. |
| `AttributeName` | String | Sim | Nome do atributo (ex: `instanceType`, `location`, `operatingSystem`). |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "AmazonEC2",
  "AttributeName": "instanceType"
}
```

---

### 3. GetProducts

Retorna os preços de produtos que correspondem aos filtros especificados. Esta é a ação principal para consultas de preço.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço. |
| `Filters` | Array | Não | Lista de filtros com `Type` (`TERM_MATCH`), `Field` e `Value`. |
| `FormatVersion` | String | Não | Versão do formato de resposta. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição (Preço de EC2 t3.micro em Virginia)

```json
{
  "ServiceCode": "AmazonEC2",
  "Filters": [
    {"Type": "TERM_MATCH", "Field": "instanceType", "Value": "t3.micro"},
    {"Type": "TERM_MATCH", "Field": "location", "Value": "US East (N. Virginia)"},
    {"Type": "TERM_MATCH", "Field": "operatingSystem", "Value": "Linux"},
    {"Type": "TERM_MATCH", "Field": "preInstalledSw", "Value": "NA"},
    {"Type": "TERM_MATCH", "Field": "tenancy", "Value": "Shared"},
    {"Type": "TERM_MATCH", "Field": "capacitystatus", "Value": "Used"}
  ],
  "MaxResults": 10
}
```

---

### 4. GetPriceListFileUrl

Retorna a URL de download de uma lista de preços completa.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `PriceListArn` | String | Sim | ARN da lista de preços. |
| `FileFormat` | String | Sim | `json` ou `csv`. |

#### Exemplo de Requisição

```json
{
  "PriceListArn": "arn:aws:pricing:us-east-1::price-list/AmazonEC2/20260101",
  "FileFormat": "json"
}
```

---

### 5. ListPriceLists

Lista as listas de preços disponíveis para um serviço.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço. |
| `EffectiveDate` | Timestamp | Sim | Data efetiva para a lista de preços. |
| `CurrencyCode` | String | Sim | Código da moeda (ex: `USD`). |
| `RegionCode` | String | Não | Código da região. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "AmazonEC2",
  "EffectiveDate": "2026-02-01T00:00:00Z",
  "CurrencyCode": "USD",
  "RegionCode": "us-east-1"
}
```
