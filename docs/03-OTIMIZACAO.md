# 3. APIs de Otimização

## 3.1 AWS Compute Optimizer API

O Compute Optimizer utiliza machine learning para analisar métricas de utilização e fornecer recomendações de right-sizing para diversos tipos de recursos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://compute-optimizer.{region}.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `ComputeOptimizerService` |
| **Service Name (IAM)** | `compute-optimizer` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `GetEnrollmentStatus` | Verifica se a conta está inscrita no Compute Optimizer. |
| `UpdateEnrollmentStatus` | Inscreve ou cancela a inscrição da conta. |
| `GetRecommendationSummaries` | Retorna um resumo das recomendações por tipo de recurso. |
| `GetEC2InstanceRecommendations` | Recomendações de right-sizing para instâncias EC2. |
| `GetAutoScalingGroupRecommendations` | Recomendações para grupos de Auto Scaling. |
| `GetEBSVolumeRecommendations` | Recomendações de otimização para volumes EBS. |
| `GetLambdaFunctionRecommendations` | Recomendações de memória para funções Lambda. |
| `GetECSServiceRecommendations` | Recomendações para serviços ECS no Fargate. |
| `GetRDSDatabaseRecommendations` | Recomendações de right-sizing para instâncias RDS. |
| `GetLicenseRecommendations` | Recomendações de otimização de licenças. |
| `ExportEC2InstanceRecommendations` | Exporta recomendações EC2 para um bucket S3. |
| `ExportAutoScalingGroupRecommendations` | Exporta recomendações de ASG para S3. |
| `ExportLambdaFunctionRecommendations` | Exporta recomendações Lambda para S3. |
| `ExportEBSVolumeRecommendations` | Exporta recomendações EBS para S3. |

### Tipos de Recomendação

O Compute Optimizer classifica cada recurso em uma das seguintes categorias:

| Classificação | Descrição |
| :--- | :--- |
| `Optimized` | O recurso está dimensionado corretamente. |
| `NotOptimized` | O recurso pode ser otimizado (over-provisioned ou under-provisioned). |
| `OverProvisioned` | O recurso tem mais capacidade do que o necessário. |
| `UnderProvisioned` | O recurso tem menos capacidade do que o necessário. |

---

## 3.2 AWS Trusted Advisor API (via Support API)

O Trusted Advisor fornece recomendações em cinco categorias: otimização de custos, performance, segurança, tolerância a falhas e limites de serviço. O acesso programático é feito via Support API e requer um plano de suporte Business ou Enterprise.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://support.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSSupport_20130415` |
| **Service Name (IAM)** | `support` |
| **Requisito** | Plano de suporte Business ou Enterprise |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `DescribeTrustedAdvisorChecks` | Lista todas as verificações disponíveis do Trusted Advisor. |
| `DescribeTrustedAdvisorCheckResult` | Retorna o resultado detalhado de uma verificação específica. |
| `DescribeTrustedAdvisorCheckSummaries` | Retorna resumos de uma ou mais verificações. |
| `DescribeTrustedAdvisorCheckRefreshStatuses` | Verifica o status de atualização de verificações. |
| `RefreshTrustedAdvisorCheck` | Solicita a atualização de uma verificação. |

### Verificações de Custo Mais Relevantes

| Check ID | Nome | Descrição |
| :--- | :--- | :--- |
| `Qch7DwouX1` | Low Utilization Amazon EC2 Instances | Identifica instâncias EC2 com baixa utilização de CPU. |
| `DAvU99Dc4C` | Underutilized Amazon EBS Volumes | Identifica volumes EBS subutilizados. |
| `Z4AUBRNSmz` | Amazon RDS Idle DB Instances | Identifica instâncias RDS ociosas. |
| `hjLMh88uM8` | Idle Load Balancers | Identifica load balancers sem tráfego. |
| `51fC20e7I2` | Unassociated Elastic IP Addresses | Identifica IPs elásticos não associados. |

---

## 3.3 Trusted Advisor API (Nova API REST)

A AWS lançou uma nova API REST para o Trusted Advisor, que oferece uma interface mais moderna e funcionalidades adicionais.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://trustedadvisor.{region}.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `trustedadvisor` |

### Ações Disponíveis

