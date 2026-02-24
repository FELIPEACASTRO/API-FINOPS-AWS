# Guia Devastadoramente Detalhado: Outras APIs de Governança

## Visão Geral

Este guia cobre um conjunto de serviços de governança que, embora menos focados em custo direto, são cruciais para a operação eficiente e bem arquitetada na nuvem, o que indiretamente impacta os custos.

---

## Service Quotas

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://servicequotas.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `servicequotas` |

### 1. ListServices

Lista os serviços para os quais você pode visualizar as cotas (limites).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

### 2. ListServiceQuotas

Lista as cotas para um serviço específico.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ServiceCode` | String | **Sim** | O código do serviço (ex: `ec2`, `vpc`). Obtido de `ListServices`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

---

## AWS License Manager

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://license-manager.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `license-manager` |

### 1. ListLicenseSpecificationsForResource

Lista as especificações de licença associadas a um recurso (como uma AMI ou instância).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ResourceArn` | String | **Sim** | O ARN do recurso. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

---

## AWS Well-Architected Tool

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://wellarchitected.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `wellarchitected` |

### 1. ListWorkloads

Lista as cargas de trabalho (workloads) que foram definidas na ferramenta.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `WorkloadNamePrefix` | String | Não | Filtra workloads por um prefixo de nome. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

---

## AWS Systems Manager (SSM)

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ssm.{region}.amazonaws.com` |
| **Protocolo** | JSON-RPC (POST) |
| **Service Name (IAM)** | `ssm` |

### 1. DescribeInstanceInformation

Retorna informações sobre suas instâncias gerenciadas pelo SSM, incluindo status do agente, plataforma e versão. Útil para garantir que o agente de gerenciamento esteja funcionando, o que é um pré-requisito para muitas automações de otimização.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `Filters` | Array de Objetos | Não | Filtra por `PingStatus` (`Online`, `Offline`), `PlatformTypes`, `ActivationIds`, etc. |
| `InstanceInformationFilterList` | Array de Objetos | Não | Filtros mais antigos e menos flexíveis. Prefira `Filters`. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

#### Exemplo de Requisição (Listar instâncias offline)

```json
{
  "Filters": [
    {
      "Key": "PingStatus",
      "Values": ["Offline"]
    }
  ]
}
```
