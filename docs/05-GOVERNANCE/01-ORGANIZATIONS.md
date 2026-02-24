# Guia Devastadoramente Detalhado: AWS Organizations API

## Visão Geral

O AWS Organizations é o serviço central para gerenciar múltiplas contas AWS. Para FinOps, ele é a fonte da verdade para entender a estrutura hierárquica da empresa (quais contas pertencem a qual departamento ou unidade de negócio) e para aplicar políticas de governança. A API permite que você liste todas as contas, navegue pela estrutura de Unidades Organizacionais (OUs) e consulte tags, que são essenciais para a alocação de custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://organizations.us-east-1.amazonaws.com` (Global) |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSOrganizationsV20161128` |
| **Service Name (IAM)** | `organizations` |

---

## 1. ListAccounts

Retorna uma lista de todas as contas que fazem parte da organização.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | O número máximo de resultados a serem retornados em uma única chamada. Se o número de contas for maior, a resposta incluirá um `NextToken`. Útil para controlar o fluxo de dados. |
| `NextToken` | String | Não | Um token fornecido em uma resposta anterior para obter a próxima página de resultados. Essencial para iterar por todas as contas em organizações grandes. |

### Exemplo de Requisição

```json
{
  "MaxResults": 20
}
```

---

## 2. DescribeAccount

Recupera informações sobre uma conta específica, como nome, e-mail, status e data em que se juntou à organização.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | **Sim** | O identificador único (ID de 12 dígitos) da conta sobre a qual você deseja obter informações. |

### Exemplo de Requisição

```json
{
  "AccountId": "123456789012"
}
```

---

## 3. ListRoots

Lista as raízes (roots) de uma organização. Uma organização tem apenas uma raiz. Esta é a chamada inicial para começar a navegar na hierarquia da sua organização de cima para baixo.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição

```json
{}
```

---

## 4. ListOrganizationalUnitsForParent

Lista as Unidades Organizacionais (OUs) que são filhas diretas de uma raiz ou de outra OU. Use esta chamada recursivamente para mapear toda a sua árvore de OUs.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ParentId` | String | **Sim** | O ID do pai (uma raiz ou outra OU) cujas OUs filhas você deseja listar. Você obtém este ID da chamada `ListRoots` ou de uma chamada anterior a esta mesma ação. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar OUs sob a raiz)

```json
{
  "ParentId": "r-a1b2"
}
```

---

## 5. ListAccountsForParent

Lista as contas que são membros diretos de uma raiz ou OU especificada.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ParentId` | String | **Sim** | O ID do pai (raiz ou OU) cujas contas membro você deseja listar. |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar contas na OU de "Produção")

```json
{
  "ParentId": "ou-a1b2-12345678"
}
```

---

## 6. ListTagsForResource

Lista as tags anexadas a um recurso do Organizations (raiz, OU ou conta). Tags em OUs ou contas são uma prática recomendada de FinOps para alocação de custos.

### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição Detalhada e Didática |
| :--- | :--- | :--- | :--- |
| `ResourceId` | String | **Sim** | O ID do recurso (ID da conta, ID da OU ou ID da raiz) cujas tags você deseja listar. |
| `NextToken` | String | Não | Token de paginação. |

### Exemplo de Requisição (Listar tags da conta 123456789012)

```json
{
  "ResourceId": "123456789012"
}
```
