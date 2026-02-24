# Guia Devastadoramente Detalhado: AWS Pricing API

## Visão Geral

A AWS Pricing API, também conhecida como Price List API, permite que você recupere informações de preços para todos os produtos e serviços da AWS de forma programática. Em vez de navegar pelo site de preços, você pode usar esta API para obter os preços públicos sob demanda ou baixar arquivos de lista de preços completos para uso offline. É fundamental para calculadoras de custo, ferramentas de estimativa e para entender o custo de diferentes arquiteturas.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://api.pricing.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSPriceListService` |
| **Service Name (IAM)** | `pricing` |

---

## 1. DescribeServices

Lista todos os serviços da AWS para os quais você pode obter informações de preços.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Não | O código de um serviço específico. Se fornecido, retorna detalhes apenas para esse serviço. Ex: `"AmazonEC2"`. |
| `FormatVersion` | String | Não | A versão do formato. Use `"aws_v1"`. |
| `MaxResults` | Integer | Não | Número máximo de resultados por página. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Listar todos os serviços)

```json
{
  "FormatVersion": "aws_v1"
}
```

---

## 2. GetProducts

Retorna os preços e atributos de um ou mais produtos (SKUs) que correspondem aos filtros que você especificar. Esta é a ação principal para consultas de preços sob demanda.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | **Sim** | O código do serviço para o qual você quer os preços. Ex: `"AmazonEC2"`, `"AmazonS3"`. Obtido da chamada `DescribeServices`. |
| `Filters` | Array de Objetos | **Sim** | Uma lista de filtros para encontrar o produto exato que você deseja. Cada objeto de filtro tem `Type`, `Field` e `Value`.<br>- `Type`: `TERM_MATCH`.<br>- `Field`: O atributo do produto a ser filtrado (ex: `instanceType`, `location`, `operatingSystem`, `tenancy`).<br>- `Value`: O valor desejado para o atributo.<br>**Uso**: Essencial para encontrar o preço de algo específico. Por exemplo, para uma instância `t2.micro` Linux On-Demand em `us-east-1`, você precisará de múltiplos filtros. |
| `FormatVersion` | String | Não | Use `"aws_v1"`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Preço de uma instância EC2 t2.micro Linux em N. Virginia)

```json
{
  "ServiceCode": "AmazonEC2",
  "Filters": [
    {
      "Type": "TERM_MATCH",
      "Field": "location",
      "Value": "US East (N. Virginia)"
    },
    {
      "Type": "TERM_MATCH",
      "Field": "instanceType",
      "Value": "t2.micro"
    },
    {
      "Type": "TERM_MATCH",
      "Field": "tenancy",
      "Value": "Shared"
    },
    {
      "Type": "TERM_MATCH",
      "Field": "operatingSystem",
      "Value": "Linux"
    },
    {
      "Type": "TERM_MATCH",
      "Field": "preInstalledSw",
      "Value": "NA"
    }
  ],
  "FormatVersion": "aws_v1"
}
```

---

## 3. GetAttributeValues

Retorna todos os valores possíveis para um atributo específico de um serviço. Útil para construir filtros dinâmicos em uma interface de usuário.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | **Sim** | O código do serviço. Ex: `"AmazonEC2"`. |
| `AttributeName` | String | **Sim** | O nome do atributo cujos valores você deseja listar. Ex: `"instanceType"`, `"region"`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Listar todos os tipos de instância EC2)

```json
{
  "ServiceCode": "AmazonEC2",
  "AttributeName": "instanceType"
}
```

---

## 4. ListPriceLists

Lista os arquivos de lista de preços disponíveis para download em massa. Esses arquivos contêm todos os preços para um determinado serviço e região.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | **Sim** | O código do serviço. |
| `EffectiveDate` | Timestamp | **Sim** | A data para a qual você quer a lista de preços. Use a data atual para obter os preços mais recentes. |
| `RegionCode` | String | Não | O código da região (ex: `us-east-1`). Se não especificado, retorna para todas as regiões. |
| `CurrencyCode` | String | **Sim** | O código da moeda (ex: `USD`). |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

### Exemplo de Requisição (Listar arquivos de preços para EC2 em USD)

```json
{
  "ServiceCode": "AmazonEC2",
  "EffectiveDate": "2026-02-23T00:00:00Z",
  "CurrencyCode": "USD"
}
```

---

## 5. GetPriceListFileUrl

Obtém a URL pré-assinada para baixar um arquivo de lista de preços específico.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `PriceListArn` | String | **Sim** | O ARN da lista de preços que você deseja baixar. Este ARN é obtido da resposta da chamada `ListPriceLists`. |
| `FileFormat` | String | **Sim** | O formato do arquivo. `CSV` ou `JSON`. |

### Exemplo de Requisição

```json
{
  "PriceListArn": "arn:aws:pricing::123456789012:price-list/AmazonEC2/20260223000000",
  "FileFormat": "CSV"
}
```
