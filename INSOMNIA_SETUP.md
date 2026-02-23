# Guia de Configuração do Insomnia para o Projeto AWS FinOps

## 1. Introdução

Este documento descreve como importar e configurar a coleção Insomnia fornecida para interagir com as APIs da AWS. O Insomnia é uma ferramenta poderosa para testes de API que simplifica o processo de autenticação e execução de requisições.

## 2. Pré-requisitos

- **Insomnia**: Versão 8.0 ou superior. Faça o download em [insomnia.rest](https://insomnia.rest/).
- **Credenciais AWS**: Um par de `Access Key ID` e `Secret Access Key` com as permissões necessárias. Consulte o arquivo `IAM_POLICY.json` para um exemplo de política de leitura.
- **ID da Conta AWS**: O ID de 12 dígitos da sua conta AWS.

## 3. Importação da Coleção

1.  Abra o aplicativo Insomnia.
2.  Clique no ícone de `+` (Create) no painel esquerdo e selecione **Import**.
3.  Na aba **From File**, clique em **Click to browse** e selecione o arquivo `insomnia_collection.json` deste repositório.
4.  O Insomnia detectará o conteúdo e mostrará os itens a serem importados. Clique em **Import**.

Após a importação, você verá um novo Workspace chamado **AWS FinOps APIs** com todas as requisições organizadas em pastas por serviço.

## 4. Configuração do Ambiente

A coleção utiliza variáveis de ambiente para gerenciar suas credenciais de forma segura. Nunca insira suas chaves diretamente nas requisições.

1.  No canto superior esquerdo, clique no seletor de ambiente (provavelmente estará como "No Environment") e selecione **Manage Environments**.
2.  Você verá um ambiente chamado **AWS Credentials**. Clique nele para editar.
3.  Preencha os valores para as seguintes variáveis:

| Variável | Descrição | Exemplo |
| :--- | :--- | :--- |
| `aws_access_key_id` | Sua AWS Access Key ID. | `AKIAIOSFODNN7EXAMPLE` |
| `aws_secret_access_key` | Sua AWS Secret Access Key. | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |
| `aws_region` | A região padrão para as requisições. | `us-east-1` |
| `aws_account_id` | O ID da sua conta AWS. | `123456789012` |

4.  O ambiente deve ficar parecido com isto (em formato JSON):

```json
{
  "aws_access_key_id": "SUA_ACCESS_KEY_ID",
  "aws_secret_access_key": "SUA_SECRET_ACCESS_KEY",
  "aws_region": "us-east-1",
  "aws_account_id": "SEU_ACCOUNT_ID"
}
```

5.  Feche a janela de gerenciamento de ambientes. O ambiente **AWS Credentials** agora estará ativo.

## 5. Executando sua Primeira Requisição

1.  Navegue até a pasta **1. Cost Management** -> **1.1. Cost Explorer**.
2.  Selecione a requisição **GetCostAndUsage - Custo Mensal por Serviço**.
3.  No painel central, você pode inspecionar o corpo (Body) da requisição. Note que ele já está pré-configurado para buscar o custo do último mês, agrupado por serviço.
4.  Clique no botão **Send** no canto superior direito.

Se tudo estiver configurado corretamente, você verá uma resposta `200 OK` no painel de resultados com os dados de custo e uso da sua conta.

## 6. Entendendo a Autenticação AWS IAM

Cada requisição na coleção está configurada para usar a autenticação **AWS IAM v4** nativa do Insomnia.

- Na aba **Auth** de qualquer requisição, você verá que o tipo de autenticação está definido como `AWS IAM`.
- Os campos `Access Key ID` e `Secret Access Key` estão preenchidos com as variáveis de ambiente que você configurou (`{{ aws_access_key_id }}` e `{{ aws_secret_access_key }}`).
- O campo `Region` também usa a variável `{{ aws_region }}`.
- O campo `Service Name` está preenchido com o identificador do serviço para a assinatura da requisição (ex: `ce` para Cost Explorer, `budgets` para Budgets).

Essa configuração garante que cada requisição seja assinada corretamente antes de ser enviada para a AWS, e você não precisa se preocupar com os detalhes complexos do processo de assinatura Signature v4.

## 7. Troubleshooting

| Erro | Causa Provável | Solução |
| :--- | :--- | :--- |
| `403 Forbidden` / `InvalidSignatureException` | Credenciais incorretas ou relógio do sistema dessincronizado. | 1. Verifique se suas chaves de acesso estão corretas no ambiente. 2. Certifique-se de que o relógio do seu computador está sincronizado com um servidor de tempo (NTP). |
| `403 Forbidden` / `AccessDeniedException` | O perfil IAM não tem permissão para executar a ação. | 1. Verifique a política IAM anexada ao seu usuário/role. 2. Use o arquivo `IAM_POLICY.json` como referência para as permissões necessárias. |
| `400 Bad Request` / `ValidationException` | Os parâmetros enviados no corpo da requisição são inválidos. | 1. Leia a mensagem de erro na resposta, ela geralmente indica qual campo está incorreto. 2. Consulte a documentação da API da AWS para a ação específica para entender os parâmetros esperados. |
| `400 Bad Request` / `UnrecognizedClientException` | O `X-Amz-Target` no header está incorreto ou não corresponde ao serviço. | Verifique o header `X-Amz-Target` na aba "Headers" da requisição e compare com a documentação da API. |

## 8. Dicas Avançadas

- **Modificar Regiões**: Para executar uma requisição em uma região diferente da padrão, você pode sobrescrever a variável de ambiente. Na aba **Auth** da requisição, substitua `{{ aws_region }}` pelo nome da região desejada (ex: `sa-east-1`).
- **Criar Ambientes Múltiplos**: Se você gerencia várias contas AWS, pode criar diferentes ambientes no Insomnia, um para cada conta, e alternar entre eles facilmente.
- **Encadeamento de Requisições**: Use a funcionalidade "Response Tag" do Insomnia para extrair um valor da resposta de uma requisição e usá-lo em outra. Por exemplo, você pode listar contas com a API do Organizations e depois usar o ID de uma conta para buscar seus custos específicos.
