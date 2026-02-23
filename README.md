# Guia Definitivo de APIs da AWS para FinOps

## Introdução

Este repositório é um guia completo e prático para a utilização das APIs nativas da AWS no contexto de **FinOps (Cloud Financial Operations)**. O objetivo é fornecer um recurso centralizado, didático e acionável para profissionais que buscam automatizar e escalar suas práticas de gerenciamento financeiro na nuvem.

O conteúdo está estruturado de acordo com os três domínios do framework FinOps: **Informar, Otimizar e Operar**. Para cada domínio, apresentamos as APIs mais relevantes, suas principais funcionalidades e exemplos de requisições. Acompanhando esta documentação, você encontrará uma coleção Insomnia abrangente com mais de 120 ações de API prontas para uso.

## 🌐 Domínio 1: Informar (Entender e Alocar)

O primeiro passo em FinOps é ter visibilidade total sobre os custos e o uso da nuvem. As APIs deste domínio são a base para a coleta, análise e alocação de custos.

### 1.1. Visibilidade de Custos e Uso

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **Cost Explorer** | `ce` | Consultar dados agregados de custo e uso, previsões e recomendações. | `GetCostAndUsage`, `GetCostForecast`, `GetReservationUtilization`, `GetSavingsPlansCoverage` |
| **Cost & Usage Reports** | `cur` | Acessar os dados mais granulares de custo e uso, entregues em um bucket S3. | `PutReportDefinition`, `DescribeReportDefinitions`, `DeleteReportDefinition` |
| **BCM Data Exports** | `bcm-data-exports` | Criar exportações personalizadas de múltiplos conjuntos de dados de billing. | `CreateExport`, `ListExports`, `GetExecution` |

### 1.2. Alocação de Custos

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **Cost Categories** | `ce` | Mapear custos da AWS para estruturas de negócio internas (times, projetos, etc.). | `CreateCostCategoryDefinition`, `ListCostCategoryDefinitions`, `UpdateCostCategoryDefinition` |
| **Resource Groups & Tagging** | `tag` | Gerenciar e consultar tags em múltiplos recursos para alocação e governança. | `GetResources`, `TagResources`, `UntagResources`, `GetTagKeys`, `GetTagValues` |
| **AWS Organizations** | `organizations` | Gerenciar contas de forma centralizada, fundamental para a alocação em multi-contas. | `ListAccounts`, `DescribeAccount`, `ListTagsForResource` |

### 1.3. Benchmarking e Métricas

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **Amazon CloudWatch** | `monitoring` | Coletar métricas de performance e uso de praticamente todos os serviços da AWS. | `GetMetricData`, `GetMetricStatistics`, `ListMetrics` |
| **S3 Storage Lens** | `s3control` | Obter visibilidade sobre o uso e atividade do armazenamento de objetos em toda a organização. | `GetStorageLensConfiguration`, `ListStorageLensConfigurations` |

## ⚙️ Domínio 2: Otimizar (Economizar e Aumentar a Eficiência)

Com a visibilidade estabelecida, o próximo passo é identificar e agir sobre as oportunidades de otimização de custos.

### 2.1. Recomendações de Otimização

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **Compute Optimizer** | `compute-optimizer` | Fornecer recomendações de *right-sizing* para EC2, EBS, Lambda, ECS e mais. | `GetEC2InstanceRecommendations`, `GetEBSVolumeRecommendations`, `GetLambdaFunctionRecommendations` |
| **Cost Optimization Hub** | `cost-optimization-hub` | Identificar, filtrar e agregar recomendações de otimização de custos de múltiplos serviços. | `ListRecommendations`, `GetRecommendation`, `ListRecommendationSummaries` |
| **Trusted Advisor** | `support` / `trustedadvisor` | Acessar recomendações de otimização de custos, segurança, performance e resiliência. | `DescribeTrustedAdvisorChecks`, `RefreshTrustedAdvisorCheck`, `ListRecommendations` (nova API) |

