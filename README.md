# Guia Definitivo de APIs da AWS para FinOps (Versão Devastadora)

## Introdução

Esta é uma versão massivamente enriquecida do guia de APIs da AWS para FinOps. Cada serviço agora tem sua própria documentação detalhada, cobrindo **TODOS os parâmetros** de cada ação de API relevante, com exemplos e explicações claras.

O objetivo é fornecer um recurso de profundidade inigualável para engenheiros, analistas de FinOps e arquitetos que buscam automação total e controle granular sobre seus custos na nuvem.

## 🗂️ Estrutura do Repositório

O repositório está organizado em pastas, uma para cada domínio principal de FinOps. Dentro de cada pasta, você encontrará arquivos Markdown dedicados a cada serviço ou grupo de serviços.

```
API-FINOPS-AWS/
├── README.md                          # Este guia principal
├── insomnia_collection_massive.json   # Coleção Insomnia com 200+ requisições
├── INSOMNIA_SETUP.md                  # Guia de configuração do Insomnia
├── IAM_POLICY_FULL.json               # Política IAM de leitura completa
└── docs/
    ├── 01-COST-MANAGEMENT/
    │   ├── 01-COST-EXPLORER.md
    │   ├── 02-COST-ANOMALY.md
    │   └── 03-COST-OPTIMIZATION-HUB.md
    ├── 02-BILLING/
    │   ├── 01-BUDGETS.md
    │   ├── 02-CUR.md
    │   ├── 03-BCM-DATA-EXPORTS.md
    │   └── 04-BILLING-CONDUCTOR.md
    ├── 03-OPTIMIZATION/
    │   ├── 01-COMPUTE-OPTIMIZER.md
    │   ├── 02-TRUSTED-ADVISOR.md
    │   ├── 03-PRICING.md
    │   └── 04-SAVINGS-PLANS.md
    ├── 04-INVENTORY/
    │   ├── 01-EC2.md
    │   ├── 02-RDS.md
    │   ├── 03-S3.md
    │   ├── 04-LAMBDA.md
    │   └── ... (outros serviços)
    ├── 05-GOVERNANCE/
    │   ├── 01-ORGANIZATIONS.md
    │   ├── 02-TAGGING.md
    │   ├── 03-CONFIG.md
    │   └── ... (outros serviços)
    └── 06-MONITORING/
        └── 01-CLOUDWATCH.md
```

## 🚀 Como Começar

1.  **Explore a Documentação**: Navegue pelas pastas `docs/` para encontrar a documentação detalhada da API que você precisa.
2.  **Importe a Coleção Insomnia**: Use o arquivo `insomnia_collection_massive.json` para importar mais de 200 requisições pré-configuradas.
3.  **Configure o Ambiente**: Siga o `INSOMNIA_SETUP.md` para configurar suas credenciais AWS.
4.  **Aplique a Política IAM**: Use `IAM_POLICY_FULL.json` como base para criar um perfil IAM com as permissões de leitura necessárias.

Este projeto agora serve como uma enciclopédia acionável para automação de FinOps na AWS. Mergulhe na documentação e comece a construir!