| Ação | Método | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `ListChecks` | GET | `/v2/checks` | Lista todas as verificações disponíveis. |
| `ListRecommendations` | GET | `/v2/recommendations` | Lista todas as recomendações ativas. |
| `GetRecommendation` | GET | `/v2/recommendations/{id}` | Retorna detalhes de uma recomendação. |
| `ListRecommendationResources` | GET | `/v2/recommendations/{id}/resources` | Lista os recursos afetados por uma recomendação. |
| `UpdateRecommendationLifecycle` | PUT | `/v2/recommendations/{id}/lifecycle` | Atualiza o ciclo de vida de uma recomendação. |
| `ListOrganizationRecommendations` | GET | `/v2/organization-recommendations` | Lista recomendações no nível da organização. |

---

## 3.4 AWS Pricing API

A Pricing API permite consultar os preços de todos os produtos e serviços da AWS de forma programática, essencial para análises de custo-benefício e comparações.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://api.pricing.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSPriceListService` |
| **Service Name (IAM)** | `pricing` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `DescribeServices` | Lista os serviços para os quais há informações de preço disponíveis, incluindo os atributos de cada serviço. |
| `GetAttributeValues` | Lista os valores disponíveis para um atributo de um serviço (ex: todos os tipos de instância EC2). |
| `GetProducts` | Retorna os preços de produtos que correspondem aos filtros especificados. |
| `GetPriceListFileUrl` | Retorna a URL de download de uma lista de preços em formato JSON ou CSV. |
| `ListPriceLists` | Lista as listas de preços disponíveis para um serviço. |

### Exemplo de Consulta de Preço EC2

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

## 3.5 Savings Plans API

A API de Savings Plans permite gerenciar e consultar informações sobre Savings Plans, que oferecem descontos significativos em troca de compromisso de uso.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://savingsplans.amazonaws.com` |
| **Protocolo** | REST (JSON) |
| **Service Name (IAM)** | `savingsplans` |

### Ações Disponíveis

| Ação | Método | Path | Descrição |
| :--- | :--- | :--- | :--- |
| `DescribeSavingsPlans` | POST | `/DescribeSavingsPlans` | Lista todos os Savings Plans da conta. |
| `DescribeSavingsPlansOfferings` | POST | `/DescribeSavingsPlansOfferings` | Lista as ofertas de SP disponíveis. |
| `DescribeSavingsPlansOfferingRates` | POST | `/DescribeSavingsPlansOfferingRates` | Lista as taxas detalhadas de uma oferta. |
| `CreateSavingsPlan` | POST | `/CreateSavingsPlan` | Compra um novo Savings Plan. |
| `DeleteQueuedSavingsPlan` | POST | `/DeleteQueuedSavingsPlan` | Cancela um SP na fila. |
| `ReturnSavingsPlan` | POST | `/ReturnSavingsPlan` | Devolve um SP (dentro do período de devolução). |

### Tipos de Savings Plans

| Tipo | Descrição | Desconto |
| :--- | :--- | :--- |
| `COMPUTE_SP` | Aplica-se a EC2, Lambda e Fargate em qualquer região. | Até 66% |
| `EC2_INSTANCE_SP` | Aplica-se a uma família de instâncias EC2 em uma região específica. | Até 72% |
| `SAGEMAKER_SP` | Aplica-se a instâncias SageMaker. | Até 64% |

---

## 3.6 Cost Optimization Hub API

O Cost Optimization Hub centraliza recomendações de otimização de custos de múltiplos serviços AWS em um único lugar.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://cost-optimization-hub.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.0` |
| **X-Amz-Target Prefix** | `CostOptimizationHubService` |
| **Service Name (IAM)** | `cost-optimization-hub` |

### Ações Disponíveis

| Ação | Descrição |
| :--- | :--- |
| `ListRecommendations` | Lista todas as recomendações de otimização de custos. |
| `ListRecommendationSummaries` | Lista resumos de recomendações agrupados por dimensão. |
| `GetRecommendation` | Retorna detalhes de uma recomendação específica. |
| `ListEnrollmentStatuses` | Lista o status de inscrição das contas. |
| `GetPreferences` | Retorna as preferências configuradas. |
| `UpdateEnrollmentStatus` | Atualiza o status de inscrição de uma conta. |
| `UpdatePreferences` | Atualiza as preferências do hub. |

## Referências

- [Compute Optimizer API Reference](https://docs.aws.amazon.com/compute-optimizer/latest/APIReference/Welcome.html)
- [Trusted Advisor API Reference](https://docs.aws.amazon.com/awssupport/latest/APIReference/Welcome.html)
- [Pricing API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations_AWS_Price_List_Service.html)
- [Savings Plans API Reference](https://docs.aws.amazon.com/savingsplans/latest/APIReference/Welcome.html)
- [Cost Optimization Hub API Reference](https://docs.aws.amazon.com/aws-cost-management/latest/APIReference/API_Operations_Cost_Optimization_Hub.html)