### 2.2. Otimização de Rate (Preços)

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **Savings Plans** | `savingsplans` | Gerenciar e analisar os Savings Plans para obter descontos em troca de compromisso de uso. | `DescribeSavingsPlans`, `DescribeSavingsPlansOfferings`, `CreateSavingsPlan` |
| **Reserved Instances** | `ec2`, `rds`, etc. | Gerenciar Instâncias Reservadas para serviços específicos como EC2, RDS, ElastiCache, etc. | `DescribeReservedInstances`, `PurchaseReservedInstancesOffering` |
| **AWS Pricing API** | `pricing` | Consultar os preços de todos os produtos e serviços da AWS de forma programática. | `GetProducts`, `DescribeServices`, `GetAttributeValues` |

## 🚀 Domínio 3: Operar (Melhoria Contínua e Governança)

Este domínio foca na automação, controle e melhoria contínua dos processos de FinOps.

### 3.1. Controle de Custos

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **AWS Budgets** | `budgets` | Criar, gerenciar e consultar orçamentos, com ações automáticas ao atingir limites. | `CreateBudget`, `UpdateBudget`, `CreateBudgetAction`, `DescribeBudgetPerformanceHistory` |
| **Cost Anomaly Detection** | `ce` | Criar monitores que usam machine learning para detectar gastos anômalos e enviar alertas. | `CreateAnomalyMonitor`, `GetAnomalies`, `ProvideAnomalyFeedback` |

### 3.2. Governança e Conformidade

| Serviço | API Principal | Finalidade | Ações Chave |
| :--- | :--- | :--- | :--- |
| **AWS Config** | `config` | Avaliar, auditar e monitorar as configurações dos recursos e sua conformidade com políticas. | `SelectResourceConfig`, `ListDiscoveredResources`, `GetComplianceDetailsByResource` |
| **Service Quotas** | `service-quotas` | Visualizar e gerenciar as cotas (limites) de serviço para evitar crescimento inesperado. | `ListServiceQuotas`, `GetServiceQuota`, `RequestServiceQuotaIncrease` |
| **AWS Well-Architected Tool** | `wellarchitected` | Acessar e gerenciar revisões de workloads para garantir a aderência às melhores práticas. | `ListWorkloads`, `GetLensReview`, `ListAnswers` |
| **License Manager** | `license-manager` | Gerenciar licenças de software e rastrear o uso para garantir conformidade e otimizar custos. | `ListLicenseConfigurations`, `ListUsageForLicenseConfiguration`, `GetLicenseConfiguration` |

## 🗂️ Anexo: APIs de Inventário de Recursos

Para uma prática de FinOps eficaz, é essencial ter um inventário completo dos recursos provisionados. Abaixo estão as ações chave para listar recursos nos principais serviços:

| Serviço | Ação Principal para Inventário |
| :--- | :--- |
| **EC2** | `DescribeInstances`, `DescribeVolumes`, `DescribeSnapshots` |
| **RDS** | `DescribeDBInstances`, `DescribeDBClusters` |
| **S3** | `ListBuckets` |
| **Lambda** | `ListFunctions` |
| **ECS** | `ListClusters`, `ListServices`, `ListTasks` |
| **EKS** | `ListClusters`, `ListNodegroups` |
| **ElastiCache** | `DescribeCacheClusters` |
| **DynamoDB** | `ListTables` |
| **ELBv2** | `DescribeLoadBalancers` |

## 🛠️ Configuração e Uso

Para começar a usar estas APIs imediatamente, siga o guia detalhado no arquivo **[INSOMNIA_SETUP.md](./INSOMNIA_SETUP.md)**. Ele cobre a importação da coleção, configuração de credenciais e como executar suas primeiras requisições.

## 🔐 Permissões IAM

Para utilizar todas as APIs desta coleção, é necessário um perfil IAM com as permissões adequadas. Fornecemos um exemplo de política de *leitura* no arquivo **[IAM_POLICY.json](./IAM_POLICY.json)**. É altamente recomendável seguir o princípio do menor privilégio e ajustar a política às suas necessidades específicas.

## Conclusão

Este repositório foi projetado para ser um acelerador para suas iniciativas de FinOps. Use-o como ponto de partida para criar dashboards personalizados, sistemas de alerta, automações de otimização e para integrar dados financeiros da nuvem em seus sistemas de BI. A automação é a chave para escalar a cultura FinOps, e estas APIs são as ferramentas para construir essa automação.
