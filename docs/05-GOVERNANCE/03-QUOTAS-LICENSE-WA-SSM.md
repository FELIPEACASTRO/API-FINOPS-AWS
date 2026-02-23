# Guia Detalhado: Service Quotas, License Manager, Well-Architected e SSM APIs (FinOps)

---

## 1. Service Quotas API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://servicequotas.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `ServiceQuotasV20190624` |
| **Service Name (IAM)** | `service-quotas` |

### 1.1 ListServices

Lista os serviços AWS que possuem cotas gerenciáveis.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados (1-100). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

### 1.2 ListServiceQuotas

Lista todas as cotas de um serviço específico.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço (ex: `ec2`, `vpc`, `lambda`). |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `QuotaCode` | String | Não | Código de uma cota específica. |
| `QuotaAppliedAtLevel` | String | Não | `ACCOUNT`, `RESOURCE`, `ALL`. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "ec2"
}
```

### 1.3 GetServiceQuota

Retorna o valor de uma cota específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço. |
| `QuotaCode` | String | Sim | Código da cota. |
| `ContextId` | String | Não | ID de contexto para cotas de nível de recurso. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "ec2",
  "QuotaCode": "L-1216C47A"
}
```

### 1.4 RequestServiceQuotaIncrease

Solicita um aumento de cota.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | Sim | Código do serviço. |
| `QuotaCode` | String | Sim | Código da cota. |
| `DesiredValue` | Double | Sim | Valor desejado para a cota. |
| `ContextId` | String | Não | ID de contexto. |

#### Exemplo de Requisição

```json
{
  "ServiceCode": "ec2",
  "QuotaCode": "L-1216C47A",
  "DesiredValue": 500
}
```

---

## 2. AWS License Manager API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://license-manager.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSLicenseManager` |
| **Service Name (IAM)** | `license-manager` |

### 2.1 ListLicenseConfigurations

Lista todas as configurações de licença.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `LicenseConfigurationArns` | Array | Não | ARNs de configurações específicas. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Array | Não | Filtros com `Name` e `Values`. |

#### Exemplo de Requisição

```json
{}
```

### 2.2 ListUsageForLicenseConfiguration

Lista o uso de uma configuração de licença.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `LicenseConfigurationArn` | String | Sim | ARN da configuração de licença. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Array | Não | Filtros. |

#### Exemplo de Requisição

```json
{
  "LicenseConfigurationArn": "arn:aws:license-manager:us-east-1:123456789012:license-configuration:lic-abc123"
}
```

### 2.3 ListResourceInventory

Lista o inventário de recursos que usam licenças gerenciadas.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |
| `Filters` | Array | Não | Filtros por `Name`, `Condition` e `Value`. |

#### Exemplo de Requisição

```json
{
  "Filters": [
    {
      "Name": "Platform",
      "Condition": "EQUALS",
      "Value": "Windows"
    }
  ]
}
```

---

## 3. AWS Well-Architected Tool API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://wellarchitected.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `wellarchitected` |

### 3.1 ListWorkloads

Lista todos os workloads registrados.

**Método HTTP**: POST  
**Path**: `/workloadsSummaries`

#### Parâmetros de Entrada (Body)

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `WorkloadNamePrefix` | String | Não | Filtro por prefixo de nome. |
| `MaxResults` | Integer | Não | Número máximo de resultados (1-50). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "MaxResults": 50
}
```

### 3.2 GetLensReview

Retorna detalhes de uma revisão de lente para um workload.

**Método HTTP**: GET  
**Path**: `/workloads/{WorkloadId}/lensReviews/{LensAlias}`

#### Parâmetros de Path

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `WorkloadId` | String | Sim | ID do workload. |
| `LensAlias` | String | Sim | Alias da lente (ex: `wellarchitected`, `serverless`). |

#### Parâmetros de Query String

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MilestoneNumber` | Integer | Não | Número do milestone. |

---

## 4. AWS Systems Manager API

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ssm.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AmazonSSM` |
| **Service Name (IAM)** | `ssm` |

### 4.1 DescribeInstanceInformation

Lista informações sobre instâncias gerenciadas pelo SSM.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `InstanceInformationFilterList` | Array | Não | Filtros por `key` e `valueSet`. Keys: `InstanceIds`, `AgentVersion`, `PingStatus`, `PlatformTypes`, `ActivationIds`, `IamRole`, `ResourceType`, `AssociationStatus`. |
| `Filters` | Array | Não | Filtros alternativos com `Key`, `Values` e `Type`. |
| `MaxResults` | Integer | Não | Número máximo de resultados (1-50). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "MaxResults": 50,
  "Filters": [
    {
      "Key": "PingStatus",
      "Values": ["Online"],
      "Type": "Equal"
    }
  ]
}
```

### 4.2 GetInventory

Retorna dados de inventário coletados das instâncias (software instalado, configurações de rede, etc.).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `Filters` | Array | Não | Filtros por `Key`, `Values` e `Type`. |
| `Aggregators` | Array | Não | Agregadores para agrupar resultados. |
| `ResultAttributes` | Array | Não | Atributos adicionais a retornar. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "Filters": [
    {
      "Key": "TypeName",
      "Values": ["AWS:Application"],
      "Type": "Equal"
    }
  ],
  "MaxResults": 50
}
```
