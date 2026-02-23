# 5. APIs de Governança e Organização

## 5.1 AWS Organizations API

O AWS Organizations é fundamental para ambientes multi-conta, permitindo gerenciar contas de forma centralizada e entender a estrutura organizacional para alocação de custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://organizations.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSOrganizationsV20161128` |
| **Service Name (IAM)** | `organizations` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeOrganization` | Retorna informações sobre a organização AWS (ID, conta master, features habilitadas). |
| `ListAccounts` | Lista todas as contas da organização com nome, email e status. |
| `DescribeAccount` | Retorna detalhes de uma conta específica. |
| `ListOrganizationalUnitsForParent` | Lista as OUs filhas de um parent (root ou outra OU). |
| `ListAccountsForParent` | Lista as contas diretamente dentro de um parent. |
| `ListTagsForResource` | Lista as tags de uma conta ou OU. |

---

## 5.2 Resource Groups Tagging API

A Tagging API é essencial para governança de tags, que é a base da alocação de custos em FinOps. Ela permite buscar recursos por tag, listar chaves e valores de tags, e verificar a compliance de tagging.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://tagging.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `ResourceGroupsTaggingAPI_20170126` |
| **Service Name (IAM)** | `tagging` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `GetResources` | Retorna recursos que possuem as tags especificadas. Suporta filtros por tipo de recurso e por tag. |
| `GetTagKeys` | Lista todas as chaves de tag em uso na conta e região. |
| `GetTagValues` | Lista todos os valores para uma chave de tag específica. |
| `TagResources` | Adiciona tags a um ou mais recursos. |
| `UntagResources` | Remove tags de um ou mais recursos. |
| `GetComplianceSummary` | Retorna um resumo de compliance de tags, agrupado por tipo de recurso, região ou chave de tag. |

### Uso Estratégico para FinOps

A Tagging API é particularmente poderosa para:

1. **Identificar recursos sem tags**: Use `GetResources` sem filtros de tag e compare com o inventário total para encontrar recursos não tagueados.
2. **Auditar consistência de tags**: Use `GetTagKeys` e `GetTagValues` para identificar variações e inconsistências nas tags (ex: "Environment" vs "environment" vs "env").
3. **Medir compliance de tagging**: Use `GetComplianceSummary` para obter métricas de compliance por tipo de recurso.

---

## 5.3 AWS Config API

O AWS Config permite avaliar, auditar e monitorar as configurações dos recursos AWS. Para FinOps, é especialmente útil para inventário avançado e consultas SQL sobre recursos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://config.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `StarlingDoveService` |
| **Service Name (IAM)** | `config` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `GetDiscoveredResourceCounts` | Retorna a contagem de recursos descobertos por tipo. |
| `ListDiscoveredResources` | Lista os recursos descobertos de um tipo específico. |
| `SelectResourceConfig` | Executa uma consulta SQL sobre os recursos configurados. **Esta é a ação mais poderosa para inventário.** |
| `DescribeComplianceByConfigRule` | Lista o status de compliance de todas as regras do Config. |
| `DescribeComplianceByResource` | Lista o status de compliance por recurso. |
| `GetComplianceDetailsByConfigRule` | Retorna detalhes de compliance de uma regra específica. |

### Consultas SQL com SelectResourceConfig

O `SelectResourceConfig` permite executar consultas SQL sobre todos os recursos rastreados pelo Config. Exemplos úteis para FinOps:

**Listar todas as instâncias EC2 com tipo e tags:**
```sql
SELECT resourceId, resourceType, configuration.instanceType, tags
WHERE resourceType = 'AWS::EC2::Instance'
```

**Encontrar volumes EBS não anexados:**
```sql
SELECT resourceId, configuration.size, configuration.volumeType
WHERE resourceType = 'AWS::EC2::Volume'
AND configuration.state = 'available'
```

**Listar recursos sem a tag "CostCenter":**
```sql
SELECT resourceId, resourceType
WHERE tags.tag('CostCenter') IS NULL
```

---

## 5.4 Service Quotas API

O Service Quotas permite visualizar e gerenciar as cotas (limites) de serviço. Para FinOps, é útil para entender os limites de recursos e planejar o crescimento.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://servicequotas.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `ServiceQuotasV20190624` |
| **Service Name (IAM)** | `service-quotas` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `ListServices` | Lista os serviços AWS que possuem cotas gerenciáveis. |
| `ListServiceQuotas` | Lista todas as cotas de um serviço específico. |
| `GetServiceQuota` | Retorna o valor de uma cota específica. |
| `ListRequestedServiceQuotaChangeHistory` | Lista o histórico de solicitações de aumento de cota. |
| `RequestServiceQuotaIncrease` | Solicita um aumento de cota. |

---

## 5.5 License Manager API

O License Manager permite gerenciar licenças de software e rastrear o uso para garantir conformidade e otimizar custos de licenciamento.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://license-manager.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSLicenseManager` |
| **Service Name (IAM)** | `license-manager` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `ListLicenseConfigurations` | Lista todas as configurações de licença. |
| `GetLicenseConfiguration` | Retorna detalhes de uma configuração de licença. |
| `ListUsageForLicenseConfiguration` | Lista o uso de uma configuração de licença. |
| `ListResourceInventory` | Lista o inventário de recursos que usam licenças gerenciadas. |

---

## 5.6 Well-Architected Tool API

O Well-Architected Tool permite acessar e gerenciar revisões de workloads para garantir a aderência às melhores práticas, incluindo o pilar de Otimização de Custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://wellarchitected.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `wellarchitected` |

### Ações Relevantes para FinOps

| Ação | Método | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `ListWorkloads` | POST | `/workloadsSummaries` | Lista todos os workloads registrados. |
| `GetWorkload` | GET | `/workloads/{id}` | Retorna detalhes de um workload. |
| `ListLensReviews` | GET | `/workloads/{id}/lensReviews` | Lista as revisões de lentes de um workload. |
| `GetLensReview` | GET | `/workloads/{id}/lensReviews/{lens}` | Retorna detalhes de uma revisão de lente. |
| `ListAnswers` | GET | `/workloads/{id}/lensReviews/{lens}/answers` | Lista as respostas de uma revisão. |
| `ListLenses` | GET | `/lenses` | Lista as lentes disponíveis. |

---

## 5.7 Systems Manager API

O Systems Manager fornece visibilidade operacional e permite gerenciar recursos em escala. Para FinOps, o inventário de software e hardware é particularmente útil.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://ssm.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AmazonSSM` |
| **Service Name (IAM)** | `ssm` |

### Ações Relevantes para FinOps

| Ação | Descrição |
| :--- | :--- |
| `DescribeInstanceInformation` | Lista informações sobre instâncias gerenciadas pelo SSM (plataforma, versão do agente, status). |
| `GetInventory` | Retorna dados de inventário coletados das instâncias (software instalado, configurações de rede, etc.). |
| `ListComplianceSummaries` | Lista resumos de compliance por tipo. |

## Referências

- [Organizations API Reference](https://docs.aws.amazon.com/organizations/latest/APIReference/Welcome.html)
- [Resource Groups Tagging API Reference](https://docs.aws.amazon.com/resourcegroupstagging/latest/APIReference/Welcome.html)
- [AWS Config API Reference](https://docs.aws.amazon.com/config/latest/APIReference/Welcome.html)
- [Service Quotas API Reference](https://docs.aws.amazon.com/servicequotas/2019-06-24/apireference/Welcome.html)
- [Well-Architected Tool API Reference](https://docs.aws.amazon.com/wellarchitected/latest/APIReference/Welcome.html)
