# Guia Detalhado: AWS Organizations API

## Visão Geral

O AWS Organizations é fundamental para ambientes multi-conta, permitindo gerenciar contas de forma centralizada e entender a estrutura organizacional para alocação de custos.

| Atributo | Valor |
| :--- | :--- |
| **Endpoint** | `https://organizations.us-east-1.amazonaws.com/` |
| **Protocolo** | JSON-RPC (POST) |
| **Content-Type** | `application/x-amz-json-1.1` |
| **X-Amz-Target Prefix** | `AWSOrganizationsV20161128` |
| **Service Name (IAM)** | `organizations` |

---

## Ações da API

### 1. ListAccounts

Lista todas as contas da organização.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados por página. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "MaxResults": 20
}
```

---

### 2. DescribeAccount

Retorna detalhes de uma conta específica.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `AccountId` | String | Sim | O ID da conta a ser descrita. |

#### Exemplo de Requisição

```json
{
  "AccountId": "123456789012"
}
```

---

### 3. ListRoots

Lista as raízes (roots) da organização. Geralmente, há apenas uma.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{}
```

---

### 4. ListOrganizationalUnitsForParent

Lista as Unidades Organizacionais (OUs) filhas de um parent (root ou outra OU).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ParentId` | String | Sim | O ID do parent (root ou OU). |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ParentId": "r-a1b2"
}
```

---

### 5. ListAccountsForParent

Lista as contas diretamente dentro de um parent (root ou OU).

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ParentId` | String | Sim | O ID do parent (root ou OU). |
| `MaxResults` | Integer | Não | Número máximo de resultados. |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ParentId": "ou-a1b2-12345678"
}
```

---

### 6. ListTagsForResource

Lista as tags de uma conta, OU ou root.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ResourceId` | String | Sim | O ID do recurso (conta, OU, root). |
| `NextToken` | String | Não | Token para paginação. |

#### Exemplo de Requisição

```json
{
  "ResourceId": "123456789012"
}
```

---

### 7. TagResource

Adiciona uma ou mais tags a um recurso.

#### Parâmetros de Entrada

| Parâmetro | Tipo | Obrigatório | Descrição |
| :--- | :--- | :--- | :--- |
| `ResourceId` | String | Sim | O ID do recurso a ser tagueado. |
| `Tags` | Array | Sim | Lista de objetos `Tag` com `Key` e `Value`. |

#### Exemplo de Requisição

```json
{
  "ResourceId": "123456789012",
  "Tags": [
    {
      "Key": "CostCenter",
      "Value": "CC-123"
    },
    {
      "Key": "Environment",
      "Value": "Production"
    }
  ]
}
```
